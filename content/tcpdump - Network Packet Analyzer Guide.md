
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Capture Filters](#capture-filters)
- [Display Filters](#display-filters)
- [Protocol Analysis](#protocol-analysis)
- [Advanced Features](#advanced-features)
- [Output Options](#output-options)
- [Best Practices](#best-practices)

## Overview

`tcpdump` is a powerful command-line packet analyzer that allows you to capture and analyze network traffic in real-time.

### Key Features
- Live packet capture
- Protocol analysis
- Filter expressions
- File capture/replay
- Detailed packet info
- Multiple output formats
- Interface selection
- Advanced filtering

## Installation

### Ubuntu (22.04/24.04)
```bash
# Install tcpdump
sudo apt update
sudo apt install tcpdump

# Allow non-root capture
sudo setcap cap_net_raw,cap_net_admin+eip $(which tcpdump)
```

### macOS
```bash
# Pre-installed on macOS
# Or using Homebrew
brew install tcpdump
```

## Basic Usage

### Simple Capture
```bash
# Basic capture
sudo tcpdump

# Capture on specific interface
sudo tcpdump -i eth0

# Show verbose output
sudo tcpdump -v

# Show more detailed output
sudo tcpdump -vv
```

### Common Options
```bash
# Don't resolve hostnames
sudo tcpdump -n

# Don't resolve ports
sudo tcpdump -nn

# Show packet contents
sudo tcpdump -X

# Show packet contents (hex and ASCII)
sudo tcpdump -XX
```

## Capture Filters

### Host Filters
```bash
# Capture specific host
sudo tcpdump host 192.168.1.1

# Source host
sudo tcpdump src host 192.168.1.1

# Destination host
sudo tcpdump dst host 192.168.1.1
```

### Port Filters
```bash
# Capture specific port
sudo tcpdump port 80

# Source port
sudo tcpdump src port 80

# Destination port
sudo tcpdump dst port 80

# Port range
sudo tcpdump portrange 100-200
```

### Protocol Filters
```bash
# TCP traffic
sudo tcpdump tcp

# UDP traffic
sudo tcpdump udp

# ICMP traffic
sudo tcpdump icmp

# IPv6 traffic
sudo tcpdump ip6
```

## Display Filters

### Packet Size
```bash
# Capture packets larger than size
sudo tcpdump greater 1000

# Capture packets smaller than size
sudo tcpdump less 100

# Specific packet length
sudo tcpdump length 1500
```

### TCP Flags
```bash
# TCP SYN packets
sudo tcpdump 'tcp[tcpflags] & (tcp-syn) != 0'

# TCP ACK packets
sudo tcpdump 'tcp[tcpflags] & (tcp-ack) != 0'

# TCP RST packets
sudo tcpdump 'tcp[tcpflags] & (tcp-rst) != 0'
```

### Complex Filters
```bash
# HTTP GET requests
sudo tcpdump 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'

# DNS queries
sudo tcpdump 'udp port 53'

# SSH traffic
sudo tcpdump 'tcp port 22'
```

## Protocol Analysis

### HTTP Traffic
```bash
# Capture HTTP traffic
sudo tcpdump -A 'tcp port 80'

# HTTP headers only
sudo tcpdump -A 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'

# HTTP POST requests
sudo tcpdump -A 'tcp port 80 and (tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x504F5354)'
```

### DNS Analysis
```bash
# DNS queries and responses
sudo tcpdump -n 'udp port 53'

# DNS queries only
sudo tcpdump -n 'udp port 53 and udp[10] & 0x80 = 0'

# DNS responses only
sudo tcpdump -n 'udp port 53 and udp[10] & 0x80 != 0'
```

### SSL/TLS Traffic
```bash
# Capture SSL/TLS
sudo tcpdump 'tcp port 443'

# SSL/TLS handshake
sudo tcpdump 'tcp port 443 and (tcp[((tcp[12] & 0xf0) >> 2)] = 0x16)'

# SSL/TLS application data
sudo tcpdump 'tcp port 443 and (tcp[((tcp[12] & 0xf0) >> 2)] = 0x17)'
```

## Advanced Features

### File Operations
```bash
# Write to file
sudo tcpdump -w capture.pcap

# Read from file
sudo tcpdump -r capture.pcap

# Split capture files
sudo tcpdump -w capture_%Y%m%d_%H%M%S.pcap -G 3600

# Rotate files
sudo tcpdump -w capture.pcap -C 1
```

### Buffer Management
```bash
# Set buffer size
sudo tcpdump -B 4096

# Set snapshot length
sudo tcpdump -s 1500

# No buffer
sudo tcpdump -U
```

### Time Stamps
```bash
# Unix timestamp
sudo tcpdump -tt

# Delta between packets
sudo tcpdump -ttt

# Human readable
sudo tcpdump -tttt
```

## Output Options

### Format Control
```bash
# Verbose ASCII
sudo tcpdump -A

# Hex and ASCII
sudo tcpdump -XX

# Print less protocol info
sudo tcpdump -q

# Custom format
sudo tcpdump -l | awk '{print $1, $3}'
```

### Packet Count
```bash
# Limit packet count
sudo tcpdump -c 100

# Print packet count summary
sudo tcpdump -v | grep "packets captured"

# Count packets by type
sudo tcpdump -v | grep -c "TCP"
```

## Best Practices

### Capture Guidelines
```bash
# Efficient capture
sudo tcpdump -n -i eth0 -s 0 -w capture.pcap

# Monitor specific traffic
sudo tcpdump -n -i eth0 port 80 or port 443

# Debug capture
sudo tcpdump -v -x -X -s 0
```

### Performance Tips
```bash
# Disable name resolution
sudo tcpdump -nn

# Optimize buffer
sudo tcpdump -B 4096 -i eth0

# Limit snapshot length
sudo tcpdump -s 96 -i eth0
```

## Quick Reference

### Essential Commands
```bash
# Basic capture
sudo tcpdump -i eth0

# Write to file
sudo tcpdump -w capture.pcap

# Read from file
sudo tcpdump -r capture.pcap

# No DNS resolution
sudo tcpdump -n
```

### Common Options
```bash
-i    # Interface
-w    # Write to file
-r    # Read from file
-n    # No DNS resolution
-v    # Verbose
-X    # Hex output
-A    # ASCII output
-c    # Packet count
```

## Example Scripts

### Traffic Monitor
```bash
#!/bin/bash
# Monitor specific traffic
INTERFACE="eth0"
OUTPUT_DIR="tcpdump_captures"

mkdir -p "$OUTPUT_DIR"

# Capture with rotation
sudo tcpdump -i "$INTERFACE" \
    -w "$OUTPUT_DIR/capture_%Y%m%d_%H%M%S.pcap" \
    -G 3600 \
    port 80 or port 443
```

### Protocol Analysis
```bash
#!/bin/bash
# Analyze specific protocols
INTERFACE="eth0"
PROTOCOLS=("tcp port 80" "tcp port 443" "udp port 53")

for proto in "${PROTOCOLS[@]}"; do
    echo "Analyzing $proto..."
    sudo tcpdump -i "$INTERFACE" -nn -c 100 "$proto"
    sleep 1
done
```

### Security Monitoring
```bash
#!/bin/bash
# Monitor suspicious traffic
INTERFACE="eth0"
LOG_FILE="security.log"

sudo tcpdump -i "$INTERFACE" -nn \
    'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0' \
    | while read line; do
        echo "$(date): $line" >> "$LOG_FILE"
    done
```

---

Remember:
- Use appropriate capture filters
- Consider storage space
- Monitor system resources
- Rotate capture files
- Handle sensitive data carefully
- Document capture conditions

For detailed information, consult the man pages (`man tcpdump`).