
## Cost Management and Resource Optimization

### Budget Configuration
```hcl
# budgets.tf
resource "google_billing_budget" "budget" {
  for_each = var.budgets

  billing_account = var.billing_account_id
  display_name    = "${var.environment}-${each.key}"

  budget_filter {
    projects = ["projects/${var.project_id}"]
    
    custom_period {
      start_date {
        year  = each.value.start_year
        month = each.value.start_month
        day   = 1
      }
      end_date {
        year  = each.value.end_year
        month = each.value.end_month
        day   = 1
      }
    }

    services = each.value.services
    labels   = each.value.labels
  }

  amount {
    specified_amount {
      currency_code = "USD"
      units        = each.value.amount
    }
  }

  threshold_rules {
    threshold_percent = 0.5
    spend_basis      = "CURRENT_SPEND"
  }

  threshold_rules {
    threshold_percent = 0.9
    spend_basis      = "FORECASTED_SPEND"
  }

  all_updates_rule {
    pubsub_topic = google_pubsub_topic.budget_alerts.id
    schema_version = "1.0"
  }
}
```

### Cost Optimization Policies
```hcl
# cost_optimization.tf
resource "google_compute_resource_policy" "instance_scheduler" {
  name        = "${var.environment}-instance-scheduler"
  project     = var.project_id
  region      = var.region
  description = "Automated instance scheduling for cost optimization"

  instance_schedule_policy {
    vm_start_schedule {
      schedule = "0 8 * * 1-5"
    }
    vm_stop_schedule {
      schedule = "0 18 * * 1-5"
    }
    time_zone = "America/New_York"
  }
}

resource "google_compute_resource_policy" "disk_snapshot" {
  name        = "${var.environment}-snapshot-schedule"
  project     = var.project_id
  region      = var.region
  description = "Automated disk snapshot schedule"

  snapshot_schedule_policy {
    schedule {
      daily_schedule {
        days_in_cycle = 1
        start_time    = "04:00"
      }
    }

    retention_policy {
      max_retention_days    = 14
      on_source_disk_delete = "KEEP_AUTO_SNAPSHOTS"
    }

    snapshot_properties {
      labels = {
        environment = var.environment
        automated  = "true"
      }
      storage_locations = [var.region]
    }
  }
}
```

### Resource Lifecycle Management
```hcl
# resource_lifecycle.tf
resource "google_compute_instance_group_manager" "lifecycle_mig" {
  name = "${var.environment}-lifecycle-mig"
  project = var.project_id
  zone    = var.zone

  version {
    instance_template = google_compute_instance_template.optimized_template.id
    name             = "primary"
  }

  auto_healing_policies {
    health_check      = google_compute_health_check.autohealing.id
    initial_delay_sec = 300
  }

  update_policy {
    type                         = "PROACTIVE"
    minimal_action              = "REPLACE"
    max_surge_fixed            = 3
    max_unavailable_fixed      = 0
    replacement_method         = "SUBSTITUTE"
    instance_redistribution_type = "PROACTIVE"
  }

  named_port {
    name = "http"
    port = 8080
  }

  stateful_internal_ip {
    interface_name = "nic0"
    delete_rule    = "ON_PERMANENT_INSTANCE_DELETION"
  }
}

# Disk Management
resource "google_compute_disk" "optimized_disk" {
  name  = "${var.environment}-optimized-disk"
  project = var.project_id
  zone    = var.zone
  type    = "pd-balanced"
  size    = var.disk_size

  provisioned_iops = var.disk_iops

  lifecycle {
    prevent_destroy = true
    ignore_changes = [
      labels["last_attached"]
    ]
  }
}
```

### Cost Analysis and Reporting
```hcl
# cost_analysis.tf
resource "google_bigquery_table" "cost_analysis" {
  dataset_id = google_bigquery_dataset.analytics.dataset_id
  project    = var.project_id
  table_id   = "cost_analysis"

  time_partitioning {
    type  = "MONTH"
    field = "usage_start_time"
  }

  clustering = ["project_id", "service"]

  schema = jsonencode([
    {
      name = "project_id",
      type = "STRING",
      mode = "REQUIRED"
    },
    {
      name = "service",
      type = "STRING",
      mode = "REQUIRED"
    },
    {
      name = "usage_start_time",
      type = "TIMESTAMP",
      mode = "REQUIRED"
    },
    {
      name = "cost",
      type = "FLOAT64",
      mode = "REQUIRED"
    },
    {
      name = "currency",
      type = "STRING",
      mode = "REQUIRED"
    }
  ])
}

# Scheduled Query for Cost Analysis
resource "google_bigquery_data_transfer_config" "cost_analysis" {
  display_name           = "Cost Analysis"
  project               = var.project_id
  location              = var.region
  data_source_id        = "scheduled_query"
  schedule              = "every 24 hours"
  destination_dataset_id = google_bigquery_dataset.analytics.dataset_id

  params = {
    query = <<EOF
      SELECT
        project.id as project_id,
        service.description as service,
        usage_start_time,
        cost,
        currency
      FROM
        `${var.billing_export_dataset}.gcp_billing_export_v1_*`
      WHERE
        _PARTITIONTIME >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
    EOF
  }
}
```

### Resource Recommendations
```hcl
# recommendations.tf
resource "google_monitoring_alert_policy" "resource_recommendations" {
  display_name = "${var.environment}-resource-recommendations"
  project      = var.project_id
  combiner     = "OR"

  conditions {
    display_name = "Idle VM Instances"
    condition_threshold {
      filter = <<-EOT
        resource.type = "gce_instance" AND
        metric.type = "compute.googleapis.com/instance/cpu/utilization" AND
        metric.labels.state = "idle"
      EOT
      duration = "86400s"
      comparison = "COMPARISON_GT"
      threshold_value = 0.1
    }
  }

  conditions {
    display_name = "Underutilized Persistent Disks"
    condition_threshold {
      filter = <<-EOT
        resource.type = "gce_disk" AND
        metric.type = "compute.googleapis.com/instance/disk/utilization"
      EOT
      duration = "604800s"
      comparison = "COMPARISON_LT"
      threshold_value = 0.2
    }
  }

  notification_channels = [
    google_monitoring_notification_channel.email.name
  ]

  documentation {
    content = <<-EOT
      Resource optimization recommendations:
      - Consider resizing or terminating idle VM instances
      - Review and possibly resize underutilized persistent disks
      - Check for unattached persistent disks
    EOT
    mime_type = "text/markdown"
  }
}
```

### Automated Cleanup
```hcl
# cleanup.tf
resource "google_cloud_scheduler_job" "cleanup_job" {
  name        = "${var.environment}-resource-cleanup"
  project     = var.project_id
  region      = var.region
  description = "Automated cleanup of unused resources"
  schedule    = "0 0 * * *"
  time_zone   = "UTC"

  http_target {
    uri         = google_cloudfunctions_function.cleanup.https_trigger_url
    http_method = "POST"
    
    oidc_token {
      service_account_email = google_service_account.cleanup_sa.email
    }
  }
}

resource "google_cloudfunctions_function" "cleanup" {
  name        = "${var.environment}-resource-cleanup"
  project     = var.project_id
  region      = var.region
  description = "Function to cleanup unused resources"

  runtime = "python39"

  available_memory_mb   = 256
  source_archive_bucket = google_storage_bucket.functions.name
  source_archive_object = google_storage_bucket_object.cleanup_function.name
  
  entry_point = "cleanup_resources"

  environment_variables = {
    PROJECT_ID = var.project_id
    DRY_RUN    = "false"
  }

  service_account_email = google_service_account.cleanup_sa.email
}
```

[Continue to Part 11 with Disaster Recovery, High Availability, and more advanced configurations?]

Would you like me to continue with the next part covering Disaster Recovery, High Availability, and more advanced GCP configurations?