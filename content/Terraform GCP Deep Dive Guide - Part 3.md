
## Monitoring, Logging, and Alerting

### Monitoring Setup
```hcl
# monitoring.tf
resource "google_monitoring_dashboard" "infrastructure" {
  dashboard_json = jsonencode({
    displayName = "${var.environment} Infrastructure Dashboard"
    gridLayout = {
      widgets = [
        {
          title = "CPU Usage by Instance"
          xyChart = {
            dataSets = [{
              timeSeriesQuery = {
                timeSeriesFilter = {
                  filter = "metric.type=\"compute.googleapis.com/instance/cpu/utilization\" resource.type=\"gce_instance\""
                  aggregation = {
                    alignmentPeriod = "60s"
                    crossSeriesReducer = "REDUCE_MEAN"
                    groupByFields = ["resource.label.instance_name"]
                  }
                }
              }
            }]
          }
        },
        {
          title = "Memory Usage"
          xyChart = {
            dataSets = [{
              timeSeriesQuery = {
                timeSeriesFilter = {
                  filter = "metric.type=\"compute.googleapis.com/instance/memory/utilization\""
                }
              }
            }]
          }
        }
      ]
    }
  })
}

# Alert Policies
resource "google_monitoring_alert_policy" "alert_policy" {
  for_each = var.alert_policies

  display_name = each.key
  project      = var.project_id
  combiner     = "OR"

  conditions {
    display_name = each.value.condition_display_name
    
    condition_threshold {
      filter          = each.value.filter
      duration        = each.value.duration
      comparison      = each.value.comparison
      threshold_value = each.value.threshold

      aggregations {
        alignment_period   = "60s"
        per_series_aligner = "ALIGN_MEAN"
      }
    }
  }

  notification_channels = [
    for channel in each.value.notification_channels :
    google_monitoring_notification_channel.channels[channel].name
  ]

  documentation {
    content   = each.value.documentation
    mime_type = "text/markdown"
  }
}

# Notification Channels
resource "google_monitoring_notification_channel" "channels" {
  for_each = var.notification_channels

  display_name = each.key
  project      = var.project_id
  type         = each.value.type

  labels = each.value.labels

  sensitive_labels {
    auth_token = each.value.auth_token
  }
}
```

## Secret Management and KMS

### Secret Manager
```hcl
# secrets.tf
resource "google_secret_manager_secret" "secrets" {
  for_each = var.secrets

  secret_id = each.key
  project   = var.project_id

  replication {
    automatic = true
  }

  labels = merge(var.common_labels, each.value.labels)
}

resource "google_secret_manager_secret_version" "versions" {
  for_each = var.secrets

  secret      = google_secret_manager_secret.secrets[each.key].id
  secret_data = each.value.data
}

resource "google_secret_manager_secret_iam_binding" "bindings" {
  for_each = var.secret_iam_bindings

  project   = var.project_id
  secret_id = google_secret_manager_secret.secrets[each.key].secret_id
  role      = each.value.role
  members   = each.value.members
}
```

### Cloud KMS
```hcl
# kms.tf
resource "google_kms_key_ring" "key_ring" {
  name     = "${var.environment}-keyring"
  project  = var.project_id
  location = var.region
}

resource "google_kms_crypto_key" "crypto_keys" {
  for_each = var.crypto_keys

  name            = each.key
  key_ring        = google_kms_key_ring.key_ring.id
  rotation_period = each.value.rotation_period

  version_template {
    algorithm        = each.value.algorithm
    protection_level = each.value.protection_level
  }

  lifecycle {
    prevent_destroy = true
  }
}

resource "google_kms_crypto_key_iam_binding" "crypto_key_binding" {
  for_each = var.crypto_key_iam_bindings

  crypto_key_id = google_kms_crypto_key.crypto_keys[each.key].id
  role          = each.value.role
  members       = each.value.members
}
```

## Identity and Access Management (IAM)

### Service Accounts and IAM
```hcl
# iam.tf
# Service Accounts
resource "google_service_account" "service_accounts" {
  for_each = var.service_accounts

  account_id   = each.key
  display_name = each.value.display_name
  project      = var.project_id
  description  = each.value.description
}

# Custom Roles
resource "google_project_iam_custom_role" "custom_roles" {
  for_each = var.custom_roles

  role_id     = each.key
  title       = each.value.title
  description = each.value.description
  permissions = each.value.permissions
  project     = var.project_id
}

# IAM Bindings
resource "google_project_iam_binding" "project_bindings" {
  for_each = var.project_iam_bindings

  project = var.project_id
  role    = each.value.role
  members = each.value.members

  condition {
    title       = each.value.condition.title
    description = each.value.condition.description
    expression  = each.value.condition.expression
  }
}

# Workload Identity
resource "google_service_account_iam_binding" "workload_identity" {
  for_each = var.workload_identity_bindings

  service_account_id = google_service_account.service_accounts[each.key].name
  role               = "roles/iam.workloadIdentityUser"
  members = [
    "serviceAccount:${var.project_id}.svc.id.goog[${each.value.namespace}/${each.value.k8s_service_account}]"
  ]
}
```

## Cloud Armor and Security

### Security Policies
```hcl
# security.tf
resource "google_compute_security_policy" "policy" {
  name    = "${var.environment}-security-policy"
  project = var.project_id

  # Default rule (deny all)
  rule {
    action   = "deny(403)"
    priority = "2147483647"
    match {
      versioned_expr = "SRC_IPS_V1"
      config {
        src_ip_ranges = ["*"]
      }
    }
    description = "Default deny rule"
  }

  # Allow specific IPs
  dynamic "rule" {
    for_each = var.allowed_ips
    content {
      action   = "allow"
      priority = 1000 + index(var.allowed_ips, rule.value)
      match {
        versioned_expr = "SRC_IPS_V1"
        config {
          src_ip_ranges = [rule.value]
        }
      }
      description = "Allow specific IP"
    }
  }

  # Custom rules
  dynamic "rule" {
    for_each = var.security_rules
    content {
      action      = rule.value.action
      priority    = rule.value.priority
      description = rule.value.description

      match {
        expr {
          expression = rule.value.expression
        }
      }

      rate_limit_options {
        conform_action = "allow"
        exceed_action  = "deny(429)"
        enforce_on_key = rule.value.rate_limit_key
        rate_limit_threshold {
          count        = rule.value.rate_limit_threshold
          interval_sec = rule.value.rate_limit_interval
        }
      }
    }
  }
}
```

## Advanced Networking

### Cloud Interconnect and VPN
```hcl
# networking_advanced.tf
# Cloud VPN
resource "google_compute_vpn_gateway" "gateway" {
  name    = "${var.environment}-vpn-gateway"
  project = var.project_id
  network = google_compute_network.vpc.id
  region  = var.region
}

resource "google_compute_vpn_tunnel" "tunnel" {
  for_each = var.vpn_tunnels

  name                    = each.key
  project                = var.project_id
  region                 = var.region
  peer_ip                = each.value.peer_ip
  shared_secret          = each.value.shared_secret
  target_vpn_gateway     = google_compute_vpn_gateway.gateway.id
  remote_traffic_selector = each.value.remote_traffic_selector
  local_traffic_selector  = each.value.local_traffic_selector

  depends_on = [
    google_compute_forwarding_rule.fr_esp,
    google_compute_forwarding_rule.fr_udp500,
    google_compute_forwarding_rule.fr_udp4500
  ]
}

# Dedicated Interconnect
resource "google_compute_interconnect_attachment" "attachment" {
  name                     = "${var.environment}-interconnect"
  project                 = var.project_id
  region                  = var.region
  type                    = "DEDICATED"
  router                  = google_compute_router.router.id
  bandwidth               = "BPS_10G"
  admin_enabled           = true
  interconnect           = var.interconnect_id
  candidate_subnets      = var.candidate_subnets
  vlan_tag8021q          = var.vlan_tag
}
```

[Continue to Part 4 with CI/CD, Cloud Build, and more advanced configurations?]

Would you like me to continue with the next part covering CI/CD, Cloud Build, and more advanced GCP configurations?