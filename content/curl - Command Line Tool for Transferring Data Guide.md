
## Table of Contents
- [Overview](#overview)
- [Basic Usage](#basic-usage)
- [HTTP Methods](#http-methods)
- [Headers & Data](#headers--data)
- [Authentication](#authentication)
- [SSL/TLS Options](#ssltls-options)
- [Output Options](#output-options)
- [Advanced Features](#advanced-features)
- [Debugging](#debugging)
- [Best Practices](#best-practices)

## Overview

curl is a powerful command-line tool for transferring data using various protocols, primarily HTTP(S), FTP, and many others.

### Key Features
- Multiple protocol support
- HTTP method support
- Header manipulation
- Authentication methods
- SSL/TLS handling
- Proxy support
- Cookie handling
- Detailed debugging

## Basic Usage

### Simple Requests
```bash
# Basic GET request
curl https://example.com

# Save output to file
curl -o output.html https://example.com

# Follow redirects
curl -L https://example.com

# Show response headers
curl -I https://example.com
```

### Common Options
```bash
# Silent mode
curl -s https://example.com

# Progress bar
curl -# https://example.com

# Include response headers
curl -i https://example.com

# Only headers
curl -I https://example.com
```

## HTTP Methods

### GET Requests
```bash
# Basic GET
curl https://api.example.com/users

# GET with parameters
curl "https://api.example.com/users?id=123&type=active"

# GET with headers
curl -H "Accept: application/json" https://api.example.com/users
```

### POST Requests
```bash
# POST form data
curl -X POST -d "name=john&age=25" https://api.example.com/users

# POST JSON data
curl -X POST \
     -H "Content-Type: application/json" \
     -d '{"name":"john","age":25}' \
     https://api.example.com/users

# POST with file
curl -X POST \
     -H "Content-Type: application/json" \
     -d @data.json \
     https://api.example.com/users
```

### Other Methods
```bash
# PUT request
curl -X PUT \
     -d '{"name":"john"}' \
     https://api.example.com/users/123

# DELETE request
curl -X DELETE https://api.example.com/users/123

# PATCH request
curl -X PATCH \
     -d '{"age":26}' \
     https://api.example.com/users/123
```

## Headers & Data

### Custom Headers
```bash
# Single header
curl -H "Authorization: Bearer token123" https://api.example.com

# Multiple headers
curl -H "Authorization: Bearer token123" \
     -H "Content-Type: application/json" \
     -H "Accept: application/json" \
     https://api.example.com
```

### Data Formats
```bash
# Form data
curl -d "param1=value1&param2=value2" https://api.example.com

# JSON data
curl -H "Content-Type: application/json" \
     -d '{"key":"value"}' \
     https://api.example.com

# File upload
curl -F "file=@photo.jpg" https://api.example.com/upload
```

## Authentication

### Basic Auth
```bash
# Username and password
curl -u username:password https://api.example.com

# Prompt for password
curl -u username https://api.example.com
```

### Bearer Token
```bash
# Authorization header
curl -H "Authorization: Bearer token123" https://api.example.com

# OAuth2 token
curl -H "Authorization: OAuth token123" https://api.example.com
```

### API Keys
```bash
# As header
curl -H "X-API-Key: key123" https://api.example.com

# As parameter
curl "https://api.example.com?api_key=key123"
```

## SSL/TLS Options

### Certificate Handling
```bash
# Ignore SSL certificate
curl -k https://example.com

# Specify certificate
curl --cacert ca.crt https://example.com

# Client certificate
curl --cert client.crt --key client.key https://example.com
```

### SSL Versions
```bash
# Force TLS version
curl --tlsv1.2 https://example.com

# Show SSL certificate
curl -vI https://example.com
```

## Output Options

### Save Output
```bash
# Save to file
curl -o output.html https://example.com

# Save with remote name
curl -O https://example.com/file.zip

# Multiple files
curl -O https://example.com/file1.zip \
     -O https://example.com/file2.zip
```

### Format Output
```bash
# Pretty print JSON
curl https://api.example.com | json_pp

# Format headers
curl -i https://example.com | less

# Download progress
curl -# -O https://example.com/large-file.zip
```

## Advanced Features

### Proxy Settings
```bash
# HTTP proxy
curl -x proxy.example.com:8080 https://api.example.com

# SOCKS proxy
curl --socks5 proxy.example.com:1080 https://api.example.com

# Proxy authentication
curl -x user:pass@proxy.example.com:8080 https://api.example.com
```

### Cookie Handling
```bash
# Send cookie
curl -b "session=123" https://example.com

# Save cookies
curl -c cookies.txt https://example.com

# Use cookie file
curl -b cookies.txt https://example.com
```

### Rate Limiting
```bash
# Limit rate
curl --limit-rate 1000B https://example.com/large-file

# Maximum time
curl --max-time 10 https://example.com
```

## Debugging

### Verbose Output
```bash
# Basic verbose
curl -v https://example.com

# More verbose
curl -vv https://example.com

# Trace
curl --trace debug.txt https://example.com
```

### Request Timing
```bash
# Show timing
curl -w "\nTime: %{time_total}s\n" https://example.com

# Detailed timing
curl -w @curl-format.txt https://example.com
```

## Best Practices

### Error Handling
```bash
# Show errors
curl -f https://example.com

# Retry on failure
curl --retry 3 https://example.com

# Retry delay
curl --retry 3 --retry-delay 2 https://example.com
```

### Security
```bash
# Verify SSL
curl --cacert ca.crt https://example.com

# Use latest TLS
curl --tlsv1.2 https://example.com

# Disable weak ciphers
curl --ciphers HIGH https://example.com
```

## Example Scripts

### API Testing
```bash
#!/bin/bash
# API test script
API_URL="https://api.example.com"
TOKEN="your-token"

# GET request
get_data() {
    curl -s \
         -H "Authorization: Bearer $TOKEN" \
         -H "Accept: application/json" \
         "$API_URL/users"
}

# POST request
create_user() {
    curl -s \
         -X POST \
         -H "Authorization: Bearer $TOKEN" \
         -H "Content-Type: application/json" \
         -d '{"name":"John","email":"john@example.com"}' \
         "$API_URL/users"
}

# Run tests
echo "Getting users:"
get_data | json_pp

echo "Creating user:"
create_user | json_pp
```

### Website Monitor
```bash
#!/bin/bash
# Website monitoring script
URLS=(
    "https://example1.com"
    "https://example2.com"
)
LOG_FILE="monitoring.log"

check_website() {
    local url=$1
    local start_time=$(date +%s.%N)
    
    response=$(curl -s -w "\n%{http_code}\n%{time_total}" -o /dev/null "$url")
    status_code=$(echo "$response" | head -n1)
    time_total=$(echo "$response" | tail -n1)
    
    echo "$(date): $url - Status: $status_code, Time: ${time_total}s" >> "$LOG_FILE"
    
    if [ "$status_code" != "200" ]; then
        echo "Alert: $url returned status $status_code"
    fi
}

for url in "${URLS[@]}"; do
    check_website "$url"
done
```

### Batch Download
```bash
#!/bin/bash
# Batch download script
URLS_FILE="urls.txt"
OUTPUT_DIR="downloads"
MAX_RETRIES=3

mkdir -p "$OUTPUT_DIR"

while read url; do
    filename=$(basename "$url")
    echo "Downloading $filename..."
    
    curl --retry $MAX_RETRIES \
         --retry-delay 2 \
         -# \
         -o "$OUTPUT_DIR/$filename" \
         "$url"
done < "$URLS_FILE"
```

---

Remember:
- Use appropriate timeouts
- Handle errors properly
- Consider rate limiting
- Secure sensitive data
- Log important operations
- Follow API guidelines

For detailed information, consult the curl documentation (`man curl`).