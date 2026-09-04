---
title: Expand virtual machine disks
sidebar_position: 2
---

# Expand virtual machine disks {#virt-expanding-vm-disks}

<a id="virt-expanding-vm-disks"></a>

Expand the  persistent volume claim (PVC) of your virtual machine disk to accomodate growing data requirements. If your storage provider does not support volume expansion, you can expand the available virtual storage of a VM by adding blank data volumes.

You cannot reduce the size of a VM disk.

## Expand a VM disk PVC by using the web console {#virt-expanding-vm-disk-pvc-web-console_virt-expanding-vm-disks}

You can increase the size of a virtual machine (VM) disk by expanding the persistent volume claim (PVC) of the disk. To specify the increased PVC volume, you can use the **VirtualMachines** page in the web console, with the VM running.

:::note

If the PVC uses the file system volume mode, the disk image file expands to the available size while reserving some space for file system overhead.

:::

**Procedure**

1. In the **Administrator** or **Virtualization** perspective, open the **VirtualMachines** page.
2. Select the running VM to open its **Details** page.
3. Select the **Configuration** tab and click **Storage**.
4. Click the options menu ![](/images/kebab.png "Options menu") next to the disk you want to expand. Select the **Edit** option.
   The **Edit disk** dialog opens.
5. In the **PersistentVolumeClaim size** field, enter the desired size.
   :::note

   You can enter any value greater than the current one. However, if the new value exceeds the available size, an error is displayed.

   :::
6. Click **Save**.

## Expanding a VM disk PVC by using the CLI {#virt-expanding-vm-disk-pvc-cli_virt-expanding-vm-disks}

You can increase the size of a virtual machine (VM) disk by expanding the persistent volume claim (PVC) of the disk. To specify the increased PVC volume, you can edit the `PersistentVolumeClaim` manifest by using the OpenShift CLI (`oc`).

:::note

If the PVC uses the file system volume mode, the disk image file expands to the available size while reserving some space for file system overhead.

:::

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).

**Procedure**

1. Edit the `PersistentVolumeClaim` manifest of the VM disk that you want to expand:
   ```terminal
   $ oc edit pvc <pvc_name>
   ```
2. Update the disk size:
   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
      name: vm-disk-expand
   spec:
     accessModes:
        - ReadWriteMany
     resources:
       requests:
          storage: 3Gi
   # ...
   ```

   - `spec.resources.requests.storage` specifies the new disk size.

## Expanding available virtual storage by adding blank data volumes {#virt-expanding-storage-with-data-volumes_virt-expanding-vm-disks}

You can expand the available storage of a virtual machine (VM) by adding blank data volumes.

**Prerequisites**

- You must have at least one persistent volume.
- You have installed the OpenShift CLI (`oc`).

**Procedure**

1. Create a `DataVolume` manifest as shown in the following example:
   ```yaml
   apiVersion: cdi.kubevirt.io/v1beta1
   kind: DataVolume
   metadata:
     name: blank-image-datavolume
   spec:
     source:
       blank: {}
     storage:
       resources:
         requests:
           storage: <2Gi>
     storageClassName: "<storage_class>"
   ```

   - `spec.storage.resources.requests.storage` specifies the amount of available space requested for the data volume.
   - `spec.storageClassName` is an optional field that specifies a storage class. If you do not specify a storage class, the default storage class is used.
2. Create the data volume by running the following command:
   ```terminal
   $ oc create -f <blank-image-datavolume>.yaml
   ```

**Additional resources**

- [Extending a basic volume in Windows](https://docs.microsoft.com/en-us/windows-server/storage/disk-management/extend-a-basic-volume)
- [Extending an existing file system partition without destroying data in Red Hat Enterprise Linux](https://access.redhat.com/solutions/29095)
- [Extending a logical volume and its file system online in Red Hat Enterprise Linux](https://access.redhat.com/solutions/24770)
- [Configuring preallocation mode for data volumes](/docs/virt/storage/virt-using-preallocation-for-datavolumes#virt-using-preallocation-for-datavolumes)
- [Managing data volume annotations](/docs/virt/storage/virt-managing-data-volume-annotations#virt-managing-data-volume-annotations)
