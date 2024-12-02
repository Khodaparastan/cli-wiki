

## Table of Contents
- [Overview](#overview)
- [Basic Usage](#basic-usage)
- [Network Configuration](#network-configuration)
- [DNS Configuration](#dns-configuration)
- [Proxy Settings](#proxy-settings)
- [VPN Management](#vpn-management)
- [System State](#system-state)
- [Troubleshooting](#troubleshooting)

## Overview

`scutil` is a powerful command-line tool for managing system configuration in macOS. It provides access to the dynamic store containing network configuration and system state information.

### Key Features
- DNS configuration management
- Network service control
- Proxy settings management
- VPN configuration
- System state queries
- Computer/hostname management

## Basic Usage

### Interactive Mode
```bash
# Enter interactive mode
scutil

# Common interactive commands
> list
> show *
> quit
```

### Computer Name Management
```bash
# Get computer name
scutil --get ComputerName

# Set computer name
sudo scutil --set ComputerName "NewName"

# Get host name
scutil --get HostName

# Set host name
sudo scutil --set HostName "hostname.local"
```

### Local Host Names
```bash
# Get local host name
scutil --get LocalHostName

# Set local host name
sudo scutil --set LocalHostName "localhostname"
```

## Network Configuration

### Check Network Status
```bash
# Get primary service
scutil --nwi | grep "Primary interface"

# Show all network information
scutil --nwi

# Get specific network interface details
scutil --nwi en0
```

### Network Services
```bash
# List all network services
scutil --nc list

# Show specific network service details
scutil --nc status "VPN Service Name"

# Start network service
scutil --nc start "VPN Service Name"

# Stop network service
scutil --nc stop "VPN Service Name"
```

## DNS Configuration

### View DNS Settings
```bash
# Show DNS configuration
scutil --dns

# Get DNS servers
scutil --dns | grep "nameserver"

# Check DNS search domains
scutil --dns | grep "search domain"
```

### DNS Cache Management
```bash
# Flush DNS cache
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder
```

## Proxy Settings

### View Proxy Configuration
```bash
# Enter interactive mode
scutil
> show State:/Network/Global/Proxies
```

### Check Specific Proxy Settings
```bash
# Web proxy (HTTP)
networksetup -getwebproxy "Wi-Fi"

# Secure web proxy (HTTPS)
networksetup -getsecurewebproxy "Wi-Fi"

# SOCKS proxy
networksetup -getsocksfirewallproxy "Wi-Fi"
```

## VPN Management

### VPN Service Control
```bash
# List VPN services
scutil --nc list

# Show VPN status
scutil --nc status "VPN Name"

# Connect VPN
scutil --nc start "VPN Name"

# Disconnect VPN
scutil --nc stop "VPN Name"
```

### VPN Configuration Details
```bash
# Show VPN configuration
scutil --nc show "VPN Name"

# Watch VPN status
scutil --nc watch "VPN Name"
```

## System State

### System Preferences
```bash
# Get system preferences
scutil --get SystemVersion

# Check preference values
defaults read /Library/Preferences/SystemConfiguration/preferences.plist
```

### Dynamic Store Access
```bash
# Enter interactive mode for dynamic store
scutil
> list
> show Setup:/System
> show State:/Network/Global/IPv4
```

## Troubleshooting

### Common Issues and Solutions

1. **Network Interface Problems**
```bash
# Check interface status
scutil --nwi

# Verify DNS configuration
scutil --dns
```

2. **VPN Connection Issues**
```bash
# Check VPN status
scutil --nc status "VPN Name"

# Show detailed VPN information
scutil --nc show "VPN Name"
```

3. **Hostname Resolution**
```bash
# Verify hostname configuration
scutil --get HostName
scutil --get LocalHostName
scutil --get ComputerName
```

### Debug Commands
```bash
# Show all network information
scutil --nwi --verbose

# Display all dynamic store entries
scutil
> list
> show *
```

## Advanced Usage

### Dynamic Store Manipulation
```bash
# Enter interactive mode
scutil
> d.init
> d.show
> get State:/Network/Global/IPv4
```

### Network Service Management
```bash
# List all network services with details
networksetup -listallnetworkservices

# Get specific service information
networksetup -getinfo "Wi-Fi"
```

## Quick Reference

### Essential Commands
```bash
# Computer Name Management
scutil --get ComputerName
sudo scutil --set ComputerName "name"

# Network Information
scutil --nwi

# DNS Configuration
scutil --dns

# VPN Control
scutil --nc list
scutil --nc status "VPN Name"
```

### Common Flags
- `--get`: Retrieve system settings
- `--set`: Modify system settings
- `--nc`: Network connection operations
- `--dns`: DNS configuration
- `--nwi`: Network information
- `--proxy`: Proxy settings

### Interactive Mode Commands
```bash
# Enter interactive mode
scutil
> list
> show Setup:/
> show State:/
> show Pattern:/
> quit
```

## Best Practices

1. **System Changes**
   - Always use sudo for system modifications
   - Backup configurations before changes
   - Verify changes after implementation

2. **Network Management**
   - Regular DNS configuration checks
   - Monitor VPN connection status
   - Document network service configurations

3. **Troubleshooting Steps**
   - Check network interface status
   - Verify DNS configuration
   - Monitor system logs
   - Test connectivity after changes

---

Remember:
- Most system-modifying commands require sudo privileges
- Always backup configurations before making changes
- Test changes in a controlled environment first
- Document all configuration changes

This guide covers the most common and useful scutil commands. For detailed information, consult the man pages (`man scutil`).