---
title: Configuring the registry for Google Cloud user-provisioned infrastructure
sidebar_position: 2
---

# Configuring the registry for Google Cloud user-provisioned infrastructure {#configuring-registry-storage-gcp-user-infrastructure}

<a id="configuring-registry-storage-gcp-user-infrastructure"></a>

Save your container images to a durable storage location by configuring the built-in image registry to use dedicated Google Cloud storage. This setup provides persistent scalable storage for your registry, separate from ephemeral cluster storage.

## Configuring a secret for the Image Registry Operator {#registry-operator-config-resources-secret-gcp_configuring-registry-storage-gcp-user-infrastructure}

In addition to the `configs.imageregistry.operator.openshift.io` and ConfigMap resources, configuration is provided to the Operator by a separate secret resource located within the `openshift-image-registry` namespace.

The `image-registry-private-configuration-user` secret provides credentials needed for storage access and management. It overrides the default credentials used by the Operator, if default credentials were found.

For GCS on Google Cloud storage, the secret is expected to contain one key whose value is the contents of a credentials file provided by Google Cloud:

- `REGISTRY_STORAGE_GCS_KEYFILE`

**Procedure**

- Create an OpenShift Container Platform secret that contains the required keys.
  ```terminal
  $ oc create secret generic image-registry-private-configuration-user --from-file=REGISTRY_STORAGE_GCS_KEYFILE=<path_to_keyfile> --namespace openshift-image-registry
  ```

## Configuring the registry storage for Google Cloud with user-provisioned infrastructure {#registry-configuring-storage-gcp-user-infra_configuring-registry-storage-gcp-user-infrastructure}

If the Registry Operator cannot create a Google Cloud bucket, you must set up the storage medium manually and configure the settings in the registry custom resource (CR).

**Prerequisites**

- A cluster on Google Cloud with user-provisioned infrastructure.
- To configure registry storage for Google Cloud, you need to provide Registry Operator cloud credentials.
- For GCS on Google Cloud storage, the secret is expected to contain one key whose value is the contents of a credentials file provided by Google Cloud:
  - `REGISTRY_STORAGE_GCS_KEYFILE`

  :::warning

  You can secure your registry images that use a Google Cloud Storage bucket by setting [public access prevention](https://cloud.google.com/storage/docs/using-public-access-prevention).

  :::

**Procedure**

1. Set up an [Object Lifecycle Management policy](https://cloud.google.com/storage/docs/lifecycle) to abort incomplete multipart uploads that are one day old.
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
       gcs:
         bucket: <bucket_name>
         projectID: <project_id>
         region: <region_name>
   ```

## Image Registry Operator configuration parameters for Google Cloud GCS {#registry-operator-configuration-resource-overview-gcp-gcs_configuring-registry-storage-gcp-user-infrastructure}

The following parameters configure are available to configure your Google Cloud GCS registry storage.

<table>
<thead>
<tr>
  <th>Parameter</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>bucket</code></td>
  <td>Bucket is the bucket name in which you want to store the registry's data. It is optional and is generated if not provided.</td>
</tr>
<tr>
  <td><code>region</code></td>
  <td>Region is the GCS location in which your bucket exists. It is optional and is set based on the installed GCS Region.</td>
</tr>
<tr>
  <td><code>projectID</code></td>
  <td>ProjectID is the Project ID of the Google Cloud project that this bucket should be associated with. It is optional.</td>
</tr>
<tr>
  <td><code>keyID</code></td>
  <td>KeyID is the KMS key ID to use for encryption. It is optional because buckets are encrypted by default on Google Cloud. This allows for the use of a custom encryption key.</td>
</tr>
</tbody>
</table>
