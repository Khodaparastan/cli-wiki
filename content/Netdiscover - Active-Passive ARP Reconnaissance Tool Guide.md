
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Scan Modes](#scan-modes)
- [Output Options](#output-options)
- [Advanced Features](#advanced-features)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## Overview

Netdiscover is an active/passive ARP reconnaissance tool, useful for network discovery and mapping local networks without the need for SNMP.

### Key Features
- Active and passive ARP scanning
- Network range scanning
- MAC vendor identification
- Fast host discovery
- Interface selection
- Custom scan ranges
- Multiple output formats
- Low network footprint

## Installation

### Ubuntu (22.04/24.04)
```bash
# Install netdiscover
sudo apt update
sudo apt install netdiscover
```

### Build from Source
```bash
# Install dependencies
sudo apt install build-essential libpcap-dev

# Clone and build
git clone https://github.com/netdiscover-scanner/netdiscover.git
cd netdiscover
./configure
make
sudo make install
```

## Basic Usage

### Simple Scans
```bash
# Basic scan
sudo netdiscover

# Scan specific range
sudo netdiscover -r 192.168.1.0/24

# Scan with specific interface
sudo netdiscover -i eth0

# Fast scan
sudo netdiscover -f
```

### Common Options
```bash
# Passive mode
sudo netdiscover -p

# Show only active hosts
sudo netdiscover -N

# Custom sleep time
sudo netdiscover -s 1
```

## Scan Modes

### Active Scanning
```bash
# Full range scan
sudo netdiscover -r 192.168.1.0/24

# Multiple ranges
sudo netdiscover -r 192.168.1.0/24,10.0.0.0/24

# Specific interface scan
sudo netdiscover -i eth0 -r 192.168.1.0/24
```

### Passive Scanning
```bash
# Basic passive mode
sudo netdiscover -p

# Passive with interface
sudo netdiscover -p -i eth0

# Passive with timeout
sudo netdiscover -p -L -t 3600
```

### Custom Scans
```bash
# Fast mode
sudo netdiscover -f -r 192.168.1.0/24

# Slow mode
sudo netdiscover -s 1 -r 192.168.1.0/24

# No header mode
sudo netdiscover -N -r 192.168.1.0/24
```

## Output Options

### Display Formats
```bash
# No header output
sudo netdiscover -N

# Quiet mode
sudo netdiscover -q

# Show packets
sudo netdiscover -d
```

### File Output
```bash
# Write to file
sudo netdiscover -r 192.168.1.0/24 > network_map.txt

# Append to file
sudo netdiscover -r 192.168.1.0/24 >> network_map.txt

# With timestamp
sudo netdiscover -r 192.168.1.0/24 | ts > network_map.txt
```

## Advanced Features

### Interface Control
```bash
# List interfaces
sudo netdiscover -l

# Multiple interfaces
sudo netdiscover -i eth0,wlan0

# Specific interface options
sudo netdiscover -i eth0 -f
```

### Range Control
```bash
# Custom range
sudo netdiscover -r 192.168.1.1-192.168.1.254

# Multiple subnets
sudo netdiscover -r 192.168.1.0/24,172.16.0.0/16

# Exclude ranges
sudo netdiscover -r 192.168.1.0/24 -x 192.168.1.1/32
```

### Timing Options
```bash
# Fast scan
sudo netdiscover -f

# Custom sleep time
sudo netdiscover -s 0.5

# Extended timeout
sudo netdiscover -t 3600
```

## Best Practices

### Scanning Guidelines
```bash
# Network discovery
sudo netdiscover -r 192.168.1.0/24 -f -N

# Passive monitoring
sudo netdiscover -p -i eth0 -L

# Thorough scan
sudo netdiscover -r 192.168.1.0/24 -s 1
```

### Resource Management
```bash
# Low impact scan
sudo netdiscover -s 2 -r 192.168.1.0/24

# Fast network mapping
sudo netdiscover -f -N -r 192.168.1.0/24

# Extended monitoring
sudo netdiscover -p -L -t 7200
```

## Quick Reference

### Essential Commands
```bash
# Basic scan
sudo netdiscover

# Range scan
sudo netdiscover -r 192.168.1.0/24

# Passive mode
sudo netdiscover -p

# Fast scan
sudo netdiscover -f
```

### Common Options
```bash
-r    # Scan range
-i    # Interface
-p    # Passive mode
-f    # Fast mode
-s    # Sleep time
-N    # No header
-L    # List mode
```

## Example Scripts

### Network Mapping Script
```bash
#!/bin/bash
# Comprehensive network mapping
OUTPUT_DIR="netdiscover_results"
INTERFACE="eth0"
NETWORK="192.168.1.0/24"

mkdir -p "$OUTPUT_DIR"
timestamp=$(date +%Y%m%d_%H%M%S)

# Active scan
echo "Running active scan..."
sudo netdiscover -i "$INTERFACE" \
    -r "$NETWORK" \
    -N > "$OUTPUT_DIR/active_scan_${timestamp}.txt"

# Passive scan
echo "Running passive scan..."
sudo timeout 300 netdiscover -p -i "$INTERFACE" \
    -N > "$OUTPUT_DIR/passive_scan_${timestamp}.txt"
```

### Continuous Monitoring
```bash
#!/bin/bash
# Continuous network monitoring
INTERFACE="eth0"
LOG_DIR="netdiscover_logs"
ROTATE_SIZE=100M

mkdir -p "$LOG_DIR"

rotate_logs() {
    if [ $(du -s "$LOG_DIR" | cut -f1) -gt $(numfmt --from=iec $ROTATE_SIZE) ]; then
        oldest_file=$(ls -t "$LOG_DIR" | tail -n 1)
        rm "$LOG_DIR/$oldest_file"
    fi
}

while true; do
    timestamp=$(date +%Y%m%d_%H%M%S)
    log_file="$LOG_DIR/netdiscover_${timestamp}.log"
    
    sudo netdiscover -p -i "$INTERFACE" -N > "$log_file" &
    pid=$!
    
    sleep 3600  # Run for 1 hour
    kill $pid
    
    rotate_logs
done
```

### Network Change Detection
```bash
#!/bin/bash
# Detect network changes
NETWORK="192.168.1.0/24"
BASELINE="baseline.txt"
CURRENT="current.txt"
DIFF="changes.txt"

# Create baseline if it doesn't exist
if [ ! -f "$BASELINE" ]; then
    sudo netdiscover -r "$NETWORK" -N > "$BASELINE"
    echo "Baseline created"
    exit 0
fi

# Current scan
sudo netdiscover -r "$NETWORK" -N > "$CURRENT"

# Compare results
diff "$BASELINE" "$CURRENT" > "$DIFF"

if [ -s "$DIFF" ]; then
    echo "Network changes detected:"
    cat "$DIFF"
    
    # Update baseline
    mv "$CURRENT" "$BASELINE"
else
    echo "No changes detected"
    rm "$CURRENT"
fi
```

---

Remember:
- Always run with sudo
- Consider network impact
- Use appropriate scan speeds
- Document findings
- Regular monitoring
- Handle results securely

For detailed information, consult the netdiscover documentation (`man netdiscover`).