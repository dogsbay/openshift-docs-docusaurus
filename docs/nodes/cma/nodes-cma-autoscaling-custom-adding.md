---
title: Understanding how to add custom metrics autoscalers
sidebar_position: 6
---

# Understanding how to add custom metrics autoscalers {#nodes-cma-autoscaling-custom-adding}

<a id="nodes-cma-autoscaling-custom-adding"></a>

To add a custom metrics autoscaler, create a `ScaledObject` custom resource for a deployment, stateful set, or custom resource. Create a `ScaledJob` custom resource for a job.

You can create only one scaled object for each workload that you want to scale. Also, you cannot use a scaled object and the horizontal pod autoscaler (HPA) on the same workload.

## Add a custom metrics autoscaler to a workload {#nodes-cma-autoscaling-custom-creating-workload_nodes-cma-autoscaling-custom-adding}

You can create a custom metrics autoscaler for a workload that is created by a `Deployment`, `StatefulSet`, or `custom resource` object.

**Prerequisites**

- The Custom Metrics Autoscaler Operator must be installed.
- If you use a custom metrics autoscaler for scaling based on CPU or memory:
  - Your cluster administrator must have properly configured cluster metrics. You can use the `oc describe PodMetrics <pod-name>` command to determine if metrics are configured. If metrics are configured, the output appears similar to the following, with CPU and Memory displayed under Usage.
    ```terminal
    $ oc describe PodMetrics openshift-kube-scheduler-ip-10-0-135-131.ec2.internal
    ```

    ```yaml title="Example output"
    Name:         openshift-kube-scheduler-ip-10-0-135-131.ec2.internal
    Namespace:    openshift-kube-scheduler
    Labels:       <none>
    Annotations:  <none>
    API Version:  metrics.k8s.io/v1beta1
    Containers:
      Name:  wait-for-host-port
      Usage:
        Memory:  0
      Name:      scheduler
      Usage:
        Cpu:     8m
        Memory:  45440Ki
    Kind:        PodMetrics
    Metadata:
      Creation Timestamp:  2019-05-23T18:47:56Z
      Self Link:           /apis/metrics.k8s.io/v1beta1/namespaces/openshift-kube-scheduler/pods/openshift-kube-scheduler-ip-10-0-135-131.ec2.internal
    Timestamp:             2019-05-23T18:47:56Z
    Window:                1m0s
    Events:                <none>
    ```
  - The pods associated with the object you want to scale must include specified memory and CPU limits. For example:
    ```yaml title="Example pod spec"
    apiVersion: v1
    kind: Pod
    # ...
    spec:
      containers:
      - name: app
        image: images.my-company.example/app:v4
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
    # ...
    ```

**Procedure**

