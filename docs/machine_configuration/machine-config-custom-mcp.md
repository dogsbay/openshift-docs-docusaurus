---
title: Creating custom machine config pools
sidebar_position: 9
---

# Creating custom machine config pools {#machine-config-creating-custom-mcp}

<a id="machine-config-creating-custom-mcp"></a>

You can create custom machine config pools (MCP) to manage compute nodes for custom use cases that extend outside of the default node types. By using a custom machine config pool, you can deploy changes targeted only at nodes in the custom pool.

Custom machine config pools inherit their configurations from the `worker` machine config pool. Changes made to the `worker` machine config pool apply to nodes in the custom pool. However, changes made to the custom machine config pool apply only to the nodes in the custom pool. For more information on custom machine config pools, see "Node configuration management with machine config pools".

:::note

Custom machine config pools for the control plane nodes are not supported.

:::

For example, you could use a custom machine config pool to create an *infrastructure* node. Components that you move to an infrastructure node do not need to be accounted for during sizing. For more information on infrastructure nodes, see "Creating infrastructure machine sets".

After you create the custom machine config pool, you can boot new nodes directly to the pool by creating a new machine set. Or, you can add existing nodes to the custom pool by using labels.

## Create a custom machine config pool with a new node {#machine-config-custom-mcp-automatic_machine-config-creating-custom-mcp}

You can create a custom machine config pool (MCP) and launch a new node directly into that pool. By launching the node directly into the new pool, you save a node reboot cycle that would be required when moving the nodes from the worker machine config pool to the custom pool.

Use the `userDataSecret` parameter in the machine set to instruct the Machine Config Operator (MCO) to add the node to a specific machine config pool. The secret contains the endpoint of the custom machine config pool. You must prefix the name of this new secret with the name of the custom machine config pool.

The following procedure shows you how to create a new custom machine config pool and launch a new node into that pool.

**Procedure**

1. Create a custom machine config pool:
   1. Create a YAML file similar to the following:
      ```yaml
      apiVersion: machineconfiguration.openshift.io/v1
      kind: MachineConfigPool
      metadata:
        name: custom
      spec:
        machineConfigSelector:
          matchExpressions:
            - {key: machineconfiguration.openshift.io/role, operator: In, values: [custom,worker]}
        nodeSelector:
          matchLabels:
            node-role.kubernetes.io/custom: ""
      ```

      where:

      <dl>
      <dt><code>metadata.name</code></dt>
      <dd>Specifies a name for the custom machine config pool.</dd>
      <dt><code>spec.machineConfigSelector.matchExpressions</code></dt>
      <dd>Specifies the node roles for the new node. This must include the <code>worker</code> role and the custom role.</dd>
      <dt><code>spec.nodeSelector.matchLabels</code></dt>
      <dd>Specifies a node selector to use when adding nodes to this pool.</dd>
      </dl>
   2. Create the machine config pool by running the following command:
      ```terminal
      $ oc create -f <file_name>.yaml
      ```
2. Create a new machine set that creates a new node in the new custom machine config pool:
   1. Create a YAML file, for example by making a copy of an existing compute machine set YAML and making the following changes:
      ```yaml
      apiVersion: machine.openshift.io/v1beta1
      kind: MachineSet
      metadata:
      # ...
        name: <machineset_name>
        namespace: openshift-machine-api
      # ...
      spec:
      # ...
        template:
      # ...
          spec:
      # ...
            providerSpec:
      # ...
              value:
      # ...
                userDataSecret:
                  name: <mcp_name>-user-data-managed
      # ...
      ```

      where:

      <dl>
      <dt><code>metadata.name</code></dt>
      <dd>Specifies a name for the machine set.</dd>
      <dt><code>metadata.namespace</code></dt>
      <dd>Specifies a namespace for the machine set. This must be <code>openshift-machine-api</code>.</dd>
      <dt><code>spec.template.spec.providerSpec.value.userDataSecret</code></dt>
      <dd>Specifies a name for the user data secret that is created. The name of the secret must start with the name of the custom machine config pool and end with <code>-user-data-managed</code>. For example <code>custom-user-data-managed</code>.</dd>
      </dl>

      These are the minimum changes required to create the new machine set. For more configuration options or to configure an all-new machine set, see "Creating infrastructure machine sets" for your platform.

      :::note

      When creating a new machine set, you should specify the latest image to use for the boot image. For more information about configuring the boot image on your cluster, see "Manually updating the boot image" for your platform. The method to specify the image varies by provider.

      :::
   2. Create the machine set by running the following command:
      ```terminal
      $ oc create -f <file_name>.yaml
      ```

      The MCO creates a new node in the new custom machine config pool.

