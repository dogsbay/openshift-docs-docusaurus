---
title: "Authentication [operator.openshift.io/v1]"
sidebar_position: 2
---

# Authentication [operator.openshift.io/v1] {#authentication-operator-openshift-io-v1}

<a id="authentication-operator-openshift-io-v1"></a>

Description
: Authentication provides information to configure an operator to manage authentication.

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
| `spec` | `object` |  |
| `status` | `object` |  |

### .spec {#_spec}

Description

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `logLevel` | `string` | logLevel is an intent based logging for an overall component.  It does not give fine grained control, but it is a simple way to manage coarse grained logging choices that operators have to interpret for their operands.<br>Valid values are: "Normal", "Debug", "Trace", "TraceAll". Defaults to "Normal". |
| `managementState` | `string` | managementState indicates whether and how the operator should manage the component |
| `observedConfig` | `` | observedConfig holds a sparse config that controller has observed from the cluster state.  It exists in spec because it is an input to the level for the operator |
| `operatorLogLevel` | `string` | operatorLogLevel is an intent based logging for the operator itself.  It does not give fine grained control, but it is a simple way to manage coarse grained logging choices that operators have to interpret for themselves.<br>Valid values are: "Normal", "Debug", "Trace", "TraceAll". Defaults to "Normal". |
| `unsupportedConfigOverrides` | `` | unsupportedConfigOverrides overrides the final configuration that was computed by the operator. Red Hat does not support the use of this field. Misuse of this field could lead to unexpected behavior or conflict with other configuration options. Seek guidance from the Red Hat support before using this field. Use of this property blocks cluster upgrades, it must be removed before upgrading your cluster. |

### .status {#_status}

Description

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `conditions` | `array` | conditions is a list of conditions and their status |
| `conditions[]` | `object` | OperatorCondition is just the standard condition fields. |
| `generations` | `array` | generations are used to determine when an item needs to be reconciled or has changed in a way that needs a reaction. |
| `generations[]` | `object` | GenerationStatus keeps track of the generation for a given resource so that decisions about forced updates can be made. |
| `latestAvailableRevision` | `integer` | latestAvailableRevision is the deploymentID of the most recent deployment |
| `oauthAPIServer` | `object` | oauthAPIServer holds status specific only to oauth-apiserver |
| `observedGeneration` | `integer` | observedGeneration is the last generation change you’ve dealt with |
| `readyReplicas` | `integer` | readyReplicas indicates how many replicas are ready and at the desired state |
| `version` | `string` | version is the level this availability applies to |

### .status.conditions {#_statusconditions}

Description
: conditions is a list of conditions and their status

Type
: ```
  `array`
  ```

### .status.conditions[] {#_statusconditions}

Description
: OperatorCondition is just the standard condition fields.

Type
: ```
  `object`
  ```

Required
: - `lastTransitionTime`
  - `status`
  - `type`

| Property | Type | Description |
| --- | --- | --- |
| `lastTransitionTime` | `string` | lastTransitionTime is the last time the condition transitioned from one status to another. This should be when the underlying condition changed.  If that is not known, then using the time when the API field changed is acceptable. |
| `message` | `string` |  |
| `reason` | `string` |  |
| `status` | `string` | status of the condition, one of True, False, Unknown. |
| `type` | `string` | type of condition in CamelCase or in foo.example.com/CamelCase. |

### .status.generations {#_statusgenerations}

Description
: generations are used to determine when an item needs to be reconciled or has changed in a way that needs a reaction.

Type
: ```
  `array`
  ```

### .status.generations[] {#_statusgenerations}

Description
: GenerationStatus keeps track of the generation for a given resource so that decisions about forced updates can be made.

Type
: ```
  `object`
  ```

Required
: - `group`
  - `name`
  - `namespace`
  - `resource`

