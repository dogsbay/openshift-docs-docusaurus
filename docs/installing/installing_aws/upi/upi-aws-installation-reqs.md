---
title: Installation requirements for user-provisioned infrastructure on AWS
sidebar_position: 2
---

# Installation requirements for user-provisioned infrastructure on AWS {#upi-aws-installation-reqs}

<a id="upi-aws-installation-reqs"></a>

Before you install OpenShift Container Platform on infrastructure that you provision, ensure that your Amazon Web Services (AWS) environment meets the installation requirements.

For a cluster that has user-provisioned infrastructure, you must deploy all of the required machines.

## Required machines for cluster installation {#installation-machine-requirements_upi-aws-installation-reqs}

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

To keep high availability of your cluster, use separate physical hosts for these cluster machines.

:::

The bootstrap and control plane machines must use Red Hat Enterprise Linux CoreOS (RHCOS) as the operating system. However, the compute machines can use Red Hat Enterprise Linux CoreOS (RHCOS), Red Hat Enterprise Linux (RHEL) 8.6 and later.

RHCOS is based on Red Hat Enterprise Linux (RHEL) 9.8 and inherits all of its hardware certifications and requirements. See [Red Hat Enterprise Linux technology capabilities and limits](https://access.redhat.com/articles/rhel-limits).

### Minimum resource requirements for cluster installation {#installation-minimum-resource-requirements_upi-aws-installation-reqs}

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

If an instance type for your platform meets the minimum requirements for cluster machines, it is supported to use in OpenShift Container Platform.

**Additional resources**

- [Optimizing storage](/docs/scalability_and_performance/optimization/optimizing-storage#optimizing-storage)

### Tested instance types for AWS {#installation-aws-tested-machine-types_upi-aws-installation-reqs}

To ensure cluster stability and performance, use one of the tested Amazon Web Services (AWS) instance types for your OpenShift Container Platform machines.

The following AWS instance types have been tested with OpenShift Container Platform.

:::note

Use the machine types included in the following charts for your AWS instances. If you use an instance type that is not listed in the chart, ensure that the instance size you use matches the minimum resource requirements in "Minimum resource requirements for cluster installation".

:::

See the following machine types based on 64-bit x86 architecture:

- `c4.*`
- `c5.*`
- `c5a.*`
- `i3.*`
- `m4.*`
- `m5.*`
- `m5a.*`
- `m6a.*`
- `m6i.*`
- `m7a.*`
- `m7i.*`
- `m7i-flex.*`
- `r4.*`
- `r5.*`
- `r5a.*`
- `r6i.*`
- `t3.*`
- `t3a.*`

### Tested instance types for AWS on 64-bit ARM infrastructures {#installation-aws-arm-tested-machine-types_upi-aws-installation-reqs}

To ensure cluster stability and performance, use one of the tested Amazon Web Services (AWS) 64-bit ARM instance types for your OpenShift Container Platform machines.

The following AWS 64-bit ARM instance types have been tested with OpenShift Container Platform.

:::note

Use the machine types included in the following charts for your AWS ARM instances. If you use an instance type that is not listed in the chart, ensure that the instance size you use matches the minimum resource requirements listed in "Minimum resource requirements for cluster installation".

:::

See the following machine types based on 64-bit ARM architecture:

- `c6g.*`
- `c7g.*`
- `c8g.*`
- `m6g.*`
- `m7g.*`
- `m7gd.*`
- `r8g.*`

## Certificate signing requests management {#csr-management_upi-aws-installation-reqs}

On user-provisioned infrastructure, you must implement a mechanism for approving cluster certificate signing requests (CSRs) after installation when your cluster has limited access to automatic machine management.

The `kube-controller-manager` only approves the kubelet client CSRs. The `machine-approver` cannot guarantee the validity of a serving certificate that kubelet credentials request because it cannot confirm that the correct machine issued the request. You must find and implement a method of verifying the validity of the kubelet serving certificate requests and approving them.

## Required AWS infrastructure components {#installation-aws-user-infra-requirements_upi-aws-installation-reqs}

To install OpenShift Container Platform on user-provisioned infrastructure in Amazon Web Services (AWS), you must manually create both the machines and their supporting infrastructure.

For more information about the integration testing for different platforms, see the [OpenShift Container Platform 4.x Tested Integrations](https://access.redhat.com/articles/4128421) page.

By using the provided CloudFormation templates, you can create stacks of AWS resources that represent the following components:

- An AWS Virtual Private Cloud (VPC)
- Networking and load balancing components
- Security groups and roles
- An OpenShift Container Platform bootstrap node
- OpenShift Container Platform control plane nodes
- An OpenShift Container Platform compute node

Or, you can manually create the components or you can reuse existing infrastructure that meets the cluster requirements. Review the CloudFormation templates for more details about how the components interrelate.

### Other infrastructure components {#installation-aws-user-infra-other-infrastructure_upi-aws-installation-reqs}

Your OpenShift Container Platform cluster on user-provisioned infrastructure in Amazon Web Services (AWS) requires additional infrastructure components, including a VPC, DNS entries, load balancers, security groups, and IAM roles.

- A VPC
- DNS entries
- Load balancers (classic or network) and listeners
- A public and a private Route 53 zone
- Security groups
- IAM roles
- S3 buckets

If you are working in a disconnected environment, you cannot reach the public IP addresses for EC2, ELB, and S3 endpoints. Depending on the level to which you want to restrict internet traffic during the installation, the following configuration options are available:

#### Option 1: Create VPC endpoints {#create-vpc-endpoints_upi-aws-installation-reqs}

Create a VPC endpoint and attach it to the subnets that the clusters are using. Name the endpoints as follows:

- `ec2.<aws_region>.amazonaws.com`
- `elasticloadbalancing.<aws_region>.amazonaws.com`
- `s3.<aws_region>.amazonaws.com`

With this option, network traffic remains private between your VPC and the required AWS services.

#### Option 2: Create a proxy without VPC endpoints {#create-proxy-without-vpc-endpoints_upi-aws-installation-reqs}

As part of the installation process, you can configure an HTTP or HTTPS proxy. With this option, internet traffic goes through the proxy to reach the required AWS services.

#### Option 3: Create a proxy with VPC endpoints {#create-proxy-with-vpc-endpoints_upi-aws-installation-reqs}

As part of the installation process, you can configure an HTTP or HTTPS proxy with VPC endpoints. Create a VPC endpoint and attach it to the subnets that the clusters are using. Name the endpoints as follows:

- `ec2.<aws_region>.amazonaws.com`
- `elasticloadbalancing.<aws_region>.amazonaws.com`
- `s3.<aws_region>.amazonaws.com`

When configuring the proxy in the `install-config.yaml` file, add these endpoints to the `noProxy` field. With this option, the proxy prevents the cluster from accessing the internet directly. However, network traffic remains private between your VPC and the required AWS services.

You must provide a suitable VPC and subnets that allow communication to your machines.

**Required VPC components**

<table>
<thead>
<tr>
  <th>Component</th>
  <th>AWS type</th>
  <th colspan="2">Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>VPC</td>
  <td><ul><li><code>AWS::EC2::VPC</code></li><li><code>AWS::EC2::VPCEndpoint</code></li></ul></td>
  <td colspan="2">You must provide a public VPC for the cluster to use. The VPC uses an endpoint that references the route tables for each subnet to improve communication with the registry that is hosted in S3.</td>
</tr>
<tr>
  <td>Public subnets</td>
  <td><ul><li><code>AWS::EC2::Subnet</code></li><li><code>AWS::EC2::SubnetNetworkAclAssociation</code></li></ul></td>
  <td colspan="2">Your VPC must have public subnets for between 1 and 3 availability zones and associate them with appropriate Ingress rules.</td>
</tr>
<tr>
  <td>Internet gateway</td>
  <td><ul><li><code>AWS::EC2::InternetGateway</code></li><li><code>AWS::EC2::VPCGatewayAttachment</code></li><li><code>AWS::EC2::RouteTable</code></li><li><code>AWS::EC2::Route</code></li><li><code>AWS::EC2::SubnetRouteTableAssociation</code></li><li><code>AWS::EC2::NatGateway</code></li><li><code>AWS::EC2::EIP</code></li></ul></td>
  <td colspan="2">You must have a public internet gateway, with public routes, attached to the VPC. In the provided templates, each public subnet has a NAT gateway with an EIP address. These NAT gateways allow cluster resources, such as private subnet instances, to reach the internet and are not required for some restricted network or proxy scenarios.</td>
</tr>
<tr>
  <td rowspan="7">Network access control</td>
  <td rowspan="7"><ul><li><code>AWS::EC2::NetworkAcl</code></li><li><code>AWS::EC2::NetworkAclEntry</code></li></ul></td>
  <td colspan="2">You must allow the VPC to access the following ports:</td>
</tr>
<tr>
  <th>Port</th>
  <th>Reason</th>
</tr>
<tr>
  <td><code>80</code></td>
  <td>Inbound HTTP traffic</td>
</tr>
<tr>
  <td><code>443</code></td>
  <td>Inbound HTTPS traffic</td>
</tr>
<tr>
  <td><code>22</code></td>
  <td>Inbound SSH traffic</td>
</tr>
<tr>
  <td><code>1024</code> - <code>65535</code></td>
  <td>Inbound ephemeral traffic</td>
</tr>
<tr>
  <td><code>0</code> - <code>65535</code></td>
  <td>Outbound ephemeral traffic</td>
</tr>
<tr>
  <td>Private subnets</td>
  <td><ul><li><code>AWS::EC2::Subnet</code></li><li><code>AWS::EC2::RouteTable</code></li><li><code>AWS::EC2::SubnetRouteTableAssociation</code></li></ul></td>
  <td colspan="2">Your VPC can have private subnets. The provided CloudFormation templates can create private subnets for between 1 and 3 availability zones. If you use private subnets, you must provide appropriate routes and tables for them.</td>
</tr>
</tbody>
</table>

Your DNS and load balancer configuration must use a public hosted zone and can use a private hosted zone similar to the one that the installation program uses if it provisions the cluster’s infrastructure. You must create a DNS entry that resolves to your load balancer. An entry for `api.<cluster_name>.<domain>` must point to the external load balancer, and an entry for `api-int.<cluster_name>.<domain>` must point to the internal load balancer.

The cluster also requires load balancers and listeners for port 6443, which the Kubernetes API and its extensions require, and port 22623, which the Ignition config files for new machines require. The targets are the control plane nodes. Port 6443 must be accessible to both clients external to the cluster and nodes within the cluster. Port 22623 must be accessible to nodes within the cluster.

**Required DNS and load balancing components**

<table>
<thead>
<tr>
  <th>Component</th>
  <th>AWS type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>DNS</td>
  <td><code>AWS::Route53::HostedZone</code></td>
  <td>The hosted zone for your internal DNS.</td>
</tr>
<tr>
  <td>Public load balancer</td>
  <td><code>AWS::ElasticLoadBalancingV2::LoadBalancer</code></td>
  <td>The load balancer for your public subnets.</td>
</tr>
<tr>
  <td>External API server record</td>
  <td><code>AWS::Route53::RecordSetGroup</code></td>
  <td>Alias records for the external API server.</td>
</tr>
<tr>
  <td>External listener</td>
  <td><code>AWS::ElasticLoadBalancingV2::Listener</code></td>
  <td>A listener on port 6443 for the external load balancer.</td>
</tr>
<tr>
  <td>External target group</td>
  <td><code>AWS::ElasticLoadBalancingV2::TargetGroup</code></td>
  <td>The target group for the external load balancer.</td>
</tr>
<tr>
  <td>Private load balancer</td>
  <td><code>AWS::ElasticLoadBalancingV2::LoadBalancer</code></td>
  <td>The load balancer for your private subnets.</td>
</tr>
<tr>
  <td>Internal API server record</td>
  <td><code>AWS::Route53::RecordSetGroup</code></td>
  <td>Alias records for the internal API server.</td>
</tr>
<tr>
  <td>Internal listener</td>
  <td><code>AWS::ElasticLoadBalancingV2::Listener</code></td>
  <td>A listener on port 22623 for the internal load balancer.</td>
</tr>
<tr>
  <td>Internal target group</td>
  <td><code>AWS::ElasticLoadBalancingV2::TargetGroup</code></td>
  <td>The target group for the internal load balancer.</td>
</tr>
<tr>
  <td>Internal listener</td>
  <td><code>AWS::ElasticLoadBalancingV2::Listener</code></td>
  <td>A listener on port 6443 for the internal load balancer.</td>
</tr>
<tr>
  <td>Internal target group</td>
  <td><code>AWS::ElasticLoadBalancingV2::TargetGroup</code></td>
  <td>The target group for the internal load balancer.</td>
</tr>
</tbody>
</table>

The control plane and worker machines require access to the following ports:

**Security groups**

<table>
<thead>
<tr>
  <th>Group</th>
  <th>Type</th>
  <th>IP Protocol</th>
  <th>Port range</th>
</tr>
</thead>
<tbody>
<tr>
  <td rowspan="4"><code>MasterSecurityGroup</code></td>
  <td rowspan="4"><code>AWS::EC2::SecurityGroup</code></td>
  <td><code>icmp</code></td>
  <td><code>0</code></td>
</tr>
<tr>
  <td><code>tcp</code></td>
  <td><code>22</code></td>
</tr>
<tr>
  <td><code>tcp</code></td>
  <td><code>6443</code></td>
</tr>
<tr>
  <td><code>tcp</code></td>
  <td><code>22623</code></td>
</tr>
<tr>
  <td rowspan="2"><code>WorkerSecurityGroup</code></td>
  <td rowspan="2"><code>AWS::EC2::SecurityGroup</code></td>
  <td><code>icmp</code></td>
  <td><code>0</code></td>
</tr>
<tr>
  <td><code>tcp</code></td>
  <td><code>22</code></td>
</tr>
<tr>
  <td rowspan="2"><code>BootstrapSecurityGroup</code></td>
  <td rowspan="2"><code>AWS::EC2::SecurityGroup</code></td>
  <td><code>tcp</code></td>
  <td><code>22</code></td>
</tr>
<tr>
  <td><code>tcp</code></td>
  <td><code>19531</code></td>
</tr>
</tbody>
</table>

The control plane machines require the following Ingress groups. Each Ingress group is an `AWS::EC2::SecurityGroupIngress` resource.

**Control plane Ingress**

<table>
<thead>
<tr>
  <th>Ingress group</th>
  <th>Description</th>
  <th>IP protocol</th>
  <th>Port range</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>MasterIngressEtcd</code></td>
  <td>etcd</td>
  <td><code>tcp</code></td>
  <td><code>2379</code>- <code>2380</code></td>
</tr>
<tr>
  <td><code>MasterIngressVxlan</code></td>
  <td>Vxlan packets</td>
  <td><code>udp</code></td>
  <td><code>6081</code></td>
</tr>
<tr>
  <td><code>MasterIngressWorkerVxlan</code></td>
  <td>Vxlan packets</td>
  <td><code>udp</code></td>
  <td><code>6081</code></td>
</tr>
<tr>
  <td><code>MasterIngressInternal</code></td>
  <td>Internal cluster communication and Kubernetes proxy metrics</td>
  <td><code>tcp</code></td>
  <td><code>9000</code> - <code>9999</code></td>
</tr>
<tr>
  <td><code>MasterIngressWorkerInternal</code></td>
  <td>Internal cluster communication</td>
  <td><code>tcp</code></td>
  <td><code>9000</code> - <code>9999</code></td>
</tr>
<tr>
  <td><code>MasterIngressKube</code></td>
  <td>Kubernetes kubelet, scheduler and controller manager</td>
  <td><code>tcp</code></td>
  <td><code>10250</code> - <code>10259</code></td>
</tr>
<tr>
  <td><code>MasterIngressWorkerKube</code></td>
  <td>Kubernetes kubelet, scheduler and controller manager</td>
  <td><code>tcp</code></td>
  <td><code>10250</code> - <code>10259</code></td>
</tr>
<tr>
  <td><code>MasterIngressIngressServices</code></td>
  <td>Kubernetes Ingress services</td>
  <td><code>tcp</code></td>
  <td><code>30000</code> - <code>32767</code></td>
</tr>
<tr>
  <td><code>MasterIngressWorkerIngressServices</code></td>
  <td>Kubernetes Ingress services</td>
  <td><code>tcp</code></td>
  <td><code>30000</code> - <code>32767</code></td>
</tr>
<tr>
  <td><code>MasterIngressGeneve</code></td>
  <td>Geneve packets</td>
  <td><code>udp</code></td>
  <td><code>6081</code></td>
</tr>
<tr>
  <td><code>MasterIngressWorkerGeneve</code></td>
  <td>Geneve packets</td>
  <td><code>udp</code></td>
  <td><code>6081</code></td>
</tr>
<tr>
  <td><code>MasterIngressIpsecIke</code></td>
  <td>IPsec IKE packets</td>
  <td><code>udp</code></td>
  <td><code>500</code></td>
</tr>
<tr>
  <td><code>MasterIngressWorkerIpsecIke</code></td>
  <td>IPsec IKE packets</td>
  <td><code>udp</code></td>
  <td><code>500</code></td>
</tr>
<tr>
  <td><code>MasterIngressIpsecNat</code></td>
  <td>IPsec NAT-T packets</td>
  <td><code>udp</code></td>
  <td><code>4500</code></td>
</tr>
<tr>
  <td><code>MasterIngressWorkerIpsecNat</code></td>
  <td>IPsec NAT-T packets</td>
  <td><code>udp</code></td>
  <td><code>4500</code></td>
</tr>
<tr>
  <td><code>MasterIngressIpsecEsp</code></td>
  <td>IPsec ESP packets</td>
  <td><code>50</code></td>
  <td><code>All</code></td>
</tr>
<tr>
  <td><code>MasterIngressWorkerIpsecEsp</code></td>
  <td>IPsec ESP packets</td>
  <td><code>50</code></td>
  <td><code>All</code></td>
</tr>
<tr>
  <td><code>MasterIngressInternalUDP</code></td>
  <td>Internal cluster communication</td>
  <td><code>udp</code></td>
  <td><code>9000</code> - <code>9999</code></td>
</tr>
<tr>
  <td><code>MasterIngressWorkerInternalUDP</code></td>
  <td>Internal cluster communication</td>
  <td><code>udp</code></td>
  <td><code>9000</code> - <code>9999</code></td>
</tr>
<tr>
  <td><code>MasterIngressIngressServicesUDP</code></td>
  <td>Kubernetes Ingress services</td>
  <td><code>udp</code></td>
  <td><code>30000</code> - <code>32767</code></td>
</tr>
<tr>
  <td><code>MasterIngressWorkerIngressServicesUDP</code></td>
  <td>Kubernetes Ingress services</td>
  <td><code>udp</code></td>
  <td><code>30000</code> - <code>32767</code></td>
</tr>
</tbody>
</table>

The worker machines require the following Ingress groups. Each Ingress group is an `AWS::EC2::SecurityGroupIngress` resource.

**Worker Ingress**

<table>
<thead>
<tr>
  <th>Ingress group</th>
  <th>Description</th>
  <th>IP protocol</th>
  <th>Port range</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>WorkerIngressVxlan</code></td>
  <td>Vxlan packets</td>
  <td><code>udp</code></td>
  <td><code>6081</code></td>
</tr>
<tr>
  <td><code>WorkerIngressWorkerVxlan</code></td>
  <td>Vxlan packets</td>
  <td><code>udp</code></td>
  <td><code>6081</code></td>
</tr>
<tr>
  <td><code>WorkerIngressInternal</code></td>
  <td>Internal cluster communication</td>
  <td><code>tcp</code></td>
  <td><code>9000</code> - <code>9999</code></td>
</tr>
<tr>
  <td><code>WorkerIngressWorkerInternal</code></td>
  <td>Internal cluster communication</td>
  <td><code>tcp</code></td>
  <td><code>9000</code> - <code>9999</code></td>
</tr>
<tr>
  <td><code>WorkerIngressKube</code></td>
  <td>Kubernetes kubelet, scheduler, and controller manager</td>
  <td><code>tcp</code></td>
  <td><code>10250</code></td>
</tr>
<tr>
  <td><code>WorkerIngressWorkerKube</code></td>
  <td>Kubernetes kubelet, scheduler, and controller manager</td>
  <td><code>tcp</code></td>
  <td><code>10250</code></td>
</tr>
<tr>
  <td><code>WorkerIngressIngressServices</code></td>
  <td>Kubernetes Ingress services</td>
  <td><code>tcp</code></td>
  <td><code>30000</code> - <code>32767</code></td>
</tr>
<tr>
  <td><code>WorkerIngressWorkerIngressServices</code></td>
  <td>Kubernetes Ingress services</td>
  <td><code>tcp</code></td>
  <td><code>30000</code> - <code>32767</code></td>
</tr>
<tr>
  <td><code>WorkerIngressGeneve</code></td>
  <td>Geneve packets</td>
  <td><code>udp</code></td>
  <td><code>6081</code></td>
</tr>
<tr>
  <td><code>WorkerIngressMasterGeneve</code></td>
  <td>Geneve packets</td>
  <td><code>udp</code></td>
  <td><code>6081</code></td>
</tr>
<tr>
  <td><code>WorkerIngressIpsecIke</code></td>
  <td>IPsec IKE packets</td>
  <td><code>udp</code></td>
  <td><code>500</code></td>
</tr>
<tr>
  <td><code>WorkerIngressMasterIpsecIke</code></td>
  <td>IPsec IKE packets</td>
  <td><code>udp</code></td>
  <td><code>500</code></td>
</tr>
<tr>
  <td><code>WorkerIngressIpsecNat</code></td>
  <td>IPsec NAT-T packets</td>
  <td><code>udp</code></td>
  <td><code>4500</code></td>
</tr>
<tr>
  <td><code>WorkerIngressMasterIpsecNat</code></td>
  <td>IPsec NAT-T packets</td>
  <td><code>udp</code></td>
  <td><code>4500</code></td>
</tr>
<tr>
  <td><code>WorkerIngressIpsecEsp</code></td>
  <td>IPsec ESP packets</td>
  <td><code>50</code></td>
  <td><code>All</code></td>
</tr>
<tr>
  <td><code>WorkerIngressMasterIpsecEsp</code></td>
  <td>IPsec ESP packets</td>
  <td><code>50</code></td>
  <td><code>All</code></td>
</tr>
<tr>
  <td><code>WorkerIngressInternalUDP</code></td>
  <td>Internal cluster communication</td>
  <td><code>udp</code></td>
  <td><code>9000</code> - <code>9999</code></td>
</tr>
<tr>
  <td><code>WorkerIngressMasterInternalUDP</code></td>
  <td>Internal cluster communication</td>
  <td><code>udp</code></td>
  <td><code>9000</code> - <code>9999</code></td>
</tr>
<tr>
  <td><code>WorkerIngressIngressServicesUDP</code></td>
  <td>Kubernetes Ingress services</td>
  <td><code>udp</code></td>
  <td><code>30000</code> - <code>32767</code></td>
</tr>
<tr>
  <td><code>WorkerIngressMasterIngressServicesUDP</code></td>
  <td>Kubernetes Ingress services</td>
  <td><code>udp</code></td>
  <td><code>30000</code> - <code>32767</code></td>
</tr>
</tbody>
</table>

You must grant the machines permissions in AWS. The provided CloudFormation templates grant the machines `Allow` permissions for the following `AWS::IAM::Role` objects and provide an `AWS::IAM::InstanceProfile` for each set of roles. If you do not use the templates, you can grant the machines the following broad permissions or the following individual permissions.

**Roles and instance profiles**

<table>
<thead>
<tr>
  <th>Role</th>
  <th>Effect</th>
  <th>Action</th>
  <th>Resource</th>
</tr>
</thead>
<tbody>
<tr>
  <td rowspan="4">Control plane</td>
  <td><code>Allow</code></td>
  <td><code>ec2:*</code></td>
  <td><code>*</code></td>
</tr>
<tr>
  <td><code>Allow</code></td>
  <td><code>elasticloadbalancing:*</code></td>
  <td><code>*</code></td>
</tr>
<tr>
  <td><code>Allow</code></td>
  <td><code>iam:PassRole</code></td>
  <td><code>*</code></td>
</tr>
<tr>
  <td><code>Allow</code></td>
  <td><code>s3:GetObject</code></td>
  <td><code>*</code></td>
</tr>
<tr>
  <td>Worker</td>
  <td><code>Allow</code></td>
  <td><code>ec2:Describe*</code></td>
  <td><code>*</code></td>
</tr>
<tr>
  <td rowspan="3">Bootstrap</td>
  <td><code>Allow</code></td>
  <td><code>ec2:Describe*</code></td>
  <td><code>*</code></td>
</tr>
<tr>
  <td><code>Allow</code></td>
  <td><code>ec2:AttachVolume</code></td>
  <td><code>*</code></td>
</tr>
<tr>
  <td><code>Allow</code></td>
  <td><code>ec2:DetachVolume</code></td>
  <td><code>*</code></td>
</tr>
</tbody>
</table>

### Cluster machines {#installation-aws-user-infra-cluster-machines_upi-aws-installation-reqs}

Your OpenShift Container Platform cluster on user-provisioned infrastructure requires `AWS::EC2::Instance` objects for bootstrap, control plane, and compute machines.

You need `AWS::EC2::Instance` objects for the following machines:

- A bootstrap machine. This machine is required during installation, but you can remove it after your cluster deploys.
- Three control plane machines. A control plane machine set does not govern the control plane machines.
- Compute machines. You must create at least two compute machines, which are also known as worker machines, during installation. A compute machine set does not govern these machines.

## Required AWS permissions for the IAM user {#installation-aws-permissions_upi-aws-installation-reqs}

To deploy all components of an OpenShift Container Platform cluster, you must grant all the required permissions to the IAM user that you create in Amazon Web Services (AWS).

:::note

Your IAM user must have the permission `tag:GetResources` in the region `us-east-1` to delete the base cluster resources. As part of the AWS API requirement, the OpenShift Container Platform installation program performs various actions in this region.

:::

When you attach the `AdministratorAccess` policy to the IAM user that you create in AWS, you grant that user all of the required permissions. To deploy all components of an OpenShift Container Platform cluster, the IAM user requires the following permissions:

**Required EC2 permissions for installation**

- `ec2:AttachNetworkInterface`
- `ec2:AuthorizeSecurityGroupEgress`
- `ec2:AuthorizeSecurityGroupIngress`
- `ec2:CopyImage`
- `ec2:CreateNetworkInterface`
- `ec2:CreateSecurityGroup`
- `ec2:CreateTags`
- `ec2:CreateVolume`
- `ec2:DeleteSecurityGroup`
- `ec2:DeleteSnapshot`
- `ec2:DeleteTags`
- `ec2:DeregisterImage`
- `ec2:DescribeAccountAttributes`
- `ec2:DescribeAddresses`
- `ec2:DescribeAvailabilityZones`
- `ec2:DescribeDhcpOptions`
- `ec2:DescribeImages`
- `ec2:DescribeInstanceAttribute`
- `ec2:DescribeInstanceCreditSpecifications`
- `ec2:DescribeInstances`
- `ec2:DescribeInstanceTypes`
- `ec2:DescribeInstanceTypeOfferings`
- `ec2:DescribeInternetGateways`
- `ec2:DescribeKeyPairs`
- `ec2:DescribeNatGateways`
- `ec2:DescribeNetworkAcls`
- `ec2:DescribeNetworkInterfaces`
- `ec2:DescribePrefixLists`
- `ec2:DescribePublicIpv4Pools` (only required if you specify `publicIpv4Pool` in `install-config.yaml`)
- `ec2:DescribeRegions`
- `ec2:DescribeRouteTables`
- `ec2:DescribeSecurityGroupRules`
- `ec2:DescribeSecurityGroups`
- `ec2:DescribeSubnets`
- `ec2:DescribeTags`
- `ec2:DescribeVolumes`
- `ec2:DescribeVpcAttribute`
- `ec2:DescribeVpcClassicLink`
- `ec2:DescribeVpcClassicLinkDnsSupport`
- `ec2:DescribeVpcEndpoints`
- `ec2:DescribeVpcs`
- `ec2:DisassociateAddress` (only required if you specify `publicIpv4Pool` in `install-config.yaml`)
- `ec2:GetEbsDefaultKmsKeyId`
- `ec2:ModifyInstanceAttribute`
- `ec2:ModifyNetworkInterfaceAttribute`
- `ec2:RevokeSecurityGroupEgress`
- `ec2:RevokeSecurityGroupIngress`
- `ec2:RunInstances`
- `ec2:TerminateInstances`

**Required permissions for creating network resources during installation**

- `ec2:AllocateAddress`
- `ec2:AssociateAddress`
- `ec2:AssociateDhcpOptions`
- `ec2:AssociateRouteTable`
- `ec2:AttachInternetGateway`
- `ec2:CreateDhcpOptions`
- `ec2:CreateInternetGateway`
- `ec2:CreateNatGateway`
- `ec2:CreateRoute`
- `ec2:CreateRouteTable`
- `ec2:CreateSubnet`
- `ec2:CreateVpc`
- `ec2:CreateVpcEndpoint`
- `ec2:ModifySubnetAttribute`
- `ec2:ModifyVpcAttribute`

:::note

If you use an existing Virtual Private Cloud (VPC), your account does not require these permissions for creating network resources.

:::

**Required Elastic Load Balancing permissions (ELB) for installation**

- `elasticloadbalancing:AddTags`
- `elasticloadbalancing:ApplySecurityGroupsToLoadBalancer`
- `elasticloadbalancing:AttachLoadBalancerToSubnets`
- `elasticloadbalancing:ConfigureHealthCheck`
- `elasticloadbalancing:CreateListener`
- `elasticloadbalancing:CreateLoadBalancer`
- `elasticloadbalancing:CreateLoadBalancerListeners`
- `elasticloadbalancing:CreateTargetGroup`
- `elasticloadbalancing:DeleteLoadBalancer`
- `elasticloadbalancing:DeregisterInstancesFromLoadBalancer`
- `elasticloadbalancing:DeregisterTargets`
- `elasticloadbalancing:DescribeInstanceHealth`
- `elasticloadbalancing:DescribeListeners`
- `elasticloadbalancing:DescribeLoadBalancerAttributes`
- `elasticloadbalancing:DescribeLoadBalancers`
- `elasticloadbalancing:DescribeTags`
- `elasticloadbalancing:DescribeTargetGroupAttributes`
- `elasticloadbalancing:DescribeTargetHealth`
- `elasticloadbalancing:ModifyLoadBalancerAttributes`
- `elasticloadbalancing:ModifyTargetGroup`
- `elasticloadbalancing:ModifyTargetGroupAttributes`
- `elasticloadbalancing:RegisterInstancesWithLoadBalancer`
- `elasticloadbalancing:RegisterTargets`
- `elasticloadbalancing:SetLoadBalancerPoliciesOfListener`
- `elasticloadbalancing:SetSecurityGroups`

:::warning

OpenShift Container Platform uses both the ELB and ELBv2 API services to provision load balancers. The permission list shows permissions required by both services. A known issue exists in the AWS web console where both services use the same `elasticloadbalancing` action prefix but do not recognize the same actions. You can ignore the warnings about the service not recognizing certain `elasticloadbalancing` actions.

:::

**Required IAM permissions for installation**

- `iam:AddRoleToInstanceProfile`
- `iam:CreateInstanceProfile`
- `iam:CreateRole`
- `iam:DeleteInstanceProfile`
- `iam:DeleteRole`
- `iam:DeleteRolePolicy`
- `iam:GetInstanceProfile`
- `iam:GetRole`
- `iam:GetRolePolicy`
- `iam:GetUser`
- `iam:ListInstanceProfilesForRole`
- `iam:ListRoles`
- `iam:ListUsers`
- `iam:PassRole`
- `iam:PutRolePolicy`
- `iam:RemoveRoleFromInstanceProfile`
- `iam:SimulatePrincipalPolicy`
- `iam:TagInstanceProfile`
- `iam:TagRole`

:::note

- If you specify an existing IAM role in the `install-config.yaml` file, the following IAM permissions are not required: `iam:CreateRole`,`iam:DeleteRole`, `iam:DeleteRolePolicy`, and `iam:PutRolePolicy`.
- If you have not created a load balancer in your AWS account, the IAM user also requires the `iam:CreateServiceLinkedRole` permission.

:::

**Required Route 53 permissions for installation**

- `route53:ChangeResourceRecordSets`
- `route53:ChangeTagsForResource`
- `route53:CreateHostedZone`
- `route53:DeleteHostedZone`
- `route53:GetChange`
- `route53:GetHostedZone`
- `route53:ListHostedZones`
- `route53:ListHostedZonesByName`
- `route53:ListResourceRecordSets`
- `route53:ListTagsForResource`
- `route53:UpdateHostedZoneComment`

**Required Amazon Simple Storage Service (S3) permissions for installation**

- `s3:CreateBucket`
- `s3:DeleteBucket`
- `s3:GetAccelerateConfiguration`
- `s3:GetBucketAcl`
- `s3:GetBucketCors`
- `s3:GetBucketLocation`
- `s3:GetBucketLogging`
- `s3:GetBucketObjectLockConfiguration`
- `s3:GetBucketPolicy`
- `s3:GetBucketRequestPayment`
- `s3:GetBucketTagging`
- `s3:GetBucketVersioning`
- `s3:GetBucketWebsite`
- `s3:GetEncryptionConfiguration`
- `s3:GetLifecycleConfiguration`
- `s3:GetReplicationConfiguration`
- `s3:ListBucket`
- `s3:PutBucketAcl`
- `s3:PutBucketPolicy`
- `s3:PutBucketTagging`
- `s3:PutEncryptionConfiguration`

**S3 permissions that cluster Operators require**

- `s3:DeleteObject`
- `s3:GetObject`
- `s3:GetObjectAcl`
- `s3:GetObjectTagging`
- `s3:GetObjectVersion`
- `s3:PutObject`
- `s3:PutObjectAcl`
- `s3:PutObjectTagging`

**Required permissions to delete base cluster resources**

- `autoscaling:DescribeAutoScalingGroups`
- `ec2:DeleteNetworkInterface`
- `ec2:DeletePlacementGroup`
- `ec2:DeleteVolume`
- `elasticloadbalancing:DeleteTargetGroup`
- `elasticloadbalancing:DescribeTargetGroups`
- `iam:DeleteAccessKey`
- `iam:DeleteUser`
- `iam:DeleteUserPolicy`
- `iam:ListAttachedRolePolicies`
- `iam:ListInstanceProfiles`
- `iam:ListRolePolicies`
- `iam:ListUserPolicies`
- `s3:DeleteObject`
- `s3:ListBucketVersions`
- `tag:GetResources`

**Required permissions to delete network resources**

- `ec2:DeleteDhcpOptions`
- `ec2:DeleteInternetGateway`
- `ec2:DeleteNatGateway`
- `ec2:DeleteRoute`
- `ec2:DeleteRouteTable`
- `ec2:DeleteSubnet`
- `ec2:DeleteVpc`
- `ec2:DeleteVpcEndpoints`
- `ec2:DetachInternetGateway`
- `ec2:DisassociateRouteTable`
- `ec2:ReleaseAddress`
- `ec2:ReplaceRouteTableAssociation`

:::note

If you use an existing VPC, your account does not require these permissions to delete network resources. Instead, your account only requires the `tag:UntagResources` permission to delete network resources.

:::

**Optional permissions for installing a cluster with a custom Key Management Service (KMS) key**

- `kms:CreateGrant`
- `kms:Decrypt`
- `kms:DescribeKey`
- `kms:Encrypt`
- `kms:GenerateDataKey`
- `kms:GenerateDataKeyWithoutPlainText`
- `kms:ListGrants`
- `kms:RevokeGrant`

:::note

If you use an Amazon Machine Image (AMI) encrypted with a customer-managed key, you must grant the `kms:ReEncrypt*` permissions in addition to these permissions.

:::

**Required permissions to delete a cluster with shared instance roles**

- `iam:UntagRole`

**Required permissions to delete a cluster with shared instance profiles**

- `tag:UntagResources`

**Additional IAM and S3 permissions required to create manifests**

- `iam:GetUserPolicy`
- `iam:ListAccessKeys`
- `iam:PutUserPolicy`
- `iam:TagUser`
- `s3:AbortMultipartUpload`
- `s3:GetBucketPublicAccessBlock`
- `s3:ListBucket`
- `s3:ListBucketMultipartUploads`
- `s3:PutBucketPublicAccessBlock`
- `s3:PutLifecycleConfiguration`

:::note

If you are managing your cloud provider credentials with mint mode, the IAM user also requires the `iam:CreateAccessKey` and `iam:CreateUser` permissions.

:::

**Optional permissions for instance and quota checks for installation**

- `servicequotas:ListAWSDefaultServiceQuotas`

**Optional permissions for the cluster owner account when installing a cluster on a shared VPC**

- `sts:AssumeRole`

**Required permissions for enabling Bring your own public IPv4 addresses (BYOIP) feature for installation**

- `ec2:DescribePublicIpv4Pools`
- `ec2:DisassociateAddress`

**Required permissions to install a cluster with dual-stack networking**

- `ec2:DescribeEgressOnlyInternetGateways`
- `ec2:CreateEgressOnlyInternetGateway`

**Required permissions to delete a cluster with dual-stack networking**

- `ec2:DeleteEgressOnlyInternetGateway`

## Obtaining an AWS Marketplace image {#installation-aws-marketplace-subscribe_upi-aws-installation-reqs}

If you are deploying an OpenShift Container Platform cluster by using an Amazon Web Services (AWS) Marketplace image, you must first subscribe through AWS. Subscribing to the offer provides you with the Amazon Machine Image (AMI) ID that the installation program uses to deploy compute nodes.

:::note

You should only modify the RHCOS image for compute machines to use an AWS Marketplace image. Control plane machines and infrastructure nodes do not require an OpenShift Container Platform subscription and use the public RHCOS default image by default, which does not incur subscription costs on your AWS bill. Therefore, you should not modify the cluster default boot image or the control plane boot images. Applying the AWS Marketplace image to them will incur additional licensing costs that cannot be recovered.

:::

**Prerequisites**

- You have an AWS account to buy the offer. This account does not have to be the same account that you use to install the cluster.

**Procedure**

1. Complete the OpenShift Container Platform subscription from the [AWS Marketplace](https://aws.amazon.com/marketplace/fulfillment?productId=59ead7de-2540-4653-a8b0-fa7926d5c845).
