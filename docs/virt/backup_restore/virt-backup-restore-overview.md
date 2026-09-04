---
title: Backing up and restoring virtual machines
sidebar_position: 2
---

# Backing up and restoring virtual machines {#virt-backup-restore-overview}

<a id="virt-backup-restore-overview"></a>

Back up and restore virtual machines by using the OpenShift API for Data Protection.

:::warning

Red Hat supports using OpenShift Virtualization 4.14 or later with OADP 1.3.x or later.

OADP versions earlier than 1.3.0 are not supported for back up and restore of OpenShift Virtualization.

:::

## Installing and configuring OADP with OpenShift Virtualization {#install-and-configure-oadp-kubevirt_virt-backup-restore-overview}

As a cluster administrator, you can install the OpenShift API for Data Protection (OADP) with OpenShift Virtualization by installing the OADP Operator and configuring a backup location. You can then install the Data Protection Application.

To install the OADP Operator in a restricted network environment, you must first disable the default software catalog sources and mirror the Operator catalog.

:::note

OpenShift API for Data Protection with OpenShift Virtualization supports the following backup and restore storage options:

- Container Storage Interface (CSI) backups
- Container Storage Interface (CSI) backups with DataMover

The following storage options are excluded:

- File system backup and restore
- Volume snapshot backup and restore

The latest version of the OADP Operator installs Velero 1.16.

:::

:::warning

Red Hat support is limited to only the following options:

- CSI backups
- CSI backups with DataMover.

:::

**Prerequisites**

- Access to the cluster as a user with the `cluster-admin` role.

**Procedure**

1. Install the OADP Operator according to the instructions for your storage provider.
2. Install the Data Protection Application (DPA) with the `kubevirt` and `openshift` OADP plug-ins.
3. Back up virtual machines by creating a `Backup` custom resource (CR).
   You restore the `Backup` CR by creating a `Restore` CR.

## Installing the Data Protection Application {#oadp-installing-dpa_virt-backup-restore-overview}

You install the Data Protection Application (DPA) by creating an instance of the `DataProtectionApplication` API.

**Prerequisites**

- You must install the OADP Operator.
- You must configure object storage as a backup location.
- If you use snapshots to back up PVs, your cloud provider must support either a native snapshot API or Container Storage Interface (CSI) snapshots.
- If the backup and snapshot locations use the same credentials, you must create a `Secret` with the default name, `cloud-credentials`.
  :::note

  If you do not want to specify backup or snapshot locations during the installation, you can create a default `Secret` with an empty `credentials-velero` file. If there is no default `Secret`, the installation will fail.

  :::

**Procedure**

1. Click **Ecosystem** → **Installed Operators** and select the OADP Operator.
2. Under **Provided APIs**, click **Create instance** in the **DataProtectionApplication** box.
3. Click **YAML View** and update the parameters of the `DataProtectionApplication` manifest:
   ```yaml
   apiVersion: oadp.openshift.io/v1alpha1
   kind: DataProtectionApplication
   metadata:
     name: <dpa_sample>
     namespace: openshift-adp
   spec:
     configuration:
       velero:
         defaultPlugins:
           - kubevirt
           - gcp
           - csi
           - openshift
         resourceTimeout: 10m
       nodeAgent:
         enable: true
         uploaderType: kopia
         podConfig:
           nodeSelector: <node_selector>
     backupLocations:
       - velero:
           provider: gcp
           default: true
           credential:
             key: cloud
             name: <default_secret>
           objectStorage:
             bucket: <bucket_name>
             prefix: <prefix>
   ```

   where:

   <dl>
   <dt><code>namespace</code></dt>
   <dd>Specifies the default namespace for OADP which is <code>openshift-adp</code>. The namespace is a variable and is configurable.</dd>
   <dt><code>kubevirt</code></dt>
   <dd>Specifies that the <code>kubevirt</code> plugin is mandatory for OpenShift Virtualization.</dd>
   <dt><code>gcp</code></dt>
   <dd>Specifies the plugin for the backup provider, for example, <code>gcp</code>, if it exists.</dd>
   <dt><code>csi</code></dt>
   <dd>Specifies that the <code>csi</code> plugin is mandatory for backing up PVs with CSI snapshots. The <code>csi</code> plugin uses the <a href="https://velero.io/docs/main/csi/">Velero CSI beta snapshot APIs</a>. You do not need to configure a snapshot location.</dd>
   <dt><code>openshift</code></dt>
   <dd>Specifies that the <code>openshift</code> plugin is mandatory.</dd>
   <dt><code>resourceTimeout</code></dt>
   <dd>Specifies how many minutes to wait for several Velero resources such as Velero CRD availability, volumeSnapshot deletion, and backup repository availability, before timeout occurs. The default is 10m.</dd>
   <dt><code>nodeAgent</code></dt>
   <dd>Specifies the administrative agent that routes the administrative requests to servers.</dd>
   <dt><code>enable</code></dt>
   <dd>Set this value to <code>true</code> if you want to enable <code>nodeAgent</code> and perform File System Backup.</dd>
   <dt><code>uploaderType</code></dt>
   <dd>Specifies the uploader type. Enter <code>kopia</code> as your uploader to use the Built-in DataMover. The <code>nodeAgent</code> deploys a daemon set, which means that the <code>nodeAgent</code> pods run on each working node. You can configure File System Backup by adding <code>spec.defaultVolumesToFsBackup: true</code> to the <code>Backup</code> CR.</dd>
   <dt><code>nodeSelector</code></dt>
   <dd>Specifies the nodes on which Kopia are available. By default, Kopia runs on all nodes.</dd>
   <dt><code>provider</code></dt>
   <dd>Specifies the backup provider.</dd>
   <dt><code>name</code></dt>
   <dd>Specifies the correct default name for the <code>Secret</code>, for example, <code>cloud-credentials-gcp</code>, if you use a default plugin for the backup provider. If specifying a custom name, then the custom name is used for the backup location. If you do not specify a <code>Secret</code> name, the default name is used.</dd>
   <dt><code>bucket</code></dt>
   <dd>Specifies a bucket as the backup storage location. If the bucket is not a dedicated bucket for Velero backups, you must specify a prefix.</dd>
   <dt><code>prefix</code></dt>
   <dd>Specifies a prefix for Velero backups, for example, <code>velero</code>, if the bucket is used for multiple purposes.</dd>
   </dl>
