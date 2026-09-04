---
title: "TokenRequest [authentication.k8s.io/v1]"
sidebar_position: 9
---

# TokenRequest [authentication.k8s.io/v1] {#tokenrequest-authentication-k8s-io-v1}

<a id="tokenrequest-authentication-k8s-io-v1"></a>

Description
: TokenRequest requests a token for a given service account.

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
| `spec` | `object` | TokenRequestSpec contains client provided parameters of a token request. |
| `status` | `object` | TokenRequestStatus is the result of a token request. |

### .spec {#_spec}

Description
: TokenRequestSpec contains client provided parameters of a token request.

Type
: ```
  `object`
  ```

Required
: - `audiences`

| Property | Type | Description |
| --- | --- | --- |
| `audiences` | `array (string)` | Audiences are the intendend audiences of the token. A recipient of a token must identify themself with an identifier in the list of audiences of the token, and otherwise should reject the token. A token issued for multiple audiences may be used to authenticate against any of the audiences listed but implies a high degree of trust between the target audiences. |
| `boundObjectRef` | `object` | BoundObjectReference is a reference to an object that a token is bound to. |
| `expirationSeconds` | `integer` | ExpirationSeconds is the requested duration of validity of the request. The token issuer may return a token with a different validity duration so a client needs to check the 'expiration' field in a response. |

### .spec.boundObjectRef {#_specboundobjectref}

Description
: BoundObjectReference is a reference to an object that a token is bound to.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | API version of the referent. |
| `kind` | `string` | Kind of the referent. Valid kinds are 'Pod' and 'Secret'. |
| `name` | `string` | Name of the referent. |
| `uid` | `string` | UID of the referent. |

### .status {#_status}

Description
: TokenRequestStatus is the result of a token request.

Type
: ```
  `object`
  ```

Required
: - `token`
  - `expirationTimestamp`

| Property | Type | Description |
| --- | --- | --- |
| `expirationTimestamp` | [`Time`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-Time) | ExpirationTimestamp is the time of expiration of the returned token. |
| `token` | `string` | Token is the opaque bearer token. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/api/v1/namespaces/{namespace}/serviceaccounts/{name}/token`
  - `POST`: create token of a ServiceAccount

### /api/v1/namespaces/{namespace}/serviceaccounts/{name}/token {#_apiv1namespaces_namespace_serviceaccounts_name_token}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the TokenRequest |

**Global query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create token of a ServiceAccount
  ```

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`TokenRequest`](/docs/rest_api/authorization_apis/tokenrequest-authentication-k8s-io-v1#tokenrequest-authentication-k8s-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`TokenRequest`](/docs/rest_api/authorization_apis/tokenrequest-authentication-k8s-io-v1#tokenrequest-authentication-k8s-io-v1) schema |
| 201 - Created | [`TokenRequest`](/docs/rest_api/authorization_apis/tokenrequest-authentication-k8s-io-v1#tokenrequest-authentication-k8s-io-v1) schema |
| 202 - Accepted | [`TokenRequest`](/docs/rest_api/authorization_apis/tokenrequest-authentication-k8s-io-v1#tokenrequest-authentication-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |
