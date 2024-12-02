
## Dataset and Table Management

### Dataset Operations
```bash
# Create dataset with detailed specifications
bq mk \
    --dataset \
    --description="Dataset description" \
    --location=US \
    --default_table_expiration=3600 \
    --default_partition_expiration=7200 \
    --label=environment:production \
    PROJECT_ID:DATASET_NAME

# Set dataset access controls
bq update --dataset \
    --access_view PROJECT_ID:DATASET.VIEW_NAME \
    --access_role READER:user@domain.com \
    --access_role WRITER:serviceAccount:sa@project.iam.gserviceaccount.com \
    PROJECT_ID:DATASET_NAME

# Update dataset properties
bq update \
    --description="New description" \
    --default_table_expiration=7200 \
    --set_label project_id:PROJECT_ID \
    PROJECT_ID:DATASET_NAME
```

### Advanced Table Creation
```bash
# Create partitioned table
bq mk \
    --table \
    --time_partitioning_type=DAY \
    --time_partitioning_field=timestamp \
    --time_partitioning_expiration=7776000 \
    --clustering_fields=user_id,product_id \
    --schema schema.json \
    PROJECT_ID:DATASET.TABLE_NAME

# Create table with complex schema
bq mk \
    --table \
    PROJECT_ID:DATASET.TABLE_NAME \
    'id:STRING,
    user:RECORD,
    user.name:STRING,
    user.age:INTEGER,
    user.addresses:RECORD,
    user.addresses.type:STRING,
    user.addresses.line1:STRING,
    user.addresses.city:STRING,
    timestamps:TIMESTAMP,
    tags:STRING'

# Create table from query
bq mk \
    --use_legacy_sql=false \
    --expiration=3600 \
    --description="Created from query" \
    --label=type:derived \
    --time_partitioning_field=date \
    --clustering_fields=category \
    --destination_table PROJECT_ID:DATASET.TABLE_NAME \
    --allow_large_results \
    'SELECT * FROM `PROJECT_ID.DATASET.SOURCE_TABLE`'
```

## Data Loading and Export

### Data Loading Operations
```bash
# Load data from GCS with advanced options
bq load \
    --source_format=CSV \
    --skip_leading_rows=1 \
    --allow_quoted_newlines \
    --allow_jagged_rows \
    --max_bad_records=10 \
    --schema_update_options=ALLOW_FIELD_ADDITION \
    --time_partitioning_type=DAY \
    --time_partitioning_field=date \
    --clustering_fields=category \
    PROJECT_ID:DATASET.TABLE_NAME \
    gs://BUCKET_NAME/path/to/*.csv \
    schema.json

# Load JSON data
bq load \
    --source_format=NEWLINE_DELIMITED_JSON \
    --json_extension=AUTO \
    --ignore_unknown_values \
    --max_bad_records=5 \
    PROJECT_ID:DATASET.TABLE_NAME \
    gs://BUCKET_NAME/path/to/*.json \
    schema.json

# Load data from multiple sources
for file in data_*.csv; do
    bq load \
        --source_format=CSV \
        --append_table \
        PROJECT_ID:DATASET.TABLE_NAME \
        gs://BUCKET_NAME/path/to/$file \
        schema.json
done
```

### Export Operations
```bash
# Export table with compression
bq extract \
    --destination_format=CSV \
    --field_delimiter='|' \
    --print_header=true \
    --compression=GZIP \
    PROJECT_ID:DATASET.TABLE_NAME \
    gs://BUCKET_NAME/exports/data_*.csv.gz

# Export specific columns
bq query \
    --destination_format=AVRO \
    --use_legacy_sql=false \
    --allow_large_results \
    --replace \
    --destination_table=PROJECT_ID:DATASET.TEMP_TABLE \
    'SELECT column1, column2 
     FROM `PROJECT_ID.DATASET.TABLE_NAME`'

bq extract \
    --destination_format=AVRO \
    PROJECT_ID:DATASET.TEMP_TABLE \
    gs://BUCKET_NAME/exports/data_*.avro
```

## Query Optimization and Management

