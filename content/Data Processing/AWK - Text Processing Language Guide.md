
## Table of Contents
- [Overview](#overview)
- [Basic Syntax](#basic-syntax)
- [Pattern Matching](#pattern-matching)
- [Field Processing](#field-processing)
- [Built-in Variables](#built-in-variables)
- [Control Structures](#control-structures)
- [Functions](#functions)
- [Advanced Features](#advanced-features)
- [Best Practices](#best-practices)

## Overview

AWK is a powerful text processing language designed for pattern scanning and processing. It's particularly useful for structured text data like logs, CSV files, and formatted output.

### Key Features
- Pattern scanning
- Field processing
- Arithmetic operations
- String manipulation
- Regular expressions
- Custom functions
- Control structures
- Built-in variables

## Basic Syntax

### Basic Structure
```awk
# Basic pattern-action
pattern { action }

# Print all lines
awk '{ print }' file.txt

# Print specific fields
awk '{ print $1, $3 }' file.txt

# Using field separator
awk -F',' '{ print $1 }' file.csv
```

### Common Options
```bash
# Set field separator
awk -F':' '{ print $1 }' /etc/passwd

# Set variable
awk -v name=value '{ print name }' file.txt

# Run AWK program file
awk -f program.awk input.txt
```

## Pattern Matching

### Basic Patterns
```awk
# Match exact string
/pattern/ { print }

# Match at beginning
/^pattern/ { print }

# Match at end
/pattern$/ { print }

# Multiple patterns
/pattern1/ && /pattern2/ { print }
```

### Regular Expressions
```awk
# Match numbers
/[0-9]+/ { print }

# Match email addresses
/[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}/ { print }

# Case insensitive match
tolower($0) ~ /pattern/ { print }
```

## Field Processing

### Field Operations
```awk
# Print specific fields
{ print $1, $3 }

# Calculate field sum
{ sum += $1 } END { print sum }

# Field count
{ print NF }

# Process specific fields
$1 ~ /pattern/ { print $2 }
```

### Field Manipulation
```awk
# Change field separator
BEGIN { FS=","; OFS="|" }

# Modify field content
{ $2 = toupper($2); print }

# Add new field
{ $NF = $NF "|" "new_value"; print }
```

## Built-in Variables

### Common Variables
```awk
# Field and record variables
NF      # Number of fields
NR      # Current record number
FNR     # Record number in current file
$0      # Entire record
$1      # First field

# Input/Output variables
FS      # Field separator (input)
OFS     # Field separator (output)
RS      # Record separator (input)
ORS     # Record separator (output)
```

### Special Variables
```awk
# File processing
FILENAME # Current filename
ARGC     # Number of arguments
ARGV     # Array of arguments

# Numeric formats
CONVFMT  # Conversion format
OFMT     # Output format
```

## Control Structures

### Conditional Statements
```awk
# If statement
if ($1 > 100) {
    print "Large value"
}

# If-else
if ($1 > 100) {
    print "Large"
} else {
    print "Small"
}

# Multiple conditions
if ($1 < 50) {
    print "Small"
} else if ($1 < 100) {
    print "Medium"
} else {
    print "Large"
}
```

### Loops
```awk
# For loop
for (i=1; i<=NF; i++) {
    print $i
}

# While loop
while (getline > 0) {
    print $0
}

# Do-while loop
do {
    print $0
} while (getline > 0)
```

## Functions

### String Functions
```awk
# Length
length($1)

# Substring
substr($1, 1, 5)

# Case conversion
toupper($1)
tolower($1)

# String replacement
gsub(/pattern/, "replacement", $1)
```

### Numeric Functions
```awk
# Mathematical functions
int(3.14)
sqrt(100)
sin(0)
rand()

# Formatting
sprintf("%.2f", $1)
```

## Advanced Features

### Arrays
```awk
# Associative arrays
{ count[$1]++ }
END { for (key in count) print key, count[key] }

# Array sorting
{ a[NR] = $0 }
END {
    n = asort(a)
    for (i=1; i<=n; i++) print a[i]
}
```

### User-Defined Functions
```awk
# Function definition
function square(x) {
    return x * x
}

# Function usage
{ print square($1) }
```

## Best Practices

### Performance Tips
```awk
# Pre-compile regex
BEGIN { pattern = "^[0-9]+$" }
$1 ~ pattern { print }

# Minimize field references
{ temp = $1; ... }

# Use appropriate data structures
# Arrays for counting
{ count[$1]++ }
```

### Error Handling
```awk
# Check field existence
$1 != "" { print $1 }

# Validate numeric input
$1 ~ /^[0-9]+$/ { print }

# Handle missing files
BEGINFILE { if (ERRNO) { print "Error:", ERRNO > "/dev/stderr"; nextfile } }
```

## Example Scripts

### Log Analysis
```awk
#!/usr/bin/awk -f
# Analyze Apache access log
BEGIN {
    FS = "\"" 
    print "Status Code Analysis"
    print "==================="
}

{
    split($3, status, " ")
    codes[status[1]]++
}

END {
    for (code in codes)
        printf "%s: %d\n", code, codes[code]
}
```

### CSV Processing
```awk
#!/usr/bin/awk -f
# Process CSV data
BEGIN {
    FS = ","
    OFS = "|"
    print "ID", "Name", "Total"
}

NR > 1 {  # Skip header
    sum = 0
    for (i=3; i<=NF; i++)
        sum += $i
    print $1, $2, sum
}
```

### Data Transformation
```awk
#!/usr/bin/awk -f
# Transform data format
BEGIN {
    FS = "\t"
    OFS = ","
}

function clean(str) {
    gsub(/^\s+|\s+$/, "", str)
    return str
}

{
    for (i=1; i<=NF; i++)
        $i = clean($i)
    if (NF > 0)
        print
}
```

---

Remember:
- Use appropriate field separators
- Consider performance for large files
- Handle edge cases
- Document complex patterns
- Test with sample data
- Use functions for reusable code

For detailed information, consult the AWK manual (`man awk`).