---
title: Evicting pods using the descheduler
sidebar_position: 3
---

# Evicting pods using the descheduler {#nodes-descheduler-configuring}

<a id="nodes-descheduler-configuring"></a>

You can run the descheduler in OpenShift Container Platform by installing the Kube Descheduler Operator and setting the required profiles and other customizations.

## Install the descheduler {#nodes-descheduler-installing_nodes-descheduler-configuring}

The descheduler is not available by default. To enable the descheduler, you must install the Kube Descheduler Operator from the software catalog and enable one or more descheduler profiles.

By default, the descheduler runs in predictive mode, which means that it only simulates pod evictions. You must change the mode to automatic for the descheduler to perform the pod evictions.

:::warning

If you have enabled hosted control planes in your cluster, set a custom priority threshold to lower the chance that pods in the hosted control plane namespaces are evicted. Set the priority threshold class name to `hypershift-control-plane`, because it has the lowest priority value (`100000000`) of the hosted control plane priority classes.

:::

**Prerequisites**

- You are logged in to OpenShift Container Platform as a user with the `cluster-admin` role.
- Access to the OpenShift Container Platform web console.

**Procedure**

1. Log in to the OpenShift Container Platform web console.
2. Create the required namespace for the Kube Descheduler Operator.
   1. Navigate to **Administration** → **Namespaces** and click **Create Namespace**.
   2. Enter `openshift-kube-descheduler-operator` in the **Name** field, enter `openshift.io/cluster-monitoring=true` in the **Labels** field to enable descheduler metrics, and click **Create**.
3. Install the Kube Descheduler Operator.
   1. Navigate to **Ecosystem** → **Software Catalog**.
   2. Type **Kube Descheduler Operator** into the filter box.
   3. Select the **Kube Descheduler Operator** and click **Install**.
   4. On the **Install Operator** page, select **A specific namespace on the cluster**. Select **openshift-kube-descheduler-operator** from the drop-down menu.
   5. Adjust the values for the **Update Channel** and **Approval Strategy** to the desired values.
   6. Click **Install**.
4. Create a descheduler instance.
   1. From the **Ecosystem** → **Installed Operators** page, click the **Kube Descheduler Operator**.
   2. Select the **Kube Descheduler** tab and click **Create KubeDescheduler**.
   3. Edit the settings as necessary.
      1. To evict pods instead of simulating the evictions, change the **Mode** field to **Automatic**.
      2. Expand the **Profiles** section to select one or more profiles to enable. The `AffinityAndTaints` profile is enabled by default. Click **Add Profile** to select additional profiles.
         :::note

         Do not enable both `TopologyAndDuplicates` and `SoftTopologyAndDuplicates`. Enabling both results in a conflict.

         :::
      3. Optional: Expand the **Profile Customizations** section to set optional configurations for the descheduler.
         - Set a custom pod lifetime value for the `LifecycleAndUtilization` profile. Use the **podLifetime** field to set a numerical value and a valid unit (`s`, `m`, or `h`). The default pod lifetime is 24 hours (`24h`).
         - Set a custom priority threshold to consider pods for eviction only if their priority is lower than a specified priority level. Use the **thresholdPriority** field to set a numerical priority threshold or use the **thresholdPriorityClassName** field to specify a certain priority class name.
           :::note

           Do not specify both **thresholdPriority** and **thresholdPriorityClassName** for the descheduler.

           :::
         - Set specific namespaces to exclude or include from descheduler operations. Expand the **namespaces** field and add namespaces to the **excluded** or **included** list. You can only either set a list of namespaces to exclude or a list of namespaces to include. Note that protected namespaces (`openshift-*`, `kube-system`, `hypershift`) are excluded by default.
         - Experimental: Set thresholds for underutilization and overutilization for the `LowNodeUtilization` strategy. Use the **devLowNodeUtilizationThresholds** field to set one of the following values:
           - `Low`: 10% underutilized and 30% overutilized
           - `Medium`: 20% underutilized and 50% overutilized (Default)
           - `High`: 40% underutilized and 70% overutilized

           :::note

           This setting is experimental and should not be used in a production environment.

           :::
      4. Optional: Use the **Descheduling Interval Seconds** field to change the number of seconds between descheduler runs. The default is `3600` seconds.
   4. Click **Create**.
      You can also configure the profiles and settings for the descheduler later using the OpenShift CLI (`oc`). If you did not adjust the profiles when creating the descheduler instance from the web console, the `AffinityAndTaints` profile is enabled by default.

## Configure descheduler profiles {#nodes-descheduler-configuring-profiles_nodes-descheduler-configuring}

