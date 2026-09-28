---
title: Compliance Operator scans
sidebar_position: 2
---

# Compliance Operator scans

<a id="compliance-operator-scans"></a>

You can use the `ScanSetting` and `ScanSettingBinding` APIs to run compliance scans with the Compliance Operator.

For more information on these API objects, run the following command:

```terminal
$ oc explain scansettings
```

or

```terminal
$ oc explain scansettingbindings
```

## Run compliance scans {#running-compliance-scans_compliance-operator-scans}

You can run a scan using the Center for Internet Security (CIS) profiles to evaluate cluster compliance against CIS benchmarks. For convenience, the Compliance Operator creates a `ScanSetting` object with reasonable defaults on startup. This `ScanSetting` object is named `default`.

:::note

For all-in-one control plane and worker nodes, the compliance scan runs twice on the worker and control plane nodes. The compliance scan might generate inconsistent scan results. You can avoid inconsistent results by defining only a single role in the `ScanSetting` object.

:::

:::warning

Compliance Operator scans report `INCONSISTENT` on clusters with multi-architecture compute machines whether the control plane uses `aarch64` or `x86` CPUs. This is due to the same rule behaving differently on different architectures. This applies only to node scans, where the Compliance Operator aggregates results from multiple nodes into a single result.

:::

For more information about inconsistent scan results, see Compliance Operator shows INCONSISTENT scan result with worker node.

**Procedure**

1. Inspect the `ScanSetting` object by running the following command:
   ```terminal
   $ oc describe scansettings default -n openshift-compliance
   ```

   ```yaml title="Example output"
   Name:                  default
   Namespace:             openshift-compliance
   Labels:                <none>
   Annotations:           <none>
   API Version:           compliance.openshift.io/v1alpha1
   Kind:                  ScanSetting
   Max Retry On Timeout:  3
   Metadata:
     Creation Timestamp:  2024-07-16T14:56:42Z
     Generation:          2
     Resource Version:    91655682
     UID:                 50358cf1-57a8-4f69-ac50-5c7a5938e402
   Raw Result Storage:
     Node Selector:
       node-role.kubernetes.io/master:
     Pv Access Modes:
       ReadWriteOnce
     Rotation:            3
     Size:                1Gi
     Storage Class Name:  standard
     Tolerations:
       Effect:              NoSchedule
       Key:                 node-role.kubernetes.io/master
       Operator:            Exists
       Effect:              NoExecute
       Key:                 node.kubernetes.io/not-ready
       Operator:            Exists
       Toleration Seconds:  300
       Effect:              NoExecute
       Key:                 node.kubernetes.io/unreachable
       Operator:            Exists
       Toleration Seconds:  300
       Effect:              NoSchedule
       Key:                 node.kubernetes.io/memory-pressure
       Operator:            Exists
   Roles:
     master
     worker
   Scan Tolerations:
     Operator:           Exists
   Schedule:             0 1 * * *
   Show Not Applicable:  false
   Strict Node Scan:     true
   Suspend:              false
   Timeout:              30m
   Events:               <none>

   ```

   where:

   <dl>
   <dt><code>Raw Result Storage.pvAccessModes.ReadWriteOnce</code></dt>
   <dd>Specifies access mode for the PV created by the Compliance Operator with the results of the scans. By default, the PV will use access mode <code>ReadWriteOnce</code> because the Compliance Operator cannot make any assumptions about the storage classes configured on the cluster. Additionally, <code>ReadWriteOnce</code> access mode is available on most clusters. If you need to fetch the scan results, you can do so by using a helper pod, which also binds the volume. Volumes that use the <code>ReadWriteOnce</code> access mode can be mounted by only one pod at time, so it is important to remember to delete the helper pods. Otherwise, the Compliance Operator will not be able to reuse the volume for subsequent scans.</dd>
   <dt><code>Raw Result Storage.Rotation</code></dt>
   <dd>Specifies that the Compliance Operator keeps the results of three subsequent scans in the volume; older scans are rotated.</dd>
   <dt><code>Raw Result Storage.size</code></dt>
   <dd>Specifies that the Compliance Operator will allocate one GB of storage for the scan results.</dd>
   <dt><code>Raw Result Storage.Storage Class Name</code></dt>
   <dd>Specifies the <code>storageClassName</code> value to use when creating the <code>PersistentVolumeClaim</code> object to store the raw results. The default value is null, which will attempt to use the default storage class configured in the cluster. If there is no default class specified, then you must set a default class.</dd>
   <dt><code>Roles</code></dt>
   <dd>If the scan setting uses any profiles that scan cluster nodes, scan these node roles.</dd>
   <dt><code>Scan Tolerations</code></dt>
   <dd>The default scan setting object scans all the nodes.</dd>
   <dt><code>Schedule</code></dt>
   <dd>The default scan setting object runs scans at 01:00 each day. As an alternative to the default scan setting, you can use <code>default-auto-apply</code>, which has the following settings:</dd>
   </dl>

   ```yaml
   Name:                      default-auto-apply
   Namespace:                 openshift-compliance
   Labels:                    <none>
   Annotations:               <none>
   API Version:               compliance.openshift.io/v1alpha1
   Auto Apply Remediations:   true
   Auto Update Remediations:  true
   Kind:                      ScanSetting
   Metadata:
     Creation Timestamp:  2022-10-18T20:21:00Z
     Generation:          1
     Managed Fields:
       API Version:  compliance.openshift.io/v1alpha1
       Fields Type:  FieldsV1
       fieldsV1:
         f:autoApplyRemediations:
         f:autoUpdateRemediations:
         f:rawResultStorage:
           .:
           f:nodeSelector:
             .:
             f:node-role.kubernetes.io/master:
           f:pvAccessModes:
           f:rotation:
           f:size:
           f:tolerations:
         f:roles:
         f:scanTolerations:
         f:schedule:
         f:showNotApplicable:
         f:strictNodeScan:
       Manager:         compliance-operator
       Operation:       Update
       Time:            2022-10-18T20:21:00Z
     Resource Version:  38840
     UID:               8cb0967d-05e0-4d7a-ac1c-08a7f7e89e84
   Raw Result Storage:
     Node Selector:
       node-role.kubernetes.io/master:
     Pv Access Modes:
       ReadWriteOnce
     Rotation:  3
     Size:      1Gi
     Tolerations:
       Effect:              NoSchedule
       Key:                 node-role.kubernetes.io/master
       Operator:            Exists
       Effect:              NoExecute
       Key:                 node.kubernetes.io/not-ready
       Operator:            Exists
       Toleration Seconds:  300
       Effect:              NoExecute
       Key:                 node.kubernetes.io/unreachable
       Operator:            Exists
       Toleration Seconds:  300
       Effect:              NoSchedule
       Key:                 node.kubernetes.io/memory-pressure
       Operator:            Exists
   Roles:
     master
     worker
   Scan Tolerations:
     Operator:           Exists
   Schedule:             0 1 * * *
   Show Not Applicable:  false
   Strict Node Scan:     true
   Events:               <none>
   ```

- Setting `autoUpdateRemediations` and `autoApplyRemediations` flags to `true` allows you to easily create `ScanSetting` objects that auto-remediate without extra steps.

1. Create a `ScanSettingBinding` object that binds to the default `ScanSetting` object and scans the cluster using the `cis` and `cis-node` profiles. For example:
   ```yaml
   apiVersion: compliance.openshift.io/v1alpha1
   kind: ScanSettingBinding
   metadata:
     name: cis-compliance
     namespace: openshift-compliance
   profiles:
     - name: ocp4-cis-node
       kind: Profile
       apiGroup: compliance.openshift.io/v1alpha1
     - name: ocp4-cis
       kind: Profile
       apiGroup: compliance.openshift.io/v1alpha1
   settingsRef:
     name: default
     kind: ScanSetting
     apiGroup: compliance.openshift.io/v1alpha1
   ```
