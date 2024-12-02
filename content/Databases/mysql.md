

# MySQL Command Line Client Guide - Part 1: Basic Usage and Connection

## Table of Contents
- [Overview](#overview)
- [Connection & Authentication](#connection--authentication)
- [Basic Commands](#basic-commands)
- [Database Operations](#database-operations)
- [Query Operations](#query-operations)
- [Output Formatting](#output-formatting)
- [Import/Export](#importexport)
- [Security & Administration](#security--administration)
- [Advanced Features](#advanced-features)
- [Best Practices](#best-practices)

## Overview

The MySQL Command-Line Client provides direct interaction with MySQL databases through a terminal interface.

### Key Features
- Interactive SQL execution
- Script execution
- Batch mode operations
- Output formatting
- Command history
- Tab completion
- Variables support
- Secure connections

## Connection & Authentication

### Basic Connection
```bash
# Standard connection
mysql -u username -p database_name

# Specific host/port
mysql -h hostname -P 3306 -u username -p database_name

# Using defaults file
mysql --defaults-file=/path/to/my.cnf

# Socket connection
mysql -S /path/to/mysql.sock -u username -p
```

### Connection Options
```bash
# Environment variables
export MYSQL_HOST=hostname
export MYSQL_TCP_PORT=3306
export MYSQL_PWD=password  # Not recommended for security

# Configuration file (~/.my.cnf)
[client]
host=hostname
port=3306
user=username
password=password
```

### SSL Connections
```bash
# Connect with SSL
mysql --ssl-ca=/path/to/ca.pem \
      --ssl-cert=/path/to/client-cert.pem \
      --ssl-key=/path/to/client-key.pem \
      -u username -p database_name

# Verify SSL
mysql> SHOW SESSION STATUS LIKE 'Ssl%';
```

## Basic Commands

### System Commands
```sql
-- Show databases
SHOW DATABASES;

-- Use database
USE database_name;

-- Show tables
SHOW TABLES;

-- Show table structure
DESCRIBE table_name;
SHOW CREATE TABLE table_name;

-- Show server status
SHOW STATUS;
SHOW VARIABLES;
```

### Help Commands
```sql
-- General help
help;
\h;

-- Command specific help
help select;

-- Show warnings
SHOW WARNINGS;

-- Show errors
SHOW ERRORS;
```

### Client Commands
```sql
-- Clear screen
\c

-- Exit client
\q
exit;
quit;

-- Edit command in editor
\e

-- Use command history
\p
```

## Database Operations

### Database Management
```sql
-- Create database
CREATE DATABASE database_name
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;

-- Show database info
SELECT DEFAULT_CHARACTER_SET_NAME, DEFAULT_COLLATION_NAME 
FROM INFORMATION_SCHEMA.SCHEMATA 
WHERE SCHEMA_NAME = 'database_name';

-- Modify database
ALTER DATABASE database_name
CHARACTER SET = utf8mb4
COLLATE = utf8mb4_unicode_ci;

-- Drop database
DROP DATABASE database_name;
```

### Table Operations
```sql
-- Create table
CREATE TABLE table_name (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;

-- Modify table
ALTER TABLE table_name
ADD COLUMN new_column INT,
MODIFY COLUMN name VARCHAR(200),
DROP COLUMN old_column;

-- Table maintenance
CHECK TABLE table_name;
REPAIR TABLE table_name;
OPTIMIZE TABLE table_name;
ANALYZE TABLE table_name;
```

## Query Operations

### Basic Queries
```sql
-- Select with formatting
SELECT * FROM table_name\G

-- Select with conditions
SELECT *
FROM table_name
WHERE condition
LIMIT 10\G

-- Insert data
INSERT INTO table_name
SET column1 = value1,
    column2 = value2;

-- Update data
UPDATE table_name
SET column1 = value1
WHERE condition;

-- Delete data
DELETE FROM table_name
WHERE condition
LIMIT 1;
```

### Transaction Control
```sql
-- Start transaction
START TRANSACTION;
BEGIN;

-- Commit changes
COMMIT;

-- Rollback changes
ROLLBACK;

-- Set savepoint
SAVEPOINT savepoint_name;
ROLLBACK TO SAVEPOINT savepoint_name;
```

## Output Formatting

### Display Options
```sql
-- Vertical output
SELECT * FROM table_name\G

-- Table format
SELECT * FROM table_name;

-- Raw output
mysql -N -B -e "SELECT * FROM table_name"

-- XML output
mysql --xml -e "SELECT * FROM table_name"

-- HTML output
mysql --html -e "SELECT * FROM table_name"
```

### Format Control
```sql
-- Set column width
mysql --auto-vertical-output

-- Silent mode
mysql --silent

-- Force protocol
mysql --protocol=TCP

-- Compress output
mysql --compress
```

[Continue to Part 2...]

# MySQL Command Line Client Guide - Part 2: Advanced Features

## Import/Export Operations

### Data Export
```bash
# Export database
mysqldump -u username -p database_name > backup.sql

# Export specific tables
mysqldump -u username -p database_name table1 table2 > tables_backup.sql

# Export with options
mysqldump --opt --events --routines --triggers \
          -u username -p database_name > full_backup.sql

# Export as CSV
mysql -u username -p -B -N \
      -e "SELECT * FROM table_name" database_name > export.csv
```

### Data Import
```bash
# Import database
mysql -u username -p database_name < backup.sql

# Import with progress
pv backup.sql | mysql -u username -p database_name

# Import CSV
LOAD DATA INFILE 'data.csv'
INTO TABLE table_name
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

## Security & Administration

### User Management
```sql
-- Create user
CREATE USER 'username'@'hostname' 
IDENTIFIED BY 'password';

-- Grant privileges
GRANT ALL PRIVILEGES 
ON database_name.* 
TO 'username'@'hostname';

-- Show grants
SHOW GRANTS FOR 'username'@'hostname';

-- Revoke privileges
REVOKE ALL PRIVILEGES 
ON database_name.* 
FROM 'username'@'hostname';
```

### Security Audit
```sql
-- Check user privileges
SELECT * FROM mysql.user\G

-- Show process list
SHOW PROCESSLIST;

-- Kill process
KILL process_id;

-- Show binary logs
SHOW BINARY LOGS;
```

## Advanced Features

### Variables and Status
```sql
-- Show global variables
SHOW GLOBAL VARIABLES;

-- Set global variable
SET GLOBAL variable_name = value;

-- Show session variables
SHOW SESSION VARIABLES;

-- Show status
SHOW STATUS WHERE Variable_name LIKE 'Threads%';
```

### Performance Schema
```sql
-- Enable performance schema
UPDATE performance_schema.setup_instruments 
SET ENABLED = 'YES', TIMED = 'YES';

-- Check top queries
SELECT DIGEST_TEXT, COUNT_STAR, AVG_TIMER_WAIT
FROM performance_schema.events_statements_summary_by_digest
ORDER BY AVG_TIMER_WAIT DESC
LIMIT 10;

-- Check table I/O
SELECT * FROM performance_schema.table_io_waits_summary_by_table
WHERE COUNT_STAR > 0
ORDER BY SUM_TIMER_WAIT DESC;
```

## Example Scripts

### Database Backup Script
```bash
#!/bin/bash
# Comprehensive backup script

BACKUP_DIR="/backup/mysql"
DATE=$(date +%Y%m%d_%H%M%S)
MYSQL_USER="backup_user"
MYSQL_PASS="backup_password"
RETENTION_DAYS=7

# Create backup directory
mkdir -p "$BACKUP_DIR"

# Function for logging
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> "$BACKUP_DIR/backup.log"
}

# Backup all databases
backup_all_databases() {
    log "Starting full backup"
    mysqldump --user="$MYSQL_USER" \
              --password="$MYSQL_PASS" \
              --all-databases \
              --events \
              --routines \
              --triggers \
              --single-transaction \
              --quick \
              --lock-tables=false \
              > "$BACKUP_DIR/full_backup_$DATE.sql"

    if [ $? -eq 0 ]; then
        log "Full backup completed successfully"
        # Compress backup
        gzip "$BACKUP_DIR/full_backup_$DATE.sql"
        log "Backup compressed"
    else
        log "Backup failed!"
        exit 1
    fi
}

# Clean old backups
clean_old_backups() {
    log "Cleaning old backups"
    find "$BACKUP_DIR" -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete
    log "Old backups cleaned"
}

# Execute backup
backup_all_databases
clean_old_backups
```

### Performance Monitoring Script
```bash
#!/bin/bash
# MySQL performance monitoring script

MYSQL_USER="monitor_user"
MYSQL_PASS="monitor_password"
LOG_FILE="mysql_performance.log"

# Function to execute MySQL queries
mysql_query() {
    mysql -u"$MYSQL_USER" -p"$MYSQL_PASS" -N -B -e "$1"
}

# Monitor key metrics
monitor_performance() {
    echo "=== Performance Report $(date) ===" >> "$LOG_FILE"
    
    # Check threads
    echo "Thread Status:" >> "$LOG_FILE"
    mysql_query "SHOW STATUS WHERE Variable_name LIKE 'Threads%'" >> "$LOG_FILE"
    
    # Check connections
    echo "Connection Status:" >> "$LOG_FILE"
    mysql_query "SHOW STATUS WHERE Variable_name LIKE 'Conn%'" >> "$LOG_FILE"
    
    # Check buffer pool
    echo "Buffer Pool Status:" >> "$LOG_FILE"
    mysql_query "SHOW STATUS WHERE Variable_name LIKE 'Innodb_buffer_pool%'" >> "$LOG_FILE"
    
    # Check slow queries
    echo "Slow Query Status:" >> "$LOG_FILE"
    mysql_query "SHOW GLOBAL STATUS WHERE Variable_name LIKE 'Slow_queries'" >> "$LOG_FILE"
    
    echo "===========================" >> "$LOG_FILE"
}

# Execute monitoring
while true; do
    monitor_performance
    sleep 300  # Run every 5 minutes
done
```

### Table Maintenance Script
```bash
#!/bin/bash
# Table maintenance script

MYSQL_USER="maintenance_user"
MYSQL_PASS="maintenance_password"
DATABASE="target_database"
LOG_FILE="maintenance.log"

# Function for logging
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> "$LOG_FILE"
}

# Maintenance function
maintain_tables() {
    tables=$(mysql -u"$MYSQL_USER" -p"$MYSQL_PASS" -N -B \
             -e "SHOW TABLES FROM $DATABASE")
    
    for table in $tables; do
        log "Maintaining table: $table"
        
        # Analyze table
        mysql -u"$MYSQL_USER" -p"$MYSQL_PASS" "$DATABASE" \
              -e "ANALYZE TABLE $table" >> "$LOG_FILE"
        
        # Optimize table
        mysql -u"$MYSQL_USER" -p"$MYSQL_PASS" "$DATABASE" \
              -e "OPTIMIZE TABLE $table" >> "$LOG_FILE"
        
        # Check table
        mysql -u"$MYSQL_USER" -p"$MYSQL_PASS" "$DATABASE" \
              -e "CHECK TABLE $table" >> "$LOG_FILE"
    done
}

# Execute maintenance
maintain_tables
```

[Continue to Part 3...]

# MySQL Command Line Client Guide - Part 3: Best Practices and Tips

## Best Practices

### Performance Optimization
```sql
-- Use indexes effectively
EXPLAIN SELECT * FROM table_name WHERE indexed_column = 'value';

-- Monitor query cache
SHOW STATUS LIKE 'Qcache%';

-- Check slow queries
SHOW VARIABLES LIKE 'slow_query%';
SHOW VARIABLES LIKE 'long_query_time';
```

### Security Best Practices
```sql
-- Use SSL for connections
SHOW VARIABLES LIKE '%ssl%';

-- Regular password updates
ALTER USER 'username'@'hostname' 
IDENTIFIED BY 'new_password';

-- Audit user privileges
SELECT * FROM mysql.user WHERE user NOT IN ('mysql.sys', 'root')\G
```

### Maintenance Guidelines
```sql
-- Regular table maintenance
CHECK TABLE table_name;
ANALYZE TABLE table_name;
OPTIMIZE TABLE table_name;

-- Monitor table sizes
SELECT 
    table_name,
    table_rows,
    data_length/1024/1024 as data_size_mb,
    index_length/1024/1024 as index_size_mb
FROM information_schema.tables
WHERE table_schema = 'database_name'
ORDER BY data_length DESC;
```

Remember:
- Always use secure connections
- Regularly backup databases
- Monitor performance metrics
- Document all changes
- Use appropriate privileges
- Keep MySQL updated
- Implement monitoring

For detailed information, consult the MySQL documentation and manual pages (`man mysql`).