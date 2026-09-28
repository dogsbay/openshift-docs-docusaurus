---
title: Configuring the cluster network range
sidebar_position: 3
---

# Configuring the cluster network range {#configuring-cluster-network-range}

<a id="configuring-cluster-network-range"></a>

To expand the cluster network range in OpenShift Container Platform to support more nodes and IP addresses, you can modify the cluster network CIDR mask after cluster installation. This procedure requires the OVN-Kubernetes network plugin and provides more IP space for additional nodes.

For example, if you deployed a cluster and specified `10.128.0.0/19` as the cluster network range and a host prefix of `23`, you are limited to 16 nodes. You can expand that to 510 nodes by changing the CIDR mask on a cluster to `/14`.

The following limitations apply when modifying the cluster network IP address range:

- The CIDR mask size specified must always be smaller than the currently configured CIDR mask size, because you can only increase IP space by adding more nodes to an installed cluster
- The host prefix cannot be modified
- Pods that are configured with an overridden default gateway must be recreated after the cluster network expands

:::warning

You cannot expand the service network CIDR range after installing the cluster, either directly or through the ServiceCIDR API. You must configure the service network CIDR during installation using the `install-config.yaml` file. To avoid service IP address exhaustion, ensure that your initial service network range is large enough to accommodate future growth.

:::

## Expand the cluster network IP address range {#nw-cluster-network-range-edit_configuring-cluster-network-range}

To expand the cluster network IP address range in OpenShift Container Platform to support more nodes, you can modify the cluster network CIDR mask using the `oc patch` command.

:::note

This change requires rolling out a new Operator configuration across the cluster, and can take up to 30 minutes to take effect.

For clusters configured with cluster proxy, expanding the cluster network range also triggers a MachineConfigPool update that reboots all nodes. Plan this operation during a maintenance window to avoid service disruption.

:::

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).
- You have logged in to the cluster with a user with `cluster-admin` privileges.
- You have ensured that the cluster uses the OVN-Kubernetes network plugin.

**Procedure**

1. To obtain the cluster network range and host prefix for your cluster, enter the following command:
   ```terminal
   $ oc get network.operator.openshift.io \
     -o jsonpath="{.items[0].spec.clusterNetwork}"
   ```

   ```text title="Example output"
   [{"cidr":"10.217.0.0/22","hostPrefix":23}]
   ```
2. To expand the cluster network IP address range, enter the following command. Use the CIDR IP address range and host prefix returned from the output of the previous command.
   ```terminal
   $ oc patch Network.config.openshift.io cluster --type='merge' --patch \
     '{
       "spec":{
         "clusterNetwork": [ {"cidr":"<network>/<cidr>","hostPrefix":<prefix>} ],
         "networkType": "OVNKubernetes"
       }
     }'
   ```

   where:

   <dl>
   <dt><code>&lt;network&gt;</code></dt>
   <dd>Specifies the network part of the <code>cidr</code> field that you obtained from the previous step. You cannot change this value.</dd>
   <dt><code>&lt;cidr&gt;</code></dt>
   <dd>Specifies the network prefix length. For example, <code>14</code>. Change this value to a smaller number than the value from the output in the previous step to expand the cluster network range.</dd>
   <dt><code>&lt;prefix&gt;</code></dt>
   <dd>Specifies the current host prefix for your cluster. This value must be the same value for the <code>hostPrefix</code> field that you obtained from the previous step.</dd>
   </dl>

   ```terminal title="Example command"
   $ oc patch Network.config.openshift.io cluster --type='merge' --patch \
     '{
       "spec":{
         "clusterNetwork": [ {"cidr":"10.217.0.0/14","hostPrefix": 23} ],
         "networkType": "OVNKubernetes"
       }
     }'
   ```

   ```text title="Example output"
   network.config.openshift.io/cluster patched
   ```
3. To confirm that the configuration is active, enter the following command. It can take up to 30 minutes for this change to take effect.
   ```terminal
   $ oc get network.operator.openshift.io \
     -o jsonpath="{.items[0].spec.clusterNetwork}"
   ```

   ```text title="Example output"
   [{"cidr":"10.217.0.0/14","hostPrefix":23}]
   ```

**Additional resources**

- [OVN-Kubernetes network plugin](/docs/networking/ovn_kubernetes_network_provider/about-ovn-kubernetes#about-ovn-kubernetes)
- [Red Hat OpenShift Network Calculator](https://access.redhat.com/labs/ocpnc/)
- [About the OVN-Kubernetes network plugin](/docs/networking/ovn_kubernetes_network_provider/about-ovn-kubernetes#about-ovn-kubernetes)
