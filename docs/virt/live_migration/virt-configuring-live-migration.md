---
title: Configuring live migration
sidebar_position: 2
---

# Configuring live migration {#virt-configuring-live-migration}

<a id="virt-configuring-live-migration"></a>

You can configure live migration settings to ensure that the migration processes do not overwhelm the cluster. You can configure live migration policies to apply different migration configurations to groups of virtual machines (VMs).

## Configuring live migration limits and timeouts {#virt-configuring-live-migration-limits_virt-configuring-live-migration}

Configure live migration limits and timeouts for the cluster by updating the `HyperConverged` custom resource (CR), which is located in the `openshift-cnv` namespace.

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).

**Procedure**

- Edit the `HyperConverged` CR and add the necessary live migration parameters:
  ```terminal
  $ oc edit hyperconvergeds.v1beta1.hco.kubevirt.io kubevirt-hyperconverged -n openshift-cnv
  ```

  Example configuration file:

  ```yaml
  apiVersion: hco.kubevirt.io/v1beta1
  kind: HyperConverged
  metadata:
    name: kubevirt-hyperconverged
    namespace: openshift-cnv
  spec:
    liveMigrationConfig:
      bandwidthPerMigration: 2048Mi
      completionTimeoutPerGiB: 800
      parallelMigrationsPerCluster: 5
      parallelOutboundMigrationsPerNode: 2
      progressTimeout: 150
      allowPostCopy: false
  ```

  where:

  <dl>
  <dt><code>bandwidthPerMigration</code></dt>
  <dd>Specifies the bandwidth of each migration in bytes per second. For example, a value of 2048Mi means 2048 MiB/s. Default: 0, which is unlimited.</dd>
  <dt><code>completionTimeoutPerGiB</code></dt>
  <dd>Specifies the length of time, in seconds per GiB of memory, at which the migration is canceled if it has not completed. For example, a VM with 6GiB memory times out if it has not completed migration in 4800 seconds. If the <code>Migration Method</code> is <code>BlockMigration</code>, the size of the migrating disks is included in the calculation.</dd>
  <dt><code>parallelMigrationsPerCluster</code></dt>
  <dd>Specifies the number of migrations running in parallel in the cluster. Default: <code>5</code>.</dd>
  <dt><code>parallelOutboundMigrationsPerNode</code></dt>
  <dd>Specifies the maximum number of outbound migrations per node. Default: <code>2</code>.</dd>
  <dt><code>progressTimeout</code></dt>
  <dd>Specifies the length of time, in seconds, at which the migration is canceled if memory copy fails to make progress. Default: <code>150</code>.</dd>
  <dt><code>allowPostCopy</code></dt>
  <dd>Specifies whether the post copy mode is enabled. You can enable post copy mode to allow the migration of one node to another to converge, even if a VM is running a heavy workload and the memory dirty rate is too high. By default, <code>allowPostCopy</code> is set to <code>false</code>.</dd>
  </dl>

  :::note

  You can restore the default value for any `spec.liveMigrationConfig` field by deleting that key/value pair and saving the file. For example, delete `progressTimeout: <value>` to restore the default `progressTimeout: 150`.

  :::

## Configure live migration for heavy workloads {#virt-configuring-live-migration-heavy_virt-configuring-live-migration}

When migrating a VM running a heavy workload, such as database processing, higher memory dirty rates can prevent the migration from completing. To address this, you can enable post copy mode, which allows the migration to complete when the pre-copy phase cannot converge.

Configure live migration for heavy workloads by updating the `HyperConverged` custom resource (CR) in the `openshift-cnv` namespace.

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).

**Procedure**