To manage cluster pod eviction behavior, select which descheduler profiles to enable.

**Prerequisites**

- You are logged in to OpenShift Container Platform as a user with the `cluster-admin` role.

**Procedure**

1. Edit the `KubeDescheduler` object:
   ```terminal
   $ oc edit kubedeschedulers.operator.openshift.io cluster -n openshift-kube-descheduler-operator
   ```
2. Specify one or more profiles in the `spec.profiles` section.
   ```yaml
   apiVersion: operator.openshift.io/v1
   kind: KubeDescheduler
   metadata:
     name: cluster
     namespace: openshift-kube-descheduler-operator
   spec:
     deschedulingIntervalSeconds: 3600
     logLevel: Normal
     managementState: Managed
     operatorLogLevel: Normal
     mode: Predictive
     profileCustomizations:
       namespaces:
         excluded:
         - my-namespace
       podLifetime: 48h
       thresholdPriorityClassName: my-priority-class-name
     evictionLimits:
       total: 20
     profiles:
     - AffinityAndTaints
     - TopologyAndDuplicates
     - LifecycleAndUtilization
     - EvictPodsWithLocalStorage
     - EvictPodsWithPVC
   ```

   where:

   <dl>
   <dt><code>spec.mode</code></dt>
   <dd>Specifies the eviction mode. By default, the descheduler does not evict pods. To evict pods, set <code>mode</code> to <code>Automatic</code>.</dd>
   <dt><code>spec.profileCustomizations.namespaces</code></dt>
   <dd>Specifies a list of user-created namespaces to include or exclude from descheduler operations. Use <code>excluded</code> to set a list of namespaces to exclude or use <code>included</code> to set a list of namespaces to include. Note that protected namespaces (<code>openshift-*</code>, <code>kube-system</code>, <code>hypershift</code>) are excluded by default. This value is optional.</dd>
   <dt><code>spec.profileCustomizations.podLifetime</code></dt>
   <dd>Specifies a custom pod lifetime value for the <code>LifecycleAndUtilization</code> profile. Valid units are <code>s</code>, <code>m</code>, or <code>h</code>. The default pod lifetime is 24 hours. This value is optional.</dd>
   <dt><code>spec.profileCustomizations.thresholdPriorityClassName</code></dt>
   <dd>Specifies a priority threshold to consider pods for eviction only if their priority is lower than the specified level. Use the <code>thresholdPriority</code> field to set a numerical priority threshold (for example, <code>10000</code>) or use the <code>thresholdPriorityClassName</code> field to specify a certain priority class name (for example, <code>my-priority-class-name</code>). If you specify a priority class name, it must already exist or the descheduler will throw an error. Do not set both <code>thresholdPriority</code> and <code>thresholdPriorityClassName</code>. This value is optional.</dd>
   <dt><code>spec.evictionLimits.total</code></dt>
   <dd>Specifies the maximum number of pods to evict during each descheduler run. This value is optional.</dd>
   <dt><code>spec.profiles</code></dt>
   <dd>Specifies one or more profiles to enable. Available profiles: <code>AffinityAndTaints</code>, <code>TopologyAndDuplicates</code>, <code>LifecycleAndUtilization</code>, <code>SoftTopologyAndDuplicates</code>, <code>EvictPodsWithLocalStorage</code>, <code>EvictPodsWithPVC</code>, <code>CompactAndScale</code>, and <code>LongLifecycle</code>. You can enable multiple profiles, but ensure that you do not enable profiles that conflict with each other. The order of the list of profiles is not important.</dd>
   </dl>
3. Save the file to apply the changes.

## Configure the descheduler interval {#nodes-descheduler-configuring-interval_nodes-descheduler-configuring}

You can configure the amount of time between descheduler runs. The default is 3600 seconds (one hour).

**Prerequisites**

- You are logged in to OpenShift Container Platform as a user with the `cluster-admin` role.

**Procedure**

1. Edit the `KubeDescheduler` object:
   ```terminal
   $ oc edit kubedeschedulers.operator.openshift.io cluster -n openshift-kube-descheduler-operator
   ```
2. Update the `deschedulingIntervalSeconds` field to the required value:
   ```yaml
   apiVersion: operator.openshift.io/v1
   kind: KubeDescheduler
   metadata:
     name: cluster
     namespace: openshift-kube-descheduler-operator
   spec:
     deschedulingIntervalSeconds: 3600
   ...
   ```

   Set the `spec.deschedulingIntervalSeconds` field to the number of seconds you want between descheduler runs. A value of `0` in this field runs the descheduler once and exits.
3. Save the file to apply the changes.
