
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Configuration](#basic-configuration)
- [Service Management](#service-management)
- [Client Configuration](#client-configuration)
- [Hidden Services](#hidden-services)
- [Security Features](#security-features)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

## Overview

Tor (The Onion Router) is a network protocol designed to provide anonymous communication over the internet.

### Key Features
- Anonymous browsing
- Hidden services
- Encrypted traffic
- Bridge support
- SOCKS proxy
- Exit node control
- Traffic routing
- Network isolation

## Installation

### Ubuntu (22.04/24.04)
```bash
# Install Tor
sudo apt update
sudo apt install tor tor-geoipdb

# Install utilities
sudo apt install proxychains-ng
```

### Build from Source
```bash
# Install dependencies
sudo apt install build-essential libevent-dev libssl-dev zlib1g-dev

# Download and build
wget https://www.torproject.org/dist/tor-[version].tar.gz
tar xzf tor-[version].tar.gz
cd tor-[version]
./configure
make
sudo make install
```

## Basic Configuration

### torrc Configuration
```bash
# /etc/tor/torrc
# Basic configuration
SocksPort 9050
ControlPort 9051
DataDirectory /var/lib/tor

# Logging
Log notice file /var/log/tor/notices.log
Log debug file /var/log/tor/debug.log

# Circuit building
NumEntryGuards 3
NumDirectoryGuards 3
```

### Access Control
```bash
# Control authentication
HashedControlPassword [password_hash]
CookieAuthentication 1

# SOCKS policy
SocksPolicy accept 127.0.0.1
SocksPolicy reject *
```

## Service Management

### Service Control
```bash
# Start Tor service
sudo systemctl start tor

# Enable at boot
sudo systemctl enable tor

# Check status
sudo systemctl status tor

# Restart service
sudo systemctl restart tor
```

### Log Monitoring
```bash
# View logs
sudo tail -f /var/log/tor/notices.log

# Debug logs
sudo tail -f /var/log/tor/debug.log

# Journal logs
sudo journalctl -u tor
```

## Client Configuration

### SOCKS Proxy
```bash
# Configure SOCKS proxy
export ALL_PROXY=socks5://127.0.0.1:9050

# Test connection
curl --socks5 127.0.0.1:9050 https://check.torproject.org/

# DNS through Tor
DNSPort 53
AutomapHostsOnResolve 1
```

### Proxychains Configuration
```bash
# /etc/proxychains.conf
strict_chain
proxy_dns
[ProxyList]
socks5 127.0.0.1 9050
```

## Hidden Services

### Basic Hidden Service
```bash
# /etc/tor/torrc
HiddenServiceDir /var/lib/tor/hidden_service/
HiddenServicePort 80 127.0.0.1:80
```

### Multiple Services
```bash
# Web service
HiddenServiceDir /var/lib/tor/web/
HiddenServicePort 80 127.0.0.1:80

# SSH service
HiddenServiceDir /var/lib/tor/ssh/
HiddenServicePort 22 127.0.0.1:22
```

## Security Features

### Bridge Configuration
```bash
# Use bridges
UseBridges 1
Bridge [bridge-line]
Bridge [bridge-line]

# obfs4 support
ClientTransportPlugin obfs4 exec /usr/bin/obfs4proxy
```

### Exit Node Control
```bash
# Exclude exit nodes
ExitNodes {us} StrictNodes 1
ExcludeExitNodes {ru},{cn}

# Restrict ports
ExitPolicy reject *:25
ExitPolicy accept *:80
ExitPolicy accept *:443
```

## Troubleshooting

### Common Issues
```bash
# Check connectivity
netstat -antlp | grep 9050

# Test SOCKS proxy
curl --socks5-hostname 127.0.0.1:9050 https://check.torproject.org/

# Debug mode
tor --verify-config
```

### Circuit Testing
```bash
# New circuit
sudo killall -HUP tor

# Monitor circuits
watch -n 1 "sudo cat /var/lib/tor/state"
```

## Best Practices

### Security Guidelines
```bash
# Secure configuration
sudo chmod 700 /var/lib/tor/hidden_service/
sudo chown debian-tor:debian-tor /var/lib/tor/hidden_service/

# Isolate streams
IsolateDestAddr 1
IsolateDestPort 1
```

### Performance Tuning
```bash
# Circuit performance
CircuitBuildTimeout 60
LearnCircuitBuildTimeout 1
MaxCircuitDirtiness 600
```

## Example Scripts

### Tor Status Monitor
```bash
#!/bin/bash
# Monitor Tor status and circuits

LOG_FILE="/var/log/tor/notices.log"
ALERT_EMAIL="admin@example.com"

check_tor_status() {
    if ! systemctl is-active --quiet tor; then
        echo "Tor service is down!" | mail -s "Tor Alert" "$ALERT_EMAIL"
        sudo systemctl restart tor
    fi
}

monitor_circuits() {
    grep "Built circuit" "$LOG_FILE" | tail -n 5
}

while true; do
    check_tor_status
    monitor_circuits
    sleep 300
done
```

### Hidden Service Setup
```bash
#!/bin/bash
# Set up new hidden service

SERVICE_NAME=$1
SERVICE_PORT=$2
TOR_DIR="/var/lib/tor"

# Create service directory
sudo mkdir -p "$TOR_DIR/${SERVICE_NAME}"
sudo chown debian-tor:debian-tor "$TOR_DIR/${SERVICE_NAME}"
sudo chmod 700 "$TOR_DIR/${SERVICE_NAME}"

# Add configuration
cat << EOF | sudo tee -a /etc/tor/torrc
HiddenServiceDir $TOR_DIR/${SERVICE_NAME}/
HiddenServicePort $SERVICE_PORT 127.0.0.1:$SERVICE_PORT
EOF

# Restart Tor
sudo systemctl restart tor

# Show onion address
sudo cat "$TOR_DIR/${SERVICE_NAME}/hostname"
```

### Tor Circuit Tester
```bash
#!/bin/bash
# Test Tor circuits and performance

TEST_URL="https://check.torproject.org/"
TESTS=5

test_circuit() {
    start_time=$(date +%s.%N)
    
    curl --socks5 127.0.0.1:9050 \
         -s -o /dev/null \
         -w "%{http_code}" \
         "$TEST_URL"
    
    end_time=$(date +%s.%N)
    duration=$(echo "$end_time - $start_time" | bc)
    
    echo "Circuit test completed in $duration seconds"
}

for ((i=1; i<=TESTS; i++)); do
    echo "Test $i of $TESTS"
    test_circuit
    sudo killall -HUP tor
    sleep 5
done
```

### Tor Network Monitor
```bash
#!/bin/bash
# Monitor Tor network status

LOG_DIR="/var/log/tor"
STATS_FILE="tor_stats.log"

gather_stats() {
    echo "=== Tor Statistics $(date) ===" >> "$LOG_DIR/$STATS_FILE"
    
    # Check connections
    netstat -an | grep 9050 | wc -l >> "$LOG_DIR/$STATS_FILE"
    
    # Check circuits
    grep "Built circuit" "$LOG_DIR/notices.log" | tail -n 5 >> "$LOG_DIR/$STATS_FILE"
    
    # Check bandwidth
    tail -n 50 "$LOG_DIR/notices.log" | grep "Bandwidth" >> "$LOG_DIR/$STATS_FILE"
}

# Rotate logs
rotate_logs() {
    if [ -f "$LOG_DIR/$STATS_FILE" ] && [ $(stat -f%z "$LOG_DIR/$STATS_FILE") -gt 1048576 ]; then
        mv "$LOG_DIR/$STATS_FILE" "$LOG_DIR/$STATS_FILE.$(date +%Y%m%d)"
    fi
}

while true; do
    gather_stats
    rotate_logs
    sleep 3600
done
```

---

Remember:
- Regular security updates
- Monitor logs
- Verify configurations
- Use strong authentication
- Keep services isolated
- Follow security best practices

For detailed information, consult the Tor documentation and manual pages.