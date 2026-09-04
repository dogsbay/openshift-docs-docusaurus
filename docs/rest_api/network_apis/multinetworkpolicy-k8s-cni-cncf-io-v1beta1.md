---
title: "MultiNetworkPolicy [k8s.cni.cncf.io/v1beta1]"
sidebar_position: 24
---

# MultiNetworkPolicy [k8s.cni.cncf.io/v1beta1] {#multinetworkpolicy-k8s-cni-cncf-io-v1beta1}

<a id="multinetworkpolicy-k8s-cni-cncf-io-v1beta1"></a>

Description
: MultiNetworkPolicy is a CRD schema to provide NetworkPolicy mechanism for net-attach-def which is specified by the Network Plumbing Working Group. MultiNetworkPolicy is identical to Kubernetes NetworkPolicy, See: https://kubernetes.io/docs/concepts/services-networking/network-policies/ .

Type
: ```
  `object`
  ```

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | Standard object’s metadata. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata |
| `spec` | `object` | Specification of the desired behavior for this MultiNetworkPolicy. |

### .spec {#_spec}

Description
: Specification of the desired behavior for this MultiNetworkPolicy.

Type
: ```
  `object`
  ```

Required
: - `podSelector`

| Property | Type | Description |
| --- | --- | --- |
| `egress` | `array` | List of egress rules to be applied to the selected pods. Outgoing traffic is allowed if there are no NetworkPolicies selecting the pod (and cluster policy otherwise allows the traffic), OR if the traffic matches at least one egress rule across all of the NetworkPolicy objects whose podSelector matches the pod. If this field is empty then this NetworkPolicy limits all outgoing traffic (and serves solely to ensure that the pods it selects are isolated by default). This field is beta-level in 1.8 |
| `egress[]` | `object` | NetworkPolicyEgressRule describes a particular set of traffic that is allowed out of pods matched by a NetworkPolicySpec’s podSelector. The traffic must match both ports and to. This type is beta-level in 1.8 |
| `ingress` | `array` | List of ingress rules to be applied to the selected pods. Traffic is allowed to a pod if there are no NetworkPolicies selecting the pod (and cluster policy otherwise allows the traffic), OR if the traffic source is the pod’s local node, OR if the traffic matches at least one ingress rule across all of the NetworkPolicy objects whose podSelector matches the pod. If this field is empty then this NetworkPolicy does not allow any traffic (and serves solely to ensure that the pods it selects are isolated by default) |
| `ingress[]` | `object` | NetworkPolicyIngressRule describes a particular set of traffic that is allowed to the pods matched by a NetworkPolicySpec’s podSelector. The traffic must match both ports and from. |
| `podSelector` | `object` | This is a label selector which selects Pods. This field follows standard label selector semantics; if present but empty, it selects all pods.  If NamespaceSelector is also set, then the NetworkPolicyPeer as a whole selects the Pods matching PodSelector in the Namespaces selected by NamespaceSelector. Otherwise it selects the Pods matching PodSelector in the policy’s own Namespace. |
| `policyTypes` | `array (string)` | List of rule types that the NetworkPolicy relates to. Valid options are 'Ingress', 'Egress', or 'Ingress,Egress'. If this field is not specified, it will default based on the existence of Ingress or Egress rules; policies that contain an Egress section are assumed to affect Egress, and all policies (whether or not they contain an Ingress section) are assumed to affect Ingress. If you want to write an egress-only policy, you must explicitly specify policyTypes [ 'Egress' ]. Likewise, if you want to write a policy that specifies that no egress is allowed, you must specify a policyTypes value that include 'Egress' (since such a policy would not include an Egress section and would otherwise default to just [ 'Ingress' ]). This field is beta-level in 1.8 |

### .spec.egress {#_specegress}

Description
: List of egress rules to be applied to the selected pods. Outgoing traffic is allowed if there are no NetworkPolicies selecting the pod (and cluster policy otherwise allows the traffic), OR if the traffic matches at least one egress rule across all of the NetworkPolicy objects whose podSelector matches the pod. If this field is empty then this NetworkPolicy limits all outgoing traffic (and serves solely to ensure that the pods it selects are isolated by default). This field is beta-level in 1.8

Type
: ```
  `array`
  ```

### .spec.egress[] {#_specegress}

