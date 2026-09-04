---
title: Installation configuration parameters for vSphere
sidebar_position: 9
---

# Installation configuration parameters for vSphere {#installation-config-parameters-vsphere}

<a id="installation-config-parameters-vsphere"></a>

Before you deploy an OpenShift Container Platform cluster on vSphere, you can configure parameters to customize your cluster and the platform that hosts it. The installation program uses the information in the `install-config.yaml` file to provision required infrastructure and deploy cluster components. When you create the `install-config.yaml` file, you can configure the values for your required parameters through the command line. Edit the `install-config.yaml` file to customize your cluster further before installation begins.

## Available installation configuration parameters for vSphere {#installation-configuration-parameters_installation-config-parameters-vsphere}

To customize your cluster installation, you can use configuration parameters in the `install-config.yaml` file.

The following tables specify the required, optional, and vSphere-specific installation configuration parameters that you can set as part of the installation process.

:::warning

After installation, you cannot change these parameters in the `install-config.yaml` file.

:::

### Required configuration parameters {#installation-configuration-parameters-required_installation-config-parameters-vsphere}

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

### Network configuration parameters {#installation-configuration-parameters-network_installation-config-parameters-vsphere}

You can customize your installation configuration based on the requirements of your existing network infrastructure. For example, you can expand the IP address block for the cluster network or configure different IP address blocks than the defaults.

Consider the following information before you configure network parameters for your cluster:

- If you use the Red Hat OpenShift Networking OVN-Kubernetes network plugin, both IPv4 and IPv6 address families are supported.
- If you deployed nodes in an OpenShift Container Platform cluster with a network that supports both IPv4 and non-link-local IPv6 addresses, configure your cluster to use a dual-stack network.
  - For clusters configured for dual-stack networking, both IPv4 and IPv6 traffic must use the same network interface as the default gateway. This ensures that in a multiple network interface controller (NIC) environment, a cluster can detect what NIC to use based on the available network interface. For more information, see "OVN-Kubernetes IPv6 and dual-stack limitations" in *About the OVN-Kubernetes network plugin*.
  - To prevent network connectivity issues, do not install a single-stack IPv4 cluster on a host that supports dual-stack networking.

  :::note

  On VMware vSphere, dual-stack networking can specify either IPv4 or IPv6 as the primary address family.

  :::

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

### Optional configuration parameters {#installation-configuration-parameters-optional_installation-config-parameters-vsphere}

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

### Additional VMware vSphere configuration parameters {#installation-configuration-parameters-additional-vsphere_installation-config-parameters-vsphere}

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
  <td>platform: vsphere: apiVIPs:</td>
  <td>Virtual IP (VIP) addresses that you configured for control plane API access.<div class="alert alert--secondary"><strong>Note</strong><p>This parameter applies only to installer-provisioned infrastructure without an external load balancer configured. You must not specify this parameter in user-provisioned infrastructure.<br><br>The <code>apiVIP</code> and <code>ingressVIP</code> parameters must come from the same network segment as the <code>networking.machineNetwork</code> parameter. If the <code>networking.machineNetwork</code> parameter is set to <code>10.0.0.0/16</code> then the API and Ingress VIPs must be in one of the <code>10.0.0.0/16</code> machine networks.</p></div><br><br><strong>Value:</strong> Multiple IP addresses</td>
</tr>
<tr>
  <td>platform: vsphere: diskType:</td>
  <td>Optional: The disk provisioning method. This value defaults to the vSphere default storage policy if not set.<br><br><strong>Value:</strong> Valid values are <code>thin</code>, <code>thick</code>, or <code>eagerZeroedThick</code>.</td>
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
  <td>Specifies the path to a vSphere datastore that stores virtual machines files for a failure domain. You must apply the <code>datastore</code> role to the vSphere vCenter datastore location.<br><br><div class="alert alert--warning"><strong>Important</strong><p>You can specify the path of any datastore that exists in a datastore cluster. By default, Storage vMotion is automatically enabled for a datastore cluster. Red Hat does not support Storage vMotion, so you must disable Storage vMotion to avoid data loss issues for your OpenShift Container Platform cluster.<br><br>If you must specify VMs across multiple datastores, use a <code>datastore</code> object to specify a failure domain in your cluster's <code>install-config.yaml</code> configuration file. For more information, see "VMware vSphere region and zone enablement".</p></div><br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: failureDomains: topology: folder:</td>
  <td>Optional: The absolute path of an existing folder where the user creates the virtual machines, for example, <code>/&lt;data_center_name&gt;/vm/&lt;folder_name&gt;/&lt;subfolder_name&gt;</code>.  If you do not provide this value, the installation program creates a top-level folder in the data center virtual machine folder that is named with the infrastructure ID. If you are providing the infrastructure for the cluster and you do not want to use the default <code>StorageClass</code> object, named <code>thin</code>, you can omit the <code>folder</code> parameter from the <code>install-config.yaml</code> file.  <br><br><strong>Value:</strong> String</td>
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
  <td>Optional: The absolute path of an existing resource pool where the installation program creates the virtual machines, for example, <code>/&lt;data_center_name&gt;/host/&lt;cluster_name&gt;/Resources/&lt;resource_pool_name&gt;/&lt;optional_nested_resource_pool_name&gt;</code>.  If you do not specify a value, the installation program installs the resources in the root of the cluster under <code>/&lt;data_center_name&gt;/host/&lt;cluster_name&gt;/Resources</code>.  <br><br><strong>Value:</strong> String</td>
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
  <td>platform: vsphere: ingressVIPs:</td>
  <td>Virtual IP (VIP) addresses that you configured for cluster Ingress.<div class="alert alert--secondary"><strong>Note</strong><p>This parameter applies only to installer-provisioned infrastructure without an external load balancer configured. You must not specify this parameter in user-provisioned infrastructure.<br><br>The <code>apiVIP</code> and <code>ingressVIP</code> parameters must come from the same network segment as the <code>networking.machineNetwork</code> parameter. If the <code>networking.machineNetwork</code> parameter is set to <code>10.0.0.0/16</code> then the API and Ingress VIPs must be in one of the <code>10.0.0.0/16</code> machine networks.</p></div><br><br><strong>Value:</strong> Multiple IP addresses</td>
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

### Deprecated VMware vSphere configuration parameters {#deprecated-parameters-vsphere_installation-config-parameters-vsphere}

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
  <td>platform: vsphere: apiVIP:</td>
  <td>The virtual IP (VIP) address that you configured for control plane API access.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>In OpenShift Container Platform 4.12 and later, the <code>apiVIP</code> configuration setting is deprecated. Instead, use a <code>List</code> format to enter a value in the <code>apiVIPs</code> configuration setting.</p></div><br><br><strong>Value:</strong> An IP address, for example <code>128.0.0.1</code>.</td>
</tr>
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
  <td>platform: vsphere: ingressVIP:</td>
  <td>Virtual IP (VIP) addresses that you configured for cluster Ingress.<div class="alert alert--secondary"><strong>Note</strong><p>In OpenShift Container Platform 4.12 and later, the <code>ingressVIP</code> configuration setting is deprecated. Instead, use a <code>List</code> format to enter a value in the <code>ingressVIPs</code> configuration setting.</p></div><br><br><strong>Value:</strong> An IP address, for example <code>128.0.0.1</code>.</td>
</tr>
<tr>
  <td>platform: vsphere: network:</td>
  <td>The network in the vCenter instance that contains the virtual IP addresses and DNS records that you configured.<br><br><strong>Value:</strong> String</td>
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

### Optional VMware vSphere machine pool configuration parameters {#installation-configuration-parameters-optional-vsphere_installation-config-parameters-vsphere}

Optional VMware vSphere machine pool configuration parameters are described in the following table:

**Optional VMware vSphere machine pool parameters**

<table>
<thead>
<tr>
  <th>Parameter</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>platform: vsphere: clusterOSImage:</td>
  <td>The location from which the installation program downloads the Red&#160;Hat Enterprise Linux CoreOS (RHCOS) image. Before setting a path value for this parameter, ensure that the default RHCOS boot image in the OpenShift Container Platform release matches the RHCOS image template or virtual machine version; otherwise, cluster installation might fail. As an alternative to this configuration, you can use the <code>topology.template</code> parameter to point to the path in your vCenter environment that includes an RHCOS image in Open Virtual Appliance (OVA) format.<br><br><strong>Value:</strong> An HTTP or HTTPS URL, optionally with a SHA-256 checksum. For example, <code>\https://mirror.openshift.com/images/rhcos-&lt;version&gt;-vmware.&lt;architecture&gt;.ova</code>.</td>
</tr>
<tr>
  <td>platform: vsphere: osDisk: diskSizeGB:</td>
  <td>The size of the disk in gigabytes.<br><br><strong>Value:</strong> Integer</td>
</tr>
<tr>
  <td>platform: vsphere: cpus:</td>
  <td>The total number of virtual processor cores to assign a virtual machine. The value of <code>platform.vsphere.cpus</code> must be a multiple of <code>platform.vsphere.coresPerSocket</code> value.<br><br><strong>Value:</strong> Integer</td>
</tr>
<tr>
  <td>platform: vsphere: coresPerSocket:</td>
  <td>The number of cores per socket in a virtual machine, where <code>platform.vsphere.cpus</code> divided by <code>platform.vsphere.coresPerSocket</code> determines the number of virtual sockets on a virtual machine. Control plane nodes and compute nodes default to <code>4</code> virtual sockets on a virtual machine.<br><br><strong>Value:</strong> Integer</td>
</tr>
<tr>
  <td>platform: vsphere: memoryMB:</td>
  <td>The size of a virtual machine's memory in megabytes.<br><br><strong>Value:</strong> Integer</td>
</tr>
<tr>
  <td>platform: vsphere: dataDisks: name:</td>
  <td>The name of the data disk to add to the virtual machines. The maximum name length is 80 characters.<br><br><div class="alert alert--warning"><strong>Important</strong><p>Installing OpenShift Container Platform on VMware vSphere using multiple data disks is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.<br><br>For more information about the support scope of Red Hat Technology Preview features, see <a href="https://access.redhat.com/support/offerings/techpreview/">Technology Preview Features Support Scope</a>.</p></div><br><br><strong>Value:</strong> String</td>
</tr>
<tr>
  <td>platform: vsphere: dataDisks: sizeGiB:</td>
  <td>The size of the data disk to add to the virtual machines. The maximum size is 16384 GiB.<br><br><strong>Value:</strong> Integer</td>
</tr>
<tr>
  <td>platform: vsphere: dataDisks: provisioningMode:</td>
  <td>Optional: The data disk provisioning method. This value defaults to the vSphere default storage policy, if not set.<br><br><strong>Value:</strong> Valid values are <code>Thin</code>, <code>Thick</code>, or <code>EagerlyZeroed</code>.</td>
</tr>
</tbody>
</table>
