
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Advanced Options](#advanced-options)
- [Performance Testing](#performance-testing)
- [Results Analysis](#results-analysis)
- [Best Practices](#best-practices)
- [Common Scenarios](#common-scenarios)

## Overview

Apache Benchmark (ab) is a simple but powerful command-line tool for measuring HTTP server performance. It's particularly useful for quick load testing and server benchmarking.

### Key Features
- HTTP/HTTPS testing
- Concurrent connections
- POST data support
- Custom headers
- Basic authentication
- Cookie support
- Detailed statistics
- Percentile analysis

## Installation

### Ubuntu (22.04/24.04)
```bash
# Install apache2-utils
sudo apt update
sudo apt install apache2-utils
```

### macOS
```bash
# Using Homebrew
brew install apache2-utils
```

## Basic Usage

### Simple Tests
```bash
# Basic test (100 requests, 1 concurrent)
ab -n 100 http://localhost/

# Concurrent requests
ab -n 1000 -c 10 http://localhost/

# With verbosity
ab -v 2 -n 100 -c 10 http://localhost/
```

### Basic Options
```bash
# Test with keep-alive
ab -k -n 1000 -c 10 http://localhost/

# Specify timeout
ab -n 1000 -c 10 -t 2 http://localhost/

# Display progress
ab -n 1000 -c 10 -r http://localhost/
```

## Advanced Options

### Authentication and Headers
```bash
# Basic authentication
ab -A username:password -n 100 http://localhost/

# Custom headers
ab -H "Accept-Encoding: gzip" -n 100 http://localhost/

# Multiple headers
ab -H "Authorization: Bearer token" \
   -H "Content-Type: application/json" \
   -n 100 http://localhost/
```

### POST Requests
```bash
# POST with data
ab -p data.json -T application/json \
   -n 100 -c 10 http://localhost/api

# POST with inline data
ab -p - -T application/x-www-form-urlencoded \
   -n 100 "http://localhost/" <<EOF
param1=value1&param2=value2
EOF
```

### Cookie Support
```bash
# Send cookie
ab -C "sessionId=abc123" -n 100 http://localhost/

# Use cookie file
ab -C cookie.txt -n 100 http://localhost/
```

## Performance Testing

### Load Testing
```bash
# Gradual load increase
ab -n 100 -c 1 http://localhost/
ab -n 100 -c 10 http://localhost/
ab -n 100 -c 50 http://localhost/

# Sustained load
ab -n 10000 -c 100 http://localhost/
```

### Time-based Testing
```bash
# Run for specific duration
ab -t 60 -c 10 http://localhost/

# With keep-alive
ab -k -t 60 -c 10 http://localhost/
```

### SSL Testing
```bash
# HTTPS testing
ab -n 100 -c 10 https://localhost/

# With SSL options
ab -n 100 -c 10 -f SSL3 https://localhost/

# Ignore SSL errors
ab -n 100 -c 10 -k -s 1 https://localhost/
```

## Results Analysis

### Understanding Output
```text
Server Software:        nginx/1.18.0
Server Hostname:        localhost
Server Port:           80

Document Path:         /
Document Length:       1234 bytes

Concurrency Level:     10
Time taken for tests:  1.234 seconds
Complete requests:     1000
Failed requests:       0
Keep-Alive requests:   1000
Total transferred:     1234567 bytes
HTML transferred:      1234000 bytes
Requests per second:   810.37 [#/sec] (mean)
Time per request:      12.340 [ms] (mean)
Time per request:      1.234 [ms] (mean, across all concurrent requests)
Transfer rate:         975.61 [Kbytes/sec] received
```

### Important Metrics
```bash
# High-level performance check
ab -n 1000 -c 10 -g results.tsv http://localhost/

# Analyze percentiles
ab -n 1000 -c 10 -e results.csv http://localhost/

# Detailed verbosity
ab -v 3 -n 100 -c 10 http://localhost/
```

## Best Practices

### Testing Guidelines
```bash
# Warm-up run
ab -n 100 -c 1 http://localhost/

# Progressive load testing
for c in 1 10 50 100; do
    echo "Testing with $c concurrent users"
    ab -n 1000 -c $c http://localhost/
    sleep 5
done
```

### Resource Monitoring
```bash
# Monitor system during test
vmstat 1

# Watch network connections
watch -n1 'netstat -an | grep ESTABLISHED | wc -l'

# Monitor server load
top -b -n 1
```

## Common Scenarios

### API Testing
```bash
# GET request with headers
ab -H "Authorization: Bearer token" \
   -H "Accept: application/json" \
   -n 1000 -c 10 http://localhost/api

# POST request
ab -p payload.json \
   -T application/json \
   -H "Authorization: Bearer token" \
   -n 1000 -c 10 http://localhost/api
```

### Web Application Testing
```bash
# Test with cookies
ab -C "session=123456" \
   -n 1000 -c 10 http://localhost/

# Form submission
ab -p form.txt \
   -T application/x-www-form-urlencoded \
   -n 100 -c 10 http://localhost/submit
```

## Quick Reference

### Essential Commands
```bash
# Basic test
ab -n 100 -c 10 http://localhost/

# Keep-alive test
ab -k -n 100 -c 10 http://localhost/

# POST test
ab -p data.json -T application/json -n 100 http://localhost/

# HTTPS test
ab -n 100 -c 10 https://localhost/
```

### Common Options
```bash
-n    # Number of requests
-c    # Concurrent requests
-k    # Keep-alive
-p    # POST file
-T    # Content-type
-H    # Custom header
-A    # Authentication
-C    # Cookie
-v    # Verbosity level
```

## Example Configurations

### Basic Load Test
```bash
#!/bin/bash
# Progressive load test
URLS="
http://localhost/page1
http://localhost/page2
"

for url in $URLS; do
    echo "Testing $url"
    for c in 10 50 100; do
        echo "Concurrency: $c"
        ab -n 1000 -c $c -r "$url"
        sleep 2
    done
done
```

### API Performance Test
```bash
#!/bin/bash
# API endpoints test
ab -n 1000 -c 10 \
   -H "Authorization: Bearer token" \
   -H "Content-Type: application/json" \
   -T application/json \
   -p payload.json \
   http://localhost/api/endpoint
```

---

Remember:
- Start with lower concurrency
- Monitor server resources
- Use appropriate timeouts
- Consider keep-alive for real-world scenarios
- Document test conditions
- Analyze all metrics

For detailed information, consult the ab man page (`man ab`).