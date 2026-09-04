---
title: Installation configuration parameters for the Agent-based Installer
sidebar_position: 9
---

# Installation configuration parameters for the Agent-based Installer {#installation-config-parameters-agent}

<a id="installation-config-parameters-agent"></a>

Before you deploy an OpenShift Container Platform cluster using the Agent-based Installer, you provide parameters to customize your cluster and the platform that hosts it.

When you create the `install-config.yaml` and `agent-config.yaml` files, you must provide values for the required parameters, and you can use the optional parameters to customize your cluster further.

## Available installation configuration parameters {#_available_installation_configuration_parameters}

To customize your cluster installation, you can use configuration parameters in the `install-config.yaml` file.

The following tables specify the required and optional installation configuration parameters that you can set as part of the Agent-based installation process.

These values are specified in the `install-config.yaml` file.

:::warning

These settings are used for installation only, and cannot be changed after installation.

:::

### Required configuration parameters {#installation-configuration-parameters-required_installation-config-parameters-agent}

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
  <td>The name of the cluster. DNS records for the cluster are all subdomains of <code>{.metadata.name}.{.baseDomain}</code>.  The cluster name is set to <code>agent-cluster</code> when you do not provide the <code>metadata.name</code> parameter through either the <code>install-config.yaml</code> or <code>agent-config.yaml</code> files. For example, installations that only use ZTP manifests do not provide the <code>metadata.name</code> parameter.  <br><br>  <strong>Value:</strong> String of lowercase letters, hyphens (<code>-</code>), and periods (<code>.</code>), such as <code>dev</code>.   </td>
</tr>
<tr>
  <td>platform:</td>
  <td>The configuration for the specific platform upon which to perform the installation: <code>baremetal</code>, <code>external</code>, <code>none</code>, <code>vsphere</code>, or <code>nutanix</code>.<br><br><strong>Value:</strong> Object</td>