1. Create a YAML file similar to the following. Only the name `<2>`, object name `<4>`, and object kind `<5>` are required:
   ```yaml title="Example scaled object"
   apiVersion: keda.sh/v1alpha1
   kind: ScaledObject
   metadata:
     annotations:
       autoscaling.keda.sh/paused-replicas: "0"
     name: scaledobject
     namespace: my-namespace
   spec:
     scaleTargetRef:
       apiVersion: apps/v1
       name: example-deployment
       kind: Deployment
       envSourceContainerName: .spec.template.spec.containers[0]
     cooldownPeriod:  200
     maxReplicaCount: 100
     minReplicaCount: 0
     metricsServer:
       auditConfig:
         logFormat: "json"
         logOutputVolumeClaim: "persistentVolumeClaimName"
         policy:
           rules:
           - level: Metadata
           omitStages: "RequestReceived"
           omitManagedFields: false
         lifetime:
           maxAge: "2"
           maxBackup: "1"
           maxSize: "50"
     fallback:
       failureThreshold: 3
       replicas: 6
       behavior: static
     pollingInterval: 30
     advanced:
       restoreToOriginalReplicaCount: false
       horizontalPodAutoscalerConfig:
         name: keda-hpa-scale-down
         behavior:
           scaleDown:
             stabilizationWindowSeconds: 300
             policies:
             - type: Percent
               value: 100
               periodSeconds: 15
     triggers:
     - type: prometheus
       metadata:
         serverAddress: https://thanos-querier.openshift-monitoring.svc.cluster.local:9092
         namespace: kedatest
         metricName: http_requests_total
         threshold: '5'
         query: sum(rate(http_requests_total{job="test-app"}[1m]))
         authModes: basic
       authenticationRef:
         name: prom-triggerauthentication
         kind: TriggerAuthentication
   ```

   where:

   <dl>
   <dt><code>metadata.annotations.autoscaling.keda.sh/paused-replicas</code></dt>
   <dd>Specifies that the Custom Metrics Autoscaler Operator is to scale the replicas to the specified value and stop autoscaling, as described in the "Pausing the custom metrics autoscaler for a workload" section. This field is optional.</dd>
   <dt><code>metadata.name</code></dt>
   <dd>Specifies a name for this custom metrics autoscaler.</dd>
   <dt><code>spec.scaleTargetRef.apiVersion</code></dt>
   <dd>Specifies the API version of the target resource. The default is <code>apps/v1</code>. This field is optional.</dd>
   <dt><code>spec.scaleTargetRef.name</code></dt>
   <dd>Specifies the name of the object that you want to scale.</dd>
   <dt><code>spec.scaleTargetRef.kind</code></dt>
   <dd>Specifies the <code>kind</code> as <code>Deployment</code>, <code>StatefulSet</code> or <code>CustomResource</code>.</dd>
   <dt><code>spec.scaleTargetRef.envSourceContainerName</code></dt>
   <dd>Specifies the name of the container in the target resource, from which the custom metrics autoscaler gets environment variables holding secrets and so forth. The default is <code>.spec.template.spec.containers[0]</code>. This field is optional.</dd>
   <dt><code>spec.cooldownPeriod</code></dt>
   <dd>Specifies the period in seconds to wait after the last trigger is reported before scaling the deployment back to <code>0</code> if the <code>minReplicaCount</code> is set to <code>0</code>. The default is <code>300</code>. This field is optional.</dd>
   <dt><code>spec.maxReplicaCount</code></dt>
   <dd>Specifies the maximum number of replicas when scaling up. The default is <code>100</code>. This field is optional.</dd>
   <dt><code>spec.minReplicaCount</code></dt>
   <dd>Specifies the minimum number of replicas when scaling down. This field is optional.</dd>
   <dt><code>spec.metricsServer</code></dt>
   <dd>Specifies the parameters for audit logs. as described in the "Configuring audit logging" section. This field is optional.</dd>
   <dt><code>spec.fallback</code></dt>
   <dd>Specifies the number of replicas to fall back to if a scaler fails to get metrics from the source for the number of times defined by the <code>failureThreshold</code> parameter. For more information on fallback behavior, see the <a href="https://keda.sh/docs/latest/reference/scaledobject-spec/#fallback">KEDA documentation</a>. This field is optional.</dd>
   <dt><code>spec.fallback.behavior</code></dt>
   <dd>Specifies the replica count to be used if a fallback occurs. Enter one of the following options or omit the parameter. This field is optional. *   Enter <code>static</code> to use the number of replicas specified by the <code>fallback.replicas</code> parameter. This is the default. *   Enter <code>currentReplicas</code> to maintain the current number of replicas. *   Enter <code>currentReplicasIfHigher</code> to maintain the current number of replicas, if that number is higher than the <code>fallback.replicas</code> parameter. If the current number of replicas is lower than the <code>fallback.replicas</code> parameter, use the <code>fallback.replicas</code> value. *   Enter <code>currentReplicasIfLower</code> to maintain the current number of replicas, if that number is lower than the <code>fallback.replicas</code> parameter. If the current number of replicas is higher than the <code>fallback.replicas</code> parameter, use the <code>fallback.replicas</code> value.</dd>
   <dt><code>spec.pollingInterval</code></dt>
   <dd>Specifies the interval in seconds to check each trigger on. The default is <code>30</code>. This field is optional.</dd>
   <dt><code>spec.advanced.restoreToOriginalReplicaCount</code></dt>
   <dd>Specifies whether to scale back the target resource to the original replica count after the scaled object is deleted. The default is <code>false</code>, which keeps the replica count as it is when the scaled object is deleted. This field is optional.</dd>
   <dt><code>spec.advanced.horizontalPodAutoscalerConfig.name</code></dt>
   <dd>Specifies a name for the horizontal pod autoscaler. The default is <code>keda-hpa-{scaled_object_name}</code>. This field is optional.</dd>
   <dt><code>spec.advanced.horizontalPodAutoscalerConfig.behavior</code></dt>
   <dd>Specifies a scaling policy to use to control the rate to scale pods up or down, as described in the "Scaling policies" section. This field is optional.</dd>
   <dt><code>spec.triggers[].type</code></dt>
   <dd>Specifies the trigger to use as the basis for scaling, as described in the "Understanding the custom metrics autoscaler triggers" section. This example uses OpenShift Container Platform monitoring.</dd>
   <dt><code>spec.triggers[].authenticationRef</code></dt>
   <dd>Specifies a trigger authentication or a cluster trigger authentication. For more information, see "Understanding the custom metrics autoscaler trigger authentication". This field is optional. *   Enter <code>TriggerAuthentication</code> to use a trigger authentication. This is the default. *   Enter <code>ClusterTriggerAuthentication</code> to use a cluster trigger authentication.</dd>
   </dl>
2. Create the custom metrics autoscaler by running the following command:
   ```terminal
   $ oc create -f <filename>.yaml
   ```

**Verification**

- View the command output to verify that the custom metrics autoscaler was created:
  ```terminal
  $ oc get scaledobject <scaled_object_name>
  ```

  ```terminal title="Example output"
  NAME            SCALETARGETKIND      SCALETARGETNAME        MIN   MAX   TRIGGERS     AUTHENTICATION               READY   ACTIVE   FALLBACK   AGE
  scaledobject    apps/v1.Deployment   example-deployment     0     50    prometheus   prom-triggerauthentication   True    True     True       17s
  ```

  Note the following fields in the output:

  - `TRIGGERS`: Indicates the trigger, or scaler, that is being used.
  - `AUTHENTICATION`: Indicates the name of any trigger authentication being used.
  - `READY`: Indicates whether the scaled object is ready to start scaling:
    - If `True`, the scaled object is ready.
    - If `False`, the scaled object is not ready because of a problem in one or more of the objects you created.
  - `ACTIVE`: Indicates whether scaling is taking place:
    - If `True`, scaling is taking place.
    - If `False`, scaling is not taking place because there are no metrics or there is a problem in one or more of the objects you created.
  - `FALLBACK`: Indicates whether the custom metrics autoscaler is able to get metrics from the source
    - If `False`, the custom metrics autoscaler is getting metrics.
    - If `True`, the custom metrics autoscaler is getting metrics because there are no metrics or there is a problem in one or more of the objects you created.

## Add a custom metrics autoscaler to a job {#nodes-cma-autoscaling-custom-creating-job_nodes-cma-autoscaling-custom-adding}

You can create a custom metrics autoscaler for any `Job` object.

:::warning

Scaling by using a scaled job is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.

