---
title: Edit the configuration of a virtual machine
sidebar_position: 8
---

# Edit the configuration of a virtual machine {#virt-edit-vms}

<a id="virt-edit-vms"></a>

You can update virtual machine (VM) configuration details like CPU, memory, and networking by using the CLI or the OpenShift Container Platform web console. In the web console, you can modify settings on the **VirtualMachine details** page or by editing the YAML file directly.

To edit a VM to configure disk sharing by using virtual disks or LUN, see "Configuring shared volumes for virtual machines".

## Changing the instance type of a VM by using the web console {#virt-change-vm-instance-type_virt-edit-vms}

You can change the instance type associated with a running virtual machine (VM) by using the web console. The change takes effect immediately.

**Prerequisites**

- You created the VM by using an instance type.

**Procedure**

1. In the OpenShift Container Platform web console, click **Virtualization** → **VirtualMachines**.
2. Select a VM to open the **VirtualMachine details** page.
3. Click the **Configuration** tab.
4. On the **Details** tab, click the instance type text to open the **Edit Instancetype** dialog. For example, click **1 CPU | 2 GiB Memory**.
5. Edit the instance type by using the **Series** and **Size** lists.
   1. Select an item from the **Series** list to show the relevant sizes for that series. For example, select **General Purpose**.
   2. Select the new instance type for the VM from the **Size** list. For example, select **medium: 1 CPUs, 4Gi Memory**, which is available in the **General Purpose** series.
6. Click **Save**.

**Verification**

1. Click the **YAML** tab.
2. Click **Reload**.
3. Review the VM YAML to confirm that the instance type changed.

## Hot plugging memory on a virtual machine {#virt-hot-plugging-memory_virt-edit-vms}

You can add or remove the amount of memory allocated to a virtual machine (VM) without having to restart the VM by using the OpenShift Container Platform web console.

**Procedure**

1. Navigate to **Virtualization** → **VirtualMachines**.
2. Select the required VM to open the **VirtualMachine details** page.
3. On the **Configuration** tab, click **Edit CPU|Memory**.
4. Enter the required amount of memory and click **Save**.
   :::note

   By hot plugging, you can increase the total amount of memory of a VM up to four times the default initial amount. Exceeding this limit requires a restart.

   :::

   The system applies these changes immediately. If the VM is able to be migrated, a live migration is triggered. If not, or if the changes cannot be live-updated, a `RestartRequired` condition is added to the VM.

   :::note

   Memory hot plugging for virtual machines requires guest operating system support for the `virtio-mem` driver. This support depends on the driver being included and enabled within the guest operating system, not on specific upstream kernel versions.

   Supported guest operating systems:

   - RHEL 9.4 and later
   - RHEL 8.10 and later (hot-unplug is disabled by default)
   - Other Linux guests require kernel version 5.16 or later and the `virtio-mem` kernel module
   - Windows guests require `virtio-mem` driver version 100.95.104.26200 or later

   :::

## Hot plugging CPUs on a virtual machine {#virt-hot-plugging-cpu_virt-edit-vms}

You can increase or decrease the number of CPU sockets allocated to a virtual machine (VM) without having to restart the VM by using the OpenShift Container Platform web console.

**Procedure**

1. Navigate to **Virtualization** → **VirtualMachines**.
2. Select the required VM to open the **VirtualMachine details** page.
3. On the **Configuration** tab, click **Edit CPU|Memory**.
4. Select the **vCPU** radio button.
5. Enter the required number of vCPU sockets and click **Save**.
   :::note

   By hot plugging, you can increase the total number of vCPU sockets of a VM up to four times the default initial number. Exceeding this limit requires a restart.

   :::

   If the VM is migratable, a live migration is triggered. If not, or if the changes cannot be live-updated, a `RestartRequired` condition is added to the VM.

   :::note

   If a VM has the `spec.template.spec.domain.devices.networkInterfaceMultiQueue` field enabled and CPUs are hot plugged, the following behavior occurs:

   - Existing network interfaces that you attach before the CPU hot plug retain their original queue count, even after you add more virtual CPUs (vCPUs). The underlying virtualization technology causes this expected behavior.
   - To update the queue count of existing interfaces to match the new vCPU configuration, you can restart the VM. A restart is only necessary if the update improves performance.
   - New VirtIO network interfaces that you hot plugged after the CPU hot plug automatically receive a queue count that matches the updated vCPU configuration.

   :::

## Editing a virtual machine by using the CLI {#virt-editing-vm-cli_virt-edit-vms}

You can edit a virtual machine (VM) by using the command line.

**Prerequisites**

- You installed the `oc` CLI.

**Procedure**

1. Obtain the virtual machine configuration by running the following command:
   ```terminal
   $ oc edit vm <vm_name>
   ```
2. Edit the YAML configuration.
3. If you edit a running virtual machine, you need to do one of the following:
   - Restart the virtual machine.
   - Run the following command for the new configuration to take effect:
     ```terminal
     $ oc apply vm <vm_name> -n <namespace>
     ```

## Adding a disk to a virtual machine {#virt-add-disk-to-vm_virt-edit-vms}

You can add a virtual disk to a virtual machine (VM) by using the OpenShift Container Platform web console.

**Procedure**

1. Navigate to **Virtualization** → **VirtualMachines** in the web console.
2. Select a VM to open the **VirtualMachine details** page.
3. On the **Disks** tab, click **Add disk**.
4. Specify the **Source**, **Name**, **Size**, **Type**, **Interface**, and **Storage Class**.
   1. Optional: You can enable preallocation if you use a blank disk source and require maximum write performance when creating data volumes. To do so, select the **Enable preallocation** checkbox.
   2. Optional: You can clear **Apply optimized StorageProfile settings** to change the **Volume Mode** and **Access Mode** for the virtual disk. If you do not specify these parameters, the system uses the default values from the `kubevirt-storage-class-defaults` config map.
5. Click **Add**.
   :::note

   If the VM is running, you must restart the VM to apply the change.

   :::

### Storage fields {#virt-storage-wizard-fields-web_virt-edit-vms}

To optimize storage performance and ensure data availability for your workloads, configure the storage fields to define the source, size, and disk characteristics of your virtual machine (VM).

<table>
<tbody>
<tr>
  <td>Field</td>
  <td>Description</td>
</tr>
<tr>
  <td>Blank (creates PVC)</td>
  <td>Create an empty disk.</td>
</tr>
<tr>
  <td>Import via URL (creates PVC)</td>
  <td>Import content via URL (HTTP or HTTPS endpoint).</td>
</tr>
<tr>
  <td>Use an existing PVC</td>
  <td>Use a PVC that is already available in the cluster.</td>
</tr>
<tr>
  <td>Clone existing PVC (creates PVC)</td>
  <td>Select an existing PVC available in the cluster and clone it.</td>
</tr>
<tr>
  <td>Import via Registry (creates PVC)</td>
  <td>Import content via container registry.</td>
</tr>
<tr>
  <td>Container (ephemeral)</td>
  <td>Upload content from a container located in a registry accessible from the cluster. The container disk should be used only for read-only filesystems such as CD-ROMs or temporary virtual machines.</td>
</tr>
<tr>
  <td>Name</td>
  <td>Name of the disk. The name can contain lowercase letters (<code>a-z</code>), numbers (<code>0-9</code>), hyphens (<code>-</code>), and periods (<code>.</code>), up to a maximum of 253 characters. The first and last characters must be alphanumeric. The name must not contain uppercase letters, spaces, or special characters.</td>
</tr>
<tr>
  <td>Size</td>
  <td>Size of the disk in GiB.</td>
</tr>
<tr>
  <td>Type</td>
  <td>Type of disk. Example: Disk or CD-ROM</td>
</tr>
<tr>
  <td>Interface</td>
  <td>Type of disk device. Supported interfaces are <strong>virtIO</strong>, <strong>SATA</strong>, and <strong>SCSI</strong>.</td>
</tr>
<tr>
  <td>Storage Class</td>
  <td>The storage class that is used to create the disk.</td>
</tr>
</tbody>
</table>

#### Advanced storage settings {#virt-storage-wizard-fields-advanced-web_virt-edit-vms}

The following advanced storage settings are optional and available for **Blank**, **Import via URL**, and **Clone existing PVC** disks.

If you do not specify these parameters, the system uses the default storage profile values.

<table>
<thead>
<tr>
  <th>Parameter</th>
  <th>Option</th>
  <th>Parameter description</th>
</tr>
</thead>
<tbody>
<tr>
  <td rowspan="2">Volume Mode</td>
  <td>Filesystem</td>
  <td>Stores the virtual disk on a file system-based volume.</td>
</tr>
<tr>
  <td>Block</td>
  <td>Stores the virtual disk directly on the block volume. Only use <code>Block</code> if the underlying storage supports it.</td>
</tr>
<tr>
  <td rowspan="3">Access Mode</td>
  <td>ReadWriteOnce (RWO)</td>
  <td>Volume can be mounted as read/write by a single node.</td>
</tr>
<tr>
  <td>ReadWriteMany (RWX)</td>
  <td>Volume can be mounted as read/write by many nodes at one time.<div class="alert alert--secondary"><strong>Note</strong><p>This mode is required for live migration.</p></div></td>
</tr>
<tr>
  <td>ReadOnlyMany (ROX)</td>
  <td>Volume can be mounted as read only by many nodes.<br><br><div class="alert alert--secondary"><strong>Note</strong><p><code>ReadWriteMany</code> access mode is required for live migration.</p></div></td>
</tr>
</tbody>
</table>

## Mounting a Windows driver disk on a virtual machine {#virt-mounting-windows-driver-disk-on-vm_virt-edit-vms}

You can mount a Windows driver disk on a virtual machine (VM) by using the OpenShift Container Platform web console.

**Procedure**

1. Navigate to **Virtualization** → **VirtualMachines**.
2. Select the required VM to open the **VirtualMachine details** page.
3. On the **Configuration** tab, click **Storage**.
4. Select the **Mount Windows drivers disk** checkbox.
   The Windows driver disk is displayed in the list of mounted disks.

## Adding a secret, config map, or service account to a virtual machine {#virt-adding-secret-configmap-service-account-to-vm_virt-edit-vms}

You can add a secret, config map, or service account to a virtual machine by using the OpenShift Container Platform web console.

These resources are added to the virtual machine as disks. You then mount the secret, config map, or service account as you would mount any other disk.

If the virtual machine is running, changes do not take effect until you restart the virtual machine. The newly added resources are marked as pending changes at the top of the page.

**Prerequisites**

- The secret, config map, or service account that you want to add must exist in the same namespace as the target virtual machine.

**Procedure**

1. Click **Virtualization** → **VirtualMachines** from the side menu.
2. Select a virtual machine to open the **VirtualMachine details** page.
3. Click **Configuration** → **Environment**.
4. Click **Add Config Map, Secret or Service Account**.
5. Click **Select a resource** and select a resource from the list. A six character serial number is automatically generated for the selected resource.
6. Optional: Click **Reload** to revert the environment to its last saved state.
7. Click **Save**.

**Verification**

1. On the **VirtualMachine details** page, click **Configuration** → **Disks** and verify that the resource is displayed in the list of disks.
2. Restart the virtual machine by clicking **Actions** → **Restart**.

You can now mount the secret, config map, or service account as you would mount any other disk.

## Updating multiple virtual machines {#virt-updating-multiple-vms_virt-edit-vms}

You can use the command line interface (CLI) to update multiple virtual machines (VMs) at the same time.

**Prerequisites**

- You installed the OpenShift CLI (`oc`).
- You have access to the OpenShift Container Platform cluster, and you have `cluster-admin` permissions.

**Procedure**

1. Create a privileged service account by running the following commands:
   ```terminal
   $ oc adm new-project kubevirt-api-lifecycle-automation
   ```

   ```terminal
   $ oc create sa kubevirt-api-lifecycle-automation -n kubevirt-api-lifecycle-automation
   ```

   ```terminal
   $ oc create clusterrolebinding kubevirt-api-lifecycle-automation --clusterrole=cluster-admin --serviceaccount=kubevirt-api-lifecycle-automation:kubevirt-api-lifecycle-automation
   ```
2. Determine the pull URL for the `kubevirt-api-lifecycle` image by running the following command:
   ```terminal
   $ oc get csv -n openshift-cnv -l=operators.coreos.com/kubevirt-hyperconverged.openshift-cnv -ojson | jq '.items[0].spec.relatedImages[] | select(.name|test(".*kubevirt-api-lifecycle-automation.*")) | .image'
   ```
