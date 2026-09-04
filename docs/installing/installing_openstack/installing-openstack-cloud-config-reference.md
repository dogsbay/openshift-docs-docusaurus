---
title: OpenStack Cloud Controller Manager reference guide
sidebar_position: 8
---

# OpenStack Cloud Controller Manager reference guide {#installing-openstack-cloud-config-reference}

<a id="installing-openstack-cloud-config-reference"></a>

The reference guide provides a comprehensive overview of the Red Hat OpenStack Platform (RHOSP) Cloud Controller Manager (CCM) config map parameters, specifically detailing load balancer options and properties automatically managed by the Operator.

## The OpenStack Cloud Controller Manager {#nw-openstack-external-ccm_installing-openstack-cloud-config-reference}

Beginning with OpenShift Container Platform 4.12, clusters that run on Red Hat OpenStack Platform (RHOSP) were switched from the legacy RHOSP cloud provider to the external OpenStack Cloud Controller Manager (CCM).

This change follows the move in Kubernetes from in-tree, legacy cloud providers to external cloud providers that are implemented by using the CCM.

To preserve user-defined configurations for the legacy cloud provider, existing configurations are mapped to new ones as part of the migration process. The OpenStack CCM searches for a configuration called `cloud-provider-config` in the `openshift-config` namespace.

:::note

The config map name `cloud-provider-config` is not statically configured. The name is derived from the `spec.cloudConfig.name` value in the `infrastructure/cluster` CRD.

:::

Found configurations are synchronized to the `cloud-conf` config map in the `openshift-cloud-controller-manager` namespace.

As part of this synchronization, the OpenStack CCM Operator alters the new config map such that its properties are compatible with the external cloud provider. The file is changed in the following ways:

- The `[Global] secret-name`, `[Global] secret-namespace`, and `[Global] kubeconfig-path` options are removed. They do not apply to the external cloud provider.
- The `[Global] use-clouds`, `[Global] clouds-file`, and `[Global] cloud` options are added.
- The entire `[BlockStorage]` section is removed. External cloud providers no longer perform storage operations. Block storage configuration is managed by the Cinder CSI driver.

Additionally, the CCM Operator enforces a number of default options. Values for these options are always overriden as follows:

```txt
[Global]
use-clouds = true
clouds-file = /etc/openstack/secret/clouds.yaml
cloud = openstack
...

[LoadBalancer]
enabled = true
```

The `clouds-value` value, `/etc/openstack/secret/clouds.yaml`, is mapped to the `openstack-cloud-credentials` config in the `openshift-cloud-controller-manager` namespace. You can modify the RHOSP cloud in this file as you do any other `clouds.yaml` file.

**Additional resources**

