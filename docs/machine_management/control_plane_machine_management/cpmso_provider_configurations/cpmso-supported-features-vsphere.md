---
title: Configuring VMware vSphere features for control plane machines
sidebar_position: 11
---

# Configuring VMware vSphere features for control plane machines {#cpmso-supported-features-vsphere}

<a id="cpmso-supported-features-vsphere"></a>

You can enable or change the configuration of features for your control plane machines by editing values in the control plane machine set specification.

When you save an update to the control plane machine set, the Control Plane Machine Set Operator updates the control plane machines according to your configured update strategy. For more information, see "Updating the control plane configuration".

## Adding tags to machines by using machine sets {#machine-api-vmw-add-tags_cpmso-supported-features-vsphere}

To ensure that your cluster remains scalable and resilient, you can use a `MachineSet` object and machine health checks to automate the provisioning and repair of nodes.

OpenShift Container Platform adds a cluster-specific tag to each virtual machine (VM) that it creates. The installation program uses these tags to select the VMs to delete when uninstalling a cluster.

In addition to the cluster-specific tags assigned to VMs, you can configure a machine set to add up to 10 additional vSphere tags to the VMs it provisions.

**Prerequisites**

- You have access to an OpenShift Container Platform cluster installed on vSphere using an account with `cluster-admin` permissions.
- You have access to the VMware vCenter console associated with your cluster.
- You have created a tag in the vCenter console.
- You have installed the OpenShift CLI (`oc`).

**Procedure**

1. Use the vCenter console to find the tag ID for any tag that you want to add to your machines:
   1. Log in to the vCenter console.
   2. From the **Home** menu, click **Tags & Custom Attributes**.
   3. Select a tag that you want to add to your machines.
   4. Use the browser URL for the tag that you select to identify the tag ID.
      ```text title="Example tag URL"
      https://vcenter.example.com/ui/app/tags/tag/urn:vmomi:InventoryServiceTag:208e713c-cae3-4b7f-918e-4051ca7d1f97:GLOBAL/permissions
      ```

      ```text title="Example tag ID"
      urn:vmomi:InventoryServiceTag:208e713c-cae3-4b7f-918e-4051ca7d1f97:GLOBAL
      ```
2. In a text editor, open the YAML file for an existing machine set or create a new one.
3. Edit the following lines under the `providerSpec` field:
   ```yaml
   tag::compute[]
   apiVersion: machine.openshift.io/v1beta1
   kind: MachineSet
   end::compute[]
   tag::controlplane[]
   apiVersion: machine.openshift.io/v1
   kind: ControlPlaneMachineSet
   end::controlplane[]
   # ...
   spec:
     template:
       spec:
         providerSpec:
           value:
             tagIDs:
             - <tag_id_value>
   # ...
   ```

   where

   <dl>
   <dt><code>spec.template.spec.providerSpec.value.tagIDs</code></dt>
   <dd>Specifies a list of up to 10 tags to add to the machines that this machine set provisions. Replace <code>&lt;tag_id_value&gt;</code> with the tag that you want to add to your machines. For example, <code>urn:vmomi:InventoryServiceTag:208e713c-cae3-4b7f-918e-4051ca7d1f97:GLOBAL</code>.</dd>
   </dl>

## Configuring data disks by using machine sets {#machineset-vsphere-data-disks_cpmso-supported-features-vsphere}

To provide persistent storage beyond the root volume for specialized application workloads, define a `dataDisks` array in the `MachineSet` YAML file to specify disk size and storage policy.

OpenShift Container Platform clusters on VMware vSphere support adding up to 29 disks to the virtual machine (VM) controller.

:::warning

Configuring vSphere data disks is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.

