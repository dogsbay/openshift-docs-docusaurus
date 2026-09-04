---
title: CIFS/SMB CSI Driver Operator
sidebar_position: 21
---

# CIFS/SMB CSI Driver Operator {#persistent-storage-csi-smb-cifs}

<a id="persistent-storage-csi-smb-cifs"></a>

You can provision and manage Common Internet File System (CIFS)/Server Message Block (SMB) network shares in OpenShift Container Platform by using the CIFS/SMB Container Storage Interface (CSI) Driver Operator, which supports dynamic volume provisioning.

## Overview of the CIFS/SMB Driver Operator {#persistent-storage-csi-smb-cifs-overview_persistent-storage-csi-smb-cifs}

You can provision persistent volumes (PVs) that mount to network file shares by using the CIFS/SMB CSI Driver Operator, which supports dynamic volume provisioning for on-demand storage.

Familiarity with persistent storage and configuring CSI volumes is recommended when working with a CSI Operator and driver. For more information, see "Understanding persistent volumes" and "Configuring CSI volumes".

After installing the CIFS/SMB CSI Driver Operator, OpenShift Container Platform installs corresponding pods for the Operator and driver in the `openshift-cluster-csi-drivers` namespace by default. This allows the CIFS/SMB CSI Driver to create CSI-provisioned persistent volumes (PVs) that mount to CIFS/SMB shares.

CIFS/SMB CSI Driver Operator
: After the CIFS/SMB CSI Driver Operator is installed, it does not create a storage class by default to use to create persistent volume claims (PVCs). However, you can manually create the CIFS/SMB `StorageClass` for dynamic provisioning (see "Dynamic provisioning"). The CIFS/SMB CSI Driver Operator supports dynamic volume provisioning by allowing storage volumes to be created on-demand. This eliminates the need for cluster administrators to pre-provision storage.

CIFS/SMB CSI driver
: The CIFS/SMB CSI driver enables you to create and mount CIFS/SMB PVs.

**Additional resources**

