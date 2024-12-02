
## Serverless and App Engine

### App Engine Configuration
```hcl
# appengine.tf
resource "google_app_engine_application" "app" {
  project     = var.project_id
  location_id = var.region
  
  database_type = "CLOUD_FIRESTORE"

  iap {
    enabled              = true
    oauth2_client_id     = var.oauth_client_id
    oauth2_client_secret = var.oauth_client_secret
  }
}

resource "google_app_engine_service" "service" {
  project = var.project_id
  service = var.service_name

  deployment {
    zip {
      source_url = "https://storage.googleapis.com/${google_storage_bucket.deployment.name}/${google_storage_bucket_object.app_zip.name}"
    }

    files {
      name = "app.yaml"
      source_url = "https://storage.googleapis.com/${google_storage_bucket.deployment.name}/app.yaml"
    }
  }

  traffic_split {
    allocations = {
      (google_app_engine_version.version.id) = 1
    }
  }
}

resource "google_app_engine_firewall_rule" "rule" {
  project      = var.project_id
  priority     = 1000
  action       = "ALLOW"
  source_range = "10.0.0.0/8"
  description  = "Allow internal traffic"
}
```

### Cloud Run Advanced Configuration
```hcl
# cloudrun_advanced.tf
resource "google_cloud_run_service" "service" {
  name     = "${var.environment}-service"
  project  = var.project_id
  location = var.region

  template {
    spec {
      service_account_name = google_service_account.cloudrun_sa.email
      
      containers {
        image = "${var.region}-docker.pkg.dev/${var.project_id}/${var.repository}/${var.image}:${var.tag}"
        
        resources {
          limits = {
            cpu    = var.cpu_limit
            memory = var.memory_limit
          }
          requests = {
            cpu    = var.cpu_request
            memory = var.memory_request
          }
        }

        ports {
          name           = "http1"
          container_port = 8080
        }

        startup_probe {
          http_get {
            path = "/healthz"
          }
          initial_delay_seconds = 10
          period_seconds       = 3
          failure_threshold    = 3
        }

        liveness_probe {
          http_get {
            path = "/health"
          }
          initial_delay_seconds = 15
          period_seconds       = 30
        }

        dynamic "env" {
          for_each = var.environment_variables
          content {
            name  = env.key
            value = env.value
          }
        }

        dynamic "env" {
          for_each = var.secret_environment_variables
          content {
            name = env.key
            value_from {
              secret_key_ref {
                name = env.value.secret_name
                key  = env.value.secret_key
              }
            }
          }
        }

        volume_mounts {
          name       = "config"
          mount_path = "/config"
        }
      }

      volumes {
        name = "config"
        config_map {
          name = google_kubernetes_config_map.config.metadata[0].name
        }
      }

      container_concurrency = var.concurrency
      timeout_seconds      = var.timeout
    }

    metadata {
      annotations = {
        "autoscaling.knative.dev/maxScale"      = var.max_instances
        "autoscaling.knative.dev/minScale"      = var.min_instances
        "run.googleapis.com/vpc-access-connector" = google_vpc_access_connector.connector.id
        "run.googleapis.com/vpc-access-egress"    = "all-traffic"
        "run.googleapis.com/cloudsql-instances"   = google_sql_database_instance.instance.connection_name
      }
      labels = var.labels
    }
  }

  traffic {
    percent         = 100
    latest_revision = true
  }

  lifecycle {
    ignore_changes = [
      template[0].metadata[0].annotations["client.knative.dev/user-image"],
      template[0].metadata[0].annotations["run.googleapis.com/client-name"],
      template[0].metadata[0].annotations["run.googleapis.com/client-version"],
    ]
  }
}

# Domain Mapping
resource "google_cloud_run_domain_mapping" "domain" {
  name     = var.domain_name
  project  = var.project_id
  location = var.region
  metadata {
    namespace = var.project_id
  }
  spec {
    route_name = google_cloud_run_service.service.name
  }
}

# IAM and Security
resource "google_cloud_run_service_iam_binding" "binding" {
  for_each = var.service_iam_bindings

  project  = var.project_id
  location = var.region
  service  = google_cloud_run_service.service.name
  role     = each.value.role
  members  = each.value.members
}
```

## Serverless VPC Access

### VPC Access Connector
```hcl
# vpc_access.tf
resource "google_vpc_access_connector" "connector" {
  name          = "${var.environment}-vpc-connector"
  project       = var.project_id
  region        = var.region
  network       = google_compute_network.vpc.id
  ip_cidr_range = var.connector_cidr_range
  
  machine_type  = "e2-micro"
  min_instances = var.min_instances
  max_instances = var.max_instances

  subnet {
    name = google_compute_subnetwork.connector_subnet.name
  }
}

resource "google_compute_subnetwork" "connector_subnet" {
  name          = "${var.environment}-connector-subnet"
  project       = var.project_id
  region        = var.region
  network       = google_compute_network.vpc.id
  ip_cidr_range = var.connector_subnet_range
  
  private_ip_google_access = true
}
```

## Cloud Endpoints and API Gateway

### API Gateway Configuration
```hcl
# api_gateway.tf
resource "google_api_gateway_api" "api" {
  provider = google-beta
  project  = var.project_id
  api_id   = "${var.environment}-api"
}

resource "google_api_gateway_api_config" "api_config" {
  provider      = google-beta
  project       = var.project_id
  api           = google_api_gateway_api.api.api_id
  api_config_id = "${var.environment}-config"

  openapi_documents {
    document {
      path     = "spec.yaml"
      contents = base64encode(file("${path.module}/specs/openapi.yaml"))
    }
  }

  gateway_config {
    backend_config {
      google_service_account = google_service_account.gateway_sa.email
    }
  }

  lifecycle {
    create_before_destroy = true
  }
}

resource "google_api_gateway_gateway" "gateway" {
  provider   = google-beta
  project    = var.project_id
  api_config = google_api_gateway_api_config.api_config.id
  gateway_id = "${var.environment}-gateway"
  region     = var.region

  display_name = "API Gateway for ${var.environment}"
  
  labels = var.labels
}
```

## Advanced Networking Features

### Cloud CDN and Load Balancing
```hcl
# cdn_lb.tf
resource "google_compute_backend_bucket" "cdn_backend" {
  name        = "${var.environment}-cdn-backend"
  project     = var.project_id
  bucket_name = google_storage_bucket.cdn_bucket.name
  enable_cdn  = true

  cdn_policy {
    cache_mode        = "CACHE_ALL_STATIC"
    client_ttl        = 3600
    default_ttl       = 3600
    max_ttl          = 86400
    negative_caching = true
    
    cache_key_policy {
      include_host           = true
      include_protocol       = true
      include_query_string  = true
    }
  }
}

resource "google_compute_url_map" "cdn_url_map" {
  name            = "${var.environment}-cdn-url-map"
  project         = var.project_id
  default_service = google_compute_backend_bucket.cdn_backend.id

  host_rule {
    hosts        = ["*.${var.domain}"]
    path_matcher = "cdn-paths"
  }

  path_matcher {
    name            = "cdn-paths"
    default_service = google_compute_backend_bucket.cdn_backend.id

    path_rule {
      paths   = ["/static/*"]
      service = google_compute_backend_bucket.cdn_backend.id
    }

    path_rule {
      paths   = ["/api/*"]
      service = google_compute_backend_service.api_backend.id
    }
  }
}
```

### Service Directory
```hcl
# service_directory.tf
resource "google_service_directory_namespace" "namespace" {
  provider     = google-beta
  project      = var.project_id
  namespace_id = "${var.environment}-namespace"
  location     = var.region

  labels = var.labels
}

resource "google_service_directory_service" "service" {
  provider    = google-beta
  service_id  = "${var.environment}-service"
  namespace   = google_service_directory_namespace.namespace.id

  metadata = {
    environment = var.environment
    version     = var.service_version
  }
}

resource "google_service_directory_endpoint" "endpoint" {
  provider    = google-beta
  endpoint_id = "${var.environment}-endpoint"
  service     = google_service_directory_service.service.id

  metadata = {
    region = var.region
    zone   = var.zone
  }

  address = var.service_address
  port    = var.service_port
}
```

[Continue to Part 6 with Data Analytics, BigQuery, and more advanced configurations?]

Would you like me to continue with the next part covering Data Analytics, BigQuery, and more advanced GCP configurations?