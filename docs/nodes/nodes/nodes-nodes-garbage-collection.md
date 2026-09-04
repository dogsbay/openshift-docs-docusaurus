---
title: Freeing node resources using garbage collection
sidebar_position: 11
---

# Freeing node resources using garbage collection {#nodes-nodes-garbage-collection}

<a id="nodes-nodes-garbage-collection"></a>

As an administrator, you can use OpenShift Container Platform to ensure that your nodes are running efficiently by freeing up resources through garbage collection.

The OpenShift Container Platform node performs two types of garbage collection:

- Container garbage collection: Removes terminated containers.
- Image garbage collection: Removes images not referenced by any running pods.

## Understanding how terminated containers are removed through garbage collection {#nodes-nodes-garbage-collection-containers_nodes-nodes-configuring}

You can help ensure that your nodes are running efficiently by using container garbage collection to remove terminated containers.

When eviction thresholds are set for garbage collection, the node tries to keep any container for any pod accessible from the API. If the pod has been deleted, the containers will be as well. Containers are preserved as long the pod is not deleted and the eviction threshold is not reached. If the node is under disk pressure, it will remove containers and their logs will no longer be accessible using `oc logs`.

- **eviction-soft** - A soft eviction threshold pairs an eviction threshold with a required administrator-specified grace period.
- **eviction-hard** - A hard eviction threshold has no grace period, and if observed, OpenShift Container Platform takes immediate action.

The following table lists the eviction thresholds:

**Variables for configuring container garbage collection**

<table>
<thead>
<tr>
  <th>Node condition</th>
  <th>Eviction signal</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>MemoryPressure</td>
  <td><code>memory.available</code></td>
  <td>The available memory on the node.</td>
</tr>
<tr>
  <td>DiskPressure</td>
  <td><ul><li><code>nodefs.available</code></li><li><code>nodefs.inodesFree</code></li><li><code>imagefs.available</code></li><li><code>imagefs.inodesFree</code></li></ul></td>
  <td>The available disk space or inodes on the node root file system, <code>nodefs</code>, or image file system, <code>imagefs</code>.</td>
</tr>
</tbody>
</table>

:::note

For `evictionHard` you must specify all of these parameters.  If you do not specify all parameters, only the specified parameters are applied and the garbage collection will not function properly.

:::

If a node is oscillating above and below a soft eviction threshold, but not exceeding its associated grace period, the corresponding node would constantly oscillate between `true` and `false`. As a consequence, the scheduler could make poor scheduling decisions.

To protect against this oscillation, use the `evictionpressure-transition-period` flag to control how long OpenShift Container Platform must wait before transitioning out of a pressure condition. OpenShift Container Platform will not set an eviction threshold as being met for the specified pressure condition for the period specified before toggling the condition back to false.

:::note

Setting the `evictionPressureTransitionPeriod` parameter to `0` configures the default value of 5 minutes. You cannot set an eviction pressure transition period to zero seconds.

:::

## Understanding how images are removed through garbage collection {#nodes-nodes-garbage-collection-images_nodes-nodes-configuring}

You can help ensure that your nodes are running efficiently by using image garbage collection to removes images that are not referenced by any running pods.

OpenShift Container Platform determines which images to remove from a node based on the disk usage that is reported by **cAdvisor**.

The policy for image garbage collection is based on two conditions:

- The percent of disk usage (expressed as an integer) which triggers image garbage collection. The default is **85**.
- The percent of disk usage (expressed as an integer) to which image garbage collection attempts to free. Default is **80**.

For image garbage collection, you can modify any of the following variables using a custom resource.

**Variables for configuring image garbage collection**

| Setting | Description |
| --- | --- |
| `imageMinimumGCAge` | The minimum age for an unused image before the image is removed by garbage collection. The default is **2m**. |
| `imageGCHighThresholdPercent` | The percent of disk usage, expressed as an integer, which triggers image garbage collection. The default is **85**. This value must be greater than the `imageGCLowThresholdPercent` value. |
| `imageGCLowThresholdPercent` | The percent of disk usage, expressed as an integer, to which image garbage collection attempts to free. The default is **80**. This value must be less than the `imageGCHighThresholdPercent` value. |

Two lists of images are retrieved in each garbage collector run:

1. A list of images currently running in at least one pod.
2. A list of images available on a host.

As new containers are run, new images appear. All images are marked with a time stamp. If the image is running (the first list above) or is newly detected (the second list above), it is marked with the current time. The remaining images are already marked from the previous spins. All images are then sorted by the time stamp.

Once the collection starts, the oldest images get deleted first until the stopping criterion is met.

## Configuring garbage collection for containers and images {#nodes-nodes-garbage-collection-configuring_nodes-nodes-configuring}

As an administrator, you can configure how OpenShift Container Platform performs garbage collection by creating a `kubeletConfig` object for each machine config pool. Performing garbage collection helps ensure that your nodes are running efficiently.

:::note

OpenShift Container Platform supports only one `kubeletConfig` object for each machine config pool.

:::

You can configure any combination of the following:

- Soft eviction for containers
- Hard eviction for containers
- Eviction for images

Container garbage collection removes terminated containers. Image garbage collection removes images that are not referenced by any running pods.

**Prerequisites**