- [Understanding persistent storage](/docs/storage/understanding-persistent-storage#understanding-persistent-storage)
- [Configuring CSI volumes](/docs/storage/container_storage_interface/persistent-storage-csi#persistent-storage-csi)
- [Dynamic provisioning](/docs/storage/container_storage_interface/persistent-storage-csi-smb-cifs#persistent-storage-csi-smb-cifs-provision-dynamic_persistent-storage-csi-smb-cifs)

## About CSI {#csi-about_persistent-storage-csi-smb-cifs}

The Container Storage Interface (CSI) enables storage vendors to deliver plugins through a standard interface without modifying Kubernetes core code, replacing traditional embedded storage drivers.

CSI Operators give OpenShift Container Platform users storage options, such as volume snapshots, that are not possible with in-tree volume plugins.

## Limitations of the CIFS/SMB Driver Operator {#persistent-storage-csi-smb-cifs-limits_persistent-storage-csi-smb-cifs}

Before you deploy Common Internet File System (CIFS)/Server Message Block (SMB) storage, review the CSI driver limitations, including no support for FIPS mode, HTTP proxy, DFS, or Kerberos authentication.

The following limitations apply to the CIFS/SMB CSI Driver Operator:

- FIPS mode is not supported:
  When Federal Information Processing Standards (FIPS) mode is enabled, the use of md4 and md5 are disabled, which prevents users from using ntlm, ntlmv2, or ntlmssp authentication. Also, signing cannot be used because it uses md5. Any CIFS mount that uses these methods fails when FIPS mode is enabled.
- Using HTTP proxy configuration to connect to outside of the cluster SMB servers is not supported by the CSI driver.
  Since CIFS/SMB is a LAN protocol, and though it can be routed to subnets, it is not designed to be extended over the WAN, and does not support HTTP proxy settings.
- The CIFS/SMB CSI Driver Operator does *not* support Windows Distributed File System (DFS).
- Kerberos authentication is not supported.
- SMB CSI was tested with Samba v4.21.2 and Windows Server 2019 and Windows Server 2022.

## Installing the CIFS/SMB CSI Driver Operator {#persistent-storage-csi-olm-operator-install_persistent-storage-csi-smb-cifs}

Install and configure the CIFS/SMB Container Storage Interface (CSI) Driver Operator to enable CIFS/SMB storage in your cluster. This Red Hat Operator is not installed by default and requires manual installation.

**Prerequisites**

- Access to the OpenShift Container Platform web console.

**Procedure**

1. Log in to the web console.
2. Install the CIFS/SMB CSI Operator:
   1. Click **Ecosystem** → **Software Catalog**.
   2. Locate the CIFS/SMB CSI Operator by typing **CIFS/SMB CSI** in the filter box.
   3. Click the **CIFS/SMB CSI Driver Operator** button.
   4. On the **CIFS/SMB CSI Driver Operator** page, click **Install**.
   5. On the **Install Operator** page, ensure that:
      - **All namespaces on the cluster (default)** is selected.
      - **Installed Namespace** is set to **openshift-cluster-csi-drivers**.
   6. Click **Install**.
      After the installation finishes, the CIFS/SMB CSI Operator is listed in the **Installed Operators** section of the web console.

**Next steps**

Install the CIFS/SMB CSI Driver.

## Installing the CIFS/SMB CSI Driver {#persistent-storage-csi-smb-cifs-driver-install_persistent-storage-csi-smb-cifs}

To use network file shares as persistent volumes (PVs) in your cluster, create a `ClusterCSIDriver` resource after installing the CIFS/SMB CSI Driver Operator.

**Prerequisites**

- Access to the OpenShift Container Platform web console.
- CIFS/SMB CSI Driver Operator installed.

**Procedure**

1. Click **Administration** → **CustomResourceDefinitions** → **ClusterCSIDriver**.
2. On the **Instances** tab, click **Create ClusterCSIDriver**.
3. Use the following YAML file:
   ```yaml
   apiVersion: operator.openshift.io/v1
   kind: ClusterCSIDriver
   metadata:
       name: smb.csi.k8s.io
   spec:
     managementState: Managed
   ```
4. Click **Create**.
5. Wait for the following Conditions to change to a "True" status:
   - `SambaDriverControllerServiceControllerAvailable`
   - `SambaDriverNodeServiceControllerAvailable`

## Dynamic provisioning for CIFS/SMB {#persistent-storage-csi-smb-cifs-provision-dynamic_persistent-storage-csi-smb-cifs}

To automatically provision Common Internet File System (CIFS)/Server Message Block (SMB) volumes without pre-creating shares, create a Secret for server credentials, a storage class for the SMB source, and persistent volume claims (PVCs).

When dynamically provisioning volumes, a subdirectory is created with the persistent volume (PV) name under `source` defined in the storage class.

**Prerequisites**

- CIFS/SMB CSI Driver Operator and driver installed.
- You are logged in to the running OpenShift Container Platform cluster.
- You have installed the SMB server and know the following information about the server:
  - Hostname
  - Share name
  - Username and password

**Procedure**

1. Create a Secret for access to the Samba server running the following command with the following example YAML file:
   ```terminal
   $ oc create -f <file_name>.yaml
   ```

   ```yaml title="Example Secret YAML file"
   apiVersion: v1
   kind: Secret
   metadata:
     name: smbcreds
     namespace: samba-server
   stringData:
     username: <username>
     password: <password>
   ```

   - `metadata.name`: Specifies the name of the Secret for the Samba server.
   - `metadata.namespace`: Specifies the namespace for the Secret for the Samba server.
   - `stringData.username`: Specifies the username for the Secret for the Samba server.
   - `stringData.password`: Specifies the password for the Secret for the Samba server.
2. Create a storage class by running the following command with the following example YAML file:
   ```terminal
   $ oc create -f <sc_file_name>.yaml
   ```

   Where `<sc_file_name>.yaml` is the name of the storage class YAML file.

   ```yaml title="Example storage class YAML file"
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: <sc_name>
   provisioner: smb.csi.k8s.io
   parameters:
     source: //<hostname>/<shares>
     csi.storage.k8s.io/provisioner-secret-name: smbcreds
     csi.storage.k8s.io/provisioner-secret-namespace: samba-server
     csi.storage.k8s.io/node-stage-secret-name: smbcreds
     csi.storage.k8s.io/node-stage-secret-namespace: samba-server
   reclaimPolicy: Delete
   volumeBindingMode: Immediate
   mountOptions:
     - dir_mode=0777
     - file_mode=0777
     - uid=1001
     - gid=1001
   ```

   - `metadata.name`: Specifies the name of the storage class.
   - `parameters.source`: The Samba server must be installed somewhere that is reachable from the cluster with `<hostname>` being the hostname for the Samba server and `<shares>` the path the server is configured to have among the exported shares.
   - `parameters.csi.storage.k8s.io/provisioner-secret-name`: Specifies the name of the Secret for the Samba server that was set in the previous step. If the `csi.storage.k8s.io/provisioner-secret` is provided, a subdirectory is created with the PV name under `source`.
   - `parameters.csi.storage.k8s.io/provisioner-secret-namespace`: Specifies the namespace for the Secret for the Samba server that was set in the previous step.
3. Create a PVC:
   1. Create a PVC by running the following command with the following example YAML file:
      ```terminal
      $ oc create -f <pv_file_name>.yaml
      ```

      Where `<pv_file_name>.yaml` is the name of the PVC YAML file.

      ```yaml title="Example PVC YAML file"
      kind: PersistentVolumeClaim
      apiVersion: v1
      metadata:
        name: <pvc_name>
      spec:
        accessModes:
          - ReadWriteMany
        resources:
          requests:
            storage: <storage_amount>
        storageClassName: <sc_name>
      ```

      - `metadata.name`: Specifies the name of the PVC.
      - `spec.resources.requests.storage`: Specifies the storage request amount.
      - `spec.storage.ClassName`: Specifies the name of the CIFS/SMB storage class that you created in the previous step.
   2. Ensure that the PVC was created and is in the "Bound" status by running the following command:
      ```terminal
      $ oc describe pvc <pvc_name>
      ```

      Where `<pvc_name>` is the name of the PVC that you created in the preceding step.

      ```terminal title="Example output"
      Name:          pvc-test
      Namespace:     default
      StorageClass:  samba
      Status:        Bound
      ...
      ```

      PVC is in Bound status.

## Static provisioning for CIFS/SMB {#persistent-storage-csi-smb-cifs-provision-static_persistent-storage-csi-smb-cifs}

You can use static provisioning to create a persistent volume (PV) and persistent volume claim (PVC) to consume existing Server Message Block protocol (SMB) shares:

**Prerequisites**

- Access to the OpenShift Container Platform web console.
- CIFS/SMB CSI Driver Operator and driver installed.
- You have installed the SMB server and know the following information about the server:
  - Hostname
  - Share name
  - Username and password

**Procedure**

1. Create a Secret for access to the Samba server running the following command with the following example YAML file: $ oc create -f <file_name>.yaml
   ```yaml title="Example Secret YAML file"
   apiVersion: v1
   kind: Secret
   metadata:
     name: smbcreds
     namespace: samba-server
   stringData:
     username: <username>
     password: <password>
   ```

   - `metadata.name`: Specifies the name of the Secret for the Samba server.
   - `metadata.namespace`: Specifies the namespace for the Secret for the Samba server.
   - `stringData.username`: Specifies the username for the Secret for the Samba server.
   - `stringData.password`: Specifies the password for the Secret for the Samba server.
2. Create a PV by running the following command with the following example YAML file:
   ```terminal
   $ oc create -f <pv_file_name>.yaml
   ```

   Where `<pv_file_name>.yaml` is the name of the PV YAML file.

   ```yaml title="Example PV YAML file"
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     annotations:
       pv.kubernetes.io/provisioned-by: smb.csi.k8s.io
     name: <pv_name>
   spec:
     capacity:
       storage: 100Gi
     accessModes:
       - ReadWriteMany
     persistentVolumeReclaimPolicy: Retain
     storageClassName: ""
     mountOptions:
       - dir_mode=0777
       - file_mode=0777
     csi:
       driver: smb.csi.k8s.io
       volumeHandle: smb-server.default.svc.cluster.local/share##
       volumeAttributes:
         source: //<hostname>/<shares>
       nodeStageSecretRef:
         name: <secret_name_shares>
         namespace: <namespace>
   ```

   - `metadata.name`: Specifies the name of the PV.
   - `spec.csi.volumeHandle` format: `{smb_server_address}<mark>{sub_dir_name}</mark>{share_name}`. Ensure that this value is unique for every share in the cluster.
   - `spec.csi.volumeAttributes.source`: The Samba server must be installed somewhere that is reachable from the cluster with `<hostname>` being the hostname for the Samba server and <shares> the path the server is configured to have among the exported shares.
   - `spec.csi.nodeStageSecretRef.name`: Specifies the name of the Secret for the shares.
   - `spec.csi.nodeStageSecretRef.namespace`: Specifies the applicable namespace.
3. Create a PVC:
   1. Create a PVC by running the following command with the following example YAML file:
      ```terminal
      $ oc create -f <pv_file_name>.yaml
      ```

      Where `<pv_file_name>.yaml` is the name of the PVC YAML file.

      ```yaml title="Example PVC YAML file"
      kind: PersistentVolumeClaim
      apiVersion: v1
      metadata:
        name: <pvc_name>
      spec:
        accessModes:
          - ReadWriteMany
        resources:
          requests:
            storage: <storage_amount>
        storageClassName: ""
        volumeName: <pv_name>
      ```

      - `metadata.name`: Specifies the name of the PVC.
      - `spec.resources.requests.storage`: Specifies the storage request amount.
      - `spec.volumeName`: Specifies the name of the PV from the first step.
   2. Ensure that the PVC was created and is in the `Bound` status by running the following command:
      ```terminal
      $ oc describe pvc <pvc_name>
      ```

      Where `<pvc_name>` is the name of the PVC that you created in the preceding step.

      ```terminal title="Example output"
      Name:          pvc-test
      Namespace:     default
      StorageClass:
      Status:        Bound
      ...
      ```

      PVC is in `Bound` status.
4. Create a deployment on Linux by running the following command with the following example YAML file:
   :::note

   The following deployment is not mandatory for using the PV and PVC created in the previous steps. It is example of how they can be used.

   :::

   ```terminal
   $ oc create -f <deployment_file_name>.yaml
   ```

   Where `<deployment_file_name>.yaml` is the name of the deployment YAML file.

   ```yaml title="Example deployment YAML file"
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     labels:
       app: nginx
     name: <deployment_name>
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: nginx
     template:
       metadata:
         labels:
           app: nginx
         name: <deployment_name>
       spec:
         nodeSelector:
           "kubernetes.io/os": linux
         containers:
           - name: <deployment_name>
             image: quay.io/centos/centos:stream8
             command:
               - "/bin/bash"
               - "-c"
               - set -euo pipefail; while true; do echo $(date) >> <mount_path>/outfile; sleep 1; done (2)
             volumeMounts:
               - name: <vol_mount_name>
                 mountPath: <mount_path>
                 readOnly: false
         volumes:
           - name: <vol_mount_name>
             persistentVolumeClaim:
               claimName: <pvc_name>
     strategy:
       rollingUpdate:
         maxSurge: 0
         maxUnavailable: 1
       type: RollingUpdate
   ```

   - `metadata.name` and `spec.template.name`: Specifies the name of the deployment.
   - `spec.template.spec.containers.command.set` and `spec.template.spec.containers.volumeMounts.mountpath`: Specifies the volume mount path.
   - `spec.template.spec.containers.volumeMounts.mountpath` and `spec.template.spec.volumes.name`: Specifies the name of the volume mount.
   - `spec.template.spec.volumes.persistentVolumeClaim.claimName`: Specifies the name of the PVC created in the preceding step.
5. Check the setup by running the `df -h` command in the container:
   ```terminal
   $ oc exec -it <pod_name> -- df -h
   ```

   Where `<pod_name>` is the name of the pod.

   ```terminal title="Example output"
   Filesystem            Size  Used Avail Use% Mounted on
   ...
   /dev/sda1              97G   21G   77G  22% /etc/hosts
   //20.43.191.64/share   97G   21G   77G  22% /mnt/smb
   ...
   ```

   In this example, there is a `/mnt/smb` directory mounted as a Common Internet File System (CIFS) filesystem.
