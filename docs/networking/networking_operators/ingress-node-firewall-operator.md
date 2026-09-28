---
title: Ingress Node Firewall Operator in OpenShift Container Platform
sidebar_position: 9
---

# Ingress Node Firewall Operator in OpenShift Container Platform {#ingress-node-firewall-operator}

<a id="ingress-node-firewall-operator"></a>

The Ingress Node Firewall Operator provides a stateless, eBPF-based firewall for managing node-level ingress traffic in OpenShift Container Platform.

## Ingress Node Firewall Operator {#nw-infw-operator-cr_ingress-node-firewall-operator}

The Ingress Node Firewall Operator provides ingress firewall rules at a node level that you can specify and manage in the firewall configurations.

To deploy the daemon set created by the Operator, you create an `IngressNodeFirewallConfig` custom resource (CR). The Operator applies the `IngressNodeFirewallConfig` CR to create ingress node firewall daemon set `daemon`, which run on all nodes that match the `nodeSelector`.

You configure `rules` of the `IngressNodeFirewall` CR and apply them to clusters using the `nodeSelector` and setting values to "true".

:::warning

The Ingress Node Firewall Operator supports only stateless firewall rules.

Network interface controllers (NICs) that do not support native XDP drivers will run at a lower performance.

For OpenShift Container Platform 4.14 or later, you must run Ingress Node Firewall Operator on RHEL 9.0 or later.

:::

## Install the Ingress Node Firewall Operator {#installing-infw-operator_ingress-node-firewall-operator}

As a cluster administrator, you can install the Ingress Node Firewall Operator to enable node-level ingress firewalling by using the OpenShift Container Platform CLI.

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).
- You have an account with administrator privileges.

**Procedure**

1. To create the `openshift-ingress-node-firewall` namespace, enter the following command:
   ```terminal
   $ cat << EOF| oc create -f -
   apiVersion: v1
   kind: Namespace
   metadata:
     labels:
       pod-security.kubernetes.io/enforce: privileged
       pod-security.kubernetes.io/enforce-version: v1.24
     name: openshift-ingress-node-firewall
   EOF
   ```
2. To create an `OperatorGroup` CR, enter the following command:
   ```terminal
   $ cat << EOF| oc create -f -
   apiVersion: operators.coreos.com/v1
   kind: OperatorGroup
   metadata:
     name: ingress-node-firewall-operators
     namespace: openshift-ingress-node-firewall
   EOF
   ```
3. Subscribe to the Ingress Node Firewall Operator.
   - To create a `Subscription` CR for the Ingress Node Firewall Operator, enter the following command:
     ```terminal
     $ cat << EOF| oc create -f -
     apiVersion: operators.coreos.com/v1alpha1
     kind: Subscription
     metadata:
       name: ingress-node-firewall-sub
       namespace: openshift-ingress-node-firewall
     spec:
       name: ingress-node-firewall
       channel: stable
       source: redhat-operators
       sourceNamespace: openshift-marketplace
     EOF
     ```
4. To verify that the Operator is installed, enter the following command:
   ```terminal
   $ oc get ip -n openshift-ingress-node-firewall
   ```

   ```terminal title="Example output"
   NAME            CSV                                         APPROVAL    APPROVED
   install-5cvnz   ingress-node-firewall.4.22.0-202211122336   Automatic   true
   ```
5. To verify the version of the Operator, enter the following command:
   ```terminal
   $ oc get csv -n openshift-ingress-node-firewall
   ```

   ```terminal title="Example output"
   NAME                                        DISPLAY                          VERSION               REPLACES                                    PHASE
   ingress-node-firewall.4.22.0-202211122336   Ingress Node Firewall Operator   4.22.0-202211122336   ingress-node-firewall.4.22.0-202211102047   Succeeded
   ```

## Install the Ingress Node Firewall Operator using the web console {#install-operator-web-console_ingress-node-firewall-operator}

As a cluster administrator, you can install the Ingress Node Firewall Operator to enable node-level ingress firewalling by using the web console.

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).
- You have an account with administrator privileges.

**Procedure**

1. Install the Ingress Node Firewall Operator:
   1. In the OpenShift Container Platform web console, click **Ecosystem** → **Software Catalog**.
   2. Select **Ingress Node Firewall Operator** from the list of available Operators, and then click **Install**.
   3. On the **Install Operator** page, under **Installed Namespace**, select **Operator recommended Namespace**.
   4. Click **Install**.
2. Verify that the Ingress Node Firewall Operator is installed successfully:
   1. Navigate to the **Ecosystem** → **Installed Operators** page.
   2. Ensure that **Ingress Node Firewall Operator** is listed in the **openshift-ingress-node-firewall** project with a **Status** of **InstallSucceeded**.
      :::note

      During installation an Operator might display a **Failed** status. If the installation later succeeds with an **InstallSucceeded** message, you can ignore the **Failed** message.

      :::

      If the Operator does not have a **Status** of **InstallSucceeded**, troubleshoot using the following steps:

      - Inspect the **Operator Subscriptions** and **Install Plans** tabs for any failures or errors under **Status**.
      - Navigate to the **Workloads** → **Pods** page and check the logs for pods in the `openshift-ingress-node-firewall` project.
      - Check the namespace of the YAML file. If the annotation is missing, you can add the annotation `workload.openshift.io/allowed=management` to the Operator namespace with the following command:
        ```terminal
        $ oc annotate ns/openshift-ingress-node-firewall workload.openshift.io/allowed=management
        ```

        :::note

        For single-node OpenShift clusters, the `openshift-ingress-node-firewall` namespace requires the `workload.openshift.io/allowed=management` annotation.

        :::

## Deploy Ingress Node Firewall Operator {#nw-infw-operator-deploying_ingress-node-firewall-operator}

To deploy the Ingress Node Firewall Operator, create a `IngressNodeFirewallConfig` custom resource that will deploy the Operator’s daemon set. You can deploy one or multiple `IngressNodeFirewall` CRDs to nodes by applying firewall rules.

**Prerequisite**

- The Ingress Node Firewall Operator is installed.

**Procedure**

1. Create the `IngressNodeFirewallConfig` inside the `openshift-ingress-node-firewall` namespace named `ingressnodefirewallconfig`.
2. Run the following command to deploy Ingress Node Firewall Operator rules:
   ```terminal
   $ oc apply -f rule.yaml
   ```

## Ingress Node Firewall configuration object {#nw-infw-operator-config-object_ingress-node-firewall-operator}

Review configuration fields so you can define how the Operator deploys the firewall.

The fields for the Ingress Node Firewall configuration object are described in the following table:

**Ingress Node Firewall Configuration object**

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
  <td>The name of the CR object. The name of the firewall rules object must be <code>ingressnodefirewallconfig</code>.</td>
</tr>
<tr>
  <td><code>metadata.namespace</code></td>
  <td><code>string</code></td>
  <td>Namespace for the Ingress Firewall Operator CR object. The <code>IngressNodeFirewallConfig</code> CR must be created inside the <code>openshift-ingress-node-firewall</code> namespace.</td>
</tr>
<tr>
  <td><code>spec.nodeSelector</code></td>
  <td><code>string</code></td>
  <td>A node selection constraint used to target nodes through specified node labels. For example:<br><br><pre><code>apiVersion: ingressnodefirewall.openshift.io/v1alpha1&#10;kind: IngressNodeFirewallConfig&#10;metadata:&#10;  name: ingressnodefirewallconfig&#10;  namespace: openshift-ingress-node-firewall&#10;spec:&#10;  nodeSelector:&#10;    node-role.kubernetes.io/worker: ""</code></pre><br><br><div class="alert alert--secondary"><strong>Note</strong><p>One label used in <code>nodeSelector</code> must match a label on the nodes in order for the daemon set to start. For example, if the node labels <code>node-role.kubernetes.io/worker</code> and <code>node-type.kubernetes.io/vm</code> are applied to a node, then at least one label must be set using <code>nodeSelector</code> for the daemon set to start.</p></div></td>
</tr>
<tr>
  <td><code>spec.ebpfProgramManagerMode</code></td>
  <td><code>boolean</code></td>
  <td>Specifies if the Node Ingress Firewall Operator uses the eBPF Manager Operator or not to manage eBPF programs. This capability is a Technology Preview feature.<br><br>For more information about the support scope of Red Hat Technology Preview features, see <a href="https://access.redhat.com/support/offerings/techpreview/">Technology Preview Features Support Scope</a>.</td>
</tr>
</tbody>
</table>

:::note

The Operator consumes the CR and creates an ingress node firewall daemon set on all the nodes that match the `nodeSelector`.

:::

### Ingress Node Firewall Operator example configuration {#nw-ingress-node-firewall-example-cr-2_ingress-node-firewall-operator}

A complete Ingress Node Firewall Configuration is specified in the following example:

```yaml title="Example of how to create an Ingress Node Firewall Configuration object"
$ cat << EOF | oc create -f -
apiVersion: ingressnodefirewall.openshift.io/v1alpha1
kind: IngressNodeFirewallConfig
metadata:
  name: ingressnodefirewallconfig
  namespace: openshift-ingress-node-firewall
spec:
  nodeSelector:
    node-role.kubernetes.io/worker: ""
EOF
```

:::note

The Operator consumes the CR object and creates an ingress node firewall daemon set on all the nodes that match the `nodeSelector`.

:::

### Ingress Node Firewall rules object {#nw-ingress-node-firewall-operator-rules-object_ingress-node-firewall-operator}

You can review rule fields and examples to define which ingress traffic is allowed or denied by using the Ingress Node Firewall rules object.

The fields for the Ingress Node Firewall rules object are described in the following table:

**Ingress Node Firewall rules object**

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
  <td>The name of the CR object.</td>
</tr>
<tr>
  <td><code>interfaces</code></td>
  <td><code>array</code></td>
  <td>The fields for this object specify the interfaces to apply the firewall rules to. For example, <code>- en0</code> and <code>- en1</code>.</td>
</tr>
<tr>
  <td><code>nodeSelector</code></td>
  <td><code>array</code></td>
  <td>You can use <code>nodeSelector</code> to select the nodes to apply the firewall rules to. Set the value of your named <code>nodeselector</code> labels to <code>true</code> to apply the rule.</td>
</tr>
<tr>
  <td><code>ingress</code></td>
  <td><code>object</code></td>
  <td><code>ingress</code> allows you to configure the rules that allow outside access to the services on your cluster.</td>
</tr>
</tbody>
</table>

#### Ingress object configuration {#nw-infw-ingress-rules-object_ingress-node-firewall-operator}

The values for the `ingress` object are defined in the following table:

`ingress`** object**

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
  <td><code>sourceCIDRs</code></td>
  <td><code>array</code></td>
  <td>Allows you to set the CIDR block. You can configure multiple CIDRs from different address families.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>Different CIDRs allow you to use the same order rule. In the case that there are multiple <code>IngressNodeFirewall</code> objects for the same nodes and interfaces with overlapping CIDRs, the <code>order</code> field will specify which rule is applied first. Rules are applied in ascending order.</p></div></td>
</tr>
<tr>
  <td><code>rules</code></td>
  <td><code>array</code></td>
  <td>Ingress firewall <code>rules.order</code> objects are ordered starting at <code>1</code> for each <code>source.CIDR</code> with up to 100 rules per CIDR. Lower order rules are executed first.<br><br><code>rules.protocolConfig.protocol</code> supports the following protocols: TCP, UDP, SCTP, ICMP and ICMPv6. ICMP and ICMPv6 rules can match against ICMP and ICMPv6 types or codes. TCP, UDP, and SCTP rules can match against a single destination port or a range of ports using <code>&lt;start : end-1&gt;</code> format.<br><br>Set <code>rules.action</code> to <code>allow</code> to apply the rule or <code>deny</code> to disallow the rule.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>Ingress firewall rules are verified using a verification webhook that blocks any invalid configuration. The verification webhook prevents you from blocking any critical cluster services such as the API server.</p></div></td>
</tr>
</tbody>
</table>

