---
title: "BaselineAdminNetworkPolicy [policy.networking.k8s.io/v1alpha1]"
sidebar_position: 6
---

# BaselineAdminNetworkPolicy [policy.networking.k8s.io/v1alpha1] {#baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1}

<a id="baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1"></a>

Description
: BaselineAdminNetworkPolicy is a cluster level resource that is part of the AdminNetworkPolicy API.

Type
: ```
  `object`
  ```

Required
: - `metadata`
  - `spec`

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | Standard object’s metadata. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata |
| `spec` | `object` | Specification of the desired behavior of BaselineAdminNetworkPolicy. |
| `status` | `object` | Status is the status to be reported by the implementation. |

### .spec {#_spec}

Description
: Specification of the desired behavior of BaselineAdminNetworkPolicy.

Type
: ```
  `object`
  ```

Required
: - `subject`

| Property | Type | Description |
| --- | --- | --- |
| `egress` | `array` | Egress is the list of Egress rules to be applied to the selected pods if they are not matched by any AdminNetworkPolicy or NetworkPolicy rules. A total of 100 Egress rules will be allowed in each BANP instance. The relative precedence of egress rules within a single BANP object will be determined by the order in which the rule is written. Thus, a rule that appears at the top of the egress rules would take the highest precedence. BANPs with no egress rules do not affect egress traffic.<br>Support: Core |
| `egress[]` | `object` | BaselineAdminNetworkPolicyEgressRule describes an action to take on a particular set of traffic originating from pods selected by a BaselineAdminNetworkPolicy’s Subject field. <network-policy-api:experimental:validation> |
| `ingress` | `array` | Ingress is the list of Ingress rules to be applied to the selected pods if they are not matched by any AdminNetworkPolicy or NetworkPolicy rules. A total of 100 Ingress rules will be allowed in each BANP instance. The relative precedence of ingress rules within a single BANP object will be determined by the order in which the rule is written. Thus, a rule that appears at the top of the ingress rules would take the highest precedence. BANPs with no ingress rules do not affect ingress traffic.<br>Support: Core |
| `ingress[]` | `object` | BaselineAdminNetworkPolicyIngressRule describes an action to take on a particular set of traffic destined for pods selected by a BaselineAdminNetworkPolicy’s Subject field. |
| `subject` | `object` | Subject defines the pods to which this BaselineAdminNetworkPolicy applies. Note that host-networked pods are not included in subject selection.<br>Support: Core |

### .spec.egress {#_specegress}

Description
: Egress is the list of Egress rules to be applied to the selected pods if they are not matched by any AdminNetworkPolicy or NetworkPolicy rules. A total of 100 Egress rules will be allowed in each BANP instance. The relative precedence of egress rules within a single BANP object will be determined by the order in which the rule is written. Thus, a rule that appears at the top of the egress rules would take the highest precedence. BANPs with no egress rules do not affect egress traffic.

  Support: Core

Type
: ```
  `array`
  ```

### .spec.egress[] {#_specegress}

Description
: BaselineAdminNetworkPolicyEgressRule describes an action to take on a particular set of traffic originating from pods selected by a BaselineAdminNetworkPolicy’s Subject field. <network-policy-api:experimental:validation>

Type
: ```
  `object`
  ```

Required
: - `action`
  - `to`

| Property | Type | Description |
| --- | --- | --- |
| `action` | `string` | Action specifies the effect this rule will have on matching traffic. Currently the following actions are supported: Allow: allows the selected traffic Deny: denies the selected traffic<br>Support: Core |
| `name` | `string` | Name is an identifier for this rule, that may be no more than 100 characters in length. This field should be used by the implementation to help improve observability, readability and error-reporting for any applied BaselineAdminNetworkPolicies.<br>Support: Core |
| `ports` | `array` | Ports allows for matching traffic based on port and protocols. This field is a list of destination ports for the outgoing egress traffic. If Ports is not set then the rule does not filter traffic via port. |
| `ports[]` | `object` | AdminNetworkPolicyPort describes how to select network ports on pod(s). Exactly one field must be set. |
| `to` | `array` | To is the list of destinations whose traffic this rule applies to. If any AdminNetworkPolicyEgressPeer matches the destination of outgoing traffic then the specified action is applied. This field must be defined and contain at least one item.<br>Support: Core |
| `to[]` | `object` | AdminNetworkPolicyEgressPeer defines a peer to allow traffic to. Exactly one of the selector pointers must be set for a given peer. If a consumer observes none of its fields are set, they must assume an unknown option has been specified and fail closed. |

### .spec.egress[].ports {#_specegressports}

