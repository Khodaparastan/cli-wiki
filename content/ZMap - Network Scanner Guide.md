
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Scan Types](#scan-types)
- [Output Options](#output-options)
- [Bandwidth Control](#bandwidth-control)
- [Advanced Features](#advanced-features)
- [Best Practices](#best-practices)

## Overview

ZMap is a fast single-packet network scanner designed for Internet-wide network surveys. It can scan the entire IPv4 address space in under 45 minutes.

### Key Features
- High-speed scanning
- Bandwidth control
- Multiple probe types
- Flexible output formats
- Blacklist support
- Random permutation
- Gateway support
- Parallel scanning

## Installation

### Ubuntu (22.04/24.04)
```bash
# Install dependencies
sudo apt update
sudo apt install zmap

# Optional dependencies
sudo apt install redis-server mongodb
```

### Build from Source
```bash
# Install build dependencies
sudo apt install build-essential cmake libgmp3-dev gengetopt libpcap-dev flex byacc libjson-c-dev pkg-config libunistring-dev

# Clone and build
git clone https://github.com/zmap/zmap.git
cd zmap
cmake .
make -j4
sudo make install
```

## Basic Usage

### Simple Scans
```bash
# Basic TCP SYN scan
sudo zmap -p 80

# Scan specific network
sudo zmap -p 80 192.168.1.0/24

# Scan multiple ports
sudo zmap -p 80,443,22

# Specify interface
sudo zmap -p 80 -i eth0
```

### Rate Control
```bash
# Set bandwidth
sudo zmap -B 10M

# Set packets per second
sudo zmap -r 100

# Set number of probes
sudo zmap -N 1000
```

## Scan Types

### TCP Scans
```bash
# SYN scan
sudo zmap -p 80 --probe-module=tcp_synscan

# ACK scan
sudo zmap -p 80 --probe-module=tcp_ackscan

# SYN-ACK scan
sudo zmap --probe-module=tcp_synackscan
```

### UDP Scans
```bash
# UDP scan
sudo zmap --probe-module=udp

# DNS scan
sudo zmap -p 53 --probe-module=dns

# NTP scan
sudo zmap -p 123 --probe-module=ntp
```

### ICMP Scans
```bash
# ICMP echo scan
sudo zmap --probe-module=icmp_echoscan

# ICMP timestamp scan
sudo zmap --probe-module=icmp_timestampscan
```

## Output Options

### Basic Output
```bash
# CSV output
sudo zmap -p 80 -o results.csv

# Multiple fields
sudo zmap -p 80 -f "saddr,daddr,sport,dport,seqnum,acknum,window"

# JSON output
sudo zmap -p 80 --output-module=json
```

### Extended Output
```bash
# Include metadata
sudo zmap -p 80 --metadata-file=meta.json

# Output statistics
sudo zmap -p 80 --status-updates-file=status.txt

# Verbose logging
sudo zmap -p 80 -v
```

### Output Filtering
```bash
# Filter successful responses
sudo zmap -p 80 --output-filter="success = 1"

# Complex filtering
sudo zmap -p 80 --output-filter="success = 1 && repeat = 0"
```

## Bandwidth Control

### Rate Limiting
```bash
# Set bandwidth limit
sudo zmap -B 10M

# Set packet rate
sudo zmap -r 100

# Dynamic rate adjustment
sudo zmap --rate-limit-dynamic
```

### Probe Control
```bash
# Set number of probes
sudo zmap -N 1000

# Set cooldown time
sudo zmap --cooldown-time=10

# Set maximum runtime
sudo zmap --max-runtime=3600
```

## Advanced Features

### Target Selection
```bash
# Use whitelist
sudo zmap -p 80 --whitelist-file=targets.txt

# Use blacklist
sudo zmap -p 80 --blacklist-file=exclude.txt

# Random seed
sudo zmap -p 80 --seed=12345
```

### MAC Addressing
```bash
# Set source MAC
sudo zmap --source-mac=00:11:22:33:44:55

# Set gateway MAC
sudo zmap --gateway-mac=00:11:22:33:44:55
```

### Advanced Probing
```bash
# Custom probe module
sudo zmap --probe-module=custom

# Set probe args
sudo zmap --probe-args="arg1,arg2"

# Multiple probes
sudo zmap --probes=3
```

## Best Practices

### Scanning Guidelines
```bash
# Ethical scanning
sudo zmap -p 80 \
    --blacklist-file=exclude.txt \
    --bandwidth=1M \
    --max-runtime=3600

# Careful scanning
sudo zmap -p 80 \
    --rate-limit-dynamic \
    --cooldown-time=10 \
    --retries=2
```

### Resource Management
```bash
# Optimize performance
sudo zmap -p 80 \
    --cores=4 \
    --queue-size=10000 \
    --recv-queue-size=500000
```

## Quick Reference

### Essential Commands
```bash
# Basic scan
sudo zmap -p 80

# Bandwidth limited scan
sudo zmap -p 80 -B 10M

# Multiple port scan
sudo zmap -p 80,443,22

# Output to file
sudo zmap -p 80 -o results.csv
```

### Common Options
```bash
-p    # Port number
-B    # Bandwidth limit
-r    # Rate limit
-i    # Interface
-o    # Output file
-v    # Verbose
-q    # Quiet
```

## Example Scripts

### Network Survey
```bash
#!/bin/bash
# Comprehensive network survey
OUTPUT_DIR="zmap_results"
PORTS=(80 443 22 21 25 53)

mkdir -p "$OUTPUT_DIR"

for port in "${PORTS[@]}"; do
    echo "Scanning port $port..."
    sudo zmap -p "$port" \
        -B 10M \
        -o "$OUTPUT_DIR/port_${port}.csv" \
        --output-fields="saddr,success" \
        --metadata-file="$OUTPUT_DIR/meta_${port}.json"
done
```

### Service Discovery
```bash
#!/bin/bash
# Service discovery script
TARGET_NET="192.168.1.0/24"
SERVICES=(
    "80:HTTP"
    "443:HTTPS"
    "22:SSH"
    "3306:MySQL"
)

for service in "${SERVICES[@]}"; do
    port=${service%%:*}
    name=${service#*:}
    
    echo "Scanning for $name (port $port)..."
    sudo zmap -p "$port" \
        "$TARGET_NET" \
        -o "discovered_${name}.csv"
done
```

### Security Audit
```bash
#!/bin/bash
# Security audit scanning
OUTPUT_DIR="security_audit"
BANDWIDTH="5M"
BLACKLIST="blacklist.txt"

mkdir -p "$OUTPUT_DIR"

# Create blacklist if not exists
touch "$BLACKLIST"

# Scan common vulnerable ports
VULN_PORTS=(21 23 445 3389 5900)

for port in "${VULN_PORTS[@]}"; do
    echo "Scanning port $port..."
    sudo zmap -p "$port" \
        -B "$BANDWIDTH" \
        --blacklist-file="$BLACKLIST" \
        --output-module=json \
        -o "$OUTPUT_DIR/vuln_port_${port}.json"
done
```

---

Remember:
- Always obtain permission
- Use appropriate bandwidth
- Respect blacklists
- Monitor system resources
- Document scan parameters
- Handle results securely

For detailed information, consult the ZMap documentation (`man zmap`).