#### Ingress Node Firewall rules object example {#nw-ingress-node-firewall-example-cr_ingress-node-firewall-operator}

A complete Ingress Node Firewall configuration is specified in the following example:

```yaml title="Example Ingress Node Firewall configuration"
apiVersion: ingressnodefirewall.openshift.io/v1alpha1
kind: IngressNodeFirewall
metadata:
  name: ingressnodefirewall
spec:
  interfaces:
  - eth0
  nodeSelector:
    matchLabels:
      <label_name>: <label_value>
  ingress:
  - sourceCIDRs:
       - 172.16.0.0/12
    rules:
    - order: 10
      protocolConfig:
        protocol: ICMP
        icmp:
          icmpType: 8 #ICMP Echo request
      action: Deny
    - order: 20
      protocolConfig:
        protocol: TCP
        tcp:
          ports: "8000-9000"
      action: Deny
  - sourceCIDRs:
       - fc00:f853:ccd:e793::0/64
    rules:
    - order: 10
      protocolConfig:
        protocol: ICMPv6
        icmpv6:
          icmpType: 128 #ICMPV6 Echo request
      action: Deny
```

A `<label_name>` and a `<label_value>` must exist on the node and must match the `nodeselector` label and value applied to the nodes you want the `ingressfirewallconfig` CR to run on. The `<label_value>` can be `true` or `false`. By using `nodeSelector` labels, you can target separate groups of nodes to apply different rules to using the `ingressfirewallconfig` CR.

#### Zero trust Ingress Node Firewall rules object example {#nw-ingress-node-firewall-zero-trust-example-cr_ingress-node-firewall-operator}

Zero trust Ingress Node Firewall rules can provide additional security to multi-interface clusters. For example, you can use zero trust Ingress Node Firewall rules to drop all traffic on a specific interface except for SSH.

A complete configuration of a zero trust Ingress Node Firewall rule for a network-interface cluster is specified in the following example:

:::warning

Users need to add all ports their application will use to their allowlist in the following case to ensure proper functionality.

:::

```yaml title="Example zero trust Ingress Node Firewall rules"
apiVersion: ingressnodefirewall.openshift.io/v1alpha1
kind: IngressNodeFirewall
metadata:
 name: ingressnodefirewall-zero-trust
spec:
 interfaces:
 - eth1
 nodeSelector:
   matchLabels:
     <ingress_firewall_label_name>: <label_value>
 ingress:
 - sourceCIDRs:
      - 0.0.0.0/0
   rules:
   - order: 10
     protocolConfig:
       protocol: TCP
       tcp:
         ports: 22
     action: Allow
   - order: 20
     action: Deny
```

:::warning

eBPF Manager Operator integration is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.

