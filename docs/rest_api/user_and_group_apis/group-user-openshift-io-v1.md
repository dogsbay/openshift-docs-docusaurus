---
title: "Group [user.openshift.io/v1]"
sidebar_position: 2
---

# Group [user.openshift.io/v1] {#group-user-openshift-io-v1}

<a id="group-user-openshift-io-v1"></a>

Description
: Group represents a referenceable set of Users

  Compatibility level 1: Stable within a major release for a minimum of 12 months or 3 minor releases (whichever is longer).

Type
: ```
  `object`
  ```

Required
: - `users`

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | metadata is the standard object’s metadata. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata |
| `users` | `array (string)` | users is the list of users in this group. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/user.openshift.io/v1/groups`
  - `DELETE`: delete collection of Group
  - `GET`: list or watch objects of kind Group
  - `POST`: create a Group
- `/apis/user.openshift.io/v1/watch/groups`
  - `GET`: watch individual changes to a list of Group. deprecated: use the 'watch' parameter with a list operation instead.
- `/apis/user.openshift.io/v1/groups/{name}`
  - `DELETE`: delete a Group
  - `GET`: read the specified Group
  - `PATCH`: partially update the specified Group
  - `PUT`: replace the specified Group
- `/apis/user.openshift.io/v1/watch/groups/{name}`
  - `GET`: watch changes to an object of kind Group. deprecated: use the 'watch' parameter with a list operation instead, filtered to a single item with the 'fieldSelector' parameter.

### /apis/user.openshift.io/v1/groups {#_apisuseropenshiftiov1groups}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of Group
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
  list or watch objects of kind Group
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`GroupList`](/docs/rest_api/objects#com-github-openshift-api-user-v1-GroupList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a Group
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`Group`](/docs/rest_api/user_and_group_apis/group-user-openshift-io-v1#group-user-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Group`](/docs/rest_api/user_and_group_apis/group-user-openshift-io-v1#group-user-openshift-io-v1) schema |
| 201 - Created | [`Group`](/docs/rest_api/user_and_group_apis/group-user-openshift-io-v1#group-user-openshift-io-v1) schema |
| 202 - Accepted | [`Group`](/docs/rest_api/user_and_group_apis/group-user-openshift-io-v1#group-user-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/user.openshift.io/v1/watch/groups {#_apisuseropenshiftiov1watchgroups}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch individual changes to a list of Group. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |

### /apis/user.openshift.io/v1/groups/{name} {#_apisuseropenshiftiov1groups_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the Group |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete a Group
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
  read the specified Group
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Group`](/docs/rest_api/user_and_group_apis/group-user-openshift-io-v1#group-user-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified Group
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Group`](/docs/rest_api/user_and_group_apis/group-user-openshift-io-v1#group-user-openshift-io-v1) schema |
| 201 - Created | [`Group`](/docs/rest_api/user_and_group_apis/group-user-openshift-io-v1#group-user-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified Group
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`Group`](/docs/rest_api/user_and_group_apis/group-user-openshift-io-v1#group-user-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Group`](/docs/rest_api/user_and_group_apis/group-user-openshift-io-v1#group-user-openshift-io-v1) schema |
| 201 - Created | [`Group`](/docs/rest_api/user_and_group_apis/group-user-openshift-io-v1#group-user-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/user.openshift.io/v1/watch/groups/{name} {#_apisuseropenshiftiov1watchgroups_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the Group |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch changes to an object of kind Group. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead, filtered to a single item with the &#x27;fieldSelector&#x27; parameter.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |
