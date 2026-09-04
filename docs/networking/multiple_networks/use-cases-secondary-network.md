---
title: Use cases for a secondary network
sidebar_position: 2
---

# Use cases for a secondary network {#use-cases-secondary-network}

<a id="use-cases-secondary-network"></a>

You can use a secondary network in situations where you require network isolation, including data plane and control plane separation.

Isolating network traffic is useful for the following performance and security reasons:

- Performance
  **Traffic management**: You can send traffic on two different planes to manage how much traffic is along each plane.
- Security
  **Network isolation**: You can send sensitive traffic onto a network plane that is managed specifically for security considerations, and you can separate private data that must not be shared between tenants or customers.

All of the pods in the cluster still use the cluster-wide default network to maintain connectivity across the cluster. Every pod has an `eth0` interface that is attached to the cluster-wide pod network. You can view the interfaces for a pod by using the `oc exec -it <pod_name> \-- ip a` command. If you add secondary network interfaces that use the Multus Container Network Interface (CNI). These secondary networks are named `net1`, `net2`, and so on.

To attach secondary network interfaces to a pod, you must create configurations that define how the interfaces are attached. Use either a `UserDefinedNetwork` custom resource (CR) or a `NetworkAttachmentDefinition` CR to specify each interface. A CNI configuration inside each of these CRs defines how that interface is created.

## Secondary networks in OpenShift Container Platform {#additional-networks-provided_use-cases-secondary-network}

OpenShift Container Platform provides the following CNI plugins for creating secondary networks in your cluster:

