
## Download the source public data file

1. Download the [baby names zip file](https://www.ssa.gov/OACT/babynames/names.zip).
2. Extract the zip file. It contains a file named `NationalReadMe.pdf` that describes the dataset schema. [Learn more about the baby names dataset](http://www.ssa.gov/OACT/babynames/background.html).
3. Open the `yob2010.txt` file. It's a comma-separated value (CSV) file that contains three columns: name, assigned sex at birth, and number of children with that name. The file has no header row.
4. Move the file to your working directory.
    - If you're working in Cloud Shell, click more_vert **More** > **Upload**, click **Choose Files**, choose the`yob2010.txt` file, and then click **Upload**.
    - If you're working in a local shell, copy or move the file `yob2010.txt` into the directory where you're running

## Create a dataset

1. Create a dataset named `babynames`:

```
bq mk babynames
```

> [!NOTE]
> A dataset name can be up to 1,024 characters long and consist of A-Z, a-z, 0-9, and the underscore. The name cannot start with a number or underscore, and it cannot have spaces.
> 

2. Confirm that the dataset `babynames` now appears in your project:

```bash
bq ls
```

## Load data into a table

In the `babynames` dataset, load the source file `yob2010.txt` into a new table that's named `names2010`:

```bash
bq load babynames.names2010 yob2010.txt name:string,assigned_sex_at_birth:string,count:integer
```

> [!NOTE]
> By default, when you load data, BigQuery expects UTF-8 encoded data. If you have data in ISO-8859-1 (or Latin-1) encoding and you have problems with it, instruct BigQuery to treat your data as Latin-1 using `bq load -E=ISO-8859-1`. For more information, see [Encoding](https://cloud.google.com/bigquery/docs/loading-data-cloud-storage-csv#encoding).

Confirm that the table `names2010` now appears in the `babynames` dataset:

```
bq ls babynames
```

Confirm that the table schema of your new `names2010` table is `name: string`, `assigned_sex_at_birth: string`, and `count: integer`:

```
bq show babynames.names2010
```

The output is similar to the following. Some columns are omitted to simplify the output.

```
  Last modified        Schema                      Total Rows   Total Bytes
----------------- ------------------------------- ------------ ------------
14 Mar 17:16:45   |- name: string                    34089       654791
                  |- assigned_sex_at_birth: string
                  |- count: integer
```

## Query table data

Determine the most popular girls' names in the data:

```
bq query --use_legacy_sql=false \    'SELECT      name,      count    FROM      `babynames.names2010`    WHERE      assigned_sex_at_birth = "F"    ORDER BY      count DESC    LIMIT 5;'
```

The output is similar to the following:

```
+----------+-------+
|   name   | count |
+----------+-------+
| Isabella | 22925 |
| Sophia   | 20648 |
| Emma     | 17354 |
| Olivia   | 17030 |
| Ava      | 15436 |
+----------+-------+
```

Determine the least popular boys' names in the data:

```
bq query --use_legacy_sql=false \    'SELECT      name,      count    FROM      `babynames.names2010`    WHERE      assigned_sex_at_birth = "M"    ORDER BY      count ASC    LIMIT 5;'
```

The output is similar to the following:

```
+----------+-------+
|   name   | count |
+----------+-------+
| Aamarion |     5 |
| Aarian   |     5 |
| Aaqib    |     5 |
| Aaidan   |     5 |
| Aadhavan |     5 |
+----------+-------+
```

The minimum count is 5 because the source data omits names with fewer than 5 occurrences.


## Clean up

Delete the `babynames` dataset:

```
bq rm --recursive=true babynames
```

The `--recursive` flag deletes all tables in the dataset, including the `names2010` table.