---
title: Adding compute machines to clusters with user-provisioned infrastructure manually
sidebar_position: 1
---

# Adding compute machines to clusters with user-provisioned infrastructure manually {#adding-compute-user-infra-general}

<a id="adding-compute-user-infra-general"></a>

To scale a OpenShift Container Platform cluster that uses user-provisioned infrastructure, you can manually add compute machines during or after installation. The postinstallation process requires some of the same configuration files and parameters that you used for installation.

## Adding compute machines to Amazon Web Services {#upi-adding-compute-aws}

You can add more compute machines to your OpenShift Container Platform cluster on AWS.

See [Adding compute machines to AWS by using CloudFormation templates](/docs/machine_management/user_infra/adding-aws-compute-user-infra#adding-aws-compute-user-infra) for more information.

## Adding compute machines to Microsoft Azure {#upi-adding-compute-azure}

You can add more compute machines to your OpenShift Container Platform cluster on Microsoft Azure.

See [Creating additional worker machines in Azure](/docs/installing/installing_azure/upi/installing-azure-user-infra#installation-creating-azure-worker_installing-azure-user-infra) for more information.

## Adding compute machines to Azure Stack Hub {#upi-adding-compute-ash}

You can add more compute machines to your OpenShift Container Platform cluster on Azure Stack Hub.

See [Creating additional worker machines in Azure Stack Hub](/docs/installing/installing_azure_stack_hub/upi/installing-azure-stack-hub-user-infra#installation-creating-azure-worker_installing-azure-stack-hub-user-infra) for more information.

## Adding compute machines to Google Cloud {#upi-adding-compute-gcp}

You can add more compute machines to your OpenShift Container Platform cluster on GCP.

See [Creating additional worker machines in Google Cloud](/docs/installing/installing_gcp/installing-restricted-networks-gcp#installation-creating-gcp-worker_installing-restricted-networks-gcp) for more information.

## Adding compute machines to vSphere {#upi-adding-compute-vsphere}

You can use compute machine sets to automate the creation of additional compute machines for your OpenShift Container Platform cluster on vSphere.

See [use compute machine sets](/docs/machine_management/creating_machinesets/creating-machineset-vsphere#creating-machineset-vsphere) for more information.

You can manually add more compute machines to your cluster.

See [Adding compute machines to vSphere manually](/docs/machine_management/user_infra/adding-vsphere-compute-user-infra#adding-vsphere-compute-user-infra) for more information.

You can add bare-metal compute machines to your cluster.

See [Adding bare-metal compute machines to a vSphere cluster](/docs/machine_management/user_infra/adding-bare-metal-compute-vsphere-user-infra#adding-bare-metal-compute-vsphere-user-infra) for more information.

:::warning

Bare-metal nodes on VMware vSphere clusters is generally available for OpenShift Container Platform 4.22.13 and later. However, this feature is Technology Preview for 4.21 through 4.22.12.

:::

## Adding compute machines to bare metal {#upi-adding-compute-baremetal}

You can add more compute machines to your OpenShift Container Platform cluster on bare metal.

See [Adding compute machines to bare metal](/docs/machine_management/user_infra/adding-bare-metal-compute-user-infra#adding-bare-metal-compute-user-infra) for more information.
