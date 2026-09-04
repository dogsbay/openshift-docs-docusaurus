---
title: "NetworkPolicy [networking.k8s.io/v1]"
sidebar_position: 26
---

# NetworkPolicy [networking.k8s.io/v1] {#networkpolicy-networking-k8s-io-v1}

<a id="networkpolicy-networking-k8s-io-v1"></a>

Description
: NetworkPolicy describes what network traffic is allowed for a set of Pods

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
| `spec` | `object` | NetworkPolicySpec provides the specification of a NetworkPolicy |

### .spec {#_spec}

Description
: NetworkPolicySpec provides the specification of a NetworkPolicy

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `egress` | `array` | egress is a list of egress rules to be applied to the selected pods. Outgoing traffic is allowed if there are no NetworkPolicies selecting the pod (and cluster policy otherwise allows the traffic), OR if the traffic matches at least one egress rule across all of the NetworkPolicy objects whose podSelector matches the pod. If this field is empty then this NetworkPolicy limits all outgoing traffic (and serves solely to ensure that the pods it selects are isolated by default). This field is beta-level in 1.8 |
| `egress[]` | `object` | NetworkPolicyEgressRule describes a particular set of traffic that is allowed out of pods matched by a NetworkPolicySpec’s podSelector. The traffic must match both ports and to. This type is beta-level in 1.8 |
| `ingress` | `array` | ingress is a list of ingress rules to be applied to the selected pods. Traffic is allowed to a pod if there are no NetworkPolicies selecting the pod (and cluster policy otherwise allows the traffic), OR if the traffic source is the pod’s local node, OR if the traffic matches at least one ingress rule across all of the NetworkPolicy objects whose podSelector matches the pod. If this field is empty then this NetworkPolicy does not allow any traffic (and serves solely to ensure that the pods it selects are isolated by default) |
| `ingress[]` | `object` | NetworkPolicyIngressRule describes a particular set of traffic that is allowed to the pods matched by a NetworkPolicySpec’s podSelector. The traffic must match both ports and from. |
| `podSelector` | [`LabelSelector`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-LabelSelector) | podSelector selects the pods to which this NetworkPolicy object applies. The array of rules is applied to any pods selected by this field. An empty selector matches all pods in the policy’s namespace. Multiple network policies can select the same set of pods. In this case, the ingress rules for each are combined additively. This field is optional. If it is not specified, it defaults to an empty selector. |
| `policyTypes` | `array (string)` | policyTypes is a list of rule types that the NetworkPolicy relates to. Valid options are ["Ingress"], ["Egress"], or ["Ingress", "Egress"]. If this field is not specified, it will default based on the existence of ingress or egress rules; policies that contain an egress section are assumed to affect egress, and all policies (whether or not they contain an ingress section) are assumed to affect ingress. If you want to write an egress-only policy, you must explicitly specify policyTypes [ "Egress" ]. Likewise, if you want to write a policy that specifies that no egress is allowed, you must specify a policyTypes value that include "Egress" (since such a policy would not include an egress section and would otherwise default to just [ "Ingress" ]). This field is beta-level in 1.8 |

### .spec.egress {#_specegress}

Description
: egress is a list of egress rules to be applied to the selected pods. Outgoing traffic is allowed if there are no NetworkPolicies selecting the pod (and cluster policy otherwise allows the traffic), OR if the traffic matches at least one egress rule across all of the NetworkPolicy objects whose podSelector matches the pod. If this field is empty then this NetworkPolicy limits all outgoing traffic (and serves solely to ensure that the pods it selects are isolated by default). This field is beta-level in 1.8

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
| `ports` | `array` | ports is a list of destination ports for outgoing traffic. Each item in this list is combined using a logical OR. If this field is empty or missing, this rule matches all ports (traffic not restricted by port). If this field is present and contains at least one item, then this rule allows traffic only if the traffic matches at least one port in the list. |
| `ports[]` | `object` | NetworkPolicyPort describes a port to allow traffic on |
| `to` | `array` | to is a list of destinations for outgoing traffic of pods selected for this rule. Items in this list are combined using a logical OR operation. If this field is empty or missing, this rule matches all destinations (traffic not restricted by destination). If this field is present and contains at least one item, this rule allows traffic only if the traffic matches at least one item in the to list. |
| `to[]` | `object` | NetworkPolicyPeer describes a peer to allow traffic to/from. Only certain combinations of fields are allowed |

### .spec.egress[].ports {#_specegressports}

