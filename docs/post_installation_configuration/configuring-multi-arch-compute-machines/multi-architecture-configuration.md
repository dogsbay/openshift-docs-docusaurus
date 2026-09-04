---
title: About clusters with multi-architecture compute machines
sidebar_position: 1
---

# About clusters with multi-architecture compute machines {#post-install-multi-architecture-configuration}

<a id="post-install-multi-architecture-configuration"></a>

An OpenShift Container Platform cluster with multi-architecture compute machines is a cluster that supports compute machines with different architectures.

Configuring multi-architecture compute machines involves some additional considerations:

- When there are nodes with multiple architectures in your cluster, the architecture of the container image that you deploy to a node must be consistent with the architecture of that node. You need to ensure that the pod is assigned to the node with the appropriate architecture and that it matches the container image architecture. For more information on assigning pods to nodes, see "Assigning pods to nodes".
- In installer-provisioned installations, you are restricted to using the infrastructure provided by a single cloud provider. Adding external nodes, regardless of their architecture, to these clusters is not supported.
- Clusters that are installed with the platform type `none` are unable to use some features, such as managing compute machines with the Machine API. This limitation applies even if the compute machines that are attached to the cluster are installed on a platform that would normally support the feature. This parameter cannot be changed after installation.
  :::warning

  See "Deploying OpenShift 4.x on non-tested platforms using the bare metal install method" before you attempt to install an OpenShift Container Platform cluster in virtualized or cloud environments.

  :::
- The Cluster Samples Operator is not supported on clusters with multi-architecture compute machines. Your cluster can be created without this capability. For more information, see "Cluster capabilities".
- For information on migrating your single-architecture cluster to a cluster that supports multi-architecture compute machines, see "Migrating to a cluster with multi-architecture compute machines".

## Configuring your cluster with multi-architecture compute machines {#multi-architecture-configuring-your-cluster_multi-architecture-configuration}

To create a cluster with multi-architecture compute machines with different installation options and platforms, see the documentation references.

**Cluster with multi-architecture compute machine installation options**

<table>
<thead>
<tr>
  <th>Documentation section</th>
  <th>Platform</th>
  <th>User-provisioned installation</th>
  <th>Installer-provisioned installation</th>
  <th>Control Plane</th>
  <th>Compute node</th>
</tr>
</thead>
<tbody>
<tr>
  <td>"Creating a cluster with multi-architecture compute machines on Azure"</td>
  <td>Microsoft Azure</td>
  <td>&#10003;</td>
  <td>&#10003;</td>
  <td><code>aarch64</code> or <code>x86_64</code></td>
  <td><code>aarch64</code>, <code>x86_64</code></td>
</tr>
<tr>
  <td>"Creating a cluster with multi-architecture compute machines on AWS"</td>
  <td>Amazon Web Services (AWS)</td>
  <td>&#10003;</td>
  <td>&#10003;</td>
  <td><code>aarch64</code> or <code>x86_64</code></td>
  <td><code>aarch64</code>, <code>x86_64</code></td>
</tr>
<tr>
  <td>"Creating a cluster with multi-architecture compute machines on Google Cloud"</td>
  <td>Google Cloud</td>
  <td></td>
  <td>&#10003;</td>
  <td><code>aarch64</code> or <code>x86_64</code></td>
  <td><code>aarch64</code>, <code>x86_64</code></td>
</tr>
<tr>
  <td rowspan="3">"Creating a cluster with multi-architecture compute machines on bare metal, IBM Power, or IBM Z"</td>
  <td>Bare metal</td>
  <td>&#10003;</td>
  <td>&#10003;</td>
  <td><code>aarch64</code> or <code>x86_64</code></td>
  <td><code>aarch64</code>, <code>x86_64</code></td>
</tr>
<tr>
  <td>IBM Power</td>
  <td>&#10003;</td>
  <td></td>
  <td><code>x86_64</code> or <code>ppc64le</code></td>
  <td><code>x86_64</code>, <code>ppc64le</code></td>
</tr>
<tr>
  <td>IBM Z</td>
  <td>&#10003;</td>
  <td></td>
  <td><code>x86_64</code> or <code>s390x</code></td>
  <td><code>x86_64</code>, <code>s390x</code></td>
</tr>
<tr>
  <td>"Creating a cluster with multi-architecture compute machines on IBM Z(R) and IBM(R) LinuxONE with z/VM"</td>
  <td>IBM Z(R) and IBM(R) LinuxONE</td>
  <td>&#10003;</td>
  <td></td>
  <td><code>x86_64</code>, <code>s390x</code></td>
  <td><code>x86_64</code>, <code>s390x</code></td>
</tr>
<tr>
  <td>"Creating a cluster with multi-architecture compute machines on IBM Z(R) and IBM(R) LinuxONE with RHEL KVM"</td>
  <td>IBM Z(R) and IBM(R) LinuxONE</td>
  <td>&#10003;</td>
  <td></td>
  <td><code>x86_64</code>, <code>s390x</code></td>
  <td><code>x86_64</code>, <code>s390x</code></td>
</tr>
<tr>
  <td>"Creating a cluster with multi-architecture compute machines on IBM Power(R)"</td>
  <td>IBM Power(R)</td>
  <td>&#10003;</td>
  <td></td>
  <td><code>x86_64</code></td>
  <td><code>x86_64</code>, <code>ppc64le</code></td>
</tr>
</tbody>
</table>

**Additional resources**

- [Creating a cluster with multi-architecture compute machines on Azure](/docs/post_installation_configuration/configuring-multi-arch-compute-machines/creating-multi-arch-compute-nodes-azure#creating-multi-arch-compute-nodes-azure)
- [Creating a cluster with multi-architecture compute machines on AWS](/docs/post_installation_configuration/configuring-multi-arch-compute-machines/creating-multi-arch-compute-nodes-aws#creating-multi-arch-compute-nodes-aws)
- [Creating a cluster with multi-architecture compute machines on Google Cloud](/docs/post_installation_configuration/configuring-multi-arch-compute-machines/creating-multi-arch-compute-nodes-google-cloud#creating-multi-arch-compute-nodes-google-cloud)
- [Creating a cluster with multi-architecture compute machines on bare metal, IBM Power, or IBM Z](/docs/post_installation_configuration/configuring-multi-arch-compute-machines/creating-multi-arch-compute-nodes-bare-metal#creating-multi-arch-compute-nodes-bare-metal)
- [Creating a cluster with multi-architecture compute machines on IBM Z(R) and IBM(R) LinuxONE with z/VM](/docs/post_installation_configuration/configuring-multi-arch-compute-machines/creating-multi-arch-compute-nodes-ibm-z#creating-multi-arch-compute-nodes-ibm-z)
- [Creating a cluster with multi-architecture compute machines on IBM Z(R) and IBM(R) LinuxONE with RHEL KVM](/docs/post_installation_configuration/configuring-multi-arch-compute-machines/creating-multi-arch-compute-nodes-ibm-z-kvm#creating-multi-arch-compute-nodes-ibm-z-kvm)
- [Creating a cluster with multi-architecture compute machines on IBM Power(R)](/docs/post_installation_configuration/configuring-multi-arch-compute-machines/creating-multi-arch-compute-nodes-ibm-power#creating-multi-arch-compute-nodes-ibm-power)

## Verifying cluster compatibility {#multi-architecture-verifying-cluster-compatibility_multi-architecture-configuration}

Before you can start adding compute nodes of different architectures to your cluster, you must verify that your cluster is multi-architecture compatible.

**Prerequisites**

- You installed the OpenShift CLI (`oc`).
- IBM Power only: Ensure that you meet the following prerequisites:
  - When using multiple architectures, hosts for OpenShift Container Platform nodes must share the same storage layer. If they do not have the same storage layer, use a storage provider such as `nfs-provisioner`.
  - You should limit the number of network hops between the compute and control plane as much as possible.

**Procedure**

1. Log in to the OpenShift CLI (`oc`).
2. You can check that your cluster uses the architecture payload by running the following command:
   ```terminal
   $ oc adm release info -o jsonpath="{ .metadata.metadata}"
   ```

**Verification**

- If you see the following output, your cluster is using the multi-architecture payload:
  ```terminal
  {
   "release.openshift.io/architecture": "multi",
   "url": "https://access.redhat.com/errata/<errata_version>"
  }
  ```

  You can then begin adding multi-arch compute nodes to your cluster.
- If you see the following output, your cluster is not using the multi-architecture payload:
  ```terminal
  {
   "url": "https://access.redhat.com/errata/<errata_version>"
  }
  ```

  :::warning

  To migrate your cluster so the cluster supports multi-architecture compute machines, see "Migrating to a cluster with multi-architecture compute machines".

  :::

**Additional resources**

- [Migrating to a cluster with multi-architecture compute machines](/docs/updating/updating_a_cluster/migrating-to-multi-payload#migrating-to-multi-payload)

**Additional resources**

- [Assigning pods to nodes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)
- [Deploying OpenShift 4.x on non-tested platforms using the bare metal install method (Red Hat Knowledgebase article)](https://access.redhat.com/articles/4207611)
- [Cluster capabilities](/docs/installing/overview/cluster-capabilities#cluster-capabilities)
- [Migrating to a cluster with multi-architecture compute machines](/docs/updating/updating_a_cluster/migrating-to-multi-payload#migrating-to-multi-payload)
