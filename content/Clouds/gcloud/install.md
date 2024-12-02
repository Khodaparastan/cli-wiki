---
title: Initial setup
tags:
  - gcloud
  - install
reference url: https://cloud.google.com/cli?hl=en
---
reference: https://cloud.google.com/cli?hl=en

> [!NOTE]
>  This is instructions for MacOS
>  
>  For other OS use links below

[Linux](https://cloud.google.com/sdk/docs/install#linux)[Debian/Ubuntu](https://cloud.google.com/sdk/docs/install#debianubuntu)[Red Hat/Fedora/CentOS](https://cloud.google.com/sdk/docs/install#red-hatfedoracentos)[macOS](https://cloud.google.com/sdk/docs/install#macos)[Windows](https://cloud.google.com/sdk/docs/install#windows)

1. Confirm that you have a supported version of Python:
    - To check your current Python version, run `python3 -V` or `python -V`. Supported versions are Python 3.8 to 3.12.
    - The main install script offers to install CPython's Python 3.11.
    - Otherwise, to install a supported Python version, please visit the Python.org[Python Releases for macOS](https://www.python.org/downloads/macos/).
    - If you have multiple Python interpreters installed on your machine, set the CLOUDSDK_PYTHON environment variable within your shell to point to the path of your preferred interpreter.
    - For more information on how to choose and configure your Python interpreter, see[`gcloud topic startup`](https://cloud.google.com/sdk/gcloud/reference/topic/startup).
2. Download one of the following:
    

> [!NOTE]
>      To determine your machine hardware name, run `uname -m` from a command line.

    
| Platform                                      | Package                                                                                                                                   | Size    |
| --------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| macOS 64-bit<br><br>(x86_64)                  | [google-cloud-cli-darwin-x86_64.tar.gz](https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-darwin-x86_64.tar.gz) | 53.7 MB |
| macOS 64-bit<br><br>(ARM64, Apple M1 silicon) | [google-cloud-cli-darwin-arm.tar.gz](https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-darwin-arm.tar.gz)       | 53.7 MB |
| macOS 32-bit<br><br>(x86)                     | [google-cloud-cli-darwin-x86.tar.gz](https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-darwin-x86.tar.gz)       | 52.4 MB |

a. Extract the archive to any location on your file system (preferably your Home directory). On macOS, this can be achieved by opening the downloaded `.tar.gz`archive file in the preferred location.

b. (Optional) Use the install script to add gcloud CLI tools to your `PATH`.You can also opt-in to command-completion for your shell, [usage statistics collection](https://cloud.google.com/sdk/docs/usage-statistics), and install Python 3.11.
       
   
    ./google-cloud-sdk/install.sh
    
3. To initialize the gcloud CLI, run [`gcloud init`](https://cloud.google.com/sdk/gcloud/reference/init):


  Optional. Install additional components using the [component manager](https://cloud.google.com/sdk/docs/managing-components).
  