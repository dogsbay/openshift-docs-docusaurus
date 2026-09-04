---
title: Configuring the OpenShift API for Data Protection with OpenShift Virtualization
sidebar_position: 1
---

# Configuring the OpenShift API for Data Protection with OpenShift Virtualization {#installing-oadp-kubevirt}

<a id="installing-oadp-kubevirt"></a>

You can install the OpenShift API for Data Protection (OADP) with OpenShift Virtualization by installing the OADP Operator and configuring a backup location. Then, you can install the Data Protection Application.

Back up and restore virtual machines by using the OpenShift API for Data Protection.

OpenShift API for Data Protection with OpenShift Virtualization supports the following backup and restore storage options:

- Container Storage Interface (CSI) backups
- Container Storage Interface (CSI) backups with DataMover

The following storage options are excluded:

- File system backup and restore
- Volume snapshot backups and restores

To install the OADP Operator in a restricted network environment, you must first disable the default software catalog sources and mirror the Operator catalog.

:::warning

Red Hat only supports the combination of OADP versions 1.3.0 and later, and OpenShift Virtualization versions 4.14 and later.

OADP versions before 1.3.0 are not supported for back up and restore of OpenShift Virtualization.

:::

## Installing and configuring OADP with OpenShift Virtualization {#install-and-configure-oadp-kubevirt_installing-oadp-kubevirt}

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

## Installing the Data Protection Application {#oadp-installing-dpa_installing-oadp-kubevirt}

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

:::warning

If you run a backup of a Microsoft Windows virtual machine (VM) immediately after the VM reboots, the backup might fail with a `PartiallyFailed` error. This is because, immediately after a VM boots, the Microsoft Windows Volume Shadow Copy Service (VSS) and Guest Agent (GA) service are not ready. The VSS and GA service being unready causes the backup to fail. In such a case, retry the backup a few minutes after the VM boots.

:::

## Backing up a single VM {#oadp-backup-single-vm_installing-oadp-kubevirt}

If you have a namespace with multiple virtual machines (VMs), and want to back up only one of them, you can use the label selector to filter the VM that needs to be included in the backup. You can filter the VM by using the `app: vmname` label.

**Prerequisites**

- You have installed the OADP Operator.
- You have multiple VMs running in a namespace.
- You have added the `kubevirt` plugin in the `DataProtectionApplication` (DPA) custom resource (CR).
- You have configured the `BackupStorageLocation` CR in the `DataProtectionApplication` CR and `BackupStorageLocation` is available.

**Procedure**

1. Configure the `Backup` CR as shown in the following example:
   ```yaml title="Example Backup CR"
   apiVersion: velero.io/v1
   kind: Backup
   metadata:
     name: vmbackupsingle
     namespace: openshift-adp
   spec:
     snapshotMoveData: true
     includedNamespaces:
     - <vm_namespace>
     labelSelector:
       matchLabels:
         app: <vm_app_name>
     storageLocation: <backup_storage_location_name>
   ```

   where:

   <dl>
   <dt><code>vm_namespace</code></dt>
   <dd>Specifies the name of the namespace where you have created the VMs.</dd>
   <dt><code>vm_app_name</code></dt>
   <dd>Specifies the VM name that needs to be backed up.</dd>
   <dt><code>backup_storage_location_name</code></dt>
   <dd>Specifies the name of the <code>BackupStorageLocation</code> CR.</dd>
   </dl>
2. To create a `Backup` CR, run the following command:
   ```terminal
   $ oc apply -f <backup_cr_file_name>
   ```

   where:

   <dl>
   <dt><code>backup_cr_file_name</code></dt>
   <dd>Specifies the name of the <code>Backup</code> CR file.</dd>
   </dl>

## Restoring a single VM {#oadp-restore-single-vm_installing-oadp-kubevirt}

After you have backed up a single virtual machine (VM) by using the label selector in the `Backup` custom resource (CR), you can create a `Restore` CR and point it to the backup. This restore operation restores a single VM.

**Prerequisites**

- You have installed the OADP Operator.
- You have backed up a single VM by using the label selector.

