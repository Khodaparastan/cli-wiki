## Layer 1 (Physical Layer) Diagnostics

### Cable and Physical Connection Testing
```bash
# Network Interface Status
# macOS
system_profiler SPNetworkDataType | grep "Link Speed"
networksetup -getmedia <interface>

# Linux
ethtool <interface> | grep "Speed|Link"
mii-tool <interface>
```

### Signal Quality and Error Detection
```bash
# Check interface errors
# Both OS
netstat -i
ifconfig <interface> | grep "errors|dropped|collisions"

# Linux specific
ethtool -S <interface>  # Detailed interface statistics
```

### Physical Layer Monitoring Tools
```bash
# Linux
ethtool --statistics <interface>
ethtool --register-dump <interface>  # Raw register data

# Check for CRC errors
ifconfig <interface> | grep "CRC"
```

### WiFi Signal Diagnostics
```bash
# macOS
/System/Library/PrivateFrameworks/Apple80211.framework/Versions/Current/Resources/airport -I
airport -s  # Scan available networks

# Linux
iwconfig
iwlist <interface> scanning
iw dev <interface> station dump  # Detailed wifi statistics
```

## Layer 2 (Data Link Layer) Diagnostics

### Interface Status and Configuration
```bash
# macOS
ifconfig
networksetup -listallhardwareports
system_profiler SPNetworkDataType

# Linux
ip link show
ethtool <interface>
mii-tool <interface>
```

### MAC Address and ARP
```bash
# View MAC address table
arp -a   # Both OS

# Clear ARP cache
# macOS
sudo arp -d -a
# Linux
sudo ip neigh flush all

# Watch ARP traffic
tcpdump -n arp
```

### VLAN Diagnostics
```bash
# Linux
ip link add link eth0 name eth0.100 type vlan id 100
vconfig show

# macOS
vlan show
```

## Layer 3 (Network Layer) Diagnostics

### IPv4 Connectivity
```bash
# Basic connectivity
ping <host>
ping -c 4 <host>    # Limited count

# Path MTU Discovery
ping -D -s 1472 <host>    # macOS
ping -M do -s 1472 <host> # Linux

# Advanced ping options
ping -f <host>      # Flood ping (requires root)
ping -i 0.2 <host>  # Fast interval
```

### IPv6 Connectivity
```bash
# Basic IPv6 ping
ping6 <ipv6_host>
ping -6 <ipv6_host>

# Link-local address test
ping6 fe80::<interface>

# IPv6 neighbor discovery
nd6 -n               # macOS
ip -6 neighbor show  # Linux
```

### Routing
```bash
# Show routing table
netstat -rn
route -n
ip route show    # Linux

# IPv6 routes
netstat -rn -f inet6
ip -6 route show     # Linux

# Trace route
traceroute <host>    # IPv4
traceroute6 <host>   # IPv6
mtr <host>          # Continuous trace
```

## Network Services Diagnostics

### DNS
```bash
# DNS lookup
dig <domain>
nslookup <domain>
host <domain>

# Reverse DNS
dig -x <ip_address>

# DNS server test
dig @8.8.8.8 <domain>

# Clear DNS cache
# macOS
sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder
# Linux
sudo systemd-resolve --flush-caches
```

### DHCP
```bash
# Release DHCP lease
# macOS
sudo ipconfig set <interface> DHCP
# Linux
sudo dhclient -r <interface>

# Request new lease
# macOS
sudo ipconfig set <interface> BOOTP
sudo ipconfig set <interface> DHCP
# Linux
sudo dhclient <interface>
```

## Network Performance Testing

### Bandwidth Testing
```bash
# Using iperf3
iperf3 -s              # Server mode
iperf3 -c <server_ip>  # Client mode

# Basic speed test
speedtest-cli
```

### Socket Statistics
```bash
# Open connections
netstat -an
ss -tuln    # Linux

# Connection statistics
netstat -s
```

## Advanced Diagnostics

### Packet Capture
```bash
# Basic capture
tcpdump -i <interface>

# Capture with filters
tcpdump -i <interface> host <ip>
tcpdump -i <interface> port 80

# Save capture
tcpdump -i <interface> -w capture.pcap
```

### Network Load
```bash
# Interface statistics
netstat -i
ip -s link show

# Real-time bandwidth monitoring
iftop
nload
```

### SSL/TLS Diagnostics
```bash
# Test SSL connection
openssl s_client -connect host:443

# Check certificate
openssl x509 -in cert.pem -text
```

