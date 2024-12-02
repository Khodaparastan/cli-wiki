---
type: cheatsheet
tags:
  - SSH
  - devops
  - Remote
Category: Networking
---

## Basic SSH Connection
```bash
# Basic SSH connection
ssh username@hostname
ssh -p 2222 username@hostname    # Connect to specific port
ssh -i /path/to/key.pem username@hostname    # Use private key
```

## Remote Command Execution Syntax
```bash
# Basic remote command execution
ssh username@hostname 'command'
ssh username@hostname "command with spaces"

# Multiple commands
ssh username@hostname 'command1 && command2'
ssh username@hostname 'command1; command2'    # Execute sequentially regardless of success
```

## Environment Variables
```bash
# Using local environment variables
ssh username@hostname "MYVAR=$MYVAR command"
ssh username@hostname 'export MYVAR=value; command'

# Preserving environment variables
ssh -t username@hostname 'bash -l -c "command"'
```

## File Operations
```bash
# Execute local script on remote host
ssh username@hostname 'bash -s' < local_script.sh

# Combine with sudo
ssh username@hostname 'sudo bash -s' < local_script.sh

# Execute and redirect output
ssh username@hostname 'command' > local_output.txt
```

## Interactive Sessions
```bash
# Force pseudo-terminal allocation
ssh -t username@hostname 'command'

# Keep session alive
ssh -o ServerAliveInterval=60 username@hostname
```

## Background Processes
```bash
# Run command in background
ssh username@hostname 'nohup command &'

# Run command that survives SSH session
ssh username@hostname 'screen -dm command'
ssh username@hostname 'tmux new-session -d "command"'
```

## SSH Options
```bash
# Disable strict host key checking
ssh -o StrictHostKeyChecking=no username@hostname 'command'

# Compression for slow connections
ssh -C username@hostname 'command'

# Quiet mode
ssh -q username@hostname 'command'
```

## X11 Forwarding
```bash
# Enable X11 forwarding
ssh -X username@hostname 'gui_application'
ssh -Y username@hostname 'gui_application'    # Trusted X11 forwarding
```

## Port Forwarding
```bash
# Local port forwarding
ssh -L local_port:remote_host:remote_port username@hostname

# Remote port forwarding
ssh -R remote_port:local_host:local_port username@hostname
```

## Performance Optimization
```bash
# Enable compression
ssh -C username@hostname 'command'

# Use faster ciphers
ssh -c aes128-gcm@openssh.com username@hostname 'command'
```

## Security
```bash
# Use specific identity file
ssh -i ~/.ssh/specific_key username@hostname 'command'

# Specify cipher
ssh -c aes256-gcm@openssh.com username@hostname 'command'

# Use specific SSH key algorithm
ssh -o PubkeyAcceptedKeyTypes=ssh-rsa username@hostname 'command'
```

## Troubleshooting
```bash
# Verbose output for debugging
ssh -v username@hostname 'command'    # Verbose
ssh -vv username@hostname 'command'   # More verbose
ssh -vvv username@hostname 'command'  # Even more verbose

# Check connection without executing command
ssh -q username@hostname 'exit 0'
```

## Best Practices
1. Always use quotes around remote commands
2. Use double quotes when variable expansion is needed
3. Use single quotes for literal strings
4. Consider using `nohup` or `screen`/`tmux` for long-running processes
5. Always handle error cases in scripts
6. Use SSH config file for frequently used connections
7. Implement proper key management
8. Use SSH agent for key management

## SSH Config File Example
```bash
# ~/.ssh/config
Host myserver
    HostName server.example.com
    User username
    Port 2222
    IdentityFile ~/.ssh/specific_key
    ServerAliveInterval 60
```
