
## Testing and Validation

### Terratest Setup
```go
// test/main_test.go
package test

import (
    "testing"
    "github.com/gruntwork-io/terratest/modules/gcp"
    "github.com/gruntwork-io/terratest/modules/terraform"
    "github.com/stretchr/testify/assert"
)

func TestTerraformBasicExample(t *testing.T) {
    t.Parallel()

    terraformOptions := &terraform.Options{
        TerraformDir: "../examples/basic",
        Vars: map[string]interface{}{
            "project_id": "test-project",
            "region":     "us-central1",
        },
        EnvVars: map[string]string{
            "GOOGLE_CREDENTIALS": "${file("credentials.json")}",
        },
    }

    defer terraform.Destroy(t, terraformOptions)
    terraform.InitAndApply(t, terraformOptions)

    instanceID := terraform.Output(t, terraformOptions, "instance_id")
    assert.NotEmpty(t, instanceID)
}
```

### Unit Testing with tftest
```hcl
# test/variables_test.go
func TestVariableValidation(t *testing.T) {
    cases := []struct {
        name          string
        variableValue interface{}
        expectError   bool
    }{
        {
            name:          "valid_instance_type",
            variableValue: "n1-standard-1",
            expectError:   false,
        },
        {
            name:          "invalid_instance_type",
            variableValue: "invalid-type",
            expectError:   true,
        },
    }

    for _, tc := range cases {
        t.Run(tc.name, func(t *testing.T) {
            _, errs := terraform.ValidateVarValue(
                "instance_type",
                tc.variableValue,
            )
            hasError := len(errs) > 0
            assert.Equal(t, tc.expectError, hasError)
        })
    }
}
```

## CI/CD Integration

### GitHub Actions Workflow
```yaml
# .github/workflows/terraform.yml
name: 'Terraform CI/CD'

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  terraform:
    name: 'Terraform'
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v2

    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v1
      with:
        terraform_version: 1.0.0

    - name: Terraform Format
      run: terraform fmt -check -recursive

    - name: Terraform Init
      run: |
        terraform init \
          -backend-config="bucket=${{ secrets.TF_STATE_BUCKET }}" \
          -backend-config="prefix=terraform/state"

    - name: Terraform Validate
      run: terraform validate

    - name: Terraform Plan
      if: github.event_name == 'pull_request'
      run: |
        terraform plan \
          -var="project_id=${{ secrets.GCP_PROJECT_ID }}" \
          -var="credentials=${{ secrets.GCP_CREDENTIALS }}" \
          -out=tfplan
      
    - name: Terraform Apply
      if: github.ref == 'refs/heads/main' && github.event_name == 'push'
      run: terraform apply -auto-approve tfplan
```

### Advanced Module Patterns

### Composite Module Pattern
```hcl
# modules/application/main.tf
module "networking" {
  source = "../networking"

  project_id = var.project_id
  region     = var.region
  vpc_config = var.vpc_config
}

module "compute" {
  source = "../compute"

  depends_on = [module.networking]

  project_id = var.project_id
  network_id = module.networking.vpc_id
  subnet_id  = module.networking.subnet_ids["main"]
}

module "load_balancer" {
  source = "../load_balancer"

  depends_on = [module.compute]

  project_id     = var.project_id
  instance_group = module.compute.instance_group
  ssl_cert       = var.ssl_certificate
}
```

### Factory Pattern Module
```hcl
# modules/instance_factory/main.tf
locals {
  instance_configs = {
    for instance in var.instances : instance.name => merge(
      var.default_config,
      instance
    )
  }
}

resource "google_compute_instance" "instances" {
  for_each = local.instance_configs

  name         = each.key
  machine_type = each.value.machine_type
  zone         = each.value.zone

  boot_disk {
    initialize_params {
      image = each.value.image
      size  = each.value.disk_size
    }
  }

  network_interface {
    network    = each.value.network
    subnetwork = each.value.subnetwork
  }

  tags = concat(
    var.common_tags,
    each.value.additional_tags
  )

  metadata = merge(
    var.common_metadata,
    each.value.additional_metadata
  )
}
```

### Resource Wrapper Pattern
```hcl
# modules/gke_cluster_wrapper/main.tf
resource "google_container_cluster" "cluster" {
  name     = var.cluster_name
  location = var.region

  # Remove default node pool
  remove_default_node_pool = true
  initial_node_count       = 1

  dynamic "private_cluster_config" {
    for_each = var.private_cluster ? [1] : []
    content {
      enable_private_nodes    = true
      enable_private_endpoint = var.private_endpoint
      master_ipv4_cidr_block = var.master_ipv4_cidr
    }
  }

  dynamic "master_authorized_networks_config" {
    for_each = length(var.authorized_networks) > 0 ? [1] : []
    content {
      dynamic "cidr_blocks" {
        for_each = var.authorized_networks
        content {
          cidr_block   = cidr_blocks.value.cidr
          display_name = cidr_blocks.value.name
        }
      }
    }
  }

  addons_config {
    http_load_balancing {
      disabled = !var.enable_http_load_balancing
    }
    horizontal_pod_autoscaling {
      disabled = !var.enable_horizontal_pod_autoscaling
    }
  }
}
```

### Monitoring and Alerting
```hcl
# monitoring.tf
resource "google_monitoring_dashboard" "dashboard" {
  dashboard_json = jsonencode({
    displayName = "Infrastructure Dashboard"
    gridLayout = {
      widgets = [
        {
          title = "CPU Usage"
          xyChart = {
            dataSets = [{
              timeSeriesQuery = {
                timeSeriesFilter = {
                  filter = "metric.type=\"compute.googleapis.com/instance/cpu/utilization\""
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

resource "google_monitoring_alert_policy" "alert_policy" {
  display_name = "High CPU Usage Alert"
  combiner     = "OR"
  conditions {
    display_name = "CPU Usage > 80%"
    condition_threshold {
      filter     = "metric.type=\"compute.googleapis.com/instance/cpu/utilization\""
      duration   = "300s"
      comparison = "COMPARISON_GT"
      threshold_value = 0.8
    }
  }

  notification_channels = [
    google_monitoring_notification_channel.email.name
  ]
}
```

Would you like me to continue with more advanced topics such as custom providers, workspaces management, or specific GCP service configurations?