---
title: About the OADP Data Mover
sidebar_position: 1
---

# About the OADP Data Mover {#about-oadp-data-mover}

<a id="about-oadp-data-mover"></a>

Use the OpenShift API for Data Protection (OADP) built-in Data Mover to move Container Storage Interface (CSI) volume snapshots to remote object storage and restore stateful applications after cluster failures. This provides disaster recovery capabilities for both containerized and virtual machine workloads.

The Data Mover uses Kopia as the uploader mechanism to read the snapshot data and write to the unified repository.

OADP supports CSI snapshots on the following:

- Red Hat OpenShift Data Foundation
- Any other cloud storage provider with the Container Storage Interface (CSI) driver that supports the Kubernetes Volume Snapshot API

## Data Mover support {#oadp-data-mover-support_about-oadp-data-mover}

Review Data Mover support and compatibility across OADP versions to understand which backups can be restored. This helps you plan version upgrades and backup strategies.

The OADP built-in Data Mover, which was introduced in OADP 1.3 as a Technology Preview, is now fully supported for both containerized and virtual machine workloads.

Supported
: The Data Mover backups taken with OADP 1.3 can be restored using OADP 1.3 and later.

Not supported
: Backups taken with OADP 1.1 or OADP 1.2 using the Data Mover feature cannot be restored using OADP 1.3 and later.

OADP 1.1 and OADP 1.2 are no longer supported. The DataMover feature in OADP 1.1 or OADP 1.2 was a Technology Preview and was never supported. DataMover backups taken with OADP 1.1 or OADP 1.2 cannot be restored on later versions of OADP.

## Enabling the built-in Data Mover {#enabling-oadp-data-mover_about-oadp-data-mover}

Enable the built-in Data Mover by configuring the CSI plugin and node agent in the `DataProtectionApplication` custom resource (CR). This provides volume-level backup and restore operations by using the Kopia uploader.

**Procedure**

- Include the CSI plugin and enable the node agent in the `DataProtectionApplication` custom resource (CR) as shown in the following example:
  ```yaml
  apiVersion: oadp.openshift.io/v1alpha1
  kind: DataProtectionApplication
  metadata:
    name: dpa-sample
  spec:
    configuration:
      nodeAgent:
        enable: true
        uploaderType: kopia
      velero:
        defaultPlugins:
        - openshift
        - aws
        - csi
        defaultSnapshotMoveData: true
        defaultVolumesToFSBackup:
        featureFlags:
        - EnableCSI
  # ...
  ```

  where:

  <dl>
  <dt><code>enable</code></dt>
  <dd>Specifies the flag to enable the node agent.</dd>
  <dt><code>uploaderType</code></dt>
  <dd>Specifies the type of uploader. The possible values are <code>restic</code> or <code>kopia</code>. The built-in Data Mover uses Kopia as the default uploader mechanism regardless of the value of the <code>uploaderType</code> field.</dd>
  <dt><code>csi</code></dt>
  <dd>Specifies the CSI plugin included in the list of default plugins.</dd>
  <dt><code>defaultVolumesToFSBackup</code></dt>
  <dd>Specifies the default behavior for volumes. In OADP 1.3.1 and later, set to <code>true</code> if you use Data Mover only for volumes that opt out of <code>fs-backup</code>. Set to <code>false</code> if you use Data Mover by default for volumes.</dd>
  </dl>

## Built-in Data Mover controller and custom resource definitions (CRDs) {#built-in-data-mover-crds_about-oadp-data-mover}

Review the custom resource definitions (CRDs) that the built-in Data Mover uses to manage volume snapshot backup and restore operations. This helps you understand how Data Mover handles data upload, download, and repository management.

The built-in Data Mover feature introduces three new API objects defined as CRDs for managing backup and restore:

- `DataDownload`: Represents a data download of a volume snapshot. The CSI plugin creates one `DataDownload` object per volume to be restored. The `DataDownload` CR includes information about the target volume, the specified Data Mover, the progress of the current data download, the specified backup repository, and the result of the current data download after the process is complete.
- `DataUpload`: Represents a data upload of a volume snapshot. The CSI plugin creates one `DataUpload` object per CSI snapshot. The `DataUpload` CR includes information about the specified snapshot, the specified Data Mover, the specified backup repository, the progress of the current data upload, and the result of the current data upload after the process is complete.
- `BackupRepository`: Represents and manages the lifecycle of the backup repositories. OADP creates a backup repository per namespace when the first CSI snapshot backup or restore for a namespace is requested.

## About incremental backup support {#oadp-about-incremental-backup-support_about-oadp-data-mover}

OADP supports incremental backups of `block` and `Filesystem` persistent volumes for both containerized, and OpenShift Virtualization workloads. The following table summarizes the support for File System Backup (FSB), Container Storage Interface (CSI), and CSI Data Mover:

**OADP backup support matrix for containerized workloads**

| Volume mode | FSB - Restic | FSB - Kopia | CSI | CSI Data Mover |
| --- | --- | --- | --- | --- |
| Filesystem | Backup supported, Incremental backup supported | Backup supported, Incremental backup supported | Backup supported | Backup supported, Incremental backup supported |
| Block | Not supported | Not supported | Backup supported | Backup supported, Incremental backup supported |

**OADP backup support matrix for OpenShift Virtualization workloads**

| Volume mode | FSB - Restic | FSB - Kopia | CSI | CSI Data Mover |
| --- | --- | --- | --- | --- |
| Filesystem | Not supported | Not supported | Backup supported | Backup supported, Incremental backup supported |
| Block | Not supported | Not supported | Backup supported | Backup supported, Incremental backup supported |

:::note

The CSI Data Mover backups use Kopia regardless of `uploaderType`.

:::

**Additional resources**

- [About Kopia](/docs/backup_and_restore/application_backup_and_restore/backing_up_and_restoring/oadp-about-kopia#oadp-about-kopia)
