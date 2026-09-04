---
title: Converting to IPv4/IPv6 dual-stack networking
sidebar_position: 5
---

# Converting to IPv4/IPv6 dual-stack networking {#converting-to-dual-stack}

<a id="converting-to-dual-stack"></a>

To enable IPv4 and IPv6 on your cluster network in OpenShift Container Platform, you can convert a single-stack cluster to dual-stack networking. After conversion, new and existing pods can use both address families when you re-create workloads as needed.

:::warning

When using dual-stack networking where IPv6 is required, you cannot use IPv4-mapped IPv6 addresses, such as `::FFFF:198.51.100.1`.

:::

**Additional resources**

- [OVN-Kubernetes purpose](/docs/networking/ovn_kubernetes_network_provider/about-ovn-kubernetes#nw-ovn-kubernetes-purpose_about-ovn-kubernetes)

## Converting to a dual-stack cluster network {#nw-dual-stack-convert_converting-to-dual-stack}

To convert your cluster network from IPv4 single-stack to dual-stack in OpenShift Container Platform, you can patch the cluster network and, on installer-provisioned infrastructure, the infrastructure custom resources. You must re-create existing pods after conversion to receive IPv6 addresses.

As a cluster administrator, you can convert your single-stack cluster network to a dual-stack cluster network.

:::warning

After converting your cluster to use dual-stack networking, you must re-create any existing pods for them to receive IPv6 addresses, because only new pods are assigned IPv6 addresses.

:::

Converting a single-stack cluster network to a dual-stack cluster network consists of creating patches and applying them to the network and infrastructure of the cluster. You can convert to a dual-stack cluster network for a cluster that runs on either installer-provisioned infrastructure or user-provisioned infrastructure.

:::note

Each patch operation that changes `clusterNetwork`, `serviceNetwork`, `apiServerInternalIPs`, and `ingressIP` objects triggers a restart of the cluster. Changing the `MachineNetworks` object does not cause a reboot of the cluster.

:::

On installer-provisioned infrastructure only, if you need to add IPv6 virtual IPs (VIPs) for API and Ingress services to an existing dual-stack-configured cluster, you need to patch only the infrastructure and not the network for the cluster.

:::warning

If you already upgraded your cluster to OpenShift Container Platform 4.16 or later and you need to convert the single-stack cluster network to a dual-stack cluster network, you must specify an existing IPv4 `machineNetwork` network configuration from the `install-config.yaml` file for API and Ingress services in the YAML configuration patch file. This configuration ensures that IPv4 traffic exists in the same network interface as the default gateway.

The following example adds an IPv4 address block for the `machineNetwork` network:

```yaml
- op: add
  path: /spec/platformSpec/baremetal/machineNetworks/-
  value: 192.168.1.0/24
  # ...
```

where:

`/spec/platformSpec/baremetal/machineNetworks/-`
: Specifies an address block for the `machineNetwork` network where your machines operate. You must select both API and Ingress IP addresses for the machine network.

:::

**Prerequisites**

- You installed the OpenShift CLI (`oc`).
- You are logged in to the cluster with a user with `cluster-admin` privileges.
- Your cluster uses the OVN-Kubernetes network plugin.
- The cluster nodes have IPv6 addresses.
- You have configured an IPv6-enabled router based on your infrastructure.

**Procedure**

1. To specify IPv6 address blocks for cluster and service networks, create a YAML configuration patch file that has a similar configuration to the following example:
   ```yaml
   - op: add
     path: /spec/clusterNetwork/-
     value:
       cidr: fd01::/48
       hostPrefix: 64
   - op: add
     path: /spec/serviceNetwork/-
     value: fd02::/112
   ```

   where:

   <dl>
   <dt><code>/spec/clusterNetwork/-</code></dt>
   <dd>Specifies an object with the <code>cidr</code> and <code>hostPrefix</code> parameters. The host prefix must be <code>64</code> or greater. The IPv6 Classless Inter-Domain Routing (CIDR) prefix must be large enough to accommodate the specified host prefix.</dd>
   <dt><code>/spec/serviceNetwork/-</code></dt>
   <dd>Specifies an IPv6 CIDR with a prefix of <code>112</code>. Kubernetes uses only the lowest 16 bits. For a prefix of <code>112</code>, IP addresses are assigned from <code>112</code> to <code>128</code> bits.</dd>
   </dl>
2. Patch the cluster network configuration by entering the following command in your CLI:
   ```terminal
   $ oc patch network.config.openshift.io cluster \
     --type='json' --patch-file <file>.yaml
   ```

   For `<file>`, specify the name of the YAML file you created.

   ```text title="Example output"
   network.config.openshift.io/cluster patched
   ```
3. On installer-provisioned infrastructure where you added IPv6 VIPs for API and Ingress services, complete the following steps:
   1. Specify IPv6 VIPs for API and Ingress services for your cluster. Create a YAML configuration patch file that has a similar configuration to the following example:
      ```yaml
      - op: add
        path: /spec/platformSpec/baremetal/machineNetworks/-
        value: fd2e:6f44:5dd8::/64
      - op: add
        path: /spec/platformSpec/baremetal/apiServerInternalIPs/-
        value: fd2e:6f44:5dd8::4
      - op: add
        path: /spec/platformSpec/baremetal/ingressIPs/-
        value: fd2e:6f44:5dd8::5
      ```

      where:

      <dl>
      <dt><code>/spec/platformSpec/baremetal/machineNetworks/-</code></dt>
      <dd>Specifies an address block for the <code>machineNetwork</code> network where your machines operate. You must select both API and Ingress IP addresses for the machine network.</dd>
      <dt><code>/spec/platformSpec/baremetal/apiServerInternalIPs/-</code></dt>
      <dd>Specifies each file path according to your platform. The example demonstrates a file path on a bare metal platform.</dd>
      </dl>
   2. Patch the infrastructure by entering the following command in your CLI:
      ```terminal
      $ oc patch infrastructure cluster \
        --type='json' --patch-file <file>.yaml
      ```

      For `<file>`, specify the name of the YAML file you created.

      ```text title="Example output"
      infrastructure/cluster patched
      ```

**Verification**

1. Show the cluster network configuration by entering the following command in your CLI:
   ```terminal
   $ oc describe network
   ```
2. Verify the successful installation of the patch on the network configuration by checking that the cluster network configuration recognizes the IPv6 address blocks that you specified in the YAML file.
   ```text title="Example output"
   # ...
   Status:
     Cluster Network:
       Cidr:               10.128.0.0/14
       Host Prefix:        23
       Cidr:               fd01::/48
       Host Prefix:        64
     Cluster Network MTU:  1400
     Network Type:         OVNKubernetes
     Service Network:
       172.30.0.0/16
       fd02::/112
   # ...
   ```
3. Complete the following additional tasks for a cluster that runs on installer-provisioned infrastructure:
   1. Show the cluster infrastructure configuration by entering the following command in your CLI:
      ```terminal
      $ oc describe infrastructure
      ```
   2. Verify the successful installation of the patch on the cluster infrastructure by checking that the infrastructure recognizes the IPv6 address blocks that you specified in the YAML file.
      ```text title="Example output"
      # ...
      spec:
      # ...
        platformSpec:
          baremetal:
            apiServerInternalIPs:
            - 192.168.123.5
            - fd2e:6f44:5dd8::4
            ingressIPs:
            - 192.168.123.10
            - fd2e:6f44:5dd8::5
      status:
      # ...
        platformStatus:
          baremetal:
            apiServerInternalIP: 192.168.123.5
            apiServerInternalIPs:
            - 192.168.123.5
            - fd2e:6f44:5dd8::4
            ingressIP: 192.168.123.10
            ingressIPs:
            - 192.168.123.10
            - fd2e:6f44:5dd8::5
      # ...
      ```

## Converting to a single-stack cluster network {#nw-dual-stack-convert-back-single-stack_converting-to-dual-stack}

To revert dual-stack networking in OpenShift Container Platform, you can edit the cluster network configuration and remove the IPv4 or IPv6 blocks you added during dual-stack conversion. You can convert back only to the same single-stack family you had before dual-stack (IPv4 or IPv6).

:::warning

If you originally converted your IPv4 single-stack cluster network to a dual-stack cluster, you can convert only back to the IPv4 single-stack cluster and not an IPv6 single-stack cluster network. The same restriction applies for converting back to an IPv6 single-stack cluster network.

:::

**Prerequisites**

- You installed the OpenShift CLI (`oc`).
- You are logged in to the cluster with a user with `cluster-admin` privileges.
- Your cluster uses the OVN-Kubernetes network plugin.
- The cluster nodes have IPv6 addresses.
- You have enabled dual-stack networking.

**Procedure**

1. Edit the `networks.config.openshift.io` custom resource (CR) by running the following command:
   ```terminal
   $ oc edit networks.config.openshift.io
   ```
2. Remove the IPv4 or IPv6 configuration that you added to the `cidr` and the `hostPrefix` parameters from completing the "Converting to a dual-stack cluster network " procedure steps.
