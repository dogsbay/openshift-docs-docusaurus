---
title: "RuntimeClass [node.k8s.io/v1]"
sidebar_position: 5
---

# RuntimeClass [node.k8s.io/v1] {#runtimeclass-node-k8s-io-v1}

<a id="runtimeclass-node-k8s-io-v1"></a>

Description
: RuntimeClass defines a class of container runtime supported in the cluster. The RuntimeClass is used to determine which container runtime is used to run all containers in a pod. RuntimeClasses are manually defined by a user or cluster provisioner, and referenced in the PodSpec. The Kubelet is responsible for resolving the RuntimeClassName reference before running the pod.  For more details, see https://kubernetes.io/docs/concepts/containers/runtime-class/

Type
: ```
  `object`
  ```

Required
: - `handler`

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `handler` | `string` | handler specifies the underlying runtime and configuration that the CRI implementation will use to handle pods of this class. The possible values are specific to the node & CRI configuration.  It is assumed that all handlers are available on every node, and handlers of the same name are equivalent on every node. For example, a handler called "runc" might specify that the runc OCI runtime (using native Linux containers) will be used to run the containers in a pod. The Handler must be lowercase, conform to the DNS Label (RFC 1123) requirements, and is immutable. |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata |
| `overhead` | `object` | Overhead structure represents the resource overhead associated with running a pod. |
| `scheduling` | `object` | Scheduling specifies the scheduling constraints for nodes supporting a RuntimeClass. |

### .overhead {#_overhead}

Description
: Overhead structure represents the resource overhead associated with running a pod.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `podFixed` | [`object (Quantity)`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-api-resource-Quantity) | podFixed represents the fixed resource overhead associated with running a pod. |

### .scheduling {#_scheduling}

Description
: Scheduling specifies the scheduling constraints for nodes supporting a RuntimeClass.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `nodeSelector` | `object (string)` | nodeSelector lists labels that must be present on nodes that support this RuntimeClass. Pods using this RuntimeClass can only be scheduled to a node matched by this selector. The RuntimeClass nodeSelector is merged with a pod’s existing nodeSelector. Any conflicts will cause the pod to be rejected in admission. |
| `tolerations` | [`array (Toleration)`](/docs/rest_api/objects#io-k8s-api-core-v1-Toleration) | tolerations are appended (excluding duplicates) to pods running with this RuntimeClass during admission, effectively unioning the set of nodes tolerated by the pod and the RuntimeClass. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/node.k8s.io/v1/runtimeclasses`
  - `DELETE`: delete collection of RuntimeClass
  - `GET`: list or watch objects of kind RuntimeClass
  - `POST`: create a RuntimeClass
- `/apis/node.k8s.io/v1/watch/runtimeclasses`
  - `GET`: watch individual changes to a list of RuntimeClass. deprecated: use the 'watch' parameter with a list operation instead.
- `/apis/node.k8s.io/v1/runtimeclasses/{name}`
  - `DELETE`: delete a RuntimeClass
  - `GET`: read the specified RuntimeClass
  - `PATCH`: partially update the specified RuntimeClass
  - `PUT`: replace the specified RuntimeClass
- `/apis/node.k8s.io/v1/watch/runtimeclasses/{name}`
  - `GET`: watch changes to an object of kind RuntimeClass. deprecated: use the 'watch' parameter with a list operation instead, filtered to a single item with the 'fieldSelector' parameter.

### /apis/node.k8s.io/v1/runtimeclasses {#_apisnodek8siov1runtimeclasses}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of RuntimeClass
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
  list or watch objects of kind RuntimeClass
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`RuntimeClassList`](/docs/rest_api/objects#io-k8s-api-node-v1-RuntimeClassList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a RuntimeClass
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`RuntimeClass`](/docs/rest_api/node_apis/runtimeclass-node-k8s-io-v1#runtimeclass-node-k8s-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`RuntimeClass`](/docs/rest_api/node_apis/runtimeclass-node-k8s-io-v1#runtimeclass-node-k8s-io-v1) schema |
| 201 - Created | [`RuntimeClass`](/docs/rest_api/node_apis/runtimeclass-node-k8s-io-v1#runtimeclass-node-k8s-io-v1) schema |
| 202 - Accepted | [`RuntimeClass`](/docs/rest_api/node_apis/runtimeclass-node-k8s-io-v1#runtimeclass-node-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/node.k8s.io/v1/watch/runtimeclasses {#_apisnodek8siov1watchruntimeclasses}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch individual changes to a list of RuntimeClass. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |

### /apis/node.k8s.io/v1/runtimeclasses/{name} {#_apisnodek8siov1runtimeclasses_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the RuntimeClass |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete a RuntimeClass
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
  read the specified RuntimeClass
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`RuntimeClass`](/docs/rest_api/node_apis/runtimeclass-node-k8s-io-v1#runtimeclass-node-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified RuntimeClass
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`RuntimeClass`](/docs/rest_api/node_apis/runtimeclass-node-k8s-io-v1#runtimeclass-node-k8s-io-v1) schema |
| 201 - Created | [`RuntimeClass`](/docs/rest_api/node_apis/runtimeclass-node-k8s-io-v1#runtimeclass-node-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified RuntimeClass
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`RuntimeClass`](/docs/rest_api/node_apis/runtimeclass-node-k8s-io-v1#runtimeclass-node-k8s-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`RuntimeClass`](/docs/rest_api/node_apis/runtimeclass-node-k8s-io-v1#runtimeclass-node-k8s-io-v1) schema |
| 201 - Created | [`RuntimeClass`](/docs/rest_api/node_apis/runtimeclass-node-k8s-io-v1#runtimeclass-node-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/node.k8s.io/v1/watch/runtimeclasses/{name} {#_apisnodek8siov1watchruntimeclasses_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the RuntimeClass |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch changes to an object of kind RuntimeClass. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead, filtered to a single item with the &#x27;fieldSelector&#x27; parameter.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |
