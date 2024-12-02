
## State Management and Workspaces

### State Management
```hcl
# Remote State Access
data "terraform_remote_state" "network" {
  backend = "gcs"
  
  config = {
    bucket      = "tf-state-prod"
    prefix      = "terraform/network"
    credentials = file(var.credentials_file)
  }
}

# State Import
# CLI Command
terraform import google_compute_instance.vm projects/project-id/zones/zone/instances/instance-name

# State Management Commands
terraform state list
terraform state show google_compute_instance.vm
terraform state mv google_compute_instance.old google_compute_instance.new
terraform state rm google_compute_instance.vm
```

### Workspace Management
```bash
# Create and manage workspaces
terraform workspace new dev
terraform workspace new prod
terraform workspace select dev
terraform workspace list

# Workspace-specific configurations
locals {
  environment = terraform.workspace
  
  instance_count = {
    dev  = 1
    prod = 3
  }
  
  instance_type = {
    dev  = "n1-standard-1"
    prod = "n1-standard-2"
  }
}
```

## Advanced Resource Configurations

### Complex Compute Configuration
```hcl
resource "google_compute_instance_template" "template" {
  name_prefix  = "instance-template-"
  machine_type = var.machine_type
  region       = var.region

  disk {
    source_image = var.boot_image
    auto_delete  = true
    boot         = true
    
    dynamic "disk_encryption_key" {
      for_each = var.enable_disk_encryption ? [1] : []
      content {
        kms_key_self_link = var.kms_key_link
      }
    }
  }

  network_interface {
    network    = var.network_id
    subnetwork = var.subnet_id

    dynamic "alias_ip_range" {
      for_each = var.alias_ip_ranges
      content {
        subnetwork_range_name = alias_ip_range.value.range_name
        ip_cidr_range        = alias_ip_range.value.cidr
      }
    }
  }

  metadata = {
    startup-script = templatefile("${path.module}/scripts/startup.sh", {
      environment = var.environment
      config      = var.app_config
    })
  }

  service_account {
    email  = var.service_account_email
    scopes = ["cloud-platform"]
  }

  lifecycle {
    create_before_destroy = true
  }
}

resource "google_compute_instance_group_manager" "mig" {
  name = "managed-instance-group"
  zone = var.zone

  version {
    instance_template = google_compute_instance_template.template.self_link
    name             = "primary"
  }

  target_pools = [google_compute_target_pool.pool.self_link]
  base_instance_name = "app"

  auto_healing_policies {
    health_check      = google_compute_health_check.autohealing.id
    initial_delay_sec = 300
  }

  update_policy {
    type                  = "PROACTIVE"
    minimal_action        = "REPLACE"
    max_surge_fixed       = 3
    max_unavailable_fixed = 0
  }
}
```

### IAM and Security
```hcl
# Custom IAM Role
resource "google_project_iam_custom_role" "custom_role" {
  role_id     = "customRole"
  title       = "Custom Role"
  description = "Custom IAM role for application"
  permissions = [
    "compute.instances.get",
    "compute.instances.list",
    "compute.instances.start",
    "compute.instances.stop"
  ]
}

# Service Account with Custom Role
resource "google_service_account" "sa" {
  account_id   = "custom-sa"
  display_name = "Custom Service Account"
}

resource "google_project_iam_binding" "sa_binding" {
  project = var.project_id
  role    = google_project_iam_custom_role.custom_role.id

  members = [
    "serviceAccount:${google_service_account.sa.email}",
  ]
}

# Secret Management
resource "google_secret_manager_secret" "secret" {
  secret_id = "app-secret"

  replication {
    automatic = true
  }
}

resource "google_secret_manager_secret_version" "version" {
  secret = google_secret_manager_secret.secret.id
  secret_data = var.secret_value
}
```

### Load Balancing and SSL
```hcl
# HTTPS Load Balancer
resource "google_compute_global_forwarding_rule" "default" {
  name       = "global-rule"
  target     = google_compute_target_https_proxy.default.id
  port_range = "443"
}

resource "google_compute_managed_ssl_certificate" "default" {
  name = "cert"

  managed {
    domains = [var.domain_name]
  }
}

resource "google_compute_target_https_proxy" "default" {
  name             = "https-proxy"
  url_map          = google_compute_url_map.default.id
  ssl_certificates = [google_compute_managed_ssl_certificate.default.id]
}

resource "google_compute_url_map" "default" {
  name            = "url-map"
  default_service = google_compute_backend_service.default.id

  host_rule {
    hosts        = [var.domain_name]
    path_matcher = "allpaths"
  }

  path_matcher {
    name            = "allpaths"
    default_service = google_compute_backend_service.default.id

    path_rule {
      paths   = ["/api/*"]
      service = google_compute_backend_service.api.id
    }
  }
}
```

## Advanced Features and Best Practices

### Data Sources and Dynamic Configurations
```hcl
# Use data sources for dynamic configuration
data "google_compute_zones" "available" {
  project = var.project_id
  region  = var.region
}

locals {
  zones = data.google_compute_zones.available.names
  
  instance_distribution = {
    for idx, zone in local.zones : zone => {
      instances = ceil(var.total_instances / length(local.zones))
      zone_index = idx
    }
  }
}

# Dynamic block usage
resource "google_compute_firewall" "rules" {
  name    = "firewall-rules"
  network = var.network_id

  dynamic "allow" {
    for_each = var.firewall_rules
    content {
      protocol = allow.value.protocol
      ports    = allow.value.ports
    }
  }
}
```

### Terraform Functions and Conditionals
```hcl
locals {
  # Complex transformations
  subnet_cidrs = [
    for subnet in var.subnets :
    cidrsubnet(var.base_cidr, 8, index(var.subnets, subnet))
  ]
  
  # Conditional resource creation
  create_nat = var.environment == "prod" ? 1 : 0
  
  # Map transformation
  instance_tags = merge(
    var.common_tags,
    {
      environment = var.environment
      timestamp  = formatdate("YYYY-MM-DD", timestamp())
    }
  )
}

# Conditional resource creation
resource "google_compute_router_nat" "nat" {
  count = local.create_nat

  name                               = "nat-${var.environment}"
  router                            = google_compute_router.router.name
  region                            = var.region
  nat_ip_allocate_option            = "AUTO_ONLY"
  source_subnetwork_ip_ranges_to_nat = "ALL_SUBNETWORKS_ALL_IP_RANGES"
}
```

### Error Handling and Validation
```hcl
# Input validation
variable "instance_count" {
  type        = number
  description = "Number of instances to create"

  validation {
    condition     = var.instance_count > 0 && var.instance_count <= 10
    error_message = "Instance count must be between 1 and 10."
  }
}

# Complex validation
variable "subnet_config" {
  type = object({
    name = string
    cidr = string
    region = string
  })

  validation {
    condition     = can(cidrhost(var.subnet_config.cidr, 0))
    error_message = "Invalid CIDR format."
  }

  validation {
    condition     = length(var.subnet_config.name) <= 63
    error_message = "Subnet name must be 63 characters or less."
  }
}
```

[Continue to Part 3 with Testing, CI/CD Integration, and Advanced Modules?]

Would you like me to continue with the next part covering testing, CI/CD integration, and advanced module patterns?