**Verification**

1. Check to see that the MCO created the new machine config pool by running the following command:
   ```terminal
   $ oc get mcp
   ```

   ```terminal title="Example output"
   NAME      CONFIG                                              UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT   AGE
   custom    rendered-custom-72be15c95699b6d39f70fce525f51bb2    True      False      False      1              1                   1                     0                      12s
   master    rendered-master-9e25b616b551d6c77f490191f45161d7    True      False      False      3              3                   3                     0                      32m
   worker    rendered-worker-72be15c95699b6d39f70fce525f51bb2    True      False      False      2              2                   2                     0                      32m
   ```

   In this example, `custom` is the new machine config pool.
2. Check to see that the MCO created the new machine set by running the following command:
   ```terminal
   $ oc get machineset -n openshift-machine-api
   ```

   ```terminal title="Example output"
   NAME                                  DESIRED   CURRENT   READY   AVAILABLE   AGE
   ci-ln-7x179fk-72292-tc5qz-custom-a    1         1         1       1           62s
   ci-ln-7x179fk-72292-tc5qz-worker-a    1         1         1       1           91m
   ci-ln-7x179fk-72292-tc5qz-worker-b    1         1         1       1           91m
   ci-ln-7x179fk-72292-tc5qz-worker-f    1         1         1       1           91m
   ```

   In this example, `ci-ln-7x179fk-72292-tc5qz-custom-a` is the new machine set.
3. Check that the MCO created the required secret by running the following command:
   ```terminal
   $ oc get secrets -n openshift-machine-api
   ```

   ```terminal title="Example output"
   NAME                                                  TYPE                      DATA   AGE
   # ...
   custom-user-data-managed                              Opaque                    2      9m
   # ...
   ```
4. Check to see that the node is in the new custom machine config pool by running the following command:
   ```terminal
   $ oc get nodes
   ```

   ```terminal title="Example output"
   NAME                                        STATUS   ROLES                    AGE   VERSION
   ci-ln-i61xqwb-72292-hz2mw-custom-9r496      Ready    custom,worker             9m   v1.35.3
   ci-ln-i61xqwb-72292-ftjn8-master-0          Ready    control-plane,master     42m   v1.35.3
   ci-ln-i61xqwb-72292-ftjn8-master-1          Ready    control-plane,master     44m   v1.35.3
   ci-ln-i61xqwb-72292-ftjn8-master-2          Ready    control-plane,master     43m   v1.35.3
   ci-ln-i61xqwb-72292-ftjn8-worker-c-2lhcl    Ready    worker                   36m   v1.35.3
   ci-ln-i61xqwb-72292-ftjn8-worker-f-qgdb7    Ready    worker                   36m   v1.35.3
   ```

   In this example, the `ci-ln-i61xqwb-72292--hz2mw-custom-9r496` is a new node that was added to the `custom` machine config pool.

## Create a custom machine config pool for an existing node {#machine-config-custom-mcp-existing_machine-config-creating-custom-mcp}

You can create custom machine config pools (MCP) and manually add an existing node into that pool. With custom machine config pools, you can deploy changes targeted at the nodes in the custom pool.

