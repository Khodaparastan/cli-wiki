
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Output Formats](#output-formats)
- [Advanced Options](#advanced-options)
- [Analysis Techniques](#analysis-techniques)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

## Overview

MTR combines the functionality of `traceroute` and `ping` into a single network diagnostic tool, providing continuous monitoring of network paths.

### Key Features
- Real-time monitoring
- Packet loss statistics
- Latency measurement
- Path visualization
- Multiple output formats
- IPv4/IPv6 support
- DNS resolution control
- Custom packet sizes

## Installation

### Ubuntu (22.04/24.04)
```bash
# Install mtr
sudo apt update
sudo apt install mtr-tiny

# Full version with GTK interface
sudo apt install mtr
```

### macOS
```bash
# Using Homebrew
brew install mtr
```

## Basic Usage

### Simple Tests
```bash
# Basic test
sudo mtr example.com

# Report mode
mtr -r example.com

# Specify count
mtr -c 50 example.com

# No DNS resolution
mtr -n example.com
```

### Common Options
```bash
# Report with 100 packets
mtr -r -c 100 example.com

# Wide report format
mtr -w example.com

# Show IP addresses
mtr -b example.com

# Split addresses
mtr -z example.com
```

## Output Formats

### Report Formats
```bash
# Standard report
mtr --report example.com

# CSV output
mtr --csv example.com

# JSON output
mtr --json example.com

# XML output
mtr --xml example.com
```

### Custom Fields
```bash
# Select specific fields
mtr --report \
    --no-dns \
    --show-ips \
    example.com

# Report with timestamps
mtr --report \
    --show-timestamps \
    example.com
```

## Advanced Options

### Packet Configuration
```bash
# Custom packet size
mtr -s 1000 example.com

# TCP instead of ICMP
mtr --tcp example.com

# UDP packets
mtr --udp example.com

# Custom port
mtr --port 443 example.com
```

### Timing Options
```bash
# Custom interval
mtr -i 0.5 example.com

# Timeout value
mtr -t 1.0 example.com

# Max number of hops
mtr -m 30 example.com
```

### Protocol Selection
```bash
# Force IPv4
mtr -4 example.com

# Force IPv6
mtr -6 example.com

# ICMP packets (default)
mtr --icmp example.com
```

## Analysis Techniques

### Network Troubleshooting
```bash
# Basic troubleshooting
mtr -r -c 100 -n example.com

# Loss pattern analysis
mtr --report \
    --show-timestamps \
    --csv \
    example.com

# Latency investigation
mtr -i 0.1 \
    --no-dns \
    --report \
    example.com
```

### Performance Analysis
```bash
# Detailed statistics
mtr --report-wide \
    --show-ips \
    --csv \
    example.com

# Multiple target comparison
for host in host1.com host2.com; do
    mtr -n -r -c 50 $host > "$host.txt"
done
```

## Troubleshooting

### Common Issues

1. **High Packet Loss**
```bash
# Detailed loss analysis
mtr -r -c 200 \
    --show-timestamps \
    --csv \
    problem_host
```

2. **Latency Issues**
```bash
# Latency monitoring
mtr -i 0.1 \
    --no-dns \
    --report \
    slow_host
```

3. **Path Problems**
```bash
# Path analysis
mtr --report-wide \
    --show-ips \
    --no-dns \
    unreachable_host
```

## Best Practices

### Testing Guidelines
```bash
# Basic network test
mtr -r -c 100 \
    --no-dns \
    --report \
    target.com

# Long-term monitoring
mtr -r -c 1000 \
    --interval 1 \
    --csv \
    target.com
```

### Documentation
```bash
# Save test results
mtr -r -c 100 target.com > report.txt

# Multiple format export
mtr --json target.com > report.json
mtr --csv target.com > report.csv
```

## Quick Reference

### Essential Commands
```bash
# Basic test
sudo mtr example.com

# Report mode
mtr -r example.com

# No DNS lookup
mtr -n example.com

# Wide report
mtr -w example.com
```

### Common Options
```bash
-r    # Report mode
-n    # No DNS resolution
-c    # Packet count
-s    # Packet size
-i    # Interval
-t    # Timeout
-m    # Max hops
```

## Example Scripts

### Basic Network Analysis
```bash
#!/bin/bash
# Network path analysis
TARGET="example.com"
OUTPUT_DIR="mtr_reports"

mkdir -p "$OUTPUT_DIR"

# Run different test types
mtr -r -c 100 "$TARGET" > "$OUTPUT_DIR/standard.txt"
mtr --json "$TARGET" > "$OUTPUT_DIR/json.json"
mtr --csv "$TARGET" > "$OUTPUT_DIR/csv.csv"
```

### Multiple Host Testing
```bash
#!/bin/bash
# Test multiple hosts
HOSTS=(
    "host1.com"
    "host2.com"
    "host3.com"
)

for host in "${HOSTS[@]}"; do
    echo "Testing $host..."
    mtr -r -c 50 "$host" > "${host}_report.txt"
done
```

### Continuous Monitoring
```bash
#!/bin/bash
# Continuous network monitoring
TARGET="example.com"
INTERVAL=300 # 5 minutes

while true; do
    timestamp=$(date +%Y%m%d_%H%M%S)
    mtr -r -c 60 "$TARGET" > "mtr_${timestamp}.txt"
    sleep $INTERVAL
done
```

### Performance Comparison
```bash
#!/bin/bash
# Compare network paths
SOURCE_HOST="example.com"
TARGETS=(
    "8.8.8.8"
    "1.1.1.1"
)

for target in "${TARGETS[@]}"; do
    echo "Testing route to $target..."
    mtr -r -c 100 \
        --no-dns \
        --csv \
        "$target" > "${target}_analysis.csv"
done
```

---

Remember:
- Use sudo when needed
- Consider DNS resolution impact
- Choose appropriate packet counts
- Monitor for extended periods
- Document findings
- Compare multiple tests
- Use appropriate protocols

For detailed information, consult the man pages (`man mtr`).