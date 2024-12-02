
## Table of Contents
- [Overview](#overview)
- [Basic Usage](#basic-usage)
- [Route Management](#route-management)
- [Gateway Configuration](#gateway-configuration)
- [Policy Routing](#policy-routing)
- [Advanced Options](#advanced-options)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

## Overview

`ip route` is the modern command for managing IP routing tables in Linux systems. Part of the `iproute2` package, it provides comprehensive routing control and network configuration capabilities.

### Key Features
- Route table management
- Policy-based routing
- Multiple routing tables
- Advanced routing metrics
- VRF support
- MPLS routing
- Traffic control integration

## Basic Usage

### Display Routes
```bash
# Show all routes
ip route show

# Show route details
ip route show detail

# Show statistics
ip -s route show

# Watch route changes
watch -n1 'ip route show'
```

### Basic Format
```bash
# General syntax
ip route {add|del|change|append|replace} prefix [via gateway] [dev interface]
```

## Route Management

### Add Routes
```bash
# Add network route
sudo ip route add 192.168.2.0/24 via 192.168.1.1

# Add route via interface
sudo ip route add 192.168.2.0/24 dev eth0

# Add default route
sudo ip route add default via 192.168.1.1
```

### Delete Routes
```bash
# Delete specific route
sudo ip route del 192.168.2.0/24

# Delete route via gateway
sudo ip route del 192.168.2.0/24 via 192.168.1.1

# Delete default route
sudo ip route del default
```

### Modify Routes
```bash
# Change existing route
sudo ip route change 192.168.2.0/24 via 192.168.1.2

# Replace route
sudo ip route replace 192.168.2.0/24 via 192.168.1.2

# Append route
sudo ip route append 192.168.2.0/24 via 192.168.1.2
```

## Gateway Configuration

### Default Gateway
```bash
# Add default gateway
sudo ip route add default via 192.168.1.1

# Add with metric
sudo ip route add default via 192.168.1.1 metric 100

# Add with interface
sudo ip route add default via 192.168.1.1 dev eth0
```

### Multiple Gateways
```bash
# Primary gateway
sudo ip route add default via 192.168.1.1 metric 100

# Backup gateway
sudo ip route add default via 192.168.1.2 metric 200
```

## Policy Routing

### Routing Tables
```bash
# Show all tables
ip route show table all

# Add route to specific table
sudo ip route add 192.168.2.0/24 via 192.168.1.1 table 100

# Delete route from table
sudo ip route del 192.168.2.0/24 table 100
```

### Route Rules
```bash
# Show rules
ip rule show

# Add rule
sudo ip rule add from 192.168.1.0/24 table 100

# Delete rule
sudo ip rule del from 192.168.1.0/24 table 100
```

## Advanced Options

### Route Metrics
```bash
# Add route with metric
sudo ip route add 192.168.2.0/24 via 192.168.1.1 metric 100

# Add with multiple metrics
sudo ip route add 192.168.2.0/24 via 192.168.1.1 metric 100 mtu 1500
```

### Route Types
```bash
# Prohibit route
sudo ip route add prohibit 192.168.2.0/24

# Blackhole route
sudo ip route add blackhole 192.168.2.0/24

# Unreachable route
sudo ip route add unreachable 192.168.2.0/24
```

### Route Caching
```bash
# Show route cache
ip route show cache

# Flush route cache
sudo ip route flush cache
```

## Troubleshooting

### Common Issues

1. **Route Conflicts**
```bash
# Check existing routes
ip route show

# Check specific network
ip route show match 192.168.2.0/24

# Check route details
ip route show 192.168.2.0/24 detail
```

2. **Gateway Issues**
```bash
# Verify gateway reachability
ping gateway_ip

# Check gateway routes
ip route get gateway_ip

# Monitor gateway
ip monitor route
```

3. **Interface Problems**
```bash
# Check interface status
ip link show dev eth0

# Verify interface routes
ip route show dev eth0

# Monitor interface
ip monitor route dev eth0
```

## Best Practices

### Route Management
```bash
# Backup current routes
ip route show > routes_backup.txt

# Verify changes
ip route get 192.168.2.1

# Monitor changes
ip monitor route
```

### Security Considerations
```bash
# Block networks
sudo ip route add blackhole 10.0.0.0/8

# Monitor route changes
watch -n1 'ip route show'

# Log changes
ip monitor route | logger
```

## Quick Reference

### Essential Commands
```bash
# Show routes
ip route show

# Add route
sudo ip route add network/mask via gateway_ip

# Delete route
sudo ip route del network/mask

# Get specific route
ip route get destination_ip
```

### Common Options
```bash
show    # Display routes
add     # Add route
del     # Delete route
change  # Modify route
replace # Replace route
get     # Get specific route
```

## Example Configurations

### Basic Network Setup
```bash
# Default gateway
sudo ip route add default via 192.168.1.1

# Local network
sudo ip route add 192.168.1.0/24 dev eth0

# Remote network
sudo ip route add 10.0.0.0/8 via 192.168.1.254
```

### Advanced Routing
```bash
# Multiple paths
sudo ip route add 192.168.2.0/24 \
  nexthop via 192.168.1.1 weight 1 \
  nexthop via 192.168.1.2 weight 2

# Policy routing
sudo ip route add 192.168.2.0/24 via 192.168.1.1 table 100
sudo ip rule add from 192.168.1.100 table 100
```

### VPN Configuration
```bash
# Add VPN route
sudo ip route add 10.0.0.0/8 dev tun0

# Split tunneling
sudo ip route add default via 10.8.0.1 table 200
sudo ip rule add from 192.168.1.0/24 table 200
```

---

Remember:
- Always backup before changes
- Test changes thoroughly
- Document configurations
- Monitor route changes
- Consider security implications
- Use appropriate metrics

For detailed information, consult the man pages (`man ip-route`).