---
title: "ComponentStatus [v1]"
sidebar_position: 4
---

# ComponentStatus [v1]

<a id="componentstatus-v1"></a>

Description
: ComponentStatus (and ComponentStatusList) holds the cluster validation info. Deprecated: This API is deprecated in v1.19+

Type
: ```
  `object`
  ```

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `conditions` | `array` | List of component conditions observed |
| `conditions[]` | `object` | Information about the condition of a component. |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | Standard object’s metadata. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata |

### .conditions {#_conditions}

Description
: List of component conditions observed

Type
: ```
  `array`
  ```

### .conditions[] {#_conditions}

Description
: Information about the condition of a component.

Type
: ```
  `object`
  ```

Required
: - `type`
  - `status`

| Property | Type | Description |
| --- | --- | --- |
| `error` | `string` | Condition error code for a component. For example, a health check error code. |
| `message` | `string` | Message about the condition for a component. For example, information about a health check. |
| `status` | `string` | Status of the condition for a component. Valid values for "Healthy": "True", "False", or "Unknown". |
| `type` | `string` | Type of condition for a component. Valid value: "Healthy" |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/api/v1/componentstatuses`
  - `GET`: list objects of kind ComponentStatus
- `/api/v1/componentstatuses/{name}`
  - `GET`: read the specified ComponentStatus

### /api/v1/componentstatuses {#_apiv1componentstatuses}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  list objects of kind ComponentStatus
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ComponentStatusList`](/docs/rest_api/objects#io-k8s-api-core-v1-ComponentStatusList) schema |
| 401 - Unauthorized | Empty |

### /api/v1/componentstatuses/{name} {#_apiv1componentstatuses_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the ComponentStatus |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read the specified ComponentStatus
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ComponentStatus`](/docs/rest_api/metadata_apis/componentstatus-v1#componentstatus-v1) schema |
| 401 - Unauthorized | Empty |
