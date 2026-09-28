---
title: Uninstalling the Secondary Scheduler Operator
sidebar_position: 4
---

# Uninstalling the Secondary Scheduler Operator {#secondary-scheduler-uninstalling}

<a id="secondary-scheduler-uninstalling"></a>

If you no longer need the Secondary Scheduler Operator for Red Hat OpenShift in your cluster, you can uninstall the Operator and remove its related resources.

## Uninstall the Secondary Scheduler Operator {#nodes-secondary-scheduler-uninstall-console_secondary-scheduler-uninstalling}

You can use the web console to uninstall the Secondary Scheduler Operator for Red Hat OpenShift if you no longer need the Operator in your cluster.

**Prerequisites**

- You are logged in to OpenShift Container Platform as a user with the `cluster-admin` role.
- You have access to the OpenShift Container Platform web console.
- The Secondary Scheduler Operator for Red Hat OpenShift is installed.

**Procedure**

1. Log in to the OpenShift Container Platform web console.
2. Uninstall the Secondary Scheduler Operator for Red Hat OpenShift Operator.
   1. Navigate to **Ecosystem** → **Installed Operators**.
   2. Click the Options menu ![](/images/kebab.png "Options menu") next to the **Secondary Scheduler Operator** entry and click **Uninstall Operator**.
   3. In the confirmation dialog, click **Uninstall**.

## Remove Secondary Scheduler Operator resources {#nodes-secondary-scheduler-remove-resources-console_secondary-scheduler-uninstalling}

Optionally, remove the custom resource definition (CRD) and associated namespace after the Secondary Scheduler Operator for Red Hat OpenShift is uninstalled. This cleans up all remaining secondary scheduler artifacts.

**Prerequisites**

- You are logged in to OpenShift Container Platform as a user with the `cluster-admin` role.
- You have access to the OpenShift Container Platform web console.

**Procedure**

1. Log in to the OpenShift Container Platform web console.
2. Remove the CRD that was installed by the Secondary Scheduler Operator:
   1. Navigate to **Administration** → **CustomResourceDefinitions**.
   2. Enter `SecondaryScheduler` in the **Name** field to filter the CRDs.
   3. Click the Options menu ![](/images/kebab.png "Options menu") next to the **SecondaryScheduler** CRD and select **Delete Custom Resource Definition**:
3. Remove the `openshift-secondary-scheduler-operator` namespace.
   1. Navigate to **Administration** → **Namespaces**.
   2. Click the Options menu ![](/images/kebab.png "Options menu") next to the **openshift-secondary-scheduler-operator** and select **Delete Namespace**.
   3. In the confirmation dialog, enter `openshift-secondary-scheduler-operator` in the field and click **Delete**.
