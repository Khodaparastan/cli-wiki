
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Data Operations](#data-operations)
- [Analysis Commands](#analysis-commands)
- [Performance Features](#performance-features)
- [Advanced Usage](#advanced-usage)
- [Best Practices](#best-practices)

## Overview

xsv is a fast CSV command line toolkit written in Rust, designed to handle large CSV files efficiently with a focus on performance.

### Key Features
- Fast CSV processing
- Memory efficient
- Index support
- Parallel processing
- Statistical operations
- Search capabilities
- Join operations
- Flexible input/output

## Installation

```bash
# Using cargo (Rust package manager)
cargo install xsv

# Ubuntu/Debian
curl -LO https://github.com/BurntSushi/xsv/releases/download/0.13.0/xsv-0.13.0-x86_64-unknown-linux-musl.tar.gz
tar xf xsv-0.13.0-x86_64-unknown-linux-musl.tar.gz

# macOS
brew install xsv
```

## Basic Usage

### File Information
```bash
# Show basic info
xsv info data.csv

# Count records
xsv count data.csv

# Show headers
xsv headers data.csv

# Sample data
xsv sample 10 data.csv
```

### Basic Operations
```bash
# Select columns
xsv select name,age data.csv

# Search for values
xsv search "pattern" data.csv

# Sort data
xsv sort -s column data.csv

# Remove duplicates
xsv dedupe data.csv
```

## Data Operations

### Column Operations
```bash
# Select specific columns
xsv select "1,3,5" data.csv

# Select by name
xsv select "name,age,city" data.csv

# Reorder columns
xsv select "age,name,city" data.csv

# Index columns
xsv index data.csv
```

### Filtering and Searching
```bash
# Search in specific column
xsv search -s column "pattern" data.csv

# Case insensitive search
xsv search -i "pattern" data.csv

# Regex search
xsv search -r "^pattern$" data.csv

# Exclude matches
xsv search -v "pattern" data.csv
```

### Transformations
```bash
# Slice rows
xsv slice -s 10 -e 20 data.csv

# Take first N rows
xsv slice -l 100 data.csv

# Random sample
xsv sample 1000 data.csv

# Replace headers
xsv headers -r new_headers.txt data.csv
```

## Analysis Commands

### Statistical Operations
```bash
# Basic statistics
xsv stats data.csv

# Stats on specific columns
xsv stats -s column1,column2 data.csv

# Frequency counts
xsv frequency -s column data.csv

# Unique values
xsv distinct -s column data.csv
```

### Join Operations
```bash
# Inner join
xsv join --left file1.csv --right file2.csv

# Left join
xsv join --left-join file1.csv file2.csv

# Join on multiple columns
xsv join -d "," col1,col2 file1.csv file2.csv
```

### Grouping
```bash
# Group by column
xsv group -s column data.csv

# Count by group
xsv frequency -s column data.csv

# Multiple group columns
xsv group -s "col1,col2" data.csv
```

## Performance Features

### Indexing
```bash
# Create index
xsv index data.csv

# Use index
xsv slice -i data.csv

# Remove index
rm data.csv.idx
```

### Parallel Processing
```bash
# Parallel search
xsv search -j 4 "pattern" data.csv

# Parallel stats
xsv stats -j 4 data.csv

# Parallel join
xsv join -j 4 file1.csv file2.csv
```

## Advanced Usage

### Complex Operations
```bash
# Multi-step processing
xsv select name,age data.csv \
    | xsv sort -s age \
    | xsv slice -l 10

# Conditional selection
xsv search "pattern" data.csv \
    | xsv select col1,col2 \
    | xsv stats

# Custom delimiter
xsv input -d "|" data.txt
```

### Data Validation
```bash
# Check for empty fields
xsv search -s column "^$" data.csv

# Validate date format
xsv search -s date "^\d{4}-\d{2}-\d{2}$" data.csv

# Count null values
xsv search -s column "^$|^NULL$" -c data.csv
```

## Best Practices

### Performance Tips
```bash
# Use indexing for large files
xsv index large.csv
xsv slice -i large.csv

# Efficient column selection
xsv select necessary_columns data.csv

# Parallel processing
xsv command -j 4 data.csv
```

### Error Handling
```bash
# Validate CSV format
xsv validate data.csv

# Check header consistency
xsv headers data.csv > headers.txt
diff headers.txt expected_headers.txt
```

## Example Scripts

### Data Processing Pipeline
```bash
#!/bin/bash
# Process sales data
INPUT="sales.csv"
OUTPUT="processed_sales.csv"

# Processing pipeline
xsv select order_id,customer,amount "$INPUT" \
    | xsv search -v "^$" \
    | xsv sort -s amount \
    | xsv dedupe > "$OUTPUT"
```

### Data Analysis Report
```bash
#!/bin/bash
# Generate analysis report
DATA="transactions.csv"
REPORT_DIR="reports"

mkdir -p "$REPORT_DIR"

# Generate statistics
xsv stats "$DATA" > "$REPORT_DIR/statistics.txt"
xsv frequency -s category "$DATA" > "$REPORT_DIR/category_freq.txt"
xsv select amount "$DATA" | xsv stats > "$REPORT_DIR/amount_stats.txt"
```

### Data Validation Script
```bash
#!/bin/bash
# Validate CSV files
validate_csv() {
    local file=$1
    echo "Validating $file..."
    
    # Check format
    if ! xsv validate "$file"; then
        echo "Invalid CSV format in $file"
        return 1
    fi
    
    # Check required columns
    if ! xsv headers "$file" | grep -q "required_column"; then
        echo "Missing required column in $file"
        return 1
    fi
    
    # Check for empty values
    empty_count=$(xsv search "^$" -c "$file")
    if [ "$empty_count" -gt 0 ]; then
        echo "Found $empty_count empty values in $file"
    fi
    
    return 0
}

# Process all CSV files
for file in *.csv; do
    validate_csv "$file"
done
```

### Data Merger
```bash
#!/bin/bash
# Merge multiple CSV files
OUTPUT="merged.csv"
TEMP_DIR="temp"

mkdir -p "$TEMP_DIR"

# Process each file
for file in data_*.csv; do
    echo "Processing $file..."
    xsv select required_columns "$file" \
        | xsv sort -s id \
        > "$TEMP_DIR/$(basename "$file")"
done

# Merge all processed files
xsv cat rows "$TEMP_DIR"/*.csv > "$OUTPUT"

# Cleanup
rm -r "$TEMP_DIR"
```

---

Remember:
- Use indexes for large files
- Leverage parallel processing
- Select only needed columns
- Validate data quality
- Handle errors appropriately
- Document processing steps

For detailed information, consult the xsv documentation (`xsv --help`) and the [GitHub repository](https://github.com/BurntSushi/xsv).