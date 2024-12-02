
## BigQuery

### Dataset Management
```bash
# Create dataset
bq mk --dataset \
    --description="Description" \
    --location=US \
    PROJECT_ID:DATASET_NAME

# List datasets
bq ls

# Update dataset
bq update --description="New Description" \
    PROJECT_ID:DATASET_NAME

# Set access
bq update --set_label project_id:PROJECT_ID \
    --source_project PROJECT_ID \
    DATASET_NAME
```

### Table Operations
```bash
# Create table
bq mk \
    --table \
    --schema name:STRING,age:INTEGER,date:TIMESTAMP \
    PROJECT_ID:DATASET.TABLE_NAME

# Load data from file
bq load \
    --source_format=CSV \
    --skip_leading_rows=1 \
    PROJECT_ID:DATASET.TABLE_NAME \
    gs://BUCKET_NAME/data.csv \
    name:STRING,age:INTEGER,date:TIMESTAMP

# Query table
bq query --use_legacy_sql=false '
    SELECT *
    FROM `PROJECT_ID.DATASET.TABLE_NAME`
    WHERE age > 25
'

# Export table
bq extract \
    PROJECT_ID:DATASET.TABLE_NAME \
    gs://BUCKET_NAME/export.csv
```

### Query Management
```bash
# Run query with parameters
bq query --parameter='age:INT64:25' '
    SELECT *
    FROM `PROJECT_ID.DATASET.TABLE_NAME`
    WHERE age > @age
'

# Save query results
bq query --destination_table=PROJECT_ID:DATASET.RESULTS \
    --replace \
    --use_legacy_sql=false '
    SELECT *
    FROM `PROJECT_ID.DATASET.TABLE_NAME`
'

# Schedule query
bq mk --transfer_config \
    --target_dataset=DATASET \
    --display_name="Scheduled Query" \
    --schedule="every 24 hours" \
    --params='{"query":"SELECT * FROM `PROJECT_ID.DATASET.TABLE_NAME`"}'
```

## Cloud Run

### Service Deployment
```bash
# Deploy service
gcloud run deploy SERVICE_NAME \
    --image gcr.io/PROJECT_ID/IMAGE \
    --platform managed \
    --region us-central1 \
    --allow-unauthenticated

# List services
gcloud run services list

# Describe service
gcloud run services describe SERVICE_NAME

# Update service
gcloud run services update SERVICE_NAME \
    --memory=2Gi \
    --concurrency=80

# Delete service
gcloud run services delete SERVICE_NAME
```

### Traffic Management
```bash
# Split traffic
gcloud run services update-traffic SERVICE_NAME \
    --to-revisions=REVISION_1=50,REVISION_2=50

# Route all traffic to latest
gcloud run services update-traffic SERVICE_NAME \
    --to-latest

# Tag revision
gcloud run revisions tag REVISION \
    --tag=prod \
    --service=SERVICE_NAME
```

### Configuration
```bash
# Set environment variables
gcloud run services update SERVICE_NAME \
    --set-env-vars KEY1=VALUE1,KEY2=VALUE2

# Set secrets
gcloud run services update SERVICE_NAME \
    --set-secrets=SECRET1=projects/PROJECT_ID/secrets/SECRET1:latest

# Set CPU and memory
gcloud run services update SERVICE_NAME \
    --memory=2Gi \
    --cpu=2
```

## Monitoring & Logging

### Cloud Monitoring
```bash
# Create alert policy
gcloud monitoring policies create \
    --notification-channels=CHANNEL_ID \
    --display-name="Alert Policy" \
    --condition-filter="metric.type=\"compute.googleapis.com/instance/cpu/utilization\" resource.type=\"gce_instance\""

# List policies
gcloud monitoring policies list

# Create dashboard
gcloud monitoring dashboards create \
    --config-from-file=dashboard.json

# List metrics
gcloud monitoring metrics list \
    --filter="metric.type = starts_with(\"compute.googleapis.com\")"
```

