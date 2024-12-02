
## Cloud SQL and Database Services

### Cloud SQL Instance
```hcl
# cloudsql.tf
resource "google_sql_database_instance" "main" {
  name                = "${var.environment}-db-instance"
  project            = var.project_id
  region             = var.region
  database_version   = "POSTGRES_14"
  deletion_protection = true

  settings {
    tier              = var.database_tier
    availability_type = var.environment == "prod" ? "REGIONAL" : "ZONAL"
    disk_size         = var.disk_size
    disk_type         = "PD_SSD"
    
    backup_configuration {
      enabled                        = true
      point_in_time_recovery_enabled = true
      start_time                     = "02:00"
      transaction_log_retention_days = 7
      backup_retention_settings {
        retained_backups = 30
        retention_unit  = "COUNT"
      }
    }

    ip_configuration {
      ipv4_enabled                                  = false
      private_network                               = google_compute_network.vpc.id
      enable_private_path_for_google_cloud_services = true
      
      dynamic "authorized_networks" {
        for_each = var.authorized_networks
        content {
          name  = authorized_networks.value.name
          value = authorized_networks.value.cidr
        }
      }
    }

    maintenance_window {
      day          = 7    # Sunday
      hour         = 2    # 2 AM
      update_track = "stable"
    }

    insights_config {
      query_insights_enabled  = true
      query_string_length    = 1024
      record_application_tags = true
      record_client_address  = false
    }
  }

  depends_on = [
    google_project_service.services["sqladmin.googleapis.com"]
  ]
}

# Databases
resource "google_sql_database" "databases" {
  for_each = toset(var.databases)
  
  name     = each.value
  instance = google_sql_database_instance.main.name
  project  = var.project_id
}

# Users
resource "google_sql_user" "users" {
  for_each = var.database_users

  name     = each.key
  instance = google_sql_database_instance.main.name
  project  = var.project_id
  password = each.value.password
}
```

## Cloud Storage and IAM

### Storage Buckets
```hcl
# storage.tf
resource "google_storage_bucket" "buckets" {
  for_each = var.storage_buckets

  name          = "${var.project_id}-${each.key}"
  project       = var.project_id
  location      = each.value.location
  storage_class = each.value.storage_class

  versioning {
    enabled = each.value.versioning_enabled
  }

  lifecycle_rule {
    condition {
      age = each.value.lifecycle_rules.age
    }
    action {
      type = each.value.lifecycle_rules.action
    }
  }

  dynamic "cors" {
    for_each = each.value.cors_rules != null ? [1] : []
    content {
      origin          = each.value.cors_rules.origins
      method          = each.value.cors_rules.methods
      response_header = each.value.cors_rules.headers
      max_age_seconds = each.value.cors_rules.max_age
    }
  }

  uniform_bucket_level_access = true
  
  encryption {
    default_kms_key_name = each.value.kms_key_name
  }
}

# IAM for buckets
resource "google_storage_bucket_iam_binding" "bucket_iam" {
  for_each = var.bucket_iam_bindings

  bucket = google_storage_bucket.buckets[each.key].name
  role   = each.value.role
  members = each.value.members
}
```

## Load Balancing and SSL

