---
title: User-managed encryption for IBM Cloud
sidebar_position: 4
---

# User-managed encryption for IBM Cloud {#user-managed-encryption-ibm-cloud}

<a id="user-managed-encryption-ibm-cloud"></a>

By default, OpenShift Container Platform uses provider-managed encryption to secure the root volumes and persistent data volumes of a cluster. You can override this by specifying an IBM(R) Key Protect root key by using the `encryptionKey` parameter in the `install-config.yaml` file.

You can specify that:

- The same root key applies to all cluster machines by specifying the key as part of the cluster’s default machine configuration. All managed storage classes are updated with this key, so data volumes provisioned after installation are also encrypted by using this key.
- Separate root keys apply to the control plane and compute machine pools.

When you bring your own root key, you change the `install-config.yaml` file to specify the Cloud Resource Name (CRN) of the root key by using the `encryptionKey` parameter.

:::note

Make sure you have integrated Key Protect with your IBM Cloud Block Storage service. For more information, see "Key Protect documentation".

:::

**Additional resources**

- [Key Protect documentation](https://cloud.ibm.com/docs/key-protect?topic=key-protect-integrate-services#grant-access)
- [Additional IBM Cloud configuration parameters](/docs/installing/installing_ibm_cloud/installation-config-parameters-ibm-cloud-vpc#installation-configuration-parameters-additional-ibm-cloud_installation-config-parameters-ibm-cloud-vpc)
- [Installing a cluster on IBM Cloud with customizations](/docs/installing/installing_ibm_cloud/installing-ibm-cloud-customizations#installing-ibm-cloud-customizations)
- [Installing a cluster on IBM Cloud with network customizations](/docs/installing/installing_ibm_cloud/installing-ibm-cloud-customizations#installing-ibm-cloud-customizations)
- [Installing a cluster on IBM Cloud into an existing VPC](/docs/installing/installing_ibm_cloud/installing-ibm-cloud-vpc#installing-ibm-cloud-vpc)
- [Installing a private cluster on IBM Cloud](/docs/installing/installing_ibm_cloud/installing-ibm-cloud-private#installing-ibm-cloud-private)