Description
: NetworkPolicyEgressRule describes a particular set of traffic that is allowed out of pods matched by a NetworkPolicySpec’s podSelector. The traffic must match both ports and to. This type is beta-level in 1.8

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ports` | `array` | List of destination ports for outgoing traffic. Each item in this list is combined using a logical OR. If this field is empty or missing, this rule matches all ports (traffic not restricted by port). If this field is present and contains at least one item, then this rule allows traffic only if the traffic matches at least one port in the list. |
| `ports[]` | `object` | NetworkPolicyPort describes a port to allow traffic on |
| `to` | `array` | List of destinations for outgoing traffic of pods selected for this rule. Items in this list are combined using a logical OR operation. If this field is empty or missing, this rule matches all destinations (traffic not restricted by destination). If this field is present and contains at least one item, this rule allows traffic only if the traffic matches at least one item in the to list. |
| `to[]` | `object` | NetworkPolicyPeer describes a peer to allow traffic from. Only certain combinations of fields are allowed |

### .spec.egress[].ports {#_specegressports}

Description
: List of destination ports for outgoing traffic. Each item in this list is combined using a logical OR. If this field is empty or missing, this rule matches all ports (traffic not restricted by port). If this field is present and contains at least one item, then this rule allows traffic only if the traffic matches at least one port in the list.

Type
: ```
  `array`
  ```

### .spec.egress[].ports[] {#_specegressports}

Description
: NetworkPolicyPort describes a port to allow traffic on

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `endPort` | `integer` | If set, indicates that the range of ports from port to endPort, inclusive, should be allowed by the policy. This field cannot be defined if the port field is not defined or if the port field is defined as a named (string) port. The endPort must be equal or greater than port. |
| `port` | `integer-or-string` | The port on the given protocol. This can either be a numerical or named port on a pod. If this field is not provided, this matches all port names and numbers. |
| `protocol` | `string` | The protocol (TCP, UDP, or SCTP) which traffic must match. If not specified, this field defaults to TCP. |

### .spec.egress[].to {#_specegressto}

Description
: List of destinations for outgoing traffic of pods selected for this rule. Items in this list are combined using a logical OR operation. If this field is empty or missing, this rule matches all destinations (traffic not restricted by destination). If this field is present and contains at least one item, this rule allows traffic only if the traffic matches at least one item in the to list.

Type
: ```
  `array`
  ```

### .spec.egress[].to[] {#_specegressto}

Description
: NetworkPolicyPeer describes a peer to allow traffic from. Only certain combinations of fields are allowed

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ipBlock` | `object` | IPBlock defines policy on a particular IPBlock. If this field is set then neither of the other fields can be. |
| `namespaceSelector` | `object` | Selects Namespaces using cluster-scoped labels. This field follows standard label selector semantics; if present but empty, it selects all namespaces.  If PodSelector is also set, then the NetworkPolicyPeer as a whole selects the Pods matching PodSelector in the Namespaces selected by NamespaceSelector. Otherwise it selects all Pods in the Namespaces selected by NamespaceSelector. |
| `podSelector` | `object` | This is a label selector which selects Pods. This field follows standard label selector semantics; if present but empty, it selects all pods.  If NamespaceSelector is also set, then the NetworkPolicyPeer as a whole selects the Pods matching PodSelector in the Namespaces selected by NamespaceSelector. Otherwise it selects the Pods matching PodSelector in the policy’s own Namespace. |

### .spec.egress[].to[].ipBlock {#_specegresstoipblock}

Description
: IPBlock defines policy on a particular IPBlock. If this field is set then neither of the other fields can be.

Type
: ```
  `object`
  ```

Required
: - `cidr`

| Property | Type | Description |
| --- | --- | --- |
| `cidr` | `string` | CIDR is a string representing the IP Block Valid examples are '192.168.1.1/24' |
| `except` | `array (string)` | Except is a slice of CIDRs that should not be included within an IP Block Valid examples are '192.168.1.1/24' Except values will be rejected if they are outside the CIDR range |

### .spec.egress[].to[].namespaceSelector {#_specegresstonamespaceselector}

Description
: Selects Namespaces using cluster-scoped labels. This field follows standard label selector semantics; if present but empty, it selects all namespaces. If PodSelector is also set, then the NetworkPolicyPeer as a whole selects the Pods matching PodSelector in the Namespaces selected by NamespaceSelector. Otherwise it selects all Pods in the Namespaces selected by NamespaceSelector.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `matchExpressions` | `array` | matchExpressions is a list of label selector requirements. The requirements are ANDed. |
| `matchExpressions[]` | `object` | A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values. |
| `matchLabels` | `object (string)` | matchLabels is a map of {key,value} pairs. A single {key,value} in the matchLabels map is equivalent to an element of matchExpressions, whose key field is 'key', the operator is 'In', and the values array contains only 'value'. The requirements are ANDed. |

