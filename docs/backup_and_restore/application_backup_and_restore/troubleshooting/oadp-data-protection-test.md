---
title: OADP Data protection test
sidebar_position: 10
---

# OADP Data protection test

<a id="oadp-data-protection-test"></a>

Validate your OADP configuration by using the `DataProtectionTest` (DPT) custom resource (CR). This helps you ensure your data protection environment is properly configured and performing according to your requirements before performing backups.

The DPT checks the upload performance of backups to object storage, CSI snapshot readiness for persistent volume claims, and storage bucket configuration such as encryption and versioning.

## OADP DataProtectionTest CR specification fields {#oadp-dpt-spec_oadp-data-protection-test}

Review the specification fields available in the `DataProtectionTest` (DPT) custom resource (CR) to configure backup location, upload speed tests, CSI volume snapshot tests, and other options. This helps you customize the DPT CR to validate your specific OADP configuration requirements.

**DPT CR spec fields**

| Field | Type | Description |
| --- | --- | --- |
| `backupLocationName` | string | Name of the `BackupStorageLocation` CR configured in the `DataProtectionApplication` (DPA) CR. |
| `backupLocationSpec` | object | Inline specification of the `BackupStorageLocation` CR. |
| `uploadSpeedTestConfig` | object | Configuration to run an upload speed test to the object storage. |
| `csiVolumeSnapshotTestConfigs` | list | List of persistent volume claims to take a snapshot of and to verify the snapshot readiness. |
| `forceRun` | boolean | Re-run the DPT CR even if status is `Complete` or `Failed`. |
| `skipTLSVerify` | boolean | Bypasses the TLS certificate validation if set to `true`. |

## OADP DataProtectionTest CR status fields {#oadp-dpt-status_oadp-data-protection-test}

Review the status fields in the `DataProtectionTest` (DPT) custom resource (CR) to monitor test progress, upload speed results, bucket metadata, and snapshot test outcomes. This helps you interpret the DPT CR results and identify any issues with your OADP configuration.

**DPT CR status fields**

| Field | Type | Description |
| --- | --- | --- |
| `phase` | string | Current phase of the DPT CR. Values are `InProgress`, `Complete`, or `Failed`. |
| `lastTested` | timestamp | The timestamp when the DPT CR was last run. |
| `uploadTest` | object | Results of the upload speed test. |
| `bucketMetadata` | object | Information about the storage bucket encryption and versioning. |
| `snapshotTests` | list | Snapshot test results for each persistent volume claim. |
| `snapshotSummary` | string | Aggregated pass/fail summary for snapshots. For example, `2/2 passed`. |
| `s3Vendor` | string | AWS S3-compatible storage bucket vendors. For example, AWS, MinIO, Ceph. |
| `errorMessage` | string | Error message if the DPT CR fails. |

## Using the DataProtectionTest custom resource {#using-data-protection-test_oadp-data-protection-test}

Configure and run the `DataProtectionTest` (DPT) custom resource (CR) to verify Container Storage Interface (CSI) snapshot readiness and data upload performance to your storage bucket. This helps you validate your OADP environment before performing backup and restore operations.

**Prerequisites**

- You have logged in to the OpenShift Container Platform cluster as a user with the `cluster-admin` role.
- You have installed the OpenShift CLI (`oc`).
- You have installed the OADP Operator.
- You have created the `DataProtectionApplication` (DPA) CR.
- You have configured a backup storage location (BSL) to store the backups.
- You have an application with persistent volume claims (PVCs) running in a separate namespace.

**Procedure**

1. Create a manifest file for the DPT CR as shown in the example:
   ```yaml
   apiVersion: oadp.openshift.io/v1alpha1
   kind: DataProtectionTest
   metadata:
     name: dpt-sample
     namespace: openshift-adp
   spec:
     backupLocationName: <bsl_name>
     csiVolumeSnapshotTestConfigs:
     - snapshotClassName: csi-gce-pd-vsc
       timeout: 90s
       volumeSnapshotSource:
         persistentVolumeClaimName: <pvc1_name>
         persistentVolumeClaimNamespace: <pvc_namespace>
     - snapshotClassName: csi-gce-pd-vsc
       timeout: 120s
       volumeSnapshotSource:
         persistentVolumeClaimName: <pvc2_name>
         persistentVolumeClaimNamespace: <pvc_namespace>
     forceRun: false
     uploadSpeedTestConfig:
       fileSize: 200MB
       timeout: 120s
   ```

   where:

   <dl>
   <dt><code>&lt;bsl_name&gt;</code></dt>
   <dd>Specifies the name of the BSL.</dd>
   <dt><code>csiVolumeSnapshotTestConfigs</code></dt>
   <dd>Specifies a list for <code>csiVolumeSnapshotTestConfigs</code>. In this example, two PVCs are being tested.</dd>
   <dt><code>&lt;pvc1_name&gt;</code></dt>
   <dd>Specifies the name of the first PVC.</dd>
   <dt><code>&lt;pvc_namespace&gt;</code></dt>
   <dd>Specifies the namespace of the PVC.</dd>
   <dt><code>&lt;pvc2_name&gt;</code></dt>
   <dd>Specifies the name of the second PVC.</dd>
   <dt><code>forceRun</code></dt>
   <dd>Set to <code>false</code> if you want to make the OADP controller skip re-running tests.</dd>
   <dt><code>uploadSpeedTestConfig</code></dt>
   <dd>Configures the upload speed test by setting the <code>fileSize</code> and <code>timeout</code> fields.</dd>
   </dl>
2. Create the DPT CR by running the following command:
   ```terminal
   $ oc create -f <dpt_file_name>
   ```

   Replace `<dpt_file_name>` with the file name of the DPT manifest.

**Verification**

1. Verify that the phase of the DPT CR is `Complete` by running the following command:
   ```terminal
   $ oc get dpt dpt-sample
   ```

   The example output is as following:

   ```terminal
   NAME         PHASE      LASTTESTED   UPLOADSPEED(MBPS)   ENCRYPTION   VERSIONING   SNAPSHOTS    AGE
   dpt-sample   Complete   17m          546                 AES256       Enabled      2/2 passed   17m
   ```
2. Verify that the CSI snapshots are ready and the data upload tests are successful by running the following command:
   ```terminal
   $ oc get dpt dpt-sample -o yaml
   ```

   The example output is as following:

   ```yaml
   apiVersion: oadp.openshift.io/v1alpha1
   kind: DataProtectionTest
   ....
   status:
     bucketMetadata:
       encryptionAlgorithm: AES256
       versioningStatus: Enabled
     lastTested: "202...:47:51Z"
     phase: Complete
     s3Vendor: AWS
     snapshotSummary: 2/2 passed
     snapshotTests:
     - persistentVolumeClaimName: mysql-data
       persistentVolumeClaimNamespace: ocp-mysql
       readyDuration: 24s
       status: Ready
     - persistentVolumeClaimName: mysql-data1
       persistentVolumeClaimNamespace: ocp-mysql
       readyDuration: 40s
       status: Ready
     uploadTest:
       duration: 3.071s
       speedMbps: 546
       success: true
   ```

   where:

   <dl>
   <dt><code>bucketMetadata</code></dt>
   <dd>Specifies the bucket metadata information.</dd>
   <dt><code>s3Vendor</code></dt>
   <dd>Specifies the S3 bucket vendor.</dd>
   <dt><code>snapshotSummary</code></dt>
   <dd>Specifies the summary of the CSI snapshot tests.</dd>
   <dt><code>uploadTest</code></dt>
   <dd>Specifies the upload test details.</dd>
   </dl>

## Running a data protection test by configuring a backup storage location specification {#oadp-dpt-use-case-bsl-spec_oadp-data-protection-test}

Configure and run the `DataProtectionTest` (DPT) custom resource (CR) by specifying an inline backup storage location (BSL) specification instead of referencing an existing BSL name. This helps you test data upload performance and CSI snapshot readiness without creating a separate BSL resource.

**Prerequisites**

- You have logged in to the OpenShift Container Platform cluster as a user with the `cluster-admin` role.
- You have installed the OpenShift CLI (`oc`).
- You have installed the OADP Operator.
- You have created the `DataProtectionApplication` (DPA) CR.
- You have configured a bucket to store the backups.
- You have created the `Secret` object to access the bucket storage.
- You have an application with persistent volume claims (PVCs) running in a separate namespace.

**Procedure**

1. Create a manifest file for the DPT CR as shown in the example:
   ```yaml
   apiVersion: oadp.openshift.io/v1alpha1
   kind: DataProtectionTest
   metadata:
     name: dpt-sample
     namespace: openshift-adp
   spec:
     backupLocationSpec:
       provider: aws
       default: true
       objectStorage:
         bucket: sample-bucket
         prefix: velero
       config:
         region: us-east-1
         profile: "default"
         insecureSkipTLSVerify: "true"
         s3Url: "https://s3.amazonaws.com/sample-bucket"
       credential:
         name: cloud-credentials
         key: cloud
     uploadSpeedTestConfig:
       fileSize: 50MB
       timeout: 120s
     csiVolumeSnapshotTestConfigs:
       - volumeSnapshotSource:
           persistentVolumeClaimName: mongo
           persistentVolumeClaimNamespace: mongo-persistent
         snapshotClassName: csi-snapclass
         timeout: 2m
     forceRun: true
     skipTLSVerify: true
   ```

   where:

   <dl>
   <dt><code>backupLocationSpec</code></dt>
   <dd>Configures the BSL spec by specifying details such as the cloud provider.</dd>
   <dt><code>sample-bucket</code></dt>
   <dd>Specifies the bucket name. In this example, the bucket name is <code>sample-bucket</code>.</dd>
   <dt><code>us-east-1</code></dt>
   <dd>Specifies the cloud provider region.</dd>
   <dt><code>credential</code></dt>
   <dd>Specifies the cloud credentials for the storage bucket.</dd>
   <dt><code>uploadSpeedTestConfig</code></dt>
   <dd>(Optional) Configures the upload speed test by setting the <code>fileSize</code> and <code>timeout</code> fields.</dd>
   <dt><code>csiVolumeSnapshotTestConfigs</code></dt>
   <dd>Configures the CSI volume snapshot test.</dd>
   <dt><code>skipTLSVerify</code></dt>
   <dd>Set to <code>true</code> to skip the TLS certificate validation during the DPT CR run.</dd>
   </dl>
2. Create the DPT CR by running the following command:
   ```terminal
   $ oc create -f <dpt_file_name>
   ```

   Replace `<dpt_file_name>` with the file name of the DPT manifest.

**Verification**

1. Verify that the phase of the DPT CR is `Complete` by running the following command:
   ```terminal
   $ oc get dpt dpt-sample
   ```

   The example output is as following:

   ```terminal
   NAME         PHASE      LASTTESTED   UPLOADSPEED(MBPS)   ENCRYPTION   VERSIONING   SNAPSHOTS    AGE
   dpt-sample   Complete   17m          546                 AES256       Enabled      2/2 passed   17m
   ```

## Running a data protection test on an Azure object storage {#oadp-dpt-use-case-azure_oadp-data-protection-test}

Run the `DataProtectionTest` (DPT) custom resource (CR) on Azure object storage by configuring the required Azure credentials, including the `STORAGE_ACCOUNT_ID` parameter in the secret object. This helps you validate your OADP configuration and verify CSI snapshot readiness on Azure clusters.

**Prerequisites**

- You have logged in to the Azure cluster as a user with the `cluster-admin` role.
- You have installed the OpenShift CLI (`oc`).
- You have installed the OADP Operator.
- You have configured a bucket to store the backups.
- You have an application with persistent volume claims (PVCs) running in a separate namespace.

**Procedure**

1. Add the `Storage Blob Data Contributor` role to Azure `storageAccount` object to avoid DPT run failure. Run the following command:
   ```terminal
   $ az role assignment create \
   --assignee "$AZURE_CLIENT_ID" \
   --role "Storage Blob Data Contributor" \
   --scope "/subscriptions/$AZURE_SUBSCRIPTION_ID/resourceGroups/$AZURE_RESOURCE_GROUP/providers/Microsoft.Storage/storageAccounts/$AZURE_STORAGE_ACCOUNT_ID"
   ```
2. In your terminal, export the Azure parameters and create a secret credentials file with the parameters as shown in the following example.
   To run the DPT CR on Azure, you need to specify the `STORAGE_ACCOUNT_ID` parameter in the secret credentials file.

   ```terminal
   AZURE_SUBSCRIPTION_ID=<subscription_id>
   AZURE_TENANT_ID=<tenant_id>
   AZURE_CLIENT_ID=<client_id>
   AZURE_CLIENT_SECRET=<client_secret>
   AZURE_RESOURCE_GROUP=<resource_group>
   AZURE_STORAGE_ACCOUNT_ID=<storage_account>
   ```
3. Create the `Secret` CR as shown in the following example:
   ```terminal
   $ oc create secret generic cloud-credentials-azure -n openshift-adp --from-file cloud=<credentials_file_path>
   ```
4. Create the `DataProtectionApplication` (DPA) CR by using the configuration shown in the following example:
   ```yaml
   apiVersion: oadp.openshift.io/v1alpha1
   kind: DataProtectionApplication
   metadata:
     name: ts-dpa
     namespace: openshift-adp
   spec:
     configuration:
       velero:
         defaultPlugins:
           - azure
           - openshift
     backupLocations:
       - velero:
           config:
             resourceGroup: oadp-....-b7q4-rg
             storageAccount: oadp...kb7q4
             subscriptionId: 53b8f5...fd54c8a
           credential:
             key: cloud
             name: cloud-credentials-azure
           provider: azure
           default: true
           objectStorage:
             bucket: <bucket_name>
             prefix: velero
   ```

   Replace `name` with the name of the `Secret` object. In this example, the name is `cloud-credentials-azure`.
5. Create the DPT CR by specifying the name of backup storage location (BSL), `VolumeSnapshotClass` object, and the persistent volume claim details as shown in the following example:
   ```yaml
   apiVersion: oadp.openshift.io/v1alpha1
   kind: DataProtectionTest
   metadata:
     name: dpt-sample
     namespace: openshift-adp
   spec:
     backupLocationName: <bsl_name>
     uploadSpeedTestConfig:
       fileSize: 40MB
       timeout: 120s
     csiVolumeSnapshotTestConfigs:
       - snapshotClassName: csi-azuredisk-vsc
         timeout: 90s
         volumeSnapshotSource:
           persistentVolumeClaimName: mysql-data
           persistentVolumeClaimNamespace: ocp-mysql
       - snapshotClassName: csi-azuredisk-vsc
         timeout: 120s
         volumeSnapshotSource:
           persistentVolumeClaimName: mysql-data1
           persistentVolumeClaimNamespace: ocp-mysql
   ```

   where:

   <dl>
   <dt><code>&lt;bsl_name&gt;</code></dt>
   <dd>Specifies the name of the BSL.</dd>
   <dt><code>csi-azuredisk-vsc</code></dt>
   <dd>Specifies the Azure snapshot class name.</dd>
   <dt><code>mysql-data</code></dt>
   <dd>Specifies the name of the persistent volume claim.</dd>
   <dt><code>ocp-mysql</code></dt>
   <dd>Specifies the name of the persistent volume claim namespace.</dd>
   </dl>
6. Run the DPT CR to verify the snapshot readiness.

## Troubleshooting the DataProtectionTest custom resource {#oadp-troubleshooting-dpt_oadp-data-protection-test}

Troubleshoot common `DataProtectionTest` (DPT) custom resource (CR) issues, such as stuck progress states, upload test failures, and snapshot test failures. This helps you identify and resolve problems with your DPT configuration.

**DPT CR troubleshooting**

| Error | Reason | Solution |
| --- | --- | --- |
| DPT stuck in `InProgress` state | Bucket credentials or bucket access failure | Check `Secret` object, bucket permissions, and logs. |
| Upload test failed | Incorrect `Secret` object or S3 endpoint | Check the `BackupStorageLocation` object config and the access keys. |
| Snapshot tests fail | Incorrect configuration of CSI snapshot controller | Check the `VolumeSnapshotClass` object availability and the CSI driver logs. |
| Bucket encryption or versioning not populated | Cloud provider limitations | Not all object storage providers expose these fields consistently. |
