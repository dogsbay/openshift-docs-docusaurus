---
title: "Identity [user.openshift.io/v1]"
sidebar_position: 3
---

# Identity [user.openshift.io/v1] {#identity-user-openshift-io-v1}

<a id="identity-user-openshift-io-v1"></a>

Description
: Identity records a successful authentication of a user with an identity provider. The information about the source of authentication is stored on the identity, and the identity is then associated with a single user object. Multiple identities can reference a single user. Information retrieved from the authentication provider is stored in the extra field using a schema determined by the provider.

  Compatibility level 1: Stable within a major release for a minimum of 12 months or 3 minor releases (whichever is longer).

Type
: ```
  `object`
  ```

Required
: - `providerName`
  - `providerUserName`
  - `user`

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `extra` | `object (string)` | extra holds extra information about this identity |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | metadata is the standard object’s metadata. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata |
| `providerName` | `string` | providerName is the source of identity information |
| `providerUserName` | `string` | providerUserName uniquely represents this identity in the scope of the provider |
| `user` | [`ObjectReference`](/docs/rest_api/objects#io-k8s-api-core-v1-ObjectReference) | user is a reference to the user this identity is associated with Both Name and UID must be set |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/user.openshift.io/v1/identities`
  - `DELETE`: delete collection of Identity
  - `GET`: list or watch objects of kind Identity
  - `POST`: create an Identity
- `/apis/user.openshift.io/v1/watch/identities`
  - `GET`: watch individual changes to a list of Identity. deprecated: use the 'watch' parameter with a list operation instead.
- `/apis/user.openshift.io/v1/identities/{name}`
  - `DELETE`: delete an Identity
  - `GET`: read the specified Identity
  - `PATCH`: partially update the specified Identity
  - `PUT`: replace the specified Identity
- `/apis/user.openshift.io/v1/watch/identities/{name}`
  - `GET`: watch changes to an object of kind Identity. deprecated: use the 'watch' parameter with a list operation instead, filtered to a single item with the 'fieldSelector' parameter.

### /apis/user.openshift.io/v1/identities {#_apisuseropenshiftiov1identities}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of Identity
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
  list or watch objects of kind Identity
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`IdentityList`](/docs/rest_api/objects#com-github-openshift-api-user-v1-IdentityList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create an Identity
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`Identity`](/docs/rest_api/user_and_group_apis/identity-user-openshift-io-v1#identity-user-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Identity`](/docs/rest_api/user_and_group_apis/identity-user-openshift-io-v1#identity-user-openshift-io-v1) schema |
| 201 - Created | [`Identity`](/docs/rest_api/user_and_group_apis/identity-user-openshift-io-v1#identity-user-openshift-io-v1) schema |
| 202 - Accepted | [`Identity`](/docs/rest_api/user_and_group_apis/identity-user-openshift-io-v1#identity-user-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/user.openshift.io/v1/watch/identities {#_apisuseropenshiftiov1watchidentities}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch individual changes to a list of Identity. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |

### /apis/user.openshift.io/v1/identities/{name} {#_apisuseropenshiftiov1identities_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the Identity |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete an Identity
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
  read the specified Identity
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Identity`](/docs/rest_api/user_and_group_apis/identity-user-openshift-io-v1#identity-user-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified Identity
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Identity`](/docs/rest_api/user_and_group_apis/identity-user-openshift-io-v1#identity-user-openshift-io-v1) schema |
| 201 - Created | [`Identity`](/docs/rest_api/user_and_group_apis/identity-user-openshift-io-v1#identity-user-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified Identity
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`Identity`](/docs/rest_api/user_and_group_apis/identity-user-openshift-io-v1#identity-user-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Identity`](/docs/rest_api/user_and_group_apis/identity-user-openshift-io-v1#identity-user-openshift-io-v1) schema |
| 201 - Created | [`Identity`](/docs/rest_api/user_and_group_apis/identity-user-openshift-io-v1#identity-user-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/user.openshift.io/v1/watch/identities/{name} {#_apisuseropenshiftiov1watchidentities_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the Identity |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch changes to an object of kind Identity. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead, filtered to a single item with the &#x27;fieldSelector&#x27; parameter.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |
