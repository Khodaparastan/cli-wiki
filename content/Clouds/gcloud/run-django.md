# Deploy wagtail

## Enable the Cloud APIs

```bash
gcloud services enable \
  run.googleapis.com \
  sql-component.googleapis.com \
  sqladmin.googleapis.com \
  compute.googleapis.com \
  cloudbuild.googleapis.com \
  secretmanager.googleapis.com \
  artifactregistry.googleapis.com
```

## Create a template project

You'll use the default Wagtail project template as your sample Wagtail project. To do this, you'll temporarily install Wagtail to generate the template.

To create this template project, use Cloud Shell to create a new directory named `wagtail-cloudrun` and navigate to it:

```
mkdir ~/wagtail-cloudrun
cd ~/wagtail-cloudrun
```

Then, install Wagtail into a temporary virtual environment:

```
virtualenv venv
source venv/bin/activate
pip install wagtail
```

Then, create a new template project in the current folder:

```
wagtail start myproject .
```

You'll now have a template Wagtail project in the current folder:

```
ls -F
```

```
Dockerfile  home/  manage.py*  myproject/  requirements.txt  search/ venv/
```

You can now exit and remove your temporary virtual environment:

```
deactivate
rm -rf venv
```

From here, Wagtail will be called within the container.

## Create the backing services

```bash
PROJECT_ID=$(gcloud config get-value core/project)
REGION=us-central1
```

## Create a service account

```bash

gcloud iam service-accounts create cloudrun-serviceaccount

SERVICE_ACCOUNT=$(gcloud iam service-accounts list \
    --filter cloudrun-serviceaccount --format "value(email)")
```

## **Create an Artifact Registry**

```bash
gcloud artifacts repositories create containers --repository-format docker --location $REGION

ARTIFACT_REGISTRY=${REGION}-docker.pkg.dev/${PROJECT_ID}/containers
```

## **Create the database**

```bash
gcloud sql instances create myinstance --project $PROJECT_ID \
  --database-version POSTGRES_14 --tier db-f1-micro --region $REGION

# In that instance, create a database:

gcloud sql databases create mydatabase --instance myinstance
```

In that same instance, create a user:

```bash
DJPASS="$(cat /dev/urandom | LC_ALL=C tr -dc 'a-zA-Z0-9' | fold -w 30 | head -n 1)"
gcloud sql users create djuser --instance myinstance --password $DJPASS
```

Grant the service account permission to connect to the instance:

```
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member serviceAccount:${SERVICE_ACCOUNT} \
    --role roles/cloudsql.client
```

## **Create the storage bucket**

Create a Cloud Storage bucket (noting the name must be globally unique):

```
GS_BUCKET_NAME=${PROJECT_ID}-media
gcloud storage buckets create gs://${GS_BUCKET_NAME} --location ${REGION} 
```

Grant permissions for the service account to administer the bucket:

```
gcloud storage buckets add-iam-policy-binding gs://${GS_BUCKET_NAME} \
	--member serviceAccount:${SERVICE_ACCOUNT} \
	--role roles/storage.admin
```

