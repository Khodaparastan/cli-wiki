
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Server Configuration](#server-configuration)
- [Client Configuration](#client-configuration)
- [Certificate Management](#certificate-management)
- [Network Configuration](#network-configuration)
- [Security Settings](#security-settings)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

## Overview

OpenVPN is an open-source VPN solution that uses SSL/TLS for secure communications. It provides flexible VPN solutions for remote access and site-to-site connections.

### Key Features
- SSL/TLS security
- Client/Server mode
- Site-to-site capabilities
- Multiple authentication methods
- Cross-platform support
- NAT traversal
- Dynamic IP support

## Installation

### Ubuntu (22.04/24.04)
```bash
# Install OpenVPN
sudo apt update
sudo apt install openvpn easy-rsa

# Install management tools
sudo apt install openvpn-auth-ldap
```

### macOS
```bash
# Using Homebrew
brew install openvpn
```

## Server Configuration

### Basic Server Setup
```bash
# Copy sample config
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /etc/openvpn/server/

# Basic server configuration
server.conf:
```
```conf
port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh2048.pem
server 10.8.0.0 255.255.255.0
push "redirect-gateway def1"
push "dhcp-option DNS 8.8.8.8"
keepalive 10 120
comp-lzo
user nobody
group nogroup
persist-key
persist-tun
status openvpn-status.log
verb 3
```

### Server Network Settings
```bash
# Enable IP forwarding
echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Configure NAT
sudo iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
```

## Client Configuration

### Basic Client Setup
```bash
# Sample client configuration
client.conf:
```
```conf
client
dev tun
proto udp
remote server.example.com 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
cert client.crt
key client.key
comp-lzo
verb 3
```

### Client Connection Scripts
```bash
# Up script
#!/bin/bash
echo "VPN connection established"
date >> /var/log/vpn-connections.log

# Down script
#!/bin/bash
echo "VPN connection terminated"
date >> /var/log/vpn-disconnections.log
```

## Certificate Management

### Initialize PKI
```bash
# Set up Easy-RSA
cd /etc/openvpn/easy-rsa
./easyrsa init-pki
./easyrsa build-ca

# Generate server certificate
./easyrsa build-server-full server nopass
./easyrsa gen-dh
```

### Client Certificates
```bash
# Generate client certificate
./easyrsa build-client-full client1 nopass

# Export client certificates
cd /etc/openvpn/easy-rsa/pki
cp ca.crt client1.crt client1.key /etc/openvpn/client/
```

### Certificate Revocation
```bash
# Revoke certificate
./easyrsa revoke client1

# Generate CRL
./easyrsa gen-crl

# Update server config
crl-verify /etc/openvpn/crl.pem
```

## Network Configuration

### Routing
```bash
# Push routes to client
push "route 192.168.1.0 255.255.255.0"

# Client specific routes
client-config-dir ccd
route 10.0.0.0 255.0.0.0

# Split tunneling
push "route 10.0.0.0 255.0.0.0"
```

### DNS Configuration
```bash
# Push DNS settings
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"
push "dhcp-option DOMAIN example.com"
```

## Security Settings

### Encryption Settings
```conf
# Strong encryption configuration
cipher AES-256-GCM
auth SHA512
tls-version-min 1.2
tls-cipher TLS-ECDHE-RSA-WITH-AES-256-GCM-SHA384
```

### Authentication
```conf
# Enable user/pass authentication
auth-user-pass-verify /etc/openvpn/auth.py via-file
script-security 2
username-as-common-name

# 2FA configuration
auth-user-pass
auth-token-user
static-challenge "Enter Google Authenticator Code" 1
```

## Troubleshooting

### Common Issues

1. **Connection Problems**
```bash
# Check server status
sudo systemctl status openvpn@server

# View logs
tail -f /var/log/openvpn.log

# Test connectivity
ping 10.8.0.1
```

2. **Certificate Issues**
```bash
# Verify certificates
openssl verify -CAfile ca.crt client1.crt

# Check certificate dates
openssl x509 -in client1.crt -noout -dates
```

3. **Routing Problems**
```bash
# Check routing table
ip route show

# Verify iptables rules
sudo iptables -L -n -v

# Test forwarding
sysctl net.ipv4.ip_forward
```

## Best Practices

### Security Recommendations
```conf
# Hardening configuration
tls-auth ta.key 0
remote-cert-tls client
verify-client-cert require
```

### Performance Optimization
```conf
# Performance settings
sndbuf 393216
rcvbuf 393216
fast-io
compress lz4-v2
```

## Quick Reference

### Essential Commands
```bash
# Start/Stop Server
sudo systemctl start openvpn@server
sudo systemctl stop openvpn@server

# Check Status
sudo systemctl status openvpn@server

# View Logs
tail -f /var/log/openvpn.log
```

### Common Options
```conf
port      # UDP/TCP port
proto     # Protocol (udp/tcp)
dev       # Interface type
ca        # CA certificate
cert      # Server certificate
key       # Private key
dh        # Diffie-Hellman parameters
server    # VPN subnet
```

## Example Configurations

### Basic Server Setup
```conf
port 1194
proto udp
dev tun
server 10.8.0.0 255.255.255.0
ca ca.crt
cert server.crt
key server.key
dh dh2048.pem
push "redirect-gateway def1"
keepalive 10 120
comp-lzo
user nobody
group nogroup
persist-key
persist-tun
status openvpn-status.log
verb 3
```

### Advanced Server Setup
```conf
port 443
proto tcp
dev tun
server 10.8.0.0 255.255.255.0
ca ca.crt
cert server.crt
key server.key
dh dh2048.pem
auth SHA512
cipher AES-256-GCM
tls-version-min 1.2
tls-auth ta.key 0
push "redirect-gateway def1"
push "dhcp-option DNS 8.8.8.8"
keepalive 10 120
comp-lzo
user nobody
group nogroup
persist-key
persist-tun
status openvpn-status.log
verb 3
```

---

Remember:
- Regularly update certificates
- Monitor logs
- Keep backups of configurations
- Test changes in staging
- Document all modifications
- Regular security audits

For detailed information, consult the OpenVPN manual and official documentation.