---
title: Disabling the web console in OpenShift Container Platform
sidebar_position: 9
---

# Disabling the web console in OpenShift Container Platform {#disabling-web-console}

<a id="disabling-web-console"></a>

You can disable the OpenShift Container Platform web console.

## Prerequisites {#_prerequisites}

- Deploy an OpenShift Container Platform cluster.

## Disabling the web console {#web-console-disable_disabling-web-console}

You can disable the web console by editing the `consoles.operator.openshift.io` resource.

**Procedure**

- Edit the `consoles.operator.openshift.io` resource:
  ```terminal
  $ oc edit consoles.operator.openshift.io cluster
  ```

  The following example displays the parameters from this resource that you can modify:

  ```yaml
  apiVersion: operator.openshift.io/v1
  kind: Console
  metadata:
    name: cluster
  spec:
    managementState: Removed
  ```

  where:

  <dl>
  <dt><code>spec.managementState.Removed</code></dt>
  <dd>Set the <code>managementState</code> parameter value to <code>Removed</code> to disable the web console. The other valid values for this parameter are <code>Managed</code>, which enables the console under the cluster’s control, and <code>Unmanaged</code>, which means that you are taking control of web console management.</dd>
  </dl>
