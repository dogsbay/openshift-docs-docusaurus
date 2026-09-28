---
title: Installing a cluster on Azure in a disconnected environment
sidebar_position: 4
---

# Installing a cluster on Azure in a disconnected environment {#installing-restricted-networks-azure-installer-provisioned}

<a id="installing-restricted-networks-azure-installer-provisioned"></a>

In OpenShift Container Platform version 4.22, you can install a cluster on Microsoft Azure in a restricted network by creating an internal mirror of the installation release content on an existing Azure Virtual Network (VNet).

:::warning

You can install an OpenShift Container Platform cluster by using mirrored installation release content, but your cluster requires internet access to use the Azure APIs.

:::

## Prerequisites {#prerequisites_installing-restricted-networks-azure-installer-provisioned}

- You have mirrored the images for a disconnected installation to your registry and obtained the `imageContentSources` data for your version of OpenShift Container Platform. For more information, see "Mirroring images for a disconnected installation by using the oc adm command".
  :::warning

  Because the installation media is on the mirror host, you can use that computer to complete all installation steps.

  :::
- You have an existing VNet in Azure. While installing a cluster in a restricted network that uses installer-provisioned infrastructure, you cannot use the installer-provisioned VNet. You must use a user-provisioned VNet that satisfies one of the following requirements:
  - The VNet contains the mirror registry.
  - The VNet has firewall rules or a peering connection to access the mirror registry hosted elsewhere.

## About installations in restricted networks {#installation-about-restricted-networks_installing-restricted-networks-azure-installer-provisioned}

You can install OpenShift Container Platform 4.22 in a restricted network without an active internet connection to obtain software components. Restricted network installations can use installer-provisioned or user-provisioned infrastructure, depending on the cloud platform to which you are installing the cluster.

If you choose to perform a restricted network installation on a cloud platform, you still require access to its cloud APIs. Some cloud functions, such as Amazon Web Service’s Route 53 DNS and IAM services, require internet access. Depending on your network, you might require less internet access for an installation on bare-metal hardware, Nutanix, or on VMware vSphere.

To complete a restricted network installation, you must create a registry that mirrors the contents of the OpenShift image registry and contains the installation media. You can create this registry on a mirror host, which can access both the internet and your closed network, or by using other methods that meet your restrictions.

### Additional limits {#installation-restricted-network-limits_installing-restricted-networks-azure-installer-provisioned}

Clusters in restricted networks have the following additional limitations and restrictions:

- The `ClusterVersion` status includes an `Unable to retrieve available updates` error.
- By default, you cannot use the contents of the Developer Catalog because you cannot access the required image stream tags.

### User-defined outbound routing {#installation-azure-user-defined-routing_installing-restricted-networks-azure-installer-provisioned}

You can configure user-defined outbound routing for a OpenShift Container Platform cluster to reach the internet without public IP addresses or a public load balancer.

You must use a pre-existing `VNet` for outbound routing when you install a cluster. The installation program does not configure this network.

When you configure a cluster to use user-defined routing, the installation program does not create the following resources:

- Outbound rules for access to the internet.
- Public IPs for the public load balancer.
- Kubernetes Service object to add the cluster machines to the public load balancer for outbound requests.

Before you set user-defined routing, ensure that the following items are available:

- Egress to the internet so that you can pull container images, unless you use an OpenShift image registry mirror.
- Access from the cluster to Azure APIs.
- Access to the required allowlist endpoints.

Several pre-existing networking setups support internet access by using user-defined routing.

#### Restricted cluster with Azure Firewall {#restricted-cluster-with-azure-firewall_installing-restricted-networks-azure-installer-provisioned}

You can use Azure Firewall to restrict the outbound routing for the Virtual Network (`VNet`) that is used to install the OpenShift Container Platform cluster. You can create a OpenShift Container Platform cluster in a restricted network by using a `VNet` with Azure Firewall and configuring user-defined routing.

:::warning

If you use Azure Firewall to restrict internet access, you must set the `publish` field to `Internal` in the `install-config.yaml` file. Azure Firewall does not work properly with Azure public load balancers.

:::

**Additional resources**

- [Providing user-defined routing with Azure Firewall](https://learn.microsoft.com/en-us/azure/aks/egress-outboundtype#deploy-a-cluster-with-outbound-type-of-udr-and-azure-firewall)
- [Azure Firewall does not work properly with Azure public load balancers](https://learn.microsoft.com/en-us/azure/firewall/integrate-lb)

## About reusing a VNet for your OpenShift Container Platform cluster {#installation-about-custom-azure-vnet_installing-restricted-networks-azure-installer-provisioned}

In OpenShift Container Platform 4.22, you can deploy a cluster into an existing Microsoft Azure Virtual Network (VNet). Deployments in an existing VNet require existing subnets and routing rules.

By deploying OpenShift Container Platform into an existing Azure VNet, you might be able to avoid service limit constraints in new accounts or more easily abide by the operational constraints that your company’s guidelines set. This is a good option to use if you cannot obtain the infrastructure creation permissions that are required to create the VNet.

### Requirements for using your VNet {#installation-about-custom-azure-vnet-requirements_installing-restricted-networks-azure-installer-provisioned}

When you deploy a cluster by using an existing VNet, you must perform additional network configuration before you install the cluster. In installer-provisioned infrastructure clusters, the installation program usually creates the following components, but it does not create them when you install into an existing VNet:

- Subnets
- Route tables
- VNets
- Network Security Groups

:::note

The installation program requires that you use the cloud-provided DNS server. Using a custom DNS server is not supported and causes the installation to fail.

:::

If you use a custom VNet, you must correctly configure it and its subnets for the installation program and the cluster to use. The installation program cannot subdivide network ranges for the cluster to use, set route tables for the subnets, or set VNet options like DHCP, so you must do so before you install the cluster.

The cluster must be able to access the resource group that contains the existing VNet and subnets. While all of the resources that the cluster creates are placed in a separate resource group that it creates, some network resources are used from a separate group. Some cluster Operators must be able to access resources in both resource groups. For example, the Machine API controller attaches NICs for the virtual machines that it creates to subnets from the networking resource group.

Your VNet must meet the following characteristics:

- The VNet’s CIDR block must contain the `Networking.MachineCIDR` range, which is the IP address pool for cluster machines.
- The VNet and its subnets must belong to the same resource group, and the subnets must be configured to use Azure-assigned DHCP IP addresses instead of static IP addresses.

You must provide two subnets within your VNet, one for the control plane machines and one for the compute machines. Because Azure distributes machines in different availability zones within the region that you specify, your cluster will have high availability by default.

:::note

By default, if you specify availability zones in the `install-config.yaml` file, the installation program distributes the control plane machines and the compute machines across availability zones within a region. To ensure high availability for your cluster, select a region with at least three availability zones. If your region contains fewer than three availability zones, the installation program places more than one control plane machine in the available zones. For more information, see "Availability zones" and "Regions".

:::

To ensure that the subnets that you provide are suitable, the installation program confirms the following data:

- All the specified subnets exist.
- There are two private subnets, one for the control plane machines and one for the compute machines.
- The subnet CIDRs belong to the machine CIDR that you specified. Machines are not provisioned in availability zones that you do not provide private subnets for. If required, the installation program creates public load balancers that manage the control plane and worker nodes, and Azure allocates a public IP address to them.

:::note

If you destroy a cluster that uses an existing VNet, the VNet is not deleted.

:::

#### Network security group requirements {#installation-about-custom-azure-vnet-nsg-requirements_installing-restricted-networks-azure-installer-provisioned}

The network security groups for the subnets that host the compute and control plane machines require specific access to ensure that the cluster communication is correct. You must create rules to allow access to the required cluster communication ports.

:::warning

The network security group rules must be in place before you install the cluster. If you attempt to install a cluster without the required access, the installation program cannot reach the Azure APIs, and installation fails.

:::

**Required ports**

<table>
<thead>
<tr>
  <th>Port</th>
  <th>Description</th>
  <th>Control plane</th>
  <th>Compute</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>80</code></td>
  <td>Allows HTTP traffic</td>
  <td></td>
  <td>x</td>
</tr>
<tr>
  <td><code>443</code></td>
  <td>Allows HTTPS traffic</td>
  <td></td>
  <td>x</td>
</tr>
<tr>
  <td><code>6443</code></td>
  <td>Allows communication to the control plane machines</td>
  <td>x</td>
  <td></td>
</tr>
<tr>
  <td><code>22623</code></td>
  <td>Allows internal communication to the machine config server for provisioning machines</td>
  <td>x</td>
  <td></td>
</tr>
<tr>
  <td><code>*</code></td>
  <td>Allows connections to Azure APIs. You must set a Destination Service Tag to <code>AzureCloud</code>. <sup>[1]</sup></td>
  <td>x</td>
  <td>x</td>
</tr>
<tr>
  <td><code>*</code></td>
  <td>Denies connections to the internet. You must set a Destination Service Tag to <code>Internet</code>. <sup>[1]</sup></td>
  <td>x</td>
  <td>x</td>
</tr>
</tbody>
</table>

1. If you are using Azure Firewall to restrict the internet access, then you can configure Azure Firewall to allow the Azure APIs. A network security group rule is not needed. For more information, see "Configuring your firewall" in "Additional resources".

:::warning

Currently, there is no supported way to block or restrict the machine config server endpoint. The machine config server must be exposed to the network so that newly-provisioned machines, which have no existing configuration or state, are able to fetch their configuration. In this model, the root of trust is the certificate signing requests (CSR) endpoint, which is where the kubelet sends its certificate signing request for approval to join the cluster. Because of this, machine configs should not be used to distribute sensitive information, such as secrets and certificates.

To ensure that the machine config server endpoints, ports 22623 and 22624, are secured in bare metal scenarios, customers must configure proper network policies.

:::

Because cluster components do not modify the user-provided network security groups, which the Kubernetes controllers update, a pseudo-network security group is created for the Kubernetes controller to modify without impacting the rest of the environment.

**Ports used for all-machine to all-machine communications**

<table>
<thead>
<tr>
  <th>Protocol</th>
  <th>Port</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>ICMP</td>
  <td>N/A</td>
  <td>Network reachability tests</td>
</tr>
<tr>
  <td rowspan="3">TCP</td>
  <td><code>1936</code></td>
  <td>Metrics</td>
</tr>
<tr>
  <td><code>9000</code>-<code>9999</code></td>
  <td>Host level services, including the node exporter on ports <code>9100</code>-<code>9101</code> and the Cluster Version Operator on port <code>9099</code>.</td>
</tr>
<tr>
  <td><code>10250</code>-<code>10259</code></td>
  <td>The default ports that Kubernetes reserves</td>
</tr>
<tr>
  <td rowspan="5">UDP</td>
  <td><code>6081</code></td>
  <td>Geneve</td>
</tr>
<tr>
  <td><code>9000</code>-<code>9999</code></td>
  <td>Host level services, including the node exporter on ports <code>9100</code>-<code>9101</code>.</td>
</tr>
<tr>
  <td><code>500</code></td>
  <td>IPsec IKE packets</td>
</tr>
<tr>
  <td><code>4500</code></td>
  <td>IPsec NAT-T packets</td>
</tr>
<tr>
  <td><code>123</code></td>
  <td>Network Time Protocol (NTP) on UDP port <code>123</code>. If you configure an external NTP time server, you must open UDP port <code>123</code>.</td>
</tr>
<tr>
  <td>TCP/UDP</td>
  <td><code>30000</code>-<code>32767</code></td>
  <td>Kubernetes node port</td>
</tr>
<tr>
  <td>ESP</td>
  <td>N/A</td>
  <td>IPsec Encapsulating Security Payload (ESP)</td>
</tr>
</tbody>
</table>

**Ports used for control plane machine to control plane machine communications**

<table>
<thead>
<tr>
  <th>Protocol</th>
  <th>Port</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>TCP</td>
  <td><code>2379</code>-<code>2380</code></td>
  <td>etcd server and peer ports</td>
</tr>
</tbody>
</table>

### Division of permissions {#installation-about-custom-azure-permissions_installing-restricted-networks-azure-installer-provisioned}

Starting with OpenShift Container Platform 4.3, you do not need all of the permissions that are required for an installation program-provisioned infrastructure cluster to deploy a cluster. This change mimics the division of permissions that you might have at your company: some individuals can create different resources in your clouds than others. For example, you might be able to create application-specific items, like instances, storage, and load balancers, but not networking-related components such as VNets, subnet, or ingress rules.

The Azure credentials that you use when you create your cluster do not need the networking permissions that are required to make VNets and core networking components within the VNet, such as subnets, routing tables, internet gateways, NAT, and VPN. You still need permission to make the application resources that the machines within the cluster require, such as load balancers, security groups, storage accounts, and nodes.

### Isolation between clusters {#installation-about-custom-azure-vnet-isolation_installing-restricted-networks-azure-installer-provisioned}

Because the cluster is unable to modify network security groups in an existing subnet, there is no way to isolate clusters from each other on the VNet.

**Additional resources**

- [About the OVN-Kubernetes network plugin](/docs/networking/ovn_kubernetes_network_provider/about-ovn-kubernetes#about-ovn-kubernetes)
- [Configuring your firewall](/docs/installing/install_config/configuring-firewall#configuring-firewall-module_configuring-firewall)
- [Availability zones](https://azure.microsoft.com/en-us/global-infrastructure/availability-zones/)
- [Regions](https://azure.microsoft.com/en-us/global-infrastructure/regions/)

## Creating the installation configuration file {#installation-initializing_installing-restricted-networks-azure-installer-provisioned}

You can customize the OpenShift Container Platform cluster you install on Microsoft Azure.

:::warning

Do not specify `windows`, `microsoft`, or other variants of these words in the `metadata.name` parameter of the `install-config.yaml` file. Specifying one of these words for the cluster name causes the installation program to generate an error message like the following example message:

```terminal
The resource name 'windows-xxxx-identity' or a part of the name is a trademarked or reserved word.
```

Additionally, specifying `login` at the beginning of the name in the `metadata.name` parameter of the `install-config.yaml` file results in the generation of an error message. You can specify `login` in the middle or end of the name.

:::

**Prerequisites**

- You have the OpenShift Container Platform installation program and the pull secret for your cluster. For a restricted network installation, these files are on your mirror host.
- You have the `imageContentSources` values that were generated during mirror registry creation.
- You have obtained the contents of the certificate for your mirror registry.
- You have retrieved a Red Hat Enterprise Linux CoreOS (RHCOS) image and uploaded it to an accessible location.
- You have an Azure subscription ID and tenant ID.
- If you are installing the cluster using a service principal, you have its application ID and password.
- If you are installing the cluster using a system-assigned managed identity, you have enabled it on the virtual machine that you will run the installation program from.
- If you are installing the cluster using a user-assigned managed identity, you have met these prerequisites:
  - You have its client ID.
  - You have assigned it to the virtual machine that you will run the installation program from.

**Procedure**

1. Optional: If you have run the installation program on this computer before, and want to use an alternative service principal or managed identity, go to the `~/.azure/` directory and delete the `osServicePrincipal.json` configuration file.
   Deleting this file prevents the installation program from automatically reusing subscription and authentication values from a previous installation.
2. Create the `install-config.yaml` file.
   1. Change to the directory that contains the installation program and run the following command:
      ```terminal
      $ ./openshift-install create install-config --dir <installation_directory>
      ```

      - `<installation_directory>`: For `<installation_directory>`, specify the directory name to store the files that the installation program creates.
        When specifying the directory:
      - Verify that the directory has the `execute` permission. This permission is required to run Terraform binaries under the installation directory.
      - Use an empty directory. Some installation assets, such as bootstrap X.509 certificates, have short expiration intervals, therefore you must not reuse an installation directory. If you want to reuse individual files from another cluster installation, you can copy them into your directory. However, the file names for the installation assets might change between releases. Use caution when copying installation files from an earlier OpenShift Container Platform version.
   2. At the prompts, provide the configuration details for your cloud:
      1. Optional: Select an SSH key to use to access your cluster machines.
         :::note

         For production OpenShift Container Platform clusters on which you want to perform installation debugging or disaster recovery, specify an SSH key that your `ssh-agent` process uses.

         :::
      2. Select **azure** as the platform to target.
         If the installation program cannot locate the `osServicePrincipal.json` configuration file from a previous installation, you are prompted for Azure subscription and authentication values.
      3. Enter the following Azure parameter values for your subscription:
         - **azure subscription id**: Enter the subscription ID to use for the cluster.
         - **azure tenant id**: Enter the tenant ID.
      4. Depending on the Azure identity you are using to deploy the cluster, do one of the following when prompted for the **azure service principal client id**:
         - If you are using a service principal, enter its application ID.
         - If you are using a system-assigned managed identity, leave this value blank.
         - If you are using a user-assigned managed identity, specify its client ID.
      5. Depending on the Azure identity you are using to deploy the cluster, do one of the following when prompted for the **azure service principal client secret**:
         - If you are using a service principal, enter its password.
         - If you are using a system-assigned managed identity, leave this value blank.
         - If you are using a user-assigned managed identity, leave this value blank.
      6. Select the region to deploy the cluster to.
      7. Select the base domain to deploy the cluster to. The base domain corresponds to the Azure DNS Zone that you created for your cluster.
      8. Enter a descriptive name for your cluster.
         :::warning

         All Azure resources that are available through public endpoints are subject to resource name restrictions, and you cannot create resources that use certain terms. For a list of terms that Azure restricts, see [Resolve reserved resource name errors](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-reserved-resource-name) in the Azure documentation.

         :::
      9. Paste the [pull secret from Red Hat OpenShift Cluster Manager](https://console.redhat.com/openshift/install/pull-secret).
3. Edit the `install-config.yaml` file to give the additional information that is required for an installation in a restricted network.
   1. Update the `pullSecret` value to contain the authentication information for your registry:
      ```yaml
      pullSecret: '{"auths":{"<mirror_host_name>:5000": {"auth": "<credentials>","email": "you@example.com"}}}'
      ```

      For `<mirror_host_name>`, specify the registry domain name that you specified in the certificate for your mirror registry, and for `<credentials>`, specify the base64-encoded user name and password for your mirror registry.
   2. Add the `additionalTrustBundle` parameter and value.
      ```yaml
      additionalTrustBundle: |
        -----BEGIN CERTIFICATE-----
        ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ
        -----END CERTIFICATE-----
      ```

      The value must be the contents of the certificate file that you used for your mirror registry. The certificate file can be an existing, trusted certificate authority, or the self-signed certificate that you generated for the mirror registry.
   3. Define the network and subnets for the VNet to install the cluster under the `platform.azure` field:
      ```yaml
      networkResourceGroupName: <vnet_resource_group>
      virtualNetwork: <vnet>
      controlPlaneSubnet: <control_plane_subnet>
      computeSubnet: <compute_subnet>
      ```

      where:

      <dl>
      <dt><code>&lt;vnet_resource_group&gt;</code></dt>
      <dd>Specifies the resource group name that contains the existing virtual network (VNet).</dd>
      <dt><code>&lt;vnet&gt;</code></dt>
      <dd>Specifies the existing virtual network name.</dd>
      <dt><code>&lt;control_plane_subnet&gt;</code></dt>
      <dd>Specifies the existing subnet name to deploy the control plane machines.</dd>
      <dt><code>&lt;compute_subnet&gt;</code></dt>
      <dd>Specifies the existing subnet name to deploy compute machines.</dd>
      </dl>
   4. Add the image content resources, which resemble the following YAML excerpt:
      ```yaml
      imageContentSources:
      - mirrors:
        - <mirror_host_name>:5000/<repo_name>/release
        source: quay.io/openshift-release-dev/ocp-release
      - mirrors:
        - <mirror_host_name>:5000/<repo_name>/release
        source: registry.redhat.io/ocp/release
      ```

      For these values, use the `imageContentSources` that you recorded during mirror registry creation.
   5. Optionally, set the publishing strategy to `Internal`:
      ```yaml
      publish: Internal
      ```

      By setting this option, you create an internal Ingress Controller and a private load balancer.

      :::warning

      Azure Firewall [does not work seamlessly](https://learn.microsoft.com/en-us/azure/firewall/integrate-lb) with Azure Public Load balancers. Thus, when using Azure Firewall for restricting internet access, the `publish` field in `install-config.yaml` should be set to `Internal`.

      :::
4. Make any other modifications to the `install-config.yaml` file that you require.
   For more information about the parameters, see "Installation configuration parameters".
5. Back up the `install-config.yaml` file so that you can use it to install multiple clusters.
   :::warning

   The `install-config.yaml` file is consumed during the installation process. If you want to reuse the file, you must back it up now.

   :::

   If previously not detected, the installation program creates an `osServicePrincipal.json` configuration file and stores this file in the `~/.azure/` directory on your computer. This ensures that the installation program can load the profile when it is creating an OpenShift Container Platform cluster on the target platform.

**Additional resources**

- [Installation configuration parameters for Azure](/docs/installing/installing_azure/installation-config-parameters-azure#installation-config-parameters-azure)

### Minimum resource requirements for cluster installation {#installation-minimum-resource-requirements_installing-restricted-networks-azure-installer-provisioned}

To ensure that your OpenShift Container Platform cluster runs as expected, each cluster machine must meet minimum CPU, memory, and storage requirements.

**Minimum resource requirements**

<table>
<thead>
<tr>
  <th>Machine</th>
  <th>Operating system</th>
  <th>vCPU</th>
  <th>Virtual RAM</th>
  <th>Storage</th>
  <th>Input/Output Per Second (IOPS)</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Bootstrap</td>
  <td>RHCOS</td>
  <td>4</td>
  <td>16 GB</td>
  <td>100 GB</td>
  <td>300</td>
</tr>
<tr>
  <td>Control plane</td>
  <td>RHCOS</td>
  <td>4</td>
  <td>16 GB</td>
  <td>100 GB</td>
  <td>300</td>
</tr>
<tr>
  <td>Compute</td>
  <td>RHCOS</td>
  <td>2</td>
  <td>8 GB</td>
  <td>100 GB</td>
  <td>300</td>
</tr>
</tbody>
</table>

- One vCPU is equal to one physical core when simultaneous multithreading (SMT), or Hyper-Threading, is not enabled. When enabled, use the following formula to calculate the corresponding ratio: (threads per core × cores) × sockets = vCPUs.
- OpenShift Container Platform and Kubernetes are sensitive to disk performance, and Red Hat recommends faster storage, particularly for etcd on the control plane nodes which require a 10 ms p99 fsync duration. On many cloud platforms, storage size and IOPS scale together, so you might need to provision more storage to get enough performance.
- As with all user-provisioned installations, if you choose to use RHEL compute machines in your cluster, you take responsibility for all operating system life cycle management and maintenance, including performing system updates, applying patches, and completing all other required tasks. OpenShift Container Platform 4.10 and later do not support RHEL 7 compute machines.

:::note

In OpenShift Container Platform version 4.22, RHCOS uses RHEL version 9.8, which updates the micro-architecture requirements. Each architecture requires the following minimum instruction set architectures (ISA):

- x86-64 architecture requires x86-64-v2 ISA
- ARM64 architecture requires ARMv8.0-A ISA
- ppc64le architecture requires IBM(R) Power9 ISA
- s390x architecture requires IBM(R) z14 ISA

For more information, see [Architectures](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html-single/9.8_release_notes/index#architectures) in the RHEL documentation.

:::

:::warning

You must use Azure virtual machines that have the `premiumIO` parameter set to `true`.

:::

If an instance type for your platform meets the minimum requirements for cluster machines, it is supported to use in OpenShift Container Platform.

### Tested instance types for Azure {#installation-azure-tested-machine-types_installing-restricted-networks-azure-installer-provisioned}

There are several Microsoft Azure instance types tested with OpenShift Container Platform. Choose a listed instance type when you install a cluster on 64-bit x86 infrastructure.

See the following machine types based on 64-bit x86 architecture:

##### General Purpose

| Azure VM Series | Family Name |
| --- | --- |
| Basv2-series | `standardBasv2Family` |
| BS-series | `standardBSFamily` |
| Bsv2-series | `standardBsv2Family` |
| Dadsv5-series | `standardDADSv5Family` |
| Dadsv6-series | `standardDadv6Family` |
| Daldsv6-series | `standardDaldv6Family` |
| Dalsv6-series | `standardDalv6Family` |
| Dasv4-series | `standardDASv4Family` |
| Dasv5-series | `standardDASv5Family` |
| Dasv6-series | `standardDav6Family` |
| DCas_cc_v5-series | `standardDCACCV5Family` |
| DCads_cc_v5-series | `standardDCADCCV5Family` |
| DCadsv5-series | `standardDCADSv5Family` |
| DCasv5-series | `standardDCASv5Family` |
| DCsv2-series | `standardDCSv2Family` |
| DCsv3-series | `standardDCSv3Family` |
| DCdsv3-series | `standardDDCSv3Family` |
| DCedsv5-series | `standardDCEDV5Family` |
| DCesv5-series | `standardDCEV5Family` |
| Ddsv4-series | `standardDDSv4Family` |
| Ddsv5-series | `standardDDSv5Family` |
| Ddsv6-series | `StandardDdsv6Family` |
| Dldsv5-series | `standardDLDSv5Family` |
| Dldsv6-series | `StandardDldsv6Family` |
| Dlsv5-series | `standardDLSv5Family` |
| Dlsv6-series | `StandardDlsv6Family` |
| DS-series | `standardDSFamily` |
| Dsv2-series | `standardDSv2Family` |
| Dsv3-series | `standardDSv3Family` |
| Dsv4-series | `standardDSv4Family` |
| Dsv5-series | `standardDSv5Family` |
| Dsv6-series | `StandardDsv6Family` |

##### Memory Optimized

| Azure VM Series | Family Name |
| --- | --- |
| Eadsv5-series | `standardEADSv5Family`, `standardEIADSv5Family` |
| Eadsv6-series | `standardEadv6Family` |
| Easv4-series | `standardEASv4Family`, `standardEIASv4Family` |
| Easv5-series | `standardEASv5Family`, `standardEIASv5Family` |
| Easv6-series | `standardEav6Family` |
| Ebdsv5-series | `standardEBDSv5Family`, `standardEIBDSv5Family` |
| Ebsv5-series | `standardEBSv5Family`, `standardEIBSv5Family` |
| ECas_cc_v5-series | `standardECACCV5Family` |
| ECads_cc_v5-series | `standardECADCCV5Family` |
| ECadsv5-series | `standardECADSv5Family` |
| ECasv5-series | `standardECASv5Family` |
| ECedsv5-series | `standardECEDV5Family` |
| ECesv5-series | `standardECEV5Family` |
| Edsv4-series | `standardEDSv4Family` |
| Edsv5-series | `standardEDSv5Family`, `standardEIDSv5Family` |
| Edsv6-series | `StandardEdsv6Family` |
| Esv3-series | `standardESv3Family`, `standardEISv3Family` |
| Esv4-series | `standardESv4Family`, `standardXEISv4Family` |
| Esv5-series | `standardESv5Family`, `standardEISv5Family` |
| Esv6-series | `StandardEsv6Family` |
| M-series | `standardMSFamily` |
| Mbdsv3-series | `StandardMBDSMediumMemoryv3Family` |
| Mbsv3-series | `StandardMBSMediumMemoryv3Family` |
| Mdsv3 High Memory-series | `standardMDSHighMemoryv3Family`, `standardMIDSHighMemoryv3Family` |
| Mdsv2 Medium Memory-series | `standardMDSMediumMemoryv2Family`, `standardMIDSMediumMemoryv2Family` |
| Mdsv3 Medium Memory-series | `standardMDSMediumMemoryv3Family` |
| Msv3 High Memory-series | `standardMISHighMemoryv3Family`, `standardMSHighMemoryv3Family` |
| Msv2 Medium Memory-series | `standardMISMediumMemoryv2Family`, `standardMSMediumMemoryv2Family` |
| Msv3 Medium Memory-series | `standardMSMediumMemoryv3Family` |

##### Compute Optimized

| Azure VM Series | Family Name |
| --- | --- |
| Falsv6-series | `StandardFalsv6Family` |
| Famsv6-series | `StandardFamsv6Family` |
| Fasv6-series | `StandardFasv6Family` |
| FS-series | `standardFSFamily` |
| Fsv2-series | `standardFSv2Family` |
| FXmdsv2-series | `StandardFXmdsv2Family` |
| FX-series | `standardFXMDVSFamily` |
| FXmsv2-series | `StandardFXmsv2Family` |

##### Storage Optimized

| Azure VM Series | Family Name |
| --- | --- |
| GS-series | `standardGSFamily` |
| Laosv4-series | `standardLaosv4Family` |
| Lasv3-series | `standardLASv3Family` |
| Lasv4-series | `standardLasv4Family` |
| Ls-series | `standardLSFamily` |
| Lsv2-series | `standardLSv2Family` |
| Lsv3-series | `standardLSv3Family` |
| Lsv4-series | `standardLsv4Family` |

##### GPU Accelerated

| Azure VM Series | Family Name |
| --- | --- |
| NC_A100_v4-series | `StandardNCADSA100v4Family` |
| NCads_H100_v5-series | `StandardNCadsH100v5Family` |
| NCCads_H100_v5-series | `StandardNCCads2023Family` |
| NCasT4_v3-series | `Standard NCASv3_T4 Family` |
| NCv3-series | `standardNCSv3Family` |
| NDasrA100_v4-series | `Standard NDASv4_A100 Family` |
| ND-H200-v5-series | `standardNDISRH200V5Family` |
| ND-H100-v5-series | `standardNDSH100v5Family` |
| NDv2-series | `standardNDSv2Family` |
| NGads_V620-series | `StandardNGADSV620v1Family` |
| NVadsA10_v5-series | `StandardNVADSA10v5Family` |
| NVads V710 v5-series | `StandardNVadsV710v5Family` |
| NVv3-series | `standardNVSv3Family` |

##### FPGA Accelerated

| Azure VM Series | Family Name |
| --- | --- |
| NPS-series | `standardNPSFamily` |

##### High Performance Compute

| Azure VM Series | Family Name |
| --- | --- |
| HBv2-series | `standardHBrsv2Family` |
| HBv4-series | `standardHBv4Family` |
| HBv5-series | `standardHBv5Family` |
| HC-series | `standardHCSFamily` |
| HX-series | `standardHXFamily` |

### Tested instance types for Azure on 64-bit ARM infrastructures {#installation-azure-arm-tested-machine-types_installing-restricted-networks-azure-installer-provisioned}

There are several Microsoft Azure ARM64 instance types tested with OpenShift Container Platform. Choose a listed instance type when you install a cluster on 64-bit ARM infrastructure.

See the following machine types based on 64-bit ARM architecture:

##### General Purpose (ARM64)

| Azure VM Series | Family Name |
| --- | --- |
| Bpsv2-series | `standardBpsv2Family` |
| Dpdsv5-series | `standardDPDSv5Family` |
| Dpldsv5-series | `standardDPLDSv5Family` |
| Dplsv5-series | `standardDPLSv5Family` |
| Dpsv5-series | `standardDPSv5Family` |
| Dpdsv6-series | `StandardDpdsv6Family` |
| Dpldsv6-series | `StandardDpldsv6Family` |
| Dplsv6-series | `StandardDplsv6Family` |
| Dpsv6-series | `StandardDpsv6Family` |

##### Memory Optimized (ARM64)

| Azure VM Series | Family Name |
| --- | --- |
| Epdsv5-series | `standardEPDSv5Family` |
| Epsv5-series | `standardEPSv5Family` |
| Epdsv6-series | `StandardEpdsv6Family` |
| Epsv6-series | `StandardEpsv6Family` |

### Enabling trusted launch for Azure VMs {#installation-azure-trusted-launch_installing-restricted-networks-azure-installer-provisioned}

To enable trusted launch on Azure virtual machines for your OpenShift Container Platform cluster, you can configure secure boot and virtualized Trusted Platform Modules in the `install-config.yaml` file. Apply the settings to control plane nodes, compute nodes, or all nodes as needed.

For more information about the sizes of virtual machines that support the trusted launch features, secure boot, and virtualized Trusted Platform Modules, see the Additional resources section.

:::warning

Trusted launch is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.

For more information about the support scope of Red Hat Technology Preview features, see [Technology Preview Features Support Scope](https://access.redhat.com/support/offerings/techpreview/).

:::

**Prerequisites**

- You have created an `install-config.yaml` file.

**Procedure**

- Edit the `install-config.yaml` file before deploying your cluster:
  - Enable trusted launch only on control plane by adding the following stanza:
    ```yaml
    controlPlane:
      platform:
        azure:
          settings:
            securityType: TrustedLaunch
            trustedLaunch:
              uefiSettings:
                secureBoot: Enabled
                virtualizedTrustedPlatformModule: Enabled
    ```
  - Enable trusted launch only on compute node by adding the following stanza:
    ```yaml
    compute:
      platform:
        azure:
          settings:
            securityType: TrustedLaunch
            trustedLaunch:
              uefiSettings:
                secureBoot: Enabled
                virtualizedTrustedPlatformModule: Enabled
    ```
  - Enable trusted launch on all nodes by adding the following stanza:
    ```yaml
    platform:
      azure:
        settings:
          securityType: TrustedLaunch
          trustedLaunch:
            uefiSettings:
              secureBoot: Enabled
              virtualizedTrustedPlatformModule: Enabled
    ```

### Enable confidential VMs {#installation-azure-confidential-vms_installing-restricted-networks-azure-installer-provisioned}

To enable confidential VMs on Azure for your OpenShift Container Platform cluster, you can configure the `install-config.yaml` file before deployment. Apply the settings to control plane nodes, compute nodes, or all nodes as needed.

You can use confidential VMs with the following VM sizes:

- DCasv5-series
- DCadsv5-series
- ECasv5-series
- ECadsv5-series
- DCesv5-series
- DCedsv5-series
- ECesv5-series
- ECedsv5-series
- NCCads_H100_v5

:::warning

Confidential VMs are currently not supported on 64-bit ARM architectures.

:::

**Prerequisites**

- You have created an `install-config.yaml` file.

**Procedure**

- Edit the `install-config.yaml` file before deploying your cluster:
  - Enable confidential VMs only on control plane by adding the following stanza:
    ```yaml
    controlPlane:
      platform:
        azure:
          settings:
            securityType: ConfidentialVM
            confidentialVM:
              uefiSettings:
                secureBoot: Enabled
                virtualizedTrustedPlatformModule: Enabled
          osDisk:
            securityProfile:
              securityEncryptionType: VMGuestStateOnly
    ```
  - Enable confidential VMs only on compute nodes by adding the following stanza:
    ```yaml
    compute:
      platform:
        azure:
          settings:
            securityType: ConfidentialVM
            confidentialVM:
              uefiSettings:
                secureBoot: Enabled
                virtualizedTrustedPlatformModule: Enabled
          osDisk:
            securityProfile:
              securityEncryptionType: VMGuestStateOnly
    ```
  - Enable confidential VMs on all nodes by adding the following stanza:
    ```yaml
    platform:
      azure:
        defaultMachinePlatform:
          settings:
            securityType: ConfidentialVM
            confidentialVM:
              uefiSettings:
                secureBoot: Enabled
                virtualizedTrustedPlatformModule: Enabled
          osDisk:
            securityProfile:
              securityEncryptionType: VMGuestStateOnly
    ```

### Configuring a dedicated disk for etcd {#installation-azure-dedicated-disks_installing-restricted-networks-azure-installer-provisioned}

You can install your OpenShift Container Platform cluster on Microsoft Azure with a dedicated data disk for `etcd`. This configuration attaches a separate managed disk to each control plane node and uses it only for `etcd` data, which can improve cluster performance and stability.

:::warning

Dedicated disk for etcd is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.

For more information about the support scope of Red Hat Technology Preview features, see [Technology Preview Features Support Scope](https://access.redhat.com/support/offerings/techpreview/).

:::

**Prerequisites**

- You have created an `install-config.yaml` file.

**Procedure**

- To configure a dedicated `etcd` disk, edit the `install-config.yaml` file and add the `diskSetup` and `dataDisks` parameters to the `controlPlane` stanza:
  ```yaml
  # ...
  controlPlane:
    architecture: amd64
    hyperthreading: Enabled
    name: master
    platform:
      azure:
        type: Standard_D4s_v5
        dataDisks:
        - nameSuffix: etcddisk
          cachingType: None
          diskSizeGB: 20
          lun: 0
    diskSetup:
    - type: etcd
      etcd:
        platformDiskID: etcddisk
    replicas: 3
  # ...
  ```

  where:

  <dl>
  <dt><code>controlPlane.platform.azure.dataDisks.nameSuffix</code></dt>
  <dd>Specifies the same value you defined for <code>platformDiskID</code>.</dd>
  <dt><code>controlPlane.platform.azure.dataDisks.cachingType</code></dt>
  <dd>Specifies <code>None</code>. Other caching requirements are not currently supported.</dd>
  <dt><code>controlPlane.platform.azure.dataDisks.diskSizeGB</code></dt>
  <dd>Specifies a disk size in GB. This value can be any integer greater than <code>0</code>.</dd>
  </dl>

  :::note

  A minimum of 20 GB ensures enough space is available for defragmentation operations.

  :::

  <dl>
  <dt><code>controlPlane.platform.azure.dataDisks.lun</code></dt>
  <dd>Specifies a logical unit number (LUN). This can be any integer from <code>0</code> through <code>63</code> that is not used by another disk.</dd>
  <dt><code>controlPlane.diskSetup.type</code></dt>
  <dd>Specifies <code>etcd</code>. This identifies <code>etcd</code> as the node component type to receive a dedicated disk.</dd>
  <dt><code>controlPlane.diskSetup.etcd.platformDiskID</code></dt>
  <dd>Specifies a name to identify the disk. This value must not exceed 12 characters.</dd>
  </dl>

### Enabling a user-managed DNS {#installation-azure-enabling-user-managed-DNS_installing-restricted-networks-azure-installer-provisioned}

You can install a cluster with a domain name server (DNS) solution that you manage instead of the default cluster-provisioned DNS solution. As a result, you can manage the API and Ingress DNS records in your own system rather than adding the records to the DNS of the cloud. For example, your organization’s security policies might not allow the use of public DNS services such as Microsoft Azure. In such scenarios, you can use your own DNS service to bypass the public DNS service and manage your own DNS for the IP addresses of the API and Ingress services.

If you enable user-managed DNS during installation, the installation program provisions DNS records for the API and Ingress services only within the cluster. To ensure access from outside the cluster, you must provision the DNS records in an external DNS service of your choice for the API and Ingress services after installation.

**Prerequisites**

- You installed the `jq` package.

**Procedure**

- Before you deploy your cluster, use a text editor to open the `install-config.yaml` file  and add the following stanza:
  - To enable user-managed DNS:
    ```yaml
    # ...

    platform:
      azure:
        userProvisionedDNS: Enabled
    ```

    where:

    <dl>
    <dt><code>userProvisionedDNS</code></dt>
    <dd>Enables user-provisioned DNS management.</dd>
    </dl>

**Next steps**

For information about provisioning your DNS records for the API server and the Ingress services, see "Provisioning your own DNS records".

### Sample customized install-config.yaml file for Azure {#installation-azure-config-yaml-simple_installing-restricted-networks-azure-installer-provisioned}

You can customize the `install-config.yaml` file to specify more details about your OpenShift Container Platform cluster’s platform or modify the values of the required parameters.

:::warning

This sample YAML file is provided for reference only. You must obtain your `install-config.yaml` file by using the installation program and modify it. For a full list and description of all installation configuration parameters, see *Installation configuration parameters for Azure*.

:::

```yaml title="Sample install-config.yaml file for Azure"
apiVersion: v1
baseDomain: example.com
pullSecret: '{"auths": ...}'
sshKey: ssh-ed25519 AAAA...
metadata:
  name: example-cluster
controlPlane:
  hyperthreading: Enabled
  name: master
  platform:
    azure:
      type: Standard_D8s_v3
  replicas: 3
compute:
- hyperthreading: Enabled
  name: worker
  platform:
    azure:
      type: Standard_D2s_v3
  replicas: 3
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
platform:
  azure:
    baseDomainResourceGroupName: example-basedomain-resourcegroup-name
    region: centralus
```

where:

`controlPlane`
: Specifies parameters that apply to control plane machines.

`compute`
: Specifies parameters that apply to compute machines.

`networking`
: Specifies parameters that apply to the cluster networking configuration. If you do not provide networking values, the installation program provides default values.

`platform`
: Specifies parameters that apply to the infrastructure platform that hosts the cluster.

**Additional resources**

- [Installation configuration parameters for Azure](/docs/installing/installing_azure/installation-config-parameters-azure#installation-config-parameters-azure)

### Configuring the cluster-wide proxy during installation {#installation-configure-proxy_installing-restricted-networks-azure-installer-provisioned}

Production environments can deny direct access to the internet and instead have an HTTP or HTTPS proxy available. You can configure a new OpenShift Container Platform cluster to use a proxy by configuring the proxy settings in the `install-config.yaml` file.

**Prerequisites**

- You have an existing `install-config.yaml` file.
- You have reviewed the sites that your cluster requires access to and determined whether any of them need to bypass the proxy. By default, the proxy handles all cluster egress traffic, including calls to hosting cloud provider APIs. You added sites to the `Proxy` object’s `spec.noProxy` field to bypass the proxy if necessary.
  :::note

  The `Proxy` object `status.noProxy` field includes the values of the `networking.machineNetwork[].cidr`, `networking.clusterNetwork[].cidr`, and `networking.serviceNetwork[]` fields from your installation configuration.

  For installations on Amazon Web Services (AWS), Google Cloud, Microsoft Azure, and Red Hat OpenStack Platform (RHOSP), the `Proxy` object `status.noProxy` field also includes the instance metadata endpoint (`169.254.169.254`).

  :::

**Procedure**

1. Edit your `install-config.yaml` file and add the proxy settings. For example:
   ```yaml
   apiVersion: v1
   baseDomain: my.domain.com
   proxy:
     httpProxy: http://<username>:<pswd>@<ip>:<port>
     httpsProxy: https://<username>:<pswd>@<ip>:<port>
     noProxy: example.com
   additionalTrustBundle: |
       -----BEGIN CERTIFICATE-----
       <MY_TRUSTED_CA_CERT>
       -----END CERTIFICATE-----
   additionalTrustBundlePolicy: <policy_to_add_additionalTrustBundle>
   # ...
   ```

   where:

   <dl>
   <dt><code>proxy.httpProxy</code></dt>
   <dd>Specifies a proxy URL to use for creating HTTP connections outside the cluster. The URL scheme must be <code>http</code>.</dd>
   <dt><code>proxy.httpsProxy</code></dt>
   <dd>Specifies a proxy URL to use for creating HTTPS connections outside the cluster.</dd>
   <dt><code>proxy.noProxy</code></dt>
   <dd>Specifies a comma-separated list of destination domain names, IP addresses, or other network CIDRs to exclude from proxying. Preface a domain with <code>.</code> to match subdomains only. For example, <code>.y.com</code> matches <code>x.y.com</code>, but not <code>y.com</code>. Use <code>*</code> to bypass the proxy for all destinations.</dd>
   <dt><code>additionalTrustBundle</code></dt>
   <dd>If you specify this value, the installation program generates a config map named <code>user-ca-bundle</code> in the <code>openshift-config</code> namespace to hold the additional CA certificates. If you specify <code>additionalTrustBundle</code> and at least one proxy setting, the <code>Proxy</code> object references the <code>user-ca-bundle</code> config map in the <code>trustedCA</code> field. The Cluster Network Operator then creates a <code>trusted-ca-bundle</code> config map that merges the contents specified for the <code>trustedCA</code> parameter with the RHCOS trust bundle. You must set the <code>additionalTrustBundle</code> field unless an authority from the RHCOS trust bundle signs the proxy’s identity certificate.</dd>
   <dt><code>additionalTrustBundlePolicy</code></dt>
   <dd>Specifies the policy that determines the configuration of the <code>Proxy</code> object to reference the <code>user-ca-bundle</code> config map in the <code>trustedCA</code> field. The allowed values are <code>Proxyonly</code> and <code>Always</code>. Use <code>Proxyonly</code> to reference the <code>user-ca-bundle</code> config map only when you configure an <code>http/https</code> proxy. Use <code>Always</code> to always reference the <code>user-ca-bundle</code> config map. The default value is <code>Proxyonly</code>. Optional parameter.</dd>
   </dl>

   :::note

   The installation program does not support the proxy `readinessEndpoints` field.

   :::

   :::note

   If the installation program times out, restart and then complete the deployment by using the `wait-for` command of the installation program. For example:

   ```terminal
   $ ./openshift-install wait-for install-complete --log-level debug
   ```

   :::
2. Save the file and reference it when installing OpenShift Container Platform.
   The installation program creates a cluster-wide proxy named `cluster` that uses the proxy settings in the `install-config.yaml` file. If you do not give proxy settings, the installation program still creates a `cluster` `Proxy` object, but it has a nil `spec`.

   :::note

   Only the `Proxy` object named `cluster` is supported, and you cannot create additional proxies.

   :::

## Alternatives to storing administrator-level secrets in the kube-system project {#installing-azure-manual-modes_installing-restricted-networks-azure-installer-provisioned}

To avoid storing administrator-level secrets in the `kube-system` project, you can configure your cluster to use manually managed long-term cloud credentials or short-term credentials that are managed outside the cluster.

By default, administrator-level secrets are stored in the `kube-system` project. If you set the `credentialsMode` parameter in the `install-config.yaml` file to `Manual`, choose one of the following credential management options:

- To manage long-term cloud credentials manually, follow the procedure in "Manually creating long-term credentials".
- To use short-term credentials that are managed outside the cluster for individual components, follow the procedures in "Configuring an Azure cluster to use short-term credentials".

**Additional resources**

- [Manually creating long-term credentials](/docs/installing/installing_azure/ipi/installing-restricted-networks-azure-installer-provisioned#manually-create-iam_installing-restricted-networks-azure-installer-provisioned)
- [Configuring an Azure cluster to use short-term credentials](/docs/installing/installing_azure/ipi/installing-restricted-networks-azure-installer-provisioned#installing-azure-with-short-term-creds_installing-restricted-networks-azure-installer-provisioned)

### Manually creating long-term credentials {#manually-create-iam_installing-restricted-networks-azure-installer-provisioned}

You can put the Cloud Credential Operator (CCO) into manual mode before OpenShift Container Platform installation if the cloud identity and access management (IAM) APIs are not reachable, or if you prefer not to store an administrator-level credential secret in the cluster `kube-system` namespace.

**Procedure**

1. If you did not set the `credentialsMode` parameter in the `install-config.yaml` configuration file to `Manual`, modify the value as shown:
   ```yaml title="Sample configuration file snippet"
   apiVersion: v1
   baseDomain: example.com
   credentialsMode: Manual
   # ...
   ```
2. If you have not previously created installation manifest files, do so by running the following command:
   ```terminal
   $ openshift-install create manifests --dir <installation_directory>
   ```

   where `<installation_directory>` is the directory in which the installation program creates files.
3. Set a `$RELEASE_IMAGE` variable with the release image from your installation file by running the following command:
   ```terminal
   $ RELEASE_IMAGE=$(./openshift-install version | awk '/release image/ {print $3}')
   ```
4. Extract the list of `CredentialsRequest` custom resources (CRs) from the OpenShift Container Platform release image by running the following command:
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
5. Create YAML files for secrets in the `openshift-install` manifests directory that you generated previously. The secrets must be stored using the namespace and secret name defined in the `spec.secretRef` for each `CredentialsRequest` object.
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

### Configuring an Azure cluster to use short-term credentials {#installing-azure-with-short-term-creds_installing-restricted-networks-azure-installer-provisioned}

To install a cluster that uses Microsoft Entra Workload ID, you must configure the Cloud Credential Operator utility and create the required Azure resources for your cluster.

#### Configuring the Cloud Credential Operator utility {#cco-ccoctl-configuring_installing-restricted-networks-azure-installer-provisioned}

To create and manage cloud credentials from outside of the cluster when the Cloud Credential Operator (CCO) is operating in manual mode, extract and prepare the CCO utility (`ccoctl`) binary.

:::note

The `ccoctl` utility is a Linux binary that must run in a Linux environment.

:::

**Prerequisites**

- You have access to an OpenShift Container Platform account with cluster administrator access.
- You have installed the OpenShift CLI (`oc`).
- You have created a global Azure account for the `ccoctl` utility to use with the following permissions:
  - `Microsoft.Resources/subscriptions/resourceGroups/read`
  - `Microsoft.Resources/subscriptions/resourceGroups/write`
  - `Microsoft.Resources/subscriptions/resourceGroups/delete`
  - `Microsoft.Authorization/roleAssignments/read`
  - `Microsoft.Authorization/roleAssignments/delete`
  - `Microsoft.Authorization/roleAssignments/write`
  - `Microsoft.Authorization/roleDefinitions/read`
  - `Microsoft.Authorization/roleDefinitions/write`
  - `Microsoft.Authorization/roleDefinitions/delete`
  - `Microsoft.Storage/storageAccounts/listkeys/action`
  - `Microsoft.Storage/storageAccounts/delete`
  - `Microsoft.Storage/storageAccounts/read`
  - `Microsoft.Storage/storageAccounts/write`
  - `Microsoft.Storage/storageAccounts/blobServices/containers/delete`
  - `Microsoft.Storage/storageAccounts/blobServices/containers/read`
  - `Microsoft.Storage/storageAccounts/blobServices/containers/write`
  - `Microsoft.ManagedIdentity/userAssignedIdentities/delete`
  - `Microsoft.ManagedIdentity/userAssignedIdentities/read`
  - `Microsoft.ManagedIdentity/userAssignedIdentities/write`
  - `Microsoft.ManagedIdentity/userAssignedIdentities/federatedIdentityCredentials/read`
  - `Microsoft.ManagedIdentity/userAssignedIdentities/federatedIdentityCredentials/write`
  - `Microsoft.ManagedIdentity/userAssignedIdentities/federatedIdentityCredentials/delete`
  - `Microsoft.Storage/register/action`
  - `Microsoft.ManagedIdentity/register/action`

**Procedure**

1. Set a variable for the OpenShift Container Platform release image by running the following command:
   ```terminal
   $ RELEASE_IMAGE=$(./openshift-install version | awk '/release image/ {print $3}')
   ```
2. Obtain the CCO container image from the OpenShift Container Platform release image by running the following command:
   ```terminal
   $ CCO_IMAGE=$(oc adm release info --image-for='cloud-credential-operator' $RELEASE_IMAGE -a ~/.pull-secret)
   ```

   :::note

   Ensure that the architecture of the `$RELEASE_IMAGE` matches the architecture of the environment in which you will use the `ccoctl` tool.

   :::
3. Extract the `ccoctl` binary from the CCO container image within the OpenShift Container Platform release image by running the following command:
   ```terminal
   $ oc image extract $CCO_IMAGE \
     --file="/usr/bin/ccoctl.<rhel_version>" \
     -a ~/.pull-secret
   ```

   For `<rhel_version>`, specify the value that corresponds to the version of Red Hat Enterprise Linux (RHEL) that the host uses. If no value is specified, `ccoctl.rhel8` is used by default. The following values are valid:

   - `rhel8`: Specify this value for hosts that use RHEL 8.
   - `rhel9`: Specify this value for hosts that use RHEL 9.
     :::note

     The `ccoctl` binary is created in the directory from where you executed the command and not in `/usr/bin/`. You must rename the directory or move the `ccoctl.<rhel_version>` binary to `ccoctl`.

     :::
4. Change the permissions to make `ccoctl` executable by running the following command:
   ```terminal
   $ chmod 775 ccoctl
   ```

**Verification**

- To verify that `ccoctl` is ready to use, display the help file. Use a relative file name when you run the command, for example:
  ```terminal
  $ ./ccoctl
  ```

  ```terminal title="Example output"
  OpenShift credentials provisioning tool

  Usage:
    ccoctl [command]

  Available Commands:
    aws          Manage credentials objects for AWS cloud
    azure        Manage credentials objects for Azure
    gcp          Manage credentials objects for Google cloud
    help         Help about any command
    ibmcloud     Manage credentials objects for IBM Cloud
    nutanix      Manage credentials objects for Nutanix

  Flags:
    -h, --help   help for ccoctl

  Use "ccoctl [command] --help" for more information about a command.
  ```

= Creating Azure resources with the Cloud Credential Operator utility

You can use the `ccoctl azure create-all` command to automate the creation of Azure resources.

:::note

By default, `ccoctl` creates objects in the directory in which the commands are run. To create the objects in a different directory, use the `--output-dir` flag. This procedure uses `<path_to_ccoctl_output_dir>` to refer to this directory.

:::

**Prerequisites**

You must have:

- Extracted and prepared the `ccoctl` binary.
- Access to your Microsoft Azure account by using the Azure CLI.

**Procedure**

1. Set a `$RELEASE_IMAGE` variable with the release image from your installation file by running the following command:
   ```terminal
   $ RELEASE_IMAGE=$(./openshift-install version | awk '/release image/ {print $3}')
   ```
2. Extract the list of `CredentialsRequest` objects from the OpenShift Container Platform release image by running the following command:
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
   <dd>Specifies to include only the manifests that your specific cluster configuration requires.</dd>
   <dt><code>&lt;path_to_directory_with_installation_configuration&gt;</code></dt>
   <dd>Specifies the location of the <code>install-config.yaml</code> file.</dd>
   <dt><code>&lt;path_to_directory_for_credentials_requests&gt;</code></dt>
   <dd>Specifies the path to the directory where you want to store the <code>CredentialsRequest</code> objects. If the specified directory does not exist, this command creates it.</dd>
   </dl>

   :::note

   This command might take a few moments to run.

   :::
3. To enable the `ccoctl` utility to detect your Azure credentials automatically, log in to the Azure CLI by running the following command:
   ```terminal
   $ az login
   ```
4. Use the `ccoctl` tool to process all `CredentialsRequest` objects by running the following command:
   ```terminal
   $ ccoctl azure create-all \
     --name=<azure_infra_name> \
     --output-dir=<ccoctl_output_dir> \
     --region=<azure_region> \
     --subscription-id=<azure_subscription_id> \
     --credentials-requests-dir=<path_to_credentials_requests_directory> \
     --dnszone-resource-group-name=<azure_dns_zone_resource_group_name> \
     --tenant-id=<azure_tenant_id> \
     --network-resource-group-name <azure_resource_group> \
     --preserve-existing-roles
   ```

   where:

   <dl>
   <dt><code>&lt;azure_infra_name&gt;</code></dt>
   <dd>Specifies the user-defined name for all created Azure resources used for tracking.</dd>
   <dt><code>&lt;ccoctl_output_dir&gt;</code></dt>
   <dd>Specifies the directory in which you want the <code>ccoctl</code> utility to create objects. By default, the utility creates objects in the directory in which the commands are run. This parameter is optional.</dd>
   <dt><code>&lt;azure_region&gt;</code></dt>
   <dd>Specifies the Azure region in which cloud resources will be created.</dd>
   <dt><code>&lt;azure_subscription_id&gt;</code></dt>
   <dd>Specifies the Azure subscription ID to use.</dd>
   <dt><code>&lt;path_to_credentials_requests_directory&gt;</code></dt>
   <dd>Specifies the directory containing the files for the component <code>CredentialsRequest</code> objects.</dd>
   <dt><code>&lt;azure_dns_zone_resource_group_name&gt;</code></dt>
   <dd>Specifies the name of the resource group containing the cluster’s base domain Azure DNS zone.</dd>
   <dt><code>&lt;azure_tenant_id&gt;</code></dt>
   <dd>Specifies the Azure tenant ID to use.</dd>
   <dt><code>&lt;azure_resource_group&gt;</code></dt>
   <dd>Specifies the virtual network resource group if it is different from the cluster resource group. This parameter is optional.</dd>
   <dt><code>--preserve-existing-roles</code></dt>
   <dd>Specifies that any custom role assignments you define on managed identities are not removed during OpenShift Container Platform updates. This parameter is optional.</dd>
   </dl>

   :::note

   If your cluster uses Technology Preview features that are enabled by the `TechPreviewNoUpgrade` feature set, you must include the `--enable-tech-preview` parameter.

   To see additional optional parameters and explanations of how to use them, run the `azure create-all --help` command.

   :::

**Verification**

- To verify that the OpenShift Container Platform secrets are created, list the files in the `<path_to_ccoctl_output_dir>/manifests` directory:
  ```terminal
  $ ls <path_to_ccoctl_output_dir>/manifests
  ```

  ```text title="Example output"
  azure-ad-pod-identity-webhook-config.yaml
  cluster-authentication-02-config.yaml
  openshift-cloud-controller-manager-azure-cloud-credentials-credentials.yaml
  openshift-cloud-network-config-controller-cloud-credentials-credentials.yaml
  openshift-cluster-api-capz-manager-bootstrap-credentials-credentials.yaml
  openshift-cluster-csi-drivers-azure-disk-credentials-credentials.yaml
  openshift-cluster-csi-drivers-azure-file-credentials-credentials.yaml
  openshift-image-registry-installer-cloud-credentials-credentials.yaml
  openshift-ingress-operator-cloud-credentials-credentials.yaml
  openshift-machine-api-azure-cloud-credentials-credentials.yaml
  ```

  You can verify that the Microsoft Entra ID service accounts are created by querying Azure. For more information, refer to Azure documentation on listing Entra ID service accounts.

#### Incorporating the Cloud Credential Operator utility manifests {#cco-ccoctl-install-creating-manifests_installing-restricted-networks-azure-installer-provisioned}

To implement short-term security credentials managed outside the cluster for individual OpenShift Container Platform components, you must move the manifest files that the Cloud Credential Operator utility (`ccoctl`) created to the correct directories for the installation program.

**Prerequisites**

- You have configured an account with the cloud platform that hosts your cluster.
- You have configured the Cloud Credential Operator utility (`ccoctl`).
- You have created the cloud provider resources that are required for your cluster with the `ccoctl` utility.

**Procedure**

1. If you did not set the `credentialsMode` parameter in the `install-config.yaml` configuration file to `Manual`, modify the value as shown:
   ```yaml title="Sample configuration file snippet"
   apiVersion: v1
   baseDomain: example.com
   credentialsMode: Manual
   # ...
   ```
2. If you used the `ccoctl` utility to create a new Azure resource group instead of using an existing resource group, modify the `resourceGroupName` parameter in the `install-config.yaml` as shown:
   ```yaml title="Sample configuration file snippet"
   apiVersion: v1
   baseDomain: example.com
   # ...
   platform:
     azure:
       resourceGroupName: <azure_infra_name>
   # ...
   ```

   The `<azure_infra_name>` value must match the user-defined name for Azure resources that was specified with the `--name` argument of the `ccoctl azure create-all` command.
3. If you have not previously created installation manifest files, do so by running the following command:
   ```terminal
   $ openshift-install create manifests --dir <installation_directory>
   ```

   where `<installation_directory>` is the directory in which the installation program creates files.
4. Copy the manifests that the `ccoctl` utility generated to the `manifests` directory that the installation program created by running the following command:
   ```terminal
   $ cp /<path_to_ccoctl_output_dir>/manifests/* ./manifests/
   ```
5. Copy the `tls` directory that contains the private key to the installation directory:
   ```terminal
   $ cp -a /<path_to_ccoctl_output_dir>/tls .
   ```

## Deploying the cluster {#installation-launching-installer_installing-restricted-networks-azure-installer-provisioned}

To deploy your OpenShift Container Platform cluster, you initialize installation by running the `openshift-install create cluster` command from the directory that contains the installation program. The installation program provisions the required infrastructure and completes the cluster setup.

:::warning

You can run the `create cluster` command of the installation program only once, during initial installation.

:::

**Prerequisites**

- You have configured an account with the cloud platform that hosts your cluster.
- You have the OpenShift Container Platform installation program and the pull secret for your cluster.
- You have an Azure subscription ID and tenant ID.

**Procedure**

- In the directory that contains the installation program, initialize the cluster deployment by running the following command:

```terminal
$ ./openshift-install create cluster --dir <installation_directory> \
    --log-level=info
```

where:

- `<installation_directory>`: Specifies the location of your customized `./install-config.yaml` file.
- `--log-level`: Specifies the log level. To view different installation details, specify `warn`, `debug`, or `error` instead of `info`.

**Verification**

When the cluster deployment completes successfully:

- The terminal displays directions for accessing your cluster, including a link to the web console and credentials for the `kubeadmin` user.
- Credential information also outputs to `<installation_directory>/.openshift_install.log`.
  :::warning

  Do not delete the installation program or the files that the installation program creates. Both are required to delete the cluster.

  :::

  The following example shows the expected output:

  ```terminal
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

## Provisioning your own DNS records {#installation-azure-provisioning-own-dns-records_installing-restricted-networks-azure-installer-provisioned}

Use the IP address of the API server to provision your own DNS record with the `api.<cluster_name>.<base_domain>.` hostname by using your cluster name and base cluster domain. Use the IP address of the Ingress service to provision your own DNS record with the `*.apps.<cluster_name>.<base_domain>.` hostname by using your cluster name and base cluster domain.

**Prerequisite**

- You have installed the Azure CLI client `(az)`.

**Procedure**

1. Add the `userProvisionedDNS` parameter to the `install-config.yaml` file and enable the parameter. For more information, see "Enabling a user-managed DNS".
2. Install your cluster.
3. If you are installing a private cluster, set the `lb_name` variable by running the following command:
   ```terminal
   $ lb_name="${infra_id}-internal"
   ```

   1. Set the `frontendipconfig_id` variable by running the following command:
      ```terminal
      $ frontendipconfig_id=$(az network lb show -n ${lb_name} -g ${cluster_resource_group_name} -ojson | jq -r ".loadBalancingRules[] | select(.frontendPort == 6443) | .frontendIPConfiguration.id")
      ```
   2. Set the `frontendipconfig_name` variable by running the following command:
      ```terminal
      $ frontendipconfig_name=${frontendipconfig_id##*/}
      ```
   3. To retrieve the IP address of the API service, run the following command:
      ```terminal
      $ az network lb frontend-ip show -n ${frontendipconfig_name} --lb-name ${lb_name} -g ${cluster_resource_group_name} --query "privateIPAddress" -otsv
      ```
4. If you are installing a public cluster, set the `lb_name` variable by running the following command:
   ```terminal
   $ lb_name="${infra_id}"
   ```

   1. Set the `frontendipconfig_id` variable by running the following command:
      ```terminal
      $ frontendipconfig_id=$(az network lb show -n ${lb_name} -g ${cluster_resource_group_name} -ojson | jq -r ".loadBalancingRules[] | select(.frontendPort == 6443) | .frontendIPConfiguration.id")
      ```
   2. Set the `frontendipconfig_name` variable by running the following command:
      ```terminal
      $ frontendipconfig_name=${frontendipconfig_id##*/}
      ```
   3. Set the `frontendpublicip_id` variable by running the following command:
      ```terminal
      $ frontendpublicip_id=$(az network lb frontend-ip show -n ${frontendipconfig_name} --lb-name ${lb_name} -g ${cluster_resource_group_name} --query "publicIPAddress.id" -otsv)
      ```
   4. To retrieve the IP address of the API service, run the following command:
      ```terminal
      $ az network public-ip show --ids ${frontendpublicip_id} --query 'ipAddress' -otsv
      ```
5. Use the IP address and your cluster name and base cluster domain to configure your own DNS record with the `api.<cluster_name>.<base_domain>.` hostname.
6. If you are installing a private cluster, set the `lb_name` variable by running the following command:
   ```terminal
   $ lb_name="${infra_id}-internal"
   ```

   1. Set the `frontendipconfig_id` variable by running the following command:
      ```terminal
      $ frontendipconfig_id=$(az network lb show -n ${lb_name} -g ${cluster_resource_group_name} -ojson | jq -r ".loadBalancingRules[] | select(.frontendPort == 443) | .frontendIPConfiguration.id")
      ```
   2. Set the `frontendipconfig_name` variable by running the following command:
      ```terminal
      $ frontendipconfig_name=${frontendipconfig_id##*/}
      ```
   3. To retrieve the IP address of the Ingress service, run the following command:
      ```terminal
      $ az network lb frontend-ip show -n ${frontendipconfig_name} --lb-name ${lb_name} -g ${cluster_resource_group_name} --query "privateIPAddress" -otsv
      ```
7. If you are installing a public cluster, set the `lb_name` variable by running the following command:
   ```terminal
   $ lb_name="${infra_id}"
   ```

   1. Set the `frontendipconfig_id` variable by running the following command:
      ```terminal
      $ frontendipconfig_id=$(az network lb show -n ${lb_name} -g ${cluster_resource_group_name} -ojson | jq -r ".loadBalancingRules[] | select(.frontendPort == 443) | .frontendIPConfiguration.id")
      ```
   2. Set the `frontendipconfig_name` variable by running the following command:
      ```terminal
      $ frontendipconfig_name=${frontendipconfig_id##*/}
      ```
   3. Set the `frontendpublicip_id` variable by running the following command:
      ```terminal
      $ frontendpublicip_id=$(az network lb frontend-ip show -n ${frontendipconfig_name} --lb-name ${lb_name} -g ${cluster_resource_group_name} --query "publicIPAddress.id" -otsv)
      ```
   4. To retrieve the IP address of the Ingress service, run the following command:
      ```terminal
      $ az network public-ip show --ids ${frontendpublicip_id} --query 'ipAddress' -otsv
      ```
8. Use the IP address and your cluster name and base cluster domain to configure your own DNS record with the `*.apps.<cluster_name>.<base_domain>.` hostname.

## Logging in to the cluster by using the CLI {#cli-logging-in-kubeadmin_installing-restricted-networks-azure-installer-provisioned}

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

**Additional resources**

- [Mirroring images for a disconnected installation by using the oc adm command](/docs/disconnected/installing-mirroring-installation-images#installation-about-mirror-registry_installing-mirroring-installation-images)
- [Customize your cluster](/docs/post_installation_configuration/cluster-tasks#available_cluster_customizations)
- [Remote health reporting](/docs/support/remote_health_monitoring/remote-health-reporting#remote-health-reporting)
