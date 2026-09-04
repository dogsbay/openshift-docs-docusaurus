---
title: Configuring ingress cluster traffic using load balancer allowed source ranges
sidebar_position: 9
---

# Configuring ingress cluster traffic using load balancer allowed source ranges {#configuring-ingress-cluster-traffic-lb-allowed-source-ranges}

<a id="configuring-ingress-cluster-traffic-lb-allowed-source-ranges"></a>

You can specify a list of IP address ranges for the Ingress Controller. This action restricts access to the load balancer service when you specify the `LoadBalancerService` value for the `endpointPublishingStrategy` parameter.

## Configuring load balancer allowed source ranges {#nw-configuring-lb-allowed-source-ranges_configuring-ingress-cluster-traffic-lb-allowed-source-ranges}

You can enable and configure the `spec.endpointPublishingStrategy.loadBalancer.allowedSourceRanges` parameter. By configuring load balancer allowed source ranges, you can limit the access to the load balancer for the Ingress Controller to a specified list of IP address ranges.

The Ingress Operator reconciles the load balancer Service and sets the `spec.loadBalancerSourceRanges` parameter based on `AllowedSourceRanges`.

:::note

If you have already set the `spec.loadBalancerSourceRanges` parameter or the load balancer service anotation `service.beta.kubernetes.io/load-balancer-source-ranges` in a previous version of OpenShift Container Platform, Ingress Controller starts reporting `Progressing=True` after an upgrade. To fix this, set `AllowedSourceRanges` that overwrites the `spec.loadBalancerSourceRanges` parameter and clears the `service.beta.kubernetes.io/load-balancer-source-ranges` annotation. Ingress Controller starts reporting `Progressing=False` again.

:::

**Prerequisites**

- You have a deployed Ingress Controller on a running cluster.

**Procedure**

- Set the allowed source ranges API for the Ingress Controller by running the following command:
  ```terminal
  $ oc -n openshift-ingress-operator patch ingresscontroller/default \
      --type=merge --patch='{"spec":{"endpointPublishingStrategy": \
      {"type":"LoadBalancerService", "loadbalancer": \
      {"scope":"External", "allowedSourceRanges":["0.0.0.0/0"]}}}}'
  ```

  where:

  <dl>
  <dt><code>allowedSourceRanges</code></dt>
  <dd>The example value <code>0.0.0.0/0</code> specifies the allowed source range.</dd>
  </dl>

## Migrating to load balancer allowed source ranges {#nw-configuring-lb-allowed-source-ranges-migration_configuring-ingress-cluster-traffic-lb-allowed-source-ranges}

If you have already set the annotation `service.beta.kubernetes.io/load-balancer-source-ranges`, you can migrate to load balancer allowed source ranges. When you set the `AllowedSourceRanges`, the Ingress Controller sets the `spec.loadBalancerSourceRanges` field based on the `AllowedSourceRanges` value and unsets the `service.beta.kubernetes.io/load-balancer-source-ranges` annotation.

:::note

If you have already set the `spec.loadBalancerSourceRanges` parameter or the load balancer service anotation `service.beta.kubernetes.io/load-balancer-source-ranges` in a previous version of OpenShift Container Platform, the Ingress Controller starts reporting `Progressing=True` after an upgrade. To fix this, set `AllowedSourceRanges` that overwrites the `spec.loadBalancerSourceRanges` parameter and clears the `service.beta.kubernetes.io/load-balancer-source-ranges` annotation. The Ingress Controller starts reporting `Progressing=False` again.

:::

**Prerequisites**

- You have set the `service.beta.kubernetes.io/load-balancer-source-ranges` annotation.

**Procedure**

1. Check that the `service.beta.kubernetes.io/load-balancer-source-ranges` is set by entering the following command:
   ```terminal
   $ oc get svc router-default -n openshift-ingress -o yaml
   ```

   ```yaml title="Example output"
   apiVersion: v1
   kind: Service
   metadata:
     annotations:
       service.beta.kubernetes.io/load-balancer-source-ranges: 192.168.0.1/32
   ```
2. Check that the `spec.loadBalancerSourceRanges` parameter is unset by entering the following command:
   ```terminal
   $ oc get svc router-default -n openshift-ingress -o yaml
   ```

   ```yaml title="Example output"
   ...
   spec:
     loadBalancerSourceRanges:
     - 0.0.0.0/0
   ...
   ```
3. Update your cluster to OpenShift Container Platform 4.22.
4. Set the allowed source ranges API for the `ingresscontroller` by running the following command:
   ```terminal
   $ oc -n openshift-ingress-operator patch ingresscontroller/default \
       --type=merge --patch='{"spec":{"endpointPublishingStrategy": \
       {"loadBalancer":{"allowedSourceRanges":["0.0.0.0/0"]}}}}'
   ```

   where:

   <dl>
   <dt><code>allowedSourceRanges</code></dt>
   <dd>The example value <code>0.0.0.0/0</code> specifies the allowed source range.</dd>
   </dl>

**Additional resources**

- [Introduction to OpenShift updates](/docs/updating/understanding_updates/intro-to-updates#understanding-openshift-updates)
