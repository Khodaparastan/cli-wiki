
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Syntax](#basic-syntax)
- [Common Use Cases](#common-use-cases)
- [Advanced Options](#advanced-options)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## Overview
iPerf3 is a tool for active measurements of the maximum achievable bandwidth on IP networks. It supports tuning various parameters related to timing, buffers, and protocols (TCP, UDP, SCTP).

### Key Features
- Measure bandwidth, loss, and jitter
- Support for IPv4 and IPv6
- Client and server functionality
- Multiple simultaneous connections
- TCP window size adjustment
- UDP bandwidth specification

## Installation

### Ubuntu (22.04/24.04)
```bash
sudo apt update
sudo apt install iperf3
```

### MacOS

```bash
brew install iperf3
```

## Basic Syntax

### Server Mode
```bash
# Basic server
iperf3 -s

# Server on specific port
iperf3 -s -p 5201

# Server with detailed output
iperf3 -s -V
```

### Client Mode
```bash
# Basic client test
iperf3 -c SERVER_IP

# Client with specific duration (in seconds)
iperf3 -c SERVER_IP -t 30

# Client with specific port
iperf3 -c SERVER_IP -p 5201
```

## Common Use Cases

### 1. TCP Bandwidth Test
```bash
# Server
iperf3 -s

# Client
iperf3 -c SERVER_IP -t 30 -i 1
```
- `-t 30`: Run for 30 seconds
- `-i 1`: Output interval every 1 second

### 2. UDP Bandwidth Test
```bash
# Server
iperf3 -s

# Client
iperf3 -c SERVER_IP -u -b 100M
```
- `-u`: Use UDP
- `-b 100M`: Set bandwidth target to 100 Mbits/sec

### 3. Multiple Parallel Streams
```bash
# Test with 10 parallel streams
iperf3 -c SERVER_IP -P 10
```

### 4. Reverse Mode Test
```bash
# Server sends, client receives
iperf3 -c SERVER_IP -R
```

## Advanced Options

### TCP Window Size
```bash
# Set TCP window size
iperf3 -c SERVER_IP -w 256K
```

### JSON Output
```bash
# Output results in JSON format
iperf3 -c SERVER_IP -J
```

### Bidirectional Test
```bash
# Run bidirectional test
iperf3 -c SERVER_IP --bidir
```

## Best Practices

1. **Testing Methodology**
   - Always run multiple tests
   - Test at different times of day
   - Use consistent test durations
   - Monitor system resources during tests

2. **Security Considerations**
   ```bash
   # Bind to specific IP
   iperf3 -s --bind SERVER_IP
   
   # Set authentication
   iperf3 -s --rsa-private-key-path /path/to/private_key
   ```

3. **Performance Tips**
   - Disable system sleep during tests
   - Close unnecessary applications
   - Monitor CPU usage
   - Consider network conditions

## Troubleshooting

### Common Issues and Solutions

1. **Connection Refused**
   ```bash
   # Check if server is running
   ps aux | grep iperf3
   
   # Verify port is open
   ss -tulpn | grep 5201
   ```

2. **Poor Performance**
   ```bash
   # Check CPU usage
   top -n 1
   
   # Monitor network interface
   ifconfig INTERFACE_NAME
   ```

3. **Permission Issues**
   ```bash
   # Check firewall rules
   sudo ufw status
   
   # Check port availability
   sudo lsof -i :5201
   ```

### Debug Mode
```bash
# Run with debug output
iperf3 -s -d
iperf3 -c SERVER_IP -d
```

## Example Test Scenarios

### 1. Network Baseline Test
```bash
# Run a 5-minute test with 1-second intervals
iperf3 -c SERVER_IP -t 300 -i 1
```

### 2. Maximum Performance Test
```bash
# Multiple streams with maximum buffer size
iperf3 -c SERVER_IP -P 10 -w 2M
```

### 3. Network Stability Test
```bash
# Long duration test with JSON output
iperf3 -c SERVER_IP -t 3600 -J > network_test.json
```

---

## Quick Reference

### Essential Commands
```bash
# Start server
iperf3 -s

# Basic client test
iperf3 -c SERVER_IP

# UDP test
iperf3 -c SERVER_IP -u

# Reverse mode
iperf3 -c SERVER_IP -R

# JSON output
iperf3 -c SERVER_IP -J

# Multiple streams
iperf3 -c SERVER_IP -P 4
```

### Common Flags
- `-s`: Server mode
- `-c`: Client mode
- `-p`: Port number
- `-t`: Time duration
- `-i`: Interval timing
- `-u`: UDP mode
- `-b`: Bandwidth target
- `-R`: Reverse mode
- `-P`: Parallel streams
- `-J`: JSON output
- `-V`: Verbose output
