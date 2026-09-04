---
title: Configure an external gateway on the default network
sidebar_position: 8
---

# Configure an external gateway on the default network {#configuring-secondary-external-gateway}

<a id="configuring-secondary-external-gateway"></a>

As a cluster administrator, you can configure an external gateway on the default network.

This feature offers the following benefits:

- Granular control over egress traffic on a per-namespace basis
- Flexible configuration of static and dynamic external gateway IP addresses
- Support for both IPv4 and IPv6 address families

## Prerequisites {#configuring-secondary-external-gateway_prerequisites}

- Your cluster uses the OVN-Kubernetes network plugin.
- Your infrastructure is configured to route traffic from the secondary external gateway.

## How OpenShift Container Platform determines the external gateway IP address {#nw-secondary-ext-gw-about_configuring-secondary-external-gateway}

You configure a secondary external gateway with the `AdminPolicyBasedExternalRoute` custom resource (CR) from the `k8s.ovn.org` API group. The CR supports static and dynamic approaches for specifying an IP address for an external gateway.

Each namespace that an `AdminPolicyBasedExternalRoute` CR targets cannot be selected by any other `AdminPolicyBasedExternalRoute` CR. A namespace cannot have concurrent secondary external gateways.

Changes to policies are isolated in the controller. If a policy fails to apply, changes to other policies do not trigger a retry of other policies. Policies are re-evaluated when updates occur to the policy or to related objects such as target namespaces, pod gateways, or the namespaces that host them from dynamic hops. When re-evaluated, the policy applies any differences from the changes.

Static assignment
: You specify an IP address directly.

Dynamic assignment
: You specify an IP address indirectly, with namespace and pod selectors, and an optional network attachment definition.

:::warning

If the name of a network attachment definition is provided, the external gateway IP address of the network attachment is used.

If the name of a network attachment definition is not provided, the external gateway IP address for the pod itself is used. However, this approach works only if the pod is configured with `hostNetwork` set to `true`.

:::

## AdminPolicyBasedExternalRoute object configuration {#nw-secondary-ext-gw-object_configuring-secondary-external-gateway}

You can define an `AdminPolicyBasedExternalRoute` object, which is cluster scoped, with specific properties.

A namespace can be selected by only one `AdminPolicyBasedExternalRoute` CR at a time.

The following tables detail supported fields for objects.

`AdminPolicyBasedExternalRoute`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>metadata.name</code></td>
  <td><code>string</code></td>
  <td>Specifies the name of the  <code>AdminPolicyBasedExternalRoute</code> object.</td>
</tr>
<tr>
  <td><code>spec.from</code></td>
  <td><code>string</code></td>
  <td>Specifies a namespace selector that the routing policies apply to. Only <code>namespaceSelector</code> is supported for external traffic. For example:<br><br><pre><code>from:&#10;  namespaceSelector:&#10;    matchLabels:&#10;      kubernetes.io/metadata.name: novxlan-externalgw-ecmp-4059</code></pre><br><br>A namespace can only be targeted by one <code>AdminPolicyBasedExternalRoute</code> CR. If a namespace is selected by more than one <code>AdminPolicyBasedExternalRoute</code> CR, a <code>failed</code> error status occurs on the second and subsequent CRs that target the same namespace. To apply updates, you must change the policy itself or related objects such as target namespaces, pod gateways, or namespaces hosting them from dynamic hops. The policy is then re-evaluated and your changes are applied.</td>
</tr>
<tr>
  <td><code>spec.nextHops</code></td>
  <td><code>object</code></td>
  <td>Specifies the destinations where the packets are forwarded to. Must be either or both of <code>static</code> and <code>dynamic</code>. You must have at least one next hop defined.</td>
</tr>
</tbody>
</table>

`nextHops`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>static</code></td>
  <td><code>array</code></td>
  <td>Specifies an array of static IP addresses.</td>
</tr>
<tr>
  <td><code>dynamic</code></td>
  <td><code>array</code></td>
  <td>Specifies an array of pod selectors corresponding to pods configured with a network attachment definition to use as the external gateway target.</td>
</tr>
</tbody>
</table>

`nextHops.static`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>ip</code></td>
  <td><code>string</code></td>
  <td>Specifies either an IPv4 or IPv6 address of the next destination hop.</td>
</tr>
<tr>
  <td><code>bfdEnabled</code></td>
  <td><code>boolean</code></td>
  <td>Optional field. Specifies whether Bi-Directional Forwarding Detection (BFD) is supported by the network. The default value is <code>false</code>.</td>
</tr>
</tbody>
</table>

`nextHops.dynamic`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>podSelector</code></td>
  <td><code>string</code></td>
  <td>Specifies a set-based label selector to filter the pods in the namespace that match this network configuration. For more information, see "Set-based requirement" in the <em>Additional resources</em> section.</td>
