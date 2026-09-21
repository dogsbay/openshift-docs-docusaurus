---
title: Telco core reference design specifications
sidebar_position: 3
---

# Telco core reference design specifications {#telco-core-ref-design-specs}

<a id="telco-core-ref-design-specs"></a>

The telco core reference design specifications (RDS) configures an OpenShift Container Platform cluster running on commodity hardware to host telco core workloads.

## Telco core RDS use model overview {#telco-core-rds-product-version-use-model-overview_telco-core}

The Telco core reference design specification (RDS) describes a platform that supports large-scale telco applications including control plane functions such as signaling and aggregation. It also includes some centralized data plane functions, for example, user plane functions (UPF). These functions generally require scalability, complex networking support, resilient software-defined storage, and support performance requirements that are less stringent and constrained than far-edge deployments such as RAN.

## About the telco core cluster use model {#telco-core-about-the-telco-core-cluster-use-model_telco-core}

The telco core cluster use model is designed for clusters running on commodity hardware. Telco core clusters support large scale telco applications including control plane functions like signaling, aggregation, session border controller (SBC), and centralized data plane functions such as 5G user plane functions (UPF). Telco core cluster functions require scalability, complex networking support, resilient software-defined storage, and support performance requirements that are less stringent and constrained than far-edge RAN deployments.

Networking requirements for telco core functions vary widely across a range of networking features and performance points. IPv6 is a requirement and dual-stack is common. Some functions need maximum throughput and transaction rate and require support for user-plane DPDK networking. Other functions use typical cloud-native patterns and can rely on OVN-Kubernetes, kernel networking, and load balancing.

Telco core clusters are configured as standard with three control plane and one or more worker nodes configured with the stock (non-RT) kernel. In support of workloads with varying networking and performance requirements, you can segment worker nodes by using `MachineConfigPool` custom resources (CR), for example, for non-user data plane or high-throughput use cases. In support of required telco operational features, core clusters have a standard set of Day 2 OLM-managed Operators installed.

**Figure 1. Telco core RDS cluster service-based architecture and networking topology**

![5G core cluster showing a service-based architecture with overlaid networking topology](/images/openshift-5g-core-cluster-architecture-networking.png)

## Reference design scope {#telco-ran-core-ref-design-spec_telco-core}

The telco core, telco RAN and telco hub reference design specifications (RDS) capture the recommended, tested, and supported configurations to get reliable and repeatable performance for clusters running the telco core and telco RAN profiles.

Each RDS includes the released features and supported configurations that are engineered and validated for clusters to run the individual profiles. The configurations provide a baseline OpenShift Container Platform installation that meets feature and KPI targets. Each RDS also describes expected variations for each individual configuration. Validation of each RDS includes many long duration and at-scale tests.

:::note

The validated reference configurations are updated for each major Y-stream release of OpenShift Container Platform. Z-stream patch releases are periodically re-tested against the reference configurations.

:::

## Deviations from the reference design {#telco-deviations-from-the-ref-design_telco-core}

Deviating from the validated telco core, telco RAN DU, and telco hub reference design specifications (RDS) can have significant impact beyond the specific component or feature that you change. Deviations require analysis and engineering in the context of the complete solution.

:::warning

All deviations from the RDS should be analyzed and documented with clear action tracking information. Due diligence is expected from partners to understand how to bring deviations into line with the reference design. This might require partners to provide additional resources to engage with Red Hat to work towards enabling their use case to achieve a best in class outcome with the platform. This is critical for the supportability of the solution and ensuring alignment across Red Hat and with partners.

:::

Deviation from the RDS can have some or all of the following consequences:

- It can take longer to resolve issues.
- There is a risk of missing project service-level agreements (SLAs), project deadlines, end provider performance requirements, and so on.
- Unapproved deviations may require escalation at executive levels.

:::note

Red Hat prioritizes the servicing of requests for deviations based on partner engagement priorities.

:::

## Telco core common baseline model {#telco-core-common-baseline-model_telco-core}

The following configurations and use models are applicable to all telco core use cases. The telco core use cases build on this common baseline of features.

Cluster topology
: The telco core reference design supports two distinct cluster configuration variants:

  - A non-schedulable control plane variant, where user workloads are strictly prohibited from running on master nodes.
  - A schedulable control plane variant, which allows for user workloads to run on master nodes to optimize resource utilization. This variant is only applicable to bare-metal control plane nodes and must be configured at installation time.

  All clusters, regardless of the variant, must conform to the following requirements:

  - A highly available control plane consisting of three or more nodes.
  - The use of multiple machine config pools.

Storage
: Telco core use cases require highly available persistent storage as provided by an external storage solution. OpenShift Data Foundation might be used to manage access to the external storage.

Networking
: Telco core cluster networking conforms to the following requirements:

  - Dual stack IPv4/IPv6 (IPv4 primary).
  - Fully disconnected - clusters do not have access to public networking at any point in their lifecycle.
  - Supports multiple networks. Segmented networking provides isolation between operations, administration and maintenance (OAM), signaling, and storage traffic.
  - Cluster network type is OVN-Kubernetes as required for IPv6 support.
  - Telco core clusters have multiple layers of networking supported by underlying RHCOS, SR-IOV Network Operator, Load Balancer and other components. These layers include the following:
    - Cluster networking layer. The cluster network configuration is defined and applied through the installation configuration. Update the configuration during Day 2 operations with the NMState Operator. Use the initial configuration to establish the following:
      - Host interface configuration.
      - Active/active bonding (LACP).
    - Secondary/additional network layer. Configure the OpenShift Container Platform CNI through network `additionalNetwork` or `NetworkAttachmentDefinition` CRs. Use the initial configuration to configure MACVLAN virtual network interfaces.
    - Application workload layer. User plane networking runs in cloud-native network functions (CNFs).

Service Mesh
: Telco CNFs can use Service Mesh. Telco core clusters typically include a Service Mesh implementation. The choice of implementation and configuration is outside the scope of this specification.

## Deployment planning {#telco-core-deployment-planning_telco-core}

Proper deployment planning is essential for telco core clusters to ensure high availability, minimize service disruption during upgrades, and optimize cluster performance.

### Worker Nodes and MachineConfigPools {#telco-core-worker-nodes-and-machineconfigpools_telco-core}

`MachineConfigPools` (MCPs) custom resources (CR) enable the subdivision of worker nodes in telco core clusters into different node groups based on customer planning parameters. Careful deployment planning using MCPs is crucial to minimize deployment and upgrade time and, more importantly, to minimize interruption of telco-grade services during cluster upgrades.

**Description**

Telco core clusters can use `MachineConfigPools` (MCPs) to split worker nodes into additional separate roles, for example, due to different hardware profiles. This allows custom tuning for each role and also plays a critical function in speeding up a telco core cluster deployment or upgrade. Multiple MCPs can be used to properly plan cluster upgrades across one or multiple maintenance windows. This is crucial because telco-grade services might otherwise be affected if careful planning is not considered.

During cluster upgrades, you can pause MCPs while you upgrade the control plane. See "Performing a canary rollout update" for more information. This ensures that worker nodes are not rebooted and running workloads remain unaffected until the MCP is unpaused.

Using careful MCP planning, you can control the timing and order of which set of nodes are upgraded at any time. For more information on how to use MCPs to plan telco upgrades, see "Applying MachineConfigPool labels to nodes before the update".

Before beginning the initial deployment, review the following engineering considerations:

**PerformanceProfile and Tuned profile association:**

When using PerformanceProfiles, remember that each Machine Config Pool (MCP) must be linked to exactly one PerformanceProfile or Tuned profile definition. Consequently, even if the desired configuration is identical for multiple MCPs, each MCP still requires its own dedicated PerformanceProfile definition.

**Planning your MCP labeling strategy:**

Plan your MCP labeling with an appropriate strategy to split your worker nodes depending on parameters such as:

- The worker node type: identifying a group of nodes with equivalent hardware profile, for example workers for control plane Network Functions (NFs) and workers for user data plane NFs.
- The number of worker nodes per worker node type.
- The minimum number of MCPs required for an equivalent hardware profile is 1, but could be larger for larger clusters. For example, you may design for more MCPs per hardware profile to support a more granular upgrade where a smaller percentage of the cluster capacity is affected with each step.
- The update strategy for nodes within an MCP is by upgrade requirements and the chosen `maxUnavailable` value:
  - Number of maintenance windows allowed
  - Duration of a maintenance window
  - Total number of worker nodes
  - Desired `maxUnavailable` (number of nodes updated concurrently) for the MCP.
- CNF requirements for worker nodes, in terms of:
  - Minimum availability per Pod required during an upgrade, configured with a pod disruption budget (PDB). PDBs are crucial to maintain telco service level Agreements (SLAs) during upgrades. For more information about PDB, see "Understanding how to use pod disruption budgets to specify the number of pods that must be up".
  - Minimum true high availability required per Pod, such that each replica runs on separate hardware.
  - Pod affinity and anti-affinity link: For more information about how to use pod affinity and anti-affinity, see "Placing pods relative to other pods using affinity and anti-affinity rules".
- Duration and number of upgrade maintenance windows during which telco-grade services might be affected.

### Zones {#telco-core-zones_telco-core}

Designing the cluster to support disruption of multiple nodes simultaneously is critical for high availability (HA) and reduced upgrade times. OpenShift Container Platform and Kubernetes use the label `topology.kubernetes.io/zone` to create pools of nodes which are subject to a common failure domain. Annotating nodes for topology (availability) zones allows high-availability workloads to spread such that each zone holds only one replica from a set of HA replicated pods. With this spread the loss of a single zone will not violate HA constraints and minimum service availability will be maintained. OpenShift Container Platform and Kubernetes apply a default TopologySpreadConstraint to all replica constructs, such as `Service`, `ReplicaSet`, `StatefulSet` or `ReplicationController` resources, which spreads the replicas based on the `topology.kubernetes.io/zone` label. This default allows zone based spread to apply without any change to your workload pod specs.

Cluster upgrades typically result in node disruption as the underlying OS is updated. In large clusters it is necessary to update multiple nodes concurrently in order to complete upgrades quickly and in as few maintenance windows as possible. By using zones to ensure pod spread, an upgrade can be applied to all nodes in a zone simultaneously (assuming sufficient spare capacity) while maintaining high availability and service availability. The recommended cluster design is to partition nodes into multiple MCPs based on the considerations above and label all nodes in a single MCP as a single zone which is distinct from zones attached to other MCPs. Using this strategy all nodes in an MCP can be updated simultaneously.

Lifecycle hooks (readiness, liveness, startup and pre-stop) play an important role in ensuring application availability. For upgrades in particular the pre-stop hook allows applications to take necessary steps to prepare for disruption prior to being evicted from the node.

Limits and requirements
: - The default TopologySpreadConstraints (TSC) only apply when an explicit TSC is not given. If your pods have explicit TSC ensure that spread based on zones is included.
  - The cluster must have sufficient spare capacity to tolerate simultaneous update of an MCP. Otherwise the `maxUnavailable` of the MCP must be set to less than 100%.
  - The ability to update all nodes in an MCP simultaneously further depends on workload design and ability to maintain required service levels with that level of disruption.

