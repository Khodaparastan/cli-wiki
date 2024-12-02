
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Monitoring Options](#monitoring-options)
- [Display Controls](#display-controls)
- [Output Formats](#output-formats)
- [Advanced Usage](#advanced-usage)
- [Best Practices](#best-practices)

## Overview

Nethogs is a small 'net top' tool that groups bandwidth by process instead of by IP or interface. It's particularly useful for finding which process is using bandwidth.

### Key Features
- Per-process bandwidth monitoring
- Real-time updates
- Multiple interface support
- Traffic direction separation
- Refresh rate control
- Interactive controls
- Minimal system impact
- Process identification

## Installation

### Ubuntu (22.04/24.04)
```bash
# Install nethogs
sudo apt update
sudo apt install nethogs
```

### macOS
```bash
# Using Homebrew
brew install nethogs
```

## Basic Usage

### Simple Monitoring
```bash
# Monitor all interfaces
sudo nethogs

# Monitor specific interface
sudo nethogs eth0

# Monitor multiple interfaces
sudo nethogs eth0 wlan0

# Set refresh rate
sudo nethogs -d 1
```

### Common Options
```bash
# Show sent/received in KB/s
sudo nethogs -v 0

# Show sent/received in total KB
sudo nethogs -v 1

# Show sent/received in total B
sudo nethogs -v 2
```

## Monitoring Options

### Interface Selection
```bash
# Single interface
sudo nethogs eth0

# Multiple interfaces
sudo nethogs eth0 wlan0 tun0

# All interfaces
sudo nethogs all

# Exclude interface
sudo nethogs -i eth0
```

### Traffic Type
```bash
# TCP only
sudo nethogs -t

# TCP and UDP
sudo nethogs -p

# All protocols
sudo nethogs
```

### Update Intervals
```bash
# Fast updates (0.1 seconds)
sudo nethogs -d 0.1

# Slow updates (5 seconds)
sudo nethogs -d 5

# Default (1 second)
sudo nethogs -d 1
```

## Display Controls

### Interactive Commands
```text
m       # Change mode (KB/s, KB, B)
r       # Sort by received
s       # Sort by sent
q       # Quit
```

### Display Modes
```bash
# KB/s mode
sudo nethogs -v 0

# Total KB mode
sudo nethogs -v 1

# Total B mode
sudo nethogs -v 2
```

### Process Information
```bash
# Show process names
sudo nethogs

# Show process IDs
sudo nethogs -p

# Show full command
sudo nethogs -C
```

## Output Formats

### Tracing Mode
```bash
# Enable tracing
sudo nethogs -t

# With timestamp
sudo nethogs -t | while read line; do
    echo "$(date): $line"
done
```

### Logging
```bash
# Basic logging
sudo nethogs | tee nethogs.log

# Structured logging
sudo nethogs | while read line; do
    echo "$(date +%Y-%m-%d_%H:%M:%S): $line" >> nethogs.log
done
```

## Advanced Usage

### Filtering
```bash
# Monitor specific device class
sudo nethogs -d usb0

# Monitor VPN traffic
sudo nethogs tun0

# Monitor Docker traffic
sudo nethogs docker0
```

### Custom Refresh
```bash
# Real-time monitoring
sudo nethogs -d 0.1

# Low resource usage
sudo nethogs -d 5

# Balance mode
sudo nethogs -d 1
```

### Process Tracking
```bash
# Track specific process
sudo nethogs | grep "firefox"

# Track multiple processes
sudo nethogs | grep -E "firefox|chrome"
```

## Best Practices

### Monitoring Guidelines
```bash
# Basic monitoring
sudo nethogs -v 0 -d 1

# Resource-efficient monitoring
sudo nethogs -v 1 -d 5

# Detailed tracking
sudo nethogs -v 2 -d 0.5
```

### Resource Usage
```bash
# Minimize impact
sudo nethogs -d 2

# Balance update rate
sudo nethogs -d 1

# Maximum detail
sudo nethogs -d 0.1
```

## Quick Reference

### Essential Commands
```bash
# Basic monitoring
sudo nethogs

# Specific interface
sudo nethogs eth0

# Multiple interfaces
sudo nethogs eth0 wlan0

# Set refresh rate
sudo nethogs -d 1
```

### Common Options
```bash
-d    # Delay (seconds)
-v    # View mode
-t    # Tracing mode
-p    # Show PIDs
-C    # Show command line
```

## Example Scripts

### Basic Monitoring Script
```bash
#!/bin/bash
# Basic network monitoring
INTERFACE="eth0"
LOG_FILE="nethogs_log.txt"

sudo nethogs "$INTERFACE" | while read line; do
    echo "$(date): $line" >> "$LOG_FILE"
done
```

### Process Bandwidth Monitor
```bash
#!/bin/bash
# Monitor specific processes
PROCESSES=("firefox" "chrome" "wget")
LOG_DIR="nethogs_logs"

mkdir -p "$LOG_DIR"

monitor_process() {
    local process=$1
    local log_file="$LOG_DIR/${process}_bandwidth.log"
    
    sudo nethogs | grep "$process" | while read line; do
        echo "$(date): $line" >> "$log_file"
    done
}

for process in "${PROCESSES[@]}"; do
    monitor_process "$process" &
done

wait
```

### Network Usage Alert
```bash
#!/bin/bash
# Alert on high bandwidth usage
THRESHOLD=1000 # KB/s
INTERFACE="eth0"

monitor_bandwidth() {
    sudo nethogs "$INTERFACE" -v 0 | while read line; do
        if [[ $line =~ [0-9]+ ]]; then
            usage=${BASH_REMATCH[0]}
            if (( usage > THRESHOLD )); then
                echo "High bandwidth usage detected: ${usage}KB/s"
                notify-send "High Bandwidth Usage" "Current usage: ${usage}KB/s"
            fi
        fi
    done
}

monitor_bandwidth
```

### Continuous Monitoring
```bash
#!/bin/bash
# Continuous monitoring with rotation
LOG_DIR="nethogs_logs"
MAX_LOGS=10
ROTATE_SIZE=100M

mkdir -p "$LOG_DIR"

rotate_logs() {
    if [ $(ls -1 "$LOG_DIR" | wc -l) -gt $MAX_LOGS ]; then
        rm "$LOG_DIR/$(ls -t "$LOG_DIR" | tail -1)"
    fi
}

while true; do
    timestamp=$(date +%Y%m%d_%H%M%S)
    log_file="$LOG_DIR/nethogs_$timestamp.log"
    
    sudo nethogs -v 0 | while read line; do
        echo "$(date): $line" >> "$log_file"
        
        if [ $(stat -f%z "$log_file") -gt $(numfmt --from=iec $ROTATE_SIZE) ]; then
            break
        fi
    done
    
    rotate_logs
done
```

---

Remember:
- Run with sudo privileges
- Monitor system impact
- Use appropriate refresh rates
- Regular log rotation
- Document unusual patterns
- Consider privacy implications

For detailed information, consult the man pages (`man nethogs`).