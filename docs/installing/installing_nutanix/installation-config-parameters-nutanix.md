---
title: Installation configuration parameters for Nutanix
sidebar_position: 7
---

# Installation configuration parameters for Nutanix {#installation-config-parameters-nutanix}

<a id="installation-config-parameters-nutanix"></a>

Before you deploy an OpenShift Container Platform cluster on Nutanix, you provide parameters to customize your cluster and the platform that hosts it. When you create the `install-config.yaml` file, you provide values for the required parameters through the command line. You can then modify the `install-config.yaml` file to customize your cluster further.

## Available installation configuration parameters for Nutanix {#installation-configuration-parameters_installation-config-parameters-nutanix}

To customize your cluster installation, you can use configuration parameters in the `install-config.yaml` file.

The following tables specify the required, optional, and Nutanix-specific installation configuration parameters that you can set as part of the installation process.

:::warning

After installation, you cannot change these parameters in the `install-config.yaml` file.

:::

### Required configuration parameters {#installation-configuration-parameters-required_installation-config-parameters-nutanix}

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
  <td>The name of the cluster. DNS records for the cluster are all subdomains of <code>{.metadata.name}.{.baseDomain}</code>.  <br><br>   <strong>Value:</strong> String of lowercase letters and hyphens (<code>-</code>), such as <code>dev</code>.  </td>
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

### Network configuration parameters {#installation-configuration-parameters-network_installation-config-parameters-nutanix}

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

### Optional configuration parameters {#installation-configuration-parameters-optional_installation-config-parameters-nutanix}

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

### Additional Nutanix configuration parameters {#installation-configuration-parameters-additional-nutanix_installation-config-parameters-nutanix}

Additional Nutanix configuration parameters are described in the following table:

**Additional Nutanix cluster parameters**

<table>
<thead>
<tr>
  <th>Parameter</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>compute: platform: nutanix: categories: key:</td>
  <td>The name of a prism category key to apply to compute VMs. This parameter must be accompanied by the <code>value</code> parameter, and both <code>key</code> and <code>value</code> parameters must exist in Prism Central. For more information on categories, see <a href="https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Guide-vpc_2022_6:ssp-ssp-categories-manage-pc-c.html">Category management</a>.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>compute: platform: nutanix: categories: value:</td>
  <td>The value of a prism category key-value pair to apply to compute VMs. This parameter must be accompanied by the <code>key</code> parameter, and both <code>key</code> and <code>value</code> parameters must exist in Prism Central.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>compute: platform: nutanix: failureDomains:</td>
  <td>The failure domains that apply to only compute machines.<br><br>Failure domains are specified in <code>platform.nutanix.failureDomains</code>.<br><br><strong>Value:</strong> List.<br><br>The name of one or more failures domains.</td>
</tr>
<tr>
  <td>compute: platform: nutanix: gpus: type:</td>
  <td>The type of identifier used to attach a GPU to a compute machine. Valid values are "Name" or "DeviceID".<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>compute: platform: nutanix: gpus: name:</td>
  <td>The name of the GPU device to attach to a compute machine. This parameter is required if the GPU <code>type</code> is "Name".<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>compute: platform: nutanix: gpus: deviceID:</td>
  <td>The device identifier of the GPU device to attach to a compute machine. This information is available in Prism Central. This parameter is required if the GPU <code>type</code> is "DeviceID".<br><br><strong>Value:</strong> Integer</td>
</tr>
<tr>
  <td>compute: platform: nutanix: project: type:</td>
  <td>The type of identifier you use to select a project for compute VMs.  Projects define logical groups of user roles for managing permissions, networks, and other parameters. For more information on projects, see <a href="https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Guide-vpc_2022_9:ssp-app-mgmt-project-env-c.html">Projects Overview</a>.<br><br><strong>Value:</strong> <code>name</code> or <code>uuid</code></td>
</tr>
<tr>
  <td>compute: platform: nutanix: project: name: or uuid:</td>
  <td>The name or UUID of a project with which compute VMs are associated. This parameter must be accompanied by the <code>type</code> parameter.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>compute: platform: nutanix: bootType:</td>
  <td>The boot type that the compute machines use. You must use the <code>Legacy</code> boot type in OpenShift Container Platform 4.22. For more information on boot types, see <a href="https://portal.nutanix.com/page/documents/kbs/details?targetId=kA07V000000H3K9SAK">Understanding UEFI, Secure Boot, and TPM in the Virtualized Environment</a>.<br><br><strong>Value:</strong> <code>Legacy</code>, <code>SecureBoot</code> or <code>UEFI</code>. The default is <code>Legacy</code>.</td>
