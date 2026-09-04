---
title: "OAuthClient [oauth.openshift.io/v1]"
sidebar_position: 5
---

# OAuthClient [oauth.openshift.io/v1] {#oauthclient-oauth-openshift-io-v1}

<a id="oauthclient-oauth-openshift-io-v1"></a>

Description
: OAuthClient describes an OAuth client

  Compatibility level 1: Stable within a major release for a minimum of 12 months or 3 minor releases (whichever is longer).

Type
: ```
  `object`
  ```

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `accessTokenInactivityTimeoutSeconds` | `integer` | accessTokenInactivityTimeoutSeconds overrides the default token inactivity timeout for tokens granted to this client. The value represents the maximum amount of time that can occur between consecutive uses of the token. Tokens become invalid if they are not used within this temporal window. The user will need to acquire a new token to regain access once a token times out. This value needs to be set only if the default set in configuration is not appropriate for this client. Valid values are: - 0: Tokens for this client never time out - X: Tokens time out if there is no activity for X seconds The current minimum allowed value for X is 300 (5 minutes)<br>WARNING: existing tokens' timeout will not be affected (lowered) by changing this value |
| `accessTokenMaxAgeSeconds` | `integer` | accessTokenMaxAgeSeconds overrides the default access token max age for tokens granted to this client. 0 means no expiration. |
| `additionalSecrets` | `array (string)` | additionalSecrets holds other secrets that may be used to identify the client.  This is useful for rotation and for service account token validation |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `grantMethod` | `string` | grantMethod is a required field which determines how to handle grants for this client. Valid grant handling methods are:  - auto:   always approves grant requests, useful for trusted clients  - prompt: prompts the end user for approval of grant requests, useful for third-party clients |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | metadata is the standard object’s metadata. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata |
| `redirectURIs` | `array (string)` | redirectURIs is the valid redirection URIs associated with a client |
| `respondWithChallenges` | `boolean` | respondWithChallenges indicates whether the client wants authentication needed responses made in the form of challenges instead of redirects |
| `scopeRestrictions` | `array` | scopeRestrictions describes which scopes this client can request.  Each requested scope is checked against each restriction.  If any restriction matches, then the scope is allowed. If no restriction matches, then the scope is denied. |
| `scopeRestrictions[]` | `object` | ScopeRestriction describe one restriction on scopes.  Exactly one option must be non-nil. |
| `secret` | `string` | secret is the unique secret associated with a client |

### .scopeRestrictions {#_scoperestrictions}

Description
: scopeRestrictions describes which scopes this client can request.  Each requested scope is checked against each restriction.  If any restriction matches, then the scope is allowed. If no restriction matches, then the scope is denied.

Type
: ```
  `array`
  ```

### .scopeRestrictions[] {#_scoperestrictions}

Description
: ScopeRestriction describe one restriction on scopes.  Exactly one option must be non-nil.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `clusterRole` | `object` | ClusterRoleScopeRestriction describes restrictions on cluster role scopes |
| `literals` | `array (string)` | ExactValues means the scope has to match a particular set of strings exactly |

### .scopeRestrictions[].clusterRole {#_scoperestrictionsclusterrole}

Description
: ClusterRoleScopeRestriction describes restrictions on cluster role scopes

Type
: ```
  `object`
  ```

Required
: - `roleNames`
  - `namespaces`
  - `allowEscalation`

| Property | Type | Description |
| --- | --- | --- |
| `allowEscalation` | `boolean` | allowEscalation indicates whether you can request roles and their escalating resources |
| `namespaces` | `array (string)` | namespaces is the list of namespaces that can be referenced.  * means any of them (including *) |
| `roleNames` | `array (string)` | roleNames is the list of cluster roles that can referenced.  * means anything |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/oauth.openshift.io/v1/oauthclients`
  - `DELETE`: delete collection of OAuthClient
  - `GET`: list or watch objects of kind OAuthClient
  - `POST`: create an OAuthClient
- `/apis/oauth.openshift.io/v1/watch/oauthclients`
  - `GET`: watch individual changes to a list of OAuthClient. deprecated: use the 'watch' parameter with a list operation instead.
- `/apis/oauth.openshift.io/v1/oauthclients/{name}`
  - `DELETE`: delete an OAuthClient
  - `GET`: read the specified OAuthClient
  - `PATCH`: partially update the specified OAuthClient
  - `PUT`: replace the specified OAuthClient
- `/apis/oauth.openshift.io/v1/watch/oauthclients/{name}`
  - `GET`: watch changes to an object of kind OAuthClient. deprecated: use the 'watch' parameter with a list operation instead, filtered to a single item with the 'fieldSelector' parameter.

### /apis/oauth.openshift.io/v1/oauthclients {#_apisoauthopenshiftiov1oauthclients}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of OAuthClient
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Status_v2`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-Status_v2) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  list or watch objects of kind OAuthClient
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`OAuthClientList`](/docs/rest_api/objects#com-github-openshift-api-oauth-v1-OAuthClientList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create an OAuthClient
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`OAuthClient`](/docs/rest_api/oauth_apis/oauthclient-oauth-openshift-io-v1#oauthclient-oauth-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`OAuthClient`](/docs/rest_api/oauth_apis/oauthclient-oauth-openshift-io-v1#oauthclient-oauth-openshift-io-v1) schema |
| 201 - Created | [`OAuthClient`](/docs/rest_api/oauth_apis/oauthclient-oauth-openshift-io-v1#oauthclient-oauth-openshift-io-v1) schema |
| 202 - Accepted | [`OAuthClient`](/docs/rest_api/oauth_apis/oauthclient-oauth-openshift-io-v1#oauthclient-oauth-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/oauth.openshift.io/v1/watch/oauthclients {#_apisoauthopenshiftiov1watchoauthclients}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch individual changes to a list of OAuthClient. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |

### /apis/oauth.openshift.io/v1/oauthclients/{name} {#_apisoauthopenshiftiov1oauthclients_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the OAuthClient |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete an OAuthClient
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Status_v2`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-Status_v2) schema |
| 202 - Accepted | [`Status_v2`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-Status_v2) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read the specified OAuthClient
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`OAuthClient`](/docs/rest_api/oauth_apis/oauthclient-oauth-openshift-io-v1#oauthclient-oauth-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified OAuthClient
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`OAuthClient`](/docs/rest_api/oauth_apis/oauthclient-oauth-openshift-io-v1#oauthclient-oauth-openshift-io-v1) schema |
| 201 - Created | [`OAuthClient`](/docs/rest_api/oauth_apis/oauthclient-oauth-openshift-io-v1#oauthclient-oauth-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified OAuthClient
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`OAuthClient`](/docs/rest_api/oauth_apis/oauthclient-oauth-openshift-io-v1#oauthclient-oauth-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`OAuthClient`](/docs/rest_api/oauth_apis/oauthclient-oauth-openshift-io-v1#oauthclient-oauth-openshift-io-v1) schema |
| 201 - Created | [`OAuthClient`](/docs/rest_api/oauth_apis/oauthclient-oauth-openshift-io-v1#oauthclient-oauth-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/oauth.openshift.io/v1/watch/oauthclients/{name} {#_apisoauthopenshiftiov1watchoauthclients_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the OAuthClient |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch changes to an object of kind OAuthClient. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead, filtered to a single item with the &#x27;fieldSelector&#x27; parameter.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |
