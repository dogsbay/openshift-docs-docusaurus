---
title: "Red&#160;Hat OpenShift support for Windows Containers overview"
sidebar_position: 1
---

# Red Hat OpenShift support for Windows Containers overview {#windows-container-overview}

<a id="windows-container-overview"></a>

You can use Red Hat OpenShift support for Windows Containers to run Windows compute nodes in an OpenShift Container Platform cluster by using the Red Hat Windows Machine Config Operator (WMCO) to install and manage Windows nodes.

## Managing Windows container workloads {#managing-windows-container-workloads_windows-container-overview}

With a Red Hat subscription, you can get support for running Windows workloads in OpenShift Container Platform.

Windows instances deployed by the WMCO are configured with the containerd container runtime. For more information, see the [release notes](/docs/windows_containers/wmco_rn/windows-containers-release-notes#windows-containers-release-notes).

You can add Windows nodes either by creating a [compute machine set](/docs/windows_containers/creating_windows_machinesets/creating-windows-machineset-aws#creating-windows-machineset-aws) or by specifying existing Bring-Your-Own-Host (BYOH) Windows instances through a [ConfigMap](/docs/windows_containers/byoh-windows-instance#byoh-windows-instance).

:::note

Compute machine sets are not supported for bare metal or provider agnostic clusters.

:::

For workloads including both Linux and Windows, OpenShift Container Platform allows you to deploy Windows workloads running on Windows Server containers while also providing traditional Linux workloads hosted on Red Hat Enterprise Linux CoreOS (RHCOS) or Red Hat Enterprise Linux (RHEL). For more information, see [getting started with Windows container workloads](/docs/windows_containers/understanding-windows-container-workloads#understanding-windows-container-workloads).

You need the WMCO to run Windows workloads in your cluster. The WMCO orchestrates the process of deploying and managing Windows workloads on a cluster. For more information, see [how to enable Windows container workloads](/docs/windows_containers/enabling-windows-container-workloads#enabling-windows-container-workloads).

You can create a Windows `MachineSet` object to create infrastructure Windows machine sets and related machines so that you can move supported Windows workloads to the new Windows machines. You can create a Windows `MachineSet` object on multiple platforms.

You can [schedule Windows workloads](/docs/windows_containers/scheduling-windows-workloads#scheduling-windows-workloads) to Windows compute nodes.

You can [perform Windows Machine Config Operator upgrades](/docs/windows_containers/windows-node-upgrades#windows-node-upgrades) to ensure that your Windows nodes have the latest updates.

You can [remove a Windows node](/docs/windows_containers/removing-windows-nodes#removing-windows-nodes) by deleting a specific machine.

You can [use Bring-Your-Own-Host (BYOH) Windows instances](/docs/windows_containers/byoh-windows-instance#byoh-windows-instance) to repurpose Windows Server VMs and bring them to OpenShift Container Platform. BYOH Windows instances benefit users who are looking to mitigate major disruptions when a Windows server goes offline. You can use BYOH Windows instances as nodes on OpenShift Container Platform 4.8 and later versions.

You can [disable Windows container workloads](/docs/windows_containers/disabling-windows-container-workloads#disabling-windows-container-workloads) by performing the following:

- Uninstalling the Windows Machine Config Operator
- Deleting the Windows Machine Config Operator namespace