**Procedure**

1. Configure the `Restore` CR as shown in the following example:
   ```yaml title="Example Restore CR"
   apiVersion: velero.io/v1
   kind: Restore
   metadata:
     name: vmrestoresingle
     namespace: openshift-adp
   spec:
     backupName: vmbackupsingle
     restorePVs: true
   ```

   where:

   <dl>
   <dt><code>vmbackupsingle</code></dt>
   <dd>Specifies the name of the backup of a single VM.</dd>
   </dl>
2. To restore the single VM, run the following command:
   ```terminal
   $ oc apply -f <restore_cr_file_name>
   ```

   where:

   <dl>
   <dt><code>restore_cr_file_name</code></dt>
   <dd>Specifies the name of the <code>Restore</code> CR file.</dd>
   </dl>

   :::note

   When you restore a backup of VMs, you might notice that the Ceph storage capacity allocated for the restore is higher than expected. This behavior is observed only during the `kubevirt` restore and if the volume type of the VM is `block`.

   Use the `rbd sparsify` tool to reclaim space on target volumes. For more details, see [Reclaiming space on target volumes](https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation/latest/html-single/managing_and_allocating_storage_resources/index#reclaiming-space-on-target-volumes_rhodf).

   :::

## Restoring a single VM from a backup of multiple VMs {#oadp-restore-single-vm-from-multiple-vm-backup_installing-oadp-kubevirt}

If you have a backup containing multiple virtual machines (VMs), and you want to restore only one VM, you can use the `LabelSelectors` section in the `Restore` CR to select the VM to restore. To ensure that the persistent volume claim (PVC) attached to the VM is correctly restored, and the restored VM is not stuck in a `Provisioning` status, use both the `app: <vm_name>` and the `kubevirt.io/created-by` labels. To match the `kubevirt.io/created-by` label, use the UID of `DataVolume` of the VM.

**Prerequisites**

- You have installed the OADP Operator.
- You have labeled the VMs that need to be backed up.
- You have a backup of multiple VMs.

**Procedure**

1. Before you take a backup of many VMs, ensure that the VMs are labeled by running the following command:
   ```terminal
   $ oc label vm <vm_name> app=<vm_name> -n openshift-adp
   ```
2. Configure the label selectors in the `Restore` CR as shown in the following example:
   ```yaml title="Example Restore CR"
   apiVersion: velero.io/v1
   kind: Restore
   metadata:
     name: singlevmrestore
     namespace: openshift-adp
   spec:
     backupName: multiplevmbackup
     restorePVs: true
     LabelSelectors:
       - matchLabels:
           kubevirt.io/created-by: <datavolume_uid>
       - matchLabels:
           app: <vm_name>
   ```

   where:

   <dl>
   <dt><code>datavolume_uid</code></dt>
   <dd>Specifies the UID of <code>DataVolume</code> of the VM that you want to restore. For example, <code>b6...53a-ddd7-4d9d-9407-a0c...e5</code>.</dd>
   <dt><code>vm_name</code></dt>
   <dd>Specifies the name of the VM that you want to restore. For example, <code>test-vm</code>.</dd>
   </dl>
3. To restore a VM, run the following command:
   ```terminal
   $ oc apply -f <restore_cr_file_name>
   ```

   where:

   <dl>
   <dt><code>restore_cr_file_name</code></dt>
   <dd>Specifies the name of the <code>Restore</code> CR file.</dd>
   </dl>

## Configuring the DPA with client burst and QPS settings {#oadp-configuring-client-burst-qps_installing-oadp-kubevirt}

The burst setting determines how many requests can be sent to the `velero` server before the limit is applied. After the burst limit is reached, the queries per second (QPS) setting determines how many additional requests can be sent per second.

You can set the burst and QPS values of the `velero` server by configuring the Data Protection Application (DPA) with the burst and QPS values. You can use the `dpa.configuration.velero.client-burst` and `dpa.configuration.velero.client-qps` fields of the DPA to set the burst and QPS values.

**Prerequisites**

- You have installed the OADP Operator.

**Procedure**

- Configure the `client-burst` and the `client-qps` fields in the DPA as shown in the following example:
  ```yaml title="Example Data Protection Application"
  apiVersion: oadp.openshift.io/v1alpha1
  kind: DataProtectionApplication
  metadata:
    name: test-dpa
    namespace: openshift-adp
  spec:
    backupLocations:
      - name: default
        velero:
          config:
            insecureSkipTLSVerify: "true"
            profile: "default"
            region: <bucket_region>
            s3ForcePathStyle: "true"
            s3Url: <bucket_url>
          credential:
            key: cloud
            name: cloud-credentials
          default: true
          objectStorage:
            bucket: <bucket_name>
            prefix: velero
          provider: aws
    configuration:
      nodeAgent:
        enable: true
        uploaderType: restic
      velero:
        client-burst: 500
        client-qps: 300
        defaultPlugins:
          - openshift
          - aws
          - kubevirt
  ```

  where:

  <dl>
  <dt><code>client-burst</code></dt>
  <dd>Specifies the <code>client-burst</code> value. In this example, the <code>client-burst</code> field is set to 500.</dd>
  <dt><code>client-qps</code></dt>
  <dd>Specifies the <code>client-qps</code> value. In this example, the <code>client-qps</code> field is set to 300.</dd>
  </dl>

## Configuring the node agent as a non-root and non-privileged user {#oadp-configuring-node-agent-non-root_installing-oadp-kubevirt}

To enhance the node agent security, you can configure the OADP Operator node agent daemonset to run as a non-root and non-privileged user by using the `spec.configuration.velero.disableFsBackup` setting in the `DataProtectionApplication` (DPA) custom resource (CR).

By setting the `spec.configuration.velero.disableFsBackup` setting to `true`, the node agent security context sets the root file system to read-only and sets the `privileged` flag to `false`.

:::note

Setting `spec.configuration.velero.disableFsBackup` to `true` enhances the node agent security by removing the need for privileged containers and enforcing a read-only root file system.

However, it also disables File System Backup (FSB) with Kopia. If your workloads rely on FSB for backing up volumes that do not support native snapshots, then you should evaluate whether the `disableFsBackup` configuration fits your use case.

:::

**Prerequisites**

- You have installed the OADP Operator.

**Procedure**

- Configure the `disableFsBackup` field in the DPA as shown in the following example:
  ```yaml
  apiVersion: oadp.openshift.io/v1alpha1
  kind: DataProtectionApplication
  metadata:
    name: ts-dpa
    namespace: openshift-adp
  spec:
    backupLocations:
    - velero:
        credential:
          key: cloud
          name: cloud-credentials
        default: true
        objectStorage:
          bucket: <bucket_name>
          prefix: velero
        provider: gcp
    configuration:
      nodeAgent:
        enable: true
        uploaderType: kopia
      velero:
        defaultPlugins:
        - csi
        - gcp
        - openshift
        disableFsBackup: true
  ```

  where:

  <dl>
  <dt><code>nodeAgent</code></dt>
  <dd>Specifies to enable the node agent in the DPA.</dd>
  <dt><code>disableFsBackup</code></dt>
  <dd>Specifies to set the <code>disableFsBackup</code> field to <code>true</code>.</dd>
  </dl>

**Verification**

1. Verify that the node agent security context is set to run as non-root and the root file system is `readOnly` by running the following command:
   ```terminal
   $ oc get daemonset node-agent -o yaml
   ```

   The example output is as following:

   ```yaml
   apiVersion: apps/v1
   kind: DaemonSet
   metadata:
     ...
     name: node-agent
     namespace: openshift-adp
     ...
   spec:
     ...
     template:
       metadata:
         ...
       spec:
         containers:
         ...
           securityContext:
             allowPrivilegeEscalation: false
             capabilities:
               drop:
               - ALL
             privileged: false
             readOnlyRootFilesystem: true
           ...
         nodeSelector:
           kubernetes.io/os: linux
         os:
           name: linux
         restartPolicy: Always
         schedulerName: default-scheduler
         securityContext:
           runAsNonRoot: true
           seccompProfile:
             type: RuntimeDefault
         serviceAccount: velero
         serviceAccountName: velero
         ....
   ```

   where:

   <dl>
   <dt><code>allowPrivilegeEscalation</code></dt>
   <dd>Specifies that the <code>allowPrivilegeEscalation</code> field is false.</dd>
   <dt><code>privileged</code></dt>
   <dd>Specifies that the <code>privileged</code> field is false.</dd>
   <dt><code>readOnlyRootFilesystem</code></dt>
   <dd>Specifies that the root file system is read-only.</dd>
   <dt><code>runAsNonRoot</code></dt>
   <dd>Specifies that the node agent is run as a non-root user.</dd>
   </dl>

## Configuring node agents and node labels {#oadp-configuring-node-agents_installing-oadp-kubevirt}

The Data Protection Application (DPA) uses the `nodeSelector` field to select which nodes can run the node agent. The `nodeSelector` field is the recommended form of node selection constraint.

**Procedure**

1. Run the node agent on any node that you choose by adding a custom label:
   ```terminal
   $ oc label node/<node_name> node-role.kubernetes.io/nodeAgent=""
   ```

   :::note

   Any label specified must match the labels on each node.

   :::
2. Use the same custom label in the `DPA.spec.configuration.nodeAgent.podConfig.nodeSelector` field, which you used for labeling nodes:
   ```terminal
   configuration:
     nodeAgent:
       enable: true
       podConfig:
         nodeSelector:
           node-role.kubernetes.io/nodeAgent: ""
   ```

   The following example is an anti-pattern of `nodeSelector` and does not work unless both labels, `node-role.kubernetes.io/infra: ""` and `node-role.kubernetes.io/worker: ""`, are on the node:

   ```terminal
       configuration:
         nodeAgent:
           enable: true
           podConfig:
             nodeSelector:
               node-role.kubernetes.io/infra: ""
               node-role.kubernetes.io/worker: ""
   ```

## Configuring node agent load affinity {#oadp-configuring-node-agent-load-affinity_installing-oadp-kubevirt}

You can schedule the node agent pods on specific nodes by using the `spec.podConfig.nodeSelector` object of the `DataProtectionApplication` (DPA) custom resource (CR).

See the following example in which you can schedule the node agent pods on nodes with the label `label.io/role: cpu-1` and `other-label.io/other-role: cpu-2`.

```yaml
...
spec:
  configuration:
    nodeAgent:
      enable: true
      uploaderType: kopia
      podConfig:
        nodeSelector:
          label.io/role: cpu-1
          other-label.io/other-role: cpu-2
        ...
```

You can add more restrictions on the node agent pods scheduling by using the `nodeagent.loadAffinity` object in the DPA spec.

**Prerequisites**

- You must be logged in as a user with `cluster-admin` privileges.
- You have installed the OADP Operator.
- You have configured the DPA CR.

**Procedure**

- Configure the DPA spec `nodegent.loadAffinity` object as shown in the following example.
  In the example, you ensure that the node agent pods are scheduled only on nodes with the label `label.io/role: cpu-1` and the label `label.io/hostname` matching with either `node1` or `node2`.

  ```yaml
  ...
  spec:
    configuration:
      nodeAgent:
        enable: true
        loadAffinity:
          - nodeSelector:
              matchLabels:
                label.io/role: cpu-1
              matchExpressions:
                - key: label.io/hostname
                  operator: In
                  values:
                    - node1
                    - node2
                    ...
  ```

  where:

  <dl>
  <dt><code>loadAffinity</code></dt>
  <dd>Specifies the <code>loadAffinity</code> object by adding the <code>matchLabels</code> and <code>matchExpressions</code> objects.</dd>
  <dt><code>matchExpressions</code></dt>
  <dd>Specifies the <code>matchExpressions</code> object to add restrictions on the node agent pods scheduling.</dd>
  </dl>

## Node agent load affinity guidelines {#oadp-node-agent-load-affinity-guidelines_installing-oadp-kubevirt}

Use the following guidelines to configure the node agent `loadAffinity` object in the `DataProtectionApplication` (DPA) custom resource (CR).

- Use the `spec.nodeagent.podConfig.nodeSelector` object for simple node matching.
- Use the `loadAffinity.nodeSelector` object without the `podConfig.nodeSelector` object for more complex scenarios.
- You can use both `podConfig.nodeSelector` and `loadAffinity.nodeSelector` objects, but the `loadAffinity` object must be equal or more restrictive as compared to the `podConfig` object. In this scenario, the `podConfig.nodeSelector` labels must be a subset of the labels used in the `loadAffinity.nodeSelector` object.
- You cannot use the `matchExpressions` and `matchLabels` fields if you have configured both `podConfig.nodeSelector` and `loadAffinity.nodeSelector` objects in the DPA.
- See the following example to configure both `podConfig.nodeSelector` and `loadAffinity.nodeSelector` objects in the DPA.
  ```yaml
  ...
  spec:
    configuration:
      nodeAgent:
        enable: true
        uploaderType: kopia
        loadAffinity:
          - nodeSelector:
              matchLabels:
                label.io/location: 'US'
                label.io/gpu: 'no'
        podConfig:
          nodeSelector:
            label.io/gpu: 'no'
  ```

## Configuring node agent load concurrency {#oadp-configuring-node-agent-load-concurrency_installing-oadp-kubevirt}

You can control the maximum number of node agent operations that can run simultaneously on each node within your cluster.

You can configure it using one of the following fields of the Data Protection Application (DPA):

- `globalConfig`: Defines a default concurrency limit for the node agent across all nodes.
- `perNodeConfig`: Specifies different concurrency limits for specific nodes based on `nodeSelector` labels. This provides flexibility for environments where certain nodes might have different resource capacities or roles.

**Prerequisites**

- You must be logged in as a user with `cluster-admin` privileges.

**Procedure**

1. If you want to use load concurrency for specific nodes, add labels to those nodes:
   ```terminal
   $ oc label node/<node_name> label.io/instance-type='large'
   ```
2. Configure the load concurrency fields for your DPA instance:
   ```yaml
     configuration:
       nodeAgent:
         enable: true
         uploaderType: kopia
         loadConcurrency:
           globalConfig: 1
           perNodeConfig:
           - nodeSelector:
                 matchLabels:
                    label.io/instance-type: large
             number: 3
   ```

   where:

   <dl>
   <dt><code>globalConfig</code></dt>
   <dd>Specifies the global concurrent number. The default value is 1, which means there is no concurrency and only one load is allowed. The <code>globalConfig</code> value does not have a limit.</dd>
   <dt><code>label.io/instance-type</code></dt>
   <dd>Specifies the label for per-node concurrency.</dd>
   <dt><code>number</code></dt>
   <dd>Specifies the per-node concurrent number. You can specify many per-node concurrent numbers, for example, based on the instance type and size. The range of per-node concurrent number is the same as the global concurrent number. If the configuration file contains a per-node concurrent number and a global concurrent number, the per-node concurrent number takes precedence.</dd>
   </dl>

## Configuring repository maintenance {#oadp-configuring-repository-maintenance_installing-oadp-kubevirt}

OADP repository maintenance is a background job, you can configure it independently of the node agent pods. This means that you can schedule the repository maintenance pod on a node where the node agent is or is not running.

You can use the repository maintenance job affinity configurations in the `DataProtectionApplication` (DPA) custom resource (CR) only if you use Kopia as the backup repository.

You have the option to configure the load affinity at the global level affecting all repositories. Or you can configure the load affinity for each repository. You can also use a combination of global and per-repository configuration.

**Prerequisites**

- You must be logged in as a user with `cluster-admin` privileges.
- You have installed the OADP Operator.
- You have configured the DPA CR.

**Procedure**

- Configure the `loadAffinity` object in the DPA spec by using either one or both of the following methods:
  - Global configuration: Configure load affinity for all repositories as shown in the following example:
    ```yaml
    ...
    spec:
      configuration:
        repositoryMaintenance:
          global:
            podResources:
              cpuRequest: "100m"
              cpuLimit: "200m"
              memoryRequest: "100Mi"
              memoryLimit: "200Mi"
            loadAffinity:
              - nodeSelector:
                  matchLabels:
                    label.io/gpu: 'no'
                  matchExpressions:
                    - key: label.io/location
                      operator: In
                      values:
                        - US
                        - EU
    ```

    where:

    <dl>
    <dt><code>repositoryMaintenance</code></dt>
    <dd>Specifies the <code>repositoryMaintenance</code> object as shown in the example.</dd>
    <dt><code>global</code></dt>
    <dd>Specifies the <code>global</code> object to configure load affinity for all repositories.</dd>
    </dl>
  - Per-repository configuration: Configure load affinity per repository as shown in the following example:
    ```yaml
    ...
    spec:
      configuration:
        repositoryMaintenance:
          myrepositoryname:
            loadAffinity:
              - nodeSelector:
                  matchLabels:
                    label.io/cpu: 'yes'
    ```

    where:

    <dl>
    <dt><code>myrepositoryname</code></dt>
    <dd>Specifies the <code>repositoryMaintenance</code> object for each repository.</dd>
    </dl>

## Configuring Velero load affinity {#oadp-configuring-velero-load-affinity_installing-oadp-kubevirt}

With each OADP deployment, there is one Velero pod and its main purpose is to schedule Velero workloads. To schedule the Velero pod, you can use the `velero.podConfig.nodeSelector` and the `velero.loadAffinity` objects in the `DataProtectionApplication` (DPA) custom resource (CR) spec.

Use the `podConfig.nodeSelector` object to assign the Velero pod to specific nodes. You can also configure the `velero.loadAffinity` object for pod-level affinity and anti-affinity.

The OpenShift scheduler applies the rules and performs the scheduling of the Velero pod deployment.

**Prerequisites**

- You must be logged in as a user with `cluster-admin` privileges.
- You have installed the OADP Operator.
- You have configured the DPA CR.

**Procedure**

- Configure the `velero.podConfig.nodeSelector` and the `velero.loadAffinity` objects in the DPA spec as shown in the following examples:
  - `velero.podConfig.nodeSelector` object configuration:
    ```yaml
    ...
    spec:
      configuration:
        velero:
          podConfig:
            nodeSelector:
              some-label.io/custom-node-role: backup-core
    ```
  - `velero.loadAffinity` object configuration:
    ```yaml
    ...
    spec:
      configuration:
        velero:
          loadAffinity:
            - nodeSelector:
                matchLabels:
                  label.io/gpu: 'no'
                matchExpressions:
                  - key: label.io/location
                    operator: In
                    values:
                      - US
                      - EU
    ```

## Configuring a priority class for node agent and Velero pods {#oadp-configuring-priority-class_installing-oadp-kubevirt}

Configure a `priorityClassName` field for the node agent and Velero pods by editing the `DataProtectionApplication` (DPA) custom resource (CR). This helps you ensure that the Kubernetes scheduler prioritizes OADP pods during resource contention.

By setting a priority class, you ensure that critical OADP pods are scheduled first after events such as worker node outages, when user workloads might otherwise consume available resources.

**Prerequisites**

- The OADP Operator is installed.
- A DPA CR is configured.
- A `PriorityClass` object is created in the cluster.

**Procedure**

1. In your DPA CR, configure the `priorityClassName` field for the node agent, Velero, or both, in the `podConfig` object as shown in the following example:
   ```yaml
   apiVersion: oadp.openshift.io/v1alpha1
   kind: DataProtectionApplication
   metadata:
     name: <dpa_name>
     namespace: openshift-adp
   spec:
   # ...
     configuration:
       nodeAgent:
         enable: true
         uploaderType: kopia
         podConfig:
           priorityClassName: <priority_class_name>
       velero:
         podConfig:
           priorityClassName: <priority_class_name>
   ```

   where:

   <dl>
   <dt><code>&lt;priority_class_name&gt;</code></dt>
   <dd>Specifies the name of an existing <code>PriorityClass</code> to apply to the pods managed by this <code>podConfig</code> object. For example, <code>system-cluster-critical</code>.</dd>
   </dl>
2. Apply the `DataProtectionApplication` CR by running the following command:
   ```terminal
   $ oc apply -f <dpa_file_name>
   ```

**Verification**

1. Verify that the node agent daemon set pods have the correct priority class by running the following command:
   ```terminal
   $ oc get pods -n openshift-adp -l name=node-agent -o jsonpath='{range .items[*]}{.metadata.name}{" "}{.spec.priorityClassName}{"\n"}{end}'
   ```

   ```terminal
   node-agent-xxxxx <priority_class_name>
   node-agent-yyyyy <priority_class_name>
   ```
2. Verify that the Velero deployment pods have the correct priority class by running the following command:
   ```terminal
   $ oc get pods -n openshift-adp -l app.kubernetes.io/name=velero -o jsonpath='{range .items[*]}{.metadata.name}{" "}{.spec.priorityClassName}{"\n"}{end}'
   ```

   ```terminal
   velero-xxxxx <priority_class_name>
   ```

## Overriding the imagePullPolicy setting in the DPA  {#oadp-configuring-imagepullpolicy_installing-oadp-kubevirt}

In OADP 1.4.0 or earlier, the Operator sets the `imagePullPolicy` field of the Velero and node agent pods to `Always` for all images.

In OADP 1.4.1 or later, the Operator first checks if each image has the `sha256` or `sha512` digest and sets the `imagePullPolicy` field accordingly:

- If the image has the digest, the Operator sets `imagePullPolicy` to `IfNotPresent`.
- If the image does not have the digest, the Operator sets `imagePullPolicy` to `Always`.

You can also override the `imagePullPolicy` field by using the `spec.imagePullPolicy` field in the Data Protection Application (DPA).

**Prerequisites**

- You have installed the OADP Operator.

**Procedure**

- Configure the `spec.imagePullPolicy` field in the DPA as shown in the following example:
  ```yaml title="Example Data Protection Application"
  apiVersion: oadp.openshift.io/v1alpha1
  kind: DataProtectionApplication
  metadata:
    name: test-dpa
    namespace: openshift-adp
  spec:
    backupLocations:
      - name: default
        velero:
          config:
            insecureSkipTLSVerify: "true"
            profile: "default"
            region: <bucket_region>
            s3ForcePathStyle: "true"
            s3Url: <bucket_url>
          credential:
            key: cloud
            name: cloud-credentials
          default: true
          objectStorage:
            bucket: <bucket_name>
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
          - kubevirt
          - csi
    imagePullPolicy: Never
  ```

  where:

  <dl>
  <dt><code>imagePullPolicy</code></dt>
  <dd>Specifies the value for <code>imagePullPolicy</code>. In this example, the <code>imagePullPolicy</code> field is set to <code>Never</code>.</dd>
  </dl>

## About incremental backup support {#oadp-about-incremental-backup-support_installing-oadp-kubevirt}

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

- [Application backup and restore operations](/docs/backup_and_restore#application-backup-restore-operations-overview_backup-restore-overview)
- [Backing up applications with File System Backup: Kopia or Restic](/docs/backup_and_restore/application_backup_and_restore/backing_up_and_restoring/oadp-backing-up-applications-restic-doc#oadp-backing-up-applications-restic-doc)
- [OADP plugins](/docs/backup_and_restore/application_backup_and_restore/oadp-features-plugins#oadp-plugins_oadp-features-plugins)
- [`Backup` custom resource (CR)](/docs/backup_and_restore/application_backup_and_restore/backing_up_and_restoring/backing-up-applications#backing-up-applications)
- [`Restore` CR](/docs/backup_and_restore/application_backup_and_restore/backing_up_and_restoring/restoring-applications#restoring-applications)
- [Using Operator Lifecycle Manager in disconnected environments](/docs/disconnected/using-olm#olm-restricted-networks)
- [Velero 1.16](https://velero.io/docs/v1.16)
