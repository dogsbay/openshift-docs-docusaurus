---
title: Managing symmetric routing with MetalLB
sidebar_position: 7
---

# Managing symmetric routing with MetalLB {#metallb-configure-return-traffic}

<a id="metallb-configure-return-traffic"></a>

As a cluster administrator, you can effectively manage traffic for pods behind a MetalLB load-balancer service with multiple host interfaces by implementing features from MetalLB, NMState, and OVN-Kubernetes. By combining these features in this context, you can provide symmetric routing, traffic segregation, and support clients on different networks with overlapping CIDR addresses.

To achieve this functionality, learn how to implement virtual routing and forwarding (VRF) instances with MetalLB, and configure egress services.

:::warning

Configuring symmetric traffic by using a VRF instance with MetalLB and an egress service is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.

For more information about the support scope of Red Hat Technology Preview features, see [Technology Preview Features Support Scope](https://access.redhat.com/support/offerings/techpreview/).

:::

## Challenges of managing symmetric routing with MetalLB {#challenges-of-managing-symmetric-routing-with-metallb_metallb-configure-return-traffic}

To resolve network isolation and asymmetric routing challenges on multiple host interfaces, implement a configuration combining MetalLB, NMState, and OVN-Kubernetes. This solution ensures symmetric routing and prevents overlapping CIDR addresses without requiring manual static route maintenance.

One option to ensure that return traffic reaches the correct client is to use static routes. However, with this solution, MetalLB cannot isolate the services and then announce each service through a different interface. Additionally, static routing requires manual configuration and requires maintenance if remote sites are added.

A further challenge of symmetric routing when implementing a MetalLB service is scenarios where external systems expect the source and destination IP address for an application to be the same. The default behavior for OpenShift Container Platform is to assign the IP address of the host network interface as the source IP address for traffic originating from pods. This is problematic with multiple host interfaces.

You can overcome these challenges by implementing a configuration that combines features from MetalLB, NMState, and OVN-Kubernetes.

## Overview of managing symmetric routing by using VRFs with MetalLB {#overview-of-managing-symmetric-routing-using-vrf-based-networks-with-metallb_metallb-configure-return-traffic}

You can overcome the challenges of implementing symmetric routing by using NMState to configure a VRF instance on a host, associating the VRF instance with a MetalLB `BGPPeer` resource, and configuring an egress service for egress traffic with OVN-Kubernetes.

**Figure 1. Network overview of managing symmetric routing by using VRFs with MetalLB**

![Network overview of managing symmetric routing by using VRFs with MetalLB](/images/357_OpenShift_MetalLB_VRF_0823.png)

The configuration process involves three stages:

1: Define a VRF and routing rules
: - Configure a `NodeNetworkConfigurationPolicy` custom resource (CR) to associate a VRF instance with a network interface.
  - Use the VRF routing table to direct ingress and egress traffic.

2: Link the VRF to a MetalLB `BGPPeer`
: - Configure a MetalLB `BGPPeer` resource to use the VRF instance on a network interface.
  - By associating the `BGPPeer` resource with the VRF instance, the designated network interface becomes the primary interface for the BGP session, and MetalLB advertises the services through this interface.

3: Configure an egress service
: - Configure an egress service to choose the network associated with the VRF instance for egress traffic.
  - Optional: Configure an egress service to use the IP address of the MetalLB load-balancer service as the source IP for egress traffic.

## Configure symmetric routing by using VRFs with MetalLB {#nw-metallb-configure-return-traffic-proc_metallb-configure-return-traffic}

To ensure that applications behind a MetalLB service use the same network path for both ingress and egress, configure symmetric routing by using Virtual Routing and Forwarding (VRF).

The example in the procedure associates a VRF routing table with MetalLB and an egress service to enable symmetric routing for ingress and egress traffic for pods behind a `LoadBalancer` service.

:::warning

- If you use the `sourceIPBy: "LoadBalancerIP"` setting in the `EgressService` CR, you must specify the `LoadBalancer` node in the `BGPAdvertisement` custom resource (CR).
- You can use the `sourceIPBy: "Network"` setting on clusters that use OVN-Kubernetes configured with the `gatewayConfig.routingViaHost` specification set to `true` only. Additionally, if you use the `sourceIPBy: "Network"` setting, you must schedule the application workload on nodes configured with the network VRF instance.

:::

**Prerequisites**

- Install the OpenShift CLI (`oc`).
- Log in as a user with `cluster-admin` privileges.
- Install the Kubernetes NMState Operator.
- Install the MetalLB Operator.
- Create a namespace for your application workload. The examples in this procedure use a namespace called `test`.

**Procedure**

1. Label the nodes that you want to participate in the VRF configuration by running the following command:
   ```terminal
   $ oc label node <node_name> vrf=true
   ```

   The examples in this procedure use the label `vrf: "true"`.
2. Create a `NodeNetworkConfigurationPolicy` CR to define the VRF instance:
   1. Create a file, such as `node-network-vrf.yaml`, with content like the following example:
      ```yaml
      apiVersion: nmstate.io/v1
      kind: NodeNetworkConfigurationPolicy
      metadata:
        name: vrfpolicy
      spec:
        nodeSelector:
          vrf: "true"
        maxUnavailable: 3
        desiredState:
          interfaces:
          - name: ens4vrf
            type: vrf
            state: up
            vrf:
              port:
              - ens4
              route-table-id: 2
          - name: ens4
            type: ethernet
            state: up
            ipv4:
              address:
              - ip: 192.168.130.130
                prefix-length: 24
              dhcp: false
              enabled: true
          routes:
            config:
            - destination: 0.0.0.0/0
              metric: 150
              next-hop-address: 192.168.130.1
              next-hop-interface: ens4
              table-id: 2
          route-rules:
            config:
            - ip-to: 172.30.0.0/16
              priority: 998
              route-table: 254
            - ip-to: 10.128.0.0/14
              priority: 998
              route-table: 254
            - ip-to: 169.254.0.0/17
              priority: 998
              route-table: 254
      # ...
      ```

      where:

      <dl>
      <dt><code>metadata.name</code></dt>
      <dd>Specifies the name of the policy.</dd>
      <dt><code>nodeSelector.vrf</code></dt>
      <dd>Specifies the policy for all nodes with the label <code>vrf:true</code>.</dd>
      <dt><code>interfaces.name.ens4vrf</code></dt>
      <dd>Specifies the name of the interface.</dd>
      <dt><code>interfaces.type</code></dt>
      <dd>Specifies the type of interface. This example creates a VRF instance.</dd>
      <dt><code>vrf.port</code></dt>
      <dd>Specifies the node interface that the VRF attaches to.</dd>
      <dt><code>vrf.route-table-id</code></dt>
      <dd>Specifies the name of the route table ID for the VRF.</dd>
      <dt><code>interfaces.name.ens4</code></dt>
      <dd>Specifies the IPv4 address of the interface associated with the VRF.</dd>
      <dt><code>routes</code></dt>
      <dd>Specifies the configuration for network routes. The <code>next-hop-address</code> field defines the IP address of the next hop for the route. The <code>next-hop-interface</code> field defines the outgoing interface for the route. In this example, the VRF routing table is <code>2</code>, which references the ID that you define in the <code>EgressService</code> CR.</dd>
      <dt><code>route-rules</code></dt>
      <dd>Specifies additional route rules. The <code>ip-to</code> fields must match the <code>Cluster Network</code> CIDR, <code>Service Network</code> CIDR, and <code>Internal Masquerade</code> subnet CIDR. You can view the values for these CIDR address specifications by running the following command: <code>oc describe network.operator/cluster</code>.</dd>
      <dt><code>route-rules.route-table</code></dt>
      <dd>Specifies the main routing table that the Linux kernel uses when calculating routes has the ID <code>254</code>.</dd>
      </dl>
   2. Apply the policy by running the following command:
      ```terminal
      $ oc apply -f node-network-vrf.yaml
      ```
   3. Verify that the policy is applied by running the following command:
      ```terminal
      $ oc get nncp vrfpolicy
      ```

      The output must show `Available` in the `STATUS` column before you continue to the next step.
3. Create a `BGPPeer` custom resource (CR):
   1. Create a file, such as `frr-via-vrf.yaml`, with content like the following example:
      ```yaml
      apiVersion: metallb.io/v1beta2
      kind: BGPPeer
      metadata:
        name: frrviavrf
        namespace: metallb-system
      spec:
        myASN: 100
        peerASN: 200
        peerAddress: 192.168.130.1
        vrf: ens4vrf
      # ...
      ```

      where:

      <dl>
      <dt><code>spec.vrf</code></dt>
      <dd>Specifies the VRF instance to associate with the BGP peer. MetalLB can advertise services and make routing decisions based on the routing information in the VRF.</dd>
      </dl>
   2. Apply the configuration for the BGP peer by running the following command:
      ```terminal
      $ oc apply -f frr-via-vrf.yaml
      ```
4. Create an `IPAddressPool` CR:
   1. Create a file, such as `first-pool.yaml`, with content like the following example:
      ```yaml
      apiVersion: metallb.io/v1beta1
      kind: IPAddressPool
      metadata:
        name: first-pool
        namespace: metallb-system
      spec:
        addresses:
        - 192.169.10.0/32
      # ...
      ```
   2. Apply the configuration for the IP address pool by running the following command:
      ```terminal
      $ oc apply -f first-pool.yaml
      ```
5. Create a `BGPAdvertisement` CR:
   1. Create a file, such as `first-adv.yaml`, with content like the following example:
      ```yaml
      apiVersion: metallb.io/v1beta1
      kind: BGPAdvertisement
      metadata:
        name: first-adv
        namespace: metallb-system
      spec:
        ipAddressPools:
          - first-pool
        peers:
          - frrviavrf
        nodeSelectors:
          - matchLabels:
              egress-service.k8s.ovn.org/test-server1: ""
      # ...
      ```

      where:

      <dl>
      <dt><code>peers</code></dt>
      <dd>In this example, MetalLB advertises a range of IP addresses from the <code>first-pool</code> IP address pool to the <code>frrviavrf</code> BGP peer.</dd>
      <dt><code>nodeSelectors</code></dt>
      <dd>In this example, the <code>EgressService</code> CR configures the source IP address for egress traffic to use the <code>LoadBalancer</code> service IP address. Therefore, you must specify the <code>LoadBalancer</code> node for return traffic to use the same return path for the traffic originating from the pod.</dd>
      </dl>
   2. Apply the configuration for the BGP advertisement by running the following command:
      ```terminal
      $ oc apply -f first-adv.yaml
      ```
6. Create an `EgressService` CR:
   1. Create a file, such as `egress-service.yaml`, with content like the following example:
      ```yaml
      apiVersion: k8s.ovn.org/v1
      kind: EgressService
      metadata:
        name: server1
        namespace: test
      spec:
        sourceIPBy: "LoadBalancerIP"
        nodeSelector:
          matchLabels:
            vrf: "true"
        network: "2"
      # ...
      ```

      where:

      <dl>
      <dt><code>metadata.name</code></dt>
      <dd>Specifies the name for the egress service. The name of the <code>EgressService</code> resource must match the name of the <code>LoadBalancer</code> service that you want to modify.</dd>
      <dt><code>metadata.namespace</code></dt>
      <dd>Specifies the namespace for the egress service. The namespace for the <code>EgressService</code> must match the namespace of the <code>LoadBalancer</code> service that you want to modify. The egress service is namespace-scoped.</dd>
      <dt><code>spec.sourceIPBy</code></dt>
      <dd>Specifies the <code>LoadBalancer</code> service ingress IP address as the source IP address for egress traffic.</dd>
      <dt><code>matchLabels.vrf</code></dt>
      <dd>If you specify <code>LoadBalancer</code> for the <code>sourceIPBy</code> specification, a single node handles the <code>LoadBalancer</code> service traffic. In this example, only a node with the label <code>vrf: "true"</code> can handle the service traffic. If you do not specify a node, OVN-Kubernetes selects a worker node to handle the service traffic. When a node is selected, OVN-Kubernetes labels the node in the following format: <code>egress-service.k8s.ovn.org/&lt;svc_namespace&gt;-&lt;svc_name&gt;: ""</code>.</dd>
      <dt><code>network</code></dt>
      <dd>Specifies the routing table ID for egress traffic. Ensure that the value matches the <code>route-table-id</code> ID defined in the <code>NodeNetworkConfigurationPolicy</code> resource, for example, <code>route-table-id: 2</code>.</dd>
      </dl>
   2. Apply the configuration for the egress service by running the following command:
      ```terminal
      $ oc apply -f egress-service.yaml
      ```

**Verification**

1. Get the external IP address for the `LoadBalancer` service by running the following command:
   ```terminal
   $ oc get svc <service_name> -n <namespace>
   ```

   where `<service_name>` is the name of your `LoadBalancer` service and `<namespace>` is the namespace where the service is deployed. Note the `EXTERNAL-IP` value from the output.
2. Verify that you can access the application endpoint of the pods running behind the MetalLB service by running the following command:
   ```terminal
   $ curl <external_ip_address>:<port_number>
   ```

   where `<external_ip_address>` is the `EXTERNAL-IP` value from the previous step, and `<port_number>` is the port number of your application endpoint.
3. Optional: If you assigned the `LoadBalancer` service ingress IP address as the source IP address for egress traffic, verify this configuration by using tools such as `tcpdump` to analyze packets received at the external client.

**Additional resources**

- [About virtual routing and forwarding](/docs/networking/multiple_networks/about-virtual-routing-and-forwarding#cnf-about-virtual-routing-and-forwarding_about-virtual-routing-and-forwarding)
- [Exposing a service through a network VRF](/docs/networking/ingress_load_balancing/metallb/metallb-configure-bgp-peers#nw-metallb-bgp-peer-vrf_configure-metallb-bgp-peers)
- [Example: Network interface with a VRF instance node network configuration policy](/docs/networking/k8s_nmstate/k8s-nmstate-updating-node-network-config#virt-example-host-vrf_k8s-nmstate-updating-node-network-config)
- [Configuring an egress service](/docs/networking/ovn_kubernetes_network_provider/configuring-egress-traffic-for-vrf-loadbalancer-services#configuring-egress-traffic-loadbalancer-services)
