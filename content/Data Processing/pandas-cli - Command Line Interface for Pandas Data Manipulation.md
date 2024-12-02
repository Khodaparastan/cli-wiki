
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Data Operations](#data-operations)
- [Data Analysis](#data-analysis)
- [Data Export](#data-export)
- [Advanced Features](#advanced-features)
- [Best Practices](#best-practices)

## Overview

pandas-cli provides command-line interface for common pandas operations, allowing quick data manipulation without writing Python scripts.

### Key Features
- Data reading/writing
- Data transformation
- Basic statistics
- Data filtering
- Column operations
- Data aggregation
- Format conversion
- Quick analysis

## Installation

```bash
# Using pip
pip install pandas-cli

# With additional features
pip install "pandas-cli[full]"

# Development version
pip install git+https://github.com/pandas-dev/pandas-cli.git
```

## Basic Usage

### Reading Data
```bash
# Read CSV
pdcli read data.csv

# Read Excel
pdcli read data.xlsx --sheet "Sheet1"

# Read JSON
pdcli read data.json

# Read with options
pdcli read data.csv --sep ";" --encoding "utf-8"
```

### Basic Information
```bash
# Show data info
pdcli info data.csv

# Show columns
pdcli columns data.csv

# Show data types
pdcli dtypes data.csv

# Quick summary
pdcli describe data.csv
```

## Data Operations

### Column Operations
```bash
# Select columns
pdcli select data.csv "col1,col2"

# Rename columns
pdcli rename data.csv "old_name:new_name"

# Drop columns
pdcli drop data.csv "col1,col2"

# Reorder columns
pdcli reorder data.csv "col2,col1,col3"
```

### Filtering
```bash
# Filter rows
pdcli filter data.csv "column > 100"

# Complex filters
pdcli filter data.csv "column1 > 100 and column2 == 'value'"

# Regex filter
pdcli filter data.csv "column.str.contains('pattern')"

# Top N rows
pdcli head data.csv --n 10
```

### Transformations
```bash
# Apply function
pdcli apply data.csv "column" "lambda x: x * 2"

# Replace values
pdcli replace data.csv "column" "old_value" "new_value"

# Fill missing values
pdcli fillna data.csv "column" "value"

# Drop duplicates
pdcli dedupe data.csv "column1,column2"
```

## Data Analysis

### Statistics
```bash
# Basic stats
pdcli stats data.csv

# Group by statistics
pdcli groupby data.csv "column" "mean,sum,count"

# Correlation
pdcli corr data.csv

# Value counts
pdcli value_counts data.csv "column"
```

### Aggregations
```bash
# Simple aggregation
pdcli agg data.csv "column" "sum"

# Multiple aggregations
pdcli agg data.csv "column1:sum,column2:mean"

# Group by aggregation
pdcli groupby data.csv "group_col" "value_col:sum"
```

### Time Series
```bash
# Resample time series
pdcli resample data.csv "date_column" "1D" "mean"

# Rolling calculations
pdcli rolling data.csv "column" 7 "mean"

# Date range selection
pdcli daterange data.csv "date_column" "2023-01-01" "2023-12-31"
```

## Data Export

### Save Operations
```bash
# Save to CSV
pdcli to_csv data.csv output.csv

# Save to Excel
pdcli to_excel data.csv output.xlsx

# Save to JSON
pdcli to_json data.csv output.json

# Save with options
pdcli to_csv data.csv output.csv --index False --sep ";"
```

### Format Conversion
```bash
# CSV to Excel
pdcli convert data.csv output.xlsx

# Excel to JSON
pdcli convert data.xlsx output.json

# JSON to CSV
pdcli convert data.json output.csv
```

## Advanced Features

### Data Merging
```bash
# Merge files
pdcli merge file1.csv file2.csv --on "id"

# Concatenate files
pdcli concat file1.csv file2.csv

# Join operations
pdcli join left.csv right.csv --how "left" --on "id"
```

### Data Reshaping
```bash
# Pivot table
pdcli pivot data.csv "index" "columns" "values"

# Melt operation
pdcli melt data.csv "id_vars" "value_vars"

# Transpose
pdcli transpose data.csv
```

## Best Practices

### Performance Tips
```bash
# Chunk processing
pdcli read large.csv --chunksize 10000

# Memory efficient
pdcli read data.csv --usecols "col1,col2"

# Optimize types
pdcli optimize data.csv
```

### Error Handling
```bash
# Validate data
pdcli validate data.csv

# Check missing values
pdcli missing data.csv

# Debug mode
pdcli --debug command data.csv
```

## Example Scripts

### Data Cleaning Pipeline
```bash
#!/bin/bash
# Clean and transform data
INPUT="raw_data.csv"
OUTPUT="clean_data.csv"

# Cleaning pipeline
pdcli read "$INPUT" \
  | pdcli drop "unnecessary_column" \
  | pdcli fillna "all" \
  | pdcli dedupe "id" \
  | pdcli to_csv "$OUTPUT"
```

### Data Analysis Report
```bash
#!/bin/bash
# Generate analysis report
DATA="sales_data.csv"
REPORT="report"

# Generate statistics
pdcli describe "$DATA" > "${REPORT}_summary.txt"
pdcli corr "$DATA" > "${REPORT}_correlation.txt"
pdcli groupby "$DATA" "category" "amount:sum,quantity:mean" > "${REPORT}_by_category.txt"
```

### Data Transformation
```bash
#!/bin/bash
# Transform multiple files
for file in data_*.csv; do
    output="transformed_${file}"
    pdcli read "$file" \
        | pdcli filter "amount > 0" \
        | pdcli apply "amount" "lambda x: round(x, 2)" \
        | pdcli to_csv "$output"
done
```

### Automated Report
```bash
#!/bin/bash
# Generate daily report
TODAY=$(date +%Y-%m-%d)
DATA="daily_data.csv"
REPORT="report_${TODAY}.xlsx"

# Process data and create report
pdcli read "$DATA" \
    | pdcli filter "date == '$TODAY'" \
    | pdcli groupby "department" "sales:sum,customers:count" \
    | pdcli sort "sales" --ascending False \
    | pdcli to_excel "$REPORT" --sheet "Daily Summary"
```

---

Remember:
- Use appropriate data types
- Handle large files efficiently
- Document transformations
- Validate data quality
- Use version control
- Keep pipelines maintainable

For detailed information, consult the pandas-cli documentation and help commands (`pdcli --help`).

Note: This is a conceptual guide for a hypothetical pandas-cli tool. While there are various CLI tools for pandas, there isn't a single standardized pandas-cli. You might want to look at alternatives like `pandas-ply`, `pandasql`, or create custom scripts using pandas in Python.