2. Create the `ScanSettingBinding` object by running:
   ```terminal
   $ oc create -f <file-name>.yaml -n openshift-compliance
   ```

   At this point in the process, the `ScanSettingBinding` object is reconciled and based on the `Binding` and the `Bound` settings. The Compliance Operator creates a `ComplianceSuite` object and the associated `ComplianceScan` objects.
3. Follow the compliance scan progress by running:
   ```terminal
   $ oc get compliancescan -w -n openshift-compliance
   ```

   The scans progress through the scanning phases and eventually reach the `DONE` phase when complete. In most cases, the result of the scan is `NON-COMPLIANT`. You can review the scan results and start applying remediations to make the cluster compliant.

## Set custom storage size for results {#compliance-custom-storage_compliance-operator-scans}

Although `ComplianceCheckResult` custom resources summarize one check across all scanned nodes, raw scanner results in ARF format are too large to store in etcd-backed Kubernetes resources. You can store them on a per-scan persistent volume and increase the default 1 GiB size by setting the `rawResultStorage.size` value in a `ScanSetting` or `ComplianceScan` resource.

A related parameter is `rawResultStorage.rotation` which controls how many scans are retained in the PV before the older scans are rotated. The default value is 3, setting the rotation policy to 0 disables the rotation. Given the default rotation policy and an estimate of 100MB per a raw ARF scan report, you can calculate the right PV size for your environment.

Because OpenShift Container Platform can be deployed in a variety of public clouds or bare metal, the Compliance Operator cannot determine available storage configurations. By default, the Compliance Operator will try to create the PV for storing results by using the default storage class of the cluster, but a custom storage class can be configured using the `rawResultStorage.StorageClassName` attribute.

:::warning

If your cluster does not specify a default storage class, this attribute must be set.

:::

- Configure the `ScanSetting` custom resource to use a standard storage class and create persistent volumes that are 10GB in size and keep the last 10 results:
  ```yaml title="Example ScanSetting CR"
  apiVersion: compliance.openshift.io/v1alpha1
  kind: ScanSetting
  metadata:
    name: default
    namespace: openshift-compliance
  rawResultStorage:
    storageClassName: standard
    rotation: 10
    size: 10Gi
  roles:
  - worker
  - master
  scanTolerations:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
    operator: Exists
  schedule: '0 1 * * *'
  ```

## Schedule the result server pod on a worker node {#running-compliance-scans-worker-node_compliance-operator-scans}

The result server pod mounts the persistent volume (PV) that stores the raw Asset Reporting Format (ARF) scan results. You can use the `nodeSelector` and `tolerations` attributes to configure the location of the result server pod to meet your organization’s requirements.

This is helpful for those environments where control plane nodes are not permitted to mount persistent volumes.

**Procedure**

- Create a `ScanSetting` custom resource (CR) for the Compliance Operator:
  1. Define the `ScanSetting` CR, and save the YAML file, for example, `rs-workers.yaml`:
     ```yaml
     apiVersion: compliance.openshift.io/v1alpha1
     kind: ScanSetting
     metadata:
       name: rs-on-workers
       namespace: openshift-compliance
     rawResultStorage:
       nodeSelector:
         node-role.kubernetes.io/worker: ""
       pvAccessModes:
       - ReadWriteOnce
       rotation: 3
       size: 1Gi
       tolerations:
       - operator: Exists
     roles:
     - worker
     - master
     scanTolerations:
       - operator: Exists
     schedule: 0 1 * * *
     ```

     where:

     <dl>
     <dt><code>rawResultStorage.nodeSelector.node-role.kubernetes.io/worker</code></dt>
     <dd>Specifies the Compliance Operator uses this node to store scan results in ARF format.</dd>
     <dt><code>rawResultStorage.tolerations.operator</code></dt>
     <dd>Specifies the result server pod tolerates all taints.</dd>
     </dl>
  2. To create the `ScanSetting` CR, run the following command:
     ```terminal
     $ oc create -f rs-workers.yaml
     ```

**Verification**

- To verify that the `ScanSetting` object is created, run the following command:
  ```terminal
  $ oc get scansettings rs-on-workers -n openshift-compliance -o yaml
  ```

  ```terminal title="Example output"
  apiVersion: compliance.openshift.io/v1alpha1
  kind: ScanSetting
  metadata:
    creationTimestamp: "2021-11-19T19:36:36Z"
    generation: 1
    name: rs-on-workers
    namespace: openshift-compliance
    resourceVersion: "48305"
    uid: 43fdfc5f-15a7-445a-8bbc-0e4a160cd46e
  rawResultStorage:
    nodeSelector:
      node-role.kubernetes.io/worker: ""
    pvAccessModes:
    - ReadWriteOnce
    rotation: 3
    size: 1Gi
    tolerations:
    - operator: Exists
  roles:
  - worker
  - master
  scanTolerations:
  - operator: Exists
  schedule: 0 1 * * *
  strictNodeScan: true
  ```

## `ScanSetting` Custom Resource {#compliance-scansetting-cr_compliance-operator-scans}

You can configure the scan limits attribute of the `ScanSetting` custom resource to override the default CPU and memory limits of scanner pods to meet your environment’s resource requirements.

The Compliance Operator uses defaults of 500Mi memory and 100m CPU for the scanner container, and 200Mi memory and 100m CPU for the `api-resource-collector` container. To set the memory limits of the Operator, modify the `Subscription` object if installed through OLM or the Operator deployment itself.

:::warning

Increasing the memory limit for the Compliance Operator or the scanner pods is needed if the default limits are not sufficient and the Operator or scanner pods are ended by the Out Of Memory (OOM) process. For more information, see Increasing Compliance Operator resource limits.

:::

## Configure the hosted control planes management cluster {#co-hcp-mgmt-config_compliance-operator-scans}

If you are hosting your own Hosted control planes or Hypershift environment and want to scan a Hosted Cluster from the management cluster, you will need to set the name and prefix namespace for the target Hosted Cluster. You can achieve this by creating a `TailoredProfile`.

:::warning

This procedure only applies to users managing their own hosted control planes environment.

:::

:::note

Only `ocp4-cis` and `ocp4-pci-dss` profiles are supported in hosted control planes management clusters.

:::

**Prerequisites**

- The Compliance Operator is installed in the management cluster.

**Procedure**

1. Obtain the `name` and `namespace` of the hosted cluster to be scanned by running the following command:
   ```terminal
   $ oc get hostedcluster -A
   ```

   ```terminal title="Example output"
   NAMESPACE       NAME                   VERSION   KUBECONFIG                              PROGRESS    AVAILABLE   PROGRESSING   MESSAGE
   local-cluster   79136a1bdb84b3c13217   4.13.5    79136a1bdb84b3c13217-admin-kubeconfig   Completed   True        False         The hosted control plane is available
   ```
2. In the management cluster, create a `TailoredProfile` extending the scan Profile and define the name and namespace of the Hosted Cluster to be scanned:
   ```yaml title="Example management-tailoredprofile.yaml"
   apiVersion: compliance.openshift.io/v1alpha1
   kind: TailoredProfile
   metadata:
     name: hypershift-cisk57aw88gry
     namespace: openshift-compliance
   spec:
     description: This profile test required rules
     extends: ocp4-cis
     title: Management namespace profile
     setValues:
     - name: ocp4-hypershift-cluster
       rationale: This value is used for HyperShift version detection
       value: 79136a1bdb84b3c13217
     - name: ocp4-hypershift-namespace-prefix
       rationale: This value is used for HyperShift control plane namespace detection
       value: local-cluster
   ```

   where:

   <dl>
   <dt><code>spec.extends</code></dt>
   <dd>Specifies the name of the <code>Profile</code> object upon which the <code>TailoredProfile</code> is built. Only <code>ocp4-cis</code> and <code>ocp4-pci-dss</code> profiles are supported in hosted control planes management clusters.</dd>
   <dt><code>spec.setValues.value</code></dt>
   <dd>Specifies the output in the previous step.</dd>
   <dt><code>spec.setValues.value</code></dt>
   <dd>Specifies the <code>NAMESPACE</code> from the output in the previous step.</dd>
   </dl>
