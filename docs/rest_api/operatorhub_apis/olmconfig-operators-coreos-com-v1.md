---
title: "OLMConfig [operators.coreos.com/v1]"
sidebar_position: 8
---

# OLMConfig [operators.coreos.com/v1] {#olmconfig-operators-coreos-com-v1}

<a id="olmconfig-operators-coreos-com-v1"></a>

Description
: OLMConfig is a resource responsible for configuring OLM.

Type
: ```
  `object`
  ```

Required
: - `metadata`

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | Standard object’s metadata. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata |
| `spec` | `object` | OLMConfigSpec is the spec for an OLMConfig resource. |
| `status` | `object` | OLMConfigStatus is the status for an OLMConfig resource. |

### .spec {#_spec}

Description
: OLMConfigSpec is the spec for an OLMConfig resource.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `features` | `object` | Features contains the list of configurable OLM features. |

### .spec.features {#_specfeatures}

Description
: Features contains the list of configurable OLM features.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `disableCopiedCSVs` | `boolean` | DisableCopiedCSVs is used to disable OLM’s "Copied CSV" feature for operators installed at the cluster scope, where a cluster scoped operator is one that has been installed in an OperatorGroup that targets all namespaces. When reenabled, OLM will recreate the "Copied CSVs" for each cluster scoped operator. |
| `packageServerSyncInterval` | `string` | PackageServerSyncInterval is used to define the sync interval for packagerserver pods. Packageserver pods periodically check the status of CatalogSources; this specifies the period using duration format (e.g. "60m"). For this parameter, only hours ("h"), minutes ("m"), and seconds ("s") may be specified. When not specified, the period defaults to the value specified within the packageserver. |

### .status {#_status}

Description
: OLMConfigStatus is the status for an OLMConfig resource.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `conditions` | `array` |  |
| `conditions[]` | `object` | Condition contains details for one aspect of the current state of this API Resource. |

### .status.conditions {#_statusconditions}

Description

Type
: ```
  `array`
  ```

### .status.conditions[] {#_statusconditions}

Description
: Condition contains details for one aspect of the current state of this API Resource.

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
| `type` | `string` | type of condition in CamelCase or in foo.example.com/CamelCase. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/operators.coreos.com/v1/olmconfigs`
  - `DELETE`: delete collection of OLMConfig
  - `GET`: list objects of kind OLMConfig
  - `POST`: create an OLMConfig
- `/apis/operators.coreos.com/v1/olmconfigs/{name}`
  - `DELETE`: delete an OLMConfig
  - `GET`: read the specified OLMConfig
  - `PATCH`: partially update the specified OLMConfig
  - `PUT`: replace the specified OLMConfig
- `/apis/operators.coreos.com/v1/olmconfigs/{name}/status`
  - `GET`: read status of the specified OLMConfig
  - `PATCH`: partially update status of the specified OLMConfig
  - `PUT`: replace status of the specified OLMConfig

### /apis/operators.coreos.com/v1/olmconfigs {#_apisoperatorscoreoscomv1olmconfigs}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of OLMConfig
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
  list objects of kind OLMConfig
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`OLMConfigList`](/docs/rest_api/objects#com-coreos-operators-v1-OLMConfigList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create an OLMConfig
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`OLMConfig`](/docs/rest_api/operatorhub_apis/olmconfig-operators-coreos-com-v1#olmconfig-operators-coreos-com-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`OLMConfig`](/docs/rest_api/operatorhub_apis/olmconfig-operators-coreos-com-v1#olmconfig-operators-coreos-com-v1) schema |
| 201 - Created | [`OLMConfig`](/docs/rest_api/operatorhub_apis/olmconfig-operators-coreos-com-v1#olmconfig-operators-coreos-com-v1) schema |
| 202 - Accepted | [`OLMConfig`](/docs/rest_api/operatorhub_apis/olmconfig-operators-coreos-com-v1#olmconfig-operators-coreos-com-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/operators.coreos.com/v1/olmconfigs/{name} {#_apisoperatorscoreoscomv1olmconfigs_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the OLMConfig |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete an OLMConfig
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
  read the specified OLMConfig
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`OLMConfig`](/docs/rest_api/operatorhub_apis/olmconfig-operators-coreos-com-v1#olmconfig-operators-coreos-com-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified OLMConfig
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`OLMConfig`](/docs/rest_api/operatorhub_apis/olmconfig-operators-coreos-com-v1#olmconfig-operators-coreos-com-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified OLMConfig
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`OLMConfig`](/docs/rest_api/operatorhub_apis/olmconfig-operators-coreos-com-v1#olmconfig-operators-coreos-com-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`OLMConfig`](/docs/rest_api/operatorhub_apis/olmconfig-operators-coreos-com-v1#olmconfig-operators-coreos-com-v1) schema |
| 201 - Created | [`OLMConfig`](/docs/rest_api/operatorhub_apis/olmconfig-operators-coreos-com-v1#olmconfig-operators-coreos-com-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/operators.coreos.com/v1/olmconfigs/{name}/status {#_apisoperatorscoreoscomv1olmconfigs_name_status}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the OLMConfig |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read status of the specified OLMConfig
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`OLMConfig`](/docs/rest_api/operatorhub_apis/olmconfig-operators-coreos-com-v1#olmconfig-operators-coreos-com-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update status of the specified OLMConfig
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`OLMConfig`](/docs/rest_api/operatorhub_apis/olmconfig-operators-coreos-com-v1#olmconfig-operators-coreos-com-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace status of the specified OLMConfig
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`OLMConfig`](/docs/rest_api/operatorhub_apis/olmconfig-operators-coreos-com-v1#olmconfig-operators-coreos-com-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`OLMConfig`](/docs/rest_api/operatorhub_apis/olmconfig-operators-coreos-com-v1#olmconfig-operators-coreos-com-v1) schema |
| 201 - Created | [`OLMConfig`](/docs/rest_api/operatorhub_apis/olmconfig-operators-coreos-com-v1#olmconfig-operators-coreos-com-v1) schema |
| 401 - Unauthorized | Empty |
