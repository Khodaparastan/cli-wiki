
## Table of Contents
- [Overview](#overview)
- [Basic Concepts](#basic-concepts)
- [Tables and Chains](#tables-and-chains)
- [Basic Commands](#basic-commands)
- [Rule Management](#rule-management)
- [NAT Configuration](#nat-configuration)
- [Advanced Options](#advanced-options)
- [Persistence](#persistence)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## Overview

`iptables` is a powerful firewall tool for Linux systems that allows configuration of the Linux kernel firewall. It provides complete control over network traffic rules.

### Key Features
- Packet filtering
- Network address translation (NAT)
- Port forwarding
- Stateful packet inspection
- Custom chains
- Traffic control

## Basic Concepts

### Tables
```bash
# Available tables
- filter (default)
- nat
- mangle
- raw
- security
```

### Built-in Chains
```bash
# Filter table chains
- INPUT
- FORWARD
- OUTPUT

# NAT table chains
- PREROUTING
- POSTROUTING
- OUTPUT
```

## Tables and Chains

### List Rules
```bash
# List all rules
sudo iptables -L

# List with line numbers
sudo iptables -L --line-numbers

# List specific chain
sudo iptables -L INPUT

# Verbose output
sudo iptables -L -v
```

### Chain Policies
```bash
# Set default policy
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# View policies
sudo iptables -L | grep policy
```

## Basic Commands

### Rule Management
```bash
# Add rule at end
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Insert rule at position
sudo iptables -I INPUT 1 -p tcp --dport 80 -j ACCEPT

# Delete rule
sudo iptables -D INPUT 1

# Flush all rules
sudo iptables -F
```

### Basic Traffic Control
```bash
# Allow established connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow loopback
sudo iptables -A INPUT -i lo -j ACCEPT

# Drop invalid packets
sudo iptables -A INPUT -m state --state INVALID -j DROP
```

## Rule Management

### Protocol Rules
```bash
# Allow TCP traffic
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Allow UDP traffic
sudo iptables -A INPUT -p udp --dport 53 -j ACCEPT

# Allow ICMP (ping)
sudo iptables -A INPUT -p icmp -j ACCEPT
```

### Source/Destination Rules
```bash
# Allow from specific IP
sudo iptables -A INPUT -s 192.168.1.100 -j ACCEPT

# Allow to specific IP
sudo iptables -A INPUT -d 192.168.1.100 -j ACCEPT

# Allow from network
sudo iptables -A INPUT -s 192.168.1.0/24 -j ACCEPT
```

### Interface Rules
```bash
# Allow incoming on interface
sudo iptables -A INPUT -i eth0 -j ACCEPT

# Allow outgoing on interface
sudo iptables -A OUTPUT -o eth0 -j ACCEPT
```

## NAT Configuration

### Source NAT (SNAT)
```bash
# Enable NAT
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# SNAT with specific IP
sudo iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source 1.2.3.4
```

### Destination NAT (DNAT)
```bash
# Port forwarding
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100:8080

# Redirect ports
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
```

## Advanced Options

### State Matching
```bash
# Match connection states
sudo iptables -A INPUT -m state --state NEW,ESTABLISHED -j ACCEPT

# Match specific states
sudo iptables -A INPUT -m state --state INVALID -j DROP
sudo iptables -A INPUT -m state --state RELATED -j ACCEPT
```

### Rate Limiting
```bash
# Limit connections
sudo iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set

# Set connection limit
sudo iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m limit --limit 3/min -j ACCEPT
```

### Custom Chains
```bash
# Create custom chain
sudo iptables -N CUSTOM_CHAIN

# Add rules to custom chain
sudo iptables -A CUSTOM_CHAIN -p tcp --dport 80 -j ACCEPT

# Link to main chain
sudo iptables -A INPUT -j CUSTOM_CHAIN
```

## Persistence

### Save Rules
```bash
# Debian/Ubuntu
sudo netfilter-persistent save

# RHEL/CentOS
sudo service iptables save

# Manual save
sudo iptables-save > /etc/iptables/rules.v4
```

### Restore Rules
```bash
# Debian/Ubuntu
sudo netfilter-persistent reload

# Manual restore
sudo iptables-restore < /etc/iptables/rules.v4
```

## Best Practices

### Basic Security Setup
```bash
# Clear existing rules
sudo iptables -F

# Set default policies
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Allow loopback
sudo iptables -A INPUT -i lo -j ACCEPT

# Allow established connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

### Logging
```bash
# Enable logging
sudo iptables -A INPUT -j LOG --log-prefix "IPTables-Dropped: "

# Log and drop
sudo iptables -A INPUT -j LOG --log-prefix "IPTables-Dropped: " --log-level 4
sudo iptables -A INPUT -j DROP
```

## Troubleshooting

### Common Issues

1. **Connection Problems**
```bash
# Check rules
sudo iptables -L -v -n

# Check specific port
sudo iptables -L INPUT -v -n | grep dpt:80

# Monitor log
sudo tail -f /var/log/syslog | grep "IPTables"
```

2. **Rule Order Issues**
```bash
# List with numbers
sudo iptables -L --line-numbers

# Move rules
sudo iptables -I INPUT 1 [rule]
```

### Debug Commands
```bash
# Trace packets
sudo iptables -t raw -A PREROUTING -p tcp --dport 80 -j TRACE

# View packet trace
sudo tail -f /var/log/messages | grep TRACE
```

## Quick Reference

### Essential Commands
```bash
# List rules
sudo iptables -L
sudo iptables -L -v
sudo iptables -L --line-numbers

# Add rules
sudo iptables -A INPUT [rule]
sudo iptables -I INPUT [position] [rule]

# Delete rules
sudo iptables -D INPUT [rule-number]
sudo iptables -F

# Save/Restore
sudo netfilter-persistent save
sudo netfilter-persistent reload
```

### Common Rules
```bash
# Allow SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow DNS
sudo iptables -A INPUT -p udp --dport 53 -j ACCEPT
```

## Example Configurations

### Basic Web Server
```bash
# Clear rules
sudo iptables -F

# Set default policies
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Allow loopback
sudo iptables -A INPUT -i lo -j ACCEPT

# Allow established
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

---

Remember:
- Always backup rules before changes
- Test rules carefully
- Consider rule order
- Document all changes
- Use persistent storage
- Monitor logs regularly

For detailed information, consult the man pages (`man iptables`) and official documentation.