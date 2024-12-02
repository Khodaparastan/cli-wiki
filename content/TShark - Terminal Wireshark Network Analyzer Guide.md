
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Capture Filters](#capture-filters)
- [Display Filters](#display-filters)
- [Output Formats](#output-formats)
- [Advanced Features](#advanced-features)
- [Analysis Techniques](#analysis-techniques)
- [Best Practices](#best-practices)

## Overview

TShark is the command-line version of Wireshark, providing powerful network protocol analysis capabilities without a GUI interface.

### Key Features
- Live packet capture
- Capture file reading
- Protocol analysis
- Advanced filtering
- Multiple output formats
- Statistics generation
- Decryption support
- Remote capture

## Installation

### Ubuntu (22.04/24.04)
```bash
# Install tshark
sudo apt update
sudo apt install tshark

# Allow non-root capture
sudo setcap cap_net_raw,cap_net_admin+eip $(which tshark)
```

### macOS
```bash
# Using Homebrew
brew install wireshark
```

## Basic Usage

### Basic Capture
```bash
# Capture on all interfaces
tshark

# Capture on specific interface
tshark -i eth0

# Capture with packet count
tshark -i eth0 -c 100

# Capture with duration
tshark -i eth0 -a duration:60
```

### Reading Files
```bash
# Read pcap file
tshark -r capture.pcap

# Read and apply filter
tshark -r capture.pcap -Y "http"

# Read specific protocols
tshark -r capture.pcap -Y "tcp.port == 80"
```

## Capture Filters

### Protocol Filters
```bash
# Capture TCP traffic
tshark -i eth0 -f "tcp"

# Capture specific port
tshark -i eth0 -f "port 80"

# Capture multiple ports
tshark -i eth0 -f "port 80 or port 443"
```

### Host Filters
```bash
# Capture specific host
tshark -i eth0 -f "host 192.168.1.1"

# Capture subnet
tshark -i eth0 -f "net 192.168.1.0/24"

# Capture source/destination
tshark -i eth0 -f "src host 192.168.1.1"
tshark -i eth0 -f "dst host 192.168.1.1"
```

### Combined Filters
```bash
# Protocol and host
tshark -i eth0 -f "tcp and host 192.168.1.1"

# Complex filters
tshark -i eth0 -f "tcp and port 80 and not host 192.168.1.1"
```

## Display Filters

### Protocol Analysis
```bash
# HTTP traffic
tshark -i eth0 -Y "http"

# HTTPS traffic
tshark -i eth0 -Y "ssl"

# DNS queries
tshark -i eth0 -Y "dns"
```

### Connection States
```bash
# TCP SYN packets
tshark -i eth0 -Y "tcp.flags.syn==1"

# Established connections
tshark -i eth0 -Y "tcp.flags.syn==1 && tcp.flags.ack==1"

# Connection problems
tshark -i eth0 -Y "tcp.analysis.retransmission"
```

### Application Layer
```bash
# HTTP GET requests
tshark -i eth0 -Y "http.request.method==GET"

# HTTP response codes
tshark -i eth0 -Y "http.response.code==404"

# DNS queries for domain
tshark -i eth0 -Y "dns.qry.name contains example.com"
```

## Output Formats

### Field Selection
```bash
# Show specific fields
tshark -i eth0 -T fields -e frame.time -e ip.src -e ip.dst

# Custom field format
tshark -i eth0 -T fields -e frame.time_epoch -e ip.src -e ip.dst -E header=y -E separator=,

# JSON output
tshark -i eth0 -T json
```

### Statistics
```bash
# Protocol hierarchy
tshark -r capture.pcap -q -z io,phs

# Conversation statistics
tshark -r capture.pcap -q -z conv,tcp

# HTTP statistics
tshark -r capture.pcap -q -z http,tree
```

### Packet Details
```bash
# Verbose output
tshark -i eth0 -V

# Specific protocols
tshark -i eth0 -O http

# Custom columns
tshark -i eth0 -T fields -e frame.time -e ip.src -e ip.dst -e http.request.method
```

## Advanced Features

### Decryption
```bash
# SSL decryption with key
tshark -i eth0 -o "ssl.keys_list:192.168.1.1,443,http,server.key"

# WPA decryption
tshark -i eth0 -o "wlan.enable_decryption:TRUE" -o "wlan.wep_key1:key"
```

### Remote Capture
```bash
# Capture from remote host
tshark -i SSH@192.168.1.1:eth0

# Save remote capture
tshark -i SSH@192.168.1.1:eth0 -w remote_capture.pcap
```

### Ring Buffer
```bash
# Rotating capture files
tshark -i eth0 -b filesize:1000 -b files:5 -w capture.pcap

# Time-based rotation
tshark -i eth0 -b duration:3600 -b files:24 -w capture.pcap
```

## Analysis Techniques

### Traffic Analysis
```bash
# Top talkers
tshark -r capture.pcap -q -z ip,endpoints

# Protocol distribution
tshark -r capture.pcap -q -z io,phs

# Connection analysis
tshark -r capture.pcap -q -z conv,tcp
```

### Security Analysis
```bash
# Find suspicious traffic
tshark -r capture.pcap -Y "http.request.method==POST && http.file_data contains password"

# Detect port scans
tshark -r capture.pcap -q -z endpoints,tcp

# Check for malformed packets
tshark -r capture.pcap -Y "malformed"
```

## Best Practices

### Capture Management
```bash
# Efficient capture
tshark -i eth0 -f "host 192.168.1.1" -w capture.pcap -b filesize:1000

# Memory management
tshark -i eth0 -B 2 -w capture.pcap

# Performance optimization
tshark -i eth0 -p -q -w capture.pcap
```

### Analysis Tips
```bash
# Quick overview
tshark -r capture.pcap -q -z io,phs

# Detailed analysis
tshark -r capture.pcap -V -Y "http"

# Performance analysis
tshark -r capture.pcap -q -z expert
```

## Quick Reference

### Essential Commands
```bash
# Basic capture
tshark -i eth0

# Read file
tshark -r capture.pcap

# Apply filter
tshark -Y "http"

# Save capture
tshark -w capture.pcap
```

### Common Options
```bash
-i    # Interface
-f    # Capture filter
-Y    # Display filter
-r    # Read file
-w    # Write file
-V    # Verbose
-T    # Output format
-z    # Statistics
```

---

Remember:
- Consider capture file size
- Use appropriate filters
- Monitor system resources
- Secure sensitive data
- Regular cleanup of captures
- Document analysis findings

For detailed information, consult the man pages (`man tshark`).