---
title: Installing a user-provisioned bare metal cluster on a disconnected environment
sidebar_position: 3
---

# Installing a user-provisioned bare metal cluster on a disconnected environment {#installing-restricted-networks-bare-metal}

<a id="installing-restricted-networks-bare-metal"></a>

In OpenShift Container Platform 4.22, you can install a cluster on bare metal infrastructure that you provision in a restricted network.

:::warning

While you might be able to follow this procedure to deploy a cluster on virtualized or cloud environments, you must be aware of additional considerations for non-bare metal platforms. Review the information in the [guidelines for deploying OpenShift Container Platform on non-tested platforms](https://access.redhat.com/articles/4207611) before you attempt to install an OpenShift Container Platform cluster in such an environment.

:::

## Prerequisites {#_prerequisites}

- You reviewed details about the [OpenShift Container Platform installation and update](/docs/architecture/architecture-installation#architecture-installation) processes.
- You read the documentation on [selecting a cluster installation method and preparing it for users](/docs/installing/overview/installing-preparing#installing-preparing).
- You [created a registry on your mirror host](/docs/disconnected/installing-mirroring-installation-images#installing-mirroring-installation-images) and obtained the `imageContentSources` data for your version of OpenShift Container Platform.
  :::warning

  Because the installation media is on the mirror host, you can use that computer to complete all installation steps.

  :::
- You provisioned [persistent storage](/docs/storage/understanding-persistent-storage#understanding-persistent-storage) for your cluster. To deploy a private image registry, your storage must provide ReadWriteMany access modes.
- If you use a firewall and plan to use the Telemetry service, you [configured the firewall to allow the sites](/docs/installing/install_config/configuring-firewall#configuring-firewall-module_configuring-firewall) that your cluster requires access to.
  :::note

  Be sure to also review this site list if you are configuring a proxy.

  :::

## About installations in restricted networks {#installation-about-restricted-networks_installing-restricted-networks-bare-metal}

You can install OpenShift Container Platform 4.22 in a restricted network without an active internet connection to obtain software components. Restricted network installations can use installer-provisioned or user-provisioned infrastructure, depending on the cloud platform to which you are installing the cluster.

If you choose to perform a restricted network installation on a cloud platform, you still require access to its cloud APIs. Some cloud functions, such as Amazon Web Service’s Route 53 DNS and IAM services, require internet access. Depending on your network, you might require less internet access for an installation on bare-metal hardware, Nutanix, or on VMware vSphere.

To complete a restricted network installation, you must create a registry that mirrors the contents of the OpenShift image registry and contains the installation media. You can create this registry on a mirror host, which can access both the internet and your closed network, or by using other methods that meet your restrictions.

:::warning

Because of the complexity of the configuration for user-provisioned installations, consider completing a standard user-provisioned infrastructure installation before you try a restricted network installation using user-provisioned infrastructure. Completing this test installation might make it easier to isolate and troubleshoot any issues that might arise during your installation in a restricted network.

:::

### Additional limits {#installation-restricted-network-limits_installing-restricted-networks-bare-metal}

Clusters in restricted networks have the following additional limitations and restrictions:

- The `ClusterVersion` status includes an `Unable to retrieve available updates` error.
- By default, you cannot use the contents of the Developer Catalog because you cannot access the required image stream tags.

## Internet access for OpenShift Container Platform {#cluster-entitlements_installing-restricted-networks-bare-metal}

In OpenShift Container Platform 4.22, you require access to the internet to obtain the images that are necessary to install your cluster.

You must have internet access to perform the following actions:

- Access Red Hat Hybrid Cloud Console to download the installation program and perform subscription management. If the cluster has internet access and you do not disable Telemetry, that service automatically entitles your cluster.
- Access Quay.io to obtain the packages that are required to install your cluster.
- Obtain the packages that are required to perform cluster updates.

## Requirements for a cluster with user-provisioned infrastructure {#installation-requirements-user-infra_installing-restricted-networks-bare-metal}

For a cluster that contains user-provisioned infrastructure, you must deploy all of the required machines.

This section describes the requirements for deploying OpenShift Container Platform on user-provisioned infrastructure.

### Required machines for cluster installation {#installation-machine-requirements_installing-restricted-networks-bare-metal}

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

:::note

As an exception, you can run zero compute machines in a bare metal cluster that consists of three control plane machines only. This provides smaller, more resource efficient clusters for cluster administrators and developers to use for testing, development, and production. Running one compute machine is not supported.

:::

:::warning

To maintain high availability of your cluster, use separate physical hosts for these cluster machines.

:::

The bootstrap and control plane machines must use Red Hat Enterprise Linux CoreOS (RHCOS) as the operating system. However, the compute machines can use Red Hat Enterprise Linux CoreOS (RHCOS), Red Hat Enterprise Linux (RHEL) 8.6 and later.

RHCOS is based on Red Hat Enterprise Linux (RHEL) 9.8 and inherits all of its hardware certifications and requirements. See [Red Hat Enterprise Linux technology capabilities and limits](https://access.redhat.com/articles/rhel-limits).

### Minimum resource requirements for cluster installation {#installation-minimum-resource-requirements_installing-restricted-networks-bare-metal}

To ensure that your OpenShift Container Platform cluster runs as expected, each cluster machine must meet minimum CPU, memory, and storage requirements.

**Minimum resource requirements**

<table>
<thead>
<tr>
  <th>Machine</th>
  <th>Operating system</th>
  <th>CPU</th>
  <th>RAM</th>
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

- One CPU is equal to one physical core when simultaneous multithreading (SMT), or Hyper-Threading, is not enabled. When enabled, use the following formula to calculate the corresponding ratio: (threads per core × cores) × sockets = CPUs.
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

If an instance type for your platform meets the minimum requirements for cluster machines, it is supported to use in OpenShift Container Platform.

**Additional resources**

- [Optimizing storage](/docs/scalability_and_performance/optimization/optimizing-storage#optimizing-storage)

### Certificate signing requests management {#csr-management_installing-restricted-networks-bare-metal}

On user-provisioned infrastructure, you must implement a mechanism for approving cluster certificate signing requests (CSRs) after installation when your cluster has limited access to automatic machine management.

The `kube-controller-manager` only approves the kubelet client CSRs. The `machine-approver` cannot guarantee the validity of a serving certificate that kubelet credentials request because it cannot confirm that the correct machine issued the request. You must find and implement a method of verifying the validity of the kubelet serving certificate requests and approving them.

**Additional resources**

- See [Configuring a three-node cluster](/docs/installing/installing_bare_metal/upi/installing-restricted-networks-bare-metal#installation-three-node-cluster_installing-restricted-networks-bare-metal) for details about deploying three-node clusters in bare metal environments.
- See [Approving the certificate signing requests for your machines](/docs/installing/installing_bare_metal/upi/installing-restricted-networks-bare-metal#installation-approve-csrs_installing-restricted-networks-bare-metal) for more information about approving cluster certificate signing requests after installation.

### Networking requirements for user-provisioned infrastructure {#installation-network-user-infra_installing-restricted-networks-bare-metal}

You must configure networking for all the Red Hat Enterprise Linux CoreOS (RHCOS) machines in `initramfs` during boot, so that they can fetch their Ignition config files.

During the initial boot, the machines require an IP address configuration that is set either through a DHCP server or statically by providing the required boot options. After a network connection is established, the machines download their Ignition config files from an HTTP or HTTPS server. The Ignition config files are then used to set the exact state of each machine. The Machine Config Operator completes more changes to the machines, such as the application of new certificates or keys, after installation.

:::note

- Consider using a DHCP server for long-term management of the cluster machines. Ensure that the DHCP server is configured to provide persistent IP addresses, DNS server information, and hostnames to the cluster machines.
- If a DHCP service is not available for your user-provisioned infrastructure, you can instead provide the IP networking configuration and the address of the DNS server to the nodes at RHCOS install time. These can be passed as boot arguments if you are installing from an ISO image. See the *Installing RHCOS and starting the OpenShift Container Platform bootstrap process* section for more information about static IP provisioning and advanced networking options.

:::

The Kubernetes API server must be able to resolve the node names of the cluster machines. If the API servers and worker nodes are in different zones, you can configure a default DNS search zone to allow the API server to resolve the node names. Another supported approach is to always refer to hosts by their fully-qualified domain names in both the node objects and all DNS requests.

#### Setting the cluster node hostnames through DHCP {#installation-host-names-dhcp-user-infra_installing-restricted-networks-bare-metal}

On Red Hat Enterprise Linux CoreOS (RHCOS) machines, the hostname is set through NetworkManager. By default, the machines obtain their hostname through DHCP. If the hostname is not provided by DHCP, set statically through kernel arguments, or another method, it is obtained through a reverse DNS lookup. Reverse DNS lookup occurs after the network has been initialized on a node and can take time to resolve. Other system services can start before this and detect the hostname as `localhost` or similar. You can avoid this by using DHCP to provide the hostname for each cluster node.

Additionally, setting the hostnames through DHCP can bypass any manual DNS record name configuration errors in environments that have a DNS split-horizon implementation.

#### Network connectivity requirements {#installation-network-connectivity-user-infra_installing-restricted-networks-bare-metal}

You must configure the network connectivity between machines to allow OpenShift Container Platform cluster components to communicate. Each machine must be able to resolve the hostnames of all other machines in the cluster.

This section provides details about the ports that are required.

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

### User-provisioned DNS requirements {#installation-dns-user-infra_installing-restricted-networks-bare-metal}

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

#### Example DNS configuration for user-provisioned clusters {#installation-dns-user-infra-example_installing-restricted-networks-bare-metal}

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

**Additional resources**

- [Validating DNS resolution for user-provisioned infrastructure](/docs/installing/installing_bare_metal/upi/installing-restricted-networks-bare-metal#installation-user-provisioned-validating-dns_installing-restricted-networks-bare-metal)

### Configuring the dnsRecordsType parameter {#installation-bare-metal-dns-record-type_installing-restricted-networks-bare-metal}

During cluster installation, you can specify the `dnsRecordsType` parameter in the `install-config.yaml` file to set if the internal DNS service or an external source provides the necessary records for `api`, `api-int`, and `ingress` DNS records.

:::warning

Configuring the dnsRecordsType parameter is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.

For more information about the support scope of Red Hat Technology Preview features, see [Technology Preview Features Support Scope](https://access.redhat.com/support/offerings/techpreview/).

:::

The `dnsRecordsType` parameter supports the following values:

- `Internal`: The default value. Setting this value causes the cluster infrastructure to automatically create and maintain the necessary DNS records.
- `External`: You can use this value only if you set the `loadBalancer.type` parameter to `UserManaged`. The cluster does not manage the DNS records. You must manually configure DNS records on an external DNS server.

**Prerequisites**

- You created DNS records, such as `api`, `api-int`, or `*.apps`.
- You configured a user-managed load balancer for your cluster.
- If you intend on setting `dnsRecordsType.External` in the `infrastructure.config.openshift.io` CR, you must initially configure cluster nodes to use the specific external server for DNS resolution.

**Procedure**

- In the `install-config.yaml` file during cluster installation, specify `TechPreviewNoUpgrade` for the `featureSet` parameter and specify `External` for the `dnsRecordsType` parameter:
  ```yaml
  apiVersion: v1
  baseDomain: example.com
  metadata:
    name: dev-cluster
  # ...
  platform:
    baremetal:
  # ...
      loadBalancer:
        type: UserManaged
      dnsRecordsType: External
  # ...
  featureSet: TechPreviewNoUpgrade
  pullSecret: '{"auths":{"<local_registry>": {"auth": "<credentials>","email": "you@example.com"}}}'
  sshKey: 'ssh-ed25519 AAAA...'
  # ...
  ```

  where:

  <dl>
  <dt><code>type.UserManaged</code></dt>
  <dd>Specifies an external load balancer for your cluster.</dd>
  <dt><code>dnsRecordsType.External</code></dt>
  <dd>Specifies that the cluster does not create internal DNS records for the core infrastructure.</dd>
  <dt><code>featureSet.TechPreviewNoUpgrade</code></dt>
  <dd>Specifies the enablement of non-default features for your cluster.</dd>
  </dl>

### Load balancing requirements for user-provisioned infrastructure {#installation-load-balancing-user-infra_installing-restricted-networks-bare-metal}

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

#### Example load balancer configuration for user-provisioned clusters {#installation-load-balancing-user-infra-example_installing-restricted-networks-bare-metal}

Reference the example API and application Ingress load balancer configuration so that you can understand how to meet the load balancing requirements for user-provisioned clusters.

The sample is an `/etc/haproxy/haproxy.cfg` configuration for an HAProxy load balancer. The example is not meant to provide advice for choosing one load balancing solution over another.

:::tip

If you are using HAProxy as a load balancer, you can check that the `haproxy` process is listening on ports `6443`, `22623`, `443`, and `80` by running `netstat -nltupe` on the HAProxy node.

:::

In the example, the same load balancer is used for the Kubernetes API and application ingress traffic. In production scenarios, you can deploy the API and application ingress load balancers separately so that you can scale the load balancer infrastructure for each in isolation.

:::note

If you are using HAProxy as a load balancer and SELinux is set to `enforcing`, you must ensure that the HAProxy service can bind to the configured TCP port by running `setsebool -P haproxy_connect_any=1`.

:::

```text title="Sample API and application Ingress load balancer configuration"
global
  log         127.0.0.1 local2
  pidfile     /var/run/haproxy.pid
  maxconn     4000
  daemon
defaults
  mode                    http
  log                     global
  option                  dontlognull
  option http-server-close
  option                  redispatch
  retries                 3
  timeout http-request    10s
  timeout queue           1m
  timeout connect         10s
  timeout client          1m
  timeout server          1m
  timeout http-keep-alive 10s
  timeout check           10s
  maxconn                 3000
listen api-server-6443
  bind *:6443
  mode tcp
  option  httpchk GET /readyz HTTP/1.0
  option  log-health-checks
  balance roundrobin
  server bootstrap bootstrap.ocp4.example.com:6443 verify none check check-ssl inter 10s fall 2 rise 3 backup
  server master0 master0.ocp4.example.com:6443 weight 1 verify none check check-ssl inter 10s fall 2 rise 3
  server master1 master1.ocp4.example.com:6443 weight 1 verify none check check-ssl inter 10s fall 2 rise 3
  server master2 master2.ocp4.example.com:6443 weight 1 verify none check check-ssl inter 10s fall 2 rise 3
listen machine-config-server-22623
  bind *:22623
  mode tcp
  server bootstrap bootstrap.ocp4.example.com:22623 check inter 1s backup
  server master0 master0.ocp4.example.com:22623 check inter 1s
  server master1 master1.ocp4.example.com:22623 check inter 1s
  server master2 master2.ocp4.example.com:22623 check inter 1s
listen ingress-router-443
  bind *:443
  mode tcp
  balance source
  server compute0 compute0.ocp4.example.com:443 check inter 1s
  server compute1 compute1.ocp4.example.com:443 check inter 1s
listen ingress-router-80
  bind *:80
  mode tcp
  balance source
  server compute0 compute0.ocp4.example.com:80 check inter 1s
  server compute1 compute1.ocp4.example.com:80 check inter 1s
```

where:

`listen api-server-6443`
: Port `6443` handles the Kubernetes API traffic and points to the control plane machines. You must configure health checks on this port to ensure that the API server is available before routing traffic.

`server bootstrap bootstrap.ocp4.example.com`
: The bootstrap entries must be in place before the OpenShift Container Platform cluster installation and they must be removed after the bootstrap process is complete.

`listen machine-config-server`
: Port `22623` handles the machine config server traffic and points to the control plane machines.

`listen ingress-router-443`
: Port `443` handles the HTTPS traffic and points to the machines that run the Ingress Controller pods. The Ingress Controller pods run on the compute machines by default.

`listen ingress-router-80`
: Port `80` handles the HTTP traffic and points to the machines that run the Ingress Controller pods. The Ingress Controller pods run on the compute machines by default.

  :::note

  If you are deploying a compact three-node cluster with zero compute nodes, the Ingress Controller pods run on the control plane nodes. In three-node cluster deployments, you must configure your application Ingress load balancer to route HTTP and HTTPS traffic to the control plane nodes.

  :::

## Creating a manifest object that includes a customized br-ex bridge {#creating-manifest-file-customized-br-ex-bridge_installing-restricted-networks-bare-metal}

By default, OpenShift Container Platform automatically configures the Open vSwitch (OVS) `br-ex` bridge on bare-metal nodes. For advanced networking requirements, you can override this default behavior on bare-metal platforms. To do this, create a `MachineConfig` object that includes an NMState configuration file.

:::note

You can use Butane to create machine configs. For more information, see "Creating a MachineConfig object by using Butane".

:::

Consider using the customized `br-ex` bridge configuration for any of the following tasks:

- You need to modify the `br-ex` bridge after you installed the cluster.
- You need to modify the maximum transmission unit (MTU) for your cluster.
- You need to update DNS values.
- You need to modify attributes for a different bond interface. Examples include MIImon (Media Independent Interface Monitor), bonding mode or Quality of Service (QoS).
- You need to enable Link Layer Discovery Protocol (LLDP) to discover and troubleshoot switch connectivity.

:::note

Use the default OVS `br-ex` bridge for standard environments.

Use the default OVS `br-ex` bridge mechanism for single network interface controller (NIC) environments with default network settings.

:::

After you install Red Hat Enterprise Linux CoreOS (RHCOS) and the system reboots, the Machine Config Operator injects Ignition configuration files into each node. This operation ensures that each node receives the `br-ex` bridge network configuration. To prevent configuration conflicts, the default OVS `br-ex` bridge mechanism is disabled.

:::warning

The following list of interface names are reserved and you cannot use the names with NMstate configurations:

- `br-ext`
- `br-int`
- `br-local`
- `br-nexthop`
- `br0`
- `ext-vxlan`
- `ext`
- `genev_sys_*`
- `int`
- `k8s-*`
- `ovn-k8s-*`
- `patch-br-*`
- `tun0`
- `vxlan_sys_*`

:::

**Prerequisites**

- Optional: You have installed the [`nmstatectl`](https://nmstate.io/user/quick_guide.html) CLI tool to validate your NMState configuration.

**Procedure**

1. Create an NMState configuration file and define a customized `br-ex` bridge network configuration in the file:
   ```yaml title="Example of an NMState configuration for a customized br-ex bridge network"
   interfaces:
   - name: enp2s0
     type: ethernet
     state: up
     mtu: 9000
     ipv4:
       enabled: false
     ipv6:
       enabled: false
   - name: br-ex
     type: ovs-bridge
     state: up
     ipv4:
       enabled: false
       dhcp: false
     ipv6:
       enabled: false
       dhcp: false
     bridge:
       options:
         mcast-snooping-enable: true
       port:
       - name: enp2s0
       - name: br-ex
   - name: br-ex
     type: ovs-interface
     state: up
     copy-mac-from: enp2s0
     mtu: 9000
     ipv4:
       enabled: true
       dhcp: true
       auto-route-metric: 48
     ipv6:
       enabled: true
       dhcp: true
       auto-route-metric: 48
   # ...
   ```

   where:

   <dl>
   <dt><code>interfaces.name</code></dt>
   <dd>Specifies the name of the interface.</dd>
   <dt><code>interfaces.type</code></dt>
   <dd>Specifies the type of ethernet.</dd>
   <dt><code>interfaces.state</code></dt>
   <dd>Specifies the requested state for the interface after creation.</dd>
   <dt><code>mtu</code></dt>
   <dd>To ensure network stability and performance, you must explicitly declare the MTU in the manifest for every interface. Do not rely on automatic MTU configuration. The MTU configured on a bridge port or VLAN-tagged interface must not exceed the maximum frame size supported by the attached physical medium. A mismatch causes packet fragmentation or connectivity loss.</dd>
   <dt><code>ipv4.enabled</code></dt>
   <dd>Disables IPv4 and IPv6 in this example.</dd>
   <dt><code>port.name</code></dt>
   <dd>Specifies the node NIC to which the bridge attaches.</dd>
   <dt><code>auto-route-metric</code></dt>
   <dd>Sets the parameter to <code>48</code> to ensure the <code>br-ex</code> default route always has the highest precedence (lowest metric). This configuration prevents routing conflicts with any other interfaces that are automatically configured by the <code>NetworkManager</code> service.</dd>
   </dl>
2. Use the `cat` command to base64-encode the contents of the NMState configuration:
   ```terminal
   $ cat <nmstate_configuration>.yml | base64
   ```

   where:

   <dl>
   <dt><code>&lt;nmstate_configuration&gt;</code></dt>
   <dd>Replace <code>&lt;nmstate_configuration&gt;</code> with the name of your NMState resource YAML file.</dd>
   </dl>
3. Create a `MachineConfig` manifest file and define a customized `br-ex` bridge network configuration analogous to the following example. The installation program automatically applies the updates from the `MachineConfig` object to your cluster.
   ```yaml
   apiVersion: machineconfiguration.openshift.io/v1
   kind: MachineConfig
   metadata:
     labels:
       machineconfiguration.openshift.io/role: worker
     name: 10-br-ex-worker
   spec:
     config:
       ignition:
         version: 3.2.0
       storage:
         files:
         - contents:
             source: data:text/plain;charset=utf-8;base64,<base64_encoded_nmstate_configuration>
           mode: 0644
           overwrite: true
           path: /etc/nmstate/openshift/worker-0.yml
         - contents:
             source: data:text/plain;charset=utf-8;base64,<base64_encoded_nmstate_configuration>
           mode: 0644
           overwrite: true
           path: /etc/nmstate/openshift/worker-1.yml
   # ...
   ```

   where:

   <dl>
   <dt><code>metadata.name</code></dt>
   <dd>Specifies the name of the policy.</dd>
   <dt><code>contents.source</code></dt>
   <dd>Writes the encoded base64 information to the specified path.</dd>
   <dt><code>path</code></dt>
   <dd>For each node in your cluster, specify the hostname path to your node and the base-64 encoded Ignition configuration file data for the machine type. The <code>worker</code> role is the default role for nodes in your cluster. You must use the <code>.yml</code> extension for configuration files. For example, use <code>$(hostname -s).yml</code> when specifying the short hostname path for each node or all nodes in the <code>MachineConfig</code> manifest file. You can apply a single global configuration to all nodes in your cluster by using the <code>/etc/nmstate/openshift/cluster.yml</code> configuration file. In this case, you do not need to specify the short hostname path for each node, such as <code>/etc/nmstate/openshift/&lt;node_hostname&gt;.yml</code>. For example:</dd>
   </dl>

   ```yaml title="Example /etc/nmstate/openshift/cluster.yml configuration file"
   # ...
         - contents:
             source: data:text/plain;charset=utf-8;base64,<base64_encoded_nmstate_configuration>
           mode: 0644
           overwrite: true
           path: /etc/nmstate/openshift/cluster.yml
   # ...
   ```

**Next steps**

- Scaling compute nodes to apply the manifest object that includes a customized `br-ex` bridge to each compute node that exists in your cluster. For more information, see "Expanding the cluster".

**Additional resources**

- [Creating a MachineConfig object by using Butane](/docs/installing/install_config/installing-customizing#installation-special-config-butane-create_installing-customizing)
- [Converting to a dual-stack cluster network](/docs/networking/ovn_kubernetes_network_provider/converting-to-dual-stack#nw-dual-stack-convert_converting-to-dual-stack)
- [Expanding the cluster](/docs/installing/installing_bare_metal/bare-metal-expanding-the-cluster#bare-metal-expanding-the-cluster)

### Scaling each machine set to compute nodes {#creating-scaling-machine-sets-compute-nodes-networking_installing-restricted-networks-bare-metal}

To scale each machine set to compute nodes, you must apply a customized `br-ex` bridge configuration to all compute nodes in your OpenShift Container Platform cluster. You must then edit your `MachineConfig` custom resource (CR) and modify its roles.

Additionally, you must create a `BareMetalHost` CR that defines information for your bare-metal machine, such as hostname, credentials, and your other required parameters. After you configure these resources, you must scale machine sets, so that the machine sets can apply the resource configuration to each compute node and reboot the nodes.

**Prerequisites**

- You created a `MachineConfig` manifest object that includes a customized `br-ex` bridge configuration.

**Procedure**

1. Edit the `MachineConfig` CR by entering the following command:
   ```terminal
   $ oc edit mc <machineconfig_custom_resource_name>
   ```
2. Add each compute node configuration to the CR, so that the CR can manage roles for each defined compute node in your cluster.
3. Create a `Secret` object named `extraworker-secret` that has a minimal static IP configuration.
4. Apply the `extraworker-secret` secret to each node in your cluster by entering the following command. This step provides each compute node access to the Ignition config file.
   ```terminal
   $ oc apply -f ./extraworker-secret.yaml
   ```
5. Create a `BareMetalHost` resource and specify the network secret in the `preprovisioningNetworkDataName` parameter:
   ```yaml title="Example BareMetalHost resource with an attached network secret"
   apiVersion: metal3.io/v1alpha1
   kind: BareMetalHost
   spec:
   # ...
     preprovisioningNetworkDataName: ostest-extraworker-0-network-config-secret
   # ...
   ```
6. To manage the `BareMetalHost` object within the `openshift-machine-api` namespace of your cluster, change to the namespace by entering the following command:
   ```terminal
   $ oc project openshift-machine-api
   ```
7. Get the machine sets:
   ```terminal
   $ oc get machinesets
   ```
8. Scale each machine set by entering the following command. You must run this command for each machine set.
   ```terminal
   $ oc scale machineset <machineset_name> --replicas=<n>
   ```

   where:

   <dl>
   <dt><code>&lt;machineset_name&gt;</code></dt>
   <dd>Specifies the name of the machine set.</dd>
   <dt><code>&lt;n&gt;</code></dt>
   <dd>Specifies the number of compute nodes.</dd>
   </dl>

## Preparing the user-provisioned infrastructure {#installation-infrastructure-user-infra_installing-restricted-networks-bare-metal}

Before you install OpenShift Container Platform on user-provisioned infrastructure, you must prepare the underlying infrastructure.

This section provides details about the high-level steps required to set up your cluster infrastructure in preparation for an OpenShift Container Platform installation. This includes configuring IP networking and network connectivity for your cluster nodes, enabling the required ports through your firewall, and setting up the required DNS and load balancing infrastructure.

After preparation, your cluster infrastructure must meet the requirements outlined in the *Requirements for a cluster with user-provisioned infrastructure* section.

**Prerequisites**

- You have reviewed the [OpenShift Container Platform 4.x Tested Integrations](https://access.redhat.com/articles/4128421) page.
- You have reviewed the infrastructure requirements detailed in the *Requirements for a cluster with user-provisioned infrastructure* section.

**Procedure**

1. If you are using DHCP to provide the IP networking configuration to your cluster nodes, configure your DHCP service.
   1. Add persistent IP addresses for the nodes to your DHCP server configuration. In your configuration, match the MAC address of the relevant network interface to the intended IP address for each node.
   2. When you use DHCP to configure IP addressing for the cluster machines, the machines also obtain the DNS server information through DHCP. Define the persistent DNS server address that is used by the cluster nodes through your DHCP server configuration.
      :::note

      If you are not using a DHCP service, you must provide the IP networking configuration and the address of the DNS server to the nodes at RHCOS install time. These can be passed as boot arguments if you are installing from an ISO image. See the *Installing RHCOS and starting the OpenShift Container Platform bootstrap process* section for more information about static IP provisioning and advanced networking options.

      :::
   3. Define the hostnames of your cluster nodes in your DHCP server configuration. See the *Setting the cluster node hostnames through DHCP* section for details about hostname considerations.
      :::note

      If you are not using a DHCP service, the cluster nodes obtain their hostname through a reverse DNS lookup.

      :::
2. Ensure that your network infrastructure provides the required network connectivity between the cluster components. See the *Networking requirements for user-provisioned infrastructure* section for details about the requirements.
3. Configure your firewall to enable the ports required for the OpenShift Container Platform cluster components to communicate. See *Networking requirements for user-provisioned infrastructure* section for details about the ports that are required.
   :::warning

   By default, port `1936` is accessible for an OpenShift Container Platform cluster, because each control plane node needs access to this port.

   For ingress health check probes, the `/healthz/ready` endpoint is available on this port.

   Avoid using the Ingress load balancer to expose this port, because doing so might result in the exposure of sensitive information, such as statistics and metrics, related to Ingress Controllers.

   :::
4. Setup the required DNS infrastructure for your cluster.
   1. Configure DNS name resolution for the Kubernetes API, the application wildcard, the bootstrap machine, the control plane machines, and the compute machines.
   2. Configure reverse DNS resolution for the Kubernetes API, the bootstrap machine, the control plane machines, and the compute machines.
      See the *User-provisioned DNS requirements* section for more information about the OpenShift Container Platform DNS requirements.
5. Validate your DNS configuration.
   1. From your installation node, run DNS lookups against the record names of the Kubernetes API, the wildcard routes, and the cluster nodes. Validate that the IP addresses in the responses correspond to the correct components.
   2. From your installation node, run reverse DNS lookups against the IP addresses of the load balancer and the cluster nodes. Validate that the record names in the responses correspond to the correct components.
      See the *Validating DNS resolution for user-provisioned infrastructure* section for detailed DNS validation steps.
6. Provision the required API and application ingress load balancing infrastructure. See the *Load balancing requirements for user-provisioned infrastructure* section for more information about the requirements.
   :::note

   Some load balancing solutions require the DNS name resolution for the cluster nodes to be in place before the load balancing is initialized.

   :::

**Additional resources**

- [Requirements for a cluster with user-provisioned infrastructure](/docs/installing/installing_bare_metal/upi/installing-restricted-networks-bare-metal#installation-requirements-user-infra_installing-restricted-networks-bare-metal)
- [Installing RHCOS and starting the OpenShift Container Platform bootstrap process](/docs/installing/installing_bare_metal/upi/installing-restricted-networks-bare-metal#creating-machines-bare-metal_installing-restricted-networks-bare-metal)
- [Setting the cluster node hostnames through DHCP](/docs/installing/installing_bare_metal/upi/installing-restricted-networks-bare-metal#installation-host-names-dhcp-user-infra_installing-restricted-networks-bare-metal)
- [Advanced RHCOS installation configuration](/docs/installing/installing_bare_metal/upi/installing-restricted-networks-bare-metal#installation-user-infra-machines-advanced_installing-restricted-networks-bare-metal)
- [Networking requirements for user-provisioned infrastructure](/docs/installing/installing_bare_metal/upi/installing-restricted-networks-bare-metal#installation-network-user-infra_installing-restricted-networks-bare-metal)
- [User-provisioned DNS requirements](/docs/installing/installing_bare_metal/upi/installing-restricted-networks-bare-metal#installation-dns-user-infra_installing-restricted-networks-bare-metal)
- [Validating DNS resolution for user-provisioned infrastructure](/docs/installing/installing_bare_metal/upi/installing-restricted-networks-bare-metal#installation-user-provisioned-validating-dns_installing-restricted-networks-bare-metal)
- [Load balancing requirements for user-provisioned infrastructure](/docs/installing/installing_bare_metal/upi/installing-restricted-networks-bare-metal#installation-load-balancing-user-infra_installing-restricted-networks-bare-metal)

## Validating DNS resolution for user-provisioned infrastructure {#installation-user-provisioned-validating-dns_installing-restricted-networks-bare-metal}

To prevent network-related installation failures and ensure node connectivity in OpenShift Container Platform, validate your DNS configuration before deploying on user-provisioned infrastructure.

:::warning

The validation steps detailed in this section must succeed before you install your cluster.

:::

**Prerequisites**

- You have configured the required DNS records for your user-provisioned infrastructure.

**Procedure**

1. From your installation node, run DNS lookups against the record names of the Kubernetes API, the wildcard routes, and the cluster nodes. Validate that the IP addresses contained in the responses correspond to the correct components.
   1. Perform a lookup against the Kubernetes API record name. Check that the result points to the IP address of the API load balancer:
      ```terminal
      $ dig +noall +answer @<nameserver_ip> api.<cluster_name>.<base_domain>
      ```

      Replace `<nameserver_ip>` with the IP address of the name server, `<cluster_name>` with your cluster name, and `<base_domain>` with your base domain name.

      ```terminal title="Example output"
      api.ocp4.example.com.		604800	IN	A	192.168.1.5
      ```
   2. Perform a lookup against the Kubernetes internal API record name. Check that the result points to the IP address of the API load balancer:
      ```terminal
      $ dig +noall +answer @<nameserver_ip> api-int.<cluster_name>.<base_domain>
      ```

      ```terminal title="Example output"
      api-int.ocp4.example.com.		604800	IN	A	192.168.1.5
      ```
   3. Test an example `*.apps.<cluster_name>.<base_domain>` DNS wildcard lookup. All of the application wildcard lookups must resolve to the IP address of the application ingress load balancer:
      ```terminal
      $ dig +noall +answer @<nameserver_ip> random.apps.<cluster_name>.<base_domain>
      ```

      ```terminal title="Example output"
      random.apps.ocp4.example.com.		604800	IN	A	192.168.1.5
      ```

      :::note

      In the example outputs, the same load balancer is used for the Kubernetes API and application ingress traffic. In production scenarios, you can deploy the API and application ingress load balancers separately so that you can scale the load balancer infrastructure for each in isolation.

      :::

      You can replace `random` with another wildcard value. For example, you can query the route to the OpenShift Container Platform console:

      ```terminal
      $ dig +noall +answer @<nameserver_ip> console-openshift-console.apps.<cluster_name>.<base_domain>
      ```

      ```terminal title="Example output"
      console-openshift-console.apps.ocp4.example.com. 604800 IN	A 192.168.1.5
      ```
   4. Run a lookup against the bootstrap DNS record name. Check that the result points to the IP address of the bootstrap node:
      ```terminal
      $ dig +noall +answer @<nameserver_ip> bootstrap.<cluster_name>.<base_domain>
      ```

      ```terminal title="Example output"
      bootstrap.ocp4.example.com.		604800	IN	A	192.168.1.96
      ```
   5. Use this method to perform lookups against the DNS record names for the control plane and compute nodes. Check that the results correspond to the IP addresses of each node.
2. From your installation node, run reverse DNS lookups against the IP addresses of the load balancer and the cluster nodes. Validate that the record names contained in the responses correspond to the correct components.
   1. Perform a reverse lookup against the IP address of the API load balancer. Check that the response includes the record names for the Kubernetes API and the Kubernetes internal API:
      ```terminal
      $ dig +noall +answer @<nameserver_ip> -x 192.168.1.5
      ```

      ```terminal title="Example output"
      5.1.168.192.in-addr.arpa. 604800	IN	PTR	api-int.ocp4.example.com.
      5.1.168.192.in-addr.arpa. 604800	IN	PTR	api.ocp4.example.com.
      ```

      where:

      <dl>
      <dt><code>api-int.ocp4.example.com</code></dt>
      <dd>Specifies the record name for the Kubernetes internal API.</dd>
      <dt><code>api.ocp4.example.com</code></dt>
      <dd>Specifies the record name for the Kubernetes API.</dd>
      </dl>

      :::note

      A PTR record is not required for the OpenShift Container Platform application wildcard. No validation step is needed for reverse DNS resolution against the IP address of the application ingress load balancer.

      :::
   2. Perform a reverse lookup against the IP address of the bootstrap node. Check that the result points to the DNS record name of the bootstrap node:
      ```terminal
      $ dig +noall +answer @<nameserver_ip> -x 192.168.1.96
      ```

      ```terminal title="Example output"
      96.1.168.192.in-addr.arpa. 604800	IN	PTR	bootstrap.ocp4.example.com.
      ```
   3. Use this method to perform reverse lookups against the IP addresses for the control plane and compute nodes. Check that the results correspond to the DNS record names of each node.

**Additional resources**

- [User-provisioned DNS requirements](/docs/installing/installing_bare_metal/upi/installing-restricted-networks-bare-metal#installation-dns-user-infra_installing-restricted-networks-bare-metal)
- [Load balancing requirements for user-provisioned infrastructure](/docs/installing/installing_bare_metal/upi/installing-restricted-networks-bare-metal#installation-load-balancing-user-infra_installing-restricted-networks-bare-metal)

## Generating a key pair for cluster node SSH access {#ssh-agent-using_installing-restricted-networks-bare-metal}

During an OpenShift Container Platform installation, you can provide an SSH public key to the installation program. The key is passed to the Red Hat Enterprise Linux CoreOS (RHCOS) nodes through their Ignition config files and is used to authenticate SSH access to the nodes. The key is added to the `~/.ssh/authorized_keys` list for the `core` user on each node, which enables password-less authentication.

The key is added to the `~/.ssh/authorized_keys` list for the `core` user on each node, which enables password-less authentication. After the key is passed to the nodes, you can use the key pair to SSH in to the RHCOS nodes as the user `core`. To access the nodes through SSH, the private key identity must be managed by SSH for your local user.

If you want to SSH in to your cluster nodes to perform installation debugging or disaster recovery, you must provide the SSH public key during the installation process. The `./openshift-install gather` command also requires the SSH public key to be in place on the cluster nodes.

:::warning

Do not skip this procedure in production environments, where disaster recovery and debugging is required.

:::

:::note

You must use a local key, not one that you configured with platform-specific approaches.

:::

**Procedure**

1. If you do not have an existing SSH key pair on your local machine to use for authentication onto your cluster nodes, create one. For example, on a computer that uses a Linux operating system, run the following command:
   ```terminal
   $ ssh-keygen -t ed25519 -N '' -f <path>/<file_name>
   ```

   Specifies the path and file name, such as `~/.ssh/id_ed25519`, of the new SSH key. If you have an existing key pair, ensure your public key is in the your `~/.ssh` directory.

   :::note

   If you plan to install an OpenShift Container Platform cluster that uses the RHEL cryptographic libraries that have been submitted to NIST for FIPS 140-2/140-3 Validation on only the `x86_64`, `ppc64le`, and `s390x` architectures, do not create a key that uses the `ed25519` algorithm. Instead, create a key that uses the `rsa` or `ecdsa` algorithm.

   :::
2. View the public SSH key:
   ```terminal
   $ cat <path>/<file_name>.pub
   ```

   For example, run the following to view the `~/.ssh/id_ed25519.pub` public key:

   ```terminal
   $ cat ~/.ssh/id_ed25519.pub
   ```
3. Add the SSH private key identity to the SSH agent for your local user, if it has not already been added. SSH agent management of the key is required for password-less SSH authentication onto your cluster nodes, or if you want to use the `./openshift-install gather` command.
   :::note

   On some distributions, default SSH private key identities such as `~/.ssh/id_rsa` and `~/.ssh/id_dsa` are managed automatically.

   :::

   1. If the `ssh-agent` process is not already running for your local user, start it as a background task:
      ```terminal
      $ eval "$(ssh-agent -s)"
      ```

      ```terminal title="Example output"
      Agent pid 31874
      ```

      :::note

      If your cluster is in FIPS mode, only use FIPS-compliant algorithms to generate the SSH key. The key must be either RSA or ECDSA.

      :::
4. Add your SSH private key to the `ssh-agent`:
   ```terminal
   $ ssh-add <path>/<file_name>
   ```

   Specify the path and file name for your SSH private key, such as `~/.ssh/id_ed25519`.

   ```terminal title="Example output"
   Identity added: /home/<you>/<path>/<file_name> (<computer_name>)
   ```

**Next steps**

- When you install OpenShift Container Platform, provide the SSH public key to the installation program. If you install a cluster on infrastructure that you provision, you must provide the key to the installation program.

**Additional resources**

- [Verifying node health](/docs/support/troubleshooting/verifying-node-health#verifying-node-health)

## Manually creating the installation configuration file {#installation-initializing-manual_installing-restricted-networks-bare-metal}

Installing the cluster requires that you manually create the installation configuration file.

**Prerequisites**

- You have an SSH public key on your local machine for use with the installation program. You can use the key for SSH authentication onto your cluster nodes for debugging and disaster recovery.
- You have obtained the OpenShift Container Platform installation program and the pull secret for your cluster.
- Obtain the `imageContentSources` section from the output of the command to mirror the repository.
- Obtain the contents of the certificate for your mirror registry.

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

   - Unless you use a registry that RHCOS trusts by default, such as `docker.io`, you must provide the contents of the certificate for your mirror repository in the `additionalTrustBundle` section. In most cases, you must provide the certificate for your mirror.
   - You must include the `imageContentSources` section from the output of the command to mirror the repository.
     :::warning

     - The `ImageContentSourcePolicy` file is generated as an output of `oc mirror` after the mirroring process is finished.
     - The `oc mirror` command generates an `ImageContentSourcePolicy` file which contains the YAML needed to define `ImageContentSourcePolicy`. Copy the text from this file and paste it into your `install-config.yaml` file.
     - You must run the 'oc mirror' command twice. The first time you run the `oc mirror` command, you get a full `ImageContentSourcePolicy` file. The second time you run the `oc mirror` command, you only get the difference between the first and second run. Because of this behavior, you must always keep a backup of these files in case you need to merge them into one complete `ImageContentSourcePolicy` file. Keeping a backup of these two output files ensures that you have a complete `ImageContentSourcePolicy` file.

     :::
3. Back up the `install-config.yaml` file so that you can use it to install many clusters.
   :::warning

   Back up the `install-config.yaml` file now, because the installation process consumes the file in the next step.

   :::

**Additional resources**

- [Installation configuration parameters for bare metal](/docs/installing/installing_bare_metal/upi/installation-config-parameters-bare-metal#installation-config-parameters-bare-metal)

### Sample install-config.yaml file for bare metal {#installation-bare-metal-config-yaml_installing-restricted-networks-bare-metal}

You can customize the `install-config.yaml` file to specify more details about your OpenShift Container Platform cluster platform or modify the values of the required parameters.

```yaml
apiVersion: v1
baseDomain: example.com
compute:
- hyperthreading: Enabled
  name: worker
  replicas: 0
controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 3
metadata:
  name: test
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  networkType: OVNKubernetes
  machineNetwork:
  - cidr: 192.168.0.0/16
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {}
fips: false

pullSecret: '{"auths":{"<local_registry>": {"auth": "<credentials>","email": "you@example.com"}}}'

sshKey: 'ssh-ed25519 AAAA...'

additionalTrustBundle: |
  -----BEGIN CERTIFICATE-----
  ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ
  -----END CERTIFICATE-----
imageContentSources:
- mirrors:
  - <local_registry>/<local_repository_name>/release
  source: quay.io/openshift-release-dev/ocp-release
- mirrors:
  - <local_registry>/<local_repository_name>/release
  source: quay.io/openshift-release-dev/ocp-v4.0-art-dev

```

where:

`baseDomain`
: Specifies the base domain of the cluster. All DNS records must be sub-domains of this base and include the cluster name.

`compute`
: Specifies the `compute` node configurations, which is a sequence of mappings. To meet the requirements of the different data structures, the first line of the `compute` section must begin with a hyphen, `-`.

`controlPlane`
: Specifies the `controlPlane` node configurations, which is a single mapping. To meet the requirements of the different data structures, the first line of the `controlPlane` section must not. Only one control plane pool is used.

`hyperthreading`
: Specifies whether to enable or disable simultaneous multithreading (SMT), or hyperthreading. By default, SMT is enabled to increase the performance of the cores in your machines. You can disable it by setting the parameter value to `Disabled`. If you disable SMT, you must disable it in all cluster machines; this includes both control plane and compute machines.

:::note

Simultaneous multithreading (SMT) is enabled by default. If SMT is not enabled in your BIOS settings, the `hyperthreading` parameter has no effect.

:::

:::warning

If you disable `hyperthreading`, whether in the BIOS or in the `install-config.yaml` file, ensure that your capacity planning accounts for the dramatically decreased machine performance.

:::

`compute.replicas`
: Specifies the number of compute machines that the cluster creates and manages for you on installer-provisioned installations. You must set this value to `0` when you install OpenShift Container Platform on user-provisioned infrastructure. Additionally for user-provisioned installations, you must manually deploy the compute machines before you finish installing the cluster.

:::note

If you are installing a three-node cluster, do not deploy any compute machines when you install the Red Hat Enterprise Linux CoreOS (RHCOS) machines.

:::

`controlPlane.replicas`
: Specifies the number of control plane machines that you add to the cluster. Because the cluster uses these values as the number of etcd endpoints in the cluster, the value must match the number of control plane machines that you deploy.

`metadata.name`
: Specifies the cluster name that you specified in your DNS records.

`networking.clusterNetwork.cidr`
: Specifies a block of IP addresses from which pod IP addresses are allocated. This block must not overlap with existing physical networks. These IP addresses are used for the pod network. If you need to access the pods from an external network, you must configure load balancers and routers to manage the traffic.

:::note

Class E CIDR range is reserved for a future use. To use the Class E CIDR range, you must ensure your networking environment accepts the IP addresses within the Class E CIDR range.

:::

`networking.cidr.hostPrefix`
: Specifies the subnet prefix length to assign to each individual node. For example, if `hostPrefix` is set to `23`, then each node is assigned a `/23` subnet out of the given `cidr`, which allows for 510 (2^(32 - 23) - 2) pod IP addresses. If you are required to provide access to nodes from an external network, configure load balancers and routers to manage the traffic.

`networking.networkType`
: Specifies the cluster network plugin to install. The default value `OVNKubernetes` is the only supported value.

`networking.serviceNetwork`
: Specifies the IP address pool to use for service IP addresses. You can enter only one IP address pool. This block must not overlap with existing physical networks. If you need to access the services from an external network, configure load balancers and routers to manage the traffic.

`platform`
: Specifies the platform. You must set the platform to `none`. You cannot provide additional platform configuration variables for your platform.

:::warning

Clusters that are installed with the platform type `none` are unable to use some features, such as managing compute machines with the Machine API. This limitation applies even if the compute machines that are attached to the cluster are installed on a platform that would normally support the feature. This parameter cannot be changed after installation.

:::

`fips`
: Specifies either enabling or disabling FIPS mode. By default, FIPS mode is not enabled. If FIPS mode is enabled, the Red Hat Enterprise Linux CoreOS (RHCOS) machines that OpenShift Container Platform runs on bypass the default Kubernetes cryptography suite and use the cryptography modules that are provided with RHCOS instead.

:::warning

To enable FIPS mode for your cluster, you must run the installation program from a Red Hat Enterprise Linux (RHEL) computer configured to operate in FIPS mode. For more information about configuring FIPS mode on RHEL, see [Switching RHEL to FIPS mode](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/security_hardening/switching-rhel-to-fips-mode_security-hardening).

When running Red Hat Enterprise Linux (RHEL) or Red Hat Enterprise Linux CoreOS (RHCOS) booted in FIPS mode, OpenShift Container Platform core components use the RHEL cryptographic libraries that have been submitted to NIST for FIPS 140-2/140-3 Validation on only the x86_64, ppc64le, and s390x architectures.

:::

`pullSecret`:: Specifies the registry domain name for `<local_registry>`, and optionally the port, that your mirror registry uses to serve content. For example, `registry.example.com` or `registry.example.com:5000`. For `<credentials>`, specify the base64-encoded user name and password for your mirror registry.

`sshKey`:: Specifies the SSH public key for the `core` user in Red Hat Enterprise Linux CoreOS (RHCOS).

:::note

For production OpenShift Container Platform clusters on which you want to perform installation debugging or disaster recovery, specify an SSH key that your `ssh-agent` process uses.

:::

`additionalTrustBundle`
: Specifies the contents of the certificate file that you used for your mirror registry.

`imageContentSources`:: Specifies the `imageContentSources` section according to the output of the command that you used to mirror the repository.

:::warning

- When using the `oc adm release mirror` command, use the output from the `imageContentSources` section.
- When using `oc mirror` command, use the `repositoryDigestMirrors` section of the `ImageContentSourcePolicy` file that results from running the command.
- `ImageContentSourcePolicy` is deprecated. For more information see *Configuring image registry repository mirroring*.

:::

**Additional resources**

- See [Load balancing requirements for user-provisioned infrastructure](/docs/installing/installing_bare_metal/upi/installing-restricted-networks-bare-metal#installation-load-balancing-user-infra_installing-restricted-networks-bare-metal) for more information on the API and application ingress load balancing requirements.

### Configuring the cluster-wide proxy during installation {#installation-configure-proxy_installing-restricted-networks-bare-metal}

Production environments can deny direct access to the internet and instead have an HTTP or HTTPS proxy available. You can configure a new OpenShift Container Platform cluster to use a proxy by configuring the proxy settings in the `install-config.yaml` file.

:::note

For bare-metal installations, if you do not assign node IP addresses from the range that is specified in the `networking.machineNetwork[].cidr` field in the `install-config.yaml` file, you must include them in the `proxy.noProxy` field.

:::

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

### Configuring a three-node cluster {#installation-three-node-cluster_installing-restricted-networks-bare-metal}

To create smaller, resource-efficient clusters for testing and production, deploy a bare-metal cluster with zero compute machines. This optional configuration uses only three control plane machines, optimizing infrastructure resources for administrators and developers.

In three-node OpenShift Container Platform environments, the three control plane machines are schedulable, which means that your application workloads run on them.

**Prerequisites**

- You have an existing `install-config.yaml` file.

**Procedure**

- Ensure that the number of compute replicas is set to `0` in your `install-config.yaml` file, as shown in the following `compute` stanza:
  ```yaml
  compute:
  - name: worker
    platform: {}
    replicas: 0
  # ...
  ```

  :::note

  You must set the value of the `replicas` parameter for the compute machines to `0` when you install OpenShift Container Platform on user-provisioned infrastructure, regardless of the number of compute machines you deploy. In installer-provisioned installations, the parameter controls the number of compute machines that the cluster creates and manages for you. This does not apply to user-provisioned installations, where you deploy the compute machines manually.

  :::

  **Next steps**

For three-node cluster installations, follow these next steps:

- If you are deploying a three-node cluster with zero compute nodes, the Ingress Controller pods run on the control plane nodes. In three-node cluster deployments, you must configure your application ingress load balancer to route HTTP and HTTPS traffic to the control plane nodes. See the *Load balancing requirements for user-provisioned infrastructure* section for more information.
- When you create the Kubernetes manifest files in the following procedure, ensure that the `mastersSchedulable` parameter in the `<installation_directory>/manifests/cluster-scheduler-02-config.yml` file is set to `true`. This enables your application workloads to run on the control plane nodes.
- Do not deploy any compute nodes when you create the Red Hat Enterprise Linux CoreOS (RHCOS) machines.

## Creating the Kubernetes manifest and Ignition config files {#installation-user-infra-generate-k8s-manifest-ignition_installing-restricted-networks-bare-metal}

Because you manually provision infrastructure, you must generate the Kubernetes manifest and Ignition config files that the cluster requires.

The installation program converts the installation configuration into Kubernetes manifests and then wraps them into Ignition configuration files. You use these Ignition files to configure the cluster machines.

:::warning

- The Ignition config files that the OpenShift Container Platform installation program generates contain certificates that expire after 24 hours, which the system then renews. If you shut down the cluster before the system renews the certificates and you later restart the cluster after the 24 hours have elapsed, the cluster automatically recovers the expired certificates. The exception is that you must manually approve the pending `node-bootstrapper` certificate signing requests (CSRs) to recover kubelet certificates. See the documentation for *Recovering from expired control plane certificates* for more information.
- Use Ignition config files within 12 hours after you generate them, because the 24-hour certificate rotates from 16 to 22 hours after you install the cluster. By using the Ignition config files within 12 hours, you can avoid installation failure if the certificate update runs during installation.

:::

**Prerequisites**

- You obtained the OpenShift Container Platform installation program. For a restricted network installation, these files are on your mirror host.
- You created the `install-config.yaml` installation configuration file.

**Procedure**

1. Change to the directory that contains the OpenShift Container Platform installation program and generate the Kubernetes manifests for the cluster:
   ```terminal
   $ ./openshift-install create manifests --dir <installation_directory>
   ```

   where:

   <dl>
   <dt><code>&lt;installation_directory&gt;</code></dt>
   <dd>Specifies the installation directory that contains the <code>install-config.yaml</code> file you created.</dd>
   </dl>

   :::warning

   If you are installing a three-node cluster, skip the following step to allow the control plane nodes to be schedulable.

   :::

   :::warning

   When you configure control plane nodes from the default unschedulable to schedulable, you require additional subscriptions because control plane nodes then become compute nodes.

   :::
2. Verify that the `mastersSchedulable` parameter in the `<installation_directory>/manifests/cluster-scheduler-02-config.yml` Kubernetes manifest file is set to `false`. This setting prevents pods from being scheduled on the control plane machines:
   1. Open the `<installation_directory>/manifests/cluster-scheduler-02-config.yml` file.
   2. Locate the `mastersSchedulable` parameter and verify that it is set to `false`.
   3. Save and exit the file.
3. To create the Ignition configuration files, run the following command from the directory that contains the installation program:
   ```terminal
   $ ./openshift-install create ignition-configs --dir <installation_directory>
   ```

   where:

   <dl>
   <dt><code>&lt;installation_directory&gt;</code></dt>
   <dd>Specifies the same installation directory. The installation program creates Ignition config files for the bootstrap, control plane, and compute nodes in the installation directory. The program also creates the <code>kubeadmin-password</code> and <code>kubeconfig</code> files in the <code>./&lt;installation_directory&gt;/auth</code> directory:</dd>
   </dl>

   ```
   .
   ├── auth
   │   ├── kubeadmin-password
   │   └── kubeconfig
   ├── bootstrap.ign
   ├── master.ign
   ├── metadata.json
   └── worker.ign
   ```

**Additional resources**

- See [Recovering from expired control plane certificates](/docs/backup_and_restore/control_plane_backup_and_restore/disaster_recovery/scenario-3-expired-certs#dr-recovering-expired-certs) for more information about recovering kubelet certificates.

## Configure chrony time service {#installation-special-config-chrony_installing-restricted-networks-bare-metal}

You must set the time server and related settings used by the chrony time service (`chronyd`) by modifying the contents of the `chrony.conf` file and passing those contents to your nodes as a machine config.

For more information on chrony best practices, see the following resources:

- [Configuring chrony (Red Hat Knowledgebase article)](https://access.redhat.com/solutions/3073261)
- [Best practices for NTP (Red Hat Knowledgebase article)](https://access.redhat.com/solutions/778603)
- [Basic chrony NTP troubleshooting (Red Hat Ceph Storage documentation)](https://docs.redhat.com/en/documentation/red_hat_ceph_storage/8/html-single/troubleshooting_guide/basic-chrony-NTP-troubleshooting_diag#basic-chrony-NTP-troubleshooting_diag)

**Procedure**

1. Create a Butane config including the contents of the `chrony.conf` file. For example, to configure chrony on worker nodes, create a `99-worker-chrony.bu` file.
   :::note

   The [Butane version](https://coreos.github.io/butane/specs/) you specify in the config file should match the OpenShift Container Platform version and always ends in `0`. For example, `4.22.0`. See "Creating machine configs with Butane" for information about Butane.

   :::

   ```yaml
   variant: openshift
   version: 4.22.0
   metadata:
     name: 99-worker-chrony
     labels:
       machineconfiguration.openshift.io/role: worker
   storage:
     files:
     - path: /etc/chrony.conf
       mode: 0644
       overwrite: true
       contents:
         inline: |
           pool 0.rhel.pool.ntp.org iburst
           driftfile /var/lib/chrony/drift
           makestep 1.0 3
           rtcsync
           logdir /var/log/chrony
   ```

   - `name: 99-worker-chrony` - Specify a name for the machine config file. On control plane nodes, substitute `master` for `worker`.
   - `machineconfiguration.openshift.io/role: worker` - On control plane nodes, substitute `master` for `worker`.
   - `mode: 0644` - Specify an octal value mode for the `mode` field in the machine config file. After creating the file and applying the changes, the `mode` is converted to a decimal value. You can check the YAML file with the command `oc get mc <mc-name> -o yaml`.
   - `pool 0.rhel.pool.ntp.org iburst` - Specify any valid, reachable time source, such as the one provided by your DHCP server.

   :::note

   For all-machine to all-machine communication, the Network Time Protocol (NTP) on UDP is port `123`. If an external NTP time server is configured, you must open UDP port `123`.

   :::
2. Use Butane to generate a `MachineConfig` object file, `99-worker-chrony.yaml`, containing the configuration to be delivered to the nodes:
   ```terminal
   $ butane 99-worker-chrony.bu -o 99-worker-chrony.yaml
   ```
3. Apply the configurations in one of two ways:
   - If the cluster is not running yet, after you generate manifest files, add the `MachineConfig` object file to the `<installation_directory>/openshift` directory, and then continue to create the cluster.
   - If the cluster is already running, apply the file:
     ```terminal
     $ oc apply -f ./99-worker-chrony.yaml
     ```

## Installing RHCOS and starting the OpenShift Container Platform bootstrap process {#creating-machines-bare-metal_installing-restricted-networks-bare-metal}

To install OpenShift Container Platform on bare-metal infrastructure that you provision, install Red Hat Enterprise Linux CoreOS (RHCOS) by using the generated Ignition config files. Providing these files ensures the bootstrap process begins automatically after the machines reboot.

If you have configured suitable networking, DNS, and load balancing infrastructure, the OpenShift Container Platform bootstrap process begins automatically after the RHCOS machines have rebooted.

To install RHCOS on the machines, follow either the steps to use an ISO image or network PXE booting.

:::note

The compute node deployment steps included in this installation document are RHCOS-specific. If you choose instead to deploy RHEL-based compute nodes, you take responsibility for all operating system life cycle management and maintenance, including performing system updates, applying patches, and completing all other required tasks. Only RHEL 8 compute machines are supported.

:::

You can configure RHCOS during ISO and PXE installations by using the following methods:

- Kernel arguments: You can use kernel arguments to provide installation-specific information. For example, you can specify the locations of the RHCOS installation files that you uploaded to your HTTP server and the location of the Ignition config file for the type of node you are installing. For a PXE installation, you can use the `APPEND` parameter to pass the arguments to the kernel of the live installer. For an ISO installation, you can interrupt the live installation boot process to add the kernel arguments. In both installation cases, you can use special `coreos.inst.*` arguments to direct the live installer, as well as standard installation boot arguments for turning standard kernel services on or off.
- Ignition configs: OpenShift Container Platform Ignition config files (`*.ign`) are specific to the type of node you are installing. You pass the location of a bootstrap, control plane, or compute node Ignition config file during the RHCOS installation so that it takes effect on first boot. In special cases, you can create a separate, limited Ignition config to pass to the live system. That Ignition config could do a certain set of tasks, such as reporting success to a provisioning system after completing installation. This special Ignition config is consumed by the `coreos-installer` to be applied on first boot of the installed system. Do not provide the standard control plane and compute node Ignition configs to the live ISO directly.
- `coreos-installer`: You can boot the live ISO installer to a shell prompt, which allows you to prepare the permanent system in a variety of ways before first boot. In particular, you can run the `coreos-installer` command to identify various artifacts to include, work with disk partitions, and set up networking. In some cases, you can configure features on the live system and copy them to the installed system.
  :::note

  As of version `0.17.0-3`, `coreos-installer` requires RHEL 9 or later to run the program. You can still use older versions of `coreos-installer` to customize RHCOS artifacts of newer OpenShift Container Platform releases and install metal images to disk. See "coreos-installer image mirror" to download older versions of the `coreos-installer` binary.

  :::

Whether to use an ISO or PXE install depends on your situation. A PXE install requires an available DHCP service and more preparation, but can make the installation process more automated. An ISO install is a more manual process and can be inconvenient if you are setting up more than a few machines.

**Additional resources**

- [coreos-installer image mirror](https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/coreos-installer/)

### Installing RHCOS by using an ISO image {#installation-user-infra-machines-iso_installing-restricted-networks-bare-metal}

To provision physical or virtual machines, install RHCOS by using a bootable ISO image.

**Prerequisites**

- You have created the Ignition config files for your cluster.
- You have configured a suitable network, DNS, and load balancing infrastructure.
- You have an HTTP server that can be accessed from your computer, and from the machines that you create.
- You have reviewed the *Advanced RHCOS installation configuration* section for different ways to configure features, such as networking and disk partitioning.

**Procedure**

1. Obtain the SHA512 digest for each of your Ignition config files. For example, you can use the following on a system running Linux to get the SHA512 digest for your `bootstrap.ign` Ignition config file:
   ```terminal
   $ sha512sum <installation_directory>/bootstrap.ign
   ```

   The digests are provided to the `coreos-installer` in a later step to validate the authenticity of the Ignition config files on the cluster nodes.
2. Upload the bootstrap, control plane, and compute node Ignition config files that the installation program created to your HTTP server. Note the URLs of these files.
   :::warning

   You can add or change configuration settings in your Ignition configs before saving them to your HTTP server. If you plan to add more compute machines to your cluster after you finish installation, do not delete these files.

   :::
3. From the installation host, validate that the Ignition config files are available on the URLs. The following example gets the Ignition config file for the bootstrap node:
   ```terminal
   $ curl -k http://<HTTP_server>/bootstrap.ign
   ```

   - <HTTP_server>: Replace `bootstrap.ign` with `master.ign` or `worker.ign` in the command to validate that the Ignition config files for the control plane and compute nodes are also available.
     ```terminal title="Example output"
       % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                      Dload  Upload   Total   Spent    Left  Speed
       0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0{"ignition":{"version":"3.2.0"},"passwd":{"users":[{"name":"core","sshAuthorizedKeys":["ssh-rsa...
     ```
4. Although it is possible to obtain the RHCOS images that are required for your preferred method of installing operating system instances from the
   [RHCOS image mirror](https://mirror.openshift.com/pub/openshift-v4/x86_64/dependencies/rhcos/)

   page, the recommended way to obtain the correct version of your RHCOS images are from the output of `openshift-install` command:

   ```terminal
   $ openshift-install coreos print-stream-json | grep '\.iso[^.]'
   ```

   **Example output**

   ```terminal
   "location": "<url>/art/storage/releases/rhcos-4.22-aarch64/<release>/aarch64/rhcos-<release>-live.aarch64.iso",
   "location": "<url>/art/storage/releases/rhcos-4.22-ppc64le/<release>/ppc64le/rhcos-<release>-live.ppc64le.iso",
   "location": "<url>/art/storage/releases/rhcos-4.22-s390x/<release>/s390x/rhcos-<release>-live.s390x.iso",
   "location": "<url>/art/storage/releases/rhcos-4.22/<release>/x86_64/rhcos-<release>-live.x86_64.iso",
   ```

   :::warning

   The RHCOS images might not change with every release of OpenShift Container Platform. You must download images with the highest version that is less than or equal to the OpenShift Container Platform version that you install. Use the image versions that match your OpenShift Container Platform version if they are available. Use only ISO images for this procedure. RHCOS qcow2 images are not supported for this installation type.

   :::

   ISO file names resemble the following example:

   `rhcos-<version>-live.<architecture>.iso`
5. Use the ISO to start the RHCOS installation. Use one of the following installation options:
   - Burn the ISO image to a disk and boot it directly.
   - Use ISO redirection by using a lights-out management (LOM) interface.
6. Boot the RHCOS ISO image without specifying any options or interrupting the live boot sequence. Wait for the installer to boot into a shell prompt in the RHCOS live environment.
   :::note

   It is possible to interrupt the RHCOS installation boot process to add kernel arguments. However, for this ISO procedure you should use the `coreos-installer` command as outlined in the following steps, instead of adding kernel arguments.

   :::
7. Run the `coreos-installer` command and specify the options that meet your installation requirements. At a minimum, you must specify the URL that points to the Ignition config file for the node type, and the device that you are installing to:
   ```terminal
   $ sudo coreos-installer install --ignition-url=http://<HTTP_server>/<node_type>.ign <device> \
   --ignition-hash=sha512-<digest> --offline
   ```

   - `<device>`: You must run the `coreos-installer` command by using `sudo`, because the `core` user does not have the required root privileges to perform the installation.
   - `<digest>`: The `--ignition-hash` option is required when the Ignition config file is obtained through an HTTP URL to validate the authenticity of the Ignition config file on the cluster node. `<digest>` is the Ignition config file SHA512 digest obtained in a preceding step.
     :::note

     If you want to provide your Ignition config files through an HTTPS server that uses TLS, you can add the internal certificate authority (CA) to the system trust store before running `coreos-installer`.

     :::

     The following example initializes a bootstrap node installation to the `/dev/sda` device. The Ignition config file for the bootstrap node is obtained from an HTTP web server with the IP address 192.168.1.2:

     ```terminal
     $ sudo coreos-installer install --ignition-url=http://192.168.1.2:80/installation_directory/bootstrap.ign /dev/sda \
     --ignition-hash=sha512-a5a2d43879223273c9b60af66b44202a1d1248fc01cf156c46d4a79f552b6bad47bc8cc78ddf0116e80c59d2ea9e32ba53bc807afbca581aa059311def2c3e3b \
     --offline
     ```
8. Monitor the progress of the RHCOS installation on the console of the machine.
   :::warning

   Be sure that the installation is successful on each node before commencing with the OpenShift Container Platform installation. Observing the installation process can also help to determine the cause of RHCOS installation issues that might arise.

   :::
9. After RHCOS installs, you must reboot the system. During the system reboot, it applies the Ignition config file that you specified.
10. Check the console output to verify that Ignition ran.
    ```terminal title="Example command"
    Ignition: ran on 2022/03/14 14:48:33 UTC (this boot)
    Ignition: user-provided config was applied
    ```
11. Continue to create the other machines for your cluster.
    :::warning

    You must create the bootstrap and control plane machines at this time. If the control plane machines are not made schedulable, also create at least two compute machines before you install OpenShift Container Platform.

    :::

    If the required network, DNS, and load balancer infrastructure are in place, the OpenShift Container Platform bootstrap process begins automatically after the RHCOS nodes have rebooted.

    :::note

    RHCOS nodes do not include a default password for the `core` user. You can access the nodes by running `ssh core@<node>.<cluster_name>.<base_domain>` as a user with access to the SSH private key that is paired to the public key that you specified in your `install_config.yaml` file. OpenShift Container Platform 4 cluster nodes running RHCOS are immutable and rely on Operators to apply cluster changes. Accessing cluster nodes by using SSH is not recommended. However, when investigating installation issues, if the OpenShift Container Platform API is not available, or the kubelet is not properly functioning on a target node, SSH access might be required for debugging or disaster recovery.

    :::

### Installing RHCOS by using PXE or iPXE booting {#installation-user-infra-machines-pxe_installing-restricted-networks-bare-metal}

You can use PXE or iPXE booting to install RHCOS on the machines.

**Prerequisites**

- You have created the Ignition config files for your cluster.
- You have configured a suitable network, DNS and load balancing infrastructure.
- You have configured suitable PXE or iPXE infrastructure.
- You have an HTTP server that can be accessed from your computer, and from the machines that you create.
- You have reviewed the *Advanced RHCOS installation configuration* section for different ways to configure features, such as networking and disk partitioning.

**Procedure**

1. Upload the bootstrap, control plane, and compute node Ignition config files that the installation program created to your HTTP server. Note the URLs of these files.
   :::warning

   You can add or change configuration settings in your Ignition configs before saving them to your HTTP server. If you plan to add more compute machines to your cluster after you finish installation, do not delete these files.

   :::
2. From the installation host, validate that the Ignition config files are available on the URLs. The following example gets the Ignition config file for the bootstrap node:
   ```terminal
   $ curl -k http://<HTTP_server>/bootstrap.ign
   ```

   - `<HTTP_server>`: Replace `bootstrap.ign` with `master.ign` or `worker.ign` in the command to validate that the Ignition config files for the control plane and compute nodes are also available.
     ```terminal title="Example output"
       % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                      Dload  Upload   Total   Spent    Left  Speed
       0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0{"ignition":{"version":"3.2.0"},"passwd":{"users":[{"name":"core","sshAuthorizedKeys":["ssh-rsa...
     ```
3. Although it is possible to obtain the RHCOS `kernel`, `initramfs` and `rootfs` files that are required for your preferred method of installing operating system instances from the
   [RHCOS image mirror](https://mirror.openshift.com/pub/openshift-v4/x86_64/dependencies/rhcos/)

   page, the recommended way to obtain the correct version of your RHCOS files are from the output of `openshift-install` command:

   ```terminal
   $ openshift-install coreos print-stream-json | grep -Eo '"https.*(kernel-|initramfs.|rootfs.)\w+(\.img)?"'
   ```

   **Example output**

   ```terminal
   "<url>/art/storage/releases/rhcos-4.22-aarch64/<release>/aarch64/rhcos-<release>-live-kernel-aarch64"
   "<url>/art/storage/releases/rhcos-4.22-aarch64/<release>/aarch64/rhcos-<release>-live-initramfs.aarch64.img"
   "<url>/art/storage/releases/rhcos-4.22-aarch64/<release>/aarch64/rhcos-<release>-live-rootfs.aarch64.img"
   "<url>/art/storage/releases/rhcos-4.22-ppc64le/49.84.202110081256-0/ppc64le/rhcos-<release>-live-kernel-ppc64le"
   "<url>/art/storage/releases/rhcos-4.22-ppc64le/<release>/ppc64le/rhcos-<release>-live-initramfs.ppc64le.img"
   "<url>/art/storage/releases/rhcos-4.22-ppc64le/<release>/ppc64le/rhcos-<release>-live-rootfs.ppc64le.img"
   "<url>/art/storage/releases/rhcos-4.22-s390x/<release>/s390x/rhcos-<release>-live-kernel-s390x"
   "<url>/art/storage/releases/rhcos-4.22-s390x/<release>/s390x/rhcos-<release>-live-initramfs.s390x.img"
   "<url>/art/storage/releases/rhcos-4.22-s390x/<release>/s390x/rhcos-<release>-live-rootfs.s390x.img"
   "<url>/art/storage/releases/rhcos-4.22/<release>/x86_64/rhcos-<release>-live-kernel-x86_64"
   "<url>/art/storage/releases/rhcos-4.22/<release>/x86_64/rhcos-<release>-live-initramfs.x86_64.img"
   "<url>/art/storage/releases/rhcos-4.22/<release>/x86_64/rhcos-<release>-live-rootfs.x86_64.img"
   ```

   :::warning

   The RHCOS artifacts might not change with every release of OpenShift Container Platform. You must download images with the highest version that is less than or equal to the OpenShift Container Platform version that you install. Only use the appropriate `kernel`, `initramfs`, and `rootfs` artifacts described below for this procedure. RHCOS QCOW2 images are not supported for this installation type.

   :::

   The file names contain the OpenShift Container Platform version number. They resemble the following examples:

   - `kernel`: `rhcos-<version>-live-kernel-<architecture>`
   - `initramfs`: `rhcos-<version>-live-initramfs.<architecture>.img`
   - `rootfs`: `rhcos-<version>-live-rootfs.<architecture>.img`
4. Upload the `rootfs`, `kernel`, and `initramfs` files to your HTTP server.
   :::warning

   If you plan to add more compute machines to your cluster after you finish installation, do not delete these files.

   :::
5. Configure the network boot infrastructure so that the machines boot from their local disks after RHCOS is installed on them.
6. Configure PXE or iPXE installation for the RHCOS images and begin the installation.
7. Modify one of the following example menu entries for your environment and verify that the image and Ignition files are properly accessible:
   - For PXE (`x86_64`):

   ```
   DEFAULT pxeboot
   TIMEOUT 20
   PROMPT 0
   LABEL pxeboot
       KERNEL http://<HTTP_server>/rhcos-<version>-live-kernel-<architecture>
       APPEND initrd=http://<HTTP_server>/rhcos-<version>-live-initramfs.<architecture>.img coreos.live.rootfs_url=http://<HTTP_server>/rhcos-<version>-live-rootfs.<architecture>.img coreos.inst.install_dev=/dev/sda coreos.inst.ignition_url=http://<HTTP_server>/bootstrap.ign
   ```

   where:

   <dl>
   <dt><code>kernel</code></dt>
   <dd>Specify the location of the live <code>kernel</code> file that you uploaded to your HTTP server. The URL must be HTTP, TFTP, or FTP; HTTPS and NFS are not supported.</dd>
   <dt><code>initrd=main</code></dt>
   <dd>If you use multiple NICs, specify a single interface in the <code>ip</code> option. For example, to use DHCP on a NIC that is named <code>eno1</code>, set <code>ip=eno1:dhcp</code>. Specify the locations of the RHCOS files that you uploaded to your HTTP server. The <code>initrd</code> parameter value is the location of the <code>initramfs</code> file, the <code>coreos.live.rootfs_url</code> parameter value is the location of the <code>rootfs</code> file, and the <code>coreos.inst.ignition_url</code> parameter value is the location of the bootstrap Ignition config file. You can also add more kernel arguments to the <code>APPEND</code> line to configure networking or other boot options.</dd>
   </dl>

   :::note

   This configuration does not enable serial console access on machines with a graphical console. To configure a different console, add one or more `console=` arguments to the `APPEND` line. For example, add `console=tty0 console=ttyS0` to set the first PC serial port as the primary console and the graphical console as a secondary console. For more information, see [How does one set up a serial terminal and/or console in Red Hat Enterprise Linux?](https://access.redhat.com/articles/7212) and "Enabling the serial console for PXE and ISO installation" in the "Advanced RHCOS installation configuration" section.

   :::

- For iPXE (`x86_64`
  - `aarch64` ):

  ```
  kernel http://<HTTP_server>/rhcos-<version>-live-kernel-<architecture> initrd=main coreos.live.rootfs_url=http://<HTTP_server>/rhcos-<version>-live-rootfs.<architecture>.img coreos.inst.install_dev=/dev/sda coreos.inst.ignition_url=http://<HTTP_server>/bootstrap.ign
  initrd --name main http://<HTTP_server>/rhcos-<version>-live-initramfs.<architecture>.img
  boot
  ```

  <dl>
  <dt><code>kernel</code></dt>
  <dd>Specify the locations of the RHCOS files that you uploaded to your HTTP server. The <code>kernel</code> parameter value is the location of the <code>kernel</code> file, the <code>initrd=main</code> argument is needed for booting on UEFI systems, the <code>coreos.live.rootfs_url</code> parameter value is the location of the <code>rootfs</code> file, and the <code>coreos.inst.ignition_url</code> parameter value is the location of the bootstrap Ignition config file. If you use multiple NICs, specify a single interface in the <code>ip</code> option. For example, to use DHCP on a NIC that is named <code>eno1</code>, set <code>ip=eno1:dhcp</code>.</dd>
  <dt><code>initrd</code></dt>
  <dd>Specify the location of the <code>initramfs</code> file that you uploaded to your HTTP server.</dd>
  </dl>

  :::note

  This configuration does not enable serial console access on machines with a graphical console.  To configure a different console, add one or more `console=` arguments to the `kernel` line.  For example, add `console=tty0 console=ttyS0` to set the first PC serial port as the primary console and the graphical console as a secondary console.  For more information, see [How does one set up a serial terminal and/or console in Red Hat Enterprise Linux?](https://access.redhat.com/articles/7212) and "Enabling the serial console for PXE and ISO installation" in the "Advanced RHCOS installation configuration" section.

  :::

  :::note

  To network boot the CoreOS `kernel` on `aarch64` architecture, you need to use a version of iPXE build with the `IMAGE_GZIP` option enabled. See [`IMAGE_GZIP` option in iPXE](https://ipxe.org/buildcfg/image_gzip).

  :::
- For PXE (with UEFI and Grub as second stage) on `aarch64`:
  ```
  menuentry 'Install CoreOS' {
      linux rhcos-<version>-live-kernel-<architecture>  coreos.live.rootfs_url=http://<HTTP_server>/rhcos-<version>-live-rootfs.<architecture>.img coreos.inst.install_dev=/dev/sda coreos.inst.ignition_url=http://<HTTP_server>/bootstrap.ign
      initrd rhcos-<version>-live-initramfs.<architecture>.img
  }
  ```

  where:

  <dl>
  <dt><code>coreos.live.rootfs_url</code></dt>
  <dd>Specify the locations of the RHCOS files that you uploaded to your HTTP/TFTP server.</dd>
  <dt><code>kernel</code></dt>
  <dd>The <code>kernel</code> parameter value is the location of the <code>kernel</code> file on your TFTP server. The <code>coreos.live.rootfs_url</code> parameter value is the location of the <code>rootfs</code> file, and the <code>coreos.inst.ignition_url</code> parameter value is the location of the bootstrap Ignition config file on your HTTP Server. If you use multiple NICs, specify a single interface in the <code>ip</code> option. For example, to use DHCP on a NIC that is named <code>eno1</code>, set <code>ip=eno1:dhcp</code>.</dd>
  <dt><code>initrd rhcos</code></dt>
  <dd>Specify the location of the <code>initramfs</code> file that you uploaded to your TFTP server.</dd>
  </dl>

1. Monitor the progress of the RHCOS installation on the console of the machine.
   :::warning

   Be sure that the installation is successful on each node before commencing with the OpenShift Container Platform installation. Observing the installation process can also help to determine the cause of RHCOS installation issues that might arise.

   :::
2. After RHCOS installs, the system reboots. During reboot, the system applies the Ignition config file that you specified.
3. Check the console output to verify that Ignition ran.
   ```terminal title="Example command"
   Ignition: ran on 2022/03/14 14:48:33 UTC (this boot)
   Ignition: user-provided config was applied
   ```
4. Continue to create the machines for your cluster.
   :::warning

   You must create the bootstrap and control plane machines at this time. If the control plane machines are not made schedulable, also create at least two compute machines before you install the cluster.

   :::

   If the required network, DNS, and load balancer infrastructure are in place, the OpenShift Container Platform bootstrap process begins automatically after the RHCOS nodes have rebooted.

   :::note

   RHCOS nodes do not include a default password for the  `core` user. You can access the nodes by running `ssh core@<node>.<cluster_name>.<base_domain>` as a user with access to the SSH private key that is paired to the public key that you specified in your `install_config.yaml` file. OpenShift Container Platform 4 cluster nodes running RHCOS are immutable and rely on Operators to apply cluster changes. Accessing cluster nodes by using SSH is not recommended. However, when investigating installation issues, if the OpenShift Container Platform API is not available, or the kubelet is not properly functioning on a target node, SSH access might be required for debugging or disaster recovery.

   :::

### Advanced RHCOS installation configuration {#installation-user-infra-machines-advanced_installing-restricted-networks-bare-metal}

To apply advanced configurations unavailable through default installation methods, manually provision Red Hat Enterprise Linux CoreOS (RHCOS) nodes for OpenShift Container Platform.

This approach enables granular control over the node infrastructure to meet specific deployment requirements.

- Passing kernel arguments to the live installer
- Running `coreos-installer` manually from the live system
- Customizing a live ISO or PXE boot image

The advanced configuration topics for manual Red Hat Enterprise Linux CoreOS (RHCOS) installations detailed in this section relate to disk partitioning, networking, and configuring Ignition in different ways.

#### Using advanced networking options for PXE and ISO installations {#installation-user-infra-machines-advanced_network_installing-restricted-networks-bare-metal}

Networking for OpenShift Container Platform nodes uses DHCP by default to gather all necessary configuration settings. You can set up static IP addresses or configure special settings.

To set up static IP addresses or configure special settings, such as bonding, you can do one of the following:

- Pass special kernel parameters when you boot the live installer.
- Use a machine config to copy networking files to the installed system.
- Configure networking from a live installer shell prompt, then copy those settings to the installed system so that they take effect when the installed system first boots.

To configure a PXE or iPXE installation, use one of the following options:

- See the "coreos-installer and boot options for ISO and PXE installations" tables.
- Use a machine config to copy networking files to the installed system.

To configure an ISO installation, use the following procedure.

**Procedure**

1. Boot the ISO installer.
2. From the live system shell prompt, configure networking for the live system by using available RHEL tools, such as `nmcli` or `nmtui`.
3. Run the `coreos-installer` command to install the system, adding the `--copy-network` option to copy networking configuration. For example:
   ```terminal
   $ sudo coreos-installer install --copy-network \
   --ignition-url=http://host/worker.ign \
   --offline \
   /dev/disk/by-id/scsi-<serial_number>
   ```

   :::warning

   The `--copy-network` option only copies networking configuration found under `/etc/NetworkManager/system-connections`. In particular, it does not copy the system hostname.

   :::
4. Reboot into the installed system.

#### Disk partitioning {#installation-user-infra-machines-advanced-disk_installing-restricted-networks-bare-metal}

Disk partitions are created on OpenShift Container Platform cluster nodes during the Red Hat Enterprise Linux CoreOS (RHCOS) installation. Each RHCOS node of a particular architecture uses the same partition layout, unless you override the default partitioning configuration.

During the RHCOS installation, the size of the root file system is increased to use any remaining available space on the target device.

:::warning

The use of a custom partition scheme on your node might result in OpenShift Container Platform not monitoring or alerting on some node partitions. For more information on monitoring host file systems when using custom partitioning, see [Understanding OpenShift File System Monitoring (eviction conditions)](https://access.redhat.com/articles/4766521).

:::

OpenShift Container Platform monitors the following two filesystem identifiers:

- `nodefs`, which is the filesystem that contains `/var/lib/kubelet`.
- `imagefs`, which is the filesystem that contains `/var/lib/containers`.

For the default partition scheme, `nodefs` and `imagefs` monitor the same root filesystem, `/`.

To override the default partitioning when installing RHCOS on an OpenShift Container Platform cluster node, you must create separate partitions. Consider a situation where you want to add a separate storage partition for your containers and container images. For example, by mounting `/var/lib/containers` in a separate partition, the kubelet separately monitors `/var/lib/containers` as the `imagefs` directory and the root file system as the `nodefs` directory.

:::warning

If you have resized your disk size to host a larger file system, consider creating a separate `/var/lib/containers` partition. Consider resizing a disk that has an `xfs` format to reduce CPU time issues caused by a high number of allocation groups.

:::

OpenShift Container Platform supports the addition of a single partition to attach storage to either the `/var` directory or a subdirectory of `/var`. For example:

- `/var/lib/containers`: Holds container-related content that can grow as more images and containers are added to a system.
- `/var/lib/etcd`: Holds data that you might want to keep separate for purposes such as performance optimization of etcd storage.
- `/var`: Holds data that you might want to keep separate for purposes such as auditing.
  :::warning

  For disk sizes larger than 100GB, and especially larger than 1TB, create a separate `/var` partition.

  :::

Storing the contents of a `/var` directory separately makes it easier to grow storage for those areas as needed and reinstall OpenShift Container Platform at a later date to keep that data intact. This method eliminates the need to re-pull containers or copy large log files during system updates.

The use of a separate partition for the `/var` directory or a subdirectory of `/var` also prevents data growth in the partitioned directory from filling up the root file system.

The following procedure sets up a separate `/var` partition by adding a machine config manifest that is wrapped into the Ignition config file for a node type during the preparation phase of an installation.

**Procedure**

1. On your installation host, change to the directory that contains the OpenShift Container Platform installation program and generate the Kubernetes manifests for the cluster:
   ```terminal
   $ openshift-install create manifests --dir <installation_directory>
   ```
2. Create a Butane config that configures the additional partition. For example, name the file `$HOME/clusterconfig/98-var-partition.bu`, change the disk device name to the name of the storage device on the `worker` systems, and set the storage size as appropriate. This example places the `/var` directory on a separate partition:
   ```yaml
   variant: openshift
   version: 4.22.0
   metadata:
     labels:
       machineconfiguration.openshift.io/role: worker
     name: 98-var-partition
   storage:
     disks:
     - device: /dev/disk/by-id/<device_name>
       partitions:
       - label: var
         start_mib: <partition_start_offset>
         size_mib: <partition_size>
         number: 5
     filesystems:
       - device: /dev/disk/by-partlabel/var
         path: /var
         format: xfs
         mount_options: [defaults, prjquota]
         with_mount_unit: true
   ```

   where:

   <dl>
   <dt><code>&lt;device_name&gt;</code></dt>
   <dd>Specifies the storage device name of the disk that you want to partition.</dd>
   <dt><code>&lt;partition_start_offset&gt;</code></dt>
   <dd>Specifies the minimum offset value for the boot disk. For best performance, specify a minimum offset value of 25000 mebibytes. The root file system is automatically resized to fill all available space up to the specified offset. If no offset value is specified, or if the specified value is smaller than the recommended minimum, the resulting root file system will be too small, and future reinstalls of RHCOS might overwrite the beginning of the data partition.</dd>
   <dt><code>&lt;partition_size&gt;</code></dt>
   <dd>Specifies the size of the data partition in mebibytes.</dd>
   <dt><code>mount_options</code></dt>
   <dd>The <code>prjquota</code> mount option must be enabled for filesystems used for container storage.</dd>
   </dl>

   :::note

   When creating a separate `/var` partition, you cannot use different instance types for compute nodes, if the different instance types do not have the same device name.

   :::
3. Create a manifest from the Butane config and save it to the `clusterconfig/openshift` directory. For example, run the following command:
   ```terminal
   $ butane $HOME/clusterconfig/98-var-partition.bu -o $HOME/clusterconfig/openshift/98-var-partition.yaml
   ```
4. Create the Ignition config files by running the following command:
   ```terminal
   $ openshift-install create ignition-configs --dir <installation_directory>
   ```

   where:

   <dl>
   <dt><code>&lt;installation_directory&gt;</code></dt>
   <dd>Specifies the name of the installation directory. Ignition config files are created for the bootstrap, control plane, and compute nodes in the installation directory:</dd>
   </dl>

   ```
   .
   ├── auth
   │   ├── kubeadmin-password
   │   └── kubeconfig
   ├── bootstrap.ign
   ├── master.ign
   ├── metadata.json
   └── worker.ign
   ```

   The files in the `<installation_directory>/manifest` and `<installation_directory>/openshift` directories are wrapped into the Ignition config files, including the file that contains the `98-var-partition` custom `MachineConfig` object.
5. Optional: You can apply the custom disk partitioning by referencing the Ignition config files during the RHCOS installations.

#### Examples of retaining existing partitions {#installation-user-infra-machines-advanced-retain-disk_installing-restricted-networks-bare-metal}

For an ISO installation, you can add options to the `coreos-installer` command that causes the installation program to maintain one or more existing partitions. For a PXE installation, you can add `coreos.inst.*` options to the `APPEND` parameter to preserve partitions.

Saved partitions might be data partitions from an existing OpenShift Container Platform system. You can identify the disk partitions you want to keep either by partition label or by number.

:::note

If you save existing partitions, and those partitions do not leave enough space for RHCOS, the installation fails without damaging the saved partitions.

:::

The following examples preserve any existing partition during an ISO installation in which the partition label begins with `data` (`data*`):

```terminal
# coreos-installer install --ignition-url http://10.0.2.2:8080/user.ign \
--save-partlabel 'data*' \
--offline \
/dev/disk/by-id/scsi-<serial_number>
```

The following example runs the `coreos-installer` in a way that preserves the sixth (6) partition on the disk:

```terminal
# coreos-installer install --ignition-url http://10.0.2.2:8080/user.ign \
--save-partindex 6 \
--offline \
/dev/disk/by-id/scsi-<serial_number>
```

The following example preserves partitions 5 and higher:

```terminal
# coreos-installer install --ignition-url http://10.0.2.2:8080/user.ign \
--save-partindex 5- \
--offline \
/dev/disk/by-id/scsi-<serial_number>
```

In the earlier examples where partition saving is used, `coreos-installer` recreates the partition immediately.

The following examples preserve existing partitions during a PXE installation. The following `APPEND` option preserves any partition in which the partition label begins with 'data' ('data*').

```terminal
coreos.inst.save_partlabel=data*
```

The following `APPEND` option preserves partitions 5 and higher:

```terminal
coreos.inst.save_partindex=5-
```

The following `APPEND` option preserves partition 6:

```terminal
coreos.inst.save_partindex=6
```

#### Identifying Ignition configs {#installation-user-infra-machines-advanced-ignition_installing-restricted-networks-bare-metal}

When doing an RHCOS manual installation, there are two types of Ignition configs that you can provide: Permanent install Ignition config and live install Ignition config.

When manually installing RHCOS, you can provide the following two types of Ignition configs:

- **Permanent install Ignition config**: Every manual RHCOS installation needs to pass one of the Ignition config files generated by `openshift-installer`, such as `bootstrap.ign`, `master.ign` and `worker.ign`, to carry out the installation.

:::warning

Do not modify these Ignition config files directly. You can update the manifest files that are wrapped into the Ignition config files, as outlined in examples in the preceding sections.

:::

For PXE installations, you can pass the Ignition configs on the `APPEND` line using the `coreos.inst.ignition_url=` option. For ISO installations, after the ISO boots to the shell prompt, you must identify the Ignition config on the `coreos-installer` command line with the `--ignition-url=` option. In both cases, only HTTP and HTTPS protocols are supported.

- **Live install Ignition config**: This type can be created by using the `coreos-installer` `customize` subcommand of `coreos-installer` and its various options. With this method, the Ignition config passes to the live install medium, runs immediately upon booting, and performs setup tasks before or after the RHCOS system installs to disk. This method must be only used for performing tasks that must be done once and not applied again later, such as with advanced partitioning that cannot be done using a machine config.

For PXE or ISO boots, you can create the Ignition config and `APPEND` the `ignition.config.url=` option to identify the location of the Ignition config. You also need to append `ignition.firstboot ignition.platform.id=metal` else the `ignition.config.url` option is ignored.

**Additional resources**

- [Getting started with nmcli](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/configuring_and_managing_networking/index#getting-started-with-nmcli_configuring-and-managing-networking)
- [Getting started with nmtui](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/configuring_and_managing_networking/index#getting-started-with-nmtui_configuring-and-managing-networking)

#### Default console configuration {#installation-user-infra-machines-advanced-console-configuration_installing-restricted-networks-bare-metal}

Red Hat Enterprise Linux CoreOS (RHCOS) nodes installed from an OpenShift Container Platform 4.22 boot image use a default console that is meant to accommodate most virtualized and bare metal setups.

Different cloud and virtualization platforms may use different default settings depending on the chosen architecture.

Bare-metal installations use the kernel default settings which typically means the graphical console is the primary console and the serial console is disabled.

The default consoles may not match your specific hardware configuration or you might have specific needs that require you to adjust the default console. For example:

- You want to access the emergency shell on the console for debugging purposes.
- Your cloud platform does not provide interactive access to the graphical console, but provides a serial console.
- You want to enable multiple consoles.

Console configuration is inherited from the boot image. This means that new nodes in existing clusters are unaffected by changes to the default console.

You can configure the console for bare metal installations in the following ways:

- Using `coreos-installer` manually on the command line.
- Using the `coreos-installer iso customize` or `coreos-installer pxe customize` subcommands with the `--dest-console` option to create a custom image that automates the process.

:::note

For advanced customization, perform console configuration using the `coreos-installer iso` or `coreos-installer pxe` subcommands, and not kernel arguments.

:::

#### Enabling the serial console for PXE and ISO installations {#installation-user-infra-machines-advanced-enabling-serial-console_installing-restricted-networks-bare-metal}

By default, the Red Hat Enterprise Linux CoreOS (RHCOS) serial console is disabled and all output is written to the graphical console. You can enable the serial console for PXE and ISO installations.

**Procedure**

1. Boot the ISO installer.
2. Run the `coreos-installer` command to install the system, adding the `--console` option once to specify the graphical console, and a second time to specify the serial console:
   ```terminal
   $ coreos-installer install \
   --console=tty0 \
   --console=ttyS0,<options> \
   --ignition-url=http://host/worker.ign \
   --offline \
   /dev/disk/by-id/scsi-<serial_number>
   ```

   where:

   <dl>
   <dt><code>--console=tty0</code></dt>
   <dd>The desired secondary console. In this case, the graphical console. Omitting this option will disable the graphical console.</dd>
   <dt><code>--console=ttyS0</code></dt>
   <dd>The desired primary console. In this case, the serial console. The <code>options</code> field defines the baud rate and other settings. A common value for this field is <code>115200n8</code>. If no options are provided, the default kernel value of <code>9600n8</code> is used. For more information on the format of this option, see <a href="https://www.kernel.org/doc/html/latest/admin-guide/serial-console.html">Linux kernel serial console</a> documentation.</dd>
   </dl>
3. Reboot into the installed system.
   :::note

   A similar outcome can be obtained by using the `coreos-installer install --append-karg` option, and specifying the console with `console=`. However, this will only set the console for the kernel and not the bootloader.

   :::

   To configure a PXE installation, make sure the `coreos.inst.install_dev` kernel command-line option is omitted, and use the shell prompt to run `coreos-installer` manually using the above ISO installation procedure.

#### Customizing a live RHCOS ISO or PXE install {#installation-user-infra-machines-advanced-customizing-iso-or-pxe_installing-restricted-networks-bare-metal}

You can use the live ISO image or PXE environment to install RHCOS by injecting an Ignition config file directly into the image. This creates a customized image that you can use to provision your system.

For an ISO image, the mechanism to do this is the `coreos-installer iso customize` subcommand, which modifies the `.iso` file with your configuration. Similarly, the mechanism for a PXE environment is the `coreos-installer pxe customize` subcommand, which creates a new `initramfs` file that includes your customizations.

The `customize` subcommand is a general-purpose tool that can embed other types of customizations as well. The following tasks are examples of some of the more common customizations:

- Inject custom CA certificates for when corporate security policy requires their use.
- Configure network settings without the need for kernel arguments.
- Embed arbitrary pre-install and post-install scripts or binaries.

#### Customizing a live RHCOS ISO image {#installation-user-infra-machines-advanced-customizing-live-iso_installing-restricted-networks-bare-metal}

You can customize a live RHCOS ISO image directly with the `coreos-installer iso customize` subcommand. When you boot the ISO image, the customizations are applied automatically. You can use this feature to configure the ISO image to automatically install RHCOS.

**Procedure**

1. Download the `coreos-installer` binary from the [`coreos-installer` image mirror](https://mirror.openshift.com/pub/openshift-v4/clients/coreos-installer/latest/) page.
2. Retrieve the RHCOS ISO image from the [RHCOS image mirror](https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/latest/) page and the Ignition config file, and then run the following command to inject the Ignition config directly into the ISO image:
   ```terminal
   $ coreos-installer iso customize rhcos-<version>-live.x86_64.iso \
       --dest-ignition bootstrap.ign \
       --dest-device /dev/disk/by-id/scsi-<serial_number>
   ```

   where:

   <dl>
   <dt><code>--dest-ignition</code></dt>
   <dd>Specifies the Ignition config file that is generated from the <code>openshift-installer</code> installation program.</dd>
   <dt><code>--dest-device</code></dt>
   <dd>When you specify this option, the ISO image automatically runs an installation. Otherwise, the image remains configured for installation, but does not install automatically unless you specify the <code>coreos.inst.install_dev</code> kernel argument.</dd>
   </dl>
3. Optional: To remove the ISO image customizations and return the image to its pristine state, run:
   ```terminal
   $ coreos-installer iso reset rhcos-<version>-live.x86_64.iso
   ```

   You can now re-customize the live ISO image or use it in its pristine state.

   Applying your customizations affects every subsequent boot of RHCOS.

##### Modifying a live install ISO image to enable the serial console {#installation-user-infra-machines-advanced-customizing-live-iso-serial-console_installing-restricted-networks-bare-metal}

To redirect system output from the default graphical interface, enable the serial console by modifying the live install ISO image. This configuration ensures access to boot messages on OpenShift Container Platform 4.12 and later clusters.

**Procedure**

1. Download the `coreos-installer` binary from the [`coreos-installer` image mirror](https://mirror.openshift.com/pub/openshift-v4/clients/coreos-installer/latest/) page.
2. Retrieve the RHCOS ISO image from the [RHCOS image mirror](https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/latest/) page and run the following command to customize the ISO image to enable the serial console to receive output:
   ```terminal
   $ coreos-installer iso customize rhcos-<version>-live.x86_64.iso \
     --dest-ignition <path> \
     --dest-console tty0 \
     --dest-console ttyS0,<options> \
     --dest-device /dev/disk/by-id/scsi-<serial_number>
   ```

   where:

   <dl>
   <dt><code>&lt;path&gt;</code></dt>
   <dd>The location of the Ignition config to install.</dd>
   <dt><code>tty0</code></dt>
   <dd>The desired secondary console. In this case, the graphical console. Omitting this option will disable the graphical console.</dd>
   <dt><code>&lt;options&gt;</code></dt>
   <dd>The desired primary console. In this case, the serial console. The <code>options</code> field defines the baud rate and other settings. A common value for this field is <code>115200n8</code>. If no options are provided, the default kernel value of <code>9600n8</code> is used. For more information on the format of this option, see the <a href="https://www.kernel.org/doc/html/latest/admin-guide/serial-console.html">Linux kernel serial console</a> documentation.</dd>
   <dt><code>&lt;serial_number&gt;</code></dt>
   <dd>The specified disk to install to. If you omit this option, the ISO image automatically runs the installation program which will fail unless you also specify the <code>coreos.inst.install_dev</code> kernel argument.</dd>
   </dl>

   :::note

   The `--dest-console` option affects the installed system and not the live ISO system. To modify the console for a live ISO system, use the `--live-karg-append` option and specify the console with `console=`.

   :::

   Your customizations are applied and affect every subsequent boot of the ISO image.
3. Optional: To remove the ISO image customizations and return the image to its original state, run the following command:
   ```terminal
   $ coreos-installer iso reset rhcos-<version>-live.x86_64.iso
   ```

   You can now recustomize the live ISO image or use it in its original state.

##### Modifying a live install ISO image to use a custom certificate authority {#installation-user-infra-machines-advanced-customizing-live-iso-ca-certs_installing-restricted-networks-bare-metal}

You can provide certificate authority (CA) certificates to Ignition with the `--ignition-ca` flag of the `customize` subcommand. You can use the CA certificates during both the installation boot and when provisioning the installed system.

:::note

Custom CA certificates affect how Ignition fetches remote resources, but they do not affect the certificates installed onto the system.

:::

**Procedure**

1. Download the `coreos-installer` binary from the [`coreos-installer` image mirror](https://mirror.openshift.com/pub/openshift-v4/clients/coreos-installer/latest/) page.
2. Retrieve the RHCOS ISO image from the [RHCOS image mirror](https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/latest/) page, and run the following command to customize the ISO image for use with a custom CA:
   ```terminal
   $ coreos-installer iso customize rhcos-<version>-live.x86_64.iso --ignition-ca cert.pem
   ```

   :::warning

   The `coreos.inst.ignition_url` kernel parameter does not work with the `--ignition-ca` flag. You must use the `--dest-ignition` flag to create a customized image for each cluster.

   :::

   Applying your custom CA certificate affects every subsequent boot of RHCOS.

##### Modifying a live install ISO image with customized network settings {#installation-user-infra-machines-advanced-customizing-live-iso_network_keyfile_installing-restricted-networks-bare-metal}

You can embed a NetworkManager keyfile into the live ISO image and pass it through to the installed system with the `--network-keyfile` flag of the `customize` subcommand.

:::warning

When creating a connection profile, you must use a `.nmconnection` filename extension in the filename of the connection profile. If you do not use a `.nmconnection` filename extension, the cluster will apply the connection profile to the live environment, but it will not apply the configuration when the cluster first boots up the nodes, resulting in a setup that does not work.

:::

**Procedure**

1. Download the `coreos-installer` binary from the [`coreos-installer` image mirror](https://mirror.openshift.com/pub/openshift-v4/clients/coreos-installer/latest/) page.
2. Create a connection profile for a bonded interface. For example, create the `bond0.nmconnection` file in your local directory with the following content:
   ```terminal
   /[connection]
   id=bond0
   type=bond
   interface-name=bond0
   multi-connect=1

   /[bond]
   miimon=100
   mode=active-backup

   /[ipv4]
   method=auto

   /[ipv6]
   method=auto
   ```
3. Create a connection profile for a secondary interface to add to the bond. For example, create the `bond0-proxy-em1.nmconnection` file in your local directory with the following content:
   ```terminal
   /[connection]
   id=em1
   type=ethernet
   interface-name=em1
   master=bond0
   multi-connect=1
   slave-type=bond
   ```
4. Create a connection profile for a secondary interface to add to the bond. For example, create the `bond0-proxy-em2.nmconnection` file in your local directory with the following content:
   ```terminal
   /[connection]
   id=em2
   type=ethernet
   interface-name=em2
   master=bond0
   multi-connect=1
   slave-type=bond
   ```
5. Retrieve the RHCOS ISO image from the [RHCOS image mirror](https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/latest/) page and run the following command to customize the ISO image with your configured networking:
   ```terminal
   $ coreos-installer iso customize rhcos-<version>-live.x86_64.iso \
       --network-keyfile bond0.nmconnection \
       --network-keyfile bond0-proxy-em1.nmconnection \
       --network-keyfile bond0-proxy-em2.nmconnection
   ```

   Network settings are applied to the live system and are carried over to the destination system.

##### Customizing a live install ISO image for an iSCSI boot device {#installation-user-infra-machines-advanced-customizing-live-iso-iscsi-manual_installing-restricted-networks-bare-metal}

You can set the iSCSI target and initiator values for automatic mounting, booting and configuration by using a customized version of the live RHCOS image.

**Prerequisites**

1. You have an iSCSI target you want to install RHCOS on.

**Procedure**

1. Download the `coreos-installer` binary from the [`coreos-installer` image mirror](https://mirror.openshift.com/pub/openshift-v4/clients/coreos-installer/latest/) page.
2. Retrieve the RHCOS ISO image from the [RHCOS image mirror](https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/latest/) page and run the following command to customize the ISO image with the following information:
   ```text
   $ coreos-installer iso customize \
       --pre-install mount-iscsi.sh \
       --post-install unmount-iscsi.sh \
       --dest-device /dev/disk/by-path/<IP_address>:<port>-iscsi-<target_iqn>-lun-<lun> \
       --dest-ignition config.ign \
       --dest-karg-append rd.iscsi.initiator=<initiator_iqn> \
       --dest-karg-append netroot=<target_iqn> \
       -o custom.iso rhcos-<version>-live.x86_64.iso
   ```

   where:

   <dl>
   <dt><code>mount-iscsi.sh</code></dt>
   <dd>Specifies the script that gets run before installation. It should contain the <code>iscsiadm</code> commands for mounting the iSCSI target and any commands enabling multipathing.</dd>
   <dt><code>unmount-iscsi.sh</code></dt>
   <dd>Specifies the script that gets run after installation. It should contain the command <code>iscsiadm --mode node --logout=all</code>.</dd>
   <dt><code>&lt;target_iqn&gt;</code></dt>
   <dd>Specifies the location of the destination system. You must provide the IP address of the target portal, the associated port number, the target iSCSI node in IQN format, and the iSCSI logical unit number (LUN).</dd>
   <dt><code>config.ign</code></dt>
   <dd>Specifies the Ignition configuration for the destination system. <code>&lt;initiator_iqn&gt;</code>:Specifies[] the iSCSI initiator, or client, name in IQN format. The initiator forms a session to connect to the iSCSI target. <code>&lt;target_iqn&gt;</code>:Specifies[] the iSCSI target, or server, name in IQN format.</dd>
   </dl>

   For more information about the iSCSI options supported by `dracut`, see the `dracut.cmdline` manual page.

**Additional resources**

- [`dracut.cmdline` manual page](https://www.man7.org/linux/man-pages/man7/dracut.cmdline.7.html)

##### Customizing a live install ISO image for an iSCSI boot device with iBFT {#installation-user-infra-machines-advanced-customizing-live-iso-iscsi-ibft_installing-restricted-networks-bare-metal}

You can set the iSCSI target and initiator values for automatic mounting, booting and configuration using a customized version of the live RHCOS image.

**Prerequisites**

1. You have an iSCSI target you want to install RHCOS on.
2. Optional: You have multipathed your iSCSI target.

**Procedure**

1. Download the `coreos-installer` binary from the [`coreos-installer` image mirror](https://mirror.openshift.com/pub/openshift-v4/clients/coreos-installer/latest/) page.
2. Retrieve the RHCOS ISO image from the [RHCOS image mirror](https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/latest/) page and run the following command to customize the ISO image with the following information:
   ```text
   $ coreos-installer iso customize \
       --pre-install mount-iscsi.sh \
       --post-install unmount-iscsi.sh \
       --dest-device /dev/mapper/mpatha \
       --dest-ignition config.ign \
       --dest-karg-append rd.iscsi.firmware=1 \
       --dest-karg-append rd.multipath=default \
       -o custom.iso rhcos-<version>-live.x86_64.iso
   ```

   where:

   <dl>
   <dt><code>mount-iscsi.sh</code></dt>
   <dd>Specifies the script that gets run before installation. It should contain the <code>iscsiadm</code> commands for mounting the iSCSI target and any commands enabling multipathing.</dd>
   <dt><code>unmount-iscsi.sh</code></dt>
   <dd>Specifies the script that gets run after installation. It should contain the command <code>iscsiadm --mode node --logout=all</code>.</dd>
   <dt><code>/dev/mapper/mpatha</code></dt>
   <dd>Specifies the path to the device. If you are using multipath, the multipath device, <code>/dev/mapper/mpatha</code>, If there are multiple multipath devices connected, or to be explicit, you can use the World Wide Name (WWN) symlink available in <code>/dev/disk/by-path</code>.</dd>
   <dt><code>config.ign</code></dt>
   <dd>Specifies the Ignition configuration for the destination system. <code>rd.iscsi.firmware=1</code>:Specifies[] the iSCSI parameter is read from the BIOS firmware.</dd>
   <dt><code>rd.multipath=default</code></dt>
   <dd>Specifies if you want to enable multipathing. Optional parameter.</dd>
   </dl>

   For more information about see the `dracut.cmdline` manual page.

**Additional resources**

- [`dracut.cmdline` manual page](https://www.man7.org/linux/man-pages/man7/dracut.cmdline.7.html)

#### Customizing a live RHCOS PXE environment {#installation-user-infra-machines-advanced-customizing-live-pxe_installing-restricted-networks-bare-metal}

You can customize a live RHCOS PXE environment directly with the `coreos-installer pxe customize` subcommand. When you boot the PXE environment, the customizations are applied automatically. You can use this feature to configure the PXE environment to automatically install RHCOS.

**Procedure**

1. Download the `coreos-installer` binary from the [`coreos-installer` image mirror](https://mirror.openshift.com/pub/openshift-v4/clients/coreos-installer/latest/) page.
2. Retrieve the RHCOS `kernel`, `initramfs`, and `rootfs` files from the [RHCOS image mirror](https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/latest/) page and the Ignition config file, and then run the following command to create a new `initramfs` file that contains the customizations from your Ignition config:
   ```terminal
   $ coreos-installer pxe customize rhcos-<version>-live-initramfs.x86_64.img \
       --dest-ignition bootstrap.ign \
       --dest-device /dev/disk/by-id/scsi-<serial_number> \
       -o rhcos-<version>-custom-initramfs.x86_64.img
   ```

   where:

   <dl>
   <dt><code>--dest-ignition</code></dt>
   <dd>Specifies the Ignition config file that is generated from <code>openshift-installer</code>.</dd>
   <dt><code>&lt;serial_number&gt;</code></dt>
   <dd>When you specify this option, the PXE environment automatically runs an install. Otherwise, the image remains configured for installation, but does not do so automatically unless you specify the <code>coreos.inst.install_dev</code> kernel argument.</dd>
   <dt><code>&lt;version&gt;</code></dt>
   <dd>Use the customized <code>initramfs</code> file in your PXE configuration. Add the <code>ignition.firstboot</code> and <code>ignition.platform.id=metal</code> kernel arguments if they are not already present.</dd>
   </dl>

   Applying your customizations affects every subsequent boot of RHCOS.

##### Modifying a live install PXE environment to enable the serial console {#installation-user-infra-machines-advanced-customizing-live-pxe-serial-console_installing-restricted-networks-bare-metal}

To redirect system output from the default graphical interface, enable the serial console by modifying the live install PXE environment. This configuration ensures access to boot messages on OpenShift Container Platform 4.12 and later clusters.

**Procedure**

1. Download the `coreos-installer` binary from the [`coreos-installer` image mirror](https://mirror.openshift.com/pub/openshift-v4/clients/coreos-installer/latest/) page.
2. Retrieve the RHCOS `kernel`, `initramfs`, and `rootfs` files from the [RHCOS image mirror](https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/latest/) page and the Ignition config file, and then run the following command to create a new customized `initramfs` file that enables the serial console to receive output:
   ```terminal
   $ coreos-installer pxe customize rhcos-<version>-live-initramfs.x86_64.img \
     --dest-ignition <path> \
     --dest-console tty0 \
     --dest-console ttyS0,<options> \
     --dest-device /dev/disk/by-id/scsi-<serial_number> \
     -o rhcos-<version>-custom-initramfs.x86_64.img
   ```

   where:

   <dl>
   <dt><code>&lt;path&gt;</code></dt>
   <dd>The location of the Ignition config to install.</dd>
   <dt><code>tty0</code></dt>
   <dd>The desired secondary console. In this case, the graphical console. Omitting this option will disable the graphical console.</dd>
   <dt><code>&lt;options&gt;</code></dt>
   <dd>The desired primary console. In this case, the serial console. The <code>options</code> field defines the baud rate and other settings. A common value for this field is <code>115200n8</code>. If no options are provided, the default kernel value of <code>9600n8</code> is used. For more information on the format of this option, see the <a href="https://www.kernel.org/doc/html/latest/admin-guide/serial-console.html">Linux kernel serial console</a> documentation.</dd>
   <dt><code>&lt;serial_number&gt;</code></dt>
   <dd>The specified disk to install to. If you omit this option, the PXE environment automatically runs the installation program which will fail unless you also specify the <code>coreos.inst.install_dev</code> kernel argument.</dd>
   <dt><code>&lt;version&gt;</code></dt>
   <dd>Use the customized <code>initramfs</code> file in your PXE configuration. Add the <code>ignition.firstboot</code> and <code>ignition.platform.id=metal</code> kernel arguments if they are not already present. Your customizations are applied and affect every subsequent boot of the PXE environment.</dd>
   </dl>

##### Modifying a live install PXE environment to use a custom certificate authority {#installation-user-infra-machines-advanced-customizing-live-pxe-ca-certs_installing-restricted-networks-bare-metal}

You can provide certificate authority (CA) certificates to Ignition with the `--ignition-ca` flag of the `customize` subcommand. You can use the CA certificates during both the installation boot and when provisioning the installed system.

:::note

Custom CA certificates affect how Ignition fetches remote resources, but they do not affect the certificates installed onto the system.

:::

**Procedure**

1. Download the `coreos-installer` binary from the [`coreos-installer` image mirror](https://mirror.openshift.com/pub/openshift-v4/clients/coreos-installer/latest/) page.
2. Retrieve the RHCOS `kernel`, `initramfs`, and `rootfs` files from the [RHCOS image mirror](https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/latest/) page, and run the following command to create a new customized `initramfs` file for use with a custom CA:
   ```terminal
   $ coreos-installer pxe customize rhcos-<version>-live-initramfs.x86_64.img \
       --ignition-ca cert.pem \
       -o rhcos-<version>-custom-initramfs.x86_64.img
   ```
3. Use the customized `initramfs` file in your PXE configuration. Add the `ignition.firstboot` and `ignition.platform.id=metal` kernel arguments if they are not already present.
   :::warning

   The `coreos.inst.ignition_url` kernel parameter does not work with the `--ignition-ca` flag. You must use the `--dest-ignition` flag to create a customized image for each cluster.

   :::

   Applying your custom CA certificate affects every subsequent boot of RHCOS.

##### Modifying a live install PXE environment with customized network settings {#installation-user-infra-machines-advanced-customizing-live-pxe_network_keyfile_installing-restricted-networks-bare-metal}

You can embed a NetworkManager keyfile into the live PXE environment and pass it through to the installed system with the `--network-keyfile` flag of the `customize` subcommand.

:::warning

When creating a connection profile, you must use a `.nmconnection` filename extension in the filename of the connection profile. If you do not use a `.nmconnection` filename extension, the cluster will apply the connection profile to the live environment, but it will not apply the configuration when the cluster first boots up the nodes, resulting in a setup that does not work.

:::

**Procedure**

1. Download the `coreos-installer` binary from the [`coreos-installer` image mirror](https://mirror.openshift.com/pub/openshift-v4/clients/coreos-installer/latest/) page.
2. Create a connection profile for a bonded interface. For example, create the `bond0.nmconnection` file in your local directory with the following content:
   ```terminal
   /[connection]
   id=bond0
   type=bond
   interface-name=bond0
   multi-connect=1

   /[bond]
   miimon=100
   mode=active-backup

   /[ipv4]
   method=auto

   /[ipv6]
   method=auto
   ```
3. Create a connection profile for a secondary interface to add to the bond. For example, create the `bond0-proxy-em1.nmconnection` file in your local directory with the following content:
   ```terminal
   /[connection]
   id=em1
   type=ethernet
   interface-name=em1
   master=bond0
   multi-connect=1
   slave-type=bond
   ```
4. Create a connection profile for a secondary interface to add to the bond. For example, create the `bond0-proxy-em2.nmconnection` file in your local directory with the following content:
   ```terminal
   /[connection]
   id=em2
   type=ethernet
   interface-name=em2
   master=bond0
   multi-connect=1
   slave-type=bond
   ```
5. Retrieve the RHCOS `kernel`, `initramfs`, and `rootfs` files from the [RHCOS image mirror](https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/latest/) page and run the following command to create a new customized `initramfs` file that contains your configured networking:
   ```terminal
   $ coreos-installer pxe customize rhcos-<version>-live-initramfs.x86_64.img \
       --network-keyfile bond0.nmconnection \
       --network-keyfile bond0-proxy-em1.nmconnection \
       --network-keyfile bond0-proxy-em2.nmconnection \
       -o rhcos-<version>-custom-initramfs.x86_64.img
   ```
6. Use the customized `initramfs` file in your PXE configuration. Add the `ignition.firstboot` and `ignition.platform.id=metal` kernel arguments if they are not already present.
   Network settings are applied to the live system and are carried over to the destination system.

##### Customizing a live install PXE environment for an iSCSI boot device {#installation-user-infra-machines-advanced-customizing-live-pxe-iscsi-manual_installing-restricted-networks-bare-metal}

You can set the iSCSI target and initiator values for automatic mounting, booting and configuration by using a customized version of the live RHCOS image.

**Prerequisites**

1. You have an iSCSI target you want to install RHCOS on.

**Procedure**

1. Download the `coreos-installer` binary from the [`coreos-installer` image mirror](https://mirror.openshift.com/pub/openshift-v4/clients/coreos-installer/latest/) page.
2. Retrieve the RHCOS `kernel`, `initramfs`, and `rootfs` files from the [RHCOS image mirror](https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/latest/) page and run the following command to create a new customized `initramfs` file with the following information:
   ```text
   $ coreos-installer pxe customize \
       --pre-install mount-iscsi.sh \
       --post-install unmount-iscsi.sh \
       --dest-device /dev/disk/by-path/<IP_address>:<port>-iscsi-<target_iqn>-lun-<lun> \
       --dest-ignition config.ign \
       --dest-karg-append rd.iscsi.initiator=<initiator_iqn> \
       --dest-karg-append netroot=<target_iqn> \
       -o custom.img rhcos-<version>-live-initramfs.x86_64.img
   ```

   where:

   <dl>
   <dt><code>mount-iscsi.sh</code></dt>
   <dd>Specifies the script that gets run before installation. It should contain the <code>iscsiadm</code> commands for mounting the iSCSI target and any commands enabling multipathing.</dd>
   <dt><code>unmount-iscsi.sh</code></dt>
   <dd>Specifies the script that gets run after installation. It should contain the command <code>iscsiadm --mode node --logout=all</code>.</dd>
   <dt><code>&lt;target_iqn&gt;</code></dt>
   <dd>Specifies the location of the destination system. You must provide the IP address of the target portal, the associated port number, the target iSCSI node in IQN format, and the iSCSI logical unit number (LUN).</dd>
   <dt><code>config.ign</code></dt>
   <dd>Specifies the Ignition configuration for the destination system.</dd>
   <dt><code>&lt;initiator_iqn&gt;</code></dt>
   <dd>Specifies the iSCSI initiator, or client, name in IQN format. The initiator forms a session to connect to the iSCSI target.</dd>
   <dt><code>&lt;target_iqn&gt;</code></dt>
   <dd>Specifies the iSCSI target, or server, name in IQN format.</dd>
   </dl>

   For more information about the iSCSI options supported by `dracut`, see the `dracut.cmdline` manual page.

##### Customizing a live install PXE environment for an iSCSI boot device with iBFT {#installation-user-infra-machines-advanced-customizing-live-pxe-iscsi-ibft_installing-restricted-networks-bare-metal}

You can set the iSCSI target and initiator values for automatic mounting, booting and configuration using a customized version of the live RHCOS image.

**Prerequisites**

1. You have an iSCSI target you want to install RHCOS on.
2. Optional: You have multipathed your iSCSI target.

**Procedure**

1. Download the `coreos-installer` binary from the [`coreos-installer` image mirror](https://mirror.openshift.com/pub/openshift-v4/clients/coreos-installer/latest/) page.
2. Retrieve the RHCOS `kernel`, `initramfs`, and `rootfs` files from the [RHCOS image mirror](https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/latest/) page and run the following command to create a new customized `initramfs` file with the following information:
   ```text
   $ coreos-installer pxe customize \
       --pre-install mount-iscsi.sh \
       --post-install unmount-iscsi.sh \
       --dest-device /dev/mapper/mpatha \
       --dest-ignition config.ign \
       --dest-karg-append rd.iscsi.firmware=1 \
       --dest-karg-append rd.multipath=default \
       -o custom.img rhcos-<version>-live-initramfs.x86_64.img
   ```

   where:

   <dl>
   <dt><code>mount-iscsi.sh</code></dt>
   <dd>Specifies the script that gets run before installation. It should contain the <code>iscsiadm</code> commands for mounting the iSCSI target.</dd>
   <dt><code>unmount-iscsi.sh</code></dt>
   <dd>Specifies the script that gets run after installation. It should contain the command <code>iscsiadm --mode node --logout=all</code>.</dd>
   <dt><code>/dev/mapper/mpatha</code></dt>
   <dd>Specifies the path to the device. If you are using multipath, the multipath device, <code>/dev/mapper/mpatha</code>, If there are multiple multipath devices connected, or to be explicit, you can use the World Wide Name (WWN) symlink available in <code>/dev/disk/by-path</code>.</dd>
   <dt><code>config.ign</code></dt>
   <dd>Specifies the Ignition configuration for the destination system.</dd>
   <dt><code>rd.iscsi.firmware=1</code></dt>
   <dd>Specifies the iSCSI parameter is read from the BIOS firmware.</dd>
   <dt><code>rd.multipath=default</code></dt>
   <dd>Specifies if you want to enable multipathing. Optional parameter.</dd>
   </dl>

   For more information about see the `dracut.cmdline` manual page.

### Networking and bonding options for ISO installations {#installation-user-infra-machines-routing-bonding_installing-restricted-networks-bare-metal}

You can configure advanced options so that you can modify the Red Hat Enterprise Linux CoreOS (RHCOS) manual installation process. The subsequent sections show examples of networking options for an ISO installation.

If you install RHCOS from an ISO image, you can add kernel arguments manually when you boot the image to configure networking for a node. If no networking arguments are specified, DHCP is activated in the initramfs when RHCOS detects that networking is required to fetch the Ignition config file.

:::warning

When adding networking arguments manually, you must also add the `rd.neednet=1` kernel argument to bring the network up in the initramfs.

:::

The following information provides examples for configuring networking and bonding on your RHCOS nodes for ISO installations. The examples describe how to use the `ip=`, `nameserver=`, and `bond=` kernel arguments.

:::note

Ordering is important when adding the kernel arguments: `ip=`, `nameserver=`, and then `bond=`.

:::

The networking options are passed to the `dracut` tool during system boot. For more information about the networking options supported by `dracut`, see `dracut.cmdline` manual page.

**Additional resources**

- [`dracut.cmdline` manual page](https://www.man7.org/linux/man-pages/man7/dracut.cmdline.7.html)

#### Configuring DHCP or static IP addresses {#configuring-dhcp-or-static-ip-addresses_installing-restricted-networks-bare-metal}

You can configure an IP address by using either DHCP or an individual static IP address. If you set a static IP, you must then identify the DNS server IP address on each node.

The configuration examples in the procedure, update the IP addresses for the following components:

- The node’s IP address to `10.10.10.2`
- The gateway address to `10.10.10.254`
- The netmask to `255.255.255.0`
- The hostname to `core0.example.com`
- The DNS server address to `4.4.4.41`
- The auto-configuration value to `none`. No auto-configuration is required when IP networking is configured statically.

**Procedure**

1. Enter a command like the following command to configure a static IP address:
   ```terminal
   ip=10.10.10.2::10.10.10.254:255.255.255.0:core0.example.com:enp1s0:none
   nameserver=4.4.4.41
   ```
2. Enter a command like the following command to configure a DHCP IP address:
   ```terminal
   ip=enp1s0:dhcp
   ```

   :::note

   When you use DHCP to configure IP addressing for the RHCOS machines, the machines also obtain the DNS server information through DHCP. For DHCP-based deployments, you can define the DNS server address that is used by the RHCOS nodes through your DHCP server configuration.

   :::
3. If two or more network interfaces and only one interface exists, disable DHCP on a single interface. In the example, the `enp1s0` interface has a static networking configuration and DHCP is disabled for `enp2s0`, which is not used:
   ```terminal
   ip=10.10.10.2::10.10.10.254:255.255.255.0:core0.example.com:enp1s0:none
   ip=::::core0.example.com:enp2s0:none
   ```
4. If you need to combine DHCP and static IP configurations on systems with multiple network interfaces, run the following example command:
   ```terminal
   ip=enp1s0:dhcp
   ip=10.10.10.2::10.10.10.254:255.255.255.0:core0.example.com:enp2s0:none
   ```

#### Configuring an IP address without a static hostname {#configuring-ip-address-without-static-hostname_installing-restricted-networks-bare-metal}

You can configure an IP address without assigning a static hostname. If a static hostname is not set by the user, the static hostname gets picked up and automatically set by a reverse DNS lookup.

The configuration examples in the procedure, update the IP addresses for the following components:

- The node’s IP address to `10.10.10.2`
- The gateway address to `10.10.10.254`
- The netmask to `255.255.255.0`
- The DNS server address to `4.4.4.41`
- The auto-configuration value to `none`. No auto-configuration is required when IP networking is configured statically.

**Procedure**

- To configure an IP address without a static hostname, enter a command like the following command:
  ```terminal
  ip=10.10.10.2::10.10.10.254:255.255.255.0::enp1s0:none
  nameserver=4.4.4.41
  ```

#### Specifying multiple network interfaces and DNS servers {#specifying-multiple-network-interfaces_installing-restricted-networks-bare-metal}

You can specify multiple network interfaces by setting multiple `ip=` entries. You can provide multiple DNS servers by adding a `nameserver=` entry for each server,

**Procedure**

- To specify multiple network interfaces for your interfaces, you can enter a command like the following command:
  ```terminal
  ip=10.10.10.2::10.10.10.254:255.255.255.0:core0.example.com:enp1s0:none
  ip=10.10.10.3::10.10.10.254:255.255.255.0:core0.example.com:enp2s0:none
  ```
- To provide multiple DNS servers by adding a `nameserver=` entry for each server, enter a command like the following command:
  ```terminal
  nameserver=1.1.1.1
  nameserver=8.8.8.8
  ```

#### Configuring default gateway and route {#configuring-default-gateway-route_installing-restricted-networks-bare-metal}

As an optional task, you can configure routes to additional networks by setting an `rd.route=` value.

:::note

When you configure one or multiple networks, one default gateway is required. If the additional network gateway is different from the primary network gateway, the default gateway must be the primary network gateway.

:::

**Procedure**

- To configure the default gateway, enter the following command:
  ```terminal
  ip=::10.10.10.254::::
  ```
- To configure the route for an additional network, enter the following command:
  ```terminal
  rd.route=20.20.20.0/24:20.20.20.254:enp2s0
  ```

#### Configuring VLANs on individual interfaces {#configuring-vlans-individual-interfaces_installing-restricted-networks-bare-metal}

As an optional task, you can configure VLANs on individual interfaces by using the `vlan=` parameter.

**Procedure**

- To configure a VLAN on a network interface and use a static IP address, run the following command:
  ```terminal
  ip=10.10.10.2::10.10.10.254:255.255.255.0:core0.example.com:enp2s0.100:none
  vlan=enp2s0.100:enp2s0
  ```
- To configure a VLAN on a network interface and to use DHCP, run the following command:
  ```terminal
  ip=enp2s0.100:dhcp
  vlan=enp2s0.100:enp2s0
  ```

#### Bonding multiple network interfaces to a single interface {#bonding-multiple-network-interfaces-to-single-interface_installing-restricted-networks-bare-metal}

As an optional task, you can bond multiple network interfaces to a single interface by using the `bond=` option.

The following example demonstrates editing the `/etc/config/network` file and specifying the following syntax for bonding multiple network interfaces to a single interface:

```terminal
bond=<name>[:<network_interfaces>][:<options>]
```

- `<name>`: Specifies the bonding device name, for example `bond0`.
- `<network_interfaces>`: Specifies a comma-separated list of physical (ethernet) interfaces, such as `em1,em2`.
- ` <options>: Specifies a comma-separated list of bonding options. Enter the  `modinfo bonding` command to see available options.

When you create a bonded interface using the `bond=` command, you must specify how the IP address is assigned and other information for the bonded interface.

**Procedure**

- To configure the bonded interface to use DHCP, edit the `/etc/config/network` file by setting the IP address for the bond to `dhcp`. For example:
  ```terminal
  ip=bond0:dhcp
  ```
- To configure the bonded interface to use a static IP address, edit the `/etc/config/network` file entering the specific IP address you want and related information. For example:
  ```terminal
  ip=10.10.10.2::10.10.10.254:255.255.255.0:core0.example.com:bond0:none
  ```

#### Bonding multiple SR-IOV network interfaces to a dual port NIC interface {#bonding-multiple-sriov-network-interfaces-to-dual-port_installing-restricted-networks-bare-metal}

You can bond multiple SR-IOV network interfaces to a dual port NIC interface by using the `bond=` option. Ensure you apply the procedure tasks to each node.

**Procedure**

1. Create the SR-IOV virtual functions (VFs) following the guidance in [Managing SR-IOV devices](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/configuring_and_managing_virtualization/managing-virtual-devices_configuring-and-managing-virtualization#managing-sr-iov-devices_managing-virtual-devices). Follow the procedure in the "Attaching SR-IOV networking devices to virtual machines" section.
2. Create the bond, attach the desired VFs to the bond and set the bond link state up following the guidance in [Configuring network bonding](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/configuring_and_managing_networking/configuring-network-bonding_configuring-and-managing-networking). Follow any of the described procedures to create the bond.
   The following examples illustrate the syntax you must use:

   - The syntax for configuring a bonded interface is `bond=<name>[:<network_interfaces>][:options]`.
     `<name>` is the bonding device name (`bond0`), `<network_interfaces>` represents the virtual functions (VFs) by their known name in the kernel and shown in the output of the `ip link` command(`eno1f0`, `eno2f0`), and *options* is a comma-separated list of bonding options. Enter `modinfo bonding` to see available options.
   - When you create a bonded interface using `bond=`, you must specify how the IP address is assigned and other information for the bonded interface.
     - To configure the bonded interface to use DHCP, set the bond’s IP address to `dhcp`. For example:
       ```terminal
       bond=bond0:eno1f0,eno2f0:mode=active-backup
       ip=bond0:dhcp::AA:BB:CC:DD:EE:FF
       ip=eno1f0:none::AA:BB:CC:DD:EE:FF
       ip=eno2f0:none::AA:BB:CC:DD:EE:FF
       ```
     - To configure the bonded interface to use a static IP address, enter the specific IP address you want and related information. For example:
       ```terminal
       bond=bond0:eno1f0,eno2f0:mode=active-backup
       ip=10.10.10.2::10.10.10.254:255.255.255.0:core0.example.com:bond0:none
       ```
3. Optional: You can use network teaming as an alternative to bonding by using the `team=` parameter.
   - The syntax for configuring a team interface is: `team=name[:network_interfaces]`
     *name* is the team device name (`team0`) and *network_interfaces* represents a comma-separated list of physical (ethernet) interfaces (`em1, em2`).

     :::note

     Teaming is planned to be deprecated when RHCOS switches to an upcoming version of RHEL. For more information, see this [Red Hat Knowledgebase Article](https://access.redhat.com/solutions/6509691).

     :::

     Use the following example to configure a network team:

     ```terminal
     team=team0:em1,em2
     ip=team0:dhcp
     ```

#### `coreos-installer` and boot options for ISO and PXE installations {#installation-user-infra-machines-coreos-installer-options_installing-restricted-networks-bare-metal}

You can install RHCOS by running `coreos-installer install <options> <device>` at the command prompt, after booting into the RHCOS live environment from an ISO image.

The following table shows the subcommands, options, and arguments you can pass to the `coreos-installer` command.

`coreos-installer`** subcommands, command-line options, and arguments**

<table>
<thead>
<tr>
  <th colspan="2"><strong>coreos-installer install subcommand</strong></th>
</tr>
</thead>
<tbody>
<tr>
  <td><strong><em>Subcommand</em></strong></td>
  <td><strong><em>Description</em></strong></td>
</tr>
<tr>
  <td><code>$ coreos-installer install &lt;options&gt; &lt;device&gt;</code></td>
  <td>Embed an Ignition config in an ISO image.</td>
</tr>
<tr>
  <td colspan="2"><strong>coreos-installer install subcommand options</strong></td>
</tr>
<tr>
  <td><strong><em>Option</em></strong></td>
  <td><strong><em>Description</em></strong></td>
</tr>
<tr>
  <td><code>-u</code>, <code>--image-url &lt;url&gt;</code></td>
  <td>Specify the image URL manually.</td>
</tr>
<tr>
  <td><code>-f</code>, <code>--image-file &lt;path&gt;</code></td>
  <td>Specify a local image file manually. Used for debugging.</td>
</tr>
<tr>
  <td><code>-i,</code> <code>--ignition-file &lt;path&gt;</code></td>
  <td>Embed an Ignition config from a file.</td>
</tr>
<tr>
  <td><code>-I</code>, <code>--ignition-url &lt;URL&gt;</code></td>
  <td>Embed an Ignition config from a URL.</td>
</tr>
<tr>
  <td><code>--ignition-hash &lt;digest&gt;</code></td>
  <td>Digest <code>type-value</code> of the Ignition config.</td>
</tr>
<tr>
  <td><code>-p</code>, <code>--platform &lt;name&gt;</code></td>
  <td>Override the Ignition platform ID for the installed system.</td>
</tr>
<tr>
  <td><code>--console &lt;spec&gt;</code></td>
  <td>Set the kernel and boot loader console for the installed system. For more information about the format of <code>&lt;spec&gt;</code>, see the <a href="https://www.kernel.org/doc/html/latest/admin-guide/serial-console.html">Linux kernel serial console</a> documentation.</td>
</tr>
<tr>
  <td><code>--append-karg &lt;arg&gt;...</code></td>
  <td>Append a default kernel argument to the installed system.</td>
</tr>
<tr>
  <td><code>--delete-karg &lt;arg&gt;...</code></td>
  <td>Delete a default kernel argument from the installed system.</td>
</tr>
<tr>
  <td><code>-n</code>, <code>--copy-network</code></td>
  <td>Copy the network configuration from the install environment.<br><br><div class="alert alert--warning"><strong>Important</strong><p>The <code>--copy-network</code> option only copies networking configuration found under <code>/etc/NetworkManager/system-connections</code>. In particular, it does not copy the system hostname.</p></div></td>
</tr>
<tr>
  <td><code>--network-dir &lt;path&gt;</code></td>
  <td>For use with <code>-n</code>. Default is <code>/etc/NetworkManager/system-connections/</code>.</td>
</tr>
<tr>
  <td><code>--save-partlabel &lt;lx&gt;..</code></td>
  <td>Save partitions with this label glob.</td>
</tr>
<tr>
  <td><code>--save-partindex &lt;id&gt;...</code></td>
  <td>Save partitions with this number or range.</td>
</tr>
<tr>
  <td><code>--insecure</code></td>
  <td>Skip RHCOS image signature verification.</td>
</tr>
<tr>
  <td><code>--insecure-ignition</code></td>
  <td>Allow Ignition URL without HTTPS or hash.</td>
</tr>
<tr>
  <td><code>--architecture &lt;name&gt;</code></td>
  <td>Target CPU architecture. Valid values are <code>x86_64</code> and <code>aarch64</code>.</td>
</tr>
<tr>
  <td><code>--preserve-on-error</code></td>
  <td>Do not clear partition table on error.</td>
</tr>
<tr>
  <td><code>-h</code>, <code>--help</code></td>
  <td>Print help information.</td>
</tr>
<tr>
  <td colspan="2"><strong>coreos-installer install subcommand argument</strong></td>
</tr>
<tr>
  <td><strong><em>Argument</em></strong></td>
  <td><strong><em>Description</em></strong></td>
</tr>
<tr>
  <td><code>&lt;device&gt;</code></td>
  <td>The destination device.</td>
</tr>
<tr>
  <td colspan="2"><strong>coreos-installer ISO subcommands</strong></td>
</tr>
<tr>
  <td><strong><em>Subcommand</em></strong></td>
  <td><strong><em>Description</em></strong></td>
</tr>
<tr>
  <td><code>$ coreos-installer iso customize &lt;options&gt; &lt;ISO_image&gt;</code></td>
  <td>Customize a RHCOS live ISO image.</td>
</tr>
<tr>
  <td><code>coreos-installer iso reset &lt;options&gt; &lt;ISO_image&gt;</code></td>
  <td>Restore a RHCOS live ISO image to default settings.</td>
</tr>
<tr>
  <td><code>coreos-installer iso ignition remove &lt;options&gt; &lt;ISO_image&gt;</code></td>
  <td>Remove the embedded Ignition config from an ISO image.</td>
</tr>
<tr>
  <td colspan="2"><strong>coreos-installer ISO customize subcommand options</strong></td>
</tr>
<tr>
  <td><strong><em>Option</em></strong></td>
  <td><strong><em>Description</em></strong></td>
</tr>
<tr>
  <td><code>--dest-ignition &lt;path&gt;</code></td>
  <td>Merge the specified Ignition config file into a new configuration fragment for the destination system.</td>
</tr>
<tr>
  <td><code>--dest-console &lt;spec&gt;</code></td>
  <td>Specify the kernel and boot loader console for the destination system.</td>
</tr>
<tr>
  <td><code>--dest-device &lt;path&gt;</code></td>
  <td>Install and overwrite the specified destination device.</td>
</tr>
<tr>
  <td><code>--dest-karg-append &lt;arg&gt;</code></td>
  <td>Add a kernel argument to each boot of the destination system.</td>
</tr>
<tr>
  <td><code>--dest-karg-delete &lt;arg&gt;</code></td>
  <td>Delete a kernel argument from each boot of the destination system.</td>
</tr>
<tr>
  <td><code>--network-keyfile &lt;path&gt;</code></td>
  <td>Configure networking by using the specified NetworkManager keyfile for live and destination systems.</td>
</tr>
<tr>
  <td><code>--ignition-ca &lt;path&gt;</code></td>
  <td>Specify an additional TLS certificate authority to be trusted by Ignition.</td>
</tr>
<tr>
  <td><code>--pre-install &lt;path&gt;</code></td>
  <td>Run the specified script before installation.</td>
</tr>
<tr>
  <td><code>--post-install &lt;path&gt;</code></td>
  <td>Run the specified script after installation.</td>
</tr>
<tr>
  <td><code>--installer-config &lt;path&gt;</code></td>
  <td>Apply the specified installer configuration file.</td>
</tr>
<tr>
  <td><code>--live-ignition &lt;path&gt;</code></td>
  <td>Merge the specified Ignition config file into a new configuration fragment for the live environment.</td>
</tr>
<tr>
  <td><code>--live-karg-append &lt;arg&gt;</code></td>
  <td>Add a kernel argument to each boot of the live environment.</td>
</tr>
<tr>
  <td><code>--live-karg-delete &lt;arg&gt;</code></td>
  <td>Delete a kernel argument from each boot of the live environment.</td>
</tr>
<tr>
  <td><code>--live-karg-replace &lt;k=o=n&gt;</code></td>
  <td>Replace a kernel argument in each boot of the live environment, in the form <code>key=old=new</code>.</td>
</tr>
<tr>
  <td><code>-f</code>, <code>--force</code></td>
  <td>Overwrite an existing Ignition config.</td>
</tr>
<tr>
  <td><code>-o</code>, <code>--output &lt;path&gt;</code></td>
  <td>Write the ISO to a new output file.</td>
</tr>
<tr>
  <td><code>-h</code>, <code>--help</code></td>
  <td>Print help information.</td>
</tr>
<tr>
  <td colspan="2"><strong>coreos-installer PXE subcommands</strong></td>
</tr>
<tr>
  <td><strong><em>Subcommand</em></strong></td>
  <td><strong><em>Description</em></strong></td>
</tr>
<tr>
  <td colspan="2">Note that not all of these options are accepted by all subcommands.</td>
</tr>
<tr>
  <td><code>coreos-installer pxe customize &lt;options&gt; &lt;path&gt;</code></td>
  <td>Customize a RHCOS live PXE boot config.</td>
</tr>
<tr>
  <td><code>coreos-installer pxe ignition wrap &lt;options&gt;</code></td>
  <td>Wrap an Ignition config in an image.</td>
</tr>
<tr>
  <td><code>coreos-installer pxe ignition unwrap &lt;options&gt; &lt;image_name&gt;</code></td>
  <td>Show the wrapped Ignition config in an image.</td>
</tr>
<tr>
  <td colspan="2"><strong>coreos-installer PXE customize subcommand options</strong></td>
</tr>
<tr>
  <td><strong><em>Option</em></strong></td>
  <td><strong><em>Description</em></strong></td>
</tr>
<tr>
  <td colspan="2">Note that not all of these options are accepted by all subcommands.</td>
</tr>
<tr>
  <td><code>--dest-ignition &lt;path&gt;</code></td>
  <td>Merge the specified Ignition config file into a new configuration fragment for the destination system.</td>
</tr>
<tr>
  <td><code>--dest-console &lt;spec&gt;</code></td>
  <td>Specify the kernel and boot loader console for the destination system.</td>
</tr>
<tr>
  <td><code>--dest-device &lt;path&gt;</code></td>
  <td>Install and overwrite the specified destination device.</td>
</tr>
<tr>
  <td><code>--network-keyfile &lt;path&gt;</code></td>
  <td>Configure networking by using the specified NetworkManager keyfile for live and destination systems.</td>
</tr>
<tr>
  <td><code>--ignition-ca &lt;path&gt;</code></td>
  <td>Specify an additional TLS certificate authority to be trusted by Ignition.</td>
</tr>
<tr>
  <td><code>--pre-install &lt;path&gt;</code></td>
  <td>Run the specified script before installation.</td>
</tr>
<tr>
  <td><code>post-install &lt;path&gt;</code></td>
  <td>Run the specified script after installation.</td>
</tr>
<tr>
  <td><code>--installer-config &lt;path&gt;</code></td>
  <td>Apply the specified installer configuration file.</td>
</tr>
<tr>
  <td><code>--live-ignition &lt;path&gt;</code></td>
  <td>Merge the specified Ignition config file into a new configuration fragment for the live environment.</td>
</tr>
<tr>
  <td><code>-o,</code> <code>--output &lt;path&gt;</code></td>
  <td>Write the initramfs to a new output file.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>This option is required for PXE environments.</p></div></td>
</tr>
<tr>
  <td><code>-h</code>, <code>--help</code></td>
  <td>Print help information.</td>
</tr>
</tbody>
</table>

You can automatically start `coreos-installer` options at boot time by passing `coreos.inst` boot arguments to the RHCOS live installer. These are provided in addition to the standard boot arguments.

- For ISO installations, the `coreos.inst` options can be added by interrupting the automatic boot at the boot loader menu. You can interrupt the automatic boot by pressing `TAB` while the **RHEL CoreOS (Live)** menu option is highlighted.
- For PXE or iPXE installations, the `coreos.inst` options must be added to the `APPEND` line before the RHCOS live installer is booted.

The following table shows the RHCOS live installer `coreos.inst` boot options for ISO and PXE installations.

`coreos.inst`** boot options**

<table>
<thead>
<tr>
  <th>Argument</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>coreos.inst.install_dev</code></td>
  <td>Required. The block device on the system to install to.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>It is recommended to use the full path, such as <code>/dev/sda</code>, although <code>sda</code> is allowed.</p></div></td>
</tr>
<tr>
  <td><code>coreos.inst.ignition_url</code></td>
  <td>Optional: The URL of the Ignition config to embed into the installed system. If no URL is specified, no Ignition config is embedded. Only HTTP and HTTPS protocols are supported.</td>
</tr>
<tr>
  <td><code>coreos.inst.save_partlabel</code></td>
  <td>Optional: Comma-separated labels of partitions to preserve during the install. Glob-style wildcards are permitted. The specified partitions do not need to exist.</td>
</tr>
<tr>
  <td><code>coreos.inst.save_partindex</code></td>
  <td>Optional: Comma-separated indexes of partitions to preserve during the install. Ranges <code>m-n</code> are permitted, and either <code>m</code> or <code>n</code> can be omitted. The specified partitions do not need to exist.</td>
</tr>
<tr>
  <td><code>coreos.inst.insecure</code></td>
  <td>Optional: Permits the OS image that is specified by <code>coreos.inst.image_url</code> to be unsigned.</td>
</tr>
<tr>
  <td><code>coreos.inst.image_url</code></td>
  <td>Optional: Download and install the specified RHCOS image.<br><br><ul><li>This argument should not be used in production environments and is intended for debugging purposes only.</li><li>While this argument can be used to install a version of RHCOS that does not match the live media, it is recommended that you instead use the media that matches the version you want to install.</li><li>If you are using <code>coreos.inst.image_url</code>, you must also use <code>coreos.inst.insecure</code>. This is because the bare-metal media are not GPG-signed for OpenShift Container Platform.</li><li>Only HTTP and HTTPS protocols are supported.</li></ul></td>
</tr>
<tr>
  <td><code>coreos.inst.skip_reboot</code></td>
  <td>Optional: The system will not reboot after installing. After the install finishes, you will receive a prompt that allows you to inspect what is happening during installation. This argument should not be used in production environments and is intended for debugging purposes only.</td>
</tr>
<tr>
  <td><code>coreos.inst.platform_id</code></td>
  <td>Optional: The Ignition platform ID of the platform the RHCOS image is being installed on. Default is <code>metal</code>. This option determines whether or not to request an Ignition config from the cloud provider, such as VMware. For example: <code>coreos.inst.platform_id=vmware</code>.</td>
</tr>
<tr>
  <td><code>ignition.config.url</code></td>
  <td>Optional: The URL of the Ignition config for the live boot. For example, this can be used to customize how <code>coreos-installer</code> is invoked, or to run code before or after the installation. This is different from <code>coreos.inst.ignition_url</code>, which is the Ignition config for the installed system.</td>
</tr>
</tbody>
</table>

### Enabling multipathing with kernel arguments on RHCOS {#rhcos-enabling-multipath_installing-restricted-networks-bare-metal}

RHCOS supports multipathing on the primary disk, allowing stronger resilience to hardware failure to achieve higher host availability.

You can enable multipathing at installation time for nodes that were provisioned in OpenShift Container Platform 4.8 or later. While postinstallation support is available by activating multipathing through the machine config, Red Hat recommends enabling multipathing during installation.

In setups where any I/O to non-optimized paths results in I/O system errors, you must enable multipathing at installation time.

:::warning

On IBM Z(R) and IBM(R) LinuxONE, you can enable multipathing only if you configured your cluster for it during installation. For more information, see "Installing RHCOS and starting the OpenShift Container Platform bootstrap process" in *Installing a cluster with z/VM on IBM Z(R) and IBM(R) LinuxONE*.

:::

The following procedure enables multipath at installation time and appends kernel arguments to the `coreos-installer install` command so that the installed system itself will use multipath beginning from the first boot.

:::note

OpenShift Container Platform does not support enabling multipathing as a day-2 activity on nodes that have been upgraded from 4.6 or earlier.

:::

**Prerequisites**

- You have created the Ignition config files for your cluster.
- You have reviewed *Installing RHCOS and starting the OpenShift Container Platform bootstrap process*.

**Procedure**

1. To enable multipath and start the `multipathd` daemon, run the following command on the installation host:
   ```terminal
   $ mpathconf --enable && systemctl start multipathd.service
   ```

   1. Optional: If booting the PXE or ISO, you can instead enable multipath by adding `rd.multipath=default` from the kernel command line.
2. Append the kernel arguments by invoking the `coreos-installer` program:
   - If there is only one multipath device connected to the machine, the device should be available at path `/dev/mapper/mpatha`. For example:
     ```terminal
     $ coreos-installer install /dev/mapper/mpatha \
     --ignition-url=http://host/worker.ign \
     --append-karg rd.multipath=default \
     --append-karg root=/dev/disk/by-label/dm-mpath-root \
     --append-karg rw \
     --offline
     ```

     - `/dev/mapper/mpatha`: Indicates the path of the single multipathed device.
   - If there are multiple multipath devices connected to the machine, instead of using `/dev/mapper/mpatha`, Red Hat recommends using the World Wide Name (WWN) symlink. The symlink is available in `/dev/disk/by-id`. For example:
     ```terminal
     $ coreos-installer install /dev/disk/by-id/wwn-<wwn_ID> \
     --ignition-url=http://host/worker.ign \
     --append-karg rd.multipath=default \
     --append-karg root=/dev/disk/by-label/dm-mpath-root \
     --append-karg rw \
     --offline
     ```

     where:

     <dl>
     <dt><code>&lt;wwn_ID&gt;</code></dt>
     <dd>Indicates the WWN ID of the target multipathed device. For example, <code>0xx194e957fcedb4841</code>. This symlink can also be used as the <code>coreos.inst.install_dev</code> kernel argument when using special <code>coreos.inst.*</code> arguments to direct the live installer. For more information, see "Installing RHCOS and starting the OpenShift Container Platform bootstrap process".</dd>
     </dl>
3. Reboot into the installed system.
4. Check that the kernel arguments worked by going to one of the worker nodes and listing the kernel command-line arguments (in `/proc/cmdline` on the host):
   ```terminal
   $ oc debug node/ip-10-0-141-105.ec2.internal
   ```

   ```terminal title="Example output"
   Starting pod/ip-10-0-141-105ec2internal-debug ...
   To use host binaries, run `chroot /host`

   sh-4.2# cat /host/proc/cmdline
   ...
   rd.multipath=default root=/dev/disk/by-label/dm-mpath-root
   ...

   sh-4.2# exit
   ```

   You should see the added kernel arguments.

#### Enabling multipathing on secondary disks {#rhcos-multipath-secondary-disk_installing-restricted-networks-bare-metal}

RHCOS also supports multipathing on a secondary disk. Instead of kernel arguments, you use Ignition to enable multipathing for the secondary disk at installation time.

**Prerequisites**

- You have read the section *Disk partitioning*.
- You have read *Enabling multipathing with kernel arguments on RHCOS*.
- You have installed the Butane utility.

**Procedure**

1. Create a Butane config with information similar to the following:
   ```yaml title="Example multipath-config.bu"
   variant: openshift
   version: 4.22.0
   systemd:
     units:
       - name: mpath-configure.service
         enabled: true
         contents: |
           [Unit]
           Description=Configure Multipath on Secondary Disk
           ConditionFirstBoot=true
           ConditionPathExists=!/etc/multipath.conf
           Before=multipathd.service
           DefaultDependencies=no

           [Service]
           Type=oneshot
           ExecStart=/usr/sbin/mpathconf --enable

           [Install]
           WantedBy=multi-user.target
       - name: mpath-var-lib-container.service
         enabled: true
         contents: |
           [Unit]
           Description=Set Up Multipath On /var/lib/containers
           ConditionFirstBoot=true
           Requires=dev-mapper-mpatha.device
           After=dev-mapper-mpatha.device
           After=ostree-remount.service
           Before=kubelet.service
           DefaultDependencies=no

           [Service]
           Type=oneshot
           ExecStart=/usr/sbin/mkfs.xfs -L containers -m reflink=1 /dev/mapper/mpatha
           ExecStart=/usr/bin/mkdir -p /var/lib/containers

           [Install]
           WantedBy=multi-user.target
       - name: var-lib-containers.mount
         enabled: true
         contents: |
           [Unit]
           Description=Mount /var/lib/containers
           After=mpath-var-lib-containers.service
           Before=kubelet.service

           [Mount]
           What=/dev/disk/by-label/dm-mpath-containers
           Where=/var/lib/containers
           Type=xfs

           [Install]
           WantedBy=multi-user.target
   ```

   where:

   <dl>
   <dt><code>Before=multipathd.service</code></dt>
   <dd>Specifies that the configuration must be set before launching the multipath daemon.</dd>
   <dt><code>ExecStart=/usr/sbin/mpathconf</code></dt>
   <dd>Specifies starting the <code>mpathconf</code> utility.</dd>
   <dt><code>ConditionFirstBoot=true</code></dt>
   <dd>Set to the value <code>true</code>.</dd>
   <dt><code>[Service]</code></dt>
   <dd>Specifies the creation of the filesystem and directory <code>/var/lib/containers</code>.</dd>
   <dt><code>Before=kubelet.service</code></dt>
   <dd>Specifies that the device must be mounted before starting any nodes.</dd>
   <dt><code>[Mount]</code></dt>
   <dd>Specifies to mount the device to the <code>/var/lib/containers</code> mount point. This location cannot be a symlink.</dd>
   </dl>
2. Create the Ignition configuration by running the following command:
   ```terminal
   $ butane --pretty --strict multipath-config.bu > multipath-config.ign
   ```
3. Continue with the rest of the first boot RHCOS installation process.
   :::warning

   Do not add the `rd.multipath` or `root` kernel arguments on the CLI during installation unless the primary disk is also multipathed.

   :::

### Installing RHCOS manually on an iSCSI boot device {#rhcos-install-iscsi-manual_installing-restricted-networks-bare-metal}

You can manually install RHCOS on an iSCSI target.

**Prerequisites**

1. You are in the RHCOS live environment.
2. You have an iSCSI target that you want to install RHCOS on.

**Procedure**

1. Mount the iSCSI target from the live environment by running the following command:
   ```text
   $ iscsiadm \
       --mode discovery \
       --type sendtargets
       --portal <IP_address> \
       --login
   ```

   where:

   <dl>
   <dt><code>&lt;IP_address&gt;</code></dt>
   <dd>Specifies the IP address of the target portal.</dd>
   </dl>
2. Install RHCOS onto the iSCSI target by running the following command and using the necessary kernel arguments, for example:
   ```text

   $ coreos-installer install \
   /dev/disk/by-path/ip-<IP_address>:<port>-iscsi-<target_iqn>-lun-<lun> \
   --append-karg rd.iscsi.initiator=<initiator_iqn> \
   --append.karg netroot=<target_iqn> \
   --console ttyS0,115200n8 \
   --ignition-file <path_to_file> \
   --offline
   ```

   where:

   <dl>
   <dt><code>/dev/disk/by-path/ip</code></dt>
   <dd>Specifies the installation location. You must provide the IP address of the target portal, the associated port number, the target iSCSI node in IQN format, and the iSCSI logical unit number (LUN).</dd>
   <dt><code>&lt;initiator_iqn&gt;</code></dt>
   <dd>Specifies the iSCSI initiator, or client, name in IQN format. The initiator forms a session to connect to the iSCSI target.</dd>
   <dt><code>&lt;target_iqn&gt;</code></dt>
   <dd>Specifies the iSCSI target, or server, name in IQN format. For more information about the iSCSI options supported by <code>dracut</code>, see the <a href="https://www.man7.org/linux/man-pages/man7/dracut.cmdline.7.html"><code>dracut.cmdline</code> manual page</a>.</dd>
   </dl>
3. Unmount the iSCSI disk with the following command:
   ```text
   $ iscsiadm --mode node --logoutall=all
   ```

   This procedure can also be performed using the `coreos-installer iso customize` or `coreos-installer pxe customize` subcommands.

### Installing RHCOS on an iSCSI boot device using iBFT {#rhcos-install-iscsi-ibft_installing-restricted-networks-bare-metal}

On a completely diskless machine, the iSCSI target and initiator values can be passed through iBFT. iSCSI multipathing is also supported.

**Prerequisites**

1. You are in the RHCOS live environment.
2. You have an iSCSI target you want to install RHCOS on.
3. Optional: You have configured multipathing for your iSCSI target.

**Procedure**

1. Mount the iSCSI target from the live environment by running the following command:
   ```text
   $ iscsiadm \
       --mode discovery \
       --type sendtargets
       --portal <IP_address> \
       --login
   ```

   where:

   <dl>
   <dt><code>&lt;IP_address&gt;</code></dt>
   <dd>Specifies the IP address of the target portal.</dd>
   </dl>
2. Optional: enable multipathing and start the daemon with the following command:
   ```text
   $ mpathconf --enable && systemctl start multipathd.service
   ```
3. Install RHCOS onto the iSCSI target by running the following command and using the necessary kernel arguments, for example:
   ```text

   $ coreos-installer install \
       /dev/mapper/mpatha \
       --append-karg rd.iscsi.firmware=1 \
       --append-karg rd.multipath=default \
       --console ttyS0 \
       --ignition-file <path_to_file> \
       --offline
   ```

   where:

   <dl>
   <dt><code>/dev/mapper/mpatha</code></dt>
   <dd>Specifies the path of a single multipathed device. If there are multiple multipath devices connected, or to be explicit, you can use the World Wide Name (WWN) symlink available in <code>/dev/disk/by-path</code>.</dd>
   <dt><code>rd.iscsi.firmware=1</code></dt>
   <dd>Specifies that the iSCSI parameter is read from the BIOS firmware.</dd>
   <dt><code>rd.multipath=default</code></dt>
   <dd>Specifies to enable multipathing. Optional parameter. For more information about the iSCSI options supported by <code>dracut</code>, see the <code>dracut.cmdline</code> manual page.</dd>
   </dl>
4. Unmount the iSCSI disk:
   ```text
   $ iscsiadm --mode node --logout=all
   ```

   You can also perform this procedure by using the `coreos-installer iso customize` or `coreos-installer pxe customize` subcommands.

## Waiting for the bootstrap process to complete {#installation-installing-bare-metal_installing-restricted-networks-bare-metal}

After your cluster nodes first boot into the persistent RHCOS environment installed to disk, wait for the OpenShift Container Platform bootstrap process to complete.

The configuration information provided through the Ignition config files is used to initialize the bootstrap process and install OpenShift Container Platform on the machines.

**Prerequisites**

- You have created the Ignition config files for your cluster.
- You have configured suitable network, DNS, and load balancing infrastructure.
- You have obtained the installation program and generated the Ignition config files for your cluster.
- You installed RHCOS on your cluster machines and provided the Ignition config files that the OpenShift Container Platform installation program generated.

**Procedure**

1. Monitor the bootstrap process:
   ```terminal
   $ ./openshift-install --dir <installation_directory> wait-for bootstrap-complete \
       --log-level=info
   ```

   where:

   <dl>
   <dt><code>&lt;installation_directory&gt;</code></dt>
   <dd>Specifies the path to the directory that stores the installation files.</dd>
   <dt><code>--log-level=info</code></dt>
   <dd>Specifies <code>warn</code>, <code>debug</code>, or <code>error</code> instead of <code>info</code> to view different installation details.</dd>
   </dl>

   ```terminal title="Example output"
   INFO Waiting up to 20m0s for the Kubernetes API at https://api.test.example.com:6443...
   INFO API v1.35.4 up
   INFO Waiting up to 1h0m0s for bootstrapping to complete...
   INFO It is now safe to remove the bootstrap resources
   ```

   The bootstrapping completion wait time varies per platform. The command succeeds when the Kubernetes API server signals that it has been bootstrapped on the control plane machines.
2. After the bootstrap process is complete, remove the bootstrap machine from the load balancer.
   :::warning

   You must remove the bootstrap machine from the load balancer at this point. You can also remove or reformat the bootstrap machine itself.

   :::

**Additional resources**

- See [Monitoring installation progress](/docs/support/troubleshooting/troubleshooting-installations#monitoring-installation-progress_troubleshooting-installations) for more information about monitoring the installation logs and retrieving diagnostic data if installation issues arise.

## Logging in to the cluster by using the CLI {#cli-logging-in-kubeadmin_installing-restricted-networks-bare-metal}

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

## Approve the certificate signing requests for your machines {#installation-approve-csrs_installing-restricted-networks-bare-metal}

To allow newly added machines to join your OpenShift Container Platform cluster, confirm that the cluster approves pending certificate signing requests (CSRs), or approve them yourself. Approve client requests first, then server requests.

**Prerequisites**

- You added machines to your cluster.

**Procedure**

1. Confirm that the cluster recognizes the machines:
   ```terminal
   $ oc get nodes
   ```

   ```terminal title="Example output"
   NAME      STATUS    ROLES   AGE  VERSION
   master-0  Ready     master  63m  v1.35.4
   master-1  Ready     master  63m  v1.35.4
   master-2  Ready     master  64m  v1.35.4
   ```

   The output lists all of the machines that you created.

   :::note

   The preceding output might not include the compute nodes until you approve some CSRs.

   :::
2. Review the pending CSRs and ensure that you see the client requests with the `Pending` or `Approved` status for each machine that you added to the cluster:
   ```terminal
   $ oc get csr
   ```

   ```terminal title="Example output"
   NAME        AGE     REQUESTOR                                                                   CONDITION
   csr-8b2br   15m     system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Pending
   csr-8vnps   15m     system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Pending
   ...
   ```

   In this example, two machines are joining the cluster. You might see more approved CSRs in the list.
3. If the CSRs were not approved, after all of the pending CSRs for the machines you added are in `Pending` status, approve the CSRs for your cluster machines:
   :::note

   You must approve your CSRs within an hour of adding the machines to the cluster. If you do not approve them within an hour, the certificates rotate, and more than two certificates are present for each node. You must approve all of these certificates. After you approve the client CSR, the kubelet creates a secondary CSR for the serving certificate, which requires manual approval. The `machine-approver` then automatically approves later serving certificate renewal requests if the kubelet requests a new certificate with the same parameters.

   :::

   :::note

   For clusters running on platforms that are not machine API enabled, such as bare metal and other user-provisioned infrastructure, you must implement a method of automatically approving the kubelet serving certificate requests (CSRs). If you do not approve a request, the `oc exec`, `oc rsh`, and `oc logs` commands cannot succeed, because the API server requires a serving certificate when it connects to the kubelet. Any operation that contacts the kubelet endpoint requires this certificate approval to be in place. The method must watch for new CSRs, confirm that the `node-bootstrapper` service account in the `system:node` or `system:admin` groups submitted the CSR, and confirm the identity of the node.

   :::

   - To approve them individually, run the following command for each valid CSR:
     ```terminal
     $ oc adm certificate approve <csr_name>
     ```

     where:

     <dl>
     <dt><code>&lt;csr_name&gt;</code></dt>
     <dd>Specifies the name of a CSR from the list of current CSRs.</dd>
     </dl>
   - To approve all pending CSRs, run the following command:
     ```terminal
     $ oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs --no-run-if-empty oc adm certificate approve
     ```

     :::note

     Some Operators might not become available until you approve some CSRs. Each node submits two CSRs, so you might need to run the command to approve CSRs many times.

     :::
4. After you approve your client requests, review the server requests for each machine that you added to the cluster:
   ```terminal
   $ oc get csr
   ```

   ```terminal title="Example output"
   NAME        AGE     REQUESTOR                                                                   CONDITION
   csr-bfd72   5m26s   system:node:ip-10-0-50-126.us-east-2.compute.internal                       Pending
   csr-c57lv   5m26s   system:node:ip-10-0-95-157.us-east-2.compute.internal                       Pending
   ...
   ```
5. If the remaining CSRs are not approved, and are in the `Pending` status, approve the CSRs for your cluster machines:
   - To approve them individually, run the following command for each valid CSR:
     ```terminal
     $ oc adm certificate approve <csr_name>
     ```

     where:

     <dl>
     <dt><code>&lt;csr_name&gt;</code></dt>
     <dd>Specifies the name of a CSR from the list of current CSRs.</dd>
     </dl>
   - To approve all pending CSRs, run the following command:
     ```terminal
     $ oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc adm certificate approve
     ```
6. After you approve all client and server CSRs, the machines have the `Ready` status. Verify this by running the following command:
   ```terminal
   $ oc get nodes
   ```

   ```terminal title="Example output"
   NAME      STATUS    ROLES   AGE  VERSION
   master-0  Ready     master  73m  v1.35.4
   master-1  Ready     master  73m  v1.35.4
   master-2  Ready     master  74m  v1.35.4
   worker-0  Ready     worker  11m  v1.35.4
   worker-1  Ready     worker  11m  v1.35.4
   ```

   :::note

   You might need to wait a few minutes after approval of the server CSRs for the machines to reach the `Ready` status.

   :::

## Initial Operator configuration {#installation-operators-config_installing-restricted-networks-bare-metal}

After the control plane initializes, you must immediately configure some Operators so that they all become available.

**Prerequisites**

- Your control plane has initialized.

**Procedure**

1. Watch the cluster components come online:
   ```terminal
   $ watch -n5 oc get clusteroperators
   ```

   ```terminal title="Example output"
   NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE
   authentication                             4.22.0    True        False         False      19m
   baremetal                                  4.22.0    True        False         False      37m
   cloud-credential                           4.22.0    True        False         False      40m
   cluster-autoscaler                         4.22.0    True        False         False      37m
   config-operator                            4.22.0    True        False         False      38m
   console                                    4.22.0    True        False         False      26m
   csi-snapshot-controller                    4.22.0    True        False         False      37m
   dns                                        4.22.0    True        False         False      37m
   etcd                                       4.22.0    True        False         False      36m
   image-registry                             4.22.0    True        False         False      31m
   ingress                                    4.22.0    True        False         False      30m
   insights                                   4.22.0    True        False         False      31m
   kube-apiserver                             4.22.0    True        False         False      26m
   kube-controller-manager                    4.22.0    True        False         False      36m
   kube-scheduler                             4.22.0    True        False         False      36m
   kube-storage-version-migrator              4.22.0    True        False         False      37m
   machine-api                                4.22.0    True        False         False      29m
   machine-approver                           4.22.0    True        False         False      37m
   machine-config                             4.22.0    True        False         False      36m
   marketplace                                4.22.0    True        False         False      37m
   monitoring                                 4.22.0    True        False         False      29m
   network                                    4.22.0    True        False         False      38m
   node-tuning                                4.22.0    True        False         False      37m
   openshift-apiserver                        4.22.0    True        False         False      32m
   openshift-controller-manager               4.22.0    True        False         False      30m
   openshift-samples                          4.22.0    True        False         False      32m
   operator-lifecycle-manager                 4.22.0    True        False         False      37m
   operator-lifecycle-manager-catalog         4.22.0    True        False         False      37m
   operator-lifecycle-manager-packageserver   4.22.0    True        False         False      32m
   service-ca                                 4.22.0    True        False         False      38m
   storage                                    4.22.0    True        False         False      37m
   ```
2. Configure the Operators that are not available.

**Additional resources**

- See [Gathering logs from a failed installation](/docs/support/troubleshooting/troubleshooting-installations#installation-bootstrap-gather_troubleshooting-installations) for details about gathering data in the event of a failed OpenShift Container Platform installation.
- See [Troubleshooting Operator issues](/docs/support/troubleshooting/troubleshooting-operator-issues#troubleshooting-operator-issues) for steps to check Operator pod health across the cluster and gather Operator logs for diagnosis.

### Disabling the default software catalog sources {#olm-restricted-networks-operatorhub_installing-restricted-networks-bare-metal}

To use only trusted or locally available Operator catalogs, disable the default software catalog sources that OpenShift Container Platform configures during installation. In a restricted network environment, you must disable the default catalogs as a cluster administrator.

**Procedure**

- Disable the sources for the default catalogs by adding `disableAllDefaultSources: true` to the `OperatorHub` object:
  ```terminal
  $ oc patch OperatorHub cluster --type json \
      -p '[{"op": "add", "path": "/spec/disableAllDefaultSources", "value": true}]'
  ```

  :::tip

  Or, you can use the web console to manage catalog sources. From the **Administration** → **Cluster Settings** → **Configuration** → **OperatorHub** page, click the **Sources** tab, where you can create, update, delete, disable, and enable individual sources.

  :::

### Image registry storage configuration {#installation-registry-storage-config_installing-restricted-networks-bare-metal}

The Image Registry Operator is not initially available for platforms that do not provide default storage. After installation, you must configure your registry to use storage so that the Registry Operator is made available.

Configure a persistent volume, which is required for production clusters. Where applicable, you can configure an empty directory as the storage location for non-production clusters.

You can also allow the image registry to use block storage types by using the `Recreate` rollout strategy during upgrades.

#### Change the image registry’s management state {#registry-change-management-state_installing-restricted-networks-bare-metal}

To start the image registry, you must change the Image Registry Operator configuration’s `managementState` from `Removed` to `Managed`.

**Procedure**

- Change `managementState` Image Registry Operator configuration from `Removed` to `Managed`. For example:
  ```terminal
  $ oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"managementState":"Managed"}}'
  ```

#### Configure registry storage for bare metal and other manual installations {#registry-configuring-storage-baremetal_installing-restricted-networks-bare-metal}

As a cluster administrator, following installation you must configure your registry to use storage.

**Prerequisites**

- You have access to the cluster as a user with the `cluster-admin` role.
- You have a cluster that uses manually-provisioned Red Hat Enterprise Linux CoreOS (RHCOS) nodes, such as bare metal.
- You have provisioned persistent storage for your cluster, such as Red Hat OpenShift Data Foundation.
  :::warning

  OpenShift Container Platform supports `ReadWriteOnce` access for image registry storage when you have only one replica. `ReadWriteOnce` access also requires that the registry uses the `Recreate` rollout strategy. To deploy an image registry that supports high availability with two or more replicas, `ReadWriteMany` access is required.

  :::
- You must have a system with at least 100Gi capacity.

**Procedure**

1. To configure your registry to use storage, change the `spec.storage.pvc` in the `configs.imageregistry/cluster` resource.
   :::note

   When you use shared storage, review your security settings to prevent outside access.

   :::
2. Verify that you do not have a registry pod:
   ```terminal
   $ oc get pod -n openshift-image-registry -l docker-registry=default
   ```

   ```terminal title="Example output"
   No resources found in openshift-image-registry namespace
   ```

   :::note

   If you do have a registry pod in your output, you do not need to continue with this procedure.

   :::
3. Check the registry configuration:
   ```terminal
   $ oc edit configs.imageregistry.operator.openshift.io
   ```

   ```yaml title="Example output"
   storage:
     pvc:
       claim:
   ```

   Leave the `claim` field blank to allow the automatic creation of an `image-registry-storage` PVC.
4. Check the `clusteroperator` status:
   ```terminal
   $ oc get clusteroperator image-registry
   ```

   ```terminal title="Example output"
   NAME             VERSION              AVAILABLE   PROGRESSING   DEGRADED   SINCE   MESSAGE
   image-registry   4.22                 True        False         False      6h50m
   ```
5. Ensure that your registry is set to managed to enable building and pushing of images.
   - Run:
     ```
     $ oc edit configs.imageregistry/cluster
     ```

     Then, change the line

     ```
     managementState: Removed
     ```

     to

     ```
     managementState: Managed
     ```

#### Configure storage for the image registry in non-production clusters {#installation-registry-storage-non-production_installing-restricted-networks-bare-metal}

You must configure storage for the Image Registry Operator. For non-production clusters, you can set the image registry to an empty directory, but you lose all images if you restart the registry.

**Procedure**

- To set the image registry storage to an empty directory:
  ```terminal
  $ oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"storage":{"emptyDir":{}}}}'
  ```

  :::warning

  Configure this option only for non-production clusters.

  :::

  If you run this command before the Image Registry Operator initializes its components, the `oc patch` command fails with the following error:

  ```terminal title="Example output"
  Error from server (NotFound): configs.imageregistry.operator.openshift.io "cluster" not found
  ```

  Wait a few minutes and run the command again.

#### Configure block registry storage for bare metal {#installation-registry-storage-block-recreate-rollout-bare-metal_installing-restricted-networks-bare-metal}

To allow the image registry to use block storage types during upgrades as a cluster administrator, you can use the `Recreate` rollout strategy.

:::warning

Block storage volumes, or block persistent volumes, are supported but not recommended for use with the image registry on production clusters. An installation where the registry is configured on block storage is not highly available because the registry cannot have more than one replica.

If you choose to use a block storage volume with the image registry, you must use a filesystem persistent volume claim (PVC).

:::

**Procedure**

1. Enter the following command to set the image registry storage as a block storage type, patch the registry so that it uses the `Recreate` rollout strategy, and runs with only one (`1`) replica:
   ```terminal
   $ oc patch config.imageregistry.operator.openshift.io/cluster --type=merge -p '{"spec":{"rolloutStrategy":"Recreate","replicas":1}}'
   ```
2. Provision the PV for the block storage device, and create a PVC for that volume. The requested block volume uses the ReadWriteOnce (RWO) access mode.
   1. Create a `pvc.yaml` file with the following contents to define a VMware vSphere `PersistentVolumeClaim` object:
      ```yaml
      kind: PersistentVolumeClaim
      apiVersion: v1
      metadata:
        name: image-registry-storage
        namespace: openshift-image-registry
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 100Gi
      ```

      where:

      <dl>
      <dt><code>metadata.name</code></dt>
      <dd>Specifies a unique name that represents the <code>PersistentVolumeClaim</code> object.</dd>
      <dt><code>metadata.namespace</code></dt>
      <dd>Specifies the <code>namespace</code> for the <code>PersistentVolumeClaim</code> object, which is <code>openshift-image-registry</code>.</dd>
      <dt><code>spec.accessModes</code></dt>
      <dd>Specifies the access mode of the persistent volume claim. With <code>ReadWriteOnce</code>, the volume can be mounted with read and write permissions by a single node.</dd>
      <dt><code>spec.resources.requests.storage</code></dt>
      <dd>The size of the persistent volume claim.</dd>
      </dl>
   2. Enter the following command to create the `PersistentVolumeClaim` object from the file:
      ```terminal
      $ oc create -f pvc.yaml -n openshift-image-registry
      ```
3. Enter the following command to edit the registry configuration so that it references the correct PVC:
   ```terminal
   $ oc edit config.imageregistry.operator.openshift.io -o yaml
   ```

   ```yaml title="Example output"
   storage:
     pvc:
       claim:
   ```

   By creating a custom PVC, you can leave the `claim` field blank for the default automatic creation of an `image-registry-storage` PVC.

## Completing installation on user-provisioned infrastructure {#installation-complete-user-infra_installing-restricted-networks-bare-metal}

To finalize the installation on user-provisioned infrastructure, complete the cluster deployment after configuring the Operators. This ensures the cluster is fully operational on the infrastructure that you provide.

**Prerequisites**

- Your control plane has initialized.
- You have completed the initial Operator configuration.

**Procedure**

1. Confirm that all the cluster components are online with the following command:
   ```terminal
   $ watch -n5 oc get clusteroperators
   ```

   ```terminal title="Example output"
   NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE
   authentication                             4.22.0    True        False         False      19m
   baremetal                                  4.22.0    True        False         False      37m
   cloud-credential                           4.22.0    True        False         False      40m
   cluster-autoscaler                         4.22.0    True        False         False      37m
   config-operator                            4.22.0    True        False         False      38m
   console                                    4.22.0    True        False         False      26m
   csi-snapshot-controller                    4.22.0    True        False         False      37m
   dns                                        4.22.0    True        False         False      37m
   etcd                                       4.22.0    True        False         False      36m
   image-registry                             4.22.0    True        False         False      31m
   ingress                                    4.22.0    True        False         False      30m
   insights                                   4.22.0    True        False         False      31m
   kube-apiserver                             4.22.0    True        False         False      26m
   kube-controller-manager                    4.22.0    True        False         False      36m
   kube-scheduler                             4.22.0    True        False         False      36m
   kube-storage-version-migrator              4.22.0    True        False         False      37m
   machine-api                                4.22.0    True        False         False      29m
   machine-approver                           4.22.0    True        False         False      37m
   machine-config                             4.22.0    True        False         False      36m
   marketplace                                4.22.0    True        False         False      37muser
   monitoring                                 4.22.0    True        False         False      29m
   network                                    4.22.0    True        False         False      38m
   node-tuning                                4.22.0    True        False         False      37m
   openshift-apiserver                        4.22.0    True        False         False      32muser
   openshift-controller-manager               4.22.0    True        False         False      30m
   openshift-samples                          4.22.0    True        False         False      32m
   operator-lifecycle-manager                 4.22.0    True        False         False      37m
   operator-lifecycle-manager-catalog         4.22.0    True        False         False      37m
   operator-lifecycle-manager-packageserver   4.22.0    True        False         False      32m
   service-ca                                 4.22.0    True        False         False      38m
   storage                                    4.22.0    True        False         False      37m
   ```

   Alternatively, the following command notifies you when all of the clusters are available. The command also retrieves and displays credentials:

   ```terminal
   $ ./openshift-install --dir <installation_directory> wait-for install-complete
   ```

   where:

   <dl>
   <dt><code>&lt;installation_directory&gt;</code></dt>
   <dd>Specifies the path to the directory that you stored the installation files in.</dd>
   </dl>

   ```terminal title="Example output"
   INFO Waiting up to 30m0s for the cluster to initialize...
   ```

   The command succeeds when the Cluster Version Operator finishes deploying the OpenShift Container Platform cluster from Kubernetes API server.

   :::warning

   - The Ignition config files that the installation program generates contain certificates that expire after 24 hours, which are then renewed at that time. If the cluster is shut down before renewing the certificates and the cluster is later restarted after the 24 hours have elapsed, the cluster automatically recovers the expired certificates. The exception is that you must manually approve the pending `node-bootstrapper` certificate signing requests (CSRs) to recover kubelet certificates. See the documentation for *Recovering from expired control plane certificates* for more information.
   - It is recommended that you use Ignition config files within 12 hours after they are generated because the 24-hour certificate rotates from 16 to 22 hours after the cluster is installed. By using the Ignition config files within 12 hours, you can avoid installation failure if the certificate update runs during installation.

   :::
2. Confirm that the Kubernetes API server is communicating with the pods.
   1. To view a list of all pods, use the following command:
      ```terminal
      $ oc get pods --all-namespaces
      ```

      ```terminal title="Example output"
      NAMESPACE                         NAME                                            READY   STATUS      RESTARTS   AGE
      openshift-apiserver-operator      openshift-apiserver-operator-85cb746d55-zqhs8   1/1     Running     1          9m
      openshift-apiserver               apiserver-67b9g                                 1/1     Running     0          3m
      openshift-apiserver               apiserver-ljcmx                                 1/1     Running     0          1m
      openshift-apiserver               apiserver-z25h4                                 1/1     Running     0          2m
      openshift-authentication-operator authentication-operator-69d5d8bf84-vh2n8        1/1     Running     0          5m
      ```
   2. View the logs for a pod that is listed in the output of the previous command by using the following command:
      ```terminal
      $ oc logs <pod_name> -n <namespace>
      ```

      where:

      <dl>
      <dt><code>&lt;namespace&gt;</code></dt>
      <dd>Specifies the pod name and namespace, as shown in the output of an earlier command. If the pod logs display, the Kubernetes API server can communicate with the cluster machines.</dd>
      </dl>
3. For an installation with Fibre Channel Protocol (FCP), additional steps are required to enable multipathing. Do not enable multipathing during installation.
   See "Enabling multipathing with kernel arguments on RHCOS" in the *Postinstallation machine configuration tasks* documentation for more information.
4. Register your cluster on the [Cluster registration](https://console.redhat.com/openshift/register) page.

## Telemetry access for OpenShift Container Platform {#cluster-telemetry_installing-restricted-networks-bare-metal}

To provide metrics about cluster health and the success of updates, the Telemetry service requires internet access. When connected, this service runs automatically by default and registers your cluster to [OpenShift Cluster Manager](https://console.redhat.com/openshift).

After you confirm that your [OpenShift Cluster Manager](https://console.redhat.com/openshift) inventory is correct, either maintained automatically by Telemetry or manually by using OpenShift Cluster Manager,use subscription watch to track your OpenShift Container Platform subscriptions at the account or multi-cluster level. For more information about subscription watch, see "Data Gathered and Used by Red Hat’s subscription services" in the *Additional resources* section.

**Additional resources**

- See [About remote health monitoring](/docs/support/remote_health_monitoring/about-remote-health-monitoring#about-remote-health-monitoring) for more information about the Telemetry service

**Next steps**

- [Validating an installation](/docs/installing/validation_and_troubleshooting/validating-an-installation#validating-an-installation).
- [Customize your cluster](/docs/post_installation_configuration/cluster-tasks#available_cluster_customizations).
- [Configure image streams](/docs/post_installation_configuration/cluster-tasks#post-install-must-gather-disconnected) for the Cluster Samples Operator and the `must-gather` tool.
- Learn how to [use Operator Lifecycle Manager in disconnected environments](/docs/disconnected/using-olm#olm-restricted-networks).
- If the mirror registry that you used to install your cluster has a trusted CA, add it to the cluster by [configuring additional trust stores](/docs/openshift_images/image-configuration#images-configuration-cas_image-configuration).
- If necessary, you can [Remote health reporting](/docs/support/remote_health_monitoring/remote-health-reporting#remote-health-reporting).
- If necessary, see [Registering your disconnected cluster](/docs/support/remote_health_monitoring/remote-health-reporting#insights-operator-register-disconnected-cluster_remote-health-reporting)
