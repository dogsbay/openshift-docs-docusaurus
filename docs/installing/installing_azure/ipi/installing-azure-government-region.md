---
title: Installing a cluster on Azure into a government region
sidebar_position: 7
---

# Installing a cluster on Azure into a government region {#installing-azure-government-region}

<a id="installing-azure-government-region"></a>

In OpenShift Container Platform version 4.22, you can install a cluster on Microsoft Azure into a government region. To configure the government region, you modify parameters in the `install-config.yaml` file before you install the cluster.

## Azure government regions {#installation-azure-about-government-region_installing-azure-government-region}

Microsoft Azure Government (MAG) is a cloud environment designed for US government agencies for US government agencies at the federal, state, and local level, as well as contractors, educational institutions, and other US customers that must run sensitive workloads on Azure. MAG is composed of government-only data center regions, all granted an Impact Level 5 Provisional Authorization.

Deploying a OpenShift Container Platform cluster to a Microsoft Azure Government (MAG) region requires manually configuring the Azure Government cloud instance and region in the `install-config.yaml` file before you install the cluster. You must also update your service principal to reference the appropriate government environment.

:::note

The Azure government region cannot be selected using the guided terminal prompts from the installation program. You must define the region manually in the `install-config.yaml` file. Remember to also set the dedicated cloud instance, like `AzureUSGovernmentCloud`, based on the region specified.

:::

**Additional resources**

