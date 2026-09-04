---
title: OADP Self-Service namespace admin use cases
sidebar_position: 3
---

# OADP Self-Service namespace admin use cases

<a id="oadp-self-service-namespace-admin-use-cases"></a>

Use OADP Self-Service as a namespace administrator to create backup storage locations, perform backup and restore operations, and review operation logs for your authorized namespaces. This helps you to manage data protection independently without cluster admin access.

## Creating a NonAdminBackupStorageLocation CR {#oadp-self-service-creating-nabsl_oadp-self-service-namespace-admin-use-cases}

Create a `NonAdminBackupStorageLocation` (NABSL) custom resource (CR) to define backup storage locations in your authorized namespace. With this feature, you can store backups in a cloud storage that meets your application requirements.

**Prerequisites**

- You are logged in to the cluster as a namespace admin user.
- The cluster administrator has installed the OADP Operator.
- The cluster administrator has configured the `DataProtectionApplication` (DPA) CR to enable OADP Self-Service.
- The cluster administrator has created a namespace for you and has authorized you to operate from that namespace.

**Procedure**

1. Create a `Secret` CR by using the cloud credentials file content for your cloud provider. Run the following command:
   ```terminal
   $ oc create secret generic cloud-credentials -n test-nac-ns --from-file <cloud_key_name>=<cloud_credentials_file>
   ```

   where:

   <dl>
   <dt><code>&lt;cloud_key_name&gt;</code></dt>
   <dd>Specifies the cloud provider key name. In this example, the <code>Secret</code> name is <code>cloud-credentials</code> and the authorized namespace name is <code>test-nac-ns</code>.</dd>
   </dl>

`<cloud_credentials_file>`
: Specifies the cloud credentials file name.

1. To create a `NonAdminBackupStorageLocation` CR, create a YAML manifest file with the following configuration:
   ```yaml title="Example NonAdminBackupStorageLocation CR"
   apiVersion: oadp.openshift.io/v1alpha1
   kind: NonAdminBackupStorageLocation
   metadata:
     name: test-nabsl
     namespace: test-nac-ns
   spec:
     backupStorageLocationSpec:
       config:
         profile: default
         region: <region_name>
       credential:
         key: cloud
         name: cloud-credentials
       objectStorage:
         bucket: <bucket_name>
         prefix: velero
       provider: aws
   ```

   where:

   <dl>
   <dt><code>namespace</code></dt>
   <dd>Specifies the namespace you are authorized to operate from. For example, <code>test-nac-ns</code>.</dd>
   <dt><code>&lt;region_name&gt;</code></dt>
   <dd>Specifies the region name for your cloud provider.</dd>
   <dt><code>&lt;bucket_name&gt;</code></dt>
   <dd>Specifies the bucket name for storing backups.</dd>
   </dl>
2. To apply the NABSL CR configuration, run the following command:
   ```terminal
   $ oc apply -f <nabsl_cr_filename>
   ```

   Replace `<nabsl_cr_filename>` with the file name containing the NABSL CR configuration.

**Verification**

1. To verify that the NABSL CR is in the `New` phase and is pending administrator approval, run the following command:
   ```terminal
   $ oc get nabsl test-nabsl -o yaml
   ```

   ```yaml title="Example output"
   apiVersion: oadp.openshift.io/v1alpha1
   kind: NonAdminBackupStorageLocation
   ...
   status:
     conditions:
     - lastTransitionTime: "2025-02-26T09:07:15Z"
       message: NonAdminBackupStorageLocation spec validation successful
       reason: BslSpecValidation
       status: "True"
       type: Accepted
     - lastTransitionTime: "2025-02-26T09:07:15Z"
       message: NonAdminBackupStorageLocationRequest approval pending
       reason: BslSpecApprovalPending
       status: "False"
       type: ClusterAdminApproved
     phase: New
     veleroBackupStorageLocation:
       nacuuid: test-nac-test-bsl-c...d4389a1930
       name: test-nac-test-bsl-cd....1930
       namespace: openshift-adp
   ```

   where:

   <dl>
   <dt><code>message</code></dt>
   <dd>Contains the <code>NonAdminBackupStorageLocationRequest approval pending</code> message.</dd>
   <dt><code>phase</code></dt>
   <dd>Specifies the status of the phase. In this example, the phase is <code>New</code>.</dd>
   </dl>
2. After the cluster administrator approves the `NonAdminBackupStorageLocationRequest` CR request, verify that the NABSL CR is successfully created by running the following command:
   ```terminal
   $ oc get nabsl test-nabsl -o yaml
   ```

   ```yaml title="Example output"
   apiVersion: oadp.openshift.io/v1alpha1
   kind: NonAdminBackupStorageLocation
   metadata:
     creationTimestamp: "2025-02-19T09:30:34Z"
     finalizers:
     - nonadminbackupstoragelocation.oadp.openshift.io/finalizer
     generation: 1
     name: test-nabsl
     namespace: test-nac-ns
     resourceVersion: "159973"
     uid: 4a..80-3260-4ef9-a3..5a-00...d1922
   spec:
     backupStorageLocationSpec:
       credential:
         key: cloud
         name: cloud-credentials
       objectStorage:
         bucket: oadp...51rrdqj
         prefix: velero
       provider: aws
   status:
     conditions:
     - lastTransitionTime: "2025-02-19T09:30:34Z"
       message: NonAdminBackupStorageLocation spec validation successful
       reason: BslSpecValidation
       status: "True"
       type: Accepted
     - lastTransitionTime: "2025-02-19T09:30:34Z"
       message: Secret successfully created in the OADP namespace
       reason: SecretCreated
       status: "True"
       type: SecretSynced
     - lastTransitionTime: "2025-02-19T09:30:34Z"
       message: BackupStorageLocation successfully created in the OADP namespace
       reason: BackupStorageLocationCreated
       status: "True"
       type: BackupStorageLocationSynced
     phase: Created
     veleroBackupStorageLocation:
       nacuuid: test-nac-..f933a-4ec1-4f6a-8099-ee...b8b26
       name: test-nac-test-nabsl-36...11ab8b26
       namespace: openshift-adp
       status:
         lastSyncedTime: "2025-02-19T11:47:10Z"
         lastValidationTime: "2025-02-19T11:47:31Z"
         phase: Available
   ```

   where:

   <dl>
   <dt><code>message: NonAdminBackupStorageLocation spec validation successful</code></dt>
   <dd>Specifies that the NABSL <code>spec</code> is validated and approved by the cluster administrator.</dd>
   <dt><code>message: Secret successfully created in the OADP namespace</code></dt>
   <dd>Specifies that the <code>secret</code> object is successfully created in the <code>openshift-adp</code> namespace.</dd>
   <dt><code>message: BackupStorageLocation successfully created in the OADP namespace</code></dt>
   <dd>Specifies that the associated <code>Velero</code> <code>BackupStorageLocation</code> is successfully created in the <code>openshift-adp</code> namespace.</dd>
   <dt><code>nacuuid</code></dt>
   <dd>Specifies the NAC that is orchestrating the NABSL CR.</dd>
   <dt><code>name</code></dt>
   <dd>Specifies the name of the associated <code>Velero</code> backup storage location object.</dd>
   <dt><code>phase: Available</code></dt>
   <dd>Specifies that the NABSL is ready for use.</dd>
   </dl>

## Creating a NonAdminBackup CR {#oadp-self-service-creating-nab_oadp-self-service-namespace-admin-use-cases}

Create a `NonAdminBackup` (NAB) custom resource (CR) to back up application resources in your authorized namespace. This helps you to protect your application data and configuration without requiring cluster administrator privileges.

After you create a NAB CR, the CR undergoes the following phases:

- The initial phase for the CR is `New`.
- The CR creation request goes to the `NonAdminController` (NAC) for reconciliation and validation.
- Upon successful validation and creation of the `Velero` backup object, the `status.phase` field of the NAB CR is updated to the next phase, which is, `Created`.

Review the following important points when creating a NAB CR:

- The `NonAdminBackup` CR creates the `Velero` backup object securely so that other namespace admin users cannot access the CR.
- As a namespace admin user, you can only specify your authorized namespace in the NAB CR. You get an error when you specify a namespace you are not authorized to use.

**Prerequisites**

- You are logged in to the cluster as a namespace admin user.
- The cluster administrator has installed the OADP Operator.
- The cluster administrator has configured the `DataProtectionApplication` (DPA) CR to enable OADP Self-Service.
- The cluster administrator has created a namespace for you and has authorized you to operate from that namespace.
- Optional: You can create and use a `NonAdminBackupStorageLocation` (NABSL) CR to store the backup data. If you do not use a NABSL CR, then the backup is stored in the default backup storage location configured in the DPA.

**Procedure**

1. To create a `NonAdminBackup` CR, create a YAML manifest file with the following configuration:
   ```yaml title="Example NonAdminBackup CR"
   apiVersion: oadp.openshift.io/v1alpha1
   kind: NonAdminBackup
   metadata:
     name: test-nab
   spec:
     backupSpec:
       defaultVolumesToFsBackup: true
       snapshotMoveData: false
       storageLocation: test-bsl
   ```

   where:

   <dl>
   <dt><code>name</code></dt>
   <dd>Specifies a name for the NAB CR. For example, <code>test-nab</code>.</dd>
   <dt><code>defaultVolumesToFsBackup</code></dt>
   <dd>Specifies whether to use File System Backup (FSB). Set to <code>true</code> to use FSB.</dd>
   <dt><code>snapshotMoveData</code></dt>
   <dd>Specifies whether to back up data volumes by using the Data Mover. Set to <code>true</code> to use Data Mover. This example uses FSB for backup.</dd>
   <dt><code>storageLocation</code></dt>
   <dd>Specifies a NABSL CR as a storage location. If you do not set a <code>storageLocation</code>, then the default backup storage location configured in the DPA is used.</dd>
   </dl>
2. To apply the NAB CR configuration, run the following command:
   ```terminal
   $ oc apply -f <nab_cr_filename>
   ```

   Replace `<nab_cr_filename>` with the file name containing the NAB CR configuration.

**Verification**

