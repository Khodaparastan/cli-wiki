

## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Scan Types](#scan-types)
- [Advanced Options](#advanced-options)
- [Output Formats](#output-formats)
- [Security Analysis](#security-analysis)
- [Best Practices](#best-practices)

## Overview

SSLyze is a fast and powerful SSL/TLS scanning tool that can analyze the SSL/TLS configuration of a server by testing multiple aspects of its SSL/TLS deployment.

### Key Features
- Certificate validation
- Cipher suite analysis
- Protocol support testing
- Vulnerability scanning
- Performance testing
- Multiple output formats
- Concurrent scanning
- Detailed reporting

## Installation

### Using pip
```bash
# Install sslyze
pip install --upgrade sslyze

# Install with all features
pip install --upgrade 'sslyze[json]'
```

### Using Docker
```bash
# Pull Docker image
docker pull nablac0d3/sslyze

# Run with Docker
docker run --rm nablac0d3/sslyze --regular example.com:443
```

## Basic Usage

### Simple Scans
```bash
# Basic scan
sslyze example.com

# Scan specific port
sslyze example.com:443

# Multiple targets
sslyze example1.com example2.com

# Scan with JSON output
sslyze --json_out results.json example.com
```

### Common Options
```bash
# Regular scan
sslyze --regular example.com

# Fast scan
sslyze --fast example.com

# Show certificate info
sslyze --certinfo example.com

# Show supported ciphers
sslyze --sslv2 --sslv3 --tlsv1 --tlsv1_1 --tlsv1_2 --tlsv1_3 example.com
```

## Scan Types

### Protocol Support
```bash
# Test all protocols
sslyze --robot --heartbleed --fallback --compression example.com

# Test specific protocols
sslyze --tlsv1_2 --tlsv1_3 example.com

# Test early TLS
sslyze --sslv2 --sslv3 --tlsv1 example.com
```

### Cipher Suites
```bash
# Test cipher suites
sslyze --regular example.com

# Test specific cipher
sslyze --tlsv1_2 --openssl_cipher_string="AES256-SHA" example.com

# Show rejected ciphers
sslyze --rejected_ciphers example.com
```

### Certificate Analysis
```bash
# Full certificate info
sslyze --certinfo_full example.com

# Basic certificate info
sslyze --certinfo example.com

# Trust store validation
sslyze --certinfo --mozilla_trust_store example.com
```

## Advanced Options

### Connection Settings
```bash
# Custom timeout
sslyze --timeout 5 example.com

# Specify client certificate
sslyze --cert client.pem --key client.key example.com

# HTTP CONNECT proxy
sslyze --https_proxy proxy.example.com:8080 example.com
```

### Performance Options
```bash
# Concurrent scanning
sslyze --concurrent 10 example.com

# Network timeout
sslyze --network_timeout 5 example.com

# Start TLS
sslyze --starttls=smtp smtp.example.com:25
```

### Custom Tests
```bash
# Custom cipher string
sslyze --openssl_cipher_string="HIGH:!aNULL" example.com

# Custom trust store
sslyze --custom_trust_store /path/to/store example.com

# Client authentication
sslyze --client_auth_credentials cert.pem key.pem example.com
```

## Output Formats

### JSON Output
```bash
# Basic JSON output
sslyze --json_out results.json example.com

# Pretty JSON output
sslyze --json_out results.json --json_pretty example.com

# Multiple targets
sslyze --json_out results.json example1.com example2.com
```

### XML Output
```bash
# XML output
sslyze --xml_out results.xml example.com

# Detailed XML
sslyze --xml_out results.xml --regular example.com
```

### Console Output
```bash
# Quiet mode
sslyze --quiet example.com

# Verbose mode
sslyze --verbose example.com

# Very verbose mode
sslyze --very_verbose example.com
```

## Security Analysis

### Vulnerability Scanning
```bash
# Check for known vulnerabilities
sslyze --robot --heartbleed --fallback example.com

# Full security scan
sslyze --regular --certinfo_full --http_headers example.com

# Check specific vulnerabilities
sslyze --robot --compression --heartbleed example.com
```

### Compliance Testing
```bash
# PCI compliance
sslyze --regular --tlsv1_2 --tlsv1_3 example.com

# HIPAA compliance
sslyze --regular --certinfo_full --http_headers example.com

# NIST compliance
sslyze --regular --tlsv1_2 --tlsv1_3 --certinfo_full example.com
```

## Best Practices

### Scanning Guidelines
```bash
# Comprehensive scan
sslyze --regular \
       --certinfo_full \
       --http_headers \
       --robot \
       --heartbleed \
       example.com

# Performance scan
sslyze --fast \
       --concurrent 10 \
       --network_timeout 5 \
       example.com
```

### Result Analysis
```bash
# Save detailed results
sslyze --regular \
       --json_out detailed_scan.json \
       --json_pretty \
       example.com

# Generate report
sslyze --regular \
       --xml_out report.xml \
       --quiet \
       example.com
```

## Quick Reference

### Essential Commands
```bash
# Basic scan
sslyze example.com

# Regular scan
sslyze --regular example.com

# Full scan
sslyze --regular --certinfo_full example.com

# Multiple targets
sslyze example1.com example2.com
```

### Common Options
```bash
--regular           # Regular scan
--certinfo         # Certificate info
--json_out         # JSON output
--xml_out          # XML output
--heartbleed       # Heartbleed test
--robot            # ROBOT test
--compression      # Compression test
```

## Example Scripts

### Basic Security Audit
```bash
#!/bin/bash
# Basic security audit script
TARGET="example.com"
OUTPUT_DIR="sslyze_results"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$OUTPUT_DIR"

sslyze --regular \
       --certinfo_full \
       --heartbleed \
       --robot \
       --json_out "$OUTPUT_DIR/scan_$DATE.json" \
       --json_pretty \
       "$TARGET"
```

### Multiple Domain Scan
```bash
#!/bin/bash
# Scan multiple domains
DOMAINS=(
    "example1.com"
    "example2.com"
    "example3.com"
)
OUTPUT_DIR="sslyze_scans"

mkdir -p "$OUTPUT_DIR"

for domain in "${DOMAINS[@]}"; do
    echo "Scanning $domain..."
    sslyze --regular \
           --json_out "$OUTPUT_DIR/${domain}_scan.json" \
           "$domain"
done
```

### Compliance Check
```bash
#!/bin/bash
# Compliance checking script
TARGET="example.com"
OUTPUT_DIR="compliance_results"

mkdir -p "$OUTPUT_DIR"

# Run comprehensive compliance scan
sslyze --regular \
       --certinfo_full \
       --http_headers \
       --robot \
       --heartbleed \
       --compression \
       --json_out "$OUTPUT_DIR/compliance_scan.json" \
       --json_pretty \
       "$TARGET"
```

---

Remember:
- Regular scanning schedule
- Document findings
- Update tool regularly
- Monitor scan duration
- Handle results securely
- Follow security policies

For detailed information, consult the SSLyze documentation (`sslyze --help`).