Description
: ports is a list of destination ports for outgoing traffic. Each item in this list is combined using a logical OR. If this field is empty or missing, this rule matches all ports (traffic not restricted by port). If this field is present and contains at least one item, then this rule allows traffic only if the traffic matches at least one port in the list.

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
| `endPort` | `integer` | endPort indicates that the range of ports from port to endPort if set, inclusive, should be allowed by the policy. This field cannot be defined if the port field is not defined or if the port field is defined as a named (string) port. The endPort must be equal or greater than port. |
| `port` | [`IntOrString`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-util-intstr-IntOrString) | port represents the port on the given protocol. This can either be a numerical or named port on a pod. If this field is not provided, this matches all port names and numbers. If present, only traffic on the specified protocol AND port will be matched. |
| `protocol` | `string` | protocol represents the protocol (TCP, UDP, or SCTP) which traffic must match. If not specified, this field defaults to TCP.<br>Possible enum values:  - `"SCTP"` is the SCTP protocol.  - `"TCP"` is the TCP protocol.  - `"UDP"` is the UDP protocol. |

### .spec.egress[].to {#_specegressto}

Description
: to is a list of destinations for outgoing traffic of pods selected for this rule. Items in this list are combined using a logical OR operation. If this field is empty or missing, this rule matches all destinations (traffic not restricted by destination). If this field is present and contains at least one item, this rule allows traffic only if the traffic matches at least one item in the to list.

Type
: ```
  `array`
  ```

### .spec.egress[].to[] {#_specegressto}

Description
: NetworkPolicyPeer describes a peer to allow traffic to/from. Only certain combinations of fields are allowed

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ipBlock` | `object` | IPBlock describes a particular CIDR (Ex. "192.168.1.0/24","2001:db8::/64") that is allowed to the pods matched by a NetworkPolicySpec’s podSelector. The except entry describes CIDRs that should not be included within this rule. |
| `namespaceSelector` | [`LabelSelector`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-LabelSelector) | namespaceSelector selects namespaces using cluster-scoped labels. This field follows standard label selector semantics; if present but empty, it selects all namespaces.<br>If podSelector is also set, then the NetworkPolicyPeer as a whole selects the pods matching podSelector in the namespaces selected by namespaceSelector. Otherwise it selects all pods in the namespaces selected by namespaceSelector. |
| `podSelector` | [`LabelSelector`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-LabelSelector) | podSelector is a label selector which selects pods. This field follows standard label selector semantics; if present but empty, it selects all pods.<br>If namespaceSelector is also set, then the NetworkPolicyPeer as a whole selects the pods matching podSelector in the Namespaces selected by NamespaceSelector. Otherwise it selects the pods matching podSelector in the policy’s own namespace. |

### .spec.egress[].to[].ipBlock {#_specegresstoipblock}

Description
: IPBlock describes a particular CIDR (Ex. "192.168.1.0/24","2001:db8::/64") that is allowed to the pods matched by a NetworkPolicySpec’s podSelector. The except entry describes CIDRs that should not be included within this rule.

Type
: ```
  `object`
  ```

Required
: - `cidr`

| Property | Type | Description |
| --- | --- | --- |
| `cidr` | `string` | cidr is a string representing the IPBlock Valid examples are "192.168.1.0/24" or "2001:db8::/64" |
| `except` | `array (string)` | except is a slice of CIDRs that should not be included within an IPBlock Valid examples are "192.168.1.0/24" or "2001:db8::/64" Except values will be rejected if they are outside the cidr range |

### .spec.ingress {#_specingress}

Description
: ingress is a list of ingress rules to be applied to the selected pods. Traffic is allowed to a pod if there are no NetworkPolicies selecting the pod (and cluster policy otherwise allows the traffic), OR if the traffic source is the pod’s local node, OR if the traffic matches at least one ingress rule across all of the NetworkPolicy objects whose podSelector matches the pod. If this field is empty then this NetworkPolicy does not allow any traffic (and serves solely to ensure that the pods it selects are isolated by default)

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
| `from` | `array` | from is a list of sources which should be able to access the pods selected for this rule. Items in this list are combined using a logical OR operation. If this field is empty or missing, this rule matches all sources (traffic not restricted by source). If this field is present and contains at least one item, this rule allows traffic only if the traffic matches at least one item in the from list. |
| `from[]` | `object` | NetworkPolicyPeer describes a peer to allow traffic to/from. Only certain combinations of fields are allowed |
| `ports` | `array` | ports is a list of ports which should be made accessible on the pods selected for this rule. Each item in this list is combined using a logical OR. If this field is empty or missing, this rule matches all ports (traffic not restricted by port). If this field is present and contains at least one item, then this rule allows traffic only if the traffic matches at least one port in the list. |
| `ports[]` | `object` | NetworkPolicyPort describes a port to allow traffic on |

### .spec.ingress[].from {#_specingressfrom}

Description
: from is a list of sources which should be able to access the pods selected for this rule. Items in this list are combined using a logical OR operation. If this field is empty or missing, this rule matches all sources (traffic not restricted by source). If this field is present and contains at least one item, this rule allows traffic only if the traffic matches at least one item in the from list.

Type
: ```
  `array`
  ```

### .spec.ingress[].from[] {#_specingressfrom}

Description
: NetworkPolicyPeer describes a peer to allow traffic to/from. Only certain combinations of fields are allowed

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ipBlock` | `object` | IPBlock describes a particular CIDR (Ex. "192.168.1.0/24","2001:db8::/64") that is allowed to the pods matched by a NetworkPolicySpec’s podSelector. The except entry describes CIDRs that should not be included within this rule. |
| `namespaceSelector` | [`LabelSelector`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-LabelSelector) | namespaceSelector selects namespaces using cluster-scoped labels. This field follows standard label selector semantics; if present but empty, it selects all namespaces.<br>If podSelector is also set, then the NetworkPolicyPeer as a whole selects the pods matching podSelector in the namespaces selected by namespaceSelector. Otherwise it selects all pods in the namespaces selected by namespaceSelector. |
| `podSelector` | [`LabelSelector`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-LabelSelector) | podSelector is a label selector which selects pods. This field follows standard label selector semantics; if present but empty, it selects all pods.<br>If namespaceSelector is also set, then the NetworkPolicyPeer as a whole selects the pods matching podSelector in the Namespaces selected by NamespaceSelector. Otherwise it selects the pods matching podSelector in the policy’s own namespace. |

