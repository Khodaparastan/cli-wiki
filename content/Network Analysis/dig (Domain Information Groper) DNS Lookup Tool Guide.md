
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Syntax](#basic-syntax)
- [Query Types](#query-types)
- [Advanced Options](#advanced-options)
- [Output Sections](#output-sections)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

## Overview

`dig` is a flexible DNS lookup utility that performs DNS lookups and displays the answers from the queried name servers. It's the preferred tool for DNS troubleshooting by network administrators.

### Key Features
- DNS record querying
- DNS server testing
- DNSSEC validation
- Trace DNS resolution path
- Reverse DNS lookups
- Multiple query formats

## Installation

### Ubuntu (22.04/24.04)
```bash
sudo apt update
sudo apt install dnsutils
```

### macOS
```bash
# Using Homebrew
brew install bind

# Pre-installed on most macOS versions
```

## Basic Syntax

### Simple Query
```bash
# Basic domain lookup
dig example.com

# Short answer
dig example.com +short

# Specific record type
dig example.com MX
```

### Query Format
```bash
# General format
dig [@server] [domain] [type] [+options]

# Example with specific DNS server
dig @8.8.8.8 example.com
```

## Query Types

### Common Record Types
```bash
# A record (IPv4)
dig example.com A

# AAAA record (IPv6)
dig example.com AAAA

# MX record (Mail)
dig example.com MX

# TXT record
dig example.com TXT

# NS record (Nameserver)
dig example.com NS

# SOA record (Start of Authority)
dig example.com SOA

# CNAME record
dig www.example.com CNAME
```

### Reverse DNS Lookup
```bash
# PTR record lookup
dig -x 8.8.8.8

# IPv6 reverse lookup
dig -x 2001:db8::1
```

## Advanced Options

### Query Options
```bash
# Trace DNS resolution path
dig example.com +trace

# Use TCP instead of UDP
dig example.com +tcp

# Show timing information
dig example.com +stats

# Disable recursion
dig example.com +norecurse

# DNSSEC validation
dig example.com +dnssec
```

### Multiple Queries
```bash
# Multiple record types
dig example.com ANY

# Multiple domains
dig example.com google.com

# Multiple types for same domain
dig example.com NS MX
```

### Output Control
```bash
# Short output
dig example.com +short

# Detailed output
dig example.com +noall +answer +authority +additional

# Custom output fields
dig example.com +noall +answer +ttl
```

## Output Sections

### Understanding dig Output
```bash
; <<>> DiG 9.16.1-Ubuntu <<>> example.com
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12345
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; QUESTION SECTION:
;example.com.			IN	A

;; ANSWER SECTION:
example.com.		86400	IN	A	93.184.216.34
```

### Section Explanations
```bash
# Show only answer section
dig example.com +noall +answer

# Show only authority section
dig example.com +noall +authority

# Show only additional section
dig example.com +noall +additional
```

## Troubleshooting

### Common Issues

1. **DNS Resolution Problems**
```bash
# Check nameservers
dig example.com NS +short

# Verify authoritative servers
dig example.com SOA

# Test specific DNS server
dig @8.8.8.8 example.com
```

2. **DNSSEC Validation**
```bash
# Check DNSSEC
dig example.com +dnssec

# Verify DS records
dig example.com DS
```

3. **Response Time Issues**
```bash
# Show query timing
dig example.com +stats

# Compare different servers
dig @8.8.8.8 example.com +stats
dig @1.1.1.1 example.com +stats
```

## Advanced Use Cases

### Batch Processing
```bash
# Read queries from file
dig -f queries.txt

# Output to file
dig example.com > results.txt
```

### DNS Server Testing
```bash
# Test response time
dig example.com +tries=1 +retry=0

# Check server capabilities
dig chaos txt version.bind @dns.server
```

### AXFR (Zone Transfer)
```bash
# Attempt zone transfer
dig @ns1.example.com example.com AXFR

# With specific transfer key
dig @ns1.example.com example.com AXFR -k transfer.key
```

## Best Practices

### DNS Querying
```bash
# Always check authoritative answer
dig example.com +noadditional +noauthority

# Verify with multiple DNS servers
dig @8.8.8.8 example.com
dig @1.1.1.1 example.com
```

### Performance Testing
```bash
# Measure response time
dig example.com +stats +tries=1

# Test multiple queries
for i in {1..5}; do dig example.com +noall +answer; done
```

## Quick Reference

### Essential Commands
```bash
# Basic lookup
dig example.com

# Short answer
dig +short example.com

# Specific record
dig example.com MX

# Reverse lookup
dig -x IP_ADDRESS

# Trace resolution
dig +trace example.com
```

### Common Options
- `+short`: Brief output
- `+trace`: Follow delegation chain
- `+dnssec`: Show DNSSEC info
- `+noall`: Turn off all display flags
- `+answer`: Show answer section
- `+stats`: Show statistics
- `+tcp`: Use TCP instead of UDP
- `+norecurse`: Disable recursion

### Output Control Options
```bash
# Minimal output
dig +noall +answer example.com

# Full details
dig +nocmd +noall +answer +authority +additional example.com

# Custom timestamp
dig +time=1 +tries=1 example.com
```

---

Remember:
- Always specify record types when needed
- Use +short for scripting
- Check authoritative answers
- Verify DNSSEC when required
- Compare multiple DNS servers
- Document query results

For detailed information, consult the man pages (`man dig`).