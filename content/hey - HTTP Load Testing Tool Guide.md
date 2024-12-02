
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Load Testing Patterns](#load-testing-patterns)
- [Output Analysis](#output-analysis)
- [Advanced Options](#advanced-options)
- [Best Practices](#best-practices)
- [Common Scenarios](#common-scenarios)

## Overview

`hey` is a modern HTTP load testing tool designed to be simple yet powerful, capable of generating load for benchmarking and functional testing.

### Key Features
- Simple command-line interface
- Multiple request methods
- Custom headers
- Request body support
- Concurrent connections
- Detailed statistics
- Response validation
- Proxy support

## Installation

### Using Go
```bash
# Install using go
go install github.com/rakyll/hey@latest
```

### Binary Installation
```bash
# Download binary
curl -L https://hey-release.s3.us-east-2.amazonaws.com/hey_linux_amd64 -o hey
chmod +x hey
sudo mv hey /usr/local/bin/
```

## Basic Usage

### Simple Tests
```bash
# Basic GET request
hey http://localhost:8080/

# Specify number of requests
hey -n 1000 http://localhost:8080/

# Set number of concurrent workers
hey -c 100 http://localhost:8080/

# Combined options
hey -n 1000 -c 100 http://localhost:8080/
```

### Request Methods
```bash
# POST request
hey -m POST http://localhost:8080/api

# PUT request
hey -m PUT http://localhost:8080/api

# DELETE request
hey -m DELETE http://localhost:8080/api
```

## Load Testing Patterns

### Basic Load Testing
```bash
# Quick load test
hey -n 2000 -c 50 http://localhost:8080/

# Sustained load
hey -n 10000 -c 100 http://localhost:8080/

# Time-based test
hey -z 30s http://localhost:8080/
```

### Advanced Patterns
```bash
# With request body
hey -m POST \
    -d '{"key":"value"}' \
    http://localhost:8080/api

# With headers
hey -H "Authorization: Bearer token" \
    -H "Content-Type: application/json" \
    http://localhost:8080/api
```

## Output Analysis

### Understanding Output
```text
Summary:
  Total:        30.0665 secs
  Slowest:      1.5234 secs
  Fastest:      0.0234 secs
  Average:      0.2988 secs
  Requests/sec: 334.4324

Response time histogram:
  0.023 [1]     |
  0.173 [2]     |∎
  0.323 [85]    |∎∎∎∎∎∎∎∎∎∎∎∎
  0.473 [7]     |∎
  0.623 [2]     |∎
  0.773 [1]     |
  0.923 [1]     |
  1.073 [0]     |
  1.223 [0]     |
  1.373 [0]     |
  1.523 [1]     |
```

### Output Options
```bash
# Disable progress output
hey -q http://localhost:8080/

# Detailed output
hey -v http://localhost:8080/

# Save output to file
hey -o report.csv http://localhost:8080/
```

## Advanced Options

### Request Configuration
```bash
# Custom headers
hey -H "User-Agent: custom-agent" \
    -H "Accept: application/json" \
    http://localhost:8080/

# Basic auth
hey -a username:password http://localhost:8080/

# Proxy settings
hey -x http://proxy:8080 http://localhost:8080/
```

### Response Validation
```bash
# Disable redirect following
hey -disable-redirects http://localhost:8080/

# Set timeout
hey -t 20s http://localhost:8080/

# Accept invalid certs
hey -disable-compression http://localhost:8080/
```

### Rate Limiting
```bash
# Requests per second
hey -q 100 http://localhost:8080/

# Burst size
hey -q 100 -burst 20 http://localhost:8080/
```

## Best Practices

### Testing Guidelines
```bash
# Warm-up run
hey -n 100 -c 10 http://localhost:8080/

# Progressive load
for c in 10 50 100 200; do
    hey -n 1000 -c $c http://localhost:8080/
    sleep 5
done
```

### Resource Monitoring
```bash
# Monitor while testing
while true; do
    date
    ps aux | grep hey
    netstat -an | grep ESTABLISHED | wc -l
    sleep 1
done
```

## Quick Reference

### Essential Commands
```bash
# Basic test
hey http://localhost:8080/

# Load test
hey -n 1000 -c 100 http://localhost:8080/

# Timed test
hey -z 30s http://localhost:8080/

# POST request
hey -m POST -d '{"key":"value"}' http://localhost:8080/
```

### Common Options
```bash
-n    # Number of requests
-c    # Number of workers
-z    # Duration of test
-q    # Requests per second
-m    # HTTP method
-H    # Headers
-d    # Request body
-o    # Output file
```

## Example Scripts

### Basic Load Test
```bash
#!/bin/bash
# Progressive load testing
URL="http://localhost:8080/"
REQUESTS=1000

for workers in 10 50 100; do
    echo "Testing with $workers workers"
    hey -n $REQUESTS -c $workers "$URL"
    sleep 5
done
```

### API Testing
```bash
#!/bin/bash
# API endpoint testing
URL="http://localhost:8080/api"
TOKEN="your-auth-token"

hey -n 1000 -c 50 \
    -m POST \
    -H "Authorization: Bearer $TOKEN" \
    -H "Content-Type: application/json" \
    -d '{"test": "data"}' \
    "$URL"
```

### Comprehensive Test Suite
```bash
#!/bin/bash
# Full test suite
BASE_URL="http://localhost:8080"
ENDPOINTS=(
    "/"
    "/api/v1"
    "/api/v2"
)

for endpoint in "${ENDPOINTS[@]}"; do
    echo "Testing $endpoint"
    hey -n 500 -c 50 "${BASE_URL}${endpoint}"
    sleep 2
done
```

### Performance Monitoring
```bash
#!/bin/bash
# Monitor and log performance
URL="http://localhost:8080/"
OUTPUT_DIR="hey_results"

mkdir -p "$OUTPUT_DIR"

run_test() {
    local workers=$1
    local timestamp=$(date +%Y%m%d_%H%M%S)
    local output_file="${OUTPUT_DIR}/test_${workers}_${timestamp}.txt"
    
    echo "Running test with $workers workers"
    hey -n 1000 -c "$workers" "$URL" > "$output_file"
}

# Run tests with different concurrency levels
for workers in 10 50 100; do
    run_test "$workers"
    sleep 5
done
```

---

Remember:
- Start with lower loads
- Monitor server resources
- Use appropriate timeouts
- Consider rate limiting
- Document test conditions
- Analyze all metrics

For detailed information, consult the hey documentation (`hey -h`).