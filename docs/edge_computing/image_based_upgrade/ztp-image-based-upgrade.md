---
title: Performing an image-based upgrade for single-node OpenShift clusters using GitOps ZTP
sidebar_position: 4
---

# Performing an image-based upgrade for single-node OpenShift clusters using GitOps ZTP {#ztp-image-based-upgrade}

<a id="ztp-image-based-upgrade"></a>

You can use a single resource on the hub cluster, the `ImageBasedGroupUpgrade` custom resource (CR), to manage an imaged-based upgrade on a selected group of managed clusters through all stages. Topology Aware Lifecycle Manager (TALM) reconciles the `ImageBasedGroupUpgrade` CR and creates the underlying resources to complete the defined stage transitions, either in a manually controlled or a fully automated upgrade flow.

For more information about the image-based upgrade, see "Understanding the image-based upgrade for single-node OpenShift clusters".

**Additional resources**

- [Understanding the image-based upgrade for single-node OpenShift clusters](/docs/edge_computing/image_based_upgrade/cnf-understanding-image-based-upgrade#cnf-understanding-image-based-upgrade)

## Managing the image-based upgrade at scale using the ImageBasedGroupUpgrade CR on the hub {#ztp-image-based-upgrade-concept_ztp-gitops}

The `ImageBasedGroupUpgrade` CR combines the `ImageBasedUpgrade` and `ClusterGroupUpgrade` APIs. For example, you can define the cluster selection and rollout strategy with the `ImageBasedGroupUpgrade` API in the same way as the `ClusterGroupUpgrade` API. The stage transitions are different from the `ImageBasedUpgrade` API. You can use the `ImageBasedGroupUpgrade` API to combine several stage transitions, also called actions, into one step that share one rollout strategy.

**Example **`ImageBasedGroupUpgrade.yaml`

```yaml
apiVersion: lcm.openshift.io/v1alpha1
kind: ImageBasedGroupUpgrade
metadata:
  name: <filename>
  namespace: default
spec:
  clusterLabelSelectors:
    - matchExpressions:
      - key: name
        operator: In
        values:
        - spoke1
        - spoke4
        - spoke6
  ibuSpec:
    seedImageRef:
      image: quay.io/seed/image:4.22.0
      version: 4.22.0
      pullSecretRef:
        name: "<seed_pull_secret>"
    extraManifests:
      - name: example-extra-manifests
        namespace: openshift-lifecycle-agent
    oadpContent:
      - name: oadp-cm
        namespace: openshift-adp
  plan:
    - actions: ["Prep", "Upgrade", "FinalizeUpgrade"]
      rolloutStrategy:
        maxConcurrency: 200
        timeout: 2400
```

Where:

- `clusterLabelSelectors`: Clusters to upgrade.
- `seedImageRef`: Target platform version, the seed image, and the secret required to access the image.

:::note

If you add the seed image pull secret in the hub cluster, in the same namespace as the `ImageBasedGroupUpgrade` resource, the secret is added to the manifest list for the `Prep` stage. The secret is recreated in each spoke cluster in the `openshift-lifecycle-agent` namespace.

:::

- `extraManifests`: Optional: Applies additional manifests, which are not in the seed image, to the target cluster. Also applies `ConfigMap` objects for custom catalog sources.
- `oadpContent`: `ConfigMap` resources that contain the OADP `Backup` and `Restore` CRs.
- `plan`: Upgrade plan details.
- `maxConcurrency`: Number of clusters to update in a batch.
- `timeout`: Timeout limit to complete the action in minutes.

### Supported action combinations {#ztp-image-based-upgrade-supported-combinations_ztp-gitops}

Actions are the list of stage transitions that TALM completes in the steps of an upgrade plan for the selected group of clusters. Each `action` entry in the `ImageBasedGroupUpgrade` CR is a separate step and a step has one or several actions that share the same rollout strategy. You can achieve more control over the rollout strategy for each action by separating actions into steps.

You can combine these actions differently in your upgrade plan and you can add the next steps later. Wait until the earlier steps either complete or fail before adding a step to your plan. The first action of an added step for clusters that failed a earlier steps must be either `Abort` or `Rollback`.

:::warning

You cannot remove actions or steps from an ongoing plan.

:::

The following table shows example plans for different levels of control over the rollout strategy:

**Example upgrade plans**

<table>
<thead>
<tr>
  <th>Example plan</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><pre><code>plan:&#10;- actions: ["Prep", "Upgrade", "FinalizeUpgrade"]&#10;  rolloutStrategy:&#10;    maxConcurrency: 200&#10;    timeout: 60</code></pre></td>
  <td>All actions share the same strategy</td>
</tr>
<tr>
  <td><pre><code>plan:&#10;- actions: ["Prep", "Upgrade"]&#10;  rolloutStrategy:&#10;    maxConcurrency: 200&#10;    timeout: 60&#10;- actions: ["FinalizeUpgrade"]&#10;  rolloutStrategy:&#10;    maxConcurrency: 500&#10;    timeout: 10</code></pre></td>
  <td>Some actions share the same strategy</td>
</tr>
<tr>
  <td><pre><code>plan:&#10;- actions: ["Prep"]&#10;  rolloutStrategy:&#10;    maxConcurrency: 200&#10;    timeout: 60&#10;- actions: ["Upgrade"]&#10;  rolloutStrategy:&#10;    maxConcurrency: 200&#10;    timeout: 20&#10;- actions: ["FinalizeUpgrade"]&#10;  rolloutStrategy:&#10;    maxConcurrency: 500&#10;    timeout: 10</code></pre></td>
  <td>All actions have different strategies</td>
</tr>
</tbody>
</table>

:::warning

Clusters that fail one of the actions will skip the remaining actions in the same step.

:::

The `ImageBasedGroupUpgrade` API accepts the following actions:

`Prep`
: Start preparing the upgrade resources by moving to the `Prep` stage.

`Upgrade`
: Start the upgrade by moving to the `Upgrade` stage.

`FinalizeUpgrade`
: Complete the upgrade on selected clusters that completed the `Upgrade` action by moving to the `Idle` stage.

`Rollback`
: Start a rollback only on successfully upgraded clusters by moving to the `Rollback` stage.

`FinalizeRollback`
: Complete the rollback by moving to the `Idle` stage.

`AbortOnFailure`
: Cancel the upgrade on selected clusters that failed the `Prep` or `Upgrade` actions by moving to the `Idle` stage.

`Abort`
: Cancel an ongoing upgrade only on clusters that are not yet upgraded by moving to the `Idle` stage.

The following action combinations are supported. A pair of brackets signifies one step in the `plan` section:

- `["Prep"]`, `["Abort"]`
- `["Prep", "Upgrade", "FinalizeUpgrade"]`
- `["Prep"]`, `["AbortOnFailure"]`, `["Upgrade"]`, `["AbortOnFailure"]`, `["FinalizeUpgrade"]`
- `["Rollback", "FinalizeRollback"]`

Use one of the following combinations when you need to resume or cancel an ongoing upgrade from a completely new `ImageBasedGroupUpgrade` CR:

- `["Upgrade","FinalizeUpgrade"]`
- `["FinalizeUpgrade"]`
- `["FinalizeRollback"]`
- `["Abort"]`
- `["AbortOnFailure"]`

### Labeling for cluster selection {#ztp-image-based-upgrade-cluster-labeling_ztp-gitops}

Use the `spec.clusterLabelSelectors` field for initial cluster selection. In addition, TALM labels the managed clusters according to the results of their last stage transition.

When a stage completes or fails, TALM marks the relevant clusters with the following labels:

- `lcm.openshift.io/ibgu-<stage>-completed`
- `lcm.openshift.io/ibgu-<stage>-failed`

Use these cluster labels to cancel or roll back an upgrade on a group of clusters after troubleshooting the issues.

:::warning

If you are using the `ImageBasedGroupUpgrade` CR to upgrade your clusters, ensure that you update the `lcm.openshift.io/ibgu-<stage>-completed` or `lcm.openshift.io/ibgu-<stage>-failed` cluster labels properly after performing troubleshooting or recovery steps on the managed clusters. This ensures that the TALM continues to manage the image-based upgrade for the cluster.

:::

For example, if you want to cancel the upgrade for all managed clusters except for clusters that successfully completed the upgrade, you can add an `Abort` action to your plan. The `Abort` action moves back the `ImageBasedUpgrade` CR to the `Idle` stage, which cancels the upgrade on clusters that are not yet upgraded. Adding a separate `Abort` action ensures that the TALM does not perform the `Abort` action on clusters that have the `lcm.openshift.io/ibgu-upgrade-completed` label.

The TALM removes the cluster labels after successfully canceling or finalizing the upgrade.

### Status monitoring {#ztp-image-based-upgrade-status-monitoring_ztp-gitops}

The `ImageBasedGroupUpgrade` CR ensures a better monitoring experience by aggregating status reporting for all clusters in one place. You can monitor the following actions:

`status.clusters.completedActions`
: Shows all completed actions defined in the `plan` section.

`status.clusters.currentAction`
: Shows all actions that are currently in progress.

`status.clusters.failedActions`
: Shows all failed actions along with a detailed error message.

## Performing an image-based upgrade on managed clusters at scale in several steps {#ztp-image-based-upgrade-procedure-steps_ztp-gitops}

For use cases when you need better control of when the upgrade interrupts your service, you can upgrade a set of your managed clusters by using the `ImageBasedGroupUpgrade` CR. You can use the `ImageBasedGroupUpgrade` CR to add actions after the earlier step is complete. After evaluating the results of the earlier steps, you can move to the next upgrade stage or troubleshoot any failed steps throughout the procedure.

:::warning

Only certain action combinations are supported and listed in *Supported action combinations*.

:::

**Prerequisites**

- You have logged in to the hub cluster as a user with `cluster-admin` privileges.
- You have created policies and `ConfigMap` objects for resources used in the image-based upgrade.
- You have installed the Lifecycle Agent and OADP Operators on all managed clusters through the hub cluster.

**Procedure**

1. Create a YAML file on the hub cluster that has the `ImageBasedGroupUpgrade` CR:
   ```yaml
   apiVersion: lcm.openshift.io/v1alpha1
   kind: ImageBasedGroupUpgrade
   metadata:
     name: <filename>
     namespace: default
   spec:
     clusterLabelSelectors:
       - matchExpressions:
         - key: name
           operator: In
           values:
           - spoke1
           - spoke4
           - spoke6
     ibuSpec:
       seedImageRef:
         image: quay.io/seed/image:4.16.0-rc.1
         version: 4.16.0-rc.1
         pullSecretRef:
           name: "<seed_pull_secret>"
       extraManifests:
         - name: example-extra-manifests
           namespace: openshift-lifecycle-agent
       oadpContent:
         - name: oadp-cm
           namespace: openshift-adp
     plan:
       - actions: ["Prep"]
         rolloutStrategy:
           maxConcurrency: 2
           timeout: 2400
   ```

   Where:

   - `clusterLabelSelectors`: Clusters to upgrade.
   - `seedImageRef`: Target platform version, the seed image, and the secret required to access the image.

   :::note

   If you add the seed image pull secret in the hub cluster, in the same namespace as the `ImageBasedGroupUpgrade` resource, the {lco} adds the secret to the manifest list for the `Prep` stage. The {lco} recreates the secret in each spoke cluster in the `openshift-lifecycle-agent` namespace.

   :::

   - `extraManifests`: Optional: Applies additional manifests, which are not in the seed image, to the target cluster. Also applies `ConfigMap` objects for custom catalog sources.
   - `oadpContent`: List of `ConfigMap` resources that contain the OADP `Backup` and `Restore` CRs.
   - `plan`: Upgrade plan details.
2. Apply the created file by running the following command on the hub cluster:
   ```terminal
   $ oc apply -f <filename>.yaml
   ```
3. Monitor the status updates by running the following command on the hub cluster:
   ```terminal
   $ oc get ibgu -o yaml
   ```

   ```yaml title="Example output"
   # ...
   status:
     clusters:
     - completedActions:
       - action: Prep
       name: spoke1
     - completedActions:
       - action: Prep
       name: spoke4
     - failedActions:
       - action: Prep
       name: spoke6
   # ...
   ```

   The earlier output of an example plan starts with the `Prep` stage only and you add actions to the plan based on the results of the earlier step. The TALM adds a label to the clusters to mark if the upgrade succeeded or failed. For example, the TALM applies the `lcm.openshift.io/ibgu-prep-failed` label to clusters that failed the `Prep` stage.

   After investigating the failure, you can add the `AbortOnFailure` step to your upgrade plan. It moves the clusters labeled with `lcm.openshift.io/ibgu-<action>-failed` back to the `Idle` stage. The TALM deletes the resources that are related to the upgrade on the selected clusters.
4. Optional: Add the `AbortOnFailure` action to your existing `ImageBasedGroupUpgrade` CR by running the following command:
   ```terminal
   $ oc patch ibgu <filename> --type=json -p \
   '[{"op": "add", "path": "/spec/plan/-", "value": {"actions": ["AbortOnFailure"], "rolloutStrategy": {"maxConcurrency": 5, "timeout": 10}}}]'
   ```

   1. Continue monitoring the status updates by running the following command:
      ```terminal
      $ oc get ibgu -o yaml
      ```
5. Add the action to your existing `ImageBasedGroupUpgrade` CR by running the following command:
   ```terminal
   $ oc patch ibgu <filename> --type=json -p \
   '[{"op": "add", "path": "/spec/plan/-", "value": {"actions": ["Upgrade"], "rolloutStrategy": {"maxConcurrency": 2, "timeout": 30}}}]'
   ```
6. Optional: Add the `AbortOnFailure` action to your existing `ImageBasedGroupUpgrade` CR by running the following command:
   ```terminal
   $ oc patch ibgu <filename> --type=json -p \
   '[{"op": "add", "path": "/spec/plan/-", "value": {"actions": ["AbortOnFailure"], "rolloutStrategy": {"maxConcurrency": 5, "timeout": 10}}}]'
   ```

   1. Continue monitoring the status updates by running the following command:
      ```terminal
      $ oc get ibgu -o yaml
      ```
7. Add the action to your existing `ImageBasedGroupUpgrade` CR by running the following command:
   ```terminal
   $ oc patch ibgu <filename> --type=json -p \
   '[{"op": "add", "path": "/spec/plan/-", "value": {"actions": ["FinalizeUpgrade"], "rolloutStrategy": {"maxConcurrency": 10, "timeout": 3}}}]'
   ```

**Verification**

- Monitor the status updates by running the following command:
  ```terminal
  $ oc get ibgu -o yaml
  ```

  ```yaml title="Example output"
  # ...
  status:
    clusters:
    - completedActions:
      - action: Prep
      - action: AbortOnFailure
      failedActions:
      - action: Upgrade
      name: spoke1
    - completedActions:
      - action: Prep
      - action: Upgrade
      - action: FinalizeUpgrade
      name: spoke4
    - completedActions:
      - action: AbortOnFailure
      failedActions:
      - action: Prep
      name: spoke6
  # ...
  ```

**Additional resources**

- [Configuring a shared container partition between ostree stateroots when using GitOps ZTP](/docs/edge_computing/image_based_upgrade/preparing_for_image_based_upgrade/cnf-image-based-upgrade-shared-container-partition#ztp-image-based-upgrade-shared-container-partition_shared-container-partition)
- [Creating ConfigMap objects for the image-based upgrade with Lifecycle Agent using GitOps ZTP](/docs/edge_computing/image_based_upgrade/preparing_for_image_based_upgrade/ztp-image-based-upgrade-prep-resources#ztp-image-based-upgrade-prep-resources)
- [About backup and snapshot locations and their secrets](/docs/backup_and_restore/application_backup_and_restore/installing/installing-oadp-ocs#oadp-about-backup-snapshot-locations_installing-oadp-ocs)
- [Creating a Backup CR](/docs/backup_and_restore/application_backup_and_restore/backing_up_and_restoring/oadp-creating-backup-cr#oadp-creating-backup-cr-doc)
- [Creating a Restore CR](/docs/backup_and_restore/application_backup_and_restore/backing_up_and_restoring/restoring-applications#oadp-creating-restore-cr_restoring-applications)
- [Supported action combinations](/docs/edge_computing/image_based_upgrade/ztp-image-based-upgrade#ztp-image-based-upgrade-supported-combinations_ztp-gitops)

## Performing an image-based upgrade on managed clusters at scale in one step {#ztp-image-based-upgrade-procedure-one-step_ztp-gitops}

For use cases when service interruption is not a concern, you can upgrade a set of your managed clusters by using the `ImageBasedGroupUpgrade` custom resource (CR). You can use the `ImageBasedGroupUpgrade` CR to combine several actions in one step with one rollout strategy. With one rollout strategy, you can reduce the upgrade time but you can only troubleshoot failed clusters after the upgrade plan is complete.

**Prerequisites**

- You have logged in to the hub cluster as a user with `cluster-admin` privileges.
- You have created policies and `ConfigMap` objects for resources used in the image-based upgrade.
- You have installed the Lifecycle Agent and OADP Operators on all managed clusters through the hub cluster.

**Procedure**

1. Create a YAML file on the hub cluster that has the `ImageBasedGroupUpgrade` CR:
   ```yaml
   apiVersion: lcm.openshift.io/v1alpha1
   kind: ImageBasedGroupUpgrade
   metadata:
     name: <filename>
     namespace: default
   spec:
     clusterLabelSelectors:
       - matchExpressions:
         - key: name
           operator: In
           values:
           - spoke1
           - spoke4
           - spoke6
     ibuSpec:
       seedImageRef:
         image: quay.io/seed/image:4.22.0
         version: 4.22.0
         pullSecretRef:
           name: "<seed_pull_secret>"
       extraManifests:
         - name: example-extra-manifests
           namespace: openshift-lifecycle-agent
       oadpContent:
         - name: oadp-cm
           namespace: openshift-adp
     plan:
       - actions: ["Prep", "Upgrade", "FinalizeUpgrade"]
         rolloutStrategy:
           maxConcurrency: 200
           timeout: 2400
   ```

   Where:

   - `clusterLabelSelectors`: Clusters to upgrade.
   - `seedImageRef`: Target platform version, the seed image, and the secret required to access the image.

   :::note

   If you add the seed image pull secret in the hub cluster, in the same namespace as the `ImageBasedGroupUpgrade` resource, the secret is added to the manifest list for the `Prep` stage. The secret is recreated in each spoke cluster in the `openshift-lifecycle-agent` namespace.

   :::

   - `extraManifests`: Optional: Applies additional manifests, which are not in the seed image, to the target cluster. Also applies `ConfigMap` objects for custom catalog sources.
   - `oadpContent`: `ConfigMap` resources that contain the OADP `Backup` and `Restore` CRs.
   - `plan`: Upgrade plan details.
   - `maxConcurrency`: Number of clusters to update in a batch.
   - `timeout`: Timeout limit to complete the action in minutes.
2. Apply the created file by running the following command on the hub cluster:
   ```terminal
   $ oc apply -f <filename>.yaml
   ```

**Verification**

- Monitor the status updates by running the following command:
  ```terminal
  $ oc get ibgu -o yaml
  ```

  ```yaml title="Example output"
  # ...
  status:
    clusters:
    - completedActions:
      - action: Prep
      failedActions:
      - action: Upgrade
      name: spoke1
    - completedActions:
      - action: Prep
      - action: Upgrade
      - action: FinalizeUpgrade
      name: spoke4
    - failedActions:
      - action: Prep
      name: spoke6
  # ...
  ```

## Canceling an image-based upgrade on managed clusters at scale {#ztp-image-based-upgrade-procedure-cancel_ztp-gitops}

You can cancel the upgrade on a set of managed clusters that completed the `Prep` stage.

:::warning

Only certain action combinations are supported and listed in *Supported action combinations*.

:::

**Prerequisites**

- You have logged in to the hub cluster as a user with `cluster-admin` privileges.

**Procedure**

1. Create a separate YAML file on the hub cluster that has the `ImageBasedGroupUpgrade` CR:
   ```yaml
   apiVersion: lcm.openshift.io/v1alpha1
   kind: ImageBasedGroupUpgrade
   metadata:
     name: <filename>
     namespace: default
   spec:
     clusterLabelSelectors:
       - matchExpressions:
         - key: name
           operator: In
           values:
           - spoke4
     ibuSpec:
       seedImageRef:
         image: quay.io/seed/image:4.16.0-rc.1
         version: 4.16.0-rc.1
         pullSecretRef:
           name: "<seed_pull_secret>"
       extraManifests:
         - name: example-extra-manifests
           namespace: openshift-lifecycle-agent
       oadpContent:
         - name: oadp-cm
           namespace: openshift-adp
     plan:
       - actions: ["Abort"]
         rolloutStrategy:
           maxConcurrency: 5
           timeout: 10
   ```

   All managed clusters that completed the `Prep` stage move back to the `Idle` stage.
2. Apply the created file by running the following command on the hub cluster:
   ```terminal
   $ oc apply -f <filename>.yaml
   ```

**Verification**

- Monitor the status updates by running the following command:
  ```terminal
  $ oc get ibgu -o yaml
  ```

  ```yaml title="Example output"
  # ...
  status:
    clusters:
    - completedActions:
      - action: Prep
      currentActions:
      - action: Abort
      name: spoke4
  # ...
  ```

**Additional resources**

- [Supported action combinations](/docs/edge_computing/image_based_upgrade/ztp-image-based-upgrade#ztp-image-based-upgrade-supported-combinations_ztp-gitops)

## Rolling back an image-based upgrade on managed clusters at scale {#ztp-image-based-upgrade-procedure-rollback_ztp-gitops}

Roll back the changes on a set of managed clusters if you find unresolvable issues after a successful upgrade. You need to create a separate `ImageBasedGroupUpgrade` CR and define the set of managed clusters that you want to roll back.

:::warning

Only certain action combinations are supported and listed in *Supported action combinations*.

:::

**Prerequisites**

- You have logged in to the hub cluster as a user with `cluster-admin` privileges.

**Procedure**

1. Create a separate YAML file on the hub cluster that has the `ImageBasedGroupUpgrade` CR:
   ```yaml
   apiVersion: lcm.openshift.io/v1alpha1
   kind: ImageBasedGroupUpgrade
   metadata:
     name: <filename>
     namespace: default
   spec:
     clusterLabelSelectors:
       - matchExpressions:
         - key: name
           operator: In
           values:
           - spoke4
     ibuSpec:
       seedImageRef:
         image: quay.io/seed/image:4.22.0-rc.1
         version: 4.22.0-rc.1
         pullSecretRef:
           name: "<seed_pull_secret>"
       extraManifests:
         - name: example-extra-manifests
           namespace: openshift-lifecycle-agent
       oadpContent:
         - name: oadp-cm
           namespace: openshift-adp
     plan:
       - actions: ["Rollback", "FinalizeRollback"]
         rolloutStrategy:
           maxConcurrency: 200
           timeout: 2400
   ```
2. Apply the created file by running the following command on the hub cluster:
   ```terminal
   $ oc apply -f <filename>.yaml
   ```

   All managed clusters that match the defined labels move back to the `Rollback` and then the `Idle` stages to complete the rollback.

**Verification**

- Monitor the status updates by running the following command:
  ```terminal
  $ oc get ibgu -o yaml
  ```

  ```yaml title="Example output"
  # ...
  status:
    clusters:
    - completedActions:
      - action: Rollback
      - action: FinalizeRollback
      name: spoke4
  # ...
  ```

**Additional resources**

- [Supported action combinations](/docs/edge_computing/image_based_upgrade/ztp-image-based-upgrade#ztp-image-based-upgrade-supported-combinations_ztp-gitops)
- [Recovering from expired control plane certificates](/docs/backup_and_restore/control_plane_backup_and_restore/disaster_recovery/scenario-3-expired-certs#dr-scenario-3-recovering-expired-certs_dr-recovering-expired-certs)

## Troubleshooting image-based upgrades with Lifecycle Agent {#cnf-image-based-upgrade-troubleshooting_ztp-gitops}

Perform troubleshooting steps on the managed clusters to resolve any issues.

:::warning

If you are using the `ImageBasedGroupUpgrade` CR to upgrade your clusters, ensure that you update the `lcm.openshift.io/ibgu-<stage>-completed` or `lcm.openshift.io/ibgu-<stage>-failed` cluster labels properly after performing troubleshooting or recovery steps on the managed clusters. This ensures that the TALM continues to manage the image-based upgrade for the cluster.

:::

### Collecting logs {#cnf-image-based-upgrade-troubleshooting-must-gather_ztp-gitops}

You can use the `oc adm must-gather` CLI to collect information for debugging and troubleshooting.

To collect data about the Operators, run the following command:

```terminal
$  oc adm must-gather \
  --dest-dir=must-gather/tmp \
  --image=$(oc -n openshift-lifecycle-agent get deployment.apps/lifecycle-agent-controller-manager -o jsonpath='{.spec.template.spec.containers[?(@.name == "manager")].image}') \
  --image=quay.io/konveyor/oadp-must-gather:latest \//
  --image=quay.io/openshift/origin-must-gather:latest
```

Where:

- `--image=quay.io/konveyor/oadp-must-gather:latest`: Optional: Add this option if you need to gather more information from the OADP Operator.
- `--image=quay.io/openshift/origin-must-gather:latest`: Optional: Add this option if you need to gather more information from the SR-IOV Operator.

### `AbortFailed` or `FinalizeFailed` error {#cnf-image-based-upgrade-troubleshooting-manual-cleanup_ztp-gitops}

Issue
: During the finalization stage or when you stop the process at the `Prep` stage, Lifecycle Agent cleans up the following resources:

  - Stateroot that is no longer required
  - Precaching resources
  - OADP CRs
  - `ImageBasedUpgrade` CR

  If the Lifecycle Agent fails to clean up these resources, it transitions to the `AbortFailed` or `FinalizeFailed` states. The condition message and log show the steps that failed, as shown in the following example:

  Example error message:

  ```yaml
  message: failed to delete all the backup CRs. Perform cleanup manually then add 'lca.openshift.io/manual-cleanup-done' annotation to ibu CR to transition back to Idle
        observedGeneration: 5
        reason: AbortFailed
        status: "False"
        type: Idle
  ```

Resolution
: 1. Inspect the logs to find the reason for failure.
  2. To prompt Lifecycle Agent to retry the cleanup, add the `lca.openshift.io/manual-cleanup-done` annotation to the `ImageBasedUpgrade` CR.

  After observing this annotation, Lifecycle Agent retries the cleanup and, if it is successful, the `ImageBasedUpgrade` stage transitions to `Idle`.

  If the cleanup fails again, you can manually clean up the resources.

### Cleaning up stateroot manually {#cnf-image-based-upgrade-troubleshooting-stateroot_ztp-gitops}

Issue
: Stopping at the `Prep` stage, Lifecycle Agent cleans up the new stateroot. When finalizing after a successful upgrade or a rollback, Lifecycle Agent cleans up the old stateroot. If this step fails, you must inspect the logs to decide why the failure occurred.

Resolution
: 1. Check if there are any existing deployments in the stateroot by running the following command:

  ```terminal
  $ ostree admin status
  ```

  1. If there are any, clean up the existing deployment by running the following command:

  ```terminal
  $ ostree admin undeploy <index_of_deployment>
  ```

  1. After cleaning up all the deployments of the stateroot, wipe the stateroot directory by running the following commands:

  :::warning

  Ensure that the booted deployment is not in this stateroot.

  :::

  ```terminal
  $ stateroot="<stateroot_to_delete>"
  ```

  ```terminal
  $ unshare -m /bin/sh -c "mount -o remount,rw /sysroot && rm -rf /sysroot/ostree/deploy/${stateroot}"
  ```

### Cleaning up OADP resources manually {#cnf-image-based-upgrade-troubleshooting-oadp-resources_ztp-gitops}

Issue
: Automatic cleanup of OADP resources can fail due to connection issues between Lifecycle Agent and the S3 backend. By restoring the connection and adding the `lca.openshift.io/manual-cleanup-done` annotation, the Lifecycle Agent can successfully cleanup backup resources.

Resolution
: 1. Check the backend connectivity by running the following command:

  ```terminal
  $ oc get backupstoragelocations.velero.io -n openshift-adp
  ```

  The following example shows successful backend connectivity:

  ```terminal
  NAME                          PHASE       LAST VALIDATED   AGE   DEFAULT
  dataprotectionapplication-1   Available   33s              8d    true
  ```

  1. Remove all backup resources and then add the `lca.openshift.io/manual-cleanup-done` annotation to the `ImageBasedUpgrade` CR.

### LVM Storage volume contents not restored {#cnf-image-based-upgrade-troubleshooting-lvms_ztp-gitops}

When you use LVM Storage to configure dynamic persistent volume storage, LVM Storage might not restore the persistent volume contents if you have configured it incorrectly.

### Missing LVM Storage-related fields in Backup CR {#cnf-image-based-upgrade-troubleshooting-lvms-backup_ztp-gitops}

Issue
: Your `Backup` CRs might be missing fields that you need to restore your persistent volumes. You can check for events in your application pod to decide if you have this issue by running the following:

  ```terminal
  $ oc describe pod <your_app_name>
  ```

  The following example output shows a pod failing due to missing LVM Storage-related fields in the `Backup` CR:

  ```terminal
  Events:
    Type     Reason            Age                From               Message
    ----     ------            ----               ----               -------
    Warning  FailedScheduling  58s (x2 over 66s)  default-scheduler  0/1 nodes are available: pod has unbound immediate PersistentVolumeClaims. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling..
    Normal   Scheduled         56s                default-scheduler  Successfully assigned default/db-1234 to sno1.example.lab
    Warning  FailedMount       24s (x7 over 55s)  kubelet            MountVolume.SetUp failed for volume "pvc-1234" : rpc error: code = Unknown desc = VolumeID is not found
  ```

Resolution
: You must include `logicalvolumes.topolvm.io` in the application `Backup` CR. Without this resource, the application restores its persistent volume claims and persistent volume manifests correctly, however, the `logicalvolume` associated with this persistent volume is not restored properly after pivot. The following example shows a correctly configured `Backup` CR:

  ```yaml
  apiVersion: velero.io/v1
  kind: Backup
  metadata:
    labels:
      velero.io/storage-location: default
    name: small-app
    namespace: openshift-adp
  spec:
    includedNamespaces:
    - test
    includedNamespaceScopedResources:
    - secrets
    - persistentvolumeclaims
    - deployments
    - statefulsets
    includedClusterScopedResources:
    - persistentVolumes
    - volumesnapshotcontents
    - logicalvolumes.topolvm.io
  ```

  To restore the persistent volumes for your application, you must configure the `includedClusterScopedResources` section as shown.

### Missing LVM Storage-related fields in Restore CR {#cnf-image-based-upgrade-troubleshooting-lvms-restore_ztp-gitops}

Issue
: LVM Storage restores the expected resources for the applications but it does not preserve the persistent volume contents after upgrading.

1. List the persistent volumes for you applications by running the following command before pivot:
   ```terminal
   $ oc get pv,pvc,logicalvolumes.topolvm.io -A
   ```

   The following shows the output before pivot:

   ```terminal
   NAME                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM            STORAGECLASS   REASON   AGE
   persistentvolume/pvc-1234   1Gi        RWO            Retain           Bound    default/pvc-db   lvms-vg1                4h45m

   NAMESPACE   NAME                           STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
   default     persistentvolumeclaim/pvc-db   Bound    pvc-1234   1Gi        RWO            lvms-vg1       4h45m

   NAMESPACE   NAME                                AGE
               logicalvolume.topolvm.io/pvc-1234   4h45m
   ```
2. List the persistent volumes for you applications by running the following command after pivot:
   ```terminal
   $ oc get pv,pvc,logicalvolumes.topolvm.io -A
   ```

   The following shows the output after pivot:

   ```terminal
   NAME                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM            STORAGECLASS   REASON   AGE
   persistentvolume/pvc-1234   1Gi        RWO            Delete           Bound    default/pvc-db   lvms-vg1                19s

   NAMESPACE   NAME                           STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
   default     persistentvolumeclaim/pvc-db   Bound    pvc-1234   1Gi        RWO            lvms-vg1       19s

   NAMESPACE   NAME                                AGE
               logicalvolume.topolvm.io/pvc-1234   18s
   ```

   <dl>
   <dt>Resolution</dt>
   <dd>The reason for this issue is that the <code>logicalvolume</code> status is not preserved in the <code>Restore</code> CR. This status is important because Velero requires this status to reference the volumes that you must preserve after pivoting. You must include the following fields in the application <code>Restore</code> CR, as shown in the following example:</dd>
   </dl>

   ```yaml
   apiVersion: velero.io/v1
   kind: Restore
   metadata:
     name: sample-vote-app
     namespace: openshift-adp
     labels:
       velero.io/storage-location: default
     annotations:
       lca.openshift.io/apply-wave: "3"
   spec:
     backupName:
       sample-vote-app
     restorePVs: <restore_pvs>
     restoreStatus:
       includedResources:
         - logicalvolumes
   ```

   where: `<restore_pvs>`:: To preserve the persistent volumes for your application, you must set `restorePVs` to `true`. `restoreStatus`:: To preserve the persistent volumes for your application, you must configure this field as shown.

### Debugging failed Backup and Restore CRs {#cnf-image-based-upgrade-troubleshooting-debugging-oadp-crs_ztp-gitops}

Issue
: The backup or restoration of artifacts failed.

Resolution
: You can debug `Backup` and `Restore` CRs and retrieve logs with the OADP CLI. The OADP CLI offers more detailed information than the OpenShift CLI tool.

1. Describe the `Backup` CR that has errors by running the following command:
   ```terminal
   $ oc oadp backup describe backup-acm-klusterlet -n openshift-adp --details
   ```
2. Describe the `Restore` CR that has errors by running the following command:
   ```terminal
   $ oc oadp restore describe restore-acm-klusterlet -n openshift-adp --details
   ```
3. Download the backed up resources to a local directory by running the following command:
   ```terminal
   $ oc oadp backup download backup-acm-klusterlet -n openshift-adp -o ~/backup-acm-klusterlet.tar.gz
   ```
