---
title: Installing a cluster on Azure Stack Hub with network customizations
sidebar_position: 3
---

# Installing a cluster on Azure Stack Hub with network customizations {#installing-azure-stack-hub-network-customizations}

<a id="installing-azure-stack-hub-network-customizations"></a>

In OpenShift Container Platform version 4.22, you can install a cluster with a customized network configuration on infrastructure that the installation program provisions on Azure Stack Hub. By customizing your network configuration, your cluster can coexist with existing IP address allocations in your environment and integrate with existing MTU and VXLAN configurations.

:::note

While you can select `azure` when using the installation program to deploy a cluster using installer-provisioned infrastructure, this option is only supported for the Azure Public Cloud.

:::

## Prerequisites {#prerequisites_installing-azure-stack-hub-network-customizations}

- You reviewed details about the [OpenShift Container Platform installation and update](/docs/architecture/architecture-installation#architecture-installation) processes.
- You read the documentation on [selecting a cluster installation method and preparing it for users](/docs/installing/overview/installing-preparing#installing-preparing).
- You have installed Azure Stack Hub version 2008 or later.
- You [configured an Azure Stack Hub account](/docs/installing/installing_azure_stack_hub/installing-azure-stack-hub-account#installing-azure-stack-hub-account) to host the cluster.
- If you use a firewall, you [configured it to allow the sites](/docs/installing/install_config/configuring-firewall#configuring-firewall-module_configuring-firewall) that your cluster requires access to.
- You verified that you have approximately 16 GB of local disk space. Installing the cluster requires that you download the RHCOS virtual hard drive (VHD) cluster image and upload it to your Azure Stack Hub environment so that it is accessible during deployment. Decompressing the VHD files requires this amount of local disk space.

## Uploading the RHCOS cluster image {#_uploading_the_op_system_cluster_image}

To make the RHCOS cluster image accessible during deployment, you can download and upload the image to your Azure Stack Hub environment.

You must download the RHCOS virtual hard disk (VHD) cluster image and upload it to your Azure Stack Hub environment so that it is accessible during deployment.

**Prerequisites**

- Generate the Ignition config files for your cluster.

**Procedure**

1. Obtain the RHCOS VHD cluster image:
   1. Export the URL of the RHCOS VHD to an environment variable.
      ```terminal
      $ export COMPRESSED_VHD_URL=$(openshift-install coreos print-stream-json | jq -r '.architectures.x86_64.artifacts.azurestack.formats."vhd.gz".disk.location')
      ```
   2. Download the compressed RHCOS VHD file locally.
      ```terminal
      $ curl -O -L ${COMPRESSED_VHD_URL}
      ```
2. Decompress the VHD file.
   :::note

   The decompressed VHD file is approximately 16 GB, so be sure that your host system has 16 GB of free space available. The VHD file can be deleted once you have uploaded it.

   :::
3. Upload the local VHD to the Azure Stack Hub environment, making sure that the blob is publicly available. For example, you can upload the VHD to a blob using the `az` cli or the web portal.

## Manually creating the installation configuration file {#installation-initializing-manual_installing-azure-stack-hub-network-customizations}

Installing the cluster requires that you manually create the installation configuration file.

**Prerequisites**

- You have an SSH public key on your local machine for use with the installation program. You can use the key for SSH authentication onto your cluster nodes for debugging and disaster recovery.
- You have obtained the OpenShift Container Platform installation program and the pull secret for your cluster.

**Procedure**

1. Create an installation directory to store your required installation assets in:
   ```terminal
   $ mkdir <installation_directory>
   ```

   :::warning

   You must create a directory. Some installation assets, such as bootstrap X.509 certificates have short expiration intervals, so you must not reuse an installation directory. If you want to reuse individual files from another cluster installation, you can copy them into your directory. However, the file names for the installation assets might change between releases. Use caution when copying installation files from an earlier OpenShift Container Platform version.

   :::
2. Customize the provided sample `install-config.yaml` file template and save the file in the `<installation_directory>`.
   :::note

   You must name this configuration file `install-config.yaml`.

   :::

   Make the following modifications:

   1. Specify the required installation parameters.
   2. Update the `platform.azure` section to specify the parameters that are specific to Azure Stack Hub.
   3. Optional: Update one or more of the default configuration parameters to customize the installation.
      For more information about the parameters, see "Installation configuration parameters".
3. Back up the `install-config.yaml` file so that you can use it to install many clusters.
   :::warning

   Back up the `install-config.yaml` file now, because the installation process consumes the file in the next step.

   :::

**Additional resources**

- [Installation configuration parameters for Azure Stack Hub](/docs/installing/installing_azure_stack_hub/installation-config-parameters-ash#installation-config-parameters-ash)

### Sample customized install-config.yaml file for Azure Stack Hub {#installation-azure-stack-hub-config-yaml_installing-azure-stack-hub-network-customizations}

You can customize the `install-config.yaml` file to specify more details about your OpenShift Container Platform cluster’s platform or modify the values of the required parameters.

:::warning

This sample YAML file is provided for reference only. Use it as a resource to enter parameter values into the installation configuration file that you created manually.

:::

```yaml
apiVersion: v1
baseDomain: example.com
credentialsMode: Manual
controlPlane:
  name: master
  platform:
    azure:
      osDisk:
        diskSizeGB: 1024
        diskType: premium_LRS
  replicas: 3
compute:
- name: worker
  platform:
    azure:
      osDisk:
        diskSizeGB: 512
        diskType: premium_LRS
  replicas: 3
metadata:
  name: test-cluster
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  machineNetwork:
  - cidr: 10.0.0.0/16
  networkType: OVNKubernetes
  serviceNetwork:
  - 172.30.0.0/16
platform:
  azure:
    armEndpoint: azurestack_arm_endpoint
    baseDomainResourceGroupName: resource_group
    region: azure_stack_local_region
    resourceGroupName: existing_resource_group
    outboundType: Loadbalancer
    cloudName: AzureStackCloud
    clusterOSimage: https://vhdsa.blob.example.example.com/vhd/rhcos-410.84.202112040202-0-azurestack.x86_64.vhd
pullSecret: '{"auths": ...}'
fips: false
sshKey: ssh-ed25519 AAAA...
additionalTrustBundle: |
    -----BEGIN CERTIFICATE-----
    <MY_TRUSTED_CA_CERT>
    -----END CERTIFICATE-----
```

where:

`baseDomain`
: Specifies the base domain for your cluster. This parameter is required.

`controlPlane`
: Specifies the configuration for the machines that form the control plane. The `controlPlane` section is a single mapping. To meet the requirements of the different data structures, the first line of the `compute` section must begin with a hyphen, `-`, and the first line of the `controlPlane` section must not. Although both sections currently define a single machine pool, it is possible that future versions of OpenShift Container Platform will support defining multiple compute pools during installation. Only one control plane pool is used. If you do not provide these parameter, the installation program provides the default value.

`compute`
: Specifies the configuration for the machines that form the compute plane. The `compute` section is a sequence of mappings. To meet the requirements of the different data structures, the first line of the `compute` section must begin with a hyphen, `-`, and the first line of the `controlPlane` section must not. Although both sections currently define a single machine pool, it is possible that future versions of OpenShift Container Platform will support defining multiple compute pools during installation. If you do not provide these parameter, the installation program provides the default value.

`controlPlane.platform.azure.osDisk.diskSizeGB`
: Specifies the size of the disk to use in GB. Minimum recommendation for control plane nodes is 1024 GB.

`compute.platform.azure.osDisk.diskSizeGB`
: Specifies the size of the disk to use in GB.

`metadata.name`
: Specifies the name of the cluster. This parameter is required.

`networking.networkType`
: Specifies the cluster network plugin to install. The default value `OVNKubernetes` is the only supported value.

`platform.azure.armEndpoint`
: Specifies the Azure Resource Manager endpoint that your Azure Stack Hub operator provides. This parameter is required.

`platform.azure.baseDomainResourceGroupName`
: Specifies the name of the resource group that contains the DNS zone for your base domain. This parameter is required.

`platform.azure.region`
: Specifies the name of your Azure Stack Hub local region. This parameter is required.

`platform.azure.resourceGroupName`
: Specifies the name of an existing resource group to install your cluster to. If undefined, a new resource group is created for the cluster.

`platform.azure.cloudName`
: Specifies the `platform.azure.cloudName` parameter. This parameter is required.

`platform.azure.clusterOSimage`
: Specifies the URL of a storage blob in the Azure Stack environment that contains an RHCOS VHD.

`pullSecret`
: Specifies the pull secret required to authenticate your cluster. This parameter is required.

`fips`
: Specifies whether to enable or disable FIPS mode. By default, FIPS mode is not enabled. If FIPS mode is enabled, the Red Hat Enterprise Linux CoreOS (RHCOS) machines that OpenShift Container Platform runs on bypass the default Kubernetes cryptography suite and use the cryptography modules that are provided with RHCOS instead.

  :::warning

  When running Red Hat Enterprise Linux (RHEL) or Red Hat Enterprise Linux CoreOS (RHCOS) booted in FIPS mode, OpenShift Container Platform core components use the RHEL cryptographic libraries that have been submitted to NIST for FIPS 140-2/140-3 Validation on only the x86_64, ppc64le, and s390x architectures.

  :::

`sshKey`
: Specifies the `sshKey` value that you use to access the machines in your cluster. This parameter is optional.

  :::note

  For production OpenShift Container Platform clusters on which you want to perform installation debugging or disaster recovery, specify an SSH key that your `ssh-agent` process uses.

  :::

`additionalTrustBundle`
: Specifies the certificate trust bundle. If the Azure Stack Hub environment is using an internal Certificate Authority (CA), adding the CA certificate is required.

## Manually manage cloud credentials {#_manually_manage_cloud_credentials}

The Cloud Credential Operator (CCO) only supports your cloud provider in manual mode. As a result, you must specify the identity and access management (IAM) secrets for your cloud provider.

**Procedure**

1. If you have not previously created installation manifest files, do so by running the following command:
   ```terminal
   $ openshift-install create manifests --dir <installation_directory>
   ```

   where `<installation_directory>` is the directory in which the installation program creates files.
2. Set a `$RELEASE_IMAGE` variable with the release image from your installation file by running the following command:
   ```terminal
   $ RELEASE_IMAGE=$(./openshift-install version | awk '/release image/ {print $3}')
   ```
3. Extract the list of `CredentialsRequest` custom resources (CRs) from the OpenShift Container Platform release image by running the following command:
   ```terminal
   $ oc adm release extract \
     --from=$RELEASE_IMAGE \
     --credentials-requests \
     --included \
     --install-config=<path_to_directory_with_installation_configuration>/install-config.yaml \
     --to=<path_to_directory_for_credentials_requests>
   ```

   where:

   <dl>
   <dt><code>--included</code></dt>
   <dd>Specifies only the manifests that your specific cluster configuration requires.</dd>
   <dt><code>&lt;path_to_directory_with_installation_configuration&gt;</code></dt>
   <dd>Specifies the location of the <code>install-config.yaml</code> file.</dd>
   <dt><code>&lt;path_to_directory_for_credentials_requests&gt;</code></dt>
   <dd>Specifies the path to the directory where you want to store the <code>CredentialsRequest</code> objects. If the specified directory does not exist, this command creates it. This command creates a YAML file for each <code>CredentialsRequest</code> object.</dd>
   </dl>

   ```yaml title="Sample CredentialsRequest object"
   apiVersion: cloudcredential.openshift.io/v1
   kind: CredentialsRequest
   metadata:
     name: <component_credentials_request>
     namespace: openshift-cloud-credential-operator
     ...
   spec:
     providerSpec:
       apiVersion: cloudcredential.openshift.io/v1
       kind: AzureProviderSpec
       roleBindings:
       - role: Contributor
     ...
   ```
4. Create YAML files for secrets in the `openshift-install` manifests directory that you generated previously. The secrets must be stored using the namespace and secret name defined in the `spec.secretRef` for each `CredentialsRequest` object.
   ```yaml title="Sample CredentialsRequest object with secrets"
   apiVersion: cloudcredential.openshift.io/v1
   kind: CredentialsRequest
   metadata:
     name: <component_credentials_request>
     namespace: openshift-cloud-credential-operator
     ...
   spec:
     providerSpec:
       apiVersion: cloudcredential.openshift.io/v1
       kind: AzureProviderSpec
       roleBindings:
       - role: Contributor
         ...
     secretRef:
       name: <component_secret>
       namespace: <component_namespace>
     ...
   ```

   ```yaml title="Sample Secret object"
   apiVersion: v1
   kind: Secret
   metadata:
     name: <component_secret>
     namespace: <component_namespace>
   data:
     azure_subscription_id: <base64_encoded_azure_subscription_id>
     azure_client_id: <base64_encoded_azure_client_id>
     azure_client_secret: <base64_encoded_azure_client_secret>
     azure_tenant_id: <base64_encoded_azure_tenant_id>
     azure_resource_prefix: <base64_encoded_azure_resource_prefix>
     azure_resourcegroup: <base64_encoded_azure_resourcegroup>
     azure_region: <base64_encoded_azure_region>
   ```

   :::warning

   Before upgrading a cluster that uses manually maintained credentials, you must ensure that the CCO is in an upgradeable state.

   :::

**Additional resources**

- [Updating a cluster using the web console](/docs/updating/updating_a_cluster/updating-cluster-web-console#manually-maintained-credentials-upgrade_updating-cluster-web-console)
- [Updating a cluster using the CLI](/docs/updating/updating_a_cluster/updating-cluster-cli#manually-maintained-credentials-upgrade_updating-cluster-cli)

## Configuring the cluster to use an internal CA {#internal-certificate-authority_installing-azure-stack-hub-network-customizations}

If the Azure Stack Hub environment is using an internal Certificate Authority (CA), update the `cluster-proxy-01-config.yaml` file to configure the cluster to use the internal CA.

**Prerequisites**

- Create the `install-config.yaml` file and specify the certificate trust bundle in `.pem` format.
- Create the cluster manifests.

**Procedure**

1. From the directory in which the installation program creates files, go to the `manifests` directory.
2. Add `user-ca-bundle` to  the `spec.trustedCA.name` field.
   ```yaml title="Example cluster-proxy-01-config.yaml file"
   apiVersion: config.openshift.io/v1
   kind: Proxy
   metadata:
     creationTimestamp: null
     name: cluster
   spec:
     trustedCA:
       name: user-ca-bundle
   status: {}
   ```
3. Optional: Back up the `manifests/ cluster-proxy-01-config.yaml` file. The installation program consumes the `manifests/` directory when you deploy the cluster.

## Network configuration phases {#nw-network-config_installing-azure-stack-hub-network-customizations}

There are two phases prior to OpenShift Container Platform installation where you can customize the network configuration. Customize settings in the `install-config.yaml` file and in the Cluster Network Operator manifest across two configuration phases.

Phase 1
: You can customize the following network-related fields in the `install-config.yaml` file before you create the manifest files:

  - `networking.networkType`
  - `networking.clusterNetwork`
  - `networking.serviceNetwork`
  - `networking.machineNetwork`
  - `nodeNetworking`

  For more information, see "Installation configuration parameters".

  :::note

  Set the `networking.machineNetwork` to match the Classless Inter-Domain Routing (CIDR) where the preferred subnet is located.

  :::

  :::warning

  The CIDR range `172.17.0.0/16` is reserved by `libVirt`. You cannot use any other CIDR range that overlaps with the `172.17.0.0/16` CIDR range for networks in your cluster.

  :::

Phase 2
: After creating the manifest files by running `openshift-install create manifests`, you can define a customized Cluster Network Operator manifest with only the fields you want to modify. You can use the manifest to specify an advanced network configuration.

During phase 2, you cannot override the values that you specified in phase 1 in the `install-config.yaml` file. However, you can customize the network plugin during phase 2.

## Specifying advanced network configuration {#modifying-nwoperator-config-startup_installing-azure-stack-hub-network-customizations}

To integrate your OpenShift Container Platform cluster with your existing network environment, you can specify advanced network configuration in a manifest before you install the cluster. Advanced network configuration can be configured only during cluster installation.

:::warning

Customizing your network configuration by modifying the OpenShift Container Platform manifest files created by the installation program is not supported. Applying a manifest file that you create, as in the following procedure, is supported.

:::

**Prerequisites**

- You have created the `install-config.yaml` file and completed any modifications to it.

**Procedure**

1. Change to the directory that contains the installation program and create the manifests:
   ```terminal
   $ ./openshift-install create manifests --dir <installation_directory>
   ```

   The `<installation_directory>` specifies the name of the directory that contains the `install-config.yaml` file for your cluster.
2. Create a stub manifest file for the advanced network configuration that is named `cluster-network-03-config.yml` in the `<installation_directory>/manifests/` directory:
   ```yaml
   apiVersion: operator.openshift.io/v1
   kind: Network
   metadata:
     name: cluster
   spec:
   ```
3. Specify the advanced network configuration for your cluster in the `cluster-network-03-config.yml` file, such as in the following example:
   ```yaml title="Enable IPsec for the OVN-Kubernetes network provider"
   apiVersion: operator.openshift.io/v1
   kind: Network
   metadata:
     name: cluster
   spec:
     defaultNetwork:
       ovnKubernetesConfig:
         ipsecConfig:
           mode: Full
   ```
4. Optional: Back up the `manifests/cluster-network-03-config.yml` file. The installation program consumes the `manifests/` directory when you create the Ignition config files.
5. Remove the Kubernetes manifest files that define the control plane machines and compute `MachineSets`:
   ```terminal
   $ rm -f openshift/99_openshift-cluster-api_master-machines-*.yaml openshift/99_openshift-cluster-api_worker-machineset-*.yaml
   ```

   Because you create and manage these resources yourself, you do not have to initialize them.

   - You can preserve the `MachineSet` files to create compute machines by using the machine API, but you must update references to them to match your environment.

## Cluster Network Operator configuration {#nw-operator-cr_installing-azure-stack-hub-network-customizations}

To manage cluster networking, configure the Cluster Network Operator (CNO) `Network` custom resource (CR) named `cluster` so the cluster uses the correct IP ranges and network plugin settings for reliable pod and service connectivity. Some settings and fields are inherited at the time of install or by the `default.Network.type` plugin, OVN-Kubernetes.

The CNO configuration inherits the following fields during cluster installation from the `Network` API in the `Network.config.openshift.io` API group:

`clusterNetwork`
: IP address pools from which pod IP addresses are allocated.

`serviceNetwork`
: IP address pool for services.

`defaultNetwork.type`
: Cluster network plugin. `OVNKubernetes` is the only supported plugin during installation.

You can specify the cluster network plugin configuration for your cluster by setting the fields for the `defaultNetwork` object in the CNO object named `cluster`.

### Cluster Network Operator configuration object {#nw-operator-cr-cno-object_installing-azure-stack-hub-network-customizations}

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
  <td>A block of IP addresses for services. If you use dual-stack networking, specify IPv4 and IPv6 address families. For example:<br><br><pre><code>spec:&#10;  serviceNetwork:&#10;  - 172.30.0.0/14&#10;  - fd02::/112</code></pre><br><br>If you install a cluster on AWS with dual-stack networking, the order of addresses must match the dual-stack configuration you selected. For example, if you specified the <code>DualStackIPv4Primary</code>, list the IPv4 address first.<br><br>   You can customize this field only in the <code>install-config.yaml</code> file before you create the manifests. The value is read-only in the manifest file. </td>
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

### defaultNetwork object configuration {#nw-operator-cr-defaultnetwork_installing-azure-stack-hub-network-customizations}

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

### Configuration for the OVN-Kubernetes network plugin {#nw-operator-configuration-parameters-for-ovn-sdn_installing-azure-stack-hub-network-customizations}

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
  <td> The maximum transmission unit (MTU) for the Geneve (Generic Network Virtualization Encapsulation) overlay network. This is detected automatically based on the MTU of the primary network interface. You do not normally need to override the detected MTU.<br><br>If the auto-detected value is not what you expect it to be, confirm that the MTU on the primary network interface on your nodes is correct. You cannot use this option to change the MTU value of the primary network interface on the nodes.<br><br>If your cluster requires different MTU values for different nodes, you must set this value to <code>100</code> less than the lowest MTU value in your cluster. For example, if some nodes in your cluster have an MTU of <code>9001</code>, and some have an MTU of <code>1500</code>, you must set this value to <code>1400</code>.  </td>
</tr>
<tr>
  <td><code>genevePort</code></td>
  <td><code>integer</code></td>
  <td> The port to use for all Geneve packets. The default value is <code>6081</code>. This value cannot be changed after cluster installation.  </td>
</tr>
<tr>
  <td><code>ipsecConfig</code></td>
  <td><code>object</code></td>
  <td> Specify a configuration object for customizing the IPsec configuration.  </td>
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
  <td>One of the following additional audit log targets:<br><br><dl><dt><code>libc</code></dt><dd>The libc <code>syslog()</code> function of the journald process on the host.</dd><dt><code>udp:&lt;host&gt;:&lt;port&gt;</code></dt><dd>A syslog server. Replace <code>&lt;host&gt;:&lt;port&gt;</code> with the host and port of the syslog server.</dd><dt><code>unix:&lt;file&gt;</code></dt><dd>A Unix Domain Socket file specified by <code>&lt;file&gt;</code>.</dd><dt><code>null</code></dt><dd>Do not send the audit logs to any additional target.</dd></dl></td>
</tr>
<tr>
  <td><code>syslogFacility</code></td>
  <td>string</td>
  <td>The syslog facility, such as <code>kern</code>, as defined by RFC5424. The default value is <code>local0</code>.</td>
</tr>
</tbody>
</table>

<a name="gatewayConfig-object_installing-azure-stack-hub-network-customizations"></a>

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

<a name="gatewayconfig-ipv4-object_installing-azure-stack-hub-network-customizations"></a>

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
  <td>The masquerade IPv4 addresses that are used internally to enable host to service traffic. The host is configured with these IP addresses as well as the shared gateway bridge interface. The default value is <code>169.254.169.0/29</code>.<div class="alert alert--warning"><strong>Important</strong><p>For OpenShift Container Platform 4.17 and later versions, clusters use <code>169.254.0.0/17</code> as the default masquerade subnet. For upgraded clusters, there is no change to the default masquerade subnet.</p></div></td>
</tr>
</tbody>
</table>

<a name="gatewayconfig-ipv6-object_installing-azure-stack-hub-network-customizations"></a>

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
  <td>The masquerade IPv6 addresses that are used internally to enable host to service traffic. The host is configured with these IP addresses as well as the shared gateway bridge interface. The default value is <code>fd69::/125</code>.<div class="alert alert--warning"><strong>Important</strong><p>For OpenShift Container Platform 4.17 and later versions, clusters use <code>fd69::/112</code> as the default masquerade subnet. For upgraded clusters, there is no change to the default masquerade subnet.</p></div></td>
</tr>
</tbody>
</table>

<a name="nw-operator-cr-ipsec_installing-azure-stack-hub-network-customizations"></a>

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

```yaml title="Example OVN-Kubernetes configuration with IPSec enabled"
defaultNetwork:
  type: OVNKubernetes
  ovnKubernetesConfig:
    mtu: 1400
    genevePort: 6081
    ipsecConfig:
      mode: Full
```

## Configuring hybrid networking with OVN-Kubernetes {#configuring-hybrid-ovnkubernetes_installing-azure-stack-hub-network-customizations}

To configure hybrid networking with OVN-Kubernetes, you can set `hybridOverlayConfig` during installation or patch the Cluster Network Operator (CNO) after installation.

:::note

This configuration is necessary to run both Linux and Windows nodes in the same cluster.

:::

**Prerequisites**

- You defined `OVNKubernetes` for the `networking.networkType` parameter in the `install-config.yaml` file. See the installation documentation for configuring OpenShift Container Platform network customizations on your chosen cloud provider for more information.

**Procedure**

1. Change to the directory that contains the installation program and create the manifests:
   ```terminal
   $ ./openshift-install create manifests --dir <installation_directory>
   ```

   For the `<installation_directory>`, specify the name of the directory that contains the `install-config.yaml` file for your cluster.
2. Create a stub manifest file for the advanced network configuration that is named `cluster-network-03-config.yml` in the `<installation_directory>/manifests/` directory:
   ```terminal
   $ cat <<EOF > <installation_directory>/manifests/cluster-network-03-config.yml
   apiVersion: operator.openshift.io/v1
   kind: Network
   metadata:
     name: cluster
   spec:
   EOF
   ```

   For `<installation_directory>`, specify the directory name that contains the `manifests/` directory for your cluster.
3. Open the `cluster-network-03-config.yml` file in an editor and specify a hybrid networking configuration similar to the following example:
   ```yaml
   apiVersion: operator.openshift.io/v1
   kind: Network
   metadata:
     name: cluster
   spec:
     defaultNetwork:
       ovnKubernetesConfig:
         hybridOverlayConfig:
           hybridClusterNetwork:
           - cidr: 10.132.0.0/14
             hostPrefix: 23
           hybridOverlayVXLANPort: 9898
   ```

   where:

   <dl>
   <dt><code>spec.defaultNetwork.ovnKubernetesConfig.hybridOverlayConfig.hybridClusterNetwork</code></dt>
   <dd>Specifies the CIDR configuration used for nodes on the additional overlay network. The <code>hybridClusterNetwork</code> CIDR must not overlap with the <code>clusterNetwork</code> CIDR.</dd>
   <dt><code>spec.defaultNetwork.ovnKubernetesConfig.hybridOverlayConfig.hybridOverlayVXLANPort</code></dt>
   <dd>Specifies a custom VXLAN port for the additional overlay network. This is required for running Windows nodes in a cluster installed on vSphere, and must not be configured for any other cloud provider. The custom port can be any open port excluding the default <code>6081</code> port. For more information on this requirement, see <a href="https://docs.microsoft.com/en-us/virtualization/windowscontainers/kubernetes/common-problems#pod-to-pod-connectivity-between-hosts-is-broken-on-my-kubernetes-cluster-running-on-vsphere">Pod-to-pod connectivity between hosts is broken</a> in the Microsoft documentation.</dd>
   </dl>

   :::note

   Windows Server Long-Term Servicing Channel (LTSC): Windows Server 2019 is not supported on clusters with a custom `hybridOverlayVXLANPort` value because this Windows server version does not support selecting a custom VXLAN port.

   :::
4. Save the `cluster-network-03-config.yml` file and quit the text editor.
5. Optional: Back up the `manifests/cluster-network-03-config.yml` file. The installation program deletes the `manifests/` directory when creating the cluster.

## Deploying the cluster {#installation-launching-installer_installing-azure-stack-hub-network-customizations}

To deploy your OpenShift Container Platform cluster, you can initialize installation by running the `openshift-install create cluster` command from the directory that contains the installation program. The installation program provisions infrastructure and completes cluster setup.

:::warning

You can run the `create cluster` command of the installation program only once, during initial installation.

:::

**Prerequisites**

- You have configured an account with the cloud platform that hosts your cluster.
- You have the OpenShift Container Platform installation program and the pull secret for your cluster.
- You have verified that the cloud provider account on your host has the correct permissions to deploy the cluster. An account with incorrect permissions causes the installation process to fail with an error message that displays the missing permissions.

**Procedure**

- In the directory that contains the installation program, initialize the cluster deployment by running the following command:

```terminal
$ ./openshift-install create cluster --dir <installation_directory> \
    --log-level=info
```

- For `<installation_directory>`, specify the location of your customized `./install-config.yaml` file.
- To view different installation details, specify `warn`, `debug`, or `error` instead of `info`.

**Verification**

When the cluster deployment completes successfully:

- The terminal displays directions for accessing your cluster, including a link to the web console and credentials for the `kubeadmin` user.
- Credential information also outputs to `<installation_directory>/.openshift_install.log`.
  :::warning

  Do not delete the installation program or the files that the installation program creates. Both are required to delete the cluster.

  :::

  ```terminal title="Example output"
  ...
  INFO Install complete!
  INFO To access the cluster as the system:admin user when using 'oc', run 'export KUBECONFIG=/home/myuser/install_dir/auth/kubeconfig'
  INFO Access the OpenShift web-console here: https://console-openshift-console.apps.mycluster.example.com
  INFO Login to the console with user: "kubeadmin", and password: "password"
  INFO Time elapsed: 36m22s
  ```

  :::warning

  - The Ignition config files that the installation program generates contain certificates that expire after 24 hours, which are then renewed at that time. If the cluster is shut down before renewing the certificates and the cluster is later restarted after the 24 hours have elapsed, the cluster automatically recovers the expired certificates. The exception is that you must manually approve the pending `node-bootstrapper` certificate signing requests (CSRs) to recover kubelet certificates. See the documentation for *Recovering from expired control plane certificates* for more information.
  - It is recommended that you use Ignition config files within 12 hours after they are generated because the 24-hour certificate rotates from 16 to 22 hours after the cluster is installed. By using the Ignition config files within 12 hours, you can avoid installation failure if the certificate update runs during installation.

  :::

## Logging in to the cluster by using the CLI {#cli-logging-in-kubeadmin_installing-azure-stack-hub-network-customizations}

To log in to your cluster as the default system user, export the `kubeconfig` file. This configuration enables the CLI to authenticate and connect to the specific API server created during OpenShift Container Platform installation.

The `kubeconfig` file is specific to a cluster and OpenShift Container Platform generates it during installation.

**Prerequisites**

- You deployed an OpenShift Container Platform cluster.
- You installed the OpenShift CLI (`oc`).

**Procedure**

1. Export the `kubeadmin` credentials by running the following command:
   ```terminal
   $ export KUBECONFIG=<installation_directory>/auth/kubeconfig
   ```

   where:

   <dl>
   <dt><code>&lt;installation_directory&gt;</code></dt>
   <dd>Specifies the path to the directory that stores the installation files.</dd>
   </dl>
2. Verify you can run `oc` commands successfully using the exported configuration by running the following command:
   ```terminal
   $ oc whoami
   ```

   ```terminal title="Example output"
   system:admin
   ```

**Next steps**

- "Customize your cluster"
- "Remote health reporting"

## Logging in to the cluster by using the web console {#logging-in-by-using-the-web-console_installing-azure-stack-hub-network-customizations}

To verify that your cluster deployed successfully and access its features, log in to the OpenShift Container Platform web console as the `kubeadmin` user.

**Prerequisites**

- You have access to the installation host.
- You completed a cluster installation and all cluster Operators are available.

**Procedure**

1. Obtain the password for the `kubeadmin` user from the `kubeadmin-password` file on the installation host:
   ```terminal
   $ cat <installation_directory>/auth/kubeadmin-password
   ```

   :::note

   Or, you can obtain the `kubeadmin` password from the `<installation_directory>/.openshift_install.log` log file on the installation host.

   :::
2. List the OpenShift Container Platform web console route:
   ```terminal
   $ oc get routes -n openshift-console | grep 'console-openshift'
   ```

   :::note

   Or, you can obtain the OpenShift Container Platform route from the `<installation_directory>/.openshift_install.log` log file on the installation host.

   :::

   ```terminal title="Example output"
   console     console-openshift-console.apps.<cluster_name>.<base_domain>            console     https   reencrypt/Redirect   None
   ```
3. Navigate to the route detailed in the output of the preceding command in a web browser and log in as the `kubeadmin` user.

**Additional resources**

- [Accessing the web console](/docs/web_console/web-console#web-console)

**Next steps**

- [Validating an installation](/docs/installing/validation_and_troubleshooting/validating-an-installation#validating-an-installation)
- [Customize your cluster](/docs/post_installation_configuration/cluster-tasks#available_cluster_customizations)
- Optional: [Remote health reporting](/docs/support/remote_health_monitoring/remote-health-reporting#remote-health-reporting)
- Optional: [Remove cloud provider credentials](/docs/post_installation_configuration/changing-cloud-credentials-configuration#manually-removing-cloud-creds_changing-cloud-credentials-configuration)
