## Machine Learning and AI Platform

### AI Platform (Vertex AI) Configuration
```hcl
# vertexai.tf
resource "google_vertex_ai_dataset" "dataset" {
  for_each = var.ml_datasets

  display_name = "${var.environment}-${each.key}"
  project      = var.project_id
  region       = var.region
  
  metadata_schema_uri = each.value.schema_uri
  
  dynamic "encryption_spec" {
    for_each = var.kms_key_name != null ? [1] : []
    content {
      kms_key_name = var.kms_key_name
    }
  }

  labels = var.labels
}

resource "google_vertex_ai_endpoint" "endpoint" {
  for_each = var.ml_endpoints

  display_name = "${var.environment}-${each.key}"
  project      = var.project_id
  region       = var.region
  
  network      = google_compute_network.vpc.id

  dynamic "encryption_spec" {
    for_each = var.kms_key_name != null ? [1] : []
    content {
      kms_key_name = var.kms_key_name
    }
  }
}

resource "google_vertex_ai_model" "model" {
  for_each = var.ml_models

  display_name = "${var.environment}-${each.key}"
  project      = var.project_id
  region       = var.region
  
  artifact_uri = each.value.artifact_uri
  container_spec {
    image_uri = each.value.container_image
    command   = each.value.command
    args      = each.value.args
    env {
      name  = "MODEL_NAME"
      value = each.key
    }
  }

  dynamic "encryption_spec" {
    for_each = var.kms_key_name != null ? [1] : []
    content {
      kms_key_name = var.kms_key_name
    }
  }
}
```

### AI Platform Pipeline
```hcl
# ml_pipeline.tf
resource "google_vertex_ai_pipeline_job" "pipeline" {
  for_each = var.ml_pipelines

  display_name = "${var.environment}-${each.key}"
  project      = var.project_id
  location     = var.region

  pipeline_spec {
    pipeline_manifest_uri = each.value.manifest_uri
    parameters = merge(
      each.value.parameters,
      {
        project_id = var.project_id
        region     = var.region
      }
    )
  }

  network = google_compute_network.vpc.id
  service_account = google_service_account.pipeline_sa.email

  scheduling {
    restart_job_on_worker_restart = true
  }

  labels = var.labels
}

resource "google_vertex_ai_tensorboard" "tensorboard" {
  display_name = "${var.environment}-tensorboard"
  project      = var.project_id
  region       = var.region

  dynamic "encryption_spec" {
    for_each = var.kms_key_name != null ? [1] : []
    content {
      kms_key_name = var.kms_key_name
    }
  }
}
```

### AutoML Configuration
```hcl
# automl.tf
resource "google_vertex_ai_dataset" "automl_dataset" {
  display_name          = "${var.environment}-automl-dataset"
  project              = var.project_id
  region               = var.region
  metadata_schema_uri  = "gs://google-cloud-aiplatform/schema/dataset/metadata/image_1.0.0.yaml"
}

resource "google_vertex_ai_training_job" "automl_job" {
  display_name = "${var.environment}-automl-job"
  project      = var.project_id
  region       = var.region

  input_data_config {
    dataset_id = google_vertex_ai_dataset.automl_dataset.id
    gcs_source = var.training_data_path
  }

  training_pipeline_spec {
    component_id = "automl-vision"
    parameters = {
      target_column      = "label"
      prediction_type    = "classification"
      budget_milli_node_hours = 8000
      model_display_name = "${var.environment}-automl-model"
    }
  }
}
```

## AI Services Integration

### Cloud Vision API
```hcl
# vision.tf
resource "google_project_service" "vision_api" {
  project = var.project_id
  service = "vision.googleapis.com"
}

resource "google_storage_bucket" "vision_output" {
  name     = "${var.project_id}-vision-output"
  project  = var.project_id
  location = var.region

  uniform_bucket_level_access = true
  
  lifecycle_rule {
    condition {
      age = 30
    }
    action {
      type = "Delete"
    }
  }
}

# Cloud Function for Vision API Processing
resource "google_cloudfunctions_function" "vision_processor" {
  name        = "${var.environment}-vision-processor"
  project     = var.project_id
  region      = var.region
  description = "Process images using Vision API"

  runtime = "python39"

  available_memory_mb   = 256
  source_archive_bucket = google_storage_bucket.function_source.name
  source_archive_object = google_storage_bucket_object.vision_function_zip.name
  
  entry_point = "process_image"
  
  event_trigger {
    event_type = "google.storage.object.finalize"
    resource   = google_storage_bucket.vision_input.name
  }

  environment_variables = {
    OUTPUT_BUCKET = google_storage_bucket.vision_output.name
  }

  service_account_email = google_service_account.vision_sa.email
}
```

### Speech-to-Text Configuration
```hcl
# speech.tf
resource "google_project_service" "speech_api" {
  project = var.project_id
  service = "speech.googleapis.com"
}

resource "google_speech_recognition_custom_class" "custom_class" {
  project      = var.project_id
  display_name = "${var.environment}-custom-class"
  
  items {
    value = "custom_word_1"
    phrases = ["phrase1", "phrase2"]
  }
}

resource "google_speech_recognition_phrase_set" "phrase_set" {
  project      = var.project_id
  display_name = "${var.environment}-phrase-set"
  
  phrases {
    value = "common phrase 1"
    boost = 10.0
  }
}
```

### Natural Language API
```hcl
# language.tf
resource "google_project_service" "language_api" {
  project = var.project_id
  service = "language.googleapis.com"
}

# Cloud Function for NL API Processing
resource "google_cloudfunctions_function" "nl_processor" {
  name        = "${var.environment}-nl-processor"
  project     = var.project_id
  region      = var.region
  description = "Process text using Natural Language API"

  runtime = "python39"

  available_memory_mb   = 256
  source_archive_bucket = google_storage_bucket.function_source.name
  source_archive_object = google_storage_bucket_object.nl_function_zip.name
  
  entry_point = "process_text"
  
  event_trigger {
    event_type = "google.pubsub.topic.publish"
    resource   = google_pubsub_topic.nl_input.id
  }

  environment_variables = {
    OUTPUT_TOPIC = google_pubsub_topic.nl_output.id
  }

  service_account_email = google_service_account.nl_sa.email
}
```

### Translation API Setup
```hcl
# translation.tf
resource "google_project_service" "translate_api" {
  project = var.project_id
  service = "translate.googleapis.com"
}

resource "google_cloud_run_service" "translator" {
  name     = "${var.environment}-translator"
  project  = var.project_id
  location = var.region

  template {
    spec {
      containers {
        image = "gcr.io/${var.project_id}/translator:latest"
        
        env {
          name  = "PROJECT_ID"
          value = var.project_id
        }
        
        resources {
          limits = {
            cpu    = "1000m"
            memory = "512Mi"
          }
        }
      }
      
      service_account_name = google_service_account.translator_sa.email
    }
  }

  traffic {
    percent         = 100
    latest_revision = true
  }
}
```

[Continue to Part 8 with Security, Identity, and more advanced configurations?]

Would you like me to continue with the next part covering Security, Identity, and more advanced GCP configurations?