</tr>
<tr>
  <td>pullSecret:</td>
  <td>Get a <a href="https://console.redhat.com/openshift/install/pull-secret">pull secret from Red Hat OpenShift Cluster Manager</a> to authenticate downloading container images for OpenShift Container Platform components from services such as Quay.io.<br><br><strong>Value:</strong><pre><code>{&#10;   "auths":{&#10;      "cloud.openshift.com":{&#10;         "auth":"b3Blb=",&#10;         "email":"you@example.com"&#10;      },&#10;      "quay.io":{&#10;         "auth":"b3Blb=",&#10;         "email":"you@example.com"&#10;      }&#10;   }&#10;}</code></pre></td>
</tr>
</tbody>
</table>

### Network configuration parameters {#installation-configuration-parameters-network_installation-config-parameters-agent}

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
  <td>Required if you use <code>networking.clusterNetwork</code>. An IP address block.<br><br>  <br><br>  If you use the OVN-Kubernetes network plugin, you can specify IPv4 and IPv6 networks.<br><br><strong>Value:</strong> An IP address block in Classless Inter-Domain Routing (CIDR) notation. The prefix length for an IPv4 block is between <code>0</code> and <code>32</code>. The prefix length for an IPv6 block is between <code>0</code> and <code>128</code>. For example, <code>10.128.0.0/14</code> or <code>fd01::/48</code>. </td>
</tr>
<tr>
  <td>networking: clusterNetwork: hostPrefix:</td>
  <td>The subnet prefix length to assign to each individual node. For example, if <code>hostPrefix</code> is set to <code>23</code> then each node is assigned a <code>/23</code> subnet out of the given <code>cidr</code>. A <code>hostPrefix</code> value of <code>23</code> provides 510 (2^(32 - 23) - 2) pod IP addresses.<br><br><strong>Value:</strong> A subnet prefix.<br><br>  <br><br>  For an IPv4 network the default value is <code>23</code>. For an IPv6 network <code>hostPrefix</code> must be set to <code>64</code>, which is the default value. </td>
</tr>
<tr>
  <td>networking: serviceNetwork:</td>
  <td>The IP address block for services. The default value is <code>172.30.0.0/16</code>.<br><br>  If you use the OVN-Kubernetes network plugin, you can specify an IP address block for both of the IPv4 and IPv6 address families.  <br><br><strong>Value:</strong> An array with an IP address block in CIDR format. For example:<br><br><pre><code>networking:&#10;  serviceNetwork:&#10;   - 172.30.0.0/16&#10;networking:&#10;  serviceNetwork:&#10;   - 172.30.0.0/16&#10;   - fd02::/112</code></pre></td>
</tr>
<tr>
  <td>networking: machineNetwork:</td>
  <td>The IP address blocks for machines.<br><br>  If you specify multiple IP address blocks, the blocks must not overlap.  <br><br>  <br><br><strong>Value:</strong> An array of objects. For example:<br><br><pre><code>networking:&#10;  machineNetwork:&#10;  - cidr: 10.0.0.0/16</code></pre></td>
</tr>
<tr>
  <td>networking: machineNetwork: cidr:</td>
  <td>Required if you use <code>networking.machineNetwork</code>. An IP address block. The default value is <code>10.0.0.0/16</code> for all platforms other than libvirt and IBM Power(R) Virtual Server. For libvirt, the default value is <code>192.168.126.0/24</code>. For IBM Power(R) Virtual Server, the default value is <code>192.168.0.0/24</code>.<br><br>  <br><br><strong>Value:</strong> An IP network block in CIDR notation.<br><br>   For example, <code>10.0.0.0/16</code> or <code>fd00::/48</code>.   <br><br><div class="alert alert--secondary"><strong>Note</strong><p>Set the <code>networking.machineNetwork</code> to match the CIDR that the preferred NIC resides in.<br><br>If you are installing a cluster on AWS with dual-stack networking, consider the following distinction:<br><br><ul><li>If the installation program creates the VPC, do not specify an IPv6 entry in <code>networking.machineNetwork</code>. The installation program will assign an IPv6 address to the VPC.</li><li>If you provide existing dual-stack subnets using the <code>platform.aws.vpc.subnets</code> parameter, you must specify IPv6 entries corresponding to either the VPC CIDR or the CIDR of the subnets.</li><li>In both cases, you must provide an IPv4 CIDR entry.</li></ul></p></div></td>
</tr>
<tr>
  <td>networking: ovnKubernetesConfig: ipv4: internalJoinSubnet:</td>
  <td>Configures the IPv4 join subnet that is used internally by <code>ovn-kubernetes</code>. This subnet must not overlap with any other subnet that OpenShift Container Platform is using, including the node network. The size of the subnet must be larger than the number of nodes. You cannot change the value after installation.<br><br><strong>Value:</strong> An IP network block in CIDR notation. The default value is <code>100.64.0.0/16</code>.</td>
</tr>
</tbody>
</table>

### Optional configuration parameters {#installation-configuration-parameters-optional_installation-config-parameters-agent}

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
  <td>Determines the instruction set architecture of the machines in the pool. Currently, clusters with varied architectures are not supported. All pools must specify the same architecture. Valid values are <code>amd64</code>, <code>arm64</code>, <code>ppc64le</code>, and <code>s390x</code>.<br><br><strong>Value:</strong> String</td>
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
  <td>Determines the instruction set architecture of the machines in the pool. Currently, clusters with varied architectures are not supported. All pools must specify the same architecture. Valid values are <code>amd64</code>, <code>arm64</code>, <code>ppc64le</code>, and <code>s390x</code>.<br><br><strong>Value:</strong> String</td>
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
  <td>The number of control plane machines to provision.<br><br><strong>Value:</strong>   Supported values are <code>3</code>, <code>4</code>, <code>5</code>, or <code>1</code> when deploying single-node OpenShift. </td>
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

### Additional bare metal configuration parameters for the Agent-based Installer {#installation-configuration-parameters-additional-bare_installation-config-parameters-agent}

Additional bare metal installation configuration parameters for the Agent-based Installer are described in the following table:

:::note

These fields are not used during the initial provisioning of the cluster, but they are available to use once the cluster has been installed. Configuring these fields at install time eliminates the need to set them as a Day 2 operation.

:::

**Additional bare metal parameters**

<table>
<thead>
<tr>
  <th>Parameter</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>platform: baremetal: clusterProvisioningIP:</td>
  <td>The IP address within the cluster where the provisioning services run. Defaults to the third IP address of the provisioning subnet. For example, <code>172.22.0.3</code> or <code>2620:52:0:1307::3</code>.<br><br><strong>Value:</strong> IPv4 or IPv6 address.</td>
</tr>
<tr>
  <td>platform: baremetal: provisioningNetwork:</td>
  <td>The <code>provisioningNetwork</code> configuration setting determines whether the cluster uses the provisioning network. If it does, the configuration setting also determines if the cluster manages the network.<br><br><code>Managed</code>: Default. Set this parameter to <code>Managed</code> to fully manage the provisioning network, including DHCP, TFTP, and so on.<br><br><code>Disabled</code>: Set this parameter to <code>Disabled</code> to disable the requirement for a provisioning network. When set to <code>Disabled</code>, you can use only virtual media based provisioning on Day 2. If <code>Disabled</code> and using power management, BMCs must be accessible from the bare-metal network. If Disabled, you must provide two IP addresses on the bare-metal network that are used for the provisioning services.<br><br><strong>Value:</strong> <code>Managed</code> or <code>Disabled</code>.</td>
</tr>
<tr>
  <td>platform: baremetal: provisioningMACAddress:</td>
  <td>The MAC address within the cluster where provisioning services run.<br><br><strong>Value:</strong> MAC address.</td>
</tr>
<tr>
  <td>platform: baremetal: provisioningNetworkCIDR:</td>
  <td>The CIDR for the network to use for provisioning. This option is required when not using the default address range on the provisioning network.<br><br><strong>Value:</strong> Valid CIDR, for example <code>10.0.0.0/16</code>.</td>
</tr>
<tr>
  <td>platform: baremetal: provisioningNetworkInterface:</td>
  <td>The name of the network interface on nodes connected to the provisioning network. Use the <code>bootMACAddress</code> configuration setting to enable Ironic to identify the IP address of the NIC instead of using the <code>provisioningNetworkInterface</code> configuration setting to identify the name of the NIC.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>platform: baremetal: provisioningDHCPRange:</td>
  <td>Defines the IP range for nodes on the provisioning network, for example <code>172.22.0.10,172.22.0.254</code>.<br><br><strong>Value:</strong> IP address range.</td>
</tr>
<tr>
  <td>platform: baremetal: hosts:</td>
  <td>Configuration for bare metal hosts.<br><br><strong>Value:</strong> Array of host configuration objects.</td>
</tr>
<tr>
  <td>platform: baremetal: hosts: name:</td>
  <td>The name of the host.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>platform: baremetal: hosts: bootMACAddress:</td>
  <td>The MAC address of the NIC used for provisioning the host.<br><br><strong>Value:</strong> MAC address.</td>
</tr>
<tr>
  <td>platform: baremetal: hosts: bmc:</td>
  <td>Configuration for the host to connect to the baseboard management controller (BMC).<br><br><strong>Value:</strong> Dictionary of BMC configuration objects.</td>
</tr>
<tr>
  <td>platform: baremetal: hosts: bmc: username:</td>
  <td>The username for the BMC.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>platform: baremetal: hosts: bmc: password:</td>
  <td>Password for the BMC.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>platform: baremetal: hosts: bmc: address:</td>
  <td>The URL for communicating with the host's BMC controller. The address configuration setting specifies the protocol. For example, <code>redfish+http://10.10.10.1:8000/redfish/v1/Systems/1234</code> enables Redfish. For more information, see "BMC addressing" in the "Deploying installer-provisioned clusters on bare metal" section.<br><br><strong>Value:</strong> URL.</td>
</tr>
<tr>
  <td>platform: baremetal: hosts: bmc: disableCertificateVerification:</td>
  <td><code>redfish</code> and <code>redfish-virtualmedia</code> need this parameter to manage BMC addresses. The value should be <code>True</code> when using a self-signed certificate for BMC addresses.<br><br><strong>Value:</strong> Boolean.</td>
</tr>
</tbody>
</table>

### Additional VMware vSphere configuration parameters {#installation-configuration-parameters-additional-vsphere_installation-config-parameters-agent}

Additional VMware vSphere configuration parameters are described in the following table:

**Additional VMware vSphere cluster parameters**

<table>
<thead>
<tr>
  <th>Parameter</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>platform: vsphere:</td>
  <td>Describes your account on the cloud platform that hosts your cluster. You can use the parameter to customize the platform. If you provide additional configuration settings for compute and control plane machines in the machine pool, the parameter is not required.<br><br><strong>Value:</strong> A dictionary of vSphere configuration objects</td>
</tr>
<tr>
  <td>platform: vsphere: failureDomains:</td>
  <td>Establishes the relationships between a region and zone. You define a failure domain by using vCenter objects, such as a <code>datastore</code> object. A failure domain defines the vCenter location for OpenShift Container Platform cluster nodes.<br><br><strong>Value:</strong> An array of failure domain configuration objects.</td>
</tr>
<tr>
  <td>platform: vsphere: failureDomains: name:</td>
  <td>The name of the failure domain.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: failureDomains: region:</td>
  <td>If you define multiple failure domains for your cluster, you must attach the tag to each vCenter data center. To define a region, use a tag from the <code>openshift-region</code> tag category. For a single vSphere data center environment, you do not need to attach a tag, but you must enter an alphanumeric value, such as <code>datacenter</code>, for the parameter. If you want to base your failure domains on host groups, attach these tags to your vSphere clusters instead of your data centers.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: failureDomains: regionType:</td>
  <td>Specifies the <code>ComputeCluster</code> region type to enable host groups.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: failureDomains: server:</td>
  <td>Specifies the fully-qualified hostname or IP address of the VMware vCenter server, so that a client can access failure domain resources. You must apply the <code>server</code> role to the vSphere vCenter server location.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: failureDomains: zone:</td>
  <td>If you define multiple failure domains for your cluster, you must attach a tag to each vCenter cluster. To define a zone, use a tag from the <code>openshift-zone</code> tag category. For a single vSphere data center environment, you do not need to attach a tag, but you must enter an alphanumeric value, such as <code>cluster</code>, for the parameter. If you want to base your failure domains on host groups, define zones that correspond to your host groups instead of your clusters. Use these tags to associate each ESXi host with its host group.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: failureDomains: zoneType:</td>
  <td>Specifies the <code>HostGroup</code> zone type to enable host groups.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: failureDomains: topology: computeCluster:</td>
  <td>The path to the vSphere compute cluster.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: failureDomains: topology: datacenter:</td>
  <td>Lists and defines the data centers where OpenShift Container Platform virtual machines (VMs) operate. The list of data centers must match the list of data centers specified in the <code>vcenters</code> field.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: failureDomains: topology: datastore:</td>
  <td>The path to the vSphere datastore that holds virtual machine files, templates, and ISO images.<div class="alert alert--warning"><strong>Important</strong><p>You can specify the path of any datastore that exists in a datastore cluster. By default, Storage vMotion is automatically enabled for a datastore cluster. Red&#160;Hat does not support Storage vMotion, so you must disable Storage vMotion to avoid data loss issues for your OpenShift Container Platform cluster.<br><br>If you must specify VMs across multiple datastores, use a <code>datastore</code> object to specify a failure domain in your cluster's <code>install-config.yaml</code> configuration file. For more information, see "VMware vSphere region and zone enablement".</p></div><br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: failureDomains: topology: folder:</td>
  <td>Optional: The absolute path of an existing folder where the user creates the virtual machines, for example, <code>/&lt;data_center_name&gt;/vm/&lt;folder_name&gt;/&lt;subfolder_name&gt;</code>.  <br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: failureDomains: topology: hostGroup:</td>
  <td>Specifies the vSphere host group to associate with the failure domain.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: failureDomains: topology: networks:</td>
  <td>Lists any network in the vCenter instance that contains the virtual IP addresses and DNS records that you configured.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: failureDomains: topology: resourcePool:</td>
  <td>Optional: The absolute path of an existing resource pool where the installation program creates the virtual machines, for example, <code>/&lt;data_center_name&gt;/host/&lt;cluster_name&gt;/Resources/&lt;resource_pool_name&gt;/&lt;optional_nested_resource_pool_name&gt;</code>.  <br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: failureDomains: topology: tagIDs:</td>
  <td>Optional: Specifies the ID of the tag to be associated by the installation program. Each VM created by OpenShift Container Platform is assigned a unique tag that is specific to the cluster. The assigned tag enables the installation program to identify and remove the associated VMs when a cluster is decommissioned. You can list up to ten additional tag IDs to be attached to the VMs provisioned by the installation program. For more information about determining the tag ID, see the <a href="https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.vcenterhost.doc/GUID-E8E854DD-AA97-4E0C-8419-CE84F93C4058.html">vSphere Tags and Attributes documentation</a>.<br><br><strong>Value:</strong> String, for example <code>urn:vmomi:InventoryServiceTag:208e713c-cae3-4b7f-918e-4051ca7d1f97:GLOBAL</code>.</td>
</tr>
<tr>
  <td>platform: vsphere: failureDomains: topology: template:</td>
  <td>Specifies the absolute path to a pre-existing Red&#160;Hat Enterprise Linux CoreOS (RHCOS) image template or virtual machine. The installation program can use the image template or virtual machine to quickly install RHCOS on vSphere hosts. Consider using this parameter as an alternative to uploading an RHCOS image on vSphere hosts. This parameter is available for use only on installer-provisioned infrastructure.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: vcenters:</td>
  <td>Configures the connection details so that services can communicate with a vCenter server.<br><br><strong>Value:</strong> An array of vCenter configuration objects.</td>
</tr>
<tr>
  <td>platform: vsphere: vcenters: datacenters:</td>
  <td>Lists and defines the data centers where OpenShift Container Platform virtual machines (VMs) operate. The list of data centers must match the list of data centers specified in the <code>failureDomains</code> field.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: vcenters: password:</td>
  <td>The password associated with the vSphere user.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: vcenters: port:</td>
  <td>The port number used to communicate with the vCenter server.<br><br><strong>Value:</strong> Integer</td>
</tr>
<tr>
  <td>platform: vsphere: vcenters: server:</td>
  <td>The fully qualified host name (FQHN) or IP address of the vCenter server.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: vcenters: user:</td>
  <td>The username associated with the vSphere user.<br><br><strong>Value:</strong> String</td>
</tr>
</tbody>
</table>

### Deprecated VMware vSphere configuration parameters {#deprecated-parameters-vsphere_installation-config-parameters-agent}

In OpenShift Container Platform 4.13, the following vSphere configuration parameters are deprecated. You can continue to use these parameters, but the installation program does not automatically specify these parameters in the `install-config.yaml` file.

The following table lists each deprecated vSphere configuration parameter:

**Deprecated VMware vSphere cluster parameters**

<table>
<thead>
<tr>
  <th>Parameter</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>platform: vsphere: cluster:</td>
  <td>The vCenter cluster to install the OpenShift Container Platform cluster in.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: datacenter:</td>
  <td>Defines the data center where OpenShift Container Platform virtual machines (VMs) operate.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: defaultDatastore:</td>
  <td>The name of the default datastore to use for provisioning volumes.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: folder:</td>
  <td>Optional: The absolute path of an existing folder where the installation program creates the virtual machines. If you do not provide this value, the installation program creates a folder that is named with the infrastructure ID in the data center virtual machine folder.<br><br><strong>Value:</strong> String, for example, <code>/&lt;data_center_name&gt;/vm/&lt;folder_name&gt;/&lt;subfolder_name&gt;</code>.</td>
</tr>
<tr>
  <td>platform: vsphere: password:</td>
  <td>The password for the vCenter user name.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: resourcePool:</td>
  <td>Optional: The absolute path of an existing resource pool where the installation program creates the virtual machines. If you do not specify a value, the installation program installs the resources in the root of the cluster under <code>/&lt;data_center_name&gt;/host/&lt;cluster_name&gt;/Resources</code>.<br><br><strong>Value:</strong> String, for example, <code>/&lt;data_center_name&gt;/host/&lt;cluster_name&gt;/Resources/&lt;resource_pool_name&gt;/&lt;optional_nested_resource_pool_name&gt;</code>.</td>
</tr>
<tr>
  <td>platform: vsphere: username:</td>
  <td>The user name to use to connect to the vCenter instance with. This user must have at least the roles and privileges that are required for <a href="https://github.com/vmware-archive/vsphere-storage-for-kubernetes/blob/master/documentation/vcp-roles.md">static or dynamic persistent volume provisioning</a> in vSphere.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: vCenter:</td>
  <td>The fully-qualified hostname or IP address of a vCenter server.<br><br><strong>Value:</strong> String</td>
</tr>
</tbody>
</table>

**Additional resources**

- [BMC addressing](/docs/installing/installing_bare_metal/ipi/ipi-install-installation-workflow#bmc-addressing_ipi-install-installation-workflow)
- [Configuring regions and zones for a VMware vCenter](/docs/installing/installing_vsphere/ipi/installing-vsphere-installer-provisioned-customizations#configuring-vsphere-regions-zones_installing-vsphere-installer-provisioned-customizations)
- [Required vCenter account privileges](/docs/installing/installing_vsphere/ipi/ipi-vsphere-installation-reqs#installation-vsphere-installer-infra-requirements-account_ipi-vsphere-installation-reqs)

## Available Agent configuration parameters {#agent-configuration-parameters_installation-config-parameters-agent}

To customize your cluster installation, configuration parameters are available to use in the `agent-config.yaml` file.

The following tables specify the required and optional Agent configuration parameters that you can set as part of the Agent-based installation process.

:::note

These settings are used for installation only, and cannot be modified after installation.

:::

### Required configuration parameters {#agent-configuration-parameters-required_installation-config-parameters-agent}

Required Agent configuration parameters are described in the following table:

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
  <td>The API version for the <code>agent-config.yaml</code> content. The current version is <code>v1beta1</code>. The installation program might also support older API versions.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>metadata:</td>
  <td>Kubernetes resource <code>ObjectMeta</code>, from which only the <code>name</code> parameter is consumed.<br><br><strong>Value:</strong> Object</td>
</tr>
<tr>
  <td>metadata: name:</td>
  <td>The name of the cluster. DNS records for the cluster are all subdomains of <code>{.metadata.name}.{.baseDomain}</code>. The value entered in the <code>agent-config.yaml</code> file is ignored, and instead the value specified in the <code>install-config.yaml</code> file is used. When you do not provide <code>metadata.name</code> through either the <code>install-config.yaml</code> or <code>agent-config.yaml</code> files, for example when you use only ZTP manifests, the cluster name is set to <code>agent-cluster</code>.<br><br><strong>Value:</strong> String of lowercase letters and hyphens (<code>-</code>), such as <code>dev</code>.</td>
</tr>
</tbody>
</table>

### Optional configuration parameters {#agent-configuration-parameters-optional_installation-config-parameters-agent}

Optional Agent configuration parameters are described in the following table:

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
  <td>rendezvousIP:</td>
  <td>The IP address of the node that performs the bootstrapping process as well as running the <code>assisted-service</code> component. You must provide the rendezvous IP address when you do not specify at least one host's IP address in the <code>networkConfig</code> parameter. If this address is not provided, one IP address is selected from the provided hosts' <code>networkConfig</code>.<br><br><strong>Value:</strong> IPv4 or IPv6 address.</td>
</tr>
<tr>
  <td>bootArtifactsBaseURL:</td>
  <td>When you use the Agent-based Installer to generate a minimal ISO image, this parameter specifies a URL where the rootfs image file can be retrieved from during cluster installation. This parameter is optional for booting minimal ISO images in connected environments.<br><br>When you use the Agent-based Installer to generate an iPXE script, this parameter specifies the URL of the server to upload Preboot Execution Environment (PXE) assets to. For more information, see "Preparing PXE assets for OpenShift Container Platform".<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>additionalNTPSources:</td>
  <td>A list of Network Time Protocol (NTP) sources to be added to all cluster hosts, which are added to any NTP sources that are configured through other means.<br><br><strong>Value:</strong> List of hostnames or IP addresses.</td>
</tr>
<tr>
  <td>hosts:</td>
  <td>Host configuration. An optional list of hosts. The number of hosts defined must not exceed the total number of hosts defined in the <code>install-config.yaml</code> file, which is the sum of the values of the <code>compute.replicas</code> and <code>controlPlane.replicas</code> parameters.<br><br><strong>Value:</strong> An array of host configuration objects.</td>
</tr>
<tr>
  <td>hosts: hostname:</td>
  <td>Hostname. Overrides the hostname obtained from either the Dynamic Host Configuration Protocol (DHCP) or a reverse DNS lookup. Each host must have a unique hostname supplied by one of these methods, although configuring a hostname through this parameter is optional.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>hosts: interfaces:</td>
  <td>Provides a table of the name and MAC address mappings for the interfaces on the host. If a <code>NetworkConfig</code> section is provided in the <code>agent-config.yaml</code> file, this table must be included and the values must match the mappings provided in the <code>NetworkConfig</code> section.<br><br><strong>Value:</strong> An array of host configuration objects.</td>
</tr>
<tr>
  <td>hosts: interfaces: name:</td>
  <td>The name of an interface on the host.<div class="alert alert--secondary"><strong>Note</strong><p>This value does not need to match the device name.</p></div><br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>hosts: interfaces: macAddress:</td>
  <td>The MAC address of an interface on the host.<br><br><strong>Value:</strong> A MAC address such as the following example: <code>00-B0-D0-63-C2-26</code>.</td>
</tr>
<tr>
  <td>hosts: role:</td>
  <td>Defines whether the host is a <code>master</code> or <code>worker</code> node. If no role is defined in the <code>agent-config.yaml</code> file, roles will be assigned at random during cluster installation.<br><br><strong>Value:</strong> <code>master</code> or <code>worker</code>.</td>
</tr>
<tr>
  <td>hosts: rootDeviceHints:</td>
  <td>Enables provisioning of the Red&#160;Hat Enterprise Linux CoreOS (RHCOS) image to a particular device. The installation program examines the devices in the order it discovers them, and compares the discovered values with the hint values. It uses the first discovered device that matches the hint value. This is the device that the operating system is written on during installation.<br><br><strong>Value:</strong> A dictionary of key-value pairs. For more information, see "Root device hints" in the "Setting up the environment for an OpenShift installation" page.</td>
</tr>
<tr>
  <td>hosts: rootDeviceHints: deviceName:</td>
  <td>The name of the device the RHCOS image is provisioned to.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>hosts: networkConfig:</td>
  <td>The host network definition. The configuration must match the Host Network Management API defined in the "Declarative Network API (nmstate documentation)".<br><br><strong>Value:</strong> A dictionary of host network configuration objects.</td>
</tr>
<tr>
  <td>minimalISO:</td>
  <td>Defines whether the Agent-based Installer generates a full ISO or a minimal ISO image. When this parameter is set to <code>True</code>, the Agent-based Installer generates an ISO without a rootfs image file, and instead contains details about where to pull the rootfs file from.<br><br>When you generate a minimal ISO, if you do not specify a rootfs URL through the <code>bootArtifactsBaseURL</code> parameter, the Agent-based Installer embeds a default URL that is accessible in environments with an internet connection.<br><br>The default value is <code>False</code>.<br><br><strong>Value:</strong> Boolean.</td>
</tr>
</tbody>
</table>

**Additional resources**

- [Preparing PXE assets for OpenShift Container Platform](/docs/installing/installing_with_agent_based_installer/prepare-pxe-assets-agent#prepare-pxe-assets-agent)
- [Declarative Network API (nmstate documentation)](https://nmstate.io/)
- [Root device hints](/docs/installing/installing_bare_metal/ipi/ipi-install-installation-workflow#root-device-hints_ipi-install-installation-workflow)
