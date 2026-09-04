---
title: Configuring the registry for Azure user-provisioned infrastructure
sidebar_position: 4
---

# Configuring the registry for Azure user-provisioned infrastructure {#configuring-registry-storage-azure-user-infrastructure}

<a id="configuring-registry-storage-azure-user-infrastructure"></a>

Save your container images to a durable storage location by configuring the built-in image registry to use dedicated Azure storage. This setup provides persistent scalable storage for your registry, separate from ephemeral cluster storage.

## Configuring a secret for the Image Registry Operator {#registry-operator-config-resources-secret-azure_configuring-registry-storage-azure-user-infrastructure}

In addition to the `configs.imageregistry.operator.openshift.io` and ConfigMap resources, configuration is provided to the Operator by a separate secret resource located within the `openshift-image-registry` namespace.

The `image-registry-private-configuration-user` secret provides credentials needed for storage access and management. It overrides the default credentials used by the Operator, if default credentials were found.

For Azure registry storage, the secret is expected to contain one key whose value is the contents of a credentials file provided by Azure:

- `REGISTRY_STORAGE_AZURE_ACCOUNTKEY`

**Procedure**

- Create an OpenShift Container Platform secret that contains the required key.
  ```terminal
  $ oc create secret generic image-registry-private-configuration-user --from-literal=REGISTRY_STORAGE_AZURE_ACCOUNTKEY=<accountkey> --namespace openshift-image-registry
  ```

## Configuring registry storage for Azure {#registry-configuring-storage-azure-user-infra_configuring-registry-storage-azure-user-infrastructure}

During installation, your cloud credentials are sufficient to create Azure Blob Storage, and the Registry Operator automatically configures storage.

**Prerequisites**

- A cluster on Azure with user-provisioned infrastructure.
- To configure registry storage for Azure, provide Registry Operator cloud credentials.
- For Azure storage the secret is expected to contain one key:
  - `REGISTRY_STORAGE_AZURE_ACCOUNTKEY`

**Procedure**

1. Create an [Azure storage container](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-quickstart-blobs-portal).
2. Fill in the storage configuration in `configs.imageregistry.operator.openshift.io/cluster`:
   ```terminal
   $ oc edit configs.imageregistry.operator.openshift.io/cluster
   ```

   ```yaml title="Example configuration"
   apiVersion: imageregistry.operator.openshift.io/v1
   kind: Config
   metadata:
     name: cluster
   spec:
     storage:
       azure:
         accountName: <storage_account_name>
         container: <container_name>
   ```

## Configuring registry storage for Azure Government {#registry-configuring-storage-azure-gov-cloud_configuring-registry-storage-azure-user-infrastructure}

During installation, your cloud credentials are sufficient to create Azure Blob Storage, and the Registry Operator automatically configures storage.

**Prerequisites**

- A cluster on Azure with user-provisioned infrastructure in a government region.
- To configure registry storage for Azure, provide Registry Operator cloud credentials.
- For Azure storage, the secret is expected to contain one key:
  - `REGISTRY_STORAGE_AZURE_ACCOUNTKEY`

**Procedure**

1. Create an [Azure storage container](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-quickstart-blobs-portal).
2. Fill in the storage configuration in `configs.imageregistry.operator.openshift.io/cluster`:
   ```terminal
   $ oc edit configs.imageregistry.operator.openshift.io/cluster
   ```

   ```yaml title="Example configuration"
   storage:
     azure:
       accountName: <storage-account-name>
       container: <container-name>
       cloudName: AzureUSGovernmentCloud
   ```

   `cloudName` is the name of the Azure cloud environment, which can be used to configure the Azure SDK with the appropriate Azure API endpoints. Defaults to `AzurePublicCloud`. You can also set `cloudName` to `AzureUSGovernmentCloud`, `AzureChinaCloud`, or `AzureGermanCloud` with sufficient credentials.