4. Click **Create**.

**Verification**

1. Verify the installation by viewing the OpenShift API for Data Protection (OADP) resources by running the following command:
   ```terminal
   $ oc get all -n openshift-adp
   ```

   ```
   NAME                                                     READY   STATUS    RESTARTS   AGE
   pod/oadp-operator-controller-manager-67d9494d47-6l8z8    2/2     Running   0          2m8s
   pod/node-agent-9cq4q                                     1/1     Running   0          94s
   pod/node-agent-m4lts                                     1/1     Running   0          94s
   pod/node-agent-pv4kr                                     1/1     Running   0          95s
   pod/velero-588db7f655-n842v                              1/1     Running   0          95s

   NAME                                                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
   service/oadp-operator-controller-manager-metrics-service   ClusterIP   172.30.70.140    <none>        8443/TCP   2m8s
   service/openshift-adp-velero-metrics-svc                   ClusterIP   172.30.10.0      <none>        8085/TCP   8h

   NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
   daemonset.apps/node-agent    3         3         3       3            3           <none>          96s

   NAME                                                READY   UP-TO-DATE   AVAILABLE   AGE
   deployment.apps/oadp-operator-controller-manager    1/1     1            1           2m9s
   deployment.apps/velero                              1/1     1            1           96s

   NAME                                                           DESIRED   CURRENT   READY   AGE
   replicaset.apps/oadp-operator-controller-manager-67d9494d47    1         1         1       2m9s
   replicaset.apps/velero-588db7f655                              1         1         1       96s
   ```
2. Verify that the `DataProtectionApplication` (DPA) is reconciled by running the following command:
   ```terminal
   $ oc get dpa dpa-sample -n openshift-adp -o jsonpath='{.status}'
   ```

   ```yaml
   {"conditions":[{"lastTransitionTime":"2023-10-27T01:23:57Z","message":"Reconcile complete","reason":"Complete","status":"True","type":"Reconciled"}]}
   ```
3. Verify the `type` is set to `Reconciled`.
4. Verify the backup storage location and confirm that the `PHASE` is `Available` by running the following command:
   ```terminal
   $ oc get backupstoragelocations.velero.io -n openshift-adp
   ```

   ```yaml
   NAME           PHASE       LAST VALIDATED   AGE     DEFAULT
   dpa-sample-1   Available   1s               3d16h   true
   ```

**Additional resources**

- [Application backup and restore operations](/docs/backup_and_restore#application-backup-restore-operations-overview_backup-restore-overview)
- [Backing up applications with File System Backup: Kopia or Restic](/docs/backup_and_restore/application_backup_and_restore/backing_up_and_restoring/oadp-backing-up-applications-restic-doc#oadp-backing-up-applications-restic-doc)
- [OADP plug-ins](/docs/backup_and_restore/application_backup_and_restore/oadp-features-plugins#oadp-plugins_oadp-features-plugins)
- [Backing up applications](/docs/backup_and_restore/application_backup_and_restore/backing_up_and_restoring/backing-up-applications#backing-up-applications)
- [Restoring applications](/docs/backup_and_restore/application_backup_and_restore/backing_up_and_restoring/restoring-applications#restoring-applications)
- [Using Operator Lifecycle Manager in disconnected environments](/docs/disconnected/using-olm#olm-restricted-networks)
