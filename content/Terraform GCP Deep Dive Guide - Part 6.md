
## Data Analytics and BigQuery

### BigQuery Configuration
```hcl
# bigquery.tf
resource "google_bigquery_dataset" "dataset" {
  dataset_id                  = "${var.environment}_dataset"
  project                     = var.project_id
  friendly_name              = "Analytics Dataset for ${var.environment}"
  description                = "Dataset for analytics data in ${var.environment}"
  location                   = var.region
  default_table_expiration_ms = var.table_expiration_ms

  access {
    role          = "OWNER"
    user_by_email = google_service_account.bq_admin.email
  }

  access {
    role           = "READER"
    group_by_email = var.analyst_group
  }

  dynamic "access" {
    for_each = var.authorized_views
    content {
      view {
        project_id = access.value.project_id
        dataset_id = access.value.dataset_id
        table_id   = access.value.table_id
      }
    }
  }

  labels = var.labels
}

resource "google_bigquery_table" "table" {
  for_each = var.tables

  dataset_id = google_bigquery_dataset.dataset.dataset_id
  project    = var.project_id
  table_id   = each.key
  schema     = file(each.value.schema_file)

  dynamic "time_partitioning" {
    for_each = each.value.partitioning != null ? [each.value.partitioning] : []
    content {
      type                     = time_partitioning.value.type
      field                    = time_partitioning.value.field
      expiration_ms           = time_partitioning.value.expiration_ms
      require_partition_filter = time_partitioning.value.require_filter
    }
  }

  dynamic "clustering" {
    for_each = each.value.clustering_fields != null ? [1] : []
    content {
      fields = each.value.clustering_fields
    }
  }

  encryption_configuration {
    kms_key_name = var.kms_key_name
  }
}

# Scheduled Queries
resource "google_bigquery_data_transfer_config" "scheduled_query" {
  for_each = var.scheduled_queries

  display_name           = each.key
  project               = var.project_id
  location              = var.region
  data_source_id        = "scheduled_query"
  schedule              = each.value.schedule
  destination_dataset_id = google_bigquery_dataset.dataset.dataset_id
  
  params = {
    query = each.value.query
  }

  service_account_name = google_service_account.bq_transfer.email
}
```

### Dataflow Pipeline
```hcl
# dataflow.tf
resource "google_dataflow_job" "pipeline" {
  for_each = var.dataflow_jobs

  name                  = "${var.environment}-${each.key}"
  project               = var.project_id
  zone                  = var.zone
  template_gcs_path     = each.value.template_path
  temp_gcs_location     = "${google_storage_bucket.dataflow_temp.url}/temp"
  service_account_email = google_service_account.dataflow_sa.email
  network               = google_compute_network.vpc.name
  subnetwork            = google_compute_subnetwork.dataflow_subnet.self_link
  
  max_workers = each.value.max_workers
  machine_type = each.value.machine_type

  parameters = merge(
    each.value.parameters,
    {
      project = var.project_id
      region  = var.region
    }
  )

  on_delete = "drain"

  additional_experiments = [
    "use_runner_v2",
    "enable_prime"
  ]
}

# Flex Template
resource "google_dataflow_flex_template_job" "flex_job" {
  provider                = google-beta
  name                    = "${var.environment}-flex-job"
  project                 = var.project_id
  location                = var.region
  container_spec_gcs_path = "${google_storage_bucket.dataflow_templates.url}/flex-template.json"

  parameters = {
    input_subscription  = google_pubsub_subscription.input.id
    output_table       = "${google_bigquery_table.output.project}:${google_bigquery_table.output.dataset_id}.${google_bigquery_table.output.table_id}"
    temp_location      = "${google_storage_bucket.dataflow_temp.url}/temp"
  }
}
```

### Pub/Sub Configuration
```hcl
# pubsub.tf
resource "google_pubsub_topic" "topics" {
  for_each = var.topics

  project = var.project_id
  name    = "${var.environment}-${each.key}"

  message_retention_duration = each.value.retention_duration
  
  dynamic "message_storage_policy" {
    for_each = each.value.allowed_regions != null ? [1] : []
    content {
      allowed_persistence_regions = each.value.allowed_regions
    }
  }

  schema_settings {
    schema   = google_pubsub_schema.schemas[each.value.schema].id
    encoding = each.value.encoding
  }

  kms_key_name = var.kms_key_name
}

resource "google_pubsub_schema" "schemas" {
  for_each = var.schemas

  project = var.project_id
  name    = "${var.environment}-${each.key}"
  type    = each.value.type
  definition = file(each.value.schema_file)
}

resource "google_pubsub_subscription" "subscriptions" {
  for_each = var.subscriptions

  project = var.project_id
  name    = "${var.environment}-${each.key}"
  topic   = google_pubsub_topic.topics[each.value.topic].id

  message_retention_duration = each.value.retention_duration
  retain_acked_messages     = each.value.retain_acked
  ack_deadline_seconds      = each.value.ack_deadline

  expiration_policy {
    ttl = each.value.expiration_ttl
  }

  retry_policy {
    minimum_backoff = each.value.min_retry_backoff
    maximum_backoff = each.value.max_retry_backoff
  }

  dynamic "dead_letter_policy" {
    for_each = each.value.dead_letter_topic != null ? [1] : []
    content {
      dead_letter_topic     = google_pubsub_topic.topics[each.value.dead_letter_topic].id
      max_delivery_attempts = each.value.max_delivery_attempts
    }
  }

  dynamic "push_config" {
    for_each = each.value.push_endpoint != null ? [1] : []
    content {
      push_endpoint = each.value.push_endpoint
      
      oidc_token {
        service_account_email = google_service_account.pubsub_sa.email
      }
    }
  }
}
```

### Data Catalog
```hcl
# datacatalog.tf
resource "google_data_catalog_entry_group" "entry_group" {
  provider      = google-beta
  project       = var.project_id
  entry_group_id = "${var.environment}-entry-group"
  display_name  = "Entry Group for ${var.environment}"
  description   = "Catalog entries for ${var.environment} data assets"
  region        = var.region
}

resource "google_data_catalog_entry" "entry" {
  for_each = var.catalog_entries

  provider         = google-beta
  project          = var.project_id
  entry_group     = google_data_catalog_entry_group.entry_group.id
  entry_id        = each.key
  display_name    = each.value.display_name
  description     = each.value.description
  type            = each.value.type

  gcs_fileset_spec {
    file_patterns = each.value.file_patterns
  }

  user_specified_system = each.value.system
  user_specified_type   = each.value.user_type

  schema = jsonencode(each.value.schema)
}

resource "google_data_catalog_tag_template" "template" {
  for_each = var.tag_templates

  provider          = google-beta
  project           = var.project_id
  tag_template_id   = "${var.environment}-${each.key}"
  display_name      = each.value.display_name
  region            = var.region

  dynamic "fields" {
    for_each = each.value.fields
    content {
      field_id     = fields.key
      display_name = fields.value.display_name
      type {
        primitive_type = fields.value.type
      }
      is_required = fields.value.required
    }
  }

  force_delete = each.value.force_delete
}
```

[Continue to Part 7 with Machine Learning, AI Platform, and more advanced configurations?]

Would you like me to continue with the next part covering Machine Learning, AI Platform, and more advanced GCP configurations?