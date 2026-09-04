---
title: Creating a compute machine set on vSphere
sidebar_position: 9
---

# Creating a compute machine set on vSphere {#creating-machineset-vsphere}

<a id="creating-machineset-vsphere"></a>

You can define and create a OpenShift Container Platform compute machine set on VMware vSphere to enable the Machine API to automatically scale and manage compute nodes in vSphere. You can create a different compute machine set to serve a specific purpose in your OpenShift Container Platform cluster on vSphere. For example, you might create infrastructure machine sets and related machines so that you can move supporting workloads to the new machines.

:::warning

You can use the advanced machine management and scaling capabilities only in clusters where the Machine API is operational. Clusters with user-provisioned infrastructure require additional validation and configuration to use the Machine API.

Clusters with the infrastructure platform type `none` cannot use the Machine API. This limitation applies even if the compute machines that are attached to the cluster are installed on a platform that supports the feature. This parameter cannot be changed after installation.

To view the platform type for your cluster, run the following command:

```terminal
$ oc get infrastructure cluster -o jsonpath='{.status.platform}'
```

:::

## Sample YAML for a compute machine set custom resource on vSphere {#machineset-yaml-vsphere_creating-machineset-vsphere}

To enable the Machine API to automate node provisioning on VMware vSphere infrastructure, define a `MachineSet` resource with parameters that are specific to vSphere, for example data center, resource pool, and template.

The sample YAML file defines a compute machine set that runs on vSphere and creates nodes that are labeled with `node-role.kubernetes.io/<role>: ""`.

In this sample, `<infrastructure_id>` is the infrastructure ID label that is based on the cluster ID that you set when you provisioned the cluster, and `<role>` is the node label to add.

```yaml
apiVersion: machine.openshift.io/v1beta1
kind: MachineSet
metadata:
  creationTimestamp: null
  labels:
    machine.openshift.io/cluster-api-cluster: <infrastructure_id>
  name: <infrastructure_id>-<role>
  namespace: openshift-machine-api
spec:
  replicas: 1
  selector:
    matchLabels:
      machine.openshift.io/cluster-api-cluster: <infrastructure_id>
      machine.openshift.io/cluster-api-machineset: <infrastructure_id>-<role>
  template:
    metadata:
      creationTimestamp: null
      labels:
        machine.openshift.io/cluster-api-cluster: <infrastructure_id>
        machine.openshift.io/cluster-api-machine-role: <role>
        machine.openshift.io/cluster-api-machine-type: <role>
        machine.openshift.io/cluster-api-machineset: <infrastructure_id>-<role>
    spec:
      metadata:
        creationTimestamp: null
        labels:
          node-role.kubernetes.io/<role>: ""
      providerSpec:
        value:
          apiVersion: machine.openshift.io/v1beta1
          credentialsSecret:
            name: vsphere-cloud-credentials
          dataDisks:
          - name: "<disk_name>"
            provisioningMode: "<mode>"
            sizeGiB: 20
          diskGiB: 120
          kind: VSphereMachineProviderSpec
          memoryMiB: 8192
          metadata:
            creationTimestamp: null
          network:
            devices:
            - networkName: "<vm_network_name>"
          numCPUs: 4
          numCoresPerSocket: 1
          snapshot: ""
          template: <vm_template_name>
          userDataSecret:
            name: worker-user-data
          workspace:
            datacenter: <vcenter_data_center_name>
            datastore: <vcenter_datastore_name>
            folder: <vcenter_vm_folder_path>
            resourcepool: <vsphere_resource_pool>
            server: <vcenter_server_ip>
```

where

`<infrastructure_id>`
: Specifies the infrastructure ID that is based on the cluster ID that you set when you provisioned the cluster. If you have the OpenShift CLI (`oc`) installed, you can obtain the infrastructure ID by running the following command:

  ```terminal
  $ oc get -o jsonpath='{.status.infrastructureName}{"\n"}' infrastructure cluster
  ```

`<infrastructure_id>-<role>`
: Specifies the infrastructure ID and node label.

`<role>`
: Specifies the node label to add.

`<disk_name>`
: Specifies one or more data disk definitions. For more information, see "Configuring data disks by using machine sets".

`<image_name>`
: Specifies the image to use as a boot image for your nodes.

`<vm_network_name>`
: Specifies the vSphere VM network to deploy the compute machine set to. This VM network must be where other compute machines reside in the cluster.

`<vm_template_name>`
: Specifies the vSphere VM template to use as a boot image for your nodes, such as `user-5ddjd-rhcos`. You should use a template with the latest OpenShift Container Platform image when adding a new machine set.

`<vcenter_data_center_name>`
: Specifies the vCenter datacenter to deploy the compute machine set on.

`<vcenter_datastore_name>`
: Specifies the vCenter datastore to deploy the compute machine set on.

`<vcenter_vm_folder_path>`
: Specifies the path to the vSphere VM folder in vCenter, such as `/dc1/vm/user-inst-5ddjd`.

`<vsphere_resource_pool>`
: Specifies the vSphere resource pool for your VMs.

`<vcenter_server_ip>`
: Specifies the vCenter server IP or fully qualified domain name.

**Additional resources**

- [Manually updating the boot image](/docs/machine_configuration/mco-update-boot-images-manual#mco-update-boot-images-manual)

## Minimum required vCenter privileges for compute machine set management {#machineset-vsphere-requirements-user-provisioned-machine-sets_creating-machineset-vsphere}

To manage compute machine sets in an OpenShift Container Platform cluster on vCenter, you must use an account with privileges to read, create, and delete the required resources. Using an account that has global administrative privileges is the simplest way to access all of the necessary permissions.

If you cannot use an account with global administrative privileges, you must create roles to grant the minimum required privileges. The following table lists the minimum vCenter roles and privileges that are required to create, scale, and delete compute machine sets and to delete machines in your OpenShift Container Platform cluster.

**Minimum vCenter roles and privileges required for compute machine set management**

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
  <td><code>InventoryService.Tagging.AttachTag</code> <code>InventoryService.Tagging.CreateCategory</code> <code>InventoryService.Tagging.CreateTag</code> <code>InventoryService.Tagging.DeleteCategory</code> <code>InventoryService.Tagging.DeleteTag</code> <code>InventoryService.Tagging.EditCategory</code> <code>InventoryService.Tagging.EditTag</code> <code>Sessions.ValidateSession</code> <code>StorageProfile.Update</code>^1^ <code>StorageProfile.View</code>^1^</td>
</tr>
<tr>
  <td>vSphere vCenter Cluster</td>
  <td>Always</td>
  <td><code>Resource.AssignVMToPool</code></td>
</tr>
<tr>
  <td>vSphere datastore</td>
  <td>Always</td>
  <td><code>Datastore.AllocateSpace</code> <code>Datastore.Browse</code></td>
</tr>
<tr>
  <td>vSphere Port Group</td>
  <td>Always</td>
  <td><code>Network.Assign</code></td>
</tr>
<tr>
  <td>Virtual Machine Folder</td>
  <td>Always</td>
  <td><code>VirtualMachine.Config.AddRemoveDevice</code> <code>VirtualMachine.Config.AdvancedConfig</code> <code>VirtualMachine.Config.Annotation</code> <code>VirtualMachine.Config.CPUCount</code> <code>VirtualMachine.Config.DiskExtend</code> <code>VirtualMachine.Config.Memory</code> <code>VirtualMachine.Config.Settings</code> <code>VirtualMachine.Interact.PowerOff</code> <code>VirtualMachine.Interact.PowerOn</code> <code>VirtualMachine.Inventory.CreateFromExisting</code> <code>VirtualMachine.Inventory.Delete</code> <code>VirtualMachine.Provisioning.Clone</code></td>
</tr>
<tr>
  <td>vSphere vCenter data center</td>
  <td>If the installation program creates the virtual machine folder.</td>
  <td><code>Resource.AssignVMToPool</code> <code>VirtualMachine.Provisioning.DeployTemplate</code></td>
</tr>
<tr>
  <td colspan="3">^1^ The <code>StorageProfile.Update</code> and <code>StorageProfile.View</code> permissions are required only for storage backends that use the Container Storage Interface (CSI).</td>
</tr>
</tbody>
</table>

The following table details the permissions and propagation settings that are required for compute machine set management.

**Required permissions and propagation settings**

<table>
<thead>
<tr>
  <th>vSphere object</th>
  <th>Folder type</th>
  <th>Propagate to children</th>
  <th>Permissions required</th>
</tr>
</thead>
<tbody>
<tr>
  <td>vSphere vCenter</td>
  <td>Always</td>
  <td>Not required</td>
  <td>Listed required privileges</td>
</tr>
<tr>
  <td rowspan="2">vSphere vCenter data center</td>
  <td>Existing folder</td>
  <td>Not required</td>
  <td><code>ReadOnly</code> permission</td>
</tr>
<tr>
  <td>Installation program creates the folder</td>
  <td>Required</td>
  <td>Listed required privileges</td>
</tr>
<tr>
  <td>vSphere vCenter Cluster</td>
  <td>Always</td>
  <td>Required</td>
  <td>Listed required privileges</td>
</tr>
<tr>
  <td>vSphere vCenter datastore</td>
  <td>Always</td>
  <td>Not required</td>
  <td>Listed required privileges</td>
</tr>
<tr>
  <td>vSphere Switch</td>
  <td>Always</td>
  <td>Not required</td>
  <td><code>ReadOnly</code> permission</td>
</tr>
<tr>
  <td>vSphere Port Group</td>
  <td>Always</td>
  <td>Not required</td>
  <td>Listed required privileges</td>
</tr>
<tr>
  <td>vSphere vCenter Virtual Machine Folder</td>
  <td>Existing folder</td>
  <td>Required</td>
  <td>Listed required privileges</td>
</tr>
</tbody>
</table>

For more information about creating an account with only the required privileges, see [vSphere Permissions and User Management Tasks](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-5372F580-5C23-4E9C-8A4E-EF1B4DD9033E.html) in the vSphere documentation.

## Requirements for clusters with user-provisioned infrastructure to use compute machine sets {#compute-machineset-upi-reqs_creating-machineset-vsphere}

To enable the Machine API to manage and scale compute nodes on user-provisioned infrastructure, you can configure a `MachineSet` YAML file with specific vSphere parameters, for example data center and disk image. To use compute machine sets on clusters that have user-provisioned infrastructure, you must ensure that you cluster configuration supports using the Machine API.

### Obtaining the infrastructure ID {#machineset-upi-reqs-infra-id_creating-machineset-vsphere}

To ensure the Machine API correctly identifies and manages virtual machines (VMs) that belong to a specific cluster, you must add the unique infrastructure ID to the `MachineSet` YAML file to label and link resources. To create compute machine sets, you must be able to supply the infrastructure ID for your cluster.

**Procedure**

- To obtain the infrastructure ID for your cluster, run the following command:
  ```terminal
  $ oc get infrastructure cluster -o jsonpath='{.status.infrastructureName}'
  ```

### Satisfying vSphere credentials requirements {#machineset-upi-reqs-vsphere-creds_creating-machineset-vsphere}

To use compute machine sets and manage virtual machine (VM) resources, the Machine API must be able to interact with vCenter. Credentials that authorize the Machine API components to interact with vCenter must exist in a secret in the `openshift-machine-api` namespace.

**Procedure**

1. To determine whether the required credentials exist, run the following command:
   ```terminal
   $ oc get secret \
     -n openshift-machine-api vsphere-cloud-credentials \
     -o go-template='{{range $k,$v := .data}}{{printf "%s: " $k}}{{if not $v}}{{$v}}{{else}}{{$v | base64decode}}{{end}}{{"\n"}}{{end}}'
   ```

   ```terminal title="Sample output"
   <vcenter-server>.password=<openshift-user-password>
   <vcenter-server>.username=<openshift-user>
   ```

   where

   <dl>
   <dt><code>&lt;vcenter_server&gt;</code></dt>
   <dd>Specifies the IP address or fully qualified domain name (FQDN) of the vCenter server and <code>&lt;openshift_user_password&gt;</code> and <code>&lt;openshift_user&gt;</code> are the OpenShift Container Platform administrator credentials to use.</dd>
   </dl>
2. If the secret does not exist, create it by running the following command:
   ```terminal
   $ oc create secret generic vsphere-cloud-credentials \
     -n openshift-machine-api \
     --from-literal=<vcenter-server>.username=<openshift-user> --from-literal=<vcenter-server>.password=<openshift-user-password>
   ```

### Satisfying Ignition configuration requirements {#machineset-upi-reqs-ignition-config_creating-machineset-vsphere}

For the Machine API to provision virtual machines (VMs) with the correct initial configuration using Ignition, a valid Ignition configuration is required. The Ignition configuration contains the `machine-config-server` address and a system trust bundle for obtaining further Ignition configurations from the Machine Config Operator.

By default, this configuration is stored in the `worker-user-data` secret in the `machine-api-operator` namespace. Compute machine sets reference the secret during the machine creation process.

**Procedure**

1. To determine whether the required secret exists, run the following command:
   ```terminal
   $ oc get secret \
     -n openshift-machine-api worker-user-data \
     -o go-template='{{range $k,$v := .data}}{{printf "%s: " $k}}{{if not $v}}{{$v}}{{else}}{{$v | base64decode}}{{end}}{{"\n"}}{{end}}'
   ```

   ```terminal title="Sample output"
   disableTemplating: false
   userData:
     {
       "ignition": {
         ...
         },
       ...
     }
   ```

   The full output is omitted here, but this is the format to use.
2. If the secret does not exist, create it by running the following command:
   ```terminal
   $ oc create secret generic worker-user-data \
     -n openshift-machine-api \
     --from-file=<installation_directory>/worker.ign
   ```

   Specifies the directory that was used to store your installation assets during cluster installation.

**Additional resources**

- [Understanding the Machine Config Operator](/docs/machine_configuration#machine-config-operator_machine-config-overview)
- [Installing RHCOS and starting the OpenShift Container Platform bootstrap process](/docs/installing/installing_vsphere/upi/installing-vsphere#installation-vsphere-machines_installing-vsphere)

## Creating a compute machine set {#machineset-creating_creating-machineset-vsphere}

To dynamically manage machine compute resources, you can create your own compute machine sets in addition to the compute machine sets created by the installation program. Use the OpenShift Container Platform CLI to automate node provisioning.

:::note

Clusters that are installed with user-provisioned infrastructure have a different networking stack than clusters with infrastructure that is provisioned by the installation program. As a result of this difference, automatic load balancer management is unsupported on clusters that have user-provisioned infrastructure. For these clusters, a compute machine set can only create `worker` and `infra` type machines.

:::

**Prerequisites**

- Deploy an OpenShift Container Platform cluster.
- Install the OpenShift CLI (`oc`).
- Log in to `oc` as a user with `cluster-admin` permission.
- Have the necessary permissions to deploy VMs in your vCenter instance and have the required access to the datastore specified.
- If your cluster uses user-provisioned infrastructure, you have satisfied the specific Machine API requirements for that configuration.

**Procedure**

1. Create a new YAML file that contains the compute machine set custom resource (CR) sample and is named `<file_name>.yaml`.
   Ensure that you set the `<clusterID>` and `<role>` parameter values.
2. Optional: If you are not sure which value to set for a specific field, you can check an existing compute machine set from your cluster.
   1. To list the compute machine sets in your cluster, run the following command:
      ```terminal
      $ oc get machinesets -n openshift-machine-api
      ```

      The following is example output:

      ```terminal
      NAME                                DESIRED   CURRENT   READY   AVAILABLE   AGE
      agl030519-vplxk-worker-us-east-1a   1         1         1       1           55m
      agl030519-vplxk-worker-us-east-1b   1         1         1       1           55m
      agl030519-vplxk-worker-us-east-1c   1         1         1       1           55m
      agl030519-vplxk-worker-us-east-1d   0         0                             55m
      agl030519-vplxk-worker-us-east-1e   0         0                             55m
      agl030519-vplxk-worker-us-east-1f   0         0                             55m
      ```
   2. To view values of a specific compute machine set custom resource (CR), run the following command:
      ```terminal
      $ oc get machineset <machineset_name> \
        -n openshift-machine-api -o yaml
      ```

      The following is example output:

      ```yaml
      apiVersion: machine.openshift.io/v1beta1
      kind: MachineSet
      metadata:
        labels:
          machine.openshift.io/cluster-api-cluster: <infrastructure_id>
        name: <infrastructure_id>-<role>
        namespace: openshift-machine-api
      spec:
        replicas: 1
        selector:
          matchLabels:
            machine.openshift.io/cluster-api-cluster: <infrastructure_id>
            machine.openshift.io/cluster-api-machineset: <infrastructure_id>-<role>
        template:
          metadata:
            labels:
              machine.openshift.io/cluster-api-cluster: <infrastructure_id>
              machine.openshift.io/cluster-api-machine-role: <role>
              machine.openshift.io/cluster-api-machine-type: <role>
              machine.openshift.io/cluster-api-machineset: <infrastructure_id>-<role>
          spec:
            providerSpec:
              ...
      ```

      where:

      <dl>
      <dt><code>metadata.labels.machine.openshift.io/cluster-api-cluster</code></dt>
      <dd>Specifies the cluster infrastructure ID.</dd>
      <dt><code>metadata.labels.name</code></dt>
      <dd>Specifies a default node label.</dd>
      </dl>

      :::note

      For clusters that have user-provisioned infrastructure, a compute machine set can only create `worker` and `infra` type machines.

      :::

      <dl>
      <dt><code>spec.template.metadata.spec.providerSpec</code></dt>
      <dd>Specifies the values of the compute machine set CR. The values are platform-specific. For more information about <code>&lt;providerSpec&gt;</code> parameters in the CR, see the sample compute machine set CR configuration for your provider.</dd>
      </dl>
   3. If you are creating a compute machine set for a cluster that has user-provisioned infrastructure, note the following important values:
      ```yaml title="Example vSphere providerSpec values"
      apiVersion: machine.openshift.io/v1beta1
      kind: MachineSet
      ...
      template:
        ...
        spec:
          providerSpec:
            value:
              apiVersion: machine.openshift.io/v1beta1
              credentialsSecret:
                name: vsphere-cloud-credentials
              dataDisks:
              - name: <disk_name>
                provisioningMode: <mode>
                sizeGiB: 10
              diskGiB: 120
              kind: VSphereMachineProviderSpec
              memoryMiB: 16384
              network:
                devices:
                  - networkName: "<vm_network_name>"
              numCPUs: 4
              numCoresPerSocket: 4
              snapshot: ""
              template: <vm_template_name>
              userDataSecret:
                name: worker-user-data
              workspace:
                datacenter: <vcenter_data_center_name>
                datastore: <vcenter_datastore_name>
                folder: <vcenter_vm_folder_path>
                resourcepool: <vsphere_resource_pool>
                server: <vcenter_server_address>
      ```

      where:

      <dl>
      <dt><code>vsphere-cloud-credentials</code></dt>
      <dd>Specifies the name of the secret in the <code>openshift-machine-api</code> namespace that contains the required vCenter credentials.</dd>
      <dt><code>&lt;disk_name&gt;</code></dt>
      <dd>Specifies the collection of data disk definitions. For more information, see "Configuring data disks by using machine sets".</dd>
      <dt><code>&lt;vm_template_name&gt;</code></dt>
      <dd>Specifies the name of the RHCOS VM template for your cluster that was created during installation.</dd>
      <dt><code>worker-user-data</code></dt>
      <dd>Specifies the name of the secret in the <code>openshift-machine-api</code> namespace that contains the required Ignition configuration credentials.</dd>
      <dt><code>&lt;vcenter_server_address&gt;</code></dt>
      <dd>Specifies the IP address or fully qualified domain name (FQDN) of the vCenter server.</dd>
      </dl>
3. Create a `MachineSet` CR by running the following command:
   ```terminal
   $ oc create -f <file_name>.yaml
   ```

**Verification**

- View the list of compute machine sets by running the following command:
  ```terminal
  $ oc get machineset -n openshift-machine-api
  ```

  The following is example output:

  ```terminal

  NAME                                DESIRED   CURRENT   READY   AVAILABLE   AGE
  agl030519-vplxk-infra-us-east-1a    1         1         1       1           11m
  agl030519-vplxk-worker-us-east-1a   1         1         1       1           55m
  agl030519-vplxk-worker-us-east-1b   1         1         1       1           55m
  agl030519-vplxk-worker-us-east-1c   1         1         1       1           55m
  agl030519-vplxk-worker-us-east-1d   0         0                             55m
  agl030519-vplxk-worker-us-east-1e   0         0                             55m
  agl030519-vplxk-worker-us-east-1f   0         0                             55m
  ```

  When the new compute machine set is available, the `DESIRED` and `CURRENT` values match. If the compute machine set is not available, wait a few minutes and run the command again.

## Labeling GPU machine sets for the cluster autoscaler {#machineset-label-gpu-autoscaler_creating-machineset-vsphere}

Label your machine sets to indicate which machines the cluster autoscaler can use for GPU-enabled nodes. Applying the accelerator label helps ensure that the autoscaler deploys the correct resources for your GPU workloads.

**Prerequisites**

- Your cluster uses a cluster autoscaler.

**Procedure**

- On the machine set that you want to create machines for the cluster autoscaler to use to deploy GPU-enabled nodes, add a `cluster-api/accelerator` label:
  ```yaml
  apiVersion: machine.openshift.io/v1beta1
  kind: MachineSet
  metadata:
    name: machine-set-name
  spec:
    template:
      spec:
        metadata:
          labels:
            cluster-api/accelerator: <accelerator_name>
  ```

  where:

  **`<accelerator_name>`**

  Specifies a label of your choice that consists of alphanumeric characters, `-`, `_`, or `.` and starts and ends with an alphanumeric character. For example, you might use `nvidia-t4` to represent Nvidia T4 GPUs, or `nvidia-a10g` for A10G GPUs.

  :::note

  You must specify the value of this label for the `spec.resourceLimits.gpus.type` parameter in your `ClusterAutoscaler` CR. For more information, see "Cluster autoscaler resource definition".

  :::

**Additional resources**

- [Cluster autoscaler resource definition](/docs/machine_management/applying-autoscaling#cluster-autoscaler-cr_applying-autoscaling)

## Adding tags to machines by using machine sets {#machine-api-vmw-add-tags_creating-machineset-vsphere}

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

## Configuring multiple network interface controllers by using machine sets {#machineset-vsphere-multiple-nics_creating-machineset-vsphere}

By configuring multiple network interface controllers (NICs), you can provide dedicated network links in the node virtual machines (VMs) for uses such as storage or databases. OpenShift Container Platform clusters on VMware vSphere support connecting up to 10 network NICs to a node.

You can use machine sets to manage this configuration.

- If you want to use multiple NICs in a vSphere cluster that was not configured to do so during installation, you can use machine sets to implement this configuration.
- If your cluster was set up during installation to use multiple NICs, machine sets that you create can use your existing failure domain configuration.
- If your failure domain configuration changes, you can use machine sets to make updates that reflect those changes.

tag:controlplane[][]

:::note

This feature is not compatible with a control plane machine set that uses more than one failure domain.

:::

end:controlplane[][]

**Prerequisites**

- You have administrator access to OpenShift CLI (`oc`) for an OpenShift Container Platform cluster on vSphere.

**Procedure**

1. For a cluster that already uses multiple NICs, obtain the following values from the `Infrastructure` resource by running the following command:
   ```terminal
   $ oc get infrastructure cluster -o=jsonpath={.spec.platformSpec.vsphere.failureDomains}
   ```

   **Required network interface controller values**

   | `Infrastructure` resource value | Placeholder value for sample machine set | Description |
   | --- | --- | --- |
   | `failureDomain.topology.networks[0]` | `<vm_network_name_1>` | The name of the first NIC to use. |
   | `failureDomain.topology.networks[1]` | `<vm_network_name_2>` | The name of the second NIC to use. |
   | `failureDomain.topology.networks[<n-1>]` | `<vm_network_name_n>` | The name of the __n__th NIC to use. Collect the name of each NIC in the `Infrastructure` resource. |
   | `failureDomain.topology.template` | `<vm_template_name>` | The vSphere VM template to use. |
   | `failureDomain.topology.datacenter` | `<vcenter_data_center_name>` | The vCenter data center to deploy the machine set on. |
   | `failureDomain.topology.datastore` | `<vcenter_datastore_name>` | The vCenter datastore to deploy the machine set on. |
   | `failureDomain.topology.folder` | `<vcenter_vm_folder_path>` | The path to the vSphere VM folder in vCenter, such as `/dc1/vm/user-inst-5ddjd`. |
   | `failureDomain.topology.computeCluster` + `/Resources` | `<vsphere_resource_pool>` | The vSphere resource pool for your VMs. |
   | `failureDomain.server` | `<vcenter_server_ip>` | The vCenter server IP or fully qualified domain name (FQDN). |
2. In a text editor, open the YAML file for an existing machine set or create a new one.
3. Use a machine set configuration formatted like the following example.
   - For a cluster that currently uses multiple NICs, use the values from the `Infrastructure` resource to populate the values in the machine set custom resource.
   - For a cluster that is not using multiple NICs, populate the values you want to use in the machine set custom resource.

   ```yaml title="Sample machine set"
   tag::compute[]
   apiVersion: machine.openshift.io/v1beta1
   kind: MachineSet
   # ...
   spec:
     template:
       spec:
         providerSpec:
           value:
             network:
               devices:
               - networkName: "<vm_network_name_1>"
               - networkName: "<vm_network_name_2>"
             template: <vm_template_name>
             workspace:
               datacenter: <vcenter_data_center_name>
               datastore: <vcenter_datastore_name>
               folder: <vcenter_vm_folder_path>
               resourcepool: <vsphere_resource_pool>
               server: <vcenter_server_ip>
   # ...
   end::compute[]
   tag::controlplane[]
   apiVersion: machine.openshift.io/v1
   kind: ControlPlaneMachineSet
   # ...
   spec:
     template:
       machines_v1beta1_machine_openshift_io:
         spec:
           providerSpec:
             value:
               network:
                 devices:
                 - networkName: "<vm_network_name_1>"
                 - networkName: "<vm_network_name_2>"
               template: <vm_template_name>
               workspace:
                 datacenter: <vcenter_data_center_name>
                 datastore: <vcenter_datastore_name>
                 folder: <vcenter_vm_folder_path>
                 resourcepool: <vsphere_resource_pool>
                 server: <vcenter_server_ip>

   # ...
   end::controlplane[]
   ```

   where: tag:compute[][]

   **`spec.template.spec.providerSpec.value.network.devices`**

   Specifies a list of up to 10 NICs to use.

   **`spec.template.spec.providerSpec.value.network.template`**

   Specifies the vSphere VM template to use, such as `user-5ddjd-rhcos`.

   **`spec.template.spec.providerSpec.value.network.workspace.datacenter`**

   Specifies the vCenter data center to deploy the machine set on.

   **`spec.template.spec.providerSpec.value.network.workspace.datastore`**

   Specifies the vCenter datastore to deploy the machine set on.

   **`spec.template.spec.providerSpec.value.network.workspace.folder`**

   Specifies the path to the vSphere VM folder in vCenter, such as `/dc1/vm/user-inst-5ddjd`.

   **`spec.template.spec.providerSpec.value.network.workspace.resourcepool`**

   Specifies the vSphere resource pool for your VMs.

   **`spec.template.spec.providerSpec.value.network.workspace.server`**

   Specifies the vCenter server IP or fully qualified domain name (FQDN).

   end:compute[][] tag:controlplane[][]

   **`spec.template.machines_v1beta1_machine_openshift_io.spec.providerSpec.value.network.devices`**

   Specifies a list of up to 10 NICs to use.

   **`spec.template.machines_v1beta1_machine_openshift_io.spec.providerSpec.value.network.template`**

   Specifies the vSphere VM template to use, such as `user-5ddjd-rhcos`.

   **`spec.template.machines_v1beta1_machine_openshift_io.spec.providerSpec.value.network.workspace.datacenter`**

   Specifies the vCenter data center to deploy the machine set on.

   **`spec.template.machines_v1beta1_machine_openshift_io.spec.providerSpec.value.network.workspace.datastore`**

   Specifies the vCenter datastore to deploy the machine set on.

   **`spec.template.machines_v1beta1_machine_openshift_io.spec.providerSpec.value.network.workspace.folder`**

   Specifies the path to the vSphere VM folder in vCenter, such as `/dc1/vm/user-inst-5ddjd`.

   **`spec.template.machines_v1beta1_machine_openshift_io.spec.providerSpec.value.network.workspace.resourcepool`**

   Specifies the vSphere resource pool for your VMs.

   **`spec.template.machines_v1beta1_machine_openshift_io.spec.providerSpec.value.network.workspace.server`**

   Specifies the vCenter server IP or fully qualified domain name (FQDN).

   end:controlplane[][]

## Configuring data disks by using machine sets {#machineset-vsphere-data-disks_creating-machineset-vsphere}

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
