
## IAM Role Management

### Basic Role Operations
```bash
# List predefined roles
gcloud iam roles list

# List custom roles
gcloud iam roles list --project=PROJECT_ID

# Get role details
gcloud iam roles describe roles/ROLE_NAME

# Create custom role
gcloud iam roles create ROLE_NAME \
    --project=PROJECT_ID \
    --title="Role Title" \
    --description="Role Description" \
    --stage=GA \
    --permissions=compute.instances.list,compute.instances.get

# Update custom role
gcloud iam roles update ROLE_NAME \
    --project=PROJECT_ID \
    --add-permissions=compute.instances.start,compute.instances.stop \
    --remove-permissions=compute.instances.delete

# Disable/Enable role
gcloud iam roles disable ROLE_NAME --project=PROJECT_ID
gcloud iam roles enable ROLE_NAME --project=PROJECT_ID
```

### Advanced Role Management
```bash
# Create role from YAML
cat > role-definition.yaml << EOF
title: "Custom Admin Role"
description: "Custom role for administrative tasks"
includedPermissions:
- compute.instances.list
- compute.instances.get
- compute.instances.start
- compute.instances.stop
stage: "GA"
EOF

gcloud iam roles create ROLE_NAME \
    --project=PROJECT_ID \
    --file=role-definition.yaml

# Clone existing role
gcloud iam roles copy \
    --source="roles/compute.admin" \
    --destination=ROLE_NAME \
    --dest-project=PROJECT_ID

# Export role definition
gcloud iam roles describe roles/ROLE_NAME \
    --project=PROJECT_ID \
    --format=yaml > role-definition.yaml
```

## Service Account Management

### Service Account Creation and Configuration
```bash
# Create service account
gcloud iam service-accounts create SA_NAME \
    --display-name="Display Name" \
    --description="Service Account Description"

# List service accounts
gcloud iam service-accounts list

# Update service account
gcloud iam service-accounts update SA_NAME@PROJECT_ID.iam.gserviceaccount.com \
    --display-name="New Display Name" \
    --description="Updated Description"

# Create and manage keys
gcloud iam service-accounts keys create key.json \
    --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com \
    --key-file-type=json

# List keys
gcloud iam service-accounts keys list \
    --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com

# Delete key
gcloud iam service-accounts keys delete KEY_ID \
    --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com
```

### Service Account IAM Bindings
```bash
# Add role binding
gcloud projects add-iam-policy-binding PROJECT_ID \
    --member="serviceAccount:SA_NAME@PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/compute.admin"

# Add multiple roles
for role in compute.admin storage.admin bigquery.admin; do
    gcloud projects add-iam-policy-binding PROJECT_ID \
        --member="serviceAccount:SA_NAME@PROJECT_ID.iam.gserviceaccount.com" \
        --role="roles/$role"
done

# Remove role binding
gcloud projects remove-iam-policy-binding PROJECT_ID \
    --member="serviceAccount:SA_NAME@PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/compute.admin"
```

## Organization and Project Administration

### Organization Management
```bash
# List organizations
gcloud organizations list

# Get organization details
gcloud organizations describe ORGANIZATION_ID

# List organization policies
gcloud resource-manager org-policies list \
    --organization=ORGANIZATION_ID

# Set organization policy
gcloud resource-manager org-policies set-policy \
    compute.disableSerialPortAccess \
    --organization=ORGANIZATION_ID \
    --boolean-policy=true

# Create folder
gcloud resource-manager folders create \
    --display-name="Folder Name" \
    --organization=ORGANIZATION_ID

# Move project to folder
gcloud projects move PROJECT_ID \
    --folder=FOLDER_ID
```

### Project Administration
```bash
# Create project with advanced settings
gcloud projects create PROJECT_ID \
    --name="Project Name" \
    --folder=FOLDER_ID \
    --labels=environment=prod,team=infrastructure \
    --set-as-default

# Update project
gcloud projects update PROJECT_ID \
    --name="New Project Name"

# Set project policies
gcloud resource-manager org-policies set-policy \
    compute.vmExternalIpAccess \
    --project=PROJECT_ID \
    --boolean-policy=false

# Enable APIs
gcloud services enable \
    compute.googleapis.com \
    container.googleapis.com \
    cloudfunctions.googleapis.com \
    --project=PROJECT_ID

# List enabled APIs
gcloud services list --enabled \
    --project=PROJECT_ID

# Disable APIs
gcloud services disable SERVICE_NAME \
    --project=PROJECT_ID \
    --force
```

## Security and Compliance

### Audit Configuration
```bash
# Create audit logs sink
gcloud logging sinks create SINK_NAME \
    storage.googleapis.com/BUCKET_NAME \
    --log-filter='resource.type="iam_role" OR 
                  resource.type="service_account" OR 
                  resource.type="project"'

# Update audit logs configuration
gcloud organizations set-iam-policy ORGANIZATION_ID policy.yaml

# Export audit logs
gcloud logging read 'resource.type="iam_role"' \
    --project=PROJECT_ID \
    --format=json > audit_logs.json
```

### Security Scanning
```bash
# Enable Security Command Center
gcloud scc settings enable \
    --organization=ORGANIZATION_ID

# List security findings
gcloud scc findings list \
    --organization=ORGANIZATION_ID \
    --filter="state=ACTIVE" \
    --format=json

# Create notification config
gcloud scc notifications create NOTIFICATION_ID \
    --organization=ORGANIZATION_ID \
    --pubsub-topic=projects/PROJECT_ID/topics/TOPIC_ID
```

## Access Management Scripts

### Role Audit Script
```bash
#!/bin/bash
# Comprehensive role audit script

PROJECT_ID="your-project-id"
OUTPUT_DIR="iam_audit"
DATE=$(date +%Y%m%d)

mkdir -p "$OUTPUT_DIR"

# Function to get all IAM bindings
get_iam_bindings() {
    local scope=$1
    local id=$2
    local output_file="$OUTPUT_DIR/${scope}_iam_bindings_${DATE}.json"

    case $scope in
        "project")
            gcloud projects get-iam-policy "$id" \
                --format=json > "$output_file"
            ;;
        "organization")
            gcloud organizations get-iam-policy "$id" \
                --format=json > "$output_file"
            ;;
    esac

    echo "IAM bindings exported to $output_file"
}

# Function to audit service accounts
audit_service_accounts() {
    local project_id=$1
    local output_file="$OUTPUT_DIR/service_accounts_${DATE}.json"

    # Get service accounts
    gcloud iam service-accounts list \
        --project="$project_id" \
        --format=json > "$output_file"

    # Get keys for each service account
    while read -r sa_email; do
        gcloud iam service-accounts keys list \
            --iam-account="$sa_email" \
            --project="$project_id" \
            --format=json >> "$OUTPUT_DIR/sa_keys_${DATE}.json"
    done < <(jq -r '.[].email' "$output_file")
}

# Function to audit custom roles
audit_custom_roles() {
    local project_id=$1
    local output_file="$OUTPUT_DIR/custom_roles_${DATE}.json"

    gcloud iam roles list \
        --project="$project_id" \
        --format=json > "$output_file"

    # Get detailed role definitions
    while read -r role_name; do
        gcloud iam roles describe "$role_name" \
            --project="$project_id" \
            --format=json >> "$OUTPUT_DIR/role_definitions_${DATE}.json"
    done < <(jq -r '.[].name' "$output_file")
}

# Main execution
get_iam_bindings "project" "$PROJECT_ID"
audit_service_accounts "$PROJECT_ID"
audit_custom_roles "$PROJECT_ID"
```

### Automated IAM Setup Script
```bash
#!/bin/bash
# Automated IAM setup script

PROJECT_ID="your-project-id"
TEAM_NAME="platform-team"

# Create custom roles
create_custom_roles() {
    local project_id=$1
    
    # Developer role
    gcloud iam roles create "${TEAM_NAME}_developer" \
        --project="$project_id" \
        --title="${TEAM_NAME} Developer" \
        --description="Custom developer role" \
        --permissions="compute.instances.get,compute.instances.list,compute.instances.start,compute.instances.stop"

    # Admin role
    gcloud iam roles create "${TEAM_NAME}_admin" \
        --project="$project_id" \
        --title="${TEAM_NAME} Admin" \
        --description="Custom admin role" \
        --permissions="compute.instances.*,storage.buckets.*"
}

# Create service accounts
create_service_accounts() {
    local project_id=$1

    # Application service account
    gcloud iam service-accounts create "${TEAM_NAME}-app" \
        --display-name="${TEAM_NAME} Application" \
        --description="Service account for applications"

    # CI/CD service account
    gcloud iam service-accounts create "${TEAM_NAME}-cicd" \
        --display-name="${TEAM_NAME} CI/CD" \
        --description="Service account for CI/CD pipelines"
}

# Set up IAM bindings
setup_iam_bindings() {
    local project_id=$1

    # Developer group bindings
    gcloud projects add-iam-policy-binding "$project_id" \
        --member="group:${TEAM_NAME}-developers@domain.com" \
        --role="projects/$project_id/roles/${TEAM_NAME}_developer"

    # Admin group bindings
    gcloud projects add-iam-policy-binding "$project_id" \
        --member="group:${TEAM_NAME}-admins@domain.com" \
        --role="projects/$project_id/roles/${TEAM_NAME}_admin"

    # Service account bindings
    gcloud projects add-iam-policy-binding "$project_id" \
        --member="serviceAccount:${TEAM_NAME}-app@$project_id.iam.gserviceaccount.com" \
        --role="roles/compute.instanceAdmin.v1"
}

# Main execution
create_custom_roles "$PROJECT_ID"
create_service_accounts "$PROJECT_ID"
setup_iam_bindings "$PROJECT_ID"
```

Remember:
- Always follow the principle of least privilege
- Regularly audit IAM policies
- Rotate service account keys
- Document all IAM changes
- Use groups instead of individual users
- Implement proper monitoring
- Regular security reviews
- Maintain change logs

For detailed information, consult the Google Cloud IAM documentation and `gcloud` command reference (`gcloud help`).

Would you like me to cover any specific aspect of IAM or administrative commands in more detail?