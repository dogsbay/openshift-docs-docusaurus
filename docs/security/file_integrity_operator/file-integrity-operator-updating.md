---
title: Updating the File Integrity Operator
sidebar_position: 5
---

# Updating the File Integrity Operator {#file-integrity-operator-updating}

<a id="file-integrity-operator-updating"></a>

As a cluster administrator, you can update the File Integrity Operator on your OpenShift Container Platform cluster.

## About preparing for an Operator update {#olm-preparing-upgrade_file-integrity-operator-updating}

You can change the update channel to start tracking and receiving updates from a newer channel to access new features and bug fixes. The subscription of an installed Operator specifies an update channel that tracks and receives updates for the Operator.

The names of update channels in a subscription can differ between Operators, but the naming scheme typically follows a common convention within a given Operator. For example, channel names might follow a minor release update stream for the application provided by the Operator (`1.2`, `1.3`) or a release frequency (`stable`, `fast`).

:::note

You cannot change installed Operators to a channel that is older than the current channel.

:::

Red Hat Customer Portal Labs include an application that helps administrators prepare to update their Operators.

You can use these tools to search for Operators and verify the available Operator versions per update channel across different releases of OpenShift Container Platform. Operators managed by Cluster Version Operator (CVO) are not included.

## Changing the update channel for an Operator {#olm-changing-update-channel_file-integrity-operator-updating}

To change the update channel for an installed Operator, you can use the OpenShift Container Platform web console. The update channel determines which Operator versions your subscription tracks and receives.

:::tip

If the approval strategy in the subscription is set to **Automatic**, the update process initiates as soon as a new Operator version is available in the selected channel. If the approval strategy is set to **Manual**, you must manually approve pending updates.

:::

**Prerequisites**

- An Operator previously installed using Operator Lifecycle Manager (OLM).

**Procedure**

1. In the web console, navigate to **Ecosystem** → **Installed Operators**.
2. Click the name of the Operator you want to change the update channel for.
3. Click the **Subscription** tab.
4. Click the name of the update channel under **Update channel**.
5. Click the newer update channel that you want to change to, then click **Save**.
6. For subscriptions with an **Automatic** approval strategy, the update begins automatically. Navigate back to the **Ecosystem** → **Installed Operators** page to monitor the progress of the update. When complete, the status changes to **Succeeded** and **Up to date**.
   For subscriptions with a **Manual** approval strategy, you can manually approve the update from the **Subscription** tab.

## Approving a pending Operator update manually {#olm-approving-pending-upgrade_file-integrity-operator-updating}

If an installed Operator has the approval strategy in its subscription set to **Manual**, you must manually approve the update before installation can begin. Manual approval reviews the changes and control when updates are applied to prevent unexpected downtime.

**Prerequisites**

- An Operator previously installed using Operator Lifecycle Manager (OLM).

**Procedure**

1. In the OpenShift Container Platform web console, navigate to **Ecosystem** → **Installed Operators**.
2. Operators that have a pending update display a status with **Upgrade available**. Click the name of the Operator you want to update.
3. Click the **Subscription** tab. Any updates requiring approval are displayed next to **Upgrade status**. For example, it might display **1 requires approval**.
4. Click **1 requires approval**, then click **Preview Install Plan**.
5. Review the resources that are listed as available for update. When satisfied, click **Approve**.
6. Navigate back to the **Ecosystem** → **Installed Operators** page to monitor the progress of the update. When complete, the status changes to **Succeeded** and **Up to date**.
