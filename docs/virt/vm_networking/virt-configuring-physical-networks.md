---
title: Configuring physical networks
sidebar_position: 15
---

# Configuring physical networks

<a id="configuring-physical-networks"></a>

As an OpenShift Container Platform administrator, you can create or configure a physical network in the OpenShift Container Platform web console without using the node network configuration policy (NNCP) page.

:::note

The **Physical networks** page is available when the NMState console plugin is installed. When you use the **Physical networks** page in the web console, the NNCP is generated automatically. If you need more flexibility or require complex settings, use the NNCP page.

:::

## Creating a physical network by using the OpenShift Container Platform web console {#creating-a-physical-network_configuring-physical-networks}

You can create physical networks for OpenShift Virtualization using the OpenShift Container Platform web console to create a network with direct layer 2 connectivity to your data center.

**Prerequisites**

- You are logged in to the OpenShift Container Platform web console as a user with `cluster-admin` permissions.

**Procedure**

1. In the OpenShift Container Platform web console, go to **Networking** → **Physical networks**.
2. Click **Create network**. The **Network configuration wizard** is displayed.
3. On the **Network identity** page, enter a name for your network.
4. On the **Nodes configuration** page, select either **Apply to all nodes on the cluster** or **Apply to specific subsets of nodes using the nodes selector**.
   :::note

   If you select specific nodes to apply the network to, you can view the matching nodes list to ensure the selection is correct. A validation error is displayed if the selected nodes overlap with another configuration associated with the same network.

   :::
5. On the **Uplink connection** page, select the network interface that you want to connect to the physical network:

Default node network
: Uses the default node network to access the outside physical network.

A single interface
: Select a specific physical network interface from the list.

  :::warning

  If the selected secondary interface has an IP address on some of the nodes, using it removes the IP address and might disrupt network services.

  :::

  Bonding interface
  : Configures bonded network interfaces to achieve resilience and higher throughput.

1. Enter a **Bonding name**.
2. Select the **Network interfaces** to bond.
3. Select the **Aggregation mode** from the drop-down menu.
4. On the **Settings** page, enter a **Bridge name** and set the **Maximum Transmission Unit (MTU)**.
5. Review the configuration details.
6. Click **Create**.

**Verification**

1. In the OpenShift Container Platform web console, go to **Networking** → **Physical networks**.
2. Locate your new network in the list.
3. Expand the network row to view the associated configurations. Verify that the **Enactment state** is **Available** and that the **Nodes** count matches your expectation.

## Expanding a OpenShift Container Platform physical network to include new nodes {#expanding-physical-network_configuring-physical-networks}

You can add one or more OpenShift Container Platform worker nodes to an existing physical network if you want to expand access to that network. Expanding a physical network creates a new configuration under the same logical physical network.

**Prerequisites**

- You are logged in to the OpenShift Container Platform web console as a user with `cluster-admin` permissions.

**Procedure**

1. In the OpenShift Container Platform web console, go to **Networking** → **Physical networks**.
2. Click the Options menu ![](/images/kebab.png "Options menu") next to the network that you want to edit.
3. Click **Configure nodes**. The **Network configuration wizard** is displayed.
   :::note

   The **Physical network name** is predefined. You cannot edit it during this process.

   :::
4. Click **Next**.
5. On the **Nodes configuration** page, select either **Apply to all nodes on the cluster** or **Apply to specific subsets of nodes using the nodes selector**.
   :::note

   If you select specific nodes, you can view the matching nodes list to ensure the selection is correct. A validation error is displayed if the selected nodes overlap with another configuration associated with the same network.

   :::
6. On the **Uplink connection** page, select the network interface to connect to the physical network:

Default node network
: Uses the default node network to access the outside physical network.

A single interface
: Select a specific physical network interface from the list.

  :::warning

  If the selected secondary interface has an IP address on some of the nodes, using removes the IP address and might disrupt network services.

  :::

  Bonding interface
  : Configures bonded network interfaces to achieve resilience and higher throughput.

  1. Enter a **Bonding name**.
  2. Select the **Network interfaces** to bond.

  :::note

  The system displays only the interfaces that all nodes have in common.

  :::

  1. Select the **Aggregation mode** from the drop down options.

1. On the **Settings** page, enter a **Bridge name** and set the **Maximum Transmission Unit (MTU)**.
2. Review the configuration details.
3. Click **Create**.

## Creating a virtual machine network from a physical network {#creating-vm-network-localnet_configuring-physical-networks}

If your use case does not permit the use of network address translation (NAT), you can give VMs direct layer 2 access by creating a VM network that uses a physical network.

**Prerequisites**

- You are logged in to the OpenShift Container Platform web console as a user with `cluster-admin` permissions.

**Procedure**

1. In the OpenShift Container Platform web console, go to **Networking** → **Physical networks**.
2. Click the Options menu ![](/images/kebab.png "Options menu") next to the network that you want to edit.
3. Click **Create a virtual machines network using this physical network**. The **Create virtual machine network** wizard is displayed with the network name populated.
4. Select a **Physical network**.
5. Optional: Select **VLAN tagging** and enter a **VLAN ID**.
6. On the **Project mapping** page, define which projects can access this network.
7. Click **Create**.
