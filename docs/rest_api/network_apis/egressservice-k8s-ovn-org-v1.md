---
title: "EgressService [k8s.ovn.org/v1]"
sidebar_position: 11
---

# EgressService [k8s.ovn.org/v1] {#egressservice-k8s-ovn-org-v1}

<a id="egressservice-k8s-ovn-org-v1"></a>

Description
: EgressService is a CRD that allows the user to request that the source IP of egress packets originating from all of the pods that are endpoints of the corresponding LoadBalancer Service would be its ingress IP. In addition, it allows the user to request that egress packets originating from all of the pods that are endpoints of the LoadBalancer service would use a different network than the main one.

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
| `spec` | `object` | EgressServiceSpec defines the desired state of EgressService |
| `status` | `object` | EgressServiceStatus defines the observed state of EgressService |

### .spec {#_spec}

Description
: EgressServiceSpec defines the desired state of EgressService

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `network` | `string` | The network which this service should send egress and corresponding ingress replies to. This is typically implemented as VRF mapping, representing a numeric id or string name of a routing table which by omission uses the default host routing. |
| `nodeSelector` | `object` | Allows limiting the nodes that can be selected to handle the service’s traffic when sourceIPBy=LoadBalancerIP. When present only a node whose labels match the specified selectors can be selected for handling the service’s traffic. When it is not specified any node in the cluster can be chosen to manage the service’s traffic. |
| `sourceIPBy` | `string` | Determines the source IP of egress traffic originating from the pods backing the LoadBalancer Service. When `LoadBalancerIP` the source IP is set to its LoadBalancer ingress IP. When `Network` the source IP is set according to the interface of the Network, leveraging the masquerade rules that are already in place. Typically these rules specify SNAT to the IP of the outgoing interface, which means the packet will typically leave with the IP of the node. |

### .spec.nodeSelector {#_specnodeselector}

Description
: Allows limiting the nodes that can be selected to handle the service’s traffic when sourceIPBy=LoadBalancerIP. When present only a node whose labels match the specified selectors can be selected for handling the service’s traffic. When it is not specified any node in the cluster can be chosen to manage the service’s traffic.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `matchExpressions` | `array` | matchExpressions is a list of label selector requirements. The requirements are ANDed. |
| `matchExpressions[]` | `object` | A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values. |
| `matchLabels` | `object (string)` | matchLabels is a map of {key,value} pairs. A single {key,value} in the matchLabels map is equivalent to an element of matchExpressions, whose key field is "key", the operator is "In", and the values array contains only "value". The requirements are ANDed. |

### .spec.nodeSelector.matchExpressions {#_specnodeselectormatchexpressions}

Description
: matchExpressions is a list of label selector requirements. The requirements are ANDed.

Type
: ```
  `array`
  ```

### .spec.nodeSelector.matchExpressions[] {#_specnodeselectormatchexpressions}

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
: EgressServiceStatus defines the observed state of EgressService

Type
: ```
  `object`
  ```

Required
: - `host`

| Property | Type | Description |
| --- | --- | --- |
| `host` | `string` | The name of the node selected to handle the service’s traffic. In case sourceIPBy=Network the field will be set to "ALL". |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/k8s.ovn.org/v1/egressservices`
  - `GET`: list objects of kind EgressService
- `/apis/k8s.ovn.org/v1/namespaces/{namespace}/egressservices`
  - `DELETE`: delete collection of EgressService
  - `GET`: list objects of kind EgressService
  - `POST`: create an EgressService
- `/apis/k8s.ovn.org/v1/namespaces/{namespace}/egressservices/{name}`
  - `DELETE`: delete an EgressService
  - `GET`: read the specified EgressService
  - `PATCH`: partially update the specified EgressService
  - `PUT`: replace the specified EgressService
- `/apis/k8s.ovn.org/v1/namespaces/{namespace}/egressservices/{name}/status`
  - `GET`: read status of the specified EgressService
  - `PATCH`: partially update status of the specified EgressService
  - `PUT`: replace status of the specified EgressService

### /apis/k8s.ovn.org/v1/egressservices {#_apisk8sovnorgv1egressservices}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  list objects of kind EgressService
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`EgressServiceList`](/docs/rest_api/objects#org-ovn-k8s-v1-EgressServiceList) schema |
| 401 - Unauthorized | Empty |

### /apis/k8s.ovn.org/v1/namespaces/{namespace}/egressservices {#_apisk8sovnorgv1namespaces_namespace_egressservices}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of EgressService
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
  list objects of kind EgressService
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`EgressServiceList`](/docs/rest_api/objects#org-ovn-k8s-v1-EgressServiceList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create an EgressService
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`EgressService`](/docs/rest_api/network_apis/egressservice-k8s-ovn-org-v1#egressservice-k8s-ovn-org-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`EgressService`](/docs/rest_api/network_apis/egressservice-k8s-ovn-org-v1#egressservice-k8s-ovn-org-v1) schema |
| 201 - Created | [`EgressService`](/docs/rest_api/network_apis/egressservice-k8s-ovn-org-v1#egressservice-k8s-ovn-org-v1) schema |
| 202 - Accepted | [`EgressService`](/docs/rest_api/network_apis/egressservice-k8s-ovn-org-v1#egressservice-k8s-ovn-org-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/k8s.ovn.org/v1/namespaces/{namespace}/egressservices/{name} {#_apisk8sovnorgv1namespaces_namespace_egressservices_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the EgressService |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete an EgressService
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
  read the specified EgressService
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`EgressService`](/docs/rest_api/network_apis/egressservice-k8s-ovn-org-v1#egressservice-k8s-ovn-org-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified EgressService
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`EgressService`](/docs/rest_api/network_apis/egressservice-k8s-ovn-org-v1#egressservice-k8s-ovn-org-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified EgressService
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`EgressService`](/docs/rest_api/network_apis/egressservice-k8s-ovn-org-v1#egressservice-k8s-ovn-org-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`EgressService`](/docs/rest_api/network_apis/egressservice-k8s-ovn-org-v1#egressservice-k8s-ovn-org-v1) schema |
| 201 - Created | [`EgressService`](/docs/rest_api/network_apis/egressservice-k8s-ovn-org-v1#egressservice-k8s-ovn-org-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/k8s.ovn.org/v1/namespaces/{namespace}/egressservices/{name}/status {#_apisk8sovnorgv1namespaces_namespace_egressservices_name_status}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the EgressService |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read status of the specified EgressService
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`EgressService`](/docs/rest_api/network_apis/egressservice-k8s-ovn-org-v1#egressservice-k8s-ovn-org-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update status of the specified EgressService
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`EgressService`](/docs/rest_api/network_apis/egressservice-k8s-ovn-org-v1#egressservice-k8s-ovn-org-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace status of the specified EgressService
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`EgressService`](/docs/rest_api/network_apis/egressservice-k8s-ovn-org-v1#egressservice-k8s-ovn-org-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`EgressService`](/docs/rest_api/network_apis/egressservice-k8s-ovn-org-v1#egressservice-k8s-ovn-org-v1) schema |
| 201 - Created | [`EgressService`](/docs/rest_api/network_apis/egressservice-k8s-ovn-org-v1#egressservice-k8s-ovn-org-v1) schema |
| 401 - Unauthorized | Empty |
