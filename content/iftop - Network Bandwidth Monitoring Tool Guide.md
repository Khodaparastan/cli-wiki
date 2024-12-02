
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Display Options](#display-options)
- [Filtering](#filtering)
- [Interface Selection](#interface-selection)
- [Advanced Features](#advanced-features)
- [Best Practices](#best-practices)

## Overview

`iftop` monitors bandwidth usage on an interface and displays a table of current bandwidth usage by pairs of hosts.

### Key Features
- Real-time bandwidth monitoring
- Per-connection bandwidth usage
- Cumulative bandwidth stats
- Port filtering
- Host filtering
- DNS resolution control
- Peak load display
- Directional traffic analysis

## Installation

### Ubuntu (22.04/24.04)
```bash
# Install iftop
sudo apt update
sudo apt install iftop
```

### macOS
```bash
# Using Homebrew
brew install iftop
```

## Basic Usage

### Simple Monitoring
```bash
# Monitor default interface
sudo iftop

# Monitor specific interface
sudo iftop -i eth0

# No DNS resolution
sudo iftop -n

# Show port numbers
sudo iftop -P
```

### Basic Options
```bash
# Show ports and no DNS
sudo iftop -nP

# Show bandwidth in bytes
sudo iftop -B

# Show ports and hostnames
sudo iftop -p
```

## Display Options

### Screen Layout
```bash
# Show/hide bar graphs
# Press 'b' while running

# Show/hide port numbers
# Press 'p' while running

# Toggle DNS resolution
# Press 'n' while running

# Show cumulative totals
# Press 't' while running
```

### Bandwidth Display
```bash
# Show bytes
sudo iftop -B

# Show bits
sudo iftop -b

# Show packets
sudo iftop -f "line contains packets"
```

### Sort Options
```bash
# Sort by source
# Press 's' while running

# Sort by destination
# Press 'd' while running

# Sort by bandwidth
# Press 'o' while running
```

## Filtering

### Host Filters
```bash
# Filter by host
sudo iftop -f "host 192.168.1.1"

# Filter by source
sudo iftop -f "src host 192.168.1.1"

# Filter by destination
sudo iftop -f "dst host 192.168.1.1"
```

### Port Filters
```bash
# Filter by port
sudo iftop -f "port 80"

# Filter multiple ports
sudo iftop -f "port 80 or port 443"

# Filter port range
sudo iftop -f "portrange 1-1024"
```

### Complex Filters
```bash
# Combine filters
sudo iftop -f "host 192.168.1.1 and port 80"

# Exclude traffic
sudo iftop -f "not port 22"

# Protocol specific
sudo iftop -f "tcp and port 80"
```

## Interface Selection

### Interface Options
```bash
# List interfaces
ip link show

# Monitor specific interface
sudo iftop -i eth0

# Monitor wireless interface
sudo iftop -i wlan0

# Monitor all interfaces
sudo iftop -i any
```

### Interface Stats
```bash
# Show interface bandwidth
sudo iftop -i eth0 -B

# Monitor specific subnet
sudo iftop -i eth0 -F 192.168.1.0/24

# Show interface errors
sudo iftop -i eth0 -e
```

## Advanced Features

### Output Control
```bash
# Line buffered output
sudo iftop -L

# Text interface only
sudo iftop -t

# Custom refresh interval
sudo iftop -u 5
```

### Traffic Analysis
```bash
# Show packet size
sudo iftop -h

# Peak traffic display
sudo iftop -P

# Show direction arrows
sudo iftop -a
```

### Logging
```bash
# Log to file
sudo iftop -t > bandwidth.log

# Continuous logging
sudo iftop -t | tee bandwidth.log

# Timestamped logging
sudo iftop -t | while read line; do
    echo "$(date): $line" >> bandwidth.log
done
```

## Best Practices

### Monitoring Guidelines
```bash
# Basic monitoring
sudo iftop -nP -i eth0

# Detailed analysis
sudo iftop -nPB -i eth0 -F 192.168.1.0/24

# Performance monitoring
sudo iftop -t -L -u 1
```

### Resource Usage
```bash
# Minimize DNS lookups
sudo iftop -n

# Reduce update frequency
sudo iftop -u 2

# Limit display items
sudo iftop -m 10
```

## Quick Reference

### Essential Commands
```bash
# Basic monitoring
sudo iftop

# No DNS resolution
sudo iftop -n

# Show ports
sudo iftop -P

# Specific interface
sudo iftop -i eth0
```

### Common Options
```bash
-n    # No DNS resolution
-P    # Show ports
-B    # Show bytes
-i    # Interface selection
-f    # Filter expression
-t    # Text mode
-p    # Show ports
```

## Example Scripts

### Basic Monitoring Script
```bash
#!/bin/bash
# Basic bandwidth monitoring
INTERFACE="eth0"
LOG_FILE="bandwidth.log"

sudo iftop -t -n -P -i "$INTERFACE" | \
while read line; do
    echo "$(date): $line" >> "$LOG_FILE"
done
```

### Traffic Analysis
```bash
#!/bin/bash
# Analyze specific traffic patterns
INTERFACE="eth0"
FILTER="port 80 or port 443"

sudo iftop -t -n -P -i "$INTERFACE" -f "$FILTER" | \
grep -v "monitoring" | \
while read line; do
    echo "$(date): $line"
done
```

### Network Usage Report
```bash
#!/bin/bash
# Generate network usage report
INTERFACE="eth0"
DURATION=3600  # 1 hour

sudo timeout $DURATION \
    iftop -t -n -P -B -i "$INTERFACE" | \
    grep "Total send rate" > usage_report.txt
```

### Continuous Monitoring
```bash
#!/bin/bash
# Continuous monitoring with rotation
INTERFACE="eth0"
LOG_DIR="iftop_logs"
ROTATE_SIZE="100M"

mkdir -p "$LOG_DIR"

while true; do
    TIMESTAMP=$(date +%Y%m%d_%H%M%S)
    LOG_FILE="$LOG_DIR/iftop_$TIMESTAMP.log"
    
    sudo iftop -t -n -P -i "$INTERFACE" > "$LOG_FILE" &
    PID=$!
    
    # Rotate logs when size exceeds limit
    while true; do
        sleep 60
        if [ $(stat -f%z "$LOG_FILE") -gt $(numfmt --from=iec $ROTATE_SIZE) ]; then
            kill $PID
            break
        fi
    done
done
```

---

Remember:
- Always run with sudo
- Consider DNS resolution impact
- Use appropriate filters
- Monitor resource usage
- Regular log rotation
- Document unusual patterns

For detailed information, consult the man pages (`man iftop`).