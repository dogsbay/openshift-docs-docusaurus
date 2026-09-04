---
title: Admission fair sharing
sidebar_position: 14
---

# Admission fair sharing

<a id="admission-fair-sharing"></a>

Use admission fair sharing to fairly distribute workloads across local Queues that share a single `ClusterQueue`.

You can balance workload admission by prioritizing workloads from local Queues that have used fewer resources historically. With admission fair sharing, you can track usage over time with a configurable decay function and apply admission penalties when workloads are admitted.

When multiple tenants share a single `ClusterQueue`, some tenants risk resource starvation. Admission fair sharing adresses this issue by meeting the following requirements:

Enforce multi-tenant fairness (business critical)
: Ensure fair distribution of cluster resources across all tenants based on their usage history.

Improve service predictability
: Guarantee each tenant gets a consistent share of resources, reducing latency spikes and preventing starvation.

Enable scalable governance
: Complement static quotas with dynamic, usage-based admission ordering that adapts as tenant demand changes.

## Configuring the Red Hat build of Kueue instance for admission fair sharing {#configuring-kueue-instance-for-admission-fair-sharing_admission-fair-sharing}

Configure Red Hat build of Kueue admission fair sharing using either the `Default` or `Custom` configuration.

**Procedure**

1. Choose the `configuration` type you want to use:
   - `Default`: Uses predefined values.
   - `Custom`: Uses values that you specify.
2. Apply your chosen configuration:
   - Use the following command to create a `Default` configuration:
     ```terminal
     $ oc patch kueue.kueue.openshift.io/cluster --type=merge -p \ '{"spec":{"config":{"admissionFairSharing":{"configuration":"Default","custom":null}}}}'
     ```

     ```yaml title="Example of Kueue instance output"
     config:
         admissionFairSharing:
           configuration: Default
     ```
   - Use the following command to create a `Custom` configuration that applies values that you specify:
     ```terminal
     $ oc patch kueue.kueue.openshift.io/cluster --type=merge -p \ '{"spec":{"config":{"admissionFairSharing":{"configuration":"Custom","custom":{"usageHalfLifeTimeSeconds":10,"usageSamplingIntervalSeconds":10,"resourceWeights":[{"name":"cpu","weight":"2.0"}]}}}}}'
     ```

     ```terminal title="Example of Kueue instance output"
       config:
         admissionFairSharing:
           configuration: Custom
           custom:
             resourceWeights:
             - name: cpu
               weight: "2.0"
             usageHalfLifeTimeSeconds: 10
             usageSamplingIntervalSeconds: 10
     ```

     <dl>
     <dt><code>resourceWeights</code></dt>
     <dd>Assigns weights to resources. The higher the weight, the higher the penalty.</dd>
     </dl>

`usageHalfLifeTimeSeconds`
: The time in seconds after which the current usage will decrease by half. That is, it controls how long the past consumption should impact future admission.

`usageSamplingIntervalSeconds`
: The frequency in seconds that Red Hat build of Kueue updates the `consumedResources` component in the `FairSharingStatus` component.

### Set resource weights {#setting-resource-weights_admission-fair-sharing}

Resources measured in bytes, like memory, require scaled-down `resourceWeights` values. Kubernetes represents memory in bytes, creating values that are billions of times larger than CPU core counts.

This numeric difference makes CPU weights ineffective unless you scale memory weights down. Without this adjustment, the raw byte value of these resources will numerically dominate human-scale resources, such as CPU cores, by several orders of magnitude, effectively making their weights meaningless.

For example, if you want to achieve an effective memory weight of `1.0`, you would need to instead specify `9.31e-10`, which corresponds to `1.0 / 1,073,741,824`.

## Configuring a cluster queue for admission fair sharing {#configuring-clusterqueue-for-admission-fair-sharing_admission-fair-sharing}

Configure the `admissionScope` section in your `ClusterQueue` object to be `UsageBasedAdmissionFairSharing`.

**Procedure**

- Specify `UsageBasedAdmissionFairSharing` as shown in the following example:
  ```yaml
  apiVersion: kueue.x-k8s.io/v1beta2
  kind: ClusterQueue
  metadata:
    name: shared-queue
  spec:
    namespaceSelector: {}
    admissionScope:
      admissionMode: UsageBasedAdmissionFairSharing
    resourceGroups:
      - coveredResources: ["cpu", "memory"]
        flavors:
          - name: afs-rf
            resources:
              - name: cpu
                nominalQuota: 2
              - name: memory
                nominalQuota: 2Gi
  ```

## Configuring a local queue for admission fair sharing (optional) {#configuring-localqueue-for-admission-fair-sharing_admission-fair-sharing}

Optionally, you can configure `fairSharing` section in your `LocalQueue` object to adjust its weight in the fair sharing calculation. The higher the weight, the lower the penalty. For example, specifying a weight of `2` treats the queue as if it is used by half as many resources.

**Procedure**

- Specify a `weight` value as shown in the following example:
  ```yaml
  apiVersion: kueue.x-k8s.io/v1beta2
  kind: LocalQueue
  metadata:
    name: team-a-queue
    namespace: team-a
  spec:
    clusterQueue: shared-queue
    fairSharing:
      weight: "2"  # This queue will be treated as if it used half as many resources
  ```

## Verifying the admission fair sharing status {#verifying-the-admission-fair-sharing-status_admission-fair-sharing}

Check the `admissionFairSharingStatus` status in the local queue.

**Procedure**

- Use the following command to verify the status of admission fair sharing:
  ```terminal
  $ oc get lq <local-queue-name> -n <local-queue-namespace> -o jsonpath={.status.fairSharing}
  ```

  ```terminal title="Example output"
  {"admissionFairSharingStatus":{"consumedResources":{"cpu":"31999m"},"lastUpdate":"2025-06-03T14:25:15Z"},"weightedShare":0}
  ```
