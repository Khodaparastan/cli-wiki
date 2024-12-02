
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Tunnel Management](#tunnel-management)
- [Configuration](#configuration)
- [Access Management](#access-management)
- [Monitoring](#monitoring)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

## Overview

Cloudflared is the command-line client for Cloudflare Tunnel, enabling secure connections between your resources and the Cloudflare network without exposing public IPs.

### Key Features
- Secure tunnel creation
- Zero Trust access
- Local development
- Service proxying
- DNS management
- Access policies
- Metrics and logging

## Installation

### Ubuntu (22.04/24.04)
```bash
# Using package manager
curl -L https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-archive-keyring.gpg >/dev/null
echo "deb [signed-by=/usr/share/keyrings/cloudflare-archive-keyring.gpg] https://pkg.cloudflare.com/cloudflared $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflared.list
sudo apt update
sudo apt install cloudflared
```

### macOS
```bash
# Using Homebrew
brew install cloudflare/cloudflare/cloudflared
```

## Basic Usage

### Authentication
```bash
# Login to Cloudflare
cloudflared tunnel login

# Verify authentication
cloudflared tunnel token
```

### Quick Start
```bash
# Create tunnel
cloudflared tunnel create my-tunnel

# Start tunnel
cloudflared tunnel run my-tunnel

# List tunnels
cloudflared tunnel list
```

## Tunnel Management

### Create and Configure
```bash
# Create new tunnel
cloudflared tunnel create tunnel-name

# Configure tunnel
cloudflared tunnel route dns tunnel-name subdomain.example.com

# Delete tunnel
cloudflared tunnel delete tunnel-name
```

### Running Tunnels
```bash
# Run with config file
cloudflared tunnel --config path/to/config.yml run

# Run with specific hostname
cloudflared tunnel --hostname example.com run

# Run with specific credentials
cloudflared tunnel --credentials-file /path/to/creds.json run
```

### Tunnel Routes
```bash
# Add DNS route
cloudflared tunnel route dns tunnel-name hostname

# List routes
cloudflared tunnel route list

# Delete route
cloudflared tunnel route dns --overwrite-dns tunnel-name hostname
```

## Configuration

### Config File
```yaml
# config.yml
tunnel: tunnel-id
credentials-file: /path/to/credentials.json

ingress:
  - hostname: example.com
    service: http://localhost:8000
  - hostname: api.example.com
    service: http://localhost:3000
  - service: http_status:404
```

### Service Configuration
```yaml
# HTTP service
ingress:
  - hostname: app.example.com
    service: http://localhost:8000
    originRequest:
      connectTimeout: 30s
      noTLSVerify: false

# TCP service
ingress:
  - hostname: ssh.example.com
    service: tcp://localhost:22
```

### Access Control
```yaml
# Access policies
ingress:
  - hostname: internal.example.com
    service: http://localhost:8000
    originRequest:
      access:
        required: true
        teamName: "example-team"
```

## Access Management

### Authentication Methods
```yaml
# Basic authentication
ingress:
  - hostname: app.example.com
    service: http://localhost:8000
    originRequest:
      auth:
        type: basic
        credentials:
          - user: password

# OAuth
ingress:
  - hostname: app.example.com
    service: http://localhost:8000
    originRequest:
      auth:
        type: oauth
```

### Access Policies
```yaml
# Team access
ingress:
  - hostname: app.example.com
    service: http://localhost:8000
    originRequest:
      access:
        required: true
        teamName: ["team1", "team2"]
```

## Monitoring

### Metrics
```bash
# Enable metrics
cloudflared tunnel --metrics localhost:2000 run

# View tunnel status
cloudflared tunnel info

# Check connection status
cloudflared tunnel status
```

### Logging
```bash
# Enable debug logging
cloudflared tunnel --loglevel debug run

# Log to file
cloudflared tunnel --logfile /path/to/tunnel.log run

# JSON logging
cloudflared tunnel --json run
```

## Troubleshooting

### Common Issues

1. **Connection Problems**
```bash
# Check tunnel status
cloudflared tunnel status

# Verify credentials
cloudflared tunnel token

# Test connectivity
cloudflared tunnel diagnose
```

2. **Configuration Issues**
```bash
# Validate config
cloudflared tunnel ingress validate

# Check DNS records
cloudflared tunnel route list

# Test specific hostname
cloudflared tunnel diagnose --hostname example.com
```

3. **Performance Issues**
```bash
# Enable tracing
cloudflared tunnel --trace run

# Monitor metrics
cloudflared tunnel --metrics localhost:2000 run
```

## Best Practices

### Security
```yaml
# Secure configuration
ingress:
  - hostname: app.example.com
    service: http://localhost:8000
    originRequest:
      noTLSVerify: false
      connectTimeout: 30s
      disableChunkedEncoding: false
```

### High Availability
```yaml
# Replica configuration
replica: 2
retries: 5
grace_period: 30s
```

## Quick Reference

### Essential Commands
```bash
# Create tunnel
cloudflared tunnel create name

# Run tunnel
cloudflared tunnel run name

# List tunnels
cloudflared tunnel list

# Delete tunnel
cloudflared tunnel delete name
```

### Common Options
```bash
--config       # Config file path
--credentials  # Credentials file
--hostname     # Tunnel hostname
--url         # Origin URL
--metrics     # Metrics address
--loglevel    # Log level
```

## Example Configurations

### Web Application
```yaml
tunnel: tunnel-id
credentials-file: /path/to/creds.json

ingress:
  - hostname: app.example.com
    service: http://localhost:3000
    originRequest:
      connectTimeout: 30s
      noTLSVerify: false
  - service: http_status:404
```

### Multiple Services
```yaml
tunnel: tunnel-id
credentials-file: /path/to/creds.json

ingress:
  - hostname: app.example.com
    service: http://localhost:3000
  - hostname: api.example.com
    service: http://localhost:8080
  - hostname: ssh.example.com
    service: tcp://localhost:22
  - service: http_status:404
```

---

Remember:
- Regular backup of credentials
- Monitor tunnel status
- Keep configuration secure
- Use access controls
- Regular updates
- Monitor logs

For detailed information, consult the official Cloudflare documentation.