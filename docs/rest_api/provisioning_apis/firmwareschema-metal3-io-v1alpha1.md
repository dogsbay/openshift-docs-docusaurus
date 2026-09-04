---
title: "FirmwareSchema [metal3.io/v1alpha1]"
sidebar_position: 5
---

# FirmwareSchema [metal3.io/v1alpha1] {#firmwareschema-metal3-io-v1alpha1}

<a id="firmwareschema-metal3-io-v1alpha1"></a>

Description
: FirmwareSchema is the Schema for the firmwareschemas API.

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
| `spec` | `object` | FirmwareSchemaSpec defines the desired state of FirmwareSchema. |

### .spec {#_spec}

Description
: FirmwareSchemaSpec defines the desired state of FirmwareSchema.

Type
: ```
  `object`
  ```

Required
: - `schema`

| Property | Type | Description |
| --- | --- | --- |
| `hardwareModel` | `string` | The hardware model associated with this schema |
| `hardwareVendor` | `string` | The hardware vendor associated with this schema |
| `schema` | `object` | Map of firmware name to schema |
| `schema{}` | `object` | Additional data describing the firmware setting. |

### .spec.schema {#_specschema}

Description
: Map of firmware name to schema

Type
: ```
  `object`
  ```

### .spec.schema{} {#_specschema}

Description
: Additional data describing the firmware setting.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `allowable_values` | `array (string)` | The allowable value for an Enumeration type setting. |
| `attribute_type` | `string` | The type of setting. |
| `lower_bound` | `integer` | The lowest value for an Integer type setting. |
| `max_length` | `integer` | Maximum length for a String type setting. |
| `min_length` | `integer` | Minimum length for a String type setting. |
| `read_only` | `boolean` | Whether or not this setting is read only. |
| `unique` | `boolean` | Whether or not this setting’s value is unique to this node, e.g. a serial number. |
| `upper_bound` | `integer` | The highest value for an Integer type setting. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/metal3.io/v1alpha1/firmwareschemas`
  - `GET`: list objects of kind FirmwareSchema
- `/apis/metal3.io/v1alpha1/namespaces/{namespace}/firmwareschemas`
  - `DELETE`: delete collection of FirmwareSchema
  - `GET`: list objects of kind FirmwareSchema
  - `POST`: create a FirmwareSchema
- `/apis/metal3.io/v1alpha1/namespaces/{namespace}/firmwareschemas/{name}`
  - `DELETE`: delete a FirmwareSchema
  - `GET`: read the specified FirmwareSchema
  - `PATCH`: partially update the specified FirmwareSchema
  - `PUT`: replace the specified FirmwareSchema

### /apis/metal3.io/v1alpha1/firmwareschemas {#_apismetal3iov1alpha1firmwareschemas}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  list objects of kind FirmwareSchema
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`FirmwareSchemaList`](/docs/rest_api/objects#io-metal3-v1alpha1-FirmwareSchemaList) schema |
| 401 - Unauthorized | Empty |

### /apis/metal3.io/v1alpha1/namespaces/{namespace}/firmwareschemas {#_apismetal3iov1alpha1namespaces_namespace_firmwareschemas}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of FirmwareSchema
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
  list objects of kind FirmwareSchema
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`FirmwareSchemaList`](/docs/rest_api/objects#io-metal3-v1alpha1-FirmwareSchemaList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a FirmwareSchema
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`FirmwareSchema`](/docs/rest_api/provisioning_apis/firmwareschema-metal3-io-v1alpha1#firmwareschema-metal3-io-v1alpha1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`FirmwareSchema`](/docs/rest_api/provisioning_apis/firmwareschema-metal3-io-v1alpha1#firmwareschema-metal3-io-v1alpha1) schema |
| 201 - Created | [`FirmwareSchema`](/docs/rest_api/provisioning_apis/firmwareschema-metal3-io-v1alpha1#firmwareschema-metal3-io-v1alpha1) schema |
| 202 - Accepted | [`FirmwareSchema`](/docs/rest_api/provisioning_apis/firmwareschema-metal3-io-v1alpha1#firmwareschema-metal3-io-v1alpha1) schema |
| 401 - Unauthorized | Empty |

### /apis/metal3.io/v1alpha1/namespaces/{namespace}/firmwareschemas/{name} {#_apismetal3iov1alpha1namespaces_namespace_firmwareschemas_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the FirmwareSchema |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete a FirmwareSchema
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
  read the specified FirmwareSchema
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`FirmwareSchema`](/docs/rest_api/provisioning_apis/firmwareschema-metal3-io-v1alpha1#firmwareschema-metal3-io-v1alpha1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified FirmwareSchema
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`FirmwareSchema`](/docs/rest_api/provisioning_apis/firmwareschema-metal3-io-v1alpha1#firmwareschema-metal3-io-v1alpha1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified FirmwareSchema
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`FirmwareSchema`](/docs/rest_api/provisioning_apis/firmwareschema-metal3-io-v1alpha1#firmwareschema-metal3-io-v1alpha1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`FirmwareSchema`](/docs/rest_api/provisioning_apis/firmwareschema-metal3-io-v1alpha1#firmwareschema-metal3-io-v1alpha1) schema |
| 201 - Created | [`FirmwareSchema`](/docs/rest_api/provisioning_apis/firmwareschema-metal3-io-v1alpha1#firmwareschema-metal3-io-v1alpha1) schema |
| 401 - Unauthorized | Empty |