1. Edit the `HyperConverged` CR and add the necessary parameters for migrating heavy workloads:
   ```terminal
   $ oc edit hyperconvergeds.v1beta1.hco.kubevirt.io kubevirt-hyperconverged -n openshift-cnv
   ```

   Example configuration file:

   ```yaml
   apiVersion: hco.kubevirt.io/v1beta1
   kind: HyperConverged
   metadata:
     name: kubevirt-hyperconverged
     namespace: openshift-cnv
   spec:
     liveMigrationConfig:
       bandwidthPerMigration: 0Mi
       completionTimeoutPerGiB: 150
       parallelMigrationsPerCluster: 5
       parallelOutboundMigrationsPerNode: 1
       progressTimeout: 150
       allowPostCopy: true
   ```

   where:

   <dl>
   <dt><code>bandwidthPerMigration</code></dt>
   <dd>Specifies the bandwidth of each migration in bytes per second. The default is <code>0</code>, which is unlimited.</dd>
   <dt><code>completionTimeoutPerGiB</code></dt>
   <dd>Specifies the length of time, in seconds per GiB of memory, at which the migration is canceled if it has not completed and post copy mode is triggered, if enabled. You can lower <code>completionTimeoutPerGiB</code> to trigger post copy mode earlier in the migration process, or raise the  <code>completionTimeoutPerGiB</code> to trigger post copy mode later in the migration process.</dd>
   <dt><code>parallelMigrationsPerCluster</code></dt>
   <dd>Specifies the number of migrations running in parallel in the cluster. The default is <code>5</code>. Keeping the <code>parallelMigrationsPerCluster</code> setting low is better when migrating heavy workloads.</dd>
   <dt><code>parallelOutboundMigrationsPerNode</code></dt>
   <dd>Specifies the maximum number of outbound migrations per node. Configure a single VM per node for heavy workloads.</dd>
   <dt><code>progressTimeout</code></dt>
   <dd>Specifies the length of time, in seconds, at which the migration is canceled if memory copy fails to make progress. Increase this parameter for large memory sizes running heavy workloads.</dd>
   <dt><code>allowPostCopy</code></dt>
   <dd>Specifies whether the post copy mode is enabled. You can enable post copy mode to allow the migration of one node to another to converge, even if a VM is running a heavy workload and the memory dirty rate is too high. Set allowPostCopy to true to enable post copy mode.</dd>
   </dl>
2. Optional: If your main network is too busy for the migration, configure a secondary, dedicated migration network.
   :::note

   Post copy mode can impact performance during the transfer, and should not be used for critical data, or with unstable networks.

   :::

## Live migration policies {#virt-live-migration-policies_virt-configuring-live-migration}

You can create live migration policies to apply different migration configurations to groups of VMs that are defined by VM or project labels.

:::tip

You can create live migration policies by using the OpenShift Container Platform web console.

:::

## Creating a live migration policy by using the CLI {#virt-configuring-a-live-migration-policy_virt-configuring-live-migration}

You can create a live migration policy by using the command line.

KubeVirt applies the live migration policy to selected virtual machines (VMs) by using any combination of labels:

- VM labels such as `size`, `os`, or `gpu`
- Project labels such as `priority`, `bandwidth`, or `hpc-workload`

For the policy to apply to a specific group of VMs, all labels on the group of VMs must match the labels of the policy.

:::note

If multiple live migration policies apply to a VM, the policy with the greatest number of matching labels takes precedence.

If multiple policies meet this criteria, the policies are sorted by alphabetical order of the matching label keys, and the first one in that order takes precedence.

:::

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).

**Procedure**

1. Edit the VM object to which you want to apply a live migration policy, and add the corresponding VM labels.
   1. Open the YAML configuration of the resource:
      ```terminal
      $ oc edit vm <vm_name>
      ```
   2. Adjust the required label values in the `.spec.template.metadata.labels` section of the configuration. For example, to mark the VM as a `production` VM for the purposes of migration policies, add the `kubevirt.io/environment: production` line:
      ```yaml
      apiVersion: migrations.kubevirt.io/v1alpha1
      kind: VirtualMachine
      metadata:
        name: <vm_name>
        namespace: default
        labels:
          app: my-app
          environment: production
      spec:
        template:
          metadata:
            labels:
              kubevirt.io/domain: <vm_name>
              kubevirt.io/size: large
              kubevirt.io/environment: production
      # ...
      ```
   3. Save and exit the configuration.
2. Configure a `MigrationPolicy` object with the corresponding labels. The following example configures a policy that applies to all VMs that are labeled as `production`:
   ```yaml
   apiVersion: migrations.kubevirt.io/v1alpha1
   kind: MigrationPolicy
   metadata:
     name: <migration_policy>
   spec:
     selectors:
       namespaceSelector:
         hpc-workloads: "True"
         xyz-workloads-type: ""
       virtualMachineInstanceSelector:
         kubevirt.io/environment: "production"
   ```

   where:

   <dl>
   <dt><code>namespaceSelector</code></dt>
   <dd>Specifies the project labels.</dd>
   <dt><code>virtualMachineInstanceSelector</code></dt>
   <dd>Specifies the VM labels.</dd>
   </dl>
3. Create the migration policy by running the following command:
   ```terminal
   $ oc create -f <migration_policy>.yaml
   ```

**Additional resources**

- [Configuring a dedicated network for live migration](/docs/virt/vm_networking/virt-dedicated-network-live-migration#virt-configuring-secondary-network-vm-live-migration_virt-dedicated-network-live-migration)
