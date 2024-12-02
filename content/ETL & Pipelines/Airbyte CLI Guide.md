
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Source Operations](#source-operations)
- [Destination Operations](#destination-operations)
- [Connection Management](#connection-management)
- [Sync Operations](#sync-operations)
- [Advanced Features](#advanced-features)
- [Best Practices](#best-practices)

## Overview

Airbyte CLI is a command-line tool for managing Airbyte data pipelines, allowing automation and integration of data synchronization workflows.

### Key Features
- Source/destination management
- Connection configuration
- Sync job control
- Pipeline automation
- Configuration import/export
- Health monitoring
- Workspace management
- OAuth handling

## Installation

```bash
# Using Python pip
pip install airbyte-cli

# Using Homebrew
brew install airbyte-cli

# Using Docker
docker pull airbyte/cli:latest
```

## Basic Usage

### Initial Setup
```bash
# Initialize CLI
airbyte init

# Configure workspace
airbyte workspace init

# List available commands
airbyte --help

# Check version
airbyte version
```

### Authentication
```bash
# Set API key
airbyte auth login --api-key YOUR_API_KEY

# Configure host
airbyte config set host http://localhost:8000

# Test connection
airbyte health-check
```

## Source Operations

### Source Management
```bash
# List available sources
airbyte source list-types

# Create source
airbyte source create \
    --name "PostgreSQL Production" \
    --type postgres \
    --config source-config.json

# List configured sources
airbyte source list

# Get source details
airbyte source get --id source_id
```

### Source Configuration
```bash
# Create source configuration
cat > source-config.json << EOF
{
    "host": "localhost",
    "port": 5432,
    "database": "production",
    "username": "user",
    "password": "pass",
    "schema": "public"
}
EOF

# Update source
airbyte source update \
    --id source_id \
    --config source-config.json

# Check source connection
airbyte source check-connection --id source_id
```

## Destination Operations

### Destination Management
```bash
# List available destinations
airbyte destination list-types

# Create destination
airbyte destination create \
    --name "Snowflake Warehouse" \
    --type snowflake \
    --config destination-config.json

# List configured destinations
airbyte destination list

# Get destination details
airbyte destination get --id destination_id
```

### Destination Configuration
```bash
# Create destination configuration
cat > destination-config.json << EOF
{
    "host": "account.snowflakecomputing.com",
    "role": "ACCOUNTADMIN",
    "warehouse": "COMPUTE_WH",
    "database": "AIRBYTE_DB",
    "schema": "PUBLIC",
    "username": "user",
    "password": "pass"
}
EOF

# Update destination
airbyte destination update \
    --id destination_id \
    --config destination-config.json

# Check destination connection
airbyte destination check-connection --id destination_id
```

## Connection Management

### Connection Setup
```bash
# Create connection
airbyte connection create \
    --source-id source_id \
    --destination-id destination_id \
    --config connection-config.json

# List connections
airbyte connection list

# Get connection details
airbyte connection get --id connection_id
```

### Connection Configuration
```bash
# Create connection configuration
cat > connection-config.json << EOF
{
    "syncCatalog": {
        "streams": [
            {
                "stream": {
                    "name": "users",
                    "jsonSchema": {},
                    "supportedSyncModes": ["full_refresh", "incremental"]
                },
                "config": {
                    "syncMode": "incremental",
                    "cursorField": ["updated_at"],
                    "destinationSyncMode": "append_dedup"
                }
            }
        ]
    },
    "schedule": {
        "units": 24,
        "timeUnit": "hours"
    },
    "status": "active"
}
EOF

# Update connection
airbyte connection update \
    --id connection_id \
    --config connection-config.json
```

## Sync Operations

### Manual Sync
```bash
# Trigger sync
airbyte connection sync --id connection_id

# Check sync status
airbyte connection get-state --id connection_id

# List sync history
airbyte connection list-jobs --id connection_id
```

### Sync Monitoring
```bash
# Get sync logs
airbyte connection get-logs --id connection_id

# Monitor sync progress
airbyte connection watch --id connection_id

# Get sync statistics
airbyte connection get-stats --id connection_id
```

## Advanced Features

### Workspace Management
```bash
# Create workspace
airbyte workspace create --name "Production"

# List workspaces
airbyte workspace list

# Switch workspace
airbyte workspace use --id workspace_id

# Update workspace
airbyte workspace update \
    --id workspace_id \
    --name "New Name"
```

### Configuration Management
```bash
# Export configuration
airbyte export --workspace-id workspace_id config.yaml

# Import configuration
airbyte import config.yaml

# Backup workspace
airbyte backup create --workspace-id workspace_id
```

## Best Practices

### Error Handling
```bash
# Enable debug mode
airbyte --debug command

# Retry failed sync
airbyte connection retry-failed --id connection_id

# Handle connection errors
airbyte connection reset --id connection_id
```

## Example Scripts

### Automated Sync Setup
```bash
#!/bin/bash
# Setup complete sync pipeline

# Configuration
SOURCE_NAME="PostgreSQL Source"
DEST_NAME="Snowflake Dest"
CONN_NAME="Postgres to Snowflake"

# Create source
source_id=$(airbyte source create \
    --name "$SOURCE_NAME" \
    --type postgres \
    --config source-config.json \
    --format json | jq -r '.sourceId')

# Create destination
dest_id=$(airbyte destination create \
    --name "$DEST_NAME" \
    --type snowflake \
    --config destination-config.json \
    --format json | jq -r '.destinationId')

# Create connection
conn_id=$(airbyte connection create \
    --name "$CONN_NAME" \
    --source-id "$source_id" \
    --destination-id "$dest_id" \
    --config connection-config.json \
    --format json | jq -r '.connectionId')

# Start initial sync
airbyte connection sync --id "$conn_id"
```

### Monitoring Script
```bash
#!/bin/bash
# Monitor sync status

WEBHOOK_URL="https://hooks.slack.com/services/xxx"
CONNECTION_ID="your_connection_id"

monitor_sync() {
    status=$(airbyte connection get-state \
        --id "$CONNECTION_ID" \
        --format json | jq -r '.status')
    
    if [ "$status" == "failed" ]; then
        logs=$(airbyte connection get-logs --id "$CONNECTION_ID")
        curl -X POST -H 'Content-type: application/json' \
            --data "{\"text\":\"Sync failed: $logs\"}" \
            "$WEBHOOK_URL"
    fi
}

# Run monitoring every 5 minutes
while true; do
    monitor_sync
    sleep 300
done
```

### Bulk Operations
```bash
#!/bin/bash
# Perform bulk operations

# Reset all failed connections
airbyte connection list --format json | \
jq -r '.[] | select(.status=="failed") | .connectionId' | \
while read -r conn_id; do
    echo "Resetting connection $conn_id"
    airbyte connection reset --id "$conn_id"
done

# Backup all configurations
BACKUP_DIR="airbyte_backup_$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

airbyte workspace list --format json | \
jq -r '.[].workspaceId' | \
while read -r workspace_id; do
    airbyte export \
        --workspace-id "$workspace_id" \
        "$BACKUP_DIR/workspace_${workspace_id}.yaml"
done
```

---

Remember:
- Regularly backup configurations
- Monitor sync status
- Handle failures gracefully
- Document custom configurations
- Use version control for configs
- Implement monitoring
- Follow security best practices

For detailed information, consult the Airbyte CLI documentation and the official Airbyte documentation.