For more information about the support scope of Red Hat Technology Preview features, see [Technology Preview Features Support Scope](https://access.redhat.com/support/offerings/techpreview/).

:::

By configuring data disks, you can attach disks to VMs and use them to store data for etcd, container images, and other uses. Separating data can help avoid filling the primary disk so that important activities such as upgrades have the resources that they require.

:::note

Adding data disks attaches them to the VM and mounts them to the location that RHCOS designates.

:::

**Prerequisites**

- You have administrator access to OpenShift CLI (`oc`) for an OpenShift Container Platform cluster on vSphere.

**Procedure**

1. In a text editor, open the YAML file for an existing machine set or create a new one.
2. Edit the following lines under the `providerSpec` field:
   ```yaml
   tag::compute[]
   apiVersion: machine.openshift.io/v1beta1
   kind: MachineSet
   end::compute[]
   tag::controlplane[]
   apiVersion: machine.openshift.io/v1
   kind: ControlPlaneMachineSet
   end::controlplane[]
   # ...
   spec:
     template:
   tag::compute[]
       spec:
         providerSpec:
           value:
             dataDisks:
             - name: "<disk_name>"
               provisioningMode: "<mode>"
               sizeGiB: 20
             - name: "<disk_name>"
               provisioningMode: "<mode>"
               sizeGiB: 20
   end::compute[]
   tag::controlplane[]
       machines_v1beta1_machine_openshift_io:
         spec:
           providerSpec:
             value:
               dataDisks:
               - name: "<disk_name>"
                 provisioningMode: "<mode>"
                 sizeGiB: 20
               - name: "<disk_name>"
                 provisioningMode: "<mode>"
                 sizeGiB: 20
   end::controlplane[]
   # ...
   ```

   where tag:compute[][]

   **`spec.template.spec.providerSpec.value.dataDisks`**

   Specifies a collection of 1-29 data disk definitions. This sample configuration shows the formatting to include two data disk definitions.

   **`spec.template.spec.providerSpec.value.dataDisks.name`**

   Specifies the name of the data disk. The name must meet the following requirements:

   - Start and end with an alphanumeric character
   - Consist only of alphanumeric characters, hyphens (`-`), and underscores (`_`)
   - Have a maximum length of 80 characters

   **`spec.template.spec.providerSpec.value.dataDisks.provisioningMode`**

   Specifies the data disk provisioning method. This value defaults to the vSphere default storage policy if not set. Valid values are `Thin`, `Thick`, and `EagerlyZeroed`.

   **`spec.template.spec.providerSpec.value.dataDisks.sizeGiB`**

   Specifies the size of the data disk in GiB. The maximum size is 16,384 GiB. end:compute[][]

   tag:controlplane[][]

   **`spec.template.machines_v1beta1_machine_openshift_io.spec.providerSpec.value.dataDisks`**

   Specifies a collection of 1-29 data disk definitions. This sample configuration shows the formatting to include two data disk definitions.

   **`spec.template.machines_v1beta1_machine_openshift_io.spec.providerSpec.value.dataDisks.name`**

   Specifies the name of the data disk. The name must meet the following requirements:

   - Start and end with an alphanumeric character
   - Consist only of alphanumeric characters, hyphens (`-`), and underscores (`_`)
   - Have a maximum length of 80 characters

   **`spec.template.machines_v1beta1_machine_openshift_io.spec.providerSpec.value.dataDisks.provisioningMode`**

   Specifies the data disk provisioning method. This value defaults to the vSphere default storage policy if not set. Valid values are `Thin`, `Thick`, and `EagerlyZeroed`.

   **`spec.template.machines_v1beta1_machine_openshift_io.spec.providerSpec.value.dataDisks.sizeGiB`**

   Specifies the size of the data disk in GiB. The maximum size is 16,384 GiB. end:controlplane[][]

**Additional resources**

- [Updating the control plane configuration](/docs/machine_management/control_plane_machine_management/cpmso-managing-machines#cpmso-feat-config-update_cpmso-managing-machines)
- [Control plane configuration options for VMware vSphere](/docs/machine_management/control_plane_machine_management/cpmso_provider_configurations/cpmso-config-options-vsphere#cpmso-config-options-vsphere)