- To verify that the NAB CR is successfully created, run the following command:
  ```terminal
  $ oc get nab test-nab -o yaml
  ```

  ```yaml title="Example output"
  apiVersion: oadp.openshift.io/v1alpha1
  kind: NonAdminBackup
  metadata:
    creationTimestamp: "2025-03-06T10:02:56Z"
    finalizers:
    - nonadminbackup.oadp.openshift.io/finalizer
    generation: 2
    name: test-nab
    namespace: test-nac-ns
    resourceVersion: "134316"
    uid: c5...4c8a8
  spec:
    backupSpec:
      csiSnapshotTimeout: 0s
      defaultVolumesToFsBackup: true
      hooks: {}
      itemOperationTimeout: 0s
      metadata: {}
      storageLocation: test-bsl
      ttl: 0s
  status:
    conditions:
    - lastTransitionTime: "202...56Z"
      message: backup accepted
      reason: BackupAccepted
      status: "True"
      type: Accepted
    - lastTransitionTime: "202..T10:02:56Z"
      message: Created Velero Backup object
      reason: BackupScheduled
      status: "True"
      type: Queued
    dataMoverDataUploads: {}
    fileSystemPodVolumeBackups:
      completed: 2
      total: 2
    phase: Created
    queueInfo:
      estimatedQueuePosition: 0
    veleroBackup:
      nacuuid: test-nac-test-nab-d2...a9b14
      name: test-nac-test-nab-d2...b14
      namespace: openshift-adp
      spec:
        csiSnapshotTimeout: 10m0s
        defaultVolumesToFsBackup: true
        excludedResources:
        - nonadminbackups
        - nonadminrestores
        - nonadminbackupstoragelocations
        - securitycontextconstraints
        - clusterroles
        - clusterrolebindings
        - priorityclasses
        - customresourcedefinitions
        - virtualmachineclusterinstancetypes
        - virtualmachineclusterpreferences
        hooks: {}
        includedNamespaces:
        - test-nac-ns
        itemOperationTimeout: 4h0m0s
        metadata: {}
        snapshotMoveData: false
        storageLocation: test-nac-test-bsl-bf..02b70a
        ttl: 720h0m0s
      status:
        completionTimestamp: "2025-0..3:13Z"
        expiration: "2025..2:56Z"
        formatVersion: 1.1.0
        hookStatus: {}
        phase: Completed
        progress:
          itemsBackedUp: 46
          totalItems: 46
        startTimestamp: "2025-..56Z"
        version: 1
        warnings: 1
  ```

  where:

  <dl>
  <dt><code>namespace</code></dt>
  <dd>Specifies the namespace name that the <code>NonAdminController</code> CR sets on the <code>Velero</code> backup object to back up.</dd>
  <dt><code>message: backup accepted</code></dt>
  <dd>Specifies that the NAC has reconciled and validated the NAB CR and has created the <code>Velero</code> backup object.</dd>
  <dt><code>fileSystemPodVolumeBackups</code></dt>
  <dd>Specifies the number of volumes that are backed up by using FSB.</dd>
  <dt><code>phase: Created</code></dt>
  <dd>Specifies that the NAB CR is in the <code>Created</code> phase.</dd>
  <dt><code>estimatedQueuePosition</code></dt>
  <dd>Specifies the queue position of the backup object. There can be multiple backups in process, and each backup object is assigned a queue position. When the backup is complete, the queue position is set to <code>0</code>.</dd>
  <dt><code>nacuuid</code></dt>
  <dd>Specifies that the NAC creates the <code>Velero</code> backup object and sets the value for the <code>nacuuid</code> field.</dd>
  <dt><code>name</code></dt>
  <dd>Specifies the name of the associated <code>Velero</code> backup object.</dd>
  <dt><code>status</code></dt>
  <dd>Specifies the status of the <code>Velero</code> backup object.</dd>
  <dt><code>phase: Completed</code></dt>
  <dd>Specifies that the <code>Velero</code> backup object is in the <code>Completed</code> phase and the backup is successful.</dd>
  </dl>

## Deleting a NonAdminBackup CR {#oadp-self-service-deleting-nab_oadp-self-service-namespace-admin-use-cases}

As a namespace admin user, you can delete a `NonAdminBackup` (NAB) custom resource (CR).

**Prerequisites**

- You are logged in to the cluster as a namespace admin user.
- The cluster administrator has installed the OADP Operator.
- The cluster administrator has configured the `DataProtectionApplication` (DPA) CR to enable OADP Self-Service.
- The cluster administrator has created a namespace for you and has authorized you to operate from that namespace.
- You have created a NAB CR in your authorized namespace.

**Procedure**

1. Edit the `NonAdminBackup` CR YAML manifest file by running the following command:
   ```terminal
   $ oc edit <nab_cr> -n <authorized_namespace>
   ```

   where:

   <dl>
   <dt><code>&lt;nab_cr&gt;</code></dt>
   <dd>Specifies the name of the NAB CR to be deleted.</dd>
   <dt><code>&lt;authorized_namespace&gt;</code></dt>
   <dd>Specifies the name of your authorized namespace.</dd>
   </dl>
2. Update the NAB CR YAML manifest file and add the `deleteBackup` flag as shown in the following example:
   ```yaml
   apiVersion: oadp.openshift.io/v1alpha1
   kind: NonAdminBackup
   metadata:
     name: <nab_cr>
   spec:
     backupSpec:
       includedNamespaces:
       - <authorized_namespace>
       deleteBackup: true
   ```

   where:

   <dl>
   <dt><code>&lt;nab_cr&gt;</code></dt>
   <dd>Specify the name of the NAB CR to be deleted.</dd>
   <dt><code>&lt;authorized_namespace&gt;</code></dt>
   <dd>Specify the name of your authorized namespace.</dd>
   <dt><code>deleteBackup: true</code></dt>
   <dd>Add the <code>deleteBackup</code> flag and set it to <code>true</code>.</dd>
   </dl>

**Verification**

- Verify that the NAB CR is deleted by running the following command:
  ```terminal
  $ oc get nab <nab_cr>
  ```

  `<nab_cr>` is the name of the NAB CR you deleted.

  You should see an output as shown in the following example:

  ```terminal
  Error from server (NotFound): nonadminbackups.oadp.openshift.io "test-nab" not found
  ```

## Creating a NonAdminRestore CR {#oadp-self-service-creating-nar_oadp-self-service-namespace-admin-use-cases}

Create a `NonAdminRestore` (NAR) custom resource (CR) to restore application resources from a backup to your authorized namespace. This provides an ability to recover your application data and configuration without requiring cluster administrator privileges.

**Prerequisites**

- You are logged in to the cluster as a namespace admin user.
- The cluster administrator has installed the OADP Operator.
- The cluster administrator has configured the `DataProtectionApplication` (DPA) CR to enable OADP Self-Service.
- The cluster administrator has created a namespace for you and has authorized you to operate from that namespace.
- You have a backup of your application by creating a `NonAdminBackup` (NAB) CR.

**Procedure**

1. To create a `NonAdminRestore` CR, create a YAML manifest file with the following configuration:
   ```yaml title="Example NonAdminRestore CR"
   apiVersion: oadp.openshift.io/v1alpha1
   kind: NonAdminRestore
   metadata:
     name: test-nar
   spec:
     restoreSpec:
       backupName: test-nab
   ```

   where:

   <dl>
   <dt><code>name</code></dt>
   <dd>Specifies a name for the NAR CR. For example, <code>test-nar</code>.</dd>
   <dt><code>backupName</code></dt>
   <dd>Specifies the name of the NAB CR you want to restore from. For example, <code>test-nab</code>.</dd>
   </dl>
2. To apply the NAR CR configuration, run the following command:
   ```terminal
   $ oc apply -f <nar_cr_filename>
   ```

   Replace `<nar_cr_filename>` with the file name containing the NAR CR configuration.

**Verification**

1. To verify that the NAR CR is successfully created, run the following command:
   ```terminal
   $ oc get nar test-nar -o yaml
   ```

   ```yaml title="Example output"
   apiVersion: oadp.openshift.io/v1alpha1
   kind: NonAdminRestore
   metadata:
     creationTimestamp: "2025-..:15Z"
     finalizers:
     - nonadminrestore.oadp.openshift.io/finalizer
     generation: 2
     name: test-nar
     namespace: test-nac-ns
     resourceVersion: "156517"
     uid: f9f5...63ef34
   spec:
     restoreSpec:
       backupName: test-nab
       hooks: {}
       itemOperationTimeout: 0s
   status:
     conditions:
     - lastTransitionTime: "2025..15Z"
       message: restore accepted
       reason: RestoreAccepted
       status: "True"
       type: Accepted
     - lastTransitionTime: "2025-03-06T11:22:15Z"
       message: Created Velero Restore object
       reason: RestoreScheduled
       status: "True"
       type: Queued
     dataMoverDataDownloads: {}
     fileSystemPodVolumeRestores:
       completed: 2
       total: 2
     phase: Created
     queueInfo:
       estimatedQueuePosition: 0
     veleroRestore:
       nacuuid: test-nac-test-nar-c...1ba
       name: test-nac-test-nar-c7...1ba
       namespace: openshift-adp
       status:
         completionTimestamp: "2025...22:44Z"
         hookStatus: {}
         phase: Completed
         progress:
           itemsRestored: 28
           totalItems: 28
         startTimestamp: "2025..15Z"
         warnings: 7
   ```

   where:

   <dl>
   <dt><code>message: restore accepted</code></dt>
   <dd>Specifies that the <code>NonAdminController</code> (NAC) CR has reconciled and validated the NAR CR.</dd>
   <dt><code>fileSystemPodVolumeRestores</code></dt>
   <dd>Specifies the number of volumes that are restored.</dd>
   <dt><code>phase: Created</code></dt>
   <dd>Specifies that the NAR CR is in the <code>Created</code> phase.</dd>
   <dt><code>estimatedQueuePosition</code></dt>
   <dd>Specifies the queue position of the restore object. There can be multiple restores in process, and each restore is assigned a queue position. When the restore is complete, the queue position is set to <code>0</code>.</dd>
   <dt><code>nacuuid</code></dt>
   <dd>Specifies that the NAC creates the <code>Velero</code> restore object and sets the <code>nacuuid</code> value.</dd>
   <dt><code>name</code></dt>
   <dd>Specifies the name of the associated <code>Velero</code> restore object.</dd>
   <dt><code>phase: Completed</code></dt>
   <dd>Specifies that the <code>Velero</code> restore object is in the <code>Completed</code> phase and the restore is successful.</dd>
   </dl>

## About NonAdminDownloadRequest CR {#oadp-self-service-about-nadr_oadp-self-service-namespace-admin-use-cases}

Review backup and restore logs by using the `NonAdminDownloadRequest` (NADR) custom resource (CR). This helps you troubleshoot backup and restore issues without cluster administrator assistance.

The NADR CR provides information that is equivalent to what a cluster administrator can access by using the `velero backup describe --details` command.

After the NADR CR request is validated, a secure download URL is generated to access the requested information.

You can download the following NADR resources:

**NADR resources**

|  |  |  |
| --- | --- | --- |
| **Resource type** | **Description** | **Equivalent to** |
| `BackupResourceList` | List of resources included in the backup | `velero backup describe --details` (resource listing) |
| `BackupContents` | Contents of files backed up | Part of backup details |
| `BackupLog` | Logs from the backup operation | `velero backup logs` |
| `BackupVolumeSnapshots` | Information about volume snapshots | `velero backup describe --details` (snapshots section) |
| `BackupItemOperations` | Information about item operations performed during backup | `velero backup describe --details` (operations section) |
| `RestoreLog` | Logs from the restore operation | `velero restore logs` |
| `RestoreResults` | Detailed results of the restore | `velero restore describe --details` |

## Reviewing NAB and NAR logs {#oadp-self-service-nab-nar-logs_oadp-self-service-namespace-admin-use-cases}

Create a `NonAdminDownloadRequest` (NADR) custom resource (CR) to access and review detailed logs for `NonAdminBackup` (NAB) and `NonAdminRestore` (NAR) operations. This helps you troubleshoot backup and restore issues independently.

:::note

You can review the NAB logs only if you are using a `NonAdminBackupStorageLocation` (NABSL) CR as a backup storage location for the backup.

:::

**Prerequisites**

- You are logged in to the cluster as a namespace admin user.
- The cluster administrator has installed the OADP Operator.
- The cluster administrator has configured the `DataProtectionApplication` (DPA) CR to enable OADP Self-Service.
- The cluster administrator has created a namespace for you and has authorized you to operate from that namespace.
- You have a backup of your application by creating a NAB CR.
- You have restored the application by creating a NAR CR.

**Procedure**

1. To review NAB CR logs, create a `NonAdminDownloadRequest` CR and specify the NAB CR name as shown in the following example:
   ```yaml title="Example NonAdminDownloadRequest CR"
   apiVersion: oadp.openshift.io/v1alpha1
   kind: NonAdminDownloadRequest
   metadata:
     name: test-nadr-backup
   spec:
     target:
       kind: BackupLog
       name: test-nab
   ```

   where:

   <dl>
   <dt><code>kind</code></dt>
   <dd>Specifies <code>BackupLog</code> as the value for the <code>kind</code> field of the NADR CR.</dd>
   <dt><code>name</code></dt>
   <dd>Specifies the name of the NAB CR.</dd>
   </dl>
2. Verify that the NADR CR is processed by running the following command:
   ```terminal
   $ oc get nadr test-nadr-backup -o yaml
   ```

   ```yaml title="Example output"
   apiVersion: oadp.openshift.io/v1alpha1
   kind: NonAdminDownloadRequest
   metadata:
     creationTimestamp: "2025-03-06T10:05:22Z"
     generation: 1
     name: test-nadr-backup
     namespace: test-nac-ns
     resourceVersion: "134866"
     uid: 520...8d9
   spec:
     target:
       kind: BackupLog
       name: test-nab
   status:
     conditions:
     - lastTransitionTime: "202...5:22Z"
       message: ""
       reason: Success
       status: "True"
       type: Processed
     phase: Created
     velero:
       status:
         downloadURL: https://...
         expiration: "202...22Z"
         phase: Processed
   ```

   where:

   <dl>
   <dt><code>downloadURL</code></dt>
   <dd>The <code>status.downloadURL</code> field contains the download URL of the NAB logs. You can use the <code>downloadURL</code> to download and review the NAB logs.</dd>
   <dt><code>phase</code></dt>
   <dd>The <code>status.phase</code> is <code>Processed</code>.</dd>
   </dl>
3. Download and analyze the backup information by using the `status.downloadURL` URL.
4. To review NAR CR logs, create a `NonAdminDownloadRequest` CR and specify the NAR CR name as shown in the following example:
   ```yaml title="Example NonAdminDownloadRequest CR"
   apiVersion: oadp.openshift.io/v1alpha1
   kind: NonAdminDownloadRequest
   metadata:
     name: test-nadr-restore
   spec:
     target:
       kind: RestoreLog
       name: test-nar
   ```

   where:

   <dl>
   <dt><code>kind</code></dt>
   <dd>Specifies <code>RestoreLog</code> as the value for the <code>kind</code> field of the NADR CR.</dd>
   <dt><code>name</code></dt>
   <dd>Specifies the name of the NAR CR.</dd>
   </dl>
5. Verify that the NADR CR is processed by running the following command:
   ```terminal
   $ oc get nadr test-nadr-restore -o yaml
   ```

   ```yaml title="Example output"
   apiVersion: oadp.openshift.io/v1alpha1
   kind: NonAdminDownloadRequest
   metadata:
     creationTimestamp: "2025-03-06T11:26:01Z"
     generation: 1
     name: test-nadr-restore
     namespace: test-nac-ns
     resourceVersion: "157842"
     uid: f3e...7862f
   spec:
     target:
       kind: RestoreLog
       name: test-nar
   status:
     conditions:
     - lastTransitionTime: "202..:01Z"
       message: ""
       reason: Success
       status: "True"
       type: Processed
     phase: Created
     velero:
       status:
         downloadURL: https://...
         expiration: "202..:01Z"
         phase: Processed
   ```

   where:

   <dl>
   <dt><code>downloadURL</code></dt>
   <dd>The <code>status.downloadURL</code> field contains the download URL of the NAR logs. You can use the <code>downloadURL</code> to download and review the NAR logs.</dd>
   <dt><code>phase</code></dt>
   <dd>The <code>status.phase</code> is <code>Processed</code>.</dd>
   </dl>
6. Download and analyze the restore information by using the `status.downloadURL` URL.
