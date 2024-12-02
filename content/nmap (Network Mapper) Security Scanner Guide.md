
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Scanning](#basic-scanning)
- [Port Scanning](#port-scanning)
- [Host Discovery](#host-discovery)
- [Service/Version Detection](#serviceversion-detection)
- [OS Detection](#os-detection)
- [Script Scanning](#script-scanning)
- [Advanced Options](#advanced-options)
- [Security Considerations](#security-considerations)
- [Best Practices](#best-practices)

## Overview

`nmap` is a powerful network scanner used for security auditing and network exploration. It can discover hosts, services, operating systems, and vulnerabilities in a network.

### Key Features
- Port scanning
- Host discovery
- Service version detection
- OS fingerprinting
- NSE scripting engine
- Network mapping
- Security auditing

## Installation

### Ubuntu (22.04/24.04)
```bash
sudo apt update
sudo apt install nmap
```

### macOS
```bash
brew install nmap
```

## Basic Scanning

### Simple Scans
```bash
# Basic scan
nmap target.com

# Scan specific IP
nmap 192.168.1.1

# Scan IP range
nmap 192.168.1.1-254

# Scan subnet
nmap 192.168.1.0/24
```

### Output Options
```bash
# Normal output
nmap -oN scan.txt target.com

# XML output
nmap -oX scan.xml target.com

# All output formats
nmap -oA scan_results target.com

# Grepable output
nmap -oG scan.grep target.com
```

## Port Scanning

### Port Selection
```bash
# Scan specific ports
nmap -p 80,443 target.com

# Scan port range
nmap -p 1-1000 target.com

# Scan all ports
nmap -p- target.com

# Top ports
nmap --top-ports 100 target.com
```

### Scan Types
```bash
# SYN scan (default)
sudo nmap -sS target.com

# TCP connect scan
nmap -sT target.com

# UDP scan
sudo nmap -sU target.com

# FIN scan
sudo nmap -sF target.com

# NULL scan
sudo nmap -sN target.com

# XMAS scan
sudo nmap -sX target.com
```

## Host Discovery

### Discovery Methods
```bash
# Ping scan
nmap -sn 192.168.1.0/24

# No ping
nmap -Pn target.com

# TCP SYN ping
nmap -PS22,80,443 target.com

# TCP ACK ping
nmap -PA80,443 target.com

# UDP ping
nmap -PU161 target.com
```

### List Scan
```bash
# List targets only
nmap -sL 192.168.1.0/24

# List with DNS resolution
nmap -sL -n 192.168.1.0/24
```

## Service/Version Detection

### Version Detection
```bash
# Basic version detection
nmap -sV target.com

# Aggressive version detection
nmap -sV --version-intensity 5 target.com

# Light version detection
nmap -sV --version-intensity 2 target.com
```

### Service Scan Options
```bash
# Version detection with default scripts
nmap -sV -sC target.com

# All version detection probes
nmap -sV --version-all target.com
```

## OS Detection

### OS Fingerprinting
```bash
# Basic OS detection
sudo nmap -O target.com

# OS detection with version detection
sudo nmap -O -sV target.com

# Aggressive OS detection
sudo nmap -O --osscan-guess target.com
```

### OS Scan Options
```bash
# Limit OS detection
sudo nmap -O --max-os-tries 1 target.com

# OS scan with timing
sudo nmap -O -T4 target.com
```

## Script Scanning

### NSE Scripts
```bash
# Default scripts
nmap -sC target.com

# Specific script
nmap --script=http-title target.com

# Multiple scripts
nmap --script=http-title,http-headers target.com

# Script categories
nmap --script=vuln target.com
```

### Script Categories
```bash
# Auth scripts
nmap --script auth target.com

# Vulnerability scripts
nmap --script vuln target.com

# Discovery scripts
nmap --script discovery target.com

# Safe scripts
nmap --script safe target.com
```

## Advanced Options

### Timing Templates
```bash
# Paranoid timing
nmap -T0 target.com

# Sneaky timing
nmap -T1 target.com

# Polite timing
nmap -T2 target.com

# Normal timing
nmap -T3 target.com

# Aggressive timing
nmap -T4 target.com

# Insane timing
nmap -T5 target.com
```

### Firewall/IDS Evasion
```bash
# Fragment packets
nmap -f target.com

# Specify MTU
nmap --mtu 24 target.com

# Decoy scanning
nmap -D RND:10 target.com

# Idle zombie scan
nmap -sI zombie_host target.com
```

### Performance Tuning
```bash
# Parallel host scan
nmap --min-hostgroup 100 target.com

# Aggressive timing
nmap -T4 --min-parallelism 100 target.com
```

## Security Considerations

### Safe Scanning
```bash
# No ping
nmap -Pn target.com

# Limited rate
nmap --max-rate 100 target.com

# Conservative timing
nmap -T2 target.com
```

### Stealth Options
```bash
# Delayed scan
nmap --scan-delay 1s target.com

# Random data
nmap --data-length 24 target.com
```

## Best Practices

### Network Scanning
```bash
# Comprehensive scan
sudo nmap -sS -sV -O -A target.com

# Quick network sweep
nmap -sn -T4 192.168.1.0/24

# Safe production scan
nmap -T2 -sT -p- target.com
```

### Documentation
```bash
# Full documentation
nmap -v -A -oA scan_results target.com

# Regular monitoring
nmap -sV --script vuln -oN weekly_scan.txt target.com
```

## Quick Reference

### Essential Commands
```bash
# Quick scan
nmap target.com

# Comprehensive scan
sudo nmap -sS -sV -O -A target.com

# Network sweep
nmap -sn 192.168.1.0/24

# Version detection
nmap -sV target.com

# Script scan
nmap -sC target.com
```

### Common Options
```bash
-sS    # SYN scan
-sT    # TCP connect scan
-sU    # UDP scan
-sV    # Version detection
-O     # OS detection
-A     # Aggressive scan
-p     # Port specification
-T0-5  # Timing template
--script # NSE scripts
-oA    # Output all formats
```

### Scan Combinations
```bash
# Full TCP scan
sudo nmap -sS -sV -O -p- target.com

# Quick vulnerability scan
nmap -sV --script vuln target.com

# Comprehensive audit
sudo nmap -sS -sV -O -A -p- -T4 target.com
```

---

Remember:
- Always obtain proper authorization before scanning
- Use appropriate timing options for target network
- Document all scanning activities
- Consider network impact
- Follow security best practices

For detailed information, consult the man pages (`man nmap`) and official documentation.