---
title: Scheduling pods using a secondary scheduler
sidebar_position: 3
---

# Scheduling pods using a secondary scheduler {#secondary-scheduler-configuring}

<a id="secondary-scheduler-configuring"></a>

You can run a custom secondary scheduler in OpenShift Container Platform by installing the Secondary Scheduler Operator, deploying the secondary scheduler, and setting the secondary scheduler in the pod definition.

## Installing the Secondary Scheduler Operator {#nodes-secondary-scheduler-install-console_secondary-scheduler-configuring}

You can install the Secondary Scheduler Operator for Red Hat OpenShift through the OpenShift Container Platform web console to configure a secondary scheduler.

**Prerequisites**

- You are logged in to OpenShift Container Platform as a user with the `cluster-admin` role.
- You have access to the OpenShift Container Platform web console.

**Procedure**

1. Log in to the OpenShift Container Platform web console.
2. Create the required namespace for the Secondary Scheduler Operator for Red Hat OpenShift.
   1. Navigate to **Administration** → **Namespaces** and click **Create Namespace**.
   2. Enter `openshift-secondary-scheduler-operator` in the **Name** field and click **Create**.
3. Install the Secondary Scheduler Operator for Red Hat OpenShift.
   1. Navigate to **Ecosystem** → **Software Catalog**.
   2. Enter **Secondary Scheduler Operator for Red Hat OpenShift** into the filter box.
   3. Select the **Secondary Scheduler Operator for Red Hat OpenShift** and click **Install**.
   4. On the **Install Operator** page:
      1. The **Update channel** is set to **stable**, which installs the latest stable release of the Secondary Scheduler Operator for Red Hat OpenShift.
      2. Select **A specific namespace on the cluster** and select **openshift-secondary-scheduler-operator** from the drop-down menu.
      3. Select an **Update approval** strategy.
         - The **Automatic** strategy allows Operator Lifecycle Manager (OLM) to automatically update the Operator when a new version is available.
         - The **Manual** strategy requires a user with appropriate credentials to approve the Operator update.
      4. Click **Install**.

**Verification**

1. Navigate to **Ecosystem** → **Installed Operators**.
2. Verify that **Secondary Scheduler Operator for Red Hat OpenShift** is listed with a **Status** of **Succeeded**.

## Deploying a secondary scheduler {#nodes-secondary-scheduler-configuring-console_secondary-scheduler-configuring}

After you have installed the Secondary Scheduler Operator, you can deploy a secondary scheduler to apply custom placement logic for specific pods.

**Prerequisites**

- You are logged in to OpenShift Container Platform as a user with the `cluster-admin` role.
- You have access to the OpenShift Container Platform web console.
- The Secondary Scheduler Operator for Red Hat OpenShift is installed.

**Procedure**

1. Log in to the OpenShift Container Platform web console.
2. Create config map to hold the configuration for the secondary scheduler.
   1. Navigate to **Workloads** → **ConfigMaps**.
   2. Click **Create ConfigMap**.
   3. In the YAML editor, enter the config map definition that contains the necessary `KubeSchedulerConfiguration` configuration. For example:
      ```yaml
      apiVersion: v1
      kind: ConfigMap
      metadata:
        name: "secondary-scheduler-config"
        namespace: "openshift-secondary-scheduler-operator"
      data:
        "config.yaml": |
          apiVersion: kubescheduler.config.k8s.io/v1
          kind: KubeSchedulerConfiguration
          leaderElection:
            leaderElect: false
          profiles:
            - schedulerName: secondary-scheduler
              plugins:
                score:
                  disabled:
                    - name: NodeResourcesBalancedAllocation
                    - name: NodeResourcesLeastAllocated
      ```

      where:

      <dl>
      <dt><code>metadata.name</code></dt>
      <dd>Specifies the name of the config map. This is used in the <strong>Scheduler Config</strong> field when creating the <code>SecondaryScheduler</code> CR.</dd>
      <dt><code>metadata.namespace</code></dt>
      <dd>Specifies the namespace to create the config map in. The namespace must be <code>openshift-secondary-scheduler-operator</code>.</dd>
      <dt><code>data."config.yaml".kind</code></dt>
      <dd>Specifies the <code>KubeSchedulerConfiguration</code> resource for the secondary scheduler. For more information, see <a href="https://kubernetes.io/docs/reference/config-api/kube-scheduler-config.v1/#kubescheduler-config-k8s-io-v1-KubeSchedulerConfiguration"><code>KubeSchedulerConfiguration</code></a> in the Kubernetes API documentation.</dd>
      <dt><code>data."config.yaml".profiles.schedulerName</code></dt>
      <dd>Specifies the name of the secondary scheduler. Pods that set their <code>spec.schedulerName</code> field to this value are scheduled with this secondary scheduler.</dd>
      <dt><code>data."config.yaml".profiles.plugins</code></dt>
      <dd>Specifies the plugins to enable or disable for the secondary scheduler. For a list default scheduling plugins, see <a href="https://kubernetes.io/docs/reference/scheduling/config/#scheduling-plugins">Scheduling plugins</a> in the Kubernetes documentation.</dd>
      </dl>
   4. Click **Create**.
