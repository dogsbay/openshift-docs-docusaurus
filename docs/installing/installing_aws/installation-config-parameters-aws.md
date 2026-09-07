---
title: Installation configuration parameters for AWS
sidebar_position: 7
---

# Installation configuration parameters for AWS {#installation-config-parameters-aws}

<a id="installation-config-parameters-aws"></a>

Before you deploy an OpenShift Container Platform cluster on Amazon Web Services (AWS), you create the `install-config.yaml` file and provide parameters to customize your cluster and the platform that hosts it. You can then modify the `install-config.yaml` file to customize your cluster further.

## Available installation configuration parameters for AWS {#installation-configuration-parameters_installation-config-parameters-aws}

To customize your cluster installation, you can use configuration parameters in the `install-config.yaml` file.

The following tables specify the required, optional, and AWS-specific installation configuration parameters that you can set as part of the installation process.

:::warning

After installation, you cannot change these parameters in the `install-config.yaml` file.

:::

### Required configuration parameters {#installation-configuration-parameters-required_installation-config-parameters-aws}

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
  <td>The name of the cluster. DNS records for the cluster are all subdomains of <code>{.metadata.name}.{.baseDomain}</code>.  <br><br>  <strong>Value:</strong> String of lowercase letters, hyphens (<code>-</code>), and periods (<code>.</code>), such as <code>dev</code>.   </td>
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

### Network configuration parameters {#installation-configuration-parameters-network_installation-config-parameters-aws}

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
  <td>Required if you use <code>networking.clusterNetwork</code>. An IP address block.<br><br>  An IPv4 network.  <br><br>  If you use the OVN-Kubernetes network plugin, you can specify IPv4 and IPv6 networks.<br><br><strong>Value:</strong> An IP address block in Classless Inter-Domain Routing (CIDR) notation. The prefix length for an IPv4 block is between <code>0</code> and <code>32</code>. The prefix length for an IPv6 block is between <code>0</code> and <code>128</code>. For example, <code>10.128.0.0/14</code> or <code>fd01::/48</code>. </td>
</tr>
<tr>
  <td>networking: clusterNetwork: hostPrefix:</td>
  <td>The subnet prefix length to assign to each individual node. For example, if <code>hostPrefix</code> is set to <code>23</code> then each node is assigned a <code>/23</code> subnet out of the given <code>cidr</code>. A <code>hostPrefix</code> value of <code>23</code> provides 510 (2^(32 - 23) - 2) pod IP addresses.<br><br><strong>Value:</strong> A subnet prefix.<br><br>  The default value is <code>23</code>.  <br><br>  For an IPv4 network the default value is <code>23</code>. For an IPv6 network <code>hostPrefix</code> must be set to <code>64</code>, which is the default value. </td>
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
  <td>Required if you use <code>networking.machineNetwork</code>. An IP address block. The default value is <code>10.0.0.0/16</code> for all platforms other than libvirt and IBM Power(R) Virtual Server. For libvirt, the default value is <code>192.168.126.0/24</code>. For IBM Power(R) Virtual Server, the default value is <code>192.168.0.0/24</code>.<br><br>  <br><br><strong>Value:</strong> An IP network block in CIDR notation.<br><br>  For example, <code>10.0.0.0/16</code>.    <br><br><div class="alert alert--secondary"><strong>Note</strong><p>Set the <code>networking.machineNetwork</code> to match the CIDR of the preferred NIC.<br><br>If you are installing a cluster on AWS with dual-stack networking, consider the following distinction:<br><br><ul><li>If the installation program creates the VPC, do not specify an IPv6 entry in <code>networking.machineNetwork</code>. The installation program will assign an IPv6 address to the VPC.</li><li>If you provide existing dual-stack subnets using the <code>platform.aws.vpc.subnets</code> parameter, you must specify IPv6 entries corresponding to either the VPC CIDR or the CIDR of the subnets.</li><li>In both cases, you must provide an IPv4 CIDR entry.</li></ul></p></div></td>
</tr>
<tr>
  <td>networking: ovnKubernetesConfig: ipv4: internalJoinSubnet:</td>
  <td>Configures the IPv4 join subnet that is used internally by <code>ovn-kubernetes</code>. This subnet must not overlap with any other subnet that OpenShift Container Platform is using, including the node network. The size of the subnet must be larger than the number of nodes. You cannot change the value after installation.<br><br><strong>Value:</strong> An IP network block in CIDR notation. The default value is <code>100.64.0.0/16</code>.</td>
