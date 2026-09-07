---
title: Installation configuration parameters for Azure
sidebar_position: 7
---

# Installation configuration parameters for Azure {#installation-config-parameters-azure}

<a id="installation-config-parameters-azure"></a>

Before you deploy an OpenShift Container Platform cluster on Microsoft Azure, you create the `install-config.yaml` file and provide parameters to customize your cluster and the platform that hosts it. You can then modify the `install-config.yaml` file to customize your cluster further.

## Available installation configuration parameters for Azure {#installation-configuration-parameters_installation-config-parameters-azure}

To customize your cluster installation, you can use configuration parameters in the `install-config.yaml` file.

The following tables specify the required, optional, and Azure-specific installation configuration parameters that you can set as part of the installation process.

:::warning

After installation, you cannot change these parameters in the `install-config.yaml` file.

:::

### Required configuration parameters {#installation-configuration-parameters-required_installation-config-parameters-azure}

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

### Network configuration parameters {#installation-configuration-parameters-network_installation-config-parameters-azure}

You can customize your installation configuration based on the requirements of your existing network infrastructure. For example, you can expand the IP address block for the cluster network or configure different IP address blocks than the defaults.

Only IPv4 addresses are supported.

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
  <td>Required if you use <code>networking.machineNetwork</code>. An IP address block. The default value is <code>10.0.0.0/16</code> for all platforms other than libvirt and IBM Power(R) Virtual Server. For libvirt, the default value is <code>192.168.126.0/24</code>. For IBM Power(R) Virtual Server, the default value is <code>192.168.0.0/24</code>.<br><br>  <br><br><strong>Value:</strong> An IP network block in CIDR notation.<br><br>  For example, <code>10.0.0.0/16</code>.    <br><br><div class="alert alert--secondary"><strong>Note</strong><p>Set the <code>networking.machineNetwork</code> to match the CIDR of the preferred NIC.<br><br>If you are installing a cluster on AWS with dual-stack networking, consider the following distinction:<br><br><ul><li>If the installation program creates the VPC, do not specify an IPv6 entry in <code>networking.machineNetwork</code>. The installation program will assign an IPv6 address to the VPC.</li><li>If you provide existing dual-stack subnets using the <code>platform.aws.vpc.subnets</code> parameter, you must specify IPv6 entries corresponding to either the VPC CIDR or the CIDR of the subnets.</li><li>In both cases, you must provide an IPv4 CIDR entry.</li></ul></p></div></td>
</tr>
<tr>
  <td>networking: ovnKubernetesConfig: ipv4: internalJoinSubnet:</td>
  <td>Configures the IPv4 join subnet that is used internally by <code>ovn-kubernetes</code>. This subnet must not overlap with any other subnet that OpenShift Container Platform is using, including the node network. The size of the subnet must be larger than the number of nodes. You cannot change the value after installation.<br><br><strong>Value:</strong> An IP network block in CIDR notation. The default value is <code>100.64.0.0/16</code>.</td>
</tr>
</tbody>
</table>

### Optional configuration parameters {#installation-configuration-parameters-optional_installation-config-parameters-azure}

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
  <td>publish:</td>
  <td>How to publish or expose the user-facing endpoints of your cluster, such as the Kubernetes API, OpenShift routes.<br><br><strong>Value:</strong>   <code>Internal</code>, <code>External</code>, or <code>Mixed</code>. To deploy a private cluster that cannot be accessed from the internet, set the <code>publish</code> parameter to <code>Internal</code>. The default value is <code>External</code>. To deploy a cluster where the API and the ingress server have different publishing strategies, set <code>publish</code> to <code>Mixed</code> and use the <code>operatorPublishingStrategy</code> parameter.  </td>
</tr>
<tr>
  <td>sshKey:</td>
  <td>The SSH key to authenticate access to your cluster machines.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>For production OpenShift Container Platform clusters on which you want to perform installation debugging or disaster recovery, specify an SSH key that your <code>ssh-agent</code> process uses.</p></div><br><br><strong>Value:</strong> For example, <code>sshKey: ssh-ed25519 AAAA..</code>.</td>
</tr>
</tbody>
</table>

:::warning

Setting this parameter to `Manual` enables alternatives to storing administrator-level secrets in the `kube-system` project, which require additional configuration steps. For more information, see "Alternatives to storing administrator-level secrets in the kube-system project".

:::

### Additional Azure configuration parameters {#installation-configuration-parameters-additional-azure_installation-config-parameters-azure}

Additional Azure configuration parameters are described in the following table.

:::note

