---
title: Configuring log levels for cert-manager and the cert-manager Operator for Red Hat OpenShift
sidebar_position: 14
---

# Configuring log levels for cert-manager and the cert-manager Operator for Red Hat OpenShift {#cert-manager-log-levels}

<a id="cert-manager-log-levels"></a>

To troubleshoot issues with the cert-manager components and the cert-manager Operator for Red Hat OpenShift, you can configure the log level verbosity.

:::note

To use different log levels for different cert-manager components, see *Customizing cert-manager Operator API fields*.

:::

## Setting a log level for cert-manager {#cert-manager-enable-operand-log-level_cert-manager-log-levels}

To troubleshoot issues and control log volume, configure the log level for the cert-manager Operator for Red Hat OpenShift. You can set specific verbosity levels to capture the necessary details for debugging or to reduce noise in your cluster logs.

**Prerequisites**

- You have access to the cluster with `cluster-admin` privileges.
- You have installed version 1.11.1 or later of the cert-manager Operator for Red Hat OpenShift.

**Procedure**

1. Edit the `CertManager` resource by running the following command:
   ```terminal
   $ oc edit certmanager.operator cluster
   ```
2. Set the log level value by editing the `spec.logLevel` section:
   ```yaml
   apiVersion: operator.openshift.io/v1alpha1
   kind: CertManager
   ...
   spec:
     logLevel: <log_level>
   ```

   The `CertManager` resource supports the following `logLevel` values:

   <dl>
   <dt><code>Normal</code></dt>
   <dd>Audits logs and records common operations. The default setting. Use this level when there are no issues.</dd>
   <dt><code>Debug</code></dt>
   <dd>Provides verbose logs. Use this level to troubleshoot minor issues.</dd>
   <dt><code>Trace</code></dt>
   <dd>Provides highly verbose logs. Use this level to troubleshoot major issues.</dd>
   <dt><code>TraceAll</code></dt>
   <dd>Provides maximum log detail. Use this level to troubleshoot serious issues.</dd>
   </dl>

   :::note

   `TraceAll` generates huge amount of logs. After setting `logLevel` to `TraceAll`, you might experience performance issues.

   :::
3. Save your changes and quit the text editor to apply your changes.
   After applying the changes, the verbosity level for the cert-manager components controller, CA injector, and webhook is updated.

## Setting a log level for the cert-manager Operator for Red Hat OpenShift {#cert-manager-enable-operator-log-level_cert-manager-log-levels}

To troubleshoot issues and control log volume, set the log level for the cert-manager Operator for Red Hat OpenShift. You can configure the verbosity of the Operator log messages to capture the specific details required for your environment.

**Prerequisites**

- You have access to the cluster with `cluster-admin` privileges.
- You have installed version 1.11.1 or later of the cert-manager Operator for Red Hat OpenShift.

**Procedure**

- Update the subscription object for cert-manager Operator for Red Hat OpenShift to provide the verbosity level for the operator logs by running the following command:
  ```terminal
  $ oc -n cert-manager-operator patch subscription openshift-cert-manager-operator --type='merge' -p '{"spec":{"config":{"env":[{"name":"OPERATOR_LOG_LEVEL","value":"v"}]}}}'
  ```

  Replace `v` with the desired log level number. The valid values for `v` can range from `1`to `10`. The default value is `2`.

**Verification**

1. The cert-manager Operator pod is redeployed. Verify that the log level of the cert-manager Operator for Red Hat OpenShift is updated by running the following command:
   ```terminal
   $ oc set env deploy/cert-manager-operator-controller-manager -n cert-manager-operator --list | grep -e OPERATOR_LOG_LEVEL -e container
   ```

   ```terminal title="Example output"
   # deployments/cert-manager-operator-controller-manager, container kube-rbac-proxy
   OPERATOR_LOG_LEVEL=9
   # deployments/cert-manager-operator-controller-manager, container cert-manager-operator
   OPERATOR_LOG_LEVEL=9
   ```
2. Verify that the log level of the cert-manager Operator for Red Hat OpenShift is updated by running the `oc logs` command:
   ```terminal
   $ oc logs deploy/cert-manager-operator-controller-manager -n cert-manager-operator
   ```

**Additional resources**

- [Customizing cert-manager Operator API fields](/docs/security/cert_manager_operator/cert-manager-customizing-api-fields#cert-manager-customizing-api-fields)
