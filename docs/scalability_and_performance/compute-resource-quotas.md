---
title: Using quotas and limit ranges
sidebar_position: 8
---

# Using quotas and limit ranges {#compute-resource-quotas}

<a id="compute-resource-quotas"></a>

As a cluster administrator, you can use quotas and limit ranges to set constraints. These constraints limit the number of objects or the amount of compute resources that are used in your project.

By using quotes and limits, you can better manage and allocate resources across all projects. You can also ensure that no projects use more resources than is appropriate for the cluster size.

A resource quota, defined by a `ResourceQuota` object, provides constraints that limit aggregate resource consumption per project. The quota can limit the quantity of objects that can be created in a project by type. Additinally, the quota can limit the total amount of compute resources and storage that might be consumed by resources in that project.

:::warning

Quotas are set by cluster administrators and are scoped to a given project. OpenShift Container Platform project owners can change quotas for their project, but not limit ranges. OpenShift Container Platform users cannot modify quotas or limit ranges.

:::

## Resources managed by quota {#admin-quota-overview_using-quotas-and-limit-ranges}

To limit aggregate resource consumption per project, define a `ResourceQuota` object. By using this object, you can restrict the number of created objects by type. You can also restrict the total amount of compute resources and storage consumed within the project.

The following tables describe the set of compute resources and object types that a quota might manage.

:::note

A pod is in a terminal state if `status.phase` is `Failed` or `Succeeded`.

:::

**Compute resources managed by quota**

<table>
<thead>
<tr>
  <th>Resource Name</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>cpu</code></td>
  <td>The sum of CPU requests across all pods in a non-terminal state cannot exceed this value. <code>cpu</code> and <code>requests.cpu</code> are the same value and can be used interchangeably.</td>
</tr>
<tr>
  <td><code>memory</code></td>
  <td>The sum of memory requests across all pods in a non-terminal state cannot exceed this value. <code>memory</code> and <code>requests.memory</code> are the same value and can be used interchangeably.</td>
</tr>
<tr>
  <td><code>ephemeral-storage</code></td>
  <td>The sum of local ephemeral storage requests across all pods in a non-terminal state cannot exceed this value. <code>ephemeral-storage</code> and <code>requests.ephemeral-storage</code> are the same value and can be used interchangeably. This resource is available only if you enabled the ephemeral storage technology preview. This feature is disabled by default.</td>
</tr>
<tr>
  <td><code>requests.cpu</code></td>
  <td>The sum of CPU requests across all pods in a non-terminal state cannot exceed this value. <code>cpu</code> and <code>requests.cpu</code> are the same value and can be used interchangeably.</td>
</tr>
<tr>
  <td><code>requests.memory</code></td>
  <td>The sum of memory requests across all pods in a non-terminal state cannot exceed this value. <code>memory</code> and <code>requests.memory</code> are the same value and can be used interchangeably.</td>
</tr>
<tr>
  <td><code>requests.ephemeral-storage</code></td>
  <td>The sum of ephemeral storage requests across all pods in a non-terminal state cannot exceed this value. <code>ephemeral-storage</code> and <code>requests.ephemeral-storage</code> are the same value and can be used interchangeably. This resource is available only if you enabled the ephemeral storage technology preview. This feature is disabled by default.</td>
</tr>
<tr>
  <td><code>limits.cpu</code></td>
  <td>The sum of CPU limits across all pods in a non-terminal state cannot exceed this value.</td>
</tr>
<tr>
  <td><code>limits.memory</code></td>
  <td>The sum of memory limits across all pods in a non-terminal state cannot exceed this value.</td>
</tr>
<tr>
  <td><code>limits.ephemeral-storage</code></td>
  <td>The sum of ephemeral storage limits across all pods in a non-terminal state cannot exceed this value. This resource is available only if you enabled the ephemeral storage technology preview. This feature is disabled by default.</td>
</tr>
</tbody>
</table>

**Storage resources managed by quota**

<table>
<thead>
<tr>
  <th>Resource Name</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>requests.storage</code></td>
  <td>The sum of storage requests across all persistent volume claims in any state cannot exceed this value.</td>
</tr>
<tr>
  <td><code>persistentvolumeclaims</code></td>
  <td>The total number of persistent volume claims that can exist in the project.</td>
</tr>
<tr>
  <td><code>&lt;storage-class-name&gt;.storageclass.storage.k8s.io/requests.storage</code></td>
  <td>The sum of storage requests across all persistent volume claims in any state that have a matching storage class, cannot exceed this value.</td>
</tr>
<tr>
  <td><code>&lt;storage-class-name&gt;.storageclass.storage.k8s.io/persistentvolumeclaims</code></td>
  <td>The total number of persistent volume claims with a matching storage class that can exist in the project.</td>
</tr>
</tbody>
</table>

**Object counts managed by quota**

<table>
<thead>
<tr>
  <th>Resource Name</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>pods</code></td>
  <td>The total number of pods in a non-terminal state that can exist in the project.</td>
</tr>
<tr>
  <td><code>replicationcontrollers</code></td>
  <td>The total number of replication controllers that can exist in the project.</td>
</tr>
<tr>
  <td><code>resourcequotas</code></td>
  <td>The total number of resource quotas that can exist in the project.</td>
</tr>
<tr>
  <td><code>services</code></td>
  <td>The total number of services that can exist in the project.</td>
</tr>
<tr>
  <td><code>secrets</code></td>
  <td>The total number of secrets that can exist in the project.</td>
</tr>
<tr>
  <td><code>configmaps</code></td>
  <td>The total number of <code>ConfigMap</code> objects that can exist in the project.</td>
</tr>
<tr>
  <td><code>persistentvolumeclaims</code></td>
  <td>The total number of persistent volume claims that can exist in the project.</td>
</tr>
<tr>
  <td><code>openshift.io/imagestreams</code></td>
  <td>The total number of image streams that can exist in the project.</td>
</tr>
</tbody>
</table>

You can configure an object count quota for these standard namespaced resource types using the `count/<resource>.<group>` syntax.

```terminal
$ oc create quota <name> --hard=count/<resource>.<group>=<quota>
```

where:

`<resource>`
: Specifies the name of the resource.

`<group>`
: Specifies the API group, if applicable. You can use the `kubectl api-resources` command for a list of resources and their associated API groups.

## Setting resource quota for extended resources {#setting-resource-quota-extended-resources_using-quotas-and-limit-ranges}

To manage the consumption of extended resources, such as `nvidia.com/gpu`, define a resource quota by using the `requests` prefix. Since overcommitment is prohibited for these resources, you must explicitly specify both requests and limits to ensure valid configuration.

**Procedure**

1. To determine how many GPUs are available on a node in your cluster, use the following command:
   ```terminal
   $ oc describe node ip-172-31-27-209.us-west-2.compute.internal | egrep 'Capacity|Allocatable|gpu'
   ```

   ```terminal title="Example output"
   openshift.com/gpu-accelerator=true
   Capacity:
    nvidia.com/gpu:  2
   Allocatable:
    nvidia.com/gpu:  2
    nvidia.com/gpu:  0           0
   ```

   In this example, 2 GPUs are available.
2. Use this command to set a quota in the namespace `nvidia`. In this example, the quota is `1`:
   ```terminal
   $ cat gpu-quota.yaml
   ```

   ```terminal title="Example output"
   apiVersion: v1
   kind: ResourceQuota
   metadata:
     name: gpu-quota
     namespace: nvidia
   spec:
     hard:
       requests.nvidia.com/gpu: 1
   ```
3. Create the quota with the following command:
   ```terminal
   $ oc create -f gpu-quota.yaml
   ```

   ```terminal title="Example output"
   resourcequota/gpu-quota created
   ```
4. Verify that the namespace has the correct quota set using the following command:
   ```terminal
   $ oc describe quota gpu-quota -n nvidia
   ```

   ```terminal title="Example output"
   Name:                    gpu-quota
   Namespace:               nvidia
   Resource                 Used  Hard
   --------                 ----  ----
   requests.nvidia.com/gpu  0     1
   ```
5. Run a pod that asks for a single GPU with the following command:
   ```terminal
   $ oc create pod gpu-pod.yaml
   ```

   ```terminal title="Example output"
   apiVersion: v1
   kind: Pod
   metadata:
     generateName: gpu-pod-s46h7
     namespace: nvidia
   spec:
     restartPolicy: OnFailure
     containers:
     - name: rhel7-gpu-pod
       image: rhel7
       env:
         - name: NVIDIA_VISIBLE_DEVICES
           value: all
         - name: NVIDIA_DRIVER_CAPABILITIES
           value: "compute,utility"
         - name: NVIDIA_REQUIRE_CUDA
           value: "cuda>=5.0"

       command: ["sleep"]
       args: ["infinity"]

       resources:
         limits:
           nvidia.com/gpu: 1
   ```
6. Verify that the pod is running with the following command:
   ```terminal
   $ oc get pods
   ```

   ```terminal title="Example output"
   NAME              READY     STATUS      RESTARTS   AGE
   gpu-pod-s46h7     1/1       Running     0          1m
   ```
7. Verify that the quota `Used` counter is correct by running the following command:
   ```terminal
   $ oc describe quota gpu-quota -n nvidia
   ```

   ```terminal title="Example output"
   Name:                    gpu-quota
   Namespace:               nvidia
   Resource                 Used  Hard
   --------                 ----  ----
   requests.nvidia.com/gpu  1     1
   ```
8. Using the following command, attempt to create a second GPU pod in the `nvidia` namespace. This is technically available on the node because it has 2 GPUs:
   ```terminal
   $ oc create -f gpu-pod.yaml
   ```

   ```terminal title="Example output"
   Error from server (Forbidden): error when creating "gpu-pod.yaml": pods "gpu-pod-f7z2w" is forbidden: exceeded quota: gpu-quota, requested: requests.nvidia.com/gpu=1, used: requests.nvidia.com/gpu=1, limited: requests.nvidia.com/gpu=1
   ```

   You receive this `Forbidden` error message because you have a quota of 1 GPU and the pod tried to allocate a second GPU, which exceeds the allowed quota.

## Quota scopes {#quota-scopes_using-quotas-and-limit-ranges}

To restrict the set of resources that a quota applies to, add associated scopes. This configuration limits usage measurement to the intersection of the enumerated scopes, ensuring that specifying a resource outside the allowed set results in a validation error.

<table>
<thead>
<tr>
  <th>Scope</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>Terminating</code></td>
  <td>Match pods where <code>spec.activeDeadlineSeconds &gt;= 0</code>.</td>
</tr>
<tr>
  <td><code>NotTerminating</code></td>
  <td>Match pods where <code>spec.activeDeadlineSeconds</code> is <code>nil</code>.</td>
</tr>
<tr>
  <td><code>BestEffort</code></td>
  <td>Match pods that have best effort quality of service for either <code>cpu</code> or <code>memory</code>.</td>
