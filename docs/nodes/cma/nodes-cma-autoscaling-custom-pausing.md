---
title: Pausing the custom metrics autoscaler for a scaled object
sidebar_position: 7
---

# Pausing the custom metrics autoscaler for a scaled object {#nodes-cma-autoscaling-custom-pausing}

<a id="nodes-cma-autoscaling-custom-pausing"></a>

You can pause and restart the autoscaling of a workload, as needed.

For example, you might want to pause autoscaling before performing cluster maintenance or to avoid resource starvation by removing non-mission-critical workloads.

## Pause a custom metrics autoscaler {#nodes-cma-autoscaling-custom-pausing-workload_nodes-cma-autoscaling-custom-pausing}

You can pause the autoscaling of a scaled object by adding the `autoscaling.keda.sh/paused-replicas` annotation to the custom metrics autoscaler for that scaled object.

The custom metrics autoscaler scales the replicas for that workload to the specified value and pauses autoscaling until the annotation is removed.

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  annotations:
    autoscaling.keda.sh/paused-replicas: "4"
# ...
```

**Procedure**

1. Use the following command to edit the `ScaledObject` CR for your workload:
   ```terminal
   $ oc edit ScaledObject scaledobject
   ```
2. Add the `autoscaling.keda.sh/paused-replicas` annotation with any value:
   ```yaml
   apiVersion: keda.sh/v1alpha1
   kind: ScaledObject
   metadata:
     annotations:
       autoscaling.keda.sh/paused-replicas: "4"
     creationTimestamp: "2023-02-08T14:41:01Z"
     generation: 1
     name: scaledobject
     namespace: my-project
     resourceVersion: '65729'
     uid: f5aec682-acdf-4232-a783-58b5b82f5dd0
   ```

   where:

   <dl>
   <dt><code>metadata.annotations.autoscaling.keda.sh/paused-replicas</code></dt>
   <dd>Specifies that the Custom Metrics Autoscaler Operator is to scale the replicas to the specified value and stop autoscaling.</dd>
   </dl>

## Restart the custom metrics autoscaler for a scaled object {#nodes-cma-autoscaling-custom-pausing-restart_nodes-cma-autoscaling-custom-pausing}

You can restart a paused custom metrics autoscaler by removing the `autoscaling.keda.sh/paused-replicas` annotation for that `ScaledObject`.

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  annotations:
    autoscaling.keda.sh/paused-replicas: "4"
# ...
```

**Procedure**

1. Use the following command to edit the `ScaledObject` CR for your workload:
   ```terminal
   $ oc edit ScaledObject scaledobject
   ```
2. Remove the `autoscaling.keda.sh/paused-replicas` annotation to restart a paused custom metrics autoscaler:
   ```yaml
   apiVersion: keda.sh/v1alpha1
   kind: ScaledObject
   metadata:
     annotations:
       autoscaling.keda.sh/paused-replicas: "4"
     creationTimestamp: "2023-02-08T14:41:01Z"
     generation: 1
     name: scaledobject
     namespace: my-project
     resourceVersion: '65729'
     uid: f5aec682-acdf-4232-a783-58b5b82f5dd0
   ```
