
## Enable APIs

```
gcloud config list project
gcloud config set project [YOUR-PROJECT-NAME]
projectname=YOUR-PROJECT-NAME
echo $projectname
```

Enable all necessary services

```
gcloud services enable compute.googleapis.com
gcloud services enable servicedirectory.googleapis.com
gcloud services enable dns.googleapis.com
```