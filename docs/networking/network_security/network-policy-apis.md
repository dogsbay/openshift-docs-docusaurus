---
title: Understanding network policy APIs
sidebar_position: 1
---

# Understanding network policy APIs {#network-policy-apis}

<a id="network-policy-apis"></a>

Network policy is defined using both cluster-scoped and namespace-scoped network policy APIs. By defining network policy across these different levels, you can create sophisticated network security configurations for your clusters, including full multi-tenant isolation.

## Network policies and their scope {#nw-anp-np-scope}

Cluster-scoped network policy
: Cluster and network administrators can use the AdminNetworkPolicy to define network policy at the cluster level. The AdminNetworkPolicy feature consists of two APIs: the `AdminNetworkPolicy` API and `BaselineAdminNetworkPolicy` API. These APIs are used to set rules that can be applied to the entire cluster, or delegated to the namespace-scoped `NetworkPolicy`.

  Policies defined using the `AdminNetworkPolicy` API take precedence over all other policy types when set to "Allow" or "Deny". However, administrators can also use "Pass" to delegate responsibility for a given policy to the namespace-scoped `NetworkPolicy` to allow application developers and namespace tenants to control specific aspects of network security for their projects.

  Policies defined using the `BaselineAdminNetworkPolicy` API apply only when no other network policy overrides them. When you use the `AdminNetworkPolicy` API to delegate an aspect of network policy to the namespace-scoped `NetworkPolicy`, you should also define a sensible minimum restriction in the `BaselineAdminNetworkPolicy`. This ensures a baseline level of network security at the cluster level in case the `NetworkPolicy` for a namespace does not provide sufficient protection.

Namespace-scoped network policy
: Application developers and namespace tenants can use the `NetworkPolicy` API to define network policy rules for a specific namespace. Rules in the `NetworkPolicy` for a namespace take precedence over cluster-wide rules configured using the BaselineAdminNetworkPolicy API, or for a cluster-wide rule that has been delegated or "passed" from the cluster-wide `AdminNetworkPolicy` API.

## How network policy is evaluated and applied {#nw-anp-np-evaluation}

When a network connection is established, the network provider (default: OVN-Kubernetes) checks the connection details against network policy rules to determine how to handle the connection.

OVN-Kubernetes evaluates connections against network policy objects in the following order:

1. Check for matches in the AdminNetworkPolicy tier.
   1. If a connection matches an `Allow` or `Deny` rule, follow that rule and stop evaluating.
   2. If a connection matches a `Pass` rule, move to the NetworkPolicy tier.
2. Check for matches in the NetworkPolicy tier.
   1. If a connection matches a rule, follow that rule and stop evaluating.
   2. If no match is found, move to the BaselineAdminNetworkPolicy tier.
3. Follow a matching rule in the BaselineAdminNetworkPolicy tier.

<a name="img-ovn-kubernetes-network-policy-evaluation"></a>**Figure 1. Evaluation of network policies by OVN-Kubernetes**

![OVN-Kubernetes Access Control List (ACL)](/images/615_OpenShift_OVN-K_ACLs_0324.png)

## Key differences between AdminNetworkPolicy and NetworkPolicy custom resources {#nw-anp-differences-networkpolicy_network-policy-apis}

To choose the right network policy API for your cluster or namespace in OpenShift Container Platform, you can compare `AdminNetworkPolicy` and `NetworkPolicy` CRs across scope, rule actions, precedence, and peer options.

The following reference table highlights differences in who can apply policies, how traffic is allowed or denied, and how OVN-Kubernetes evaluates each API.

| Policy elements | AdminNetworkPolicy | NetworkPolicy |
| --- | --- | --- |
| Applicable user | Cluster administrator or equivalent | Namespace owners |
| Scope | Cluster | Namespace |
| Drop traffic | Supported with an explicit `Deny` action set as a rule. | Supported via implicit `Deny` isolation at policy creation time. |
| Delegate traffic | Supported with an `Pass` action set as a rule. | Not applicable. |
| Allow traffic | Supported with an explicit `Allow` action set as a rule. | The default action for all rules is to allow. |
| Rule precedence within the policy | Depends on the order in which they appear within an ANP. The higher the rule’s position the higher the precedence. | Rules are additive. |
| Policy precedence | Among ANPs the `priority` field sets the order for evaluation. The lower the priority number higher the policy precedence. | There is no policy ordering between policies. |
| Feature precedence | Evaluated first via tier 1 ACL and BANP is evaluated last via tier 3 ACL. | Enforced after ANP and before BANP, they are evaluated in tier 2 of the ACL. |
| Matching pod selection | Can apply different rules across namespaces. | Can apply different rules across pods in single namespace. |
| Cluster egress traffic | Supported via `nodes` and `networks` peers | Supported through `ipBlock` field along with accepted CIDR syntax. |
| Cluster ingress traffic | Not supported. | Not supported. |
| Fully qualified domain names (FQDN) peer support | Not supported. | Not supported. |
| Namespace selectors | Supports advanced selection of Namespaces with the use of `namespaces.matchLabels` field. | Supports label based namespace selection with the use of `namespaceSelector` field. |
