
## Table of Contents
- [Overview](#overview)
- [Basic Usage](#basic-usage)
- [Route Management](#route-management)
- [Gateway Configuration](#gateway-configuration)
- [Network Interface Routes](#network-interface-routes)
- [Advanced Options](#advanced-options)
- [Troubleshooting](#troubleshooting)
- [Modern Alternatives](#modern-alternatives)
- [Best Practices](#best-practices)

## Overview

`route` is a traditional command for viewing and manipulating the IP routing table. While considered legacy (replaced by `ip route` in modern systems), it's still widely used and available.

### Key Features
- Display routing table
- Add/delete routes
- Configure default gateway
- Interface routing
- Network management
- Static route configuration

## Basic Usage

### Display Routes
```bash
# Show routing table
route -n

# Show verbose output
route -v

# Show statistics
route -s
```

### Basic Format
```bash
# General syntax
route [add|del] [-net|-host] target [netmask Nm] [gw Gw] [dev If]
```

## Route Management

### Add Routes
```bash
# Add network route
sudo route add -net 192.168.2.0 netmask 255.255.255.0 gw 192.168.1.1

# Add host route
sudo route add -host 192.168.2.10 gw 192.168.1.1

# Add default gateway
sudo route add default gw 192.168.1.1
```

### Delete Routes
```bash
# Delete network route
sudo route del -net 192.168.2.0 netmask 255.255.255.0

# Delete host route
sudo route del -host 192.168.2.10

# Delete default gateway
sudo route del default
```

## Gateway Configuration

### Default Gateway
```bash
# Add default gateway
sudo route add default gw 192.168.1.1

# Add default gateway with metric
sudo route add default gw 192.168.1.1 metric 1

# Add gateway for specific network
sudo route add -net 192.168.2.0/24 gw 192.168.1.1
```

### Multiple Gateways
```bash
# Add primary gateway
sudo route add default gw 192.168.1.1 metric 1

# Add secondary gateway
sudo route add default gw 192.168.1.2 metric 2
```

## Network Interface Routes

### Interface Specific Routes
```bash
# Add route via interface
sudo route add -net 192.168.2.0/24 dev eth0

# Add host route via interface
sudo route add -host 192.168.2.10 dev eth0

# Add default route via interface
sudo route add default dev eth0
```

### Reject Routes
```bash
# Reject network
sudo route add -net 192.168.2.0/24 reject

# Reject host
sudo route add -host 192.168.2.10 reject
```

## Advanced Options

### Metrics and Flags
```bash
# Add route with metric
sudo route add -net 192.168.2.0/24 gw 192.168.1.1 metric 2

# Add route with specific flags
sudo route add -net 192.168.2.0/24 gw 192.168.1.1 mss 1400
```

### Network Masks
```bash
# Using CIDR notation
sudo route add -net 192.168.2.0/24 gw 192.168.1.1

# Using netmask
sudo route add -net 192.168.2.0 netmask 255.255.255.0 gw 192.168.1.1
```

## Troubleshooting

### Common Issues

1. **Gateway Unreachable**
```bash
# Check gateway status
ping gateway_ip

# Verify interface status
ifconfig interface_name

# Check route table
route -n
```

2. **Route Conflicts**
```bash
# Show all routes
route -n

# Check specific network
route -n | grep network_address

# Verify metrics
route -n | sort -k 5
```

3. **Interface Problems**
```bash
# Check interface status
ifconfig interface_name

# Verify interface routes
route -n | grep interface_name
```

## Modern Alternatives

### IP Route Command
```bash
# Show routes (modern equivalent)
ip route show

# Add route
ip route add 192.168.2.0/24 via 192.168.1.1

# Delete route
ip route del 192.168.2.0/24

# Add default gateway
ip route add default via 192.168.1.1
```

### Network Manager
```bash
# Show connections
nmcli connection show

# Modify route
nmcli connection modify "Connection Name" +ipv4.routes "192.168.2.0/24 192.168.1.1"

# Apply changes
nmcli connection up "Connection Name"
```

## Best Practices

### Route Management
```bash
# Document current routes
route -n > route_backup.txt

# Verify before changes
route -n | grep target_network

# Test after changes
ping destination_ip
```

### Security Considerations
```bash
# Reject unwanted networks
sudo route add -net 10.0.0.0/8 reject

# Monitor route changes
watch -n 1 'route -n'

# Log route changes
route -n | logger
```

## Quick Reference

### Essential Commands
```bash
# Show routes
route -n

# Add network route
sudo route add -net network/mask gw gateway_ip

# Add host route
sudo route add -host host_ip gw gateway_ip

# Delete route
sudo route del -net network/mask
```

### Common Options
```bash
-n    # Show numerical addresses
-v    # Verbose output
-net  # Network route
-host # Host route
gw    # Gateway
dev   # Interface
```

## Example Configurations

### Basic Network Setup
```bash
# Add default gateway
sudo route add default gw 192.168.1.1

# Add local network
sudo route add -net 192.168.1.0/24 dev eth0

# Add remote network
sudo route add -net 10.0.0.0/8 gw 192.168.1.254
```

### Multiple Network Setup
```bash
# Primary route
sudo route add default gw 192.168.1.1 metric 1

# Secondary route
sudo route add default gw 192.168.1.2 metric 2

# Specific network route
sudo route add -net 192.168.2.0/24 gw 192.168.1.1
```

### VPN Configuration
```bash
# Add VPN network
sudo route add -net 10.0.0.0/8 dev tun0

# Add VPN gateway
sudo route add default gw 10.0.0.1 metric 100

# Block direct access
sudo route add -net 192.168.2.0/24 reject
```

---

Remember:
- Always backup current routes
- Test changes carefully
- Document all modifications
- Consider using modern alternatives
- Monitor route changes
- Keep security in mind

For detailed information, consult the man pages (`man route`) and consider modern alternatives like `ip route`.