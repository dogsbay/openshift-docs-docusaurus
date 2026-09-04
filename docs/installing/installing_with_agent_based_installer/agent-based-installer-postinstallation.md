---
title: Postinstallation tasks
sidebar_position: 10
---

# Postinstallation tasks {#agent-based-installer-postinstallation}

<a id="agent-based-installer-postinstallation"></a>

After using the Agent-based Installer to deploy your cluster, you can perform post-installation procedures such as customizing a `br-ex` bridge for nodes in your cluster. Customizing your cluster can help prepare the cluster for specific workloads and deployment requirements.

## Creating a manifest object that includes a customized br-ex bridge {#creating-manifest-file-customized-br-ex-bridge-post_agent-based-installer-postinstallation}

Use the default OVS br-ex bridge configuration for standard environments. This configuration applies when you have a single network interface controller (NIC) and standard OVS settings.

By default, OpenShift Container Platform automatically configures the Open vSwitch (OVS) `br-ex` bridge on bare-metal nodes. For advanced networking requirements, you can override this default behavior on bare-metal platforms. To do this, create an `NodeNetworkConfigurationPolicy` (NNCP) custom resource (CR) that includes an NMState configuration file.

The Kubernetes NMState Operator uses the NMState configuration file to create a customized `br-ex` bridge network configuration. This configuration applies to each node in your cluster.

:::warning

After creating the `NodeNetworkConfigurationPolicy` CR, copy content from the installation NMState configuration file into the NNCP CR. An incomplete NNCP CR can result in loss of network connectivity, because the NNCP overrides all existing policies.

:::

Consider using the customized `br-ex` bridge configuration for any of the following tasks:

- You need to modify the `br-ex` bridge after you installed the cluster.
- You need to modify the maximum transmission unit (MTU) for your cluster.
- You need to update DNS values.
- You need to modify attributes for a different bond interface, such as MIImon (Media Independent Interface Monitor), bonding mode, or Quality of Service (QoS).
- You need to enable Link Layer Discovery Protocol (LLDP) to discover and troubleshoot switch connectivity.

:::warning

The following list of interface names are reserved and you cannot use the names with NMstate configurations:

- `br-ext`
- `br-int`
- `br-local`
- `br-nexthop`
- `br0`
- `ext-vxlan`
- `ext`
- `genev_sys_*`
- `int`
- `k8s-*`
- `ovn-k8s-*`
- `patch-br-*`
- `tun0`
- `vxlan_sys_*`

:::

**Prerequisites**

- You have installed the Kubernetes NMState Operator.
- You have identified the specific nodes where you want to apply the policy.

**Procedure**

- Create a `NodeNetworkConfigurationPolicy` (NNCP) CR and define a customized `br-ex` bridge network configuration. The `br-ex` NNCP CR must include the OVN-Kubernetes masquerade IP address and subnet of your network. The example NNCP CR includes default values in the `ipv4.address.ip` and `ipv6.address.ip` parameters. You can set the masquerade IP address in the `ipv4.address.ip`, `ipv6.address.ip`, or both parameters.
  :::warning

  As a post-installation task, you cannot change the primary IP address of the customized `br-ex` bridge. If you want to convert your single-stack cluster network to a dual-stack cluster network, you can add or change a secondary IPv6 address in the NNCP CR, but the existing primary IP address cannot be changed.

  :::

  ```yaml
  apiVersion: nmstate.io/v1
  kind: NodeNetworkConfigurationPolicy
  metadata:
    name: worker-0-br-ex
  spec:
    nodeSelector:
      kubernetes.io/hostname: worker-0
    desiredState:
      interfaces:
      - name: enp2s0
        type: ethernet
        state: up
        mtu: 9000
        ipv4:
          enabled: false
        ipv6:
          enabled: false
      - name: br-ex
        type: ovs-bridge
        state: up
        ipv4:
          enabled: false
          dhcp: false
        ipv6:
          enabled: false
          dhcp: false
        bridge:
          options:
            mcast-snooping-enable: true
          port:
          - name: enp2s0
          - name: br-ex
      - name: br-ex
        type: ovs-interface
        state: up
        copy-mac-from: enp2s0
        mtu: 9000
        ipv4:
          enabled: true
          dhcp: true
          auto-route-metric: 48
          address:
          - ip: "169.254.0.2"
            prefix-length: 17
        ipv6:
          enabled: true
          dhcp: true
          auto-route-metric: 48
          address:
          - ip: "fd69::2"
          prefix-length: 112
  # ...
  ```

  where:

  <dl>
  <dt><code>metadata.name</code></dt>
  <dd>Specifies the name of the policy.</dd>
  <dt><code>interfaces.name</code></dt>
  <dd>Specifies the name of the interface.</dd>
  <dt><code>interfaces.type</code></dt>
  <dd>Specifies the type of ethernet.</dd>
  <dt><code>interfaces.state</code></dt>
  <dd>Specifies the requested state for the interface after creation.</dd>
  <dt><code>mtu</code></dt>
  <dd>To ensure network stability and performance, you must explicitly declare the MTU in the manifest for every interface. Do not rely on automatic MTU configuration. The MTU configured on a bridge port or VLAN-tagged interface must not exceed the maximum frame size supported by the attached physical medium. A mismatch causes packet fragmentation or connectivity loss.</dd>
  <dt><code>ipv4.enabled</code></dt>
  <dd>Disables IPv4 and IPv6 in this example.</dd>
  <dt><code>port.name</code></dt>
  <dd>Specifies the node NIC to which the bridge is attached.</dd>
  <dt><code>address.ip</code></dt>
  <dd>Shows the default IPv4 and IPv6 IP addresses. Ensure that you set the masquerade IPv4 and IPv6 IP addresses of your network.</dd>
  <dt><code>auto-route-metric</code></dt>
  <dd>Set the parameter to <code>48</code> to ensure the <code>br-ex</code> default route always has the highest precedence (lowest metric). This configuration prevents routing conflicts with any other interfaces automatically configured by the <code>NetworkManager</code> service.</dd>
  </dl>

**Next steps**

- Scaling compute nodes to apply the manifest object that includes a customized `br-ex` bridge to each compute node that exists in your cluster. For more information, see "Expanding the cluster" in the *Additional resources* section.

**Additional resources**

- [Converting to a dual-stack cluster network](/docs/networking/ovn_kubernetes_network_provider/converting-to-dual-stack#nw-dual-stack-convert_converting-to-dual-stack)
- [Expanding the cluster](/docs/installing/installing_bare_metal/bare-metal-expanding-the-cluster#bare-metal-expanding-the-cluster)