### .spec.egress[].to[].namespaceSelector.matchExpressions {#_specegresstonamespaceselectormatchexpressions}

Description
: matchExpressions is a list of label selector requirements. The requirements are ANDed.

Type
: ```
  `array`
  ```

### .spec.egress[].to[].namespaceSelector.matchExpressions[] {#_specegresstonamespaceselectormatchexpressions}

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

### .spec.egress[].to[].podSelector {#_specegresstopodselector}

Description
: This is a label selector which selects Pods. This field follows standard label selector semantics; if present but empty, it selects all pods. If NamespaceSelector is also set, then the NetworkPolicyPeer as a whole selects the Pods matching PodSelector in the Namespaces selected by NamespaceSelector. Otherwise it selects the Pods matching PodSelector in the policy’s own Namespace.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `matchExpressions` | `array` | matchExpressions is a list of label selector requirements. The requirements are ANDed. |
| `matchExpressions[]` | `object` | A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values. |
| `matchLabels` | `object (string)` | matchLabels is a map of {key,value} pairs. A single {key,value} in the matchLabels map is equivalent to an element of matchExpressions, whose key field is 'key', the operator is 'In', and the values array contains only 'value'. The requirements are ANDed. |

### .spec.egress[].to[].podSelector.matchExpressions {#_specegresstopodselectormatchexpressions}

Description
: matchExpressions is a list of label selector requirements. The requirements are ANDed.

Type
: ```
  `array`
  ```

### .spec.egress[].to[].podSelector.matchExpressions[] {#_specegresstopodselectormatchexpressions}

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
: List of ingress rules to be applied to the selected pods. Traffic is allowed to a pod if there are no NetworkPolicies selecting the pod (and cluster policy otherwise allows the traffic), OR if the traffic source is the pod’s local node, OR if the traffic matches at least one ingress rule across all of the NetworkPolicy objects whose podSelector matches the pod. If this field is empty then this NetworkPolicy does not allow any traffic (and serves solely to ensure that the pods it selects are isolated by default)

Type
: ```
  `array`
  ```

### .spec.ingress[] {#_specingress}

Description
: NetworkPolicyIngressRule describes a particular set of traffic that is allowed to the pods matched by a NetworkPolicySpec’s podSelector. The traffic must match both ports and from.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `from` | `array` | List of sources which should be able to access the pods selected for this rule. Items in this list are combined using a logical OR operation. If this field is empty or missing, this rule matches all sources (traffic not restricted by source). If this field is present and contains at least one item, this rule allows traffic only if the traffic matches at least one item in the from list. |
| `from[]` | `object` | NetworkPolicyPeer describes a peer to allow traffic from. Only certain combinations of fields are allowed |
| `ports` | `array` | List of ports which should be made accessible on the pods selected for this rule. Each item in this list is combined using a logical OR. If this field is empty or missing, this rule matches all ports (traffic not restricted by port). If this field is present and contains at least one item, then this rule allows traffic only if the traffic matches at least one port in the list. |
| `ports[]` | `object` | NetworkPolicyPort describes a port to allow traffic on |

### .spec.ingress[].from {#_specingressfrom}

Description
: List of sources which should be able to access the pods selected for this rule. Items in this list are combined using a logical OR operation. If this field is empty or missing, this rule matches all sources (traffic not restricted by source). If this field is present and contains at least one item, this rule allows traffic only if the traffic matches at least one item in the from list.

Type
: ```
  `array`
  ```

### .spec.ingress[].from[] {#_specingressfrom}

