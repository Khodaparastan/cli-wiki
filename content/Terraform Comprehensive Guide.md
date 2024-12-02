
I'll break this into multiple parts due to the extensive nature of Terraform configuration and best practices.

## Part 1: Basic Structure and Provider Configuration

### Directory Structure
```bash
project/
├── main.tf           # Main configuration file
├── variables.tf      # Variable declarations
├── outputs.tf        # Output declarations
├── terraform.tfvars  # Variable values
├── provider.tf       # Provider configuration
├── backend.tf        # Backend configuration
└── modules/
    ├── networking/
    ├── compute/
    └── storage/
```

### Provider Configuration
```hcl
# provider.tf
terraform {
  required_version = ">= 1.0.0"
  
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 4.0"
    }
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}

# GCP Provider
provider "google" {
  project     = var.project_id
  region      = var.region
  zone        = var.zone
  credentials = file(var.credentials_file)
}

# AWS Provider
provider "aws" {
  region     = var.aws_region
  access_key = var.aws_access_key
  secret_key = var.aws_secret_key
  
  default_tags {
    tags = {
      Environment = var.environment
      Project     = var.project_name
      ManagedBy   = "Terraform"
    }
  }
}
```

### Backend Configuration
```hcl
# backend.tf
terraform {
  backend "gcs" {
    bucket      = "tf-state-prod"
    prefix      = "terraform/state"
    credentials = "path/to/credentials.json"
  }
}

# Alternative S3 backend
terraform {
  backend "s3" {
    bucket         = "tf-state-prod"
    key            = "terraform/state/prod.tfstate"
    region         = "us-west-1"
    encrypt        = true
    dynamodb_table = "terraform-lock"
  }
}
```

### Variables and Outputs
```hcl
# variables.tf
variable "project_id" {
  description = "GCP Project ID"
  type        = string
}

variable "environment" {
  description = "Environment name"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "machine_types" {
  description = "Machine types for different environments"
  type        = map(string)
  default = {
    dev     = "n1-standard-1"
    staging = "n1-standard-2"
    prod    = "n1-standard-4"
  }
}

# outputs.tf
output "vpc_id" {
  description = "ID of the VPC"
  value       = module.networking.vpc_id
}

output "instance_ips" {
  description = "Public IPs of instances"
  value       = module.compute.instance_ips
  sensitive   = true
}
```

### Main Configuration
```hcl
# main.tf
locals {
  common_labels = {
    environment = var.environment
    project     = var.project_id
    managed_by  = "terraform"
  }
  
  network_name = "vpc-${var.environment}"
}

module "networking" {
  source = "./modules/networking"

  project_id = var.project_id
  region     = var.region
  vpc_name   = local.network_name
  subnets    = var.subnet_configs

  labels = local.common_labels
}

module "compute" {
  source = "./modules/compute"

  depends_on = [module.networking]

  project_id    = var.project_id
  network_id    = module.networking.vpc_id
  machine_type  = var.machine_types[var.environment]
  instance_name = "app-${var.environment}"
  
  labels = local.common_labels
}
```

## Module Structure

### Networking Module
```hcl
# modules/networking/main.tf
resource "google_compute_network" "vpc" {
  name                    = var.vpc_name
  auto_create_subnetworks = false
  project                 = var.project_id
}

resource "google_compute_subnetwork" "subnets" {
  for_each = var.subnets

  name          = each.key
  ip_cidr_range = each.value.cidr
  region        = var.region
  network       = google_compute_network.vpc.id
  project       = var.project_id

  secondary_ip_range = each.value.secondary_ranges
  
  dynamic "log_config" {
    for_each = var.enable_flow_logs ? [1] : []
    content {
      aggregation_interval = "INTERVAL_5_SEC"
      flow_sampling       = 0.5
      metadata           = "INCLUDE_ALL_METADATA"
    }
  }
}

# modules/networking/variables.tf
variable "vpc_name" {
  description = "Name of the VPC"
  type        = string
}

variable "subnets" {
  description = "Subnet configurations"
  type = map(object({
    cidr = string
    secondary_ranges = list(object({
      range_name    = string
      ip_cidr_range = string
    }))
  }))
}

# modules/networking/outputs.tf
output "vpc_id" {
  description = "ID of the created VPC"
  value       = google_compute_network.vpc.id
}

output "subnet_ids" {
  description = "Map of subnet names to IDs"
  value = {
    for name, subnet in google_compute_subnetwork.subnets :
    name => subnet.id
  }
}
```

### Compute Module
```hcl
# modules/compute/main.tf
resource "google_compute_instance" "vm" {
  name         = var.instance_name
  machine_type = var.machine_type
  zone         = var.zone
  project      = var.project_id

  boot_disk {
    initialize_params {
      image = var.boot_image
      size  = var.boot_disk_size
      type  = var.boot_disk_type
    }
  }

  network_interface {
    network    = var.network_id
    subnetwork = var.subnet_id

    dynamic "access_config" {
      for_each = var.assign_public_ip ? [1] : []
      content {
        // Ephemeral public IP
      }
    }
  }

  metadata_startup_script = file(var.startup_script)

  service_account {
    email  = var.service_account_email
    scopes = var.service_account_scopes
  }

  labels = var.labels

  lifecycle {
    prevent_destroy = var.environment == "prod"
  }
}

# Instance group
resource "google_compute_instance_group" "group" {
  count = var.create_instance_group ? 1 : 0

  name        = "${var.instance_name}-group"
  description = "Instance group for ${var.instance_name}"
  zone        = var.zone
  network     = var.network_id

  instances = [google_compute_instance.vm.self_link]

  named_port {
    name = "http"
    port = 80
  }
}
```

[Continue to Part 2 with more advanced configurations, state management, and best practices?]

Would you like me to continue with the next part covering more advanced Terraform configurations, state management, and best practices?