
## Table of Contents
- [Overview](#overview)
- [Basic Usage](#basic-usage)
- [Filters](#filters)
- [Data Transformation](#data-transformation)
- [Array Operations](#array-operations)
- [Object Operations](#object-operations)
- [Advanced Features](#advanced-features)
- [Best Practices](#best-practices)

## Overview

jq is a lightweight command-line JSON processor. It's like sed for JSON data - you can use it to slice, filter, map, and transform structured data.

### Key Features
- JSON parsing and formatting
- Data filtering and transformation
- Complex data queries
- Array and object manipulation
- Stream processing
- Custom functions
- Regular expressions
- Mathematical operations

## Basic Usage

### Simple Queries
```bash
# Pretty print JSON
jq '.' file.json

# Get specific field
jq '.fieldname' file.json

# Array element
jq '.[0]' file.json

# Multiple fields
jq '{name: .name, age: .age}' file.json
```

### Common Options
```bash
# Compact output (no pretty printing)
jq -c '.' file.json

# Raw output (no quotes)
jq -r '.name' file.json

# Slurp array
jq -s '.' file1.json file2.json
```

## Filters

### Basic Filters
```bash
# Select fields
jq '.[] | {name: .name, age: .age}'

# Filter by value
jq '.[] | select(.age > 30)'

# Filter by existence
jq '.[] | select(.optional != null)'

# Multiple conditions
jq '.[] | select(.age > 30 and .type == "user")'
```

### Complex Filters
```bash
# Nested selection
jq '.users[] | select(.address.city == "London")'

# Regular expressions
jq '.[] | select(.name | test("^J.*"))'

# Contains
jq '.[] | select(.tags | contains(["important"]))'
```

## Data Transformation

### Value Modifications
```bash
# Modify values
jq '.[] | .price *= 1.1'

# String operations
jq '.name |= ascii_upcase'

# Date formatting
jq '.date |= fromdate | strftime("%Y-%m-%d")'
```

### Structure Changes
```bash
# Rename fields
jq '{ user: .name, years: .age }'

# Add fields
jq '. += { "status": "active" }'

# Delete fields
jq 'del(.unwanted_field)'
```

## Array Operations

### Array Manipulation
```bash
# Map transformation
jq '.[] | map(.price *= 1.1)'

# Filter array
jq '[.[] | select(.price > 100)]'

# Sort array
jq 'sort_by(.price)'

# Unique values
jq 'unique_by(.id)'
```

### Array Aggregation
```bash
# Count elements
jq '.[] | length'

# Sum values
jq '[.[].price] | add'

# Group by field
jq 'group_by(.category)'

# Reduce operation
jq 'reduce .[] as $item (0; . + $item.price)'
```

## Object Operations

### Object Manipulation
```bash
# Merge objects
jq '. * {"new": "field"}'

# Pick fields
jq 'pick(["name", "age"])'

# Remove null fields
jq 'with_entries(select(.value != null))'

# Transform keys
jq 'with_entries(.key |= ascii_downcase)'
```

### Nested Operations
```bash
# Deep merge
jq 'recursively_merge(input)'

# Deep search
jq '.. | select(.id == "123")?'

# Flatten structure
jq 'flatten'
```

## Advanced Features

### Custom Functions
```bash
# Define function
jq 'def add_tax($rate): . * (1 + $rate);
    .price | add_tax(0.2)'

# Multiple functions
jq '
def calculate_total: .price * .quantity;
def add_tax($rate): . * (1 + $rate);
.[] | calculate_total | add_tax(0.2)
'
```

### Variables
```bash
# Using variables
jq --arg name "John" '.[] | select(.name == $name)'

# Multiple variables
jq --arg min "10" --arg max "20" \
   '.[] | select(.price >= ($min|tonumber) and .price <= ($max|tonumber))'
```

## Best Practices

### Performance Tips
```bash
# Stream large files
jq -c '.[]' large.json

# Selective parsing
jq -R 'fromjson? | select(.type == "error")'

# Efficient filtering
jq '[.[] | select(.important)] | .[0:10]'
```

### Error Handling
```bash
# Check for null
jq 'if . == null then empty else . end'

# Default values
jq '.missing // "default"'

# Try/catch equivalent
jq 'try .field catch "error"'
```

## Example Scripts

### Data Analysis
```bash
#!/bin/bash
# Analyze JSON logs
jq '
  def timestamp: fromdate | strftime("%Y-%m-%d");
  reduce .[] as $item ({};
    .[$item.type] += 1 |
    .dates[$item.timestamp | timestamp] += 1
  )
' logs.json
```

### Data Transformation
```bash
#!/bin/bash
# Transform data structure
jq '
  def normalize_user:
    {
      id: .user_id,
      name: .user_name,
      email: .user_email,
      active: (.status == "active")
    };
  
  .users | map(normalize_user)
' input.json
```

### API Response Processing
```bash
#!/bin/bash
# Process API response
jq '
  def process_response:
    select(.status == "success") |
    .data |
    map({
      id: .id,
      name: .name,
      total: (.price * .quantity)
    }) |
    sort_by(.total) |
    reverse |
    .[0:5];  # top 5 by total
    
  process_response
' response.json
```

### Configuration Generator
```bash
#!/bin/bash
# Generate configuration
jq -n '
{
  app: {
    name: $name,
    version: $version,
    environment: $env,
    settings: {
      debug: ($debug == "true"),
      max_connections: ($max_conn | tonumber),
      timeout: ($timeout | tonumber)
    }
  }
}' \
  --arg name "MyApp" \
  --arg version "1.0.0" \
  --arg env "production" \
  --arg debug "false" \
  --arg max_conn "100" \
  --arg timeout "30"
```

---

Remember:
- Use appropriate filters for performance
- Handle errors gracefully
- Document complex transformations
- Test with sample data
- Consider memory usage for large files
- Use raw output when needed

For detailed information, consult the jq manual (`man jq`) and the [jq documentation](https://stedolan.github.io/jq/manual/).