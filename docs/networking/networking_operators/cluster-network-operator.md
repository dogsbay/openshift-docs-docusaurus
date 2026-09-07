---
title: Cluster Network Operator in OpenShift Container Platform
sidebar_position: 6
---

# Cluster Network Operator in OpenShift Container Platform {#cluster-network-operator}

<a id="cluster-network-operator"></a>

With the Cluster Network Operator, you can manage networking in OpenShift Container Platform, including how to view status, enable IP forwarding, and collect logs.

You can use the Cluster Network Operator (CNO) to deploy and manage cluster network components on an OpenShift Container Platform cluster, including the Container Network Interface (CNI) network plugin selected for the cluster during installation.

## Cluster Network Operator {#nw-cluster-network-operator_cluster-network-operator}

The Cluster Network Operator implements the `network` API from the `operator.openshift.io` API group. The Operator deploys the OVN-Kubernetes network plugin, or the network provider plugin that you selected during cluster installation, by using a daemon set.

The Cluster Network Operator is deployed during installation as a Kubernetes `Deployment`.

**Procedure**

1. Run the following command to view the Deployment status:
   ```terminal
   $ oc get -n openshift-network-operator deployment/network-operator
   ```

   ```terminal title="Example output"
   NAME               READY   UP-TO-DATE   AVAILABLE   AGE
   network-operator   1/1     1            1           56m
   ```
2. Run the following command to view the state of the Cluster Network Operator:
   ```terminal
   $ oc get clusteroperator/network
   ```

   ```terminal title="Example output"
   NAME      VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE
   network   4.16.1     True        False         False      50m
   ```

   The following fields provide information about the status of the operator: `AVAILABLE`, `PROGRESSING`, and `DEGRADED`. The `AVAILABLE` field is `True` when the Cluster Network Operator reports an available status condition.

## Viewing the cluster network configuration {#nw-cno-view_cluster-network-operator}

You can view your OpenShift Container Platform cluster network configuration by using the `oc describe` command for the `network.config/cluster` resource.

**Procedure**

- Use the `oc describe` command to view the cluster network configuration:
  ```terminal
  $ oc describe network.config/cluster
  ```

  ```terminal title="Example output"
  Name:         cluster
  Namespace:
  Labels:       <none>
  Annotations:  <none>
  API Version:  config.openshift.io/v1
  Kind:         Network
  Metadata:
    Creation Timestamp:  2024-08-08T11:25:56Z
    Generation:          3
    Resource Version:    29821
    UID:                 808dd2be-5077-4ff7-b6bb-21b7110126c7
  Spec:
    Cluster Network:
      Cidr:         10.128.0.0/14
      Host Prefix:  23
    External IP:
      Policy:
    Network Diagnostics:
      Mode:
      Source Placement:
      Target Placement:
    Network Type:  OVNKubernetes
    Service Network:
      172.30.0.0/16
  Status
    Cluster Network:
      Cidr:               10.128.0.0/14
      Host Prefix:        23
    Cluster Network MTU:  1360
    Conditions:
      Last Transition Time:  2024-08-08T11:51:50Z
      Message:
      Observed Generation:   0
      Reason:                AsExpected
      Status:                True
      Type:                  NetworkDiagnosticsAvailable
    Network Type:            OVNKubernetes
    Service Network:
      172.30.0.0/16
  Events:  <none>
  ```

  where:

  <dl>
  <dt><code>spec</code></dt>
  <dd>Specifies the field that displays the configured state of the cluster network.</dd>
  <dt><code>Status</code></dt>
  <dd>Displays the current state of the cluster network configuration.</dd>
  </dl>

## Viewing Cluster Network Operator status {#nw-cno-status_cluster-network-operator}

You can inspect the status and view the details of the Cluster Network Operator by using the `oc describe` command.

**Procedure**

- Run the following command to view the status of the Cluster Network Operator:
  ```terminal
  $ oc describe clusteroperators/network
  ```

## Enabling IP forwarding globally {#nw-cno-enable-ip-forwarding_cluster-network-operator}

From OpenShift Container Platform 4.14 onward, OVN-Kubernetes disables global IP forwarding by default. By setting the Cluster Network Operator `gatewayConfig.ipForwarding` spec to `Global`, you can enable cluster-wide forwarding.

**Procedure**

1. Backup the existing network configuration by running the following command:
   ```terminal
   $ oc get network.operator cluster -o yaml > network-config-backup.yaml
   ```
2. Run the following command to modify the existing network configuration:
   ```terminal
   $ oc edit network.operator cluster
   ```

   1. Add or update the following block under `spec` as illustrated in the following example:
      ```yaml
      spec:
        clusterNetwork:
        - cidr: 10.128.0.0/14
          hostPrefix: 23
        serviceNetwork:
        - 172.30.0.0/16
        networkType: OVNKubernetes
        clusterNetworkMTU: 8900
        defaultNetwork:
          ovnKubernetesConfig:
            gatewayConfig:
              ipForwarding: Global
      ```
   2. Save and close the file.
3. After applying the changes, the OpenShift Cluster Network Operator (CNO) applies the update across the cluster. You can monitor the progress by using the following command:
   ```terminal
   $ oc get clusteroperators network
   ```

   The status should eventually report as `Available`, `Progressing=False`, and `Degraded=False`.
4. Alternatively, you can enable IP forwarding globally by running the following command:
   ```terminal
   $ oc patch network.operator cluster -p '{"spec":{"defaultNetwork":{"ovnKubernetesConfig":{"gatewayConfig":{"ipForwarding": "Global"}}}}}' --type=merge
   ```

   :::note

   The other valid option for this parameter is `Restricted` in case you want to revert this change. `Restricted` is the default and with that setting global IP address forwarding is disabled.

   :::

## Viewing Cluster Network Operator logs {#nw-cno-logs_cluster-network-operator}

You can view Cluster Network Operator logs by using the `oc logs` command.

**Procedure**

- Run the following command to view the logs of the Cluster Network Operator:
  ```terminal
  $ oc logs --namespace=openshift-network-operator deployment/network-operator
  ```

## Cluster Network Operator configuration {#nw-operator-cr_cluster-network-operator}

To manage cluster networking, configure the Cluster Network Operator (CNO) `Network` custom resource (CR) named `cluster` so the cluster uses the correct IP ranges and network plugin settings for reliable pod and service connectivity. Some settings and fields are inherited at the time of install or by the `default.Network.type` plugin, OVN-Kubernetes.

The CNO configuration inherits the following fields during cluster installation from the `Network` API in the `Network.config.openshift.io` API group:

`clusterNetwork`
: IP address pools from which pod IP addresses are allocated.

`serviceNetwork`
: IP address pool for services.

`defaultNetwork.type`
: Cluster network plugin. `OVNKubernetes` is the only supported plugin during installation.

:::note

After cluster installation, you can only modify the `clusterNetwork` IP address range. The `serviceNetwork` range cannot be modified post-installation, either directly or by using the `ServiceCIDR` API.

:::

You can specify the cluster network plugin configuration for your cluster by setting the fields for the `defaultNetwork` object in the CNO object named `cluster`.

### Cluster Network Operator configuration object {#nw-operator-cr-cno-object_cluster-network-operator}

The fields for the Cluster Network Operator (CNO) are described in the following table:

**Cluster Network Operator configuration object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>metadata.name</code></td>
  <td><code>string</code></td>
  <td>The name of the CNO object. This name is always <code>cluster</code>.</td>
</tr>
<tr>
  <td><code>spec.clusterNetwork</code></td>
  <td><code>array</code></td>
  <td>A list specifying the blocks of IP addresses from which pod IP addresses are allocated and the subnet prefix length assigned to each individual node in the cluster. If you use dual-stack networking, specify IPv4 and IPv6 address families. For example:<br><br><pre><code>spec:&#10;  clusterNetwork:&#10;  - cidr: 10.128.0.0/19&#10;    hostPrefix: 23&#10;  - cidr: fd01::/48&#10;    hostPrefix: 64</code></pre><br><br>If you install a cluster on AWS with dual-stack networking, the order of addresses must match the dual-stack configuration you selected. For example, if you specified the <code>DualStackIPv4Primary</code>, list the IPv4 address first.</td>
</tr>
<tr>
  <td><code>spec.serviceNetwork</code></td>
  <td><code>array</code></td>
  <td>A block of IP addresses for services. If you use dual-stack networking, specify IPv4 and IPv6 address families. For example:<br><br><pre><code>spec:&#10;  serviceNetwork:&#10;  - 172.30.0.0/14&#10;  - fd02::/112</code></pre><br><br>If you install a cluster on AWS with dual-stack networking, the order of addresses must match the dual-stack configuration you selected. For example, if you specified the <code>DualStackIPv4Primary</code>, list the IPv4 address first.<br><br>  This value is ready-only and inherited from the <code>Network.config.openshift.io</code> object named <code>cluster</code> during cluster installation.  </td>
