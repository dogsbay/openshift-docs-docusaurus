---
title: Deleting a network policy
sidebar_position: 5
---

# Deleting a network policy {#deleting-network-policy}

<a id="deleting-network-policy"></a>

As a cluster administrator, you can delete a network policy from a namespace.

## Delete a network policy using the CLI {#nw-networkpolicy-delete-cli_deleting-network-policy}

You can delete a network policy in a namespace.

:::note

If you log in with `cluster-admin` privileges, you can delete network policies in any namespace in the cluster.

:::

:::note

If you log in with `cluster-admin` privileges, you can delete network policies in any namespace in the cluster. In the web console, you can delete policies directly in YAML or by using the **Actions** menu.

:::

**Prerequisites**

- Your cluster uses a network plugin that supports `NetworkPolicy` objects, such as the OVN-Kubernetes network plugin, with `mode: NetworkPolicy` set.
- You installed the OpenShift CLI (`oc`).
- You logged in to the cluster with a user with `admin` privileges.
- You are working in the namespace where the network policy exists.

**Procedure**

- To delete a network policy object, enter the following command. Successful output lists the name of the policy object and the `deleted` status.
  ```terminal
  $ oc delete networkpolicy <policy_name> -n <namespace>
  ```

  where:

  <dl>
  <dt><code>&lt;policy_name&gt;</code></dt>
  <dd>Specifies the name of the network policy.</dd>
  <dt><code>&lt;namespace&gt;</code></dt>
  <dd>Optional parameter. If you defined the object in a different namespace than the current namespace, the parameter specifices the namespace.</dd>
  </dl>