### .spec.ingress[].from[].ipBlock {#_specingressfromipblock}

Description
: IPBlock describes a particular CIDR (Ex. "192.168.1.0/24","2001:db8::/64") that is allowed to the pods matched by a NetworkPolicySpec’s podSelector. The except entry describes CIDRs that should not be included within this rule.

Type
: ```
  `object`
  ```

Required
: - `cidr`

| Property | Type | Description |
| --- | --- | --- |
| `cidr` | `string` | cidr is a string representing the IPBlock Valid examples are "192.168.1.0/24" or "2001:db8::/64" |
| `except` | `array (string)` | except is a slice of CIDRs that should not be included within an IPBlock Valid examples are "192.168.1.0/24" or "2001:db8::/64" Except values will be rejected if they are outside the cidr range |

### .spec.ingress[].ports {#_specingressports}

Description
: ports is a list of ports which should be made accessible on the pods selected for this rule. Each item in this list is combined using a logical OR. If this field is empty or missing, this rule matches all ports (traffic not restricted by port). If this field is present and contains at least one item, then this rule allows traffic only if the traffic matches at least one port in the list.

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
| `endPort` | `integer` | endPort indicates that the range of ports from port to endPort if set, inclusive, should be allowed by the policy. This field cannot be defined if the port field is not defined or if the port field is defined as a named (string) port. The endPort must be equal or greater than port. |
| `port` | [`IntOrString`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-util-intstr-IntOrString) | port represents the port on the given protocol. This can either be a numerical or named port on a pod. If this field is not provided, this matches all port names and numbers. If present, only traffic on the specified protocol AND port will be matched. |
| `protocol` | `string` | protocol represents the protocol (TCP, UDP, or SCTP) which traffic must match. If not specified, this field defaults to TCP.<br>Possible enum values:  - `"SCTP"` is the SCTP protocol.  - `"TCP"` is the TCP protocol.  - `"UDP"` is the UDP protocol. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/networking.k8s.io/v1/networkpolicies`
  - `GET`: list or watch objects of kind NetworkPolicy
- `/apis/networking.k8s.io/v1/watch/networkpolicies`
  - `GET`: watch individual changes to a list of NetworkPolicy. deprecated: use the 'watch' parameter with a list operation instead.
- `/apis/networking.k8s.io/v1/namespaces/{namespace}/networkpolicies`
  - `DELETE`: delete collection of NetworkPolicy
  - `GET`: list or watch objects of kind NetworkPolicy
  - `POST`: create a NetworkPolicy
- `/apis/networking.k8s.io/v1/watch/namespaces/{namespace}/networkpolicies`
  - `GET`: watch individual changes to a list of NetworkPolicy. deprecated: use the 'watch' parameter with a list operation instead.
- `/apis/networking.k8s.io/v1/namespaces/{namespace}/networkpolicies/{name}`
  - `DELETE`: delete a NetworkPolicy
  - `GET`: read the specified NetworkPolicy
  - `PATCH`: partially update the specified NetworkPolicy
  - `PUT`: replace the specified NetworkPolicy
- `/apis/networking.k8s.io/v1/watch/namespaces/{namespace}/networkpolicies/{name}`
  - `GET`: watch changes to an object of kind NetworkPolicy. deprecated: use the 'watch' parameter with a list operation instead, filtered to a single item with the 'fieldSelector' parameter.

### /apis/networking.k8s.io/v1/networkpolicies {#_apisnetworkingk8siov1networkpolicies}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  list or watch objects of kind NetworkPolicy
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`NetworkPolicyList`](/docs/rest_api/objects#io-k8s-api-networking-v1-NetworkPolicyList) schema |
| 401 - Unauthorized | Empty |

### /apis/networking.k8s.io/v1/watch/networkpolicies {#_apisnetworkingk8siov1watchnetworkpolicies}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch individual changes to a list of NetworkPolicy. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |

### /apis/networking.k8s.io/v1/namespaces/{namespace}/networkpolicies {#_apisnetworkingk8siov1namespaces_namespace_networkpolicies}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of NetworkPolicy
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |

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
  list or watch objects of kind NetworkPolicy
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`NetworkPolicyList`](/docs/rest_api/objects#io-k8s-api-networking-v1-NetworkPolicyList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a NetworkPolicy
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`NetworkPolicy`](/docs/rest_api/network_apis/networkpolicy-networking-k8s-io-v1#networkpolicy-networking-k8s-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`NetworkPolicy`](/docs/rest_api/network_apis/networkpolicy-networking-k8s-io-v1#networkpolicy-networking-k8s-io-v1) schema |
| 201 - Created | [`NetworkPolicy`](/docs/rest_api/network_apis/networkpolicy-networking-k8s-io-v1#networkpolicy-networking-k8s-io-v1) schema |
| 202 - Accepted | [`NetworkPolicy`](/docs/rest_api/network_apis/networkpolicy-networking-k8s-io-v1#networkpolicy-networking-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/networking.k8s.io/v1/watch/namespaces/{namespace}/networkpolicies {#_apisnetworkingk8siov1watchnamespaces_namespace_networkpolicies}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch individual changes to a list of NetworkPolicy. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |

### /apis/networking.k8s.io/v1/namespaces/{namespace}/networkpolicies/{name} {#_apisnetworkingk8siov1namespaces_namespace_networkpolicies_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the NetworkPolicy |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete a NetworkPolicy
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
  read the specified NetworkPolicy
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`NetworkPolicy`](/docs/rest_api/network_apis/networkpolicy-networking-k8s-io-v1#networkpolicy-networking-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified NetworkPolicy
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`NetworkPolicy`](/docs/rest_api/network_apis/networkpolicy-networking-k8s-io-v1#networkpolicy-networking-k8s-io-v1) schema |
| 201 - Created | [`NetworkPolicy`](/docs/rest_api/network_apis/networkpolicy-networking-k8s-io-v1#networkpolicy-networking-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified NetworkPolicy
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`NetworkPolicy`](/docs/rest_api/network_apis/networkpolicy-networking-k8s-io-v1#networkpolicy-networking-k8s-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`NetworkPolicy`](/docs/rest_api/network_apis/networkpolicy-networking-k8s-io-v1#networkpolicy-networking-k8s-io-v1) schema |
| 201 - Created | [`NetworkPolicy`](/docs/rest_api/network_apis/networkpolicy-networking-k8s-io-v1#networkpolicy-networking-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/networking.k8s.io/v1/watch/namespaces/{namespace}/networkpolicies/{name} {#_apisnetworkingk8siov1watchnamespaces_namespace_networkpolicies_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the NetworkPolicy |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch changes to an object of kind NetworkPolicy. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead, filtered to a single item with the &#x27;fieldSelector&#x27; parameter.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |
