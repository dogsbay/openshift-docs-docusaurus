---
title: Configuring the OpenShift API for Data Protection (OADP) with more than one Backup Storage Location
sidebar_position: 1
---

# Configuring the OpenShift API for Data Protection (OADP) with more than one Backup Storage Location {#configuring-oadp-multiple-bsl}

<a id="configuring-oadp-multiple-bsl"></a>

Configure multiple backup storage locations (BSLs) in the Data Protection Application (DPA) to store backups across different regions or storage providers. This provides flexibility and redundancy for your backup strategy.

OADP supports multiple credentials for configuring more than one BSL, so that you can specify the credentials to use with any BSL.

## Configuring the DPA with more than one BSL {#oadp-configuring-dpa-multiple-bsl_configuring-oadp-multiple-bsl}

Configure the `DataProtectionApplication` (DPA) custom resource (CR) with multiple `BackupStorageLocation` (BSL) resources to store backups across different locations using provider-specific credentials. This provides backup distribution and location-specific restore capabilities.

For example, you have configured the following two BSLs:

- Configured one BSL in the DPA and set it as the default BSL.
- Created another BSL independently by using the `BackupStorageLocation` CR.

As you have already set the BSL created through the DPA as the default, you cannot set the independently created BSL again as the default. This means, at any given time, you can set only one BSL as the default BSL.

**Prerequisites**

- You must install the OADP Operator.
- You must create the secrets by using the credentials provided by the cloud provider.

**Procedure**

1. Configure the `DataProtectionApplication` CR with more than one `BackupStorageLocation` CR. See the following example:
   ```yaml title="Example DPA"
   apiVersion: oadp.openshift.io/v1alpha1
   kind: DataProtectionApplication
   #...
   backupLocations:
     - name: aws
       velero:
         provider: aws
         default: true
         objectStorage:
           bucket: <bucket_name>
           prefix: <prefix>
         config:
           region: <region_name>
           profile: "default"
         credential:
           key: cloud
           name: cloud-credentials
     - name: odf
       velero:
         provider: aws
         default: false
         objectStorage:
           bucket: <bucket_name>
           prefix: <prefix>
         config:
           profile: "default"
           region: <region_name>
           s3Url: <url>
           insecureSkipTLSVerify: "true"
           s3ForcePathStyle: "true"
         credential:
           key: cloud
           name: <custom_secret_name_odf>
   #...
   ```

   where:

   <dl>
   <dt><code>name: aws</code></dt>
   <dd>Specifies a name for the first BSL.</dd>
   <dt><code>default: true</code></dt>
   <dd>Indicates that this BSL is the default BSL. If a BSL is not set in the <code>Backup CR</code>, the default BSL is used. You can set only one BSL as the default.</dd>
   <dt><code>&lt;bucket_name&gt;</code></dt>
   <dd>Specifies the bucket name.</dd>
   <dt><code>&lt;prefix&gt;</code></dt>
   <dd>Specifies a prefix for Velero backups. For example, <code>velero</code>.</dd>
   <dt><code>&lt;region_name&gt;</code></dt>
   <dd>Specifies the AWS region for the bucket.</dd>
   <dt><code>cloud-credentials</code></dt>
   <dd>Specifies the name of the default <code>Secret</code> object that you created.</dd>
   <dt><code>name: odf</code></dt>
   <dd>Specifies a name for the second BSL.</dd>
   <dt><code>&lt;url&gt;</code></dt>
   <dd>Specifies the URL of the S3 endpoint.</dd>
   <dt><code>&lt;custom_secret_name_odf&gt;</code></dt>
   <dd>Specifies the correct name for the <code>Secret</code>. For example, <code>custom_secret_name_odf</code>. If you do not specify a <code>Secret</code> name, the default name is used.</dd>
   </dl>
2. Specify the BSL to be used in the backup CR. See the following example.
   ```yaml title="Example backup CR"
   apiVersion: velero.io/v1
   kind: Backup
   # ...
   spec:
     includedNamespaces:
     - <namespace>
     storageLocation: <backup_storage_location>
     defaultVolumesToFsBackup: true
   ```

   where:

   <dl>
   <dt><code>&lt;namespace&gt;</code></dt>
   <dd>Specifies the namespace to back up.</dd>
   <dt><code>&lt;backup_storage_location&gt;</code></dt>
   <dd>Specifies the storage location.</dd>
   </dl>

## Changing the default backup storage location in a DPA {#oadp-changing-default-bsl_configuring-oadp-multiple-bsl}

Understand why the `default` field of the `backupLocations` object in a `DataProtectionApplication` (DPA) does not change the default `BackupStorageLocation` (BSL) when you configure multiple backup locations. This helps you to identify the root cause and apply the correct workaround.

When a DPA includes multiple backup locations, updating the `default` field to change the default BSL does not take effect. After you set `default: true` on a different BSL and set `default: false` on the previously default BSL, the `oc get bsl` command shows that the original BSL remains the default.

Root cause
: The OADP Operator sets the `default` field on a BSL resource only during initial creation of the DPA. On subsequent reconciliations, the Operator preserves the default BSL.

  This behavior is intentional. Velero independently manages the `default` field on BSL resources. If the OADP Operator overwrites this field on every reconciliation, it conflicts with Velero, causing a race condition where both controllers compete over the value.

  To prevent this conflict, the Operator checks whether the BSL already exists. If it exists, the Operator preserves the current `default` value instead of applying the value from the DPA specification.

Workaround
: To change the default BSL, delete and re-create the DPA. Deleting and re-creating the DPA ensures that the BSL resources are created from scratch. The new `default` setting is applied during initial creation and re-deployment of the DPA.

**Procedure**

1. Delete the existing DPA by running the following command:
   ```terminal
   $ oc delete dpa <dpa_name> -n openshift-adp
   ```

   Replace `<dpa_name>` with the name of the DPA custom resource.
2. Re-create the DPA with the updated default BSL configuration by running the following command:
   ```terminal
   $ oc apply -f <updated_dpa_file>
   ```

   Replace `<updated_dpa_file>` with the file name of the updated DPA custom resource definition.

## Configuring two backup BSLs with different cloud credentials {#oadp-multiple-bsl-use-case_configuring-oadp-multiple-bsl}

Configure two backup storage locations with different cloud credentials to back up applications to multiple storage targets. With this setup, you can distribute backups across different storage providers for redundancy.

**Prerequisites**

- You must install the OADP Operator.
- You must configure two backup storage locations: AWS S3 and Multicloud Object Gateway (MCG).
- You must have an application with a database deployed on a Red Hat OpenShift cluster.

**Procedure**

1. Create the first `Secret` for the AWS S3 storage provider with the default name by running the following command:
   ```terminal
   $ oc create secret generic cloud-credentials -n openshift-adp --from-file cloud=<aws_credentials_file_name>
   ```

   where:

   <dl>
   <dt><code>&lt;aws_credentials_file_name&gt;</code></dt>
   <dd>Specifies the name of the cloud credentials file for AWS S3.</dd>
   </dl>
2. Create the second `Secret` for MCG with a custom name by running the following command:
   ```terminal
   $ oc create secret generic mcg-secret -n openshift-adp --from-file cloud=<MCG_credentials_file_name>
   ```

   where:

   <dl>
   <dt><code>&lt;MCG_credentials_file_name&gt;</code></dt>
   <dd>Specifies the name of the cloud credentials file for MCG. Note the name of the <code>mcg-secret</code> custom secret.</dd>
   </dl>
3. Configure the DPA with the two BSLs as shown in the following example.
   ```yaml title="Example DPA"
   apiVersion: oadp.openshift.io/v1alpha1
   kind: DataProtectionApplication
   metadata:
     name: two-bsl-dpa
     namespace: openshift-adp
   spec:
     backupLocations:
     - name: aws
       velero:
         config:
           profile: default
           region: <region_name>
         credential:
           key: cloud
           name: cloud-credentials
         default: true
         objectStorage:
           bucket: <bucket_name>
           prefix: velero
         provider: aws
     - name: mcg
       velero:
         config:
           insecureSkipTLSVerify: "true"
           profile: noobaa
           region: <region_name>
           s3ForcePathStyle: "true"
           s3Url: <s3_url>
         credential:
           key: cloud
           name: mcg-secret
         objectStorage:
           bucket: <bucket_name_mcg>
           prefix: velero
         provider: aws
     configuration:
       nodeAgent:
         enable: true
         uploaderType: kopia
       velero:
         defaultPlugins:
         - openshift
         - aws
   ```

   where:

   <dl>
   <dt><code>&lt;region_name&gt;</code></dt>
   <dd>Specifies the AWS region for the bucket.</dd>
   <dt><code>&lt;bucket_name&gt;</code></dt>
   <dd>Specifies the AWS S3 bucket name.</dd>
   <dt><code>region: &lt;region_name&gt;</code></dt>
   <dd>Specifies the region, following the naming convention of the documentation of MCG.</dd>
   <dt><code>&lt;s3_url&gt;</code></dt>
   <dd>Specifies the URL of the S3 endpoint for MCG.</dd>
   <dt><code>mcg-secret</code></dt>
   <dd>Specifies the name of the custom secret for MCG storage.</dd>
   <dt><code>&lt;bucket_name_mcg&gt;</code></dt>
   <dd>Specifies the MCG bucket name.</dd>
   </dl>
4. Create the DPA by running the following command:
   ```terminal
   $ oc create -f <dpa_file_name>
   ```

   where:

   <dl>
   <dt><code>&lt;dpa_file_name&gt;</code></dt>
   <dd>Specifies the file name of the DPA you configured.</dd>
   </dl>
5. Verify that the DPA has reconciled by running the following command:
   ```terminal
   $ oc get dpa -o yaml
   ```
6. Verify that the BSLs are available by running the following command:
   ```terminal
   $ oc get bsl
   ```

   ```terminal title="Example output"
   NAME   PHASE       LAST VALIDATED   AGE     DEFAULT
   aws    Available   5s               3m28s   true
   mcg    Available   5s               3m28s
   ```
7. Create a backup CR with the default BSL.
   :::note

   In the following example, the `storageLocation` field is not specified in the backup CR.

   :::

   ```yaml title="Example backup CR"
   apiVersion: velero.io/v1
   kind: Backup
   metadata:
     name: test-backup1
     namespace: openshift-adp
   spec:
     includedNamespaces:
     - <mysql_namespace>
     defaultVolumesToFsBackup: true
   ```

   where:

   <dl>
   <dt><code>&lt;mysql_namespace&gt;</code></dt>
   <dd>Specifies the namespace for the application installed in the cluster.</dd>
   </dl>
8. Create a backup by running the following command:
   ```terminal
   $ oc apply -f <backup_file_name>
   ```

   where:

   <dl>
   <dt><code>&lt;backup_file_name&gt;</code></dt>
   <dd>Specifies the name of the backup CR file.</dd>
   </dl>
9. Verify that the backup completed with the default BSL by running the following command:
   ```terminal
   $ oc get backups.velero.io <backup_name> -o yaml
   ```

   where:

   <dl>
   <dt><code>&lt;backup_name&gt;</code></dt>
   <dd>Specifies the name of the backup.</dd>
   </dl>
10. Create a backup CR by using MCG as the BSL. In the following example, note that the second `storageLocation` value is specified at the time of backup CR creation.
    ```yaml title="Example backup CR"
    apiVersion: velero.io/v1
    kind: Backup
    metadata:
      name: test-backup1
      namespace: openshift-adp
    spec:
      includedNamespaces:
      - <mysql_namespace>
      storageLocation: mcg
      defaultVolumesToFsBackup: true
    ```

    where:

    <dl>
    <dt><code>&lt;mysql_namespace&gt;</code></dt>
    <dd>Specifies the namespace for the application installed in the cluster.</dd>
    <dt><code>mcg</code></dt>
    <dd>Specifies the second storage location.</dd>
    </dl>
11. Create a second backup by running the following command:
    ```terminal
    $ oc apply -f <backup_file_name>
    ```

    where:

    <dl>
    <dt><code>&lt;backup_file_name&gt;</code></dt>
    <dd>Specifies the name of the backup CR file.</dd>
    </dl>
12. Verify that the backup completed with the storage location as MCG by running the following command:
    ```terminal
    $ oc get backups.velero.io <backup_name> -o yaml
    ```

    where:

    <dl>
    <dt><code>&lt;backup_name&gt;</code></dt>
    <dd>Specifies the name of the backup.</dd>
    </dl>

**Additional resources**

- [Creating profiles for different credentials](/docs/backup_and_restore/application_backup_and_restore/installing/installing-oadp-aws#oadp-aws-secrets-for-different-credentials_installing-oadp-aws)