By default, if you specify availability zones in the `install-config.yaml` file, the installation program distributes the control plane machines and the compute machines across [these availability zones](https://azure.microsoft.com/en-us/global-infrastructure/availability-zones/) within [a region](https://azure.microsoft.com/en-us/global-infrastructure/regions). To ensure high availability for your cluster, select a region with at least three availability zones. If your region contains fewer than three availability zones, the installation program places more than one control plane machine in the available zones.

:::

**Additional Azure parameters**

<table>
<thead>
<tr>
  <th>Parameter</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>compute: platform: azure: bootDiagnostics: type:</td>
  <td>Enables boot diagnostics collection for compute machines. The <code>type</code> field specifies the Azure boot diagnostics type for the created compute machines.<br><br>The following values are associated with the boot diagnostics type:<br><br><dl><dt><code>UserManaged</code></dt><dd>When you set <code>type</code> to <code>UserManaged</code>, you must provide values for <code>resourceGroup</code> and <code>storageAccountName</code>. For <code>storageAccountName</code> and  OpenShift Container Platform cluster nodes, use the same region and subscription.</dd><dt><code>Managed</code></dt><dd>When you set <code>type</code> to <code>Managed</code>, Azure stores the boot diagnostics data blobs  in a managed storage account.</dd><dt><code>Disabled</code></dt><dd>When you set <code>type</code> to <code>Disabled</code>, you turn off the parameter.</dd></dl><strong>Value:</strong> String, for example <code>Enabled</code>.</td>
</tr>
<tr>
  <td>compute: platform: azure: bootDiagnostics: resourceGroup:</td>
  <td>Specifies the name of the Azure resource group that contains the diagnostic storage account for compute machines. Use <code>resourceGroup</code> only when you set <code>type</code> to <code>UserManaged</code>.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>compute: platform: azure: bootDiagnostics: storageAccountName:</td>
  <td>Specifies the Azure storage account to store the diagnostic logs for compute machines. Use <code>storageAccountName</code> only when you set<code>type</code> to <code>UserManaged</code>.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>compute: platform: azure: encryptionAtHost:</td>
  <td>Enables host-level encryption for compute machines. You can enable this encryption alongside user-managed server-side encryption. This feature encrypts temporary, ephemeral, cached, and un-managed disks on the VM host. This is not a prerequisite for user-managed server-side encryption.<br><br><strong>Value:</strong> <code>true</code> or <code>false</code>. The default is <code>false</code>.</td>
</tr>
<tr>
  <td>compute: platform: azure: osDisk: diskSizeGB:</td>
  <td>The Azure disk size for the VM.<br><br><strong>Value:</strong> Integer that represents the size of the disk in GB. The default is <code>128</code>.</td>
</tr>
<tr>
  <td>compute: platform: azure: osDisk: diskType:</td>
  <td>Defines the type of disk.<br><br><strong>Value:</strong> <code>standard_LRS</code>, <code>premium_LRS</code>, or <code>standardSSD_LRS</code>. The default is <code>premium_LRS</code>.</td>
</tr>
<tr>
  <td>compute: platform: azure: ultraSSDCapability:</td>
  <td>Enables the use of Azure ultra disks for persistent storage on compute nodes. This requires that your Azure region and zone have ultra disks available.<br><br><strong>Value:</strong> <code>Enabled</code>, <code>Disabled</code>. The default is <code>Disabled</code>.</td>
</tr>
<tr>
  <td>compute: platform: azure: osDisk: diskEncryptionSet: resourceGroup:</td>
  <td>The name of the Azure resource group that contains the disk encryption set from the installation prerequisites. This resource group should be different from the resource group where you install the cluster to avoid deleting your Azure encryption key when the cluster is destroyed. This value is only necessary if you intend to install the cluster with user-managed disk encryption.<br><br><strong>Value:</strong> String, for example <code>production_encryption_resource_group</code>.</td>
</tr>
<tr>
  <td>compute: platform: azure: osDisk: diskEncryptionSet: name:</td>
  <td>The name of the disk encryption set that contains the encryption key from the installation prerequisites.<br><br><strong>Value:</strong> String, for example <code>production_disk_encryption_set</code>.</td>
</tr>
<tr>
  <td>compute: platform: azure: osDisk: diskEncryptionSet: subscriptionId:</td>
  <td>Defines the Azure subscription of the disk encryption set where the disk encryption set resides. This secondary disk encryption set is used to encrypt compute machines.<br><br><strong>Value:</strong> String, in the format <code>00000000-0000-0000-0000-000000000000</code>.</td>
</tr>
<tr>
  <td>compute: platform: azure: osImage: publisher:</td>
  <td>Optional. By default, the installation program downloads and installs the Red&#160;Hat Enterprise Linux CoreOS (RHCOS) image that is used to boot compute machines. You can override the default behavior by using a custom RHCOS image that is available from the Azure Marketplace. The installation program uses this image for compute machines only.<br><br><strong>Value:</strong> String. The name of the image publisher.</td>
</tr>
<tr>
  <td>compute: platform: azure: osImage: offer:</td>
  <td>The name of Azure Marketplace offer that is associated with the custom RHCOS image. If you use <code>compute.platform.azure.osImage.publisher</code>, this field is required.<br><br><strong>Value:</strong> String. The name of the image offer.</td>
</tr>
<tr>
  <td>compute: platform: azure: osImage: sku:</td>
  <td>An instance of the Azure Marketplace offer. If you use <code>compute.platform.azure.osImage.publisher</code>, this field is required.<br><br><strong>Value:</strong> String. The SKU of the image offer.</td>
</tr>
<tr>
  <td>compute: platform: azure: osImage: version:</td>
  <td>The version number of the image SKU. If you use <code>compute.platform.azure.osImage.publisher</code>, this field is required.<br><br><strong>Value:</strong> String. The version of the image to use.</td>
</tr>
<tr>
  <td>compute: platform: azure: identity: type:</td>
  <td>The type of identity used for compute virtual machines. The <code>UserAssigned</code> identity is a standalone Azure resource provided by the user and assigned to compute virtual machines. If you specify <code>identity.type</code> as <code>UserAssigned</code>, but do not provide a user-assigned identity, the installation program creates the identity. If you provide a user-assigned identity, the Azure account that you use to create the identity must have either the "User Access Administrator" or "RBAC Access Admin" roles.<br><br><strong>Value:</strong> <code>UserAssigned</code> or <code>None</code>. If you do not specify a value, the installation program generates a user-assigned identity.</td>
</tr>
<tr>
  <td>compute: platform: azure: identity: userAssignedIdentities: - name: resourceGroup: subscription:</td>
  <td>A group of parameters that specify the name of the user-assigned identity, and the resource group and subscription that contain the identity. All three values must be provided to specify a user-assigned identity. Only one user-assigned identity can be supplied. Supplying more than one user-assigned identity is an experimental feature, which may be enabled with the <code>MachineAPIMigration</code> feature gate.<br><br><strong>Value:</strong> Array of strings.</td>
</tr>
<tr>
  <td>compute: platform: azure: vmNetworkingType:</td>
  <td>Enables accelerated networking. Accelerated networking enables single root I/O virtualization (SR-IOV) to a VM, improving its networking performance. If instance type of compute machines support <code>Accelerated</code> networking, by default, the installation program enables <code>Accelerated</code> networking, otherwise the default networking type is <code>Basic</code>.<br><br><strong>Value:</strong> <code>Accelerated</code> or <code>Basic</code>.</td>
</tr>
<tr>
  <td>compute: platform: azure: type:</td>
  <td>Defines the Azure instance type for compute machines.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>compute: platform: azure: zones:</td>
  <td>The availability zones where the installation program creates compute machines.<br><br><strong>Value:</strong> String list</td>
</tr>
<tr>
  <td>compute: platform: azure: settings: securityType:</td>
  <td>Enables confidential VMs or trusted launch for compute nodes. This option is not enabled by default.<br><br><strong>Value:</strong> <code>ConfidentialVM</code> or <code>TrustedLaunch</code>.</td>
</tr>
<tr>
  <td>compute: platform: azure: settings: confidentialVM: uefiSettings: secureBoot:</td>
  <td>Enables secure boot on compute nodes if you are using confidential VMs.<br><br><strong>Value:</strong> <code>Enabled</code> or <code>Disabled</code>. The default is <code>Disabled</code>.</td>
</tr>
<tr>
  <td>compute: platform: azure: settings: confidentialVM: uefiSettings: virtualizedTrustedPlatformModule:</td>
  <td>Enables the virtualized Trusted Platform Module (vTPM) feature on compute nodes if you are using confidential VMs.<br><br><strong>Value:</strong> <code>Enabled</code> or <code>Disabled</code>. The default is <code>Disabled</code>.</td>
</tr>
<tr>
  <td>compute: platform: azure: settings: trustedLaunch: uefiSettings: secureBoot:</td>
  <td>Enables secure boot on compute nodes if you are using trusted launch.<br><br><strong>Value:</strong> <code>Enabled</code> or <code>Disabled</code>. The default is <code>Disabled</code>.</td>
</tr>
<tr>
  <td>compute: platform: azure: settings: trustedLaunch: uefiSettings: virtualizedTrustedPlatformModule:</td>
  <td>Enables the vTPM feature on compute nodes if you are using trusted launch.<br><br><strong>Value:</strong> <code>Enabled</code> or <code>Disabled</code>. The default is <code>Disabled</code>.</td>
</tr>
<tr>
  <td>compute: platform: azure: osDisk: securityProfile: securityEncryptionType:</td>
  <td>Enables the encryption of the virtual machine guest state for compute nodes. This parameter can only be used if you use Confidential VMs.<br><br><strong>Value:</strong> <code>VMGuestStateOnly</code> is the only supported value.</td>
</tr>
<tr>
  <td>controlPlane: diskSetup:</td>
  <td>Specifies node component information for dedicated disk configuration.<br><br><strong>Value:</strong> Array of objects. Each object includes the <code>type</code> and <code>etcd</code> parameters as described in the following rows of the table.<br><br><div class="alert alert--warning"><strong>Important</strong><p>Dedicated disk for <code>etcd</code> on Microsoft Azure is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process. For more information about the support scope of Red Hat Technology Preview features, see <a href="https://access.redhat.com/support/offerings/techpreview/">Technology Preview Features Support Scope</a>.</p></div></td>
</tr>
<tr>
  <td>controlPlane: diskSetup: - type:</td>
  <td>Specifies which node component type to assign a dedicated disk.<br><br><strong>Value:</strong> <code>etcd</code> is the only supported value.<br><br><div class="alert alert--warning"><strong>Important</strong><p>Dedicated disk for <code>etcd</code> on Microsoft Azure is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process. For more information about the support scope of Red Hat Technology Preview features, see <a href="https://access.redhat.com/support/offerings/techpreview/">Technology Preview Features Support Scope</a>.</p></div></td>
</tr>
<tr>
  <td>controlPlane: diskSetup: - etcd:</td>
  <td>Specifies parameters for an <code>etcd</code> dedicated disk.<br><br><strong>Value</strong>: The <code>platformDiskID</code> object is the only supported value.<br><br><div class="alert alert--warning"><strong>Important</strong><p>Dedicated disk for <code>etcd</code> on Microsoft Azure is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process. For more information about the support scope of Red Hat Technology Preview features, see <a href="https://access.redhat.com/support/offerings/techpreview/">Technology Preview Features Support Scope</a>.</p></div></td>
</tr>
<tr>
  <td>controlPlane: diskSetup: - etcd: platformDiskID:</td>
  <td>Specifies a name to identify the dedicated disk.<br><br><strong>Value:</strong> String. Must not exceed 12 characters.<br><br><div class="alert alert--warning"><strong>Important</strong><p>Dedicated disk for <code>etcd</code> on Microsoft Azure is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process. For more information about the support scope of Red Hat Technology Preview features, see <a href="https://access.redhat.com/support/offerings/techpreview/">Technology Preview Features Support Scope</a>.</p></div></td>
</tr>
<tr>
  <td>controlPlane: platform: azure: identity: type:</td>
  <td>The type of identity used for control plane virtual machines. The <code>UserAssigned</code> identity is a standalone Azure resource provided by the user and assigned to control plane virtual machines. If you specify <code>identity.type</code> as <code>UserAssigned</code>, but do not provide a user-assigned identity, the installation program creates the identity. If you provide a user-assigned identity, the Azure account that you use to create the identity must have either the "User Access Administrator" or "RBAC Access Admin" roles.<br><br><strong>Value:</strong> <code>UserAssigned</code> or <code>None</code>. If you do not specify a value, the installation program generates a user-assigned identity.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: bootDiagnostics: type:</td>
  <td>Enables boot diagnostics collection for control plane machines. The <code>type</code> field specifies the Azure boot diagnostics type for the created control plane machines.<br><br>The following values are associated with the boot diagnostics type:<br><br><dl><dt><code>UserManaged</code></dt><dd>When you set <code>type</code> to <code>UserManaged</code>, you must provide the values for <code>resourceGroup</code> and <code>storageAccountName</code>. For <code>storageAccountName</code> and OpenShift Container Platform cluster nodes, ensure that you use the same region and subscription.</dd><dt><code>Managed</code></dt><dd>When you set <code>type</code> to <code>Managed</code>, Azure stores the boot diagnostics data blobs in a managed storage account.</dd><dt><code>Disabled</code></dt><dd>When you set <code>type</code> to <code>Disabled</code>, you turn off the parameter.</dd></dl><strong>Value:</strong> String. For control plane machines, the default value is <code>Managed</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: identity: userAssignedIdentities: - name: resourceGroup: subscription:</td>
  <td>A group of parameters that specify the name of the user-assigned identity, and the resource group and subscription that contain the identity. All three values must be provided to specify a user-assigned identity. Only one user-assigned identity can be supplied. Supplying more than one user-assigned identity is an experimental feature, which may be enabled with the <code>MachineAPIMigration</code> feature gate.<br><br><strong>Value:</strong> Array of strings.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: bootDiagnostics: resourceGroup:</td>
  <td>Specifies the name of the Azure resource group that contains the diagnostic storage account for control plane machines. Use <code>resourceGroup</code> only when you set <code>type</code> to <code>UserManaged</code>.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: bootDiagnostics: storageAccountName:</td>
  <td>Specifies the Azure storage account to store the diagnostic logs for control plane machines. Use <code>storageAccountName</code> only when you set <code>type</code> to <code>UserManaged</code>.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: dataDisks:</td>
  <td>Specifies dedicated disk parameters.<br><br><strong>Value:</strong> Array of objects. Each object includes <code>nameSuffix</code>, <code>cachingType</code>, <code>diskSizeGB</code>, and <code>lun</code> as described in the following rows of the table.<br><br><div class="alert alert--warning"><strong>Important</strong><p>Dedicated disk for <code>etcd</code> on Microsoft Azure is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process. For more information about the support scope of Red Hat Technology Preview features, see <a href="https://access.redhat.com/support/offerings/techpreview/">Technology Preview Features Support Scope</a>.</p></div></td>
</tr>
<tr>
  <td>controlPlane: platform: azure: dataDisks: - nameSuffix:</td>
  <td>Specifies the same value you defined for <code>platformDiskID</code>.<br><br><strong>Value:</strong> String.<br><br><div class="alert alert--warning"><strong>Important</strong><p>Dedicated disk for <code>etcd</code> on Microsoft Azure is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process. For more information about the support scope of Red Hat Technology Preview features, see <a href="https://access.redhat.com/support/offerings/techpreview/">Technology Preview Features Support Scope</a>.</p></div></td>
</tr>
<tr>
  <td>controlPlane: platform: azure: dataDisks: - cachingType:</td>
  <td>Specifies the caching requirements for the disk.<br><br><strong>Value:</strong> <code>None</code> is the only value currently supported.<br><br><div class="alert alert--warning"><strong>Important</strong><p>Dedicated disk for <code>etcd</code> on Microsoft Azure is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process. For more information about the support scope of Red Hat Technology Preview features, see <a href="https://access.redhat.com/support/offerings/techpreview/">Technology Preview Features Support Scope</a>.</p></div></td>
</tr>
<tr>
  <td>controlPlane: platform: azure: dataDisks: - diskSizeGB:</td>
  <td>Specifies a dedicated disk size in GB.<br><br><strong>Value:</strong> Integer greater than <code>0</code>.<br><br><div class="alert alert--warning"><strong>Important</strong><p>Dedicated disk for <code>etcd</code> on Microsoft Azure is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process. For more information about the support scope of Red Hat Technology Preview features, see <a href="https://access.redhat.com/support/offerings/techpreview/">Technology Preview Features Support Scope</a>.</p></div></td>
</tr>
<tr>
  <td>controlPlane: platform: azure: dataDisks: - lun:</td>
  <td>Specifies a logical unit number (LUN) for the dedicated disk.<br><br><strong>Value:</strong> Integer from <code>0</code> through <code>63</code> that is not used by another disk.<br><br><div class="alert alert--warning"><strong>Important</strong><p>Dedicated disk for <code>etcd</code> on Microsoft Azure is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process. For more information about the support scope of Red Hat Technology Preview features, see <a href="https://access.redhat.com/support/offerings/techpreview/">Technology Preview Features Support Scope</a>.</p></div></td>
</tr>
<tr>
  <td>controlPlane: platform: azure: settings: securityType:</td>
  <td>Enables confidential VMs or trusted launch for control plane nodes. This option is not enabled by default.<br><br><strong>Value:</strong> <code>ConfidentialVM</code> or <code>TrustedLaunch</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: settings: confidentialVM: uefiSettings: secureBoot:</td>
  <td>Enables secure boot on control plane nodes if you are using confidential VMs.<br><br><strong>Value:</strong> <code>Enabled</code> or <code>Disabled</code>. The default is <code>Disabled</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: settings: confidentialVM: uefiSettings: virtualizedTrustedPlatformModule:</td>
  <td>Enables the vTPM feature on control plane nodes if you are using confidential VMs.<br><br><strong>Value:</strong> <code>Enabled</code> or <code>Disabled</code>. The default is <code>Disabled</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: settings: trustedLaunch: uefiSettings: secureBoot:</td>
  <td>Enables secure boot on control plane nodes if you are using trusted launch.<br><br><strong>Value:</strong> <code>Enabled</code> or <code>Disabled</code>. The default is <code>Disabled</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: settings: trustedLaunch: uefiSettings: virtualizedTrustedPlatformModule:</td>
  <td>Enables the vTPM feature on control plane nodes if you are using trusted launch.<br><br><strong>Value:</strong> <code>Enabled</code> or <code>Disabled</code>. The default is <code>Disabled</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: osDisk: securityProfile: securityEncryptionType:</td>
  <td>Enables the encryption of the virtual machine guest state for control plane nodes. This parameter can only be used if you use Confidential VMs.<br><br><strong>Value:</strong> <code>VMGuestStateOnly</code> is the only supported value.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: type:</td>
  <td>Defines the Azure instance type for control plane machines.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: zones:</td>
  <td>The availability zones where the installation program creates control plane machines.<br><br><strong>Value:</strong> String list</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: bootDiagnostics: type:</td>
  <td>Enables boot diagnostics collection for all machines. The <code>type</code> field specifies the Azure boot diagnostics type for all the created machines.<br><br>The following values are associated with the boot diagnostics type:<br><br><dl><dt><code>UserManaged</code></dt><dd>When you set <code>type</code> to <code>UserManaged</code>, you must provide the values for <code>resourceGroup</code> and <code>storageAccountName</code>. For <code>storageAccountName</code> and OpenShift Container Platform cluster nodes, ensure that you use the same region and subscription.</dd><dt><code>Managed</code></dt><dd>When you set <code>type</code> to <code>Managed</code>, Azure stores the boot diagnostics data blobs in a managed storage account.</dd><dt><code>Disabled</code></dt><dd>When you set <code>type</code> to <code>Disabled</code>, you turn off the parameter.</dd></dl><strong>Value:</strong> String, for example <code>Enabled</code>.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: bootDiagnostics: resourceGroup:</td>
  <td>Specifies the name of the Azure resource group that contains the diagnostic storage account for all machines. Use <code>resourceGroup</code> only when you set <code>type</code> to <code>UserManaged</code>.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: bootDiagnostics: storageAccountName:</td>
  <td>Specifies the Azure storage account to store the diagnostic logs for all machines. Use <code>storageAccountName</code> only when you set <code>type</code> to <code>UserManaged</code>.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: settings: securityType:</td>
  <td>Enables confidential VMs or trusted launch for all nodes. This option is not enabled by default.<br><br><strong>Value:</strong> <code>ConfidentialVM</code> or <code>TrustedLaunch</code>.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: settings: confidentialVM: uefiSettings: secureBoot:</td>
  <td>Enables secure boot on all nodes if you are using confidential VMs.<br><br><strong>Value:</strong> <code>Enabled</code> or <code>Disabled</code>. The default is <code>Disabled</code>.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: settings: confidentialVM: uefiSettings: virtualizedTrustedPlatformModule:</td>
  <td>Enables the virtualized Trusted Platform Module (vTPM) feature on all nodes if you are using confidential VMs.<br><br><strong>Value:</strong> <code>Enabled</code> or <code>Disabled</code>. The default is <code>Disabled</code>.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: settings: trustedLaunch: uefiSettings: secureBoot:</td>
  <td>Enables secure boot on all nodes if you are using trusted launch.<br><br><strong>Value:</strong> <code>Enabled</code> or <code>Disabled</code>. The default is <code>Disabled</code>.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: settings: trustedLaunch: uefiSettings: virtualizedTrustedPlatformModule:</td>
  <td>Enables the vTPM feature on all nodes if you are using trusted launch.<br><br><strong>Value:</strong> <code>Enabled</code> or <code>Disabled</code>. The default is <code>Disabled</code>.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: identity: type:</td>
  <td>The type of identity used for all virtual machines. The <code>UserAssigned</code> identity is a standalone Azure resource provided by the user and assigned to all virtual machines. If you specify <code>identity.type</code> as <code>UserAssigned</code>, but do not provide a user-assigned identity, the installation program creates the identity. If you provide a user-assigned identity, the Azure account that you use to create the identity must have either the "User Access Administrator" or "RBAC Access Admin" roles.<br><br><strong>Value:</strong> <code>UserAssigned</code> or <code>None</code>. If you do not specify a value, the installation program generates a user-assigned identity.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: identity: userAssignedIdentities: - name: resourceGroup: subscription:</td>
  <td>A group of parameters that specify the name of the user-assigned identity, and the resource group and subscription that contain the identity. All three values must be provided to specify a user-assigned identity. Only one user-assigned identity can be supplied. Supplying more than one user-assigned identity is an experimental feature, which may be enabled with the <code>MachineAPIMigration</code> feature gate.<br><br><strong>Value:</strong> Array of strings.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: osDisk: securityProfile: securityEncryptionType:</td>
  <td>Enables the encryption of the virtual machine guest state for all nodes. This parameter can only be used if you use Confidential VMs.<br><br><strong>Value:</strong> <code>VMGuestStateOnly</code> is the only supported value.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: encryptionAtHost:</td>
  <td>Enables host-level encryption for compute machines. You can enable this encryption alongside user-managed server-side encryption. This feature encrypts temporary, ephemeral, cached, and un-managed disks on the VM host. This parameter is not a prerequisite for user-managed server-side encryption.<br><br><strong>Value:</strong> <code>true</code> or <code>false</code>. The default is <code>false</code>.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: osDisk: diskEncryptionSet: name:</td>
  <td>The name of the disk encryption set that contains the encryption key from the installation prerequisites.<br><br><strong>Value:</strong> String, for example, <code>production_disk_encryption_set</code>.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: osDisk: diskEncryptionSet: resourceGroup:</td>
  <td>The name of the Azure resource group that contains the disk encryption set from the installation prerequisites. To avoid deleting your Azure encryption key when the cluster is destroyed, this resource group must be different from the resource group where you install the cluster. This value is necessary only if you intend to install the cluster with user-managed disk encryption.<br><br><strong>Value:</strong> String, for example, <code>production_encryption_resource_group</code>.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: osDisk: diskEncryptionSet: subscriptionId:</td>
  <td>Defines the Azure subscription of the disk encryption set where the disk encryption set resides. This secondary disk encryption set is used to encrypt compute machines.<br><br><strong>Value:</strong> String, in the format <code>00000000-0000-0000-0000-000000000000</code>.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: osDisk: diskSizeGB:</td>
  <td>The Azure disk size for the VM.<br><br><strong>Value:</strong> Integer that represents the size of the disk in GB. The default is <code>128</code>.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: osDisk: diskType:</td>
  <td>Defines the type of disk.<br><br><strong>Value:</strong> <code>premium_LRS</code> or <code>standardSSD_LRS</code>. The default is <code>premium_LRS</code>.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: osImage: publisher:</td>
  <td>Optional. By default, the installation program downloads and installs the Red&#160;Hat Enterprise Linux CoreOS (RHCOS) image that is used to boot control plane and compute machines. You can override the default behavior by using a custom RHCOS image that is available from the Azure Marketplace. The installation program uses this image for both types of machines. Control plane machines do not contribute to licensing costs when using the default image. But, if you apply an Azure Marketplace image for a control plane machine, usage costs do apply.<br><br><strong>Value:</strong> String. The name of the image publisher.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: osImage: offer:</td>
  <td>The name of Azure Marketplace offer that is associated with the custom RHCOS image. If you use <code>platform.azure.defaultMachinePlatform.osImage.publisher</code>, this field is required.<br><br><strong>Value:</strong> String. The name of the image offer.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: osImage: sku:</td>
  <td>An instance of the Azure Marketplace offer. If you use <code>platform.azure.defaultMachinePlatform.osImage.publisher</code>, this field is required.<br><br><strong>Value:</strong> String. The SKU of the image offer.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: osImage: version:</td>
  <td>The version number of the image SKU. If you use <code>platform.azure.defaultMachinePlatform.osImage.publisher</code>, this field is required.<br><br><strong>Value:</strong> String. The version of the image to use.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: type:</td>
  <td>The Azure instance type for control plane and compute machines.<br><br><strong>Value:</strong> The Azure instance type.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: zones:</td>
  <td>The availability zones where the installation program creates compute and control plane machines.<br><br><strong>Value:</strong> String list.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: encryptionAtHost:</td>
  <td>Enables host-level encryption for control plane machines. You can enable this encryption alongside user-managed server-side encryption. This feature encrypts temporary, ephemeral, cached, and un-managed disks on the VM host. This is not a prerequisite for user-managed server-side encryption.<br><br><strong>Value:</strong> <code>true</code> or <code>false</code>. The default is <code>false</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: osDisk: diskEncryptionSet: resourceGroup:</td>
  <td>The name of the Azure resource group that contains the disk encryption set from the installation prerequisites. This resource group should be different from the resource group where you install the cluster to avoid deleting your Azure encryption key when the cluster is destroyed. This value is only necessary if you intend to install the cluster with user-managed disk encryption.<br><br><strong>Value:</strong> String, for example <code>production_encryption_resource_group</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: osDisk: diskEncryptionSet: name:</td>
  <td>The name of the disk encryption set that contains the encryption key from the installation prerequisites.<br><br><strong>Value:</strong> String, for example <code>production_disk_encryption_set</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: osDisk: diskEncryptionSet: subscriptionId:</td>
  <td>Defines the Azure subscription of the disk encryption set where the disk encryption set resides. This secondary disk encryption set is used to encrypt control plane machines.<br><br><strong>Value:</strong> String, in the format <code>00000000-0000-0000-0000-000000000000</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: osDisk: diskSizeGB:</td>
  <td>The Azure disk size for the VM.<br><br><strong>Value:</strong> Integer that represents the size of the disk in GB. The default is <code>1024</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: osDisk: diskType:</td>
  <td>Defines the type of disk.<br><br><strong>Value:</strong> <code>premium_LRS</code> or <code>standardSSD_LRS</code>. The default is <code>premium_LRS</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: osImage: publisher:</td>
  <td>Optional. By default, the installation program downloads and installs the Red&#160;Hat Enterprise Linux CoreOS (RHCOS) image that is used to boot control plane machines. You can override the default behavior by using a custom RHCOS image that is available from the Azure Marketplace. The installation program uses this image for control plane machines only. Control plane machines do not contribute to licensing costs when using the default image. But, if you apply an Azure Marketplace image for a control plane machine, usage costs do apply.<br><br><strong>Value:</strong> String. The name of the image publisher.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: osImage: offer:</td>
  <td>The name of Azure Marketplace offer that is associated with the custom RHCOS image. If you use <code>controlPlane.platform.azure.osImage.publisher</code>, this field is required.<br><br><strong>Value:</strong> String. The name of the image offer.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: osImage: sku:</td>
  <td>An instance of the Azure Marketplace offer. If you use <code>controlPlane.platform.azure.osImage.publisher</code>, this field is required.<br><br><strong>Value:</strong> String. The SKU of the image offer.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: osImage: version:</td>
  <td>The version number of the image SKU. If you use <code>controlPlane.platform.azure.osImage.publisher</code>, this field is required.<br><br><strong>Value:</strong> String. The version of the image to use.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: ultraSSDCapability:</td>
  <td>Enables the use of Azure ultra disks for persistent storage on control plane machines. This requires that your Azure region and zone have ultra disks available.<br><br><strong>Value:</strong> <code>Enabled</code>, <code>Disabled</code>. The default is <code>Disabled</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: azure: vmNetworkingType:</td>
  <td>Enables accelerated networking. Accelerated networking enables single root I/O virtualization (SR-IOV) to a VM, improving its networking performance. If instance type of control plane machines support <code>Accelerated</code> networking, by default, the installation program enables <code>Accelerated</code> networking, otherwise the default networking type is <code>Basic</code>.<br><br><strong>Value:</strong> <code>Accelerated</code> or <code>Basic</code>.</td>
</tr>
<tr>
  <td>platform: azure: baseDomainResourceGroupName:</td>
  <td>The name of the resource group that contains the DNS zone for your base domain.<br><br><strong>Value:</strong> String, for example <code>production_cluster</code>.</td>
</tr>
<tr>
  <td>platform: azure: resourceGroupName:</td>
  <td>The name of an already existing resource group to install your cluster to. This resource group must be empty and only used for this specific cluster; the cluster components assume ownership of all resources in the resource group. If you limit the service principal scope of the installation program to this resource group, you must ensure all other resources used by the installation program in your environment have the necessary permissions, such as the public DNS zone and virtual network. Destroying the cluster by using the installation program deletes this resource group.<br><br><strong>Value:</strong> String, for example <code>existing_resource_group</code>.</td>
</tr>
<tr>
  <td>platform: azure: outboundType:</td>
  <td>The outbound routing strategy used to connect your cluster to the internet. The following strategies are available:<br><br><dl><dt><code>UserDefinedRouting</code></dt><dd>Specifies to the installation program that you will provide and configure your own networking infrastructure for outbound access. The outbound routing must be configured before installing a cluster. The installation program does not configure user-defined routing.</dd><dt><code>LoadBalancer</code></dt><dd>Specifies that a single load balancer will be provisioned to provide outbound access for your cluster. This is the default value.</dd><dt><code>NATGatewaySingleZone</code></dt><dd>Specifies that the installation program will create one NAT Gateway. If you provide your own subnets via the <code>platform.azure.subnets</code> parameter, the installation program will attach the NAT Gateway to the compute subnet you specify. If you do not provide your own subnets, the installation program will create a subnet for the control plane and a subnet for the compute plane, and attach the NAT Gateway to the compute subnet.</dd><dt><code>NATGatewayMultiZone</code></dt><dd>Specifies that the installation program will create multiple NAT Gateways. If you provide your own subnets via the <code>platform.azure.subnets</code> parameter, the installation program creates a NAT Gateway for each subnet with the <code>node</code> role, assigns a zone to each NAT Gateway, and associates a NAT Gateway to each subnet. If you do not provide your own subnets, the installation program creates a compute subnet and NAT Gateway for each zone in the region, then attaches them to each other.</dd></dl>If you specify either the <code>NATGatewaySingleZone</code> or the <code>NATGatewayMultiZone</code> routing strategy, your account must have the <code>Microsoft.Network/natGateways/read</code> and <code>Microsoft.Network/natGateways/write</code> permissions. NAT Gateways can only be used for compute machines.<br><br><strong>Value:</strong> <code>LoadBalancer</code>, <code>UserDefinedRouting</code>, <code>NATGatewaySingleZone</code>, or <code>NATGatewayMultiZone</code>. The default is <code>LoadBalancer</code>.</td>
</tr>
<tr>
  <td>platform: azure: region:</td>
  <td>The name of the Azure region that hosts your cluster.<br><br><strong>Value:</strong> Any valid region name, such as <code>centralus</code>.</td>
</tr>
<tr>
  <td>platform: azure: subnets: - name: role:</td>
  <td>A list of one or more pairs of parameters which specify the name and role of a pre-existing subnet. The installation program will use the provided subnets for the specified roles. You can only specify one subnet with the <code>control-plane</code> role. If you specify pre-existing subnets, you must also set the <code>platform.azure.networkResourceGroupName</code> and <code>platform.azure.virtualNetwork</code> parameters. Pre-existing subnets that you provide must use the same region as you specified in the <code>platform.azure.region</code> parameter. If you use the <code>NATGatewaySingleZone</code> outbound routing strategy, you can only specify one subnet with the <code>node</code> role.<br><br><strong>Value:</strong> <code>name</code> specifies the name of the subnet. Valid <code>role</code> values are <code>node</code> or <code>control-plane</code>.</td>
</tr>
<tr>
  <td>platform: azure: userProvisionedDNS:</td>
  <td>Enables user-provisioned DNS instead of the default cluster-provisioned DNS solution. If you use this feature, you must provide your own DNS solution that includes records for <code>api.&lt;cluster_name&gt;.&lt;base_domain&gt;.</code> and <code>*.apps.&lt;cluster_name&gt;.&lt;base_domain&gt;.</code>.<br><br><strong>Value:</strong> <code>Enabled</code> or <code>Disabled</code>. The default value is <code>Disabled</code>.</td>
</tr>
<tr>
  <td>platform: azure: zone:</td>
  <td>List of availability zones to place machines in. For high availability, specify at least two zones.<br><br><strong>Value:</strong> List of zones, for example <code>["1", "2", "3"]</code>.</td>
</tr>
<tr>
  <td>platform: azure: customerManagedKey: keyVault: name:</td>
  <td>Specifies the name of the key vault that contains the encryption key that is used to encrypt Azure storage.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>platform: azure: customerManagedKey: keyVault: keyName:</td>
  <td>Specifies the name of the user-managed encryption key that is used to encrypt Azure storage.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>platform: azure: customerManagedKey: keyVault: resourceGroup:</td>
  <td>Specifies the name of the resource group that contains the key vault and managed identity.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>platform: azure: customerManagedKey: userAssignedIdentityKey:</td>
  <td>Specifies the name of the user-assigned managed identity that resides in the resource group with the key vault and has access to the user-managed key.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: ultraSSDCapability:</td>
  <td>Enables the use of Azure ultra disks for persistent storage on control plane and compute machines. This requires that your Azure region and zone have ultra disks available.<br><br><strong>Value:</strong> <code>Enabled</code>, <code>Disabled</code>. The default is <code>Disabled</code>.</td>
</tr>
<tr>
  <td>platform: azure: networkResourceGroupName:</td>
  <td>The name of the resource group that contains the existing VNet that you want to deploy your cluster to. This name cannot be the same as the <code>platform.azure.baseDomainResourceGroupName</code>.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>platform: azure: virtualNetwork:</td>
  <td>The name of the existing VNet that you want to deploy your cluster to.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>platform: azure: controlPlaneSubnet:</td>
  <td>The name of the existing subnet in your VNet that you want to deploy your control plane machines to.<br><br><strong>Value:</strong> Valid CIDR, for example <code>10.0.0.0/16</code>.</td>
</tr>
<tr>
  <td>platform: azure: computeSubnet:</td>
  <td>The name of the existing subnet in your VNet that you want to deploy your compute machines to.<br><br><strong>Value:</strong> Valid CIDR, for example <code>10.0.0.0/16</code>.</td>
</tr>
<tr>
  <td>platform: azure: cloudName:</td>
  <td>The name of the Azure cloud environment that is used to configure the Azure SDK with the appropriate Azure API endpoints. If empty, the default value <code>AzurePublicCloud</code> is used.<br><br><strong>Value:</strong> Any valid cloud environment, such as <code>AzurePublicCloud</code> or <code>AzureUSGovernmentCloud</code>.</td>
</tr>
<tr>
  <td>platform: azure: defaultMachinePlatform: vmNetworkingType:</td>
  <td>Enables accelerated networking. Accelerated networking enables single root I/O virtualization (SR-IOV) to a VM, improving its networking performance.<br><br><strong>Value:</strong> <code>Accelerated</code> or <code>Basic</code>. If instance type of control plane and compute machines support <code>Accelerated</code> networking, by default, the installation program enables <code>Accelerated</code> networking, otherwise the default networking type is <code>Basic</code>.</td>
</tr>
<tr>
  <td>operatorPublishingStrategy: apiserver:</td>
  <td>Determines whether the load balancers that service the API are public or private. Set this parameter to <code>Internal</code> to prevent the API server from being accessible outside of your VNet. Set this parameter to <code>External</code> to make the API server accessible outside of your VNet. If you set this parameter, you must set the <code>publish</code> parameter to <code>Mixed</code>.<br><br><strong>Value:</strong> <code>External</code> or <code>Internal</code>. The default value is <code>External</code>.</td>
</tr>
<tr>
  <td>operatorPublishingStrategy: ingress:</td>
  <td>Determines whether the DNS resources that the cluster creates for ingress traffic are publicly visible. Set this parameter to <code>Internal</code> to prevent the ingress VIP from being publicly accessible. Set this parameter to <code>External</code> to make the ingress VIP publicly accessible. If you set this parameter, you must set the <code>publish</code> parameter to <code>Mixed</code>.<br><br><strong>Value:</strong> <code>External</code> or <code>Internal</code>. The default value is <code>External</code>.</td>
</tr>
</tbody>
</table>

:::note

You cannot customize [Azure Availability Zones](https://azure.microsoft.com/en-us/global-infrastructure/availability-zones/) or [Use tags to organize your Azure resources](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-using-tags) with an Azure cluster.

:::
