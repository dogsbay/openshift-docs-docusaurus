---
title: "OperatorHub [config.openshift.io/v1]"
sidebar_position: 23
---

# OperatorHub [config.openshift.io/v1] {#operatorhub-config-openshift-io-v1}

<a id="operatorhub-config-openshift-io-v1"></a>

Description
: OperatorHub is the Schema for the operatorhubs API. It can be used to change the state of the default hub sources for OperatorHub on the cluster from enabled to disabled and vice versa.

  Compatibility level 1: Stable within a major release for a minimum of 12 months or 3 minor releases (whichever is longer).

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
| `spec` | `object` | OperatorHubSpec defines the desired state of OperatorHub |
| `status` | `object` | OperatorHubStatus defines the observed state of OperatorHub. The current state of the default hub sources will always be reflected here. |

### .spec {#_spec}

Description
: OperatorHubSpec defines the desired state of OperatorHub

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `disableAllDefaultSources` | `boolean` | disableAllDefaultSources allows you to disable all the default hub sources. If this is true, a specific entry in sources can be used to enable a default source. If this is false, a specific entry in sources can be used to disable or enable a default source. |
| `sources` | `array` | sources is the list of default hub sources and their configuration. If the list is empty, it implies that the default hub sources are enabled on the cluster unless disableAllDefaultSources is true. If disableAllDefaultSources is true and sources is not empty, the configuration present in sources will take precedence. The list of default hub sources and their current state will always be reflected in the status block. |
| `sources[]` | `object` | HubSource is used to specify the hub source and its configuration |

### .spec.sources {#_specsources}

Description
: sources is the list of default hub sources and their configuration. If the list is empty, it implies that the default hub sources are enabled on the cluster unless disableAllDefaultSources is true. If disableAllDefaultSources is true and sources is not empty, the configuration present in sources will take precedence. The list of default hub sources and their current state will always be reflected in the status block.

Type
: ```
  `array`
  ```

### .spec.sources[] {#_specsources}

Description
: HubSource is used to specify the hub source and its configuration

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `disabled` | `boolean` | disabled is used to disable a default hub source on cluster |
| `name` | `string` | name is the name of one of the default hub sources |

### .status {#_status}

Description
: OperatorHubStatus defines the observed state of OperatorHub. The current state of the default hub sources will always be reflected here.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `sources` | `array` | sources encapsulates the result of applying the configuration for each hub source |
| `sources[]` | `object` | HubSourceStatus is used to reflect the current state of applying the configuration to a default source |

### .status.sources {#_statussources}

Description
: sources encapsulates the result of applying the configuration for each hub source

Type
: ```
  `array`
  ```

### .status.sources[] {#_statussources}

Description
: HubSourceStatus is used to reflect the current state of applying the configuration to a default source

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `disabled` | `boolean` | disabled is used to disable a default hub source on cluster |
| `message` | `string` | message provides more information regarding failures |
| `name` | `string` | name is the name of one of the default hub sources |
| `status` | `string` | status indicates success or failure in applying the configuration |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/config.openshift.io/v1/operatorhubs`
  - `DELETE`: delete collection of OperatorHub
  - `GET`: list objects of kind OperatorHub
  - `POST`: create an OperatorHub
- `/apis/config.openshift.io/v1/operatorhubs/{name}`
  - `DELETE`: delete an OperatorHub
  - `GET`: read the specified OperatorHub
  - `PATCH`: partially update the specified OperatorHub
  - `PUT`: replace the specified OperatorHub
- `/apis/config.openshift.io/v1/operatorhubs/{name}/status`
  - `GET`: read status of the specified OperatorHub
  - `PATCH`: partially update status of the specified OperatorHub
  - `PUT`: replace status of the specified OperatorHub

### /apis/config.openshift.io/v1/operatorhubs {#_apisconfigopenshiftiov1operatorhubs}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of OperatorHub
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
  list objects of kind OperatorHub
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`OperatorHubList`](/docs/rest_api/objects#io-openshift-config-v1-OperatorHubList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create an OperatorHub
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`OperatorHub`](/docs/rest_api/config_apis/operatorhub-config-openshift-io-v1#operatorhub-config-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`OperatorHub`](/docs/rest_api/config_apis/operatorhub-config-openshift-io-v1#operatorhub-config-openshift-io-v1) schema |
| 201 - Created | [`OperatorHub`](/docs/rest_api/config_apis/operatorhub-config-openshift-io-v1#operatorhub-config-openshift-io-v1) schema |
| 202 - Accepted | [`OperatorHub`](/docs/rest_api/config_apis/operatorhub-config-openshift-io-v1#operatorhub-config-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/config.openshift.io/v1/operatorhubs/{name} {#_apisconfigopenshiftiov1operatorhubs_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the OperatorHub |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete an OperatorHub
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
  read the specified OperatorHub
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`OperatorHub`](/docs/rest_api/config_apis/operatorhub-config-openshift-io-v1#operatorhub-config-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified OperatorHub
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`OperatorHub`](/docs/rest_api/config_apis/operatorhub-config-openshift-io-v1#operatorhub-config-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified OperatorHub
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`OperatorHub`](/docs/rest_api/config_apis/operatorhub-config-openshift-io-v1#operatorhub-config-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`OperatorHub`](/docs/rest_api/config_apis/operatorhub-config-openshift-io-v1#operatorhub-config-openshift-io-v1) schema |
| 201 - Created | [`OperatorHub`](/docs/rest_api/config_apis/operatorhub-config-openshift-io-v1#operatorhub-config-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/config.openshift.io/v1/operatorhubs/{name}/status {#_apisconfigopenshiftiov1operatorhubs_name_status}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the OperatorHub |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read status of the specified OperatorHub
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`OperatorHub`](/docs/rest_api/config_apis/operatorhub-config-openshift-io-v1#operatorhub-config-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update status of the specified OperatorHub
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`OperatorHub`](/docs/rest_api/config_apis/operatorhub-config-openshift-io-v1#operatorhub-config-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace status of the specified OperatorHub
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`OperatorHub`](/docs/rest_api/config_apis/operatorhub-config-openshift-io-v1#operatorhub-config-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`OperatorHub`](/docs/rest_api/config_apis/operatorhub-config-openshift-io-v1#operatorhub-config-openshift-io-v1) schema |
| 201 - Created | [`OperatorHub`](/docs/rest_api/config_apis/operatorhub-config-openshift-io-v1#operatorhub-config-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |
