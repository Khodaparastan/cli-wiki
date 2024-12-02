
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Scan Types](#scan-types)
- [Rate Control](#rate-control)
- [Output Options](#output-options)
- [Advanced Features](#advanced-features)
- [Best Practices](#best-practices)

## Overview

Masscan is one of the fastest port scanners available, capable of scanning the entire Internet in under 6 minutes while transmitting 10 million packets per second.

### Key Features
- Ultra-high-speed scanning
- Custom packet generation
- Banner grabbing
- Multiple output formats
- Rate control
- IPv4/IPv6 support
- TCP/UDP scanning
- Script support

## Installation

### Ubuntu (22.04/24.04)
```bash
# Install from repository
sudo apt update
sudo apt install masscan

# Build from source
git clone https://github.com/robertdavidgraham/masscan
cd masscan
make
sudo make install
```

### macOS
```bash
# Using Homebrew
brew install masscan
```

## Basic Usage

### Simple Scans
```bash
# Basic port scan
sudo masscan -p80 192.168.1.0/24

# Multiple ports
sudo masscan -p22,80,443 192.168.1.0/24

# Port ranges
sudo masscan -p1-1000 192.168.1.0/24

# Specific interface
sudo masscan -p80 192.168.1.0/24 --interface eth0
```

### Common Options
```bash
# Set rate limit
sudo masscan -p80 192.168.1.0/24 --rate 1000

# Include banners
sudo masscan -p80 192.168.1.0/24 --banners

# Exclude hosts
sudo masscan -p80 192.168.1.0/24 --excludefile exclude.txt
```

## Scan Types

### TCP Scans
```bash
# SYN scan
sudo masscan -p80 192.168.1.0/24 --tcp-flags syn

# FIN scan
sudo masscan -p80 192.168.1.0/24 --tcp-flags fin

# Custom flags
sudo masscan -p80 192.168.1.0/24 --tcp-flags syn,ack
```

### UDP Scans
```bash
# UDP scan
sudo masscan -pU:53 192.168.1.0/24

# Combined TCP/UDP
sudo masscan -p U:53,T:80 192.168.1.0/24
```

### Service Detection
```bash
# Banner grabbing
sudo masscan -p80,443 192.168.1.0/24 --banners

# HTTP detection
sudo masscan -p80 192.168.1.0/24 --http-user-agent "Mozilla/5.0"

# SSL/TLS detection
sudo masscan -p443 192.168.1.0/24 --banners --heartbleed
```

## Rate Control

### Speed Settings
```bash
# Packets per second
sudo masscan -p80 192.168.1.0/24 --rate 10000

# Adaptive timing
sudo masscan -p80 192.168.1.0/24 --rate-control

# Maximum rate
sudo masscan -p80 192.168.1.0/24 --max-rate 100000
```

### Timing Controls
```bash
# Wait time
sudo masscan -p80 192.168.1.0/24 --wait 0

# Connection timeout
sudo masscan -p80 192.168.1.0/24 --connection-timeout 30

# Retry count
sudo masscan -p80 192.168.1.0/24 --retries 2
```

## Output Options

### File Formats
```bash
# Binary output
sudo masscan -p80 192.168.1.0/24 -oB scan.bin

# XML output
sudo masscan -p80 192.168.1.0/24 -oX scan.xml

# JSON output
sudo masscan -p80 192.168.1.0/24 -oJ scan.json

# Simple list
sudo masscan -p80 192.168.1.0/24 -oL scan.txt
```

### Custom Output
```bash
# Grepable output
sudo masscan -p80 192.168.1.0/24 --output-format grepable

# Custom format
sudo masscan -p80 192.168.1.0/24 --output-format "ip port protocol state"
```

## Advanced Features

### Target Selection
```bash
# Multiple ranges
sudo masscan -p80 192.168.1.0/24 10.0.0.0/8

# Include file
sudo masscan -p80 -iL targets.txt

# Random targets
sudo masscan -p80 0.0.0.0/0 --excludefile exclude.txt
```

### Advanced Configuration
```bash
# Custom source port
sudo masscan -p80 192.168.1.0/24 --source-port 61000

# Router MAC address
sudo masscan -p80 192.168.1.0/24 --router-mac 00:11:22:33:44:55

# IPv6 support
sudo masscan -p80 2001:db8::/64
```

## Best Practices

### Scanning Guidelines
```bash
# Safe scanning
sudo masscan -p80,443 192.168.1.0/24 \
    --rate 1000 \
    --excludefile exclude.txt \
    --wait 0

# Thorough scan
sudo masscan -p1-65535 192.168.1.0/24 \
    --rate 5000 \
    --banners \
    --retries 2
```

### Resource Management
```bash
# Efficient scanning
sudo masscan -p80 192.168.1.0/24 \
    --rate 10000 \
    --max-rate 50000 \
    --connection-timeout 30
```

## Quick Reference

### Essential Commands
```bash
# Basic scan
sudo masscan -p80 192.168.1.0/24

# Multiple ports
sudo masscan -p22,80,443 192.168.1.0/24

# Rate control
sudo masscan -p80 192.168.1.0/24 --rate 1000

# Output to file
sudo masscan -p80 192.168.1.0/24 -oX scan.xml
```

### Common Options
```bash
-p           # Port specification
--rate      # Packet rate
--banners   # Banner grabbing
-oX         # XML output
-oJ         # JSON output
--interface # Network interface
--exclude   # Exclude targets
```

## Example Scripts

### Network Discovery
```bash
#!/bin/bash
# Comprehensive network discovery
OUTPUT_DIR="masscan_results"
NETWORK="192.168.1.0/24"
PORTS="21,22,23,25,80,443,3306,3389"

mkdir -p "$OUTPUT_DIR"

# Run scan with multiple output formats
sudo masscan -p$PORTS "$NETWORK" \
    --rate 1000 \
    --banners \
    -oX "$OUTPUT_DIR/scan.xml" \
    -oJ "$OUTPUT_DIR/scan.json" \
    --output-format grepable > "$OUTPUT_DIR/scan.grep"
```

### Service Detection
```bash
#!/bin/bash
# Service detection script
TARGET="192.168.1.0/24"
OUTPUT_DIR="service_detection"

mkdir -p "$OUTPUT_DIR"

# Web services
sudo masscan -p80,443 "$TARGET" \
    --banners \
    --http-user-agent "Mozilla/5.0" \
    -oJ "$OUTPUT_DIR/web_services.json"

# Database services
sudo masscan -p3306,5432,1521,1433 "$TARGET" \
    --banners \
    -oJ "$OUTPUT_DIR/db_services.json"

# Remote access
sudo masscan -p22,3389,5900 "$TARGET" \
    --banners \
    -oJ "$OUTPUT_DIR/remote_access.json"
```

### Security Audit
```bash
#!/bin/bash
# Security audit scanning
TARGET_NETWORK="192.168.1.0/24"
EXCLUDE_FILE="exclude.txt"
OUTPUT_DIR="security_audit"

mkdir -p "$OUTPUT_DIR"

# Common vulnerable ports
VULN_PORTS="21,23,445,3389,5900"

# Run security scan
sudo masscan -p$VULN_PORTS "$TARGET_NETWORK" \
    --rate 500 \
    --banners \
    --excludefile "$EXCLUDE_FILE" \
    --output-format "ip port protocol state banner" \
    > "$OUTPUT_DIR/security_scan.txt"

# Parse results
grep -i "vulnerable" "$OUTPUT_DIR/security_scan.txt" > "$OUTPUT_DIR/vulnerabilities.txt"
```

---

Remember:
- Obtain proper authorization
- Use appropriate scan rates
- Respect network limitations
- Document scan configurations
- Monitor system impact
- Handle results securely

For detailed information, consult the Masscan documentation (`man masscan`).