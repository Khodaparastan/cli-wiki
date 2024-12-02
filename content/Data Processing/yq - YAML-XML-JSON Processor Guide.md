
## Table of Contents
- [Overview](#overview)
- [Basic Usage](#basic-usage)
- [YAML Operations](#yaml-operations)
- [Data Transformation](#data-transformation)
- [Multi-Document Processing](#multi-document-processing)
- [Format Conversion](#format-conversion)
- [Advanced Features](#advanced-features)
- [Best Practices](#best-practices)

## Overview

yq is a lightweight and portable command-line YAML, JSON, and XML processor. It's written in Go and uses similar syntax to jq.

### Key Features
- YAML/JSON/XML processing
- Multi-document support
- Format conversion
- In-place editing
- Expression evaluation
- Array operations
- Document merging
- Path-based updates

## Basic Usage

### Simple Operations
```bash
# Read YAML
yq '.' file.yaml

# Get specific field
yq '.field' file.yaml

# Get nested field
yq '.parent.child' file.yaml

# Get array element
yq '.array[0]' file.yaml
```

### Common Options
```bash
# Pretty print
yq -P '.' file.yaml

# In-place edit
yq -i '.field = "value"' file.yaml

# Output format
yq -o=json '.' file.yaml
```

## YAML Operations

### Basic Queries
```bash
# Select multiple fields
yq '{name: .name, age: .age}' file.yaml

# Filter arrays
yq '.users[] | select(.age > 30)' file.yaml

# Count elements
yq '.array | length' file.yaml

# Get keys
yq 'keys' file.yaml
```

### Modifications
```bash
# Set value
yq '.field = "new value"' file.yaml

# Add field
yq '.newField = "value"' file.yaml

# Delete field
yq 'del(.field)' file.yaml

# Append to array
yq '.array += ["new item"]' file.yaml
```

## Data Transformation

### Value Operations
```bash
# String operations
yq '.name |= upcase' file.yaml

# Number operations
yq '.price *= 1.1' file.yaml

# Boolean operations
yq '.active |= not' file.yaml
```

### Structure Modifications
```bash
# Rename fields
yq '.new_name = .old_name | del(.old_name)' file.yaml

# Merge objects
yq '. *+ {"new": "field"}' file.yaml

# Sort arrays
yq '[.array[] | sort_by(.field)]' file.yaml
```

## Multi-Document Processing

### Document Operations
```bash
# Process all documents
yq eval-all '.' multiple-docs.yaml

# Merge documents
yq eval-all '. as $item ireduce ({}; . * $item)' multiple-docs.yaml

# Select specific document
yq eval-all 'select(documentIndex == 0)' multiple-docs.yaml
```

### Document Manipulation
```bash
# Split document
yq eval-all '.items[] | splitDoc' file.yaml

# Join documents
yq eval-all '. as $item ireduce ([]; . + $item)' files*.yaml

# Filter documents
yq eval-all 'select(.type == "config")' multiple-docs.yaml
```

## Format Conversion

### YAML/JSON Conversion
```bash
# YAML to JSON
yq -o=json '.' file.yaml

# JSON to YAML
yq -P '.' file.json

# Pretty JSON
yq -o=json -P '.' file.yaml
```

### XML Operations
```bash
# XML to YAML
yq -p=xml '.' file.xml

# YAML to XML
yq -o=xml '.' file.yaml

# XML attribute handling
yq -p=xml '.. | select(has("@attr"))' file.xml
```

## Advanced Features

### Expression Evaluation
```bash
# Conditional operations
yq 'if .type == "prod" then .replicas = 3 else .replicas = 1 end' file.yaml

# Complex expressions
yq '.items[] | select(.price > 100 and .stock > 0)' file.yaml

# Custom operators
yq '.price -= 10 | .stock += 5' file.yaml
```

### Path Operations
```bash
# Get paths
yq paths file.yaml

# Path-based updates
yq 'with(select(.type == "service"); .port = 8080)' file.yaml

# Path filtering
yq '.. | select(. == "value")' file.yaml
```

## Best Practices

### Performance Tips
```bash
# Stream processing
yq -s '.' large.yaml

# Efficient filtering
yq 'select(.important == true)' file.yaml

# Batch processing
yq eval-all '.' *.yaml
```

### Error Handling
```bash
# Check existence
yq 'select(.field != null)' file.yaml

# Default values
yq '.missing // "default"' file.yaml

# Error logging
yq --verbose '.' file.yaml 2>errors.log
```

## Example Scripts

### Kubernetes Config Management
```bash
#!/bin/bash
# Update Kubernetes deployments
yq eval '
  .spec.template.spec.containers[0].image = "newimage:latest" |
  .spec.replicas = 3
' deployment.yaml
```

### Configuration Generator
```bash
#!/bin/bash
# Generate configuration files
yq eval-all '
  .env = "production" |
  .settings.debug = false |
  .settings.timeout = 30 |
  .settings.maxConnections = 100
' template.yaml
```

### YAML Validation
```bash
#!/bin/bash
# Validate YAML files
validate_yaml() {
    local file=$1
    if ! yq '.' "$file" >/dev/null 2>&1; then
        echo "Invalid YAML: $file"
        return 1
    fi
    echo "Valid YAML: $file"
    return 0
}

for file in *.yaml; do
    validate_yaml "$file"
done
```

### Multi-Environment Config
```bash
#!/bin/bash
# Generate environment-specific configs
ENVIRONMENTS=("dev" "staging" "prod")

for env in "${ENVIRONMENTS[@]}"; do
    yq eval-all "
        select(documentIndex == 0) * load(\"${env}-values.yaml\") |
        .environment = \"${env}\"
    " base.yaml > "config-${env}.yaml"
done
```

### Document Merger
```bash
#!/bin/bash
# Merge multiple YAML documents
yq eval-all '
  # Merge all documents
  . as $item ireduce ({}; . * $item) |
  # Sort keys
  sort_keys(.)
' *.yaml > merged.yaml
```
