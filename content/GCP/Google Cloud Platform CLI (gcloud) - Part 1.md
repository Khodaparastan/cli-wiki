

## Table of Contents
1. [Installation & Setup](#installation--setup)
2. [Basic Configuration](#basic-configuration)
3. [Project Management](#project-management)
4. [IAM & Security](#iam--security)
5. [Compute Services](#compute-services)
6. [Storage Services](#storage-services)
7. [Database Services](#database-services)
8. [Networking](#networking)
9. [Kubernetes (GKE)](#kubernetes-gke)
10. [BigQuery](#bigquery)
11. [Cloud Run](#cloud-run)
12. [Monitoring & Logging](#monitoring--logging)
13. [Best Practices](#best-practices)

Let me start with the first few sections, and I'll continue with the rest.

## Installation & Setup

### Installation
```bash
# Debian/Ubuntu
echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
sudo apt-get install apt-transport-https ca-certificates gnupg
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -
sudo apt-get update && sudo apt-get install google-cloud-sdk

# macOS
brew install --cask google-cloud-sdk

# Windows
# Download installer from https://cloud.google.com/sdk/docs/install
```

### Initial Setup
```bash
# Initialize gcloud
gcloud init

# Auth login
gcloud auth login

# Application default credentials
gcloud auth application-default login

# Service account auth
gcloud auth activate-service-account --key-file=key.json

# List active account
gcloud auth list
```

## Basic Configuration

### Configuration Management
```bash
# List all properties
gcloud config list

# Set default project
gcloud config set project PROJECT_ID

# Set default region
gcloud config set compute/region us-central1

# Set default zone
gcloud config set compute/zone us-central1-a

# Create configuration
gcloud config configurations create CONFIGURATION_NAME

# Switch configurations
gcloud config configurations activate CONFIGURATION_NAME
```

### Project Management
```bash
# List projects
gcloud projects list

# Create new project
gcloud projects create PROJECT_ID --name="Project Name"

# Set project
gcloud config set project PROJECT_ID

# Get project info
gcloud projects describe PROJECT_ID

# Delete project
gcloud projects delete PROJECT_ID
```

### Environment Setup
```bash
# Install components
gcloud components install COMPONENT_ID

# Update components
gcloud components update

# List available components
gcloud components list
```

## IAM & Security

### Service Accounts
```bash
# Create service account
gcloud iam service-accounts create SA_NAME \
    --description="Description" \
    --display-name="Display Name"

# List service accounts
gcloud iam service-accounts list

# Create key
gcloud iam service-accounts keys create key.json \
    --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com

# Add role to service account
gcloud projects add-iam-policy-binding PROJECT_ID \
    --member="serviceAccount:SA_NAME@PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/role.name"
```

### IAM Roles
```bash
# List roles
gcloud iam roles list

# Create custom role
gcloud iam roles create ROLE_ID \
    --project=PROJECT_ID \
    --title="Role Title" \
    --description="Description" \
    --permissions=permission1,permission2

# Update role
gcloud iam roles update ROLE_ID \
    --project=PROJECT_ID \
    --add-permissions=permission3

# Get IAM policy
gcloud projects get-iam-policy PROJECT_ID

# Set IAM policy
gcloud projects set-iam-policy PROJECT_ID policy.yaml
```

### Security
```bash
# Enable services
gcloud services enable SERVICE_NAME

# List enabled services
gcloud services list

# Disable services
gcloud services disable SERVICE_NAME

# Create SSL certificate
gcloud compute ssl-certificates create CERT_NAME \
    --domains=domain.com
```

## Compute Services

### Compute Engine
```bash
# Create instance
gcloud compute instances create INSTANCE_NAME \
    --machine-type=e2-medium \
    --image-family=debian-10 \
    --image-project=debian-cloud \
    --boot-disk-size=10GB

# List instances
gcloud compute instances list

# SSH into instance
gcloud compute ssh INSTANCE_NAME

# Start instance
gcloud compute instances start INSTANCE_NAME

# Stop instance
gcloud compute instances stop INSTANCE_NAME

# Delete instance
gcloud compute instances delete INSTANCE_NAME

# Create instance template
gcloud compute instance-templates create TEMPLATE_NAME \
    --machine-type=e2-medium \
    --image-family=debian-10 \
    --image-project=debian-cloud

# Create instance group
gcloud compute instance-groups managed create GROUP_NAME \
    --template=TEMPLATE_NAME \
    --size=2 \
    --zone=ZONE
```

### Disk Management
```bash
# Create disk
gcloud compute disks create DISK_NAME \
    --size=100GB \
    --type=pd-ssd

# Attach disk
gcloud compute instances attach-disk INSTANCE_NAME \
    --disk=DISK_NAME

# List disks
gcloud compute disks list

# Snapshot disk
gcloud compute disks snapshot DISK_NAME \
    --snapshot-names=SNAPSHOT_NAME

# Delete disk
gcloud compute disks delete DISK_NAME
```

[Continue to Part 2...]

Would you like me to continue with the next sections? I'll cover Storage Services, Database Services, Networking, Kubernetes (GKE), BigQuery, Cloud Run, and more in detail.