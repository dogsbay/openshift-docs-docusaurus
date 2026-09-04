---
title: "ClusterRole [rbac.authorization.k8s.io/v1]"
sidebar_position: 3
---

# ClusterRole [rbac.authorization.k8s.io/v1] {#clusterrole-rbac-authorization-k8s-io-v1}

<a id="clusterrole-rbac-authorization-k8s-io-v1"></a>

Description
: ClusterRole is a cluster level, logical grouping of PolicyRules that can be referenced as a unit by a RoleBinding or ClusterRoleBinding.

Type
: ```
  `object`
  ```

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `aggregationRule` | `object` | AggregationRule describes how to locate ClusterRoles to aggregate into the ClusterRole |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | Standard object’s metadata. |
| `rules` | `array` | Rules holds all the PolicyRules for this ClusterRole |
| `rules[]` | `object` | PolicyRule holds information that describes a policy rule, but does not contain information about who the rule applies to or which namespace the rule applies to. |

### .aggregationRule {#_aggregationrule}

Description
: AggregationRule describes how to locate ClusterRoles to aggregate into the ClusterRole

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `clusterRoleSelectors` | [`array (LabelSelector)`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-LabelSelector) | ClusterRoleSelectors holds a list of selectors which will be used to find ClusterRoles and create the rules. If any of the selectors match, then the ClusterRole’s permissions will be added |

### .rules {#_rules}

Description
: Rules holds all the PolicyRules for this ClusterRole

Type
: ```
  `array`
  ```

### .rules[] {#_rules}

Description
: PolicyRule holds information that describes a policy rule, but does not contain information about who the rule applies to or which namespace the rule applies to.

Type
: ```
  `object`
  ```

Required
: - `verbs`

| Property | Type | Description |
| --- | --- | --- |
| `apiGroups` | `array (string)` | APIGroups is the name of the APIGroup that contains the resources.  If multiple API groups are specified, any action requested against one of the enumerated resources in any API group will be allowed. "" represents the core API group and "*" represents all API groups. |
| `nonResourceURLs` | `array (string)` | NonResourceURLs is a set of partial urls that a user should have access to.  *s are allowed, but only as the full, final step in the path Since non-resource URLs are not namespaced, this field is only applicable for ClusterRoles referenced from a ClusterRoleBinding. Rules can either apply to API resources (such as "pods" or "secrets") or non-resource URL paths (such as "/api"),  but not both. |
| `resourceNames` | `array (string)` | ResourceNames is an optional white list of names that the rule applies to.  An empty set means that everything is allowed. |
| `resources` | `array (string)` | Resources is a list of resources this rule applies to. '*' represents all resources. |
| `verbs` | `array (string)` | Verbs is a list of Verbs that apply to ALL the ResourceKinds contained in this rule. '*' represents all verbs. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/rbac.authorization.k8s.io/v1/clusterroles`
  - `DELETE`: delete collection of ClusterRole
  - `GET`: list or watch objects of kind ClusterRole
  - `POST`: create a ClusterRole
- `/apis/rbac.authorization.k8s.io/v1/watch/clusterroles`
  - `GET`: watch individual changes to a list of ClusterRole. deprecated: use the 'watch' parameter with a list operation instead.
- `/apis/rbac.authorization.k8s.io/v1/clusterroles/{name}`
  - `DELETE`: delete a ClusterRole
  - `GET`: read the specified ClusterRole
  - `PATCH`: partially update the specified ClusterRole
  - `PUT`: replace the specified ClusterRole
- `/apis/rbac.authorization.k8s.io/v1/watch/clusterroles/{name}`
  - `GET`: watch changes to an object of kind ClusterRole. deprecated: use the 'watch' parameter with a list operation instead, filtered to a single item with the 'fieldSelector' parameter.

### /apis/rbac.authorization.k8s.io/v1/clusterroles {#_apisrbacauthorizationk8siov1clusterroles}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of ClusterRole
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
  list or watch objects of kind ClusterRole
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ClusterRoleList`](/docs/rest_api/objects#io-k8s-api-rbac-v1-ClusterRoleList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a ClusterRole
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`ClusterRole`](/docs/rest_api/rbac_apis/clusterrole-rbac-authorization-k8s-io-v1#clusterrole-rbac-authorization-k8s-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ClusterRole`](/docs/rest_api/rbac_apis/clusterrole-rbac-authorization-k8s-io-v1#clusterrole-rbac-authorization-k8s-io-v1) schema |
| 201 - Created | [`ClusterRole`](/docs/rest_api/rbac_apis/clusterrole-rbac-authorization-k8s-io-v1#clusterrole-rbac-authorization-k8s-io-v1) schema |
| 202 - Accepted | [`ClusterRole`](/docs/rest_api/rbac_apis/clusterrole-rbac-authorization-k8s-io-v1#clusterrole-rbac-authorization-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/rbac.authorization.k8s.io/v1/watch/clusterroles {#_apisrbacauthorizationk8siov1watchclusterroles}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch individual changes to a list of ClusterRole. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |

### /apis/rbac.authorization.k8s.io/v1/clusterroles/{name} {#_apisrbacauthorizationk8siov1clusterroles_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the ClusterRole |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete a ClusterRole
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
  read the specified ClusterRole
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ClusterRole`](/docs/rest_api/rbac_apis/clusterrole-rbac-authorization-k8s-io-v1#clusterrole-rbac-authorization-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified ClusterRole
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ClusterRole`](/docs/rest_api/rbac_apis/clusterrole-rbac-authorization-k8s-io-v1#clusterrole-rbac-authorization-k8s-io-v1) schema |
| 201 - Created | [`ClusterRole`](/docs/rest_api/rbac_apis/clusterrole-rbac-authorization-k8s-io-v1#clusterrole-rbac-authorization-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified ClusterRole
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`ClusterRole`](/docs/rest_api/rbac_apis/clusterrole-rbac-authorization-k8s-io-v1#clusterrole-rbac-authorization-k8s-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ClusterRole`](/docs/rest_api/rbac_apis/clusterrole-rbac-authorization-k8s-io-v1#clusterrole-rbac-authorization-k8s-io-v1) schema |
| 201 - Created | [`ClusterRole`](/docs/rest_api/rbac_apis/clusterrole-rbac-authorization-k8s-io-v1#clusterrole-rbac-authorization-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/rbac.authorization.k8s.io/v1/watch/clusterroles/{name} {#_apisrbacauthorizationk8siov1watchclusterroles_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the ClusterRole |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch changes to an object of kind ClusterRole. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead, filtered to a single item with the &#x27;fieldSelector&#x27; parameter.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |
