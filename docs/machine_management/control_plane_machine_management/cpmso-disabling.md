---
title: Disabling the control plane machine set
sidebar_position: 8
---

# Disabling the control plane machine set {#cpmso-disabling}

<a id="cpmso-disabling"></a>

Disable the control plane machine set if you need to manually manage control plane machines or troubleshoot Operator behavior.

The `.spec.state` field in an activated `ControlPlaneMachineSet` custom resource (CR) cannot be changed from `Active` to `Inactive`. To disable the control plane machine set, you must delete the CR so that it is removed from the cluster.

When you delete the CR, the Control Plane Machine Set Operator performs cleanup operations and disables the control plane machine set. The Operator then removes the CR from the cluster and creates an inactive control plane machine set with default settings.

## Deleting the control plane machine set {#cpmso-deleting_cpmso-disabling}

To stop managing control plane machines with the control plane machine set on your cluster, you must delete the `ControlPlaneMachineSet` custom resource (CR).

**Procedure**

- Delete the control plane machine set CR by running the following command:
  ```terminal
  $ oc delete controlplanemachineset.machine.openshift.io cluster \
    -n openshift-machine-api
  ```

**Verification**

- Check the control plane machine set custom resource state. A result of `Inactive` indicates that the removal and replacement process is successful. A `ControlPlaneMachineSet` CR exists but is not activated.

## Checking the control plane machine set custom resource state {#cpmso-checking-status_cpmso-disabling}

Check the state of the control plane machine set custom resource to determine if it is active, inactive, or missing before making configuration changes.

**Procedure**

- Determine the state of the CR by running the following command:
  ```terminal
  $ oc get controlplanemachineset.machine.openshift.io cluster \
    --namespace openshift-machine-api
  ```

  - A result of `Active` indicates that the `ControlPlaneMachineSet` CR exists and is activated. No administrator action is required.
  - A result of `Inactive` indicates that a `ControlPlaneMachineSet` CR exists but is not activated.
  - A result of `NotFound` indicates that there is no existing `ControlPlaneMachineSet` CR.

## Re-enabling the control plane machine set {#cpmso-reenabling_cpmso-disabling}

Restore automated control plane management after previously disabling the control plane machine set.

To re-enable the control plane machine set, you must ensure that the configuration in the CR is correct for your cluster and activate it.

For more information, see "Activating the control plane machine set custom resource".

**Additional resources**

- [Activating the control plane machine set custom resource](/docs/machine_management/control_plane_machine_management/cpmso-getting-started#cpmso-activating_cpmso-getting-started)
