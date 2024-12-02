
```bash
gcloud sql --help
gcloud sql instances --help
gcloud sql instances create --help
```

Cloud SQL permissions

For a list of roles and their associated permissions, see [Cloud SQL roles](https://cloud.google.com/sql/docs/mysql/iam-roles).

|Task|Required additional permissions|
|---|---|
|Displaying the instance listing page|`cloudsql.instances.list`  <br>`resourcemanager.projects.get`|
|Creating an instance|`cloudsql.instances.create`  <br>`cloudsql.instances.get`  <br>`cloudsql.instances.list`  <br>`resourcemanager.projects.get`|
|Connecting to an instance from the Cloud Shell|`cloudsql.instances.get`  <br>`cloudsql.instances.list`  <br>`cloudsql.instances.update`  <br>`resourcemanager.projects.get`|
|Creating a user|`cloudsql.instances.get`  <br>`cloudsql.instances.list`  <br>`cloudsql.users.create`  <br>`cloudsql.users.list`  <br>`resourcemanager.projects.get`|
|Viewing instance information|`cloudsql.databases.list`  <br>`cloudsql.instances.get`  <br>`cloudsql.instances.list`  <br>`cloudsql.users.list`  <br>`monitoring.timeSeries.list`  <br>`resourcemanager.projects.get`|
|Viewing instance metadata in Dataplex Catalog|`cloudsql.schemas.view`|
## Required permissions for gcloud sql commands

| Command                                 | Required permissions                                              |
| --------------------------------------- | ----------------------------------------------------------------- |
| `gcloud sql backups create`             | `cloudsql.backupRuns.create`                                      |
| `gcloud sql backups delete`             | `cloudsql.backupRuns.delete`                                      |
| `gcloud sql backups describe`           | `cloudsql.backupRuns.get`                                         |
| `gcloud sql backups list`               | `cloudsql.backupRuns.list`                                        |
| `gcloud sql backups restore`            | `cloudsql.backupRuns.get`  <br>`cloudsql.instances.restoreBackup` |
| `gcloud sql connect`                    | `cloudsql.instances.get`  <br>`cloudsql.instances.update`         |
| `gcloud sql databases create`           | `cloudsql.databases.create`                                       |
| `gcloud sql databases delete`           | `cloudsql.databases.delete`                                       |
| `gcloud sql databases describe`         | `cloudsql.databases.get`                                          |
| `gcloud sql databases list`             | `cloudsql.databases.list`                                         |
| `gcloud sql databases patch`            | `cloudsql.databases.get`  <br>`cloudsql.databases.update`         |
| `gcloud sql export`                     | `cloudsql.instances.export`  <br>`cloudsql.instances.get`         |
| `gcloud sql flags list`                 | None                                                              |
| `gcloud sql import`                     | `cloudsql.instances.import`                                       |
| `gcloud sql instances clone`            | `cloudsql.instances.clone`                                        |
| `gcloud sql instances create`           | `cloudsql.instances.create`                                       |
| `gcloud sql instances delete`           | `cloudsql.instances.delete`                                       |
| `gcloud sql instances describe`         | `cloudsql.instances.get`                                          |
| `gcloud sql instances failover`         | `cloudsql.instances.failover`                                     |
| `gcloud sql instances import`           | `cloudsql.instances.import`                                       |
| `gcloud sql instances list`             | `cloudsql.instances.list`                                         |
| `gcloud sql instances patch`            | `cloudsql.instances.get`  <br>`cloudsql.instances.update`         |
| `gcloud sql instances promote-replica`  | `cloudsql.instances.promoteReplica`                               |
| `gcloud sql instances reset-ssl-config` | `cloudsql.instances.resetSslConfig`                               |
| `gcloud sql instances restart`          | `cloudsql.instances.restart`                                      |
| `gcloud sql instances restore-backup`   | `cloudsql.backupRuns.get`  <br>`cloudsql.instances.restoreBackup` |
| `gcloud sql operations describe`        | `cloudsql.instances.get`                                          |
| `gcloud sql operations list`            | `cloudsql.instances.get`                                          |
| `gcloud sql operations wait`            | `cloudsql.instances.get`                                          |
| `gcloud sql ssl client-certs create`    | `cloudsql.sslCerts.create`                                        |
| `gcloud sql ssl client-certs delete`    | `cloudsql.sslCerts.delete`                                        |
| `gcloud sql ssl client-certs describe`  | `cloudsql.sslCerts.list`                                          |
| `gcloud sql ssl client-certs list`      | `cloudsql.sslCerts.list`                                          |
| `gcloud sql tiers list`                 | None                                                              |
| `gcloud sql users create`               | `cloudsql.users.create`                                           |
| `gcloud sql users delete`               | `cloudsql.users.delete`                                           |
| `gcloud sql users list`                 | `cloudsql.users.list`                                             |
| `gcloud sql users set-password`         | `cloudsql.users.update`                                           |