Description
: NetworkPolicyPeer describes a peer to allow traffic from. Only certain combinations of fields are allowed

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ipBlock` | `object` | IPBlock defines policy on a particular IPBlock. If this field is set then neither of the other fields can be. |
| `namespaceSelector` | `object` | Selects Namespaces using cluster-scoped labels. This field follows standard label selector semantics; if present but empty, it selects all namespaces.  If PodSelector is also set, then the NetworkPolicyPeer as a whole selects the Pods matching PodSelector in the Namespaces selected by NamespaceSelector. Otherwise it selects all Pods in the Namespaces selected by NamespaceSelector. |
| `podSelector` | `object` | This is a label selector which selects Pods. This field follows standard label selector semantics; if present but empty, it selects all pods.  If NamespaceSelector is also set, then the NetworkPolicyPeer as a whole selects the Pods matching PodSelector in the Namespaces selected by NamespaceSelector. Otherwise it selects the Pods matching PodSelector in the policy’s own Namespace. |

### .spec.ingress[].from[].ipBlock {#_specingressfromipblock}

Description
: IPBlock defines policy on a particular IPBlock. If this field is set then neither of the other fields can be.

Type
: ```
  `object`
  ```

Required
: - `cidr`

| Property | Type | Description |
| --- | --- | --- |
| `cidr` | `string` | CIDR is a string representing the IP Block Valid examples are '192.168.1.1/24' |
| `except` | `array (string)` | Except is a slice of CIDRs that should not be included within an IP Block Valid examples are '192.168.1.1/24' Except values will be rejected if they are outside the CIDR range |

### .spec.ingress[].from[].namespaceSelector {#_specingressfromnamespaceselector}

Description
: Selects Namespaces using cluster-scoped labels. This field follows standard label selector semantics; if present but empty, it selects all namespaces. If PodSelector is also set, then the NetworkPolicyPeer as a whole selects the Pods matching PodSelector in the Namespaces selected by NamespaceSelector. Otherwise it selects all Pods in the Namespaces selected by NamespaceSelector.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `matchExpressions` | `array` | matchExpressions is a list of label selector requirements. The requirements are ANDed. |
| `matchExpressions[]` | `object` | A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values. |
| `matchLabels` | `object (string)` | matchLabels is a map of {key,value} pairs. A single {key,value} in the matchLabels map is equivalent to an element of matchExpressions, whose key field is 'key', the operator is 'In', and the values array contains only 'value'. The requirements are ANDed. |

### .spec.ingress[].from[].namespaceSelector.matchExpressions {#_specingressfromnamespaceselectormatchexpressions}

Description
: matchExpressions is a list of label selector requirements. The requirements are ANDed.

Type
: ```
  `array`
  ```

### .spec.ingress[].from[].namespaceSelector.matchExpressions[] {#_specingressfromnamespaceselectormatchexpressions}

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

### .spec.ingress[].from[].podSelector {#_specingressfrompodselector}

Description
: This is a label selector which selects Pods. This field follows standard label selector semantics; if present but empty, it selects all pods. If NamespaceSelector is also set, then the NetworkPolicyPeer as a whole selects the Pods matching PodSelector in the Namespaces selected by NamespaceSelector. Otherwise it selects the Pods matching PodSelector in the policy’s own Namespace.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `matchExpressions` | `array` | matchExpressions is a list of label selector requirements. The requirements are ANDed. |
| `matchExpressions[]` | `object` | A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values. |
| `matchLabels` | `object (string)` | matchLabels is a map of {key,value} pairs. A single {key,value} in the matchLabels map is equivalent to an element of matchExpressions, whose key field is 'key', the operator is 'In', and the values array contains only 'value'. The requirements are ANDed. |

### .spec.ingress[].from[].podSelector.matchExpressions {#_specingressfrompodselectormatchexpressions}

Description
: matchExpressions is a list of label selector requirements. The requirements are ANDed.

Type
: ```
  `array`
  ```

### .spec.ingress[].from[].podSelector.matchExpressions[] {#_specingressfrompodselectormatchexpressions}

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
: List of ports which should be made accessible on the pods selected for this rule. Each item in this list is combined using a logical OR. If this field is empty or missing, this rule matches all ports (traffic not restricted by port). If this field is present and contains at least one item, then this rule allows traffic only if the traffic matches at least one port in the list.

Type
: ```
  `array`
  ```

### .spec.ingress[].ports[] {#_specingressports}

Description
: NetworkPolicyPort describes a port to allow traffic on

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `endPort` | `integer` | If set, indicates that the range of ports from port to endPort, inclusive, should be allowed by the policy. This field cannot be defined if the port field is not defined or if the port field is defined as a named (string) port. The endPort must be equal or greater than port. |
| `port` | `integer-or-string` | The port on the given protocol. This can either be a numerical or named port on a pod. If this field is not provided, this matches all port names and numbers. |
| `protocol` | `string` | The protocol (TCP, UDP, or SCTP) which traffic must match. If not specified, this field defaults to TCP. |

### .spec.podSelector {#_specpodselector}

Description
: This is a label selector which selects Pods. This field follows standard label selector semantics; if present but empty, it selects all pods. If NamespaceSelector is also set, then the NetworkPolicyPeer as a whole selects the Pods matching PodSelector in the Namespaces selected by NamespaceSelector. Otherwise it selects the Pods matching PodSelector in the policy’s own Namespace.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `matchExpressions` | `array` | matchExpressions is a list of label selector requirements. The requirements are ANDed. |
| `matchExpressions[]` | `object` | A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values. |
| `matchLabels` | `object (string)` |  |

### .spec.podSelector.matchExpressions {#_specpodselectormatchexpressions}

Description
: matchExpressions is a list of label selector requirements. The requirements are ANDed.

Type
: ```
  `array`
  ```

### .spec.podSelector.matchExpressions[] {#_specpodselectormatchexpressions}

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

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/k8s.cni.cncf.io/v1beta1/multi-networkpolicies`
  - `GET`: list objects of kind MultiNetworkPolicy
