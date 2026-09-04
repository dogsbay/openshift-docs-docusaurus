---
title: Persistent storage using Azure File
sidebar_position: 3
---

# Persistent storage using Azure File

<a id="persistent-storage-using-azure-file"></a>

OpenShift Container Platform supports Microsoft Azure File volumes. You can provision your OpenShift Container Platform cluster with persistent storage using Azure. Some familiarity with Kubernetes and Azure is assumed.

The Kubernetes persistent volume framework allows administrators to provision a cluster with persistent storage and gives users a way to request those resources without having any knowledge of the underlying infrastructure. You can provision Azure File volumes dynamically.

Persistent volumes are not bound to a single project or namespace, and you can share them across the OpenShift Container Platform cluster. Persistent volume claims are specific to a project or namespace, and can be requested by users for use in applications.

:::warning

High availability of storage in the infrastructure is left to the underlying storage provider.

:::

:::warning

Azure File volumes use Server Message Block.

:::

:::warning

OpenShift Container Platform 4.13 and later provides automatic migration for the Azure File in-tree volume plugin to its equivalent CSI driver.

CSI automatic migration should be seamless. Migration does not change how you use all existing API objects, such as persistent volumes, persistent volume claims, and storage classes. For more information about migration, see "CSI automatic migration".

:::

**Additional resources**

- [CSI automatic migration](/docs/storage/container_storage_interface/persistent-storage-csi-migration#persistent-storage-csi-migration)
- [Azure Files](https://azure.microsoft.com/en-us/services/storage/files/)

## Create the Azure File share persistent volume claim {#create-azure-file-secret_persistent-storage-azure-file}

To create the persistent volume claim, you must first define a `Secret` object that contains the Azure account and key. This secret is used in the `PersistentVolume` definition, and will be referenced by the persistent volume claim for use in applications.

**Prerequisites**

- An Azure File share exists.
- The credentials to access this share, specifically the storage account and key, are available.

**Procedure**

1. Create a `Secret` object that contains the Azure File credentials:
   ```terminal
   $ oc create secret generic __<secret-name>__ --from-literal=azurestorageaccountname=__<storage-account> --from-literal=azurestorageaccountkey=__<storage-account-key>
   ```

   where:

   <dl>
   <dt><code>&lt;secret-name&gt;</code></dt>
   <dd>Specifies the Azure File storage account name.</dd>
   <dt><code>&lt;storage-account-key&gt;</code></dt>
   <dd>Specifies the Azure File storage account key.</dd>
   </dl>
2. Create a `PersistentVolume` object that references the `Secret` object you created:
   ```yaml
   apiVersion: "v1"
   kind: "PersistentVolume"
   metadata:
     name: "pv0001"
   spec:
     capacity:
       storage: "5Gi"
     accessModes:
       - "ReadWriteOnce"
     storageClassName: azure-file-sc
     azureFile:
       secretName: <secret-name>
       shareName: <share-name>
       readOnly: false
   ```

   where:

   <dl>
   <dt><code>metadata.name</code></dt>
   <dd>Specifies the name of the persistent volume.</dd>
   <dt><code>spec.capacity.storage</code></dt>
   <dd>Specifies the size of this persistent volume, for example <code>5Gi</code>.</dd>
   <dt><code>spec.azureFile.secretName</code></dt>
   <dd>Specifies the name of the secret that contains the Azure File share credentials.</dd>
   <dt><code>spec.azureFile.shareName</code></dt>
   <dd>Specifies the name of the Azure File share.</dd>
   </dl>
3. Create a `PersistentVolumeClaim` object that maps to the persistent volume you created:
   ```yaml
   apiVersion: "v1"
   kind: "PersistentVolumeClaim"
   metadata:
     name: "claim1"
   spec:
     accessModes:
       - "ReadWriteOnce"
     resources:
       requests:
         storage: "5Gi"
     storageClassName: azure-file-sc
     volumeName: "pv0001"
   ```

   where:

   <dl>
   <dt><code>metadata.name</code></dt>
   <dd>Specifies the name of the persistent volume claim.</dd>
   <dt><code>spec.resources.requests.storage</code></dt>
   <dd>Specifies the size of this persistent volume claim, for example <code>5Gi</code>.</dd>
   <dt><code>spec.storageClassName</code></dt>
   <dd>Specifies the name of the existing <code>PersistentVolume</code> object that references the Azure File share. Specify the storage class used in the <code>PersistentVolume</code> definition.</dd>
   <dt><code>spec.volumeName</code></dt>
   <dd>Specifies the name of the existing <code>PersistentVolume</code> object that references the Azure File share.</dd>
   </dl>

## Mount the Azure File share in a pod {#create-azure-file-pod_persistent-storage-azure-file}

After you create a persistent volume (PV), you can use the PV inside by an application.

The following example demonstrates mounting this share inside of a pod.

**Prerequisites**

- A persistent volume claim exists that is mapped to the underlying Azure File share.

**Procedure**

- Create a pod that mounts the existing persistent volume claim:
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: pod-name
  spec:
    containers:
      ...
      volumeMounts:
      - mountPath: "/data"
        name: azure-file-share
    volumes:
      - name: azure-file-share
        persistentVolumeClaim:
          claimName: claim1
  ```

  where:

  <dl>
  <dt><code>metadata.name</code></dt>
  <dd>Specifies the name of the pod.</dd>
  <dt><code>spec.containers.volumeMounts.mountPath</code></dt>
  <dd>Specifies the path to mount the Azure File share inside the pod, for example <code>/data</code>. Do not mount to the container root, <code>/</code>, or any path that is the same in the host and the container. This can corrupt your host system if the container is sufficiently privileged, such as the host <code>/dev/pts</code> files. It is safe to mount the host by using <code>/host</code>.</dd>
  <dt><code>spec.volumes.persistentVolumeClaim.claimName</code></dt>
  <dd>Specifies the name of the <code>PersistentVolumeClaim</code> object that has been previously created.</dd>
  </dl>
