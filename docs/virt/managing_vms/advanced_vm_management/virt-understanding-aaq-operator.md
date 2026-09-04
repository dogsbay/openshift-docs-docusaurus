---
title: Configuring the Application-Aware Quota (AAQ) Operator
sidebar_position: 2
---

# Configuring the Application-Aware Quota (AAQ) Operator {#virt-understanding-aaq-operator}

<a id="virt-understanding-aaq-operator"></a>

You can use the Application-Aware Quota (AAQ) Operator to customize and manage resource quotas for individual components in an OpenShift Container Platform cluster.

## About the AAQ Operator {#virt-about-aaq-operator_virt-understanding-aaq-operator}

The Application-Aware Quota (AAQ) Operator provides more flexible and extensible quota management compared to the native `ResourceQuota` object in the OpenShift Container Platform platform.

In a multi-tenant cluster environment, where multiple workloads operate on shared infrastructure and resources, using the Kubernetes native `ResourceQuota` object to limit aggregate CPU and memory consumption presents infrastructure overhead and live migration challenges for OpenShift Virtualization workloads.

OpenShift Virtualization requires significant compute resource allocation to handle virtual machine (VM) live migrations and manage VM infrastructure overhead. When upgrading OpenShift Virtualization, you must migrate VMs to upgrade the `virt-launcher` pod. However, migrating a VM in the presence of a resource quota can cause the migration, and subsequently the upgrade, to fail.

With AAQ, you can allocate resources for VMs without interfering with cluster-level activities such as upgrades and node maintenance. The AAQ Operator also supports non-compute resources which eliminates the need to manage both the native resource quota and AAQ API objects separately.

### AAQ Operator controller and custom resources {#aaq-controller-and-crds_virt-understanding-aaq-operator}

The AAQ Operator introduces two new API objects defined as custom resource definitions (CRDs) for managing alternative quota implementations across multiple namespaces:

- `ApplicationAwareResourceQuota`: Sets aggregate quota restrictions enforced per namespace. The `ApplicationAwareResourceQuota` API is compatible with the native `ResourceQuota` object and shares the same specification and status definitions.
  Example manifest:

  ```yaml
  apiVersion: aaq.kubevirt.io/v1alpha1
  kind: ApplicationAwareResourceQuota
  metadata:
    name: example-resource-quota
  spec:
    hard:
      requests.memory: 1Gi
      limits.memory: 1Gi
      requests.cpu/vmi: "1"
      requests.memory/vmi: 1Gi
  # ...
  ```

  - `spec.hard.requests.cpu/vmi` defines the maximum amount of CPU that is allowed for VM workloads in the default namespace.
  - `spec.hard.requests.memory/vmi` defines the maximum amount of RAM that is allowed for VM workloads in the default namespace.
- `ApplicationAwareClusterResourceQuota`: Mirrors the `ApplicationAwareResourceQuota` object at a cluster scope. It is compatible with the native `ClusterResourceQuota` API object and shares the same specification and status definitions. When creating an AAQ cluster quota, you can select multiple namespaces based on annotation selection, label selection, or both by editing the `spec.selector.labels` or `spec.selector.annotations` fields. You can only create an `ApplicationAwareClusterResourceQuota` object if the `spec.applicationAwareConfig.allowApplicationAwareClusterResourceQuota` field in the `HyperConverged` custom resource (CR) is set to `true`.
  Example manifest:

  ```yaml
  apiVersion: aaq.kubevirt.io/v1alpha1
  kind: ApplicationAwareClusterResourceQuota
  metadata:
    name: example-resource-quota
  spec:
    quota:
      hard:
        requests.memory: 1Gi
        limits.memory: 1Gi
        requests.cpu/vmi: "1"
        requests.memory/vmi: 1Gi
    selector:
      annotations: null
      labels:
        matchLabels:
          kubernetes.io/metadata.name: default
  # ...
  ```

  :::note

  If both `spec.selector.labels` and `spec.selector.annotations` fields are set, only namespaces that match both are selected.

  :::

The AAQ controller uses a scheduling gate mechanism to evaluate whether there is enough of a resource available to run a workload. If so, the scheduling gate is removed from the pod and it is considered ready for scheduling. The quota usage status is updated to indicate how much of the quota is used.

If the CPU and memory requests and limits for the workload exceed the enforced quota usage limit, the pod remains in `SchedulingGated` status until there is enough quota available. The AAQ controller creates an event of type `Warning` with details on why the quota was exceeded.  You can view the event details by using the `oc get events` command.

:::warning

Pods that have the `spec.nodeName` field set to a specific node cannot use namespaces that match the `spec.namespaceSelector` labels defined in the `HyperConverged` CR.

:::

## Enabling the Application Aware Quota Operator {#virt-enabling-aaq-operator_virt-understanding-aaq-operator}

Enable the Application Aware Quota (AAQ) operator to manage resource quotas to extend native resource management capabilities. Enabling AAQ from the web console eliminates the need to manually edit the HyperConverged custom resource by using the CLI.

**Prerequisites**

- You have cluster administrator privileges.
- The OpenShift Virtualization operator is installed and running.

**Procedure**

1. In the OpenShift Container Platform web console, click **Virtualization** → **Settings**.
2. Under **Resource Management**, toggle the **Application Aware Quotas (AAQ)** to On.
   The operator deployment begins automatically. The status changes from **Disabled** to **Enabled** after the operator pods are running.
3. Optional: Click the **Edit** icon next to the **Quota calculation method** field to change the calculation method.
   A modal dialog is displayed with the following options:

   - **Virtual resources**: Measures only the virtual CPU and memory allocated to virtual machines (VMs), excluding pod runtime overhead. This is the default option for virtualization workloads.
   - **Virtual Machine Instance (VMI) pod usage**: Measures the total virtual CPU and memory consumption of the VM pod, including both the virtual machine and pod runtime overhead.
   - **Dedicated virtual resources**: Measures the virtual CPU and memory resources assigned to virtual machines and their associated pods, tracking quota usage separately for each type.
4. Select your preferred quota calculation method and click **Save**.

**Verification**

1. Verify that a new **Quotas** option is displayed in the left navigation menu under **Virtualization**. This indicates that AAQ is ready for quota creation and management.

## Configuring the AAQ Operator by using the CLI {#virt-configuring-aaq-operator_virt-understanding-aaq-operator}

You can configure the AAQ Operator by specifying the fields of the `spec.applicationAwareConfig` object in the `HyperConverged` custom resource (CR).

**Prerequisites**

- You have access to the cluster as a user with `cluster-admin` privileges.
- You have installed the OpenShift CLI (`oc`).

**Procedure**

- Update the `HyperConverged` CR by running the following command:
  ```terminal
  $ oc patch hyperconvergeds.v1beta1.hco.kubevirt.io kubevirt-hyperconverged -n openshift-cnv --type merge -p '{
    "spec": {
      "applicationAwareConfig": {
        "vmiCalcConfigName": "DedicatedVirtualResources",
        "namespaceSelector": {
          "matchLabels": {
            "app": "my-app"
          }
        },
        "allowApplicationAwareClusterResourceQuota": true
      }
    }
  }'
  ```

  where:

  <dl>
  <dt><code>vmiCalcConfigName</code></dt>
  <dd>Specifies how resource counting is managed for pods that run virtual machine (VM) workloads. Possible values are: *   <code>VmiPodUsage</code>: Counts compute resources for pods associated with VMs in the same way as native resource quotas and excludes migration-related resources. *   <code>VirtualResources</code>: Counts compute resources based on the VM specifications, using the VM RAM size for memory and virtual CPUs for processing. *   <code>DedicatedVirtualResources</code> (default): Similar to <code>VirtualResources</code>, but separates resource tracking for pods associated with VMs by adding a <code>/vmi</code> suffix to CPU and memory resource names. For example, <code>requests.cpu/vmi</code> and <code>requests.memory/vmi</code>.</dd>
  <dt><code>namespaceSelector</code></dt>
  <dd>Determines the namespaces for which an AAQ scheduling gate is added to pods when they are created. If a namespace selector is not defined, the AAQ Operator targets namespaces with the <code>application-aware-quota/enable-gating</code> label as default.</dd>
  <dt><code>allowApplicationAwareClusterResourceQuota</code></dt>
  <dd>If set to <code>true</code>, you can create and manage the <code>ApplicationAwareClusterResourceQuota</code> object. Setting this attribute to <code>true</code> can increase scheduling time.</dd>
  </dl>

**Additional resources**

- [Resource quotas per project](/docs/applications/quotas/quotas-setting-per-project#quotas-setting-per-project)
- [Resource quotas across multiple projects](/docs/applications/quotas/quotas-setting-across-multiple-projects#quotas-setting-across-multiple-projects)
- [`ResourceQuota` API reference](/docs/rest_api/schedule_and_quota_apis/resourcequota-v1#resourcequota-v1)
- [`ClusterResourceQuota` API reference](/docs/rest_api/schedule_and_quota_apis/clusterresourcequota-quota-openshift-io-v1#clusterresourcequota-quota-openshift-io-v1)
- [Pod scheduling gates specification](/docs/rest_api/workloads_apis/pod-v1#spec-schedulinggates)
- [Viewing system event information in an OpenShift Container Platform cluster](/docs/nodes/clusters/nodes-containers-events#nodes-containers-events)
