
## Table of Contents
- [Network Discovery](#network-discovery)
- [Connectivity Testing](#connectivity-testing)
- [Port Analysis](#port-analysis)
- [Network Performance](#network-performance)
- [DNS Operations](#dns-operations)
- [VPN Management](#vpn-management)
- [Firewall Operations](#firewall-operations)
- [Traffic Analysis](#traffic-analysis)
- [Security Scanning](#security-scanning)

## Network Discovery

### Scan Network Range
```bash
# Quick network scan
nmap -sn 192.168.1.0/24

# Detailed host scan
nmap -A 192.168.1.0/24

# List all active IPs
ip neigh show
arp -a
```

### Interface Management
```bash
# Show interfaces
ip link show
ifconfig -a

# Interface details
ip addr show dev eth0
ethtool eth0

# Monitor interface traffic
iftop -i eth0
```

## Connectivity Testing

### Basic Connectivity
```bash
# Test reachability
ping -c 4 target.com

# Trace route
traceroute target.com
mtr target.com

# Quick port check
nc -zv target.com 80
```

### Advanced Testing
```bash
# TCP connection test
telnet target.com 80

# SSL connection test
openssl s_client -connect target.com:443

# UDP port test
nc -zuv target.com 53
```

## Port Analysis

### Port Scanning
```bash
# Quick port scan
nmap -F target.com

# Full port scan
nmap -p- target.com

# Service version detection
nmap -sV target.com
```

### Open Port Monitoring
```bash
# List listening ports
netstat -tulpn
ss -tulpn

# Check specific port
lsof -i :80
```

## Network Performance

### Bandwidth Testing
```bash
# Server mode
iperf3 -s

# Client mode
iperf3 -c server_ip

# UDP test
iperf3 -u -c server_ip -b 100M
```

### Connection Monitoring
```bash
# Monitor connections
netstat -ant | grep ESTABLISHED
ss -ant | grep ESTABLISHED

# Connection statistics
netstat -s
ss -s
```

## DNS Operations

### DNS Queries
```bash
# Basic lookup
dig example.com

# Reverse lookup
dig -x IP_ADDRESS

# DNS trace
dig +trace example.com

# All records
dig example.com ANY
```

### DNS Troubleshooting
```bash
# Check nameservers
dig NS example.com

# Test specific nameserver
dig @8.8.8.8 example.com

# Check MX records
dig MX example.com
```

## VPN Management

### OpenVPN
```bash
# Start OpenVPN server
systemctl start openvpn@server

# Check VPN status
systemctl status openvpn@server

# Client connection
openvpn --config client.ovpn
```

### Cloudflare WARP
```bash
# Connect WARP
warp-cli connect

# Check status
warp-cli status

# Enable Zero Trust
warp-cli teams-enroll <token>
```

## Firewall Operations

### UFW Management
```bash
# Allow service
sudo ufw allow ssh
sudo ufw allow http

# Check status
sudo ufw status verbose

# Enable logging
sudo ufw logging on
```

### iptables Rules
```bash
# Allow incoming port
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Allow established connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Save rules
sudo netfilter-persistent save
```

## Traffic Analysis

### Packet Capture
```bash
# Capture on interface
tcpdump -i eth0

# Capture specific port
tcpdump -i eth0 port 80

# Write to file
tcpdump -i eth0 -w capture.pcap
```

### Connection Tracking
```bash
# Track connections
netstat -ant
ss -ant

# Monitor real-time
watch -n1 'netstat -ant | grep ESTABLISHED'
```

## Security Scanning

### Network Security
```bash
# Security scan
nmap -sV --script vuln target.com

# Check open ports
nmap -sS target.com

# SSL certificate check
openssl s_client -connect target.com:443
```

### Service Testing
```bash
# Test web server
curl -Iv https://target.com

# Test mail server
nc -v mail.target.com 25

# Test SSH server
ssh -v user@target.com
```

## Quick Solutions for Common Tasks

### Website Accessibility
```bash
# Full check
curl -Iv https://website.com
nc -zv website.com 443
dig website.com
```

### Mail Server Issues
```bash
# Check mail setup
dig MX domain.com
nc -zv mail.domain.com 25
openssl s_client -connect mail.domain.com:587
```

### Network Performance Issues
```bash
# Basic tests
ping gateway_ip
mtr problem_host
iperf3 -c test_server
```

### DNS Problems
```bash
# Full DNS check
dig +trace domain.com
nslookup domain.com
dig domain.com ANY
```

### Security Audit
```bash
# Basic security check
nmap -sV target.com
sudo ufw status
netstat -tulpn
```

### VPN Troubleshooting
```bash
# Connection check
ping vpn_gateway
traceroute vpn_gateway
systemctl status openvpn
```

---

Remember:
- Always check permissions before running commands
- Document changes to network configuration
- Keep logs of troubleshooting steps
- Use appropriate flags for verbosity
- Consider security implications
- Backup configurations before changes

This cheatsheet covers the most common networking tasks. For detailed options and advanced usage, refer to the respective man pages or documentation.