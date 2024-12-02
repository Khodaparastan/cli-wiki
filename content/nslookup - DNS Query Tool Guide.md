
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Query Types](#query-types)
- [Interactive Mode](#interactive-mode)
- [Advanced Options](#advanced-options)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

## Overview

`nslookup` is a network administration tool for querying DNS servers to obtain domain name or IP address mapping information. While considered legacy in some contexts, it remains widely used due to its simplicity and availability across platforms.

### Key Features
- DNS record querying
- Reverse DNS lookups
- Interactive query mode
- Server selection
- Multiple record type support
- Cross-platform compatibility

## Installation

### Ubuntu (22.04/24.04)
```bash
sudo apt update
sudo apt install dnsutils
```

### macOS
```bash
# Pre-installed on macOS
# No installation needed
```

## Basic Usage

### Simple Queries
```bash
# Basic domain lookup
nslookup example.com

# IP address lookup (reverse DNS)
nslookup 8.8.8.8

# Using specific DNS server
nslookup example.com 8.8.8.8
```

### Command Format
```bash
# General syntax
nslookup [-option] [name | -] [server]

# Example with specific server
nslookup example.com 1.1.1.1
```

## Query Types

### Common Record Types
```bash
# A record (IPv4)
nslookup -type=A example.com

# AAAA record (IPv6)
nslookup -type=AAAA example.com

# MX record (Mail)
nslookup -type=MX example.com

# NS record (Nameserver)
nslookup -type=NS example.com

# SOA record (Start of Authority)
nslookup -type=SOA example.com

# TXT record
nslookup -type=TXT example.com

# CNAME record
nslookup -type=CNAME www.example.com
```

### Multiple Record Query
```bash
# Query all records
nslookup -type=ANY example.com

# Query specific multiple records
nslookup -query=MX example.com
nslookup -query=NS example.com
```

## Interactive Mode

### Basic Interactive Commands
```bash
# Enter interactive mode
nslookup
> server 8.8.8.8
> set type=A
> example.com
> exit
```

### Interactive Mode Options
```bash
# Common commands in interactive mode
> set all          # Show current settings
> set type=ANY     # Set query type
> set debug        # Enable debug mode
> set nodebug      # Disable debug mode
> help             # Show help
```

### Server Selection
```bash
# Change DNS server
> server 1.1.1.1
> server 8.8.8.8

# View current server
> set all
```

## Advanced Options

### Debug Options
```bash
# Enable debugging
nslookup -debug example.com

# Verbose output
nslookup -debug -verbose example.com
```

### Query Control
```bash
# Set timeout
nslookup -timeout=5 example.com

# Disable recursion
nslookup -norecurse example.com

# Use TCP instead of UDP
nslookup -vc example.com
```

### Output Control
```bash
# Show detailed information
nslookup -debug example.com

# Display only answer
nslookup -type=A -noquestion example.com
```

## Troubleshooting

### Common Issues

1. **DNS Resolution Problems**
```bash
# Check with different DNS server
nslookup example.com 8.8.8.8
nslookup example.com 1.1.1.1

# Verify authoritative servers
nslookup -type=NS example.com
```

2. **Reverse DNS Issues**
```bash
# Check reverse DNS
nslookup IP_ADDRESS

# Verify PTR record
nslookup -type=PTR IP_ADDRESS
```

3. **Connection Problems**
```bash
# Test with TCP
nslookup -vc example.com

# Increase timeout
nslookup -timeout=10 example.com
```

### Debug Mode
```bash
# Enable debugging
nslookup -debug example.com

# Show query path
nslookup -trace example.com
```

## Best Practices

### DNS Server Testing
```bash
# Test multiple DNS servers
nslookup example.com 8.8.8.8
nslookup example.com 1.1.1.1
nslookup example.com 9.9.9.9
```

### Record Verification
```bash
# Verify critical records
nslookup -type=MX domain.com
nslookup -type=NS domain.com
nslookup -type=SOA domain.com
```

### Security Checks
```bash
# Check SPF records
nslookup -type=TXT domain.com

# Verify DMARC
nslookup -type=TXT _dmarc.domain.com
```

## Quick Reference

### Essential Commands
```bash
# Basic lookup
nslookup example.com

# Specific record type
nslookup -type=A example.com

# Reverse lookup
nslookup IP_ADDRESS

# Using specific server
nslookup example.com 8.8.8.8
```

### Common Options
```bash
# Query types
-type=A          # IPv4 address
-type=AAAA       # IPv6 address
-type=MX         # Mail server
-type=NS         # Name server
-type=PTR        # Pointer record
-type=SOA        # Start of authority
-type=TXT        # Text record

# Other options
-debug           # Enable debugging
-timeout=N       # Set timeout
-query=TYPE      # Specify query type
-norecurse       # Disable recursion
```

### Interactive Mode Commands
```bash
# Enter interactive mode
nslookup
> server DNS_SERVER
> set type=RECORD_TYPE
> domain.com
> exit
```

## Examples for Common Tasks

### Mail Server Verification
```bash
# Check mail records
nslookup -type=MX domain.com

# Verify SPF
nslookup -type=TXT domain.com
```

### Domain Configuration
```bash
# Check all NS records
nslookup -type=NS domain.com

# Verify SOA record
nslookup -type=SOA domain.com
```

### Security Checks
```bash
# DMARC verification
nslookup -type=TXT _dmarc.domain.com

# DKIM verification
nslookup -type=TXT selector._domainkey.domain.com
```

---

Remember:
- Always verify critical DNS records
- Use multiple DNS servers for validation
- Document query results
- Consider using modern alternatives (dig) for advanced tasks
- Keep timeout values reasonable

For more detailed information, consult the man pages (`man nslookup`).