## System Network Configuration

### Network Services Status
```bash
# macOS
sudo launchctl list | grep network
networksetup -listallnetworkservices

# Linux
systemctl status network
systemctl status NetworkManager
```

### Firewall Diagnostics
```bash
# macOS
sudo pfctl -sa
sudo pfctl -sr

# Linux
sudo iptables -L
sudo nft list ruleset
```

## Network Quality Tests
```bash
# macOS
networkQuality    # Built-in tool in newer versions

# Both OS
mtr <host>       # Network path analysis
curl -o /dev/null http://speedtest.net/mini -w "%{time_total}\n"   # Basic latency test
```

### Common Issues Resolution
```bash
# Reset entire network stack macOS
sudo ifconfig <interface> down
sudo route flush
sudo ifconfig <interface> up

# Linux
sudo systemctl restart NetworkManager
sudo netplan apply    # Ubuntu
```


## Layer 4 (Transport Layer) Diagnostics

### TCP Connection Analysis
```bash
# Active TCP connections
netstat -tnp
ss -tnp  # Linux, more detailed

# TCP connection states
netstat -ant | awk '{print $6}' | sort | uniq -c

# Detailed TCP statistics
netstat -st  # Both OS
cat /proc/net/tcp  # Linux
```

### TCP Connection Testing
```bash
# Test specific port
nc -zv host port
telnet host port

# TCP connection tracking (Linux)
conntrack -L
ss -t -a
```

### UDP Diagnostics
```bash
# Show UDP sockets
netstat -unp
ss -unp  # Linux

# UDP port testing
nc -zu host port

# Monitor UDP traffic
tcpdump udp
```

### Socket Statistics
```bash
# Detailed socket stats
# macOS
netstat -av
lsof -i

# Linux
ss -s  # Summary statistics
ss -m  # Memory usage
```

### Performance Testing
```bash
# TCP Window Size
sysctl net.inet.tcp.window_size  # macOS
sysctl net.ipv4.tcp_window_scaling  # Linux

# Connection Latency
hping3 -S -p 80 host  # TCP SYN timing
```

## Layer 5 (Session Layer) Diagnostics

### Session Management
```bash
# Active sessions
who
w
last

# SSH Sessions
who | grep pts
```

### Application Session Testing
```bash
# Test SSL/TLS sessions
openssl s_client -connect host:443 -state
openssl s_client -connect host:443 -debug

# Session timeout testing
curl -v --max-time 10 https://host
```

### Session Protocol Analysis
```bash
# Monitor session establishment
tcpdump -i any 'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0'

# Track session state changes
tcpdump -i any 'tcp[tcpflags] & (tcp-syn|tcp-fin|tcp-rst) != 0'
```

## Cross-Layer Analysis Tools

### Network Protocol Analyzer
```bash
# Wireshark CLI (tshark)
tshark -i <interface>
tshark -i <interface> -f "port 80 or port 443"

# Advanced filtering
tshark -i <interface> -Y "tcp.flags.syn==1"
```

### Connection Flow Analysis
```bash
# netflow analysis (if installed)
nfdump -R /path/to/flows
fprobe <interface>

# Connection tracking
conntrack -L
conntrack -E  # Event monitoring
```


## Advanced Diagnostics

### Hardware Diagnostics
```bash
# PCI Network Device Info
lspci -vv | grep -A 10 Network
system_profiler SPNetworkDataType  # macOS

# Driver Information
ethtool -i <interface>
kextstat | grep -i network  # macOS
```

### Protocol-Specific Tools
```bash
# SSL/TLS Analysis
ssldump -i <interface>
sslscan host:port

# TCP Dump with Session Context
tcpdump -i <interface> -s 0 -w capture.pcap
```


## Performance Optimization
```bash
# TCP Optimization
sysctl -w net.ipv4.tcp_window_scaling=1
sysctl -w net.ipv4.tcp_timestamps=1

# Session Optimization
sysctl -w net.ipv4.tcp_max_syn_backlog=4096
```

## Layer 6 (Presentation Layer) Diagnostics

### Data Encoding/Decoding
```bash
# Character Set Analysis
file -i <filename>
chardet <filename>

# Text Encoding Conversion
iconv -f UTF-8 -t ASCII file.txt
iconv -l  # List available encodings
```

### Compression Testing
```bash
# Test compression algorithms
gzip -t file.gz
bzip2 -t file.bz2

# Check compression ratio
gzip -l file.gz
```

### Encryption Diagnostics
```bash
# SSL/TLS Certificate Analysis
openssl x509 -in cert.pem -text -noout
openssl verify cert.pem

# Check SSL/TLS Configuration
openssl s_client -connect host:443 -tls1_2
nmap --script ssl-enum-ciphers -p 443 host
```

### MIME Type Analysis
```bash
# Check MIME types
file --mime-type file
mimetype file    # Linux

# HTTP Content-Type checking
curl -I https://example.com
```

## Layer 7 (Application Layer) Diagnostics

### HTTP/HTTPS Diagnostics
```bash
# Basic HTTP testing
curl -v https://example.com
wget --debug https://example.com

# HTTP headers
curl -I https://example.com
curl -D - https://example.com

# HTTP performance
curl -w "\
    time_namelookup:  %{time_namelookup}\n\
    time_connect:  %{time_connect}\n\
    time_appconnect:  %{time_appconnect}\n\
    time_pretransfer:  %{time_pretransfer}\n\
    time_redirect:  %{time_redirect}\n\
    time_starttransfer:  %{time_starttransfer}\n\
    ----------\n\
    time_total:  %{time_total}\n" \
    -o /dev/null -s https://example.com
```

### DNS Diagnostics
```bash
# DNS lookup tools
dig +trace domain.com
host -a domain.com
nslookup -debug domain.com

# DNS zone transfer
dig @nameserver domain.com AXFR

# DNS propagation
dig domain.com @8.8.8.8
dig domain.com @1.1.1.1

# DNS record types
dig domain.com ANY
dig domain.com MX
dig domain.com TXT
```

### Email Protocol Testing
```bash
# SMTP testing
telnet mail-server 25
nc -v mail-server 25

# Test email sending
swaks --to user@domain.com --from sender@domain.com

# Check mail server records
dig domain.com MX
dig domain.com TXT    # SPF records
```

### Web Application Testing
```bash
# Load testing
ab -n 1000 -c 10 https://example.com/
siege -c 10 -t 30S https://example.com/

# Response codes
curl -o /dev/null -s -w "%{http_code}\n" https://example.com/

# Content verification
curl -s https://example.com/ | grep "expected_text"
```

### Application Performance Monitoring
```bash
# Response time monitoring
httping -c 10 example.com

# Server response headers
curl -s -D - example.com -o /dev/null

# TLS handshake timing
curl -w "TLS handshake: %{time_appconnect}\n" -o /dev/null -s https://example.com
```

## Protocol-Specific Diagnostics

### FTP Testing
```bash
# FTP connection test
ftp -n server <<EOF
quote USER username
quote PASS password
quit
EOF

# FTP through curl
curl -v ftp://server/
```

### Database Connection Testing
```bash
# MySQL
mysqladmin ping -h hostname
mysqlcheck -h hostname -u user -p

# PostgreSQL
pg_isready -h hostname
psql -h hostname -p 5432 -U username -c "\l"
```

### API Testing
```bash
# REST API testing
curl -X GET "https://api.example.com/endpoint" \
  -H "Authorization: Bearer token"

# API response timing
curl -w "@curl-format.txt" -o /dev/null -s "https://api.example.com"
```

## Application Layer Security

### Web Security Testing
```bash
# SSL/TLS security check
ssllabs-scan example.com
testssl.sh example.com

# HTTP security headers
curl -s -D - https://example.com | grep -i "security"
```

### Application Firewall Testing
```bash
# WAF detection
wafw00f https://example.com

# Basic security scanning
nikto -h https://example.com
```

## Troubleshooting Tools

### Application Layer Protocol Analysis
```bash
# HTTP traffic analysis
tcpdump -A -s 0 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'

# HTTPS traffic (requires key)
ssldump -i any -k keyfile.pem
```

### Service Discovery
```bash
# Network service scanning
nmap -sV -p- host
nmap -sC -sV host

# Service version detection
nc -zv host port
```

## Performance Analysis

### Load Testing Tools
```bash
# HTTP load testing
wrk -t12 -c400 -d30s https://example.com
hey -n 1000 -c 100 https://example.com

# Concurrent connections
apache2-utils
ab -c 100 -n 1000 https://example.com/
```

### Monitoring Tools
```bash
# Real-time HTTP traffic
httpry -i eth0
tcpflow -i eth0 -c port 80

# Application logs analysis
tail -f /var/log/apache2/access.log | goaccess
```