Description
: Ports allows for matching traffic based on port and protocols. This field is a list of destination ports for the outgoing egress traffic. If Ports is not set then the rule does not filter traffic via port.

Type
: ```
  `array`
  ```

### .spec.egress[].ports[] {#_specegressports}

Description
: AdminNetworkPolicyPort describes how to select network ports on pod(s). Exactly one field must be set.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `namedPort` | `string` | NamedPort selects a port on a pod(s) based on name.<br>Support: Extended<br><network-policy-api:experimental> |
| `portNumber` | `object` | Port selects a port on a pod(s) based on number.<br>Support: Core |
| `portRange` | `object` | PortRange selects a port range on a pod(s) based on provided start and end values.<br>Support: Core |

### .spec.egress[].ports[].portNumber {#_specegressportsportnumber}

Description
: Port selects a port on a pod(s) based on number.

  Support: Core

Type
: ```
  `object`
  ```

Required
: - `port`
  - `protocol`

| Property | Type | Description |
| --- | --- | --- |
| `port` | `integer` | Number defines a network port value.<br>Support: Core |
| `protocol` | `string` | Protocol is the network protocol (TCP, UDP, or SCTP) which traffic must match. If not specified, this field defaults to TCP.<br>Support: Core |

### .spec.egress[].ports[].portRange {#_specegressportsportrange}

Description
: PortRange selects a port range on a pod(s) based on provided start and end values.

  Support: Core

Type
: ```
  `object`
  ```

Required
: - `end`
  - `start`

| Property | Type | Description |
| --- | --- | --- |
| `end` | `integer` | End defines a network port that is the end of a port range, the End value must be greater than Start.<br>Support: Core |
| `protocol` | `string` | Protocol is the network protocol (TCP, UDP, or SCTP) which traffic must match. If not specified, this field defaults to TCP.<br>Support: Core |
| `start` | `integer` | Start defines a network port that is the start of a port range, the Start value must be less than End.<br>Support: Core |

### .spec.egress[].to {#_specegressto}

Description
: To is the list of destinations whose traffic this rule applies to. If any AdminNetworkPolicyEgressPeer matches the destination of outgoing traffic then the specified action is applied. This field must be defined and contain at least one item.

  Support: Core

Type
: ```
  `array`
  ```

### .spec.egress[].to[] {#_specegressto}

Description
: AdminNetworkPolicyEgressPeer defines a peer to allow traffic to. Exactly one of the selector pointers must be set for a given peer. If a consumer observes none of its fields are set, they must assume an unknown option has been specified and fail closed.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `namespaces` | `object` | Namespaces defines a way to select all pods within a set of Namespaces. Note that host-networked pods are not included in this type of peer.<br>Support: Core |
| `networks` | `array (string)` | Networks defines a way to select peers via CIDR blocks. This is intended for representing entities that live outside the cluster, which can’t be selected by pods, namespaces and nodes peers, but note that cluster-internal traffic will be checked against the rule as well. So if you Allow or Deny traffic to `"0.0.0.0/0"`, that will allow or deny all IPv4 pod-to-pod traffic as well. If you don’t want that, add a rule that Passes all pod traffic before the Networks rule.<br>Each item in Networks should be provided in the CIDR format and should be IPv4 or IPv6, for example "10.0.0.0/8" or "fd00::/8".<br>Networks can have upto 25 CIDRs specified.<br>Support: Extended<br><network-policy-api:experimental> |
| `nodes` | `object` | Nodes defines a way to select a set of nodes in the cluster. This field follows standard label selector semantics; if present but empty, it selects all Nodes.<br>Support: Extended<br><network-policy-api:experimental> |
| `pods` | `object` | Pods defines a way to select a set of pods in a set of namespaces. Note that host-networked pods are not included in this type of peer.<br>Support: Core |

### .spec.egress[].to[].namespaces {#_specegresstonamespaces}

Description
: Namespaces defines a way to select all pods within a set of Namespaces. Note that host-networked pods are not included in this type of peer.

  Support: Core

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `matchExpressions` | `array` | matchExpressions is a list of label selector requirements. The requirements are ANDed. |
| `matchExpressions[]` | `object` | A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values. |
| `matchLabels` | `object (string)` | matchLabels is a map of {key,value} pairs. A single {key,value} in the matchLabels map is equivalent to an element of matchExpressions, whose key field is "key", the operator is "In", and the values array contains only "value". The requirements are ANDed. |

### .spec.egress[].to[].namespaces.matchExpressions {#_specegresstonamespacesmatchexpressions}

Description
: matchExpressions is a list of label selector requirements. The requirements are ANDed.

Type
: ```
  `array`
  ```

### .spec.egress[].to[].namespaces.matchExpressions[] {#_specegresstonamespacesmatchexpressions}

Description
: A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `operator`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key is the label key that the selector applies to. |
| `operator` | `string` | operator represents a key’s relationship to a set of values. Valid operators are In, NotIn, Exists and DoesNotExist. |
| `values` | `array (string)` | values is an array of string values. If the operator is In or NotIn, the values array must be non-empty. If the operator is Exists or DoesNotExist, the values array must be empty. This array is replaced during a strategic merge patch. |

### .spec.egress[].to[].nodes {#_specegresstonodes}

Description
: Nodes defines a way to select a set of nodes in the cluster. This field follows standard label selector semantics; if present but empty, it selects all Nodes.

  Support: Extended

  <network-policy-api:experimental>

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `matchExpressions` | `array` | matchExpressions is a list of label selector requirements. The requirements are ANDed. |
| `matchExpressions[]` | `object` | A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values. |
| `matchLabels` | `object (string)` | matchLabels is a map of {key,value} pairs. A single {key,value} in the matchLabels map is equivalent to an element of matchExpressions, whose key field is "key", the operator is "In", and the values array contains only "value". The requirements are ANDed. |

### .spec.egress[].to[].nodes.matchExpressions {#_specegresstonodesmatchexpressions}

Description
: matchExpressions is a list of label selector requirements. The requirements are ANDed.

Type
: ```
  `array`
  ```

### .spec.egress[].to[].nodes.matchExpressions[] {#_specegresstonodesmatchexpressions}

Description
: A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `operator`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key is the label key that the selector applies to. |
| `operator` | `string` | operator represents a key’s relationship to a set of values. Valid operators are In, NotIn, Exists and DoesNotExist. |
| `values` | `array (string)` | values is an array of string values. If the operator is In or NotIn, the values array must be non-empty. If the operator is Exists or DoesNotExist, the values array must be empty. This array is replaced during a strategic merge patch. |

### .spec.egress[].to[].pods {#_specegresstopods}

Description
: Pods defines a way to select a set of pods in a set of namespaces. Note that host-networked pods are not included in this type of peer.

  Support: Core

Type
: ```
  `object`
  ```

Required
: - `namespaceSelector`
  - `podSelector`

| Property | Type | Description |
| --- | --- | --- |
| `namespaceSelector` | `object` | NamespaceSelector follows standard label selector semantics; if empty, it selects all Namespaces. |
| `podSelector` | `object` | PodSelector is used to explicitly select pods within a namespace; if empty, it selects all Pods. |

### .spec.egress[].to[].pods.namespaceSelector {#_specegresstopodsnamespaceselector}

Description
: NamespaceSelector follows standard label selector semantics; if empty, it selects all Namespaces.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `matchExpressions` | `array` | matchExpressions is a list of label selector requirements. The requirements are ANDed. |
| `matchExpressions[]` | `object` | A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values. |
| `matchLabels` | `object (string)` | matchLabels is a map of {key,value} pairs. A single {key,value} in the matchLabels map is equivalent to an element of matchExpressions, whose key field is "key", the operator is "In", and the values array contains only "value". The requirements are ANDed. |

### .spec.egress[].to[].pods.namespaceSelector.matchExpressions {#_specegresstopodsnamespaceselectormatchexpressions}

Description
: matchExpressions is a list of label selector requirements. The requirements are ANDed.

Type
: ```
  `array`
  ```

### .spec.egress[].to[].pods.namespaceSelector.matchExpressions[] {#_specegresstopodsnamespaceselectormatchexpressions}

Description
: A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `operator`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key is the label key that the selector applies to. |
| `operator` | `string` | operator represents a key’s relationship to a set of values. Valid operators are In, NotIn, Exists and DoesNotExist. |
| `values` | `array (string)` | values is an array of string values. If the operator is In or NotIn, the values array must be non-empty. If the operator is Exists or DoesNotExist, the values array must be empty. This array is replaced during a strategic merge patch. |

### .spec.egress[].to[].pods.podSelector {#_specegresstopodspodselector}

Description
: PodSelector is used to explicitly select pods within a namespace; if empty, it selects all Pods.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `matchExpressions` | `array` | matchExpressions is a list of label selector requirements. The requirements are ANDed. |
| `matchExpressions[]` | `object` | A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values. |
| `matchLabels` | `object (string)` | matchLabels is a map of {key,value} pairs. A single {key,value} in the matchLabels map is equivalent to an element of matchExpressions, whose key field is "key", the operator is "In", and the values array contains only "value". The requirements are ANDed. |

### .spec.egress[].to[].pods.podSelector.matchExpressions {#_specegresstopodspodselectormatchexpressions}

Description
: matchExpressions is a list of label selector requirements. The requirements are ANDed.

Type
: ```
  `array`
  ```

### .spec.egress[].to[].pods.podSelector.matchExpressions[] {#_specegresstopodspodselectormatchexpressions}

Description
: A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `operator`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key is the label key that the selector applies to. |
| `operator` | `string` | operator represents a key’s relationship to a set of values. Valid operators are In, NotIn, Exists and DoesNotExist. |
| `values` | `array (string)` | values is an array of string values. If the operator is In or NotIn, the values array must be non-empty. If the operator is Exists or DoesNotExist, the values array must be empty. This array is replaced during a strategic merge patch. |

### .spec.ingress {#_specingress}

Description
: Ingress is the list of Ingress rules to be applied to the selected pods if they are not matched by any AdminNetworkPolicy or NetworkPolicy rules. A total of 100 Ingress rules will be allowed in each BANP instance. The relative precedence of ingress rules within a single BANP object will be determined by the order in which the rule is written. Thus, a rule that appears at the top of the ingress rules would take the highest precedence. BANPs with no ingress rules do not affect ingress traffic.

  Support: Core

Type
: ```
  `array`
  ```

### .spec.ingress[] {#_specingress}

Description
: BaselineAdminNetworkPolicyIngressRule describes an action to take on a particular set of traffic destined for pods selected by a BaselineAdminNetworkPolicy’s Subject field.

Type
: ```
  `object`
  ```

Required
: - `action`
  - `from`

| Property | Type | Description |
| --- | --- | --- |
| `action` | `string` | Action specifies the effect this rule will have on matching traffic. Currently the following actions are supported: Allow: allows the selected traffic Deny: denies the selected traffic<br>Support: Core |
| `from` | `array` | From is the list of sources whose traffic this rule applies to. If any AdminNetworkPolicyIngressPeer matches the source of incoming traffic then the specified action is applied. This field must be defined and contain at least one item.<br>Support: Core |
| `from[]` | `object` | AdminNetworkPolicyIngressPeer defines an in-cluster peer to allow traffic from. Exactly one of the selector pointers must be set for a given peer. If a consumer observes none of its fields are set, they must assume an unknown option has been specified and fail closed. |
| `name` | `string` | Name is an identifier for this rule, that may be no more than 100 characters in length. This field should be used by the implementation to help improve observability, readability and error-reporting for any applied BaselineAdminNetworkPolicies.<br>Support: Core |
| `ports` | `array` | Ports allows for matching traffic based on port and protocols. This field is a list of ports which should be matched on the pods selected for this policy i.e the subject of the policy. So it matches on the destination port for the ingress traffic. If Ports is not set then the rule does not filter traffic via port.<br>Support: Core |
| `ports[]` | `object` | AdminNetworkPolicyPort describes how to select network ports on pod(s). Exactly one field must be set. |

### .spec.ingress[].from {#_specingressfrom}

Description
: From is the list of sources whose traffic this rule applies to. If any AdminNetworkPolicyIngressPeer matches the source of incoming traffic then the specified action is applied. This field must be defined and contain at least one item.

  Support: Core

Type
: ```
  `array`
  ```

### .spec.ingress[].from[] {#_specingressfrom}

Description
: AdminNetworkPolicyIngressPeer defines an in-cluster peer to allow traffic from. Exactly one of the selector pointers must be set for a given peer. If a consumer observes none of its fields are set, they must assume an unknown option has been specified and fail closed.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `namespaces` | `object` | Namespaces defines a way to select all pods within a set of Namespaces. Note that host-networked pods are not included in this type of peer.<br>Support: Core |
| `pods` | `object` | Pods defines a way to select a set of pods in a set of namespaces. Note that host-networked pods are not included in this type of peer.<br>Support: Core |

### .spec.ingress[].from[].namespaces {#_specingressfromnamespaces}

Description
: Namespaces defines a way to select all pods within a set of Namespaces. Note that host-networked pods are not included in this type of peer.

  Support: Core

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `matchExpressions` | `array` | matchExpressions is a list of label selector requirements. The requirements are ANDed. |
| `matchExpressions[]` | `object` | A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values. |
| `matchLabels` | `object (string)` | matchLabels is a map of {key,value} pairs. A single {key,value} in the matchLabels map is equivalent to an element of matchExpressions, whose key field is "key", the operator is "In", and the values array contains only "value". The requirements are ANDed. |

### .spec.ingress[].from[].namespaces.matchExpressions {#_specingressfromnamespacesmatchexpressions}

Description
: matchExpressions is a list of label selector requirements. The requirements are ANDed.

Type
: ```
  `array`
  ```

### .spec.ingress[].from[].namespaces.matchExpressions[] {#_specingressfromnamespacesmatchexpressions}

Description
: A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `operator`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key is the label key that the selector applies to. |
| `operator` | `string` | operator represents a key’s relationship to a set of values. Valid operators are In, NotIn, Exists and DoesNotExist. |
| `values` | `array (string)` | values is an array of string values. If the operator is In or NotIn, the values array must be non-empty. If the operator is Exists or DoesNotExist, the values array must be empty. This array is replaced during a strategic merge patch. |

### .spec.ingress[].from[].pods {#_specingressfrompods}

Description
: Pods defines a way to select a set of pods in a set of namespaces. Note that host-networked pods are not included in this type of peer.

  Support: Core

Type
: ```
  `object`
  ```

Required
: - `namespaceSelector`
  - `podSelector`

| Property | Type | Description |
| --- | --- | --- |
| `namespaceSelector` | `object` | NamespaceSelector follows standard label selector semantics; if empty, it selects all Namespaces. |
| `podSelector` | `object` | PodSelector is used to explicitly select pods within a namespace; if empty, it selects all Pods. |

### .spec.ingress[].from[].pods.namespaceSelector {#_specingressfrompodsnamespaceselector}

Description
: NamespaceSelector follows standard label selector semantics; if empty, it selects all Namespaces.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `matchExpressions` | `array` | matchExpressions is a list of label selector requirements. The requirements are ANDed. |
| `matchExpressions[]` | `object` | A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values. |
| `matchLabels` | `object (string)` | matchLabels is a map of {key,value} pairs. A single {key,value} in the matchLabels map is equivalent to an element of matchExpressions, whose key field is "key", the operator is "In", and the values array contains only "value". The requirements are ANDed. |

### .spec.ingress[].from[].pods.namespaceSelector.matchExpressions {#_specingressfrompodsnamespaceselectormatchexpressions}

Description
: matchExpressions is a list of label selector requirements. The requirements are ANDed.

Type
: ```
  `array`
  ```

### .spec.ingress[].from[].pods.namespaceSelector.matchExpressions[] {#_specingressfrompodsnamespaceselectormatchexpressions}

Description
: A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `operator`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key is the label key that the selector applies to. |
| `operator` | `string` | operator represents a key’s relationship to a set of values. Valid operators are In, NotIn, Exists and DoesNotExist. |
| `values` | `array (string)` | values is an array of string values. If the operator is In or NotIn, the values array must be non-empty. If the operator is Exists or DoesNotExist, the values array must be empty. This array is replaced during a strategic merge patch. |

### .spec.ingress[].from[].pods.podSelector {#_specingressfrompodspodselector}

Description
: PodSelector is used to explicitly select pods within a namespace; if empty, it selects all Pods.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `matchExpressions` | `array` | matchExpressions is a list of label selector requirements. The requirements are ANDed. |
| `matchExpressions[]` | `object` | A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values. |
| `matchLabels` | `object (string)` | matchLabels is a map of {key,value} pairs. A single {key,value} in the matchLabels map is equivalent to an element of matchExpressions, whose key field is "key", the operator is "In", and the values array contains only "value". The requirements are ANDed. |

### .spec.ingress[].from[].pods.podSelector.matchExpressions {#_specingressfrompodspodselectormatchexpressions}

Description
: matchExpressions is a list of label selector requirements. The requirements are ANDed.

Type
: ```
  `array`
  ```

### .spec.ingress[].from[].pods.podSelector.matchExpressions[] {#_specingressfrompodspodselectormatchexpressions}

Description
: A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `operator`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key is the label key that the selector applies to. |
| `operator` | `string` | operator represents a key’s relationship to a set of values. Valid operators are In, NotIn, Exists and DoesNotExist. |
| `values` | `array (string)` | values is an array of string values. If the operator is In or NotIn, the values array must be non-empty. If the operator is Exists or DoesNotExist, the values array must be empty. This array is replaced during a strategic merge patch. |

### .spec.ingress[].ports {#_specingressports}

Description
: Ports allows for matching traffic based on port and protocols. This field is a list of ports which should be matched on the pods selected for this policy i.e the subject of the policy. So it matches on the destination port for the ingress traffic. If Ports is not set then the rule does not filter traffic via port.

  Support: Core

Type
: ```
  `array`
  ```

### .spec.ingress[].ports[] {#_specingressports}

