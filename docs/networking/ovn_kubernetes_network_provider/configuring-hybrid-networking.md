---
title: Configuring hybrid networking
sidebar_position: 16
---

# Configuring hybrid networking

<a id="configuring-hybrid-networking"></a>

In OpenShift Container Platform, you can configure OVN-Kubernetes hybrid networking so Linux and Windows nodes run Linux and Windows workloads in the same cluster.

## Configuring hybrid networking with OVN-Kubernetes {#configuring-hybrid-ovnkubernetes_configuring-hybrid-networking}

To configure hybrid networking with OVN-Kubernetes, you can set `hybridOverlayConfig` during installation or patch the Cluster Network Operator (CNO) after installation.

:::note

This configuration is necessary to run both Linux and Windows nodes in the same cluster.

:::

**Prerequisites**

- Install the OpenShift CLI (`oc`).
- Log in to the cluster as a user with `cluster-admin` privileges.
- Ensure that the cluster uses the OVN-Kubernetes network plugin.

**Procedure**

1. To configure the OVN-Kubernetes hybrid network overlay, enter the following command:
   ```terminal
   $ oc patch networks.operator.openshift.io cluster --type=merge \
     -p '{
       "spec":{
         "defaultNetwork":{
           "ovnKubernetesConfig":{
             "hybridOverlayConfig":{
               "hybridClusterNetwork":[
                 {
                   "cidr": "<cidr>",
                   "hostPrefix": <prefix>
                 }
               ],
               "hybridOverlayVXLANPort": <overlay_port>
             }
           }
         }
       }
     }'
   ```

   where:

   <dl>
   <dt><code>&lt;cidr&gt;</code></dt>
   <dd>Specifies the CIDR configuration used for nodes on the additional overlay network. This CIDR must not overlap with the cluster network CIDR.</dd>
   <dt><code>&lt;prefix&gt;</code></dt>
   <dd>Specifies the subnet prefix length to assign to each individual node. For example, if <code>hostPrefix</code> is set to <code>23</code>, then each node is assigned a <code>/23</code> subnet out of the given <code>cidr</code>, which allows for 510 (2^(32 - 23) - 2) pod IP addresses. If you are required to provide access to nodes from an external network, configure load balancers and routers to manage the traffic.</dd>
   <dt><code>&lt;overlay_port&gt;</code></dt>
   <dd>Specifies a custom VXLAN port for the additional overlay network. This is required for running Windows nodes in a cluster installed on vSphere, and must not be configured for any other cloud provider. The custom port can be any open port excluding the default <code>6081</code> port. For more information on this requirement, see <a href="https://docs.microsoft.com/en-us/virtualization/windowscontainers/kubernetes/common-problems#pod-to-pod-connectivity-between-hosts-is-broken-on-my-kubernetes-cluster-running-on-vsphere">Pod-to-pod connectivity between hosts is broken</a> in the Microsoft documentation.</dd>
   </dl>

   :::note

   Windows Server Long-Term Servicing Channel (LTSC): Windows Server 2019 is not supported on clusters with a custom `hybridOverlayVXLANPort` value because this Windows server version does not support selecting a custom VXLAN port.

   :::

   ```text title="Example output"
   network.operator.openshift.io/cluster patched
   ```
2. To confirm that the configuration is active, enter the following command. It can take several minutes for the update to apply.
   ```terminal
   $ oc get network.operator.openshift.io -o jsonpath="{.items[0].spec.defaultNetwork.ovnKubernetesConfig}"
   ```

**Additional resources**

- [Installing a cluster on AWS with network customizations](/docs/installing/installing_aws/ipi/installing-aws-customizations#installing-aws-customizations)
- [Installing a cluster on Azure with customizations](/docs/installing/installing_azure/ipi/installing-azure-customizations#installing-azure-customizations)