3. Create the `SecondaryScheduler` CR:
   1. Navigate to **Ecosystem** → **Installed Operators**.
   2. Select **Secondary Scheduler Operator for Red Hat OpenShift**.
   3. Select the **Secondary Scheduler** tab and click **Create SecondaryScheduler**.
   4. The **Name** field defaults to `cluster`; do not change this name.
   5. The **schedulerConfig** field defaults to `secondary-scheduler-config`. Ensure that this value matches the name of the config map created earlier in this procedure.
   6. In the **schedulerImage** field, enter the image name for your custom scheduler.
      :::warning

      Red Hat does not directly support the functionality of your custom secondary scheduler.

      :::
   7. Optional: To enable high availability for the secondary scheduler, configure the following settings:
      1. Expand the **topology** section.
      2. In the **mode** field, select **HighlyAvailable**.
      3. In the **maxReplicas** field, enter the maximum number of secondary scheduler replicas to deploy. If unset, the maximum number of replicas is `3`.
      4. In the **tolerations** field, enter tolerations to allow scheduler replicas on tainted nodes. If unset, no taints are tolerated.

      :::note

      To configure node selectors, use the **YAML view** option. Add the `spec.topology.highlyAvailableTopology.nodeSelector` field and enter the necessary node labels to target a specific group of nodes for scheduler replica placement. If unset, all nodes are considered.

      :::
   8. Click **Create**.

## Scheduling a pod using the secondary scheduler {#nodes-secondary-scheduler-pod-console_secondary-scheduler-configuring}

To schedule a pod by using the secondary scheduler, set the `schedulerName` field in the pod definition.

**Prerequisites**

- You are logged in to OpenShift Container Platform as a user with the `cluster-admin` role.
- You have access to the OpenShift Container Platform web console.
- The Secondary Scheduler Operator for Red Hat OpenShift is installed.
- A secondary scheduler is configured.

**Procedure**

1. Log in to the OpenShift Container Platform web console.
2. Navigate to **Workloads** → **Pods**.
3. Click **Create Pod**.
4. In the YAML editor, enter the desired pod configuration and add the `schedulerName` field:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: nginx
     namespace: default
   spec:
     securityContext:
       runAsNonRoot: true
       seccompProfile:
         type: RuntimeDefault
     containers:
       - name: nginx
         image: nginx:1.14.2
         ports:
           - containerPort: 80
         securityContext:
           allowPrivilegeEscalation: false
           capabilities:
             drop: [ALL]
     schedulerName: secondary-scheduler
   ```

   The `spec.schedulerName` field must match the name that is defined in the config map when you configured the secondary scheduler.
5. Click **Create**.

**Verification**

1. Log in to the OpenShift CLI.
2. Describe the pod using the following command:
   ```terminal
   $ oc describe pod nginx -n default
   ```

   ```text title="Example output"
   Name:         nginx
   Namespace:    default
   Priority:     0
   Node:         ci-ln-t0w4r1k-72292-xkqs4-worker-b-xqkxp/10.0.128.3
   ...
   Events:
     Type    Reason          Age   From                 Message
     ----    ------          ----  ----                 -------
     Normal  Scheduled       12s   secondary-scheduler  Successfully assigned default/nginx to ci-ln-t0w4r1k-72292-xkqs4-worker-b-xqkxp
   ...
   ```
3. In the events table, find the event with a message similar to `Successfully assigned <namespace>/<pod_name> to <node_name>`.
4. In the "From" column, verify that the event was generated from the secondary scheduler and not the default scheduler.
   :::note

   You can also check the `secondary-scheduler-*` pod logs in the `openshift-secondary-scheduler-namespace` to verify that the pod was scheduled by the secondary scheduler.

   :::
