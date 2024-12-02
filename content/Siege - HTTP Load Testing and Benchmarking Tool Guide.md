
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Advanced Options](#advanced-options)
- [Configuration](#configuration)
- [Statistics Analysis](#statistics-analysis)
- [Load Testing Strategies](#load-testing-strategies)
- [Best Practices](#best-practices)

## Overview

Siege is a multi-threaded HTTP load testing and benchmarking utility designed to let web developers measure their code under duress.

### Key Features
- Concurrent user simulation
- URL file support
- HTTP/HTTPS testing
- Cookie support
- Basic authentication
- POST/PUT data support
- Detailed statistics
- Configuration file support

## Installation

### Ubuntu (22.04/24.04)
```bash
# Install siege
sudo apt update
sudo apt install siege

# Verify installation
siege --version
```

### macOS
```bash
# Using Homebrew
brew install siege
```

## Basic Usage

### Simple Tests
```bash
# Basic test (default 15 seconds)
siege http://localhost/

# Specify concurrent users
siege -c 25 http://localhost/

# Set duration
siege -t 30S http://localhost/

# Specify number of repetitions
siege -r 100 http://localhost/
```

### Basic Options
```bash
# Verbose output
siege -v http://localhost/

# Quiet mode
siege -q http://localhost/

# Internet mode (random delay)
siege -i http://localhost/
```

## Advanced Options

### Authentication and Headers
```bash
# Basic authentication
siege --auth=username:password http://localhost/

# Add header
siege --header="X-Custom-Header: Value" http://localhost/

# Multiple headers
siege --header="Authorization: Bearer token" \
      --header="Content-Type: application/json" \
      http://localhost/
```

### POST Requests
```bash
# POST with data
siege --content-type="application/json" \
      --data='{"key":"value"}' \
      http://localhost/api

# POST from file
siege --content-type="application/json" \
      --data-file=payload.json \
      http://localhost/api
```

## Configuration

### Siege Configuration File
```bash
# View current config
siege.config

# Default location: ~/.siegerc
```

### Sample Configuration
```ini
# .siegerc
verbose = true
quiet = false
color = on
protocol = HTTP/1.1
connection = keep-alive
concurrent = 25
time = 30S
internet = false
delay = 1
timeout = 30
parser = true
cache = false
header = Content-Type: application/json
header = X-Custom-Header: Value
```

### URL File Usage
```bash
# Create URL file
cat > urls.txt << EOF
http://localhost/page1
http://localhost/page2
POST http://localhost/api data=value
EOF

# Run with URL file
siege -f urls.txt
```

## Statistics Analysis

### Understanding Output
```text
Transactions:                   1000 hits
Availability:                 100.00 %
Elapsed time:                  59.99 secs
Data transferred:              1.23 MB
Response time:                 0.15 secs
Transaction rate:             16.67 trans/sec
Throughput:                    0.02 MB/sec
Concurrency:                   2.50
Successful transactions:       1000
Failed transactions:              0
Longest transaction:           1.23
Shortest transaction:          0.01
```

### Logging Options
```bash
# Enable logging
siege --log=/path/to/siege.log http://localhost/

# Log format
siege --mark="Test 1" http://localhost/

# JSON output
siege --json http://localhost/
```

## Load Testing Strategies

### Progressive Load Testing
```bash
#!/bin/bash
# Increase load progressively
for c in 10 25 50 100; do
    echo "Testing with $c concurrent users"
    siege -c $c -t 30S http://localhost/
    sleep 10
done
```

### Extended Duration Tests
```bash
# Long-running test
siege -c 50 -t 1H http://localhost/

# With delay between requests
siege -c 50 -d 1 -t 1H http://localhost/
```

### Mixed Request Testing
```bash
# Create mixed request file
cat > mixed.txt << EOF
GET http://localhost/api/users
POST http://localhost/api/data {"id":1}
GET http://localhost/api/status
EOF

siege -f mixed.txt -c 25 -t 30S
```

## Best Practices

### Testing Guidelines
```bash
# Warm-up run
siege -c 1 -t 10S http://localhost/

# Progressive testing
siege -c 10 -t 1M http://localhost/
siege -c 25 -t 1M http://localhost/
siege -c 50 -t 1M http://localhost/
```

### Resource Monitoring
```bash
# Monitor system resources
while true; do
    date
    uptime
    netstat -an | grep ESTABLISHED | wc -l
    sleep 5
done
```

## Quick Reference

### Essential Commands
```bash
# Basic test
siege http://localhost/

# Concurrent users and time
siege -c 25 -t 30S http://localhost/

# URL file test
siege -f urls.txt

# Internet simulation
siege -i -c 25 -t 30S http://localhost/
```

### Common Options
```bash
-c    # Concurrent users
-t    # Time duration
-r    # Number of repetitions
-d    # Delay between requests
-f    # URL file
-i    # Internet simulation
-v    # Verbose output
-q    # Quiet mode
```

## Example Configurations

### API Load Test
```bash
#!/bin/bash
# API endpoint testing
siege -c 25 \
      --content-type="application/json" \
      --header="Authorization: Bearer token" \
      -t 5M \
      http://localhost/api
```

### Web Application Test
```bash
#!/bin/bash
# Create URL list
cat > urls.txt << EOF
http://localhost/
http://localhost/login
http://localhost/dashboard
http://localhost/api/status
EOF

# Run test with delays
siege -f urls.txt -c 50 -d 1 -t 30M
```

### Performance Benchmark
```bash
#!/bin/bash
# Full benchmark suite
ENDPOINTS=(
    "http://localhost/api/v1"
    "http://localhost/api/v2"
)

for endpoint in "${ENDPOINTS[@]}"; do
    echo "Testing $endpoint"
    siege -c 25 -t 1M "$endpoint"
    sleep 5
done
```

---

Remember:
- Start with low concurrency
- Monitor server resources
- Use appropriate delays
- Consider internet simulation for realism
- Log results for analysis
- Watch for error rates

For detailed information, consult the siege man page (`man siege`).