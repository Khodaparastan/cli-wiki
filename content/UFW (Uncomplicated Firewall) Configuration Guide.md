
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Rule Management](#rule-management)
- [Application Profiles](#application-profiles)
- [Advanced Configuration](#advanced-configuration)
- [Logging](#logging)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

## Overview

UFW (Uncomplicated Firewall) is a user-friendly interface for managing iptables. It provides a simplified way to configure a Linux firewall while retaining the reliability and security of iptables.

### Key Features
- Simple command syntax
- Application integration
- IPv4 and IPv6 support
- Logging capabilities
- Rate limiting
- Port management

## Installation

### Ubuntu (22.04/24.04)
```bash
# Install UFW
sudo apt update
sudo apt install ufw

# Check status
sudo ufw status
```

### Initial Setup
```bash
# Reset all rules
sudo ufw reset

# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

## Basic Usage

### Status Management
```bash
# Check firewall status
sudo ufw status
sudo ufw status verbose
sudo ufw status numbered

# Enable/Disable firewall
sudo ufw enable
sudo ufw disable

# Reload firewall
sudo ufw reload
```

### Basic Rules
```bash
# Allow incoming port
sudo ufw allow 22

# Deny incoming port
sudo ufw deny 80

# Allow specific service
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
```

## Rule Management

### Port Rules
```bash
# Allow specific port
sudo ufw allow 3306

# Allow port range
sudo ufw allow 6000:6007/tcp

# Allow from specific IP
sudo ufw allow from 192.168.1.100

# Allow to specific port
sudo ufw allow from 192.168.1.100 to any port 22
```

### Protocol Rules
```bash
# Allow TCP traffic
sudo ufw allow 80/tcp

# Allow UDP traffic
sudo ufw allow 53/udp

# Allow both TCP and UDP
sudo ufw allow 80,443/tcp
```

### Network Rules
```bash
# Allow subnet
sudo ufw allow from 192.168.1.0/24

# Allow specific IP range
sudo ufw allow from 192.168.1.1-192.168.1.100

# Allow to specific network interface
sudo ufw allow in on eth0 to any port 80
```

## Application Profiles

### Managing Profiles
```bash
# List available applications
sudo ufw app list

# Show application info
sudo ufw app info PROFILE

# Allow application
sudo ufw allow "Apache Full"
sudo ufw allow "OpenSSH"
```

### Common Profiles
```bash
# Web server
sudo ufw allow "Apache"
sudo ufw allow "Nginx Full"

# SSH server
sudo ufw allow "OpenSSH"

# Mail server
sudo ufw allow "Postfix"
```

## Advanced Configuration

### Rate Limiting
```bash
# Limit SSH connections
sudo ufw limit ssh

# Limit custom port
sudo ufw limit 3306/tcp

# Limit with specific IP
sudo ufw limit from 192.168.1.100
```

### Delete Rules
```bash
# Delete by number
sudo ufw status numbered
sudo ufw delete 2

# Delete specific rule
sudo ufw delete allow 80
sudo ufw delete deny 443

# Delete by full rule
sudo ufw delete allow from 192.168.1.100
```

### Custom Rules
```bash
# Allow multiple ports
sudo ufw allow 80,443/tcp

# Allow range with specific IP
sudo ufw allow from 192.168.1.100 to any port 60000:61000

# Allow specific in/out
sudo ufw allow in on eth0 from 192.168.1.100
sudo ufw allow out on eth0 to 192.168.1.100
```

## Logging

### Enable/Disable Logging
```bash
# Enable logging
sudo ufw logging on

# Set logging level
sudo ufw logging low
sudo ufw logging medium
sudo ufw logging high

# Disable logging
sudo ufw logging off
```

### View Logs
```bash
# View UFW logs
sudo tail -f /var/log/ufw.log

# Filter logs
sudo grep "UFW BLOCK" /var/log/ufw.log

# Monitor real-time
sudo journalctl -f -n 100 | grep "UFW"
```

## Troubleshooting

### Common Issues

1. **Connection Problems**
```bash
# Check rule status
sudo ufw status verbose

# Test specific rule
sudo ufw status | grep 80

# Check logs
sudo tail -f /var/log/ufw.log
```

2. **Rule Conflicts**
```bash
# List all rules
sudo ufw show added

# Check rule order
sudo ufw status numbered

# Reset rules if needed
sudo ufw reset
```

3. **Service Access**
```bash
# Verify service rules
sudo ufw app info "Apache Full"

# Test connection
nc -zv localhost 80
```

## Best Practices

### Security Configuration
```bash
# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Essential services
sudo ufw allow ssh
sudo ufw limit ssh
```

### Maintenance
```bash
# Regular status check
sudo ufw status verbose

# Backup rules
sudo cp /etc/ufw/user.rules /etc/ufw/user.rules.backup

# Review logs
sudo tail -f /var/log/ufw.log
```

## Quick Reference

### Essential Commands
```bash
# Status management
sudo ufw status
sudo ufw enable
sudo ufw disable

# Basic rules
sudo ufw allow ssh
sudo ufw deny http
sudo ufw limit https

# Rule management
sudo ufw delete RULE
sudo ufw reset
```

### Common Rules
```bash
# Web server
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# SSH server
sudo ufw allow 22/tcp
sudo ufw limit ssh

# Database
sudo ufw allow 3306/tcp
```

### Application Rules
```bash
# List applications
sudo ufw app list

# Allow applications
sudo ufw allow "Apache Full"
sudo ufw allow "OpenSSH"
sudo ufw allow "Nginx Full"
```

## Example Configurations

### Basic Web Server
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow "Apache Full"
sudo ufw enable
```

### Secure Server
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw limit ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw logging on
```

### Database Server
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 192.168.1.0/24 to any port 3306
sudo ufw limit ssh
```

---

Remember:
- Always backup before making changes
- Test rules before implementing
- Use limiting for sensitive services
- Keep logs for monitoring
- Regular review of rules
- Document all changes

For detailed information, consult the man pages (`man ufw`) and official documentation.