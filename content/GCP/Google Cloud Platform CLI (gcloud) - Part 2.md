
## Storage Services

### Cloud Storage
```bash
# Create bucket
gsutil mb -l us-central1 gs://BUCKET_NAME

# List buckets
gsutil ls

# Copy files
gsutil cp LOCAL_FILE gs://BUCKET_NAME/
gsutil cp -r LOCAL_DIR gs://BUCKET_NAME/

# Sync directories
gsutil -m rsync -r LOCAL_DIR gs://BUCKET_NAME/

# Set bucket policy
gsutil iam ch user:user@domain.com:objectViewer gs://BUCKET_NAME

# Enable versioning
gsutil versioning set on gs://BUCKET_NAME

# Configure lifecycle
gsutil lifecycle set lifecycle.json gs://BUCKET_NAME

# Set CORS
gsutil cors set cors.json gs://BUCKET_NAME

# Enable bucket logging
gsutil logging set on -b gs://LOG_BUCKET gs://BUCKET_NAME
```

### Object Management
```bash
# List objects
gsutil ls gs://BUCKET_NAME/**

# Get object metadata
gsutil stat gs://BUCKET_NAME/OBJECT_NAME

# Set object ACL
gsutil acl set private gs://BUCKET_NAME/OBJECT_NAME

# Make object public
gsutil acl ch -u AllUsers:R gs://BUCKET_NAME/OBJECT_NAME

# Set object metadata
gsutil setmeta -h "Content-Type:application/json" gs://BUCKET_NAME/OBJECT_NAME
```

## Database Services

### Cloud SQL
```bash
# Create instance
gcloud sql instances create INSTANCE_NAME \
    --database-version=MYSQL_8_0 \
    --tier=db-f1-micro \
    --region=us-central1

# List instances
gcloud sql instances list

# Create database
gcloud sql databases create DATABASE_NAME \
    --instance=INSTANCE_NAME

# Create user
gcloud sql users create USERNAME \
    --instance=INSTANCE_NAME \
    --password=PASSWORD

# Connect to instance
gcloud sql connect INSTANCE_NAME --user=USERNAME

# Import database
gcloud sql import sql INSTANCE_NAME \
    gs://BUCKET_NAME/backup.sql \
    --database=DATABASE_NAME

# Export database
gcloud sql export sql INSTANCE_NAME \
    gs://BUCKET_NAME/backup.sql \
    --database=DATABASE_NAME
```

### Cloud Spanner
```bash
# Create instance
gcloud spanner instances create INSTANCE_NAME \
    --config=regional-us-central1 \
    --description="Description" \
    --nodes=1

# Create database
gcloud spanner databases create DATABASE_NAME \
    --instance=INSTANCE_NAME

# Execute DDL
gcloud spanner databases ddl update DATABASE_NAME \
    --instance=INSTANCE_NAME \
    --ddl="CREATE TABLE table_name (...);"

# Query database
gcloud spanner databases execute-sql DATABASE_NAME \
    --instance=INSTANCE_NAME \
    --sql="SELECT * FROM table_name"
```

## Networking

### VPC Networks
```bash
# Create VPC
gcloud compute networks create VPC_NAME \
    --subnet-mode=custom

# Create subnet
gcloud compute networks subnets create SUBNET_NAME \
    --network=VPC_NAME \
    --region=us-central1 \
    --range=10.0.0.0/24

# List networks
gcloud compute networks list

# Create firewall rule
gcloud compute firewall-rules create RULE_NAME \
    --network=VPC_NAME \
    --allow=tcp:80,tcp:443 \
    --source-ranges=0.0.0.0/0

# Create VPC peering
gcloud compute networks peerings create PEERING_NAME \
    --network=VPC_NAME \
    --peer-project=PEER_PROJECT \
    --peer-network=PEER_VPC
```

### Load Balancing
```bash
# Create health check
gcloud compute health-checks create http HEALTH_CHECK_NAME \
    --port=80

# Create backend service
gcloud compute backend-services create BACKEND_NAME \
    --protocol=HTTP \
    --health-checks=HEALTH_CHECK_NAME \
    --global

# Create URL map
gcloud compute url-maps create URL_MAP_NAME \
    --default-service=BACKEND_NAME

# Create target proxy
gcloud compute target-http-proxies create PROXY_NAME \
    --url-map=URL_MAP_NAME

# Create forwarding rule
gcloud compute forwarding-rules create RULE_NAME \
    --global \
    --target-http-proxy=PROXY_NAME \
    --ports=80
```

## Kubernetes (GKE)

### Cluster Management
```bash
# Create cluster
gcloud container clusters create CLUSTER_NAME \
    --num-nodes=3 \
    --machine-type=e2-medium \
    --zone=us-central1-a

# Get credentials
gcloud container clusters get-credentials CLUSTER_NAME \
    --zone=us-central1-a

# List clusters
gcloud container clusters list

# Resize cluster
gcloud container clusters resize CLUSTER_NAME \
    --num-nodes=5 \
    --zone=us-central1-a

# Update cluster
gcloud container clusters upgrade CLUSTER_NAME \
    --master --cluster-version=1.24 \
    --zone=us-central1-a
```

### Node Pool Management
```bash
# Create node pool
gcloud container node-pools create POOL_NAME \
    --cluster=CLUSTER_NAME \
    --machine-type=e2-medium \
    --num-nodes=3

# List node pools
gcloud container node-pools list \
    --cluster=CLUSTER_NAME

# Update node pool
gcloud container node-pools update POOL_NAME \
    --cluster=CLUSTER_NAME \
    --enable-autoscaling \
    --min-nodes=1 \
    --max-nodes=5

# Delete node pool
gcloud container node-pools delete POOL_NAME \
    --cluster=CLUSTER_NAME
```

