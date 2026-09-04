---
title: "Namespace [v1]"
sidebar_position: 10
---

# Namespace [v1]

<a id="namespace-v1"></a>

Description
: Namespace provides a scope for Names. Use of multiple namespaces is optional.

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
| `spec` | `object` | NamespaceSpec describes the attributes on a Namespace. |
| `status` | `object` | NamespaceStatus is information about the current status of a Namespace. |

### .spec {#_spec}

Description
: NamespaceSpec describes the attributes on a Namespace.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `finalizers` | `array (string)` | Finalizers is an opaque list of values that must be empty to permanently remove object from storage. More info: https://kubernetes.io/docs/tasks/administer-cluster/namespaces/ |

### .status {#_status}

Description
: NamespaceStatus is information about the current status of a Namespace.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `conditions` | `array` | Represents the latest available observations of a namespace’s current state. |
| `conditions[]` | `object` | NamespaceCondition contains details about state of namespace. |
| `phase` | `string` | Phase is the current lifecycle phase of the namespace. More info: https://kubernetes.io/docs/tasks/administer-cluster/namespaces/<br>Possible enum values:  - `"Active"` means the namespace is available for use in the system  - `"Terminating"` means the namespace is undergoing graceful termination |

### .status.conditions {#_statusconditions}

Description
: Represents the latest available observations of a namespace’s current state.

Type
: ```
  `array`
  ```

### .status.conditions[] {#_statusconditions}

Description
: NamespaceCondition contains details about state of namespace.

Type
: ```
  `object`
  ```

Required
: - `type`
  - `status`

| Property | Type | Description |
| --- | --- | --- |
| `lastTransitionTime` | [`Time`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-Time) | Last time the condition transitioned from one status to another. |
| `message` | `string` | Human-readable message indicating details about last transition. |
| `reason` | `string` | Unique, one-word, CamelCase reason for the condition’s last transition. |
| `status` | `string` | Status of the condition, one of True, False, Unknown. |
| `type` | `string` | Type of namespace controller condition. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/api/v1/namespaces`
  - `GET`: list or watch objects of kind Namespace
  - `POST`: create a Namespace
- `/api/v1/watch/namespaces`
  - `GET`: watch individual changes to a list of Namespace. deprecated: use the 'watch' parameter with a list operation instead.
- `/api/v1/namespaces/{name}`
  - `DELETE`: delete a Namespace
  - `GET`: read the specified Namespace
  - `PATCH`: partially update the specified Namespace
  - `PUT`: replace the specified Namespace
- `/api/v1/watch/namespaces/{name}`
  - `GET`: watch changes to an object of kind Namespace. deprecated: use the 'watch' parameter with a list operation instead, filtered to a single item with the 'fieldSelector' parameter.
- `/api/v1/namespaces/{name}/status`
  - `GET`: read status of the specified Namespace
  - `PATCH`: partially update status of the specified Namespace
  - `PUT`: replace status of the specified Namespace
- `/api/v1/namespaces/{name}/finalize`
  - `PUT`: replace finalize of the specified Namespace

### /api/v1/namespaces {#_apiv1namespaces}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  list or watch objects of kind Namespace
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`NamespaceList`](/docs/rest_api/objects#io-k8s-api-core-v1-NamespaceList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a Namespace
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`Namespace`](/docs/rest_api/metadata_apis/namespace-v1#namespace-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Namespace`](/docs/rest_api/metadata_apis/namespace-v1#namespace-v1) schema |
| 201 - Created | [`Namespace`](/docs/rest_api/metadata_apis/namespace-v1#namespace-v1) schema |
| 202 - Accepted | [`Namespace`](/docs/rest_api/metadata_apis/namespace-v1#namespace-v1) schema |
| 401 - Unauthorized | Empty |

### /api/v1/watch/namespaces {#_apiv1watchnamespaces}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch individual changes to a list of Namespace. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |

### /api/v1/namespaces/{name} {#_apiv1namespaces_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the Namespace |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete a Namespace
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
  read the specified Namespace
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Namespace`](/docs/rest_api/metadata_apis/namespace-v1#namespace-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified Namespace
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Namespace`](/docs/rest_api/metadata_apis/namespace-v1#namespace-v1) schema |
| 201 - Created | [`Namespace`](/docs/rest_api/metadata_apis/namespace-v1#namespace-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified Namespace
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`Namespace`](/docs/rest_api/metadata_apis/namespace-v1#namespace-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Namespace`](/docs/rest_api/metadata_apis/namespace-v1#namespace-v1) schema |
| 201 - Created | [`Namespace`](/docs/rest_api/metadata_apis/namespace-v1#namespace-v1) schema |
| 401 - Unauthorized | Empty |

### /api/v1/watch/namespaces/{name} {#_apiv1watchnamespaces_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the Namespace |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch changes to an object of kind Namespace. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead, filtered to a single item with the &#x27;fieldSelector&#x27; parameter.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |

### /api/v1/namespaces/{name}/status {#_apiv1namespaces_name_status}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the Namespace |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read status of the specified Namespace
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Namespace`](/docs/rest_api/metadata_apis/namespace-v1#namespace-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update status of the specified Namespace
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Namespace`](/docs/rest_api/metadata_apis/namespace-v1#namespace-v1) schema |
| 201 - Created | [`Namespace`](/docs/rest_api/metadata_apis/namespace-v1#namespace-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace status of the specified Namespace
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`Namespace`](/docs/rest_api/metadata_apis/namespace-v1#namespace-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Namespace`](/docs/rest_api/metadata_apis/namespace-v1#namespace-v1) schema |
| 201 - Created | [`Namespace`](/docs/rest_api/metadata_apis/namespace-v1#namespace-v1) schema |
| 401 - Unauthorized | Empty |

### /api/v1/namespaces/{name}/finalize {#_apiv1namespaces_name_finalize}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the Namespace |

**Global query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace finalize of the specified Namespace
  ```

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`Namespace`](/docs/rest_api/metadata_apis/namespace-v1#namespace-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Namespace`](/docs/rest_api/metadata_apis/namespace-v1#namespace-v1) schema |
| 201 - Created | [`Namespace`](/docs/rest_api/metadata_apis/namespace-v1#namespace-v1) schema |
| 401 - Unauthorized | Empty |
