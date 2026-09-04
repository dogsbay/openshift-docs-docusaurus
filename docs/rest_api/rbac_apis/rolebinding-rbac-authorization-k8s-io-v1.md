---
title: "RoleBinding [rbac.authorization.k8s.io/v1]"
sidebar_position: 4
---

# RoleBinding [rbac.authorization.k8s.io/v1] {#rolebinding-rbac-authorization-k8s-io-v1}

<a id="rolebinding-rbac-authorization-k8s-io-v1"></a>

Description
: RoleBinding references a role, but does not contain it.  It can reference a Role in the same namespace or a ClusterRole in the global namespace. It adds who information via Subjects and namespace information by which namespace it exists in.  RoleBindings in a given namespace only have effect in that namespace.

Type
: ```
  `object`
  ```

Required
: - `roleRef`

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | Standard object’s metadata. |
| `roleRef` | `object` | RoleRef contains information that points to the role being used |
| `subjects` | `array` | Subjects holds references to the objects the role applies to. |
| `subjects[]` | `object` | Subject contains a reference to the object or user identities a role binding applies to.  This can either hold a direct API object reference, or a value for non-objects such as user and group names. |

### .roleRef {#_roleref}

Description
: RoleRef contains information that points to the role being used

Type
: ```
  `object`
  ```

Required
: - `apiGroup`
  - `kind`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `apiGroup` | `string` | APIGroup is the group for the resource being referenced |
| `kind` | `string` | Kind is the type of resource being referenced |
| `name` | `string` | Name is the name of resource being referenced |

### .subjects {#_subjects}

Description
: Subjects holds references to the objects the role applies to.

Type
: ```
  `array`
  ```

### .subjects[] {#_subjects}

Description
: Subject contains a reference to the object or user identities a role binding applies to.  This can either hold a direct API object reference, or a value for non-objects such as user and group names.

Type
: ```
  `object`
  ```

Required
: - `kind`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `apiGroup` | `string` | APIGroup holds the API group of the referenced subject. Defaults to "" for ServiceAccount subjects. Defaults to "rbac.authorization.k8s.io" for User and Group subjects. |
| `kind` | `string` | Kind of object being referenced. Values defined by this API group are "User", "Group", and "ServiceAccount". If the Authorizer does not recognized the kind value, the Authorizer should report an error. |
| `name` | `string` | Name of the object being referenced. |
| `namespace` | `string` | Namespace of the referenced object.  If the object kind is non-namespace, such as "User" or "Group", and this value is not empty the Authorizer should report an error. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/rbac.authorization.k8s.io/v1/rolebindings`
  - `GET`: list or watch objects of kind RoleBinding
- `/apis/rbac.authorization.k8s.io/v1/watch/rolebindings`
  - `GET`: watch individual changes to a list of RoleBinding. deprecated: use the 'watch' parameter with a list operation instead.
- `/apis/rbac.authorization.k8s.io/v1/namespaces/{namespace}/rolebindings`
  - `DELETE`: delete collection of RoleBinding
  - `GET`: list or watch objects of kind RoleBinding
  - `POST`: create a RoleBinding
- `/apis/rbac.authorization.k8s.io/v1/watch/namespaces/{namespace}/rolebindings`
  - `GET`: watch individual changes to a list of RoleBinding. deprecated: use the 'watch' parameter with a list operation instead.
- `/apis/rbac.authorization.k8s.io/v1/namespaces/{namespace}/rolebindings/{name}`
  - `DELETE`: delete a RoleBinding
  - `GET`: read the specified RoleBinding
  - `PATCH`: partially update the specified RoleBinding
  - `PUT`: replace the specified RoleBinding
- `/apis/rbac.authorization.k8s.io/v1/watch/namespaces/{namespace}/rolebindings/{name}`
  - `GET`: watch changes to an object of kind RoleBinding. deprecated: use the 'watch' parameter with a list operation instead, filtered to a single item with the 'fieldSelector' parameter.

### /apis/rbac.authorization.k8s.io/v1/rolebindings {#_apisrbacauthorizationk8siov1rolebindings}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  list or watch objects of kind RoleBinding
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`RoleBindingList`](/docs/rest_api/objects#io-k8s-api-rbac-v1-RoleBindingList) schema |
| 401 - Unauthorized | Empty |

### /apis/rbac.authorization.k8s.io/v1/watch/rolebindings {#_apisrbacauthorizationk8siov1watchrolebindings}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch individual changes to a list of RoleBinding. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |

### /apis/rbac.authorization.k8s.io/v1/namespaces/{namespace}/rolebindings {#_apisrbacauthorizationk8siov1namespaces_namespace_rolebindings}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of RoleBinding
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
  list or watch objects of kind RoleBinding
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`RoleBindingList`](/docs/rest_api/objects#io-k8s-api-rbac-v1-RoleBindingList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a RoleBinding
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`RoleBinding`](/docs/rest_api/rbac_apis/rolebinding-rbac-authorization-k8s-io-v1#rolebinding-rbac-authorization-k8s-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`RoleBinding`](/docs/rest_api/rbac_apis/rolebinding-rbac-authorization-k8s-io-v1#rolebinding-rbac-authorization-k8s-io-v1) schema |
| 201 - Created | [`RoleBinding`](/docs/rest_api/rbac_apis/rolebinding-rbac-authorization-k8s-io-v1#rolebinding-rbac-authorization-k8s-io-v1) schema |
| 202 - Accepted | [`RoleBinding`](/docs/rest_api/rbac_apis/rolebinding-rbac-authorization-k8s-io-v1#rolebinding-rbac-authorization-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/rbac.authorization.k8s.io/v1/watch/namespaces/{namespace}/rolebindings {#_apisrbacauthorizationk8siov1watchnamespaces_namespace_rolebindings}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch individual changes to a list of RoleBinding. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |

### /apis/rbac.authorization.k8s.io/v1/namespaces/{namespace}/rolebindings/{name} {#_apisrbacauthorizationk8siov1namespaces_namespace_rolebindings_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the RoleBinding |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete a RoleBinding
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
  read the specified RoleBinding
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`RoleBinding`](/docs/rest_api/rbac_apis/rolebinding-rbac-authorization-k8s-io-v1#rolebinding-rbac-authorization-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified RoleBinding
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`RoleBinding`](/docs/rest_api/rbac_apis/rolebinding-rbac-authorization-k8s-io-v1#rolebinding-rbac-authorization-k8s-io-v1) schema |
| 201 - Created | [`RoleBinding`](/docs/rest_api/rbac_apis/rolebinding-rbac-authorization-k8s-io-v1#rolebinding-rbac-authorization-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified RoleBinding
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`RoleBinding`](/docs/rest_api/rbac_apis/rolebinding-rbac-authorization-k8s-io-v1#rolebinding-rbac-authorization-k8s-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`RoleBinding`](/docs/rest_api/rbac_apis/rolebinding-rbac-authorization-k8s-io-v1#rolebinding-rbac-authorization-k8s-io-v1) schema |
| 201 - Created | [`RoleBinding`](/docs/rest_api/rbac_apis/rolebinding-rbac-authorization-k8s-io-v1#rolebinding-rbac-authorization-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/rbac.authorization.k8s.io/v1/watch/namespaces/{namespace}/rolebindings/{name} {#_apisrbacauthorizationk8siov1watchnamespaces_namespace_rolebindings_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the RoleBinding |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch changes to an object of kind RoleBinding. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead, filtered to a single item with the &#x27;fieldSelector&#x27; parameter.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |
