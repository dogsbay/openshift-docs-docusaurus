---
title: Cluster maintenance
sidebar_position: 3
---

# Cluster maintenance {#troubleshooting-cluster-maintenance}

<a id="troubleshooting-cluster-maintenance"></a>

When deploying OpenShift Container Platform on bare-metal infrastructure, you must pay more attention to certain configurations which can have a significant impact on cluster stability. You can troubleshoot more effectively by completing these tasks:

- Monitor for failed or failing hardware components
- Periodically check the status of the cluster Operators

:::note

For hardware monitoring, contact your hardware vendor to find the appropriate logging tool for your specific hardware.

:::

## Checking cluster Operators {#troubleshooting-clusters-check-cluster-operators_troubleshooting-cluster-maintenance}

Periodically check the status of your cluster Operators to find issues early.

**Procedure**

- Check the status of the cluster Operators by running the following command:
  ```terminal
  $ oc get co
  ```

## Watching for failed pods {#troubleshooting-clusters-check-for-failed-pods_troubleshooting-cluster-maintenance}

To reduce troubleshooting time, regularly monitor for failed pods in your cluster.

**Procedure**

- To watch for failed pods, run the following command:
  ```terminal
  $ oc get po -A | grep -Eiv 'complete|running'
  ```
