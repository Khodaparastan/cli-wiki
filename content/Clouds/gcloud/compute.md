---
title: Virtual machine
tags:
  - vm
  - compute
---

  
For more specific or complicated Compute Engine instance creation, see the following resources:

- [Create a VM that uses a user-managed service account](https://cloud.google.com/compute/docs/access/create-enable-service-accounts-for-instances).
- [Create Windows Server instances](https://cloud.google.com/compute/docs/instances/windows/creating-managing-windows-instances).
- [Create SQL Server instances](https://cloud.google.com/compute/docs/instances/sql-server/creating-sql-server-instances).
- [Create instances on sole-tenant nodes](https://cloud.google.com/compute/docs/nodes/create-nodes).
- [Create a VM instance with a custom hostname](https://cloud.google.com/compute/docs/instances/custom-hostname-vm).
- [Create VM instances that use the gVNIC network interface](https://cloud.google.com/compute/docs/networking/using-gvnic#create_a_vm_with_gvnic_support).
- [Create and start an Arm VM instance](https://cloud.google.com/compute/docs/instances/create-arm-vm-instance).
- [Create a VM instance with attached GPUs](https://cloud.google.com/compute/docs/gpus/create-vm-with-gpus).
- [Create a VM instance with a high performance computing (HPC) image](https://cloud.google.com/compute/docs/instances/create-hpc-vm).
- [Create VMs in bulk](https://cloud.google.com/compute/docs/instances/multiple/create-in-bulk).
- [Create a VM instance with an attached instance schedule](https://cloud.google.com/compute/docs/instances/schedule-instance-start-stop#attaching_to_a_new_VM).
- [Create a managed instance group (MIG)](https://cloud.google.com/compute/docs/instance-groups/creating-groups-of-managed-instances#basic_scenarios_for_creating_a_mig).
- [Create a Confidential VM instance](https://cloud.google.com/confidential-computing/confidential-vm/docs/creating-cvm-instance).
- [Reserve instances](https://cloud.google.com/compute/docs/instances/reservations-overview) and [consume reserved instances](https://cloud.google.com/compute/docs/instances/reservations-overview#consuming_reserved_instances).
- [Configure a VM instance with higher bandwidth](https://cloud.google.com/compute/docs/networking/configure-vm-with-high-bandwidth-configuration).

If you are bringing an existing license, see [Bringing your own licenses](https://cloud.google.com/compute/docs/nodes/bringing-your-own-licenses).

## Before you begin

- When creating compute instances from images or disks by using the Google Cloud CLI or REST, there's a limit of 20 instances per second. If you need to create a higher number of instances per second, [request a higher quota limit](https://cloud.google.com/docs/quotas/view-manage#requesting_higher_quota)for the **Images** resource.
- If you haven't already, set up authentication. [Authentication](https://cloud.google.com/compute/docs/authentication) is the process by which your identity is verified for access to Google Cloud services and APIs. To run code or samples from a local development environment, you can authenticate to Compute Engine as follows.  
    
    1. [Install](https://cloud.google.com/sdk/docs/install) the Google Cloud CLI, then [initialize](https://cloud.google.com/sdk/docs/initializing) it by running the following command:
        
        `gcloud init`
        
        **Note:** If you installed the gcloud CLI previously, make sure you have the latest version by running `gcloud components update`.
        
    2. [Set a default region and zone](https://cloud.google.com/compute/docs/gcloud-compute#set_default_zone_and_region_in_your_local_client).
    

### Required roles

To get the permissions that you need to create VMs, ask your administrator to grant you the [Compute Instance Admin (v1)](https://cloud.google.com/iam/docs/understanding-roles#compute.instanceAdmin.v1) (`roles/compute.instanceAdmin.v1`) IAM role on the project. For more information about granting roles, see [Manage access to projects, folders, and organizations](https://cloud.google.com/iam/docs/granting-changing-revoking-access).

This predefined role contains the permissions required to create VMs. To see the exact permissions that are required, expand the **Required permissions** section:

#### Required permissions

You might also be able to get these permissions with [custom roles](https://cloud.google.com/iam/docs/creating-custom-roles) or other [predefined roles](https://cloud.google.com/iam/docs/understanding-roles).

## Create a VM instance from an image

This section explains how to create a VM from a [public OS image](https://cloud.google.com/compute/docs/images) or a [custom image](https://cloud.google.com/compute/docs/images/create-delete-deprecate-private-images). A VM contains a bootloader, a boot file system, and an OS image.

### View a list of public images available on Compute Engine

Before you create a VM by using a public image, review the list of public images that are available on Compute Engine.

For more information about the features available with each public image, see [Feature support by operating system](https://cloud.google.com/compute/docs/images/os-details).

1. Run the following command:
    
    gcloud compute images list
    
2. Make a note of the name of the image or image family and the name of the project containing the image.
    
3. Optional: To determine whether the image supports [Shielded VM](https://cloud.google.com/compute/shielded-vm/docs/shielded-vm) features, run the following command:
    
    `gcloud compute images describe IMAGE_NAME \`
        `--project=IMAGE_PROJECT`
    
    Replace the following:
    
    - `IMAGE_NAME`: name of the image to check for support of Shielded VM features
    - `IMAGE_PROJECT`: [project](https://cloud.google.com/compute/docs/images#os-compute-support) containing the image
    
    If the image supports Shielded VM features, the following line appears in the output: `type: UEFI_COMPATIBLE`.
    

### Create a VM instance from a public image

Google, open source communities, and third-party vendors provide and maintain [public OS images](https://cloud.google.com/compute/docs/images#os-compute-support). By default, all Google Cloud projects can create VMs from public OS images. However, if your Google Cloud project has a defined list of [trusted images](https://cloud.google.com/compute/docs/images/restricting-image-access), you can use only the images on that list to create a VM.

If you create a [Shielded VM](https://cloud.google.com/compute/shielded-vm/docs/shielded-vm) image with a [local SSD](https://cloud.google.com/compute/docs/disks#localssds), you can't shield data with [integrity monitoring](https://cloud.google.com/compute/shielded-vm/docs/shielded-vm#integrity-monitoring) or the [virtual platform trusted module (vTPM)](https://cloud.google.com/compute/shielded-vm/docs/shielded-vm#vtpm).


1. Select a [public image](https://cloud.google.com/compute/docs/instances/create-start-instance#view-images). Make a note of the name of the image or image family and the name of the project containing the image.
2. Use the [`gcloud compute instances create` command](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create) to create a VM from an image family or from a specific version of an OS image.
    
    If you specify the optional `--shielded-secure-boot` flag, Compute Engine creates a VM with all three of the [Shielded VM](https://cloud.google.com/compute/shielded-vm/docs/shielded-vm) features enabled:
    
    - [Virtual trusted platform module (vTPM)](https://cloud.google.com/compute/shielded-vm/docs/shielded-vm#vtpm)
    - [Integrity monitoring](https://cloud.google.com/compute/shielded-vm/docs/shielded-vm#integrity-monitoring)
    - [Secure Boot](https://cloud.google.com/compute/shielded-vm/docs/shielded-vm#secure-boot)
    
    After Compute Engine starts your VM, you must stop the VM to [modify Shielded VM](https://cloud.google.com/compute/shielded-vm/docs/modifying-shielded-vm) options.
    
	```bash

    gcloud compute instances create VM_NAME \
        --zone=ZONE \
        [--image=IMAGE | --image-family=IMAGE_FAMILY] \
        --image-project=IMAGE_PROJECT \
        --machine-type=MACHINE_TYPE
    
	```
    
    Replace the following:
    
    - `VM_NAME`: [name](https://cloud.google.com/compute/docs/naming-resources#resource-name-format) of the new VM
    - `ZONE`: zone to create the instance in
    - `IMAGE` or `IMAGE_FAMILY`: specify one of the following:
        
        - `IMAGE`: a specific version of a public image
            
            For example, `--image=debian-10-buster-v20200309`.
            
        - `IMAGE_FAMILY`: an [image family](https://cloud.google.com/compute/docs/images/os-details#general-info).
            
            This creates the VM from the most recent, non-deprecated OS image. For example, if you specify `--image-family=debian-10`, Compute Engine creates a VM from the latest version of the OS image in the Debian 10 image family.
            
    - `IMAGE_PROJECT`: [project](https://cloud.google.com/compute/docs/images/os-details#general-info) containing the image
        
    - `MACHINE_TYPE`: machine type, [predefined](https://cloud.google.com/compute/docs/machine-resource) or [custom](https://cloud.google.com/compute/docs/instances/creating-instance-with-custom-machine-type), for the new VM
        
        To get a list of the machine types available in a zone, use the [`gcloud compute machine-types list`command](https://cloud.google.com/sdk/gcloud/reference/compute/machine-types/list) with the `--zones` flag.
        
3. Verify that Compute Engine created the VM:
    
    gcloud compute instances describe VM_NAME
    
    Replace `VM_NAME` with the name of the VM.
    

### Create a bare metal instance from a public image

Google, open source communities, and third-party vendors provide and maintain [public OS images](https://cloud.google.com/compute/docs/images#os-compute-support). By default, all Google Cloud projects can create bare metal instances using supported public OS images. However, if your Google Cloud project has a defined list of [trusted images](https://cloud.google.com/compute/docs/images/restricting-image-access), you can use only the images on that list to create a bare metal instance.
 

1. Select a [public image](https://cloud.google.com/compute/docs/instances/create-start-instance#view-images) that supports bare metal instances. Make a note of the name of the image or image family and the name of the project containing the image.
2. Use the [`gcloud compute instances create` command](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create) to create a bare metal instance from an image family or from a specific version of an OS image.
    
	```
    gcloud compute instances create INSTANCE_NAME \
        --zone=ZONE \
        --machine-type=MACHINE_TYPE \
        --network-interface=nic-type=IDPF \
        --maintenance-policy=TERMINATE \
        --create-disk=boot=yes,image=projects/IMAGE_PROJECT/global/images/IMAGE,provisioned-iops=IOPS,provisioned-throughput=THROUGHPUT,size=SIZE,type=hyperdisk-balanced \
        --no-shielded-secure-boot
	```
    
    Replace the following:
    
    - `INSTANCE_NAME`: a [name](https://cloud.google.com/compute/docs/naming-resources#resource-name-format) for the new bare metal instance
    - `ZONE`: zone to create the bare metal instance in
    - `MACHINE_TYPE`: the bare metal machine type to use for the instance. The name of the machine type must end in `-metal`.
        
        To get a list of the machine types available in a zone, use the [`gcloud compute machine-types list`command](https://cloud.google.com/sdk/gcloud/reference/compute/machine-types/list) with the `--zones` flag.
        
    - `IMAGE_PROJECT`: the [image project](https://cloud.google.com/compute/docs/images/os-details#general-info) that contains the image
        
    - `IMAGE`: specify one of the following:
        
        - A specific version of the OS image—for example, `sles-15-sp4-sap-v20240208-x86-6`.
        - An [image family](https://cloud.google.com/compute/docs/images/os-details), which must be formatted as `family/IMAGE_FAMILY`. This creates the instance from the most recent, non-deprecated OS image. For example, if you specify `family/sles-15-sp4-sap`, Compute Engine creates a bare metal instance from the latest version of the OS image in the SUSE Linux Enterprise Server 15 SP4 image family. For more information about using image families, see [Image families best practices](https://cloud.google.com/compute/docs/images/image-families-best-practices).
    - `IOPS`: Optional: the highest number of I/O operations per second (IOPS) that the disk can handle.
        
    - `THROUGHPUT`: Optional: an integer that represents the highest throughput, measured in MiB per second, that the disk can handle.
        
    - `SIZE`: Optional: the size of the new disk. The value must be a whole number. The default unit of measurement is GiB.
        
3. Verify that Compute Engine created the instance:
    
    gcloud compute instances describe INSTANCE_NAME
    
    Replace `INSTANCE_NAME` with the name of the new instance.
    

### Create a VM from a custom image

A custom image belongs only to your project. To create a VM with a custom image, you must first [create a custom image](https://cloud.google.com/compute/docs/images/create-delete-deprecate-private-images#creating_a_custom_image)if you don't already have one.

**Note:** You must have access to the custom image to use it when you create a VM. By default, you have access to all custom images in your project. However, if your project has a defined list of [trusted images](https://cloud.google.com/compute/docs/images/restricting-image-access), you can use only the images on that list to create a VM.

1. In one of the following development environments, set up the gcloud CLI:
    
    - **Cloud Shell**: to use an online terminal with the gcloud CLI already set up, activate Cloud Shell.
        
        Activate Cloud Shell on this page
        
        At the bottom of this page, a Cloud Shell session starts and displays a command-line prompt. It can take a few seconds for the session to initialize.
        
    - **Local shell**: to use a local development environment, [install](https://cloud.google.com/sdk/docs/install) and [initialize](https://cloud.google.com/sdk/docs/initializing) the gcloud CLI.
        
2. Run the [`gcloud compute instances create` command](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create) to create a VM with a custom image:
    
	```
    gcloud compute instances create VM_NAME \
        --image-project IMAGE_PROJECT \
        [--image IMAGE | --image-family IMAGE_FAMILY]
        --subnet SUBNET
    
	```
    Replace the following:
    
    - `VM_NAME`: name of the VM
    - `IMAGE_PROJECT`: Project ID that contains the image
    - `IMAGE` or `IMAGE_FAMILY`: specify one of the following:
        
        - `IMAGE`: name of your custom image
            
            For example, `--image=my-debian-image-v2`.
            
        - `IMAGE_FAMILY`: if you created your custom images as part of a [custom image family](https://cloud.google.com/compute/docs/images#custom-families), specify that custom image family.
            
            This creates the VM from the most recent, non-deprecated OS image and OS version in your custom image family. For example, if you specify `--image-family=my-debian-family`, Compute Engine creates a VM from the latest OS image in your custom `my-debian-family`image family.
            
        
        **Note:** Compute Engine uses the default image family and project if you don't specify an image. The default image family and project are `debian-10` and `debian-cloud`, respectively.
        
    - `SUBNET`: If the subnet and instance are in the same project, replace SUBNET with the name of a subnet that is in the same region as the instance. 
        
    

### Create a VM instance with additional non-boot disks

When you create a VM, you can also create and attach additional non-boot disks to the VM at the same time.

However, if you want to create a disk in multi-writer mode, you can't create the disk at the same time that you create the VM. You must [create the disk](https://cloud.google.com/compute/docs/disks/add-hyperdisk) first, then you can [attach it to the VM](https://cloud.google.com/compute/docs/disks/attach-disks).

1. In one of the following development environments, set up the gcloud CLI:
    
    - **Cloud Shell**: to use an online terminal with the gcloud CLI already set up, activate Cloud Shell.
        
        Activate Cloud Shell on this page
        
        At the bottom of this page, a Cloud Shell session starts and displays a command-line prompt. It can take a few seconds for the session to initialize.
        
    - **Local shell**: to use a local development environment, [install](https://cloud.google.com/sdk/docs/install) and [initialize](https://cloud.google.com/sdk/docs/initializing) the gcloud CLI.
        
2. Run the [`gcloud compute instances create` command](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create) to create a VM with additional non-boot disks.
    
    You can add up to 127 non-boot disks while you're creating your VM. Specify the `--create-disk` flag for each non-boot disk you create.
    
    To create non-boot disks from a public or stock image, specify the `image` or `image-family` and `image-project` properties with the `--create-disk` flag. To create a blank disk, don't include these properties. You can optionally include properties for the disk `size` and `type`. Include the property `replica-zones` to create regional persistent disks.
    
	```
    gcloud compute instances create VM_NAME \
      --zone=ZONE \
      [--image=IMAGE | --image-family=IMAGE_FAMILY] \
      --image-project=IMAGE_PROJECT \
      --create-disk [image=DISK_IMAGE | image-family=DISK_IMAGE_FAMILY ], \
        image-project=DISK_IMAGE_PROJECT,size=SIZE_GB,type=DISK_TYPE \
      --create-disk device-name=DISK_NAME, \
        replica-zones=^:^ZONE:REMOTE-ZONE,boot=false
    
	```
    Replace the following:
    
    - `VM_NAME`: [name](https://cloud.google.com/compute/docs/naming-resources#resource-name-format) of the new VM
    - `ZONE`: zone to create the VM in
    - `IMAGE` or `IMAGE_FAMILY`. Specify one of the following:
        
        - `IMAGE`: a specific version of a public image
            
            For example, `--image=debian-10-buster-v20200309`.
            
        - `IMAGE_FAMILY`: an [image family](https://cloud.google.com/compute/docs/images/os-details#general-info)
            
            This creates the VM from the most recent, non-deprecated OS image. For example, if you specify `--image-family=debian-10`, Compute Engine creates a VM from the latest version of the OS image in the Debian 10 image family.
            
    - `IMAGE_PROJECT`: [project](https://cloud.google.com/compute/docs/images/os-details#general-info) containing the image
        
    - For additional disks, replace the following:
        
        - `DISK_IMAGE` or `DISK_IMAGE_FAMILY`: Specify one of the following:
            - `DISK_IMAGE`: name of the image that you want to use as a non-boot disk
            - `DISK_IMAGE_FAMILY`: an image family to use as a non-boot disk
        - `DISK_IMAGE_PROJECT`: an image project to which the disk image belongs
        - `SIZE_GB`: Optional: size of the non-boot disk
        - `DISK_TYPE`: Optional: full or partial URL for the [type](https://cloud.google.com/compute/docs/disks#disk-types) of the persistent disk
            
            For example, `https://www.googleapis.com/compute/v1/projects/PROJECT_ID/zones/ZONE/diskTypes/pd-ssd`. To view the available disk types, run the [`gcloud compute disk-types list`command](https://cloud.google.com/sdk/gcloud/reference/compute/disk-types/list).
            
        - `DISK_NAME`: Optional: the disk name displayed to the guest OS after the VM is created.
            
        - `REMOTE_ZONE`: the zone where the disk should be replicated to
            
        
        For blank disks, don't specify the `DISK_IMAGE`, `DISK_IMAGE_FAMILY`, or `DISK_IMAGE_PROJECT`parameters.
        
        For zonal disks, don't specify the disk property `replica-zones`.
        

[Format and mount](https://cloud.google.com/compute/docs/disks/format-mount-disk-linux) the disks before using them.

### Create a VM instance from a shared image

If another user has [shared an image with you](https://cloud.google.com/compute/docs/images/managing-access-custom-images), you can use the image to create a VM.

1. In the Google Cloud console, go to the **Create an instance** page.
    
    [Go to Create an instance](https://console.cloud.google.com/compute/instancesAdd)
    
2. Specify a **Name** for your VM. For more information, see [Resource naming convention](https://cloud.google.com/compute/docs/naming-resources#resource-name-format).
3. Optional: Change the **Zone** for this VM. If you select **Any**, Google automatically chooses a zone for you based on machine type and availability.
4. Select a **Machine configuration** for your VM.
5. In the **Boot disk** section, click **Change** to configure your boot disk, and then do the following: 

1. Select the **Custom Images** tab.
2. To select the image project, click **Select a project**, and then do the following:
    1. Select the project that contains the image.
    2. Click **Open**.
3. In the **Image** list, click the image that you want to import.
4. Select the type and size of your boot disk.
5. To confirm your boot disk options, click **Select**.

7. To permit HTTP or HTTPS traffic to the VM, in the **Firewall** section, select **Allow HTTP traffic** or **Allow HTTPS traffic**.
    
    The Google Cloud console adds a network tag to your VM and creates the corresponding ingress firewall rule that allows all incoming traffic on `tcp:80` (HTTP) or `tcp:443` (HTTPS). The network tag associates the firewall rule with the VM. For more information, see [Firewall rules overview](https://cloud.google.com/firewall/docs/firewalls) in the Cloud Next Generation Firewall documentation.
    
8. To start and create a VM, click **Create**.

## Create a VM from a snapshot

You can create a new VM from a snapshot in the following ways:

- **Restoring a VM boot disk:** If you backed up a VM's boot disk with a [snapshot](https://cloud.google.com/compute/docs/disks/create-snapshots), you can use that snapshot to create a new VM. For instructions, see [Restoring a boot disk snapshot to a new VM](https://cloud.google.com/compute/docs/disks/restore-snapshot#create-vm-boot-disk).
    
- **Restoring a non-boot disk:** If you backed up a non-boot disk with a snapshot, you can restore the snapshot to a new non-boot disk when you create a VM. For instructions, see [Creating a VM with a non-boot disk based on a snapshot](https://cloud.google.com/compute/docs/disks/restore-snapshot#create-vm-nonboot-disk).
    

To quickly create more than one VM with the same boot disk, [create a custom image](https://cloud.google.com/compute/docs/images/create-delete-deprecate-private-images), then [create VMs from that image](https://cloud.google.com/compute/docs/instances/create-start-instance#create_a_vm_from_a_custom_image)instead of using a snapshot.

## Create a VM instance from a container image

To deploy and launch a container on a Compute Engine VM, specify a container image name and optional configuration parameters when you create the VM. Compute Engine creates the VM by using the latest version of the [Container-optimized OS public image](https://cloud.google.com/container-optimized-os/docs), which has Docker installed. Then, Compute Engine launches the container when the VM starts. For more information, see [Deploying containers on VMs](https://cloud.google.com/compute/docs/containers/deploying-containers).

To create a VM from a container image, you must use the Google Cloud console or `gcloud`.

1. In one of the following development environments, set up the gcloud CLI:
    
    - **Cloud Shell**: to use an online terminal with the gcloud CLI already set up, activate Cloud Shell.
        
        Activate Cloud Shell on this page
        
        At the bottom of this page, a Cloud Shell session starts and displays a command-line prompt. It can take a few seconds for the session to initialize.
        
    - **Local shell**: to use a local development environment, [install](https://cloud.google.com/sdk/docs/install) and [initialize](https://cloud.google.com/sdk/docs/initializing) the gcloud CLI.
        
2. Run the [`gcloud compute instances create-with-container` command](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create-with-container):
	```
    
    gcloud compute instances create-with-container VM_NAME \
      --container-image=CONTAINER_IMAGE
    
	```
    Replace the following:
    
    - `VM_NAME`: [name](https://cloud.google.com/compute/docs/naming-resources#resource-name-format) for the new VM.
    - `CONTAINER_IMAGE`: name of the container image.
    
    For example, the following command creates a VM named `nginx-vm`, which launches and runs the container image:
    
    `gcr.io/cloud-marketplace/google/nginx1:1.12`
    
    gcloud compute instances create-with-container nginx-vm \
      --container-image=gcr.io/cloud-marketplace/google/nginx1:1.12
    
    To deploy an Apache container image from Docker Hub, always specify the full Docker image name:
    
    `docker.io/httpd:2.4`.
    

## Create a VM instance in a specific subnet

By default, Google Cloud creates an [auto mode VPC network](https://cloud.google.com/vpc/docs/vpc#default-network) called `default` for each project. To use a different network or a subnet that you manually created in an auto mode or custom mode VPC network, you must specify the subnet when you create the VM.

While creating a VM in a subnet, consider these rules:

- If you don't specify a network or subnet, Compute Engine uses the default VPC network and the auto subnet that's in the same region as the VM.
- If you don't specify a network, Compute Engine infers the network from the subnet specified.
- If you specify a network, you must specify a subnet and it must belong to the same network. Otherwise, VM creation fails.

**Note:** You must create the subnet that you want to use before you create the VM. For more information, see [Create and manage VPC networks](https://cloud.google.com/vpc/docs/create-modify-vpc-networks).

1. In one of the following development environments, set up the gcloud CLI:
    
    - **Cloud Shell**: to use an online terminal with the gcloud CLI already set up, activate Cloud Shell.
        
        Activate Cloud Shell on this page
        
        At the bottom of this page, a Cloud Shell session starts and displays a command-line prompt. It can take a few seconds for the session to initialize.
        
    - **Local shell**: to use a local development environment, [install](https://cloud.google.com/sdk/docs/install) and [initialize](https://cloud.google.com/sdk/docs/initializing) the gcloud CLI.
        
2. Using the Google Cloud CLI, follow the same instructions to [create a VM from an image](https://cloud.google.com/compute/docs/instances/create-start-instance#startinginstancewithimage) or a [snapshot](https://cloud.google.com/compute/docs/instances/create-start-instance#createsnapshot), and add the `--subnet=SUBNET_NAME` and `--zone=ZONE` flags when you run the [`gcloud compute instances create` command](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create):
	```
    
    gcloud compute instances create VM_NAME \
      --network=NETWORK_NAME \
      --subnet=SUBNET_NAME \
      --zone=ZONE
    
	```
    Replace the following:
    
    - `VM_NAME`: [name](https://cloud.google.com/compute/docs/naming-resources#resource-name-format) of the VM
    - `NETWORK_NAME`: Optional: name of the network
    - `SUBNET_NAME`: name of the subnet
        
        To view a list of subnets in the network, use the [`gcloud compute networks subnets list`command](https://cloud.google.com/sdk/gcloud/reference/compute/networks/list).
        
    - `ZONE`: zone where the VM is created, such as `europe-west1-b`
        
        The VM's region is inferred from the zone.