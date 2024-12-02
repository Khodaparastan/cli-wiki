

## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Syntax](#basic-syntax)
- [Common Use Cases](#common-use-cases)
- [Advanced Options](#advanced-options)
- [Modern Alternatives](#modern-alternatives)
- [Troubleshooting](#troubleshooting)

## Overview

`netstat` (network statistics) is a command-line tool for monitoring network connections, routing tables, interface statistics, and more. While considered legacy on modern systems, it remains widely used and available.

### Key Features
- Display network connections
- Show routing tables
- List interface statistics
- View network protocol statistics
- Monitor listening ports
- Show multicast memberships

## Installation

### Ubuntu (22.04/24.04)
```bash
sudo apt update
sudo apt install net-tools
```

### macOS
```bash
# Pre-installed on macOS
# No installation needed
```

## Basic Syntax

### Display All Connections
```bash
# Show all active connections
netstat -a

# Show all TCP connections
netstat -at

# Show all UDP connections
netstat -au
```

### Listening Ports
```bash
# Show listening ports (numeric)
netstat -ltn

# Show listening ports with process info
sudo netstat -ltpn
```

### Interface Statistics
```bash
# Show interface statistics
netstat -i

# Show extended interface statistics
netstat -ie
```

## Common Use Cases

### 1. Monitor TCP Connections
```bash
# Show all TCP connections with process info
sudo netstat -tap

# Show only established connections
netstat -tan | grep ESTABLISHED

# Count connections by state
netstat -ant | awk '{print $6}' | sort | uniq -c
```

### 2. Check Listening Services
```bash
# Show all listening TCP ports
sudo netstat -tlpn

# Show all listening UDP ports
sudo netstat -ulpn

# Show both TCP and UDP listening ports
sudo netstat -tulpn
```

### 3. Network Statistics
```bash
# Show protocol statistics
netstat -s

# Show TCP protocol statistics
netstat -st

# Show UDP protocol statistics
netstat -su
```

### 4. Routing Table Information
```bash
# Display routing table
netstat -r

# Display routing table with numeric addresses
netstat -rn
```

## Advanced Options

### Continuous Output
```bash
# Update statistics every 2 seconds
netstat -c

# Continuous display with specific interface
netstat -ci eth0
```

### Custom Display Format
```bash
# Show specific columns
netstat -an --tcp | awk '{print $4,$5,$6}'

# Show connection states with counts
netstat -ant | grep -v LISTEN | awk '{print $6}' | sort | uniq -c
```

### Socket Information
```bash
# Display all raw socket information
netstat --raw

# Show Unix domain sockets
netstat -x
```

## Modern Alternatives

### SS Command (Recommended)
```bash
# Equivalent to netstat -tulpn
ss -tulpn

# Show all TCP connections
ss -ta

# Show listening sockets
ss -lt
```

### IP Command
```bash
# Show interface statistics
ip -s link

# Show routing table
ip route
```

## Common Flags Reference

### Essential Options
- `-a`: Show all connections
- `-t`: TCP connections
- `-u`: UDP connections
- `-l`: Listening sockets
- `-p`: Show process information
- `-n`: Show numeric addresses
- `-r`: Show routing table
- `-s`: Show statistics
- `-i`: Show interface statistics
- `-c`: Continuous output
- `-v`: Verbose output

### Output Format
```bash
# Numeric output (no name resolution)
netstat -n

# Extended information
netstat -e

# Wide format display
netstat -W
```

## Troubleshooting

### Common Issues and Solutions

1. **Permission Denied**
```bash
# Use sudo for process information
sudo netstat -tulpn

# Check user permissions
id
```

2. **High CPU Usage**
```bash
# Use numeric output to avoid DNS lookups
netstat -n

# Limit output with grep
netstat -tan | grep ESTABLISHED
```

3. **Slow Command Response**
```bash
# Avoid name resolution
netstat -an

# Focus on specific protocol
netstat -tan
```

### Debug Tips

1. **Find Programs Using Ports**
```bash
# Check specific port usage
sudo netstat -tulpn | grep :80

# Find all programs with network connections
sudo netstat -tapn
```

2. **Monitor Connection States**
```bash
# Watch connection states
watch -n 1 'netstat -ant | grep -v LISTEN | awk "{print \$6}" | sort | uniq -c'
```

## Example Scripts

### Connection Monitor
```bash
#!/bin/bash
# Monitor connection count by state
while true; do
    clear
    echo "Connection States:"
    netstat -ant | awk '{print $6}' | sort | uniq -c
    sleep 2
done
```

### Port Scanner
```bash
#!/bin/bash
# Check common ports
for port in 80 443 22 21 25 3306; do
    netstat -an | grep ":$port " | grep LISTEN
done
```

## Best Practices

1. **Performance Considerations**
   - Use `-n` to avoid DNS lookups
   - Filter output with grep when possible
   - Use modern alternatives (ss) for better performance

2. **Security Monitoring**
   ```bash
   # Check for unusual ports
   netstat -tulpn | grep LISTEN
   
   # Monitor established connections
   netstat -antp | grep ESTABLISHED
   ```

3. **Regular Monitoring**
   - Monitor connection states
   - Track listening ports
   - Check for unauthorized services

## Quick Reference

### Most Used Commands
```bash
# List all listening ports
sudo netstat -tulpn

# Show all active connections
netstat -an

# Show process information
sudo netstat -tap

# Show routing table
netstat -r

# Show interface statistics
netstat -i
```

### Common Combinations
```bash
# Full TCP connection details
sudo netstat -tapn

# All listening services with processes
sudo netstat -tulpn

# Active internet connections
netstat -tun
```

Remember:
- Always use `sudo` when process information is needed
- Consider using modern alternatives like `ss` for better performance
- Use `-n` flag to speed up output when hostname resolution isn't needed

---

This guide covers the most common and useful netstat commands. For system-specific variations, always consult the man pages (`man netstat`).