Description
: AdminNetworkPolicyPort describes how to select network ports on pod(s). Exactly one field must be set.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `namedPort` | `string` | NamedPort selects a port on a pod(s) based on name.<br>Support: Extended<br><network-policy-api:experimental> |
| `portNumber` | `object` | Port selects a port on a pod(s) based on number.<br>Support: Core |
| `portRange` | `object` | PortRange selects a port range on a pod(s) based on provided start and end values.<br>Support: Core |

### .spec.ingress[].ports[].portNumber {#_specingressportsportnumber}

Description
: Port selects a port on a pod(s) based on number.

  Support: Core

Type
: ```
  `object`
  ```

Required
: - `port`
  - `protocol`

| Property | Type | Description |
| --- | --- | --- |
| `port` | `integer` | Number defines a network port value.<br>Support: Core |
| `protocol` | `string` | Protocol is the network protocol (TCP, UDP, or SCTP) which traffic must match. If not specified, this field defaults to TCP.<br>Support: Core |

### .spec.ingress[].ports[].portRange {#_specingressportsportrange}

Description
: PortRange selects a port range on a pod(s) based on provided start and end values.

  Support: Core

Type
: ```
  `object`
  ```

Required
: - `end`
  - `start`

| Property | Type | Description |
| --- | --- | --- |
| `end` | `integer` | End defines a network port that is the end of a port range, the End value must be greater than Start.<br>Support: Core |
| `protocol` | `string` | Protocol is the network protocol (TCP, UDP, or SCTP) which traffic must match. If not specified, this field defaults to TCP.<br>Support: Core |
| `start` | `integer` | Start defines a network port that is the start of a port range, the Start value must be less than End.<br>Support: Core |

### .spec.subject {#_specsubject}

Description
: Subject defines the pods to which this BaselineAdminNetworkPolicy applies. Note that host-networked pods are not included in subject selection.

  Support: Core

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `namespaces` | `object` | Namespaces is used to select pods via namespace selectors. |
| `pods` | `object` | Pods is used to select pods via namespace AND pod selectors. |

### .spec.subject.namespaces {#_specsubjectnamespaces}

Description
: Namespaces is used to select pods via namespace selectors.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `matchExpressions` | `array` | matchExpressions is a list of label selector requirements. The requirements are ANDed. |
| `matchExpressions[]` | `object` | A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values. |
| `matchLabels` | `object (string)` | matchLabels is a map of {key,value} pairs. A single {key,value} in the matchLabels map is equivalent to an element of matchExpressions, whose key field is "key", the operator is "In", and the values array contains only "value". The requirements are ANDed. |

### .spec.subject.namespaces.matchExpressions {#_specsubjectnamespacesmatchexpressions}

Description
: matchExpressions is a list of label selector requirements. The requirements are ANDed.

Type
: ```
  `array`
  ```

### .spec.subject.namespaces.matchExpressions[] {#_specsubjectnamespacesmatchexpressions}

Description
: A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `operator`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key is the label key that the selector applies to. |
| `operator` | `string` | operator represents a key’s relationship to a set of values. Valid operators are In, NotIn, Exists and DoesNotExist. |
| `values` | `array (string)` | values is an array of string values. If the operator is In or NotIn, the values array must be non-empty. If the operator is Exists or DoesNotExist, the values array must be empty. This array is replaced during a strategic merge patch. |

### .spec.subject.pods {#_specsubjectpods}

Description
: Pods is used to select pods via namespace AND pod selectors.

Type
: ```
  `object`
  ```

Required
: - `namespaceSelector`
  - `podSelector`

| Property | Type | Description |
| --- | --- | --- |
| `namespaceSelector` | `object` | NamespaceSelector follows standard label selector semantics; if empty, it selects all Namespaces. |
| `podSelector` | `object` | PodSelector is used to explicitly select pods within a namespace; if empty, it selects all Pods. |

### .spec.subject.pods.namespaceSelector {#_specsubjectpodsnamespaceselector}

Description
: NamespaceSelector follows standard label selector semantics; if empty, it selects all Namespaces.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `matchExpressions` | `array` | matchExpressions is a list of label selector requirements. The requirements are ANDed. |
| `matchExpressions[]` | `object` | A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values. |
| `matchLabels` | `object (string)` | matchLabels is a map of {key,value} pairs. A single {key,value} in the matchLabels map is equivalent to an element of matchExpressions, whose key field is "key", the operator is "In", and the values array contains only "value". The requirements are ANDed. |

### .spec.subject.pods.namespaceSelector.matchExpressions {#_specsubjectpodsnamespaceselectormatchexpressions}

Description
: matchExpressions is a list of label selector requirements. The requirements are ANDed.

Type
: ```
  `array`
  ```

