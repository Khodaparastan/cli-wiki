
## Security and Identity Management

### Organization Policies
```hcl
# org_policies.tf
resource "google_organization_policy" "policies" {
  for_each = var.org_policies

  org_id     = var.organization_id
  constraint = each.key

  dynamic "boolean_policy" {
    for_each = each.value.type == "boolean" ? [1] : []
    content {
      enforced = each.value.enforced
    }
  }

  dynamic "list_policy" {
    for_each = each.value.type == "list" ? [1] : []
    content {
      inherit_from_parent = each.value.inherit_from_parent
      suggested_value     = each.value.suggested_value
      
      dynamic "allow" {
        for_each = each.value.allow_list != null ? [1] : []
        content {
          values = each.value.allow_list
        }
      }
      
      dynamic "deny" {
        for_each = each.value.deny_list != null ? [1] : []
        content {
          values = each.value.deny_list
        }
      }
    }
  }
}

# Example policies configuration
locals {
  org_policies = {
    "constraints/compute.disableSerialPortAccess" = {
      type     = "boolean"
      enforced = true
    }
    "constraints/compute.vmExternalIpAccess" = {
      type      = "list"
      deny_list = ["*"]
    }
    "constraints/iam.allowedPolicyMemberDomains" = {
      type       = "list"
      allow_list = [var.allowed_domain]
    }
  }
}
```

### Identity-Aware Proxy (IAP)
```hcl
# iap.tf
resource "google_iap_brand" "brand" {
  project = var.project_id
  support_email     = var.support_email
  application_title = "${var.environment} Application"
}

resource "google_iap_client" "client" {
  display_name = "${var.environment}-client"
  brand        = google_iap_brand.brand.name
}

resource "google_iap_tunnel_instance_iam_binding" "tunnel_iam" {
  project = var.project_id
  zone    = var.zone
  instance = google_compute_instance.instance.name
  role     = "roles/iap.tunnelResourceAccessor"
  members  = var.tunnel_access_members
}

resource "google_iap_web_backend_service_iam_binding" "backend_iam" {
  project = var.project_id
  web_backend_service = google_compute_backend_service.backend.name
  role     = "roles/iap.httpsResourceAccessor"
  members  = var.web_access_members
}
```

### VPC Service Controls
```hcl
# vpc_service_controls.tf
resource "google_access_context_manager_service_perimeter" "perimeter" {
  parent = "accessPolicies/${google_access_context_manager_access_policy.policy.name}"
  name   = "accessPolicies/${google_access_context_manager_access_policy.policy.name}/servicePerimeters/${var.perimeter_name}"
  title  = "${var.environment} Service Perimeter"
  
  status {
    restricted_services = [
      "bigquery.googleapis.com",
      "storage.googleapis.com",
      "cloudfunctions.googleapis.com"
    ]

    resources = [
      "projects/${var.project_number}"
    ]

    access_levels = [
      google_access_context_manager_access_level.access_level.name
    ]

    vpc_accessible_services {
      enable_restriction = true
      allowed_services  = ["RESTRICTED-SERVICES"]
    }

    ingress_policies {
      ingress_from {
        identities = [
          "serviceAccount:${google_service_account.allowed_sa.email}"
        ]
        sources {
          resource = "projects/${var.allowed_project_number}"
        }
      }
      ingress_to {
        resources = ["*"]
        operations {
          service_name = "storage.googleapis.com"
          method_selectors {
            method = "google.storage.objects.get"
          }
        }
      }
    }
  }
}

resource "google_access_context_manager_access_level" "access_level" {
  parent = "accessPolicies/${google_access_context_manager_access_policy.policy.name}"
  name   = "accessPolicies/${google_access_context_manager_access_policy.policy.name}/accessLevels/${var.access_level_name}"
  title  = "${var.environment} Access Level"
  
  basic {
    conditions {
      ip_subnetworks = var.allowed_ip_ranges
      
      required_access_levels = [
        google_access_context_manager_access_level.prerequisite_level.name
      ]

      members = [
        "user:${var.allowed_user}",
        "serviceAccount:${var.allowed_service_account}"
      ]

      regions = [
        "US",
        "EU"
      ]

      device_policy {
        require_screen_lock = true
        require_admin_approval = true
        require_corp_owned = true
      }
    }
  }
}
```

### Security Command Center
```hcl
# security_center.tf
resource "google_scc_source" "custom_source" {
  provider = google-beta
  
  display_name = "${var.environment}-security-source"
  organization = var.organization_id
  description  = "Custom security source for ${var.environment}"
}

resource "google_scc_notification_config" "notification" {
  provider     = google-beta
  config_id    = "${var.environment}-scc-notification"
  organization = var.organization_id
  description  = "Security Command Center notification config"
  
  pubsub_topic = google_pubsub_topic.scc_notifications.id
  
  streaming_config {
    filter = "category=\"OPEN_FIREWALL\""
  }
}

resource "google_scc_source_iam_binding" "binding" {
  provider = google-beta
  source  = google_scc_source.custom_source.name
  role    = "roles/securitycenter.sourcesAdmin"
  members = var.scc_admin_members
}
```

### Binary Authorization
```hcl
# binary_authorization.tf
resource "google_binary_authorization_policy" "policy" {
  project = var.project_id

  global_policy_evaluation_mode = "ENABLE"
  
  admission_whitelist_patterns {
    name_pattern = "gcr.io/${var.project_id}/*"
  }

  default_admission_rule {
    evaluation_mode  = "ALWAYS_DENY"
    enforcement_mode = "ENFORCED_BLOCK_AND_AUDIT_LOG"
  }

  cluster_admission_rules {
    cluster         = "${var.region}.${google_container_cluster.cluster.name}"
    evaluation_mode = "REQUIRE_ATTESTATION"
    enforcement_mode = "ENFORCED_BLOCK_AND_AUDIT_LOG"
    
    require_attestations_by = [
      google_binary_authorization_attestor.attestor.name
    ]
  }
}

resource "google_binary_authorization_attestor" "attestor" {
  name = "${var.environment}-attestor"
  project = var.project_id
  
  attestation_authority_note {
    note_reference = google_container_analysis_note.note.name
    
    public_keys {
      ascii_armored_pgp_public_key = file(var.pgp_public_key_file)
      comment                      = "Attestor public key"
    }
  }
}

resource "google_container_analysis_note" "note" {
  name = "${var.environment}-attestor-note"
  project = var.project_id
  
  attestation_authority {
    hint {
      human_readable_name = "Binary Authorization Attestor"
    }
  }
}
```

[Continue to Part 9 with Compliance, Audit Logging, and more advanced configurations?]

Would you like me to continue with the next part covering Compliance, Audit Logging, and more advanced GCP configurations?