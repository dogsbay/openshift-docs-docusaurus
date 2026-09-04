---
title: "Backing up applications with File System Backup: Kopia or Restic"
sidebar_position: 6
---

# Backing up applications with File System Backup: Kopia or Restic {#oadp-backing-up-applications-restic-doc}

<a id="oadp-backing-up-applications-restic-doc"></a>

Use OADP File System Backup (FSB) with Kopia or Restic to back up and restore Kubernetes volumes attached to pods when snapshots are not available. This helps you to protect application data on NFS or other non-snapshot storage.

If your cloud provider does not support snapshots or if your applications are on NFS data volumes, you can create backups by using FSB.

FSB integration with OADP provides a solution for backing up and restoring almost any type of Kubernetes volumes. This integration is an additional capability of OADP and is not a replacement for existing functionality.

You back up Kubernetes resources, internal images, and persistent volumes with Kopia or Restic by editing the `Backup` custom resource (CR).

You do not need to specify a snapshot location in the `DataProtectionApplication` CR.

:::note

In OADP version 1.3 and later, you can use either Kopia or Restic for backing up applications.

For the Built-in DataMover, you must use Kopia.

In OADP version 1.2 and earlier, you can only use Restic for backing up applications.

:::

:::warning

FSB does not support backing up `hostPath` volumes. For more information, see *FSB limitations*.

:::

:::warning

The `.../.snapshot` directory is a snapshot copy directory, which is used by several NFS servers. This directory has read-only access by default, so Velero cannot restore to this directory.

Do not give Velero write access to the `.snapshot` directory, and disable client access to this directory.

:::

## Backing up applications with File System Backup {#oadp-backingup-file-system-backup_backing-up-applications}

Create a `Backup` custom resource (CR) to back up applications by using File System Backup (FSB) with Kopia as the uploader. This helps you to protect Kubernetes volumes attached to pods when snapshots are not available or when using NFS data volumes.

**Prerequisites**

- You must install the OpenShift API for Data Protection (OADP) Operator.
- You must not disable the default `nodeAgent` installation by setting `spec.configuration.nodeAgent.enable` to `false` in the `DataProtectionApplication` CR.
- You must select Kopia or Restic as the uploader by setting `spec.configuration.nodeAgent.uploaderType` to `kopia` or `restic` in the `DataProtectionApplication` CR.
- The `DataProtectionApplication` CR must be in a `Ready` state.

**Procedure**

- Create the `Backup` CR, as in the following example:
  ```yaml
  apiVersion: velero.io/v1
  kind: Backup
  metadata:
    name: <backup>
    labels:
      velero.io/storage-location: default
    namespace: openshift-adp
  spec:
    defaultVolumesToFsBackup: true
  ...
  ```

  where:

  <dl>
  <dt><code>defaultVolumesToFsBackup: true</code></dt>
  <dd>Specifies the FSB setting within the <code>spec</code> block for OADP version 1.2 and later. In OADP version 1.1, add <code>defaultVolumesToRestic: true</code> instead.</dd>
  </dl>

**Additional resources**

- [FSB limitations](https://velero.io/docs/v1.12/file-system-backup/#limitations)