3. Create the `TailoredProfile`:
   ```terminal
   $ oc create -n openshift-compliance -f mgmt-tp.yaml
   ```

## Apply resource requests and limits {#compliance-applying-resource-requests-and-limits_compliance-operator-scans}

You can configure a container’s requests and limits for memory and CPU to define how much CPU time and memory that the container can use.

When the kubelet starts a container as part of a Pod, the kubelet passes that container’s requests and limits for memory and CPU to the container runtime. In Linux, the container runtime configures the kernel cgroups that apply and enforce the limits you defined.

The CPU limit defines how much CPU time the container can use. During each scheduling interval, the Linux kernel checks to see if this limit is exceeded. If so, the kernel waits before allowing the cgroup to resume execution.

If several different containers (cgroups) want to run on a contended system, workloads with larger CPU requests are allocated more CPU time than workloads with small requests. The memory request is used during Pod scheduling. On a node that uses cgroups v2, the container runtime might use the memory request as a hint to set `memory.min` and `memory.low` values.

If a container attempts to allocate more memory than this limit, the Linux kernel out-of-memory subsystem activates and intervenes by stopping one of the processes in the container that tried to allocate memory. The memory limit for the Pod or container can also apply to pages in memory-backed volumes, such as an emptyDir.

The kubelet tracks `tmpfs` `emptyDir` volumes as container memory is used, rather than as local ephemeral storage. If a container exceeds its memory request and the node that it runs on becomes short of memory overall, the Pod’s container might be evicted.

:::warning

A container might not exceed its CPU limit for extended periods. Container run times do not stop Pods or containers for excessive CPU usage. To determine whether a container cannot be scheduled or is being killed due to resource limits, see *Troubleshooting the Compliance Operator*.

:::

## Schedule Pods with container resource requests {#compliance-scheduling-pods-with-resource-requests_compliance-operator-scans}

You can specify CPU and memory resource requests and limits for containers to ensure that pods are placed on nodes with sufficient capacity, preventing resource shortages.

Although memory or CPU resource usage on nodes is very low, the scheduler might still refuse to place a Pod on a node if the capacity check fails to protect against a resource shortage on a node.

For each container, you can specify the following resource limits and request:

```terminal
spec.containers[].resources.limits.cpu
spec.containers[].resources.limits.memory
spec.containers[].resources.limits.hugepages-<size>
spec.containers[].resources.requests.cpu
spec.containers[].resources.requests.memory
spec.containers[].resources.requests.hugepages-<size>
```

Although you can specify requests and limits for only individual containers, it is also useful to consider the overall resource requests and limits for a pod. For a particular resource, a container resource request or limit is the sum of the resource requests or limits of that type for each container in the pod.

```yaml title="Example container resource requests and limits"
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  securityContext:
    runAsNonRoot: true
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: images.my-company.example/app:v4
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop: [ALL]
  - name: log-aggregator
    image: images.my-company.example/log-aggregator:v6
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop: [ALL]
```

where:

`spec.containers.resources.requests`
: Specifies that the container is requesting 64 Mi of memory and 250 m CPU.

`spec.containers.resources.limits`
: Specifies the container’s limits are 128 Mi of memory and 500 m CPU.

**Additional resources**

- [Increasing Compliance Operator resource limits](/docs/security/compliance_operator/co-scans/compliance-operator-troubleshooting#compliance-increasing-operator-limits_compliance-troubleshooting)
- [Compliance Operator shows INCONSISTENT scan result with worker node](https://access.redhat.com/solutions/6970861)
- [Managing Compliance Operator result and remediation](/docs/security/compliance_operator/co-scans/compliance-operator-remediation#compliance-operator-remediation)
