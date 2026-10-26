---
title: Manage virtual machine instances
sidebar_position: 13
---

# Manage virtual machine instances {#virt-manage-vmis}

<a id="virt-manage-vmis"></a>

Manage standalone virtual machine instances (VMIs) that were created independently outside of the OpenShift Virtualization environment through the web console by using `oc` or `virtctl` commands from the command-line interface (CLI).

The `virtctl` command provides more virtualization options than the `oc` command. For example, you can use `virtctl` to pause a VM or expose a port.

## Virtual machine instances {#virt-about-vmis_virt-manage-vmis}

In OpenShift Virtualization, a virtual machine instance (VMI) is an active, powered-on virtual instance, as opposed to a virtual machine, which is the saved configuration. You can create standalone VMIs independently without a parent template, and you can manage them directly.

A standalone VMI is created and started independently with a script, through automation, or by using other methods in the CLI. In your environment, you might have standalone VMIs that were developed and started outside of the OpenShift Virtualization environment. You can continue to manage those standalone VMIs by using the CLI. You can also use the web console for specific tasks associated with standalone VMIs:

- List standalone VMIs and their details.
- Edit labels and annotations for a standalone VMI.
- Delete a standalone VMI.

When you delete a VM, the associated VMI is automatically deleted. You delete a standalone VMI directly because it is not owned by VMs or other objects.

:::note

Before you uninstall OpenShift Virtualization, list and view the standalone VMIs by using the CLI or the web console. Then, delete any outstanding VMIs.

:::

When you edit a VM, some settings might be applied to the VMIs dynamically and without the need for a restart. Any change made to a VM object that cannot be applied to the VMIs dynamically will trigger the `RestartRequired` VM condition. Changes are effective on the next reboot, and the condition is removed.

## List all VMIs by using the CLI {#virt-listing-vmis-cli_virt-manage-vmis}

You can use the OpenShift CLI (`oc`) to list all virtual machine instances (VMIs) in your cluster. The list includes standalone VMIs and VMIs that are owned by virtual machines (VMs).

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).

**Procedure**

- List all VMIs by running the following command:
  ```terminal
  $ oc get vmis -A
  ```

## List standalone VMIs by using the web console {#virt-listing-vmis-web_virt-manage-vmis}

You can list the standalone virtual machine instances (VMIs) in your cluster by using the OpenShift Container Platform web console. The console does not show VMIs that are owned by a virtual machine (VM).

:::note

VMIs that are owned by VMs or other objects are not displayed in the web console. The web console displays only standalone VMIs. If you want to list all VMIs in your cluster, you must use the CLI.

:::

**Procedure**

- Click **Virtualization** → **VirtualMachines** from the side menu.
  You can identify a standalone VMI by a dark colored badge next to its name.

## Search for standalone VMIs by using the web console {#virt-searching-vmis-web_virt-manage-vmis}

You can search for standalone virtual machine instances (VMIs) from the search bar on the **VirtualMachines** page of the OpenShift Container Platform web console. Advanced search filters narrow the results by name, project, labels, and IP address.

**Procedure**

1. In the OpenShift Container Platform console, click **Virtualization** → **VirtualMachines** from the side menu.
2. In the search bar at the top of the page, type a VM name, label, or IP address.
3. In the suggestions list, choose one of the following options:
   - Click a VM name to open its details page.
   - Click **All search results found for ...** to view results on a dedicated page.
   - Click a related suggestion to prefill search filters.
4. Optional: To open advanced search options, click the sliders icon next to the search bar. Expand the **Details** section and specify one or more of the available filters: **Name**, **Project**, **Description**, **Labels**, **Date created**, **vCPU**, and **Memory**.
5. Optional: Expand the **Network** section and enter an IP address to filter by.
6. Click **Search**.
7. Optional: If Advanced Cluster Management (ACM) is installed, use the **Cluster** dropdown to search across multiple clusters.
8. Optional: Click the **Save search** icon to store your search in the `kubevirt-user-settings` ConfigMap.

## Edit a standalone VMI by using the web console {#virt-editing-vmis-web_virt-manage-vmis}

You can edit the annotations and labels of a standalone virtual machine instance (VMI) by using the OpenShift Container Platform web console. No other fields are editable.

**Procedure**

1. In the OpenShift Container Platform console, click **Virtualization** → **VirtualMachines** from the side menu.
2. Select a standalone VMI to open the **VirtualMachineInstance details** page.
3. On the **Details** tab, click the pencil icon beside **Annotations** or **Labels**.
4. Make the relevant changes and click **Save**.

## Delete a standalone VMI by using the CLI {#virt-deleting-vmis-cli_virt-manage-vmis}

You can delete a standalone virtual machine instance (VMI) by using the OpenShift CLI (`oc`). A standalone VMI has no owning VM, so no other object deletes it for you.

**Prerequisites**

- Identify the name of the VMI that you want to delete.
- You have installed the OpenShift CLI (`oc`).

**Procedure**

- Delete the VMI by running the following command:
  ```terminal
  $ oc delete vmi <vmi_name>
  ```

## Delete a standalone VMI by using the web console {#virt-deleting-vmis-web_virt-manage-vmis}

You can delete a standalone virtual machine instance (VMI) by using the OpenShift Container Platform web console. The deletion is permanent and you cannot recover the VMI.

**Procedure**

1. In the OpenShift Container Platform web console, click **Virtualization** → **VirtualMachines** from the side menu.
2. Click **Actions** → **Delete VirtualMachineInstance**.
3. In the confirmation pop-up window, click **Delete** to permanently delete the standalone VMI.

**Additional resources**

- [Using the CLI tools](/docs/virt/getting_started/virt-using-the-cli-tools#virt-using-the-cli-tools)
