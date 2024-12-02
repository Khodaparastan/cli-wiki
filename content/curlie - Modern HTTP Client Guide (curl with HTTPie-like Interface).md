
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [HTTP Methods](#http-methods)
- [Headers & Data](#headers--data)
- [Authentication](#authentication)
- [Output Formatting](#output-formatting)
- [Advanced Features](#advanced-features)
- [Best Practices](#best-practices)

## Overview

curlie is a frontend to curl that adds the ease of use of HTTPie while retaining curl's powerful features.

### Key Features
- HTTPie-like syntax
- curl compatibility
- JSON highlighting
- Intuitive interface
- Header formatting
- Color output
- Automatic formatting
- curl command translation

## Installation

### Using Go
```bash
# Install using go
go install github.com/rs/curlie@latest
```

### Using Package Managers
```bash
# macOS
brew install curlie

# Arch Linux
yay -S curlie
```

## Basic Usage

### Simple Requests
```bash
# GET request
curlie example.com

# Show headers
curlie -I example.com

# Follow redirects
curlie -L example.com

# Verbose output
curlie -v example.com
```

### Common Options
```bash
# Silent mode
curlie -s example.com

# Include response headers
curlie -i example.com

# Show curl command
curlie --curl example.com
```

## HTTP Methods

### GET Requests
```bash
# Basic GET
curlie get api.example.com/users

# GET with parameters
curlie get api.example.com/users id==123 type==active

# GET with headers
curlie get api.example.com/users Accept:application/json
```

### POST Requests
```bash
# POST JSON
curlie post api.example.com/users \
    name=john \
    age:=25

# POST form data
curlie -f post api.example.com/users \
    name=john \
    age=25

# POST raw JSON
curlie post api.example.com/users \
    Content-Type:application/json \
    @data.json
```

### Other Methods
```bash
# PUT request
curlie put api.example.com/users/123 \
    name=john \
    age:=26

# DELETE request
curlie delete api.example.com/users/123

# PATCH request
curlie patch api.example.com/users/123 \
    age:=26
```

## Headers & Data

### Custom Headers
```bash
# Add headers
curlie api.example.com \
    Authorization:"Bearer token123" \
    Accept:application/json

# Content-Type
curlie api.example.com \
    Content-Type:application/json \
    data:='{"key":"value"}'
```

### Data Formats
```bash
# JSON data
curlie post api.example.com \
    key=value \
    array:='[1,2,3]'

# Form data
curlie -f post api.example.com \
    file@/path/to/file.txt

# Raw data
curlie post api.example.com \
    @raw.txt
```

## Authentication

### Basic Auth
```bash
# Username and password
curlie -a username:password api.example.com

# Bearer token
curlie api.example.com \
    Authorization:"Bearer token123"
```

### API Authentication
```bash
# API key in header
curlie api.example.com \
    X-API-Key:key123

# API key in query
curlie api.example.com apiKey==key123
```

## Output Formatting

### Response Formatting
```bash
# Pretty JSON
curlie get api.example.com/users

# Raw output
curlie -r get api.example.com/users

# Download file
curlie -o file.txt example.com/file
```

### Debug Information
```bash
# Show verbose info
curlie -v api.example.com

# Show timing
curlie -v api.example.com \
    -w "%{time_total}\n"

# Show curl command
curlie --curl api.example.com
```

## Advanced Features

### Proxy Settings
```bash
# HTTP proxy
curlie -x proxy.example.com:8080 api.example.com

# SOCKS proxy
curlie --proxy socks5://proxy.example.com api.example.com
```

### SSL Options
```bash
# Ignore SSL
curlie -k api.example.com

# Custom cert
curlie --cacert ca.crt api.example.com

# Client cert
curlie --cert client.crt --key client.key api.example.com
```

## Best Practices

### Error Handling
```bash
# Show errors
curlie -f api.example.com

# Retry on failure
curlie --retry 3 api.example.com

# Set timeout
curlie --max-time 10 api.example.com
```

### Debugging
```bash
# Debug mode
curlie -v api.example.com

# Show curl command
curlie --curl api.example.com

# Save full session
curlie -v --output-dir ./debug api.example.com
```

## Example Scripts

### API Testing Script
```bash
#!/bin/bash
# API testing script
API_URL="https://api.example.com"
TOKEN="your-token"

# GET request
test_get() {
    echo "Testing GET endpoint..."
    curlie get "$API_URL/users" \
        Authorization:"Bearer $TOKEN"
}

# POST request
test_post() {
    echo "Testing POST endpoint..."
    curlie post "$API_URL/users" \
        Authorization:"Bearer $TOKEN" \
        name=test \
        email=test@example.com
}

# Run tests
test_get
test_post
```

### Batch API Requests
```bash
#!/bin/bash
# Batch API requests
API_URL="https://api.example.com"
ENDPOINTS=(
    "users"
    "posts"
    "comments"
)

for endpoint in "${ENDPOINTS[@]}"; do
    echo "Fetching $endpoint..."
    curlie get "$API_URL/$endpoint" \
        Accept:application/json \
        -o "${endpoint}.json"
done
```

### API Health Check
```bash
#!/bin/bash
# API health check script
ENDPOINTS=(
    "https://api1.example.com/health"
    "https://api2.example.com/health"
)
LOG_FILE="health_check.log"

check_endpoint() {
    local url=$1
    echo "Checking $url..."
    
    response=$(curlie -s -w "%{http_code}" get "$url")
    status=$?
    
    if [ $status -eq 0 ] && [ "$response" == "200" ]; then
        echo "$(date): $url - OK" >> "$LOG_FILE"
    else
        echo "$(date): $url - FAIL (Status: $response)" >> "$LOG_FILE"
        notify_admin "$url" "$response"
    fi
}

notify_admin() {
    local url=$1
    local status=$2
    echo "Alert: $url returned status $status"
}

for endpoint in "${ENDPOINTS[@]}"; do
    check_endpoint "$endpoint"
    sleep 1
done
```

### Performance Testing
```bash
#!/bin/bash
# API performance testing
URL="https://api.example.com/endpoint"
ITERATIONS=10
RESULTS_FILE="performance.csv"

echo "timestamp,duration" > "$RESULTS_FILE"

for i in $(seq 1 $ITERATIONS); do
    timestamp=$(date +%s)
    duration=$(curlie -s -w "%{time_total}" "$URL" -o /dev/null)
    echo "$timestamp,$duration" >> "$RESULTS_FILE"
    sleep 1
done

echo "Results saved to $RESULTS_FILE"
```

