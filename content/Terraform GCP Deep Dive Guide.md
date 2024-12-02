
## Part 1: Core GCP Infrastructure

### Project and Organization Setup
```hcl
# project_setup.tf
resource "google_project" "project" {
  name            = "My Project"
  project_id      = var.project_id
  folder_id       = var.folder_id
  billing_account = var.billing_account_id

  labels = {
    environment = var.environment
    team        = var.team_name
  }
}

resource "google_project_service" "services" {
  for_each = toset([
    "compute.googleapis.com",
    "container.googleapis.com",
    "cloudfunctions.googleapis.com",
    "cloudrun.googleapis.com",
    "cloudbuild.googleapis.com",
    "monitoring.googleapis.com",
    "logging.googleapis.com",
    "cloudkms.googleapis.com",
    "secretmanager.googleapis.com"
  ])

  project = google_project.project.project_id
  service = each.key

  disable_on_destroy = false
}
```

### VPC and Networking
```hcl
# networking.tf
resource "google_compute_network" "vpc" {
  name                            = "${var.environment}-vpc"
  project                         = var.project_id
  auto_create_subnetworks        = false
  delete_default_routes_on_create = true
  mtu                            = 1500
}

resource "google_compute_subnetwork" "subnets" {
  for_each = var.subnets

  name          = each.key
  project       = var.project_id
  network       = google_compute_network.vpc.id
  region        = each.value.region
  ip_cidr_range = each.value.cidr

  private_ip_google_access = true
  
  secondary_ip_range = [
    for range in each.value.secondary_ranges : {
      range_name    = range.name
      ip_cidr_range = range.cidr
    }
  ]

  log_config {
    aggregation_interval = "INTERVAL_5_SEC"
    flow_sampling       = 0.5
    metadata           = "INCLUDE_ALL_METADATA"
  }
}

# Cloud NAT
resource "google_compute_router" "router" {
  name    = "${var.environment}-router"
  project = var.project_id
  region  = var.region
  network = google_compute_network.vpc.id
}

resource "google_compute_router_nat" "nat" {
  name                               = "${var.environment}-nat"
  project                           = var.project_id
  router                            = google_compute_router.router.name
  region                            = var.region
  nat_ip_allocate_option            = "AUTO_ONLY"
  source_subnetwork_ip_ranges_to_nat = "ALL_SUBNETWORKS_ALL_IP_RANGES"

  log_config {
    enable = true
    filter = "ERRORS_ONLY"
  }
}
```

### Firewall Rules
```hcl
# firewall.tf
locals {
  common_tags = ["allow-internal", var.environment]
}

resource "google_compute_firewall" "internal" {
  name    = "${var.environment}-allow-internal"
  project = var.project_id
  network = google_compute_network.vpc.id

  allow {
    protocol = "icmp"
  }

  allow {
    protocol = "tcp"
    ports    = ["0-65535"]
  }

  allow {
    protocol = "udp"
    ports    = ["0-65535"]
  }

  source_ranges = [
    for subnet in google_compute_subnetwork.subnets : subnet.ip_cidr_range
  ]

  target_tags = local.common_tags
}

resource "google_compute_firewall" "iap" {
  name    = "${var.environment}-allow-iap"
  project = var.project_id
  network = google_compute_network.vpc.id

  allow {
    protocol = "tcp"
    ports    = ["22", "3389"]
  }

  source_ranges = ["35.235.240.0/20"]  # IAP range
  target_tags   = ["allow-iap"]
}
```

### GKE Cluster
```hcl
# gke.tf
resource "google_container_cluster" "primary" {
  name     = "${var.environment}-gke-cluster"
  project  = var.project_id
  location = var.region

  # Use regional cluster for high availability
  node_locations = [
    "${var.region}-a",
    "${var.region}-b",
    "${var.region}-c"
  ]

  network    = google_compute_network.vpc.id
  subnetwork = google_compute_subnetwork.subnets["gke"].id

  # Enable Autopilot or use standard cluster
  enable_autopilot = var.enable_autopilot

  dynamic "private_cluster_config" {
    for_each = var.private_cluster ? [1] : []
    content {
      enable_private_nodes    = true
      enable_private_endpoint = false
      master_ipv4_cidr_block = "172.16.0.0/28"
    }
  }

  ip_allocation_policy {
    cluster_secondary_range_name  = "pods"
    services_secondary_range_name = "services"
  }

  master_authorized_networks_config {
    dynamic "cidr_blocks" {
      for_each = var.authorized_networks
      content {
        cidr_block   = cidr_blocks.value.cidr
        display_name = cidr_blocks.value.name
      }
    }
  }

  maintenance_policy {
    recurring_window {
      start_time = "2022-01-01T00:00:00Z"
      end_time   = "2022-01-02T00:00:00Z"
      recurrence = "FREQ=WEEKLY;BYDAY=SA,SU"
    }
  }

  workload_identity_config {
    workload_pool = "${var.project_id}.svc.id.goog"
  }

  addons_config {
    http_load_balancing {
      disabled = false
    }
    horizontal_pod_autoscaling {
      disabled = false
    }
    network_policy_config {
      disabled = false
    }
    gcp_filestore_csi_driver_config {
      enabled = true
    }
  }
}

# Node Pool (if not using Autopilot)
resource "google_container_node_pool" "primary_nodes" {
  count = var.enable_autopilot ? 0 : 1

  name       = "${var.environment}-node-pool"
  project    = var.project_id
  location   = var.region
  cluster    = google_container_cluster.primary.name
  node_count = var.node_count

  autoscaling {
    min_node_count = var.min_nodes
    max_node_count = var.max_nodes
  }

  management {
    auto_repair  = true
    auto_upgrade = true
  }

  node_config {
    machine_type = var.machine_type
    disk_size_gb = var.disk_size_gb
    disk_type    = "pd-ssd"

    oauth_scopes = [
      "https://www.googleapis.com/auth/cloud-platform"
    ]

    labels = {
      environment = var.environment
    }

    taint {
      key    = "dedicated"
      value  = "special"
      effect = "NO_SCHEDULE"
    }

    workload_metadata_config {
      mode = "GKE_METADATA"
    }
  }
}
```

[Continue to Part 2 with Cloud SQL, Cloud Storage, Load Balancing, and more GCP services?]

Would you like me to continue with the next part covering more GCP services and their Terraform configurations?