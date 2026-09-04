---
title: Removing a pod from a secondary network
sidebar_position: 5
---

# Removing a pod from a secondary network {#removing-pod}

<a id="removing-pod"></a>

To disconnect a pod from specific network configurations in OpenShift Container Platform, you can remove the pod from a secondary network. Delete the pod to remove its connection to the secondary network.

## Remove a pod from a secondary network {#nw-multus-remove-pod_removing-pod}

To disconnect a pod from specific network configurations in OpenShift Container Platform, you can remove the pod from a secondary network. Delete the pod using the `oc delete pod` command to remove its connection to the secondary network.

**Prerequisites**

- A secondary network is attached to the pod.
- Install the OpenShift CLI (`oc`).
- Log in to the cluster.

**Procedure**

- Delete the pod by entering the following command:
  ```terminal
  $ oc delete pod <name> -n <namespace>
  ```

  where:

  <dl>
  <dt><code>&lt;name&gt;</code></dt>
  <dd>Specifies the name of the pod.</dd>
  <dt><code>&lt;namespace&gt;</code></dt>
  <dd>Specifies the namespace that contains the pod.</dd>
  </dl>