| Property | Type | Description |
| --- | --- | --- |
| `group` | `string` | group is the group of the thing you’re tracking |
| `hash` | `string` | hash is an optional field set for resources without generation that are content sensitive like secrets and configmaps |
| `lastGeneration` | `integer` | lastGeneration is the last generation of the workload controller involved |
| `name` | `string` | name is the name of the thing you’re tracking |
| `namespace` | `string` | namespace is where the thing you’re tracking is |
| `resource` | `string` | resource is the resource type of the thing you’re tracking |

### .status.oauthAPIServer {#_statusoauthapiserver}

Description
: oauthAPIServer holds status specific only to oauth-apiserver

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `latestAvailableRevision` | `integer` | latestAvailableRevision is the latest revision used as suffix of revisioned secrets like encryption-config. A new revision causes a new deployment of pods. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/operator.openshift.io/v1/authentications`
  - `DELETE`: delete collection of Authentication
  - `GET`: list objects of kind Authentication
  - `POST`: create an Authentication
- `/apis/operator.openshift.io/v1/authentications/{name}`
  - `DELETE`: delete an Authentication
  - `GET`: read the specified Authentication
  - `PATCH`: partially update the specified Authentication
  - `PUT`: replace the specified Authentication
- `/apis/operator.openshift.io/v1/authentications/{name}/status`
  - `GET`: read status of the specified Authentication
  - `PATCH`: partially update status of the specified Authentication
  - `PUT`: replace status of the specified Authentication

### /apis/operator.openshift.io/v1/authentications {#_apisoperatoropenshiftiov1authentications}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of Authentication
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
  list objects of kind Authentication
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`AuthenticationList`](/docs/rest_api/objects#io-openshift-operator-v1-AuthenticationList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create an Authentication
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`Authentication`](/docs/rest_api/operator_apis/authentication-operator-openshift-io-v1#authentication-operator-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Authentication`](/docs/rest_api/operator_apis/authentication-operator-openshift-io-v1#authentication-operator-openshift-io-v1) schema |
| 201 - Created | [`Authentication`](/docs/rest_api/operator_apis/authentication-operator-openshift-io-v1#authentication-operator-openshift-io-v1) schema |
| 202 - Accepted | [`Authentication`](/docs/rest_api/operator_apis/authentication-operator-openshift-io-v1#authentication-operator-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/operator.openshift.io/v1/authentications/{name} {#_apisoperatoropenshiftiov1authentications_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the Authentication |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete an Authentication
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
  read the specified Authentication
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Authentication`](/docs/rest_api/operator_apis/authentication-operator-openshift-io-v1#authentication-operator-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified Authentication
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Authentication`](/docs/rest_api/operator_apis/authentication-operator-openshift-io-v1#authentication-operator-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified Authentication
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`Authentication`](/docs/rest_api/operator_apis/authentication-operator-openshift-io-v1#authentication-operator-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Authentication`](/docs/rest_api/operator_apis/authentication-operator-openshift-io-v1#authentication-operator-openshift-io-v1) schema |
| 201 - Created | [`Authentication`](/docs/rest_api/operator_apis/authentication-operator-openshift-io-v1#authentication-operator-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/operator.openshift.io/v1/authentications/{name}/status {#_apisoperatoropenshiftiov1authentications_name_status}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the Authentication |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read status of the specified Authentication
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Authentication`](/docs/rest_api/operator_apis/authentication-operator-openshift-io-v1#authentication-operator-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update status of the specified Authentication
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Authentication`](/docs/rest_api/operator_apis/authentication-operator-openshift-io-v1#authentication-operator-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace status of the specified Authentication
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`Authentication`](/docs/rest_api/operator_apis/authentication-operator-openshift-io-v1#authentication-operator-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Authentication`](/docs/rest_api/operator_apis/authentication-operator-openshift-io-v1#authentication-operator-openshift-io-v1) schema |
| 201 - Created | [`Authentication`](/docs/rest_api/operator_apis/authentication-operator-openshift-io-v1#authentication-operator-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |
