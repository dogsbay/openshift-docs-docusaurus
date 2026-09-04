---
title: "Console [config.openshift.io/v1]"
sidebar_position: 8
---

# Console [config.openshift.io/v1] {#console-config-openshift-io-v1}

<a id="console-config-openshift-io-v1"></a>

Description
: Console holds cluster-wide configuration for the web console, including the logout URL, and reports the public URL of the console. The canonical name is `cluster`.

  Compatibility level 1: Stable within a major release for a minimum of 12 months or 3 minor releases (whichever is longer).

Type
: ```
  `object`
  ```

Required
: - `spec`

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | Standard object’s metadata. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata |
| `spec` | `object` | spec holds user settable values for configuration |
| `status` | `object` | status holds observed values from the cluster. They may not be overridden. |

### .spec {#_spec}

Description
: spec holds user settable values for configuration

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `authentication` | `object` | ConsoleAuthentication defines a list of optional configuration for console authentication. |

### .spec.authentication {#_specauthentication}

Description
: ConsoleAuthentication defines a list of optional configuration for console authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `logoutRedirect` | `string` | An optional, absolute URL to redirect web browsers to after logging out of the console. If not specified, it will redirect to the default login page. This is required when using an identity provider that supports single sign-on (SSO) such as: - OpenID (Keycloak, Azure) - RequestHeader (GSSAPI, SSPI, SAML) - OAuth (GitHub, GitLab, Google) Logging out of the console will destroy the user’s token. The logoutRedirect provides the user the option to perform single logout (SLO) through the identity provider to destroy their single sign-on session. |

### .status {#_status}

Description
: status holds observed values from the cluster. They may not be overridden.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `consoleURL` | `string` | The URL for the console. This will be derived from the host for the route that is created for the console. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/config.openshift.io/v1/consoles`
  - `DELETE`: delete collection of Console
  - `GET`: list objects of kind Console
  - `POST`: create a Console
- `/apis/config.openshift.io/v1/consoles/{name}`
  - `DELETE`: delete a Console
  - `GET`: read the specified Console
  - `PATCH`: partially update the specified Console
  - `PUT`: replace the specified Console
- `/apis/config.openshift.io/v1/consoles/{name}/status`
  - `GET`: read status of the specified Console
  - `PATCH`: partially update status of the specified Console
  - `PUT`: replace status of the specified Console

### /apis/config.openshift.io/v1/consoles {#_apisconfigopenshiftiov1consoles}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of Console
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
  list objects of kind Console
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ConsoleList`](/docs/rest_api/objects#io-openshift-config-v1-ConsoleList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a Console
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`Console`](/docs/rest_api/config_apis/console-config-openshift-io-v1#console-config-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Console`](/docs/rest_api/config_apis/console-config-openshift-io-v1#console-config-openshift-io-v1) schema |
| 201 - Created | [`Console`](/docs/rest_api/config_apis/console-config-openshift-io-v1#console-config-openshift-io-v1) schema |
| 202 - Accepted | [`Console`](/docs/rest_api/config_apis/console-config-openshift-io-v1#console-config-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/config.openshift.io/v1/consoles/{name} {#_apisconfigopenshiftiov1consoles_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the Console |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete a Console
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
  read the specified Console
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Console`](/docs/rest_api/config_apis/console-config-openshift-io-v1#console-config-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified Console
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Console`](/docs/rest_api/config_apis/console-config-openshift-io-v1#console-config-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified Console
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`Console`](/docs/rest_api/config_apis/console-config-openshift-io-v1#console-config-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Console`](/docs/rest_api/config_apis/console-config-openshift-io-v1#console-config-openshift-io-v1) schema |
| 201 - Created | [`Console`](/docs/rest_api/config_apis/console-config-openshift-io-v1#console-config-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/config.openshift.io/v1/consoles/{name}/status {#_apisconfigopenshiftiov1consoles_name_status}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the Console |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read status of the specified Console
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Console`](/docs/rest_api/config_apis/console-config-openshift-io-v1#console-config-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update status of the specified Console
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Console`](/docs/rest_api/config_apis/console-config-openshift-io-v1#console-config-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace status of the specified Console
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`Console`](/docs/rest_api/config_apis/console-config-openshift-io-v1#console-config-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Console`](/docs/rest_api/config_apis/console-config-openshift-io-v1#console-config-openshift-io-v1) schema |
| 201 - Created | [`Console`](/docs/rest_api/config_apis/console-config-openshift-io-v1#console-config-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |
