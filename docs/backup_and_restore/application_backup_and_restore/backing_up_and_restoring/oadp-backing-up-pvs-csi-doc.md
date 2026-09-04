---
title: Backing up persistent volumes with CSI snapshots
sidebar_position: 5
---

# Backing up persistent volumes with CSI snapshots {#oadp-backing-up-pvs-csi-doc}

<a id="oadp-backing-up-pvs-csi-doc"></a>

Back up persistent volumes with Container Storage Interface (CSI) snapshots by editing the `VolumeSnapshotClass` custom resource (CR) before you create the `Backup` CR. This helps you to leverage cloud-native snapshot capabilities for faster and more efficient backups.

For more information, see *CSI volume snapshots* and *Creating a Backup CR*.

## Backing up persistent volumes with CSI snapshots {#backingup-persistent-volumes_backing-up-applications}

Configure the `VolumeSnapshotClass` custom resource (CR) to back up persistent volumes with Container Storage Interface (CSI) snapshots. This helps you to prepare your cloud storage for CSI-based backup operations.

**Prerequisites**

- The cloud provider must support CSI snapshots.
- You must enable CSI in the `DataProtectionApplication` CR.

**Procedure**

- Add the `metadata.labels.velero.io/csi-volumesnapshot-class: "true"` key-value pair to the `VolumeSnapshotClass` CR:
  ```yaml title="Example configuration file"
  apiVersion: snapshot.storage.k8s.io/v1
  kind: VolumeSnapshotClass
  metadata:
    name: <volume_snapshot_class_name>
    labels:
      velero.io/csi-volumesnapshot-class: "true"
    annotations:
      snapshot.storage.kubernetes.io/is-default-class: true
  driver: <csi_driver>
  deletionPolicy: <deletion_policy_type>
  ```

  where:

  <dl>
  <dt><code>velero.io/csi-volumesnapshot-class: "true"</code></dt>
  <dd>Must be set to <code>true</code>.</dd>
  <dt><code>snapshot.storage.kubernetes.io/is-default-class: true</code></dt>
  <dd>If you are restoring this volume in another cluster with the same driver, make sure that you set the <code>snapshot.storage.kubernetes.io/is-default-class</code> parameter to <code>false</code> instead of setting it to <code>true</code>. Otherwise, the restore will partially fail.</dd>
  <dt><code>&lt;deletion_policy_type&gt;</code></dt>
  <dd>Specifies the deletion policy type. OADP supports the <code>Retain</code> and <code>Delete</code> deletion policy types for CSI and Data Mover backup and restore.</dd>
  </dl>

**Next steps**

- You can now create a `Backup` CR.

**Additional resources**

- [CSI volume snapshots](/docs/storage/container_storage_interface/persistent-storage-csi-snapshots#persistent-storage-csi-snapshots-overview_persistent-storage-csi-snapshots)
- [Creating a Backup CR](/docs/backup_and_restore/application_backup_and_restore/backing_up_and_restoring/oadp-creating-backup-cr#oadp-creating-backup-cr-doc)
