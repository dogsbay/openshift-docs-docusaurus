---
title: vSphere installation requirements for user-provisioned infrastructure
sidebar_position: 1
---

# vSphere installation requirements for user-provisioned infrastructure {#upi-vsphere-installation-reqs}

<a id="upi-vsphere-installation-reqs"></a>

Before you begin an installation on infrastructure that you provision, be sure that your vSphere environment meets the following installation requirements.

## VMware vSphere infrastructure requirements {#installation-vsphere-infrastructure_upi-vsphere-installation-reqs}

You must install an OpenShift Container Platform cluster on one of the following versions of a VMware vSphere instance that meets the requirements for the components that you use:

- Version 8.0 Update 1 or later, or VMware Cloud Foundation 5.0 or later
- VMware vSphere Foundation 9 or later, or VMware Cloud Foundation 9 or later

Both of these releases support Container Storage Interface (CSI) migration, which is enabled by default on OpenShift Container Platform 4.22.

:::note

Red Hat follows Broadcom’s End of Support dates for VMware products that OpenShift Container Platform runs on. After a VMware product version reaches End of Support, that version is no longer supported for use with OpenShift Container Platform.

:::

You can host the VMware vSphere infrastructure on-premise or on a [VMware Cloud Verified provider](https://cloud.vmware.com/providers) that meets the requirements outlined in the following tables:

**Version requirements for vSphere virtual environments**

| Virtual environment product | Required version |
| --- | --- |
| VMware virtual hardware | 15 or later |
| vSphere ESXi hosts | 8.0 Update 1 or later, or VMware vSphere Foundation 9 or later; VMware Cloud Foundation 5.0 or later, or VMware Cloud Foundation 9 or later |
| vCenter host | 8.0 Update 1 or later, or VMware vSphere Foundation 9 or later; VMware Cloud Foundation 5.0 or later, or VMware Cloud Foundation 9 or later |

:::warning

You must ensure that the time on your ESXi hosts is synchronized before you install OpenShift Container Platform. See [Editing the Time Configuration Settings of Your ESXi Host (Broadcom documentation)](https://techdocs.broadcom.com/us/en/vmware-cis/vsphere/vsphere/8-0/vcenter-and-host-management/host-configuration-host-management/synchronizing-clocks-on-the-vsphere-network-host-management/editing-time-configuration-for-a-host-host-management.html).

:::

**Minimum supported vSphere version for VMware components**

| Component | Minimum supported versions | Description |
| --- | --- | --- |
| Hypervisor | vSphere 8.0 Update 1 or later, or VMware Cloud Foundation 5.0 or later with virtual hardware version 15; VMware vSphere Foundation 9 or later, or VMware Cloud Foundation 9 or later | This hypervisor version is the minimum version that Red Hat Enterprise Linux CoreOS (RHCOS) supports. For more information about supported hardware on the latest version of Red Hat Enterprise Linux (RHEL) that is compatible with RHCOS, see [Hardware](https://catalog.redhat.com/hardware/search) on the Red Hat Customer Portal. |
| Networking (NSX) | vSphere 8.0 Update 1 or later, or VMware Cloud Foundation 5.0 or later; VMware vSphere Foundation 9 or later, or VMware Cloud Foundation 9 or later | Red Hat uses the Partner Certification process to verify NSX compatibility. |
| CPU micro-architecture | x86-64-v2 or higher | OpenShift Container Platform version 4.13 and later are based on the RHEL 9.2 host operating system, which raised the microarchitecture requirements to x86-64-v2. See [Architectures](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html-single/9.2_release_notes/index#architectures) in the RHEL documentation. |

:::warning

To ensure the best performance conditions for your cluster workloads that operate on Oracle(R) Cloud Infrastructure (OCI) and on the Oracle(R) Cloud VMware Solution (OCVS) service, ensure volume performance units (VPUs) for your block volume are sized for your workloads.

The following list provides some guidance in selecting the VPUs needed for specific performance needs:

- Test or proof of concept environment: 100 GB, and 20 to 30 VPUs.
- Base-production environment: 500 GB, and 60 VPUs.
- Heavy-use production environment: More than 500 GB, and 100 or more VPUs.

Consider allocating additional VPUs to give enough capacity for updates and scaling activities. See [Block Volume Performance Levels (Oracle documentation)](https://docs.oracle.com/en-us/iaas/Content/Block/Concepts/blockvolumeperformance.htm).

:::

:::note

The following additional VMware vSphere Foundation and VMware Cloud Foundation components are outside the scope of Red Hat support:

- Management: VCF Operations, VCF Automation, VCF Fleet Management, and VCF Identity Broker.
- Networking: VMware NSX Container Plugin (NCP).
- Migration: VMware HCX.

:::

## VMware vSphere CSI Driver Operator requirements {#vsphere-csi-driver-reqs_upi-vsphere-installation-reqs}

To successfully install and operate the vSphere CSI Driver Operator, verify that your environment meets the minimum VMware vSphere, vCenter, and virtual machine version requirements.

To install the vSphere Container Storage Interface (CSI) Driver Operator, the following requirements must be met:

- VMware vSphere version 8.0 Update 1 or later; or VMware vSphere Foundation (VVF) 9; or VMware Cloud Foundation (VCF) 5 or later
- vCenter version 8.0 Update 1 or later; or VVF 9; or VCF 5 or later
- Virtual machines of hardware version 15 or later
- No third-party vSphere CSI driver already installed in the cluster

If a third-party vSphere CSI driver is present in the cluster, OpenShift Container Platform does not overwrite it. The presence of a third-party vSphere CSI driver prevents OpenShift Container Platform from updating to OpenShift Container Platform 4.13 or later.

:::note

The VMware vSphere CSI Driver Operator is supported only on clusters deployed with `platform: vsphere` in the installation manifest.

:::

You can create a custom role for the Container Storage Interface (CSI) driver, the vSphere CSI Driver Operator, and the vSphere Problem Detector Operator. The custom role can include privilege sets that assign a minimum set of permissions to each vSphere object. This means that the CSI driver, the vSphere CSI Driver Operator, and the vSphere Problem Detector Operator can establish a basic interaction with these objects.

:::warning

Installing an OpenShift Container Platform cluster in a vCenter is tested against a full list of privileges as described in the "Required vCenter account privileges" section. By adhering to the full list of privileges, you can reduce the possibility of unexpected and unsupported behaviors that might occur when creating a custom role with a set of restricted privileges.

:::

To remove a third-party CSI driver, see "Removing a third-party vSphere CSI Driver".

**Additional resources**

- [Removing a third-party vSphere CSI Driver](/docs/storage/container_storage_interface/persistent-storage-csi-vsphere#persistent-storage-csi-vsphere-install-issues_persistent-storage-csi-vsphere)
- [Updating hardware on nodes running in vSphere](/docs/updating/updating_a_cluster/updating-hardware-on-nodes-running-on-vsphere#updating-hardware-on-nodes-running-on-vsphere)
- [Minimum permissions for the storage components](/docs/installing/installing_vsphere/upi/upi-vsphere-installation-reqs#installation-vsphere-minimum-permissions-storage_upi-vsphere-installation-reqs)

## Requirements for a cluster with user-provisioned infrastructure {#reqs-for-a-cluster-with-user-provisioned-infrastructure_upi-vsphere-installation-reqs}

For a cluster that contains user-provisioned infrastructure, you must deploy all of the required machines.

### vCenter requirements {#installation-vsphere-installer-infra-requirements_upi-vsphere-installation-reqs}

Before you install an OpenShift Container Platform cluster on your vCenter that uses infrastructure that you provided, you must prepare your environment.

#### Required vCenter account privileges {#installation-vsphere-installer-infra-requirements-account_upi-vsphere-installation-reqs}

To install an OpenShift Container Platform cluster in a vCenter, your vSphere account must include privileges for reading and creating the required resources. Using an account that has global administrative privileges is the simplest way to access all of the necessary permissions.

:::note

The following tables do not explicitly list the ESXi host object. In the vSphere hierarchy, ESXi hosts are child objects of the cluster. If you apply your custom role to the vSphere vCenter Cluster object with the "Propagate to children" setting enabled, the required privileges automatically propagate down to the ESXi hosts. You do not need to apply permissions directly to individual ESXi host objects.

:::

**Roles and privileges required for installation in vSphere API**

<table>
<thead>
<tr>
  <th>vSphere object for role</th>
  <th>When required</th>
  <th>Required privileges in vSphere API</th>
</tr>
</thead>
<tbody>
<tr>
  <td>vSphere vCenter</td>
  <td>Always</td>
  <td><ul><li><code>Cns.Searchable</code></li><li><code>InventoryService.Tagging.AttachTag</code></li><li><code>InventoryService.Tagging.CreateCategory</code></li><li><code>InventoryService.Tagging.CreateTag</code></li><li><code>InventoryService.Tagging.DeleteCategory</code></li><li><code>InventoryService.Tagging.DeleteTag</code></li><li><code>InventoryService.Tagging.EditCategory</code></li><li><code>InventoryService.Tagging.EditTag</code></li><li><code>Sessions.ValidateSession</code></li><li><code>StorageProfile.Update</code></li><li><code>StorageProfile.View</code></li></ul></td>
</tr>
<tr>
  <td>vSphere vCenter Cluster</td>
  <td>Always</td>
  <td><ul><li><code>Host.Config.Storage</code></li><li><code>Resource.AssignVMToPool</code></li><li><code>VApp.AssignResourcePool</code></li><li><code>VApp.Import</code></li><li><code>VirtualMachine.Config.AddNewDisk</code></li></ul></td>
</tr>
<tr>
  <td>vSphere vCenter Resource Pool</td>
  <td>For a provided existing resource pool</td>
  <td><ul><li><code>Resource.AssignVMToPool</code></li><li><code>VApp.AssignResourcePool</code></li><li><code>VApp.Import</code></li><li><code>VirtualMachine.Config.AddNewDisk</code></li></ul></td>
</tr>
<tr>
  <td>vSphere Datastore</td>
  <td>Always</td>
  <td><ul><li><code>Datastore.AllocateSpace</code></li><li><code>Datastore.Browse</code></li><li><code>Datastore.FileManagement</code></li><li><code>InventoryService.Tagging.ObjectAttachable</code></li></ul></td>
</tr>
<tr>
  <td>vSphere Port Group</td>
  <td>Always</td>
  <td><code>Network.Assign</code></td>
</tr>
<tr>
  <td>Virtual Machine Folder</td>
  <td>Always</td>
  <td><ul><li><code>InventoryService.Tagging.ObjectAttachable</code></li><li><code>Resource.AssignVMToPool</code></li><li><code>VApp.Import</code></li><li><code>VirtualMachine.Config.AddExistingDisk</code></li><li><code>VirtualMachine.Config.AddNewDisk</code></li><li><code>VirtualMachine.Config.AddRemoveDevice</code></li><li><code>VirtualMachine.Config.AdvancedConfig</code></li><li><code>VirtualMachine.Config.Annotation</code></li><li><code>VirtualMachine.Config.CPUCount</code></li><li><code>VirtualMachine.Config.DiskExtend</code></li><li><code>VirtualMachine.Config.DiskLease</code></li><li><code>VirtualMachine.Config.EditDevice</code></li><li><code>VirtualMachine.Config.Memory</code></li><li><code>VirtualMachine.Config.RemoveDisk</code></li><li><code>VirtualMachine.Config.Rename</code></li><li><code>Host.Config.Storage</code></li><li><code>VirtualMachine.Config.ResetGuestInfo</code></li><li><code>VirtualMachine.Config.Resource</code></li><li><code>VirtualMachine.Config.Settings</code></li><li><code>VirtualMachine.Config.UpgradeVirtualHardware</code></li><li><code>VirtualMachine.Interact.GuestControl</code></li><li><code>VirtualMachine.Interact.PowerOff</code></li><li><code>VirtualMachine.Interact.PowerOn</code></li><li><code>VirtualMachine.Interact.Reset</code></li><li><code>VirtualMachine.Inventory.Create</code></li><li><code>VirtualMachine.Inventory.CreateFromExisting</code></li><li><code>VirtualMachine.Inventory.Delete</code></li><li><code>VirtualMachine.Provisioning.Clone</code></li><li><code>VirtualMachine.Provisioning.MarkAsTemplate</code></li><li><code>VirtualMachine.Provisioning.DeployTemplate</code></li></ul></td>
</tr>
<tr>
  <td>vSphere vCenter data center</td>
  <td><code>VirtualMachine.Inventory.Create</code> and <code>VirtualMachine.Inventory.Delete</code> privileges are optional if your cluster does not use the Machine API. See the "Minimum permissions for the Machine API" table.</td>
  <td><ul><li><code>InventoryService.Tagging.ObjectAttachable</code></li><li><code>Resource.AssignVMToPool</code></li><li><code>VirtualMachine.Config.AddExistingDisk</code></li><li><code>VirtualMachine.Config.AddNewDisk</code></li><li><code>VirtualMachine.Config.AddRemoveDevice</code></li><li><code>VirtualMachine.Config.AdvancedConfig</code></li><li><code>VirtualMachine.Config.Annotation</code></li><li><code>VirtualMachine.Config.CPUCount</code></li><li><code>VirtualMachine.Config.DiskExtend</code></li><li><code>VirtualMachine.Config.DiskLease</code></li><li><code>VirtualMachine.Config.EditDevice</code></li><li><code>VirtualMachine.Config.Memory</code></li><li><code>VirtualMachine.Config.RemoveDisk</code></li><li><code>VirtualMachine.Config.Rename</code></li><li><code>VirtualMachine.Config.ResetGuestInfo</code></li><li><code>VirtualMachine.Config.Resource</code></li><li><code>VirtualMachine.Config.Settings</code></li><li><code>VirtualMachine.Config.UpgradeVirtualHardware</code></li><li><code>VirtualMachine.Interact.GuestControl</code></li><li><code>VirtualMachine.Interact.PowerOff</code></li><li><code>VirtualMachine.Interact.PowerOn</code></li><li><code>VirtualMachine.Interact.Reset</code></li><li><code>VirtualMachine.Inventory.Create</code></li><li><code>VirtualMachine.Inventory.CreateFromExisting</code></li><li><code>VirtualMachine.Inventory.Delete</code></li><li><code>VirtualMachine.Provisioning.Clone</code></li><li><code>VirtualMachine.Provisioning.DeployTemplate</code></li><li><code>VirtualMachine.Provisioning.MarkAsTemplate</code></li><li><code>Folder.Create</code></li><li><code>Folder.Delete</code></li></ul></td>
</tr>
</tbody>
</table>

**Roles and privileges required for installation in vCenter graphical user interface (GUI)**

<table>
<thead>
<tr>
  <th>vSphere object for role</th>
  <th>When required</th>
  <th>Required privileges in vCenter GUI</th>
</tr>
</thead>
<tbody>
<tr>
  <td>vSphere vCenter</td>
  <td>Always</td>
  <td><ul><li><code>Cns.Searchable</code></li><li><code>"vSphere Tagging"."Assign or Unassign vSphere Tag"</code></li><li><code>"vSphere Tagging"."Create vSphere Tag Category"</code></li><li><code>"vSphere Tagging"."Create vSphere Tag"</code></li><li><code>vSphere Tagging"."Delete vSphere Tag Category"</code></li><li><code>"vSphere Tagging"."Delete vSphere Tag"</code></li><li><code>"vSphere Tagging"."Edit vSphere Tag Category"</code></li><li><code>"vSphere Tagging"."Edit vSphere Tag"</code></li><li><code>Sessions."Validate session"</code></li><li><code>"VM storage policies"."Update VM storage policies"</code></li><li><code>"VM storage policies"."View VM storage policies"</code></li></ul></td>
</tr>
<tr>
  <td>vSphere vCenter Cluster</td>
  <td>Always</td>
  <td><ul><li><code>Host.Configuration."Storage partition configuration"</code></li><li><code>Resource."Assign virtual machine to resource pool"</code></li><li><code>VApp."Assign resource pool"</code></li><li><code>VApp.Import</code></li><li><code>"Virtual machine"."Change Configuration"."Add new disk"</code></li></ul></td>
</tr>
<tr>
  <td>vSphere vCenter Resource Pool</td>
  <td>If providing an existing resource pool</td>
  <td><ul><li><code>Host.Configuration."Storage partition configuration"</code></li><li><code>Resource."Assign virtual machine to resource pool"</code></li><li><code>VApp."Assign resource pool"</code></li><li><code>VApp.Import</code></li><li><code>"Virtual machine"."Change Configuration"."Add new disk"</code></li></ul></td>
</tr>
<tr>
  <td>vSphere Datastore</td>
  <td>Always</td>
  <td><ul><li><code>Datastore."Allocate space"</code></li><li><code>Datastore."Browse datastore"</code></li><li><code>Datastore."Low level file operations"</code></li><li><code>"vSphere Tagging"."Assign or Unassign vSphere Tag on Object"</code></li></ul></td>
</tr>
<tr>
  <td>vSphere Port Group</td>
  <td>Always</td>
  <td><code>Network."Assign network"</code></td>
</tr>
<tr>
  <td>Virtual Machine Folder</td>
  <td>Always</td>
  <td><ul><li><code>"vSphere Tagging"."Assign or Unassign vSphere Tag on Object"</code></li><li><code>Resource."Assign virtual machine to resource pool"</code></li><li><code>VApp.Import</code></li><li><code>"Virtual machine"."Change Configuration"."Add existing disk"</code></li><li><code>"Virtual machine"."Change Configuration"."Add new disk"</code></li><li><code>"Virtual machine"."Change Configuration"."Add or remove device"</code></li><li><code>"Virtual machine"."Change Configuration"."Advanced configuration"</code></li><li><code>"Virtual machine"."Change Configuration"."Set annotation"</code></li><li><code>"Virtual machine"."Change Configuration"."Change CPU count"</code></li><li><code>"Virtual machine"."Change Configuration"."Extend virtual disk"</code></li><li><code>"Virtual machine"."Change Configuration"."Acquire disk lease"</code></li><li><code>"Virtual machine"."Change Configuration"."Modify device settings"</code></li><li><code>"Virtual machine"."Change Configuration"."Change Memory"</code></li><li><code>"Virtual machine"."Change Configuration"."Remove disk"</code></li><li><code>"Virtual machine"."Change Configuration".Rename</code></li><li><code>"Virtual machine"."Change Configuration"."Reset guest information"</code></li><li><code>"Virtual machine"."Change Configuration"."Change resource"</code></li><li><code>"Virtual machine"."Change Configuration"."Change Settings"</code></li><li><code>"Virtual machine"."Change Configuration"."Upgrade virtual machine compatibility"</code></li><li><code>"Virtual machine".Interaction."Guest operating system management by VIX API"</code></li><li><code>"Virtual machine".Interaction."Power off"</code></li><li><code>"Virtual machine".Interaction."Power on"</code></li><li><code>"Virtual machine".Interaction.Reset</code></li><li><code>"Virtual machine"."Edit Inventory"."Create new"</code></li><li><code>"Virtual machine"."Edit Inventory"."Create from existing"</code></li><li><code>"Virtual machine"."Edit Inventory"."Remove"</code></li><li><code>"Virtual machine".Provisioning."Clone virtual machine"</code></li><li><code>"Virtual machine".Provisioning."Mark as template"</code></li><li><code>"Virtual machine".Provisioning."Deploy template"</code></li></ul></td>
</tr>
<tr>
  <td>vSphere vCenter data center</td>
  <td><code>VirtualMachine.Inventory.Create</code> and <code>VirtualMachine.Inventory.Delete</code> privileges are optional if your cluster does not use the Machine API.</td>
  <td><ul><li><code>"vSphere Tagging"."Assign or Unassign vSphere Tag on Object"</code></li><li><code>Resource."Assign virtual machine to resource pool"</code></li><li><code>VApp.Import</code></li><li><code>"Virtual machine"."Change Configuration"."Add existing disk"</code></li><li><code>"Virtual machine"."Change Configuration"."Add new disk"</code></li><li><code>"Virtual machine"."Change Configuration"."Add or remove device"</code></li><li><code>"Virtual machine"."Change Configuration"."Advanced configuration"</code></li><li><code>"Virtual machine"."Change Configuration"."Set annotation"</code></li><li><code>"Virtual machine"."Change Configuration"."Change CPU count"</code></li><li><code>"Virtual machine"."Change Configuration"."Extend virtual disk"</code></li><li><code>"Virtual machine"."Change Configuration"."Acquire disk lease"</code></li><li><code>"Virtual machine"."Change Configuration"."Modify device settings"</code></li><li><code>"Virtual machine"."Change Configuration"."Change Memory"</code></li><li><code>"Virtual machine"."Change Configuration"."Remove disk"</code></li><li><code>"Virtual machine"."Change Configuration".Rename</code></li><li><code>"Virtual machine"."Change Configuration"."Reset guest information"</code></li><li><code>"Virtual machine"."Change Configuration"."Change resource"</code></li><li><code>"Virtual machine"."Change Configuration"."Change Settings"</code></li><li><code>"Virtual machine"."Change Configuration"."Upgrade virtual machine compatibility"</code></li><li><code>"Virtual machine".Interaction."Guest operating system management by VIX API"</code></li><li><code>"Virtual machine".Interaction."Power off"</code></li><li><code>"Virtual machine".Interaction."Power on"</code></li><li><code>"Virtual machine".Interaction.Reset</code></li><li><code>"Virtual machine"."Edit Inventory"."Create new"</code></li><li><code>"Virtual machine"."Edit Inventory"."Create from existing"</code></li><li><code>"Virtual machine"."Edit Inventory"."Remove"</code></li><li><code>"Virtual machine".Provisioning."Clone virtual machine"</code></li><li><code>"Virtual machine".Provisioning."Deploy template"</code></li><li><code>"Virtual machine".Provisioning."Mark as template"</code></li><li><code>Folder."Create folder"</code></li><li><code>Folder."Delete folder"</code></li></ul></td>
</tr>
</tbody>
</table>

Additionally, the user requires some `ReadOnly` permissions, and some of the roles require permission to propagate the permissions to child objects. These settings vary depending on whether or not you install the cluster into an existing folder.

**Required permissions and propagation settings**

<table>
<thead>
<tr>
  <th>vSphere object</th>
  <th>When required</th>
  <th>Propagate to children</th>
  <th>Permissions required</th>
</tr>
</thead>
<tbody>
<tr>
  <td>vSphere vCenter</td>
  <td>Always</td>
  <td>False</td>
  <td>Listed required privileges</td>
</tr>
<tr>
  <td>vSphere vCenter data center</td>
  <td>Existing folder</td>
  <td>False</td>
  <td><code>ReadOnly</code> permission</td>
</tr>
<tr>
  <td>vSphere vCenter Cluster</td>
  <td>Always</td>
  <td>True</td>
  <td>Listed required privileges</td>
</tr>
<tr>
  <td>vSphere vCenter Datastore</td>
  <td>Always</td>
  <td>False</td>
  <td>Listed required privileges</td>
</tr>
<tr>
  <td>vSphere Switch</td>
  <td>Always</td>
  <td>False</td>
  <td><code>ReadOnly</code> permission</td>
</tr>
<tr>
  <td>vSphere Port Group</td>
  <td>Always</td>
  <td>False</td>
  <td>Listed required privileges</td>
</tr>
<tr>
  <td>vSphere vCenter Virtual Machine Folder</td>
  <td>Existing folder</td>
  <td>True</td>
  <td>Listed required privileges</td>
</tr>
<tr>
  <td>vSphere vCenter Resource Pool</td>
  <td>Existing resource pool</td>
  <td>True</td>
  <td>Listed required privileges</td>
</tr>
</tbody>
</table>

For more information about creating an account with only the required privileges, see [vSphere Permissions and User Management Tasks](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-5372F580-5C23-4E9C-8A4E-EF1B4DD9033E.html) in the vSphere documentation.

#### Minimum required vCenter account privileges {#installation-vsphere-installer-infra-minimum-requirements_upi-vsphere-installation-reqs}

After you create a custom role and assign privileges to the role, you can create permissions by selecting specific vSphere objects. You can then assign the custom role to a user or group for each object.

Before you create permissions or request for the creation of permissions for a vSphere object, decide what minimum permissions apply to the vSphere object. By doing this task, you can ensure a basic interaction exists between a vSphere object and OpenShift Container Platform architecture.

:::warning

If you create a custom role and you do not assign privileges to it, the vSphere Server by default assigns a `Read Only` role to the custom role. Note that for the cloud provider API, the custom role only needs to inherit the privileges of the `Read Only` role.

:::

Consider creating a custom role when an account with global administrative privileges does not meet your needs.

:::warning

Red Hat does not support configuring an account without including the required privileges. Red Hat tests OpenShift Container Platform cluster installations in vCenter against the full list of privileges described in the "Required vCenter account privileges" section. By adhering to the full list of privileges, you can reduce the possibility of unexpected behaviors that might occur when creating a custom role with a restricted set of privileges. You must retain the full set of privileges from the "Required vCenter account privileges" section after cluster installation. Reducing the account to only the permissions listed in the minimum permission tables in the "Minimum required vCenter account privileges" section after installation is not supported and can cause unexpected cluster behavior. The minimum permission tables are for reference only; they show which privileges apply to which OpenShift Container Platform components (such as storage or the Machine API) when you design or audit custom roles. The supported configuration is to assign the full set of privileges from the "Required vCenter account privileges" section at all times, both during and after installation.

:::

The following tables specify how the required vCenter account privileges provided earlier in this document are relevant to different aspects of OpenShift Container Platform architecture.

<a name="post-installation-vsphere-minimum-permissions_upi-vsphere-installation-reqs"></a>

**Minimum permissions for postinstallation management of components**

<table>
<thead>
<tr>
  <th>vSphere object for role</th>
  <th>When required</th>
  <th>Required privileges</th>
</tr>
</thead>
<tbody>
<tr>
  <td>vSphere vCenter</td>
  <td>Always</td>
  <td><ul><li><code>Cns.Searchable</code></li><li><code>InventoryService.Tagging.AttachTag</code></li><li><code>InventoryService.Tagging.CreateCategory</code></li><li><code>InventoryService.Tagging.CreateTag</code></li><li><code>InventoryService.Tagging.DeleteCategory</code></li><li><code>InventoryService.Tagging.DeleteTag</code></li><li><code>InventoryService.Tagging.EditCategory</code></li><li><code>InventoryService.Tagging.EditTag</code></li><li><code>Sessions.ValidateSession</code></li><li><code>StorageProfile.Update</code></li><li><code>StorageProfile.View</code></li></ul></td>
</tr>
<tr>
  <td>vSphere vCenter Cluster</td>
  <td>If you intend to create VMs in the cluster root</td>
  <td><ul><li><code>Host.Config.Storage</code></li><li><code>Resource.AssignVMToPool</code></li></ul></td>
</tr>
<tr>
  <td>vSphere vCenter Resource Pool</td>
  <td>If you included an existing resource pool in the <code>install-config.yaml</code> file</td>
  <td><code>Host.Config.Storage</code></td>
</tr>
<tr>
  <td>vSphere Datastore</td>
  <td>Always</td>
  <td><ul><li><code>Datastore.AllocateSpace</code></li><li><code>Datastore.Browse</code></li><li><code>Datastore.FileManagement</code></li><li><code>InventoryService.Tagging.ObjectAttachable</code></li></ul></td>
</tr>
<tr>
  <td>vSphere Port Group</td>
  <td>Always</td>
  <td><code>Network.Assign</code></td>
</tr>
<tr>
  <td>Virtual Machine Folder</td>
  <td>Always</td>
  <td><ul><li><code>VirtualMachine.Config.AddExistingDisk</code></li><li><code>VirtualMachine.Config.AddRemoveDevice</code></li><li><code>VirtualMachine.Config.AdvancedConfig</code></li><li><code>VirtualMachine.Config.Annotation</code></li><li><code>VirtualMachine.Config.CPUCount</code></li><li><code>VirtualMachine.Config.DiskExtend</code></li><li><code>VirtualMachine.Config.Memory</code></li><li><code>VirtualMachine.Config.Settings</code></li><li><code>VirtualMachine.Interact.PowerOff</code></li><li><code>VirtualMachine.Interact.PowerOn</code></li><li><code>VirtualMachine.Inventory.CreateFromExisting</code></li><li><code>VirtualMachine.Inventory.Delete</code></li><li><code>VirtualMachine.Provisioning.Clone</code></li><li><code>VirtualMachine.Provisioning.DeployTemplate</code></li></ul></td>
</tr>
<tr>
  <td>vSphere vCenter data center</td>
  <td><code>VirtualMachine.Inventory.Create</code> and <code>VirtualMachine.Inventory.Delete</code> privileges are optional if your cluster does not use the Machine API. If your cluster does use the Machine API and you want to set the minimum set of permissions for the API, see the "Minimum permissions for the Machine API" table.</td>
  <td><ul><li><code>Resource.AssignVMToPool</code></li><li><code>VirtualMachine.Config.AddExistingDisk</code></li><li><code>VirtualMachine.Config.AddRemoveDevice</code></li><li><code>VirtualMachine.Interact.PowerOff</code></li><li><code>VirtualMachine.Interact.PowerOn</code></li><li><code>VirtualMachine.Provisioning.DeployTemplate</code></li></ul></td>
</tr>
</tbody>
</table>

<a name="installation-vsphere-minimum-permissions-storage_upi-vsphere-installation-reqs"></a>

**Minimum permissions for the storage components**

<table>
<thead>
<tr>
  <th>vSphere object for role</th>
  <th>When required</th>
  <th>Required privileges</th>
</tr>
</thead>
<tbody>
<tr>
  <td>vSphere vCenter</td>
  <td>Always</td>
  <td><ul><li><code>Cns.Searchable</code></li><li><code>InventoryService.Tagging.CreateCategory</code></li><li><code>InventoryService.Tagging.CreateTag</code></li><li><code>InventoryService.Tagging.EditCategory</code></li><li><code>InventoryService.Tagging.EditTag</code></li><li><code>StorageProfile.Update</code></li><li><code>StorageProfile.View</code></li></ul></td>
</tr>
<tr>
  <td>vSphere vCenter Cluster</td>
  <td>If you intend to create VMs in the cluster root</td>
  <td><code>Host.Config.Storage</code></td>
</tr>
<tr>
  <td>vSphere vCenter Resource Pool</td>
  <td>If you included an existing resource pool in the <code>install-config.yaml</code> file</td>
  <td><code>Host.Config.Storage</code></td>
</tr>
<tr>
  <td>vSphere Datastore</td>
  <td>Always</td>
  <td><ul><li><code>Datastore.Browse</code></li><li><code>Datastore.FileManagement</code></li><li><code>InventoryService.Tagging.ObjectAttachable</code></li></ul></td>
</tr>
<tr>
  <td>vSphere Port Group</td>
  <td>Always</td>
  <td><code>Read Only</code></td>
</tr>
<tr>
  <td>Virtual Machine Folder</td>
  <td>Always</td>
  <td><ul><li><code>VirtualMachine.Config.AddExistingDisk</code></li><li><code>VirtualMachine.Config.AddRemoveDevice</code></li></ul></td>
</tr>
<tr>
  <td>vSphere vCenter data center</td>
  <td><code>VirtualMachine.Inventory.Create</code> and <code>VirtualMachine.Inventory.Delete</code> privileges are optional if your cluster does not use the Machine API. If your cluster does use the Machine API and you want to set the minimum set of permissions for the API, see the "Minimum permissions for the Machine API" table.</td>
  <td><ul><li><code>VirtualMachine.Config.AddExistingDisk</code></li><li><code>VirtualMachine.Config.AddRemoveDevice</code></li></ul></td>
</tr>
</tbody>
</table>

<a name="post-installation-vsphere-minimum-machine-api_upi-vsphere-installation-reqs"></a>

**Minimum permissions for the Machine API**

<table>
<thead>
<tr>
  <th>vSphere object for role</th>
  <th>When required</th>
  <th>Required privileges</th>
</tr>
</thead>
<tbody>
<tr>
  <td>vSphere vCenter</td>
  <td>Always</td>
  <td><ul><li><code>InventoryService.Tagging.AttachTag</code></li><li><code>InventoryService.Tagging.CreateCategory</code></li><li><code>InventoryService.Tagging.CreateTag</code></li><li><code>InventoryService.Tagging.DeleteCategory</code></li><li><code>InventoryService.Tagging.DeleteTag</code></li><li><code>InventoryService.Tagging.EditCategory</code></li><li><code>InventoryService.Tagging.EditTag</code></li><li><code>Sessions.ValidateSession</code></li><li><code>StorageProfile.Update</code></li><li><code>StorageProfile.View</code></li></ul></td>
</tr>
<tr>
  <td>vSphere vCenter Cluster</td>
  <td>If you intend to create VMs in the cluster root</td>
  <td><code>Resource.AssignVMToPool</code></td>
</tr>
<tr>
  <td>vSphere vCenter Resource Pool</td>
  <td>If you included an existing resource pool in the <code>install-config.yaml</code> file</td>
  <td><code>Read Only</code></td>
</tr>
<tr>
  <td>vSphere Datastore</td>
  <td>Always</td>
  <td><ul><li><code>Datastore.AllocateSpace</code></li><li><code>Datastore.Browse</code></li></ul></td>
</tr>
<tr>
  <td>vSphere Port Group</td>
  <td>Always</td>
  <td><code>Network.Assign</code></td>
</tr>
<tr>
  <td>Virtual Machine Folder</td>
  <td>Always</td>
  <td><ul><li><code>VirtualMachine.Config.AddRemoveDevice</code></li><li><code>VirtualMachine.Config.AdvancedConfig</code></li><li><code>VirtualMachine.Config.Annotation</code></li><li><code>VirtualMachine.Config.CPUCount</code></li><li><code>VirtualMachine.Config.DiskExtend</code></li><li><code>VirtualMachine.Config.Memory</code></li><li><code>VirtualMachine.Config.Settings</code></li><li><code>VirtualMachine.Interact.PowerOff</code></li><li><code>VirtualMachine.Interact.PowerOn</code></li><li><code>VirtualMachine.Inventory.CreateFromExisting</code></li><li><code>VirtualMachine.Inventory.Delete</code></li><li><code>VirtualMachine.Provisioning.Clone</code></li><li><code>VirtualMachine.Provisioning.DeployTemplate</code></li></ul></td>
</tr>
<tr>
  <td>vSphere vCenter data center</td>
  <td><code>VirtualMachine.Inventory.Create</code> and <code>VirtualMachine.Inventory.Delete</code> privileges are optional if your cluster does not use the Machine API.</td>
  <td><ul><li><code>Resource.AssignVMToPool</code></li><li><code>VirtualMachine.Interact.PowerOff</code></li><li><code>VirtualMachine.Interact.PowerOn</code></li><li><code>VirtualMachine.Provisioning.DeployTemplate</code></li></ul></td>
</tr>
</tbody>
</table>

#### Using OpenShift Container Platform with vMotion {#installation-vsphere-installer-infra-requirements-vmotion_upi-vsphere-installation-reqs}

If you intend on using vMotion in your vSphere environment, consider the following before installing an OpenShift Container Platform cluster.

- Using Storage vMotion can cause issues and is not supported.
- Using VMware compute vMotion to migrate the workloads for both OpenShift Container Platform compute machines and control plane machines is generally supported, where *generally* implies that you meet all VMware best practices for vMotion. To help ensure the uptime of your compute and control plane nodes, ensure that you follow the VMware best practices for vMotion, and use VMware anti-affinity rules to improve the availability of OpenShift Container Platform during maintenance or hardware issues.
  For more information about vMotion and anti-affinity rules, see the VMware vSphere documentation for  [vMotion networking requirements](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.vcenterhost.doc/GUID-3B41119A-1276-404B-8BFB-A32409052449.html) and [VM anti-affinity rules](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.resmgmt.doc/GUID-FBE46165-065C-48C2-B775-7ADA87FF9A20.html).
- If you are using VMware vSphere volumes in your pods, migrating a VM across datastores, either manually or through Storage vMotion, causes invalid references within OpenShift Container Platform persistent volume (PV) objects that can result in data loss.
- OpenShift Container Platform does not support selective migration of virtual machine disks (VMDKs) across datastores, using datastore clusters for VM provisioning or for dynamic or static provisioning of PVs, or using a datastore that is part of a datastore cluster for dynamic or static provisioning of PVs.
  :::warning

  You can specify the path of any datastore that exists in a datastore cluster. By default, Storage Distributed Resource Scheduler (SDRS), which uses Storage vMotion, is automatically enabled for a datastore cluster. Red Hat does not support Storage vMotion, so you must disable SDRS to avoid data loss issues for your OpenShift Container Platform cluster. If you must specify VMs across many datastores, use a `datastore` object to specify a failure domain in your cluster’s `install-config.yaml` configuration file. For more information, see "VMware vSphere region and zone enablement".

  :::

#### Cluster resources {#installation-vsphere-installer-infra-requirements-resources_upi-vsphere-installation-reqs}

When you deploy an OpenShift Container Platform cluster that uses infrastructure that you provided, you must create the following resources in your vCenter instance:

- 1 Folder
- 1 Tag category
- 1 Tag
- Virtual machines:
  - 1 template
  - 1 temporary bootstrap node
  - 3 control plane nodes
  - 3 compute machines

Although these resources use 856 GB of storage, the bootstrap node gets deleted during the cluster installation process. At a minimum, a standard cluster requires 800 GB of storage.

If you deploy more compute machines, the OpenShift Container Platform cluster will use more storage.

#### Cluster limits {#installation-vsphere-installer-infra-requirements-limits_upi-vsphere-installation-reqs}

Available resources vary between clusters. A limit exists for the number of possible clusters within vCenter, primarily by available storage space and any limitations on the number of required resources. Be sure to consider both limitations to the vCenter resources that the cluster creates and the resources that you require to deploy a cluster, such as IP addresses and networks.

#### Networking requirements {#installation-vsphere-installer-infra-requirements-networking_upi-vsphere-installation-reqs}

You can use Dynamic Host Configuration Protocol (DHCP) for the network and configure the DHCP server to set persistent IP addresses to machines in your cluster. In the DHCP lease, you must configure the DHCP to use the default gateway.

:::note

You do not need to use the DHCP for the network if you want to provision nodes with static IP addresses.

:::

If you specify nodes or groups of nodes on different VLANs for a cluster that you want to install on user-provisioned infrastructure, you must ensure that machines in your cluster meet the requirements outlined in the "Network connectivity requirements" section of the *Networking requirements for user-provisioned infrastructure* document.

If you are installing to a restricted environment, the VM in your restricted network must have access to vCenter so that it can provision and manage nodes, persistent volume claims (PVCs), and other resources.

:::note

Ensure that each OpenShift Container Platform node in the cluster has access to a Network Time Protocol (NTP) server that is discoverable by DHCP. Installation is possible without an NTP server. However, asynchronous server clocks can cause errors, which the NTP server prevents.

:::

Additionally, you must create the following networking resources before you install the OpenShift Container Platform cluster:

#### DNS records {#installation-vsphere-installer-infra-requirements-dns-records_upi-vsphere-installation-reqs}

You must create DNS records for two static IP addresses in the appropriate DNS server for the vCenter instance that hosts your OpenShift Container Platform cluster. In each record, `<cluster_name>` is the cluster name and `<base_domain>` is the cluster base domain that you specify when you install the cluster. A complete DNS record takes the form: `<component>.<cluster_name>.<base_domain>.`.

**Required DNS records**

<table>
<thead>
<tr>
  <th>Component</th>
  <th>Record</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>API VIP</td>
  <td><code>api.&lt;cluster_name&gt;.&lt;base_domain&gt;.</code></td>
  <td>This DNS A/AAAA or CNAME (Canonical Name) record must point to the load balancer for the control plane machines. This record must be resolvable by both clients external to the cluster and from all the nodes within the cluster.</td>
</tr>
<tr>
  <td>Ingress VIP</td>
  <td><code>*.apps.&lt;cluster_name&gt;.&lt;base_domain&gt;.</code></td>
  <td>A wildcard DNS A/AAAA or CNAME record that points to the load balancer that targets the machines that run the Ingress router pods, which are the worker nodes by default. This record must be resolvable by both clients external to the cluster and from all the nodes within the cluster.</td>
</tr>
</tbody>
</table>

**Additional resources**

- [Creating a compute machine set on vSphere](/docs/machine_management/creating_machinesets/creating-machineset-vsphere#creating-machineset-vsphere_creating-machineset-vsphere)

### Required machines for cluster installation {#installation-machine-requirements_upi-vsphere-installation-reqs}

You must specify the minimum required machines or hosts for your cluster so that your cluster remains stable if a node fails.

The smallest OpenShift Container Platform clusters require the following hosts:

:::warning

For a cluster that has user-provisioned infrastructure, you must deploy all of the required machines.

:::

**Minimum required hosts**

| Hosts | Description |
| --- | --- |
| One temporary bootstrap machine | The cluster requires the bootstrap machine to deploy the OpenShift Container Platform cluster on the three control plane machines. You can remove the bootstrap machine after you install the cluster. |
| Three control plane machines | The control plane machines run the Kubernetes and OpenShift Container Platform services that form the control plane. |
| At least two compute machines, which are also known as worker machines. | The workloads requested by OpenShift Container Platform users run on the compute machines. |

:::warning

To maintain high availability of your cluster, use separate physical hosts for these cluster machines.

:::

The bootstrap and control plane machines must use Red Hat Enterprise Linux CoreOS (RHCOS) as the operating system. However, the compute machines can use Red Hat Enterprise Linux CoreOS (RHCOS), Red Hat Enterprise Linux (RHEL) 8.6 and later.

RHCOS is based on Red Hat Enterprise Linux (RHEL) 9.8 and inherits all of its hardware certifications and requirements. See [Red Hat Enterprise Linux technology capabilities and limits](https://access.redhat.com/articles/rhel-limits).

### Minimum resource requirements for cluster installation {#installation-minimum-resource-requirements_upi-vsphere-installation-reqs}

To ensure that your OpenShift Container Platform cluster runs as expected, each cluster machine must meet minimum CPU, memory, and storage requirements.

**Minimum resource requirements**

<table>
<thead>
<tr>
  <th>Machine</th>
  <th>Operating system</th>
  <th>vCPU</th>
  <th>Virtual RAM</th>
  <th>Storage</th>
  <th>Input/Output Per Second (IOPS)</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Bootstrap</td>
  <td>RHCOS</td>
  <td>4</td>
  <td>16 GB</td>
  <td>100 GB</td>
  <td>300</td>
</tr>
<tr>
  <td>Control plane</td>
  <td>RHCOS</td>
  <td>4</td>
  <td>16 GB</td>
  <td>100 GB</td>
  <td>300</td>
</tr>
<tr>
  <td>Compute</td>
  <td>RHCOS</td>
  <td>2</td>
  <td>8 GB</td>
  <td>100 GB</td>
  <td>300</td>
</tr>
</tbody>
</table>

- OpenShift Container Platform and Kubernetes are sensitive to disk performance, and Red Hat recommends faster storage, particularly for etcd on the control plane nodes which require a 10 ms p99 fsync duration. On many cloud platforms, storage size and IOPS scale together, so you might need to provision more storage to get enough performance.
- As with all user-provisioned installations, if you choose to use RHEL compute machines in your cluster, you take responsibility for all operating system life cycle management and maintenance, including performing system updates, applying patches, and completing all other required tasks. OpenShift Container Platform 4.10 and later do not support RHEL 7 compute machines.

:::note

In OpenShift Container Platform version 4.22, RHCOS uses RHEL version 9.8, which updates the micro-architecture requirements. Each architecture requires the following minimum instruction set architectures (ISA):

- x86-64 architecture requires x86-64-v2 ISA
- ARM64 architecture requires ARMv8.0-A ISA
- ppc64le architecture requires IBM(R) Power9 ISA
- s390x architecture requires IBM(R) z14 ISA

For more information, see [Architectures](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html-single/9.8_release_notes/index#architectures) in the RHEL documentation.

:::

If an instance type for your platform meets the minimum requirements for cluster machines, it is supported to use in OpenShift Container Platform.

:::warning

Do not use memory ballooning in OpenShift Container Platform clusters. Memory ballooning can cause cluster-wide instabilities, service degradation, or other undefined behaviors.

- Control plane machines must have committed memory equal to or greater than the published minimum resource requirements for a cluster installation.
- Compute machines must have a minimum reservation equal to or greater than the published minimum resource requirements for a cluster installation.

These minimum CPU and memory requirements do not account for resources required by user workloads.

For more information, see the Red Hat Knowledgebase article [Memory Ballooning and OpenShift](https://access.redhat.com/articles/7074533).

:::

**Additional resources**

- [Optimizing storage](/docs/scalability_and_performance/optimization/optimizing-storage#optimizing-storage)

### Requirements for encrypting virtual machines {#installation-vsphere-encrypted-vms_upi-vsphere-installation-reqs}

You can encrypt your virtual machines prior to installing OpenShift Container Platform 4.22 by meeting the following requirements.

- You have configured a Standard key provider in vSphere. For more information, see [Adding a KMS to vCenter Server](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.vsan.doc/GUID-AC06B3C3-901F-402E-B25F-1EE7809D1264.html).
  :::warning

  The Native key provider in vCenter is not supported. For more information, see [vSphere Native Key Provider Overview](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-54B9FBA2-FDB1-400B-A6AE-81BF3AC9DF97.html).

  :::
- You have enabled host encryption mode on all of the ESXi hosts that are hosting the cluster. For more information, see [Enabling host encryption mode](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-A9E1F016-51B3-472F-B8DE-803F6BDB70BC.html).
- You have a vSphere account which has all cryptographic privileges enabled. For more information, see [Cryptographic Operations Privileges](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-660CCB35-847F-46B3-81CA-10DDDB9D7AA9.html).

When you deploy the OVF template in the section titled "Installing RHCOS and starting the OpenShift Container Platform bootstrap process", select the option to "Encrypt this virtual machine" when you are selecting storage for the OVF template. After completing cluster installation, create a storage class that uses the encryption storage policy you used to encrypt the virtual machines.

**Additional resources**

- [Creating an encrypted storage class](/docs/storage/container_storage_interface/persistent-storage-csi-vsphere#vsphere-pv-encryption)

### Certificate signing requests management {#csr-management_upi-vsphere-installation-reqs}

On user-provisioned infrastructure, you must implement a mechanism for approving cluster certificate signing requests (CSRs) after installation when your cluster has limited access to automatic machine management.

The `kube-controller-manager` only approves the kubelet client CSRs. The `machine-approver` cannot guarantee the validity of a serving certificate that kubelet credentials request because it cannot confirm that the correct machine issued the request. You must find and implement a method of verifying the validity of the kubelet serving certificate requests and approving them.

### Networking requirements for user-provisioned infrastructure {#installation-network-user-infra_upi-vsphere-installation-reqs}

You must configure networking for all the Red Hat Enterprise Linux CoreOS (RHCOS) machines in `initramfs` during boot, so that they can fetch their Ignition config files.

:::warning

Ensure you enable the `disk.EnableUUID` parameter on all virtual machines in your cluster.

:::

During the initial boot, the machines require an IP address configuration that is set either through a DHCP server or statically by providing the required boot options. After a network connection is established, the machines download their Ignition config files from an HTTP or HTTPS server. The Ignition config files are then used to set the exact state of each machine. The Machine Config Operator completes more changes to the machines, such as the application of new certificates or keys, after installation.

:::note

- Consider using a DHCP server for long-term management of the cluster machines. Ensure that the DHCP server is configured to provide persistent IP addresses, DNS server information, and hostnames to the cluster machines.
- If a DHCP service is not available for your user-provisioned infrastructure, you can instead provide the IP networking configuration and the address of the DNS server to the nodes at RHCOS install time. These can be passed as boot arguments if you are installing from an ISO image. See the *Installing RHCOS and starting the OpenShift Container Platform bootstrap process* section for more information about static IP provisioning and advanced networking options.

:::

The Kubernetes API server must be able to resolve the node names of the cluster machines. If the API servers and worker nodes are in different zones, you can configure a default DNS search zone to allow the API server to resolve the node names. Another supported approach is to always refer to hosts by their fully-qualified domain names in both the node objects and all DNS requests.

#### Setting the cluster node hostnames through DHCP {#installation-host-names-dhcp-user-infra_upi-vsphere-installation-reqs}

On Red Hat Enterprise Linux CoreOS (RHCOS) machines, the hostname is set through NetworkManager. By default, the machines obtain their hostname through DHCP. If the hostname is not provided by DHCP, set statically through kernel arguments, or another method, it is obtained through a reverse DNS lookup. Reverse DNS lookup occurs after the network has been initialized on a node and can take time to resolve. Other system services can start before this and detect the hostname as `localhost` or similar. You can avoid this by using DHCP to provide the hostname for each cluster node.

Additionally, setting the hostnames through DHCP can bypass any manual DNS record name configuration errors in environments that have a DNS split-horizon implementation.

#### Network connectivity requirements {#installation-network-connectivity-user-infra_upi-vsphere-installation-reqs}

You must configure the network connectivity between machines to allow OpenShift Container Platform cluster components to communicate. Each machine must be able to resolve the hostnames of all other machines in the cluster.

This section provides details about the ports that are required.

:::warning

In connected OpenShift Container Platform environments, all nodes are required to have internet access to pull images for platform containers and provide telemetry data to Red Hat.

:::

**Ports used for all-machine to all-machine communications**

<table>
<thead>
<tr>
  <th>Protocol</th>
  <th>Port</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>ICMP</td>
  <td>N/A</td>
  <td>Network reachability tests</td>
</tr>
<tr>
  <td rowspan="4">TCP</td>
  <td><code>1936</code></td>
  <td>Metrics</td>
</tr>
<tr>
  <td><code>9000</code>-<code>9999</code></td>
  <td>Host level services, including the node exporter on ports <code>9100</code>-<code>9101</code> and the Cluster Version Operator on port <code>9099</code>.</td>
</tr>
<tr>
  <td><code>10250</code>-<code>10259</code></td>
  <td>The default ports that Kubernetes reserves</td>
</tr>
<tr>
  <td><code>22623</code></td>
  <td>The port handles traffic from the Machine Config Server and directs the traffic to the control plane machines.</td>
</tr>
<tr>
  <td rowspan="5">UDP</td>
  <td><code>6081</code></td>
  <td>Geneve</td>
</tr>
<tr>
  <td><code>9000</code>-<code>9999</code></td>
  <td>Host level services, including the node exporter on ports <code>9100</code>-<code>9101</code>.</td>
</tr>
<tr>
  <td><code>500</code></td>
  <td>IPsec IKE packets</td>
</tr>
<tr>
  <td><code>4500</code></td>
  <td>IPsec NAT-T packets</td>
</tr>
<tr>
  <td><code>123</code></td>
  <td>Network Time Protocol (NTP) on UDP port <code>123</code>. If an external NTP time server is configured, you must open UDP port <code>123</code>.</td>
</tr>
<tr>
  <td>TCP/UDP</td>
  <td><code>30000</code>-<code>32767</code></td>
  <td>Kubernetes node port</td>
</tr>
<tr>
  <td>ESP</td>
  <td>N/A</td>
  <td>IPsec Encapsulating Security Payload (ESP)</td>
</tr>
</tbody>
</table>

**Ports used for all-machine to control plane communications**

<table>
<thead>
<tr>
  <th>Protocol</th>
  <th>Port</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>TCP</td>
  <td><code>6443</code></td>
  <td>Kubernetes API</td>
</tr>
</tbody>
</table>

**Ports used for control plane machine to control plane machine communications**

<table>
<thead>
<tr>
  <th>Protocol</th>
  <th>Port</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>TCP</td>
  <td><code>2379</code>-<code>2380</code></td>
  <td>etcd server and peer ports</td>
</tr>
</tbody>
</table>

#### NTP configuration for user-provisioned infrastructure {#_ntp_configuration_for_user-provisioned_infrastructure}

OpenShift Container Platform clusters are configured to use a public Network Time Protocol (NTP) server by default. If you want to use a local enterprise NTP server, or if your cluster is being deployed in a disconnected network, you can configure the cluster to use a specific time server. For more information, see the documentation for *Configuring chrony time service*.

If a DHCP server provides NTP server information, the chrony time service on the Red Hat Enterprise Linux CoreOS (RHCOS) machines read the information and can sync the clock with the NTP servers.

**Additional resources**

- [Configuring chrony time service](/docs/installing/install_config/installing-customizing#installation-special-config-chrony_installing-customizing)

### User-provisioned DNS requirements {#installation-dns-user-infra_upi-vsphere-installation-reqs}

In OpenShift Container Platform deployments, you must ensure that cluster components meet certain DNS name resolution criteria for internal communication, certificate validation, and automated node discovery purposes.

The following is a list of required cluster components:

- The Kubernetes API
- The OpenShift Container Platform application wildcard
- The bootstrap and control plane machines
- The compute machines

Reverse DNS resolution is also required for the Kubernetes API, the bootstrap machine, the control plane machines, and the compute machines.

DNS A/AAAA or CNAME records are used for name resolution and PTR records are used for reverse name resolution. The reverse records are important because Red Hat Enterprise Linux CoreOS (RHCOS) uses the reverse records to set the hostnames for all the nodes, unless the hostnames are provided by DHCP. Additionally, the reverse records are used to generate the certificate signing requests (CSR) that OpenShift Container Platform needs to operate.

:::note

It is recommended to use a DHCP server to provide the hostnames to each cluster node. See the *DHCP recommendations for user-provisioned infrastructure* section for more information.

:::

The following DNS records are required for a user-provisioned OpenShift Container Platform cluster and they must be in place before installation. In each record, `<cluster_name>` is the cluster name and `<base_domain>` is the base domain that you specify in the `install-config.yaml` file. A complete DNS record takes the form: `<component>.<cluster_name>.<base_domain>.`.

**Required DNS records**

<table>
<thead>
<tr>
  <th>Component</th>
  <th>Record</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td rowspan="2">Kubernetes API</td>
  <td><code>api.&lt;cluster_name&gt;.&lt;base_domain&gt;.</code></td>
  <td>A DNS A/AAAA or CNAME record, and a DNS PTR record, to identify the API load balancer. These records must be resolvable by both clients external to the cluster and from all the nodes within the cluster.</td>
</tr>
<tr>
  <td><code>api-int.&lt;cluster_name&gt;.&lt;base_domain&gt;.</code></td>
  <td>A DNS A/AAAA or CNAME record, and a DNS PTR record, to internally identify the API load balancer. These records must be resolvable from all the nodes within the cluster.<div class="alert alert--warning"><strong>Important</strong><p>The API server must be able to resolve the worker nodes by the hostnames that are recorded in Kubernetes. If the API server cannot resolve the node names, then proxied API calls can fail, and you cannot retrieve logs from pods.</p></div></td>
</tr>
<tr>
  <td>Routes</td>
  <td><code>*.apps.&lt;cluster_name&gt;.&lt;base_domain&gt;.</code></td>
  <td>A wildcard DNS A/AAAA or CNAME record that refers to the application ingress load balancer. The application ingress load balancer targets the machines that run the Ingress Controller pods.   The Ingress Controller pods run on the compute machines by default.  These records must be resolvable by both clients external to the cluster and from all the nodes within the cluster.<br><br>For example, <code>console-openshift-console.apps.&lt;cluster_name&gt;.&lt;base_domain&gt;</code> is used as a wildcard route to the OpenShift Container Platform console.</td>
</tr>
<tr>
  <td>Bootstrap machine</td>
  <td><code>bootstrap.&lt;cluster_name&gt;.&lt;base_domain&gt;.</code></td>
  <td>A DNS A/AAAA or CNAME record, and a DNS PTR record, to identify the bootstrap machine. These records must be resolvable by the nodes within the cluster.</td>
</tr>
<tr>
  <td>Control plane machines</td>
  <td><code>&lt;control_plane&gt;&lt;n&gt;.&lt;cluster_name&gt;.&lt;base_domain&gt;.</code></td>
  <td>DNS A/AAAA or CNAME records and DNS PTR records to identify each machine for the control plane nodes. These records must be resolvable by the nodes within the cluster.</td>
</tr>
<tr>
  <td>Compute machines</td>
  <td><code>&lt;compute&gt;&lt;n&gt;.&lt;cluster_name&gt;.&lt;base_domain&gt;.</code></td>
  <td>DNS A/AAAA or CNAME records and DNS PTR records to identify each machine for the worker nodes. These records must be resolvable by the nodes within the cluster.</td>
</tr>
</tbody>
</table>

:::note

In OpenShift Container Platform 4.4 and later, you do not need to specify etcd host and SRV records in your DNS configuration.

:::

:::tip

You can use the `dig` command to verify name and reverse name resolution. See the section on *Validating DNS resolution for user-provisioned infrastructure* for detailed validation steps.

:::

#### Example DNS configuration for user-provisioned clusters {#installation-dns-user-infra-example_upi-vsphere-installation-reqs}

Reference the example DNS configurations to understand how A and PTR record configuration samples meet the DNS requirements for deploying OpenShift Container Platform on user-provisioned infrastructure.

The DNS configuration examples provided here are for reference only and are not meant to provide advice for choosing one DNS solution over another.

In the examples, the cluster name is `ocp4` and the base domain is `example.com`.

The following example is a BIND zone file that shows sample DNS A records for name resolution in a user-provisioned cluster.

:::note

In the example, the same load balancer is used for the Kubernetes API and application ingress traffic. In production scenarios, you can deploy the API and application ingress load balancers separately so that you can scale the load balancer infrastructure for each in isolation.

:::

```text
$TTL 1W
@	IN	SOA	ns1.example.com.	root (
			2019070700	; serial
			3H		; refresh (3 hours)
			30M		; retry (30 minutes)
			2W		; expiry (2 weeks)
			1W )		; minimum (1 week)
	IN	NS	ns1.example.com.
	IN	MX 10	smtp.example.com.
;
;
ns1.example.com.		IN	A	192.168.1.5
smtp.example.com.		IN	A	192.168.1.5
;
helper.example.com.		IN	A	192.168.1.5
helper.ocp4.example.com.	IN	A	192.168.1.5
;
api.ocp4.example.com.		IN	A	192.168.1.5
api-int.ocp4.example.com.	IN	A	192.168.1.5
;
*.apps.ocp4.example.com.	IN	A	192.168.1.5
;
bootstrap.ocp4.example.com.	IN	A	192.168.1.96
;
control-plane0.ocp4.example.com.	IN	A	192.168.1.97
control-plane1.ocp4.example.com.	IN	A	192.168.1.98
;
control-plane2.ocp4.example.com.	IN	A	192.168.1.99
;
compute0.ocp4.example.com.	IN	A	192.168.1.11
compute1.ocp4.example.com.	IN	A	192.168.1.7
;
;EOF
```

where:

`api.ocp4.example.com.`
: Provides name resolution for the Kubernetes API. The record refers to the IP address of the API load balancer.

`api-int.ocp4.example.com.`
: Provides name resolution for the Kubernetes API. The record refers to the IP address of the API load balancer and is used for internal cluster communications.

`*.apps.ocp4.example.com.`
: Provides name resolution for the wildcard routes. The record refers to the IP address of the application ingress load balancer. The application ingress load balancer targets the machines that run the Ingress Controller pods.

`bootstrap.ocp4.example.com`
: Provides name resolution for the bootstrap machine.

`control-plane0.ocp4.example.com`
: Provides name resolution for the control plane machines.

`compute0.ocp4.example.com.`
: Provides name resolution for the compute machines.

The following example BIND zone file shows sample PTR records for reverse name resolution in a user-provisioned cluster:

```text
$TTL 1W
@	IN	SOA	ns1.example.com.	root (
			2019070700	; serial
			3H		; refresh (3 hours)
			30M		; retry (30 minutes)
			2W		; expiry (2 weeks)
			1W )		; minimum (1 week)
	IN	NS	ns1.example.com.
;
5.1.168.192.in-addr.arpa.	IN	PTR	api.ocp4.example.com.
5.1.168.192.in-addr.arpa.	IN	PTR	api-int.ocp4.example.com.
;
96.1.168.192.in-addr.arpa.	IN	PTR	bootstrap.ocp4.example.com.
;
97.1.168.192.in-addr.arpa.	IN	PTR	control-plane0.ocp4.example.com.
98.1.168.192.in-addr.arpa.	IN	PTR	control-plane1.ocp4.example.com.
;
99.1.168.192.in-addr.arpa.	IN	PTR	control-plane2.ocp4.example.com.
;
11.1.168.192.in-addr.arpa.	IN	PTR	compute0.ocp4.example.com.
7.1.168.192.in-addr.arpa.	IN	PTR	compute1.ocp4.example.com.
;
;EOF
```

where:

`api.ocp4.example.com.`
: Provides reverse DNS resolution for the Kubernetes API. The PTR record refers to the record name of the API load balancer.

`api-int.ocp4.example.com.`
: Provides reverse DNS resolution for the Kubernetes API. The PTR record refers to the record name of the API load balancer and is used for internal cluster communications.

`bootstrap.ocp4.example.com.`
: Provides reverse DNS resolution for the bootstrap machine.

`control-plane0.ocp4.example.com.`
: Provides rebootstrap.ocp4.example.com.verse DNS resolution for the control plane machines.

`compute0.ocp4.example.com.`
: Provides reverse DNS resolution for the compute machines.

:::note

A PTR record is not required for the OpenShift Container Platform application wildcard.

:::

### Load balancing requirements for user-provisioned infrastructure {#installation-load-balancing-user-infra_upi-vsphere-installation-reqs}

Before you install OpenShift Container Platform, you must provision the API and application Ingress load balancing infrastructure. In production scenarios, you can deploy the API and application Ingress load balancers separately so that you can scale the load balancer infrastructure for each in isolation.

:::note

If you want to deploy the API and application Ingress load balancers with a Red Hat Enterprise Linux (RHEL) instance, you must purchase the RHEL subscription separately.

:::

The load balancing infrastructure must meet the following requirements:

- API load balancer: Provides a common endpoint for users, both human and machine, to interact with and configure the platform. Configure the following conditions:
  - Layer 4 load balancing only. This can be referred to as Raw TCP or SSL Passthrough mode.
  - A stateless load balancing algorithm. The options vary based on the load balancer implementation.

  :::warning

  Do not configure session persistence for an API load balancer. Configuring session persistence for a Kubernetes API server might cause performance issues from excess application traffic for your OpenShift Container Platform cluster and the Kubernetes API that runs inside the cluster.

  :::

Configure the following ports on both the front and back of the API load balancers:

| Port | Back-end machines (pool members) | Internal | External | Description |
| --- | --- | --- | --- | --- |
| `6443` | Bootstrap and control plane. You remove the bootstrap machine from the load balancer after the bootstrap machine initializes the cluster control plane. You must configure the `/readyz` endpoint for the API server health check probe. | X | X | Kubernetes API server |
| `22623` | Bootstrap and control plane. You remove the bootstrap machine from the load balancer after the bootstrap machine initializes the cluster control plane. | X |  | Machine config server |

:::note

The load balancer must be configured to take a maximum of 30 seconds from the time the API server turns off the `/readyz` endpoint to the removal of the API server instance from the pool. Within the time frame after `/readyz` returns an error or becomes healthy, the endpoint must have been removed or added. Probing every 5 or 10 seconds, with two successful requests to become healthy and three to become unhealthy, are well-tested values.

:::

- Application Ingress load balancer: Provides an ingress point for application traffic flowing in from outside the cluster. A working configuration for the Ingress router is required for an OpenShift Container Platform cluster. Configure the following conditions:
  - Layer 4 load balancing only. This can be referred to as Raw TCP or SSL Passthrough mode.
  - A connection-based or session-based persistence is recommended, based on the options available and types of applications that will be hosted on the platform.

  :::tip

  If the true IP address of the client can be seen by the application Ingress load balancer, enabling source IP-based session persistence can improve performance for applications that use end-to-end TLS encryption.

  :::

Configure the following ports on both the front and back of the load balancers:

**Application Ingress load balancer**

| Port | Back-end machines (pool members) | Internal | External | Description |
| --- | --- | --- | --- | --- |
| `443` | The machines that run the Ingress Controller pods, compute, or worker, by default. | X | X | HTTPS traffic |
| `80` | The machines that run the Ingress Controller pods, compute, or worker, by default. | X | X | HTTP traffic |

:::note

If you are deploying a three-node cluster with zero compute nodes, the Ingress Controller pods run on the control plane nodes. In three-node cluster deployments, you must configure your application Ingress load balancer to route HTTP and HTTPS traffic to the control plane nodes.

:::

#### Example load balancer configuration for user-provisioned clusters {#installation-load-balancing-user-infra-example_upi-vsphere-installation-reqs}

Reference the example API and application Ingress load balancer configuration so that you can understand how to meet the load balancing requirements for user-provisioned clusters.

The sample is an `/etc/haproxy/haproxy.cfg` configuration for an HAProxy load balancer. The example is not meant to provide advice for choosing one load balancing solution over another.

:::tip

If you are using HAProxy as a load balancer, you can check that the `haproxy` process is listening on ports `6443`, `22623`, `443`, and `80` by running `netstat -nltupe` on the HAProxy node.

:::

In the example, the same load balancer is used for the Kubernetes API and application ingress traffic. In production scenarios, you can deploy the API and application ingress load balancers separately so that you can scale the load balancer infrastructure for each in isolation.

:::note

If you are using HAProxy as a load balancer and SELinux is set to `enforcing`, you must ensure that the HAProxy service can bind to the configured TCP port by running `setsebool -P haproxy_connect_any=1`.

:::

```text title="Sample API and application Ingress load balancer configuration"
global
  log         127.0.0.1 local2
  pidfile     /var/run/haproxy.pid
  maxconn     4000
  daemon
defaults
  mode                    http
  log                     global
  option                  dontlognull
  option http-server-close
  option                  redispatch
  retries                 3
  timeout http-request    10s
  timeout queue           1m
  timeout connect         10s
  timeout client          1m
  timeout server          1m
  timeout http-keep-alive 10s
  timeout check           10s
  maxconn                 3000
listen api-server-6443
  bind *:6443
  mode tcp
  option  httpchk GET /readyz HTTP/1.0
  option  log-health-checks
  balance roundrobin
  server bootstrap bootstrap.ocp4.example.com:6443 verify none check check-ssl inter 10s fall 2 rise 3 backup
  server master0 master0.ocp4.example.com:6443 weight 1 verify none check check-ssl inter 10s fall 2 rise 3
  server master1 master1.ocp4.example.com:6443 weight 1 verify none check check-ssl inter 10s fall 2 rise 3
  server master2 master2.ocp4.example.com:6443 weight 1 verify none check check-ssl inter 10s fall 2 rise 3
listen machine-config-server-22623
  bind *:22623
  mode tcp
  server bootstrap bootstrap.ocp4.example.com:22623 check inter 1s backup
  server master0 master0.ocp4.example.com:22623 check inter 1s
  server master1 master1.ocp4.example.com:22623 check inter 1s
  server master2 master2.ocp4.example.com:22623 check inter 1s
listen ingress-router-443
  bind *:443
  mode tcp
  balance source
  server compute0 compute0.ocp4.example.com:443 check inter 1s
  server compute1 compute1.ocp4.example.com:443 check inter 1s
listen ingress-router-80
  bind *:80
  mode tcp
  balance source
  server compute0 compute0.ocp4.example.com:80 check inter 1s
  server compute1 compute1.ocp4.example.com:80 check inter 1s
```

where:

`listen api-server-6443`
: Port `6443` handles the Kubernetes API traffic and points to the control plane machines. You must configure health checks on this port to ensure that the API server is available before routing traffic.

`server bootstrap bootstrap.ocp4.example.com`
: The bootstrap entries must be in place before the OpenShift Container Platform cluster installation and they must be removed after the bootstrap process is complete.

`listen machine-config-server`
: Port `22623` handles the machine config server traffic and points to the control plane machines.

`listen ingress-router-443`
: Port `443` handles the HTTPS traffic and points to the machines that run the Ingress Controller pods. The Ingress Controller pods run on the compute machines by default.

`listen ingress-router-80`
: Port `80` handles the HTTP traffic and points to the machines that run the Ingress Controller pods. The Ingress Controller pods run on the compute machines by default.

  :::note

  If you are deploying a compact three-node cluster with zero compute nodes, the Ingress Controller pods run on the control plane nodes. In three-node cluster deployments, you must configure your application Ingress load balancer to route HTTP and HTTPS traffic to the control plane nodes.

  :::