- [Microsoft Azure Government (MAG)](https://docs.microsoft.com/en-us/azure/azure-government/documentation-government-welcome)
- [Impact Level 5 Provisional Authorization](https://docs.microsoft.com/en-us/microsoft-365/compliance/offering-dod-disa-l2-l4-l5?view=o365-worldwide#dod-impact-level-5-provisional-authorization)

## Private clusters {#private-clusters-default_installing-azure-government-region}

You can deploy a private OpenShift Container Platform cluster that does not expose external endpoints. Private clusters are accessible from only an internal network and are not visible to the internet.

By default, OpenShift Container Platform is provisioned to use publicly-accessible DNS and endpoints. A private cluster sets the DNS, Ingress Controller, and API server to private when you deploy your cluster. This means that the cluster resources are only accessible from your internal network and are not visible to the internet.

:::warning

If the cluster has any public subnets, load balancer services created by administrators might be publicly accessible. To ensure cluster security, verify that these services are explicitly annotated as private.

:::

To deploy a private cluster, you must:

- Use existing networking that meets your requirements. Your cluster resources might be shared between other clusters on the network.
- Deploy from a machine that has access to: ** The API services for the cloud to which you provision. ** The hosts on the network that you provision. ** The internet to obtain installation media.

### Private clusters in Azure {#private-clusters-about-azure_installing-azure-government-region}

To create a private cluster on Microsoft Azure, you must provide an existing private VNet and subnets to host the cluster. The installation program must also be able to resolve the DNS records that the cluster requires. The installation program configures the Ingress Operator and API server for only internal traffic.

Depending how your network connects to the private VNET, you might need to use a DNS forwarder to resolve the cluster’s private DNS records. The cluster’s machines use `168.63.129.16` internally for DNS resolution. For more information, see "What is Azure Private DNS?" and "What is IP address 168.63.129.16?".

The cluster still requires access to internet to access the Azure APIs.

The following items are not required or created when you install a private cluster:

- A `BaseDomainResourceGroup`, since the cluster does not create public records
- Public IP addresses
- Public DNS records
- Public endpoints
  ```
  The cluster is configured so that the Operators do not create public records for the cluster and all cluster machines are placed in the private subnets that you specify.
  ```

#### Limitations {#private-clusters-limitations-azure_installing-azure-government-region}

Private clusters on Azure are subject to only the limitations that are associated with the use of an existing VNet.

**Additional resources**

- [What is Azure Private DNS?](https://docs.microsoft.com/en-us/azure/dns/private-dns-overview)
- [What is IP address 168.63.129.16?](https://docs.microsoft.com/en-us/azure/virtual-network/what-is-ip-address-168-63-129-16)

### User-defined outbound routing {#installation-azure-user-defined-routing_installing-azure-government-region}

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

## About reusing a VNet for your OpenShift Container Platform cluster {#installation-about-custom-azure-vnet_installing-azure-government-region}

In OpenShift Container Platform 4.22, you can deploy a cluster into an existing Microsoft Azure Virtual Network (VNet). Deployments in an existing VNet require existing subnets and routing rules.

By deploying OpenShift Container Platform into an existing Azure VNet, you might be able to avoid service limit constraints in new accounts or more easily abide by the operational constraints that your company’s guidelines set. This is a good option to use if you cannot obtain the infrastructure creation permissions that are required to create the VNet.

### Requirements for using your VNet {#installation-about-custom-azure-vnet-requirements_installing-azure-government-region}

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

#### Network security group requirements {#installation-about-custom-azure-vnet-nsg-requirements_installing-azure-government-region}

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

### Division of permissions {#installation-about-custom-azure-permissions_installing-azure-government-region}

Starting with OpenShift Container Platform 4.3, you do not need all of the permissions that are required for an installation program-provisioned infrastructure cluster to deploy a cluster. This change mimics the division of permissions that you might have at your company: some individuals can create different resources in your clouds than others. For example, you might be able to create application-specific items, like instances, storage, and load balancers, but not networking-related components such as VNets, subnet, or ingress rules.

The Azure credentials that you use when you create your cluster do not need the networking permissions that are required to make VNets and core networking components within the VNet, such as subnets, routing tables, internet gateways, NAT, and VPN. You still need permission to make the application resources that the machines within the cluster require, such as load balancers, security groups, storage accounts, and nodes.

### Isolation between clusters {#installation-about-custom-azure-vnet-isolation_installing-azure-government-region}

Because the cluster is unable to modify network security groups in an existing subnet, there is no way to isolate clusters from each other on the VNet.

**Additional resources**

- [About the OVN-Kubernetes network plugin](/docs/networking/ovn_kubernetes_network_provider/about-ovn-kubernetes#about-ovn-kubernetes)
- [Configuring your firewall](/docs/installing/install_config/configuring-firewall#configuring-firewall-module_configuring-firewall)
- [Availability zones](https://azure.microsoft.com/en-us/global-infrastructure/availability-zones/)
- [Regions](https://azure.microsoft.com/en-us/global-infrastructure/regions/)

## Manually creating the installation configuration file {#installation-initializing-manual_installing-azure-government-region}

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
   1. Edit the `install-config.yaml` file so that the value of the `platform.azure.cloudName` parameter is `AzureUSGovernmentCloud`.

   :::note

   You must name this configuration file `install-config.yaml`.

   :::
3. Back up the `install-config.yaml` file so that you can use it to install many clusters.
   :::warning

   Back up the `install-config.yaml` file now, because the installation process consumes the file in the next step.

   :::

**Additional resources**

- [Installation configuration parameters for Azure](/docs/installing/installing_azure/installation-config-parameters-azure#installation-config-parameters-azure)

### Minimum resource requirements for cluster installation {#installation-minimum-resource-requirements_installing-azure-government-region}

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

**Additional resources**

- [Optimizing storage](/docs/scalability_and_performance/optimization/optimizing-storage#optimizing-storage)

### Tested instance types for Azure {#installation-azure-tested-machine-types_installing-azure-government-region}

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

### Enabling trusted launch for Azure VMs {#installation-azure-trusted-launch_installing-azure-government-region}

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

### Enable confidential VMs {#installation-azure-confidential-vms_installing-azure-government-region}

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

### Sample customized install-config.yaml file for Azure {#installation-azure-config-yaml-simple_installing-azure-government-region}

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

### Configuring the cluster-wide proxy during installation {#installation-configure-proxy_installing-azure-government-region}

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

**Additional resources**

- [Accelerated Networking for Microsoft Azure VMs](/docs/machine_management/creating_machinesets/creating-machineset-azure#machineset-azure-accelerated-networking_creating-machineset-azure)

## Deploying the cluster {#installation-launching-installer_installing-azure-government-region}

To deploy your OpenShift Container Platform cluster, you initialize installation by running the `openshift-install create cluster` command from the directory that contains the installation program. The installation program provisions the required infrastructure and completes the cluster setup.

:::warning

You can run the `create cluster` command of the installation program only once, during initial installation.

:::

**Prerequisites**

- You have configured an account with the cloud platform that hosts your cluster.
- You have the OpenShift Container Platform installation program and the pull secret for your cluster.
- You have an Azure subscription ID and tenant ID.
- If you are installing the cluster using a service principal, you have its application ID and password.
- If you are installing the cluster using a system-assigned managed identity, you have enabled it on the virtual machine that you will run the installation program from.
- If you are installing the cluster using a user-assigned managed identity, you have met these prerequisites:
  - You have its client ID.
  - You have assigned it to the virtual machine that you will run the installation program from.

**Procedure**

```
. Optional: If you have run the installation program on this computer before, and want to use an alternative service principal or managed identity, go to the `~/.azure/` directory and delete the `osServicePrincipal.json` configuration file.
```

Deleting this file prevents the installation program from automatically reusing subscription and authentication values from a previous installation.

1. In the directory that contains the installation program, initialize the cluster deployment by running the following command:

```terminal
$ ./openshift-install create cluster --dir <installation_directory> \
    --log-level=info
```

where:

- `<installation_directory>`: Specifies the location of your customized `./install-config.yaml` file.
- `--log-level`: Specifies the log level. To view different installation details, specify `warn`, `debug`, or `error` instead of `info`.

If the installation program cannot locate the `osServicePrincipal.json` configuration file from a previous installation, you are prompted for Azure subscription and authentication values. . Enter the following Azure parameter values for your subscription: *** *azure subscription id**: Enter the subscription ID to use for the cluster. *** *azure tenant id**: Enter the tenant ID. . Depending on the Azure identity you are using to deploy the cluster, do one of the following when prompted for the **azure service principal client id**: ** If you are using a service principal, enter its application ID. ** If you are using a system-assigned managed identity, leave this value blank. ** If you are using a user-assigned managed identity, specify its client ID. . Depending on the Azure identity you are using to deploy the cluster, do one of the following when prompted for the **azure service principal client secret**: ** If you are using a service principal, enter its password. ** If you are using a system-assigned managed identity, leave this value blank. ** If you are using a user-assigned managed identity,leave this value blank.

:::note

If previously not detected, the installation program creates an `osServicePrincipal.json` configuration file and stores this file in the `~/.azure/` directory on your computer. This ensures that the installation program can load the profile when it is creating an OpenShift Container Platform cluster on the target platform.

:::

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

## Logging in to the cluster by using the CLI {#cli-logging-in-kubeadmin_installing-azure-government-region}

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

- [Accessing the web console](/docs/web_console/web-console#web-console)
- [Customize your cluster](/docs/post_installation_configuration/cluster-tasks#available_cluster_customizations)
- [Remote health reporting](/docs/support/remote_health_monitoring/remote-health-reporting#remote-health-reporting)
