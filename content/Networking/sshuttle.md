---
title: sshuttle
tags:
  - ssh
  - tunnel
  - vpn
---
```bash

sshuttle -vHr USERNAME@YOUR_VPS_IP_ADDRESS 0.0.0.0/0 --dns --no-latency-control	

# with Auto-Password

echo YOUR_ROOT_PASSWORD | sshpass -p YOUR_ROOT_PASSWORD sshuttle -r root@250.240.230.220 0/0 -x 250.240.230.220 --dns --no-latency-control --ssh-cmd "sshpass -p YOUR_SERVER_PASSWORD ssh"

```