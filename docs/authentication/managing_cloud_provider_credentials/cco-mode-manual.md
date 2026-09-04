---
title: About the Cloud Credential Operator in manual mode with long-term credentials for components
sidebar_position: 4
---

# About the Cloud Credential Operator in manual mode with long-term credentials for components {#cco-mode-manual}

<a id="cco-mode-manual"></a>

You can manage your cloud credentials instead of the Cloud Credential Operator (CCO) by setting the Operator to manual mode.

You can use manual mode with your Amazon Web Services (AWS), global Microsoft Azure, Microsoft Azure Stack Hub, Google Cloud, IBM Cloud(R), or Nutanix cluster.

To use manual mode, you must examine the `CredentialsRequest` CRs in the release image for the version of OpenShift Container Platform that you are running or installing, create corresponding credentials in the underlying cloud provider, and create Kubernetes Secrets in the correct namespaces to satisfy all `CredentialsRequest` CRs for the cluster’s cloud provider. Some platforms use the CCO utility (`ccoctl`) to facilitate this process during installation and updates.

Using manual mode with long-term credentials allows each cluster component to have only the permissions it requires, without storing an administrator-level credential in the cluster. This mode also does not require connectivity to services such as the AWS public IAM endpoint. However, you must manually reconcile permissions with new release images for every upgrade.

For information about configuring your cloud provider to use manual mode, see the manual credentials management options for your cloud provider.

:::note

An AWS, global Azure, or Google Cloud cluster that uses manual mode can be configured to use short-term credentials for different components. For more information, see "Manual mode with short-term credentials for components".

:::

**Additional resources**

- [Manually creating long-term credentials for AWS](/docs/installing/installing_aws/ipi/installing-aws-customizations#manually-create-iam_installing-aws-customizations)
- [Manually creating long-term credentials for Azure](/docs/installing/installing_azure/ipi/installing-azure-customizations#manually-create-iam_installing-azure-customizations)
- [Manually creating long-term credentials for Google Cloud](/docs/installing/installing_gcp/installing-gcp-customizations#manually-create-iam_installing-gcp-customizations)
- [Configuring IAM for IBM Cloud(R)](/docs/installing/installing_ibm_cloud/configuring-iam-ibm-cloud#configuring-iam-ibm-cloud)
- [Configuring IAM for Nutanix](/docs/installing/installing_nutanix/installing-nutanix-installer-provisioned#manually-create-iam-nutanix_installing-nutanix-installer-provisioned)
- [About the Cloud Credential Operator in manual mode with short-term credentials for components](/docs/authentication/managing_cloud_provider_credentials/cco-short-term-creds#cco-short-term-creds)
- [Preparing to update a cluster with manually maintained credentials](/docs/updating/preparing_for_updates/preparing-manual-creds-update#preparing-manual-creds-update)
