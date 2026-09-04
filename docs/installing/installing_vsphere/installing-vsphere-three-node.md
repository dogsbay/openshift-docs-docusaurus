---
title: Installing a three-node cluster on vSphere
sidebar_position: 6
---

# Installing a three-node cluster on vSphere {#installing-vsphere-three-node}

<a id="installing-vsphere-three-node"></a>

In OpenShift Container Platform version 4.22, you can install a three-node cluster on VMware vSphere. A three-node cluster consists of three control plane machines, which also act as compute machines. This type of cluster provides a smaller, more resource efficient cluster, for cluster administrators and developers to use for testing, development, and production.

You can install a three-node cluster using either installer-provisioned or user-provisioned infrastructure.

## Configuring a three-node cluster {#installation-three-node-cluster_installing-vsphere-three-node}

To configure a three-node cluster, set the number of worker nodes to `0` in the `install-config.yaml` file before you deploy the cluster.

Setting the number of worker nodes to `0` ensures that the control plane machines are schedulable. This allows application workloads to be scheduled to run from the control plane nodes.

:::note

Because application workloads run from control plane nodes, additional subscriptions are required, as the control plane nodes are considered to be compute nodes.

:::

**Prerequisites**

- You have an existing `install-config.yaml` file.

**Procedure**

1. Set the number of compute replicas to `0` in your `install-config.yaml` file, as shown in the following `compute` stanza:

.Example `install-config.yaml` file for a three-node cluster

```yaml
apiVersion: v1
baseDomain: example.com
compute:
- name: worker
  platform: {}
  replicas: 0
# ...
```

1. If you are deploying a cluster with user-provisioned infrastructure:
   - Configure your application ingress load balancer to route HTTP and HTTPS traffic to the control plane nodes. In a three-node cluster, the Ingress Controller pods run on the control plane nodes. For more information, see the "Load balancing requirements for user-provisioned infrastructure".
   - After you create the Kubernetes manifest files, make sure that the `spec.mastersSchedulable` parameter is set to `true` in `cluster-scheduler-02-config.yml` file. You can locate this file in `<installation_directory>/manifests`. For more information, see "Creating the Kubernetes manifest and Ignition config files" in "Installing a cluster on vSphere with user-provisioned infrastructure".
   - Do not create additional worker nodes.

.Example `cluster-scheduler-02-config.yml` file for a three-node cluster

```yaml
apiVersion: config.openshift.io/v1
kind: Scheduler
metadata:
  creationTimestamp: null
  name: cluster
spec:
  mastersSchedulable: true
  policy:
    name: ""
status: {}
```

**Additional resources**

- [Installing a cluster on vSphere with customizations](/docs/installing/installing_vsphere/ipi/installing-vsphere-installer-provisioned-customizations#installing-vsphere-installer-provisioned-customizations)
- [Installing a cluster on vSphere with user-provisioned infrastructure](/docs/installing/installing_vsphere/upi/installing-vsphere#installing-vsphere)