1. Obtain the label associated with the static `MachineConfigPool` CRD for the type of node you want to configure by entering the following command:
   ```terminal
   $ oc edit machineconfigpool <name>
   ```

   For example:

   ```terminal
   $ oc edit machineconfigpool worker
   ```

   ```yaml title="Example output"
   apiVersion: machineconfiguration.openshift.io/v1
   kind: MachineConfigPool
   metadata:
     creationTimestamp: "2022-11-16T15:34:25Z"
     generation: 4
     labels:
       pools.operator.machineconfiguration.openshift.io/worker: ""
     name: worker
   #...
   ```

   where:

   <dl>
   <dt><code>metadata.labels</code></dt>
   <dd>Specifies a label to use with the kubelet configuration.</dd>
   </dl>

   :::tip

   If the label is not present, add a key/value pair such as:

   ```
   $ oc label machineconfigpool worker custom-kubelet=small-pods
   ```

   :::

**Procedure**

1. Create a custom resource (CR) for your configuration change.
   :::warning

   If there is one file system, or if `/var/lib/kubelet` and `/var/lib/containers/` are in the same file system, the settings with the highest values trigger evictions, as those are met first. The file system triggers the eviction.

   :::

   ```yaml title="Sample configuration for a container garbage collection CR"
   apiVersion: machineconfiguration.openshift.io/v1
   kind: KubeletConfig
   metadata:
     name: worker-kubeconfig
   spec:
     machineConfigPoolSelector:
       matchLabels:
         pools.operator.machineconfiguration.openshift.io/worker: ""
     kubeletConfig:
       evictionSoft:
         memory.available: "500Mi"
         nodefs.available: "10%"
         nodefs.inodesFree: "5%"
         imagefs.available: "15%"
         imagefs.inodesFree: "10%"
       evictionSoftGracePeriod:
         memory.available: "1m30s"
         nodefs.available: "1m30s"
         nodefs.inodesFree: "1m30s"
         imagefs.available: "1m30s"
         imagefs.inodesFree: "1m30s"
       evictionHard:
         memory.available: "200Mi"
         nodefs.available: "5%"
         nodefs.inodesFree: "4%"
         imagefs.available: "10%"
         imagefs.inodesFree: "5%"
       evictionPressureTransitionPeriod: 3m
       imageMinimumGCAge: 5m
       imageGCHighThresholdPercent: 80
       imageGCLowThresholdPercent: 75
   #...
   ```

   where:

   <dl>
   <dt><code>metadata.name</code></dt>
   <dd>Specifies a name for the object.</dd>
   <dt><code>spec.machineConfigPoolSelector.matchLabels</code></dt>
   <dd>Specifies the label from the machine config pool.</dd>
   <dt><code>spec.kubeletConfig.evictionSoft</code></dt>
   <dd>Specifies a soft eviction and eviction thresholds for container garbage collection.</dd>
   <dt><code>spec.kubeletConfig.evictionSoftGracePeriod</code></dt>
   <dd>Specifies a grace period for the soft eviction of containers. This parameter does not apply to <code>eviction-hard</code>.</dd>
   <dt><code>spec.kubeletConfig.evictionHard</code></dt>
   <dd>Specifies a soft eviction and eviction thresholds for container garbage collection. For <code>evictionHard</code> you must specify all of these parameters. If you do not specify all parameters, only the specified parameters are applied and the garbage collection will not function properly.</dd>
   <dt><code>spec.kubeletConfig.evictionPressureTransitionPeriod</code></dt>
   <dd>Specifies the duration to wait before transitioning out of an eviction pressure condition for container garbage collection. Setting the <code>evictionPressureTransitionPeriod</code> parameter to <code>0</code> configures the default value of 5 minutes.</dd>
   <dt><code>spec.kubeletConfig.imageMinimumGCAge</code></dt>
   <dd>Specifies the minimum age for an unused image before the image is removed by garbage collection.</dd>
   <dt><code>spec.kubeletConfig.imageGCHighThresholdPercent</code></dt>
   <dd>Specifies the percent of disk usage, expressed as an integer, that triggers image garbage collection. This value must be greater than the <code>imageGCLowThresholdPercent</code> value.</dd>
   <dt><code>spec.kubeletConfig.imageGCHighThresholdPercent</code></dt>
   <dd>Specifies the percent of disk usage, expressed as an integer, to which image garbage collection attempts to free. This value must be less than the <code>imageGCHighThresholdPercent</code> value.</dd>
   </dl>
2. Run the following command to create the CR:
   ```terminal
   $ oc create -f <file_name>.yaml
   ```

   For example:

   ```terminal
   $ oc create -f gc-container.yaml
   ```

   ```terminal title="Example output"
   kubeletconfig.machineconfiguration.openshift.io/gc-container created
   ```

**Verification**

- Verify that garbage collection is active by entering the following command. The Machine Config Pool you specified in the custom resource appears with `UPDATING` as 'true` until the change is fully implemented:
  ```terminal
  $ oc get machineconfigpool
  ```

  ```terminal title="Example output"
  NAME     CONFIG                                   UPDATED   UPDATING
  master   rendered-master-546383f80705bd5aeaba93   True      False
  worker   rendered-worker-b4c51bb33ccaae6fc4a6a5   False     True
  ```
