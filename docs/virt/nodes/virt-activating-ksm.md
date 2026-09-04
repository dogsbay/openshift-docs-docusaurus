---
title: Activating kernel samepage merging (KSM)
sidebar_position: 5
---

# Activating kernel samepage merging (KSM) {#virt-activating-ksm}

<a id="virt-activating-ksm"></a>

OpenShift Virtualization can activate kernel samepage merging (KSM) when nodes are overloaded. KSM deduplicates identical data found in the memory pages of virtual machines (VMs). If you have very similar VMs, KSM can make it possible to schedule more VMs on a single node.

:::warning

You must only use KSM with trusted workloads.

:::

## Prerequisites {#prerequisites_virt-activating-ksm}

- Ensure that an administrator has configured KSM support on any nodes where you want OpenShift Virtualization to activate KSM.

## About using OpenShift Virtualization to activate KSM {#virt-about-ksm_virt-activating-ksm}

You can configure OpenShift Virtualization to activate kernel samepage merging (KSM) when nodes experience memory overload.

### Configuration methods {#virt-ksm-configuration-methods}

You can enable or disable the KSM activation feature for all nodes by using the OpenShift Container Platform web console or by editing the `HyperConverged` custom resource (CR). The `HyperConverged` CR supports more granular configuration.

CR configuration
: <a name="virt-ksm-cr-configuration"></a>

  You can configure the KSM activation feature by editing the `spec.ksmConfiguration` stanza of the `HyperConverged` CR.

  - You enable the feature and configure settings by editing the `ksmConfiguration` stanza.
  - You disable the feature by deleting the `ksmConfiguration` stanza.
  - You can allow OpenShift Virtualization to enable KSM on only a subset of nodes by adding node selection syntax to the `ksmConfiguration.nodeLabelSelector` field.

  :::note

  Even if the KSM activation feature is disabled in OpenShift Virtualization, an administrator can still enable KSM on nodes that support it.

  :::

### KSM node labels {#virt-ksm-node-labels}

OpenShift Virtualization identifies nodes that are configured to support KSM and applies the following node labels:

`kubevirt.io/ksm-handler-managed: "false"`
: This label is set to `"true"` when OpenShift Virtualization activates KSM on a node that is experiencing memory overload. This label is not set to `"true"` if an administrator activates KSM.

`kubevirt.io/ksm-enabled: "false"`
: This label is set to `"true"` when KSM is activated on a node, even if OpenShift Virtualization did not activate KSM.

These labels are not applied to nodes that do not support KSM.

## Configuring KSM activation by using the web console {#virt-configure-ksm-web_virt-activating-ksm}

You can allow OpenShift Virtualization to activate kernel samepage merging (KSM) on all nodes in your cluster by using the OpenShift Container Platform web console.

**Procedure**

1. From the side menu, click **Virtualization** → **Settings**.
2. Select the **Cluster** tab.
3. Expand **Resource management**.
4. Enable or disable the feature for all nodes using the **Kernel Samepage Merging (KSM)** toggle button.

## Configuring KSM activation by using the CLI {#virt-configure-ksm-cli_virt-activating-ksm}

You can enable or disable OpenShift Virtualization's kernel samepage merging (KSM) activation feature by editing the `HyperConverged` custom resource (CR). Use this method if you want OpenShift Virtualization to activate KSM on only a subset of nodes.

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).

**Procedure**

1. Open the `HyperConverged` CR in your default editor by running the following command:
   ```terminal
   $ oc edit hyperconvergeds.v1beta1.hco.kubevirt.io kubevirt-hyperconverged -n openshift-cnv
   ```
2. Edit the `ksmConfiguration` stanza:
   - To enable the KSM activation feature for all nodes, set the `nodeLabelSelector` value to `{}`. For example:
     ```yaml
     apiVersion: hco.kubevirt.io/v1beta1
     kind: HyperConverged
     metadata:
       name: kubevirt-hyperconverged
       namespace: openshift-cnv
     spec:
       ksmConfiguration:
         nodeLabelSelector: {}
     # ...
     ```
   - To enable the KSM activation feature on a subset of nodes, edit the `nodeLabelSelector` field. Add syntax that matches the nodes where you want OpenShift Virtualization to enable KSM. For example, the following configuration allows OpenShift Virtualization to enable KSM on nodes where both `<first_example_key>` and `<second_example_key>` are set to `"true"`:
     ```yaml
     apiVersion: hco.kubevirt.io/v1beta1
     kind: HyperConverged
     metadata:
       name: kubevirt-hyperconverged
       namespace: openshift-cnv
     spec:
       ksmConfiguration:
         nodeLabelSelector:
           matchLabels:
             <first_example_key>: "true"
             <second_example_key>: "true"
     # ...
     ```
   - To disable the KSM activation feature, delete the `ksmConfiguration` stanza. For example:
     ```yaml
     apiVersion: hco.kubevirt.io/v1beta1
     kind: HyperConverged
     metadata:
       name: kubevirt-hyperconverged
       namespace: openshift-cnv
     spec:
     # ...
     ```
3. Save the file.

**Additional resources**

- [Specifying nodes for virtual machines](/docs/virt/managing_vms/advanced_vm_management/virt-specifying-nodes-for-vms#virt-specifying-nodes-for-vms)
- [Placing pods on specific nodes using node selectors](/docs/nodes/scheduling/nodes-scheduler-node-selectors#nodes-scheduler-node-selectors)
- [Managing kernel samepage merging](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html-single/configuring_and_managing_virtualization/index#proc_managing-ksm_optimizing-virtual-machine-cpu-performance)