</tr>
</tbody>
</table>

### Optional configuration parameters {#installation-configuration-parameters-optional_installation-config-parameters-aws}

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
  <td>Determines the instruction set architecture of the machines in the pool. Currently, clusters with varied architectures are not supported. All pools must specify the same architecture. Valid values are <code>amd64</code> and <code>arm64</code>.<br><br>  Not all installation options support the 64-bit ARM architecture. To verify if your installation option is supported on your platform, see <em>Supported installation methods for different platforms</em> in <em>Selecting a cluster installation method and preparing it for users</em>.  <br><br><strong>Value:</strong> String</td>
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
  <td>Determines the instruction set architecture of the machines in the pool. Currently, clusters with varied architectures are not supported. All pools must specify the same architecture. Valid values are <code>amd64</code> and <code>arm64</code>.<br><br>  Not all installation options support the 64-bit ARM architecture. To verify if your installation option is supported on your platform, see <em>Supported installation methods for different platforms</em> in <em>Selecting a cluster installation method and preparing it for users</em>.  <br><br><strong>Value:</strong> String</td>
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
  <td>platform: aws: lbType:</td>
  <td>Required to set the NLB load balancer type in AWS. Valid values are <code>Classic</code> or <code>NLB</code>. If no value is specified, the installation program defaults to <code>Classic</code>. The installation program sets the value provided here in the ingress cluster configuration object. If you do not specify a load balancer type for other Ingress Controllers, they use the type set in this parameter.<br><br>If you installed your cluster using the <code>DualStackIPv4Primary</code> or <code>DualStackIPv6Primary</code> values for the <code>platform.aws.ipFamily</code> parameter, any services that have IPv6 addresses must use the NLB load balancer type. The classic load balancer (CLB) does not support IPv6.<br><br><strong>Value:</strong> <code>Classic</code> or <code>NLB</code>. If you do not set the <code>platform.aws.ipFamily</code> parameter or set it to <code>IPv4</code>, the default value is <code>Classic</code>. If you set the <code>platform.aws.ipFamily</code> parameter to <code>DualStackIPv4Primary</code> or <code>DualStackIPv6Primary</code>, the default value is <code>NLB</code>.</td>
</tr>
<tr>
  <td>publish:</td>
  <td>How to publish or expose the user-facing endpoints of your cluster, such as the Kubernetes API, OpenShift routes.<br><br><strong>Value:</strong>  <code>Internal</code> or <code>External</code>. To deploy a private cluster that cannot be accessed from the internet, set the <code>publish</code> parameter to <code>Internal</code>. The default value is <code>External</code>.   </td>
</tr>
<tr>
  <td>sshKey:</td>
  <td>The SSH key to authenticate access to your cluster machines.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>For production OpenShift Container Platform clusters on which you want to perform installation debugging or disaster recovery, specify an SSH key that your <code>ssh-agent</code> process uses.</p></div><br><br><strong>Value:</strong> For example, <code>sshKey: ssh-ed25519 AAAA..</code>.</td>
</tr>
</tbody>
</table>

:::note

If your AWS account has service control policies (SCP) enabled, you must configure the `credentialsMode` parameter to `Mint`, `Passthrough`, or `Manual`.

:::

:::warning

Setting this parameter to `Manual` enables alternatives to storing administrator-level secrets in the `kube-system` project, which require additional configuration steps. For more information, see "Alternatives to storing administrator-level secrets in the kube-system project".

:::

### Optional AWS configuration parameters {#installation-configuration-parameters-optional-aws_installation-config-parameters-aws}

Optional AWS configuration parameters are described in the following table:

**Optional AWS parameters**

<table>
<thead>
<tr>
  <th>Parameter</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>compute: platform: aws: amiID:</td>
  <td>The AWS AMI used to boot compute machines for the cluster. This is required for regions that require a custom RHCOS AMI.<br><br><strong>Value:</strong> Any published or custom RHCOS AMI that belongs to the set AWS region. See _RHCOS AMIs for AWS infrastructure_ for available AMI IDs.</td>
</tr>
<tr>
  <td>compute: platform: aws: iamProfile:</td>
  <td>The name of the IAM instance profile that you use for the machine. If you want the installation program to create the IAM instance profile for you, do not use the <code>iamProfile</code> parameter. You can specify either the <code>iamProfile</code> or <code>iamRole</code> parameter, but you cannot specify both.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>compute: platform: aws: iamRole:</td>
  <td>The name of the IAM instance role that you use for the machine. When you specify an IAM role, the installation program creates an instance profile. If you want the installation program to create the IAM instance role for you, do not select the <code>iamRole</code> parameter. You can specify either the <code>iamRole</code> or <code>iamProfile</code> parameter, but you cannot specify both.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>compute: platform: aws: rootVolume: iops:</td>
  <td>The Input/Output Operations Per Second (IOPS) that is reserved for the root volume.<br><br><strong>Value:</strong> Integer, for example <code>4000</code>.</td>
