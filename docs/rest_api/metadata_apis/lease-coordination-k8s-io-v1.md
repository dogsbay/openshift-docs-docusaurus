---
title: "Lease [coordination.k8s.io/v1]"
sidebar_position: 9
---

# Lease [coordination.k8s.io/v1] {#lease-coordination-k8s-io-v1}

<a id="lease-coordination-k8s-io-v1"></a>

Description
: Lease defines a lease concept.

Type
: ```
  `object`
  ```

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata |
| `spec` | `object` | LeaseSpec is a specification of a Lease. |

### .spec {#_spec}

Description
: LeaseSpec is a specification of a Lease.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `acquireTime` | [`MicroTime`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-MicroTime) | acquireTime is a time when the current lease was acquired. |
| `holderIdentity` | `string` | holderIdentity contains the identity of the holder of a current lease. If Coordinated Leader Election is used, the holder identity must be equal to the elected LeaseCandidate.metadata.name field. |
| `leaseDurationSeconds` | `integer` | leaseDurationSeconds is a duration that candidates for a lease need to wait to force acquire it. This is measured against the time of last observed renewTime. |
| `leaseTransitions` | `integer` | leaseTransitions is the number of transitions of a lease between holders. |
| `preferredHolder` | `string` | PreferredHolder signals to a lease holder that the lease has a more optimal holder and should be given up. This field can only be set if Strategy is also set. |
| `renewTime` | [`MicroTime`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-MicroTime) | renewTime is a time when the current holder of a lease has last updated the lease. |
| `strategy` | `string` | Strategy indicates the strategy for picking the leader for coordinated leader election. If the field is not specified, there is no active coordination for this lease. (Alpha) Using this field requires the CoordinatedLeaderElection feature gate to be enabled. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/coordination.k8s.io/v1/leases`
  - `GET`: list or watch objects of kind Lease
- `/apis/coordination.k8s.io/v1/watch/leases`
  - `GET`: watch individual changes to a list of Lease. deprecated: use the 'watch' parameter with a list operation instead.
- `/apis/coordination.k8s.io/v1/namespaces/{namespace}/leases`
  - `DELETE`: delete collection of Lease
  - `GET`: list or watch objects of kind Lease
  - `POST`: create a Lease
- `/apis/coordination.k8s.io/v1/watch/namespaces/{namespace}/leases`
  - `GET`: watch individual changes to a list of Lease. deprecated: use the 'watch' parameter with a list operation instead.
- `/apis/coordination.k8s.io/v1/namespaces/{namespace}/leases/{name}`
  - `DELETE`: delete a Lease
  - `GET`: read the specified Lease
  - `PATCH`: partially update the specified Lease
  - `PUT`: replace the specified Lease
- `/apis/coordination.k8s.io/v1/watch/namespaces/{namespace}/leases/{name}`
  - `GET`: watch changes to an object of kind Lease. deprecated: use the 'watch' parameter with a list operation instead, filtered to a single item with the 'fieldSelector' parameter.

### /apis/coordination.k8s.io/v1/leases {#_apiscoordinationk8siov1leases}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  list or watch objects of kind Lease
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`LeaseList`](/docs/rest_api/objects#io-k8s-api-coordination-v1-LeaseList) schema |
| 401 - Unauthorized | Empty |

### /apis/coordination.k8s.io/v1/watch/leases {#_apiscoordinationk8siov1watchleases}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch individual changes to a list of Lease. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |

### /apis/coordination.k8s.io/v1/namespaces/{namespace}/leases {#_apiscoordinationk8siov1namespaces_namespace_leases}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of Lease
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
  list or watch objects of kind Lease
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`LeaseList`](/docs/rest_api/objects#io-k8s-api-coordination-v1-LeaseList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a Lease
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`Lease`](/docs/rest_api/metadata_apis/lease-coordination-k8s-io-v1#lease-coordination-k8s-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Lease`](/docs/rest_api/metadata_apis/lease-coordination-k8s-io-v1#lease-coordination-k8s-io-v1) schema |
| 201 - Created | [`Lease`](/docs/rest_api/metadata_apis/lease-coordination-k8s-io-v1#lease-coordination-k8s-io-v1) schema |
| 202 - Accepted | [`Lease`](/docs/rest_api/metadata_apis/lease-coordination-k8s-io-v1#lease-coordination-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/coordination.k8s.io/v1/watch/namespaces/{namespace}/leases {#_apiscoordinationk8siov1watchnamespaces_namespace_leases}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch individual changes to a list of Lease. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |

### /apis/coordination.k8s.io/v1/namespaces/{namespace}/leases/{name} {#_apiscoordinationk8siov1namespaces_namespace_leases_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the Lease |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete a Lease
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
  read the specified Lease
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Lease`](/docs/rest_api/metadata_apis/lease-coordination-k8s-io-v1#lease-coordination-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified Lease
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Lease`](/docs/rest_api/metadata_apis/lease-coordination-k8s-io-v1#lease-coordination-k8s-io-v1) schema |
| 201 - Created | [`Lease`](/docs/rest_api/metadata_apis/lease-coordination-k8s-io-v1#lease-coordination-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified Lease
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`Lease`](/docs/rest_api/metadata_apis/lease-coordination-k8s-io-v1#lease-coordination-k8s-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Lease`](/docs/rest_api/metadata_apis/lease-coordination-k8s-io-v1#lease-coordination-k8s-io-v1) schema |
| 201 - Created | [`Lease`](/docs/rest_api/metadata_apis/lease-coordination-k8s-io-v1#lease-coordination-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/coordination.k8s.io/v1/watch/namespaces/{namespace}/leases/{name} {#_apiscoordinationk8siov1watchnamespaces_namespace_leases_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the Lease |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch changes to an object of kind Lease. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead, filtered to a single item with the &#x27;fieldSelector&#x27; parameter.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |
