
## Table of Contents
- [Overview](#overview)
- [Basic Usage](#basic-usage)
- [Command Syntax](#command-syntax)
- [Pattern Space Operations](#pattern-space-operations)
- [Regular Expressions](#regular-expressions)
- [Advanced Features](#advanced-features)
- [Common Use Cases](#common-use-cases)
- [Best Practices](#best-practices)

## Overview

sed (stream editor) is a powerful text processing tool that performs text transformations on an input stream (file or input from a pipeline).

### Key Features
- Line-based text processing
- Regular expression support
- In-place file editing
- Multiple commands
- Pattern matching
- Hold buffer operations
- Address ranges
- Branching capabilities

## Basic Usage

### Simple Substitution
```bash
# Basic substitution
sed 's/old/new/' file.txt

# Global substitution (all occurrences)
sed 's/old/new/g' file.txt

# Case insensitive substitution
sed 's/old/new/gi' file.txt

# In-place editing
sed -i 's/old/new/g' file.txt
```

### Common Options
```bash
# Make backup before editing
sed -i.bak 's/old/new/g' file.txt

# Extended regular expressions
sed -E 's/regex/replacement/g' file.txt

# Quiet mode
sed -n 's/pattern/replacement/p' file.txt
```

## Command Syntax

### Address Specification
```bash
# Specific line
sed '5s/old/new/' file.txt

# Line range
sed '1,5s/old/new/' file.txt

# Pattern match
sed '/pattern/s/old/new/' file.txt

# Last line
sed '$s/old/new/' file.txt
```

### Multiple Commands
```bash
# Using semicolon
sed 's/one/1/g; s/two/2/g' file.txt

# Using -e option
sed -e 's/one/1/g' -e 's/two/2/g' file.txt

# Using script file
sed -f commands.sed file.txt
```

## Pattern Space Operations

### Basic Operations
```bash
# Print pattern space
sed 'p' file.txt

# Delete pattern space
sed 'd' file.txt

# Next line
sed 'n' file.txt

# Append line
sed 'a\new line' file.txt

# Insert line
sed 'i\new line' file.txt
```

### Hold Buffer Operations
```bash
# Copy to hold buffer
sed 'h' file.txt

# Get from hold buffer
sed 'g' file.txt

# Exchange with hold buffer
sed 'x' file.txt

# Append to hold buffer
sed 'H' file.txt
```

## Regular Expressions

### Basic Patterns
```bash
# Beginning of line
sed 's/^pattern/new/' file.txt

# End of line
sed 's/pattern$/new/' file.txt

# Word boundaries
sed 's/\bword\b/new/g' file.txt

# Character classes
sed 's/[0-9]/X/g' file.txt
```

### Extended Patterns
```bash
# One or more occurrences
sed -E 's/a+/A/g' file.txt

# Zero or more occurrences
sed -E 's/a*/A/g' file.txt

# Optional occurrence
sed -E 's/colou?r/color/g' file.txt

# Grouping
sed -E 's/(word1|word2)/new/g' file.txt
```

## Advanced Features

### Branching
```bash
# Test and branch
sed '/pattern/b label' file.txt

# Test and branch if not matched
sed '/pattern/!b label' file.txt

# Define label
sed ': label
     commands
     b label' file.txt
```

### Flow Control
```bash
# If-then structure
sed '/pattern/{
    s/old/new/
    s/another/change/
}' file.txt

# Unless structure
sed '/pattern/!{
    s/old/new/
}' file.txt
```

### Multi-line Operations
```bash
# Join lines
sed 'N;s/\n/ /' file.txt

# Process multi-line pattern
sed '/start/,/end/s/old/new/' file.txt

# Delete until pattern
sed '/pattern/,$d' file.txt
```

## Common Use Cases

### Text Transformation
```bash
# Remove empty lines
sed '/^$/d' file.txt

# Remove comments
sed 's/#.*$//' file.txt

# Remove leading/trailing spaces
sed 's/^[ \t]*//;s/[ \t]*$//' file.txt

# Convert Windows to Unix line endings
sed 's/\r$//' file.txt
```

### Data Extraction
```bash
# Extract emails
sed -n 's/.*\([a-zA-Z0-9._%+-]\+@[a-zA-Z0-9.-]\+\.[a-zA-Z]\{2,\}\).*/\1/p' file.txt

# Extract IP addresses
sed -n 's/.*\([0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\).*/\1/p' file.txt
```

## Best Practices

### Performance Tips
```bash
# Use appropriate delimiters
sed 's|/path/to/file|/new/path|g' file.txt

# Minimize pattern space operations
sed -n '10,20p' file.txt

# Use extended regex when needed
sed -E 's/complex|pattern/simple/g' file.txt
```

### Error Handling
```bash
# Check for file existence
[ -f "$file" ] && sed 's/old/new/g' "$file"

# Handle special characters
sed 's/[\/&]/\\&/g' file.txt
```

## Example Scripts

### Log Processing
```bash
#!/bin/bash
# Process log files
sed '
    # Remove timestamps
    s/^[0-9]\{4\}-[0-9]\{2\}-[0-9]\{2\} [0-9]\{2\}:[0-9]\{2\}:[0-9]\{2\} //
    # Extract error messages
    /ERROR/!d
    # Format output
    s/ERROR: \(.*\)/[\1]/
' logfile.txt
```

### Configuration File Update
```bash
#!/bin/bash
# Update configuration values
sed -i.bak '
    # Update specific settings
    s/^SETTING1=.*/SETTING1=newvalue/
    s/^SETTING2=.*/SETTING2=newvalue/
    # Remove commented lines
    /^#/d
    # Add missing settings
    $a\SETTING3=value3
' config.txt
```

### HTML Processing
```bash
#!/bin/bash
# Process HTML files
sed '
    # Remove HTML comments
    s/<!--.*-->//g
    # Remove empty lines
    /^$/d
    # Format tags
    s/<[^>]*>/\n&\n/g
    # Remove extra spaces
    s/^[ \t]*//
    s/[ \t]*$//
' webpage.html
```

---

Remember:
- Test commands on sample data first
- Make backups before in-place editing
- Use appropriate regex delimiters
- Consider performance for large files
- Document complex transformations
- Handle special characters properly

For detailed information, consult the sed manual (`man sed`).