</tr>
<tr>
  <td>compute: platform: aws: rootVolume: size:</td>
  <td>The size in GiB of the root volume.<br><br><strong>Value:</strong> Integer, for example <code>500</code>.</td>
</tr>
<tr>
  <td>compute: platform: aws: rootVolume: type:</td>
  <td>The type of the root volume.<br><br><strong>Value:</strong> Valid <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html">AWS EBS volume type</a>, such as <code>io1</code>.</td>
</tr>
<tr>
  <td>compute: platform: aws: rootVolume: throughput:</td>
  <td>The maximum throughput of the root volume. This throughput can be customized only for the gp3 volume type. The minimum value is 125 MiB/s and the maximum value is 2000 MiB/s.<br><br><strong>Value:</strong> Integer, for example <code>1000</code>.</td>
</tr>
<tr>
  <td>compute: platform: aws: rootVolume: kmsKeyARN:</td>
  <td>The Amazon Resource Name (key ARN) of a KMS key. This is required to encrypt operating system volumes of worker nodes with a specific KMS key.<br><br><strong>Value:</strong> Valid <a href="https://docs.aws.amazon.com/kms/latest/developerguide/find-cmk-id-arn.html">key ID or the key ARN</a>.</td>
</tr>
<tr>
  <td>compute: platform: aws: type:</td>
  <td>The EC2 instance type for the compute machines.<br><br><strong>Value:</strong> Valid AWS instance type, such as <code>m4.2xlarge</code>. See the "Tested instance types for AWS" table on the "Installing a cluster on AWS with customizations" page.</td>
</tr>
<tr>
  <td>compute: platform: aws: zones:</td>
  <td>The availability zones where the installation program creates machines for the compute machine pool. If you provide your own VPC, you must provide a subnet in that availability zone.<br><br><strong>Value:</strong> A list of valid AWS availability zones, such as <code>us-east-1c</code>, in a <a href="https://yaml.org/spec/1.2/spec.html#sequence//">YAML sequence</a>.</td>
</tr>
<tr>
  <td>compute: platform: aws: hostPlacement: affinity:</td>
  <td>Specifies the affinity setting for placing compute machines on AWS Dedicated Hosts. When set to <code>DedicatedHost</code>, machines are pinned to the specific Dedicated Hosts listed in the <code>dedicatedHost</code> field. If a machine is stopped and restarted, the machine returns to the same physical host. When set to <code>AnyAvailable</code>, machines are not pinned to specific Dedicated Hosts. If a machine is stopped and restarted, AWS can place the machine on any available Dedicated Host that matches the instance type and availability zone.<br><br><div class="alert alert--warning"><strong>Important</strong><p>AWS Dedicated Host support is a Technology Preview feature only. Technology Preview features are not supported with Red&#160;Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.<br><br>For more information about the support scope of Red Hat Technology Preview features, see <a href="https://access.redhat.com/support/offerings/techpreview/">Technology Preview Features Support Scope</a>.</p></div><br><br><strong>Value:</strong> <code>DedicatedHost</code> or <code>AnyAvailable</code>.</td>
</tr>
<tr>
  <td>compute: platform: aws: hostPlacement: dedicatedHost:</td>
  <td>A list of AWS Dedicated Host entries for compute machines. Required when <code>hostPlacement.affinity</code> is set to <code>DedicatedHost</code>. Must be omitted when <code>hostPlacement.affinity</code> is set to <code>AnyAvailable</code>.<br><br><div class="alert alert--warning"><strong>Important</strong><p>AWS Dedicated Host support is a Technology Preview feature only. Technology Preview features are not supported with Red&#160;Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.<br><br>For more information about the support scope of Red Hat Technology Preview features, see <a href="https://access.redhat.com/support/offerings/techpreview/">Technology Preview Features Support Scope</a>.</p></div><br><br><strong>Value:</strong> A list of objects.</td>
</tr>
<tr>
  <td>compute: platform: aws: hostPlacement: dedicatedHost: - id:</td>
  <td>The ID of the AWS Dedicated Host. The value must start with <code>h-</code> followed by 17 lowercase hexadecimal characters.<br><br><div class="alert alert--warning"><strong>Important</strong><p>AWS Dedicated Host support is a Technology Preview feature only. Technology Preview features are not supported with Red&#160;Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.<br><br>For more information about the support scope of Red Hat Technology Preview features, see <a href="https://access.redhat.com/support/offerings/techpreview/">Technology Preview Features Support Scope</a>.</p></div><br><br><strong>Value:</strong> String, for example <code>h-015c6d3ffa1d43d38</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: aws: amiID:</td>
  <td>The AWS AMI used to boot control plane machines for the cluster. This is required for regions that require a custom RHCOS AMI.<br><br><strong>Value:</strong> Any published or custom RHCOS AMI that belongs to the set AWS region. See _RHCOS AMIs for AWS infrastructure_ for available AMI IDs.</td>
</tr>
<tr>
  <td>controlPlane: platform: aws: iamProfile:</td>
  <td>The name of the IAM instance profile that you use for the machine. If you want the installation program to create the IAM instance profile for you, do not use the <code>iamProfile</code> parameter. You can specify either the <code>iamProfile</code> or <code>iamRole</code> parameter, but you cannot specify both.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>controlPlane: platform: aws: iamRole:</td>
  <td>The name of the IAM instance role that you use for the machine. When you specify an IAM role, the installation program creates an instance profile. If you want the installation program to create the IAM instance role for you, do not use the <code>iamRole</code> parameter. You can specify either the <code>iamRole</code> or <code>iamProfile</code> parameter, but you cannot specify both.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>controlPlane: platform: aws: rootVolume: iops:</td>
  <td>The Input/Output Operations Per Second (IOPS) that is reserved for the root volume on control plane machines.<br><br><strong>Value:</strong> Integer, for example <code>4000</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: aws: rootVolume: size:</td>
  <td>The size in GiB of the root volume for control plane machines.<br><br><strong>Value:</strong> Integer, for example <code>500</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: aws: rootVolume: type:</td>
  <td>The type of the root volume for control plane machines.<br><br><strong>Value:</strong> Valid <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html">AWS EBS volume type</a>, such as <code>io1</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: aws: rootVolume: throughput:</td>
  <td>The maximum throughput of the root volume. This throughput can be customized only for the gp3 volume type. The minimum value is 125 MiB/s and the maximum value is 2000 MiB/s.<br><br><strong>Value:</strong> Integer, for example <code>1000</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: aws: rootVolume: kmsKeyARN:</td>
  <td>The Amazon Resource Name (key ARN) of a KMS key. This is required to encrypt operating system volumes of control plane nodes with a specific KMS key.<br><br><strong>Value:</strong> Valid <a href="https://docs.aws.amazon.com/kms/latest/developerguide/find-cmk-id-arn.html">key ID and the key ARN</a>.</td>
</tr>
<tr>
  <td>controlPlane: platform: aws: type:</td>
  <td>The EC2 instance type for the control plane machines.<br><br><strong>Value:</strong> Valid AWS instance type, such as <code>m6i.xlarge</code>. See the "Tested instance types for AWS" table on the "Installing a cluster on AWS with customizations" page.</td>
</tr>
<tr>
  <td>controlPlane: platform: aws: zones:</td>
  <td>The availability zones where the installation program creates machines for the control plane machine pool.<br><br><strong>Value:</strong> A list of valid AWS availability zones, such as <code>us-east-1c</code>, in a <a href="https://yaml.org/spec/1.2/spec.html#sequence//">YAML sequence</a>.</td>
</tr>
<tr>
  <td>platform: aws: amiID:</td>
  <td>The AWS AMI used to boot all machines for the cluster. If set, the AMI must belong to the same region as the cluster. This is required for regions that require a custom RHCOS AMI.<br><br><strong>Value:</strong> Any published or custom RHCOS AMI that belongs to the set AWS region. See _RHCOS AMIs for AWS infrastructure_ for available AMI IDs.</td>
</tr>
<tr>
  <td>platform: aws: hostedZone:</td>
  <td>An existing Route 53 private hosted zone for the cluster. You can only use a pre-existing hosted zone when also supplying your own VPC. The hosted zone must already be associated with the user-provided VPC before installation. Also, the domain of the hosted zone must be the cluster domain or a parent of the cluster domain. If undefined, the installation program creates a new hosted zone.<br><br><strong>Value:</strong> String, for example <code>Z3URY6TWQ91KVV</code>.</td>