</tr>
<tr>
  <td><code>spec.defaultNetwork</code></td>
  <td><code>object</code></td>
  <td>Configures the network plugin for the cluster network.</td>
</tr>
<tr>
  <td><code>spec.additionalRoutingCapabilities.providers</code></td>
  <td><code>array</code></td>
  <td>This setting enables a dynamic routing provider. The FRR routing capability provider is required for the route advertisement feature. The only supported value is <code>FRR</code>.<br><br><ul><li><code>FRR</code>: The FRR routing provider</li></ul><br><br><pre><code>spec:&#10;  additionalRoutingCapabilities:&#10;    providers:&#10;    - FRR</code></pre></td>
</tr>
</tbody>
</table>

:::warning

For a cluster that needs to deploy objects across multiple networks, ensure that you specify the same value for the `clusterNetwork.hostPrefix` parameter for each network type that is defined in the `install-config.yaml` file. Setting a different value for each `clusterNetwork.hostPrefix` parameter can impact the OVN-Kubernetes network plugin, where the plugin cannot effectively route object traffic among different nodes.

:::

### defaultNetwork object configuration {#nw-operator-cr-defaultnetwork_cluster-network-operator}

The values for the `defaultNetwork` object are defined in the following table:

`defaultNetwork`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>type</code></td>
  <td><code>string</code></td>
  <td><code>OVNKubernetes</code>. The Red Hat OpenShift Networking network plugin is selected during installation. This value cannot be changed after cluster installation.<div class="alert alert--secondary"><strong>Note</strong><p>OpenShift Container Platform uses the OVN-Kubernetes network plugin by default.</p></div></td>
</tr>
<tr>
  <td><code>ovnKubernetesConfig</code></td>
  <td><code>object</code></td>
  <td>This object is only valid for the OVN-Kubernetes network plugin.</td>
</tr>
</tbody>
</table>

### Configuration for the OVN-Kubernetes network plugin {#nw-operator-configuration-parameters-for-ovn-sdn_cluster-network-operator}

The following table describes the configuration fields for the OVN-Kubernetes network plugin:

`ovnKubernetesConfig`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>mtu</code></td>
  <td><code>integer</code></td>
  <td>  The maximum transmission unit (MTU) for the Geneve (Generic Network Virtualization Encapsulation) overlay network. This value is normally configured automatically. </td>
</tr>
<tr>
  <td><code>genevePort</code></td>
  <td><code>integer</code></td>
  <td>  The UDP port for the Geneve overlay network. </td>
</tr>
<tr>
  <td><code>ipsecConfig</code></td>
  <td><code>object</code></td>
  <td>  An object describing the IPsec mode for the cluster. </td>
</tr>
<tr>
  <td><code>ipv4</code></td>
  <td><code>object</code></td>
  <td>Specifies a configuration object for IPv4 settings.</td>
</tr>
<tr>
  <td><code>ipv6</code></td>
  <td><code>object</code></td>
  <td>Specifies a configuration object for IPv6 settings.</td>
</tr>
<tr>
  <td><code>policyAuditConfig</code></td>
  <td><code>object</code></td>
  <td>Specify a configuration object for customizing network policy audit logging. If unset, the defaults audit log settings are used.</td>
</tr>
<tr>
  <td><code>routeAdvertisements</code></td>
  <td><code>string</code></td>
  <td>Specifies whether to advertise cluster network routes. The default value is <code>Disabled</code>.<ul><li><code>Enabled</code>: Import routes to the cluster network and advertise cluster network routes as configured in <code>RouteAdvertisements</code> objects.</li><li><code>Disabled</code>: Do not import routes to the cluster network or advertise cluster network routes.</li></ul></td>
</tr>
<tr>
  <td><code>gatewayConfig</code></td>
  <td><code>object</code></td>
  <td>Optional: Specify a configuration object for customizing how egress traffic is sent to the node gateway. Valid values are <code>Shared</code> and <code>Local</code>. The default value is <code>Shared</code>. In the default setting, the Open vSwitch (OVS) outputs traffic directly to the node IP interface. If you are using hardware offloading, Red Hat recommends to use the default <code>Shared</code> gateway mode to bypass the host routing plane. In the <code>Local</code> setting, it traverses the host network; consequently, it gets applied to the routing table of the host.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>While migrating egress traffic, you can expect some disruption to workloads and service traffic until the Cluster Network Operator (CNO) successfully rolls out the changes.</p></div></td>
</tr>
</tbody>
</table>