For more information about the support scope of Red Hat Technology Preview features, see [Technology Preview Features Support Scope](https://access.redhat.com/support/offerings/techpreview/).

:::

**Prerequisites**

- The Custom Metrics Autoscaler Operator must be installed.

**Procedure**

1. Create a YAML file similar to the following:
   ```yaml
   kind: ScaledJob
   apiVersion: keda.sh/v1alpha1
   metadata:
     name: scaledjob
     namespace: my-namespace
   spec:
     failedJobsHistoryLimit: 5
     jobTargetRef:
       activeDeadlineSeconds: 600
       backoffLimit: 6
       parallelism: 1
       completions: 1
       template:
         metadata:
           name: pi
         spec:
           containers:
           - name: pi
             image: perl
             command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
     maxReplicaCount: 100
     pollingInterval: 30
     successfulJobsHistoryLimit: 5
     failedJobsHistoryLimit: 5
     envSourceContainerName:
     rolloutStrategy: gradual
     scalingStrategy:
       strategy: "custom"
       customScalingQueueLengthDeduction: 1
       customScalingRunningJobPercentage: "0.5"
       pendingPodConditions:
         - "Ready"
         - "PodScheduled"
         - "AnyOtherCustomPodCondition"
       multipleScalersCalculation : "max"
     triggers:
     - type: prometheus
       metadata:
         serverAddress: https://thanos-querier.openshift-monitoring.svc.cluster.local:9092
         namespace: kedatest
         metricName: http_requests_total
         threshold: '5'
         query: sum(rate(http_requests_total{job="test-app"}[1m]))
         authModes: "bearer"
       authenticationRef:
         name: prom-cluster-triggerauthentication
   ```

   where:

   <dl>
   <dt><code>spec.jobTargetRef.activeDeadlineSeconds</code></dt>
   <dd>Specifies the maximum duration the job can run.</dd>
   <dt><code>spec.jobTargetRef.backoffLimit</code></dt>
   <dd>Specifies the number of retries for a job. The default is <code>6</code>.</dd>
   <dt><code>spec.jobTargetRef.parallelism</code></dt>
   <dd>Specifies how many pod replicas a job should run in parallel; defaults to <code>1</code>. This field is optional. *   For non-parallel jobs, leave unset. When unset, the default is <code>1</code>.</dd>
   <dt><code>spec.jobTargetRef.completions</code></dt>
   <dd>Specifies how many successful pod completions are needed to mark a job completed. This field is optional. *   For non-parallel jobs, leave unset. When unset,  the default is <code>1</code>. *   For parallel jobs with a fixed completion count, specify the number of completions. *   For parallel jobs with a work queue, leave unset. When unset the default is the value of the <code>parallelism</code> parameter.</dd>
   <dt><code>spec.jobTargetRef.template</code></dt>
   <dd>Specifies the template for the pod the controller creates.</dd>
   <dt><code>spec.maxReplicaCount</code></dt>
   <dd>Specifies the maximum number of replicas when scaling up. The default is <code>100</code>. This field is optional.</dd>
   <dt><code>spec.pollingInterval</code></dt>
   <dd>Specifies the interval in seconds to check each trigger on. The default is <code>30</code>. This field is optional.</dd>
   <dt><code>spec.successfulJobsHistoryLimit</code></dt>
   <dd>Specifies the number of successful finished jobs should be kept. The default is <code>100</code>. This field is optional.</dd>
   <dt><code>spec.failedJobsHistoryLimit</code></dt>
   <dd>Specifies how many failed jobs should be kept. The default is <code>100</code>. This field is optional.</dd>
   <dt><code>spec.envSourceContainerName</code></dt>
   <dd>Specifies the name of the container in the target resource, from which the custom autoscaler gets environment variables holding secrets and so forth. The default is <code>.spec.template.spec.containers[0]</code>. This field is optional.</dd>
   <dt><code>spec.rolloutStrategy</code></dt>
   <dd>Specifies whether existing jobs are terminated whenever a scaled job is being updated. This field is optional. *   <code>default</code>: The autoscaler terminates an existing job if its associated scaled job is updated. The autoscaler recreates the job with the latest specs. *   <code>gradual</code>: The autoscaler does not terminate an existing job if its associated scaled job is updated. The autoscaler creates new jobs with the latest specs.</dd>
   <dt><code>spec.scalingStrategy</code></dt>
   <dd>Specifies a scaling strategy: <code>default</code>, <code>custom</code>, or <code>accurate</code>. The default is <code>default</code>. This field is optional.</dd>
   <dt><code>spec.triggers[].type</code></dt>
   <dd>Specifies the trigger to use as the basis for scaling. For more information, see "Understanding custom metrics autoscaler triggers".</dd>
   <dt><code>spec.triggers[].authenticationRef</code></dt>
   <dd>Specifies a trigger authentication or a cluster trigger authentication. For more information, see "Understanding custom metrics autoscaler trigger authentications". This field is optional. *   Enter <code>TriggerAuthentication</code> to use a trigger authentication. This is the default. *   Enter <code>ClusterTriggerAuthentication</code> to use a cluster trigger authentication.</dd>
   </dl>
2. Create the custom metrics autoscaler by running the following command:
   ```terminal
   $ oc create -f <filename>.yaml
   ```

**Verification**

- View the command output to verify that the custom metrics autoscaler was created:
  ```terminal
  $ oc get scaledjob <scaled_job_name>
  ```

  ```terminal title="Example output"
  NAME        MAX   TRIGGERS     AUTHENTICATION              READY   ACTIVE    AGE
  scaledjob   100   prometheus   prom-triggerauthentication  True    True      8s
  ```

  Note the following fields in the output:

  - `TRIGGERS`: Indicates the trigger, or scaler, that is being used.
  - `AUTHENTICATION`: Indicates the name of any trigger authentication being used.
  - `READY`: Indicates whether the scaled object is ready to start scaling:
    - If `True`, the scaled object is ready.
    - If `False`, the scaled object is not ready because of a problem in one or more of the objects you created.
  - `ACTIVE`: Indicates whether scaling is taking place:
    - If `True`, scaling is taking place.
    - If `False`, scaling is not taking place because there are no metrics or there is a problem in one or more of the objects you created.

**Additional resources**

- [Understanding custom metrics autoscaler triggers](/docs/nodes/cma/nodes-cma-autoscaling-custom-trigger#nodes-cma-autoscaling-custom-overview-trigger)
- [Understanding custom metrics autoscaler trigger authentications](/docs/nodes/cma/nodes-cma-autoscaling-custom-trigger-auth#nodes-cma-autoscaling-custom-trigger-auth)