</tr>
<tr>
  <td>platform: aws: hostedZoneRole:</td>
  <td>An Amazon Resource Name (ARN) for an existing IAM role in the account containing the specified hosted zone. The installation program and cluster operators assume this role when performing operations on the hosted zone. Use this parameter only when you are installing a cluster into a shared VPC.<br><br><strong>Value:</strong> String, for example <code>arn:aws:iam::1234567890:role/shared-vpc-role</code>.</td>
</tr>
<tr>
  <td>platform: aws: userProvisionedDNS:</td>
  <td>Enables user-provisioned DNS instead of the default cluster-provisioned DNS solution. If you use this feature, you must provide your own DNS solution that includes records for <code>api.&lt;cluster_name&gt;.&lt;base_domain&gt;.</code> and <code>*.apps.&lt;cluster_name&gt;.&lt;base_domain&gt;.</code>. <code>userProvisionedDNS</code> is a Technology Preview feature.<br><br><strong>Value:</strong> <code>Enabled</code> or <code>Disabled</code>. The default value is <code>Disabled</code>.</td>
</tr>
<tr>
  <td>platform: aws: region:</td>
  <td>The AWS region that the installation program creates all cluster resources in.<br><br><strong>Value:</strong> Any valid <a href="https://docs.aws.amazon.com/general/latest/gr/rande.html">AWS region</a>, such as <code>us-east-1</code>. You can use the AWS CLI to access the regions available based on your selected instance type by running the following command:<br><br><pre><code>$ aws ec2 describe-instance-type-offerings --filters Name=instance-type,Values=c7g.xlarge</code></pre><br><br>  <div class="alert alert--warning"><strong>Important</strong><p>When running on ARM based AWS instances, ensure that you enter a region where AWS Graviton processors are available. See <a href="https://aws.amazon.com/ec2/graviton/#Global_availability">Global availability</a> map in the AWS documentation. Currently, AWS Graviton3 processors are only available in some regions.</p></div> </td>
</tr>
<tr>
  <td>platform: aws: serviceEndpoints: - name: url:</td>
  <td>The AWS service endpoint name and URL. Custom endpoints are only required for cases where alternative AWS endpoints, such as FIPS, must be used. Custom API endpoints can be specified for EC2, S3, IAM, Elastic Load Balancing, Tagging, Route 53, and STS AWS services.<br><br><strong>Value:</strong> Valid <a href="https://docs.aws.amazon.com/general/latest/gr/rande.html">AWS service endpoint</a> name and valid <a href="https://docs.aws.amazon.com/general/latest/gr/rande.html">AWS service endpoint</a> URL.</td>
</tr>
<tr>
  <td>platform: aws: userTags:</td>
  <td>A map of keys and values that the installation program adds as tags to all resources that it creates.<br><br><strong>Value:</strong> Any valid YAML map, such as key value pairs in the <code>&lt;key&gt;: &lt;value&gt;</code> format. For more information about AWS tags, see <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Tags.html">Tagging Your Amazon EC2 Resources</a> in the AWS documentation.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>You can add up to 25 user-defined tags during installation. The remaining 25 tags are reserved for OpenShift Container Platform.</p></div></td>
</tr>
<tr>
  <td>platform: aws: propagateUserTags:</td>
  <td>A flag that directs in-cluster Operators to include the specified user tags in the tags of the AWS resources that the Operators create.<br><br><strong>Value:</strong> Boolean values, for example <code>true</code> or <code>false</code>.</td>
</tr>
<tr>
  <td>platform: aws: publicIpv4Pool:</td>
  <td>The public IPv4 pool ID that is used to allocate Elastic IPs (EIPs) when <code>publish</code> is set to <code>External</code>. You must provision and advertise the pool in the same AWS account and region of the cluster. You must ensure that you have 2n + 1 IPv4 addresses available in the pool where <em>n</em> is the total number of AWS zones used to deploy the Network Load Balancer (NLB) for API, NAT gateways, and bootstrap node. For more information about bring your own IP addresses (BYOIP) in AWS, see <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-byoip.html#byoip-onboard">Onboard your BYOIP</a>.<br><br><strong>Value:</strong> A valid <a href="https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-public-ipv4-pools.html">public IPv4 pool id</a><br><br><div class="alert alert--secondary"><strong>Note</strong><p>You can enable BYOIP only for customized installations that do not have any network restrictions.</p></div></td>
