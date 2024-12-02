
## Network Analysis & Monitoring
```bash
# tcpdump - Packet analyzer
tcpdump -i any port 80

# tshark - CLI version of Wireshark
tshark -i eth0 -f "port 80"

# mtr - Modern traceroute/ping combo
mtr google.com

# iftop - Network bandwidth monitor
sudo iftop -i eth0

# nethogs - Per-process bandwidth monitor
sudo nethogs eth0
```

## System Monitoring
```bash
# htop - Interactive process viewer
htop

# glances - System monitor with web interface
glances

# nmon - Performance monitor
nmon

# sysstat (iostat, mpstat, sar)
iostat -x 1
sar -n DEV 1
```

## Network Testing
```bash
# wrk - HTTP benchmarking tool
wrk -t12 -c400 -d30s http://localhost

# ab - Apache benchmark tool
ab -n 1000 -c 100 http://localhost/

# siege - HTTP load testing
siege -c 100 -t 30S http://localhost

# hey - HTTP load generator
hey -n 1000 http://localhost/
```

## DNS Tools
```bash
# doggo - Modern DNS client
doggo example.com

# dnstracer - Trace DNS path
dnstracer example.com

# dnsdist - DNS loadbalancer
dnsdist -C dnsdist.conf

# pdns - PowerDNS utilities
pdns_control ping
```

## SSL/TLS Analysis
```bash
# sslyze - SSL/TLS scanner
sslyze --regular example.com

# testssl.sh - SSL/TLS testing
./testssl.sh example.com

# sslscan - SSL/TLS scanner
sslscan example.com
```

## Web Testing
```bash
# httpie - Modern HTTP client
http GET example.com

# curlie - Curl with httpie UI
curlie example.com

# xh - Friendly HTTP client
xh example.com

# websocat - WebSocket client
websocat ws://example.com/socket
```

## Network Discovery
```bash
# masscan - Fast port scanner
sudo masscan -p1-65535 192.168.1.0/24

# zmap - Network scanner
sudo zmap -p 80 192.168.1.0/24

# arp-scan - ARP scanner
sudo arp-scan --localnet

# netdiscover - Active/passive ARP recon
sudo netdiscover -r 192.168.1.0/24
```

## Bandwidth Testing
```bash
# speedtest-cli - Speed test
speedtest-cli

# fast-cli - Netflix speed test
fast-cli

# bmon - Bandwidth monitor
bmon

# cbm - Color bandwidth meter
cbm
```

## Network Configuration
```bash
# nmtui - NetworkManager TUI
nmtui

# iw - Wireless configuration
iw dev wlan0 scan

# iwctl - iwd wireless daemon client
iwctl

# bridge-utils
brctl show
```

## Protocol Analysis
```bash
# termshark - Terminal Wireshark
termshark

# ngrep - Network grep
ngrep -d eth0 "^GET |^POST "

# tcpflow - TCP flow recorder
tcpflow -i eth0 port 80

# snort - Network IDS
snort -dev -l ./log
```

## System Tools
```bash
# tmux - Terminal multiplexer
tmux

# screen - Terminal multiplexer
screen

# atop - Advanced system monitor
atop

# dstat - Versatile resource stats
dstat -cdngy
```

## Log Analysis
```bash
# goaccess - Real-time log analyzer
goaccess access.log -c

# logwatch - Log analyzer and reporter
logwatch --detail high

# multitail - Multiple log viewer
multitail /var/log/syslog /var/log/auth.log

# lnav - Log file navigator
lnav /var/log/*
```

## Performance Analysis
```bash
# perf - Linux profiling
perf stat command

# strace - System call tracer
strace command

# sysdig - System activity monitor
sysdig

# bpftrace - System analysis
bpftrace -e 'tracepoint:syscalls:sys_enter_*'
```

## Container Networking
```bash
# ctop - Container metrics viewer
ctop

# dive - Docker image explorer
dive image:tag

# kubectl - Kubernetes CLI
kubectl get pods -o wide

# stern - Multi pod/container log tailing
stern pod-query
```

## Security Tools
```bash
# fail2ban-client - Ban management
sudo fail2ban-client status

# rkhunter - Rootkit hunter
sudo rkhunter --check

# lynis - Security auditing
sudo lynis audit system

# nikto - Web server scanner
nikto -h example.com
```

## File Transfer
```bash
# rsync - Fast file transfer
rsync -avz source/ dest/

# magic-wormhole - Secure file transfer
wormhole send file.txt

# croc - File transfer
croc send file.txt

# ffsend - Firefox Send client
ffsend upload file.txt
```

## Useful Utilities
```bash
# jq - JSON processor
curl api.example.com | jq .

# yq - YAML processor
yq eval '.key' file.yaml

# fzf - Fuzzy finder
history | fzf

# ripgrep - Fast grep
rg pattern
```

## Installation
```bash
# Ubuntu/Debian
sudo apt install \
    htop glances nmon sysstat \
    iftop nethogs bmon \
    mtr-tiny tcpdump tshark \
    httpie curl \
    tmux screen \
    iotop atop \
    rsync

# macOS (Homebrew)
brew install \
    htop glances nmon \
    iftop nethogs bmon \
    mtr tcpdump wireshark \
    httpie curl \
    tmux screen \
    rsync
```

Remember:
- Keep tools updated
- Check man pages for detailed usage
- Consider security implications
- Test in safe environment first
- Monitor system impact
- Document configurations

These tools significantly enhance network troubleshooting and system administration capabilities. Choose tools based on your specific needs and system requirements.