</tr>
<tr>
  <td>compute: platform: nutanix: dataDisks: dataSourceImage: name:</td>
  <td>Optional. The name of the data source image for the virtual machine disk in Prism Central.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>compute: platform: nutanix: dataDisks: dataSourceImage: referenceName:</td>
  <td>Optional. The reference name of the data source image in the failure domain. If you use this parameter, you must configure a matching <code>dataSourceImage</code> with the same <code>referenceName</code> in each failure domain that the compute nodes occupy. For more information about configuring failure domains, see <em>Configuring failure domains</em> in the <em>Installing a cluster on Nutanix</em> page.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>compute: platform: nutanix: dataDisks: dataSourceImage: uuid:</td>
  <td>The UUID of the data source image in Prism Central. This value is required.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>compute: platform: nutanix: dataDisks: deviceProperties: adapterType:</td>
  <td>The adapter type of the disk address. If the disk type is "Disk", valid values are "SCSI", "IDE", "PCI", "SATA" or "SPAPR". If the disk type is "CDRom", valid values are "IDE" or "SATA".<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>compute: platform: nutanix: dataDisks: deviceProperties: deviceIndex:</td>
  <td>The index of the disk address. Valid values are non-negative integers including <code>0</code>. The device index for disks that share the same adapter type should start at 0 and increase consecutively. The default value is <code>0</code>. For each virtual machine, the <code>Disk.SCSI.0</code> and <code>CDRom.IDE.0</code> indices are reserved. If you use the <code>Disk.SCSI</code> or <code>CDRom.IDE</code> disk and adapter types, the <code>deviceIndex</code> should start at <code>1</code>.<br><br><strong>Value:</strong> Non-negative integer, including <code>0</code>.</td>
</tr>
<tr>
  <td>compute: platform: nutanix: dataDisks: deviceProperties: deviceType:</td>
  <td>The disk device type. Valid values are "Disk" and "CDRom".<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>compute: platform: nutanix: dataDisks: diskSize:</td>
  <td>The size of the disk to attach to the virtual machine. The minimum size is 1Gb.<br><br><strong>Value:</strong> Quantity format, such as 100G or 100Gi. For more information on this format, see link:https://pkg.go.dev/k8s.io/apimachinery/pkg/api/resource#Format.</td>
</tr>
<tr>
  <td>compute: platform: nutanix: dataDisks: storageConfig: diskMode:</td>
  <td>The disk mode. Valid values are <code>Standard</code> or <code>Flash</code>, and the default is <code>Standard</code>.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>compute: platform: nutanix: dataDisks: storageConfig: storageContainer: name:</td>
  <td>Optional. The name of the storage container object used by the virtual machine disk in Prism Central.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>compute: platform: nutanix: dataDisks: storageConfig: storageContainer: referenceName:</td>
  <td>Optional. The reference name of the storage container in the failure domain. If you use this, you must configure a matching <code>storageContainer</code> with the same <code>referenceName</code> in each failure domain the compute nodes occupy. For more information about configuring failure domains, see <em>Configuring failure domains</em> in the <em>Installing a cluster on Nutanix</em> page.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>compute: platform: nutanix: dataDisks: storageConfig: storageContainer: uuid:</td>
  <td>The UUID of the storage container in Prism Central.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>controlPlane: platform: nutanix: categories: key:</td>
  <td>The name of a prism category key to apply to control plane VMs. This parameter must be accompanied by the <code>value</code> parameter, and both <code>key</code> and <code>value</code> parameters must exist in Prism Central. For more information on categories, see <a href="https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Guide-vpc_2022_6:ssp-ssp-categories-manage-pc-c.html">Category management</a>.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>controlPlane: platform: nutanix: categories: value:</td>
  <td>The value of a prism category key-value pair to apply to control plane VMs. This parameter must be accompanied by the <code>key</code> parameter, and both <code>key</code> and <code>value</code> parameters must exist in Prism Central.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>controlPlane: platform: nutanix: failureDomains:</td>
  <td>The failure domains that apply to only control plane machines.<br><br>Failure domains are specified in <code>platform.nutanix.failureDomains</code>.<br><br><strong>Value:</strong> List.<br><br>The name of one or more failures domains.</td>
</tr>
<tr>
  <td>controlPlane: platform: nutanix: project: type:</td>
  <td>The type of identifier you use to select a project for control plane VMs.  Projects define logical groups of user roles for managing permissions, networks, and other parameters. For more information on projects, see <a href="https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Guide-vpc_2022_9:ssp-app-mgmt-project-env-c.html">Projects Overview</a>.<br><br><strong>Value:</strong> <code>name</code> or <code>uuid</code></td>
</tr>
<tr>
  <td>controlPlane: platform: nutanix: project: name: or uuid:</td>
  <td>The name or UUID of a project with which control plane VMs are associated. This parameter must be accompanied by the <code>type</code> parameter.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: nutanix: defaultMachinePlatform: categories: key:</td>
  <td>The name of a prism category key to apply to all VMs. This parameter must be accompanied by the <code>value</code> parameter, and both <code>key</code> and <code>value</code> parameters must exist in Prism Central. For more information on categories, see <a href="https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Guide-vpc_2022_6:ssp-ssp-categories-manage-pc-c.html">Category management</a>.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: nutanix: defaultMachinePlatform: categories: value:</td>
  <td>The value of a prism category key-value pair to apply to all VMs. This parameter must be accompanied by the <code>key</code> parameter, and both <code>key</code> and <code>value</code> parameters must exist in Prism Central.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: nutanix: defaultMachinePlatform: failureDomains:</td>
  <td>The failure domains that apply to both control plane and compute machines.<br><br>Failure domains are specified in <code>platform.nutanix.failureDomains</code>.<br><br><strong>Value:</strong> List.<br><br>The name of one or more failures domains.</td>
