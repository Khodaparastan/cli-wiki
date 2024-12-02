
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Configuration](#configuration)
- [Zero Trust Integration](#zero-trust-integration)
- [Split Tunneling](#split-tunneling)
- [Troubleshooting](#troubleshooting)
- [CLI Management](#cli-management)
- [Best Practices](#best-practices)

## Overview

Cloudflare WARP is a modern VPN client that provides secure, fast connections to the internet through Cloudflare's network, with optional Zero Trust capabilities.

### Key Features
- Secure DNS (1.1.1.1)
- Zero Trust integration
- Split tunneling
- Network performance optimization
- Malware blocking
- Device registration
- Gateway policies
- Cross-platform support

## Installation

### Ubuntu (22.04/24.04)
```bash
# Download and install
curl https://pkg.cloudflareclient.com/pubkey.gpg | sudo gpg --yes --dearmor --output /usr/share/keyrings/cloudflare-warp-archive-keyring.gpg

echo "deb [arch=amd64 signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflare-client.list

sudo apt update
sudo apt install cloudflare-warp
```

### macOS
```bash
# Using Homebrew
brew install --cask cloudflare-warp

# Direct download
curl -L https://1.1.1.1/Cloudflare_WARP.pkg -o Cloudflare_WARP.pkg
sudo installer -pkg Cloudflare_WARP.pkg -target /
```

## Basic Usage

### CLI Control
```bash
# Register WARP
warp-cli register

# Connect to WARP
warp-cli connect

# Disconnect from WARP
warp-cli disconnect

# Check status
warp-cli status
```

### Account Management
```bash
# Login to Zero Trust organization
warp-cli teams-enroll <team-token>

# Check account type
warp-cli account

# Switch modes
warp-cli set-mode warp
warp-cli set-mode proxy
warp-cli set-mode doh
```

## Configuration

### Basic Settings
```bash
# Enable/Disable WARP
warp-cli enable-warp
warp-cli disable-warp

# Configure DNS
warp-cli set-custom-endpoint <endpoint>
warp-cli set-families-mode

# Set preferences
warp-cli set-proxy-port 40000
warp-cli set-timezone auto
```

### Network Settings
```bash
# Configure proxy settings
warp-cli set-proxy-mode off
warp-cli set-proxy-mode automatic
warp-cli set-proxy-mode manual

# Configure endpoints
warp-cli add-trusted-ssid "WiFi-Name"
warp-cli remove-trusted-ssid "WiFi-Name"
```

## Zero Trust Integration

### Team Enrollment
```bash
# Enroll in Zero Trust
warp-cli teams-enroll <token>

# Check enrollment status
warp-cli teams-enrollment-status

# Leave team
warp-cli teams-unenroll
```

### Device Management
```bash
# Register device
warp-cli device-register

# Check device status
warp-cli device-status

# Reset device registration
warp-cli device-reset
```

## Split Tunneling

### Configure Split Tunnels
```bash
# Add exclude route
warp-cli add-exclude-route 192.168.1.0/24

# Remove exclude route
warp-cli remove-exclude-route 192.168.1.0/24

# List excluded routes
warp-cli list-exclude-routes
```

### Application Split Tunneling
```bash
# Add excluded application
warp-cli add-excluded-app "/path/to/application"

# Remove excluded application
warp-cli remove-excluded-app "/path/to/application"

# List excluded applications
warp-cli list-excluded-apps
```

## Troubleshooting

### Common Issues

1. **Connection Problems**
```bash
# Check WARP status
warp-cli status

# Diagnose connection
warp-cli diagnose

# Reset WARP
warp-cli reset
```

2. **Performance Issues**
```bash
# Check connection details
warp-cli warp-stats

# Monitor connection
warp-cli trace

# Test connectivity
warp-cli test
```

3. **Authentication Issues**
```bash
# Check registration
warp-cli registration-status

# Verify team enrollment
warp-cli teams-enrollment-status

# Reset registration
warp-cli delete
warp-cli register
```

## CLI Management

### System Commands
```bash
# Service management
sudo systemctl status warp-svc
sudo systemctl restart warp-svc

# Check logs
sudo journalctl -u warp-svc

# Clear settings
warp-cli clear-device-name
warp-cli delete
```

### Configuration Management
```bash
# Backup configuration
warp-cli generate-conf > warp-backup.conf

# Reset configuration
warp-cli reset

# Update client
warp-cli update
```

## Best Practices

### Security Settings
```bash
# Enable security features
warp-cli enable-dns-log
warp-cli set-families-mode
warp-cli set-gateway-port random

# Configure trusted networks
warp-cli add-trusted-ssid "Office-WiFi"
warp-cli set-switch-locked true
```

### Performance Optimization
```bash
# Optimize settings
warp-cli set-mode warp+doh
warp-cli set-custom-endpoint closest
warp-cli set-keepalive 25
```

## Quick Reference

### Essential Commands
```bash
# Basic control
warp-cli connect
warp-cli disconnect
warp-cli status

# Account management
warp-cli register
warp-cli teams-enroll
warp-cli account

# Configuration
warp-cli set-mode
warp-cli set-proxy-port
warp-cli set-families-mode
```

### Common Options
```bash
connect         # Connect to WARP
disconnect      # Disconnect from WARP
status          # Check connection status
register        # Register device
teams-enroll    # Enroll in Zero Trust
set-mode        # Change WARP mode
```

## Example Configurations

### Basic Setup
```bash
# Initial configuration
warp-cli register
warp-cli set-mode warp
warp-cli enable-dns-log
warp-cli connect
```

### Zero Trust Setup
```bash
# Zero Trust configuration
warp-cli teams-enroll <token>
warp-cli set-mode warp+doh
warp-cli add-exclude-route 192.168.0.0/16
warp-cli connect
```

### Split Tunnel Configuration
```bash
# Configure split tunneling
warp-cli add-exclude-route 10.0.0.0/8
warp-cli add-exclude-route 172.16.0.0/12
warp-cli add-exclude-route 192.168.0.0/16
warp-cli add-excluded-app "/usr/bin/ssh"
```

---

Remember:
- Regular updates
- Monitor connection status
- Backup configurations
- Document custom settings
- Review security policies
- Keep logs for troubleshooting

For detailed information, consult the official Cloudflare WARP documentation.