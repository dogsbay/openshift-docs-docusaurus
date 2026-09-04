---
title: Uninstalling the Leader Worker Set Operator
sidebar_position: 4
---

# Uninstalling the Leader Worker Set Operator {#lws-uninstalling}

<a id="lws-uninstalling"></a>

If you no longer need the Leader Worker Set Operator in your cluster, you can uninstall the Operator and remove its related resources.

## Uninstalling the Leader Worker Set Operator {#lws-uninstall_lws-uninstalling}

You can use the web console to uninstall the Leader Worker Set Operator if you no longer need the Operator in your cluster.

**Prerequisites**

- You have access to the cluster with `cluster-admin` privileges.
- You have access to the OpenShift Container Platform web console.
- You have installed the Leader Worker Set Operator.

**Procedure**

1. Log in to the OpenShift Container Platform web console.
2. Navigate to **Operators** → **Installed Operators**.
3. Select `openshift-lws-operator` from the **Project** dropdown list.
4. Delete the `LeaderWorkerSetOperator` instance.
   1. Click **Leader Worker Set Operator** and select the **LeaderWorkerSetOperator** tab.
   2. Click the Options menu ![](/images/kebab.png "Options menu") next to the **cluster** entry and select **Delete LeaderWorkerSetOperator**.
   3. In the confirmation dialog, click **Delete**.
5. Uninstall the Leader Worker Set Operator.
   1. Navigate to **Operators** → **Installed Operators**.
   2. Click the Options menu ![](/images/kebab.png "Options menu") next to the **Leader Worker Set Operator** entry and click **Uninstall Operator**.
   3. In the confirmation dialog, click **Uninstall**.

## Uninstalling Leader Worker Set Operator resources {#lws-remove-resources_lws-uninstalling}

Optionally, remove custom resources (CRs) and the associated namespace after the Leader Worker Set Operator is uninstalled. This cleans up all remaining Leader Worker Set artifacts.

**Prerequisites**

- You have access to the cluster with `cluster-admin` privileges.
- You have access to the OpenShift Container Platform web console.
- You have uninstalled the Leader Worker Set Operator.

**Procedure**

1. Log in to the OpenShift Container Platform web console.
2. Remove CRDs that were created when the Leader Worker Set Operator was installed:
   1. Navigate to **Administration** → **CustomResourceDefinitions**.
   2. Enter `LeaderWorkerSetOperator` in the **Name** field to filter the CRDs.
   3. Click the Options menu ![](/images/kebab.png "Options menu") next to the **LeaderWorkerSetOperator** CRD and select **Delete CustomResourceDefinition**.
   4. In the confirmation dialog, click **Delete**.
3. Delete the `openshift-lws-operator` namespace.
   1. Navigate to **Administration** → **Namespaces**.
   2. Enter `openshift-lws-operator` into the filter box.
   3. Click the Options menu ![](/images/kebab.png "Options menu") next to the **openshift-lws-operator** entry and select **Delete Namespace**.
   4. In the confirmation dialog, enter `openshift-lws-operator` and click **Delete**.
