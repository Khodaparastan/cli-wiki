
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Advanced Options](#advanced-options)
- [Scripting](#scripting)
- [Results Analysis](#results-analysis)
- [Best Practices](#best-practices)
- [Common Scenarios](#common-scenarios)

## Overview

`wrk` is a modern HTTP benchmarking tool capable of generating significant load using a single multi-core CPU. It combines a multithreaded design with scalable event notification systems.

### Key Features
- HTTP/HTTPS testing
- Lua scripting support
- Multiple threads
- Connection keep-alive
- Custom headers
- Request generation
- Response handling
- Detailed statistics

## Installation

### Ubuntu (22.04/24.04)
```bash
# Install dependencies
sudo apt install build-essential libssl-dev git

# Clone and build
git clone https://github.com/wg/wrk.git
cd wrk
make
sudo cp wrk /usr/local/bin
```

### macOS
```bash
# Using Homebrew
brew install wrk
```

## Basic Usage

### Simple Benchmarks
```bash
# Basic test (default: 2 threads, 10 connections, 10s)
wrk http://localhost:8080/

# Specify duration
wrk -d 30s http://localhost:8080/

# Custom threads and connections
wrk -t12 -c400 -d30s http://localhost:8080/
```

### Connection Options
```bash
# Keep-alive connections
wrk -t2 -c100 -d30s --latency http://localhost:8080/

# Disable keep-alive
wrk -t2 -c100 -d30s --timeout 2s http://localhost:8080/
```

## Advanced Options

### Headers and Methods
```bash
# Add headers
wrk -H "Accept-Encoding: gzip" http://localhost:8080/

# Multiple headers
wrk -H "Authorization: Bearer token" \
    -H "Content-Type: application/json" \
    http://localhost:8080/

# Specify HTTP method
wrk -s post.lua http://localhost:8080/
```

### Timeout and Rates
```bash
# Set timeout
wrk -t2 -c100 -d30s --timeout 30s http://localhost:8080/

# Limit request rate
wrk -t2 -c100 -d30s --rate 1000 http://localhost:8080/

# Connection timeout
wrk -t2 -c100 -d30s --connect-timeout 5s http://localhost:8080/
```

## Scripting

### Basic Lua Script
```lua
-- request.lua
wrk.method = "POST"
wrk.body   = '{"key": "value"}'
wrk.headers["Content-Type"] = "application/json"
```

### Advanced Scripts
```lua
-- advanced.lua
counter = 0
request = function()
   path = "/item/" .. counter
   counter = counter + 1
   return wrk.format("GET", path)
end

response = function(status, headers, body)
   if status ~= 200 then
      print("Error: " .. status)
   end
end
```

### Random Data
```lua
-- random.lua
math.randomseed(os.time())
request = function()
   id = math.random(1, 100)
   path = "/api/items/" .. id
   return wrk.format("GET", path)
end
```

## Results Analysis

### Understanding Output
```bash
Running 30s test @ http://localhost:8080/
  12 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    45.12ms   23.45ms  123.45ms   70.25%
    Req/Sec     1.23k   145.23     1.45k    75.00%
  70234 requests in 30.01s, 10.25MB read
  Socket errors: connect 0, read 0, write 0, timeout 12
Requests/sec:   2341.23
Transfer/sec:    349.23KB
```

### Custom Reporting
```lua
-- report.lua
done = function(summary, latency, requests)
   io.write("------------------------------\n")
   io.write("Test Summary:\n")
   io.write(string.format("  Requests/sec: %.2f\n", requests.rate))
   io.write(string.format("  Transfer/sec: %.2f\n", requests.bytes))
   io.write(string.format("  Avg Latency: %.2fms\n", latency.mean))
   io.write("------------------------------\n")
end
```

## Best Practices

### Load Testing Guidelines
```bash
# Warm-up run
wrk -t2 -c10 -d10s http://localhost:8080/

# Gradual load increase
wrk -t2 -c100 -d30s http://localhost:8080/
wrk -t4 -c200 -d30s http://localhost:8080/
wrk -t8 -c400 -d30s http://localhost:8080/
```

### Resource Management
```bash
# Monitor system resources
vmstat 1

# Watch network connections
watch -n1 'netstat -an | grep ESTABLISHED | wc -l'

# Monitor target system
top -b -n 1
```

## Common Scenarios

### REST API Testing
```lua
-- api_test.lua
wrk.method = "POST"
wrk.body   = '{"username": "test", "password": "test123"}'
wrk.headers["Content-Type"] = "application/json"
wrk.headers["Authorization"] = "Bearer token"
```

### Load Testing with Authentication
```lua
-- auth_test.lua
token = nil

setup = function(thread)
   -- Perform login and get token
   local auth_body = '{"username":"test","password":"test123"}'
   local auth_headers = "Content-Type: application/json"
   -- Store token for requests
   token = "Bearer " .. response.token
end

request = function()
   return wrk.format("GET", "/api/data", {
      ["Authorization"] = token,
      ["Content-Type"] = "application/json"
   })
end
```

### Performance Testing
```lua
-- performance.lua
requests = 0
errors = 0

request = function()
   requests = requests + 1
   return wrk.format("GET", "/")
end

response = function(status, headers, body)
   if status ~= 200 then
      errors = errors + 1
   end
end

done = function(summary, latency, requests)
   io.write(string.format("Requests: %d\n", requests))
   io.write(string.format("Errors: %d\n", errors))
end
```

## Quick Reference

### Essential Commands
```bash
# Basic test
wrk -t2 -c100 -d30s http://localhost:8080/

# With script
wrk -t2 -c100 -d30s -s script.lua http://localhost:8080/

# With headers
wrk -H "Key: Value" http://localhost:8080/

# With latency stats
wrk --latency http://localhost:8080/
```

### Common Options
```bash
-c    # Connections to keep open
-d    # Duration of test
-t    # Number of threads to use
-s    # Load Lua script
-H    # Add header
--latency    # Print detailed latency statistics
--timeout    # Socket/request timeout
```

## Example Scripts

### Basic POST Request
```lua
-- post.lua
wrk.method = "POST"
wrk.body   = '{"test": "data"}'
wrk.headers["Content-Type"] = "application/json"
```

### Random Path Generator
```lua
-- random_path.lua
paths = {"/api/v1/users", "/api/v1/items", "/api/v1/orders"}

request = function()
   local path = paths[math.random(#paths)]
   return wrk.format("GET", path)
end
```

---

Remember:
- Start with lower loads
- Monitor system resources
- Analyze results carefully
- Consider target system capacity
- Document test scenarios
- Use appropriate timeouts

For detailed information, consult the wrk GitHub repository and documentation.