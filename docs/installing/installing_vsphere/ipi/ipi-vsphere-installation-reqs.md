---
title: vSphere installation requirements
sidebar_position: 1
---

# vSphere installation requirements {#ipi-vsphere-installation-reqs}

<a id="ipi-vsphere-installation-reqs"></a>

Before you begin an installation using installer-provisioned infrastructure, be sure that your vSphere environment meets the following installation requirements.

## VMware vSphere infrastructure requirements {#installation-vsphere-infrastructure_ipi-vsphere-installation-reqs}

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

## Network connectivity requirements {#installation-vsphere-installer-network-requirements_ipi-vsphere-installation-reqs}

You must configure the network connectivity between machines to allow OpenShift Container Platform cluster components to communicate.

Review the following details about the required network ports.

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
  <td>VRRP</td>
  <td>N/A</td>
  <td>Required for keepalived</td>
</tr>
<tr>
  <td>ICMP</td>
  <td>N/A</td>
  <td>Network reachability tests</td>
</tr>
<tr>
  <td rowspan="3">TCP</td>
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
  <td rowspan="4">UDP</td>
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

## VMware vSphere CSI Driver Operator requirements {#vsphere-csi-driver-reqs_ipi-vsphere-installation-reqs}

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

- [Removing a third-party vSphere CSI Driver Operator](/docs/storage/container_storage_interface/persistent-storage-csi-vsphere#persistent-storage-csi-vsphere-install-issues_persistent-storage-csi-vsphere)
- [Updating hardware on nodes running on vSphere](/docs/updating/updating_a_cluster/updating-hardware-on-nodes-running-on-vsphere#updating-hardware-on-nodes-running-on-vsphere)
- [Minimum permissions for the storage components](/docs/installing/installing_vsphere/ipi/ipi-vsphere-installation-reqs#installation-vsphere-minimum-permissions-storage_ipi-vsphere-installation-reqs)

## vCenter requirements {#installation-vsphere-installer-infra-requirements_ipi-vsphere-installation-reqs}

Before you install an OpenShift Container Platform cluster on your vCenter that uses infrastructure that the installation program provisions, you must prepare your environment.

### Required vCenter account privileges {#installation-vsphere-installer-infra-requirements-account_ipi-vsphere-installation-reqs}

To install an OpenShift Container Platform cluster in a vCenter, the installation program requires access to an account with privileges to read and create the required resources. Using an account that has global administrative privileges is the simplest way to access all of the necessary permissions.

If you cannot use an account with global administrative privileges, you must create roles to grant the privileges necessary for OpenShift Container Platform cluster installation. Most of the privileges are always required. Some privileges are required only if you plan for the installation program to provision a folder to contain the OpenShift Container Platform cluster on your vCenter instance, which is the default behavior. You must create or change vSphere roles for the specified objects to grant the required privileges.

The installation program requires an additional role to create a vSphere virtual machine folder.

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
  <td>The installation program creates the virtual machine folder.</td>
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
  <td>The installation program creates the virtual machine folder.</td>
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
  <td rowspan="2">vSphere vCenter data center</td>
  <td>Existing folder</td>
  <td>False</td>
  <td><code>ReadOnly</code> permission</td>
</tr>
<tr>
  <td>Installation program creates the folder</td>
  <td>True</td>
  <td>Listed required privileges</td>
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

### Minimum required vCenter account privileges {#installation-vsphere-installer-infra-minimum-requirements_ipi-vsphere-installation-reqs}

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

<a name="installation-vsphere-minimum-permissions-ipi_ipi-vsphere-installation-reqs"></a>

**Minimum permissions on installer-provisioned infrastructure**

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
  <td><ul><li><code>Host.Config.Storage</code></li><li><code>Resource.AssignVMToPool</code></li><li><code>VApp.AssignResourcePool</code></li><li><code>VApp.Import</code></li><li><code>VirtualMachine.Config.AddNewDisk</code></li></ul></td>
</tr>
<tr>
  <td>vSphere vCenter Resource Pool</td>
  <td>If you included an existing resource pool in the <code>install-config.yaml</code> file</td>
  <td><ul><li><code>Host.Config.Storage</code></li><li><code>Resource.AssignVMToPool</code></li><li><code>VApp.AssignResourcePool</code></li><li><code>VApp.Import</code>minimum`</li></ul></td>
</tr>
<tr>
  <td>vSphere Datastore</td>
  <td>If you referenced a datastore in the <code>install-config.yaml</code> file</td>
  <td><ul><li><code>Datastore.Browse</code></li><li><code>Datastore.FileManagement</code></li><li><code>InventoryService.Tagging.ObjectAttachable</code></li></ul></td>
</tr>
<tr>
  <td>vSphere Port Group</td>
  <td>Always</td>
  <td><code>Network.Assign</code></td>
</tr>
<tr>
  <td>Virtual Machine Folder</td>
  <td>Always</td>
  <td><ul><li><code>InventoryService.Tagging.ObjectAttachable</code></li><li><code>Resource.AssignVMToPool</code></li><li><code>VApp.Import</code></li><li><code>VirtualMachine.Config.AddExistingDisk</code></li><li><code>VirtualMachine.Config.AddNewDisk</code></li><li><code>VirtualMachine.Config.AddRemoveDevice</code></li><li><code>VirtualMachine.Config.AdvancedConfig</code></li><li><code>VirtualMachine.Config.Annotation</code></li><li><code>VirtualMachine.Config.CPUCount</code></li><li><code>VirtualMachine.Config.DiskExtend</code></li><li><code>VirtualMachine.Config.DiskLease</code></li><li><code>VirtualMachine.Config.EditDevice</code></li><li><code>VirtualMachine.Config.Memory</code></li><li><code>VirtualMachine.Config.RemoveDisk</code></li><li><code>VirtualMachine.Config.Rename</code></li><li><code>VirtualMachine.Config.ResetGuestInfo</code></li><li><code>VirtualMachine.Config.Resource</code></li><li><code>VirtualMachine.Config.Settings</code></li><li><code>VirtualMachine.Config.UpgradeVirtualHardware</code></li><li><code>VirtualMachine.Interact.GuestControl</code></li><li><code>VirtualMachine.Interact.PowerOff</code></li><li><code>VirtualMachine.Interact.PowerOn</code></li><li><code>VirtualMachine.Interact.Reset</code></li><li><code>VirtualMachine.Inventory.Create</code></li><li><code>VirtualMachine.Inventory.CreateFromExisting</code></li><li><code>VirtualMachine.Inventory.Delete</code></li><li><code>VirtualMachine.Provisioning.Clone</code></li><li><code>VirtualMachine.Provisioning.MarkAsTemplate</code></li><li><code>VirtualMachine.Provisioning.DeployTemplate</code></li></ul></td>
</tr>
<tr>
  <td>vSphere vCenter data center</td>
  <td>If the virtual machine folder does not already exist, the installation program creates the virtual machine folder. If your cluster does use the Machine API and you want to set the minimum set of permissions for the API, see the "Minimum permissions for the Machine API" table.</td>
  <td><ul><li><code>Folder.Create</code></li><li><code>Folder.Delete</code></li><li><code>InventoryService.Tagging.ObjectAttachable</code></li><li><code>Resource.AssignVMToPool</code></li><li><code>VApp.Import</code></li><li><code>VirtualMachine.Config.AddExistingDisk</code></li><li><code>VirtualMachine.Config.AddNewDisk</code></li><li><code>VirtualMachine.Config.AddRemoveDevice</code></li><li><code>VirtualMachine.Config.AdvancedConfig</code></li><li><code>VirtualMachine.Config.Annotation</code></li><li><code>VirtualMachine.Config.CPUCount</code></li><li><code>VirtualMachine.Config.DiskExtend</code></li><li><code>VirtualMachine.Config.DiskLease</code></li><li><code>VirtualMachine.Config.EditDevice</code></li><li><code>VirtualMachine.Config.Memory</code></li><li><code>VirtualMachine.Config.RemoveDisk</code></li><li><code>VirtualMachine.Config.Rename</code></li><li><code>VirtualMachine.Config.ResetGuestInfo</code></li><li><code>VirtualMachine.Config.Resource</code></li><li><code>VirtualMachine.Config.Settings</code></li><li><code>VirtualMachine.Config.UpgradeVirtualHardware</code></li><li><code>VirtualMachine.Interact.GuestControl</code></li><li><code>VirtualMachine.Interact.PowerOff</code></li><li><code>VirtualMachine.Interact.PowerOn</code></li><li><code>VirtualMachine.Interact.Reset</code></li><li><code>VirtualMachine.Inventory.Create</code></li><li><code>VirtualMachine.Inventory.CreateFromExisting</code></li><li><code>VirtualMachine.Inventory.Delete</code></li><li><code>VirtualMachine.Provisioning.Clone</code></li><li><code>VirtualMachine.Provisioning.DeployTemplate</code></li><li><code>VirtualMachine.Provisioning.MarkAsTemplate</code></li></ul></td>
</tr>
</tbody>
</table>

<a name="post-installation-vsphere-minimum-permissions_ipi-vsphere-installation-reqs"></a>

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
  <td>If the virtual machine folder does not already exist, the installation program creates the virtual machine folder.</td>
  <td><ul><li><code>Resource.AssignVMToPool</code></li><li><code>VirtualMachine.Config.AddExistingDisk</code></li><li><code>VirtualMachine.Config.AddRemoveDevice</code></li><li><code>VirtualMachine.Interact.PowerOff</code></li><li><code>VirtualMachine.Interact.PowerOn</code></li><li><code>VirtualMachine.Provisioning.DeployTemplate</code></li></ul></td>
</tr>
</tbody>
</table>

<a name="installation-vsphere-minimum-permissions-storage_ipi-vsphere-installation-reqs"></a>

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
  <td>If the virtual machine folder does not already exist, the installation program creates the virtual machine folder.</td>
  <td><ul><li><code>VirtualMachine.Config.AddExistingDisk</code></li><li><code>VirtualMachine.Config.AddRemoveDevice</code></li></ul></td>
</tr>
</tbody>
</table>

<a name="post-installation-vsphere-minimum-machine-api_ipi-vsphere-installation-reqs"></a>

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
  <td>If the virtual machine folder does not already exist, the installation program creates the virtual machine folder.</td>
  <td><ul><li><code>Resource.AssignVMToPool</code></li><li><code>VirtualMachine.Interact.PowerOff</code></li><li><code>VirtualMachine.Interact.PowerOn</code></li><li><code>VirtualMachine.Provisioning.DeployTemplate</code></li></ul></td>
</tr>
</tbody>
</table>

### Using OpenShift Container Platform with vMotion {#installation-vsphere-installer-infra-requirements-vmotion_ipi-vsphere-installation-reqs}

If you intend on using vMotion in your vSphere environment, consider the following before installing an OpenShift Container Platform cluster.

- Using Storage vMotion can cause issues and is not supported.
- Using VMware compute vMotion to migrate the workloads for both OpenShift Container Platform compute machines and control plane machines is generally supported, where *generally* implies that you meet all VMware best practices for vMotion. To help ensure the uptime of your compute and control plane nodes, ensure that you follow the VMware best practices for vMotion, and use VMware anti-affinity rules to improve the availability of OpenShift Container Platform during maintenance or hardware issues.
  For more information about vMotion and anti-affinity rules, see the VMware vSphere documentation for  [vMotion networking requirements](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.vcenterhost.doc/GUID-3B41119A-1276-404B-8BFB-A32409052449.html) and [VM anti-affinity rules](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.resmgmt.doc/GUID-FBE46165-065C-48C2-B775-7ADA87FF9A20.html).
- If you are using VMware vSphere volumes in your pods, migrating a VM across datastores, either manually or through Storage vMotion, causes invalid references within OpenShift Container Platform persistent volume (PV) objects that can result in data loss.
- OpenShift Container Platform does not support selective migration of virtual machine disks (VMDKs) across datastores, using datastore clusters for VM provisioning or for dynamic or static provisioning of PVs, or using a datastore that is part of a datastore cluster for dynamic or static provisioning of PVs.
  :::warning

  You can specify the path of any datastore that exists in a datastore cluster. By default, Storage Distributed Resource Scheduler (SDRS), which uses Storage vMotion, is automatically enabled for a datastore cluster. Red Hat does not support Storage vMotion, so you must disable SDRS to avoid data loss issues for your OpenShift Container Platform cluster. If you must specify VMs across many datastores, use a `datastore` object to specify a failure domain in your cluster’s `install-config.yaml` configuration file. For more information, see "VMware vSphere region and zone enablement".

  :::

### Cluster resources {#installation-vsphere-installer-infra-requirements-resources_ipi-vsphere-installation-reqs}

When you deploy an OpenShift Container Platform cluster that uses installer-provisioned infrastructure, the installation program must be able to create several resources in your vCenter instance.

A standard OpenShift Container Platform installation creates the following vCenter resources:

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

### Cluster limits {#installation-vsphere-installer-infra-requirements-limits_ipi-vsphere-installation-reqs}

Available resources vary between clusters. A limit exists for the number of possible clusters within vCenter, primarily by available storage space and any limitations on the number of required resources. Be sure to consider both limitations to the vCenter resources that the cluster creates and the resources that you require to deploy a cluster, such as IP addresses and networks.

### Networking requirements {#installation-vsphere-installer-infra-requirements-networking_ipi-vsphere-installation-reqs}

You can use Dynamic Host Configuration Protocol (DHCP) for the network and configure the DHCP server to set persistent IP addresses to machines in your cluster. In the DHCP lease, you must configure the DHCP to use the default gateway.

:::note

You do not need to use the DHCP for the network if you want to provision nodes with static IP addresses.

:::

If you are installing to a restricted environment, the VM in your restricted network must have access to vCenter so that it can provision and manage nodes, persistent volume claims (PVCs), and other resources.

:::note

Ensure that each OpenShift Container Platform node in the cluster has access to a Network Time Protocol (NTP) server that is discoverable by DHCP. Installation is possible without an NTP server. However, asynchronous server clocks can cause errors, which the NTP server prevents.

:::

Additionally, you must create the following networking resources before you install the OpenShift Container Platform cluster:

### Required IP addresses {#installation-vsphere-installer-infra-requirements-_ipi-vsphere-installation-reqs}

For a network that uses DHCP, an installer-provisioned vSphere installation requires two static IP addresses:

- The **API** address for accessing the cluster API.
- The **Ingress** address for cluster ingress traffic.

You must give these IP addresses to the installation program when you install the OpenShift Container Platform cluster.

### DNS records {#installation-vsphere-installer-infra-requirements-dns-records_ipi-vsphere-installation-reqs}

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

### Static IP addresses for vSphere nodes {#installation-vsphere-installer-infra-static-ip-nodes_ipi-vsphere-installation-reqs}

You can provision bootstrap, control plane, and compute nodes to be configured with static IP addresses in environments where Dynamic Host Configuration Protocol (DHCP) does not exist. To configure this environment, you must provide values to the `platform.vsphere.hosts.role` parameter in the `install-config.yaml` file.

By default, the installation program is configured to use the DHCP for the network, but this network has limited configurable capabilities.

After you define one or more machine pools in your `install-config.yaml` file, you can define network definitions for nodes on your network. Ensure that the number of network definitions matches the number of machine pools that you configured for your cluster.

```yaml title="Example network configuration that specifies different roles"
# ...
platform:
  vsphere:
    hosts:
    - role: bootstrap
      networkDevice:
        ipAddrs:
        - 192.168.204.10/24
        gateway: 192.168.204.1
        nameservers:
        - 192.168.204.1
    - role: control-plane
      networkDevice:
        ipAddrs:
        - 192.168.204.11/24
        gateway: 192.168.204.1
        nameservers:
        - 192.168.204.1
    - role: control-plane
      networkDevice:
        ipAddrs:
        - 192.168.204.12/24
        gateway: 192.168.204.1
        nameservers:
        - 192.168.204.1
    - role: control-plane
      networkDevice:
        ipAddrs:
        - 192.168.204.13/24
        gateway: 192.168.204.1
        nameservers:
        - 192.168.204.1
    - role: compute
      networkDevice:
        ipAddrs:
        - 192.168.204.14/24
        gateway: 192.168.204.1
        nameservers:
        - 192.168.204.1
# ...
```

where:

`role`
: Specifies a network definition value of `bootstrap`, `control-plane`, or `compute`. You must list at least one `bootstrap` network definition in your `install-config.yaml` configuration file.

`ipAddrs`
: Specifies IPv4, IPv6, or both IP addresses that the installation program passes to the network interface. The machine API controller assigns all configured IP addresses to the default network interface.

`gateway`
: Specifies the default gateway for the network interface.

`nameservers`
: Specifies up to 3 DNS nameservers.

After you deployed your cluster to run nodes with static IP addresses, you can scale a machine to use one of these static IP addresses. Additionally, you can use a machine set to configure a machine to use one of the configured static IP addresses.

**Additional resources**

- [Scaling machines to use static IP addresses](/docs/post_installation_configuration/node-tasks#nodes-vsphere-scaling-machines-static-ip_post-install-node-tasks)
- [Using a machine set to scale machines with configured static IP addresses](/docs/post_installation_configuration/node-tasks#nodes-vsphere-machine-set-scaling-static-ip_post-install-node-tasks)
