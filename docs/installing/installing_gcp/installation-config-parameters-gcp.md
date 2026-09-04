---
title: Installation configuration parameters for Google Cloud
sidebar_position: 13
---

# Installation configuration parameters for Google Cloud {#installation-config-parameters-gcp}

<a id="installation-config-parameters-gcp"></a>

Before you deploy an OpenShift Container Platform cluster on Google Cloud, you provide parameters to customize your cluster and the platform that hosts it. When you create the `install-config.yaml` file, you provide values for the required parameters through the command line. You can then modify the `install-config.yaml` file to customize your cluster further.

## Available installation configuration parameters for Google Cloud {#installation-configuration-parameters_installation-config-parameters-gcp}

To customize your cluster installation, you can use configuration parameters in the `install-config.yaml` file.

The following tables specify the required, optional, and Google Cloud-specific installation configuration parameters that you can set as part of the installation process.

:::warning

After installation, you cannot change these parameters in the `install-config.yaml` file.

:::

### Required configuration parameters {#installation-configuration-parameters-required_installation-config-parameters-gcp}

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

### Network configuration parameters {#installation-configuration-parameters-network_installation-config-parameters-gcp}

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
  <td>Required if you use <code>networking.machineNetwork</code>. An IP address block. The default value is <code>10.0.0.0/16</code> for all platforms other than libvirt and IBM Power(R) Virtual Server. For libvirt, the default value is <code>192.168.126.0/24</code>. For IBM Power(R) Virtual Server, the default value is <code>192.168.0.0/24</code>.<br><br>  <br><br><strong>Value:</strong> An IP network block in CIDR notation.<br><br>  For example, <code>10.0.0.0/16</code>.    <br><br><div class="alert alert--secondary"><strong>Note</strong><p>Set the <code>networking.machineNetwork</code> to match the CIDR that the preferred NIC resides in.<br><br>If you are installing a cluster on AWS with dual-stack networking, consider the following distinction:<br><br><ul><li>If the installation program creates the VPC, do not specify an IPv6 entry in <code>networking.machineNetwork</code>. The installation program will assign an IPv6 address to the VPC.</li><li>If you provide existing dual-stack subnets using the <code>platform.aws.vpc.subnets</code> parameter, you must specify IPv6 entries corresponding to either the VPC CIDR or the CIDR of the subnets.</li><li>In both cases, you must provide an IPv4 CIDR entry.</li></ul></p></div></td>
</tr>
<tr>
  <td>networking: ovnKubernetesConfig: ipv4: internalJoinSubnet:</td>
  <td>Configures the IPv4 join subnet that is used internally by <code>ovn-kubernetes</code>. This subnet must not overlap with any other subnet that OpenShift Container Platform is using, including the node network. The size of the subnet must be larger than the number of nodes. You cannot change the value after installation.<br><br><strong>Value:</strong> An IP network block in CIDR notation. The default value is <code>100.64.0.0/16</code>.</td>
</tr>
</tbody>
</table>

### Optional configuration parameters {#installation-configuration-parameters-optional_installation-config-parameters-gcp}

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
  <td>Determines the instruction set architecture of the machines in the pool. Currently, clusters with varied architectures are not supported. All pools must specify the same architecture. Valid values are <code>amd64</code> and <code>arm64</code>.<br><br>  <br><br><strong>Value:</strong> String</td>
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
  <td>Determines the instruction set architecture of the machines in the pool. Currently, clusters with varied architectures are not supported. All pools must specify the same architecture. Valid values are <code>amd64</code> and <code>arm64</code>.<br><br>  <br><br><strong>Value:</strong> String</td>
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
  <td>How to publish or expose the user-facing endpoints of your cluster, such as the Kubernetes API, OpenShift routes.<br><br><strong>Value:</strong>  <code>Internal</code> or <code>External</code>. To deploy a private cluster that cannot be accessed from the internet, set the <code>publish</code> parameter to <code>Internal</code>. The default value is <code>External</code>.   </td>
</tr>
<tr>
  <td>sshKey:</td>
  <td>The SSH key to authenticate access to your cluster machines.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>For production OpenShift Container Platform clusters on which you want to perform installation debugging or disaster recovery, specify an SSH key that your <code>ssh-agent</code> process uses.</p></div><br><br><strong>Value:</strong> For example, <code>sshKey: ssh-ed25519 AAAA..</code>.</td>
</tr>
</tbody>
</table>

:::note

If you are installing on Google Cloud into a shared virtual private cloud (VPC), `credentialsMode` must be set to `Passthrough` or `Manual`.

:::

:::warning

Setting this parameter to `Manual` enables alternatives to storing administrator-level secrets in the `kube-system` project, which require additional configuration steps. For more information, see "Alternatives to storing administrator-level secrets in the kube-system project".

:::

### Additional Google Cloud configuration parameters {#installation-configuration-parameters-additional-gcp_installation-config-parameters-gcp}

Additional Google Cloud configuration parameters are described in the following table:

**Additional Google Cloud parameters**

<table>
<thead>
<tr>
  <th>Parameter</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>controlPlane: platform: gcp: osImage: project:</td>
  <td>Optional. By default, the installation program downloads and installs the Red&#160;Hat Enterprise Linux CoreOS (RHCOS) image that is used to boot control plane machines. You can override the default behavior by specifying the location of a custom RHCOS image that the installation program is to use for control plane machines only. Control plane machines do not contribute to licensing costs when using the default image. But, if you apply a Google Cloud Marketplace image for a control plane machine, usage costs do apply.<br><br><strong>Value:</strong> String. The name of Google Cloud project where the image is located.</td>
</tr>
<tr>
  <td>controlPlane: platform: gcp: osImage: name:</td>
  <td>The name of the custom RHCOS image that the installation program is to use to boot control plane machines. If you use <code>controlPlane.platform.gcp.osImage.project</code>, this field is required.<br><br><strong>Value:</strong> String. The name of the RHCOS image.</td>
</tr>
<tr>
  <td>compute: platform: gcp: osImage: project:</td>
  <td>Optional. By default, the installation program downloads and installs the RHCOS image that is used to boot compute machines. You can override the default behavior by specifying the location of a custom RHCOS image that the installation program is to use for compute machines only.<br><br><strong>Value:</strong> String. The name of Google Cloud project where the image is located.</td>
</tr>
<tr>
  <td>compute: platform: gcp: osImage: name:</td>
  <td>The name of the custom RHCOS image that the installation program is to use to boot compute machines. If you use <code>compute.platform.gcp.osImage.project</code>, this field is required.<br><br><strong>Value:</strong> String. The name of the RHCOS image.</td>
</tr>
<tr>
  <td>compute: platform: gcp: serviceAccount:</td>
  <td>Specifies the email address of a Google Cloud service account to be used during installations. This service account is used to provision compute machines.<br><br><strong>Value:</strong> String. The email address of the service account.</td>
</tr>
<tr>
  <td>platform: gcp: firewallRulesManagement:</td>
  <td>Specifies the firewall management policy for the cluster. <code>Managed</code> indicates that the firewall rules will be created and destroyed by the cluster. <code>Unmanaged</code> indicates that the user should create and destroy the firewall rules. For shared VPC installation, if the credential you provided the installation program doesn't have firewall rules management permissions, the <code>firewallRulesManagement</code> parameter can be absent or set to <code>Unmanaged</code>. For non-shared VPC installation, if the credential you provided the installation program doesn't have firewall rules management permissions, the <code>firewallRulesManagement</code> parameter must be set to <code>Unmanaged</code>. If you manage your own firewall rules, you must pre-configure the VPC network and the firewall rules before the installation.<br><br><strong>Value:</strong> String. <code>Managed</code> or <code>Unmanaged</code>. The default value is <code>Managed</code>.</td>
</tr>
<tr>
  <td>platform: gcp: network:</td>
  <td>The name of the existing Virtual Private Cloud (VPC) where you want to deploy your cluster. If you want to deploy your cluster into a shared VPC, you must set <code>platform.gcp.networkProjectID</code> with the name of the Google Cloud project that contains the shared VPC.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>platform: gcp: networkProjectID:</td>
  <td>Optional. The name of the Google Cloud project that contains the shared VPC where you want to deploy your cluster.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>platform: gcp: projectID:</td>
  <td>The name of the Google Cloud project where the installation program installs the cluster.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>platform: gcp: dns: privateZone: name:</td>
  <td>The name of the private DNS zone. This parameter is only used during shared VPC installations. You can use a private DNS zone in a service project that is distinct from the projects specified by the <code>projectID</code> or <code>networkProjectID</code> parameters.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>platform: gcp: dns: privateZone: projectID:</td>
  <td>The ID of the project that contains the private zone from the <code>privateZone.name</code> parameter.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>platform: gcp: userProvisionedDNS:</td>
  <td>Enables user-provisioned DNS instead of the default cluster-provisioned DNS solution. If you use this feature, you must provide your own DNS solution that includes records for <code>api.&lt;cluster_name&gt;.&lt;base_domain&gt;.</code> and <code>*.apps.&lt;cluster_name&gt;.&lt;base_domain&gt;.</code>.<br><br><strong>Value:</strong> <code>Enabled</code> or <code>Disabled</code>. The default value is <code>Disabled</code>.</td>
</tr>
<tr>
  <td>platform: gcp: region:</td>
  <td>The name of the Google Cloud region that hosts your cluster.<br><br><strong>Value:</strong> Any valid region name, such as <code>us-central1</code>.</td>
</tr>
<tr>
  <td>platform: gcp: controlPlaneSubnet:</td>
  <td>The name of the existing subnet where you want to deploy your control plane machines.<br><br><strong>Value:</strong> The subnet name.</td>
</tr>
<tr>
  <td>platform: gcp: computeSubnet:</td>
  <td>The name of the existing subnet where you want to deploy your compute machines.<br><br><strong>Value:</strong> The subnet name.</td>
</tr>
<tr>
  <td>platform: gcp: defaultMachinePlatform: zones:</td>
  <td>The availability zones where the installation program creates machines.<br><br><strong>Value:</strong> A list of valid <a href="https://cloud.google.com/compute/docs/regions-zones#available">Google Cloud availability zones</a>, such as <code>us-central1-a</code>, in a <a href="https://yaml.org/spec/1.2/spec.html#sequence//">YAML sequence</a>.<div class="alert alert--warning"><strong>Important</strong><p>When running your cluster on Google Cloud 64-bit ARM infrastructures, ensure that you use a zone where Ampere Altra Arm CPU's are available. You can find which zones are compatible with 64-bit ARM processors in the "Google Cloud availability zones" link.</p></div></td>
</tr>
<tr>
  <td>platform: gcp: defaultMachinePlatform: osDisk: diskSizeGB:</td>
  <td>The size of the disk in gigabytes (GB).<br><br><strong>Value:</strong> Any size between 16 GB and 65536 GB.</td>
</tr>
<tr>
  <td>platform: gcp: defaultMachinePlatform: osDisk: diskType:</td>
  <td>The <a href="https://cloud.google.com/compute/docs/disks#disk-types">Google Cloud disk type</a>.<br><br><strong>Value:</strong> The default disk type for all machines. Valid values are <code>pd-balanced</code>, <code>pd-ssd</code>, <code>pd-standard</code>, or <code>hyperdisk-balanced</code>. The default value is <code>pd-ssd</code>. Control plane machines cannot use the <code>pd-standard</code> disk type, so if you specify <code>pd-standard</code> as the default machine platform disk type, you must specify a different disk type using the <code>controlPlane.platform.gcp.osDisk.diskType</code> parameter.</td>
</tr>
<tr>
  <td>platform: gcp: defaultMachinePlatform: osImage: project:</td>
  <td>Optional. By default, the installation program downloads and installs the RHCOS image that is used to boot control plane and compute machines. You can override the default behavior by specifying the location of a custom RHCOS image that the installation program is to use for both types of machines.<br><br><strong>Value:</strong> String. The name of Google Cloud project where the image is located.</td>
</tr>
<tr>
  <td>platform: gcp: defaultMachinePlatform: osImage: name:</td>
  <td>The name of the custom RHCOS image that the installation program is to use to boot control plane and compute machines. If you use <code>platform.gcp.defaultMachinePlatform.osImage.project</code>, this field is required.<br><br><strong>Value:</strong> String. The name of the RHCOS image.</td>
</tr>
<tr>
  <td>platform: gcp: defaultMachinePlatform: tags:</td>
  <td>Optional. Additional network tags to add to the control plane and compute machines.<br><br><strong>Value:</strong> One or more strings, for example <code>network-tag1</code>.</td>
</tr>
<tr>
  <td>platform: gcp: defaultMachinePlatform: type:</td>
  <td>The <a href="https://cloud.google.com/compute/docs/machine-types">Google Cloud machine type</a> for control plane and compute machines.<br><br><strong>Value:</strong> The Google Cloud machine type, for example <code>n1-standard-4</code>.</td>
</tr>
<tr>
  <td>platform: gcp: defaultMachinePlatform: osDisk: encryptionKey: kmsKey: name:</td>
  <td>The name of the customer managed encryption key to be used for machine disk encryption.<br><br><strong>Value:</strong> The encryption key name.</td>
</tr>
<tr>
  <td>platform: gcp: defaultMachinePlatform: osDisk: encryptionKey: kmsKey: keyRing:</td>
  <td>The name of the Key Management Service (KMS) key ring to which the KMS key belongs.<br><br><strong>Value:</strong> The KMS key ring name.</td>
</tr>
<tr>
  <td>platform: gcp: defaultMachinePlatform: osDisk: encryptionKey: kmsKey: location:</td>
  <td>The <a href="https://cloud.google.com/kms/docs/locations">Google Cloud location</a> in which the KMS key ring exists.<br><br><strong>Value:</strong> The Google Cloud location.</td>
</tr>
<tr>
  <td>platform: gcp: defaultMachinePlatform: osDisk: encryptionKey: kmsKey: projectID:</td>
  <td>The ID of the project in which the KMS key ring exists. This value defaults to the value of the <code>platform.gcp.projectID</code> parameter if it is not set.<br><br><strong>Value:</strong> The Google Cloud project ID.</td>
</tr>
<tr>
  <td>platform: gcp: defaultMachinePlatform: osDisk: encryptionKey: kmsKeyServiceAccount:</td>
  <td>The Google Cloud service account used for the encryption request for control plane and compute machines. If absent, the Compute Engine default service account is used. For more information about Google Cloud service accounts, see Google's documentation on <a href="https://cloud.google.com/compute/docs/access/service-accounts#compute_engine_service_account">service accounts</a>.<br><br><strong>Value:</strong> The Google Cloud service account email, for example <code>&lt;service_account_name&gt;@&lt;project_id&gt;.iam.gserviceaccount.com</code>.</td>
</tr>
<tr>
  <td>platform: gcp: defaultMachinePlatform: secureBoot:</td>
  <td>Whether to enable Shielded VM secure boot for all machines in the cluster. Shielded VMs have additional security protocols such as secure boot, firmware and integrity monitoring, and rootkit protection. For more information on Shielded VMs, see Google's documentation on <a href="https://cloud.google.com/shielded-vm">Shielded VMs</a>.<br><br><strong>Value:</strong> <code>Enabled</code> or <code>Disabled</code>. The default value is <code>Disabled</code>.</td>
</tr>
<tr>
  <td>platform: gcp: defaultMachinePlatform: confidentialCompute:</td>
  <td>Whether to use Confidential VMs for all machines in the cluster. Confidential VMs provide encryption for data during processing. For more information on Confidential computing, see Google's documentation about <a href="https://cloud.google.com/confidential-computing">Confidential Computing</a>.<br><br>Supported values are:<br><br><ul><li><code>Enabled</code>, which automatically selects a Confidential Computing platform</li></ul><div class="alert alert--warning"><strong>Important</strong><p>The <code>Enabled</code> value selects Confidential Computing with AMD Secure Encrypted Virtualization (AMD SEV), which is deprecated.</p></div><ul><li><code>Disabled</code>, which disables Confidential Computing</li><li><code>AMDEncryptedVirtualizationNestedPaging</code>, which enables Confidential Computing with AMD Secure Encrypted Virtualization Secure Nested Paging (AMD SEV-SNP)</li><li><code>AMDEncryptedVirtualization</code>, which enables Confidential Computing with AMD Secure Encrypted Virtualization (AMD SEV)</li></ul><div class="alert alert--warning"><strong>Important</strong><p>The use of Confidential Computing with AMD Secure Encrypted Virtualization (AMD SEV) has been deprecated and will be removed in a future release.</p></div><ul><li><code>IntelTrustedDomainExtensions</code>, which enables Confidential Computing with Intel Trusted Domain Extensions (Intel TDX)</li></ul>If you specify any value other than <code>Disabled</code>, you must set <code>platform.gcp.defaultMachinePlatform.onHostMaintenance</code> to <code>Terminate</code>, and you must specify a region and machine type that support Confidential Computing. For more information, see Google's documentation about <a href="https://cloud.google.com/confidential-computing/confidential-vm/docs/supported-configurations#machine-type-cpu-zone">Supported configurations</a>.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>platform: gcp: defaultMachinePlatform: onHostMaintenance:</td>
  <td>Specifies the behavior of all VMs during a host maintenance event, such as a software or hardware update. For Confidential VMs, this parameter must be set to <code>Terminate</code>. Confidential VMs do not support live VM migration.<br><br><strong>Value:</strong> <code>Terminate</code> or <code>Migrate</code>. The default value is <code>Migrate</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: gcp: osDisk: encryptionKey: kmsKey: name:</td>
  <td>The name of the customer managed encryption key to be used for control plane machine disk encryption.<br><br><strong>Value:</strong> The encryption key name.</td>
</tr>
<tr>
  <td>controlPlane: platform: gcp: osDisk: encryptionKey: kmsKey: keyRing:</td>
  <td>For control plane machines, the name of the KMS key ring to which the KMS key belongs.<br><br><strong>Value:</strong> The KMS key ring name.</td>
</tr>
<tr>
  <td>controlPlane: platform: gcp: osDisk: encryptionKey: kmsKey: location:</td>
  <td>For control plane machines, the Google Cloud location in which the key ring exists. For more information about KMS locations, see Google's documentation on <a href="https://cloud.google.com/kms/docs/locations">Cloud KMS locations</a>.<br><br><strong>Value:</strong> The Google Cloud location for the key ring.</td>
</tr>
<tr>
  <td>controlPlane: platform: gcp: osDisk: encryptionKey: kmsKey: projectID:</td>
  <td>For control plane machines, the ID of the project in which the KMS key ring exists. This value defaults to the VM project ID if not set.<br><br><strong>Value:</strong> The Google Cloud project ID.</td>
</tr>
<tr>
  <td>controlPlane: platform: gcp: osDisk: encryptionKey: kmsKeyServiceAccount:</td>
  <td>The Google Cloud service account used for the encryption request for control plane machines. If absent, the Compute Engine default service account is used. For more information about Google Cloud service accounts, see Google's documentation on <a href="https://cloud.google.com/compute/docs/access/service-accounts#compute_engine_service_account">service accounts</a>.<br><br><strong>Value:</strong> The Google Cloud service account email, for example <code>&lt;service_account_name&gt;@&lt;project_id&gt;.iam.gserviceaccount.com</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: gcp: osDisk: diskSizeGB:</td>
  <td>The size of the disk in gigabytes (GB). This value applies to control plane machines.<br><br><strong>Value:</strong> Any integer between 16 and 65536.</td>