- `/apis/k8s.cni.cncf.io/v1beta1/namespaces/{namespace}/multi-networkpolicies`
  - `DELETE`: delete collection of MultiNetworkPolicy
  - `GET`: list objects of kind MultiNetworkPolicy
  - `POST`: create a MultiNetworkPolicy
- `/apis/k8s.cni.cncf.io/v1beta1/namespaces/{namespace}/multi-networkpolicies/{name}`
  - `DELETE`: delete a MultiNetworkPolicy
  - `GET`: read the specified MultiNetworkPolicy
  - `PATCH`: partially update the specified MultiNetworkPolicy
  - `PUT`: replace the specified MultiNetworkPolicy

### /apis/k8s.cni.cncf.io/v1beta1/multi-networkpolicies {#_apisk8scnicncfiov1beta1multi-networkpolicies}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  list objects of kind MultiNetworkPolicy
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`MultiNetworkPolicyList`](/docs/rest_api/objects#io-cncf-cni-k8s-v1beta1-MultiNetworkPolicyList) schema |
| 401 - Unauthorized | Empty |

### /apis/k8s.cni.cncf.io/v1beta1/namespaces/{namespace}/multi-networkpolicies {#_apisk8scnicncfiov1beta1namespaces_namespace_multi-networkpolicies}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of MultiNetworkPolicy
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
  list objects of kind MultiNetworkPolicy
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`MultiNetworkPolicyList`](/docs/rest_api/objects#io-cncf-cni-k8s-v1beta1-MultiNetworkPolicyList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a MultiNetworkPolicy
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`MultiNetworkPolicy`](/docs/rest_api/network_apis/multinetworkpolicy-k8s-cni-cncf-io-v1beta1#multinetworkpolicy-k8s-cni-cncf-io-v1beta1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`MultiNetworkPolicy`](/docs/rest_api/network_apis/multinetworkpolicy-k8s-cni-cncf-io-v1beta1#multinetworkpolicy-k8s-cni-cncf-io-v1beta1) schema |
| 201 - Created | [`MultiNetworkPolicy`](/docs/rest_api/network_apis/multinetworkpolicy-k8s-cni-cncf-io-v1beta1#multinetworkpolicy-k8s-cni-cncf-io-v1beta1) schema |
| 202 - Accepted | [`MultiNetworkPolicy`](/docs/rest_api/network_apis/multinetworkpolicy-k8s-cni-cncf-io-v1beta1#multinetworkpolicy-k8s-cni-cncf-io-v1beta1) schema |
| 401 - Unauthorized | Empty |

### /apis/k8s.cni.cncf.io/v1beta1/namespaces/{namespace}/multi-networkpolicies/{name} {#_apisk8scnicncfiov1beta1namespaces_namespace_multi-networkpolicies_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the MultiNetworkPolicy |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete a MultiNetworkPolicy
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
  read the specified MultiNetworkPolicy
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`MultiNetworkPolicy`](/docs/rest_api/network_apis/multinetworkpolicy-k8s-cni-cncf-io-v1beta1#multinetworkpolicy-k8s-cni-cncf-io-v1beta1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified MultiNetworkPolicy
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`MultiNetworkPolicy`](/docs/rest_api/network_apis/multinetworkpolicy-k8s-cni-cncf-io-v1beta1#multinetworkpolicy-k8s-cni-cncf-io-v1beta1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified MultiNetworkPolicy
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`MultiNetworkPolicy`](/docs/rest_api/network_apis/multinetworkpolicy-k8s-cni-cncf-io-v1beta1#multinetworkpolicy-k8s-cni-cncf-io-v1beta1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`MultiNetworkPolicy`](/docs/rest_api/network_apis/multinetworkpolicy-k8s-cni-cncf-io-v1beta1#multinetworkpolicy-k8s-cni-cncf-io-v1beta1) schema |
| 201 - Created | [`MultiNetworkPolicy`](/docs/rest_api/network_apis/multinetworkpolicy-k8s-cni-cncf-io-v1beta1#multinetworkpolicy-k8s-cni-cncf-io-v1beta1) schema |
| 401 - Unauthorized | Empty |
