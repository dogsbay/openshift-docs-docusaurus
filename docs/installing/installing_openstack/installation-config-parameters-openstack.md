---
title: Installation configuration parameters for OpenStack
sidebar_position: 12
---

# Installation configuration parameters for OpenStack {#installation-config-parameters-openstack}

<a id="installation-config-parameters-openstack"></a>

Before you deploy an OpenShift Container Platform cluster on Red Hat OpenStack Platform (RHOSP), provide parameters to customize your environment. Provide required parameters through the CLI when generating the `install-config.yaml` file. You can then further customize the file.

## Available installation configuration parameters for OpenStack {#installation-configuration-parameters_installation-config-parameters-openstack}

To customize your cluster installation, you can use configuration parameters in the `install-config.yaml` file.

The following tables specify the required, optional, and OpenStack-specific installation configuration parameters that you can set as part of the installation process.

:::warning

After installation, you cannot change these parameters in the `install-config.yaml` file.

:::

### Required configuration parameters {#installation-configuration-parameters-required_installation-config-parameters-openstack}

Required installation configuration parameters are described in the following table:

**Required parameters**

<table>
<thead>
<tr>
  <th>Parameter</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>apiVersion:</td>
  <td>The API version for the <code>install-config.yaml</code> content. The current version is <code>v1</code>. The installation program might also support older API versions.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>baseDomain:</td>
  <td>The base domain of your cloud provider. The base domain is used to create routes to your OpenShift Container Platform cluster components. The full DNS name for your cluster is a combination of the <code>baseDomain</code> and <code>metadata.name</code> parameter values that uses the <code>&lt;metadata.name&gt;.&lt;baseDomain&gt;</code> format.<br><br><strong>Value:</strong> A fully-qualified domain or subdomain name, such as <code>example.com</code>.</td>
</tr>
<tr>
  <td>metadata:</td>
  <td>Kubernetes resource <code>ObjectMeta</code>, from which only the <code>name</code> parameter is consumed.<br><br><strong>Value:</strong> Object</td>
</tr>
<tr>
  <td>metadata: name:</td>
  <td>The name of the cluster. DNS records for the cluster are all subdomains of <code>{.metadata.name}.{.baseDomain}</code>.  <br><br>  <strong>Value:</strong> String of lowercase letters, hyphens (<code>-</code>), and periods (<code>.</code>), such as <code>dev</code>.    The string must be 14 characters or fewer long. </td>
</tr>
<tr>
  <td>platform:</td>
  <td>The configuration for the specific platform upon which to perform the installation: <code>aws</code>, <code>baremetal</code>, <code>azure</code>, <code>gcp</code>, <code>ibmcloud</code>, <code>nutanix</code>, <code>openstack</code>, <code>powervs</code>, <code>vsphere</code>, or <code>{}</code>. For additional information about <code>platform.&lt;platform&gt;</code> parameters, consult the table for your specific platform that follows.<br><br><strong>Value:</strong> Object</td>
</tr>
<tr>
  <td>pullSecret:</td>
  <td>Get a <a href="https://console.redhat.com/openshift/install/pull-secret">pull secret from Red Hat OpenShift Cluster Manager</a> to authenticate downloading container images for OpenShift Container Platform components from services such as Quay.io.<br><br><strong>Value:</strong><pre><code>{&#10;   "auths":{&#10;      "cloud.openshift.com":{&#10;         "auth":"b3Blb=",&#10;         "email":"you@example.com"&#10;      },&#10;      "quay.io":{&#10;         "auth":"b3Blb=",&#10;         "email":"you@example.com"&#10;      }&#10;   }&#10;}</code></pre></td>
</tr>
</tbody>
</table>

### Network configuration parameters {#installation-configuration-parameters-network_installation-config-parameters-openstack}

You can customize your installation configuration based on the requirements of your existing network infrastructure. For example, you can expand the IP address block for the cluster network or configure different IP address blocks than the defaults.

Consider the following information before you configure network parameters for your cluster:

- If you use the Red Hat OpenShift Networking OVN-Kubernetes network plugin, both IPv4 and IPv6 address families are supported.
- If you deployed nodes in an OpenShift Container Platform cluster with a network that supports both IPv4 and non-link-local IPv6 addresses, configure your cluster to use a dual-stack network.
  - For clusters configured for dual-stack networking, both IPv4 and IPv6 traffic must use the same network interface as the default gateway. This ensures that in a multiple network interface controller (NIC) environment, a cluster can detect what NIC to use based on the available network interface. For more information, see "OVN-Kubernetes IPv6 and dual-stack limitations" in *About the OVN-Kubernetes network plugin*.
  - To prevent network connectivity issues, do not install a single-stack IPv4 cluster on a host that supports dual-stack networking.

If you configure your cluster to use both IP address families, review the following requirements:

- Both IP families must use the same network interface for the default gateway.
- Both IP families must have the default gateway.
- You must specify IPv4 and IPv6 addresses in the same order for all network configuration parameters. For example, in the following configuration, IPv4 addresses are listed before IPv6 addresses:
  ```yaml
  networking:
    clusterNetwork:
    - cidr: 10.128.0.0/14
      hostPrefix: 23
    - cidr: fd00:10:128::/56
      hostPrefix: 64
    serviceNetwork:
    - 172.30.0.0/16
    - fd00:172:16::/112
  ```

  If you are installing your cluster on AWS, the order of address families must match the `platform.aws.ipFamily` parameter. For example, if you specified the `DualStackIPv6Primary` parameter, you must list the IPv6 address first.

:::note

Globalnet is not supported with Red Hat OpenShift Data Foundation disaster recovery solutions. For regional disaster recovery scenarios, ensure that you use a non-overlapping range of private IP addresses for the cluster and service networks in each cluster.

:::

**Network parameters**

<table>
<thead>
<tr>
  <th>Parameter</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>networking:</td>
  <td>The configuration for the cluster network.<br><br><strong>Value:</strong> Object<br><br><div class="alert alert--secondary"><strong>Note</strong><p>You cannot change parameters specified by the <code>networking</code> object after installation.</p></div></td>
</tr>
<tr>
  <td>networking: networkType:</td>
  <td>The Red Hat OpenShift Networking network plugin to install.<br><br><strong>Value:</strong>    <code>OVNKubernetes</code>. <code>OVNKubernetes</code> is a Container Network Interface (CNI) plugin for Linux networks and hybrid networks that contain both Linux and Windows servers. The default value is <code>OVNKubernetes</code>.   </td>
</tr>
<tr>
  <td>networking: clusterNetwork:</td>
  <td>The IP address blocks for pods.<br><br>The default value is <code>10.128.0.0/14</code> with a host prefix of <code>/23</code>.<br><br>If you specify multiple IP address blocks, the blocks must not overlap.<br><br><strong>Value:</strong> An array of objects. For example:<br><br><pre><code>networking:&#10;  clusterNetwork:&#10;  - cidr: 10.128.0.0/14&#10;    hostPrefix: 23&#10;networking:&#10;  clusterNetwork:&#10;  - cidr: 10.128.0.0/14&#10;    hostPrefix: 23&#10;  - cidr: fd01::/48&#10;    hostPrefix: 64</code></pre></td>
</tr>
<tr>
  <td>networking: clusterNetwork: cidr:</td>
  <td>Required if you use <code>networking.clusterNetwork</code>. An IP address block.<br><br>  An IPv4 network.  <br><br> </td>
</tr>
<tr>
  <td>networking: clusterNetwork: hostPrefix:</td>
  <td>The subnet prefix length to assign to each individual node. For example, if <code>hostPrefix</code> is set to <code>23</code> then each node is assigned a <code>/23</code> subnet out of the given <code>cidr</code>. A <code>hostPrefix</code> value of <code>23</code> provides 510 (2^(32 - 23) - 2) pod IP addresses.<br><br><strong>Value:</strong> A subnet prefix.<br><br>  The default value is <code>23</code>.  <br><br> </td>
</tr>
<tr>
  <td>networking: serviceNetwork:</td>
  <td>The IP address block for services. The default value is <code>172.30.0.0/16</code>.<br><br>  <br><br><strong>Value:</strong> An array with an IP address block in CIDR format. For example:<br><br><pre><code>networking:&#10;  serviceNetwork:&#10;   - 172.30.0.0/16&#10;networking:&#10;  serviceNetwork:&#10;   - 172.30.0.0/16&#10;   - fd02::/112</code></pre></td>
</tr>
<tr>
  <td>networking: machineNetwork:</td>
  <td>The IP address blocks for machines.<br><br>  If you specify multiple IP address blocks, the blocks must not overlap.  <br><br>  <br><br><strong>Value:</strong> An array of objects. For example:<br><br><pre><code>networking:&#10;  machineNetwork:&#10;  - cidr: 10.0.0.0/16</code></pre></td>
</tr>
<tr>
  <td>networking: machineNetwork: cidr:</td>
  <td>Required if you use <code>networking.machineNetwork</code>. An IP address block. The default value is <code>10.0.0.0/16</code> for all platforms other than libvirt and IBM Power(R) Virtual Server. For libvirt, the default value is <code>192.168.126.0/24</code>. For IBM Power(R) Virtual Server, the default value is <code>192.168.0.0/24</code>.<br><br>  <br><br><strong>Value:</strong> An IP network block in CIDR notation.<br><br>  For example, <code>10.0.0.0/16</code>.    <br><br><div class="alert alert--secondary"><strong>Note</strong><p>Set the <code>networking.machineNetwork</code> to match the CIDR that the preferred NIC resides in.<br><br>If you are installing a cluster on AWS with dual-stack networking, consider the following distinction:<br><br><ul><li>If the installation program creates the VPC, do not specify an IPv6 entry in <code>networking.machineNetwork</code>. The installation program will assign an IPv6 address to the VPC.</li><li>If you provide existing dual-stack subnets using the <code>platform.aws.vpc.subnets</code> parameter, you must specify IPv6 entries corresponding to either the VPC CIDR or the CIDR of the subnets.</li><li>In both cases, you must provide an IPv4 CIDR entry.</li></ul></p></div></td>
</tr>
<tr>
  <td>networking: ovnKubernetesConfig: ipv4: internalJoinSubnet:</td>
  <td>Configures the IPv4 join subnet that is used internally by <code>ovn-kubernetes</code>. This subnet must not overlap with any other subnet that OpenShift Container Platform is using, including the node network. The size of the subnet must be larger than the number of nodes. You cannot change the value after installation.<br><br><strong>Value:</strong> An IP network block in CIDR notation. The default value is <code>100.64.0.0/16</code>.</td>
</tr>
</tbody>
</table>

### Optional configuration parameters {#installation-configuration-parameters-optional_installation-config-parameters-openstack}

Optional installation configuration parameters are described in the following table:

**Optional parameters**

<table>
<thead>
<tr>
  <th>Parameter</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>additionalTrustBundle:</td>
  <td>A PEM-encoded X.509 certificate bundle that is added to the nodes' trusted certificate store. This trust bundle might also be used when a proxy has been configured.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>capabilities:</td>
  <td>Controls the installation of optional core cluster components. You can reduce the footprint of your OpenShift Container Platform cluster by disabling optional components. For more information, see the "Cluster capabilities" page in <em>Installing</em>.<br><br><strong>Value:</strong> String array</td>
</tr>
<tr>
  <td>capabilities: baselineCapabilitySet:</td>
  <td>Selects an initial set of optional capabilities to enable. Valid values are <code>None</code>, <code>v4.11</code>, <code>v4.12</code> and <code>vCurrent</code>. The default value is <code>vCurrent</code>.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>capabilities: additionalEnabledCapabilities:</td>
  <td>Extends the set of optional capabilities beyond what you specify in <code>baselineCapabilitySet</code>. You can specify multiple capabilities in this parameter.<br><br><strong>Value:</strong> String array</td>
</tr>
<tr>
  <td>cpuPartitioningMode:</td>
  <td>Enables workload partitioning, which isolates OpenShift Container Platform services, cluster management workloads, and infrastructure pods to run on a reserved set of CPUs. You can only enable workload partitioning during installation. You cannot disable it after installation. While this field enables workload partitioning, it does not configure workloads to use specific CPUs. For more information, see the <em>Workload partitioning</em> page in the <em>Scalability and Performance</em> section.<br><br><strong>Value:</strong> <code>None</code> or <code>AllNodes</code>. <code>None</code> is the default value.</td>
</tr>
<tr>
  <td>compute:</td>
  <td>The configuration for the machines that comprise the compute nodes.<br><br><strong>Value:</strong> Array of <code>MachinePool</code> objects.<br><br>  <br><br>   </td>
</tr>
<tr>
  <td>compute: architecture:</td>
  <td>Determines the instruction set architecture of the machines in the pool. Currently, clusters with varied architectures are not supported. All pools must specify the same architecture. Valid values are <code>amd64</code> (the default).<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>compute: hyperthreading:</td>
  <td>Whether to enable or disable simultaneous multithreading, or <code>hyperthreading</code>, on compute machines. By default, simultaneous multithreading is enabled to increase the performance of your machines' cores.<br><br><div class="alert alert--warning"><strong>Important</strong><p>If you disable simultaneous multithreading, ensure that your capacity planning accounts for the dramatically decreased machine performance.</p></div><br><br><strong>Value:</strong> <code>Enabled</code> or <code>Disabled</code></td>
</tr>
<tr>
  <td>compute: name:</td>
  <td>Required if you use <code>compute</code>. The name of the machine pool.<br><br><strong>Value:</strong> <code>worker</code></td>
</tr>
<tr>
  <td>compute: platform:</td>
  <td>Required if you use <code>compute</code>. Use this parameter to specify the cloud provider to host the worker machines. This parameter value must match the <code>controlPlane.platform</code> parameter value.<br><br> </td>
</tr>
<tr>
  <td>compute: replicas:</td>
  <td>The number of compute machines, which are also known as worker machines, to provision.<br><br><strong>Value:</strong> A positive integer greater than or equal to <code>2</code>. The default value is <code>3</code>.</td>
</tr>
<tr>
  <td>featureSet:</td>
  <td>Enables the cluster for a feature set. A feature set is a collection of OpenShift Container Platform features that are not enabled by default. For more information about enabling a feature set during installation, see "Enabling features using feature gates".<br><br><strong>Value:</strong> String. The name of the feature set to enable, such as <code>TechPreviewNoUpgrade</code>.</td>
</tr>
<tr>
  <td>controlPlane:</td>
  <td>The configuration for the machines that form the control plane.<br><br><strong>Value:</strong> Array of <code>MachinePool</code> objects.</td>
</tr>
<tr>
  <td>controlPlane: architecture:</td>
  <td>Determines the instruction set architecture of the machines in the pool. Currently, clusters with varied architectures are not supported. All pools must specify the same architecture. Valid values are <code>amd64</code> (the default).<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>controlPlane: hyperthreading:</td>
  <td>Whether to enable or disable simultaneous multithreading, or <code>hyperthreading</code>, on control plane machines. By default, simultaneous multithreading is enabled to increase the performance of your machines' cores.<br><br><div class="alert alert--warning"><strong>Important</strong><p>If you disable simultaneous multithreading, ensure that your capacity planning accounts for the dramatically decreased machine performance.</p></div><br><br><strong>Value:</strong> <code>Enabled</code> or <code>Disabled</code></td>
</tr>
<tr>
  <td>controlPlane: name:</td>
  <td>Required if you use <code>controlPlane</code>. The name of the machine pool.<br><br><strong>Value:</strong> <code>master</code></td>
</tr>
<tr>
  <td>controlPlane: platform:</td>
  <td>Required if you use <code>controlPlane</code>. Use this parameter to specify the cloud provider that hosts the control plane machines. This parameter value must match the <code>compute.platform</code> parameter value.<br><br> </td>
</tr>
<tr>
  <td>controlPlane: replicas:</td>
  <td>The number of control plane machines to provision.<br><br><strong>Value:</strong>  Supported values are <code>3</code>, or <code>1</code> when deploying single-node OpenShift.  </td>
</tr>
<tr>
  <td>arbiter: name:</td>
  <td>The OpenShift Container Platform cluster requires a name for arbiter nodes. For example, <code>arbiter</code>.</td>
</tr>
<tr>
  <td>arbiter: replicas:</td>
  <td>The <code>replicas</code> parameter sets the number of arbiter nodes for the OpenShift Container Platform cluster. You cannot set this field to a value that is greater than 1.</td>
</tr>
<tr>
  <td>credentialsMode:</td>
  <td>The Cloud Credential Operator (CCO) mode. If no mode is specified, the CCO dynamically tries to determine the capabilities of the provided credentials, with a preference for mint mode on the platforms where multiple modes are supported.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>Not all CCO modes are supported for all cloud providers. For more information about CCO modes, see the "Managing cloud provider credentials" entry in the <em>Authentication and authorization</em> content.</p></div><br><br><strong>Value:</strong> <code>Mint</code>, <code>Passthrough</code>, <code>Manual</code> or an empty string (<code>""</code>).</td>
</tr>
<tr>
  <td>fips:</td>
  <td>Enable or disable FIPS mode. The default is <code>false</code> (disabled). If you enable FIPS mode, the Red&#160;Hat Enterprise Linux CoreOS (RHCOS) machines that OpenShift Container Platform runs on bypass the default Kubernetes cryptography suite and use the cryptography modules that RHCOS provides instead.<br><br>
<div class="alert alert--warning"><strong>Important</strong><p>To enable FIPS mode for your cluster, you must run the installation program from a Red&#160;Hat Enterprise Linux (RHEL) computer configured to operate in FIPS mode. For more information about configuring FIPS mode on RHEL, see <a href="https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/security_hardening/switching-rhel-to-fips-mode_security-hardening">Switching RHEL to FIPS mode</a>. When running Red&#160;Hat Enterprise Linux (RHEL) or Red&#160;Hat Enterprise Linux CoreOS (RHCOS) booted in FIPS mode, OpenShift Container Platform core components use the RHEL cryptographic libraries that have been submitted to NIST for FIPS 140-2/140-3 Validation on only the x86_64, ppc64le, and s390x architectures.</p></div><br><br><div class="alert alert--warning"><strong>Important</strong><p>If you are using Azure File storage, you cannot enable FIPS mode.</p></div><br><br><strong>Value:</strong> <code>false</code> or <code>true</code></td>
</tr>
<tr>
  <td>endpoint: name: <endpoint_name> clusterUseOnly: <code>true</code> or <code>false</code></td>
  <td>The <code>name</code> parameter contains the name of the Private Service Connect (PSC) endpoints.<br><br><div class="alert alert--warning"><strong>Important</strong><p>When <code>clusterUseOnly</code> is <code>false</code>, its default setting, you must run the installation program from a bastion host that is within the same VPC where you want to deploy the cluster.</p></div><br><br>When you want the installation program to use the public API endpoints and cluster Operators to use the API endpoint overrides, set <code>clusterUseOnly</code> to <code>true</code>. When you want both the installation program and the cluster Operators to use the API endpoint overrides, for example if you are running the installation program from a bastion host that is within the same VPC where you want to deploy the cluster, set <code>clusterUseOnly</code> to <code>false</code> . The parameter is optional and defaults to <code>false</code>.<br><br><strong>Value:</strong> String or boolean</td>
</tr>
<tr>
  <td>imageContentSources:</td>
  <td>Sources and repositories for the release-image content.<br><br><strong>Value:</strong> Array of objects. Includes a <code>source</code> and, optionally, <code>mirrors</code>, as described in the following rows of this table.</td>
</tr>
<tr>
  <td>imageContentSources: source:</td>
  <td>Required if you use <code>imageContentSources</code>. Specify the repository that users refer to, for example, in image pull specifications.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>imageContentSources: mirrors:</td>
  <td>Specify one or more repositories that might also contain the same images.<br><br><strong>Value:</strong> Array of strings</td>
</tr>
<tr>
  <td>osImageStream:</td>
  <td>Specifies the image stream that will be used for all machines in the cluster. <code>osImageStream</code> is a Technology Preview feature. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.<br><br><strong>Value:</strong> String. Valid values are <code>rhel-9</code> or <code>rhel-10</code>.</td>
</tr>
<tr>
  <td>publish:</td>
  <td>How to publish or expose the user-facing endpoints of your cluster, such as the Kubernetes API, OpenShift routes.<br><br><strong>Value:</strong>    <code>Internal</code> or <code>External</code>. The default value is <code>External</code>.<br><br>Setting this field to <code>Internal</code> is not supported on non-cloud platforms. </td>
</tr>
<tr>
  <td>sshKey:</td>
  <td>The SSH key to authenticate access to your cluster machines.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>For production OpenShift Container Platform clusters on which you want to perform installation debugging or disaster recovery, specify an SSH key that your <code>ssh-agent</code> process uses.</p></div><br><br><strong>Value:</strong> For example, <code>sshKey: ssh-ed25519 AAAA..</code>.</td>
</tr>
</tbody>
</table>

### Additional Red Hat OpenStack Platform (RHOSP) configuration parameters {#installation-configuration-parameters-additional-osp_installation-config-parameters-openstack}

Additional RHOSP configuration parameters are described in the following table:

**Additional RHOSP parameters**

<table>
<thead>
<tr>
  <th>Parameter</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>compute: platform: openstack: rootVolume: size:</td>
  <td>For compute machines, the size in gigabytes of the root volume. If you do not set this value, machines use ephemeral storage.<br><br><strong>Value:</strong> Integer, for example <code>30</code>.</td>
</tr>
<tr>
  <td>compute: platform: openstack: rootVolume: types:</td>
  <td>For compute machines, the root volume types.<br><br><strong>Value:</strong> A list of strings, for example, {<code>performance-host1</code>, <code>performance-host2</code>, <code>performance-host3</code>}. <sup>[1]</sup></td>
</tr>
<tr>
  <td>compute: platform: openstack: rootVolume: type:</td>
  <td>For compute machines, the root volume's type. This property is deprecated and is replaced by <code>compute.platform.openstack.rootVolume.types</code>.<br><br><strong>Value:</strong> String, for example, <code>performance</code>. <sup>[2]</sup></td>
</tr>
<tr>
  <td>compute: platform: openstack: rootVolume: zones:</td>
  <td>For compute machines, the Cinder availability zone to install root volumes on. If you do not set a value for this parameter, the installation program selects the default availability zone. This parameter is mandatory when <code>compute.platform.openstack.zones</code> is defined.<br><br><strong>Value:</strong> A list of strings, for example <code>["zone-1", "zone-2"]</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: openstack: rootVolume: size:</td>
  <td>For control plane machines, the size in gigabytes of the root volume. If you do not set this value, machines use ephemeral storage.<br><br><strong>Value:</strong> Integer, for example <code>30</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: openstack: rootVolume: types:</td>
  <td>For control plane machines, the root volume types.<br><br><strong>Value:</strong> A list of strings, for example, {<code>performance-host1</code>, <code>performance-host2</code>, <code>performance-host3</code>}. <sup>[1]</sup></td>
</tr>
<tr>
  <td>controlPlane: platform: openstack: rootVolume: type:</td>
  <td>For control plane machines, the root volume's type. This property is deprecated and is replaced by <code>compute.platform.openstack.rootVolume.types</code>.<br><br><strong>Value:</strong> String, for example, <code>performance</code>. <sup>[2]</sup></td>
</tr>
<tr>
  <td>controlPlane: platform: openstack: rootVolume: zones:</td>
  <td>For control plane machines, the Cinder availability zone to install root volumes on. If you do not set this value, the installation program selects the default availability zone. This parameter is mandatory when <code>controlPlane.platform.openstack.zones</code> is defined.<br><br><strong>Value:</strong> A list of strings, for example <code>["zone-1", "zone-2"]</code>.</td>
</tr>
<tr>
  <td>platform: openstack: cloud:</td>
  <td>The name of the RHOSP cloud to use from the list of clouds in the <code>clouds.yaml</code> file.<br><br>In the cloud configuration in the <code>clouds.yaml</code> file, if possible, use application credentials rather than a user name and password combination. Using application credentials avoids disruptions from secret propagation that follow user name and password rotation.<br><br><strong>Value:</strong> String, for example <code>MyCloud</code>.</td>
</tr>
<tr>
  <td>platform: openstack: externalNetwork:</td>
  <td>The RHOSP external network name to be used for installation.<br><br><strong>Value:</strong> String, for example <code>external</code>.</td>
</tr>
<tr>
  <td>platform: openstack: computeFlavor:</td>
  <td>The RHOSP flavor to use for control plane and compute machines.<br><br>This property is deprecated. To use a flavor as the default for all machine pools, add it as the value of the <code>type</code> key in the <code>platform.openstack.defaultMachinePlatform</code> property. You can also set a flavor value for each machine pool individually.<br><br><strong>Value:</strong> String, for example <code>m1.xlarge</code>.</td>
</tr>
</tbody>
</table>

1. If the machine pool defines `zones`, the count of types can either be a single item or match the number of items in `zones`. For example, the count of types cannot be 2 if there are 3 items in `zones`.
2. If you have any existing reference to this property, the installation program populates the corresponding value in the `controlPlane.platform.openstack.rootVolume.types` field.

### Optional RHOSP configuration parameters {#installation-configuration-parameters-optional-osp_installation-config-parameters-openstack}

Optional RHOSP configuration parameters are described in the following table:

**Optional RHOSP parameters**

<table>
<thead>
<tr>
  <th>Parameter</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>compute: platform: openstack: additionalNetworkIDs:</td>
  <td>Additional networks that are associated with compute machines. Allowed address pairs are not created for additional networks.<br><br><strong>Value:</strong> A list of one or more UUIDs as strings. For example, <code>fa806b2f-ac49-4bce-b9db-124bc64209bf</code>.</td>
</tr>
<tr>
  <td>compute: platform: openstack: additionalSecurityGroupIDs:</td>
  <td>Additional security groups that are associated with compute machines.<br><br><strong>Value:</strong> A list of one or more UUIDs as strings. For example, <code>7ee219f3-d2e9-48a1-96c2-e7429f1b0da7</code>.</td>
</tr>
<tr>
  <td>compute: platform: openstack: zones:</td>
  <td>RHOSP Compute (Nova) availability zones (AZs) to install machines on. If this parameter is not set, the installation program relies on the default settings for Nova that the RHOSP administrator configured.<br><br><strong>Value:</strong> A list of strings. For example, <code>["zone-1", "zone-2"]</code>.</td>
</tr>
<tr>
  <td>compute: platform: openstack: serverGroupPolicy:</td>
  <td>The server group policy to apply to the group that contains the compute machines in the pool. You cannot change server group policies or affiliations after creation. Supported options include <code>anti-affinity</code>, <code>soft-affinity</code>, and <code>soft-anti-affinity</code>. The default value is <code>soft-anti-affinity</code>.<br><br>An <code>affinity</code> policy prevents migrations and therefore affects RHOSP upgrades. The <code>affinity</code> policy is not supported.<br><br>If you use a strict <code>anti-affinity</code> policy, an additional RHOSP host is required during instance migration.<br><br><strong>Value:</strong> A server group policy to apply to the machine pool. For example, <code>soft-affinity</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: openstack: additionalNetworkIDs:</td>
  <td>Additional networks that are associated with control plane machines. Allowed address pairs are not created for additional networks.<br><br>Additional networks that are attached to a control plane machine are also attached to the bootstrap node.<br><br><strong>Value:</strong> A list of one or more UUIDs as strings. For example, <code>fa806b2f-ac49-4bce-b9db-124bc64209bf</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: openstack: additionalSecurityGroupIDs:</td>
  <td>Additional security groups that are associated with control plane machines.<br><br><strong>Value:</strong> A list of one or more UUIDs as strings. For example, <code>7ee219f3-d2e9-48a1-96c2-e7429f1b0da7</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: openstack: zones:</td>
  <td>RHOSP Compute (Nova) availability zones (AZs) to install machines on. If this parameter is not set, the installation program relies on the default settings for Nova that the RHOSP administrator configured.<br><br><strong>Value:</strong> A list of strings. For example, <code>["zone-1", "zone-2"]</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: openstack: serverGroupPolicy:</td>
  <td>Server group policy to apply to the group that contains the control plane machines in the pool. You cannot change server group policies or affiliations after creation. Supported options include <code>anti-affinity</code>, <code>soft-affinity</code>, and <code>soft-anti-affinity</code>. The default value is <code>soft-anti-affinity</code>.<br><br>An <code>affinity</code> policy prevents migrations, and therefore affects RHOSP upgrades. The <code>affinity</code> policy is not supported.<br><br>If you use a strict <code>anti-affinity</code> policy, an additional RHOSP host is required during instance migration.<br><br><strong>Value:</strong> A server group policy to apply to the machine pool. For example, <code>soft-affinity</code>.</td>
</tr>
<tr>
  <td>platform: openstack: apiVIPs:</td>
  <td>IP address on the machine network to assign to the API VIP. If multiple addresses are present, they must consist of exactly one IPv4 and one IPv6 address.<br><br><strong>Value:</strong> An array of strings. For example, <code>[ "192.168.1.10", "2001:db8::10" ]</code>.</td>
</tr>
<tr>
  <td>platform: openstack: clusterOSImage:</td>
  <td>The location from which the installation program downloads the RHCOS image.<br><br>You must set this parameter to perform an installation in a restricted network.<br><br><strong>Value:</strong> An HTTP or HTTPS URL, optionally with an SHA-256 checksum.<br><br>For example, <code>\http://mirror.example.com/images/rhcos-43.81.201912131630.0-openstack.x86_64.qcow2.gz?sha256=ffebbd68e8a1f2a245ca19522c16c86f67f9ac8e4e0c1f0a812b068b16f7265d</code>. The value can also be the name of an existing Glance image, for example <code>my-rhcos</code>.</td>
</tr>
<tr>
  <td>platform: openstack: clusterOSImageProperties:</td>
  <td>Properties to add to the installation program-uploaded ClusterOSImage in Glance. This property is ignored if <code>platform.openstack.clusterOSImage</code> is set to an existing Glance image.<br><br>You can use this property to exceed the default persistent volume (PV) limit for RHOSP of 26 PVs per node. To exceed the limit, set the <code>hw_scsi_model</code> property value to <code>virtio-scsi</code> and the <code>hw_disk_bus</code> value to  <code>scsi</code>.<br><br>You can also use this property to enable the QEMU guest agent by including the <code>hw_qemu_guest_agent</code> property with a value of <code>yes</code>.<br><br><strong>Value:</strong> A set of string properties. For example:<br><br><pre><code>clusterOSImageProperties:&#10;    hw_scsi_model: "virtio-scsi"&#10;    hw_disk_bus: "scsi"&#10;    hw_qemu_guest_agent: "yes"</code></pre></td>
</tr>
<tr>
  <td>platform: openstack: controlPlanePort: fixedIPs:</td>
  <td>Subnets for the machines to use.<br><br><strong>Value:</strong> A list of subnet names or UUIDs to use in cluster installation.</td>
</tr>
<tr>
  <td>platform: openstack: controlPlanePort: network:</td>
  <td>A network for the machines to use.<br><br><strong>Value:</strong> The UUID or name of an RHOSP network to use in cluster installation.</td>
</tr>
<tr>
  <td>platform: openstack: defaultMachinePlatform:</td>
  <td>The default machine pool platform configuration.<br><br><strong>Value:</strong><pre><code>{&#10;   "type": "ml.large",&#10;   "rootVolume": {&#10;      "size": 30,&#10;      "type": "performance"&#10;   }&#10;}</code></pre></td>
</tr>
<tr>
  <td>platform: openstack: ingressFloatingIP:</td>
  <td>An existing floating IP address to associate with the Ingress port. To use this property, you must also define the <code>platform.openstack.externalNetwork</code> property.<br><br><strong>Value:</strong> An IP address, for example <code>128.0.0.1</code>.</td>
</tr>
<tr>
  <td>platform: openstack: ingressVIPs:</td>
  <td>An IP address or addresses on the machine network to assign to the ingress VIP. If multiple addresses are provided, they must consist of exactly one IPv4 and one IPv6 address.<br><br><strong>Value:</strong> An array of strings. For example, <code>[ "192.168.1.11", "2001:db8::11" ]</code>.</td>
</tr>
<tr>
  <td>platform: openstack: apiFloatingIP:</td>
  <td>An existing floating IP address to associate with the API load balancer. To use this property, you must also define the <code>platform.openstack.externalNetwork</code> property.<br><br><strong>Value:</strong> An IP address, for example <code>128.0.0.1</code>.</td>
</tr>
<tr>
  <td>platform: openstack: externalDNS:</td>
  <td>IP addresses for external DNS servers that cluster instances use for DNS resolution.<br><br><strong>Value:</strong> A list of IP addresses as strings. For example, <code>["8.8.8.8", "192.168.1.12"]</code>.</td>
</tr>
<tr>
  <td>platform: openstack: loadbalancer:</td>
  <td>Whether or not to use the default, internal load balancer. If the value is set to <code>UserManaged</code>, this default load balancer is disabled so that you can deploy a cluster that uses an external, user-managed load balancer. If the parameter is not set, or if the value is <code>OpenShiftManagedDefault</code>, the cluster uses the default load balancer.<br><br><strong>Value:</strong> <code>UserManaged</code> or <code>OpenShiftManagedDefault</code>.</td>
</tr>
<tr>
  <td>platform: openstack: machinesSubnet:</td>
  <td>The UUID of a RHOSP subnet that the cluster's nodes use. Nodes and virtual IP (VIP) ports are created on this subnet.<br><br>The first item in <code>networking.machineNetwork</code> must match the value of <code>machinesSubnet</code>.<br><br>If you deploy to a custom subnet, you cannot specify an external DNS server to the OpenShift Container Platform installer. Instead, <a href="https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/16.0/html/command_line_interface_reference/subnet">add DNS to the subnet in RHOSP</a>.<br><br><strong>Value:</strong> A UUID as a string. For example, <code>fa806b2f-ac49-4bce-b9db-124bc64209bf</code>.</td>
</tr>
</tbody>
</table>
