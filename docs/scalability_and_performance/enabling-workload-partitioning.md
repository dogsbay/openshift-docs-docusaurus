---
title: Workload partitioning
sidebar_position: 23
---

# Workload partitioning {#enabling-workload-partitioning}

<a id="enabling-workload-partitioning"></a>

Workload partitioning separates compute node CPU resources into distinct CPU sets. Ensure that you keep platform pods on the specified cores to avoid interrupting the CPUs the customer workloads are running on.

The minimum number of reserved CPUs required for the cluster management is four CPU Hyper-Threads (HTs).

In the context of enabling workload partitioning and managing CPU resources effectively, the cluster might not permit incorrectly configured nodes to join the cluster through a node admission webhook. When the workload partitioning feature is enabled, the machine config pools for control plane nodes and compute nodes get supplied with configurations for nodes to use. Adding new nodes to these pools ensures the pools correctly get configured before joining the cluster.

Currently, nodes must have uniform configurations per machine config pool to ensure that correct CPU affinity is set across all nodes within that pool. After admission, nodes within the cluster identify themselves as supporting a new resource type called `management.workload.openshift.io/cores` and accurately report their CPU capacity. Workload partitioning can be enabled during cluster installation only by adding the additional field `cpuPartitioningMode` to the `install-config.yaml` file.

When workload partitioning is enabled, the `management.workload.openshift.io/cores` resource allows the scheduler to correctly assign pods based on the `cpushares` capacity of the host, not just the default `cpuset`. This ensures more precise allocation of resources for workload partitioning scenarios.

Workload partitioning ensures that CPU requests and limits specified in the pod’s configuration are respected. In OpenShift Container Platform 4.16 or later, accurate CPU usage limits are set for platform pods through CPU partitioning. As workload partitioning uses the custom resource type of `management.workload.openshift.io/cores`, the values for requests and limits are the same due to a requirement by Kubernetes for extended resources. However, the annotations modified by workload partitioning correctly reflect the desired limits.

:::note

Extended resources cannot be overcommitted, so request and limit must be equal if both are present in a container spec.

:::

## Enable workload partitioning {#enabling-workload-partitioning_enabling-workload-partitioning}

To partition cluster management pods into a specified CPU affinity, enable workload partitioning. This configuration ensures that management pods operate within the reserved CPU limits defined in your Performance Profile.

Consider additional post-installation Operators that use workload partitioning when calculating how many reserved CPU cores to set aside for the platform.

Workload partitioning isolates user workloads from platform workloads using standard Kubernetes scheduling capabilities.

:::note

You can enable workload partitioning only during cluster installation. You cannot disable workload partitioning post-installation. However, you can change the CPU configuration for `reserved` and `isolated` CPUs post-installation.

:::

The procedure demonstrates enabling workload partitioning cluster-wide.

**Procedure**

- In the `install-config.yaml` file, add the additional field `cpuPartitioningMode` and set it to `AllNodes`.
  ```yaml
  apiVersion: v1
  baseDomain: devcluster.openshift.com
  cpuPartitioningMode: AllNodes
  compute:
    - architecture: amd64
      hyperthreading: Enabled
      name: worker
      platform: {}
      replicas: 3
  controlPlane:
    architecture: amd64
    hyperthreading: Enabled
    name: master
    platform: {}
    replicas: 3
  ```

  - `cpuPartitioningMode`: Specifies the cluster to set up for CPU partitioning at install time. The default value is `None`, which ensures that no CPU partitioning is enabled at install time.

## Performance profiles and workload partitioning {#performance-profile-workload-partitioning_enabling-workload-partitioning}

To enable workload partitioning, apply a performance profile.

An appropriately configured performance profile specifies the `isolated` and `reserved` CPUs. Create a performance profile by using the Performance Profile Creator (PPC) tool.

```yaml title="Sample performance profile configuration"
apiVersion: performance.openshift.io/v2
kind: PerformanceProfile
metadata:
  # if you change this name make sure the 'include' line in TunedPerformancePatch.yaml
  # matches this name: include=openshift-node-performance-${PerformanceProfile.metadata.name}
  # Also in file 'validatorCRs/informDuValidator.yaml':
  # name: 50-performance-${PerformanceProfile.metadata.name}
  name: openshift-node-performance-profile
  annotations:
    ran.openshift.io/reference-configuration: "ran-du.redhat.com"
spec:
  additionalKernelArgs:
    - "rcupdate.rcu_normal_after_boot=0"
    - "efi=runtime"
    - "vfio_pci.enable_sriov=1"
    - "vfio_pci.disable_idle_d3=1"
    - "module_blacklist=irdma"
  cpu:
    isolated: $isolated
    reserved: $reserved
  hugepages:
    defaultHugepagesSize: $defaultHugepagesSize
    pages:
      - size: $size
        count: $count
        node: $node
  machineConfigPoolSelector:
    pools.operator.machineconfiguration.openshift.io/$mcp: ""
  nodeSelector:
    node-role.kubernetes.io/$mcp: ''
  numa:
    topologyPolicy: "restricted"
  # To use the standard (non-realtime) kernel, set enabled to false
  realTimeKernel:
    enabled: true
  workloadHints:
    # WorkloadHints defines the set of upper level flags for different type of workloads.
    # See https://github.com/openshift/cluster-node-tuning-operator/blob/master/docs/performanceprofile/performance_profile.md#workloadhints
    # for detailed descriptions of each item.
    # The configuration below is set for a low latency, performance mode.
    realTime: true
    highPowerConsumption: false
    perPodPowerManagement: false
```

**PerformanceProfile CR options for single-node OpenShift clusters**

<table>
<thead>
<tr>
  <th>PerformanceProfile CR field</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>metadata.name</code></td>
  <td>Ensure that <code>name</code> matches the following fields set in related GitOps ZTP custom resources (CRs):<br><br><ul><li><code>include=openshift-node-performance-${PerformanceProfile.metadata.name}</code> in <code>TunedPerformancePatch.yaml</code></li><li><code>name: 50-performance-${PerformanceProfile.metadata.name}</code> in <code>validatorCRs/informDuValidator.yaml</code></li></ul></td>
</tr>
<tr>
  <td><code>spec.additionalKernelArgs</code></td>
  <td><code>"efi=runtime"</code> Configures UEFI secure boot for the cluster host.</td>
</tr>
<tr>
  <td><code>spec.cpu.isolated</code></td>
  <td>Set the isolated CPUs. Ensure all of the Hyper-Threading pairs match.<br><br><div class="alert alert--warning"><strong>Important</strong><p>The reserved and isolated CPU pools must not overlap and together must span all available cores. CPU cores that are not accounted for cause an undefined behaviour in the system.</p></div></td>
</tr>
<tr>
  <td><code>spec.cpu.reserved</code></td>
  <td>Set the reserved CPUs. When workload partitioning is enabled, system processes, kernel threads, and system container threads are restricted to these CPUs. All CPUs that are not isolated should be reserved.</td>
</tr>
<tr>
  <td><code>spec.hugepages.pages</code></td>
  <td><ul><li>Set the number of huge pages (<code>count</code>)</li><li>Set the huge pages size (<code>size</code>).</li><li>Set <code>node</code> to the NUMA node where the <code>hugepages</code> are allocated (<code>node</code>)</li></ul></td>
</tr>
<tr>
  <td><code>spec.realTimeKernel</code></td>
  <td>Set <code>enabled</code> to <code>true</code> to use the realtime kernel.</td>
</tr>
<tr>
  <td><code>spec.workloadHints</code></td>
  <td>Use <code>workloadHints</code> to define the set of top level flags for different type of workloads. The example configuration configures the cluster for low latency and high performance.</td>
</tr>
</tbody>
</table>

**Additional resources**

- [About the Performance Profile Creator](/docs/scalability_and_performance/cnf-tuning-low-latency-nodes-with-perf-profile#cnf-about-the-profile-creator-tool_cnf-tuning-low-latency-nodes-with-perf-profile)
