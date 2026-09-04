---
title: About BGP EVPN for primary cluster user-defined networks
sidebar_position: 1
---

# About BGP EVPN for primary cluster user-defined networks {#about-bgp-evpn-user-defined-networks}

<a id="about-bgp-evpn-user-defined-networks"></a>

Ethernet Virtual Private Network (EVPN) extends OVN-Kubernetes Border Gateway Protocol (BGP) support to transport primary cluster user-defined network (CUDN) traffic across VXLAN overlays, providing seamless, isolated layer 2 and layer 3 connectivity to the data center network.

## Overview of BGP EVPN with OVN-Kubernetes {#nw-bgp-evpn-about_about-bgp-evpn-user-defined-networks}

Border Gateway Protocol Ethernet Virtual Private Networking (BGP EVPN) is a standards-based control plane that exchanges layer 2 and layer 3 overlay network reachability within the data center. Enabling this feature on OpenShift Container Platform allows a `ClusterUserDefinedNetwork` (CUDN) overlay network to use the EVPN control plane for deeper integration with the data center network.

Before EVPN support for primary cluster user-defined networks (CUDNs), VRF-lite was the only way to advertise CUDNs into one or more Virtual Routing and Forwarding (VRF) tables while keeping them isolated on the network. However, that approach required a VLAN interface per VRF and a separate BGP session for each VRF.

BGP EVPN simplifies that workflow: you avoid extra VLAN interfaces and per-VRF BGP sessions by using a single multiprotocol BGP (MP-BGP) session toward the fabric.

EVPN defines the following two VPN types that you can deploy with primary CUDNs when you attach the overlay to the fabric:

MAC-VRF
: Stretches a layer 2 segment across the EVPN fabric for east-west bridged traffic and workloads such as VM live migration.

IP-VRF
: Provides layer 3 routing across the EVPN fabric for routed north-south traffic.

Enabling EVPN transport on a primary cluster user-defined network (CUDN) implements EVPN using VXLAN overlay. FRR implements the BGP EVPN control plane and integrates with Linux network devices for the VXLAN data plane.

This design does not model the cluster default pod network as an EVPN VPN. EVPN applies only to primary CUDNs that you configure for EVPN transport. EVPN on primary CUDNs requires and builds on the OVN-Kubernetes route advertisements feature. You use `RouteAdvertisements` custom resources to associate fabric-facing FRR configuration with labeled primary CUDNs that use EVPN transport. For more information, see "About route advertisements".

### Benefits of BGP EVPN with primary CUDNs {#bgp-evpn-primary-use-cases_about-bgp-evpn-user-defined-networks}

BGP EVPN extends the isolated primary network in each namespace to an external BGP EVPN fabric, improving segmentation between tenants and external sites while simplifying network operations by aligning CUDNs with fabric route targets and shared uplinks instead of complex per-node routes and host-specific wiring.

Primary CUDNs that use BGP EVPN offer the following benefits:

Fabric integration
: Map tenant networks to the same route-target and external virtual routing and forwarding (VRF) segments that you already use on an external BGP EVPN fabric.

Layer 2 stretch for mobility
: Extend a layer 2 segment into the cluster so that virtual machines or pods can keep addresses stable. EVPN MAC mobility can help when workloads move between nodes.

Simpler attachment than per-node local wiring
: Reduce dependence on host-specific local attachment patterns when the fabric can carry many tenant VNIs over shared uplinks.

Isolation for overlapping tenant networks
: Connect primary CUDNs that use overlapping address space to different external EVPN segments while preserving isolation between tenants.

### Platform support and feature compatibility {#bgp-evpn-supported-platforms_about-bgp-evpn-user-defined-networks}

BGP EVPN with primary CUDNs is supported only on bare-metal clusters.

The following table lists the supported and unsupported features when EVPN is enabled on primary CUDNs.

**BGP EVPN with primary CUDNs feature compatibility**

<table>
<thead>
<tr>
  <th>Network feature ^</th>
  <th>Supported</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Egress firewall</td>
  <td>Supported</td>
</tr>
<tr>
  <td>Egress QoS</td>
  <td>Supported</td>
</tr>
<tr>
  <td>Network QoS</td>
  <td>Supported</td>
</tr>
<tr>
  <td>Network Policy</td>
  <td>Supported</td>
</tr>
<tr>
  <td>Services (Cluster IP)</td>
  <td>Supported</td>
</tr>
<tr>
  <td>Services (NodePort, External IP, LoadBalancer)</td>
  <td>Limited<sup>[1]</sup></td>
</tr>
<tr>
  <td>Multicast (MAC-VRF)</td>
  <td>Supported</td>
</tr>
<tr>
  <td>Multicast (IP-VRF)</td>
  <td>Unsupported</td>
</tr>
<tr>
  <td>Multiple External Gateways (MEG)</td>
  <td>Unsupported</td>
</tr>
<tr>
  <td>EgressIP</td>
  <td>Unsupported</td>
</tr>
<tr>
  <td>Egress Service</td>
  <td>Unsupported</td>
</tr>
<tr>
  <td>IPsec</td>
  <td>Unsupported</td>
</tr>
</tbody>
</table>

1. For `Service` objects (NodePort, External IP, and LoadBalancer), the node IP, external IP, and LoadBalancer IP must be reachable on the EVPN fabric at the appropriate cluster nodes. OVN-Kubernetes does not advertise those addresses onto the fabric. Service traffic behaves normally when the fabric delivers packets to the nodes.

### Limitations {#bgp-evpn-limitations_about-bgp-evpn-user-defined-networks}

The following limitations apply to BGP EVPN with primary CUDNs:

- BGP EVPN with primary CUDNs is supported only on bare-metal clusters.
- BGP EVPN with primary CUDNs requires setting `gatewayConfig.routingViaHost` to `true` and `gatewayConfig.ipForwarding` to `Global` in the Cluster Network Operator configuration. For more information, see "Configuring a gateway".
- A maximum of 4094 combined MAC-VRF and IP-VRF instances per `VTEP` applies because OVN-Kubernetes uses one shared VXLAN device per `VTEP`, maps VLAN IDs to VNIs on that device, and the Linux bridge VLAN space is limited to 4094.
- Only IPv4 VTEP addresses are supported. IPv6 VTEPs are not supported because of limitations in FRR, which implements the BGP EVPN control plane.
- The VXLAN destination UDP port is 4789. Customizing the UDP port is not supported.

## Enabling BGP EVPN on a primary ClusterUserDefinedNetwork CR {#nw-enabling-bgp-evpn-primary-cudn_about-bgp-evpn-user-defined-networks}

To connect primary `ClusterUserDefinedNetwork` (CUDN) custom resource (CR) traffic to an external BGP EVPN fabric in OpenShift Container Platform, you create `FRRConfiguration`, `VTEP`, `RouteAdvertisements`, and `ClusterUserDefinedNetwork` resources. Use these resources to establish BGP EVPN toward peers on your provider network and to advertise labeled CUDNs that use EVPN transport.

**Prerequisites**

- You have enabled route advertisements in the Cluster Network Operator configuration. For more information, see "Enabling route advertisements".
- You have set `gatewayConfig.routingViaHost` to `true` and `gatewayConfig.ipForwarding` to `Global` in the Cluster Network Operator configuration. BGP EVPN for primary CUDNs is only supported with this gateway configuration. For more information, see "Configuring a gateway".
- You have logged in to the OpenShift Container Platform cluster as an administrator.
- You have the autonomous system numbers (ASNs), BGP neighbor addresses, expected virtual network identifiers (VNIs), and route targets if you set them explicitly. If you omit route targets, the implementation derives default values from your ASN and VNI.
  - VNIs must be unique and unused across multiple IP-VRF or MAC-VRF instances.
- For Unmanaged `VTEP`:
  - The Kubernetes NMState Operator is available if you choose the NNCP option for Unmanaged VTEP addresses.

**Procedure**

1. Create an `FRRConfiguration` object that establishes BGP peering toward your provider network. The manifest must define at least one BGP router in the default VRF that will act as the EVPN underlay. Multiple routers can be defined for redundancy. OVN-Kubernetes reads neighbors from that router to enable EVPN toward the fabric.
   1. Use the following example as a template when creating the `FRRConfiguration` object. Use a descriptive label so `RouteAdvertisements` can select this configuration in a subsequent step.
      ```yaml
      apiVersion: frrk8s.metallb.io/v1beta1
      kind: FRRConfiguration
      metadata:
        name: <frrconfiguration_evpn_name>
        namespace: openshift-frr-k8s
        labels:
          <frr_label_key>: <frr_label_value>
      spec:
        nodeSelector: {}
        bgp:
          routers:
            - asn: <cluster_asn>
              neighbors:
                - address: <peer_address>
                  asn: <peer_asn>
      ```

      where:

      <dl>
      <dt><code>&lt;frrconfiguration_evpn_name&gt;</code></dt>
      <dd>Specifies the name of the <code>FRRConfiguration</code> object.</dd>
      <dt><code>&lt;frr_label_key&gt;</code> and <code>&lt;frr_label_value&gt;</code></dt>
      <dd>Specifies a label key and value on the <code>FRRConfiguration</code> object that must match <code>frrConfigurationSelector.matchLabels</code> in the <code>RouteAdvertisements</code> object.</dd>
      <dt><code>spec.nodeSelector</code></dt>
      <dd>Specifies a <code>nodeSelector</code> so the configuration applies to the nodes where you want FRR to establish BGP toward the fabric.</dd>
      <dt><code>&lt;cluster_asn&gt;</code></dt>
      <dd>Specifies the autonomous system number (ASN) of the OpenShift Container Platform cluster.</dd>
      <dt><code>&lt;peer_address&gt;</code></dt>
      <dd>Specifies the address of the BGP neighbor on the provider fabric.</dd>
      <dt><code>&lt;peer_asn&gt;</code></dt>
      <dd>Specifies the ASN of the BGP neighbor on the provider fabric.</dd>
      </dl>
   2. Apply the `FRRConfiguration` object by entering the following command:
      ```terminal
      $ oc apply -f <frrconfiguration_evpn_name>.yaml
      ```