`ovnKubernetesConfig.ipv4`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>internalTransitSwitchSubnet</code></td>
  <td>string</td>
  <td>If your existing network infrastructure overlaps with the <code>100.88.0.0/16</code> IPv4 subnet, you can specify a different IP address range for internal use by OVN-Kubernetes. The subnet for the distributed transit switch that enables east-west traffic. This subnet cannot overlap with any other subnets used by OVN-Kubernetes or on the host itself. It must be large enough to accommodate one IP address per node in your cluster.<br><br>The default value is <code>100.88.0.0/16</code>.</td>
</tr>
<tr>
  <td><code>internalJoinSubnet</code></td>
  <td>string</td>
  <td>If your existing network infrastructure overlaps with the <code>100.64.0.0/16</code> IPv4 subnet, you can specify a different IP address range for internal use by OVN-Kubernetes. You must ensure that the IP address range does not overlap with any other subnet used by your OpenShift Container Platform installation. The IP address range must be larger than the maximum number of nodes that can be added to the cluster. For example, if the <code>clusterNetwork.cidr</code> value is <code>10.128.0.0/14</code> and the <code>clusterNetwork.hostPrefix</code> value is <code>/23</code>, then the maximum number of nodes is <code>2^(23-14)=512</code>.<br><br>The default value is <code>100.64.0.0/16</code>.</td>
</tr>
</tbody>
</table>

`ovnKubernetesConfig.ipv6`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>internalTransitSwitchSubnet</code></td>
  <td>string</td>
  <td>If your existing network infrastructure overlaps with the <code>fd97::/64</code> IPv6 subnet, you can specify a different IP address range for internal use by OVN-Kubernetes. The subnet for the distributed transit switch that enables east-west traffic. This subnet cannot overlap with any other subnets used by OVN-Kubernetes or on the host itself. It must be large enough to accommodate one IP address per node in your cluster.<br><br>The default value is <code>fd97::/64</code>.</td>
</tr>
<tr>
  <td><code>internalJoinSubnet</code></td>
  <td>string</td>
  <td>If your existing network infrastructure overlaps with the <code>fd98::/64</code> IPv6 subnet, you can specify a different IP address range for internal use by OVN-Kubernetes. You must ensure that the IP address range does not overlap with any other subnet used by your OpenShift Container Platform installation. The IP address range must be larger than the maximum number of nodes that can be added to the cluster.<br><br>The default value is <code>fd98::/64</code>.</td>
</tr>
</tbody>
</table>

`policyAuditConfig`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>rateLimit</code></td>
  <td>integer</td>
  <td>The maximum number of messages to generate every second per node. The default value is <code>20</code> messages per second.</td>
</tr>
<tr>
  <td><code>maxFileSize</code></td>
  <td>integer</td>
  <td>The maximum size for the audit log in bytes. The default value is <code>50000000</code> or 50 MB.</td>
</tr>
<tr>
  <td><code>maxLogFiles</code></td>
  <td>integer</td>
  <td>The maximum number of log files that are retained.</td>
</tr>
<tr>
  <td><code>destination</code></td>
  <td>string</td>
  <td>One of the following additional audit log targets:<br><br><dl><dt><code>libc</code></dt><dd>The libc <code>syslog()</code> function of the journald process on the host.</dd><dt><code>udp:&lt;host&gt;:&lt;port&gt;</code></dt><dd>A syslog server. Replace <code>&lt;host&gt;:&lt;port&gt;</code> with the host and port of the syslog server.</dd><dt><code>unix:&lt;file&gt;</code></dt><dd>A UNIX Domain Socket file specified by <code>&lt;file&gt;</code>.</dd><dt><code>null</code></dt><dd>Do not send the audit logs to any additional target.</dd></dl></td>
</tr>
<tr>
  <td><code>syslogFacility</code></td>
  <td>string</td>
  <td>The syslog facility, such as <code>kern</code>, as defined by RFC5424. The default value is <code>local0</code>.</td>
</tr>
</tbody>
</table>

<a name="gatewayConfig-object_cluster-network-operator"></a>

`gatewayConfig`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>routingViaHost</code></td>
  <td><code>boolean</code></td>
  <td>Set this field to <code>true</code> to send egress traffic from pods to the host networking stack. For highly-specialized installations and applications that rely on manually configured routes in the kernel routing table, you might want to route egress traffic to the host networking stack. By default, egress traffic is processed in OVN to exit the cluster and is not affected by specialized routes in the kernel routing table. The default value is <code>false</code>.<br><br>This field has an interaction with the Open vSwitch hardware offloading feature. If you set this field to <code>true</code>, you do not receive the performance benefits of the offloading because egress traffic is processed by the host networking stack.</td>