</tr>
<tr>
  <td>platform: nutanix: defaultMachinePlatform: project: type:</td>
  <td>The type of identifier you use to select a project for all VMs. Projects define logical groups of user roles for managing permissions, networks, and other parameters. For more information on projects, see <a href="https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Guide-vpc_2022_9:ssp-app-mgmt-project-env-c.html">Projects Overview</a>.<br><br><strong>Value:</strong> <code>name</code> or <code>uuid</code>.</td>
</tr>
<tr>
  <td>platform: nutanix: defaultMachinePlatform: project: name: or uuid:</td>
  <td>The name or UUID of a project with which all VMs are associated. This parameter must be accompanied by the <code>type</code> parameter.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: nutanix: defaultMachinePlatform: bootType:</td>
  <td>The boot type for all machines. You must use the <code>Legacy</code> boot type in OpenShift Container Platform 4.22. For more information on boot types, see <a href="https://portal.nutanix.com/page/documents/kbs/details?targetId=kA07V000000H3K9SAK">Understanding UEFI, Secure Boot, and TPM in the Virtualized Environment</a>.<br><br><strong>Value:</strong> <code>Legacy</code>, <code>SecureBoot</code> or <code>UEFI</code>. The default is <code>Legacy</code>.</td>
</tr>
<tr>
  <td>platform: nutanix: apiVIP:</td>
  <td>The virtual IP (VIP) address that you configured for control plane API access.<br><br><strong>Value:</strong> IP address</td>
</tr>
<tr>
  <td>platform: nutanix: failureDomains: - name: prismElement: name: uuid: subnetUUIDs: -</td>
  <td>By default, the installation program installs cluster machines to a single Prism Element instance. A maximum of 32 subnets for each failure domain (Prism Element) in an OpenShift Container Platform cluster is supported. All <code>subnetUUID</code> values must be unique. You can specify additional Prism Element instances for fault tolerance, and then apply them to:<br><br><ul><li>The cluster's default machine configuration</li><li>Only control plane or compute machine pools</li></ul><strong>Value:</strong> A list of configured failure domains.<br><br>For more information on usage, see "Configuring a failure domain" in "Installing a cluster on Nutanix".</td>
</tr>
<tr>
  <td>platform: nutanix: ingressVIP:</td>
  <td>The virtual IP (VIP) address that you configured for cluster ingress.<br><br><strong>Value:</strong> IP address</td>
</tr>
<tr>
  <td>platform: nutanix: prismCentral: endpoint: address:</td>
  <td>The Prism Central domain name or IP address.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: nutanix: prismCentral: endpoint: port:</td>
  <td>The port that is used to log into Prism Central.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: nutanix: prismCentral: password:</td>
  <td>The password for the Prism Central user name.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: nutanix: preloadedOSImageName:</td>
  <td>Instead of creating and uploading a RHCOS image object for each OpenShift Container Platform cluster, this parameter uses the named, preloaded RHCOS image object from the Prism Elements to which the OpenShift Container Platform cluster is deployed.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: nutanix: prismCentral: username:</td>
  <td>The user name that is used to log into Prism Central.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: nutanix: prismElements: endpoint: address:</td>
  <td>The Prism Element domain name or IP address. [<sup>1</sup>]<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: nutanix: prismElements: endpoint: port:</td>
  <td>The port that is used to log into Prism Element.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: nutanix: prismElements: uuid:</td>
  <td>The universally unique identifier (UUID) for Prism Element.<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: nutanix: subnetUUIDs:</td>
  <td>The UUID of the Prism Element network that contains the virtual IP addresses and DNS records that you configured. [<sup>2</sup>]<br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: nutanix: clusterOSImage:</td>
  <td>Optional: By default, the installation program downloads and installs the Red&#160;Hat Enterprise Linux CoreOS (RHCOS) image. If Prism Central does not have internet access, you can override the default behavior by hosting the RHCOS image on any HTTP server and pointing the installation program to the image.<br><br><strong>Value:</strong> An HTTP or HTTPS URL, optionally with a SHA-256 checksum. For example, \http://example.com/images/rhcos-47.83.202103221318-0-nutanix.x86_64.qcow2</td>
</tr>
</tbody>
</table>

1. The `prismElements` section holds a list of Prism Elements (clusters). A Prism Element encompasses all of the Nutanix resources, for example virtual machines and subnets, that are used to host the OpenShift Container Platform cluster.
2. A maximum of 32 subnets for each Prism Element in an OpenShift Container Platform cluster is supported. All `subnetUUID` values must be unique.