</tr>
<tr>
  <td>controlPlane: platform: gcp: osDisk: diskType:</td>
  <td>The <a href="https://cloud.google.com/compute/docs/disks#disk-types">Google Cloud disk type</a> for control plane machines.<br><br><strong>Value:</strong> Valid values are <code>pd-balanced</code>, <code>pd-ssd</code>, or <code>hyperdisk-balanced</code>. The default value is <code>pd-ssd</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: gcp: tags:</td>
  <td>Optional. Additional network tags to add to the control plane machines. If set, this parameter overrides the <code>platform.gcp.defaultMachinePlatform.tags</code> parameter for control plane machines.<br><br><strong>Value:</strong> One or more strings, for example <code>control-plane-tag1</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: gcp: type:</td>
  <td>The <a href="https://cloud.google.com/compute/docs/machine-types">Google Cloud machine type</a> for control plane machines. If set, this parameter overrides the <code>platform.gcp.defaultMachinePlatform.type</code> parameter.<br><br><strong>Value:</strong> The Google Cloud machine type, for example <code>n1-standard-4</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: gcp: zones:</td>
  <td>The availability zones where the installation program creates control plane machines.<br><br><strong>Value:</strong> A list of valid <a href="https://cloud.google.com/compute/docs/regions-zones#available">Google Cloud availability zones</a>, such as <code>us-central1-a</code>, in a <a href="https://yaml.org/spec/1.2/spec.html#sequence//">YAML sequence</a>.<div class="alert alert--warning"><strong>Important</strong><p>When running your cluster on Google Cloud 64-bit ARM infrastructures, ensure that you use a zone where Ampere Altra Arm CPU's are available. You can find which zones are compatible with 64-bit ARM processors in the "Google Cloud availability zones" link.</p></div></td>
</tr>
<tr>
  <td>controlPlane: platform: gcp: secureBoot:</td>
  <td>Whether to enable Shielded VM secure boot for control plane machines. Shielded VMs have additional security protocols such as secure boot, firmware and integrity monitoring, and rootkit protection. For more information on Shielded VMs, see Google's documentation on <a href="https://cloud.google.com/shielded-vm">Shielded VMs</a>.<br><br><strong>Value:</strong> <code>Enabled</code> or <code>Disabled</code>. The default value is <code>Disabled</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: gcp: confidentialCompute:</td>
  <td>Whether to use Confidential VMs for control plane machines. Confidential VMs provide encryption for data during processing. For more information on Confidential computing, see Google's documentation about <a href="https://cloud.google.com/confidential-computing">Confidential Computing</a>.<br><br>Supported values are:<br><br><ul><li><code>Enabled</code>, which automatically selects a Confidential Computing platform</li></ul><div class="alert alert--warning"><strong>Important</strong><p>The <code>Enabled</code> value selects Confidential Computing with AMD Secure Encrypted Virtualization (AMD SEV), which is deprecated.</p></div><ul><li><code>Disabled</code>, which disables Confidential Computing</li><li><code>AMDEncryptedVirtualizationNestedPaging</code>, which enables Confidential Computing with AMD Secure Encrypted Virtualization Secure Nested Paging (AMD SEV-SNP)</li><li><code>AMDEncryptedVirtualization</code>, which enables Confidential Computing with AMD Secure Encrypted Virtualization (AMD SEV)</li></ul><div class="alert alert--warning"><strong>Important</strong><p>The use of Confidential Computing with AMD Secure Encrypted Virtualization (AMD SEV) has been deprecated and will be removed in a future release.</p></div><ul><li><code>IntelTrustedDomainExtensions</code>, which enables Confidential Computing with Intel Trusted Domain Extensions (Intel TDX)</li></ul>If you specify any value other than <code>Disabled</code>, you must set <code>controlPlane.platform.gcp.defaultMachinePlatform.onHostMaintenance</code> to <code>Terminate</code>.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>controlPlane: platform: gcp: onHostMaintenance:</td>
  <td>Specifies the behavior of control plane VMs during a host maintenance event, such as a software or hardware update. For Confidential VMs, this parameter must be set to <code>Terminate</code>. Confidential VMs do not support live VM migration.<br><br><strong>Value:</strong> <code>Terminate</code> or <code>Migrate</code>. The default value is <code>Migrate</code>.</td>
</tr>
<tr>
  <td>controlPlane: platform: gcp: serviceAccount:</td>
  <td>Specifies the email address of a Google Cloud service account to be used during installations. This service account is used to provision control plane machines.<div class="alert alert--warning"><strong>Important</strong><p>In the case of shared VPC installations, when the service account is not provided, the installation program service account must have the <code>resourcemanager.projects.getIamPolicy</code> and <code>resourcemanager.projects.setIamPolicy</code> permissions in the host project.</p></div><br><br><strong>Value:</strong> String. The email address of the service account.</td>
</tr>
<tr>
  <td>compute: platform: gcp: osDisk: encryptionKey: kmsKey: name:</td>
  <td>The name of the customer managed encryption key to be used for compute machine disk encryption.<br><br><strong>Value:</strong> The encryption key name.</td>
</tr>
<tr>
  <td>compute: platform: gcp: osDisk: encryptionKey: kmsKey: keyRing:</td>
  <td>For compute machines, the name of the KMS key ring to which the KMS key belongs.<br><br><strong>Value:</strong> The KMS key ring name.</td>
</tr>
<tr>
  <td>compute: platform: gcp: osDisk: encryptionKey: kmsKey: location:</td>
  <td>For compute machines, the Google Cloud location in which the key ring exists. For more information about KMS locations, see Google's documentation on <a href="https://cloud.google.com/kms/docs/locations">Cloud KMS locations</a>.<br><br><strong>Value:</strong> The Google Cloud location for the key ring.</td>
</tr>
<tr>
  <td>compute: platform: gcp: osDisk: encryptionKey: kmsKey: projectID:</td>
  <td>For compute machines, the ID of the project in which the KMS key ring exists. This value defaults to the VM project ID if not set.<br><br><strong>Value:</strong> The Google Cloud project ID.</td>
</tr>
<tr>
  <td>compute: platform: gcp: osDisk: encryptionKey: kmsKeyServiceAccount:</td>
  <td>The Google Cloud service account used for the encryption request for compute machines. If this value is not set, the Compute Engine default service account is used. For more information about Google Cloud service accounts, see Google's documentation on <a href="https://cloud.google.com/compute/docs/access/service-accounts#compute_engine_service_account">service accounts</a>.<br><br><strong>Value:</strong> The Google Cloud service account email, for example <code>&lt;service_account_name&gt;@&lt;project_id&gt;.iam.gserviceaccount.com</code>.</td>
</tr>
<tr>
  <td>compute: platform: gcp: osDisk: diskSizeGB:</td>
  <td>The size of the disk in gigabytes (GB). This value applies to compute machines.<br><br><strong>Value:</strong> Any integer between 16 and 65536.</td>
</tr>
<tr>
  <td>compute: platform: gcp: osDisk: diskType:</td>
  <td>The <a href="https://cloud.google.com/compute/docs/disks#disk-types">Google Cloud disk type</a> for compute machines.<br><br><strong>Value:</strong> Valid values are <code>pd-balanced</code>, <code>pd-ssd</code>, <code>pd-standard</code>, or <code>hyperdisk-balanced</code>. The default value is <code>pd-ssd</code>.</td>
</tr>
<tr>
  <td>compute: platform: gcp: tags:</td>
  <td>Optional. Additional network tags to add to the compute machines. If set, this parameter overrides the <code>platform.gcp.defaultMachinePlatform.tags</code> parameter for compute machines.<br><br><strong>Value:</strong> One or more strings, for example <code>compute-network-tag1</code>.</td>
</tr>
<tr>
  <td>compute: platform: gcp: type:</td>
  <td>The <a href="https://cloud.google.com/compute/docs/machine-types">Google Cloud machine type</a> for compute machines. If set, this parameter overrides the <code>platform.gcp.defaultMachinePlatform.type</code> parameter.<br><br><strong>Value:</strong> The Google Cloud machine type, for example <code>n1-standard-4</code>.</td>
</tr>
<tr>
  <td>compute: platform: gcp: zones:</td>
  <td>The availability zones where the installation program creates compute machines.<br><br><strong>Value:</strong> A list of valid <a href="https://cloud.google.com/compute/docs/regions-zones#available">Google Cloud availability zones</a>, such as <code>us-central1-a</code>, in a <a href="https://yaml.org/spec/1.2/spec.html#sequence//">YAML sequence</a>.<div class="alert alert--warning"><strong>Important</strong><p>When running your cluster on Google Cloud 64-bit ARM infrastructures, ensure that you use a zone where Ampere Altra Arm CPU's are available. You can find which zones are compatible with 64-bit ARM processors in the "Google Cloud availability zones" link.</p></div></td>
</tr>
<tr>
  <td>compute: platform: gcp: secureBoot:</td>
  <td>Whether to enable Shielded VM secure boot for compute machines. Shielded VMs have additional security protocols such as secure boot, firmware and integrity monitoring, and rootkit protection. For more information on Shielded VMs, see Google's documentation on <a href="https://cloud.google.com/shielded-vm">Shielded VMs</a>.<br><br><strong>Value:</strong> <code>Enabled</code> or <code>Disabled</code>. The default value is <code>Disabled</code>.</td>
</tr>
<tr>
  <td>compute: platform: gcp: confidentialCompute:</td>
  <td>Whether to use Confidential VMs for compute machines. Confidential VMs provide encryption for data during processing. For more information on Confidential computing, see Google's documentation on <a href="https://cloud.google.com/confidential-computing">Confidential computing</a>.<br><br>Supported values are:<br><br><ul><li><code>Enabled</code>, which automatically selects a Confidential Computing platform</li></ul><div class="alert alert--warning"><strong>Important</strong><p>The <code>Enabled</code> value selects Confidential Computing with AMD Secure Encrypted Virtualization (AMD SEV), which is deprecated.</p></div><ul><li><code>Disabled</code>, which disables Confidential Computing</li><li><code>AMDEncryptedVirtualizationNestedPaging</code>, which enables Confidential Computing with AMD Secure Encrypted Virtualization Secure Nested Paging (AMD SEV-SNP)</li><li><code>AMDEncryptedVirtualization</code>, which enables Confidential Computing with AMD Secure Encrypted Virtualization (AMD SEV)</li></ul><div class="alert alert--warning"><strong>Important</strong><p>The use of Confidential Computing with AMD Secure Encrypted Virtualization (AMD SEV) has been deprecated and will be removed in a future release.</p></div><ul><li><code>IntelTrustedDomainExtensions</code>, which enables Confidential Computing with Intel Trusted Domain Extensions (Intel TDX)</li></ul>If you specify any value other than <code>Disabled</code>, you must set <code>compute.platform.gcp.onHostMaintenance</code> to <code>Terminate</code>.<br><br><strong>Value:</strong> String.</td>
</tr>
<tr>
  <td>compute: platform: gcp: onHostMaintenance:</td>
  <td>Specifies the behavior of compute VMs during a host maintenance event, such as a software or hardware update. For Confidential VMs, this parameter must be set to <code>Terminate</code>. Confidential VMs do not support live VM migration.<br><br><strong>Value:</strong> <code>Terminate</code> or <code>Migrate</code>. The default value is <code>Migrate</code>.</td>
</tr>
</tbody>
</table>
