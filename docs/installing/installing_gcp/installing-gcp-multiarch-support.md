---
title: Installing a cluster with the support for configuring multi-architecture compute machines
sidebar_position: 15
---

# Installing a cluster with the support for configuring multi-architecture compute machines {#installing-gcp-multiarch-support}

<a id="installing-gcp-multiarch-support"></a>

You can install an OpenShift Container Platform cluster on Google Cloud with multi-architecture support to run workloads on compute machines with different CPU architectures.

:::note

When you have nodes with different architectures in your cluster, the architecture of your image must be consistent with the architecture of the node. You must verify that the pod is assigned to the node with the appropriate architecture and that it matches the image architecture. For more information about assigning pods to nodes, see "Scheduling workloads on clusters with multi-architecture compute machines".

:::

You can install a Google Cloud cluster with the support for configuring multi-architecture compute machines. After installing the Google Cloud cluster, you can add multi-architecture compute machines to the cluster in the following ways:

- Adding 64-bit x86 compute machines to a cluster that uses 64-bit ARM control plane machines and already includes 64-bit ARM compute machines. In this case, 64-bit x86 is considered the secondary architecture.
- Adding 64-bit ARM compute machines to a cluster that uses 64-bit x86 control plane machines and already includes 64-bit x86 compute machines. In this case, 64-bit ARM is considered the secondary architecture.

:::note

Before adding a secondary architecture node to your cluster, it is recommended to install the Multiarch Tuning Operator, and deploy a `ClusterPodPlacementConfig` custom resource. For more information, see "Managing workloads on multi-architecture clusters by using the Multiarch Tuning Operator".

:::

## Installing a cluster with multi-architecture support {#installing-a-cluster-with-multiarch-support_installing-gcp-multiarch-support}

You can install a cluster with multi-architecture support to use compute machines with different Central Processing Unit (CPU) architectures. Modifying your configuration file helps ensure your control plane and worker nodes deploy with the correct architecture.

**Prerequisites**

- You installed the OpenShift CLI (`oc`).
- You have the OpenShift Container Platform installation program.
- You downloaded the pull secret for your cluster.

**Procedure**

1. Check that the `openshift-install` binary is using the `multi` payload by running the following command:
   ```terminal
   $ ./openshift-install version
   ```

   ```terminal title="Example output"
   ./openshift-install 4.22.0
   built from commit abc123etc
   release image quay.io/openshift-release-dev/ocp-release@sha256:abc123wxyzetc
   release architecture multi
   default architecture amd64
   ```

   The output must contain `release architecture multi` to indicate that the `openshift-install` binary is using the `multi` payload.
2. Update the `install-config.yaml` file to configure the architecture for the nodes.
   ```yaml title="Sample install-config.yaml file with multi-architecture configuration"
   apiVersion: v1
   baseDomain: example.openshift.com
   compute:
   - architecture: amd64
     hyperthreading: Enabled
     name: worker
     platform: {}
     replicas: 3
   controlPlane:
     architecture: arm64
     name: master
     platform: {}
     replicas: 3
   # ...
   ```

   where:

   <dl>
   <dt><code>compute.architecture</code></dt>
   <dd>Specifies the architecture of the worker node. You can set this field to either <code>arm64</code> or <code>amd64</code>.</dd>
   <dt><code>controlPlane.architecture</code></dt>
   <dd>Specifies the control plane node architecture. You can set this field to either <code>arm64</code> or <code>amd64</code>.</dd>
   </dl>

**Additional resources**

- [Deploying the cluster](/docs/installing/installing_gcp/installing-gcp-customizations#installation-launching-installer_installing-gcp-customizations)
- [Scheduling workloads on clusters with multi-architecture compute machines](/docs/post_installation_configuration/configuring-multi-arch-compute-machines/multi-architecture-compute-managing#scheduling-workloads-on-clusters-with-multi-architecture-compute-machines)
- [Managing workloads on multi-architecture clusters by using the Multiarch Tuning Operator](/docs/post_installation_configuration/configuring-multi-arch-compute-machines/multiarch-tuning-operator#multiarch-tuning-operator)
