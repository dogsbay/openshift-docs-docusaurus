---
title: Hot-plug virtual disks to running VMs
sidebar_position: 1
---

# Hot-plug virtual disks to running VMs {#virt-hot-plugging-virtual-disks}

<a id="virt-hot-plugging-virtual-disks"></a>

You can hot-plug or hot-unplug virtual disks from running VMs to dynamically adjust storage without downtime.

Only data volumes and persistent volume claims (PVCs) can be hot plugged and hot-unplugged. You cannot hot plug or hot-unplug container disks.

A hot plugged disk remains attached to the VM even after reboot. You must unplug the disk to remove it from the VM.

:::note

Each VM has a `virtio-scsi` controller so that hot plugged disks can use the SCSI bus. The `virtio-scsi` controller overcomes the limitations of VirtIO while retaining its performance advantages. It is highly scalable and supports hot plugging over 4 million disks.

When you hot plug disks to the VirtIO (`virtio-blk`) bus, each disk uses a PCI Express (PCIe) slot in the VM. The number of PCIe slots is limited and pre-set automatically at the VM creation as specified in the "Available VirtIO Ports" table. Therefore, you can use `virtio-blk` for a small number of disks that does not exceed the number of available slots.

:::

## Hot plugging and hot unplugging a disk by using the web console {#virt-hot-plugging-disks-ui_virt-hot-plugging-virtual-disks}

You can hot plug a disk by attaching it to a virtual machine (VM) while the VM is running by using the OpenShift Container Platform web console.

The hot plugged disk remains attached to the VM until you unplug it.

**Prerequisites**

- You must have a data volume or persistent volume claim (PVC) available for hot plugging.

**Procedure**

1. Navigate to **Virtualization** → **VirtualMachines** in the web console.
2. Select a running VM to view its details.
3. On the **VirtualMachine details** page, click **Configuration** → **Storage**.
4. Add a hot plugged disk:
   1. Click **Add**.
   2. In the **Add disk (hot plugged)** window, select the disk from the **Source** list and click **Save**.
5. Optional: Select the type of the interface bus. The options are **VirtIO** and **SCSI**. The default bus type is **VirtIO**.
6. Optional: Change the type of the interface bus of an existing hot plugged disk:
   1. Click the Options menu ![](/images/kebab.png "Options menu") beside the disk and select the **Edit** option.
   2. In the **Interface** field, select the desired option.
7. Optional: Unplug a hot plugged disk:
   1. Click the Options menu ![](/images/kebab.png "Options menu") beside the disk and select **Detach**.
   2. Click **Detach**.

## Hot plug and hot unplug a disk by using the CLI {#virt-hot-plugging-disk-cli_virt-hot-plugging-virtual-disks}

You can hot plug and hot unplug a disk while a virtual machine (VM) is running by using the command line.

The hot plugged disk remains attached to the VM until you unplug it.

**Prerequisites**

- You must have at least one data volume or persistent volume claim (PVC) available for hot plugging.

**Procedure**

- Hot plug a disk by running the following command:
  ```terminal
  $ virtctl addvolume <virtual-machine|virtual-machine-instance> \
    --volume-name=<datavolume|PVC> \
    [--bus <bus_type>] [--persist] [--serial=<label_name>]
  ```

  where:

  <dl>
  <dt><code>--bus &lt;bus_type&gt;</code></dt>
  <dd>Optional: Specifies the bus type of the added disk. Supported values are <code>virtio</code> and <code>scsi</code>. The default bus type is <code>virtio</code>.</dd>
  <dt><code>--persist</code></dt>
  <dd>Optional: Specifies that the virtual disk is permanently mounted on a virtual machine. This flag does not apply to virtual machine instances.</dd>
  <dt><code>--serial=&lt;label_name&gt;</code></dt>
  <dd>Optional: Specifies an alphanumeric string label of your choice to identify the hot plugged disk in a guest virtual machine. If you do not specify this option, the label defaults to the name of the hot plugged data volume or PVC.</dd>
  </dl>
- Hot unplug a disk by running the following command:
  ```terminal
  $ virtctl removevolume <virtual-machine|virtual-machine-instance> \
    --volume-name=<datavolume|PVC>
  ```

**Additional resources**

- [Available VirtIO Ports](https://kubevirt.io/user-guide/storage/hotplug_volumes/#available-virtio-ports)