</tr>
<tr>
  <td><code>NotBestEffort</code></td>
  <td>Match pods that do not have best effort quality of service for <code>cpu</code> and <code>memory</code>.</td>
</tr>
<tr>
  <td><code>CrossNamespacePodAffinity</code></td>
  <td>Match all pod objects that have cross-namespace pod (anti)affinity mentioned.</td>
</tr>
<tr>
  <td><code>PriorityClass</code></td>
  <td>Match all pod objects that have priority class mentioned.</td>
</tr>
<tr>
  <td><code>VolumeAttributesClass</code></td>
  <td>Match all persistent volume claims (PVCs) that have volume attributes class mentioned.</td>
</tr>
</tbody>
</table>

A `BestEffort` scope restricts a quota to limiting the following resources:

- pods

A `Terminating`, `NotTerminating`, and `NotBestEffort` scope restricts a quota to tracking the following resources:

- `pods`
- `memory`
- `requests.memory`
- `limits.memory`
- `cpu`
- `requests.cpu`
- `limits.cpu`
- `ephemeral-storage`
- `requests.ephemeral-storage`
- `limits.ephemeral-storage`

:::note

Ephemeral storage requests and limits apply only if you enabled the ephemeral storage technology preview. This feature is disabled by default.

:::

You can also limit a quota with the optional `scopeSelector` field. In `scopeSelector.matchExpressions`, set a `scopeName`, an `operator`, and, when required, a `values` array. Scopes such as `PriorityClass` and `VolumeAttributesClass` match resources when the selector selects them.

The `operator` field supports the following values:

- `In`
- `NotIn`
- `Exists`
- `DoesNotExist`

If the `operator` is `In` or `NotIn`, the `values` field must include at least one value. If the `operator` is `Exists` or `DoesNotExist`, do not set `values`.

```yaml title="Example ResourceQuota scoped to a PriorityClass"
apiVersion: v1
kind: ResourceQuota
metadata:
  name: pods-high-priority
spec:
  hard:
    pods: "10"
    requests.cpu: "1"
    requests.memory: 1Gi
  scopeSelector:
    matchExpressions:
    - scopeName: PriorityClass
      operator: In
      values:
      - high-priority
```

**Additional resources**

- [Resources managed by quotas](/docs/applications/quotas/quotas-setting-per-project#quotas-setting-per-project_quotas-setting-per-project)
- [Resource requests and overcommitment](/docs/nodes/clusters/nodes-cluster-overcommit#nodes-cluster-overcommit-resource-requests_nodes-cluster-overcommit)

## Admin quota usage {#admin-quota-usage_using-quotas-and-limit-ranges}

To ensure projects remain within defined constraints, monitor admin quota usage. After a resource quota for a project is first created, the project restricts the ability to create any new resources that can violate a quota constraint until it has calculated updated usage statistics.

Quota enforcement
: After a resource quota for a project is first created, the project restricts the ability to create any new resources that can violate a quota constraint until the quota has calculated updated usage statistics.

  After a quota is created and usage statistics are updated, the project accepts the creation of new content. When you create or modify resources, your quota usage is incremented immediately upon the request to create or modify the resource.

  When you delete a resource, your quota use is decremented during the next full recalculation of quota statistics for the project.

  A configurable amount of time determines how long the quota takes to reduce quota usage statistics to their current observed system value.

  If project modifications exceed a quota usage limit, the server denies the action and returns an appropriate error message to the user. The error message explains the quota constraint violated and what their currently observed usage statistics are in the system.

Requests compared to limits
: When allocating compute resources by quota, each container can specify a request and a limit value each for CPU, memory, and ephemeral storage. Quotas can restrict any of these values.

  If the quota has a value specified for `requests.cpu` or `requests.memory`, then the quota requires that every incoming container makes an explicit request for those resources. If the quota has a value specified for `limits.cpu` or `limits.memory`, the quota requires that every incoming container specify an explicit limit for those resources.

### Sample resource quota definitions {#sample-resource-quota-definitions_using-quotas-and-limit-ranges}

To properly structure your quota configurations, reference these sample `ResourceQuota` definitions. These YAML examples demonstrate how to specify hard limits for compute resources, storage, and object counts to ensure your project complies with cluster policies.

```yaml title="Example core-object-counts.yaml"
apiVersion: v1
kind: ResourceQuota
metadata:
  name: core-object-counts
spec:
  hard:
    configmaps: "10"
    persistentvolumeclaims: "4"
    replicationcontrollers: "20"
    secrets: "10"
    services: "10"
# ...
```

where:

`configmaps`
: Specifies the total number of `ConfigMap` objects that can exist in the project.

`persistentvolumeclaims`
: Specifies the total number of persistent volume claims (PVCs) that can exist in the project.

`replicationcontrollers`
: Specifies the total number of replication controllers that can exist in the project.

`secrets`
: Specifies the total number of secrets that can exist in the project.

`services`
: Specifies the total number of services that can exist in the project.

```yaml title="Example openshift-object-counts.yaml"
apiVersion: v1
kind: ResourceQuota
metadata:
  name: openshift-object-counts
spec:
  hard:
    openshift.io/imagestreams: "10"
# ...
```

where:

`openshift.io/imagestreams`
: Specifies the total number of image streams that can exist in the project.

```yaml title="Example compute-resources.yaml"
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
spec:
  hard:
    pods: "4"
    requests.cpu: "1"
    requests.memory: 1Gi
    requests.ephemeral-storage: 2Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    limits.ephemeral-storage: 4Gi
# ...
```

where:

`pods`
: Specifies the total number of pods in a non-terminal state that can exist in the project.

`requests.cpu`
: Specifies that across all pods in a non-terminal state, the sum of CPU requests cannot exceed 1 core.

`requests.memory`
: Specifies that across all pods in a non-terminal state, the sum of memory requests cannot exceed 1 Gi.

`requests.ephemeral-storage`
: Specifies that across all pods in a non-terminal state, the sum of ephemeral storage requests cannot exceed 2 Gi.

`limits.cpu`
: Specifies that across all pods in a non-terminal state, the sum of CPU limits cannot exceed 2 cores.

`limits.memory`
: Specifies that across all pods in a non-terminal state, the sum of memory limits cannot exceed 2 Gi.

`limits.ephemeral-storage`
: Specifies that across all pods in a non-terminal state, the sum of ephemeral storage limits cannot exceed 4 Gi.

```yaml title="Example besteffort.yaml"
apiVersion: v1
kind: ResourceQuota
metadata:
  name: besteffort
spec:
  hard:
    pods: "1"
  scopes:
  - BestEffort
# ...
```

where:

`pods`
: Specifies the total number of pods in a non-terminal state with `BestEffort` quality of service that can exist in the project.

`scopes`
: Specifies a restriction on the quota to only match pods that have `BestEffort` quality of service for either memory or CPU.

```yaml title="Example compute-resources-long-running.yaml"
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources-long-running
spec:
  hard:
    pods: "4"
    limits.cpu: "4"
    limits.memory: "2Gi"
    limits.ephemeral-storage: "4Gi"
  scopes:
  - NotTerminating
# ...
```

where:

`pods`
: Specifies the total number of pods in a non-terminal state.

`limits.cpu`
: Specifies that across all pods in a non-terminal state, the sum of CPU limits cannot exceed this value.

`limits.memory`
: Specifies that across all pods in a non-terminal state, the sum of memory limits cannot exceed this value.

`limits.ephemeral-storage`
: Specifies that across all pods in a non-terminal state, the sum of ephemeral storage limits cannot exceed this value.

`scopes`
: Specifies a restriction on the quota that only matches pods where `spec.activeDeadlineSeconds` is set to `nil`. Build pods fall under `NotTerminating` unless the `RestartNever` policy is applied.

```yaml title="Example compute-resources-time-bound.yaml"
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources-time-bound
spec:
  hard:
    pods: "2"
    limits.cpu: "1"
    limits.memory: "1Gi"
    limits.ephemeral-storage: "1Gi"
  scopes:
  - Terminating
# ...
```

where:

`pods`
: Specifies the total number of pods in a non-terminal state.

`limits.cpu`
: Specifies that across all pods in a non-terminal state, the sum of CPU limits cannot exceed this value.

`limits.memory`
: Specifies that across all pods in a non-terminal state, the sum of memory limits cannot exceed this value.

`limits.ephemeral-storage`
: Specifies that across all pods in a non-terminal state, the sum of ephemeral storage limits cannot exceed this value.

`scopes`
: Specifies a restriction on the quota that only matches pods where `spec.activeDeadlineSeconds>=0`. For example, this quota would charge for build pods, but not long running pods such as a web server or database.

```yaml title="Example storage-consumption.yaml"
apiVersion: v1
kind: ResourceQuota
metadata:
  name: storage-consumption
spec:
  hard:
    persistentvolumeclaims: "10"
    requests.storage: "50Gi"
    gold.storageclass.storage.k8s.io/requests.storage: "10Gi"
    silver.storageclass.storage.k8s.io/requests.storage: "20Gi"
    silver.storageclass.storage.k8s.io/persistentvolumeclaims: "5"
    bronze.storageclass.storage.k8s.io/requests.storage: "0"
    bronze.storageclass.storage.k8s.io/persistentvolumeclaims: "0"
# ...
```

where:

`persistentvolumeclaims`
: Specifies the total number of PVCs in a project.

`requests.storage`
: Specifies that across all PVCs in a project, the sum of storage requested cannot exceed this value.

`gold.storageclass.storage.k8s.io/requests.storage`
: Specifies that across all PVCs in a project, the sum of storage requested in the gold storage class cannot exceed this value.

`silver.storageclass.storage.k8s.io/requests.storage`
: Specifies that across all PVCs in a project, the sum of storage requested in the silver storage class cannot exceed this value.

`silver.storageclass.storage.k8s.io/persistentvolumeclaims`
: Specifies that across PVCs in a project, the total number of claims in the silver storage class cannot exceed this value.

`bronze.storageclass.storage.k8s.io/requests.storage`
: Specifies that across all PVCs in a project, the sum of storage requested in the bronze storage class cannot exceed this value. When this is set to `0`, the bronze storage class cannot request storage.

`bronze.storageclass.storage.k8s.io/persistentvolumeclaims`
: Specifies that across all PVCs in a project, the sum of storage requested in the bronze storage class cannot exceed this value. When this is set to `0`, the bronze storage class cannot create claims.

### Creating a quota {#creating-a-quota_using-quotas-and-limit-ranges}

To create a quota, define a `ResourceQuota` object in a file and apply the file to a project. By doing this task, you can restrict aggregate resource consumption and object counts within the project to ensure the project complies with cluster policies.

**Procedure**

- To apply resource constraints to a specific project, create a `ResourceQuota` object by using the OpenShift CLI (`oc`). Run the following `oc create` command with your definition file to enforce the limits on aggregate resource consumption and object counts specified for that namespace:
  ```terminal
  $ oc create -f <resource_quota_definition> [-n <project_name>]
  ```

  ```terminal title="Example command to create a ResourceQuota object"
  $ oc create -f core-object-counts.yaml -n demoproject
  ```

### Creating object count quotas {#creating-object-count-quotas_using-quotas-and-limit-ranges}

To manage the consumption of standard namespaced resource types, create an object count quota. By creating an object count quota within a OpenShift Container Platform project, you can set defined limits on the number of objects, such as `BuildConfig` and `DeploymentConfig` objects.

When you use a resource quota, OpenShift Container Platform charges an object against the quota if the object exists in server storage. These quotas protect against exhaustion of storage resources.

**Procedure**

1. To configure an object count quota for a resource, run the following command:
   ```terminal
   $ oc create quota <name> --hard=count/<resource>.<group>=<quota>,count/<resource>.<group>=<quota>
   ```

   ```terminal title="Example showing object count quota"
   $ oc create quota test --hard=count/deployments.extensions=2,count/replicasets.extensions=4,count/pods=3,count/secrets=4
   resourcequota "test" created
   ```
2. To inspect the detailed status of the object count quota, use the following `oc describe` command:
   ```terminal
   $ oc describe quota test
   ```

   ```terminal title="Example output"
   Name:                         test
   Namespace:                    quota
   Resource                      Used  Hard
   --------                      ----  ----
   count/deployments.extensions  0     2
   count/pods                    0     3
   count/replicasets.extensions  0     4
   count/secrets                 0     4
   ```

   This example limits the listed resources to the hard limit in each project in the cluster.

### Viewing a quota {#viewing-a-quota_using-quotas-and-limit-ranges}

To monitor usage statistics against defined hard limits, navigate to the **Quota** page in the web console. Alternatively, you can use the CLI to view detailed quota information for the project.

**Procedure**

1. Get the list of quotas defined in the project by entering the following command:
   ```terminal title="Example command with a project called demoproject"
   $ oc get quota -n demoproject
   ```

   ```terminal title="Example output"
   NAME                AGE
   besteffort          11m
   compute-resources   2m
   core-object-counts  29m
   ```
2. Describe the target quota by entering the following command:
   ```terminal title="Example command for the core-object-counts quota"
   $ oc describe quota core-object-counts -n demoproject
   ```

   ```terminal title="Example output"
   Name:			core-object-counts
   Namespace:		demoproject
   Resource		Used	Hard
   --------		----	----
   configmaps		3	10
   persistentvolumeclaims	0	4
   replicationcontrollers	3	20
   secrets			9	10
   services		2	10
   ```

### Configuring quota synchronization period {#configuring-quota-synchronization-period_using-quotas-and-limit-ranges}

When a set of resources are deleted, the synchronization time frame of resources is determined by the `resource-quota-sync-period` setting in the `/etc/origin/master/master-config.yaml` file. You can change the `resource-quota-sync-period` setting to have the set of resources regenerate in the needed amount of time (in seconds) for the resources to be once again available.

:::note

Before quota usage is restored, you might encounter problems when attempting to reuse the resources.

:::

Adjusting the regeneration time can be helpful for creating resources and determining resource usage when automation is used.

:::note

The `resource-quota-sync-period` setting balances system performance. Reducing the sync period can result in a heavy load on the controller.

:::

**Procedure**

1. To specify the time required for resources to regenerate and become available again, edit the `resource-quota-sync-period` setting. With this configuration, you can set the synchronization interval in seconds.
   ```yaml title="Example of the resource-quota-sync-period setting"
   kubernetesMasterConfig:
     apiLevels:
     - v1beta3
     - v1
     apiServerArguments: null
     controllerArguments:
       resource-quota-sync-period:
         - "10s"
   # ...
   ```
2. Restart the controller services to apply them to your cluster by entering the following commands:
   ```terminal
   $ master-restart api
   ```

   ```terminal
   $ master-restart controllers
   ```

### Setting a quota to consume a resource {#setting-quota-to-consume-resource_using-quotas-and-limit-ranges}

To restrict the amount of a resource that a user can consume, set a quota. By doing this task, you can prevent unbounded usage of resources, such as storage classes, ensuring that project consumption remains within defined limits.

If a quota does not manage a resource, a user has no restriction on the amount of that resource that can be consumed. For example, if there is no quota on storage related to the gold storage class, the amount of gold storage a project can create is unbounded.

For high-cost compute or storage resources, administrators can require an explicit quota be granted to consume a resource. For example, if a project was not explicitly given quota for storage related to the gold storage class, users of that project would not be able to create any storage of that type.

The example in the procedure shows how the quota system intercepts every operation that creates or updates a `PersistentVolumeClaim` resource. The quota system checks what resources controlled by quota would be consumed. If there is no covering quota for those resources in the project, the request is denied. In this example, if a user creates a `PersistentVolumeClaim` resource that uses storage associated with the gold storage class and there is no matching quota in the project, the request is denied.

**Procedure**

- Add the following stanza to the `master-config.yaml` file. This stanza requires explicit quota to consume a particular resource.
  ```yaml
  admissionConfig:
    pluginConfig:
      ResourceQuota:
        configuration:
          apiVersion: resourcequota.admission.k8s.io/v1alpha1
          kind: Configuration
          limitedResources:
          - resource: persistentvolumeclaims
          matchContains:
          - gold.storageclass.storage.k8s.io/requests.storage
  # ...
  ```

  where:

  <dl>
  <dt><code>configuration.resource</code></dt>
  <dd>Specifies the group or resource whose consumption is limited by default.</dd>
  </dl>

`configuration.matchContains`
: Specifies the name of the resource tracked by quota associated with the group or resource to limit by default.

**Additional resources**

- [Resources managed by quotas](/docs/applications/quotas/quotas-setting-per-project#quotas-resources-managed_quotas-setting-per-project)
- [Working with projects](/docs/applications/projects/working-with-projects#working-with-projects-create-project_working-with-projects-create-project)
- [Understanding deployments](/docs/applications/deployments/what-deployments-are#what-deployments-are_what-deployments-are)

## Limit ranges in a LimitRange object  {#admin-quota-limits_using-quotas-and-limit-ranges}

To define compute resource constraints at the object level, create a `LimitRange` object. By creating this object, you can specify the exact amount of resources that an individual pod, container, image, image stream, or persistent volume claim can consume.

All requests to create and modify resources are evaluated against each `LimitRange` object in the project. If the resource violates any of the enumerated constraints, the resource is rejected. If the resource does not set an explicit value, and if the constraint supports a default value, the default value is applied to the resource.

For CPU and memory limits, if you specify a maximum value but do not specify a minimum limit, the resource can consume more CPU and memory resources than the maximum value.

```yaml title="Core limit range object definition"
apiVersion: "v1"
kind: "LimitRange"
metadata:
  name: "core-resource-limits"
spec:
  limits:
    - type: "Pod"
      max:
        cpu: "2"
        memory: "1Gi"
      min:
        cpu: "200m"
        memory: "6Mi"
    - type: "Container"
      max:
        cpu: "2"
        memory: "1Gi"
      min:
        cpu: "100m"
        memory: "4Mi"
      default:
        cpu: "300m"
        memory: "200Mi"
      defaultRequest:
        cpu: "200m"
        memory: "100Mi"
      maxLimitRequestRatio:
        cpu: "10"
# ...
```

where:

`metadata.name`
: Specifies the name of the limit range object.

`max.cpu`
: Specifies the maximum amount of CPU that a pod can request on a node across all containers.

`max.memory`
: Specifies the maximum amount of memory that a pod can request on a node across all containers.

`min.cpu`
: Specifies the minimum amount of CPU that a pod can request on a node across all containers. If you do not set a `min` value or you set `min` to `0`, the result is no limit and the pod can consume more than the `max` CPU value.

`min.memory`
: Specifies the minimum amount of memory that a pod can request on a node across all containers. If you do not set a `min` value or you set `min` to `0`, the result is no limit and the pod can consume more than the `max` memory value.

`max.cpu`
: Specifies the maximum amount of CPU that a single container in a pod can request.

`max.memory`
: Specifies the maximum amount of memory that a single container in a pod can request.

`min.cpu`
: Specifies the minimum amount of CPU that a single container in a pod can request. If you do not set a `min` value or you set `min` to `0`, the result is no limit and the pod can consume more than the `max` CPU value.

`max.memory`
: Specifies the minimum amount of memory that a single container in a pod can request. If you do not set a `min` value or you set `min` to `0`, the result is no limit and the pod can consume more than the `max` memory value.

`default.cpu`
: Specifies the default CPU limit for a container if you do not specify a limit in the pod specification.

`default.memory`
: Specifies the default memory limit for a container if you do not specify a limit in the pod specification.

`defaultRequest.cpu`
: Specifies the default CPU request for a container if you do not specify a request in the pod specification.

`defaultRequest.memory`
: Specifies the default memory request for a container if you do not specify a request in the pod specification.

`maxLimitRequestRatio.cpu`
: Specifies the maximum limit-to-request ratio for a container.

```yaml title="OpenShift Container Platform Limit range object definition"
apiVersion: "v1"
kind: "LimitRange"
metadata:
  name: "openshift-resource-limits"
spec:
  limits:
    - type: openshift.io/Image
      max:
        storage: 1Gi
    - type: openshift.io/ImageStream
      max:
        openshift.io/image-tags: 20
        openshift.io/images: 30
    - type: "Pod"
      max:
        cpu: "2"
        memory: "1Gi"
        ephemeral-storage: "1Gi"
      min:
        cpu: "1"
        memory: "1Gi"
# ...
```

where:

`limits.max.storage`
: Specifies the maximum size of an image that can be pushed to an internal registry.

`limits.max.openshift.io/image-tags`
: Specifies the maximum number of unique references counted from tag definitions in the `imagestream.spec.tags` resource.

`limits.max.openshift.io/images`
: Specifies the maximum number of unique image identities, or digests, recorded in the `imagestream.status.tags` resource.

`type.max.cpu`
: Specifies the maximum amount of CPU that a pod can request on a node across all containers.

`type.max.memory`
: Specifies the maximum amount of memory that a pod can request on a node across all containers.

`type.max.ephemeral-storage`
: Specifies the maximum amount of ephemeral storage that a pod can request on a node across all containers.

`min.cpu`
: Specifies the minimum amount of CPU that a pod can request on a node across all containers. See the Supported Constraints table for important information.

`min.memory`
: Specifies the minimum amount of memory that a pod can request on a node across all containers. If you do not set a `min` value or you set `min` to `0`, the result is no limit and the pod can consume more than the `max` memory value.

You can specify both core and OpenShift Container Platform resources in one limit range object.

### Container limits {#container-limits_using-quotas-and-limit-ranges}

After you create the `LimitRange` object, you can specify the exact amount of resources that a container can consume.

The following list shows resources that a container can consume:

- CPU
- Memory

The following table shows the supported constraints for a container. If specified, the constraints must hold true for each container.

**Supported constraints**

<table>
<thead>
<tr>
  <th>Constraint</th>
  <th>Behavior</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>Min</code></td>
  <td><code>Min[&lt;resource&gt;]</code> less than or equal to <code>container.resources.requests[&lt;resource&gt;]</code> (required) less than or equal to <code>container/resources.limits[&lt;resource&gt;]</code> (optional)<br><br>If the configuration defines a <code>min</code> CPU, the request value must be greater than the CPU value. If you do not set a <code>min</code> value or you set <code>min</code> to <code>0</code>, the result is no limit and the pod can consume more of the resource than the <code>max</code> value.</td>
</tr>
<tr>
  <td><code>Max</code></td>
  <td><code>container.resources.limits[&lt;resource&gt;]</code> (required) less than or equal to <code>Max[&lt;resource&gt;]</code><br><br>If the configuration defines a <code>max</code> CPU, you do not need to define a CPU request value. However, you must set a limit that satisfies the maximum CPU constraint that is specified in the limit range.</td>
</tr>
<tr>
  <td><code>MaxLimitRequestRatio</code></td>
  <td><code>MaxLimitRequestRatio[&lt;resource&gt;]</code> less than or equal to (<code>container.resources.limits[&lt;resource&gt;]</code> / <code>container.resources.requests[&lt;resource&gt;]</code>)<br><br>If the limit range defines a <code>maxLimitRequestRatio</code> constraint, any new containers must have both a <code>request</code> and a <code>limit</code> value. Additionally, OpenShift Container Platform calculates a limit-to-request ratio by dividing the <code>limit</code> by the <code>request</code>. The result should be an integer greater than 1.<br><br>For example, if a container has <code>cpu: 500</code> in the <code>limit</code> value, and <code>cpu: 100</code> in the <code>request</code> value, the limit-to-request ratio for <code>cpu</code> is <code>5</code>. This ratio must be less than or equal to the <code>maxLimitRequestRatio</code>.</td>
</tr>
</tbody>
</table>

The following list shows default resources that a container can consume:

- `Default[<resource>]`: Defaults `container.resources.limit[<resource>]` to specified value if none.
- `Default Requests[<resource>]`: Defaults `container.resources.requests[<resource>]` to specified value if none.

### Pod limits {#pod-limits_using-quotas-and-limit-ranges}

After you create the `LimitRange` object, you can specify the exact amount of resources that a pod can consume.

A pod can consume the following resources:

- CPU
- Memory

The following table shows the supported constraints for a pod. Across all pods, the following behavior must hold true:

<table>
<thead>
<tr>
  <th>Constraint</th>
  <th>Enforced behavior</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>Min</code></td>
  <td><code>Min[&lt;resource&gt;]</code> less than or equal to <code>container.resources.requests[&lt;resource&gt;]</code> (required) less than or equal to <code>container.resources.limits[&lt;resource&gt;]</code>. If you do not set a <code>min</code> value or you set <code>min</code> to <code>0</code>, the result is no limit and the pod can consume more of the resource than the <code>max</code> value.</td>
</tr>
<tr>
  <td><code>Max</code></td>
  <td><code>container.resources.limits[&lt;resource&gt;]</code> (required) less than or equal to <code>Max[&lt;resource&gt;]</code>.</td>
</tr>
<tr>
  <td><code>MaxLimitRequestRatio</code></td>
  <td><code>MaxLimitRequestRatio[&lt;resource&gt;]</code> less than or equal to (<code>container.resources.limits[&lt;resource&gt;]</code> / <code>container.resources.requests[&lt;resource&gt;]</code>).</td>
</tr>
</tbody>
</table>

### Image limits {#image-limits_using-quotas-and-limit-ranges}

After you create the `LimitRange` object, you can specify the exact amount of resources that an image can consume.

An image can consume the following resources:

- Storage
- `openshift.io/Image`

The following table shows the supported constraints for an image. If specified, the constraints must hold true for each image.

**Image limits**

<table>
<thead>
<tr>
  <th>Constraint</th>
  <th>Behavior</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>Max</code></td>
  <td><code>image.dockerimagemetadata.size</code> less than or equal to <code>Max[&lt;resource&gt;]</code></td>
</tr>
</tbody>
</table>

:::note

To prevent blobs that exceed the limit from being uploaded to the registry, you must configure the registry to enforce quota. The `REGISTRY_MIDDLEWARE_REPOSITORY_OPENSHIFT_ENFORCEQUOTA` environment variable must be set to `true`. By default, the environment variable is set to `true` for new deployments.

:::

### Image stream limits {#image-stream-limits_using-quotas-and-limit-ranges}

After you create the `LimitRange` object, you can specify the exact amount of resources that an image stream can consume.

An image stream can consume the following resources:

- `openshift.io/image-tags`
- `openshift.io/images`
- `openshift.io/ImageStream`

The `openshift.io/image-tags` limit bounds unique references derived from tag definitions in the `imagestream.spec.tags` resource. A reference can be an `ImageStreamTag`, an `ImageStreamImage`, or a `DockerImage`. You can use the `oc tag` and `oc import-image` commands to create tags. Internal and external references are not distinguished, and each unique reference in the spec is counted once. Updates that would exceed the limit are rejected, including updates from pushes to the internal registry that add or change tag definitions.

The `openshift.io/images` limit bounds unique image identities recorded in `imagestream.status.tags`. The name is equivalent to the digest for the image. It limits how many distinct images the stream can reference in status, including from registry pushes. Internal and external references are not distinguished.

:::warning

Do not read `openshift.io/image-tags` and `openshift.io/images` as "tag names versus images per tag." The first limit is computed from the `ImageStream` `spec.tags` resource. The second is computed from the `imagestream.status.tags` resource. Both limits can cause image stream updates to fail when a push or other operation would exceed them.

:::

The following table shows the supported constraints for an image stream. If specified, the constraints must hold true for each image stream.

<table>
<thead>
<tr>
  <th>Constraint</th>
  <th>Behavior</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>Max[openshift.io/image-tags]</code></td>
  <td><code>length( uniqueimagetags( imagestream.spec.tags ) )</code> less than or equal to <code>Max[openshift.io/image-tags]</code><br><br><code>uniqueimagetags</code> returns unique references to images of given spec tags.</td>
</tr>
<tr>
  <td><code>Max[openshift.io/images]</code></td>
  <td><code>length( uniqueimages( imagestream.status.tags ) )</code> less than or equal to <code>Max[openshift.io/images]</code><br><br><code>uniqueimages</code> returns unique image names found in status tags. The name is equal to the digest for the image.</td>
</tr>
</tbody>
</table>

### PersistentVolumeClaim limits {#persistent-volume-claim-limits_using-quotas-and-limit-ranges}

After you create the `LimitRange` object, you can specify the exact amount of resources that a `PersistentVolumeClaim` resource can consume.

A `PersistentVolumeClaim` resource can consume storage resources.

The following table shows the supported constraints for a persistent volume claim. If specified, the constraints must hold true for each persistent volume claim.

`PersistentVolumeClaim`** resource limits**

<table>
<thead>
<tr>
  <th>Constraint</th>
  <th>Enforced behavior</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>Min</code></td>
  <td>Min[<resource>] +<=+ claim.spec.resources.requests[<resource>] (required)</td>
</tr>
<tr>
  <td><code>Max</code></td>
  <td>claim.spec.resources.requests[<resource>] (required) +<=+ Max[<resource>]</td>
</tr>
</tbody>
</table>

```json title="Limit range object definition example"
{
  "apiVersion": "v1",
  "kind": "LimitRange",
  "metadata": {
    "name": "pvcs"
  },
  "spec": {
    "limits": [{
        "type": "PersistentVolumeClaim",
        "min": {
          "storage": "2Gi"
        },
        "max": {
          "storage": "50Gi"
        }
      }
    ]
  }
}
```

where:

`metadata.name`
: Specifies the name of the limit range object.

`limits.min.storage`
: Specifies the minimum amount of storage that can be requested in a persistent volume claim.

`limits.max.storage`
: Specifies the maximum amount of storage that can be requested in a persistent volume claim.

**Additional resources**

- [Managing images streams](/docs/openshift_images/image-streams-manage#images-imagestream-use_image-streams-managing)
- [Restrict resource consumption with limit ranges](/docs/nodes/clusters/nodes-cluster-limit-ranges#nodes-cluster-limit-stream-limits_nodes-cluster-limit-stream-limits)
- [About limit ranges](/docs/nodes/clusters/nodes-cluster-limit-ranges#nodes-cluster-limit-ranges-about_nodes-cluster-limit-ranges)
- [Recommended control plane practices](/docs/scalability_and_performance/recommended-performance-scale-practices/recommended-control-plane-practices#recommended-scale-practices_recommended-control-plane-practices)
- [Understanding ephemeral storage](/docs/storage/understanding-ephemeral-storage#storage-ephemeral-storage-overview_understanding-ephemeral-storage)

## Limit range operations {#admin-limit-operations_using-quotas-and-limit-ranges}

You can create, view, and delete limit ranges in a project.

You can view any limit ranges that are defined in a project by navigating in the web console to the **Quota** page for the project. You can also use the CLI to view limit range details.

**Procedure**

- To create the object, enter the following command:
  ```terminal
  $ oc create -f <limit_range_file> -n <project>
  ```
- To view the list of limit range objects that exist in a project, enter the following command:
  ```terminal title="Example command with a project called demoproject"
  $ oc get limits -n demoproject
  ```

  ```terminal title="Example output"
  NAME              AGE
  resource-limits   6d
  ```
- To describe a limit range, enter the following command:
  ```terminal title="Example command with a limit range called resource-limits"
  $ oc describe limits resource-limits -n demoproject
  ```

  ```terminal title="Example output"
  Name:                           resource-limits
  Namespace:                      demoproject
  Type                            Resource                Min     Max     Default Request Default Limit   Max Limit/Request Ratio
  ----                            --------                ---     ---     --------------- -------------   -----------------------
  Pod                             cpu                     200m    2       -               -               -
  Pod                             memory                  6Mi     1Gi     -               -               -
  Container                       cpu                     100m    2       200m            300m            10
  Container                       memory                  4Mi     1Gi     100Mi           200Mi           -
  openshift.io/Image              storage                 -       1Gi     -               -               -
  openshift.io/ImageStream        openshift.io/image      -       12      -               -               -
  openshift.io/ImageStream        openshift.io/image-tags -       10      -               -               -
  ```
- To delete a limit range, enter the following command:
  ```terminal
  $ oc delete limits <limit_name>
  ```

**Additional resources**

- [Resource quotas per projects](/docs/applications/quotas/quotas-setting-per-project#quotas-setting-per-project_quotas-setting-per-project)
