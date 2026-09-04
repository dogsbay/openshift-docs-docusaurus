---
title: Creating a Backup CR
sidebar_position: 2
---

# Creating a Backup CR {#oadp-creating-backup-cr-doc}

<a id="oadp-creating-backup-cr-doc"></a>

Back up Kubernetes resources, internal images, and persistent volumes (PVs) by creating a `Backup` custom resource (CR). This helps you to protect your application data and configuration for disaster recovery.

**Prerequisites**

- You must install the OpenShift API for Data Protection (OADP) Operator.
- The `DataProtectionApplication` CR must be in a `Ready` state.
- Backup location prerequisites:
  - You must have S3 object storage configured for Velero.
  - You must have a backup location configured in the `DataProtectionApplication` CR.
- Snapshot location prerequisites:
  - Your cloud provider must have a native snapshot API or support Container Storage Interface (CSI) snapshots.
  - For CSI snapshots, you must create a `VolumeSnapshotClass` CR to register the CSI driver.
  - You must have a volume location configured in the `DataProtectionApplication` CR.

**Procedure**

1. Retrieve the `backupStorageLocations` CRs by entering the following command:
   ```terminal
   $ oc get backupstoragelocations.velero.io -n openshift-adp
   ```

   ```terminal
   NAMESPACE       NAME              PHASE       LAST VALIDATED   AGE   DEFAULT
   openshift-adp   velero-sample-1   Available   11s              31m
   ```
2. Create a `Backup` CR, as in the following example:
   ```yaml
   apiVersion: velero.io/v1
   kind: Backup
   metadata:
     name: <backup>
     labels:
       velero.io/storage-location: default
     namespace: openshift-adp
   spec:
     hooks: {}
     includedNamespaces:
     - <namespace>
     includedResources: []
     excludedResources: []
     storageLocation: <velero-sample-1>
     ttl: 720h0m0s
     labelSelector:
       matchLabels:
         app: <label_1>
         app: <label_2>
         app: <label_3>
     orLabelSelectors:
     - matchLabels:
         app: <label_1>
         app: <label_2>
         app: <label_3>
   ```

   where:

   <dl>
   <dt><code>&lt;namespace&gt;</code></dt>
   <dd>Specifies an array of namespaces to back up.</dd>
   <dt><code>includedResources</code></dt>
   <dd>Optional: Specifies an array of resources to include in the backup. Resources might be shortcuts (for example, <code>po</code> for <code>pods</code>) or fully-qualified. If unspecified, all resources are included.</dd>
   <dt><code>excludedResources</code></dt>
   <dd>Optional: Specifies an array of resources to exclude from the backup. Resources might be shortcuts (for example, <code>po</code> for <code>pods</code>) or fully-qualified.</dd>
   <dt><code>&lt;velero-sample-1&gt;</code></dt>
   <dd>Specifies the name of the <code>backupStorageLocations</code> CR.</dd>
   <dt><code>labelSelector</code></dt>
   <dd>Specifies a map of {key,value} pairs of backup resources that have <strong>all</strong> the specified labels.</dd>
   <dt><code>orLabelSelectors</code></dt>
   <dd>Specifies a map of {key,value} pairs of backup resources that have <strong>one or more</strong> of the specified labels.</dd>
   </dl>
3. Verify that the status of the `Backup` CR is `Completed`:
   ```terminal
   $ oc get backups.velero.io -n openshift-adp <backup> -o jsonpath='{.status.phase}'
   ```