### Advanced Query Operations
```bash
# Create materialized view
bq query \
    --use_legacy_sql=false \
    --destination_table=PROJECT_ID:DATASET.MAT_VIEW \
    --materialized_view_tail_bytes=10000000000 \
    'CREATE MATERIALIZED VIEW `PROJECT_ID.DATASET.MAT_VIEW`
     AS SELECT 
        category,
        COUNT(*) as count,
        SUM(amount) as total_amount
     FROM `PROJECT_ID.DATASET.TABLE_NAME`
     GROUP BY category'

# Query with parameters
bq query \
    --use_legacy_sql=false \
    --parameter='start_date:TIMESTAMP:2024-01-01 00:00:00' \
    --parameter='end_date:TIMESTAMP:2024-12-31 23:59:59' \
    --parameter='min_amount:FLOAT64:1000.0' \
    'SELECT *
     FROM `PROJECT_ID.DATASET.TABLE_NAME`
     WHERE date BETWEEN @start_date AND @end_date
     AND amount > @min_amount'

# Create scheduled query
bq mk \
    --transfer_config \
    --target_dataset=DATASET \
    --display_name="Daily Aggregation" \
    --schedule="every 24 hours" \
    --schedule_start_time="2024-01-01T00:00:00Z" \
    --service_account_name="sa@project.iam.gserviceaccount.com" \
    --params='{"query":
        "CREATE OR REPLACE TABLE `PROJECT_ID.DATASET.DAILY_STATS` AS
         SELECT 
           DATE(timestamp) as date,
           COUNT(*) as count,
           SUM(amount) as total_amount
         FROM `PROJECT_ID.DATASET.TABLE_NAME`
         GROUP BY date"}'
```

### Performance Optimization
```bash
# Analyze query
bq query \
    --use_legacy_sql=false \
    --dry_run \
    'SELECT * FROM `PROJECT_ID.DATASET.TABLE_NAME`'

# Create partitioned table from query
bq query \
    --use_legacy_sql=false \
    --destination_table=PROJECT_ID:DATASET.PARTITIONED_TABLE \
    --time_partitioning_field=date \
    --clustering_fields=category \
    --replace \
    'SELECT * FROM `PROJECT_ID.DATASET.TABLE_NAME`'

# Optimize join operations
bq query \
    --use_legacy_sql=false \
    --parameter='date:DATE:2024-01-01' \
    'WITH small_table AS (
        SELECT * 
        FROM `PROJECT_ID.DATASET.SMALL_TABLE`
        WHERE date = @date
    )
    SELECT 
        t1.*,
        t2.additional_info
    FROM `PROJECT_ID.DATASET.LARGE_TABLE` t1
    JOIN small_table t2
    ON t1.id = t2.id'
```

## Monitoring and Administration

### Cost Control
```bash
# Set custom quota
bq update \
    --project_id=PROJECT_ID \
    --quota=1000000000 \
    PROJECT_ID:DATASET

# Monitor query costs
bq query \
    --use_legacy_sql=false \
    --dry_run \
    'SELECT * FROM `PROJECT_ID.DATASET.TABLE_NAME`'

# Set cost controls
bq query \
    --maximum_bytes_billed=1000000000 \
    --use_legacy_sql=false \
    'SELECT * FROM `PROJECT_ID.DATASET.TABLE_NAME`'
```

### Monitoring
```bash
# Create logging sink
bq mk \
    --transfer_config \
    --target_dataset=AUDIT_DATASET \
    --display_name="Query Audit" \
    --data_source="google_cloud_audit_logs" \
    --params='{
        "service_name": "bigquery.googleapis.com",
        "log_type": "all"
    }'

# Monitor long-running queries
bq query \
    --use_legacy_sql=false \
    'SELECT 
        job_id,
        user_email,
        total_bytes_processed,
        total_slot_ms,
        creation_time,
        end_time
     FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
     WHERE creation_time > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 24 HOUR)
     AND state = "RUNNING"
     ORDER BY creation_time DESC'
```

### Security and Access Control
```bash
# Set row-level security
bq query \
    --use_legacy_sql=false \
    'CREATE ROW ACCESS POLICY filter_by_department
     ON `PROJECT_ID.DATASET.TABLE_NAME`
     GRANT TO ("user@domain.com", "group@domain.com")
     FILTER USING (department = SESSION_USER())'

# Create authorized view
bq query \
    --use_legacy_sql=false \
    'CREATE VIEW `PROJECT_ID.DATASET.SECURE_VIEW`
     AS SELECT 
        user_id,
        SUM(amount) as total_amount
     FROM `PROJECT_ID.DATASET.TABLE_NAME`
     WHERE department = SESSION_USER()
     GROUP BY user_id'

# Grant view access
bq update \
    --view_uris=PROJECT_ID:DATASET.SECURE_VIEW \
    PROJECT_ID:DATASET.TABLE_NAME
```

Remember:
- Always test queries with `--dry_run` first
- Use partitioning and clustering for large tables
- Implement appropriate access controls
- Monitor query costs and performance
- Use materialized views for frequent queries
- Implement appropriate backup strategies
- Document all configurations and schemas
- Regular performance auditing

For detailed information, consult the BigQuery documentation and `bq` command reference (`bq help`).

Would you like me to cover any specific aspect of either Compute Engine or BigQuery in more detail?