</tr>
<tr>
  <td><code>ipForwarding</code></td>
  <td><code>object</code></td>
  <td>You can control IP forwarding for all traffic on OVN-Kubernetes managed interfaces by using the <code>ipForwarding</code> specification in the <code>Network</code> resource. Specify <code>Restricted</code> to only allow IP forwarding for Kubernetes related traffic. Specify <code>Global</code> to allow forwarding of all IP traffic. For new installations, the default is <code>Restricted</code>. For updates to OpenShift Container Platform 4.14 or later, the default is <code>Global</code>.<div class="alert alert--secondary"><strong>Note</strong><p>The default value of <code>Restricted</code> sets the IP forwarding to drop.</p></div></td>
</tr>
<tr>
  <td><code>ipv4</code></td>
  <td><code>object</code></td>
  <td>Optional: Specify an object to configure the internal OVN-Kubernetes masquerade address for host to service traffic for IPv4 addresses.</td>
</tr>
<tr>
  <td><code>ipv6</code></td>
  <td><code>object</code></td>
  <td>Optional: Specify an object to configure the internal OVN-Kubernetes masquerade address for host to service traffic for IPv6 addresses.</td>
</tr>
</tbody>
</table>

<a name="gatewayconfig-ipv4-object_cluster-network-operator"></a>

`gatewayConfig.ipv4`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>internalMasqueradeSubnet</code></td>
  <td><code>string</code></td>
  <td>The masquerade IPv4 addresses that are used internally to enable host to service traffic. The host is configured with these IP addresses and the shared gateway bridge interface. The default value is <code>169.254.169.0/29</code>.<div class="alert alert--warning"><strong>Important</strong><p>For OpenShift Container Platform 4.17 and later versions, clusters use <code>169.254.0.0/17</code> as the default masquerade subnet. For upgraded clusters, there is no change to the default masquerade subnet.</p></div></td>
</tr>
</tbody>
</table>

<a name="gatewayconfig-ipv6-object_cluster-network-operator"></a>

`gatewayConfig.ipv6`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>internalMasqueradeSubnet</code></td>
  <td><code>string</code></td>
  <td>The masquerade IPv6 addresses that are used internally to enable host to service traffic. The host is configured with these IP addresses and the shared gateway bridge interface. The default value is <code>fd69::/125</code>.<div class="alert alert--warning"><strong>Important</strong><p>For OpenShift Container Platform 4.17 and later versions, clusters use <code>fd69::/112</code> as the default masquerade subnet. For upgraded clusters, there is no change to the default masquerade subnet.</p></div></td>
</tr>
</tbody>
</table>

<a name="nw-operator-cr-ipsec_cluster-network-operator"></a>

`ipsecConfig`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>mode</code></td>
  <td><code>string</code></td>
  <td>Specifies the behavior of the IPsec implementation. Must be one of the following values:<br><br><ul><li><code>Disabled</code>: IPsec is not enabled on cluster nodes.</li><li><code>External</code>: IPsec is enabled for network traffic with external hosts.</li><li><code>Full</code>: IPsec is enabled for pod traffic and network traffic with external hosts.</li></ul></td>
</tr>
</tbody>
</table>

:::note

You can only change the configuration for your cluster network plugin during cluster installation, except for the `gatewayConfig` field that can be changed at runtime as a postinstallation activity.

:::

```yaml title="Example OVN-Kubernetes configuration with IPsec enabled"
defaultNetwork:
  type: OVNKubernetes
  ovnKubernetesConfig:
    mtu: 1400
    genevePort: 6081
    ipsecConfig:
      mode: Full
```

### Cluster Network Operator example configuration {#nw-operator-example-cr_cluster-network-operator}

A complete CNO configuration is specified in the following example:

```yaml title="Example Cluster Network Operator object"
apiVersion: operator.openshift.io/v1
kind: Network
metadata:
  name: cluster
spec:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  serviceNetwork:
  - 172.30.0.0/16
  networkType: OVNKubernetes
```

**Additional resources**

- [`Network` API in the `operator.openshift.io` API group](/docs/rest_api/operator_apis/network-operator-openshift-io-v1#network-operator-openshift-io-v1)
- [Expanding the cluster network IP address range](/docs/networking/configuring_network_settings/configuring-cluster-network-range#nw-cluster-network-range-edit_configuring-cluster-network-range)
- [How to configure OVN to use kernel routing table](https://access.redhat.com/solutions/6969174)
