
## Table of Contents
- [Overview](#overview)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Data Operations](#data-operations)
- [Query Operations](#query-operations)
- [File Operations](#file-operations)
- [Advanced Features](#advanced-features)
- [Best Practices](#best-practices)

## Overview

DuckDB is an embedded analytical database with a powerful CLI, designed for fast analytics on structured data, particularly effective for data science and analytics workflows.

### Key Features
- OLAP database operations
- SQL interface
- Direct file querying
- Parallel processing
- Vector operations
- JSON/CSV/Parquet support
- Python integration
- In-memory processing

## Installation

```bash
# Ubuntu/Debian
sudo apt-get install duckdb

# macOS
brew install duckdb

# Using pip (Python interface)
pip install duckdb

# CLI Access
duckdb
```

## Basic Usage

### Starting DuckDB
```bash
# Start interactive shell
duckdb

# Start with database file
duckdb database.db

# Execute single query
duckdb -c "SELECT * FROM table"

# Execute script
duckdb < script.sql
```

### Basic Commands
```sql
-- Show tables
.tables

-- Show schema
.schema table_name

-- Show system tables
.system

-- Exit
.exit
```

## Data Operations

### Table Creation
```sql
-- Create table
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    name VARCHAR,
    age INTEGER
);

-- Create from CSV
CREATE TABLE users AS 
SELECT * FROM read_csv_auto('users.csv');

-- Create from Parquet
CREATE TABLE data AS 
SELECT * FROM read_parquet('data.parquet');
```

### Data Import
```sql
-- Import CSV
COPY users FROM 'users.csv' (DELIMITER ',', HEADER);

-- Import JSON
SELECT * FROM read_json_auto('data.json');

-- Import Excel
SELECT * FROM read_excel('data.xlsx');

-- Import multiple files
SELECT * FROM read_csv_auto('*.csv');
```

## Query Operations

### Basic Queries
```sql
-- Select data
SELECT * FROM users WHERE age > 25;

-- Aggregations
SELECT 
    department,
    COUNT(*) as count,
    AVG(salary) as avg_salary
FROM employees
GROUP BY department;

-- Window functions
SELECT *,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC)
FROM employees;
```

### Advanced Analytics
```sql
-- Time series analysis
SELECT 
    date_trunc('month', date) as month,
    SUM(amount) as total
FROM transactions
GROUP BY 1
ORDER BY 1;

-- Moving averages
SELECT *,
    AVG(value) OVER (
        ORDER BY date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as moving_avg
FROM metrics;

-- Correlation analysis
SELECT 
    corr(price, quantity) as price_quantity_correlation
FROM sales;
```

## File Operations

### Direct File Querying
```sql
-- Query CSV directly
SELECT * FROM read_csv_auto('data.csv')
WHERE column > 100;

-- Query Parquet
SELECT * FROM read_parquet('data.parquet')
WHERE date >= '2023-01-01';

-- Query multiple files
SELECT * FROM read_csv_auto('data_*.csv')
WHERE category = 'A';
```

### Export Operations
```sql
-- Export to CSV
COPY (SELECT * FROM users) TO 'users.csv' WITH (HEADER 1);

-- Export to Parquet
COPY (SELECT * FROM users) TO 'users.parquet' (FORMAT PARQUET);

-- Export query results
COPY (
    SELECT department, AVG(salary) 
    FROM employees 
    GROUP BY department
) TO 'dept_salaries.csv' WITH (HEADER 1);
```

## Advanced Features

### Parallel Processing
```sql
-- Set threads
SET threads TO 4;

-- Parallel query
SELECT * FROM read_csv_auto('large_file.csv')
WHERE value > 1000
PARALLEL 4;
```

### Window Functions
```sql
-- Ranking
SELECT *,
    RANK() OVER (PARTITION BY category ORDER BY value DESC) as rank
FROM data;

-- Running totals
SELECT *,
    SUM(amount) OVER (ORDER BY date) as running_total
FROM transactions;
```

### JSON Operations
```sql
-- Parse JSON
SELECT json_extract(data, '$.key') as value
FROM json_table;

-- Create JSON
SELECT json_object('name', name, 'age', age)
FROM users;
```

## Best Practices

### Performance Optimization
```sql
-- Create indexes
CREATE INDEX idx_user_id ON users(id);

-- Use PRAGMA statements
PRAGMA threads=4;
PRAGMA memory_limit='4GB';

-- Optimize joins
SELECT /*+ JOIN_ORDER(users, orders) */ *
FROM users 
JOIN orders ON users.id = orders.user_id;
```

### Memory Management
```sql
-- Set memory limit
SET memory_limit='8GB';

-- Monitor memory usage
SELECT * FROM pragma_database_size();

-- Clear memory
PRAGMA memory_clear();
```

## Example Scripts

### Data Processing Pipeline
```sql
-- data_pipeline.sql
-- Process sales data

-- Create staging table
CREATE TABLE staging AS
SELECT * FROM read_csv_auto('raw_sales.csv');

-- Clean and transform
CREATE TABLE cleaned_sales AS
SELECT 
    date,
    COALESCE(product_id, 'UNKNOWN') as product_id,
    ROUND(amount, 2) as amount,
    CASE 
        WHEN quantity < 0 THEN 0 
        ELSE quantity 
    END as quantity
FROM staging
WHERE date IS NOT NULL;

-- Create summary
CREATE TABLE sales_summary AS
SELECT 
    date_trunc('month', date) as month,
    product_id,
    SUM(amount) as total_amount,
    SUM(quantity) as total_quantity,
    COUNT(*) as transaction_count
FROM cleaned_sales
GROUP BY 1, 2
ORDER BY 1, 2;

-- Export results
COPY (
    SELECT * FROM sales_summary
) TO 'sales_summary.parquet' (FORMAT PARQUET);
```

### Analytics Report Generator
```bash
#!/bin/bash
# Generate analytics report

DB="analytics.db"
REPORT_DIR="reports"
DATE=$(date +%Y%m%d)

mkdir -p "$REPORT_DIR"

# Execute analytics queries
duckdb "$DB" << EOF
-- Daily metrics
COPY (
    SELECT 
        date,
        COUNT(*) as transactions,
        SUM(amount) as total_amount,
        AVG(amount) as avg_amount
    FROM transactions
    WHERE date >= CURRENT_DATE - INTERVAL '30 days'
    GROUP BY date
    ORDER BY date
) TO '${REPORT_DIR}/daily_metrics_${DATE}.csv' WITH (HEADER 1);

-- Product performance
COPY (
    SELECT 
        p.category,
        COUNT(*) as sales_count,
        SUM(t.amount) as total_revenue,
        AVG(t.amount) as avg_transaction
    FROM transactions t
    JOIN products p ON t.product_id = p.id
    GROUP BY p.category
    ORDER BY total_revenue DESC
) TO '${REPORT_DIR}/product_performance_${DATE}.csv' WITH (HEADER 1);
EOF
```

### Data Quality Check
```sql
-- data_quality.sql
-- Perform data quality checks

-- Check for nulls
SELECT 
    column_name,
    COUNT(*) as null_count
FROM (
    SELECT * FROM information_schema.columns
    WHERE table_name = 'target_table'
) as cols
CROSS JOIN (
    SELECT * FROM target_table
) as data
WHERE data[cols.column_name] IS NULL
GROUP BY column_name
HAVING null_count > 0;

-- Check for duplicates
SELECT 
    id,
    COUNT(*) as duplicate_count
FROM target_table
GROUP BY id
HAVING COUNT(*) > 1;

-- Value range checks
SELECT 
    MIN(value) as min_value,
    MAX(value) as max_value,
    AVG(value) as avg_value,
    STDDEV(value) as stddev_value
FROM numeric_table
WHERE value IS NOT NULL;
```

---

Remember:
- Use appropriate data types
- Create indexes for large tables
- Leverage parallel processing
- Monitor memory usage
- Regular backups
- Document queries and transformations

For detailed information, consult the DuckDB documentation (https://duckdb.org/docs/).