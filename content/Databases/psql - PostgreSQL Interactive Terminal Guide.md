
# Part 1: psql - PostgreSQL Interactive Terminal Guide

## Table of Contents
- [Overview](#overview)
- [Connection & Authentication](#connection--authentication)
- [Basic Commands](#basic-commands)
- [Navigation & Information](#navigation--information)
- [Query Execution](#query-execution)
- [Output Formatting](#output-formatting)
- [Advanced Features](#advanced-features)
- [Scripting & Automation](#scripting--automation)
- [Best Practices](#best-practices)

## Overview

psql is PostgreSQL's interactive terminal interface, providing command-line access to PostgreSQL databases with advanced features for both interactive and scripted use.

### Key Features
- Interactive SQL execution
- Meta-commands
- Script execution
- Output formatting
- Command history
- Tab completion
- Variable support
- Conditional execution

## Connection & Authentication

### Basic Connection
```bash
# Connect to database
psql -d database_name

# Connect with user
psql -U username -d database_name

# Connect to specific host/port
psql -h hostname -p 5432 -d database_name

# Connect with URL
psql postgresql://username:password@hostname:5432/database_name
```

### Connection Options
```bash
# Environment variables
export PGHOST=hostname
export PGPORT=5432
export PGDATABASE=dbname
export PGUSER=username
export PGPASSWORD=password

# Service file (~/.pg_service.conf)
[service_name]
host=hostname
port=5432
dbname=database_name
user=username
```

### SSL Connections
```bash
# Connect with SSL
psql "sslmode=verify-full sslcert=client-cert.pem \
      sslkey=client-key.pem sslrootcert=root.crt \
      host=hostname dbname=database_name"
```

## Basic Commands

### Meta-Commands
```sql
-- List databases
\l
\l+  -- with additional information

-- Connect to database
\c database_name

-- List tables
\dt
\dt+  -- with additional information

-- List schemas
\dn
\dn+

-- Describe table
\d table_name
\d+ table_name  -- with additional information
```

### Help Commands
```sql
-- General help
\?

-- SQL command help
\h

-- Specific command help
\h SELECT

-- Command history
\s

-- Edit command in editor
\e

-- Edit function in editor
\ef function_name
```

## Navigation & Information

### Object Information
```sql
-- List roles
\du

-- List tablespaces
\db

-- List functions
\df

-- List views
\dv

-- List sequences
\ds

-- List indexes
\di

-- Show table access privileges
\z table_name
```

### System Information
```sql
-- Show configuration
\show

-- Show client encoding
\encoding

-- Show search path
\echo :search_path

-- Show version
SELECT version();
```

## Query Execution

### Basic Execution
```sql
-- Execute query
SELECT * FROM table_name;

-- Execute from file
\i script.sql

-- Execute command and save results
\o output.txt
SELECT * FROM table_name;
\o

-- Timing of queries
\timing
SELECT count(*) FROM large_table;
```

### Transaction Control
```sql
-- Begin transaction
\set AUTOCOMMIT off
BEGIN;

-- Commit transaction
COMMIT;

-- Rollback transaction
ROLLBACK;

-- Set savepoint
SAVEPOINT my_savepoint;
ROLLBACK TO my_savepoint;
```

## Output Formatting

### Display Options
```sql
-- Set output format
\x auto  -- Extended display mode
\x on    -- Always extended
\x off   -- Never extended

-- Set field separator
\pset fieldsep ','

-- Set null display
\pset null '[NULL]'

-- Set line style
\pset linestyle ascii
\pset linestyle unicode
```

### Format Control
```sql
-- Border styles
\pset border 2

-- Toggle headers
\pset headers off

-- Toggle footer
\pset footer off

-- Set format
\pset format unaligned
\pset format aligned
\pset format wrapped
\pset format html
\pset format latex
```

[Continue to Part 2...]

# Part 2: psql Advanced Guide

## Advanced Features

### Variables and Interpolation
```sql
-- Set variables
\set variable_name 'value'

-- Use variables
SELECT :variable_name;

-- Conditional execution
\if :{?variable_name}
    \echo "Variable is set"
\else
    \echo "Variable is not set"
\endif
```

### Custom Prompts
```sql
-- Set prompt
\set PROMPT1 '%[%033[1;33;40m%]%n@%/%R%[%033[0m%]%# '
\set PROMPT2 '[more] %R > '

-- Include timing in prompt
\set PROMPT1 '%[%033[1;33;40m%]%n@%/%R%[%033[0m%]%# [%x] '
```

### Scripting Features
```sql
-- Error handling
\set ON_ERROR_STOP on

-- Echo commands
\set ECHO all

-- Quiet mode
\set QUIET on

-- Verbosity
\set VERBOSITY verbose
```

## Scripting & Automation

### Backup and Restore
```bash
# Backup database
pg_dump dbname > backup.sql

# Restore database
psql dbname < backup.sql

# Custom format backup
pg_dump -Fc dbname > backup.dump

# Restore custom format
pg_restore -d dbname backup.dump
```

### Maintenance Scripts
```bash
#!/bin/bash
# Database maintenance script

DBNAME="mydb"
LOGFILE="maintenance.log"

psql -d "$DBNAME" << EOF >> "$LOGFILE" 2>&1
-- Vacuum analyze
VACUUM ANALYZE;

-- Update statistics
ANALYZE;

-- Reindex
REINDEX DATABASE "$DBNAME";
EOF
```

### Monitoring Scripts
```bash
#!/bin/bash
# Database monitoring script

DBNAME="mydb"
EMAIL="dba@example.com"

# Check long running queries
psql -d "$DBNAME" -X -A -t << EOF | mail -s "Long Running Queries" "$EMAIL"
SELECT pid, 
       now() - pg_stat_activity.query_start AS duration,
       query
FROM pg_stat_activity
WHERE state != 'idle'
  AND now() - pg_stat_activity.query_start > interval '5 minutes';
EOF
```

## Best Practices

### Performance Monitoring
```sql
-- Check table sizes
SELECT relname as table_name,
       pg_size_pretty(pg_total_relation_size(relid)) as total_size
FROM pg_catalog.pg_statio_user_tables
ORDER BY pg_total_relation_size(relid) DESC;

-- Check index usage
SELECT schemaname, tablename, indexname, idx_scan, idx_tup_read
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Check cache hit ratio
SELECT 
    sum(heap_blks_read) as heap_read,
    sum(heap_blks_hit)  as heap_hit,
    sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) as ratio
FROM pg_statio_user_tables;
```

### Security Best Practices
```sql
-- Check user privileges
SELECT grantee, privilege_type 
FROM information_schema.role_table_grants 
WHERE table_name='sensitive_table';

-- Audit connections
SELECT datname, usename, client_addr, backend_start
FROM pg_stat_activity;

-- Check ssl status
SHOW ssl;
SELECT * FROM pg_stat_ssl;
```

### Example Scripts

#### Database Health Check
```bash
#!/bin/bash
# Comprehensive database health check

DBNAME="production_db"
REPORT="health_check_$(date +%Y%m%d).txt"

psql -d "$DBNAME" << EOF > "$REPORT"
\qecho '=== Database Size ==='
SELECT pg_size_pretty(pg_database_size(current_database()));

\qecho '\n=== Table Sizes ==='
SELECT relname as "Table",
       pg_size_pretty(pg_total_relation_size(relid)) as "Total Size",
       pg_size_pretty(pg_relation_size(relid)) as "Data Size",
       pg_size_pretty(pg_total_relation_size(relid) - pg_relation_size(relid)) 
         as "Index Size"
FROM pg_catalog.pg_statio_user_tables
ORDER BY pg_total_relation_size(relid) DESC
LIMIT 10;

\qecho '\n=== Cache Hit Ratio ==='
SELECT 
    relname as "Table",
    heap_blks_read as heap_read,
    heap_blks_hit as heap_hit,
    CASE WHEN heap_blks_hit + heap_blks_read = 0 
         THEN 0 
         ELSE round(100.0 * heap_blks_hit / (heap_blks_hit + heap_blks_read), 2) 
    END as hit_ratio
FROM pg_statio_user_tables
ORDER BY heap_blks_hit + heap_blks_read DESC
LIMIT 10;

\qecho '\n=== Index Usage ==='
SELECT 
    schemaname as "Schema",
    tablename as "Table",
    indexname as "Index",
    idx_scan as "Index Scans",
    idx_tup_read as "Tuples Read",
    idx_tup_fetch as "Tuples Fetched"
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC
LIMIT 10;

\qecho '\n=== Bloat Estimation ==='
WITH constants AS (
  SELECT current_setting('block_size')::numeric AS bs,
         23 AS hdr,
         8 AS ma
),
bloat_info AS (
  SELECT
    schemaname, tablename, reltuples::numeric as tups,
    relpages::numeric as pages, relam, bs,
    ceil((cc.reltuples*((datahdr+ma-
      (CASE WHEN datahdr%ma=0 THEN ma ELSE datahdr%ma END))+nullhdr2+4))/(bs-20::float)) 
      AS otta
  FROM pg_class cc
  JOIN pg_namespace nn ON cc.relnamespace = nn.oid
  JOIN constants ON true
  LEFT JOIN pg_statistic ON relid=cc.oid
)
SELECT
  schemaname as "Schema",
  tablename as "Table",
  ROUND(((pages/otta::numeric)*100 - 100)::numeric, 2) as "Bloat%",
  pg_size_pretty((bs*(pages-otta))::bigint) as "Wasted Size"
FROM bloat_info
WHERE pages > otta
ORDER BY ((pages/otta::numeric)*100 - 100) DESC
LIMIT 10;
EOF

echo "Health check report generated: $REPORT"
```

#### Automated Backup Script
```bash
#!/bin/bash
# Automated backup script with retention

BACKUP_DIR="/backup/postgres"
DBNAME="production_db"
RETENTION_DAYS=7
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/$DBNAME_$DATE.custom"
LOG_FILE="$BACKUP_DIR/backup.log"

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> "$LOG_FILE"
}

# Create backup directory if it doesn't exist
mkdir -p "$BACKUP_DIR"

# Perform backup
log "Starting backup of $DBNAME"
pg_dump -Fc "$DBNAME" > "$BACKUP_FILE"

if [ $? -eq 0 ]; then
    log "Backup completed successfully: $BACKUP_FILE"
    
    # Compress backup
    gzip "$BACKUP_FILE"
    log "Backup compressed"
    
    # Remove old backups
    find "$BACKUP_DIR" -name "*.custom.gz" -mtime +$RETENTION_DAYS -delete
    log "Old backups removed"
else
    log "Backup failed!"
    exit 1
fi

# Check backup size
BACKUP_SIZE=$(du -h "$BACKUP_FILE.gz" | cut -f1)
log "Backup size: $BACKUP_SIZE"

# Verify backup
log "Verifying backup"
pg_restore -l "$BACKUP_FILE.gz" >/dev/null 2>&1
if [ $? -eq 0 ]; then
    log "Backup verification successful"
else
    log "Backup verification failed!"
    exit 1
fi
```

---

Remember:
- Use appropriate connection security
- Maintain good password practices
- Regular backups
- Monitor performance
- Document commands and scripts
- Use version control for scripts
- Test in development first

For detailed information, consult the PostgreSQL documentation and psql manual (`man psql`).