</tr>
<tr>
  <td>platform: aws: bestEffortDeleteIgnition:</td>
  <td>An optional flag that determines whether to ignore errors when deleting Ignition objects from the S3 bucket. By default, the installation program fails if it cannot delete the Ignition objects.<br><br><strong>Value:</strong> <code>true</code> or <code>false</code>. The default value is <code>false</code>, which causes the installation program to fail on S3 Ignition deletion errors.</td>
</tr>
<tr>
  <td>platform: aws: ipFamily:</td>
  <td>The IP address family for networks used by the cluster. Specify <code>IPv4</code> for IPv4-only networking, <code>DualStackIPv4Primary</code> for dual-stack networking with IPv4 as the primary address family, or <code>DualStackIPv6Primary</code> for dual-stack networking with IPv6 as the primary address family. When using dual-stack, the VPC and subnets must be configured with both IPv4 and IPv6 CIDR blocks.<br><br>Consider the following requirements if you use dual-stack networking:<br><br><ul><li>All API and Ingress load balancers must be Network Load Balancers (NLB). Classic Load Balancers (CLB) do not support IPv6 addressing.</li><li>All machines in a dual-stack cluster must be Nitro-based and support IPv6 addressing.</li><li>If you are installing a cluster using existing subnets, all provided subnets must be configured with dual-stack address pools.</li><li>If you are installing a cluster using Local Zones, you must provide dual-stack subnets. The installation program cannot automatically provision dual-stack subnets in Local Zones.</li><li>Installing a cluster using dual-stack networking is not supported in Wavelength Zones.</li></ul><div class="alert alert--warning"><strong>Important</strong><p>Dual-stack networking on AWS is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process. For more information about the support scope of Red Hat Technology Preview features, see <a href="https://access.redhat.com/support/offerings/techpreview/">Technology Preview Features Support Scope</a>.</p></div><br><br><strong>Value:</strong> "IPv4", "DualStackIPv4Primary", or "DualStackIPv6Primary". The default value is "IPv4".</td>
</tr>
<tr>
  <td>platform: aws: vpc: subnets:</td>
  <td>A list of subnets in an existing VPC to be used in place of automatically created subnets. You specify a subnet by providing the subnet ID and an optional list of roles that apply to that subnet. If you specify subnet IDs but do not specify roles for any subnet, the subnets' roles are decided automatically. If you do not specify any roles, you must ensure that any other subnets in your VPC have the <code>kubernetes.io/cluster/.<strong>: .</strong></code> or <code>kubernetes.io/cluster/unmanaged: true</code> tags.<br><br>The subnets must be part of the same <code>networking.machineNetwork[].cidr</code> ranges that you specify. If you provide dual-stack subnets using this parameter, you must specify IPv6 entries in the <code>networking.machineNetwork[].cidr</code> parameter.<br><br>For a public cluster, specify a public and a private subnet for each availability zone.<br><br>For a private cluster, specify a private subnet for each availability zone.<br><br>For clusters that use AWS Local Zones, you must add AWS Local Zone subnets to this list to ensure edge machine pool creation.<br><br><strong>Value:</strong> List of pairs of <code>id</code> and <code>roles</code> parameters.</td>
</tr>
<tr>
  <td>platform: aws: vpc: subnets: - id:</td>
  <td>The ID of an existing subnet to be used in place of a subnet created by the installation program.<br><br><strong>Value:</strong> String. The subnet ID must be a unique ID containing only alphanumeric characters, beginning with "subnet-". The ID must be exactly 24 characters long.</td>
</tr>
<tr>
  <td>platform: aws: vpc: subnets: - id: roles: - type:</td>
  <td>One or more roles that apply to the subnet specified by <code>platform.aws.vpc.subnets.id</code>. If you specify a role for any subnet, each subnet must have at least one assigned role, and the <code>ClusterNode</code>, <code>IngressControllerLB</code>, <code>ControlPlaneExternalLB</code>, <code>BootstrapNode</code>, and <code>ControlPlaneInternalLB</code> roles must be assigned to at least one subnet. However, if the cluster scope is internal, then the <code>ControlPlaneExternalLB</code> role is not required.<br><br>You can only assign the <code>EdgeNode</code> role to subnets in AWS Local Zones.<br><br><strong>Value:</strong> List of one or more role types. Valid values include <code>ClusterNode</code>, <code>EdgeNode</code>, <code>BootstrapNode</code>, <code>IngressControllerLB</code>, <code>ControlPlaneExternalLB</code>, and <code>ControlPlaneInternalLB</code>.</td>
</tr>
</tbody>
</table>
