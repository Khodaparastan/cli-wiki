---
title: Global network firewall policy with Tags
reference url: https://codelabs.developers.google.com/network-firewall-policy-tags
---
With the introduction of network firewall policy,  [Google Cloud's firewall policies](https://cloud.google.com/vpc/docs/firewall-policies-overview) now consists of the following components:

1. [Hierarchical Firewall Policy](https://cloud.google.com/vpc/docs/firewall-policies)
2. [VPC Firewall Rules](https://cloud.google.com/vpc/docs/firewall-policies-rule-details)
3. Network Firewall Policy ( [Global](https://cloud.google.com/vpc/docs/network-firewall-policies) and  [Regional](https://cloud.google.com/vpc/docs/regional-firewall-policies))

Hierarchical firewall policies are supported at the organization and folder nodes within the resource hierarchy, whereas VPC firewall rules and network firewall policies get applied at the VPC level. A big difference between VPC firewall rules and network firewall policies is that VPC firewall rules can be applied only to a single VPC network, whereas network firewall policies can get attached to a single VPC or group of VPCs, amongst other benefits like batch update.

Finally, we also have the  [implied firewall rules](https://cloud.google.com/vpc/docs/firewalls#default_firewall_rules) that come with every VPC network:

- An egress rule whose action is allow, destination is 0.0.0.0/0
- An ingress rule whose action is deny, source is 0.0.0.0/0

By default, the enforcement sequence is shown in the following diagram:

![abae4597af782b2b.png](https://codelabs.developers.google.com/static/network-firewall-policy-tags/img/abae4597af782b2b.png)

Please note that the enforcement order between the VPC firewall rules and the global network firewall policy can be swapped. Customers can specify the enforcement order at any time with a  [gcloud command](https://cloud.google.com/sdk/gcloud/reference/compute/networks/update).

## **Tags**

The new  [Tags integrated in network firewall policy rules](https://cloud.google.com/vpc/docs/tags-firewalls-overview) are key-value pair resources defined at the organization level of the Google Cloud resource hierarchy. Such a Tag contains an IAM access control, as the name implies, that specifies who can do what on the tag. IAM permissions, for instance, allow one to specify which principals can assign values to tags and which principals can attach tags to resources. Once a Tag has been applied to a resource, network firewall rules can use it to allow and deny traffic.

Tags adhere to Google Cloud's inheritance resource model, meaning tags and their values are passed down across the hierarchy from their parents. As a result, tags may be created in one place and then used by other folders and projects throughout the resource hierarchy. Visit  [this page](https://cloud.google.com/iam/docs/tags-access-control) for further details on tags and access restriction.

Tags should not be confused with  [network tags](https://cloud.google.com/vpc/docs/add-remove-network-tags), the latter are strings that can be added to Compute Engine instances; they are associated with the instance and vanish when the instance is decommissioned. VPC firewall rules may include network tags, but since they are not regarded as cloud resources, they are not subject to IAM access control.

**_Note that Tags and IAM-governed Tags are being used interchangeably in this document._**

## **What you'll build**

This codelab has two parts - the first one demonstrates network firewall policies and Tags using a single VPC network, and the second will show how to use Tags across peered VPC networks as per the diagram below. Thus, this codelab requires a single project and ability to create multiple VPC networks.

![b9acf9823df8be04.jpeg](https://codelabs.developers.google.com/static/network-firewall-policy-tags/img/b9acf9823df8be04.jpeg)

## **What you'll learn**

- How to create a network firewall policy
- How to create and use Tags with network firewall policy
- How to use Tags over VPC Network Peering

## **What you'll need**

- Google Cloud project
- Knowledge of deploying instances and configuring networking components
- VPC firewall configuration knowledge

## 2. Before you begin 

**Create/update variables**

This codelab makes use of $variables to aid gcloud configuration implementation in Cloud Shell.

Inside Cloud Shell, perform the following:

**Note**: You can get the organization id by using the Cloud Console or `gcloud projects get-ancestors [project-id]`.

```bash
gcloud config set project [project-id]
export project_id=`gcloud config list --format="value(core.project)"`
export org_id=[org]
export region=us-central1
export zone=us-central1-a
export prefix=fwpolicy
```

**Note**: If you have worked with Google Cloud before and have **`gcloud`** installed, up-to-date, and authenticated to your project already, feel free to use your local computer instead of Cloud Shell for this lab. **_Unless specifically noted, all gcloud commands should be run from a computer that already has_** ﻿***`gcloud`***﻿ ***installed or Cloud Shell.***

## 3. Create VPC network and subnet 

**Note**: Make sure that the Compute Engine API is enabled. If you get an error message by running the commands below or are not sure, run `gcloud services enable compute.googleapis.com` and try again.

**VPC Network**

Create **fwpolicy-vpc1:**

```
gcloud compute networks create $prefix-vpc1 --subnet-mode=custom 
```

**Subnets**

Create the respective subnets in the selected region:

```
gcloud compute networks subnets create $prefix-vpc1-subnet \
   --range=10.0.0.0/24 --network=$prefix-vpc1 --region=$region
```

**Cloud NAT**

Create the Cloud Routers and Cloud NAT gateways for **fwpolicy-pc1**:

```
gcloud compute routers create $prefix-vpc1-cr \
  --region=$region --network=$prefix-vpc1


gcloud compute routers nats create $prefix-vpc1-cloudnat \
   --router=$prefix-vpc1-cr --router-region=$region \
   --auto-allocate-nat-external-ips \
   --nat-all-subnet-ip-ranges
```


## 4. Create instances 

**Note**: For security reasons, this codelab assumes that  [Identity-Aware Proxy for TCP Forwarding is enabled](https://cloud.google.com/iap/docs/using-tcp-forwarding), in order to allow SSH access without assigning public IP addresses to the instances. Please configure it before proceeding.

Create a firewall rule that allows ingress SSH traffic from the IAP ranges in case it has not been defined yet as part of the IAP setup:




```
gcloud compute firewall-rules create allow-ssh-ingress-from-iap-vpc1 \
  --direction=INGRESS \
  --action=allow \
  --network=$prefix-vpc1 \
  --rules=tcp:22 \
  --source-ranges=35.235.240.0/20
```

Create the **fwpolicy-vpc1** client and web-server instances:
```

gcloud compute instances create $prefix-vpc1-www \
   --subnet=$prefix-vpc1-subnet --no-address --zone $zone \
   --metadata startup-script='#! /bin/bash
apt-get update
apt-get install apache2 -y
a2ensite default-ssl
a2enmod ssl
# Read VM network configuration:
md_vm="http://169.254.169.254/computeMetadata/v1/instance/"
vm_hostname="$(curl $md_vm/name -H "Metadata-Flavor:Google" )"
filter="{print \$NF}"
vm_network="$(curl $md_vm/network-interfaces/0/network \
-H "Metadata-Flavor:Google" | awk -F/ "${filter}")"
vm_zone="$(curl $md_vm/zone \
-H "Metadata-Flavor:Google" | awk -F/ "${filter}")"

# Apache configuration:
echo "Page on $vm_hostname in network $vm_network zone $vm_zone" | \
tee /var/www/html/index.html
systemctl restart apache2'

gcloud compute instances create $prefix-vpc1-client \
    --subnet=$prefix-vpc1-subnet --no-address --zone $zone
```

Since there are no VPC firewall rules defined (other than the allow SSH rule that should have been created when configuring IAP as per the beginning of this section), and by default all ingress traffic is denied, the client instances will not be able to access the respective web servers. In order to verify that the request will timeout, open a new window and initiate an SSH session to the **fwpolicy-vpc1-client** instance and try to curl the web server:

```
user@fwpolicy-vpc1-client$ curl fwpolicy-vpc1-www --connect-timeout 2
```

Expected output:

```
curl: (28) Connection timed out after 2001 milliseconds
```

Optionally, verify that there are no VPC firewall rules defined for **fwpolicy-vpc1** through Cloud Shell:

```
gcloud compute firewall-rules list --filter="network:$prefix-vpc1"
```

## 5. Global network firewall policy 

Create a global network firewall policy:

```bash
gcloud compute network-firewall-policies create \
   $prefix-example --description \
   "firewall-policy-description" --global
```

Add a rule allowing web traffic:

```bash
gcloud compute network-firewall-policies rules create 500 \
    --action allow \
    --description "allow-web" \
    --layer4-configs tcp:80,tcp:443 \
    --firewall-policy $prefix-example \
    --src-ip-ranges 10.0.0.0/16 \
    --global-firewall-policy --enable-logging
```

Describe the network firewall policy and verify that the rule was successfully `added`:

```bash
 gcloud compute network-firewall-policies describe \
    $prefix-example --global
```
 
Expected output (scroll up to the beginning of the output; note that the implicit rules are also displayed):

```
```
```
creationTimestamp: '2022-09-23T12:46:53.677-07:00'
description: "firewall-policy-description"
fingerprint: Np1Rup09Amc=
id: '7021772628738421698'
kind: compute#firewallPolicy
name: fwpolicy-example
ruleTupleCount: 13
rules:
- action: allow
  description: allow-web
  direction: INGRESS
  disabled: false
  enableLogging: true
  kind: compute#firewallPolicyRule
  match:
    layer4Configs:
    - ipProtocol: tcp
      ports:
      - '80'
    - ipProtocol: tcp
      ports:
      - '443'
    srcIpRanges:
    - 10.0.0.0/16
  priority: 500
  ruleTupleCount: 5
...


```
Associate the network firewall policy to **fwpolicy-vpc1**:

```
gcloud compute network-firewall-policies associations create \
     --firewall-policy $prefix-example \
     --network $prefix-vpc1 \
     --name $prefix-vpc1-association \
     --global-firewall-policy

```
Validate that it was successfully applied to **fwpolicy-vpc1** network:

gcloud compute networks get-effective-firewalls $prefix-vpc1

Expected output (note that if there are hierarchical firewall policies taking precedence, the relevant rules will be displayed at the top):
```

TYPE                     FIREWALL_POLICY_NAME     PRIORITY    ACTION     DIRECTION  IP_RANGES
network-firewall-policy  fwpolicy-example      500         ALLOW      INGRESS    10.0.0.0/16
network-firewall-policy  fwpolicy-example      2147483645  GOTO_NEXT  INGRESS    ::/0
network-firewall-policy  fwpolicy-example      2147483647  GOTO_NEXT  INGRESS    0.0.0.0/0
network-firewall-policy  fwpolicy-example      2147483644  GOTO_NEXT  EGRESS     ::/0
network-firewall-policy  fwpolicy-example      2147483646  GOTO_NEXT  EGRESS     0.0.0.0/0

```
Validate that it was successfully applied to **fwpolicy-vpc1** web server as well:

```
gcloud compute instances network-interfaces \
   get-effective-firewalls $prefix-vpc1-www --zone $zone

```
The expected output is similar to the previous command (**fwpolicy-vpc1** effective firewalls):

```
TYPE                     FIREWALL_POLICY_NAME     PRIORITY    ACTION     DIRECTION  IP_RANGES
network-firewall-policy  fwpolicy-example      500         ALLOW      INGRESS    10.0.0.0/16
network-firewall-policy  fwpolicy-example      2147483645  GOTO_NEXT  INGRESS    ::/0
network-firewall-policy  fwpolicy-example      2147483647  GOTO_NEXT  INGRESS    0.0.0.0/0
network-firewall-policy  fwpolicy-example      2147483644  GOTO_NEXT  EGRESS     ::/0
network-firewall-policy  fwpolicy-example      2147483646  GOTO_NEXT  EGRESS     0.0.0.0/0

```
Switch back to the vpc1-client SSH session and try to curl again (note that the command below assumes that `fwpolicy`was used as prefix; please adjust the `curl` command accordingly if a different name was used instead):

```
user@vpc1-client$ curl fwpolicy-vpc1-www --connect-timeout 2
Page on vpc1-www in network vpc1 zone us-central1-a

```
From Cloud Shell, validate that the network firewall policy is applied to **fwpolicy-vpc1**:

```
gcloud compute network-firewall-policies describe \
   $prefix-example --global

```
Expected output (scroll up to the beginning of the output):

```
---
associations:
- attachmentTarget: https://www.googleapis.com/compute/v1/projects/PROJECT_ID/global/networks/fwpolicy-vpc1
  name: fwpolicy-vpc1-association
...
```

## 6. IAM-governed Tags 

A tag is a key-value pair that can be attached to an organization, folder, or project. See  [Creating and managing tags](https://cloud.google.com/resource-manager/docs/tags/tags-creating-and-managing) and the  [required permissions](https://cloud.google.com/resource-manager/docs/tags/tags-creating-and-managing#required-permissions) for more details.

The tagAdmin role lets you create new tags, update, and delete existing tags. An organization administrator can grant this role. From Cloud Shell,  [update the IAM Policy to add the tagAdmin role to your user](https://cloud.google.com/vpc/docs/use-tags-for-firewalls#grant-tag-roles). Use the  [permissions reference](https://cloud.google.com/iam/docs/permissions-reference)page to see which permissions are included in each predefined role.

gcloud organizations add-iam-policy-binding $org_id \
  --member user:[user@example.com] --role roles/resourcemanager.tagAdmin

Run the command below to verify what users have the resourcemanager.tagAdmin role:

gcloud organizations get-iam-policy $org_id --flatten=bindings \
  --filter=bindings.role:roles/resourcemanager.tagAdmin

Create a new Tag Key:

gcloud resource-manager tags keys create tags-vpc1 \
   --parent organizations/$org_id \
   --purpose GCE_FIREWALL \
   --purpose-data network=$project_id/$prefix-vpc1

Expected output:

```
Waiting for TagKey [tags-vpc1] to be created...done.                                                                                                                
createTime: '2022-09-23T20:49:01.162228Z'
etag: PwvmFuHO4wK1y6c5Ut2n5w==
name: tagKeys/622132302133
namespacedName: ORGANIZATION_ID/tags-vpc1
parent: organizations/ORGANIZATION_ID
purpose: GCE_FIREWALL
purposeData:
  network: https://www.googleapis.com/compute/v1/projects/PROJECT_ID/global/networks/6749205358365096383
shortName: tags-vpc1
updateTime: '2022-09-23T20:49:03.873776Z'

```
**Note**: Make sure to confirm that **GCE_FIREWALL** is displayed as highlighted above, otherwise the Tags for firewalls functionality will not work as expected.

Create new tag values:
```

gcloud resource-manager tags values create web-servers \
   --parent=$org_id/tags-vpc1

gcloud resource-manager tags values create web-clients \
   --parent=$org_id/tags-vpc1

```
Validate that the tags values were successfully created:

```
gcloud resource-manager tags values list \
   --parent=$org_id/tags-vpc1

```
Expected output:

```
NAME                    SHORT_NAME   DESCRIPTION
tagValues/349564376683  web-servers
tagValues/780363571446  web-clients

```
From Cloud Shell, describe the existing network firewall policy rule to confirm that tags are not being used:

```
gcloud compute network-firewall-policies rules describe 500 \
    --firewall-policy $prefix-example \
    --global-firewall-policy

```
Expected output:

```
---
action: allow
description: allow-web
direction: INGRESS
disabled: false
enableLogging: true
kind: compute#firewallPolicyRule
match:
  layer4Configs:
  - ipProtocol: tcp
    ports:
    - '80'
  - ipProtocol: tcp
    ports:
    - '443'
  srcIpRanges:
  - 10.0.0.0/16
priority: 500
ruleTupleCount: 5
```

From Cloud Shell, update the rule to only allow traffic from **vpc1-tags/web-clients** tag key, and install the rule on instances with the **vpc1-tags/web-servers** tag key.

```
gcloud compute network-firewall-policies rules update 500 \
    --firewall-policy $prefix-example \
    --src-secure-tags $org_id/tags-vpc1/web-clients \
    --target-secure-tags $org_id/tags-vpc1/web-servers \
    --global-firewall-policy

```
From Cloud Shell, describe the existing network firewall policy rule to confirm that tags were successfully applied and are reported as **EFFECTIVE**:

```

gcloud compute network-firewall-policies rules describe 500 \
    --firewall-policy $prefix-example \
    --global-firewall-policy

```

Expected output:

```
---
action: allow
description: allow-web
direction: INGRESS
disabled: false
enableLogging: false
kind: compute#firewallPolicyRule
match:
  layer4Configs:
  - ipProtocol: tcp
    ports:
    - '80'
  - ipProtocol: tcp
    ports:
    - '443'
  srcIpRanges:
  - 10.0.0.0/16
  srcSecureTags:
  - name: tagValues/479619031616
    state: EFFECTIVE
priority: 500
ruleTupleCount: 7
targetSecureTags:
- name: tagValues/230424970229
  state: EFFECTIVE

```
From Cloud Shell, let's verify that the rule was applied to **vpc1**:

gcloud compute networks get-effective-firewalls $prefix-vpc1

Expected output:

```
network-firewall-policy  fwpolicy-example      500         ALLOW      INGRESS    10.0.0.0/16
network-firewall-policy  fwpolicy-example      2147483645  GOTO_NEXT  INGRESS    ::/0
network-firewall-policy  fwpolicy-example      2147483647  GOTO_NEXT  INGRESS    0.0.0.0/0
network-firewall-policy  fwpolicy-example      2147483644  GOTO_NEXT  EGRESS     ::/0
network-firewall-policy  fwpolicy-example      2147483646  GOTO_NEXT  EGRESS     0.0.0.0/0

```
Verify that even though the network firewall policy is still associated to the VPC network, the rule that allows web traffic is not applied to the web server anymore, because the Tag was not added to the instances:

```
gcloud compute instances network-interfaces \
   get-effective-firewalls $prefix-vpc1-www --zone $zone
```

Expected output (note that the firewall rule with priority 500 is not displayed):

```
network-firewall-policy  fwpolicy-example      2147483645  GOTO_NEXT  INGRESS    ::/0
network-firewall-policy  fwpolicy-example      2147483647  GOTO_NEXT  INGRESS    0.0.0.0/0
network-firewall-policy  fwpolicy-example      2147483644  GOTO_NEXT  EGRESS     ::/0
network-firewall-policy  fwpolicy-example      2147483646  GOTO_NEXT  EGRESS     0.0.0.0/0

```
[Grant the Tag User role to the specific tag and user](https://cloud.google.com/vpc/docs/tags-firewalls-overview#iam). Use the  [permissions reference](https://cloud.google.com/iam/docs/permissions-reference) page to see which permissions are included in each predefined role.
```

gcloud resource-manager tags keys add-iam-policy-binding \
  $org_id/tags-vpc1 \
  --member user:[email] --role roles/resourcemanager.tagUser

gcloud projects add-iam-policy-binding $project_id \
  --member user:[email] --role roles/resourcemanager.tagUser

```
Verify that the role was successfully added:
```

gcloud resource-manager tags keys get-iam-policy $org_id/tags-vpc1

gcloud projects get-iam-policy $project_id --flatten=bindings \
   --filter=bindings.role:roles/resourcemanager.tagUser

```
Expected output:
```

bindings:
- members:
  - user:[user]
  role: roles/resourcemanager.tagUser
...

```
Apply the tag to the **fwpolicy-vpc1-www** instance:

```
gcloud resource-manager tags bindings create \
  --location $zone \
  --tag-value $org_id/tags-vpc1/web-servers \
  --parent \
//compute.googleapis.com/projects/$project_id/zones/$zone/instances/$prefix-vpc1-www

```
Expected output:

```
Waiting for TagBinding for parent [//compute.googleapis.com/projects/PROJECT_ID/zones/us-central1-a/instances/38369703403698502] and tag value [tagValues/34
9564376683] to be created with [operations/rctb.us-central1-a.6144808968019372877]...done.                                                                            
done: true
metadata:
  '@type': type.googleapis.com/google.cloud.resourcemanager.v3.CreateTagBindingMetadata
name: operations/rctb.us-central1-a.6144808968019372877
response:
  '@type': type.googleapis.com/google.cloud.resourcemanager.v3.TagBinding
  name: tagBindings/%2F%2Fcompute.googleapis.com%2Fprojects%2FPROJECT_NUMBER%2Fzones%2Fus-central1-a%2Finstances%2F38369703403698502/tagValues/349564376683
  parent: //compute.googleapis.com/projects/PROJECT_NUMBER/zones/us-central1-a/instances/38369703403698502
  tagValue: tagValues/349564376683
```

Verify the bindings:

```
gcloud resource-manager tags bindings list --location $zone --effective --parent //compute.googleapis.com/projects/$project_id/zones/$zone/instances/$prefix-vpc1-www 

```
Expected output:

```
namespacedTagKey: ORGANIZATION_ID/tags-vpc1
namespacedTagValue: ORGANIZATION_ID/tags-vpc1/web-servers
tagKey: tagKeys/622132302133
tagValue: tagValues/349564376683
```

Verify the effective firewall rules again:
```

gcloud compute instances network-interfaces \
   get-effective-firewalls $prefix-vpc1-www --zone $zone
```

Expected output:
```

network-firewall-policy  fwpolicy-example      490         ALLOW      INGRESS    10.0.0.0/16
network-firewall-policy  fwpolicy-example      2147483645  GOTO_NEXT  INGRESS    ::/0
network-firewall-policy  fwpolicy-example      2147483647  GOTO_NEXT  INGRESS    0.0.0.0/0
network-firewall-policy  fwpolicy-example      2147483644  GOTO_NEXT  EGRESS     ::/0
network-firewall-policy  fwpolicy-example      2147483646  GOTO_NEXT  EGRESS     0.0.0.0/0

```
Switch back to the **fwpolicy-vpc1-client** SSH session tab and try to curl:

```
user@fwpolicy-vpc1-client$ curl fwpolicy-vpc1-www --connect-timeout 2
```

Were you able to connect?

**Note**: Even though the source tag was not added to the client instance as specified in the firewall rule, the source CIDR was also allowed. When both source CIDR and source Tags are specified,  [the inbound connection is allowed if there is a match for either condition.](https://cloud.google.com/sdk/gcloud/reference/compute/network-firewall-policies/rules/update)

To verify that, update the rule to remove the source CIDR criteria through Cloud Shell.
```

gcloud compute network-firewall-policies rules update 500 \
    --firewall-policy $prefix-example \
    --src-ip-ranges "" \
    --global-firewall-policy

gcloud compute network-firewall-policies rules describe 500 \
    --firewall-policy $prefix-example \
    --global-firewall-policy

action: allow
description: allow-web
direction: INGRESS
disabled: false
enableLogging: false
kind: compute#firewallPolicyRule
match:
  layer4Configs:
  - ipProtocol: tcp
    ports:
    - '80'
  - ipProtocol: tcp
    ports:
    - '443'
  srcSecureTags:
  - name: tagValues/479619031616
    state: EFFECTIVE
priority: 490
ruleTupleCount: 7
targetSecureTags:
- name: tagValues/230424970229
  state: EFFECTIVE
```

Switch back to the **fwpolicy-vpc1-client** SSH session tab and try again:

user@fwpolicy-vpc1-client$ curl fwpolicy-vpc1-www --connect-timeout 2

The connection should timeout this time, since the tag was not added to **fwpolicy-vpc1-client**. From Cloud Shell, add it and try once again.

```
gcloud resource-manager tags bindings create \
  --location $zone \
  --tag-value $org_id/tags-vpc1/web-clients \
  --parent \
//compute.googleapis.com/projects/$project_id/zones/$zone/instances/$prefix-vpc1-client

```
Switch back to the **fwpolicy-vpc1-client** SSH session tab and try again, which should now succeed.

user@fwpolicy-vpc1-client$ curl fwpolicy-vpc1-www --connect-timeout 2

## 7. IAM-governed Tags over VPC Network Peering 

From Cloud Shell, create a new VPC, subnet and client, and setup VPC Network Peering between the networks:
```bash

gcloud compute networks create $prefix-vpc2 --subnet-mode=custom 

gcloud compute networks subnets create $prefix-vpc2-subnet \
   --range=10.0.1.0/24 --network=$prefix-vpc2 --region=$region

gcloud compute instances create $prefix-vpc2-client \
   --subnet=$prefix-vpc2-subnet --no-address --zone $zone

gcloud compute networks peerings create vpc1-to-vpc2 \
   --network=$prefix-vpc1 \
   --peer-project $project_id \
   --peer-network $prefix-vpc2

gcloud compute networks peerings create vpc2-to-vpc1 \
    --network=$prefix-vpc2 \
    --peer-project $project_id \
    --peer-network $prefix-vpc1

```
**Note**: For security reasons, this codelab assumes that  [Identity-Aware Proxy for TCP Forwarding is enabled](https://cloud.google.com/iap/docs/using-tcp-forwarding), in order to allow SSH access without assigning public IP addresses to the instances. Please configure it before proceeding.

Create a firewall rule that allows ingress SSH traffic from the IAP ranges in case it has not been defined yet as part of the IAP setup:

```
gcloud compute firewall-rules create allow-ssh-ingress-from-iap-vpc2 \
  --direction=INGRESS \
  --action=allow \
  --network=$prefix-vpc2 \
  --rules=tcp:22 \
  --source-ranges=35.235.240.0/20
```

Even though Tags are org-wide objects,  [tag keys are associated to a specific VPC](https://cloud.google.com/vpc/docs/use-tags-for-firewalls#create-tags) and as such cannot be applied to instances in different networks. Thus, it's required to create a new tag key and value applicable to vpc2:
```

gcloud resource-manager tags keys create tags-vpc2 \
   --parent organizations/$org_id \
   --purpose GCE_FIREWALL \
   --purpose-data network=$project_id/$prefix-vpc2

gcloud resource-manager tags values create web-clients \
   --parent=$org_id/tags-vpc2

```
Apply the new tag to the **fwpolicy-vpc2-client** instance:

```
gcloud resource-manager tags bindings create \
  --location $zone \
  --tag-value $org_id/tags-vpc2/web-clients \
  --parent \
//compute.googleapis.com/projects/$project_id/zones/$zone/instances/$prefix-vpc2-client
```

Optionally, list **fwpolicy-vpc2-client**'s bindings:

```
gcloud resource-manager tags bindings list --location $zone --effective --parent //compute.googleapis.com/projects/$project_id/zones/$zone/instances/$prefix-vpc2-client

```
Expected output:

```
namespacedTagKey: ORGANIZATION_ID/tags-vpc2
namespacedTagValue: ORGANIZATION_ID/tags-vpc2/web-clients
tagKey: tagKeys/916316350251
tagValue: tagValues/633150043992

```
From Cloud Shell, describe the existing network firewall policy rule to confirm that the new tags are not being used:

```
gcloud compute network-firewall-policies rules describe 500 \
    --firewall-policy $prefix-example \
    --global-firewall-policy

```
Expected output:

```
---
action: allow
description: allow-web
direction: INGRESS
disabled: false
enableLogging: true
kind: compute#firewallPolicyRule
match:
  layer4Configs:
  - ipProtocol: tcp
    ports:
    - '80'
  - ipProtocol: tcp
    ports:
    - '443'
  srcSecureTags:
  - name: tagValues/479619031616
    state: EFFECTIVE
priority: 500
ruleTupleCount: 6
targetSecureTags:
- name: tagValues/230424970229
  state: EFFECTIVE

```
Update the existing firewall rule to allow the tags from the peered VPC network:

```
gcloud compute network-firewall-policies rules update 500 \
    --firewall-policy $prefix-example \
    --src-secure-tags $org_id/tags-vpc1/web-clients,$org_id/tags-vpc2/web-clients \
    --global-firewall-policy
```

Describe the firewall rule to make sure it was successfully applied and are reported as **EFFECTIVE**:

```
gcloud compute network-firewall-policies rules describe 500 \
    --firewall-policy $prefix-example \
    --global-firewall-policy
```

Expected output:

```
---
action: allow
description: allow-web
direction: INGRESS
disabled: false
enableLogging: false
kind: compute#firewallPolicyRule
match:
  layer4Configs:
  - ipProtocol: tcp
    ports:
    - '80'
  - ipProtocol: tcp
    ports:
    - '443'
  srcSecureTags:
  - name: tagValues/479619031616
    state: EFFECTIVE
  - name: tagValues/633150043992
    state: EFFECTIVE
priority: 500
ruleTupleCount: 7
targetSecureTags:
- name: tagValues/230424970229
  state: EFFECTIVE
```

Find out **fwpolicy-vpc1-www**'s IP through the gcloud command below:

```
gcloud compute instances list --filter=vpc1-www
```

Connect to the **fwpolicy-vpc2-client** through SSH and try to curl the **fwpolicy-vpc1**'s IP:

```
user@fwpolicy-vpc2-client$ curl [fwpolicy-vpc1-www_IP] --connect-timeout 2
```

You should be able to connect to the **fwpolicy-vpc1-www** server. Proceed to the next section for the clean-up steps.

## 8. Cleanup steps 

From Cloud Shell, remove the instances, Cloud NAT and Cloud Router:

```
gcloud -q compute instances delete $prefix-vpc2-client --zone=$zone

gcloud -q compute instances delete $prefix-vpc1-client --zone=$zone

gcloud -q compute instances delete $prefix-vpc1-www --zone=$zone

gcloud -q compute routers nats delete $prefix-vpc1-cloudnat \
--router=$prefix-vpc1-cr --router-region=$region

gcloud -q compute routers delete $prefix-vpc1-cr --region=$region
```

Remove the global network firewall policy and Tags:

```
gcloud -q resource-manager tags values delete \
   $org_id/tags-vpc2/web-clients

gcloud -q resource-manager tags keys delete $org_id/tags-vpc2

gcloud -q resource-manager tags values delete \
   $org_id/tags-vpc1/web-servers

gcloud -q resource-manager tags values delete \
   $org_id/tags-vpc1/web-clients

gcloud -q resource-manager tags keys delete $org_id/tags-vpc1

gcloud -q compute network-firewall-policies associations delete \
     --firewall-policy $prefix-example \
     --name $prefix-vpc1-association \
     --global-firewall-policy

gcloud -q compute network-firewall-policies delete \
   $prefix-example --global

gcloud -q compute firewall-rules delete allow-ssh-ingress-from-iap-vpc1

gcloud -q compute firewall-rules delete allow-ssh-ingress-from-iap-vpc2
```

Perform the steps below if tagAdmin and tagUsers roles were changed:

```
gcloud organizations remove-iam-policy-binding $org_id \
  --member user:[email] --role roles/resourcemanager.tagAdmin

gcloud organizations remove-iam-policy-binding $org_id \
  --member user:[email] --role roles/resourcemanager.tagUser
```

Finally, remove the VPC Network Peerings, subnets and VPC networks:

```
gcloud -q compute networks peerings delete vpc1-to-vpc2 \
    --network $prefix-vpc1

gcloud -q compute networks peerings delete vpc2-to-vpc1 \
    --network $prefix-vpc2

gcloud -q compute networks subnets delete $prefix-vpc1-subnet \
    --region $region

gcloud -q compute networks subnets delete $prefix-vpc2-subnet \
    --region $region

gcloud -q compute networks delete $prefix-vpc1

gcloud -q compute networks delete $prefix-vpc2
```

## 9. Congratulations! 

Congratulations, you've successfully configured and validated a global network firewall policy with Tags configuration.