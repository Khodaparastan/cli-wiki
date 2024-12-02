
You can list the active account name:

```bash
gcloud auth list
```

To set the active account, run:

```bash
gcloud config list project
```

You can list the project ID with this command:

```bash
gcloud config list project
```

Set your region and zone:

```bash
gcloud config set compute/zone "ZONE"
export ZONE=$(gcloud config get compute/zone)

gcloud config set compute/region "REGION"
export REGION=$(gcloud config get compute/region)
```

Enable the [Compute Engine API](https://console.cloud.google.com/flows/enableapi?apiid=compute) by executing the following:

```bash
gcloud services enable compute.googleapis.com
```

 Create Cloud Storage bucket

> [!NOTE]
> Use of the `$DEVSHELL_PROJECT_ID` environment variable within Cloud Shell is to help ensure the names of objects are unique. Since all Project IDs within Google Cloud must be unique, appending the Project ID should make other names unique as well.

```bash
gsutil mb gs://fancy-store-$DEVSHELL_PROJECT_ID
```

Copy data from bucket to local dir

```bash
gsutil -m cp -r gs://fancy-store-[DEVSHELL_PROJECT_ID]/dir/* ./dir
```

Create a vm with starting script from bucket

```bash
export VMNAME=backend
export VMTYPE=e2-standard-2
export VMTAG=backend

gcloud compute instances create $VMNMAE \
    --zone=$ZONE \
    --machine-type=$VMTYPE \
    --tags=backend \
   --metadata=startup-script-url=https://storage.googleapis.com/fancy-store-$DEVSHELL_PROJECT_ID/startup-script.sh
```

list instances of vm in the project

```bash
gcloud compute instances list
```

Create firewall rules to allow access to port 8080 

```bash
gcloud compute firewall-rules create fw-fe \
    --allow tcp:8080 \
    --target-tags=$VMTAG
```

stop vm

```bash
gcloud compute instances stop $VMNAME --zone=$ZONE
```

Create managed instance groups

```bash
export INSGROUP=fancy-vm
gcloud compute instance-templates create INSGROUP \
    --source-instance-zone=$ZONE \
    --source-instance=$VMNAME
```

list template instances

```bash
gcloud compute instance-templates list
```

delete vm that we made  template from

```bash
gcloud compute instances delete $VMNAME --zone=$ZONE
```

Create managed instance group

```bash
export $MINSGROUP=fancy-be-mig

gcloud compute instance-groups managed create $MINSGROUP \
    --zone=$ZONE \
    --base-instance-name $INSGROUP \
    --size 2 \
    --template $INSGROUP
```

Since these are non-standard ports, you specify named ports to identify these. Named ports are key:value pair metadata representing the service name and the port that it's running on. Named ports can be assigned to an instance group, which indicates that the service is available on all instances in the group. This information is used by the HTTP Load Balancing service that will be configured later.

```bash
gcloud compute instance-groups set-named-ports $MINSGROUP \
    --zone=$ZONE \
    --named-ports orders:8081,products:8082
```

Create a health check that repairs the instance if it returns "unhealthy" 3 consecutive times:

```bash
gcloud compute health-checks create http fancy-fe-hc \
    --port 8080 \
    --check-interval 30s \
    --healthy-threshold 1 \
    --timeout 10s \
    --unhealthy-threshold 3
```

```bash

```