---
title: Local storage overview
sidebar_position: 1
---

# Local storage overview {#ways-to-provision-local-storage_{context}}

<a id="ways-to-provision-local-storage_{context}"></a>

Local storage provides direct access to disks attached to cluster nodes, delivering lower latency and higher throughput than network-attached or cloud-based storage. Use local storage for performance-sensitive workloads, single-node clusters, or environments without cloud storage infrastructure.

## Overview of local storage options {#persistent-storage-local-top-level-overview_ways-to-provision-local-storage}

OpenShift Container Platform provides three solutions for provisioning node-local storage. Each offers different capabilities for dynamic provisioning and topology awareness. Node-local storage binds workloads to specific nodes.

You can use any of the following solutions to provision local storage:

- HostPath Provisioner (HPP)
- Local Storage Operator (LSO)
- Logical Volume Manager (LVM) Storage

:::warning

These solutions support provisioning only node-local storage. The workloads are bound to the nodes that provide the storage. If the node becomes unavailable, the workload also becomes unavailable. To maintain workload availability despite node failures, you must ensure storage data replication through active or passive replication mechanisms.

:::

### Overview of HostPath Provisioner functionality {#overview-of-hpp-functionality_ways-to-provision-local-storage}

You can perform the following actions using HostPath Provisioner (HPP):

- Map the host filesystem paths to storage classes for provisioning local storage.
- Statically create storage classes to configure filesystem paths on a node for storage consumption.
- Statically provision Persistent Volumes (PVs) based on the storage class.
- Create workloads and PersistentVolumeClaims (PVCs) while being aware of the underlying storage topology.

:::note

HPP is available in upstream Kubernetes. However, it is not recommended to use HPP from upstream Kubernetes.

:::

### Overview of Local Storage Operator functionality {#overview-of-lso-functionality_ways-to-provision-local-storage}

You can perform the following actions using Local Storage Operator (LSO):

- Assign the storage devices (disks or partitions) to the storage classes without modifying the device configuration.
- Statically provision PVs and storage classes by configuring the `LocalVolume` custom resource (CR).
- Create workloads and PVCs while being aware of the underlying storage topology.

:::note

LSO is developed and delivered by Red Hat.

:::

### Overview of LVM Storage functionality {#overview-of-lvm-storage-functionality_ways-to-provision-local-storage}

You can perform the following actions using Logical Volume Manager (LVM) Storage:

- Configure storage devices (disks or partitions) as lvm2 volume groups and expose the volume groups as storage classes.
- Create workloads and request storage by using PVCs without considering the node topology.

LVM Storage uses the TopoLVM CSI driver to dynamically allocate storage space to the nodes in the topology and provision PVs.

:::note

LVM Storage is developed and maintained by Red Hat. The CSI driver provided with LVM Storage is the upstream project "topolvm".

:::

## Comparison of LVM Storage, LSO, and HPP {#comparison-of-solutions-to-provision-node-local-storage_ways-to-provision-local-storage}

Compare LVM Storage, Local Storage Operator (LSO), and HostPath Provisioner (HPP) across storage types, core features, performance, and isolation to determine the best local storage provisioning solution for your cluster.

### Comparison of the support for storage types and filesystems {#comparing-storage-types_ways-to-provision-local-storage}

The following table compares the support for storage types and filesystems provided by LVM Storage, Local Storage Operator (LSO), and HostPath Provisioner (HPP) to provision local storage:

**Comparison of the support for storage types and filesystems**

<table>
<thead>
<tr>
  <th>Functionality</th>
  <th>LVM Storage</th>
  <th>LSO</th>
  <th>HPP</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Support for block storage</td>
  <td>Yes</td>
  <td>Yes</td>
  <td>No</td>
</tr>
<tr>
  <td>Support for file storage</td>
  <td>Yes</td>
  <td>Yes</td>
  <td>Yes</td>
</tr>
<tr>
  <td>Support for object storage <sup>[1]</sup></td>
  <td>No</td>
  <td>No</td>
  <td>No</td>
</tr>
<tr>
  <td>Available filesystems</td>
  <td><code>ext4</code>, <code>xfs</code></td>
  <td><code>ext4</code>, <code>xfs</code></td>
  <td>Any mounted system available on the node is supported.</td>
</tr>
</tbody>
</table>

1. None of the solutions (LVM Storage, LSO, and HPP) provide support for object storage. Therefore, if you want to use object storage, you need an S3 object storage solution, such as `MultiClusterGateway` from the Red Hat OpenShift Data Foundation. All of the solutions can serve as underlying storage providers for the S3 object storage solutions.

### Comparison of the support for core functionalities {#comparing-core-functionalities_ways-to-provision-local-storage}

The following table compares how LVM Storage, Local Storage Operator (LSO), and HostPath Provisioner (HPP) support core functionalities for provisioning local storage:

**Comparison of the support for core functionalities**