### Cloud Logging
```bash
# Write log entry
gcloud logging write LOG_NAME "Log message" \
    --payload-type=json \
    --payload='{"key":"value"}'

# Read logs
gcloud logging read "resource.type=gce_instance" \
    --limit=10 \
    --format=json

# Create log sink
gcloud logging sinks create SINK_NAME \
    storage.googleapis.com/BUCKET_NAME \
    --log-filter="resource.type=gce_instance"

# Create log metric
gcloud logging metrics create METRIC_NAME \
    --description="Description" \
    --filter="resource.type=gce_instance"
```

### Error Reporting
```bash
# List error groups
gcloud error-reporting events list

# Get error details
gcloud error-reporting events describe EVENT_ID

# Delete error events
gcloud error-reporting events delete EVENT_ID
```

## Best Practices

### Project Organization
```bash
# Create project structure
for env in prod staging dev; do
    gcloud projects create "${PROJECT_NAME}-${env}" \
        --folder=FOLDER_ID
    
    # Set up basic services
    gcloud services enable compute.googleapis.com \
        --project="${PROJECT_NAME}-${env}"
    
    # Set up IAM
    gcloud projects add-iam-policy-binding "${PROJECT_NAME}-${env}" \
        --member="group:${env}-admins@domain.com" \
        --role="roles/editor"
done
```

### Security Best Practices
```bash
# Audit project permissions
gcloud asset search-all-iam-policies \
    --scope="projects/${PROJECT_ID}"

# Enable OS Login
gcloud compute project-info add-metadata \
    --metadata enable-oslogin=TRUE

# Enable VPC Service Controls
gcloud access-context-manager perimeters create PERIMETER_NAME \
    --title="Perimeter" \
    --resources="projects/${PROJECT_NUMBER}" \
    --restricted-services="storage.googleapis.com"
```

### Automation Scripts

#### Infrastructure Setup
```bash
#!/bin/bash
# Complete infrastructure setup

PROJECT_ID="your-project-id"
REGION="us-central1"
NETWORK_NAME="main-vpc"

# Set up project
gcloud config set project $PROJECT_ID

# Enable required APIs
apis=(
    "compute.googleapis.com"
    "container.googleapis.com"
    "cloudbuild.googleapis.com"
    "cloudrun.googleapis.com"
)

for api in "${apis[@]}"; do
    gcloud services enable $api
done

# Create VPC
gcloud compute networks create $NETWORK_NAME \
    --subnet-mode=custom

# Create subnets
gcloud compute networks subnets create private-subnet \
    --network=$NETWORK_NAME \
    --region=$REGION \
    --range=10.0.0.0/20 \
    --enable-private-ip-google-access

# Set up Cloud NAT
gcloud compute routers create nat-router \
    --network=$NETWORK_NAME \
    --region=$REGION

gcloud compute routers nats create nat-config \
    --router=nat-router \
    --region=$REGION \
    --nat-all-subnet-ip-ranges \
    --auto-allocate-nat-external-ips
```

#### Monitoring Setup
```bash
#!/bin/bash
# Set up monitoring infrastructure

# Create notification channel
CHANNEL_ID=$(gcloud monitoring notification-channels create \
    --display-name="Email Alerts" \
    --type=email \
    --channel-labels=email_address="alerts@domain.com" \
    --format="get(name)")

# Create alert policies
policies=(
    "CPU Usage:compute.googleapis.com/instance/cpu/utilization>0.8"
    "Memory Usage:compute.googleapis.com/instance/memory/utilization>0.8"
    "Disk Usage:compute.googleapis.com/instance/disk/utilization>0.9"
)

for policy in "${policies[@]}"; do
    IFS=':' read -r name metric <<< "$policy"
    
    gcloud monitoring policies create \
        --notification-channels=$CHANNEL_ID \
        --display-name="$name Alert" \
        --condition-filter="metric.type=\"$metric\""
done
```

Remember:
- Always use least privilege access
- Implement proper logging and monitoring
- Use service accounts appropriately
- Keep configurations in version control
- Implement proper backup strategies
- Use cost management tools
- Regular security audits
- Document all configurations

For detailed information, consult the Google Cloud documentation and gcloud command reference (`gcloud help`).

Would you like me to cover any specific aspect in more detail or add more examples for any particular service?