The following procedure shows you how to create a new custom machine config pool and add an existing node into that pool.

**Procedure**

1. Create a custom machine config pool:
   1. Create a YAML file similar to the following:
      ```yaml
      apiVersion: machineconfiguration.openshift.io/v1
      kind: MachineConfigPool
      metadata:
        name: custom
      spec:
        machineConfigSelector:
          matchExpressions:
            - {key: machineconfiguration.openshift.io/role, operator: In, values: [custom,worker]}
        nodeSelector:
          matchLabels:
            node-role.kubernetes.io/custom: ""
      ```

      where:

      <dl>
      <dt><code>metadata.name</code></dt>
      <dd>Specifies a name for the machine config pool.</dd>
      <dt><code>spec.machineConfigSelector.matchExpressions</code></dt>
      <dd>Specifies the node roles for the new node. This must include the <code>worker</code> role and the custom role.</dd>
      <dt><code>spec.nodeSelector.matchLabels</code></dt>
      <dd>Specifies a node selector to use when adding nodes to this pool.</dd>
      </dl>
   2. Create the machine config pool by running the following command:
      ```terminal
      $ oc create -f <file_name>.yaml
      ```
2. Add a label to the worker nodes that you want to move to the new custom pool by running the following command:
   ```terminal
   $ oc label node <node_name> <node_selector>
   ```

   Replace `<node_name>` with the name of the node that you want to move and replace `<node-selector>` with the node selector you added to the machine config pool.

   ```terminal title="Example command"
   $ oc label node ci-ln-g5tpp5k-72292-hz2mw-worker-b-ps8xh node-role.kubernetes.io/custom=""
   ```

**Verification**

1. Check to see that the MCO created the new machine config pool by running the following command:
   ```terminal
   $ oc get mcp
   ```

   ```terminal title="Example output"
   NAME      CONFIG                                              UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT   AGE
   custom    rendered-custom-72be15c95699b6d39f70fce525f51bb2    True      False      False      1              1                   1                     0                      12s
   master    rendered-master-9e25b616b551d6c77f490191f45161d7    True      False      False      3              3                   3                     0                      32m
   worker    rendered-worker-72be15c95699b6d39f70fce525f51bb2    True      False      False      2              2                   2                     0                      32m
   ```

   In this example, `custom` is the new machine config pool.
2. Check to see if the node is in that pool by running the following command:
   ```terminal
   $ oc get nodes
   ```

   ```terminal title="Example output"
   NAME                                       STATUS   ROLES                  AGE   VERSION
   ci-ln-i61xqwb-72292-ftjn8-master-0         Ready    control-plane,master   42m   v1.35.3
   ci-ln-i61xqwb-72292-ftjn8-master-1         Ready    control-plane,master   44m   v1.35.3
   ci-ln-i61xqwb-72292-ftjn8-master-2         Ready    control-plane,master   43m   v1.35.3
   ci-ln-g5tpp5k-72292-hz2mw-worker-b-ps8xh   Ready    custom,worker          36m   v1.35.3
   ci-ln-i61xqwb-72292-ftjn8-worker-c-2lhcl   Ready    worker                 36m   v1.35.3
   ci-ln-i61xqwb-72292-ftjn8-worker-f-qgdb7   Ready    worker                 36m   v1.35.3
   ```

   In this example, the `ci-ln-g5tpp5k-72292-hz2mw-worker-b-ps8xh` node is an existing node that was moved to the `custom` machine config pool.

**Additional resources**

- [Node configuration management with machine config pools](/docs/machine_configuration#architecture-machine-config-pools_machine-config-overview)
- [Manually updating the boot image](/docs/machine_configuration/mco-update-boot-images-manual#mco-update-boot-images-manual)
- [Creating infrastructure machine sets](/docs/machine_management/creating-infrastructure-machinesets#creating-infrastructure-machinesets)