</tr>
<tr>
  <td><code>namespaceSelector</code></td>
  <td><code>string</code></td>
  <td>Specifies a <code>set-based</code> selector to filter the namespaces that the <code>podSelector</code> applies to. You must specify a value for this field.</td>
</tr>
<tr>
  <td><code>bfdEnabled</code></td>
  <td><code>boolean</code></td>
  <td>Optional field. Specifies whether Bi-Directional Forwarding Detection (BFD) is supported by the network. The default value is <code>false</code>.</td>
</tr>
<tr>
  <td><code>networkAttachmentName</code></td>
  <td><code>string</code></td>
  <td>Optional field. Specifies the name of a network attachment definition. The name must match the list of logical networks associated with the pod. If this field is not specified, the host network of the pod is used. However, the pod must be configured as a host network pod to use the host network.</td>
</tr>
</tbody>
</table>

**Additional resources**

- [Set-based requirement (Kubernetes)](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#set-based-requirement)

## Example secondary external gateway configurations {#example-secondary-external-gateway-configurations_configuring-secondary-external-gateway}

Reference the `AdminPolicyBasedExternalRoute` objects to better understand secondary external gateway configurations.

In the following example, the `AdminPolicyBasedExternalRoute` object configures two static IP addresses as external gateways for pods in namespaces with the `kubernetes.io/metadata.name: novxlan-externalgw-ecmp-4059` label:

```yaml
apiVersion: k8s.ovn.org/v1
kind: AdminPolicyBasedExternalRoute
metadata:
  name: default-route-policy
spec:
  from:
    namespaceSelector:
      matchLabels:
        kubernetes.io/metadata.name: novxlan-externalgw-ecmp-4059
  nextHops:
    static:
    - ip: "172.18.0.8"
    - ip: "172.18.0.9"
# ...
```

In the following example, the `AdminPolicyBasedExternalRoute` object configures a dynamic external gateway. The IP addresses used for the external gateway are derived from the additional network attachments associated with each of the selected pods.

```yaml
apiVersion: k8s.ovn.org/v1
kind: AdminPolicyBasedExternalRoute
metadata:
  name: shadow-traffic-policy
spec:
  from:
    namespaceSelector:
      matchLabels:
        externalTraffic: ""
  nextHops:
    dynamic:
    - podSelector:
        matchLabels:
          gatewayPod: ""
      namespaceSelector:
        matchLabels:
          shadowTraffic: ""
      networkAttachmentName: shadow-gateway
    - podSelector:
        matchLabels:
          gigabyteGW: ""
      namespaceSelector:
        matchLabels:
          gatewayNamespace: ""
      networkAttachmentName: gateway
# ...
```

In the following example, the `AdminPolicyBasedExternalRoute` object configures both static and dynamic external gateways:

```yaml
apiVersion: k8s.ovn.org/v1
kind: AdminPolicyBasedExternalRoute
metadata:
  name: multi-hop-policy
spec:
  from:
    namespaceSelector:
      matchLabels:
        trafficType: "egress"
  nextHops:
    static:
    - ip: "172.18.0.8"
    - ip: "172.18.0.9"
    dynamic:
    - podSelector:
        matchLabels:
          gatewayPod: ""
      namespaceSelector:
        matchLabels:
          egressTraffic: ""
      networkAttachmentName: gigabyte
# ...
```

## Configure a secondary external gateway {#nw-secondary-ext-gw-configure_configuring-secondary-external-gateway}

You can configure an external gateway on the default network for a namespace in your cluster.

**Prerequisites**

- You installed the OpenShift CLI (`oc`).
- You are logged in to the cluster with a user with `cluster-admin` privileges.

**Procedure**

1. Create a YAML file that contains an `AdminPolicyBasedExternalRoute` object. For more information, see "AdminPolicyBasedExternalRoute object configuration".
2. To create an admin policy based external route, enter the following command:
   ```terminal
   $ oc create -f <file>.yaml
   ```

   - `<file>`: Specifies the name of the YAML file that you created in a previous step.
     ```text title="Example output"
     adminpolicybasedexternalroute.k8s.ovn.org/default-route-policy created
     ```
3. To confirm that the admin policy based external route was created, enter the following command:
   ```terminal
   $ oc describe apbexternalroute <name> | tail -n 6
   ```

   - `<name>`: Specifies the name of the `AdminPolicyBasedExternalRoute` object.
     ```text title="Example output"
     Status:
       Last Transition Time:  2023-04-24T15:09:01Z
       Messages:
       Configured external gateway IPs: 172.18.0.8
       Status:  Success
     Events:  <none>
     ```

**Additional resources**

- [Understanding multiple networks](/docs/networking/multiple_networks/understanding-multiple-networks#understanding-multiple-networks)
