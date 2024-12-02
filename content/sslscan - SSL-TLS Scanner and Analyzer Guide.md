
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Protocol Testing](#protocol-testing)
- [Cipher Analysis](#cipher-analysis)
- [Certificate Checks](#certificate-checks)
- [Advanced Features](#advanced-features)
- [Best Practices](#best-practices)

## Overview

sslscan tests SSL/TLS enabled services to discover supported cipher suites, protocol versions, certificate information, and security vulnerabilities.

### Key Features
- Protocol version detection
- Cipher suite enumeration
- Certificate analysis
- Heartbleed testing
- Renegotiation testing
- TLS compression testing
- Detailed reporting
- Multiple output formats

## Installation

### Ubuntu (22.04/24.04)
```bash
# Install sslscan
sudo apt update
sudo apt install sslscan
```

### Build from Source
```bash
# Clone and build
git clone https://github.com/rbsec/sslscan.git
cd sslscan
make static
sudo make install
```

## Basic Usage

### Simple Scans
```bash
# Basic scan
sslscan example.com

# Scan specific port
sslscan example.com:443

# Show certificate
sslscan --show-certificate example.com

# No color output
sslscan --no-colour example.com
```

### Common Options
```bash
# Verbose output
sslscan --verbose example.com

# Show supported ciphers
sslscan --show-ciphers example.com

# Show client ciphers
sslscan --show-client-ciphers example.com
```

## Protocol Testing

### Version Testing
```bash
# Test all SSL/TLS versions
sslscan --ssl2 --ssl3 --tls10 --tls11 --tls12 --tls13 example.com

# Test specific version
sslscan --tls12 example.com

# No SSLv2/SSLv3
sslscan --no-ssl2 --no-ssl3 example.com
```

### Protocol Options
```bash
# Show TLS extensions
sslscan --tlsextensions example.com

# Check fallback support
sslscan --fallback example.com

# Check compression
sslscan --compression example.com
```

## Cipher Analysis

### Cipher Testing
```bash
# Show supported ciphers
sslscan --show-ciphers example.com

# Test specific cipher
sslscan --cipher=AES256-SHA example.com

# Show cipher bits
sslscan --show-cipher-bits example.com
```

### Cipher Categories
```bash
# Show weak ciphers
sslscan --show-weak example.com

# No weak ciphers
sslscan --no-weak example.com

# Show cipher preferences
sslscan --show-cipher-preferred example.com
```

## Certificate Checks

### Certificate Analysis
```bash
# Show certificate details
sslscan --show-certificate example.com

# Check certificate chain
sslscan --show-chain example.com

# Check certificate trust
sslscan --no-check-certificate example.com
```

### Certificate Information
```bash
# Show certificate subject
sslscan --show-subject example.com

# Show certificate issuer
sslscan --show-issuer example.com

# Show certificate dates
sslscan --show-dates example.com
```

## Advanced Features

### Output Options
```bash
# XML output
sslscan --xml=output.xml example.com

# JSON output
sslscan --json=output.json example.com

# No color output
sslscan --no-colour example.com
```

### Connection Options
```bash
# Specify source IP
sslscan --source-ip=192.168.1.100 example.com

# Use STARTTLS
sslscan --starttls example.com

# Specify SNI hostname
sslscan --sni-name=example.com example.com
```

## Best Practices

### Security Scanning
```bash
# Comprehensive security scan
sslscan \
    --show-certificate \
    --show-ciphers \
    --show-weak \
    --tlsextensions \
    --no-ssl2 \
    --no-ssl3 \
    example.com
```

### Compliance Testing
```bash
# PCI compliance check
sslscan \
    --no-ssl2 \
    --no-ssl3 \
    --no-weak \
    --show-certificate \
    example.com
```

## Quick Reference

### Essential Commands
```bash
# Basic scan
sslscan example.com

# Full scan
sslscan --show-certificate --show-ciphers example.com

# Security scan
sslscan --no-ssl2 --no-ssl3 --show-weak example.com

# Output to file
sslscan --xml=output.xml example.com
```

### Common Options
```bash
--show-certificate    # Show certificate
--show-ciphers       # Show supported ciphers
--show-weak          # Show weak ciphers
--no-colour          # Disable colored output
--xml                # XML output
--starttls          # Use STARTTLS
```

## Example Scripts

### Basic SSL Audit
```bash
#!/bin/bash
# Basic SSL/TLS audit script
OUTPUT_DIR="sslscan_results"
TARGET="example.com"

mkdir -p "$OUTPUT_DIR"
timestamp=$(date +%Y%m%d_%H%M%S)

# Run comprehensive scan
sslscan \
    --show-certificate \
    --show-ciphers \
    --show-weak \
    --xml="$OUTPUT_DIR/scan_${timestamp}.xml" \
    "$TARGET"
```

### Multiple Domain Scanner
```bash
#!/bin/bash
# Scan multiple domains
OUTPUT_DIR="ssl_scans"
DOMAINS_FILE="domains.txt"

mkdir -p "$OUTPUT_DIR"

while read domain; do
    echo "Scanning $domain..."
    timestamp=$(date +%Y%m%d_%H%M%S)
    
    sslscan \
        --no-colour \
        --show-certificate \
        --show-ciphers \
        --xml="$OUTPUT_DIR/${domain}_${timestamp}.xml" \
        "$domain"
        
    sleep 2
done < "$DOMAINS_FILE"
```

### Security Compliance Check
```bash
#!/bin/bash
# Security compliance checking script
TARGET="example.com"
OUTPUT_FILE="compliance_report.txt"

echo "SSL/TLS Compliance Report for $TARGET" > "$OUTPUT_FILE"
echo "Date: $(date)" >> "$OUTPUT_FILE"
echo "----------------------------------------" >> "$OUTPUT_FILE"

# Run compliance scan
sslscan \
    --no-ssl2 \
    --no-ssl3 \
    --no-weak \
    --show-certificate \
    --show-ciphers \
    "$TARGET" | tee -a "$OUTPUT_FILE"

# Check for vulnerabilities
if grep -q "SSLv2" "$OUTPUT_FILE"; then
    echo "WARNING: SSLv2 is enabled" >> "$OUTPUT_FILE"
fi

if grep -q "SSLv3" "$OUTPUT_FILE"; then
    echo "WARNING: SSLv3 is enabled" >> "$OUTPUT_FILE"
fi

if grep -q "Weak" "$OUTPUT_FILE"; then
    echo "WARNING: Weak ciphers detected" >> "$OUTPUT_FILE"
fi
```

### Certificate Monitor
```bash
#!/bin/bash
# Monitor SSL certificates for expiration
DOMAINS_FILE="domains.txt"
ALERT_DAYS=30
EMAIL="admin@example.com"

check_certificate() {
    domain=$1
    expiry_date=$(sslscan --no-colour --show-certificate "$domain" | 
                  grep "Not After" | 
                  awk '{print $4, $5, $7}')
    
    expiry_epoch=$(date -d "$expiry_date" +%s)
    current_epoch=$(date +%s)
    days_remaining=$(( ($expiry_epoch - $current_epoch) / 86400 ))
    
    if [ $days_remaining -lt $ALERT_DAYS ]; then
        echo "WARNING: Certificate for $domain expires in $days_remaining days"
        mail -s "Certificate Expiry Warning: $domain" "$EMAIL" <<EOF
The SSL certificate for $domain will expire in $days_remaining days.
Please renew the certificate soon.

Expiry date: $expiry_date
EOF
    fi
}

while read domain; do
    check_certificate "$domain"
done < "$DOMAINS_FILE"
```

---

Remember:
- Regular certificate monitoring
- Document scan results
- Follow security best practices
- Keep tool updated
- Handle sensitive data securely
- Consider rate limiting

For detailed information, consult the sslscan documentation (`man sslscan`).