3. Deploy `Kubevirt-Api-Lifecycle-Automation` by creating a job object as shown in the following example:
   ```yaml
   apiVersion: batch/v1
   kind: Job
   metadata:
    name: kubevirt-api-lifecycle-automation
    namespace: kubevirt-api-lifecycle-automation
   spec:
    template:
     spec:
      containers:
      - name: kubevirt-api-lifecycle-automation
        image: quay.io/openshift-virtualization/kubevirt-api-lifecycle-automation:v4.22
        imagePullPolicy: Always
        env:
        - name: MACHINE_TYPE_GLOB
          value: smth-glob9.10.0
        - name: RESTART_REQUIRED
          value: "true"
        - name: NAMESPACE
          value: "default"
        - name: LABEL_SELECTOR
          value: my-vm
        securityContext:
         allowPrivilegeEscalation: false
         capabilities:
          drop:
          - ALL
         privileged: false
         runAsNonRoot: true
         seccompProfile:
          type: RuntimeDefault
      restartPolicy: Never
      serviceAccountName: kubevirt-api-lifecycle-automation
   ```

   where:

   <dl>
   <dt><code>quay.io/openshift-virtualization/kubevirt-api-lifecycle-automation:v4.22</code></dt>
   <dd>Specifies the pull URL for your image. Replace the image value in this example with your pull URL for the image.</dd>
   <dt><code>MACHINE_TYPE_GLOB</code></dt>
   <dd>Specifies the pattern that is used to detect deprecated machine types that need to be upgraded. Replace the <code>MACHINE_TYPE_GLOB</code> value with your own pattern.</dd>
   <dt><code>RESTART_REQUIRED</code></dt>
   <dd>Specifies whether VMs should be restarted after the machine type is updated. If the <code>RESTART_REQUIRED</code> environment variable is set to <code>true</code>, VMs are restarted after the machine type is updated. If you do not want VMs to be restarted, set this value to <code>false</code>.</dd>
   <dt><code>NAMESPACE</code></dt>
   <dd>Specifies the namespace to look for VMs in. Leave the parameter empty for the job to go over all namespaces in the cluster.</dd>
   <dt><code>LABEL_SELECTOR</code></dt>
   <dd>Specifies which VMs receive the job action. If you want the job to go over all VMs in the cluster, do not assign a value to the parameter.</dd>
   </dl>

### Performing bulk actions on virtual machines {#virt-performing-actions-on-multiple-virtual-machines_virt-edit-vms}

You can perform bulk actions on multiple virtual machines (VMs) simultaneously by using the **VirtualMachines** list view in the web console. This allows you to efficiently manage a group of VMs with minimal manual effort.

Available bulk actions:

- **Label VMs** - Add, edit, or remove labels that are applied across selected VMs.
- **Delete VMs** - Select multiple VMs to delete. The confirmation dialog displays the number of VMs selected for deletion.
- **Move VMs to folder** - Move selected VMs to a folder. All VMs must belong to the same namespace.
- **LiveMigration** - Perform live migration of multiple selected VMs. The confirmation dialog displays the number of VMs selected for migration. The target node is chosen automatically; there is no option of specifying it.
- **Take snapshot** - Take snapshots of multiple VMs. The **Take snapshots** dialog allows you to enter a suffix for the names of the resulting snapshots.

## Configuring multiple I/O threads for fast storage access {#virt-configure-multiple-iothreads_virt-edit-vms}

You can improve storage performance by configuring multiple I/O threads for a virtual machine (VM) that uses fast storage, such as solid-state drive (SSD) or non-volatile memory express (NVMe). This configuration option is only available by editing YAML of the VM.

:::note

Multiple I/O threads are supported only when `blockMultiQueue` is enabled and the disk bus is set to `virtio`. You must set this configuration for the configuration to work correctly.

:::

**Procedure**

1. Click **Virtualization** → **VirtualMachines** from the side menu.
2. Select a virtual machine to open the **VirtualMachine details** page.
3. Click the **YAML** tab to open the VM manifest.
4. In the YAML editor, locate the `spec.template.spec.domain` section and add or modify the following fields:
   ```yaml
   domain:
     ioThreadsPolicy: supplementalPool
     ioThreads:
       supplementalPoolThreadCount: 4
     devices:
       blockMultiQueue: true
       disks:
       - name: datavolume
         disk:
           bus: virtio
   # ...
   ```
5. Click **Save**.
   :::warning

   The `spec.template.spec.domain` setting cannot be changed while the VM is running. You must stop the VM before applying the changes, and then restart the VM for the new settings to take effect.

   :::

**Additional resources**

- [Configuring shared volumes for virtual machines](/docs/virt/managing_vms/virtual_disks/virt-configuring-shared-volumes-for-vms#virt-configuring-shared-volumes-for-vms)
- [Understanding config maps](/docs/nodes/pods/nodes-pods-configmaps#nodes-pods-configmap-overview_builds-configmaps)
- [Providing sensitive data to pods](/docs/nodes/pods/nodes-pods-secrets#nodes-pods-secrets-about)
- [Understanding and creating service accounts](/docs/authentication/understanding-and-creating-service-accounts#service-accounts-overview)
