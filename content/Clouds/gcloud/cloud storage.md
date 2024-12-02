
## gcloud  CLI init:

```bash
gcloud init
```

export bucket name (it must be unique)

```bash
export BUCKETNAME=my-awesome-bucket
```

## Create bucket

```bash
gcloud storage buckets create gs://$BUCKETNAME/ --uniform-bucket-level-access
```

## Upload an object into your bucket:

 ```bash
gcloud storage cp Desktop/kitten.png gs://$BUCKETNAME
```

## Download the object from your bucket 

```bash
gcloud storage cp gs://$BUCKETNAME/kitten.png Desktop/kitten2.png
```

## Copy the object to a folder in the bucket

 ```bash
gcloud storage cp gs://$BUCKETNAME/kitten.png gs://$BUCKETNAME/just-a-folder/kitten3.png
```

## List contents of a bucket or folder

```bash
gcloud storage ls gs://$BUCKETNAME
```

## List details for an object

 ```bash
gcloud storage ls gs://$BUCKETNAME/kitten.png --long
```

## Make the objects publicly accessible

- Use the `gcloud storage buckets add-iam-policy-binding` command to grant all users permission to read the images stored in your bucket:

```bash
gcloud storage buckets add-iam-policy-binding gs://$BUCKETNAME --member=allUsers --role=roles/storage.objectViewer
```

- To remove this access, use the command:

```bash
gcloud storage buckets remove-iam-policy-binding gs://$BUCKETNAME --member=allUsers --role=roles/storage.objectViewer
```

## Give someone access to your bucket

Use the `gcloud storage buckets add-iam-policy-binding` command to give a specific email address permission to add objects to your bucket: 

```bash
gcloud storage buckets add-iam-policy-binding gs://$BUCKETNAME --member=user:jane@gmail.com --role=roles/storage.objectCreator
```

 To remove this permission, use the command:

```bash
gcloud storage buckets remove-iam-policy-binding gs://$BUCKETNAME --member=user:jane@gmail.com --role=roles/storage.objectCreator
   
```

## Delete an object

Use the `gcloud storage rm` command to delete one of your images:

```bash
gcloud storage rm gs://$BUCKETNAME/kitten.png
```

## Clean up

Use the `gcloud storage rm` command with the `--recursive` flag to delete the bucket and anything inside of it:

```bash
gcloud storage rm gs://$BUCKETNAME --recursive
```