### Global HTTPS Load Balancer
```hcl
# load_balancer.tf
# SSL Certificate
resource "google_compute_managed_ssl_certificate" "default" {
  name    = "${var.environment}-ssl-cert"
  project = var.project_id

  managed {
    domains = var.domains
  }
}

# External IP Address
resource "google_compute_global_address" "default" {
  name    = "${var.environment}-lb-ip"
  project = var.project_id
}

# HTTP(S) Load Balancer
resource "google_compute_global_forwarding_rule" "https" {
  name       = "${var.environment}-https-lb"
  project    = var.project_id
  target     = google_compute_target_https_proxy.default.id
  port_range = "443"
  ip_address = google_compute_global_address.default.address
}

resource "google_compute_target_https_proxy" "default" {
  name             = "${var.environment}-https-proxy"
  project          = var.project_id
  url_map          = google_compute_url_map.default.id
  ssl_certificates = [google_compute_managed_ssl_certificate.default.id]
}

resource "google_compute_url_map" "default" {
  name            = "${var.environment}-url-map"
  project         = var.project_id
  default_service = google_compute_backend_service.default.id

  dynamic "host_rule" {
    for_each = var.host_rules
    content {
      hosts        = [host_rule.value.host]
      path_matcher = host_rule.value.path_matcher
    }
  }

  dynamic "path_matcher" {
    for_each = var.path_matchers
    content {
      name            = path_matcher.key
      default_service = google_compute_backend_service.services[path_matcher.value.default_service].id

      dynamic "path_rule" {
        for_each = path_matcher.value.paths
        content {
          paths   = path_rule.value.paths
          service = google_compute_backend_service.services[path_rule.value.service].id
        }
      }
    }
  }
}

# Backend Services
resource "google_compute_backend_service" "services" {
  for_each = var.backend_services

  name                  = "${var.environment}-${each.key}"
  project               = var.project_id
  protocol              = each.value.protocol
  port_name             = each.value.port_name
  load_balancing_scheme = "EXTERNAL"
  timeout_sec          = 30

  dynamic "backend" {
    for_each = each.value.instance_groups
    content {
      group           = backend.value
      balancing_mode  = "UTILIZATION"
      capacity_scaler = 1.0
    }
  }

  health_check {
    check_interval_sec = 5
    timeout_sec       = 5
    healthy_threshold = 2
    port              = each.value.health_check_port
    request_path      = each.value.health_check_path
  }

  security_policy = each.value.security_policy
}
```

## Cloud Functions and Cloud Run

### Cloud Functions
```hcl
# functions.tf
resource "google_storage_bucket_object" "function_archive" {
  name   = "${var.function_name}-${data.archive_file.function.output_md5}.zip"
  bucket = google_storage_bucket.functions.name
  source = data.archive_file.function.output_path
}

resource "google_cloudfunctions_function" "function" {
  name        = var.function_name
  project     = var.project_id
  region      = var.region
  description = var.description

  runtime               = "python39"
  available_memory_mb   = var.memory
  source_archive_bucket = google_storage_bucket.functions.name
  source_archive_object = google_storage_bucket_object.function_archive.name
  trigger_http         = true
  entry_point         = var.entry_point

  environment_variables = var.environment_variables

  service_account_email = google_service_account.function_sa.email

  vpc_connector = google_vpc_access_connector.connector.id
  
  ingress_settings = "ALLOW_INTERNAL_ONLY"
}

# VPC Connector
resource "google_vpc_access_connector" "connector" {
  name          = "${var.environment}-vpc-connector"
  project       = var.project_id
  region        = var.region
  network       = google_compute_network.vpc.id
  ip_cidr_range = var.connector_cidr
}
```

### Cloud Run
```hcl
# cloudrun.tf
resource "google_cloud_run_service" "service" {
  name     = var.service_name
  project  = var.project_id
  location = var.region

  template {
    spec {
      containers {
        image = var.container_image
        
        resources {
          limits = {
            cpu    = var.cpu_limit
            memory = var.memory_limit
          }
        }

        env {
          name  = "ENV"
          value = var.environment
        }

        dynamic "env" {
          for_each = var.environment_variables
          content {
            name  = env.key
            value = env.value
          }
        }
      }

      service_account_name = google_service_account.cloudrun_sa.email
    }

    metadata {
      annotations = {
        "autoscaling.knative.dev/maxScale"      = var.max_instances
        "run.googleapis.com/vpc-access-connector" = google_vpc_access_connector.connector.id
        "run.googleapis.com/vpc-access-egress"    = "all-traffic"
      }
    }
  }

  traffic {
    percent         = 100
    latest_revision = true
  }

  autogenerate_revision_name = true
}

# IAM
resource "google_cloud_run_service_iam_member" "public" {
  count = var.public_access ? 1 : 0

  location = google_cloud_run_service.service.location
  project  = google_cloud_run_service.service.project
  service  = google_cloud_run_service.service.name
  role     = "roles/run.invoker"
  member   = "allUsers"
}
```

[Continue to Part 3 with Monitoring, Logging, Secret Management, and more advanced GCP configurations?]

Would you like me to continue with the next part covering more GCP services and advanced configurations?