For more information about the support scope of Red Hat Technology Preview features, see [Technology Preview Features Support Scope](https://access.redhat.com/support/offerings/techpreview/).

:::

## Ingress Node Firewall Operator integration {#ingress-node-firewall-operator_ingress-node-firewall-operator}

Learn when to use eBPF Manager to load and manage Ingress Node Firewall programs.

The Ingress Node Firewall uses [eBPF](https://www.kernel.org/doc/html/latest/bpf/index.html) programs to implement some of its key firewall functionality. By default these eBPF programs are loaded into the kernel using a mechanism specific to the Ingress Node Firewall. You can configure the Ingress Node Firewall Operator to use the eBPF Manager Operator for loading and managing these programs instead.

When this integration is enabled, the following limitations apply:

- The Ingress Node Firewall Operator uses TCX if XDP is not available and TCX is incompatible with bpfman.
- The Ingress Node Firewall Operator daemon set pods remain in the `ContainerCreating` state until the firewall rules are applied.
- The Ingress Node Firewall Operator daemon set pods run as privileged.

## Configure Ingress Node Firewall Operator to use the eBPF Manager Operator {#bpfman-infw-configure_ingress-node-firewall-operator}

Configure the Ingress Node Firewall to use eBPF Manager for program lifecycle control.

The Ingress Node Firewall uses [eBPF](https://www.kernel.org/doc/html/latest/bpf/index.html) programs to implement some of its key firewall functionality. By default these eBPF programs are loaded into the kernel using a mechanism specific to the Ingress Node Firewall.

As a cluster administrator, you can configure the Ingress Node Firewall Operator to use the eBPF Manager Operator for loading and managing these programs instead, adding additional security and observability functionality.

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).
- You have an account with administrator privileges.
- You installed the Ingress Node Firewall Operator.
- You have installed the eBPF Manager Operator.

**Procedure**

1. Apply the following labels to the `ingress-node-firewall-system` namespace:
   ```terminal
   $ oc label namespace openshift-ingress-node-firewall \
       pod-security.kubernetes.io/enforce=privileged \
       pod-security.kubernetes.io/warn=privileged --overwrite
   ```
2. Edit the `IngressNodeFirewallConfig` object named `ingressnodefirewallconfig` and set the `ebpfProgramManagerMode` field:
   ```yaml title="Ingress Node Firewall Operator configuration object"
   apiVersion: ingressnodefirewall.openshift.io/v1alpha1
   kind: IngressNodeFirewallConfig
   metadata:
     name: ingressnodefirewallconfig
     namespace: openshift-ingress-node-firewall
   spec:
     nodeSelector:
       node-role.kubernetes.io/worker: ""
     ebpfProgramManagerMode: <ebpf_mode>
   ```

   where:

   `<ebpf_mode>`: Specifies whether or not the Ingress Node Firewall Operator uses the eBPF Manager Operator to manage eBPF programs. Must be either `true` or `false`. If unset, eBPF Manager is not used.

## View Ingress Node Firewall Operator rules {#nw-infw-operator-viewing_ingress-node-firewall-operator}

Inspect existing rules and configs to confirm the firewall is applied as intended.

**Procedure**

1. Run the following command to view all current rules :
   ```terminal
   $ oc get ingressnodefirewall
   ```
2. Choose one of the returned `<resource>` names and run the following command to view the rules or configs:
   ```terminal
   $ oc get <resource> <name> -o yaml
   ```

## Troubleshoot the Ingress Node Firewall Operator {#nw-infw-operator-troubleshooting_ingress-node-firewall-operator}

You can verify the status and view the logs to diagnose ingress firewall deployment or rule issues.

**Procedure**

- Run the following command to list installed Ingress Node Firewall custom resource definitions (CRD):
  ```terminal
  $ oc get crds | grep ingressnodefirewall
  ```

  ```terminal title="Example output"
  NAME               READY   UP-TO-DATE   AVAILABLE   AGE
  ingressnodefirewallconfigs.ingressnodefirewall.openshift.io       2022-08-25T10:03:01Z
  ingressnodefirewallnodestates.ingressnodefirewall.openshift.io    2022-08-25T10:03:00Z
  ingressnodefirewalls.ingressnodefirewall.openshift.io             2022-08-25T10:03:00Z
  ```
- Run the following command to view the state of the Ingress Node Firewall Operator:
  ```terminal
  $ oc get pods -n openshift-ingress-node-firewall
  ```

  ```terminal title="Example output"
  NAME                                       READY  STATUS         RESTARTS  AGE
  ingress-node-firewall-controller-manager   2/2    Running        0         5d21h
  ingress-node-firewall-daemon-pqx56         3/3    Running        0         5d21h
  ```

  The following fields provide information about the status of the Operator: `READY`, `STATUS`, `AGE`, and `RESTARTS`. The `STATUS` field is `Running` when the Ingress Node Firewall Operator is deploying a daemon set to the assigned nodes.
- Run the following command to collect all ingress firewall node pods' logs:
  ```terminal
  $ oc adm must-gather – gather_ingress_node_firewall
  ```

  The logs are available in the sos node’s report containing eBPF `bpftool` outputs at `/sos_commands/ebpf`. These reports include lookup tables used or updated as the ingress firewall XDP handles packet processing, updates statistics, and emits events.

**Additional resources**

- [About the eBPF Manager Operator](/docs/networking/networking_operators/ebpf_manager/ebpf-manager-operator-about#bpfman-operator-about)