Since objects stored in the bucket will have a different origin (a bucket URL rather than a Cloud Run URL), you need to configure the  [Cross Origin Resource Sharing (CORS)](https://cloud.google.com/storage/docs/configuring-cors) settings.

Create a new file called `cors.json`, with the following contents:
```

touch cors.json
cloudshell edit cors.json
```

### **cors.json**

```json
[    {      "origin": ["*"],      "responseHeader": ["Content-Type"],      "method": ["GET"],      "maxAgeSeconds": 3600    }]
```

Apply this CORS configuration to the newly created storage bucket:

```
gsutil cors set cors.json gs://$GS_BUCKET_NAME
```

## **Store configuration as secret**

Having set up the backing services, you'll now store these values in a file protected using Secret Manager.

Secret Manager allows you to store, manage, and access secrets as binary blobs or text strings. It works well for storing configuration information such as database passwords, API keys, or TLS certificates needed by an application at runtime.

First, create a file with the values for the  [database connection string](https://github.com/joke2k/django-environ#supported-types), media bucket, a secret key for Django (used for cryptographic signing of sessions and tokens), and to enable debugging:

```
echo DATABASE_URL=\"postgres://djuser:${DJPASS}@//cloudsql/${PROJECT_ID}:${REGION}:myinstance/mydatabase\" > .env

echo GS_BUCKET_NAME=\"${GS_BUCKET_NAME}\" >> .env

echo SECRET_KEY=\"$(cat /dev/urandom | LC_ALL=C tr -dc 'a-zA-Z0-9' | fold -w 50 | head -n 1)\" >> .env

echo DEBUG=True >> .env
```

Then, create a secret called `application_settings`, using that file as the secret:

```
gcloud secrets create application_settings --data-file .env
```

Allow the service account access to this secret:

```
gcloud secrets add-iam-policy-binding application_settings \
  --member serviceAccount:${SERVICE_ACCOUNT} --role roles/secretmanager.secretAccessor
```

Confirm the secret has been created by listing the secrets:

```
gcloud secrets versions list application_settings
```

After confirming the secret has been created, remove the local file:

```
rm .env
```

## Configure settings

Find the generated `base.py` settings file, and rename it to `basesettings.py` in the main `myproject` folder:

mv myproject/settings/base.py myproject/basesettings.py

Using the Cloud Shell web editor, create a new `settings.py` file, with the following code:

touch myproject/settings.py
cloudshell edit myproject/settings.py

### **myproject/settings.py**

```python
import io
import os
from urllib.parse import urlparse

import environ

# Import the original settings from each template
from .basesettings import *

# Load the settings from the environment variable
env = environ.Env()
env.read_env(io.StringIO(os.environ.get("APPLICATION_SETTINGS", None)))

# Setting this value from django-environ
SECRET_KEY = env("SECRET_KEY")

# Ensure myproject is added to the installed applications
if "myproject" not in INSTALLED_APPS:
    INSTALLED_APPS.append("myproject")

# If defined, add service URLs to Django security settings
CLOUDRUN_SERVICE_URLS = env("CLOUDRUN_SERVICE_URLS", default=None)
if CLOUDRUN_SERVICE_URLS:
    CSRF_TRUSTED_ORIGINS = env("CLOUDRUN_SERVICE_URLS").split(",")
    # Remove the scheme from URLs for ALLOWED_HOSTS
    ALLOWED_HOSTS = [urlparse(url).netloc for url in CSRF_TRUSTED_ORIGINS]
else:
    ALLOWED_HOSTS = ["*"]

# Default false. True allows default landing pages to be visible
DEBUG = env("DEBUG", default=False)

# Set this value from django-environ
DATABASES = {"default": env.db()}

# Change database settings if using the Cloud SQL Auth Proxy
if os.getenv("USE_CLOUD_SQL_AUTH_PROXY", None):
    DATABASES["default"]["HOST"] = "127.0.0.1"
    DATABASES["default"]["PORT"] = 5432

# Define static storage via django-storages[google]
GS_BUCKET_NAME = env("GS_BUCKET_NAME")
STATICFILES_DIRS = []
GS_DEFAULT_ACL = "publicRead"
STORAGES = {
    "default": {
        "BACKEND": "storages.backends.gcloud.GoogleCloudStorage",
    },
    "staticfiles": {
        "BACKEND": "storages.backends.gcloud.GoogleCloudStorage",
    },
}
```

Take the time to read the commentary added about each configuration.

Note that you may see linting errors on this file. This is expected. Cloud Shell does not have context of the requirements for this project, and thus may report invalid imports, and unused imports.

Then, remove the old settings folder.

rm -rf myproject/settings/

You will then have two settings files: one from Wagtail, and one you just created that builds from these settings:

ls myproject/*settings*

myproject/basesettings.py  myproject/settings.py

Finally, open the `manage.py` settings file, and update the configuration to tell Wagtail to point to the main `settings.py`file.

cloudshell edit manage.py

### manage.py line (before)

```
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "myproject.settings.dev")
```

### **manage.py line (after)**

```
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "myproject.settings")
```

Make the same configuration change for the `myproject/wsgi.py` file:

cloudshell edit myproject/wsgi.py

### **myproject/wsgi.py line (before)**

```
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "myproject.settings.dev")
```

### **myproject/wsgi.py line (after)**

```
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "myproject.settings")
```

Remove the automatically created Dockerfile:

rm Dockerfile

## **Python dependencies**

Locate the `requirements.txt` file, and append the following packages:

cloudshell edit requirements.txt

### **requirements.txt (append)**

gunicorn
psycopg2-binary
django-storages[google]
django-environ

## **Define your application image**

Cloud Run will run any container as long as it conforms to the  [Cloud Run Container Contract](https://cloud.google.com/run/docs/reference/container-contract). This tutorial opts to omit a `Dockerfile`, but instead use  [Cloud Native Buildpacks](https://cloud.google.com/build/docs/building/build-containers#use-buildpacks). Buildpacks assist in building containers for common languages, including Python.

This tutorial opts to  [customize the `Procfile`](https://cloud.google.com/docs/buildpacks/python#application_entrypoint) used to start the web application.

To containerize the template project, first create a new file named `Procfile` in the top level of your project (in the same directory as  `manage.py`), and copy the following content:

touch Procfile
cloudshell edit Procfile

### **Procfile**

```
web: gunicorn --bind 0.0.0.0:$PORT --workers 1 --threads 8 --timeout 0 myproject.wsgi:application
```



## Create Cloud Run jobs
Now that the image exists, you can create Cloud Run jobs using it.

These jobs use the image previously built, but use different command values. These map to the values in the Procfile.

Create a job for the migration:

```
gcloud run jobs create migrate \
  --region $REGION \
  --image ${ARTIFACT_REGISTRY}/myimage \
  --set-cloudsql-instances ${PROJECT_ID}:${REGION}:myinstance \
  --set-secrets APPLICATION_SETTINGS=application_settings:latest \
  --service-account $SERVICE_ACCOUNT \
  --command migrate
```
Create a job for the user creation:

```

gcloud run jobs create createuser \
  --region $REGION \
  --image ${ARTIFACT_REGISTRY}/myimage \
  --set-cloudsql-instances ${PROJECT_ID}:${REGION}:myinstance \
  --set-secrets APPLICATION_SETTINGS=application_settings:latest \
  --set-secrets DJANGO_SUPERUSER_PASSWORD=django_superuser_password:latest \
  --service-account $SERVICE_ACCOUNT \
  --command createuser
```

## Execute Cloud Run jobs

With the job configurations in place, run the migrations:

```
gcloud run jobs execute migrate --region $REGION --wait
```

Ensure this command output says the execution "successfully completed".

You will run this command later when you make updates to your application.

With the database setup, create the user using the job:

```
gcloud run jobs execute createuser --region $REGION --wait
```

Ensure this command output says the execution "successfully completed".

You will not have to run this command again.


## Deploy to Cloud Run

With the backing services created and populated, you can now create the Cloud Run service to access them.

The initial deployment of your containerized application to Cloud Run is created using the following command:


```
gcloud run deploy wagtail-cloudrun \
  --region $REGION \
  --image ${ARTIFACT_REGISTRY}/myimage \
  --set-cloudsql-instances ${PROJECT_ID}:${REGION}:myinstance \
  --set-secrets APPLICATION_SETTINGS=application_settings:latest \
  --service-account $SERVICE_ACCOUNT \
  --allow-unauthenticated
```

Wait a few moments until the deployment is complete. On success, the command line displays the service URL:


```
Service [wagtail-cloudrun] revision [wagtail-cloudrun-00001-...] has been deployed and is serving 100 percent of traffic.
Service URL: https://wagtail-cloudrun-...run.app
```


### Retrieve your service URL:


```
CLOUDRUN_SERVICE_URLS=$(gcloud run services describe wagtail-cloudrun \
  --region $REGION  \
  --format "value(metadata.annotations[\"run.googleapis.com/urls\"])" | tr -d '"[]')
echo $CLOUDRUN_SERVICE_URLS
```

Set this value as an environment variable on your Cloud Run service:

```
gcloud run services update wagtail-cloudrun \
  --region $REGION \
  --update-env-vars "^##^CLOUDRUN_SERVICE_URLS=$CLOUDRUN_SERVICE_URLS"
```

Logging into the Django Admin
To access the Django admin interface, append /admin to your service URL.

Now log in with the username "admin" and retrieve your password using the following command:

```
gcloud secrets versions access latest --secret django_superuser_password && echo ""
```