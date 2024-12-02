
## Basic Setup and Configuration
```bash
# Enable ARD via Terminal
sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/kickstart -activate

# Configure ARD Access
sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/kickstart -configure -access -on -privs -all

# Restart ARD Service
sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/kickstart -restart
```

## Command Line Control
```bash
# Start ARD Agent
sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/ARDAgent

# Stop ARD Agent
sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/ARDAgent -stop

# Check ARD Status
sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/ARDAgent -status
```

## Remote Commands Execution
```bash
# Execute command on single machine
sudo ARDAgent -target <IP_ADDRESS> -exec "/path/to/command"

# Execute command on multiple machines
sudo ARDAgent -targets "<IP1,IP2,IP3>" -exec "/path/to/command"

# Execute with specific user credentials
sudo ARDAgent -target <IP_ADDRESS> -username <USER> -password <PASS> -exec "/path/to/command"
```

## File Management
```bash
# Copy file to remote machine
sudo ARDAgent -target <IP_ADDRESS> -copy "/path/to/local/file" -destination "/path/on/remote"

# Copy multiple files
sudo ARDAgent -target <IP_ADDRESS> -copy "/path/to/files/*" -destination "/path/on/remote"

# Install package
sudo ARDAgent -target <IP_ADDRESS> -install "/path/to/package.pkg"
```

## System Reports
```bash
# Generate system report
sudo ARDAgent -target <IP_ADDRESS> -report system

# Generate hardware report
sudo ARDAgent -target <IP_ADDRESS> -report hardware

# Generate software report
sudo ARDAgent -target <IP_ADDRESS> -report software
```

## Screen Sharing
```bash
# Start screen sharing session
sudo ARDAgent -target <IP_ADDRESS> -screenshare

# Start screen sharing in observe mode
sudo ARDAgent -target <IP_ADDRESS> -observe

# Start screen sharing with control
sudo ARDAgent -target <IP_ADDRESS> -control
```

## Task Automation
```bash
# Schedule a task
sudo ARDAgent -target <IP_ADDRESS> -schedule "task_name" -at "YYYY-MM-DD HH:MM" -exec "/path/to/command"

# Create recurring task
sudo ARDAgent -target <IP_ADDRESS> -schedule "task_name" -repeat "daily|weekly|monthly" -exec "/path/to/command"
```

## Security Settings
```bash
# Set access privileges
sudo ARDAgent -configure -access -privs -all|-none|-limited

# Configure authentication
sudo ARDAgent -configure -allowAccessFor -specifiedUsers

# Set encryption
sudo ARDAgent -configure -clientopts -setencryption -yes|-no
```

## Network Configuration
```bash
# Set custom port
sudo ARDAgent -configure -port <PORT_NUMBER>

# Enable/disable UDP
sudo ARDAgent -configure -udp -active|-inactive

# Configure network discovery
sudo ARDAgent -configure -networkbrowse -active|-inactive
```

## Maintenance and Troubleshooting
```bash
# Clear ARD database
sudo ARDAgent -remove -db

# Reset all settings
sudo ARDAgent -configure -reset

# Generate debug logs
sudo ARDAgent -debug -level [1-3]
```

## Best Practices
1. Always use secure passwords and encryption
2. Regularly backup ARD database
3. Use computer lists for better organization
4. Document custom scripts and commands
5. Implement proper access control
6. Monitor task execution logs
7. Use encrypted connections when possible
8. Regularly update ARD software

## Configuration File Location
```bash
# Main configuration
/Library/Preferences/com.apple.RemoteDesktop.plist

# User preferences
~/Library/Preferences/com.apple.RemoteDesktop.plist
```

## Common Flags
```bash
-quiet          # Suppress output
-verbose        # Detailed output
-timeout <secs> # Set command timeout
-async          # Asynchronous execution
-wait           # Wait for completion
```

Remember to:
- Replace `<IP_ADDRESS>` with actual target IP
- Use appropriate permissions when executing commands
- Test commands on a single machine before mass deployment
- Keep logs for troubleshooting
- Verify network connectivity before remote execution
