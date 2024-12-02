## Create the job


To  [create the job](https://cloud.google.com/run/docs/create-jobs), you can use either the Cloud Console or the gcloud CLI using the following command:

```
gcloud beta run jobs create JOB_NAME --image IMAGE_URL --region europe-west9 OPTIONS
```

Replace:

- **`JOB_NAME`** with the name of the job you want to create.
- **`IMAGE_URL`** with a reference to the container image.
- Optionally, replace **`OPTIONS`** with any of the available flags. For a complete list of flags, run **`gcloud beta run jobs create --help`**.

Example flags include:

- **`--tasks`** for the number of tasks to run.
- **`-–max retries`** for the number of times a failed task is retried.
- **`--parallelism`** for the maximum number of tasks that can run in parallel.
- **`--execute-now`** to execute the job immediately after it is created.
- **`--async`** to exit the job immediately after creating a new execution.

You can also use the usual Cloud Run features to secure your Cloud Run job and connect it to the rest of your Google Cloud Platform (GCP) environment.

## Execute the job

To  [execute the job](https://cloud.google.com/run/docs/execute/jobs), you can use either the Cloud Console or the gcloud CLI using the following command:

- In the  [Cloud Console](https://console.cloud.google.com/run/jobs), click on the job name, and click **Execute** near the top of the page.
- In the gcloud CLI, use the following command:

```
gcloud beta run jobs execute JOB_NAME --region europe-west9 EXECUTION_OPTIONS
```

Replace **`JOB_NAME`** with the name of the job. Optionally, replace **`EXECUTION_OPTIONS`** to specify:

- **Immediate** job execution after you create the job.

```
gcloud beta run jobs create JOB_NAME --region europe-west9 --execute-now
```

- If you want to **wait** until the execution is complete before exiting.

```
gcloud beta run jobs create JOB_NAME --region europe-west9 --wait
```

- If you want to **exit immediately** after creating a new execution.

```
gcloud beta run jobs create JOB_NAME --region europe-west9 --async
```

## [6. Execute the job on a schedule](https://codelabs.developers.google.com/codelabs/codelabs/cloud-run-jobs#5)

If you want to  [execute your job on a schedule](https://cloud.google.com/run/docs/execute/jobs-on-schedule), use Cloud Scheduler.

For example, you may want to create and send invoices at regular intervals, or save the results of a database query as XML and upload the file every few hours.

With Cloud Scheduler, you can schedule the job and manage all your automation tasks from one place.

- Run your batch and big data jobs at the same time each week, day, or hour with guaranteed execution and retries in case of failures.
- Automate many of the tedious tasks associated with running cloud infrastructure in a reliable and fully managed manner.
- Automate virtually anything.
- View and manage all your jobs from a single UI or command-line interface.

Once you  [set up your environment](https://cloud.google.com/scheduler/docs/setup) to enable your project to use Cloud Scheduler, you create a Cloud Run job, then define a schedule by entering the name, region, description, frequency, and timezone. Cloud Scheduler then executes the Cloud Run job at the frequency you specify.

## [7. View the job execution status](https://codelabs.developers.google.com/codelabs/codelabs/cloud-run-jobs#6)

Once your job executes, you can view logs in  [Cloud Logging](https://cloud.google.com/run/docs/logging) logs and monitoring data in  [Cloud Monitoring](https://cloud.google.com/run/docs/monitoring).

To view logs, you can:

- Use the Cloud Run page in the Cloud Console.
- Use Cloud Logging Logs Explorer in the Cloud Console.

Both of these viewing methods examine the same logs stored in  [Cloud Logging](https://cloud.google.com/logging/docs), but the Cloud Logging Logs Explorer provides more details and more filtering capabilities.

[Cloud Monitoring](https://cloud.google.com/monitoring/docs) provides Cloud Run performance monitoring and  [metrics](https://cloud.google.com/monitoring/api/metrics_gcp#gcp-run), along with  [alerts](https://cloud.google.com/monitoring/alerts) to send notifications when certain metric thresholds are exceeded. Cloud Run is automatically integrated with Cloud Monitoring with no setup or configuration required. This means that metrics of your Cloud Run jobs are captured automatically when they are running.

You can view metrics either in Cloud Monitoring or in the Cloud Run page in the console. Cloud Monitoring provides more charting and filtering options.

## View job executions in your projec

To list all of the job executions for all jobs in your project:

```
gcloud run jobs executions list
```

To list only the executions for a specific job:

```
gcloud run jobs executions list --job JOB_NAME
```

Replace `JOB_NAME` with the name of the job you are filtering on.

For other ways to refine the returned list, including the use of filters, see [jobs executions list](https://cloud.google.com/sdk/gcloud/reference/beta/run/jobs/executions/list).

To get the name of the latest execution for a specific job, use the [`--format` flag](https://cloud.google.com/sdk/gcloud/reference#--format):

```
gcloud run jobs describe JOB_NAME --format="value(status.latestCreatedExecution.name)"
```

Replace `JOB_NAME` with the name of the job you are filtering on.

## View job execution details

To view details about a job execution:

Use the command:

```
gcloud run jobs executions describe EXECUTION_NAME
```

Replace `EXECUTION_NAME` with the name of the execution.

You can use the [`--format` flag](https://cloud.google.com/sdk/gcloud/reference#--format) to format the output and to get additional information. For example as YAML:

```
gcloud run jobs executions describe EXECUTION_NAME --format yaml
```

## Delete a job execution

To delete a job execution:
1. Use the command:
	```
	gcloud run jobs executions delete EXECUTION_NAME
	```

	Replace `EXECUTION_NAME` with the name of the execution.

2. If prompted to confirm, respond `y`. Upon success, a success message will be displayed.