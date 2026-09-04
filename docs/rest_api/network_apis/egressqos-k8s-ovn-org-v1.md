---
title: "EgressQoS [k8s.ovn.org/v1]"
sidebar_position: 10
---

# EgressQoS [k8s.ovn.org/v1] {#egressqos-k8s-ovn-org-v1}

<a id="egressqos-k8s-ovn-org-v1"></a>

Description
: EgressQoS is a CRD that allows the user to define a DSCP value for pods egress traffic on its namespace to specified CIDRs. Traffic from these pods will be checked against each EgressQoSRule in the namespace’s EgressQoS, and if there is a match the traffic is marked with the relevant DSCP value.

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
| `spec` | `object` | EgressQoSSpec defines the desired state of EgressQoS |
| `status` | `object` | EgressQoSStatus defines the observed state of EgressQoS |

### .spec {#_spec}

Description
: EgressQoSSpec defines the desired state of EgressQoS

Type
: ```
  `object`
  ```

Required
: - `egress`

| Property | Type | Description |
| --- | --- | --- |
| `egress` | `array` | a collection of Egress QoS rule objects |
| `egress[]` | `object` |  |

### .spec.egress {#_specegress}

Description
: a collection of Egress QoS rule objects

Type
: ```
  `array`
  ```

### .spec.egress[] {#_specegress}

Description

Type
: ```
  `object`
  ```

Required
: - `dscp`

| Property | Type | Description |
| --- | --- | --- |
| `dscp` | `integer` | DSCP marking value for matching pods' traffic. |
| `dstCIDR` | `string` | DstCIDR specifies the destination’s CIDR. Only traffic heading to this CIDR will be marked with the DSCP value. This field is optional, and in case it is not set the rule is applied to all egress traffic regardless of the destination. |
| `podSelector` | `object` | PodSelector applies the QoS rule only to the pods in the namespace whose label matches this definition. This field is optional, and in case it is not set results in the rule being applied to all pods in the namespace. |

### .spec.egress[].podSelector {#_specegresspodselector}

Description
: PodSelector applies the QoS rule only to the pods in the namespace whose label matches this definition. This field is optional, and in case it is not set results in the rule being applied to all pods in the namespace.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `matchExpressions` | `array` | matchExpressions is a list of label selector requirements. The requirements are ANDed. |
| `matchExpressions[]` | `object` | A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values. |
| `matchLabels` | `object (string)` | matchLabels is a map of {key,value} pairs. A single {key,value} in the matchLabels map is equivalent to an element of matchExpressions, whose key field is "key", the operator is "In", and the values array contains only "value". The requirements are ANDed. |

### .spec.egress[].podSelector.matchExpressions {#_specegresspodselectormatchexpressions}

Description
: matchExpressions is a list of label selector requirements. The requirements are ANDed.

Type
: ```
  `array`
  ```