### .spec.subject.pods.namespaceSelector.matchExpressions[] {#_specsubjectpodsnamespaceselectormatchexpressions}

Description
: A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `operator`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key is the label key that the selector applies to. |
| `operator` | `string` | operator represents a key’s relationship to a set of values. Valid operators are In, NotIn, Exists and DoesNotExist. |
| `values` | `array (string)` | values is an array of string values. If the operator is In or NotIn, the values array must be non-empty. If the operator is Exists or DoesNotExist, the values array must be empty. This array is replaced during a strategic merge patch. |

### .spec.subject.pods.podSelector {#_specsubjectpodspodselector}

Description
: PodSelector is used to explicitly select pods within a namespace; if empty, it selects all Pods.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `matchExpressions` | `array` | matchExpressions is a list of label selector requirements. The requirements are ANDed. |
| `matchExpressions[]` | `object` | A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values. |
| `matchLabels` | `object (string)` | matchLabels is a map of {key,value} pairs. A single {key,value} in the matchLabels map is equivalent to an element of matchExpressions, whose key field is "key", the operator is "In", and the values array contains only "value". The requirements are ANDed. |

### .spec.subject.pods.podSelector.matchExpressions {#_specsubjectpodspodselectormatchexpressions}

Description
: matchExpressions is a list of label selector requirements. The requirements are ANDed.

Type
: ```
  `array`
  ```

### .spec.subject.pods.podSelector.matchExpressions[] {#_specsubjectpodspodselectormatchexpressions}

Description
: A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `operator`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key is the label key that the selector applies to. |
| `operator` | `string` | operator represents a key’s relationship to a set of values. Valid operators are In, NotIn, Exists and DoesNotExist. |
| `values` | `array (string)` | values is an array of string values. If the operator is In or NotIn, the values array must be non-empty. If the operator is Exists or DoesNotExist, the values array must be empty. This array is replaced during a strategic merge patch. |

### .status {#_status}

Description
: Status is the status to be reported by the implementation.

Type
: ```
  `object`
  ```

Required
: - `conditions`

