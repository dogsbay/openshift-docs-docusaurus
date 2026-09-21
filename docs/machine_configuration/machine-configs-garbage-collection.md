---
title: Managing unused rendered machine configs
sidebar_position: 10
---

# Managing unused rendered machine configs {#machine-configs-garbage-collection}

<a id="machine-configs-garbage-collection"></a>

You can remove old, unused rendered machine configs by using the `oc adm prune renderedmachineconfigs` command. By removing these objects, you can free up disk space and reduce performance issues.

The Machine Config Operator (MCO) does not perform any garbage collection activities. This means that all rendered machine configs remain in the cluster. Each time a user or controller applies a new machine config, the MCO creates new rendered configs for each affected machine config pool. Over time, this can lead to a large number of rendered machine configs, which can make working with machine configs confusing. Having too many rendered machine configs can also contribute to disk space issues and performance issues with etcd.

By using the `oc adm prune renderedmachineconfigs` command with the `--confirm` flag, you can remove all unused rendered machine configs or only those in a specific machine config pool. You can also remove a specific number of unused rendered machine configs, keeping some older machine configs in case you want to check older configurations.

You can use the `oc adm prune renderedmachineconfigs` command without the `--confirm` flag to see which rendered machine configs would be removed.

Use the `list` subcommand to display all the rendered machine configs in the cluster or a specific machine config pool.

:::note

The `oc adm prune renderedmachineconfigs` command deletes only rendered machine configs that are not in use. If a rendered machine configs are in use by a machine config pool, the rendered machine config is not deleted. In this case, the command output specifies the reason that the rendered machine config was not deleted.

:::

## View rendered machine configs {#machineconfig-garbage-collect-viewing_machine-configs-garbage-collection}

You can view a list of rendered machine configs by using the `oc adm prune renderedmachineconfigs` command with the `list` subcommand to determine which objects you can remove.

For example, the command in the following procedure would list all rendered machine configs for the `worker` machine config pool.

**Procedure**

- Optional: List the rendered machine configs by using the following command:
  ```terminal
  $ oc adm prune renderedmachineconfigs list --in-use=false --pool-name=worker
  ```

  where:

  <dl>
  <dt>list</dt>
  <dd>Displays a list of rendered machine configs in your cluster.</dd>
  <dt><code>--in-use</code></dt>
  <dd>Optional: Specifies whether to display only the used machine configs or all machine configs from the specified pool. If <code>true</code>, the output lists the rendered machine configs that are being used by a machine config pool. If <code>false</code>, the output lists all rendered machine configs in the cluster. The default value is <code>false</code>.</dd>
  <dt><code>--pool-name</code></dt>
  <dd>Optional: Specifies the machine config pool from which to display the machine configs.</dd>
  </dl>

  ```terminal title="Example output"
  worker

  rendered-worker-f38bf61ced3c920cf5a29a200ed43243 -- 2025-01-21 13:45:01 +0000 UTC (Currently in use: false)
  rendered-worker-fc94397dc7c43808c7014683c208956e-- 2025-01-30 17:20:53 +0000 UTC (Currently in use: false)
  rendered-worker-708c652868f7597eaa1e2622edc366ef -- 2025-01-31 18:01:16 +0000 UTC (Currently in use: true)
  ```
- List the rendered machine configs that you can remove automatically by running the following command. Any rendered machine config marked with the `as it's currently in use` message in the command output cannot be removed.
  ```terminal
  $ oc adm prune renderedmachineconfigs --pool-name=worker
  ```

  The command runs in dry-run mode, and no machine configs are removed. where:

  <dl>
  <dt><code>--pool-name</code></dt>
  <dd>Optional: Displays the machine configs in the specified machine config pool.</dd>
  </dl>

  ```terminal title="Example output"
  Dry run enabled - no modifications will be made. Add --confirm to remove rendered machine configs.
  dry-run deleting rendered MachineConfig rendered-worker-f38bf61ced3c920cf5a29a200ed43243
  dry-run deleting MachineConfig rendered-worker-fc94397dc7c43808c7014683c208956e
  Skip dry-run deleting rendered MachineConfig rendered-worker-708c652868f7597eaa1e2622edc366ef as it's currently in use
  ```

## Remove unused rendered machine configs {#machineconfig-garbage-collect-removing_machine-configs-garbage-collection}

You can remove unused rendered machine configs by using the `oc adm prune renderedmachineconfigs` command with the `--confirm` command, reducing disk space and performance issues.

If any rendered machine config is not deleted, the command output indicates which was not deleted and lists the reason for skipping the deletion.

**Procedure**

1. Optional: List the rendered machine configs that you can remove automatically by running the following command. Any rendered machine config marked with the `as it's currently in use` message in the command output cannot be removed.
   ```terminal
   $ oc adm prune renderedmachineconfigs --pool-name=worker
   ```

   ```terminal title="Example output"
   Dry run enabled - no modifications will be made. Add --confirm to remove rendered machine configs.
   dry-run deleting rendered MachineConfig rendered-worker-f38bf61ced3c920cf5a29a200ed43243
   dry-run deleting MachineConfig rendered-worker-fc94397dc7c43808c7014683c208956e
   Skip dry-run deleting rendered MachineConfig rendered-worker-708c652868f7597eaa1e2622edc366ef as it's currently in use
   ```

   where:

   <dl>
   <dt>pool-name</dt>
   <dd>Optional: Specifies the machine config pool where you want to delete the machine configs from.</dd>
   </dl>
2. Remove the unused rendered machine configs by running the following command. The command in the following procedure would delete the two oldest unused rendered machine configs in the `worker` machine config pool.
   ```terminal
   $ oc adm prune renderedmachineconfigs --pool-name=worker --count=2 --confirm
   ```

   where:

   <dl>
   <dt><code>--count</code></dt>
   <dd>Optional: Specifies the maximum number of unused rendered machine configs you want to delete, starting with the oldest.</dd>
   <dt><code>--confirm</code></dt>
   <dd>Indicates that pruning should occur, instead of performing a dry-run.</dd>
   <dt><code>--pool-name</code></dt>
   <dd>Optional: Specifies the machine config pool from which you want to delete the machine. If not specified, all the pools are evaluated.</dd>
   </dl>

   ```terminal title="Example output"
   deleting rendered MachineConfig rendered-worker-f38bf61ced3c920cf5a29a200ed43243
   deleting rendered MachineConfig rendered-worker-fc94397dc7c43808c7014683c208956e
   Skip deleting rendered MachineConfig rendered-worker-708c652868f7597eaa1e2622edc366ef as it's currently in use
   ```
