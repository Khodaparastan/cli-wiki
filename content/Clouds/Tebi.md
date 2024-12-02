
## AWS CLI

# AWS CLI[¶](https://docs.tebi.io/software_examples/aws_cli.html#aws-cli "Permalink to this heading")

The AWS Command Line Interface (CLI) is a unified tool to manage AWS services.

The AWS CLI can also be used to work with other S3-compatible object storage systems, including Tebi.

## Installation[¶](https://docs.tebi.io/software_examples/aws_cli.html#installation "Permalink to this heading")

You can download the latest version of `aws` CLI from the official site [download page](https://aws.amazon.com/cli/).

## Configuring AWS CLI[¶](https://docs.tebi.io/software_examples/aws_cli.html#configuring-aws-cli "Permalink to this heading")

For general use, the `aws configure` command is the fastest way to set up your AWS CLI installation.

**Configuration Process Example:**

1$ aws configure --profile tebi
2AWS Access Key ID [None]:  YOUR_KEY
3AWS Secret Access Key [None]: YOUR_SECRET
4Default region name [None]:
5Default output format [None]:

## Running AWS CLI[¶](https://docs.tebi.io/software_examples/aws_cli.html#running-aws-cli "Permalink to this heading")

Note

Add the `--endpoint-url https://s3.tebi.io` command line parameter to use the AWS CLI tool with Tebi.

**See all buckets**

aws --endpoint-url https://s3.tebi.io --profile tebi s3 ls

**Make a new bucket**

aws --endpoint-url https://s3.tebi.io --profile tebi s3 mb BUCKET_NAME

**List the contents of a bucket**

aws --endpoint-url https://s3.tebi.io --profile tebi s3 ls BUCKET_NAME

Sync `/home/local/directory` to the remote `BUCKET_NAME`, deleting any excess files in the bucket.

aws --endpoint-url https://s3.tebi.io --profile tebi s3 sync /home/local/directory BUCKET_NAME


## s3cmd

# S3Cmd[¶](https://docs.tebi.io/software_examples/s3cmd.html#s3cmd "Permalink to this heading")

S3cmd is a free command line tool and client used for uploading, retrieving, and managing data in cloud storage service providers that use the S3 protocol.

It is best suited for power users who are familiar with command line programs. It is also ideal for batch scripts and automated backup to S3, triggered from cron, etc.

## Installation[¶](https://docs.tebi.io/software_examples/s3cmd.html#installation "Permalink to this heading")

You can download the latest version of the `s3cmd` tool from the [official site download page](https://s3tools.org/download).

## Configuring S3Cmd[¶](https://docs.tebi.io/software_examples/s3cmd.html#configuring-s3cmd "Permalink to this heading")

For general use, the `s3cmd --configure` command is the fastest way to set up your S3Cmd installation.

**Configuration Process Example:**

 1$ s3cmd -c tebi.cfg --configure
 2
 3Enter new values or accept defaults in brackets with Enter.
 4Refer to user manual for detailed description of all options.
 5
 6Access key and Secret key are your identifiers for Amazon S3. Leave them empty for using the env variables.
 7Access Key: YOUR_KEY
 8Secret Key: YOUR_SECRET
 9Default Region [US]:
10
11Use "s3.amazonaws.com" for S3 Endpoint and not modify it to the target Amazon S3.
12S3 Endpoint [s3.amazonaws.com]: s3.tebi.io
13
14Use "%(bucket)s.s3.amazonaws.com" to the target Amazon S3. "%(bucket)s" and "%(location)s" vars can be used
15if the target S3 system supports dns based buckets.
16DNS-style bucket+hostname:port template for accessing a bucket [%(bucket)s.s3.amazonaws.com]: %(bucket)s.s3.tebi.io
17
18Encryption password is used to protect your files from reading
19by unauthorized persons while in transfer to S3
20Encryption password:
21Path to GPG program [/usr/bin/gpg]:
22
23When using secure HTTPS protocol all communication with Amazon S3
24servers is protected from 3rd party eavesdropping. This method is
25slower than plain HTTP, and can only be proxied with Python 2.7 or newer
26Use HTTPS protocol [Yes]:
27
28On some networks all internet access must go through a HTTP proxy.
29Try setting it here if you can't connect to S3 directly
30HTTP Proxy server name:
31
32New settings:
33  Access Key: YOUR_KEY
34  Secret Key: YOUR_SECRET
35  Default Region: US
36  S3 Endpoint: s3.tebi.io
37  DNS-style bucket+hostname:port template for accessing a bucket: %(bucket)s.s3.tebi.io
38  Encryption password:
39  Path to GPG program: /usr/bin/gpg
40  Use HTTPS protocol: True
41  HTTP Proxy server name:
42  HTTP Proxy server port: 0
43
44Test access with supplied credentials? [Y/n] n
45
46Save settings? [y/N] y
47Configuration saved to 'tebi.cfg'

## Running S3Cmd[¶](https://docs.tebi.io/software_examples/s3cmd.html#running-s3cmd "Permalink to this heading")

**See all buckets**

s3cmd -c tebi.cfg ls

**Make a new bucket**

s3cmd -c tebi.cfg mb s3://BUCKET_NAME

**List the contents of a bucket**

s3cmd -c tebi.cfg ls s3://BUCKET_NAME

Sync `/home/local/directory` to the remote `BUCKET_NAME`, deleting any excess files in the bucket.

s3cmd -c tebi.cfg sync /home/local/directory s3://BUCKET_NAME