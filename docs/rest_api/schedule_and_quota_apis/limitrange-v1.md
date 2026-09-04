---
title: "LimitRange [v1]"
sidebar_position: 6
---

# LimitRange [v1]

<a id="limitrange-v1"></a>

Description
: LimitRange sets resource usage limits for each kind of resource in a Namespace.

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
| `spec` | `object` | LimitRangeSpec defines a min/max usage limit for resources that match on kind. |

### .spec {#_spec}

Description
: LimitRangeSpec defines a min/max usage limit for resources that match on kind.

Type
: ```
  `object`
  ```

Required
: - `limits`

| Property | Type | Description |
| --- | --- | --- |
| `limits` | `array` | Limits is the list of LimitRangeItem objects that are enforced. |
| `limits[]` | `object` | LimitRangeItem defines a min/max usage limit for any resource that matches on kind. |

### .spec.limits {#_speclimits}

Description
: Limits is the list of LimitRangeItem objects that are enforced.

Type
: ```
  `array`
  ```

### .spec.limits[] {#_speclimits}

Description
: LimitRangeItem defines a min/max usage limit for any resource that matches on kind.

Type
: ```
  `object`
  ```

Required
: - `type`

| Property | Type | Description |
| --- | --- | --- |
| `default` | [`object (Quantity)`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-api-resource-Quantity) | Default resource requirement limit value by resource name if resource limit is omitted. |
| `defaultRequest` | [`object (Quantity)`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-api-resource-Quantity) | DefaultRequest is the default resource requirement request value by resource name if resource request is omitted. |
| `max` | [`object (Quantity)`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-api-resource-Quantity) | Max usage constraints on this kind by resource name. |
| `maxLimitRequestRatio` | [`object (Quantity)`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-api-resource-Quantity) | MaxLimitRequestRatio if specified, the named resource must have a request and limit that are both non-zero where limit divided by request is less than or equal to the enumerated value; this represents the max burst for the named resource. |
| `min` | [`object (Quantity)`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-api-resource-Quantity) | Min usage constraints on this kind by resource name. |
| `type` | `string` | Type of resource that this limit applies to. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/api/v1/limitranges`
  - `GET`: list or watch objects of kind LimitRange
- `/api/v1/watch/limitranges`
  - `GET`: watch individual changes to a list of LimitRange. deprecated: use the 'watch' parameter with a list operation instead.
- `/api/v1/namespaces/{namespace}/limitranges`
  - `DELETE`: delete collection of LimitRange
  - `GET`: list or watch objects of kind LimitRange
  - `POST`: create a LimitRange
- `/api/v1/watch/namespaces/{namespace}/limitranges`
  - `GET`: watch individual changes to a list of LimitRange. deprecated: use the 'watch' parameter with a list operation instead.
- `/api/v1/namespaces/{namespace}/limitranges/{name}`
  - `DELETE`: delete a LimitRange
  - `GET`: read the specified LimitRange
  - `PATCH`: partially update the specified LimitRange
  - `PUT`: replace the specified LimitRange
- `/api/v1/watch/namespaces/{namespace}/limitranges/{name}`
  - `GET`: watch changes to an object of kind LimitRange. deprecated: use the 'watch' parameter with a list operation instead, filtered to a single item with the 'fieldSelector' parameter.

### /api/v1/limitranges {#_apiv1limitranges}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  list or watch objects of kind LimitRange
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`LimitRangeList`](/docs/rest_api/objects#io-k8s-api-core-v1-LimitRangeList) schema |
| 401 - Unauthorized | Empty |

### /api/v1/watch/limitranges {#_apiv1watchlimitranges}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch individual changes to a list of LimitRange. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |

### /api/v1/namespaces/{namespace}/limitranges {#_apiv1namespaces_namespace_limitranges}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of LimitRange
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
  list or watch objects of kind LimitRange
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`LimitRangeList`](/docs/rest_api/objects#io-k8s-api-core-v1-LimitRangeList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a LimitRange
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`LimitRange`](/docs/rest_api/schedule_and_quota_apis/limitrange-v1#limitrange-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`LimitRange`](/docs/rest_api/schedule_and_quota_apis/limitrange-v1#limitrange-v1) schema |
| 201 - Created | [`LimitRange`](/docs/rest_api/schedule_and_quota_apis/limitrange-v1#limitrange-v1) schema |
| 202 - Accepted | [`LimitRange`](/docs/rest_api/schedule_and_quota_apis/limitrange-v1#limitrange-v1) schema |
| 401 - Unauthorized | Empty |

### /api/v1/watch/namespaces/{namespace}/limitranges {#_apiv1watchnamespaces_namespace_limitranges}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch individual changes to a list of LimitRange. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |

### /api/v1/namespaces/{namespace}/limitranges/{name} {#_apiv1namespaces_namespace_limitranges_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the LimitRange |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete a LimitRange
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
  read the specified LimitRange
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`LimitRange`](/docs/rest_api/schedule_and_quota_apis/limitrange-v1#limitrange-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified LimitRange
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`LimitRange`](/docs/rest_api/schedule_and_quota_apis/limitrange-v1#limitrange-v1) schema |
| 201 - Created | [`LimitRange`](/docs/rest_api/schedule_and_quota_apis/limitrange-v1#limitrange-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified LimitRange
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`LimitRange`](/docs/rest_api/schedule_and_quota_apis/limitrange-v1#limitrange-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`LimitRange`](/docs/rest_api/schedule_and_quota_apis/limitrange-v1#limitrange-v1) schema |
| 201 - Created | [`LimitRange`](/docs/rest_api/schedule_and_quota_apis/limitrange-v1#limitrange-v1) schema |
| 401 - Unauthorized | Empty |

### /api/v1/watch/namespaces/{namespace}/limitranges/{name} {#_apiv1watchnamespaces_namespace_limitranges_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the LimitRange |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch changes to an object of kind LimitRange. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead, filtered to a single item with the &#x27;fieldSelector&#x27; parameter.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |
