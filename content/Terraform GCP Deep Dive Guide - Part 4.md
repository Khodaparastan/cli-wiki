

## Cloud Build and CI/CD

### Cloud Build Configuration
```hcl
# cloudbuild.tf
resource "google_cloudbuild_trigger" "build_trigger" {
  name        = "${var.environment}-build-trigger"
  project     = var.project_id
  description = "Build and deploy to ${var.environment}"

  github {
    owner = var.github_owner
    name  = var.github_repo
    push {
      branch = "^${var.branch_name}$"
    }
  }

  included_files = ["src/**", "terraform/**"]
  
  service_account = google_service_account.cloudbuild_sa.id

  build {
    step {
      name = "gcr.io/cloud-builders/docker"
      args = [
        "build",
        "-t", "gcr.io/$PROJECT_ID/${var.app_name}:$COMMIT_SHA",
        "."
      ]
    }

    step {
      name = "gcr.io/cloud-builders/docker"
      args = [
        "push",
        "gcr.io/$PROJECT_ID/${var.app_name}:$COMMIT_SHA"
      ]
    }

    step {
      name = "hashicorp/terraform"
      args = [
        "init",
        "-backend-config=bucket=${var.tf_state_bucket}",
        "-backend-config=prefix=${var.environment}"
      ]
      dir = "terraform"
    }

    step {
      name = "hashicorp/terraform"
      args = [
        "apply",
        "-var=image_tag=$COMMIT_SHA",
        "-auto-approve"
      ]
      dir = "terraform"
    }

    artifacts {
      images = ["gcr.io/$PROJECT_ID/${var.app_name}:$COMMIT_SHA"]
    }
  }
}

# Cloud Build Service Account
resource "google_service_account" "cloudbuild_sa" {
  account_id   = "cloudbuild-sa"
  display_name = "Cloud Build Service Account"
  project      = var.project_id
}

resource "google_project_iam_member" "cloudbuild_roles" {
  for_each = toset([
    "roles/cloudbuild.builds.builder",
    "roles/container.developer",
    "roles/storage.admin",
    "roles/cloudkms.cryptoKeyEncrypterDecrypter"
  ])

  project = var.project_id
  role    = each.value
  member  = "serviceAccount:${google_service_account.cloudbuild_sa.email}"
}
```

### Artifact Registry
```hcl
# artifacts.tf
resource "google_artifact_registry_repository" "repository" {
  provider = google-beta

  project       = var.project_id
  location      = var.region
  repository_id = "${var.environment}-repo"
  description   = "Docker repository for ${var.environment}"
  format        = "DOCKER"

  docker_config {
    immutable_tags = true
  }

  cleanup_policy_dry_run = false

  cleanup_policies {
    id     = "keep-minimum-versions"
    action = "KEEP"
    most_recent_versions {
      keep_count = 5
    }
  }

  cleanup_policies {
    id     = "delete-old-versions"
    action = "DELETE"
    condition {
      older_than = "2592000s" # 30 days
      tag_state  = "TAGGED"
    }
  }
}

# IAM for Artifact Registry
resource "google_artifact_registry_repository_iam_member" "repository_iam" {
  for_each = var.repository_iam_members

  project    = google_artifact_registry_repository.repository.project
  location   = google_artifact_registry_repository.repository.location
  repository = google_artifact_registry_repository.repository.name
  role       = each.value.role
  member     = each.value.member
}
```

## Cloud Scheduler and Tasks

### Cloud Scheduler
```hcl
# scheduler.tf
resource "google_cloud_scheduler_job" "jobs" {
  for_each = var.scheduler_jobs

  name             = each.key
  project          = var.project_id
  region           = var.region
  description      = each.value.description
  schedule         = each.value.schedule
  time_zone        = each.value.time_zone
  attempt_deadline = each.value.deadline

  retry_config {
    retry_count          = each.value.retry_count
    max_retry_duration   = each.value.max_retry_duration
    min_backoff_duration = each.value.min_backoff_duration
    max_backoff_duration = each.value.max_backoff_duration
    max_doublings       = each.value.max_doublings
  }

  http_target {
    uri         = each.value.target_uri
    http_method = each.value.http_method
    
    headers = each.value.headers

    body = base64encode(jsonencode(each.value.body))

    oauth_token {
      service_account_email = google_service_account.scheduler_sa[each.key].email
    }
  }
}

# Service Account for Scheduler
resource "google_service_account" "scheduler_sa" {
  for_each = var.scheduler_jobs

  account_id   = "scheduler-${each.key}"
  display_name = "Service Account for ${each.key} scheduler"
  project      = var.project_id
}
```

### Cloud Tasks
```hcl
# cloudtasks.tf
resource "google_cloud_tasks_queue" "queue" {
  for_each = var.task_queues

  name     = each.key
  project  = var.project_id
  location = var.region

  rate_limits {
    max_concurrent_dispatches = each.value.max_concurrent
    max_dispatches_per_second = each.value.max_rate
  }

  retry_config {
    max_attempts       = each.value.max_attempts
    max_retry_duration = each.value.max_retry_duration
    min_backoff       = each.value.min_backoff
    max_backoff       = each.value.max_backoff
    max_doublings     = each.value.max_doublings
  }

  stackdriver_logging_config {
    sampling_ratio = 1.0
  }
}

# IAM for Cloud Tasks
resource "google_cloud_tasks_queue_iam_member" "queue_invoker" {
  for_each = var.task_queues

  name     = google_cloud_tasks_queue.queue[each.key].name
  project  = var.project_id
  location = var.region
  role     = "roles/cloudtasks.enqueuer"
  member   = "serviceAccount:${google_service_account.task_sa[each.key].email}"
}
```

## Advanced Compute Configuration

### Instance Groups and Autoscaling
```hcl
# compute_advanced.tf
resource "google_compute_instance_template" "template" {
  name_prefix  = "${var.environment}-template-"
  project      = var.project_id
  machine_type = var.machine_type
  region       = var.region

  disk {
    source_image = var.boot_image
    auto_delete  = true
    boot         = true
    disk_type    = "pd-ssd"
    disk_size_gb = var.boot_disk_size

    dynamic "disk_encryption_key" {
      for_each = var.kms_key_id != null ? [1] : []
      content {
        kms_key_self_link = var.kms_key_id
      }
    }
  }

  network_interface {
    network    = var.network_id
    subnetwork = var.subnet_id

    dynamic "alias_ip_range" {
      for_each = var.alias_ip_ranges
      content {
        ip_cidr_range         = alias_ip_range.value.cidr
        subnetwork_range_name = alias_ip_range.value.range_name
      }
    }
  }

  metadata = {
    startup-script = templatefile("${path.module}/scripts/startup.sh", {
      environment = var.environment
      config      = var.app_config
    })
    shutdown-script = file("${path.module}/scripts/shutdown.sh")
  }

  service_account {
    email  = google_service_account.instance_sa.email
    scopes = ["cloud-platform"]
  }

  lifecycle {
    create_before_destroy = true
  }

  labels = var.labels
  tags   = var.network_tags
}

resource "google_compute_region_instance_group_manager" "mig" {
  name               = "${var.environment}-mig"
  project            = var.project_id
  base_instance_name = "${var.environment}-instance"
  region             = var.region

  version {
    instance_template = google_compute_instance_template.template.id
    name             = "primary"
  }

  target_pools = [google_compute_target_pool.pool.id]

  named_port {
    name = "http"
    port = 8080
  }

  auto_healing_policies {
    health_check      = google_compute_health_check.autohealing.id
    initial_delay_sec = 300
  }

  update_policy {
    type                           = "PROACTIVE"
    minimal_action                 = "REPLACE"
    most_disruptive_allowed_action = "REPLACE"
    max_surge_fixed               = 3
    max_unavailable_fixed         = 0
    min_ready_sec                 = 60
  }
}

resource "google_compute_region_autoscaler" "autoscaler" {
  name    = "${var.environment}-autoscaler"
  project = var.project_id
  region  = var.region
  target  = google_compute_region_instance_group_manager.mig.id

  autoscaling_policy {
    max_replicas    = var.max_replicas
    min_replicas    = var.min_replicas
    cooldown_period = 60

    cpu_utilization {
      target = 0.7
    }

    metric {
      name   = "custom.googleapis.com/metric"
      target = 100
      type   = "GAUGE"
    }

    scale_in_control {
      max_scaled_in_replicas {
        fixed = 2
      }
      time_window_sec = 300
    }
  }
}
```

[Continue to Part 5 with Serverless, App Engine, and more advanced configurations?]

Would you like me to continue with the next part covering Serverless, App Engine, and more advanced GCP configurations?