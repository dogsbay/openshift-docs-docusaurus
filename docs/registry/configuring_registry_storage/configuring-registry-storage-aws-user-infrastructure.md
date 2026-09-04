---
title: Configuring the registry for AWS user-provisioned infrastructure
sidebar_position: 1
---

# Configuring the registry for AWS user-provisioned infrastructure {#configuring-registry-storage-aws-user-infrastructure}

<a id="configuring-registry-storage-aws-user-infrastructure"></a>

Save your container images to a durable storage location by configuring the built-in image registry to use dedicated AWS storage. This setup provides persistent scalable storage for your registry, separate from ephemeral cluster storage.

## Configuring a secret for the Image Registry Operator {#registry-operator-config-resources-secret-aws_configuring-registry-storage-aws-user-infrastructure}

In addition to the `configs.imageregistry.operator.openshift.io` and ConfigMap resources, configuration is provided to the Operator by a separate secret resource located within the `openshift-image-registry` namespace.

The `image-registry-private-configuration-user` secret provides credentials needed for storage access and management. It overrides the default credentials used by the Operator, if default credentials were found.

For S3 on AWS storage, the secret is expected to contain two keys:

- `REGISTRY_STORAGE_S3_ACCESSKEY`
- `REGISTRY_STORAGE_S3_SECRETKEY`

**Procedure**

- Create an OpenShift Container Platform secret that contains the required keys.
  ```terminal
  $ oc create secret generic image-registry-private-configuration-user --from-literal=REGISTRY_STORAGE_S3_ACCESSKEY=myaccesskey --from-literal=REGISTRY_STORAGE_S3_SECRETKEY=mysecretkey --namespace openshift-image-registry
  ```

## Configuring registry storage for AWS with user-provisioned infrastructure {#registry-configuring-storage-aws-user-infra_configuring-registry-storage-aws-user-infrastructure}

If the Registry Operator cannot automatically create and configure an Amazon S3 bucket during installation, you must manually configure registry storage for your cluster.

:::warning

To secure your registry images in Amazon Web Services (AWS), [block public access](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket-publicaccessblockconfiguration.html) to the S3 bucket.

:::

**Prerequisites**

- You have a cluster on AWS with user-provisioned infrastructure.
- For Amazon S3 storage, the secret must contain two keys:
  - `REGISTRY_STORAGE_S3_ACCESSKEY`
  - `REGISTRY_STORAGE_S3_SECRETKEY`

**Procedure**

1. Set up a [Bucket Lifecycle Policy](https://docs.aws.amazon.com/AmazonS3/latest/dev/mpuoverview.html#mpu-abort-incomplete-mpu-lifecycle-config) to cancel incomplete multipart uploads that are one day old.
2. Enter the storage configuration in `configs.imageregistry.operator.openshift.io/cluster`:
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
       s3:
         bucket: <bucket_name>
         region: <region_name>
   ```

## Image Registry Operator configuration parameters for AWS S3 {#registry-operator-configuration-resource-overview-aws-s3_configuring-registry-storage-aws-user-infrastructure}

The following configuration parameters are available for AWS S3 registry storage.

The image registry `spec.storage.s3` configuration parameter holds the information to configure the registry to use the AWS S3 service for back-end storage. See the [S3 storage driver documentation](https://docs.docker.com/registry/storage-drivers/s3/) for more information.

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
  <td><code>chunkSizeMiB</code></td>
  <td>ChunkSizeMiB is the size of the multipart upload chunks of the S3 API. The default is <code>10</code> MiB with a minimum of <code>5</code> MiB.</td>
</tr>
<tr>
  <td><code>region</code></td>
  <td>Region is the AWS region in which your bucket exists. It is optional and is set based on the installed AWS Region.</td>
</tr>
<tr>
  <td><code>regionEndpoint</code></td>
  <td>RegionEndpoint is the endpoint for S3 compatible storage services. It is optional and defaults based on the Region that is provided.</td>
</tr>
<tr>
  <td><code>virtualHostedStyle</code></td>
  <td>VirtualHostedStyle enables using S3 virtual hosted style bucket paths with a custom RegionEndpoint. It is optional and defaults to false.<br><br>Set this parameter to deploy OpenShift Container Platform to hidden regions.</td>
</tr>
<tr>
  <td><code>encrypt</code></td>
  <td>Encrypt specifies whether or not the registry stores the image in encrypted format. It is optional and defaults to false.</td>
</tr>
<tr>
  <td><code>keyID</code></td>
  <td>KeyID is the KMS key ID to use for encryption. It is optional. Encrypt must be true, or this parameter is ignored.</td>
</tr>
<tr>
  <td><code>cloudFront</code></td>
  <td>CloudFront configures Amazon Cloudfront as the storage middleware in a registry. It is optional.</td>
</tr>
<tr>
  <td><code>trustedCA</code></td>
  <td>The namespace for the config map referenced by <code>trustedCA</code> is <code>openshift-config</code>. The key for the bundle in the config map is <code>ca-bundle.crt</code>. It is optional.</td>
</tr>
</tbody>
</table>

:::note

When the value of the `regionEndpoint` parameter is configured to a URL of a Rados Gateway, an explicit port must not be specified. For example:

```yaml
regionEndpoint: http://rook-ceph-rgw-ocs-storagecluster-cephobjectstore.openshift-storage.svc.cluster.local
```

:::
