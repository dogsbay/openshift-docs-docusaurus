---
title: Installing the custom metrics autoscaler
sidebar_position: 3
---

# Installing the custom metrics autoscaler {#nodes-cma-autoscaling-custom-install}

<a id="nodes-cma-autoscaling-custom-install"></a>

You can use the OpenShift Container Platform web console to install the Custom Metrics Autoscaler Operator.

The installation creates the following five CRDs:

- `ClusterTriggerAuthentication`
- `KedaController`
- `ScaledJob`
- `ScaledObject`
- `TriggerAuthentication`

The installation process also creates the `KedaController` custom resource (CR). You can modify the default `KedaController` CR, if needed. For more information, see "Editing the Keda Controller CR".

:::note

If you are installing a Custom Metrics Autoscaler Operator version lower than 2.17.2, you must manually create the Keda Controller CR. You can use the procedure described in "Editing the Keda Controller CR" to create the CR.

:::

## Install the custom metrics autoscaler {#nodes-cma-autoscaling-custom-install_nodes-cma-autoscaling-custom-install}

You can use the following procedure to install the Custom Metrics Autoscaler Operator.

**Prerequisites**

- Remove any previously-installed Technology Preview versions of the Cluster Metrics Autoscaler Operator.
- Remove any versions of the community-based KEDA.
  Also, remove the KEDA 1.x custom resource definitions by running the following commands:

  ```terminal
  $ oc delete crd scaledobjects.keda.k8s.io
  ```

  ```terminal
  $ oc delete crd triggerauthentications.keda.k8s.io
  ```
- Optional: If you need the Custom Metrics Autoscaler Operator to connect to off-cluster services, such as an external Kafka cluster or an external Prometheus service, put any required service CA certificates into a config map. The config map must exist in the same namespace where the Operator is installed. For example:
  ```terminal
  $ oc create configmap -n openshift-keda thanos-cert  --from-file=ca-cert.pem
  ```

**Procedure**

1. In the OpenShift Container Platform web console, click **Ecosystem** → **Software Catalog**.
2. Choose **Custom Metrics Autoscaler** from the list of available Operators, and click **Install**.
3. On the **Install Operator** page, ensure that the **All namespaces on the cluster (default)** option is selected for **Installation Mode**. This installs the Operator in all namespaces.
4. Ensure that the **openshift-keda** namespace is selected for **Installed Namespace**. OpenShift Container Platform creates the namespace, if not present in your cluster.
5. Click **Install**.
6. Verify the installation by listing the Custom Metrics Autoscaler Operator components:
   1. Navigate to **Workloads** → **Pods**.
   2. Select the `openshift-keda` project from the drop-down menu and verify that the `custom-metrics-autoscaler-operator-*` pod is running.
   3. Navigate to **Workloads** → **Deployments** to verify that the `custom-metrics-autoscaler-operator` deployment is running.
7. Optional: Verify the installation in the OpenShift CLI using the following commands:
   ```terminal
   $ oc get all -n openshift-keda
   ```

   The output appears similar to the following:

   ```terminal title="Example output"
   NAME                                                      READY   STATUS    RESTARTS   AGE
   pod/custom-metrics-autoscaler-operator-5fd8d9ffd8-xt4xp   1/1     Running   0          18m

   NAME                                                 READY   UP-TO-DATE   AVAILABLE   AGE
   deployment.apps/custom-metrics-autoscaler-operator   1/1     1            1           18m

   NAME                                                            DESIRED   CURRENT   READY   AGE
   replicaset.apps/custom-metrics-autoscaler-operator-5fd8d9ffd8   1         1         1       18m
   ```

## Edit the Keda Controller CR {#nodes-cma-autoscaling-keda-controller-edit_nodes-cma-autoscaling-custom-install}

You can use the following procedure to modify the `KedaController` custom resource (CR), which is automatically installed during the installation of the Custom Metrics Autoscaler Operator.

**Procedure**

1. In the OpenShift Container Platform web console, click **Ecosystem** → **Installed Operators**.
2. Click **Custom Metrics Autoscaler**.
3. On the **Operator Details** page, click the **KedaController** tab.
4. On the **KedaController** tab, click **Create KedaController** and edit the file.
   ```yaml
   kind: KedaController
   apiVersion: keda.sh/v1alpha1
   metadata:
     name: keda
     namespace: openshift-keda
   spec:
     watchNamespace: ''
     operator:
       logLevel: info
       logEncoder: console
       caConfigMaps:
       - thanos-cert
       - kafka-cert
       volumeMounts:
       - mountPath: /<path_to_directory>
         name: <name>
       volumes:
       - name: <volume_name>
         emptyDir:
           medium: Memory
     metricsServer:
       logLevel: '0'
       auditConfig:
         logFormat: "json"
         logOutputVolumeClaim: "persistentVolumeClaimName"
         policy:
           rules:
           - level: Metadata
           omitStages: ["RequestReceived"]
           omitManagedFields: false
         lifetime:
           maxAge: "2"
           maxBackup: "1"
           maxSize: "50"
     serviceAccount: {}
   ```

   where:

   <dl>
   <dt><code>spec.watchNamespace</code></dt>
   <dd>Specifies a single namespace in which the Custom Metrics Autoscaler Operator scales applications. Leave it blank or leave it empty to scale applications in all namespaces. This field should have a namespace or be empty. The default value is empty.</dd>
   <dt><code>spec.operator.logLevel</code></dt>
   <dd>Specifies the level of verbosity for the Custom Metrics Autoscaler Operator log messages. The allowed values are <code>debug</code>, <code>info</code>, <code>error</code>. The default is <code>info</code>.</dd>
   <dt><code>spec.operator.logEncoder</code></dt>
   <dd>Specifies the logging format for the Custom Metrics Autoscaler Operator log messages. The allowed values are <code>console</code> or <code>json</code>. The default is <code>console</code>.</dd>
   <dt><code>spec.operator.caConfigMaps</code></dt>
   <dd>Specifies one or more config maps with CA certificates, which the Custom Metrics Autoscaler Operator can use to connect securely to TLS-enabled metrics sources. This field is optional.</dd>
   <dt><code>spec.operator.volumeMounts</code></dt>
   <dd>Specifies the container mount path. This field is optional.</dd>
   <dt><code>spec.operator.volumes</code></dt>
   <dd>Specifies a <code>volumes</code> block to list each projected volume source. This field is optional.</dd>
   <dt><code>spec.metricsServer.logLevel</code></dt>
   <dd>Specifies the logging level for the Custom Metrics Autoscaler Metrics Server. The allowed values are <code>0</code> for <code>info</code> and <code>4</code> for <code>debug</code>. The default is <code>0</code>.</dd>
   <dt><code>spec.metricsServer.auditConfig</code></dt>
   <dd>Specifies the audit logging configuration for the Custom Metrics Autoscaler Operator and the audit policy to use, as described in the "Configuring audit logging" section.</dd>
   </dl>
5. Click **Save** to save the changes.