<table>
<thead>
<tr>
  <th>Functionality</th>
  <th>LVM Storage</th>
  <th>LSO</th>
  <th>HPP</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Support for automatic file system formatting</td>
  <td>Yes</td>
  <td>Yes</td>
  <td>N/A</td>
</tr>
<tr>
  <td>Support for dynamic provisioning</td>
  <td>Yes</td>
  <td>No</td>
  <td>No</td>
</tr>
<tr>
  <td>Support for using software Redundant Array of Independent Disks (RAID) arrays</td>
  <td>Yes<br><br>Supported on 4.15 and later.</td>
  <td>Yes</td>
  <td>Yes</td>
</tr>
<tr>
  <td>Support for transparent disk encryption</td>
  <td>Yes Supported on 4.16 and later.</td>
  <td>Yes</td>
  <td>Yes</td>
</tr>
<tr>
  <td>Support for volume based disk encryption</td>
  <td>No</td>
  <td>No</td>
  <td>No</td>
</tr>
<tr>
  <td>Support for disconnected installation</td>
  <td>Yes</td>
  <td>Yes</td>
  <td>Yes</td>
</tr>
<tr>
  <td>Support for PVC expansion</td>
  <td>Yes</td>
  <td>No</td>
  <td>No</td>
</tr>
<tr>
  <td>Support for volume snapshots and volume clones</td>
  <td>Yes</td>
  <td>No</td>
  <td>No</td>
</tr>
<tr>
  <td>Support for thin provisioning</td>
  <td>Yes Devices are thin-provisioned by default.</td>
  <td>Yes<br><br>You can configure the devices to point to the thin-provisioned volumes</td>
  <td>Yes<br><br>You can configure a path to point to the thin-provisioned volumes.</td>
</tr>
<tr>
  <td>Support for automatic disk discovery and setup</td>
  <td>Yes Automatic disk discovery is available during installation and runtime. You can also dynamically add the disks to the <code>LVMCluster</code> custom resource (CR) to increase the storage capacity of the existing storage classes.</td>
  <td>Technology Preview<br><br>Automatic disk discovery is available during installation.</td>
  <td>No</td>
</tr>
</tbody>
</table>

### Comparison of performance and isolation capabilities {#comparing-performance-and-isolation-boundary_ways-to-provision-local-storage}

The following table compares the performance and isolation capabilities of LVM Storage, Local Storage Operator (LSO), and HostPath Provisioner (HPP) in provisioning local storage.

**Comparison of performance and isolation capabilities**

<table>
<thead>
<tr>
  <th>Functionality</th>
  <th>LVM Storage</th>
  <th>LSO</th>
  <th>HPP</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Performance</td>
  <td>I/O speed is shared for all workloads that use the same storage class.<br><br>Block storage allows direct I/O operations.<br><br>Thin provisioning can affect the performance.</td>
  <td>I/O depends on the LSO configuration.<br><br>Block storage allows direct I/O operations.</td>
  <td>I/O speed is shared for all workloads that use the same storage class.<br><br>The restrictions imposed by the underlying filesystem can affect the I/O speed.</td>
</tr>
<tr>
  <td>Isolation boundary <sup>[1]</sup></td>
  <td>LVM Logical Volume (LV)<br><br>It provides higher level of isolation compared to HPP.</td>
  <td>LVM Logical Volume (LV)<br><br>It provides higher level of isolation compared to HPP</td>
  <td>Filesystem path<br><br>It provides lower level of isolation compared to LSO and LVM Storage.</td>
</tr>
</tbody>
</table>

1. Isolation boundary refers to the level of separation between different workloads or applications that use local storage resources.

### Comparison of the support for additional functionalities {#comparing-additional-functionalities_ways-to-provision-local-storage}

The following table compares the additional features provided by LVM Storage, Local Storage Operator (LSO), and HostPath Provisioner (HPP) to provision local storage:

**Comparison of the support for additional functionalities**

<table>
<thead>
<tr>
  <th>Functionality</th>
  <th>LVM Storage</th>
  <th>LSO</th>
  <th>HPP</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Support for generic ephemeral volumes</td>
  <td>Yes</td>
  <td>No</td>
  <td>No</td>
</tr>
<tr>
  <td>Support for CSI inline ephemeral volumes</td>
  <td>No</td>
  <td>No</td>
  <td>No</td>
</tr>
<tr>
  <td>Support for storage topology</td>
  <td>Yes Supports CSI node topology</td>
  <td>Yes<br><br>LSO provides partial support for storage topology through node tolerations.</td>
  <td>No</td>
</tr>
<tr>
  <td>Support for <code>ReadWriteMany</code> (RWX) access mode <sup>[1]</sup></td>
  <td>No</td>
  <td>No</td>
  <td>No</td>
</tr>
</tbody>
</table>

1. All of the solutions (LVM Storage, LSO, and HPP) have the `ReadWriteOnce` (RWO) access mode. RWO access mode allows access from multiple pods on the same node.