### .spec.egress[].podSelector.matchExpressions[] {#_specegresspodselectormatchexpressions}

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
: EgressQoSStatus defines the observed state of EgressQoS

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `conditions` | `array` | An array of condition objects indicating details about status of EgressQoS object. |
| `conditions[]` | `object` | Condition contains details for one aspect of the current state of this API Resource. --- This struct is intended for direct use as an array at the field path .status.conditions.  For example,<br>	type FooStatus struct{ 	    // Represents the observations of a foo’s current state. 	    // Known .status.conditions.type are: "Available", "Progressing", and "Degraded" 	    // +patchMergeKey=type 	    // +patchStrategy=merge 	    // +listType=map 	    // +listMapKey=type 	    Conditions []metav1.Condition `json:"conditions,omitempty" patchStrategy:"merge" patchMergeKey:"type" protobuf:"bytes,1,rep,name=conditions"`<br>	    // other fields 	} |
| `status` | `string` | A concise indication of whether the EgressQoS resource is applied with success. |

### .status.conditions {#_statusconditions}

Description
: An array of condition objects indicating details about status of EgressQoS object.

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

- `/apis/k8s.ovn.org/v1/egressqoses`
  - `GET`: list objects of kind EgressQoS
- `/apis/k8s.ovn.org/v1/namespaces/{namespace}/egressqoses`
  - `DELETE`: delete collection of EgressQoS
  - `GET`: list objects of kind EgressQoS
  - `POST`: create an EgressQoS
- `/apis/k8s.ovn.org/v1/namespaces/{namespace}/egressqoses/{name}`
  - `DELETE`: delete an EgressQoS
  - `GET`: read the specified EgressQoS
  - `PATCH`: partially update the specified EgressQoS
  - `PUT`: replace the specified EgressQoS
- `/apis/k8s.ovn.org/v1/namespaces/{namespace}/egressqoses/{name}/status`
  - `GET`: read status of the specified EgressQoS
  - `PATCH`: partially update status of the specified EgressQoS
  - `PUT`: replace status of the specified EgressQoS

### /apis/k8s.ovn.org/v1/egressqoses {#_apisk8sovnorgv1egressqoses}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  list objects of kind EgressQoS
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`EgressQoSList`](/docs/rest_api/objects#org-ovn-k8s-v1-EgressQoSList) schema |
| 401 - Unauthorized | Empty |

### /apis/k8s.ovn.org/v1/namespaces/{namespace}/egressqoses {#_apisk8sovnorgv1namespaces_namespace_egressqoses}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of EgressQoS
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
  list objects of kind EgressQoS
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`EgressQoSList`](/docs/rest_api/objects#org-ovn-k8s-v1-EgressQoSList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create an EgressQoS
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`EgressQoS`](/docs/rest_api/network_apis/egressqos-k8s-ovn-org-v1#egressqos-k8s-ovn-org-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`EgressQoS`](/docs/rest_api/network_apis/egressqos-k8s-ovn-org-v1#egressqos-k8s-ovn-org-v1) schema |
| 201 - Created | [`EgressQoS`](/docs/rest_api/network_apis/egressqos-k8s-ovn-org-v1#egressqos-k8s-ovn-org-v1) schema |
| 202 - Accepted | [`EgressQoS`](/docs/rest_api/network_apis/egressqos-k8s-ovn-org-v1#egressqos-k8s-ovn-org-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/k8s.ovn.org/v1/namespaces/{namespace}/egressqoses/{name} {#_apisk8sovnorgv1namespaces_namespace_egressqoses_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the EgressQoS |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete an EgressQoS
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
  read the specified EgressQoS
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`EgressQoS`](/docs/rest_api/network_apis/egressqos-k8s-ovn-org-v1#egressqos-k8s-ovn-org-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified EgressQoS
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`EgressQoS`](/docs/rest_api/network_apis/egressqos-k8s-ovn-org-v1#egressqos-k8s-ovn-org-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified EgressQoS
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`EgressQoS`](/docs/rest_api/network_apis/egressqos-k8s-ovn-org-v1#egressqos-k8s-ovn-org-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`EgressQoS`](/docs/rest_api/network_apis/egressqos-k8s-ovn-org-v1#egressqos-k8s-ovn-org-v1) schema |
| 201 - Created | [`EgressQoS`](/docs/rest_api/network_apis/egressqos-k8s-ovn-org-v1#egressqos-k8s-ovn-org-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/k8s.ovn.org/v1/namespaces/{namespace}/egressqoses/{name}/status {#_apisk8sovnorgv1namespaces_namespace_egressqoses_name_status}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the EgressQoS |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read status of the specified EgressQoS
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`EgressQoS`](/docs/rest_api/network_apis/egressqos-k8s-ovn-org-v1#egressqos-k8s-ovn-org-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update status of the specified EgressQoS
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`EgressQoS`](/docs/rest_api/network_apis/egressqos-k8s-ovn-org-v1#egressqos-k8s-ovn-org-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace status of the specified EgressQoS
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`EgressQoS`](/docs/rest_api/network_apis/egressqos-k8s-ovn-org-v1#egressqos-k8s-ovn-org-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`EgressQoS`](/docs/rest_api/network_apis/egressqos-k8s-ovn-org-v1#egressqos-k8s-ovn-org-v1) schema |
| 201 - Created | [`EgressQoS`](/docs/rest_api/network_apis/egressqos-k8s-ovn-org-v1#egressqos-k8s-ovn-org-v1) schema |
| 401 - Unauthorized | Empty |
