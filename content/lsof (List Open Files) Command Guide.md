
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Network Connections](#network-connections)
- [Process Analysis](#process-analysis)
- [File System Operations](#file-system-operations)
- [Advanced Options](#advanced-options)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

## Overview

`lsof` (List Open Files) is a powerful diagnostic tool that provides information about files opened by processes. It's essential for system administration, debugging, and security analysis.

### Key Features
- List open files
- Monitor network connections
- Track process file usage
- Identify deleted files
- Memory mapping analysis
- Port monitoring

## Installation

### Ubuntu (22.04/24.04)
```bash
sudo apt update
sudo apt install lsof
```

### macOS
```bash
# Pre-installed on macOS
# Or install via Homebrew
brew install lsof
```

## Basic Usage

### List All Open Files
```bash
# List all open files
sudo lsof

# List files with statistics
sudo lsof -s

# Count number of open files
sudo lsof | wc -l
```

### Filter by User
```bash
# Files opened by user
lsof -u username

# Files NOT opened by user
lsof -u ^username

# Files opened by multiple users
lsof -u user1,user2
```

### Process Specific
```bash
# Files opened by PID
lsof -p 1234

# Files opened by multiple PIDs
lsof -p 1234,5678

# Exclude PIDs
lsof -p ^1234
```

## Network Connections

### Network Related Commands
```bash
# Show all network connections
sudo lsof -i

# Show TCP connections
sudo lsof -i tcp

# Show UDP connections
sudo lsof -i udp

# Show specific port
sudo lsof -i :80
```

### Network Filters
```bash
# Specific IP address
sudo lsof -i@192.168.1.100

# Specific port range
sudo lsof -i :1-1024

# Specific protocol and port
sudo lsof -i tcp:22

# Show listening ports
sudo lsof -i -sTCP:LISTEN
```

### IPv4/IPv6
```bash
# Show only IPv4
sudo lsof -i 4

# Show only IPv6
sudo lsof -i 6

# Show both
sudo lsof -i 4 -i 6
```

## Process Analysis

### Process Specific Information
```bash
# Files opened by command name
lsof -c nginx

# Multiple command names
lsof -c nginx -c apache2

# Case insensitive search
lsof -c /nginx/i
```

### Process States
```bash
# Show running processes
lsof -s TCP:ESTABLISHED

# Show listening processes
lsof -s TCP:LISTEN

# Show specific states
lsof -s TCP:CLOSE_WAIT
```

### Process Tree
```bash
# Show parent processes
lsof -R

# Show process group ID
lsof -g

# Show process owner
lsof -l
```

## File System Operations

### File and Directory Operations
```bash
# Files in directory
lsof +D /path/to/directory

# Recursive directory search
lsof +d /path/to/directory

# Specific file
lsof /path/to/file
```

### File Types
```bash
# Show regular files
lsof -F f

# Show directory files
lsof -F d

# Show character special files
lsof -F c
```

### Deleted Files
```bash
# Show deleted files
lsof +L1

# Show files with link count 0
lsof +L0

# Files deleted but still open
lsof | grep deleted
```

## Advanced Options

### Output Format
```bash
# Custom field selection
lsof -F pcfn

# Field separator
lsof -F0

# Wide output
lsof -w
```

### Time Options
```bash
# Repeat mode
lsof -r 5

# Repeat until no files
lsof +r 5

# Timeout
lsof -S 2
```

### Memory Operations
```bash
# Show memory maps
lsof -m

# Show file sizes
lsof -s

# Show offset
lsof -o
```

## Troubleshooting

### Common Issues

1. **Permission Issues**
```bash
# Run with sudo
sudo lsof

# Check specific user permissions
sudo lsof -u username
```

2. **Process Access**
```bash
# Check blocked processes
sudo lsof -b

# Avoid blocking
sudo lsof -n
```

3. **Network Issues**
```bash
# Check specific port
sudo lsof -i :port_number

# Check all network activity
sudo lsof -i -n
```

## Best Practices

### Performance Optimization
```bash
# Avoid DNS lookups
lsof -n

# Avoid port name lookups
lsof -P

# Combine optimizations
lsof -nP
```

### Security Monitoring
```bash
# Monitor suspicious ports
sudo lsof -i :22,80,443

# Check unknown processes
sudo lsof -i | grep ESTABLISHED

# Monitor file changes
watch -n 1 'lsof | wc -l'
```

## Quick Reference

### Essential Commands
```bash
# Basic listing
sudo lsof

# Network connections
sudo lsof -i

# Process files
lsof -p PID

# User files
lsof -u username

# Port check
sudo lsof -i :port
```

### Common Options
```bash
-i    # Internet connections
-p    # Process ID
-u    # Username
-c    # Command name
+D    # Directory
-n    # No DNS lookup
-P    # No port names
-r    # Repeat mode
```

## Example Use Cases

### Web Server Monitoring
```bash
# Monitor web server ports
sudo lsof -i :80,443

# Check web server processes
sudo lsof -c apache2 -c nginx

# Monitor web log access
sudo lsof | grep access.log
```

### System Troubleshooting
```bash
# Find largest open files
sudo lsof -s | sort -nr -k7 | head -10

# Check for deleted but open files
sudo lsof +L1

# Monitor specific application
watch -n 1 'lsof -c application_name'
```

---

Remember:
- Use sudo when needed
- Consider performance impact
- Filter output appropriately
- Regular monitoring
- Document findings
- Use with other tools

For detailed information, consult the man pages (`man lsof`).