---
title: Installation requirements for IBM Z and IBM LinuxONE infrastructure
sidebar_position: 1
---

# Installation requirements for IBM Z and IBM LinuxONE infrastructure {#installing-ibm-z-reqs}

<a id="installing-ibm-z-reqs"></a>

Before you begin an installation on IBM Z(R) infrastructure, be sure that your IBM Z(R) environment meets the following installation requirements.

For a cluster that contains user-provisioned infrastructure, you must deploy all of the required machines.

## Required machines for cluster installation {#installation-machine-requirements_installing-ibm-z-reqs}

You must specify the minimum required machines or hosts for your cluster so that your cluster remains stable if a node fails.

The smallest OpenShift Container Platform clusters require the following hosts:

:::warning

For a cluster that has user-provisioned infrastructure, you must deploy all of the required machines.

:::

**Minimum required hosts**

| Hosts | Description |
| --- | --- |
| One temporary bootstrap machine | The cluster requires the bootstrap machine to deploy the OpenShift Container Platform cluster on the three control plane machines. You can remove the bootstrap machine after you install the cluster. |
| Three control plane machines | The control plane machines run the Kubernetes and OpenShift Container Platform services that form the control plane. |
| At least two compute machines, which are also known as worker machines. | The workloads requested by OpenShift Container Platform users run on the compute machines. |

:::warning

To improve high availability of your cluster, distribute the control plane machines over different hypervisor instances on at least two physical machines.

:::

The bootstrap, control plane, and compute machines must use Red Hat Enterprise Linux CoreOS (RHCOS) as the operating system.

RHCOS is based on Red Hat Enterprise Linux (RHEL) 9.8 and inherits all of its hardware certifications and requirements. See [Red Hat Enterprise Linux technology capabilities and limits](https://access.redhat.com/articles/rhel-limits).

### Minimum resource requirements for cluster installation {#installation-minimum-resource-requirements_installing-ibm-z-reqs}

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
  <td>N/A</td>
</tr>
<tr>
  <td>Control plane</td>
  <td>RHCOS</td>
  <td>4</td>
  <td>16 GB</td>
  <td>100 GB</td>
  <td>N/A</td>
</tr>
<tr>
  <td>Compute</td>
  <td>RHCOS</td>
  <td>2</td>
  <td>8 GB</td>
  <td>100 GB</td>
  <td>N/A</td>
</tr>
</tbody>
</table>

- One physical core (IFL) provides two logical cores (threads) when SMT-2 is enabled. The hypervisor can provide two or more vCPUs.

:::note

In OpenShift Container Platform version 4.22, RHCOS uses RHEL version 9.8, which updates the micro-architecture requirements. Each architecture requires the following minimum instruction set architectures (ISA):

- x86-64 architecture requires x86-64-v2 ISA
- ARM64 architecture requires ARMv8.0-A ISA
- ppc64le architecture requires IBM(R) Power9 ISA
- s390x architecture requires IBM(R) z14 ISA

For more information, see [Architectures](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html-single/9.8_release_notes/index#architectures) in the RHEL documentation.

:::

If an instance type for your platform meets the minimum requirements for cluster machines, it is supported to use in OpenShift Container Platform.

**Additional resources**

- [Bridging a HiperSockets LAN with a z/VM Virtual Switch (IBM(R) Documentation)](https://www.ibm.com/docs/en/zvm/latest?topic=networks-bridging-hipersockets-lan-zvm-virtual-switch)
- [Scaling HyperPAV alias devices on Linux guests on z/VM](https://public.dhe.ibm.com/software/dw/linux390/perf/zvm_hpav00.pdf)
- [Processors Resource/Systems Manager Planning Guide (IBM(R) Documentation)](https://www.ibm.com/docs/en/systems-hardware/zsystems/3932-A02?topic=library-prsm-planning-guide)
- [IBM Dynamic Partition Manager (DPM) Guide (IBM(R) Documentation)](https://www.ibm.com/docs/en/systems-hardware/zsystems/3932-A02?topic=library-dynamic-partition-manager-dpm-guide)
- [Topics in LPAR performance](https://www.vm.ibm.com/library/presentations/lparperf.pdf)
- [Recommended host practices for IBM Z(R) & IBM(R) LinuxONE environments](/docs/scalability_and_performance/ibm-z-recommended-host-practices#ibm-z-recommended-host-practices)

### Minimum IBM Z system environment {#minimum-ibm-z-system-requirements_installing-ibm-z-reqs}

To install OpenShift Container Platform on IBM Z(R) or IBM(R) LinuxONE, your environment must meet minimum hardware, operating system, network, and storage requirements.

**Supported IBM(R) hardware**

|  | z/VM | LPAR | RHEL KVM |
| --- | --- | --- | --- |
| IBM(R) z17 (all models) | supported | supported | supported |
| IBM(R) z16 (all models) | supported | supported | supported |
| IBM(R) z15 (all models) | supported | supported | supported |
| IBM(R) z14 (all models) | supported | supported | supported |
| IBM(R) LinuxONE 4 (all models) | supported | supported | supported |
| IBM(R) LinuxONE 5 (all models) | supported | supported | supported |
| IBM(R) LinuxONE III (all models) | supported | supported | supported |
| IBM(R) LinuxONE Emperor II | supported | supported | supported |
| IBM(R) LinuxONE Rockhopper II | supported | supported | supported |

When running OpenShift Container Platform on IBM Z(R) in an LPAR without a hypervisor, use the Dynamic Partition Manager (DPM) to manage your machine.

The RHEL KVM host in your environment must meet certain requirements to host the virtual machines that you plan for the OpenShift Container Platform environment. See [Enabling virtualization on IBM Z(R)](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_and_managing_virtualization/assembly_enabling-virtualization-in-rhel-9_configuring-and-managing-virtualization#enabling-virtualization-on-ibm-z_assembly_enabling-virtualization-in-rhel-9).

:::note

For detailed system requirements, see [Linux on IBM Z(R)/IBM(R) LinuxONE tested platforms](https://www.ibm.com/support/pages/linux-ibm-zibm-linuxone-tested-platforms) (IBM Support).

:::

#### Hardware requirements {#ibm-z-hardware-requirements_installing-ibm-z-reqs}

- The equivalent of six Integrated Facilities for Linux (IFL), which are SMT2 enabled, for each cluster.
- At least one network connection to both connect to the `LoadBalancer` service and to serve data for traffic outside the cluster.

:::warning

- You can use dedicated or shared IFLs to assign sufficient compute resources. Resource sharing is one of the key strengths of IBM Z(R). However, you must adjust the capacity correctly on each hypervisor layer and ensure that there are sufficient resources for every OpenShift Container Platform cluster.
- Since the overall performance of the cluster can be impacted, the LPARs that are used to set up the OpenShift Container Platform clusters must provide sufficient compute capacity. In this context, LPAR weight management, entitlements, and CPU shares on the hypervisor level play an important role. For more information, see "Recommended host practices for IBM Z & IBM LinuxONE environments".

:::

#### IBM Z operating system requirements {#ibm-z-operating-system-requirements_installing-ibm-z-reqs}

**Operating system requirements**

|  | z/VM | LPAR | RHEL KVM |
| --- | --- | --- | --- |
| Hypervisor | One instance of z/VM 7.2 or later | IBM(R) z14 or later with DPM or PR/SM | One LPAR running on RHEL 8.6 or later with KVM, which is managed by libvirt |
| OpenShift Container Platform control plane machines | Three guest virtual machines | Three LPARs | Three guest virtual machines |
| OpenShift Container Platform compute machines | Two guest virtual machines | Two LPARs | Two guest virtual machines |
| Temporary OpenShift Container Platform bootstrap machine | One machine | One machine | One machine |

#### IBM Z network connectivity {#ibm-z-network-connectivity_installing-ibm-z-reqs}

**Network connectivity requirements**

|  | z/VM | LPAR | RHEL KVM |
| --- | --- | --- | --- |
| Network Interface Card (NIC) | One single z/VM virtual NIC in layer 2 mode | - | - |
| Virtual switch (vSwitch) | z/VM VSWITCH in layer 2 Ethernet mode | - | - |
| Network adapter | Direct-attached OSA, RoCE, or HiperSockets | Direct-attached OSA, RoCE, or HiperSockets | A RHEL KVM host configured with OSA, RoCE, or HiperSockets<br>Either a RHEL KVM host that is configured to use bridged networking in libvirt or MacVTap to connect the network to the guests.<br>See [Types of virtual network connections](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/configuring_and_managing_virtualization/index#types-of-virtual-machine-network-connections_configuring-virtual-machine-network-connections). |

#### Disk storage {#ibm-z-disk-storage_installing-ibm-z-reqs}

**Disk storage requirements**

|  | z/VM | LPAR | RHEL KVM |
| --- | --- | --- | --- |
| Fibre Connection (FICON) | z/VM minidisks, fullpack minidisks, or dedicated DASDs, all of which must be formatted as CDL, which is the default. To reach the minimum required DASD size for Red Hat Enterprise Linux CoreOS (RHCOS) installations, you need extended address volumes (EAV). If available, use HyperPAV to ensure optimal performance. | Dedicated DASDs that must be formatted as CDL, which is the default. To reach the minimum required DASD size for Red Hat Enterprise Linux CoreOS (RHCOS) installations, you need extended address volumes (EAV). If available, use HyperPAV to ensure optimal performance. | Virtual block device |
| Fibre Channel Protocol (FCP) | Dedicated FCP or EDEV | Dedicated FCP or EDEV | Virtual block device |
| QCOW | Not supported | Not supported | Supported |
| NVMe | Not supported | Supported | Virtual block device |

### Preferred IBM Z system environment {#preferred-ibm-z-system-requirements_installing-ibm-z-reqs}

While OpenShift Container Platform runs on the minimum IBM Z(R) hardware requirements, using the preferred system environment improves performance and supports production workloads.

#### Hardware requirements {#preferred-ibm-z-hardware-requirements_installing-ibm-z-reqs}

- Three logical partitions (LPARs) that each have the equivalent of six Integrated Facilities for Linux (IFLs), which are SMT2 enabled, for each cluster.
- Two network connections to both connect to the `LoadBalancer` service and to serve data for traffic outside the cluster.
- HiperSockets that are attached to a node directly as a device. To directly connect HiperSockets to a node, you must set up a gateway to the external network via a RHEL 8 guest to bridge to the HiperSockets network.
  :::note

  When installing in a z/VM environment, you can also bridge HiperSockets with one z/VM VSWITCH to be transparent to the z/VM guest.

  :::

#### IBM Z operating system requirements {#preferred-ibm-z-operating-system-requirements_installing-ibm-z-reqs}

**Operating system requirements**

|  | z/VM | LPAR | RHEL KVM |
| --- | --- | --- | --- |
| Hypervisor | One instance of z/VM 7.2 or later | IBM(R) z14 or later with DPM or PR/S | One LPAR running on RHEL 8.6 or later with KVM, which is managed by libvirt |
| OpenShift Container Platform control plane machines | Three guest virtual machines | Three LPARs | Three guest virtual machines |
| OpenShift Container Platform compute machines | Six guest virtual machines | Six LPARs | Six guest virtual machines |
| Temporary OpenShift Container Platform bootstrap machine | One machine | One machine | One machine |

:::note

When working in a z/VM environment, to ensure the availability of integral components in an overcommitted environment, increase the priority of the control plane by using the CP command `SET SHARE`. Do the same for infrastructure nodes, if they exist. See [SET SHARE](https://www.ibm.com/docs/en/zvm/latest?topic=commands-set-share) (IBM(R) Documentation).

:::

**Additional resources**

- [Optimizing storage](/docs/scalability_and_performance/optimization/optimizing-storage#optimizing-storage)

### Certificate signing requests management {#csr-management_installing-ibm-z-reqs}

On user-provisioned infrastructure, you must implement a mechanism for approving cluster certificate signing requests (CSRs) after installation when your cluster has limited access to automatic machine management.

The `kube-controller-manager` only approves the kubelet client CSRs. The `machine-approver` cannot guarantee the validity of a serving certificate that kubelet credentials request because it cannot confirm that the correct machine issued the request. You must find and implement a method of verifying the validity of the kubelet serving certificate requests and approving them.

### Networking requirements for user-provisioned infrastructure {#installation-network-user-infra_installing-ibm-z-reqs}

You must configure networking for all the Red Hat Enterprise Linux CoreOS (RHCOS) machines in `initramfs` during boot, so that they can fetch their Ignition config files.

During the initial boot, the machines require an IP address configuration that is set either through a DHCP server or statically by providing the required boot options. After a network connection is established, the machines download their Ignition config files from an HTTP or HTTPS server. The Ignition config files are then used to set the exact state of each machine. The Machine Config Operator completes more changes to the machines, such as the application of new certificates or keys, after installation.

:::note

- Consider using a DHCP server for long-term management of the cluster machines. Ensure that the DHCP server is configured to provide persistent IP addresses, DNS server information, and hostnames to the cluster machines.
- If a DHCP service is not available for your user-provisioned infrastructure, you can instead provide the IP networking configuration and the address of the DNS server to the nodes at RHCOS install time. These can be passed as boot arguments if you are installing from an ISO image. See the *Installing RHCOS and starting the OpenShift Container Platform bootstrap process* section for more information about static IP provisioning and advanced networking options.

:::

The Kubernetes API server must be able to resolve the node names of the cluster machines. If the API servers and worker nodes are in different zones, you can configure a default DNS search zone to allow the API server to resolve the node names. Another supported approach is to always refer to hosts by their fully-qualified domain names in both the node objects and all DNS requests.

#### Setting the cluster node hostnames through DHCP {#installation-host-names-dhcp-user-infra_installing-ibm-z-reqs}

On Red Hat Enterprise Linux CoreOS (RHCOS) machines, the hostname is set through NetworkManager. By default, the machines obtain their hostname through DHCP. If the hostname is not provided by DHCP, set statically through kernel arguments, or another method, it is obtained through a reverse DNS lookup. Reverse DNS lookup occurs after the network has been initialized on a node and can take time to resolve. Other system services can start before this and detect the hostname as `localhost` or similar. You can avoid this by using DHCP to provide the hostname for each cluster node.

Additionally, setting the hostnames through DHCP can bypass any manual DNS record name configuration errors in environments that have a DNS split-horizon implementation.

#### Network connectivity requirements {#installation-network-connectivity-user-infra_installing-ibm-z-reqs}

You must configure the network connectivity between machines to allow OpenShift Container Platform cluster components to communicate. Each machine must be able to resolve the hostnames of all other machines in the cluster.

This section provides details about the ports that are required.

:::warning

In connected OpenShift Container Platform environments, all nodes are required to have internet access to pull images for platform containers and provide telemetry data to Red Hat.

:::

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
  <td rowspan="4">TCP</td>
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
  <td><code>22623</code></td>
  <td>The port handles traffic from the Machine Config Server and directs the traffic to the control plane machines.</td>
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
  <td>Network Time Protocol (NTP) on UDP port <code>123</code>. If an external NTP time server is configured, you must open UDP port <code>123</code>.</td>
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

**Ports used for all-machine to control plane communications**

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
  <td><code>6443</code></td>
  <td>Kubernetes API</td>
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

#### NTP configuration for user-provisioned infrastructure {#_ntp_configuration_for_user-provisioned_infrastructure}

OpenShift Container Platform clusters are configured to use a public Network Time Protocol (NTP) server by default. If you want to use a local enterprise NTP server, or if your cluster is being deployed in a disconnected network, you can configure the cluster to use a specific time server. For more information, see the documentation for *Configuring chrony time service*.

If a DHCP server provides NTP server information, the chrony time service on the Red Hat Enterprise Linux CoreOS (RHCOS) machines read the information and can sync the clock with the NTP servers.

**Additional resources**

- [Configuring chrony time service](/docs/installing/install_config/installing-customizing#installation-special-config-chrony_installing-customizing)

### User-provisioned DNS requirements {#installation-dns-user-infra_installing-ibm-z-reqs}

In OpenShift Container Platform deployments, you must ensure that cluster components meet certain DNS name resolution criteria for internal communication, certificate validation, and automated node discovery purposes.

The following is a list of required cluster components:

- The Kubernetes API
- The OpenShift Container Platform application wildcard
- The bootstrap and control plane machines
- The compute machines

Reverse DNS resolution is also required for the Kubernetes API, the bootstrap machine, the control plane machines, and the compute machines.

DNS A/AAAA or CNAME records are used for name resolution and PTR records are used for reverse name resolution. The reverse records are important because Red Hat Enterprise Linux CoreOS (RHCOS) uses the reverse records to set the hostnames for all the nodes, unless the hostnames are provided by DHCP. Additionally, the reverse records are used to generate the certificate signing requests (CSR) that OpenShift Container Platform needs to operate.

:::note

It is recommended to use a DHCP server to provide the hostnames to each cluster node. See the *DHCP recommendations for user-provisioned infrastructure* section for more information.

:::

The following DNS records are required for a user-provisioned OpenShift Container Platform cluster and they must be in place before installation. In each record, `<cluster_name>` is the cluster name and `<base_domain>` is the base domain that you specify in the `install-config.yaml` file. A complete DNS record takes the form: `<component>.<cluster_name>.<base_domain>.`.

**Required DNS records**

<table>
<thead>
<tr>
  <th>Component</th>
  <th>Record</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td rowspan="2">Kubernetes API</td>
  <td><code>api.&lt;cluster_name&gt;.&lt;base_domain&gt;.</code></td>
  <td>A DNS A/AAAA or CNAME record, and a DNS PTR record, to identify the API load balancer. These records must be resolvable by both clients external to the cluster and from all the nodes within the cluster.</td>
</tr>
<tr>
  <td><code>api-int.&lt;cluster_name&gt;.&lt;base_domain&gt;.</code></td>
  <td>A DNS A/AAAA or CNAME record, and a DNS PTR record, to internally identify the API load balancer. These records must be resolvable from all the nodes within the cluster.<div class="alert alert--warning"><strong>Important</strong><p>The API server must be able to resolve the worker nodes by the hostnames that are recorded in Kubernetes. If the API server cannot resolve the node names, then proxied API calls can fail, and you cannot retrieve logs from pods.</p></div></td>
</tr>
<tr>
  <td>Routes</td>
  <td><code>*.apps.&lt;cluster_name&gt;.&lt;base_domain&gt;.</code></td>
  <td>A wildcard DNS A/AAAA or CNAME record that refers to the application ingress load balancer. The application ingress load balancer targets the machines that run the Ingress Controller pods.   The Ingress Controller pods run on the compute machines by default.  These records must be resolvable by both clients external to the cluster and from all the nodes within the cluster.<br><br>For example, <code>console-openshift-console.apps.&lt;cluster_name&gt;.&lt;base_domain&gt;</code> is used as a wildcard route to the OpenShift Container Platform console.</td>
</tr>
<tr>
  <td>Bootstrap machine</td>
  <td><code>bootstrap.&lt;cluster_name&gt;.&lt;base_domain&gt;.</code></td>
  <td>A DNS A/AAAA or CNAME record, and a DNS PTR record, to identify the bootstrap machine. These records must be resolvable by the nodes within the cluster.</td>
</tr>
<tr>
  <td>Control plane machines</td>
  <td><code>&lt;control_plane&gt;&lt;n&gt;.&lt;cluster_name&gt;.&lt;base_domain&gt;.</code></td>
  <td>DNS A/AAAA or CNAME records and DNS PTR records to identify each machine for the control plane nodes. These records must be resolvable by the nodes within the cluster.</td>
</tr>
<tr>
  <td>Compute machines</td>
  <td><code>&lt;compute&gt;&lt;n&gt;.&lt;cluster_name&gt;.&lt;base_domain&gt;.</code></td>
  <td>DNS A/AAAA or CNAME records and DNS PTR records to identify each machine for the worker nodes. These records must be resolvable by the nodes within the cluster.</td>
</tr>
</tbody>
</table>

:::note

In OpenShift Container Platform 4.4 and later, you do not need to specify etcd host and SRV records in your DNS configuration.

:::

:::tip

You can use the `dig` command to verify name and reverse name resolution. See the section on *Validating DNS resolution for user-provisioned infrastructure* for detailed validation steps.

:::

#### Example DNS configuration for user-provisioned clusters {#installation-dns-user-infra-example_installing-ibm-z-reqs}

Reference the example DNS configurations to understand how A and PTR record configuration samples meet the DNS requirements for deploying OpenShift Container Platform on user-provisioned infrastructure.

The DNS configuration examples provided here are for reference only and are not meant to provide advice for choosing one DNS solution over another.

In the examples, the cluster name is `ocp4` and the base domain is `example.com`.

The following example is a BIND zone file that shows sample DNS A records for name resolution in a user-provisioned cluster.

:::note

In the example, the same load balancer is used for the Kubernetes API and application ingress traffic. In production scenarios, you can deploy the API and application ingress load balancers separately so that you can scale the load balancer infrastructure for each in isolation.

:::

```text
$TTL 1W
@	IN	SOA	ns1.example.com.	root (
			2019070700	; serial
			3H		; refresh (3 hours)
			30M		; retry (30 minutes)
			2W		; expiry (2 weeks)
			1W )		; minimum (1 week)
	IN	NS	ns1.example.com.
	IN	MX 10	smtp.example.com.
;
;
ns1.example.com.		IN	A	192.168.1.5
smtp.example.com.		IN	A	192.168.1.5
;
helper.example.com.		IN	A	192.168.1.5
helper.ocp4.example.com.	IN	A	192.168.1.5
;
api.ocp4.example.com.		IN	A	192.168.1.5
api-int.ocp4.example.com.	IN	A	192.168.1.5
;
*.apps.ocp4.example.com.	IN	A	192.168.1.5
;
bootstrap.ocp4.example.com.	IN	A	192.168.1.96
;
control-plane0.ocp4.example.com.	IN	A	192.168.1.97
control-plane1.ocp4.example.com.	IN	A	192.168.1.98
;
control-plane2.ocp4.example.com.	IN	A	192.168.1.99
;
compute0.ocp4.example.com.	IN	A	192.168.1.11
compute1.ocp4.example.com.	IN	A	192.168.1.7
;
;EOF
```

where:

`api.ocp4.example.com.`
: Provides name resolution for the Kubernetes API. The record refers to the IP address of the API load balancer.

`api-int.ocp4.example.com.`
: Provides name resolution for the Kubernetes API. The record refers to the IP address of the API load balancer and is used for internal cluster communications.

`*.apps.ocp4.example.com.`
: Provides name resolution for the wildcard routes. The record refers to the IP address of the application ingress load balancer. The application ingress load balancer targets the machines that run the Ingress Controller pods.

`bootstrap.ocp4.example.com`
: Provides name resolution for the bootstrap machine.

`control-plane0.ocp4.example.com`
: Provides name resolution for the control plane machines.

`compute0.ocp4.example.com.`
: Provides name resolution for the compute machines.

The following example BIND zone file shows sample PTR records for reverse name resolution in a user-provisioned cluster:

```text
$TTL 1W
@	IN	SOA	ns1.example.com.	root (
			2019070700	; serial
			3H		; refresh (3 hours)
			30M		; retry (30 minutes)
			2W		; expiry (2 weeks)
			1W )		; minimum (1 week)
	IN	NS	ns1.example.com.
;
5.1.168.192.in-addr.arpa.	IN	PTR	api.ocp4.example.com.
5.1.168.192.in-addr.arpa.	IN	PTR	api-int.ocp4.example.com.
;
96.1.168.192.in-addr.arpa.	IN	PTR	bootstrap.ocp4.example.com.
;
97.1.168.192.in-addr.arpa.	IN	PTR	control-plane0.ocp4.example.com.
98.1.168.192.in-addr.arpa.	IN	PTR	control-plane1.ocp4.example.com.
;
99.1.168.192.in-addr.arpa.	IN	PTR	control-plane2.ocp4.example.com.
;
11.1.168.192.in-addr.arpa.	IN	PTR	compute0.ocp4.example.com.
7.1.168.192.in-addr.arpa.	IN	PTR	compute1.ocp4.example.com.
;
;EOF
```

where:

`api.ocp4.example.com.`
: Provides reverse DNS resolution for the Kubernetes API. The PTR record refers to the record name of the API load balancer.

`api-int.ocp4.example.com.`
: Provides reverse DNS resolution for the Kubernetes API. The PTR record refers to the record name of the API load balancer and is used for internal cluster communications.

`bootstrap.ocp4.example.com.`
: Provides reverse DNS resolution for the bootstrap machine.

`control-plane0.ocp4.example.com.`
: Provides rebootstrap.ocp4.example.com.verse DNS resolution for the control plane machines.

`compute0.ocp4.example.com.`
: Provides reverse DNS resolution for the compute machines.

:::note

A PTR record is not required for the OpenShift Container Platform application wildcard.

:::

### Load balancing requirements for user-provisioned infrastructure {#installation-load-balancing-user-infra_installing-ibm-z-reqs}

Before you install OpenShift Container Platform, you must provision the API and application Ingress load balancing infrastructure. In production scenarios, you can deploy the API and application Ingress load balancers separately so that you can scale the load balancer infrastructure for each in isolation.

:::note

If you want to deploy the API and application Ingress load balancers with a Red Hat Enterprise Linux (RHEL) instance, you must purchase the RHEL subscription separately.

:::

The load balancing infrastructure must meet the following requirements:

- API load balancer: Provides a common endpoint for users, both human and machine, to interact with and configure the platform. Configure the following conditions:
  - Layer 4 load balancing only. This can be referred to as Raw TCP or SSL Passthrough mode.
  - A stateless load balancing algorithm. The options vary based on the load balancer implementation.

  :::warning

  Do not configure session persistence for an API load balancer. Configuring session persistence for a Kubernetes API server might cause performance issues from excess application traffic for your OpenShift Container Platform cluster and the Kubernetes API that runs inside the cluster.

  :::

Configure the following ports on both the front and back of the API load balancers:

| Port | Back-end machines (pool members) | Internal | External | Description |
| --- | --- | --- | --- | --- |
| `6443` | Bootstrap and control plane. You remove the bootstrap machine from the load balancer after the bootstrap machine initializes the cluster control plane. You must configure the `/readyz` endpoint for the API server health check probe. | X | X | Kubernetes API server |
| `22623` | Bootstrap and control plane. You remove the bootstrap machine from the load balancer after the bootstrap machine initializes the cluster control plane. | X |  | Machine config server |

:::note

The load balancer must be configured to take a maximum of 30 seconds from the time the API server turns off the `/readyz` endpoint to the removal of the API server instance from the pool. Within the time frame after `/readyz` returns an error or becomes healthy, the endpoint must have been removed or added. Probing every 5 or 10 seconds, with two successful requests to become healthy and three to become unhealthy, are well-tested values.

:::

- Application Ingress load balancer: Provides an ingress point for application traffic flowing in from outside the cluster. A working configuration for the Ingress router is required for an OpenShift Container Platform cluster. Configure the following conditions:
  - Layer 4 load balancing only. This can be referred to as Raw TCP or SSL Passthrough mode.
  - A connection-based or session-based persistence is recommended, based on the options available and types of applications that will be hosted on the platform.

  :::tip

  If the true IP address of the client can be seen by the application Ingress load balancer, enabling source IP-based session persistence can improve performance for applications that use end-to-end TLS encryption.

  :::

Configure the following ports on both the front and back of the load balancers:

**Application Ingress load balancer**

| Port | Back-end machines (pool members) | Internal | External | Description |
| --- | --- | --- | --- | --- |
| `443` | The machines that run the Ingress Controller pods, compute, or worker, by default. | X | X | HTTPS traffic |
| `80` | The machines that run the Ingress Controller pods, compute, or worker, by default. | X | X | HTTP traffic |

:::note

If you are deploying a three-node cluster with zero compute nodes, the Ingress Controller pods run on the control plane nodes. In three-node cluster deployments, you must configure your application Ingress load balancer to route HTTP and HTTPS traffic to the control plane nodes.

:::