- **bridge**: To configure a bridge-based secondary network to allow pods on the same host to communicate with each other and the host, use the following procedure:
  - [Configure a bridge-based secondary network](/docs/networking/multiple_networks/secondary_networks/creating-secondary-nwt-other-cni#nw-multus-bridge-object_configuring-additional-network-cni)
- **bond-cni**: To provide a method for aggregating multiple network interfaces into a single logical *bonded* interface, use the following procedure:
  - [Configure a Bond CNI secondary network](/docs/networking/multiple_networks/secondary_networks/creating-secondary-nwt-other-cni#nw-multus-bond-cni-object_configuring-additional-network-cni)
- **host-device**: To allow pods access to a physical Ethernet network device on the host system, use the following procedure:
  - [Configure a host-device secondary network](/docs/networking/multiple_networks/secondary_networks/creating-secondary-nwt-other-cni#nw-multus-host-device-object_configuring-additional-network-cni)
- **ipvlan**: Allow pods on a host to communicate with other hosts and pods on those hosts, similar to a macvlan-based secondary network. Unlike a macvlan-based secondary network, each pod shares the same MAC address as the parent physical network interface. Use the following procedure:
  - [Configure an ipvlan-based secondary network](/docs/networking/multiple_networks/secondary_networks/creating-secondary-nwt-other-cni#nw-multus-ipvlan-object_configuring-additional-network-cni)
- **VLAN**: To allow VLAN-based network isolation and connectivity for pods, use the following procedure:
  - [Configure a VLAN-based secondary network](/docs/networking/multiple_networks/secondary_networks/creating-secondary-nwt-other-cni#nw-multus-vlan-object_configuring-additional-network-cni)
- **macvlan**: To allow pods on a host to communicate with other hosts and pods on those hosts by using a physical network interface. Each pod that is attached to a macvlan-based secondary network is provided a unique MAC address:
  - [Configure a macvlan-based secondary network](/docs/networking/multiple_networks/secondary_networks/creating-secondary-nwt-other-cni#nw-multus-macvlan-object_configuring-additional-network-cni)
- **TAP**: A TAP device enables user space programs to send and receive network packets. To create a TAP device inside the container namespace, use the following procedure:
  - [Configure a TAP-based secondary network](/docs/networking/multiple_networks/secondary_networks/creating-secondary-nwt-other-cni#nw-multus-tap-object_configuring-additional-network-cni)
- **SR-IOV**: To allow pods to attach to a virtual function (VF) interface on SR-IOV capable hardware on the host system.
  - [Configure an SR-IOV based secondary network](/docs/networking/hardware_networks/about-sriov#about-sriov)
- **route-override**: To allow pods to override and set routes, use the following procedure:
  - [Configure a `route-override` based secondary network](/docs/networking/multiple_networks/secondary_networks/creating-secondary-nwt-other-cni#nw-route-override-cni_configuring-additional-network-cni)

## UserDefinedNetwork and NetworkAttachmentDefinition support matrix {#support-matrix-for-udn-nad_use-cases-secondary-network}

You can use user defined networks and network attachment definitions to define and configure customized networks for your needs.

By creating `UserDefinedNetwork` and `NetworkAttachmentDefinition` custom resources (CRs), cluster administrators can complete the following tasks:

- Create customizable network configurations
- Define their own network topologies
- Ensure network isolation
- Manage IP addressing for workloads
- Configure advanced network features

By creating a `ClusterUserDefinedNetwork` CR, administrators can create and define secondary networks that span multiple namespaces at the cluster level.

User-defined networks and network attachment definitions can serve as both the primary and secondary network interface, and each support `layer2` and `layer3` topologies.

:::note

As of OpenShift Container Platform 4.19, the use of the `Localnet` topology by `ClusterUserDefinedNetwork` CRs is generally available. This configuration is the preferred method for connecting physical networks to virtual networks. Or, you can use the `NetworkAttachmentDefinition` CR to create secondary networks with `Localnet` topologies.

:::

The following section highlights the supported features of the `UserDefinedNetwork` and `NetworkAttachmentDefinition` CRs when used as either the primary or secondary network. A separate table for the `ClusterUserDefinedNetwork` CR is also included.

**Primary network support matrix for **`UserDefinedNetwork`** and **`NetworkAttachmentDefinition`** CRs**

<table>
<thead>
<tr>
  <th>Network feature ^</th>
  <th>Layer2 topology ^</th>
  <th>Layer3 topology</th>
</tr>
</thead>
<tbody>
<tr>
  <td>east-west traffic</td>
  <td>&#10003;</td>
  <td>&#10003;</td>
</tr>
<tr>
  <td>north-south traffic</td>
  <td>&#10003;</td>
  <td>&#10003;</td>
</tr>
<tr>
  <td>Persistent IPs</td>
  <td>&#10003;</td>
  <td>X</td>
</tr>
<tr>
  <td>Services</td>
  <td>&#10003;</td>
  <td>&#10003;</td>
</tr>
<tr>
  <td>Routes</td>
  <td>X</td>
  <td>X</td>
</tr>
<tr>
  <td><code>EgressIP</code> resource</td>
  <td>&#10003;</td>
  <td>&#10003;</td>
</tr>
<tr>
  <td>Multicast</td>
  <td>X</td>
  <td>&#10003;</td>
</tr>
<tr>
  <td><code>NetworkPolicy</code> resource</td>
  <td>&#10003;</td>
  <td>&#10003;</td>
</tr>
<tr>
  <td><code>MultinetworkPolicy</code> resource</td>
  <td>X</td>
  <td>X</td>
</tr>
</tbody>
</table>

where:

Multicast
: Must be enabled in the namespace, and it is only available between OVN-Kubernetes network pods. For more information, see "About multicast".

`NetworkPolicy` resource
: When creating a `ClusterUserDefinedNetwork` CR with a primary network type, network policies must be created *after* the `UserDefinedNetwork` CR.

**Secondary network support matrix for **`UserDefinedNetwork`** and **`NetworkAttachmentDefinition`** CRs**

<table>
<thead>
<tr>
  <th>Network feature ^</th>
  <th>Layer2 topology ^</th>
  <th>Layer3 topology ^</th>
  <th>Localnet topology</th>
</tr>
</thead>
<tbody>
<tr>
  <td>east-west traffic</td>
  <td>&#10003;</td>
  <td>&#10003;</td>
  <td>&#10003; (<code>NetworkAttachmentDefinition</code> CR only)</td>
</tr>
<tr>
  <td>north-south traffic</td>
  <td>X</td>
  <td>X</td>
  <td>&#10003; (<code>NetworkAttachmentDefinition</code> CR only)</td>
</tr>
<tr>
  <td>Persistent IPs</td>
  <td>&#10003;</td>
  <td>X</td>
  <td>&#10003; (<code>NetworkAttachmentDefinition</code> CR only)</td>
</tr>
<tr>
  <td>Services</td>
  <td>X</td>
  <td>X</td>
  <td>X</td>
</tr>
<tr>
  <td>Routes</td>
  <td>X</td>
  <td>X</td>
  <td>X</td>
</tr>
<tr>
  <td><code>EgressIP</code> resource</td>
  <td>X</td>
  <td>X</td>
  <td>X</td>
</tr>
<tr>
  <td>Multicast</td>
  <td>X</td>
  <td>X</td>
  <td>X</td>
</tr>
<tr>
  <td><code>NetworkPolicy</code> resource</td>
  <td>X</td>
  <td>X</td>
  <td>X</td>
</tr>
<tr>
  <td><code>MultinetworkPolicy</code> resource</td>
  <td>&#10003;</td>
  <td>&#10003;</td>
  <td>&#10003; (<code>NetworkAttachmentDefinition</code> CR only)</td>
</tr>
</tbody>
</table>

The Localnet topology is unavailable for use with the `UserDefinedNetwork` CR. It is only supported on secondary networks for `NetworkAttachmentDefinition` CRs.

**Support matrix for **`ClusterUserDefinedNetwork`** CRs**

<table>
<thead>
<tr>
  <th>Network feature ^</th>
  <th>Layer2 topology ^</th>
  <th>Layer3 topology ^</th>
  <th>Localnet topology</th>
</tr>
</thead>
<tbody>
<tr>
  <td>east-west traffic</td>
  <td>&#10003;</td>
  <td>&#10003;</td>
  <td>&#10003;</td>
</tr>
<tr>
  <td>north-south traffic</td>
  <td>&#10003;</td>
  <td>&#10003;</td>
  <td>&#10003;</td>
</tr>
<tr>
  <td>Persistent IPs</td>
  <td>&#10003;</td>
  <td>X</td>
  <td>&#10003;</td>
</tr>
<tr>
  <td>Services</td>
  <td>&#10003;</td>
  <td>&#10003;</td>
  <td></td>
</tr>
<tr>
  <td>Routes</td>
  <td>X</td>
  <td>X</td>
  <td></td>
</tr>
<tr>
  <td><code>EgressIP</code> resource</td>
  <td>&#10003;</td>
  <td>&#10003;</td>
  <td></td>
</tr>
<tr>
  <td>Multicast</td>
  <td>X</td>
  <td>&#10003;</td>
  <td></td>
</tr>
<tr>
  <td><code>MultinetworkPolicy</code> resource</td>
  <td>X</td>
  <td>X</td>
  <td>&#10003;</td>
</tr>
<tr>
  <td><code>NetworkPolicy</code> resource</td>
  <td>&#10003;</td>
  <td>&#10003;</td>
  <td></td>
</tr>
</tbody>
</table>

where:

Multicast
: must be enabled in the namespace, and it is only available between OVN-Kubernetes network pods. For more information, see "About multicast".

`NetworkPolicy` resource
: When creating a `ClusterUserDefinedNetwork` CR with a primary network type, network policies must be created *after* the `UserDefinedNetwork` CR.

**Additional resources**

- [Enabling multicast for a project](/docs/networking/ovn_kubernetes_network_provider/enabling-multicast#nw-ovn-kubernetes-enabling-multicast)