| Property | Type | Description |
| --- | --- | --- |
| `conditions` | `array` |  |
| `conditions[]` | `object` | Condition contains details for one aspect of the current state of this API Resource. --- This struct is intended for direct use as an array at the field path .status.conditions.  For example,<br>	type FooStatus struct{ 	    // Represents the observations of a foo’s current state. 	    // Known .status.conditions.type are: "Available", "Progressing", and "Degraded" 	    // +patchMergeKey=type 	    // +patchStrategy=merge 	    // +listType=map 	    // +listMapKey=type 	    Conditions []metav1.Condition `json:"conditions,omitempty" patchStrategy:"merge" patchMergeKey:"type" protobuf:"bytes,1,rep,name=conditions"`<br>	    // other fields 	} |

### .status.conditions {#_statusconditions}

Description

Type
: ```
  `array`
  ```

### .status.conditions[] {#_statusconditions}

Description
: ## Condition contains details for one aspect of the current state of this API Resource.

  This struct is intended for direct use as an array at the field path .status.conditions.  For example,

  ```
    type FooStatus struct{
        // Represents the observations of a foo’s current state.
        // Known .status.conditions.type are: "Available", "Progressing", and "Degraded"
        // +patchMergeKey=type
        // +patchStrategy=merge
        // +listType=map
        // +listMapKey=type
        Conditions []metav1.Condition `json:"conditions,omitempty" patchStrategy:"merge" patchMergeKey:"type" protobuf:"bytes,1,rep,name=conditions"`

        // other fields
    }
  ```

Type
: ```
  `object`
  ```

Required
: - `lastTransitionTime`
  - `message`
  - `reason`
  - `status`
  - `type`

| Property | Type | Description |
| --- | --- | --- |
| `lastTransitionTime` | `string` | lastTransitionTime is the last time the condition transitioned from one status to another. This should be when the underlying condition changed.  If that is not known, then using the time when the API field changed is acceptable. |
| `message` | `string` | message is a human readable message indicating details about the transition. This may be an empty string. |
| `observedGeneration` | `integer` | observedGeneration represents the .metadata.generation that the condition was set based upon. For instance, if .metadata.generation is currently 12, but the .status.conditions[x].observedGeneration is 9, the condition is out of date with respect to the current state of the instance. |
| `reason` | `string` | reason contains a programmatic identifier indicating the reason for the condition’s last transition. Producers of specific condition types may define expected values and meanings for this field, and whether the values are considered a guaranteed API. The value should be a CamelCase string. This field may not be empty. |
| `status` | `string` | status of the condition, one of True, False, Unknown. |
| `type` | `string` | type of condition in CamelCase or in foo.example.com/CamelCase. --- Many .condition.type values are consistent across resources like Available, but because arbitrary conditions can be useful (see .node.status.conditions), the ability to deconflict is important. The regex it matches is (dns1123SubdomainFmt/)?(qualifiedNameFmt) |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/policy.networking.k8s.io/v1alpha1/baselineadminnetworkpolicies`
  - `DELETE`: delete collection of BaselineAdminNetworkPolicy
  - `GET`: list objects of kind BaselineAdminNetworkPolicy
  - `POST`: create a BaselineAdminNetworkPolicy
- `/apis/policy.networking.k8s.io/v1alpha1/baselineadminnetworkpolicies/{name}`
  - `DELETE`: delete a BaselineAdminNetworkPolicy
  - `GET`: read the specified BaselineAdminNetworkPolicy
  - `PATCH`: partially update the specified BaselineAdminNetworkPolicy
  - `PUT`: replace the specified BaselineAdminNetworkPolicy
- `/apis/policy.networking.k8s.io/v1alpha1/baselineadminnetworkpolicies/{name}/status`
  - `GET`: read status of the specified BaselineAdminNetworkPolicy
  - `PATCH`: partially update status of the specified BaselineAdminNetworkPolicy
  - `PUT`: replace status of the specified BaselineAdminNetworkPolicy

### /apis/policy.networking.k8s.io/v1alpha1/baselineadminnetworkpolicies {#_apispolicynetworkingk8siov1alpha1baselineadminnetworkpolicies}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of BaselineAdminNetworkPolicy
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Status`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-Status) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  list objects of kind BaselineAdminNetworkPolicy
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`BaselineAdminNetworkPolicyList`](/docs/rest_api/objects#io-k8s-networking-policy-v1alpha1-BaselineAdminNetworkPolicyList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a BaselineAdminNetworkPolicy
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`BaselineAdminNetworkPolicy`](/docs/rest_api/network_apis/baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1#baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`BaselineAdminNetworkPolicy`](/docs/rest_api/network_apis/baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1#baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1) schema |
| 201 - Created | [`BaselineAdminNetworkPolicy`](/docs/rest_api/network_apis/baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1#baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1) schema |
| 202 - Accepted | [`BaselineAdminNetworkPolicy`](/docs/rest_api/network_apis/baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1#baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1) schema |
| 401 - Unauthorized | Empty |

### /apis/policy.networking.k8s.io/v1alpha1/baselineadminnetworkpolicies/{name} {#_apispolicynetworkingk8siov1alpha1baselineadminnetworkpolicies_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the BaselineAdminNetworkPolicy |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete a BaselineAdminNetworkPolicy
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Status`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-Status) schema |
| 202 - Accepted | [`Status`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-Status) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read the specified BaselineAdminNetworkPolicy
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`BaselineAdminNetworkPolicy`](/docs/rest_api/network_apis/baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1#baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified BaselineAdminNetworkPolicy
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`BaselineAdminNetworkPolicy`](/docs/rest_api/network_apis/baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1#baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified BaselineAdminNetworkPolicy
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`BaselineAdminNetworkPolicy`](/docs/rest_api/network_apis/baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1#baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`BaselineAdminNetworkPolicy`](/docs/rest_api/network_apis/baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1#baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1) schema |
| 201 - Created | [`BaselineAdminNetworkPolicy`](/docs/rest_api/network_apis/baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1#baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1) schema |
| 401 - Unauthorized | Empty |

### /apis/policy.networking.k8s.io/v1alpha1/baselineadminnetworkpolicies/{name}/status {#_apispolicynetworkingk8siov1alpha1baselineadminnetworkpolicies_name_status}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the BaselineAdminNetworkPolicy |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read status of the specified BaselineAdminNetworkPolicy
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`BaselineAdminNetworkPolicy`](/docs/rest_api/network_apis/baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1#baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update status of the specified BaselineAdminNetworkPolicy
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`BaselineAdminNetworkPolicy`](/docs/rest_api/network_apis/baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1#baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace status of the specified BaselineAdminNetworkPolicy
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`BaselineAdminNetworkPolicy`](/docs/rest_api/network_apis/baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1#baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`BaselineAdminNetworkPolicy`](/docs/rest_api/network_apis/baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1#baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1) schema |
| 201 - Created | [`BaselineAdminNetworkPolicy`](/docs/rest_api/network_apis/baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1#baselineadminnetworkpolicy-policy-networking-k8s-io-v1alpha1) schema |
| 401 - Unauthorized | Empty |