2. Create a VXLAN tunnel endpoint, or `VTEP`, object in `Unmanaged` mode. The `VTEP` custom resource defines VTEP IPs.
   1. Use the following example as a template when creating the `VTEP` object:
      ```yaml
      apiVersion: k8s.ovn.org/v1
      kind: VTEP
      metadata:
        name: <evpn_vtep_name>
      spec:
        mode: Unmanaged
        cidrs:
          - <vtep_cidr>
      ```

      where:

      <dl>
      <dt><code>&lt;evpn_vtep_name&gt;</code></dt>
      <dd>Specifies the name of the <code>VTEP</code> object.</dd>
      <dt><code>Unmanaged</code></dt>
      <dd>Must be set to <code>Unmanaged</code> mode. <code>Managed</code> mode is unsupported for this release.</dd>
      <dt><code>&lt;vtep_cidr&gt;</code></dt>
      <dd>Specifies an IPv4 prefix that includes the VTEP primary address on each participating node. On every such node, configure one IPv4 address from this range as the primary address on an interface. OVN-Kubernetes discovers that address and uses it as the VTEP IP for that node. For how the address is used and how to configure the interface, see the following note.</dd>
      </dl>

      :::note

      In `Unmanaged` mode, OVN-Kubernetes does not allocate a VTEP address. Consider the following when configuring the VTEP IP:

      - Each node needs one IPv4 address from `<vtep_cidr>` configured as the primary address of an interface on that node.
      - OVN-Kubernetes discovers the IPv4 address configured on the interface and uses it as the VTEP IP for that node. OVN-Kubernetes then advertises the VTEP IP on the BGP EVPN underlay so that it is reachable from other nodes.
      - Avoid using an interface associated with a physical link carrier when using redundant BGP peering. Instead, use a dummy interface where the IP is configured as the primary address.

      See the following optional step to configure a dummy interface and assign a primary IPv4 address from `<vtep_cidr>` by using the Kubernetes NMState Operator.

      :::

      1. Optional. Configure a dummy interface with a `NodeNetworkConfigurationPolicy` (NNCP) object from the Kubernetes NMState Operator. Assign a primary IPv4 address on it that falls within the VTEP CIDR. Use the following manifest as a template.
         ```yaml
         apiVersion: nmstate.io/v1
         kind: NodeNetworkConfigurationPolicy
         metadata:
           name: <nncp_evpn_vtep_name>
         spec:
           nodeSelector: {}
           desiredState:
             interfaces:
               - name: <evpn_vtep_dummy_name>
                 type: dummy
                 state: up
                 ipv4:
                   enabled: true
                   address:
                     - ip: <vtep_ipv4>
                       prefix-length: 32
                   dhcp: false
                 ipv6:
                   enabled: false
         ```

         where:

         <dl>
         <dt><code>&lt;nncp_evpn_vtep_name&gt;</code></dt>
         <dd>Specifies the name of the <code>NodeNetworkConfigurationPolicy</code> object.</dd>
         <dt><code>spec.nodeSelector</code></dt>
         <dd>Specifies a <code>nodeSelector</code> so the policy applies to the node or nodes you expect to host EVPN workloads.</dd>
         <dt><code>&lt;evpn_vtep_dummy_name&gt;</code></dt>
         <dd>Specifies the name for the dummy device on the node.</dd>
         <dt><code>&lt;vtep_ipv4&gt;</code></dt>
         <dd>Specifies an IPv4 address from <code>&lt;vtep_cidr&gt;</code> for this node. Use a different address on each node.</dd>
         </dl>

         For information about applying and verifying the `NodeNetworkConfigurationPolicy` object, see "Managing the NodeNetworkConfigurationPolicy manifest file".
   2. Apply the `VTEP` object by entering the following command:
      ```terminal
      $ oc apply -f <evpn_vtep_name>.yaml
      ```
   3. Verify that the `VTEP` CR is created and accepted by entering the following command. A healthy VTEP has a valid VTEP IP and reports `Accepted: True` and `Reason: Allocated`.
      ```terminal
      $ oc get vtep
      ```

      ```terminal title="Example output"
      NAME                ACCEPTED   REASON
      <evpn_vtep_name>    True       Allocated
      ```

      If `Accepted` is `False`, check node annotations.
3. Create a `RouteAdvertisements` object.
   1. Use the following example as a template when creating the `RouteAdvertisements` object. Ensure that the `RouteAdvertisements` object selects a labeled `FRRConfiguration` object and labeled primary `ClusterUserDefinedNetwork` objects so that the pod network is advertised for EVPN.
      ```yaml
      apiVersion: k8s.ovn.org/v1
      kind: RouteAdvertisements
      metadata:
        name: <routeadvertisements_evpn_name>
      spec:
        targetVRF: auto
        advertisements:
          - PodNetwork
        nodeSelector: {}
        frrConfigurationSelector:
          matchLabels:
            <frr_label_key>: <frr_label_value>
        networkSelectors:
          - networkSelectionType: ClusterUserDefinedNetworks
            clusterUserDefinedNetworkSelector:
              networkSelector:
                matchLabels:
                  <cudn_label_key>: <cudn_label_value>
      ```

      where:

      <dl>
      <dt><code>spec.targetVRF</code></dt>
      <dd>Specifies the target VRF. Each CUDN is advertised in its own IP-VRF by design.</dd>
      <dt><code>spec.advertisements</code></dt>
      <dd>Specifies the advertisements to configure. Only pod IP advertisements are supported. Egress IP advertisements are not supported for EVPN.</dd>
      <dt><code>spec.nodeSelector</code></dt>
      <dd>Specifies the nodes from which advertisements are propagated. Pod IP advertisements are supported only from all nodes. This limitation is not specific to EVPN and applies to <code>RouteAdvertisements</code> objects.</dd>
      <dt><code>&lt;routeadvertisements_evpn_name&gt;</code></dt>
      <dd>Specifies the name of the <code>RouteAdvertisements</code> object.</dd>
      <dt><code>&lt;frr_label_key&gt;</code> and <code>&lt;frr_label_value&gt;</code></dt>
      <dd>Specifies a label key and value on the <code>FRRConfiguration</code> object that must match <code>frrConfigurationSelector.matchLabels</code> in the <code>RouteAdvertisements</code> object. The selected <code>FRRConfiguration</code> must define at least a default VRF BGP router; OVN-Kubernetes uses neighbors from that router to enable EVPN for those peers.</dd>
      <dt><code>&lt;cudn_label_key&gt;</code> and <code>&lt;cudn_label_value&gt;</code></dt>
      <dd>Specifies a label key and value on the <code>ClusterUserDefinedNetwork</code> object that must match <code>networkSelectors.clusterUserDefinedNetworkSelector.networkSelector.matchLabels</code> in the <code>RouteAdvertisements</code> object. For example, the key <code>evpn</code> with the value <code>enabled</code> selects every CUDN labeled <code>evpn: enabled</code>.</dd>
      </dl>

      For IP-VRF primary CUDNs, if the `FRRConfiguration` defines a VRF-specific BGP router whose name matches the CUDN name, the IP-VRF EVPN router is derived from that router. If that router does not exist, the IP-VRF EVPN router is derived from the underlay (default VRF) router.
   2. Apply the `RouteAdvertisements` object by entering the following command:
      ```terminal
      $ oc apply -f <routeadvertisements_evpn_name>.yaml
      ```
   3. Verify that the `RouteAdvertisements` CR is created and accepted by entering the following command. A healthy `RouteAdvertisements` CR reports `Accepted: True` and `Reason: Accepted`.
      ```terminal
      $ oc get routeadvertisements -o \
          custom-columns='NAME:.metadata.name,ACCEPTED:.status.conditions[?(@.type=="Accepted")].status,REASON:.status.conditions[?(@.type=="Accepted")].reason'
      ```

      ```terminal title="Example output"
      NAME                               ACCEPTED   REASON
      <routeadvertisements_evpn_name>    True       Accepted
      ```

      If `Accepted` is `False`, check that the `networkSelectors` field matches an existing `ClusterUserDefinedNetwork` resource, that the `frrConfigurationSelector` object matches an existing `FRRConfiguration` resource, and that the `FRRConfiguration` contains at least a default VRF router with neighbors.
4. Create a Layer 2 or Layer 3 primary `ClusterUserDefinedNetwork` object.
   :::warning

   You cannot retroactively enable EVPN on an existing `ClusterUserDefinedNetwork` object whose specification is immutable. You must create a new `ClusterUserDefinedNetwork` object for EVPN workloads instead.

   :::

   1. Create a Layer 2 primary CUDN object across the EVPN network as a MAC-VRF. Use the following example as a template:
      ```yaml
      apiVersion: k8s.ovn.org/v1
      kind: ClusterUserDefinedNetwork
      metadata:
        name: <l2_cudn_name>
        labels:
          <cudn_label_key>: <cudn_label_value>
      spec:
        namespaceSelector:
          matchLabels:
            kubernetes.io/metadata.name: <namespace_label_value>
        network:
          topology: Layer2
          layer2:
            role: Primary
            subnets:
              - <cudn_subnet>
          transport: EVPN
          evpn:
            vtep: <evpn_vtep_name>
            macVRF:
              vni: <mac_vrf_vni>
              routeTarget: <mac_vrf_rt>
      ```

      where:

      <dl>
      <dt><code>&lt;l2_cudn_name&gt;</code></dt>
      <dd>Specifies the name of the Layer 2 <code>ClusterUserDefinedNetwork</code> object.</dd>
      <dt><code>&lt;cudn_label_key&gt;</code> and <code>&lt;cudn_label_value&gt;</code></dt>
      <dd>Specifies a label key and value on the CUDN that must match <code>networkSelectors</code> in the <code>RouteAdvertisements</code> object.</dd>
      <dt><code>&lt;namespace_label_value&gt;</code></dt>
      <dd>Specifies the value of the label so that the CUDN applies to the namespace or namespaces that you expect to host EVPN workloads.</dd>
      <dt><code>&lt;cudn_subnet&gt;</code></dt>
      <dd>Specifies the subnet for the CUDN, for example, <code>10.0.10.0/24</code>.</dd>
      <dt><code>&lt;evpn_vtep_name&gt;</code></dt>
      <dd>Specifies the <code>metadata.name</code> of the <code>VTEP</code> object that this CUDN uses for EVPN transport. It must match the name you set when you created the <code>VTEP</code> object.</dd>
      <dt><code>&lt;mac_vrf_vni&gt;</code></dt>
      <dd>Specifies the VNI for the MAC VRF, for example, <code>100</code>.</dd>
      <dt><code>&lt;mac_vrf_rt&gt;</code></dt>
      <dd>Optional. Specifies the route target for the MAC VRF, for example, <code>65000:100</code>.</dd>
      </dl>
   2. Optional. Add an IP-VRF on top of the MAC-VRF. This enables Layer 3 routing for the same network, allowing pods to reach external routed destinations via the IP-VRF while maintaining Layer 2 reachability via the MAC VRF. Use the following example as a template:
      ```yaml
      # ...
          evpn:
            vtep: <evpn_vtep_name>
            macVRF:
              vni: <mac_vrf_vni>
              routeTarget: <mac_vrf_rt>
            ipVRF:
              vni: <ip_vrf_vni>
              routeTarget: <ip_vrf_rt>
      # ...
      ```

      where:

      <dl>
      <dt><code>&lt;ip_vrf_vni&gt;</code></dt>
      <dd>Specifies the VNI for the IP VRF, for example, <code>101</code>.</dd>
      <dt><code>&lt;ip_vrf_rt&gt;</code></dt>
      <dd>Optional. Specifies the route target for the IP VRF, for example, <code>65000:200</code>.</dd>
      </dl>
   3. Create a Layer 3 primary CUDN object that uses routing via the IP-VRF. In the following Layer 3 topology, each node maintains its own local Layer 2 domain (per-node subnets), and inter-node communication is handled via EVPN Type 5 routes to advertise these IP prefixes across the fabric. Use the following example as a template:
      ```yaml
      apiVersion: k8s.ovn.org/v1
      kind: ClusterUserDefinedNetwork
      metadata:
        name: <l3_cudn_name>
        labels:
          <cudn_label_key>: <cudn_label_value>
      spec:
        namespaceSelector:
          matchLabels:
            kubernetes.io/metadata.name: <namespace_label_value>
        network:
          topology: Layer3
          layer3:
            role: Primary
            subnets:
              - cidr: <cudn_subnet>
                hostSubnet: 24
          transport: EVPN
          evpn:
            vtep: <evpn_vtep_name>
            ipVRF:
              vni: <ip_vrf_vni>
              routeTarget: <ip_vrf_rt>
      ```

      where:

      <dl>
      <dt><code>&lt;l3_cudn_name&gt;</code></dt>
      <dd>Specifies the name of the Layer 3 <code>ClusterUserDefinedNetwork</code> object.</dd>
      <dt><code>&lt;cudn_label_key&gt;</code> and <code>&lt;cudn_label_value&gt;</code></dt>
      <dd>Specifies a label key and value on the CUDN that must match <code>networkSelectors</code> in the <code>RouteAdvertisements</code> object.</dd>
      <dt><code>&lt;namespace_label_value&gt;</code></dt>
      <dd>Specifies the value of the label so that the CUDN applies to the namespace or namespaces that you expect to host EVPN workloads.</dd>
      <dt><code>&lt;cudn_subnet&gt;</code></dt>
      <dd>Specifies the subnet for the CUDN, for example, <code>10.0.20.0/16</code>.</dd>
      <dt><code>&lt;evpn_vtep_name&gt;</code></dt>
      <dd>Specifies the name of the <code>VTEP</code> object that this CUDN uses for EVPN transport. It must match the name you set when you created the <code>VTEP</code> object.</dd>
      <dt><code>&lt;ip_vrf_vni&gt;</code></dt>
      <dd>Specifies the VNI for the IP VRF, for example, <code>200</code>.</dd>
      <dt><code>&lt;ip_vrf_rt&gt;</code></dt>
      <dd>Optional. Specifies the route target for the IP VRF, for example, <code>65000:101</code>.</dd>
      </dl>
   4. Apply the Layer 2 or Layer 3 `ClusterUserDefinedNetwork` object by entering the following command:
      ```terminal
      $ oc apply -f <clusteruserdefinednetwork_name>.yaml
      ```
   5. Verify that the `ClusterUserDefinedNetwork` CR is created and accepted by entering the following command. A healthy `ClusterUserDefinedNetwork` CR reports `Accepted: True` and `Reason: EVPNTransportAccepted`.
      Check the `ClusterUserDefinedNetwork` status by entering the following command:

      ```terminal
      $ oc get clusteruserdefinednetwork <clusteruserdefinednetwork_name> -o \
          custom-columns='NAME:.metadata.name,ACCEPTED:.status.conditions[?(@.type=="clusteruserdefinednetwork")].status,REASON:.status.conditions[?(@.type=="clusteruserdefinednetwork")].reason'
      ```

      ```terminal title="Example output"
      NAME                                ACCEPTED   REASON
      <clusteruserdefinednetwork_name>    True       EVPNTransportAccepted
      ```

      If `Accepted` is `False`, check that the referenced `VTEP` CR exists and is itself accepted.

**Verification**

- Ensure that pods in a selected namespace can reach peers on the same CUDN and, where your design allows, endpoints on the external EVPN fabric.

**Additional resources**

- [About BGP routing](/docs/networking/advanced_networking/bgp_routing/about-bgp-routing#about-bgp-routing)
- [About route advertisements](/docs/networking/advanced_networking/route_advertisements/about-route-advertisements#about-route-advertisements)
- [Enabling route advertisements](/docs/networking/advanced_networking/route_advertisements/enabling-route-advertisements#nw-route-advertisements-enable_enabling-route-advertisements)
- [Configuring a gateway](/docs/networking/ovn_kubernetes_network_provider/configuring-gateway#configuring-gateway)
- [Managing the NodeNetworkConfigurationPolicy manifest file](/docs/networking/k8s_nmstate/k8s-nmstate-updating-node-network-config#virt-manage-nncp-cli_k8s-nmstate-updating-node-network-config)
- [About user-defined networks](/docs/networking/multiple_networks/primary_networks/about-user-defined-networks#about-user-defined-networks)
- [ClusterUserDefinedNetwork k8s.ovn.org/v1](/docs/rest_api/network_apis/clusteruserdefinednetwork-k8s-ovn-org-v1#specifications)
- [FRRouting User Guide: BGP](https://docs.frrouting.org/en/latest/bgp.html)
- [FRRouting User Guide: EVPN](https://docs.frrouting.org/en/latest/evpn.html)