- [Cloud Controller Manager (Kubernetes documentation)](https://kubernetes.io/docs/concepts/architecture/cloud-controller/)

## The OpenStack Cloud Controller Manager (CCM) config map {#cluster-cloud-controller-config_installing-openstack-cloud-config-reference}

An RHOSP CCM config map defines how your cluster interacts with your RHOSP cloud. By default, the configuration is stored under the `cloud.conf` key in the `cloud-conf` config map in the `openshift-cloud-controller-manager` namespace.

:::warning

The `cloud-conf` config map is generated from the `cloud-provider-config` config map in the `openshift-config` namespace.

To change the settings that are described by the `cloud-conf` config map, modify the `cloud-provider-config` config map.

As part of this synchronization, the CCM Operator overrides some options. For more information, see "The RHOSP Cloud Controller Manager".

:::

For example:

```yaml title="An example cloud-conf config map"
apiVersion: v1
data:
  cloud.conf: |
    [Global]
    secret-name = openstack-credentials
    secret-namespace = kube-system
    region = regionOne
    [LoadBalancer]
    enabled = True
kind: ConfigMap
metadata:
  creationTimestamp: "2022-12-20T17:01:08Z"
  name: cloud-conf
  namespace: openshift-cloud-controller-manager
  resourceVersion: "2519"
  uid: cbbeedaf-41ed-41c2-9f37-4885732d3677
```

`apiVersion.data.cloud.conf`: Specifies global options by using a `clouds.yaml` file rather than modifying the config map.

The following options are present in the config map. Except when indicated otherwise, they are mandatory for clusters that run on RHOSP.

### Load balancer options {#ccm-config-lb-options_installing-openstack-cloud-config-reference}

You can configure load balancer options to control how the Cloud Controller Manager (CCM) creates and manages RHOSP Octavia load balancers for services in your cluster.

:::note

Neutron-LBaaS support is deprecated.

:::

<table>
<thead>
<tr>
  <th>Option</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>enabled</code></td>
  <td>Enables the <code>LoadBalancer</code> service type integration. The default value is <code>true</code>.</td>
</tr>
<tr>
  <td><code>floating-network-id</code></td>
  <td>Optional. The external network used to create floating IP addresses for load balancer virtual IP addresses (VIPs). If there are multiple external networks in the cloud, you must set this option or specify the <code>loadbalancer.openstack.org/floating-network-id</code> label in the service annotation.</td>
</tr>
<tr>
  <td><code>floating-subnet-id</code></td>
  <td>Optional. The external network subnet used to create floating IP addresses for the load balancer VIP. Can be overridden by the service annotation <code>loadbalancer.openstack.org/floating-subnet-id</code>.</td>
</tr>
<tr>
  <td><code>floating-subnet</code></td>
  <td>Optional. A name pattern (glob or regular expression if starting with <code>~</code>) for the external network subnet used to create floating IP addresses for the load balancer VIP. Can be overridden by the service annotation <code>loadbalancer.openstack.org/floating-subnet</code>. If multiple subnets match the pattern, the first one with available IP addresses is used.</td>
</tr>
<tr>
  <td><code>floating-subnet-tags</code></td>
  <td>Optional. Tags for the external network subnet used to create floating IP addresses for the load balancer VIP. Can be overridden by the service annotation <code>loadbalancer.openstack.org/floating-subnet-tags</code>. If multiple subnets match these tags, the first one with available IP addresses is used.<br><br>If the RHOSP network is configured with sharing disabled, for example, with the <code>--no-share</code> flag used during creation, this option is unsupported. Set the network to share to use this option.</td>
</tr>
<tr>
  <td><code>lb-method</code></td>
  <td>The load balancing algorithm used to create the load balancer pool. For the Amphora provider the value can be <code>ROUND_ROBIN</code>, <code>LEAST_CONNECTIONS</code>, or <code>SOURCE_IP</code>. The default value is <code>ROUND_ROBIN</code>.<br><br>For the OVN provider, only the <code>SOURCE_IP_PORT</code> algorithm is supported.<br><br>For the Amphora provider, if using the <code>LEAST_CONNECTIONS</code> or <code>SOURCE_IP</code> methods, configure the <code>create-monitor</code> option as <code>true</code> in the <code>cloud-provider-config</code> config map on the <code>openshift-config</code> namespace and <code>ETP:Local</code> on the load-balancer type service to allow balancing algorithm enforcement in the client to service endpoint connections.</td>
</tr>
<tr>
  <td><code>lb-provider</code></td>
  <td>Optional. Used to specify the provider of the load balancer, for example, <code>amphora</code> or <code>octavia</code>. Only the Amphora and Octavia providers are supported.</td>
</tr>
<tr>
  <td><code>lb-version</code></td>
  <td>Optional. The load balancer API version. Only <code>"v2"</code> is supported.</td>
</tr>
<tr>
  <td><code>subnet-id</code></td>
  <td>The ID of the Networking service subnet on which load balancer VIPs are created. For dual stack deployments, leave this option unset. The OpenStack cloud provider automatically selects which subnet to use for a load balancer.</td>
</tr>
<tr>
  <td><code>network-id</code></td>
  <td>The ID of the Networking service network on which load balancer VIPs are created. Unnecessary if <code>subnet-id</code> is set. If this property is not set, the network is automatically selected based on the network that cluster nodes use.</td>
</tr>
<tr>
  <td><code>create-monitor</code></td>
  <td>Creates a health monitor for the service load balancer. A health monitor is required for services that declare <code>externalTrafficPolicy: Local</code>. The default value is <code>false</code>.<br><br>This option is unsupported if you use RHOSP earlier than version 17 with the <code>ovn</code> provider.</td>
</tr>
<tr>
  <td><code>monitor-delay</code></td>
  <td>The interval in seconds by which probes are sent to members of the load balancer. The default value is <code>5</code>.</td>
</tr>
<tr>
  <td><code>monitor-max-retries</code></td>
  <td>The number of successful checks that are required to change the operating status of a load balancer member to <code>ONLINE</code>. The valid range is <code>1</code> to <code>10</code>, and the default value is <code>1</code>.</td>
</tr>
<tr>
  <td><code>monitor-timeout</code></td>
  <td>The time in seconds that a monitor waits to connect to the back end before it times out. The default value is <code>3</code>.</td>
</tr>
<tr>
  <td><code>internal-lb</code></td>
  <td>Whether or not to create an internal load balancer without floating IP addresses. The default value is <code>false</code>.</td>
</tr>
<tr>
  <td><code>LoadBalancerClass "ClassName"</code></td>
  <td>This is a config section that comprises a set of options:<br><br><ul><li><code>floating-network-id</code></li><li><code>floating-subnet-id</code></li><li><code>floating-subnet</code></li><li><code>floating-subnet-tags</code></li><li><code>network-id</code></li><li><code>subnet-id</code></li></ul>The behavior of these options is the same as that of the identically named options in the load balancer section of the CCM config file.<br><br>You can set the <code>ClassName</code> value by specifying the service annotation <code>loadbalancer.openstack.org/class</code>.</td>
</tr>
<tr>
  <td><code>max-shared-lb</code></td>
  <td>The maximum number of services that can share a load balancer. The default value is <code>2</code>.</td>
</tr>
</tbody>
</table>

### Options that the Operator overrides {#cluster-cloud-controller-config-overrides_installing-openstack-cloud-config-reference}

The CCM Operator overrides specific options, which you might recognize from configuring RHOSP. Do not configure these options. The options are for informational purposes only.

**Options overridden by the CCM Operator**

<table>
<thead>
<tr>
  <th>Option</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>auth-url</code></td>
  <td>The RHOSP Identity service URL. For example, <code>http://128.110.154.166/identity</code>.</td>
</tr>
<tr>
  <td><code>os-endpoint-type</code></td>
  <td>The type of endpoint to use from the service catalog.</td>
</tr>
<tr>
  <td><code>username</code></td>
  <td>The Identity service user name.</td>
</tr>
<tr>
  <td><code>password</code></td>
  <td>The Identity service user password.</td>
</tr>
<tr>
  <td><code>domain-id</code></td>
  <td>The Identity service user domain ID.</td>
</tr>
<tr>
  <td><code>domain-name</code></td>
  <td>The Identity service user domain name.</td>
</tr>
<tr>
  <td><code>tenant-id</code></td>
  <td>The Identity service project ID. Leave this option unset if you are using Identity service application credentials.<br><br>In version 3 of the Identity API, which changed the identifier <code>tenant</code> to <code>project</code>, the value of <code>tenant-id</code> is automatically mapped to the project construct in the API.</td>
</tr>
<tr>
  <td><code>tenant-name</code></td>
  <td>The Identity service project name.</td>
</tr>
<tr>
  <td><code>tenant-domain-id</code></td>
  <td>The Identity service project domain ID.</td>
</tr>
<tr>
  <td><code>tenant-domain-name</code></td>
  <td>The Identity service project domain name.</td>
</tr>
<tr>
  <td><code>user-domain-id</code></td>
  <td>The Identity service user domain ID.</td>
</tr>
<tr>
  <td><code>user-domain-name</code></td>
  <td>The Identity service user domain name.</td>
</tr>
<tr>
  <td><code>use-clouds</code></td>
  <td>Whether to fetch authorization credentials from a <code>clouds.yaml</code> file. Options set in this section are prioritized over values read from the <code>clouds.yaml</code> file.<br><br>The CCM Operator searches for the file in the following places:<br><br><ol><li>The value of the <code>clouds-file</code> option.</li><li>A file path stored in the environment variable <code>OS_CLIENT_CONFIG_FILE</code>.</li><li>The directory <code>pkg/openstack</code>.</li><li>The directory <code>~/.config/openstack</code>.</li><li>The directory <code>/etc/openstack</code>.</li></ol></td>
</tr>
<tr>
  <td><code>clouds-file</code></td>
  <td>The file path of a <code>clouds.yaml</code> file. It is used if the <code>use-clouds</code> option is set to <code>true</code>.</td>
</tr>
<tr>
  <td><code>cloud</code></td>
  <td>The named cloud in the <code>clouds.yaml</code> file that you want to use. It is used if the <code>use-clouds</code> option is set to <code>true</code>.</td>
</tr>
</tbody>
</table>
