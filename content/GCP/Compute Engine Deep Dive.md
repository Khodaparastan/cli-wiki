
## Instance Management

### Basic Instance Operations
```bash
# Create instance with detailed specifications
gcloud compute instances create INSTANCE_NAME \
    --machine-type=n2-standard-2 \
    --zone=us-central1-a \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --boot-disk-size=50GB \
    --boot-disk-type=pd-ssd \
    --network=default \
    --subnet=default \
    --network-tier=PREMIUM \
    --maintenance-policy=MIGRATE \
    --service-account=SA_EMAIL \
    --scopes=https://www.googleapis.com/auth/cloud-platform \
    --tags=http-server,https-server \
    --metadata=startup-script='#!/bin/bash
        apt-get update
        apt-get install -y nginx'

# Create instance from template with overrides
gcloud compute instances create-with-container INSTANCE_NAME \
    --container-image=gcr.io/PROJECT_ID/IMAGE \
    --machine-type=n2-standard-2 \
    --boot-disk-size=50GB \
    --container-env=KEY1=VALUE1,KEY2=VALUE2 \
    --container-mount-host-path=mount-path=/data,host-path=/mnt/data \
    --container-restart-policy=always
```

### Advanced Instance Configuration
```bash
# Create instance with multiple disks
gcloud compute instances create INSTANCE_NAME \
    --machine-type=n2-standard-2 \
    --boot-disk-size=50GB \
    --boot-disk-type=pd-ssd \
    --create-disk=name=data-disk-1,size=200GB,type=pd-ssd,auto-delete=no \
    --create-disk=name=data-disk-2,size=500GB,type=pd-standard \
    --metadata=serial-port-enable=true

# Create instance with GPU
gcloud compute instances create INSTANCE_NAME \
    --machine-type=n1-standard-4 \
    --accelerator=type=nvidia-tesla-t4,count=1 \
    --maintenance-policy=TERMINATE \
    --metadata=install-nvidia-driver=True

# Create preemptible instance
gcloud compute instances create INSTANCE_NAME \
    --machine-type=n2-standard-2 \
    --preemptible \
    --maintenance-policy=TERMINATE
```

### Networking Configuration
```bash
# Create instance with multiple network interfaces
gcloud compute instances create INSTANCE_NAME \
    --machine-type=n2-standard-2 \
    --network-interface=network=default,subnet=default \
    --network-interface=network=vpc-2,subnet=subnet-2,no-address \
    --private-network-ip=10.0.0.2

# Configure instance with static IP
gcloud compute addresses create STATIC_IP_NAME \
    --region=us-central1

gcloud compute instances create INSTANCE_NAME \
    --machine-type=n2-standard-2 \
    --address=STATIC_IP_NAME

# Configure instance with alias IP ranges
gcloud compute instances create INSTANCE_NAME \
    --machine-type=n2-standard-2 \
    --network-interface=network=default,subnet=default,aliases=/24
```

### Instance Groups and Templates
```bash
# Create instance template
gcloud compute instance-templates create TEMPLATE_NAME \
    --machine-type=n2-standard-2 \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --boot-disk-size=50GB \
    --metadata=startup-script='#!/bin/bash
        apt-get update
        apt-get install -y nginx' \
    --tags=http-server,https-server

# Create managed instance group
gcloud compute instance-groups managed create GROUP_NAME \
    --template=TEMPLATE_NAME \
    --size=3 \
    --zone=us-central1-a \
    --health-check=HC_NAME \
    --initial-delay=300

# Configure autoscaling
gcloud compute instance-groups managed set-autoscaling GROUP_NAME \
    --zone=us-central1-a \
    --max-num-replicas=10 \
    --min-num-replicas=2 \
    --target-cpu-utilization=0.7 \
    --cool-down-period=300

# Update instance group
gcloud compute instance-groups managed rolling-action start-update GROUP_NAME \
    --version=template=NEW_TEMPLATE \
    --max-surge=3 \
    --max-unavailable=0
```

### Maintenance and Operations
```bash
# Create snapshot schedule
gcloud compute resource-policies create snapshot-schedule SCHEDULE_NAME \
    --region=us-central1 \
    --max-retention-days=14 \
    --on-source-disk-delete=keep-auto-snapshots \
    --daily-schedule \
    --start-time=04:00

# Attach snapshot schedule to disk
gcloud compute disks add-resource-policies DISK_NAME \
    --resource-policies=SCHEDULE_NAME \
    --zone=us-central1-a

# Configure maintenance window
gcloud compute instances update-container INSTANCE_NAME \
    --container-image=gcr.io/PROJECT_ID/NEW_IMAGE \
    --container-restart-policy=always

# Configure live migration
gcloud compute instances set-scheduling INSTANCE_NAME \
    --maintenance-policy=MIGRATE \
    --max-run-duration=24h
```

### Monitoring and Logging
```bash
# Create custom metric
gcloud monitoring metrics create custom.googleapis.com/METRIC_NAME \
    --metric-kind=gauge \
    --value-type=double \
    --description="Description"

# Create alert policy
gcloud monitoring policies create \
    --condition-filter='metric.type="compute.googleapis.com/instance/cpu/utilization" 
        resource.type="gce_instance" 
        metric.label.instance_name="INSTANCE_NAME"' \
    --condition-threshold-value=0.8 \
    --condition-threshold-duration=300s \
    --notification-channels=CHANNEL_ID \
    --display-name="High CPU Usage Alert"

# Configure logging
gcloud logging sinks create SINK_NAME \
    storage.googleapis.com/BUCKET_NAME \
    --log-filter='resource.type="gce_instance" 
        resource.labels.instance_id="INSTANCE_ID"'
```

[Continue with BigQuery Deep Dive...]

Would you like me to continue with the comprehensive BigQuery deep dive next?