
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Query Types](#query-types)
- [Output Formats](#output-formats)
- [Advanced Features](#advanced-features)
- [DNS Security](#dns-security)
- [Best Practices](#best-practices)

## Overview

`doggo` is a modern DNS client for the command line, designed to be user-friendly and feature-rich, with colorized output and extensive query options.

### Key Features
- Modern CLI interface
- Colorized output
- Multiple output formats
- DNSSEC validation
- Multiple transport protocols
- JSON output support
- DNS-over-TLS/HTTPS
- Extensive query types

## Installation

### Ubuntu (22.04/24.04)
```bash
# Using snap
sudo snap install doggo

# From source
go install github.com/mr-karan/doggo@latest
```

### macOS
```bash
# Using Homebrew
brew install doggo
```

## Basic Usage

### Simple Queries
```bash
# Basic query
doggo example.com

# Specify record type
doggo A example.com

# Multiple record types
doggo A MX example.com

# Short output
doggo --short example.com
```

### DNS Servers
```bash
# Use specific nameserver
doggo @8.8.8.8 example.com

# Use multiple nameservers
doggo @8.8.8.8 @1.1.1.1 example.com

# Use custom port
doggo @8.8.8.8:53 example.com
```

## Query Types

### Common Record Types
```bash
# A records
doggo A example.com

# AAAA records
doggo AAAA example.com

# MX records
doggo MX example.com

# TXT records
doggo TXT example.com

# NS records
doggo NS example.com

# All records
doggo ANY example.com
```

### Special Queries
```bash
# Reverse DNS lookup
doggo -x 8.8.8.8

# DNSSEC records
doggo DS example.com
doggo DNSKEY example.com

# Service discovery
doggo SRV _http._tcp.example.com
```

## Output Formats

### Format Options
```bash
# JSON output
doggo --json example.com

# Minimal output
doggo --short example.com

# Table format
doggo --table example.com

# Raw DNS wire format
doggo --wire example.com
```

### Customizing Output
```bash
# Select specific fields
doggo --json-fields name,type,ttl example.com

# Pretty JSON
doggo --json --pretty example.com

# No colors
doggo --no-color example.com
```

## Advanced Features

### Transport Protocols
```bash
# DNS-over-TLS
doggo @tls://1.1.1.1 example.com

# DNS-over-HTTPS
doggo @https://cloudflare-dns.com/dns-query example.com

# TCP only
doggo --tcp example.com
```

### Query Options
```bash
# Set timeout
doggo --timeout 5s example.com

# Set number of retries
doggo --retry 3 example.com

# Enable trace
doggo --trace example.com
```

### DNSSEC Validation
```bash
# Enable DNSSEC validation
doggo --dnssec example.com

# Check DNSSEC chain
doggo --dnssec --trace example.com

# Show DNSSEC details
doggo --dnssec --debug example.com
```

## DNS Security

### Secure DNS Queries
```bash
# Use DNS-over-TLS
doggo @tls://dns.google example.com

# Use DNS-over-HTTPS
doggo @https://dns.google/dns-query example.com

# Verify DNSSEC
doggo --dnssec example.com
```

### Security Options
```bash
# Enable all security features
doggo --dnssec \
      --no-nsid \
      --no-edns \
      example.com

# Check for DNS poisoning
doggo --verify-all @8.8.8.8 @1.1.1.1 example.com
```

## Best Practices

### Query Guidelines
```bash
# Complete domain check
doggo A AAAA MX TXT example.com

# Security check
doggo --dnssec \
      --verify-all \
      @8.8.8.8 example.com

# Performance testing
doggo --timeout 2s \
      --retry 2 \
      --stats example.com
```

### Troubleshooting
```bash
# Debug mode
doggo --debug example.com

# Trace queries
doggo --trace example.com

# Show statistics
doggo --stats example.com
```

## Quick Reference

### Essential Commands
```bash
# Basic query
doggo example.com

# Specific record
doggo A example.com

# Multiple records
doggo A MX TXT example.com

# Reverse lookup
doggo -x 8.8.8.8
```

### Common Options
```bash
--json       # JSON output
--short      # Minimal output
--table      # Table format
--debug      # Debug mode
--trace      # Trace queries
--dnssec     # DNSSEC validation
--timeout    # Query timeout
--retry      # Number of retries
```

## Example Use Cases

### Domain Verification
```bash
#!/bin/bash
# Complete domain check
domain="example.com"

echo "Checking $domain..."
doggo --dnssec \
      --verify-all \
      A AAAA MX TXT \
      "$domain"
```

### Mail Server Verification
```bash
#!/bin/bash
# Check mail records
domain="example.com"

echo "Checking mail records for $domain..."
doggo MX "$domain"
doggo TXT "$domain" | grep -i "spf"
doggo TXT "_dmarc.$domain"
```

### Security Audit
```bash
#!/bin/bash
# Security checks
domain="example.com"

echo "Security audit for $domain..."
doggo --dnssec \
      --verify-all \
      --debug \
      DNSKEY DS NS \
      "$domain"
```
