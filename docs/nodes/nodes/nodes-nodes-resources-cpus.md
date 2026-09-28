---
title: Allocating specific CPUs for nodes in a cluster
sidebar_position: 13
---

# Allocating specific CPUs for nodes in a cluster {#nodes-nodes-resources-cpus}

<a id="nodes-nodes-resources-cpus"></a>

When using the static CPU Manager policy, you can explicitly define a list of CPUs that are reserved for critical system processes on specific nodes. Reserving CPUs for critical system processes can help ensure cluster stability.

For example, on a system with 24 CPUs, you could reserve CPUs numbered 0 - 3 for the control plane allowing the compute nodes to use CPUs 4 - 23.

## Reserve CPUs for nodes {#nodes-nodes-resources-cpus-reserve_nodes-nodes-resources-cpus}

You can explicitly define a list of CPUs that are reserved for critical system processes on specific nodes by creating a `KubeletConfig` custom resource (CR) to define the `reservedSystemCPUs` parameter. Reserving CPUs for critical system processes can help ensure cluster stability.

This list supersedes the CPUs that might be reserved by using the `systemReserved` parameter.

For more information on the `systemReserved` parameter, see "Allocating resources for nodes in an OpenShift Container Platform cluster".

**Prerequisites**

1. You have the label associated with the machine config pool (MCP) for the type of node you want to configure:

**Procedure**

1. Create a YAML file for the `KubeletConfig` CR:
   ```yaml
   apiVersion: machineconfiguration.openshift.io/v1
   kind: KubeletConfig
   metadata:
     name: set-reserved-cpus
   spec:
     kubeletConfig:
       reservedSystemCPUs: "0,1,2,3"
     machineConfigPoolSelector:
       matchLabels:
         pools.operator.machineconfiguration.openshift.io/worker: ""
   #...
   ```

   where:

   <dl>
   <dt><code>metadata.name</code></dt>
   <dd>Specifies a name for the CR.</dd>
   <dt><code>spec.kubeletConfig.reservedSystemCPUs</code></dt>
   <dd>Specifies the core IDs of the CPUs you want to reserve for the nodes associated with the MCP.</dd>
   <dt><code>spec.machineConfigPoolSelector.matchLabels</code></dt>
   <dd>Specifies the label from the MCP.</dd>
   </dl>
2. Create the CR object:
   ```terminal
   $ oc create -f <file_name>.yaml
   ```

**Additional resources**

- [Setting up CPU Manager](/docs/scalability_and_performance/using-cpu-manager#setting_up_cpu_manager_using-cpu-manager-and-topology-manager)
- [Allocating resources for nodes in an OpenShift Container Platform cluster](/docs/nodes/nodes/nodes-nodes-resources-configuring#nodes-nodes-resources-configuring-about_nodes-nodes-resources-configuring)