Engineering Considerations
: - Pod drain times can significantly impact node update times. Ensure the workload design allows pods to be drained quickly.
  - PodDisruptionBudgets (PDB) are used to enforce high availability requirements.
    - With sufficient zones in the cluster design, a zone can be disrupted or lost without violating the PDB. If there are insufficient zones, or other scheduling constraints restrict the set of available nodes, the scheduling of pods might violate the PDB when the zone is disrupted or lost. During upgrades with simultaneous node updates this might lead to partial serialization of updates.
    - PDB with 0 disruptable pods will block node drain and require administrator intervention. This pattern should be avoided for fast and automated upgrades.

**Additional resources**

- [Performing a canary rollout update](/docs/updating/updating_a_cluster/update-using-custom-machine-config-pools#update-using-custom-machine-config-pools)
- [Applying MachineConfigPool labels to nodes before the update](/docs/updating/updating_a_cluster/update-using-custom-machine-config-pools#update-using-custom-machine-config-pools)
- [Understanding how to use pod disruption budgets to specify the number of pods that must be up](/docs/nodes/pods/nodes-pods-configuring#nodes-pods-pod-disruption-about_nodes-pods-configuring)
- [Placing pods relative to other pods using affinity and anti-affinity rules](/docs/nodes/scheduling/nodes-scheduler-pod-affinity#nodes-scheduler-pod-affinity)

## Telco core cluster common use model engineering considerations {#telco-core-cluster-common-use-model-engineering-considerations_telco-core}

The following engineering considerations apply to the telco core cluster common use model.

- Cluster workloads are detailed in "Application workloads".
- Worker nodes should run on either of the following CPUs:
  - Intel 3rd Generation Xeon (IceLake) CPUs or better when supported by OpenShift Container Platform, or CPUs with the silicon security bug (Spectre and similar) mitigations turned off. Skylake and older CPUs can experience 40% transaction performance drops when Spectre and similar mitigations are enabled.
  - AMD EPYC Zen 4 CPUs (Genoa, Bergamo) or AMD EPYC Zen 5 CPUs (Turin) when supported by OpenShift Container Platform.
  - Intel Sierra Forest CPUs when supported by the OpenShift Container Platform.
  - IRQ balancing is enabled on worker nodes. The `PerformanceProfile` CR sets the `globallyDisableIrqLoadBalancing` parameter to a value of `false`. Guaranteed QoS pods are annotated to ensure isolation as described in "CPU partitioning and performance tuning".
- All cluster nodes should have the following features:
  - Have Hyper-Threading enabled
  - Have x86_64 CPU architecture
  - Have the stock (non-realtime) kernel enabled
  - Are not configured for workload partitioning
- The balance between power management and maximum performance varies between machine config pools in the cluster. The following configurations should be consistent for all nodes in a machine config pools group.
  - Cluster scaling. See "Scalability" for more information.
  - Clusters should be able to scale to at least 120 nodes.
- CPU partitioning is configured using a `PerformanceProfile` CR and is applied to nodes on a per `MachineConfigPool` basis. See "CPU partitioning and performance tuning" for additional considerations.
- CPU requirements for OpenShift Container Platform depend on the configured feature set and application workload characteristics. For a cluster configured according to the reference configuration running a simulated workload of 3000 pods as created by the kube-burner node-density test, the following CPU requirements are validated:
  - The minimum number of reserved CPUs for control plane and worker nodes is 2 CPUs (4 hyper-threads) per NUMA node.
  - The NICs used for non-DPDK network traffic should be configured to use at most 32 RX/TX queues.
  - Nodes with large numbers of pods or other resources might require additional reserved CPUs. The remaining CPUs are available for user workloads.
    :::note

    Variations in OpenShift Container Platform configuration, workload size, and workload characteristics require additional analysis to determine the effect on the number of required CPUs for the OpenShift platform.

    :::

### Application workloads {#telco-core-application-workloads_telco-core}

Application workloads running on telco core clusters can include a mix of high performance cloud-native network functions (CNFs) and traditional best-effort or burstable pod workloads.

Guaranteed QoS scheduling is available to pods that require exclusive or dedicated use of CPUs due to performance or security requirements. Typically, pods that run high performance or latency sensitive CNFs by using user plane networking (for example, DPDK) require exclusive use of dedicated whole CPUs achieved through node tuning and guaranteed QoS scheduling. When creating pod configurations that require exclusive CPUs, be aware of the potential implications of hyper-threaded systems. Pods should request multiples of 2 CPUs when the entire core (2 hyper-threads) must be allocated to the pod.

Pods running network functions that do not require high throughput or low latency networking should be scheduled with best-effort or burstable QoS pods and do not require dedicated or isolated CPU cores.

Engineering considerations
: Plan telco core workloads and cluster resources by using the following information:

  - As of OpenShift Container Platform 4.19, `cgroup v1` is no longer supported and has been removed. All workloads must now be compatible with `cgroup v2`. For more information, see [Red Hat Enterprise Linux 9 changes in the context of Red Hat OpenShift workloads](https://www.redhat.com/en/blog/rhel-9-changes-context-red-hat-openshift-workloads).
  - CNF applications should conform to the latest version of [Red Hat Best Practices for Kubernetes](https://redhat-best-practices-for-k8s.github.io/guide/).
  - Use a mix of best-effort and burstable QoS pods as required by your applications.
    - Use guaranteed QoS pods with proper configuration of reserved or isolated CPUs in the `PerformanceProfile` CR that configures the node.
    - Guaranteed QoS Pods must include annotations for fully isolating CPUs.
    - Best effort and burstable pods are not guaranteed exclusive CPU use. Workloads can be preempted by other workloads, operating system daemons, or kernel tasks.
  - Use exec probes sparingly and only when no other suitable option is available.
    - Do not use exec probes if a CNF uses CPU pinning. Use other probe implementations, for example, `httpGet` or `tcpSocket`.
    - When you need to use exec probes, limit the exec probe frequency and quantity. The maximum number of exec probes must be kept below 10, and the frequency must not be set to less than 10 seconds.
    - You can use startup probes, because they do not use significant resources at steady-state operation. The limitation on exec probes applies primarily to liveness and readiness probes. Exec probes cause much higher CPU usage on management cores compared to other probe types because they require process forking.
  - Use pre-stop hooks to allow the application workload to perform required actions before pod disruption, such as during an upgrade or node maintenance. The hooks enable a pod to save state to persistent storage, offload traffic from a Service, or signal other Pods.

### Signaling workloads {#telco-core-signaling-workloads_telco-core}

Signaling workloads typically use SCTP, REST, gRPC, or similar TCP or UDP protocols. Signaling workloads support hundreds of thousands of transactions per second (TPS) by using a secondary multus CNI configured as MACVLAN or SR-IOV interface. These workloads can run in pods with either guaranteed or burstable QoS.

## Telco core RDS components {#telco-core-rds-components_telco-core}

The following sections describe the various OpenShift Container Platform components and configurations that you use to configure and deploy clusters to run telco core workloads.

### CPU partitioning and performance tuning {#telco-core-cpu-partitioning-and-performance-tuning_telco-core}

CPU partitioning separates sensitive workloads from general-purpose tasks, interrupts, and driver work queues to improve performance and reduce latency.

New in this release
: - There are no reference design updates in this release.

Description
: CPU partitioning improves performance and reduces latency by separating sensitive workloads from general-purpose tasks, interrupts, and driver work queues. The CPUs allocated to those auxiliary processes are referred to as **reserved** in the following sections. In a system with Hyper-Threading enabled, a CPU is one hyper-thread.

Limits and requirements
: - The operating system needs a certain amount of CPU to perform all the support tasks, including kernel networking.
    - A system with just user plane networking applications (DPDK) needs at least one core (2 hyper-threads when enabled) reserved for the operating system and the infrastructure components.
  - In a system with Hyper-Threading enabled, core sibling threads must always be in the same pool of CPUs.
  - The set of reserved and isolated cores must include all CPU cores.
  - Core 0 of each NUMA node must be included in the reserved CPU set.
  - Low latency workloads require special configuration to avoid being affected by interrupts, kernel scheduler, or other parts of the platform.

For more information, see "Creating a performance profile".

Engineering considerations
: - As of OpenShift 4.19, `cgroup v1` is no longer supported and has been removed. All workloads must now be compatible with `cgroup v2`. For more information, see [Red Hat Enterprise Linux 9 changes in the context of Red Hat OpenShift workloads](https://www.redhat.com/en/blog/rhel-9-changes-context-red-hat-openshift-workloads).
  - The minimum reserved capacity (`systemReserved`) required can be found by following the guidance in [Which amount of CPU and memory are recommended to reserve for the system in OCP 4 nodes?](https://access.redhat.com/solutions/5843241).
    - The specific values must be custom-tuned for each cluster based on its size and application workload.
    - The minimum recommended `systemReserved` memory is 11Gi for worker nodes and 30Gi for control plane nodes.
  - For schedulable control planes, the minimum recommended reserved capacity is at least 16 CPUs.
  - The actual required reserved CPU capacity depends on the cluster configuration and workload attributes.
  - The reserved CPU value must be rounded up to a full core (2 hyper-threads) alignment.
  - Changes to CPU partitioning cause the nodes contained in the relevant machine config pool to be drained and rebooted.
  - The reserved CPUs reduce the pod density, because the reserved CPUs are removed from the allocatable capacity of the OpenShift Container Platform node.
  - The real-time workload hint should be enabled for real-time capable workloads.
    - Applying the real-time `workloadHint` setting results in the `nohz_full` kernel command line parameter being applied to improve performance of high performance applications. When you apply the `workloadHint` setting, any isolated or burstable pods that do not have the `cpu-quota.crio.io: "disable"` annotation and a proper `runtimeClassName` value, are subject to CRI-O rate limiting. When you set the `workloadHint` parameter, be aware of the tradeoff between increased performance and the potential impact of CRI-O rate limiting. Ensure that required pods are correctly annotated.
  - Hardware without IRQ affinity support affects isolated CPUs. All server hardware must support IRQ affinity to ensure that pods with guaranteed CPU QoS can fully use allocated CPUs.
  - OVS dynamically manages its `cpuset` entry to adapt to network traffic needs. You do not need to reserve an additional CPU for handling high network throughput on the primary CNI.
  - If workloads running on the cluster use kernel level networking, the RX/TX queue count for the participating NICs should be set to 16 or 32 queues if the hardware permits it. Be aware of the default queue count. With no configuration, the default queue count is one RX/TX queue per online CPU; which can result in too many interrupts being allocated.
  - The irdma kernel module might result in the allocation of too many interrupt vectors on systems with high core counts. To prevent this condition the reference configuration excludes this kernel module from loading through a kernel commandline argument in the `PerformanceProfile` resource. Typically Core workloads do not require this kernel module.
  - To enable the `acpi_idle` CPUIdle driver, for example for Intel FlexRAN workloads, add `intel_idle.max_cstate=0` to the `additionalKernelArgs` list in the `PerformanceProfile` resource.
  - The `TunedPerformancePatch.yaml` file in the reference configures the `kernel.panic_on_unrecovered_nmi` sysctl parameter to enable triggering a kernel panic through BMC Non-Maskable Interrupt (NMI) on x86_64 architures. This provides a mechanism to force a kernel panic for system recovery and diagnostic purposes when nodes become unresponsive.

  :::note

  Some drivers do not deallocate the interrupts even after reducing the queue count.

  :::

### Workloads on schedulable control planes {#telco-core-workloads-on-schedulable-control-planes_telco-core}

You can enable schedulable control planes to run workloads on control plane nodes, utilizing idle CPU capacity on bare-metal machines.

Enabling workloads on control plane nodes
: You can enable schedulable control planes to run workloads on control plane nodes, utilizing idle CPU capacity on bare-metal machines for potential cost savings. This feature is only applicable to clusters with bare-metal control plane nodes.

  There are two distinct parts to this functionality:

1. Allowing workloads on control plane nodes: This feature can be configured after initial cluster installation, allowing you to enable it when you need to run workloads on those nodes.
2. Enabling workload partitioning: This is a critical isolation measure that protects the control plane from interference by regular workloads, ensuring cluster stability and reliability. Workload partitioning must be configured during the initial "day zero" cluster installation and cannot be enabled later.

If you plan to run workloads on your control plane nodes, you must first enable workload partitioning during the initial setup. You can then enable the schedulable control plane feature at a later time.

Workload characterization and limitations
: You must test and verify workloads to ensure that applications do not interfere with core cluster functions. It is recommended that you start with lightweight containers that do not heavily load the CPU or networking.

  Certain workloads are not permitted on control plane nodes due to the risk to cluster stability. This includes any workload that reconfigures kernel arguments or system global `sysctls`, as this can lead to unpredictable outcomes for the cluster.

  To ensure stability, you must adhere to the following:

  - Make sure all non-trivial workloads have memory limits defined. This protects the control plane in case of a memory leak.
  - Avoid excessively loading reserved CPUs, for example, by heavy use of exec probes.
  - Avoid heavy kernel-based networking usage, as it can increase reserved CPU load through software networking components like OVS.

NUMA Resources Operator support
: As part of this enablement, the NUMA Resources Operator is supported on control plane nodes. The functional behavior of the NUMA Resources Operator remains the same, but its use on control plane nodes is explicitly permitted.

**Additional resources**

- [Creating a performance profile](/docs/scalability_and_performance/cnf-tuning-low-latency-nodes-with-perf-profile#cnf-create-performance-profiles_cnf-tuning-low-latency-nodes-with-perf-profile)
- [Configuring host firmware for low latency and high performance](/docs/edge_computing/ztp-reference-cluster-configuration-for-vdu#ztp-du-configuring-host-firmware-requirements_sno-configure-for-vdu)

### Service Mesh {#telco-core-service-mesh_telco-core}

Telco core cloud-native functions (CNFs) typically require a Service Mesh implementation.

Description
: Telco core cloud-native functions (CNFs) typically require a Service Mesh implementation. Specific Service Mesh features and performance requirements are dependent on the application. The selection of Service Mesh implementation and configuration is outside the scope of this documentation. The implementation must account for the impact of Service Mesh on cluster resource usage and performance, including additional latency introduced in pod networking.

**Additional resources**

- [About OpenShift Service Mesh](/docs/service_mesh/v2x/ossm-about#ossm-about)

### Networking {#telco-core-networking_telco-core}

The following diagram describes the telco core reference design networking configuration.

**Figure 2. Telco core reference design networking configuration**

![Overview of the telco core reference design networking configuration](/images/openshift-telco-core-rds-networking.png)

:::note

The diagram shows the recommended network topology, where bonding uses ports from the same dual-port NIC. Bonding ports across different NICs is not recommended.

:::

New in this release
: - Per-interface IP forwarding
  - Support for alt-names on network interfaces
  - Host firewall rules for multi-node clusters
  - MetalLB `ConfigurationStatus` CRD
  - Extended BGP/L2 advertisements with service selectors
  - Support for Intel E830 Jasper Beach in telco core

:::note

If you have custom `FRRConfiguration` CRs in the `metallb-system` namespace, you must move them under the `openshift-network-operator` namespace.

:::

Description
: - The cluster is configured for dual-stack IP (IPv4 and IPv6).
  - The validated physical network configuration consists of two dual-port NICs. One NIC is shared among the primary CNI (OVN-Kubernetes) and IPVLAN and MACVLAN traffic, while the second one is dedicated to SR-IOV VF-based pod traffic.
  - A Linux bonding interface (`bond0`) is created in active-active IEEE 802.3ad LACP mode with the two NIC ports attached. The top-of-rack networking equipment must support and be configured for multi-chassis link aggregation (mLAG) technology.
  - VLAN interfaces are created on top of `bond0`, including for the primary CNI.
  - Bond and VLAN interfaces are created at cluster install time during the network configuration stage of the installation. Except for the `vlan0` VLAN used by the primary CNI, all other VLANs can be created during Day 2 activities with the Kubernetes NMstate Operator.
  - MACVLAN and IPVLAN interfaces are created with their corresponding CNIs. They do not share the same base interface. For more information, see "Cluster Network Operator".
  - SR-IOV VFs are managed by the SR-IOV Network Operator.
  - To ensure consistent source IP addresses for pods behind a LoadBalancer Service, configure an `EgressIP` CR and specify the `podSelector` parameter. EgressIP is further discussed in the "Cluster Network Operator" section.
  - You can implement service traffic separation by doing the following:
    1. Configure VLAN interfaces and specific kernel IP routes on the nodes using `NodeNetworkConfigurationPolicy` CRs.
    2. Create a MetalLB `BGPPeer` CR for each VLAN to establish peering with the remote BGP router.
    3. Define a MetalLB `BGPAdvertisement` CR to specify which IP address pools should be advertised to a selected list of `BGPPeer` resources. The following diagram illustrates how specific service IP addresses are advertised externally through specific VLAN interfaces. Services routes are defined in `BGPAdvertisement` CRs and configured with values for `IPAddressPool1` and `BGPPeer1` fields.
  - Host-level per-interface IP forwarding can be enabled or disabled by using `net.ipv4.conf.<interface>.forwarding` through the install network configuration (Day 1) or the Kubernetes NMState Operator (Day 2).

**Figure 3. Telco core reference design MetalLB service separation**

![Telco core reference design MetalLB service separation](/images/openshift-telco-core-rds-metallb-service-separation.png)

**Additional resources**

- [Understanding networking](/docs/networking/networking_overview/understanding-networking#understanding-networking)

#### Cluster Network Operator {#telco-core-cluster-network-operator_telco-core}

The Cluster Network Operator (CNO) deploys and manages the cluster network components including the default OVN-Kubernetes network plugin during cluster installation.

New in this release
: - There are no reference design updates in this release.

Description
: The Cluster Network Operator (CNO) deploys and manages the cluster network components including the default OVN-Kubernetes network plugin during cluster installation. The CNO allows configuration of primary interface MTU settings, OVN gateway modes to use node routing tables for pod egress, and additional secondary networks such as MACVLAN.

  In support of network traffic separation, multiple network interfaces are configured through the CNO. Traffic steering to these interfaces is configured through static routes applied by using the NMState Operator. To ensure that pod traffic is properly routed, OVN-K is configured with the `routingViaHost` option enabled. This setting uses the kernel routing table and the applied static routes rather than OVN for pod egress traffic.

  The Whereabouts CNI plugin is used to provide dynamic IPv4 and IPv6 addressing for additional pod network interfaces without the use of a DHCP server.

Limits and requirements
: - OVN-Kubernetes is required for IPv6 support.
  - Large MTU cluster support requires connected network equipment to be set to the same or larger value. MTU size up to 8900 is supported.
  - MACVLAN and IPVLAN cannot co-locate on the same main interface due to their reliance on the same underlying kernel mechanism, specifically the `rx_handler`. This handler allows a third-party module to process incoming packets before the host processes them, and only one such handler can be registered per network interface. Since both MACVLAN and IPVLAN need to register their own `rx_handler` to function, they conflict and cannot coexist on the same interface. Review the source code for more details:
  - [linux/v6.10.2/source/drivers/net/ipvlan/ipvlan_main.c#L82](https://elixir.bootlin.com/linux/v6.10.2/source/drivers/net/ipvlan/ipvlan_main.c#L82)
  - [linux/v6.10.2/source/drivers/net/macvlan.c#L1260](https://elixir.bootlin.com/linux/v6.10.2/source/drivers/net/macvlan.c#L1260)

- Alternative NIC configurations include splitting the shared NIC into multiple NICs or using a single dual-port NIC, though they have not been tested and validated.
- Clusters with single-stack IP configuration are not validated.
- EgressIP
  - EgressIP failover time depends on the `reachabilityTotalTimeoutSeconds` parameter in the `Network` CR. This parameter determines the frequency of probes used to detect when the selected egress node is unreachable. The recommended value of this parameter is `1` second.
  - When EgressIP is configured with multiple egress nodes, the failover time is expected to be on the order of seconds or longer.
  - On nodes with additional network interfaces EgressIP traffic will egress through the interface on which the EgressIP address has been assigned. For more information, see "Configuring an egress IP address".
- Pod-level SR-IOV bonding mode must be set to `active-backup` and a value in `miimon` must be set (`100` is recommended).

Engineering considerations
: - Pod egress traffic is managed by kernel routing table using the `routingViaHost` option. Appropriate static routes must be configured in the host.

**Additional resources**

- [Cluster Network Operator](/docs/networking/networking_operators/cluster-network-operator#nw-cluster-network-operator_cluster-network-operator)
- [Configuring an egress IP address](/docs/networking/ovn_kubernetes_network_provider/configuring-egress-ips-ovn#configuring-egress-ips-ovn)

#### Load balancer {#telco-core-load-balancer_telco-core}

MetalLB is a load-balancer implementation for bare metal Kubernetes clusters that uses standard routing protocols.

New in this release
: - There are no reference design updates in this release.

:::warning

If you have custom `FRRConfiguration` CRs in the `metallb-system` namespace, you must move them under the `openshift-network-operator` namespace.

:::

Description
: MetalLB is a load-balancer implementation for bare metal Kubernetes clusters that uses standard routing protocols. It enables a Kubernetes service to get an external IP address which is also added to the host network for the cluster. The MetalLB Operator deploys and manages the lifecycle of a MetalLB instance in a cluster. Some use cases might require features not available in MetalLB, such as stateful load balancing. Where necessary, you can use an external third party load balancer. Selection and configuration of an external load balancer is outside the scope of this specification. When an external third-party load balancer is used, the integration effort must include enough analysis to ensure all performance and resource utilization requirements are met.

Limits and requirements
: - Stateful load balancing is not supported by MetalLB. An alternate load balancer implementation must be used if this is a requirement for workload CNFs.
  - You must ensure that the external IP address is routable from clients to the host network for the cluster.

Engineering considerations
: - MetalLB is used in BGP mode only for telco core use models.
  - For telco core use models, MetalLB is supported only with the OVN-Kubernetes network provider used in local gateway mode. See `routingViaHost` in "Cluster Network Operator".
  - BGP configuration in MetalLB is expected to vary depending on the requirements of the network and peers.
    - You can configure address pools with variations in addresses, aggregation length, auto assignment, and so on.
    - MetalLB uses BGP for announcing routes only. Only the `transmitInterval` and `minimumTtl` parameters are relevant in this mode. Other parameters in the BFD profile should remain close to the defaults as shorter values can lead to false negatives and affect performance.

**Additional resources**

- [When to use MetalLB](/docs/networking/networking_operators/metallb-operator/about-metallb#nw-metallb-when-metallb_about-metallb-and-metallb-operator)

#### SR-IOV {#telco-core-sr-iov_telco-core}

SR-IOV enables physical functions (PFs) to be divided into multiple virtual functions (VFs) for higher throughput performance while keeping pods isolated.

New in this release
: - There are no reference design updates in this release.

Description
: SR-IOV enables physical functions (PFs) to be divided into multiple virtual functions (VFs). VFs can then be assigned to multiple pods to achieve higher throughput performance while keeping the pods isolated. The SR-IOV Network Operator provisions and manages SR-IOV CNI, network device plugin, and other components of the SR-IOV stack.

Limits and requirements
: - Only certain network interfaces are supported. See "Supported devices" for more information.
  - Enabling SR-IOV and IOMMU: the SR-IOV Network Operator automatically enables IOMMU on the kernel command line.
  - SR-IOV VFs do not receive link state updates from the PF. If a link down detection is required, it must be done at the protocol level.
  - `MultiNetworkPolicy` CRs can be applied to `netdevice` networks only. This is because the implementation uses iptables, which cannot manage vfio interfaces.

Engineering considerations
: - SR-IOV interfaces in `vfio` mode are typically used to enable additional secondary networks for applications that require high throughput or low latency.
  - The `SriovOperatorConfig` CR must be explicitly created. This CR is included in the reference configuration policies, which causes it to be created during initial deployment.
  - NICs that do not support firmware updates with UEFI secure boot or kernel lockdown must be preconfigured with sufficient virtual functions (VFs) enabled to support the number of VFs required by the application workload. For Mellanox NICs, you must disable the Mellanox vendor plugin in the SR-IOV Network Operator. For more information see, "Configuring an SR-IOV network device".
  - To change the MTU value of a VF after the pod has started, do not configure the `SriovNetworkNodePolicy` MTU field. Instead, use the Kubernetes NMState Operator to set the MTU of the related PF.

**Additional resources**

- [About Single Root I/O Virtualization (SR-IOV) hardware networks](/docs/networking/hardware_networks/about-sriov#about-sriov)
- [Red Hat certified hardware (Red Hat Ecosystem Catalog)](https://catalog.redhat.com/en/hardware)
- [Configuring the SR-IOV Network Operator on Mellanox cards when Secure Boot is enabled](/docs/networking/hardware_networks/configuring-sriov-device#nw-sriov-nic-mlx-secure-boot_configuring-sriov-device)

#### NMState Operator {#telco-core-nmstate-operator_telco-core}

The Kubernetes NMState Operator provides a Kubernetes API for performing state-driven network configuration across cluster nodes.

New in this release
: - There are no reference design updates in this release.

Description
: The Kubernetes NMState Operator provides a Kubernetes API for performing state-driven network configuration across cluster nodes. It enables network interface configurations, static IPs and DNS, VLANs, trunks, bonding, static routes, MTU, and enabling promiscuous mode on the secondary interfaces. The cluster nodes periodically report on the state of each node’s network interfaces to the API server.

Limits and requirements
: Not applicable

Engineering considerations
: - Initial networking configuration is applied using `NMStateConfig` content in the installation CRs. The NMState Operator is used only when required for network updates.
  - When SR-IOV virtual functions are used for host networking, the NMState Operator (via `nodeNetworkConfigurationPolicy` CRs) is used to configure VF interfaces, such as VLANs and MTU.

**Additional resources**

- [Kubernetes NMState Operator](/docs/networking/networking_operators/k8s-nmstate-about-the-k8s-nmstate-operator#k8s-nmstate-about-the-k8s-nmstate-operator)

### Logging {#telco-core-logging_telco-core}

The Cluster Logging Operator enables collection and shipping of logs off the node for remote archival and analysis.

New in this release
: - There are no reference design updates in this release.

Description
: The Cluster Logging Operator enables collection and shipping of logs off the node for remote archival and analysis. The reference configuration uses Kafka to ship audit and infrastructure logs to a remote archive.

Limits and requirements
: Not applicable

Engineering considerations
: - The impact of cluster CPU use is based on the number or size of logs generated and the amount of log filtering configured.
  - The reference configuration does not include shipping of application logs. The inclusion of application logs in the configuration requires you to evaluate the application logging rate and have sufficient additional CPU resources allocated to the reserved set.

**Additional resources**

- [Logging 6.0](https://docs.openshift.com/container-platform/4.17/observability/logging/logging-6.0/log6x-about.html)

### Power Management {#telco-core-power-management_telco-core}

Use the performance profile to configure clusters with high power mode, low power mode, or mixed mode depending on workload latency sensitivity.

New in this release
: - There are no reference design updates in this release.

Description
: Use the Performance profile to configure clusters with high power mode, low power mode, or mixed mode. The choice of power mode depends on the characteristics of the workloads running on the cluster, particularly how sensitive they are to latency. Configure the maximum latency for a low-latency pod by using the per-pod power management C-states feature.

Limits and requirements
: - Power configuration relies on appropriate BIOS configuration, for example, enabling C-states and P-states. Configuration varies between hardware vendors.

Engineering considerations
: - Latency: To ensure that latency-sensitive workloads meet requirements, you require a high-power or a per-pod power management configuration. Per-pod power management is only available for Guaranteed QoS pods with dedicated pinned CPUs.

**Additional resources**

- [performance.openshift.io/v2 API reference](/docs/rest_api/node_apis/performanceprofile-performance-openshift-io-v2#spec-workloadhints)
- [Configuring power saving for nodes that run colocated high and low priority workloads](/docs/scalability_and_performance/cnf-tuning-low-latency-nodes-with-perf-profile#cnf-configuring-power-saving-for-nodes_cnf-tuning-low-latency-nodes-with-perf-profile)

### Storage {#telco-core-storage_telco-core}

Cloud native storage services can be provided by OpenShift Data Foundation or other third-party solutions for telco core clusters.

New in this release
: - There are no reference design updates in this release.

Description
: Cloud native storage services can be provided by OpenShift Data Foundation or other third-party solutions.

  OpenShift Data Foundation is a Red Hat Ceph Storage based software-defined storage solution for containers. It provides block storage, file system storage, and on-premise object storage, which can be dynamically provisioned for both persistent and non-persistent data requirements. Telco core applications require persistent storage.

  :::note

  All storage data might not be encrypted in flight. To reduce risk, isolate the storage network from other cluster networks. The storage network must not be reachable, or routable, from other cluster networks. Only nodes directly attached to the storage network should be allowed to gain access to it.

  :::

**Additional resources**

- [Red Hat OpenShift Data Foundation](/docs/storage/persistent_storage/persistent-storage-ocs#red-hat-openshift-data-foundation)

#### OpenShift Data Foundation {#telco-core-openshift-data-foundation_telco-core}

OpenShift Data Foundation is a software-defined storage service for containers that provides block storage, file system storage, and on-premise object storage.

New in this release
: - There are no reference design updates in this release.

Description
: OpenShift Data Foundation is a software-defined storage service for containers. OpenShift Data Foundation can be deployed in one of two modes:

  - Internal mode, where OpenShift Data Foundation software components are deployed as software containers directly on the OpenShift cluster nodes, together with other containerized applications.
  - External mode, where OpenShift Data Foundation is deployed on a dedicated storage cluster, which is usually a separate Red Hat Ceph Storage cluster running on Red Hat Enterprise Linux. These storage services are running externally to the application workload cluster.

For telco core clusters, storage support is provided by OpenShift Data Foundation storage services running in external mode, for several reasons:

- Separating dependencies between OpenShift Container Platform and Ceph operations allows for independent OpenShift Container Platform and OpenShift Data Foundation updates.
- Separation of operations functions for the Storage and OpenShift Container Platform infrastructure layers, is a typical customer requirement for telco core use cases.
- External Red Hat Ceph Storage clusters can be re-used by multiple OpenShift Container Platform clusters deployed in the same region.

OpenShift Data Foundation supports separation of storage traffic using secondary CNI networks.

Limits and requirements
: - In an IPv4/IPv6 dual-stack networking environment, OpenShift Data Foundation uses IPv4 addressing. For more information, see [IPv6 support](https://docs.redhat.com/en/documentation/red_hat_openshift_data_foundation/latest/html/planning_your_deployment/network-requirements_rhodf#ipv6-support_rhodf).

Engineering considerations
: - OpenShift Data Foundation network traffic should be isolated from other traffic on a dedicated network, for example, by using VLAN isolation.
  - Workload requirements must be scoped before attaching multiple OpenShift Container Platform clusters to an external OpenShift Data Foundation cluster to ensure enough throughput, bandwidth, and performance KPIs.

#### Additional storage solutions {#telco-core-additional-storage-solutions_telco-core}

You can use other storage solutions to provide persistent storage for telco core clusters. The configuration and integration of these solutions is outside the scope of the reference design specifications (RDS).

Integration of the storage solution into the telco core cluster must include proper sizing and performance analysis to ensure the storage meets overall performance and resource usage requirements.

## Telco core deployment components {#telco-reference-core-deployment-components_telco-core}

The following sections describe the various OpenShift Container Platform components and configurations that you use to configure the hub cluster with Red Hat Advanced Cluster Management (RHACM).

### Red Hat Advanced Cluster Management {#telco-core-red-hat-advanced-cluster-management_telco-core}

RHACM provides Multi Cluster Engine (MCE) installation and ongoing lifecycle management for deployed clusters.

New in this release
: - There are no reference design updates in this release.

Description
: RHACM provides Multi Cluster Engine (MCE) installation and ongoing GitOps ZTP lifecycle management for deployed clusters. You manage cluster configuration and upgrades declaratively by applying `Policy` custom resources (CRs) to clusters during maintenance windows.

  You apply policies with the RHACM policy controller as managed by TALM. Configuration, upgrades, and cluster status are managed through the policy controller.

  When installing managed clusters, RHACM applies labels and initial ignition configuration to individual nodes in support of custom disk partitioning, allocation of roles, and allocation to machine config pools. You define these configurations with `SiteConfig` or `ClusterInstance` CRs.

Limits and requirements
: - Hub cluster sizing is discussed in [Sizing your cluster](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.15/html-single/install/index#sizing-your-cluster).
  - RHACM scaling limits are described in [Performance and Scalability](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.15/html-single/install/index#performance-and-scalability).

Engineering considerations
: - When managing multiple clusters with unique content per installation, site, or deployment, using RHACM hub templating is strongly recommended. With RHACM hub templating, you can apply a consistent set of policies to clusters while providing unique values per installation.

**Additional resources**

- [Using GitOps ZTP to provision clusters at the network far edge](/docs/edge_computing/ztp-deploying-far-edge-clusters-at-scale#about-ztp_ztp-deploying-far-edge-clusters-at-scale)
- [Red Hat Advanced Cluster Management for Kubernetes](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes)

### Topology Aware Lifecycle Manager {#telco-core-topology-aware-lifecycle-manager_telco-core}

TALM manages how changes are rolled out to managed clusters in the network.

New in this release
: - There are no reference design updates in this release.

Description
: TALM is an Operator that runs only on the hub cluster. TALM manages how changes including cluster and Operator upgrades, configurations, and so on, are rolled out to managed clusters in the network. TALM has the following core features:

  - Provides sequenced updates of cluster configurations and upgrades (OpenShift Container Platform and Operators) as defined by cluster policies.
  - Provides for deferred application of cluster updates.
  - Supports progressive rollout of policy updates to sets of clusters in user configurable batches.
  - Allows for per-cluster actions by adding `ztp-done` or similar user-defined labels to clusters.

Limits and requirements
: - Supports concurrent cluster deployments in batches of 400

Engineering considerations
: - Only policies with the `ran.openshift.io/ztp-deploy-wave` annotation are applied by TALM during initial cluster installation.
  - Any policy can be remediated by TALM under control of a user created `ClusterGroupUpgrade` CR.
  - Set the `MachineConfigPool` (`mcp`) CR `paused` field to true during a cluster upgrade maintenance window and set the `maxUnavailable` field to the maximum tolerable value. This prevents multiple cluster node reboots during upgrade, which results in a shorter overall upgrade. When you unpause the `mcp` CR, all the configuration changes are applied with a single reboot.

  :::note

  During installation, custom `mcp` CRs can be paused along with setting `maxUnavailable` to 100% to improve installation times.

  :::

  - You can orchestrate upgrades for OpenShift Container Platform, Day 2 OLM operators, and custom configurations using a ClusterGroupUpgrade (CGU) CR that defines the required policies.
    - An EUS to EUS upgrade can be orchestrated using chained CGU CRs.
    - Control of MCP pause can be managed through policy in the CGU CRs for a full control plane and worker node rollout of upgrades.
    - For more information, see "Performing an EUS-to-EUS update for telco core clusters".

**Additional resources**

- [Updating managed clusters with the Topology Aware Lifecycle Manager](/docs/edge_computing/cnf-talm-for-cluster-upgrades#cnf-talm-for-cluster-updates)

### GitOps Operator and GitOps ZTP plugins {#telco-core-gitops-operator-and-ztp-plugins_telco-core}

The GitOps Operator provides a GitOps-driven infrastructure for managing cluster deployment and configuration.

New in this release
: - There are no reference design updates in this release.

Description
: The GitOps Operator provides a GitOps driven infrastructure for managing cluster deployment and configuration. Cluster definitions and configuration are maintained in a Git repository.

  The SiteConfig Operator generates installation CRs from `ClusterInstance` CRs.

  :::warning

  From OpenShift Container Platform 4.21, you must use `ClusterInstance` CRs and the SiteConfig Operator to define managed cluster installations. With this release, support for defining managed cluster installations with the `SiteConfig` CR is removed.

  :::

  GitOps ZTP plugins provide support for automatically wrapping configuration CRs in policies based on RHACM `PolicyGenerator` CRs.

  You should structure the Git repository according to the release version, with all necessary artifacts added to the repository, such as the `ClusterInstance`, `PolicyGenerator`, and supporting reference CRs. This structure enables deploying and managing multiple versions of the OpenShift Container Platform and configuration versions to clusters simultaneously and through upgrades.

  The recommended Git structure keeps reference CRs in a directory separate from customer or partner provided content. This means that you can import reference updates by simply overwriting existing content. Customer or partner supplied CRs can be provided in a parallel directory to the reference CRs for easy inclusion in the generated configuration policies.

Limits and requirements
: - Each ArgoCD application supports up to 1000 nodes on a hub cluster conforming to the Hub RDS. Multiple ArgoCD applications can be used to achieve the maximum number of clusters supported by a single hub cluster.
  - Use the `extraManifestsRefs` field in the `ClusterInstance` CR to reference `ConfigMap` resources that contain additional manifests to apply at install time.

Engineering considerations
: - To avoid confusion or unintentional overwriting when updating content, you should use unique and distinguishable names for custom CRs in the `reference-crs/` directory under core-overlay and extra manifests in git.
  - The `ClusterInstance` CR allows multiple `ConfigMap` references in the `extraManifestsRefs` field for additional install-time manifests.

**Additional resources**

- [Preparing the GitOps ZTP site configuration repository for version independence](/docs/edge_computing/ztp-preparing-the-hub-cluster#ztp-preparing-the-ztp-git-repository-ver-ind_ztp-preparing-the-hub-cluster)
- [Adding custom content to the GitOps ZTP pipeline](/docs/edge_computing/policygentemplate_for_ztp/ztp-advanced-policy-config#ztp-adding-new-content-to-gitops-ztp_ztp-advanced-policy-config)

### Agent-based Installer {#telco-core-agent-based-installer_telco-core}

The Agent-based Installer provides an installation method for OpenShift Container Platform in environments without existing deployment infrastructure.

New in this release
: - There are no reference design updates in this release.

Description
: The recommended method for Telco Core cluster installation is using Red Hat Advanced Cluster Management. The Agent Based Installer (ABI) is a separate installation flow for Openshift in environments without existing infrastructure for running cluster deployments. Use the ABI to install OpenShift Container Platform on bare-metal servers without requiring additional servers or VMs for managing the installation, but does not provide ongoing lifecycle management, monitoring or automations. The ABI can be run on any system for example, from a laptop to generate an ISO installation image. The ISO is used as the installation media for the cluster control plane nodes. You can monitor the progress by using the ABI from any system with network connectivity to the control plane node’s API interfaces.

  ABI supports the following:

  - Installation from declarative CRs
  - Installation in disconnected environments
  - No additional servers required to support installation, for example, the bastion node is no longer needed

Limits and requirements
: - Disconnected installation requires a registry with all required content mirrored and reachable from the installed host.

Engineering considerations
: - Networking configuration should be applied as NMState configuration during installation as opposed to Day 2 configuration using the NMState Operator.

**Additional resources**

- [Installing an OpenShift Container Platform cluster with the Agent-based Installer](/docs/installing/installing_with_agent_based_installer/installing-with-agent-based-installer#installing-with-agent-based-installer)

## Monitoring {#telco-core-monitoring_telco-core}

The Cluster Monitoring Operator (CMO) is included by default in OpenShift Container Platform and provides monitoring for the platform components and optionally user projects.

New in this release
: - There are no reference design updates in this release.

Description
: The Cluster Monitoring Operator (CMO) is included by default in OpenShift Container Platform and provides monitoring (metrics, dashboards, and alerting) for the platform components and optionally user projects. You can customize the default log retention period, custom alert rules, and so on.

  Configuration of the monitoring stack is done through a single string value in the cluster-monitoring-config ConfigMap. The reference tuning merges content from two requirements:

  - Prometheus configuration is extended to forward alerts to the Red Hat Advanced Cluster Management (RHACM) hub cluster for alert aggregation. If required, you can extend this configuration to forward alerts to additional locations.
  - Prometheus retention period is reduced from the default. The primary metrics storage is typically external to the cluster. Metrics storage on the core cluster is typically a backup to that central store and available for local troubleshooting purposes. You can add a `remoteWrite` endpoint to the Prometheus configuration to directly forward metrics to the central store.

In addition to the default configuration, telco core clusters typically configure pod CPU and memory metrics and alerts for user workloads.

Engineering considerations
: - The Prometheus retention period is specified by the user. The value used is a tradeoff between operational requirements for maintaining historical data on the cluster against CPU and storage resources. Longer retention periods increase the need for storage and require additional CPU to manage the indexing of data.

**Additional resources**

- [About OpenShift Container Platform monitoring](https://docs.redhat.com/en/documentation/monitoring_stack_for_red_hat_openshift/latest/html/about_monitoring/about-ocp-monitoring)

## Scheduling {#telco-core-scheduling_telco-core}

The scheduler is a cluster-wide component responsible for selecting the right node for a given workload.

New in this release
: - There are no reference design updates in this release.

Description
: The scheduler is a cluster-wide component responsible for selecting the right node for a given workload. It is a core part of the platform and does not require any specific configuration in the common deployment scenarios. However, there are few specific use cases described in the following section.

  NUMA-aware scheduling can be enabled through the NUMA Resources Operator. For more information, see "Scheduling NUMA-aware workloads".

Limits and requirements
: - The default scheduler does not understand the NUMA locality of workloads. It only knows about the sum of all free resources on a worker node. This might cause workloads to be rejected when scheduled to a node with the topology manager policy set to single-numa-node or restricted. For more information, see "Topology Manager policies"..
    - For example, consider a pod requesting 6 CPUs and being scheduled to an empty node that has 4 CPUs per NUMA node. The total allocatable capacity of the node is 8 CPUs. The scheduler places the pod on the empty node. The node local admission fails, as there are only 4 CPUs available in each of the NUMA nodes.
  - All clusters with multi-NUMA nodes are required to use the NUMA Resources Operator. See "Installing the NUMA Resources Operator" for more information. Use the `machineConfigPoolSelector` field in the `KubeletConfig` CR to select all nodes where NUMA aligned scheduling is required.
  - All machine config pools must have consistent hardware configuration. For example, all nodes are expected to have the same NUMA zone count.

Engineering considerations
: - Pods might require annotations for correct scheduling and isolation. For more information about annotations, see "CPU partitioning and performance tuning".
  - You can configure SR-IOV virtual function NUMA affinity to be ignored during scheduling by using the excludeTopology field in `SriovNetworkNodePolicy` CR.

**Additional resources**

- [Installing the NUMA Resources Operator](/docs/scalability_and_performance/cnf-numa-aware-scheduling#installing-the-numa-resources-operator_numa-aware)
- [Scheduling NUMA-aware workloads](/docs/scalability_and_performance/cnf-numa-aware-scheduling#cnf-numa-aware-scheduling)
- [Topology Manager policies](/docs/scalability_and_performance/using-cpu-manager#topology-manager-policies_using-cpu-manager-and-topology-manager)

## Node Configuration {#telco-core-node-configuration_telco-core}

Node configuration for telco core clusters includes additional kernel modules, container mount namespace settings, and kdump configuration.

New in this release
: - There are no reference design updates in this release.

Limits and requirements
: - Analyze additional kernel modules to determine impact on CPU load, system performance, and ability to meet KPIs.

  **Additional kernel modules**

<table>
<thead>
<tr>
  <th>Feature</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Additional kernel modules</td>
  <td>Install the following kernel modules by using <code>MachineConfig</code> CRs to provide extended kernel functionality to CNFs.<br><br><ul><li>sctp</li><li>ip_gre</li><li>nf_tables</li><li>nf_conntrack</li><li>nft_ct</li><li>nft_limit</li><li>nft_log</li><li>nft_nat</li><li>nft_chain_nat</li><li>nf_reject_ipv4</li><li>nf_reject_ipv6</li><li>nfnetlink_log</li></ul></td>
</tr>
<tr>
  <td>Container mount namespace hiding</td>
  <td>Reduce the frequency of kubelet housekeeping and eviction monitoring to reduce CPU usage. Creates a container mount namespace, visible to kubelet/CRI-O, to reduce system mount scanning overhead.</td>
</tr>
<tr>
  <td>Kdump enable</td>
  <td>Optional configuration (enabled by default)</td>
</tr>
</tbody>
</table>

**Additional resources**

- [Automatic kernel crash dumps with kdump](/docs/edge_computing/ztp-reference-cluster-configuration-for-vdu#ztp-sno-du-enabling-kdump_sno-configure-for-vdu)
- [Optimizing CPU usage with mount namespace encapsulation](/docs/scalability_and_performance/optimization/optimizing-cpu-usage#optimizing-cpu-usage)

### Host firmware and boot loader configuration {#telco-core-host-firmware-and-boot-loader-configuration_telco-core}

Configure host firmware and boot loader settings to support telco core cluster requirements.

New in this release
: - There are no reference design updates in this release.

Engineering considerations
: - Enabling secure boot is the recommended configuration.

  :::note

  When secure boot is enabled, only signed kernel modules are loaded by the kernel. Out-of-tree drivers are not supported.

  :::

### Kubelet Settings {#telco-core-kubelet-settings_telco-core}

Some CNF workloads make use of sysctls which are not in the list of system-wide safe `sysctls`. Generally network sysctls are namespaced and can be enabled by using the `kubeletconfig.experimental` annotation in the PerformanceProfile as a string of JSON in the form `allowedUnsafeSysctls`.

Additionally, the `systemReserved` memory can be configured through the same `kubeletconfig.experimental` annotation to reserve memory for system daemons and kernel processes.

```yaml title="Example snippet showing allowedUnsafeSysctls and systemReserved"
apiVersion: performance.openshift.io/v2
kind: PerformanceProfile
metadata:
  name: {{ .metadata.name }}
  annotations:
    # allowedUnsafeSysctls: some pods want the kernel stack to ignore IPv6 router Advertisement.
    # systemReserved: when used, it should be tailored for each environment.
    kubeletconfig.experimental: |
      {
       "allowedUnsafeSysctls":["net.ipv6.conf.all.accept_ra"],
       "systemReserved":{"memory":"11Gi"}
      }
```

:::note

Although these are namespaced they may allow a pod to consume memory or other resources beyond any limits specified in the pod description. You must ensure that these `sysctls` do not exhaust platform resources.

:::

## Disconnected environment {#telco-core-disconnected-environment_telco-core}

Telco core clusters are expected to be installed in networks without direct access to the internet.

New in this release
: - Added requirement to use the `oc mirror` plugin v2 for mirroring image signatures in disconnected environments.

Description
: Telco core clusters are expected to be installed in networks without direct access to the internet. All container images needed to install, configure, and operate the cluster must be available in a disconnected registry. This includes OpenShift Container Platform images, Day 2 OLM Operator images, and application workload images. The use of a disconnected environment provides multiple benefits, including:

  - Security - limiting access to the cluster
  - Curated content - the registry is populated based on curated and approved updates for clusters

Limits and requirements
: - A unique name is required for all custom `CatalogSource` resources. Do not reuse the default catalog names.

Engineering considerations
: - A valid time source must be configured as part of cluster installation.
  - In OpenShift Container Platform 4.22 and later, pulling OpenShift images from a disconnected mirror registry requires copying the image signatures into that registry during the mirroring process. The `oc adm mirror` command does not mirror signatures and must not be used. Instead, use the `oc mirror` plugin v2 to ensure signatures are properly mirrored.

**Additional resources**

- [Mirroring images for a disconnected installation by using the oc-mirror plugin v2](/docs/disconnected/about-installing-oc-mirror-v2#about-installing-oc-mirror-v2)
- [About cluster updates in a disconnected environment](/docs/disconnected/updating#about-disconnected-updates)
- [Using sysctl in containers](/docs/nodes/containers/nodes-containers-sysctls#nodes-containers-sysctls)

## Security {#telco-core-security_telco-core}

Telco core clusters require hardening against multiple attack vectors through various security features and configurations.

New in this release
: - Starting with OpenShift Container Platform 4.22, administrators can use the `oc commatrix` plugin to automatically generate `MachineConfig` CRs with a default set of nftables firewall rules. The generated `MachineConfig` CRs can then be added to the cluster configuration to improve security.

Description
: Telco customers are security conscious and require clusters to be hardened against multiple attack vectors. In OpenShift Container Platform, there is no single component or feature responsible for securing a cluster. Described below are various security oriented features and configurations for the use models covered in the telco core RDS.

  - SecurityContextConstraints (SCC): All workload pods should be run with `restricted-v2` or `restricted` SCC.
  - Seccomp: All pods should run with the `RuntimeDefault` (or stronger) seccomp profile.
  - Rootless DPDK pods: Many user-plane networking (DPDK) CNFs require pods to run with root privileges. With this feature, a conformant DPDK pod can be run without requiring root privileges. Rootless DPDK pods create a tap device in a rootless pod that injects traffic from a DPDK application to the kernel.
  - Storage: The storage network should be isolated and non-routable to other cluster networks. See the "Storage" section for additional details.

  Security-conscious cluster administrators can configure nftables rules to restrict inbound network flows. See [the OpenShift Container Platform documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html/installation_configuration/configuring-firewall.html#network-commatrix-plugin-intro_configuring-firewall) for a supported method for implementing custom nftables firewall rules in OpenShift Container Platform cluster nodes with the `oc commatrix` plugin. You can add the generated `MachineConfig` CRs to the cluster configuration at install time as extra manifests. When the cluster is managed under a Hub RDS compliant RHACM cluster the content of the `MachineConfig` resource is maintained across the life of the cluster through a `Policy` resource.

  It is crucial to carefully consider the operational implications before deploying this method, including:

  - Early application: The rules are applied at boot time, before the network is fully operational. Ensure the rules do not inadvertently block essential services required during the boot process.
  - Risk of misconfiguration: Errors in your custom rules can lead to unintended consequences, potentially leading to performance impact or blocking legitimate traffic or isolating nodes. Thoroughly test your rules in a non-production environment before deploying them to your main cluster.
  - External endpoints: OpenShift Container Platform requires access to external endpoints to function. For more information about the firewall allowlist, see "Configuring your firewall for OpenShift Container Platform". Ensure that cluster nodes are permitted access to those endpoints.
  - Node reboot: Unless node disruption policies are configured, applying the `MachineConfig` CR with the required firewall settings causes a node reboot. Be aware of this impact and schedule a maintenance window accordingly.

  :::note

  Node disruption policies are available in OpenShift Container Platform 4.17 and later.

  :::

  - Network flow matrix: For more information about managing ingress traffic, see "Network flow matrix". You can restrict ingress traffic to essential flows to improve network security. The matrix provides insights into base cluster services but excludes traffic generated by Day 2 Operators.
  - Cluster version updates and upgrades: Exercise caution when updating or upgrading OpenShift Container Platform clusters. Recent changes to the platform’s firewall requirements might require adjustments to network port permissions. While the documentation provides guidelines, note that these requirements can evolve over time. To minimize disruptions, you should test any updates or upgrades in a staging environment before applying them in production. This helps you to identify and address potential compatibility issues related to firewall configuration changes.

Limits and requirements
: - Rootless DPDK pods requires the following additional configuration:
    - Configure the `container_t` SELinux context for the tap plugin.
    - Enable the `container_use_devices` SELinux boolean for the cluster host.

Engineering considerations
: - For rootless DPDK pod support, enable the SELinux `container_use_devices` boolean on the host to allow the tap device to be created. This introduces an acceptable security risk.

**Additional resources**

- [Configuring your firewall for OpenShift Container Platform](/docs/installing/install_config/configuring-firewall#configuring-firewall_configuring-firewall)
- [OpenShift Container Platform network flow matrix](/docs/installing/install_config/configuring-firewall#network-flow-matrix_configuring-firewall)
- [Managing security context constraints](/docs/authentication/managing-security-context-constraints#managing-pod-security-policies)
- [Using node disruption policies to minimize disruption from machine config changes](/docs/machine_configuration/machine-config-node-disruption#machine-config-node-disruption_machine-configs-configure)

## cert-manager Operator {#telco-core-cert-manager-operator_telco-core}

The cert-manager Operator for OpenShift Container Platform manages the lifecycle of TLS certificates for cluster components and workloads.

New in this release
: - The cert-manager Operator is a new optional component in this release.

Description
: The cert-manager Operator for OpenShift Container Platform manages the lifecycle of TLS certificates for cluster components and workloads. The cert-manager Operator automates certificate issuance, renewal, and rotation, eliminating manual certificate management. The reference configuration includes the cert-manager Operator to optionally manage certificates for the API server and ingress controller endpoints.

Limits and requirements
: - The reference configuration includes only the ACME DNS01 challenge type for platform certificate issuance.

Engineering considerations
: - Use RHACM `CertificatePolicy` resources on the hub cluster to monitor certificate expiration and compliance across managed clusters.

## Scalability {#telco-core-scalability_telco-core}

Telco core clusters can scale to at least 120 nodes to support large-scale telco application workloads.

New in this release
: - There are no reference design updates in this release.

Description
: Scale clusters as described in "Limits and requirements". Scaling of workloads is described in "Application workloads".

Limits and requirements
: - A telco core cluster can scale to at least 120 nodes.

## Telco core reference configuration CRs {#telco-core-reference-configuration-crs_telco-core}

Use the following custom resources (CRs) to configure and deploy OpenShift Container Platform clusters with the telco core profile. Use the CRs to form the common baseline used in all the specific use models unless otherwise indicated.

### Extract the telco core reference design configuration CRs {#telco-core-rds-container_telco-core}

You can extract the complete set of custom resources (CRs) for the telco core profile from the `telco-core-rds-rhel9` container image. The container image has both the required CRs, and the optional CRs, for the telco core profile.

**Prerequisites**

- You have installed `podman`.

**Procedure**

1. Log on to the container image registry with your credentials by running the following command:
   ```terminal
   $ podman login registry.redhat.io
   ```
2. Extract the content from the `telco-core-rds-rhel9` container image by running the following commands:
   ```terminal
   $ mkdir -p ./out
   ```

   ```terminal
   $ podman run -it registry.redhat.io/openshift4/openshift-telco-core-rds-rhel9:v4.22 | base64 -d | tar xv -C out
   ```

**Verification**

- The `out` directory has the following directory structure. You can view the telco core CRs in the `out/telco-core-rds/` directory by running the following command:
  ```terminal
  $ tree -L 4
  ```

  ```text title="Example output"
  .
  ├── configuration
  │   ├── compare.sh
  │   ├── core-baseline.yaml
  │   ├── core-finish.yaml
  │   ├── core-overlay.yaml
  │   ├── core-upgrade.yaml
  │   ├── kustomization.yaml
  │   ├── Makefile
  │   ├── ns.yaml
  │   ├── README.md
  │   ├── reference-crs
  │   │   ├── custom-manifests
  │   │   │   ├── mcp-worker-1.yaml
  │   │   │   ├── mcp-worker-2.yaml
  │   │   │   ├── mcp-worker-3.yaml
  │   │   │   └── README.md
  │   │   ├── optional
  │   │   │   ├── logging
  │   │   │   ├── networking
  │   │   │   ├── other
  │   │   │   └── tuning
  │   │   └── required
  │   │       ├── networking
  │   │       ├── other
  │   │       ├── performance
  │   │       ├── scheduling
  │   │       └── storage
  │   ├── reference-crs-kube-compare
  │   │   ├── compare_ignore
  │   │   ├── comparison-overrides.yaml
  │   │   ├── metadata.yaml
  │   │   ├── optional
  │   │   │   ├── logging
  │   │   │   ├── networking
  │   │   │   ├── other
  │   │   │   └── tuning
  │   │   ├── ReferenceVersionCheck.yaml
  │   │   ├── required
  │   │   │   ├── networking
  │   │   │   ├── other
  │   │   │   ├── performance
  │   │   │   ├── scheduling
  │   │   │   └── storage
  │   │   ├── unordered_list.tmpl
  │   │   └── version_match.tmpl
  │   └── template-values
  │       ├── hw-types.yaml
  │       └── regional.yaml
  ├── install
  │   ├── custom-manifests
  │   │   ├── mcp-worker-1.yaml
  │   │   ├── mcp-worker-2.yaml
  │   │   └── mcp-worker-3.yaml
  │   ├── example-standard.yaml
  │   ├── extra-manifests
  │   │   ├── control-plane-load-kernel-modules.yaml
  │   │   ├── kdump-master.yaml
  │   │   ├── kdump-worker.yaml
  │   │   ├── mc_rootless_pods_selinux.yaml
  │   │   ├── mount_namespace_config_master.yaml
  │   │   ├── mount_namespace_config_worker.yaml
  │   │   ├── sctp_module_mc.yaml
  │   │   └── worker-load-kernel-modules.yaml
  │   └── README.md
  └── README.md
  ```

### Compare a cluster with the telco core reference configuration {#using-cluster-compare-telco_core_telco-core}

After you deploy a telco core cluster, you can use the `cluster-compare` plugin to assess the cluster’s compliance with the telco core reference design specifications (RDS). The `cluster-compare` plugin is an OpenShift CLI (`oc`) plugin. The plugin uses a telco core reference configuration to validate the cluster with the telco core custom resources (CRs).

The plugin-specific reference configuration for telco core is packaged in a container image with the telco core CRs.

For further information about the `cluster-compare` plugin, see "Understanding the cluster-compare plugin".

**Prerequisites**

- You have access to the cluster as a user with the `cluster-admin` role.
- You have credentials to access the `registry.redhat.io` container image registry.
- You installed the `cluster-compare` plugin.

**Procedure**

1. Log on to the container image registry with your credentials by running the following command:
   ```terminal
   $ podman login registry.redhat.io
   ```
2. Extract the content from the `telco-core-rds-rhel9` container image by running the following commands:
   ```terminal
   $ mkdir -p ./out
   ```

   ```terminal
   $ podman run -it registry.redhat.io/openshift4/openshift-telco-core-rds-rhel9:v4.22 | base64 -d | tar xv -C out
   ```

   You can view the reference configuration in the `out/telco-core-rds/configuration/reference-crs-kube-compare` directory  by running the following command:

   ```terminal
   $ tree -L 2
   ```
3. Example output
   ```text
   .
   ├── compare_ignore
   ├── comparison-overrides.yaml
   ├── metadata.yaml
   ├── optional
   │   ├── logging
   │   ├── networking
   │   ├── other
   │   └── tuning
   ├── ReferenceVersionCheck.yaml
   ├── required
   │   ├── networking
   │   ├── other
   │   ├── performance
   │   ├── scheduling
   │   └── storage
   ├── unordered_list.tmpl
   └── version_match.tmpl
   ```

   - `metadata.yaml` is the configuration file for the reference configuration.
   - `optional` is the directory for optional templates.
   - `required` is the directory for required templates.
4. Compare the configuration for your cluster to the telco core reference configuration by running the following command:
   ```terminal
   $ oc cluster-compare -r out/telco-core-rds/configuration/reference-crs-kube-compare/metadata.yaml
   ```

   ```terminal title="Example output"
   W1212 14:13:06.281590   36629 compare.go:425] Reference Contains Templates With Types (kind) Not Supported By Cluster: BFDProfile, BGPAdvertisement, BGPPeer, ClusterLogForwarder, Community, IPAddressPool, MetalLB, MultiNetworkPolicy, NMState, NUMAResourcesOperator, NUMAResourcesScheduler, NodeNetworkConfigurationPolicy, SriovNetwork, SriovNetworkNodePolicy, SriovOperatorConfig, StorageCluster

   ...

   **********************************

   Cluster CR: config.openshift.io/v1_OperatorHub_cluster
   Reference File: required/other/operator-hub.yaml
   Diff Output: diff -u -N /tmp/MERGED-2801470219/config-openshift-io-v1_operatorhub_cluster /tmp/LIVE-2569768241/config-openshift-io-v1_operatorhub_cluster
   --- /tmp/MERGED-2801470219/config-openshift-io-v1_operatorhub_cluster	2024-12-12 14:13:22.898756462 +0000
   +++ /tmp/LIVE-2569768241/config-openshift-io-v1_operatorhub_cluster	2024-12-12 14:13:22.898756462 +0000
   @@ -1,6 +1,6 @@
    apiVersion: config.openshift.io/v1
    kind: OperatorHub
    metadata:
   +  annotations:
   +    include.release.openshift.io/hypershift: "true"
      name: cluster
   -spec:
   -  disableAllDefaultSources: true

   **********************************

   Summary
   CRs with diffs: 3/4
   CRs in reference missing from the cluster: 22
   other:
     other:
       Missing CRs:
       - optional/other/control-plane-load-kernel-modules.yaml
       - optional/other/worker-load-kernel-modules.yaml
   required-networking:
     networking-root:
       Missing CRs:
       - required/networking/nodeNetworkConfigurationPolicy.yaml
     networking-sriov:
       Missing CRs:
       - required/networking/sriov/sriovNetwork.yaml
       - required/networking/sriov/sriovNetworkNodePolicy.yaml
       - required/networking/sriov/SriovOperatorConfig.yaml
       - required/networking/sriov/SriovSubscription.yaml
       - required/networking/sriov/SriovSubscriptionNS.yaml
       - required/networking/sriov/SriovSubscriptionOperGroup.yaml
   required-other:
     scheduling:
       Missing CRs:
       - required/other/catalog-source.yaml
       - required/other/icsp.yaml
   required-performance:
     performance:
       Missing CRs:
       - required/performance/PerformanceProfile.yaml
   required-scheduling:
     scheduling:
       Missing CRs:
       - required/scheduling/nrop.yaml
       - required/scheduling/NROPSubscription.yaml
       - required/scheduling/NROPSubscriptionNS.yaml
       - required/scheduling/NROPSubscriptionOperGroup.yaml
       - required/scheduling/sched.yaml
   required-storage:
     storage-odf:
       Missing CRs:
       - required/storage/odf-external/01-rook-ceph-external-cluster-details.secret.yaml
       - required/storage/odf-external/02-ocs-external-storagecluster.yaml
       - required/storage/odf-external/odfNS.yaml
       - required/storage/odf-external/odfOperGroup.yaml
       - required/storage/odf-external/odfSubscription.yaml
   No CRs are unmatched to reference CRs
   Metadata Hash: fe41066bac56517be02053d436c815661c9fa35eec5922af25a1be359818f297
   No patched CRs
   ```

   Where:

   <dl>
   <dt>Cluster CR</dt>
   <dd>The CR under comparison. The plugin displays each CR with a difference from the corresponding template.</dd>
   <dt>Reference File</dt>
   <dd>The template matching with the CR for comparison.</dd>
   <dt>Diff Output</dt>
   <dd>The output in Linux diff format shows the difference between the template and the cluster CR.</dd>
   <dt>Summary</dt>
   <dd>After the plugin reports the line diffs for each CR, the summary of differences are reported.</dd>
   <dt>CRs with diffs</dt>
   <dd>The number of CRs in the comparison with differences from the corresponding templates.</dd>
   <dt>CRs in reference missing from the cluster</dt>
   <dd>The number of CRs represented in the reference configuration, but missing from the live cluster.</dd>
   <dt>Missing CRs</dt>
   <dd>The list of CRs represented in the reference configuration, but missing from the live cluster.</dd>
   <dt>No CRs are unmatched to reference CRs</dt>
   <dd>The CRs that did not match to a corresponding template in the reference configuration.</dd>
   <dt>Metadata Hash</dt>
   <dd>Identifies the reference configuration.</dd>
   <dt>No patched CRs</dt>
   <dd>The list of patched CRs.</dd>
   </dl>

**Additional resources**

- [Understanding the cluster-compare plugin](/docs/scalability_and_performance/cluster-compare/understanding-the-cluster-compare-plugin#understanding-the-cluster-compare-plugin)

### Node configuration reference CRs {#node-configuration-crs_telco-core}

Use the following custom resources (CRs) to configure node settings for the telco core profile.

**Node configuration CRs**

| Component | Reference CR | Description | Optional |
| --- | --- | --- | --- |
| Additional kernel modules | `control-plane-load-kernel-modules.yaml` | Optional. Configures the kernel modules for control plane nodes. | No |
| Additional kernel modules | `sctp_module_mc.yaml` | Optional. Loads the SCTP kernel module in worker nodes. | No |
| Additional kernel modules | `worker-load-kernel-modules.yaml` | Optional. Configures kernel modules for worker nodes. | No |
| Container mount namespace hiding | `mount_namespace_config_master.yaml` | Configures a mount namespace for sharing container-specific mounts between kubelet and CRI-O on control plane nodes. | No |
| Container mount namespace hiding | `mount_namespace_config_worker.yaml` | Configures a mount namespace for sharing container-specific mounts between kubelet and CRI-O on worker nodes. | No |
| Kdump enable | `kdump-master.yaml` | Configures kdump crash reporting on master nodes. | No |
| Kdump enable | `kdump-worker.yaml` | Configures kdump crash reporting on worker nodes. | No |

### Cluster infrastructure reference CRs {#cluster-infrastructure-crs_telco-core}

Use the following custom resources (CRs) to configure cluster infrastructure components for the telco core profile.

**Cluster infrastructure CRs**

| Component | Reference CR | Description | Optional |
| --- | --- | --- | --- |
| Cluster logging | `ClusterLogForwarder.yaml` | Configures a log forwarding instance with the specified service account and verifies that the configuration is valid. | Yes |
| Cluster logging | `ClusterLogNS.yaml` | Configures the cluster logging namespace. | Yes |
| Cluster logging | `ClusterLogOperGroup.yaml` | Creates the Operator group in the openshift-logging namespace, allowing the Cluster Logging Operator to watch and manage resources. | Yes |
| Cluster logging | `ClusterLogServiceAccount.yaml` | Configures the cluster logging service account. | Yes |
| Cluster logging | `ClusterLogServiceAccountAuditBinding.yaml` | Grants the collect-audit-logs cluster role to the logs collector service account. | Yes |
| Cluster logging | `ClusterLogServiceAccountInfrastructureBinding.yaml` | Allows the collector service account to collect logs from infrastructure resources. | Yes |
| Cluster logging | `ClusterLogSubscription.yaml` | Creates a subscription resource for the Cluster Logging Operator with manual approval for install plans. | Yes |
| Disconnected configuration | `catalog-source.yaml` | Defines a disconnected Red Hat Operators catalog. | No |
| Disconnected configuration | `idms.yaml` | Defines a list of mirrored repository digests for the disconnected registry. | No |
| Disconnected configuration | `operator-hub.yaml` | Defines an OperatorHub configuration which disables all default sources. | No |
| Monitoring and observability | `monitoring-config-cm.yaml` | Configuring storage and retention for Prometheus and Alertmanager. | Yes |
| Power management | `PerformanceProfile.yaml` | Defines a performance profile resource, specifying CPU isolation, hugepages configuration, and workload hints for performance optimization on selected nodes. | No |

### Resource tuning reference CRs {#resource-tuning-crs_telco-core}

Use the following custom resources (CRs) to configure resource tuning for the telco core profile.

**Resource tuning CRs**

| Component | Reference CR | Description | Optional |
| --- | --- | --- | --- |
| System reserved capacity | `control-plane-system-reserved.yaml` | Optional. Configures kubelet, enabling auto-sizing reserved resources for the control plane node pool. | No |

### Networking reference CRs {#networking-crs_telco-core}

Use the following custom resources (CRs) to configure networking components for the telco core profile.

**Networking CRs**

| Component | Reference CR | Description | Optional |
| --- | --- | --- | --- |
| Baseline | `Network.yaml` | Configures the default cluster network, specifying OVN Kubernetes settings like routing via the host. It also allows the definition of additional networks, including custom CNI configurations, and enables the use of MultiNetworkPolicy CRs for network policies across multiple networks. | No |
| Baseline | `networkAttachmentDefinition.yaml` | Optional. Defines a NetworkAttachmentDefinition resource specifying network configuration details such as node selector and CNI configuration. | No |
| Load Balancer | `addr-pool.yaml` | Configures MetalLB to manage a pool of IP addresses with auto-assign enabled for dynamic allocation of IPs from the specified range. | No |
| Load Balancer | `bfd-profile.yaml` | Configures bidirectional forwarding detection (BFD) with customized intervals, detection multiplier, and modes for quicker network fault detection and load balancing failover. | No |
| Load Balancer | `bgp-advr.yaml` | Defines a BGP advertisement resource for MetalLB, specifying how an IP address pool is advertised to BGP peers. This enables fine-grained control over traffic routing and announcements. | No |
| Load Balancer | `bgp-peer.yaml` | Defines a BGP peer in MetalLB, representing a BGP neighbor for dynamic routing. | No |
| Load Balancer | `community.yaml` | Defines a MetalLB community, which groups one or more BGP communities under a named resource. Communities can be applied to BGP advertisements to control routing policies and change traffic routing. | No |
| Load Balancer | `metallb.yaml` | Defines the MetalLB resource in the cluster. | No |
| Load Balancer | `metallbNS.yaml` | Defines the metallb-system namespace in the cluster. | No |
| Load Balancer | `metallbOperGroup.yaml` | Defines the Operator group for the MetalLB Operator. | No |
| Load Balancer | `metallbSubscription.yaml` | Creates a subscription resource for the MetalLB Operator with manual approval for install plans. | No |
| Multus - Tap CNI for rootless DPDK pods | `mc_rootless_pods_selinux.yaml` | Configures a MachineConfig resource which sets an SELinux boolean for the tap CNI plugin on worker nodes. | Yes |
| NMState Operator | `NMState.yaml` | Defines an NMState resource that is used by the NMState Operator to manage node network configurations. | No |
| NMState Operator | `NMStateNS.yaml` | Creates the NMState Operator namespace. | No |
| NMState Operator | `NMStateOperGroup.yaml` | Creates the Operator group in the openshift-nmstate namespace, allowing the NMState Operator to watch and manage resources. | No |
| NMState Operator | `NMStateSubscription.yaml` | Creates a subscription for the NMState Operator, managed through OLM. | No |
| SR-IOV Network Operator | `sriovNetwork.yaml` | Defines an SR-IOV network specifying network capabilities, IP address management (ipam), and the associated network namespace and resource. | No |
| SR-IOV Network Operator | `sriovNetworkNodePolicy.yaml` | Configures network policies for SR-IOV devices on specific nodes, including customization of device selection, VF allocation (numVfs), node-specific settings (nodeSelector), and priorities. | No |
| SR-IOV Network Operator | `SriovOperatorConfig.yaml` | Configures various settings for the SR-IOV Operator, including enabling the injector and Operator webhook, disabling pod draining, and defining the node selector for the configuration daemon. | No |
| SR-IOV Network Operator | `SriovSubscription.yaml` | Creates a subscription for the SR-IOV Network Operator, managed through OLM. | No |
| SR-IOV Network Operator | `SriovSubscriptionNS.yaml` | Creates the SR-IOV Network Operator subscription namespace. | No |
| SR-IOV Network Operator | `SriovSubscriptionOperGroup.yaml` | Creates the Operator group for the SR-IOV Network Operator, allowing it to watch and manage resources in the target namespace. | No |

### Scheduling reference CRs {#scheduling-crs_telco-core}

Use the following custom resources (CRs) to configure scheduling for the telco core profile.

**Scheduling CRs**

| Component | Reference CR | Description | Optional |
| --- | --- | --- | --- |
| NUMA-aware scheduler | `nrop.yaml` | Enables the NUMA Resources Operator, aligning workloads with specific NUMA node configurations. Required for clusters with multi-NUMA nodes. | No |
| NUMA-aware scheduler | `NROPSubscription.yaml` | Creates a subscription for the NUMA Resources Operator, managed through OLM. Required for clusters with multi-NUMA nodes. | No |
| NUMA-aware scheduler | `NROPSubscriptionNS.yaml` | Creates the NUMA Resources Operator subscription namespace. Required for clusters with multi-NUMA nodes. | No |
| NUMA-aware scheduler | `NROPSubscriptionOperGroup.yaml` | Creates the Operator group in the numaresources-operator namespace, allowing the NUMA Resources Operator to watch and manage resources. Required for clusters with multi-NUMA nodes. | No |
| NUMA-aware scheduler | `sched.yaml` | Configures a topology-aware scheduler in the cluster that can handle NUMA aware scheduling of pods across nodes. | No |
| NUMA-aware scheduler | `Scheduler.yaml` | Configures control plane nodes as non-schedulable for workloads. | No |

### Storage reference CRs {#storage-crs_telco-core}

Use the following custom resources (CRs) to configure storage for the telco core profile.

**Storage CRs**

| Component | Reference CR | Description | Optional |
| --- | --- | --- | --- |
| External ODF configuration | `01-rook-ceph-external-cluster-details.secret.yaml` | Defines a Secret resource containing base64-encoded configuration data for an external Ceph cluster in the openshift-storage namespace. | No |
| External ODF configuration | `02-ocs-external-storagecluster.yaml` | Defines an OpenShift Container Storage (OCS) storage resource which configures the cluster to use an external storage back end. | No |
| External ODF configuration | `odfNS.yaml` | Creates the monitored openshift-storage namespace for the OpenShift Data Foundation Operator. | No |
| External ODF configuration | `odfOperGroup.yaml` | Creates the Operator group in the openshift-storage namespace, allowing the OpenShift Data Foundation Operator to watch and manage resources. | No |
| External ODF configuration | `odfSubscription.yaml` | Creates the OpenShift Data Foundation Operator subscription, managed through OLM. | No |

### Security reference CRs {#security-crs_telco-core}

Use the following custom resources (CRs) to configure security components for the telco core profile.

**Security CRs**

| Component | Reference CR | Description | Optional |
| --- | --- | --- | --- |
| Cert-Manager | `certManagerNS.yaml` | Defines the cert-manager-operator namespace. | Yes |
| Cert-Manager | `certManagerOperatorgroup.yaml` | Defines the OperatorGroup for cert-manager. | Yes |
| Cert-Manager | `certManagerSubscription.yaml` | Installs the OpenShift cert-manager operator. | Yes |
| Cert-Manager | `certManagerClusterIssuer.yaml` | Configures an ACME ClusterIssuer using Let’s Encrypt with DNS-01 challenge. | Yes |
| Cert-Manager | `apiServerCertificate.yaml` | Creates a certificate for the API Server endpoint. | Yes |
| Cert-Manager | `ingressCertificate.yaml` | Creates a wildcard certificate for the Ingress/Router. | Yes |
| Cert-Manager | `apiServerConfig.yaml` | Configures OpenShift to use the cert-manager generated API Server certificate. | Yes |
| Cert-Manager | `ingressControllerConfig.yaml` | Configures OpenShift to use the cert-manager generated Ingress certificate. | Yes |

## Telco core reference configuration software specifications {#telco-core-software-stack_telco-core}

The Red Hat telco core 4.22 solution has been validated using the following Red Hat software products for OpenShift Container Platform clusters.

**Telco core cluster validated software components**

| Component | Software version |
| --- | --- |
| Red Hat Advanced Cluster Management (RHACM) | 2.17 |
| Red Hat OpenShift GitOps | 1.20 |
| cert-manager Operator | 1.19 |
| Cluster Logging Operator | 6.5 |
| OpenShift Data Foundation | 4.22 |
| SR-IOV Network Operator | 4.22 |
| MetalLB | 4.22 |
| NMState Operator | 4.22 |
| NUMA-aware scheduler | 4.22 |

- OpenShift Data Foundation is expected to be updated to 4.22 when the aligned OpenShift Data Foundation version is released.
- The Cluster Logging Operator is expected to be updated to 6.6 when the aligned version is released.
- The cert-manager Operator and Red Hat OpenShift GitOps Operator are platform-agnostic operators. The support lifecycle for these operators is independent from the support lifecycle for OpenShift Container Platform. You might need to update to a newer minor version of these operators at the end of an operator lifecycle, or when planning to update the OpenShift Container Platform cluster to continue support. For support lifecycle details for platform-agnostic operators, see [OpenShift Operator Life Cycles](https://access.redhat.com/support/policy/updates/openshift_operators).
