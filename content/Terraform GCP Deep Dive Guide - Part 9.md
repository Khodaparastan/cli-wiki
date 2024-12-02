
## Compliance and Audit Logging

### Audit Logging Configuration
```hcl
# audit_logging.tf
resource "google_project_iam_audit_config" "audit_config" {
  project = var.project_id
  
  service = "allServices"
  
  audit_log_config {
    log_type = "ADMIN_READ"
  }
  
  audit_log_config {
    log_type = "DATA_READ"
    exempted_members = var.exempted_members
  }
  
  audit_log_config {
    log_type = "DATA_WRITE"
  }
}

# Log Sinks
resource "google_logging_project_sink" "audit_sink" {
  for_each = var.log_sinks

  name        = "${var.environment}-${each.key}"
  project     = var.project_id
  destination = each.value.destination

  filter = each.value.filter

  unique_writer_identity = true

  dynamic "exclusions" {
    for_each = each.value.exclusions
    content {
      name        = exclusions.key
      description = exclusions.value.description
      filter      = exclusions.value.filter
    }
  }
}

# Log Metrics
resource "google_logging_metric" "logging_metric" {
  for_each = var.log_metrics

  name    = "${var.environment}-${each.key}"
  project = var.project_id
  filter  = each.value.filter

  metric_descriptor {
    metric_kind = each.value.metric_kind
    value_type  = each.value.value_type
    unit        = each.value.unit
    labels {
      key         = "severity"
      value_type  = "STRING"
      description = "Severity of the event"
    }
  }

  label_extractors = {
    "severity" = "EXTRACT(severity)"
  }
}
```

### Log Export and Aggregation
```hcl
# log_export.tf
resource "google_storage_bucket" "log_archive" {
  name          = "${var.project_id}-log-archive"
  project       = var.project_id
  location      = var.region
  storage_class = "COLDLINE"

  lifecycle_rule {
    condition {
      age = 365
    }
    action {
      type = "Delete"
    }
  }

  retention_policy {
    is_locked = true
    retention_period = 2592000 # 30 days
  }
}

resource "google_bigquery_dataset" "logs_dataset" {
  dataset_id  = "${var.environment}_logs"
  project     = var.project_id
  location    = var.region
  description = "Dataset for centralized logging"

  default_table_expiration_ms = 8640000000 # 100 days

  access {
    role          = "OWNER"
    user_by_email = google_service_account.logs_admin.email
  }

  access {
    role           = "READER"
    group_by_email = var.security_group
  }
}

resource "google_logging_project_sink" "aggregated_sink" {
  name        = "${var.environment}-aggregated-sink"
  project     = var.project_id
  destination = "bigquery.googleapis.com/projects/${var.project_id}/datasets/${google_bigquery_dataset.logs_dataset.dataset_id}"

  filter = <<EOF
    resource.type="gce_instance" OR
    resource.type="cloud_function" OR
    resource.type="cloud_run_revision" OR
    resource.type="gke_cluster"
EOF

  unique_writer_identity = true

  bigquery_options {
    use_partitioned_tables = true
  }
}
```

### Compliance Monitoring
```hcl
# compliance.tf
resource "google_monitoring_dashboard" "compliance_dashboard" {
  dashboard_json = jsonencode({
    displayName = "Compliance Dashboard"
    gridLayout = {
      widgets = [
        {
          title = "IAM Changes"
          xyChart = {
            dataSets = [{
              timeSeriesQuery = {
                timeSeriesFilter = {
                  filter = "resource.type=\"project\" AND protoPayload.methodName=\"SetIamPolicy\""
                }
              }
            }]
          }
        },
        {
          title = "Security Group Changes"
          xyChart = {
            dataSets = [{
              timeSeriesQuery = {
                timeSeriesFilter = {
                  filter = "resource.type=\"group\" AND protoPayload.methodName=~\"groups.*\""
                }
              }
            }]
          }
        }
      ]
    }
  })
}

# Compliance Alerts
resource "google_monitoring_alert_policy" "compliance_alerts" {
  for_each = var.compliance_alerts

  display_name = "${var.environment}-${each.key}"
  project      = var.project_id
  combiner     = "OR"

  conditions {
    display_name = each.value.condition_name
    
    condition_threshold {
      filter          = each.value.filter
      duration        = each.value.duration
      comparison      = each.value.comparison
      threshold_value = each.value.threshold
    }
  }

  notification_channels = [
    for channel in each.value.notification_channels :
    google_monitoring_notification_channel.compliance_channels[channel].name
  ]

  documentation {
    content   = each.value.documentation
    mime_type = "text/markdown"
  }
}
```

### Data Governance
```hcl
# data_governance.tf
resource "google_data_catalog_policy_tag" "policy_tags" {
  for_each = var.policy_tags

  provider     = google-beta
  project      = var.project_id
  taxonomy     = google_data_catalog_taxonomy.taxonomy.id
  display_name = each.value.display_name
  description  = each.value.description

  dynamic "child_policy_tags" {
    for_each = each.value.children
    content {
      display_name = child_policy_tags.value.display_name
      description  = child_policy_tags.value.description
    }
  }
}

resource "google_data_catalog_taxonomy" "taxonomy" {
  provider     = google-beta
  project      = var.project_id
  region       = var.region
  display_name = "${var.environment} Data Taxonomy"
  description  = "Taxonomy for data classification"
  
  activated_policy_types = [
    "FINE_GRAINED_ACCESS_CONTROL"
  ]
}

resource "google_data_catalog_taxonomy_iam_binding" "taxonomy_iam" {
  provider = google-beta
  project  = var.project_id
  taxonomy = google_data_catalog_taxonomy.taxonomy.id
  role     = "roles/datacatalog.categoryFineGrainedReader"
  members  = var.taxonomy_readers
}
```

### Asset Inventory
```hcl
# asset_inventory.tf
resource "google_cloud_asset_project_feed" "asset_feed" {
  project      = var.project_id
  feed_id      = "${var.environment}-asset-feed"
  content_type = "RESOURCE"

  asset_types = [
    "compute.googleapis.com/Instance",
    "storage.googleapis.com/Bucket",
    "iam.googleapis.com/ServiceAccount"
  ]

  feed_output_config {
    pubsub_destination {
      topic = google_pubsub_topic.asset_feed.id
    }
  }

  condition {
    expression = "resource.location.startsWith('us-')"
  }
}

resource "google_cloud_asset_organization_search" "asset_search" {
  provider = google-beta
  
  scope = "organizations/${var.organization_id}"
  query = "state:ACTIVE"
  
  search_results_storage_config {
    storage_destination = "gs://${google_storage_bucket.asset_inventory.name}/search-results"
  }
}

resource "google_cloud_asset_folder_feed" "folder_feed" {
  provider     = google-beta
  folder       = var.folder_id
  feed_id      = "${var.environment}-folder-feed"
  content_type = "IAM_POLICY"

  feed_output_config {
    pubsub_destination {
      topic = google_pubsub_topic.folder_feed.id
    }
  }
}
```

[Continue to Part 10 with Cost Management, Resource Optimization, and more advanced configurations?]

Would you like me to continue with the next part covering Cost Management, Resource Optimization, and more advanced GCP configurations?