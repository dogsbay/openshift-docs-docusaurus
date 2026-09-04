---
title: "PodDisruptionBudget [policy/v1]"
sidebar_position: 3
---

# PodDisruptionBudget [policy/v1] {#poddisruptionbudget-policy-v1}

<a id="poddisruptionbudget-policy-v1"></a>

Description
: PodDisruptionBudget is an object to define the max disruption that can be caused to a collection of pods

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
| `spec` | `object` | PodDisruptionBudgetSpec is a description of a PodDisruptionBudget. |
| `status` | `object` | PodDisruptionBudgetStatus represents information about the status of a PodDisruptionBudget. Status may trail the actual state of a system. |

### .spec {#_spec}

Description
: PodDisruptionBudgetSpec is a description of a PodDisruptionBudget.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `maxUnavailable` | [`IntOrString`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-util-intstr-IntOrString) | An eviction is allowed if at most "maxUnavailable" pods selected by "selector" are unavailable after the eviction, i.e. even in absence of the evicted pod. For example, one can prevent all voluntary evictions by specifying 0. This is a mutually exclusive setting with "minAvailable". |
| `minAvailable` | [`IntOrString`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-util-intstr-IntOrString) | An eviction is allowed if at least "minAvailable" pods selected by "selector" will still be available after the eviction, i.e. even in the absence of the evicted pod.  So for example you can prevent all voluntary evictions by specifying "100%". |
| `selector` | [`LabelSelector`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-LabelSelector) | Label query over pods whose evictions are managed by the disruption budget. A null selector will match no pods, while an empty ({}) selector will select all pods within the namespace. |
| `unhealthyPodEvictionPolicy` | `string` | UnhealthyPodEvictionPolicy defines the criteria for when unhealthy pods should be considered for eviction. Current implementation considers healthy pods, as pods that have status.conditions item with type="Ready",status="True".<br>Valid policies are IfHealthyBudget and AlwaysAllow. If no policy is specified, the default behavior will be used, which corresponds to the IfHealthyBudget policy.<br>IfHealthyBudget policy means that running pods (status.phase="Running"), but not yet healthy can be evicted only if the guarded application is not disrupted (status.currentHealthy is at least equal to status.desiredHealthy). Healthy pods will be subject to the PDB for eviction.<br>AlwaysAllow policy means that all running pods (status.phase="Running"), but not yet healthy are considered disrupted and can be evicted regardless of whether the criteria in a PDB is met. This means perspective running pods of a disrupted application might not get a chance to become healthy. Healthy pods will be subject to the PDB for eviction.<br>Additional policies may be added in the future. Clients making eviction decisions should disallow eviction of unhealthy pods if they encounter an unrecognized policy in this field.<br>Possible enum values:  - `"AlwaysAllow"` policy means that all running pods (status.phase="Running"), but not yet healthy are considered disrupted and can be evicted regardless of whether the criteria in a PDB is met. This means perspective running pods of a disrupted application might not get a chance to become healthy. Healthy pods will be subject to the PDB for eviction.  - `"IfHealthyBudget"` policy means that running pods (status.phase="Running"), but not yet healthy can be evicted only if the guarded application is not disrupted (status.currentHealthy is at least equal to status.desiredHealthy). Healthy pods will be subject to the PDB for eviction. |

### .status {#_status}

Description
: PodDisruptionBudgetStatus represents information about the status of a PodDisruptionBudget. Status may trail the actual state of a system.

Type
: ```
  `object`
  ```

Required
: - `disruptionsAllowed`
  - `currentHealthy`
  - `desiredHealthy`
  - `expectedPods`

| Property | Type | Description |
| --- | --- | --- |
| `conditions` | [`array (Condition)`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-Condition) | Conditions contain conditions for PDB. The disruption controller sets the DisruptionAllowed condition. The following are known values for the reason field (additional reasons could be added in the future): - SyncFailed: The controller encountered an error and wasn’t able to compute               the number of allowed disruptions. Therefore no disruptions are               allowed and the status of the condition will be False. - InsufficientPods: The number of pods are either at or below the number                     required by the PodDisruptionBudget. No disruptions are                     allowed and the status of the condition will be False. - SufficientPods: There are more pods than required by the PodDisruptionBudget.                   The condition will be True, and the number of allowed                   disruptions are provided by the disruptionsAllowed property. |
| `currentHealthy` | `integer` | current number of healthy pods |
| `desiredHealthy` | `integer` | minimum desired number of healthy pods |
| `disruptedPods` | [`object (Time)`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-Time) | DisruptedPods contains information about pods whose eviction was processed by the API server eviction subresource handler but has not yet been observed by the PodDisruptionBudget controller. A pod will be in this map from the time when the API server processed the eviction request to the time when the pod is seen by PDB controller as having been marked for deletion (or after a timeout). The key in the map is the name of the pod and the value is the time when the API server processed the eviction request. If the deletion didn’t occur and a pod is still there it will be removed from the list automatically by PodDisruptionBudget controller after some time. If everything goes smooth this map should be empty for the most of the time. Large number of entries in the map may indicate problems with pod deletions. |
| `disruptionsAllowed` | `integer` | Number of pod disruptions that are currently allowed. |
| `expectedPods` | `integer` | total number of pods counted by this disruption budget |
| `observedGeneration` | `integer` | Most recent generation observed when updating this PDB status. DisruptionsAllowed and other status information is valid only if observedGeneration equals to PDB’s object generation. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/policy/v1/poddisruptionbudgets`
  - `GET`: list or watch objects of kind PodDisruptionBudget
- `/apis/policy/v1/watch/poddisruptionbudgets`
  - `GET`: watch individual changes to a list of PodDisruptionBudget. deprecated: use the 'watch' parameter with a list operation instead.
- `/apis/policy/v1/namespaces/{namespace}/poddisruptionbudgets`
  - `DELETE`: delete collection of PodDisruptionBudget
  - `GET`: list or watch objects of kind PodDisruptionBudget
  - `POST`: create a PodDisruptionBudget
- `/apis/policy/v1/watch/namespaces/{namespace}/poddisruptionbudgets`
  - `GET`: watch individual changes to a list of PodDisruptionBudget. deprecated: use the 'watch' parameter with a list operation instead.
- `/apis/policy/v1/namespaces/{namespace}/poddisruptionbudgets/{name}`
  - `DELETE`: delete a PodDisruptionBudget
  - `GET`: read the specified PodDisruptionBudget
  - `PATCH`: partially update the specified PodDisruptionBudget
  - `PUT`: replace the specified PodDisruptionBudget
- `/apis/policy/v1/watch/namespaces/{namespace}/poddisruptionbudgets/{name}`
  - `GET`: watch changes to an object of kind PodDisruptionBudget. deprecated: use the 'watch' parameter with a list operation instead, filtered to a single item with the 'fieldSelector' parameter.
- `/apis/policy/v1/namespaces/{namespace}/poddisruptionbudgets/{name}/status`
  - `GET`: read status of the specified PodDisruptionBudget
  - `PATCH`: partially update status of the specified PodDisruptionBudget
  - `PUT`: replace status of the specified PodDisruptionBudget

### /apis/policy/v1/poddisruptionbudgets {#_apispolicyv1poddisruptionbudgets}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  list or watch objects of kind PodDisruptionBudget
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`PodDisruptionBudgetList`](/docs/rest_api/objects#io-k8s-api-policy-v1-PodDisruptionBudgetList) schema |
| 401 - Unauthorized | Empty |

### /apis/policy/v1/watch/poddisruptionbudgets {#_apispolicyv1watchpoddisruptionbudgets}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch individual changes to a list of PodDisruptionBudget. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |

### /apis/policy/v1/namespaces/{namespace}/poddisruptionbudgets {#_apispolicyv1namespaces_namespace_poddisruptionbudgets}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of PodDisruptionBudget
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
  list or watch objects of kind PodDisruptionBudget
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`PodDisruptionBudgetList`](/docs/rest_api/objects#io-k8s-api-policy-v1-PodDisruptionBudgetList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a PodDisruptionBudget
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`PodDisruptionBudget`](/docs/rest_api/policy_apis/poddisruptionbudget-policy-v1#poddisruptionbudget-policy-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`PodDisruptionBudget`](/docs/rest_api/policy_apis/poddisruptionbudget-policy-v1#poddisruptionbudget-policy-v1) schema |
| 201 - Created | [`PodDisruptionBudget`](/docs/rest_api/policy_apis/poddisruptionbudget-policy-v1#poddisruptionbudget-policy-v1) schema |
| 202 - Accepted | [`PodDisruptionBudget`](/docs/rest_api/policy_apis/poddisruptionbudget-policy-v1#poddisruptionbudget-policy-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/policy/v1/watch/namespaces/{namespace}/poddisruptionbudgets {#_apispolicyv1watchnamespaces_namespace_poddisruptionbudgets}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch individual changes to a list of PodDisruptionBudget. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |

### /apis/policy/v1/namespaces/{namespace}/poddisruptionbudgets/{name} {#_apispolicyv1namespaces_namespace_poddisruptionbudgets_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the PodDisruptionBudget |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete a PodDisruptionBudget
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
  read the specified PodDisruptionBudget
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`PodDisruptionBudget`](/docs/rest_api/policy_apis/poddisruptionbudget-policy-v1#poddisruptionbudget-policy-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified PodDisruptionBudget
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`PodDisruptionBudget`](/docs/rest_api/policy_apis/poddisruptionbudget-policy-v1#poddisruptionbudget-policy-v1) schema |
| 201 - Created | [`PodDisruptionBudget`](/docs/rest_api/policy_apis/poddisruptionbudget-policy-v1#poddisruptionbudget-policy-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified PodDisruptionBudget
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`PodDisruptionBudget`](/docs/rest_api/policy_apis/poddisruptionbudget-policy-v1#poddisruptionbudget-policy-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`PodDisruptionBudget`](/docs/rest_api/policy_apis/poddisruptionbudget-policy-v1#poddisruptionbudget-policy-v1) schema |
| 201 - Created | [`PodDisruptionBudget`](/docs/rest_api/policy_apis/poddisruptionbudget-policy-v1#poddisruptionbudget-policy-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/policy/v1/watch/namespaces/{namespace}/poddisruptionbudgets/{name} {#_apispolicyv1watchnamespaces_namespace_poddisruptionbudgets_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the PodDisruptionBudget |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch changes to an object of kind PodDisruptionBudget. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead, filtered to a single item with the &#x27;fieldSelector&#x27; parameter.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |

### /apis/policy/v1/namespaces/{namespace}/poddisruptionbudgets/{name}/status {#_apispolicyv1namespaces_namespace_poddisruptionbudgets_name_status}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the PodDisruptionBudget |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read status of the specified PodDisruptionBudget
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`PodDisruptionBudget`](/docs/rest_api/policy_apis/poddisruptionbudget-policy-v1#poddisruptionbudget-policy-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update status of the specified PodDisruptionBudget
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`PodDisruptionBudget`](/docs/rest_api/policy_apis/poddisruptionbudget-policy-v1#poddisruptionbudget-policy-v1) schema |
| 201 - Created | [`PodDisruptionBudget`](/docs/rest_api/policy_apis/poddisruptionbudget-policy-v1#poddisruptionbudget-policy-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace status of the specified PodDisruptionBudget
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`PodDisruptionBudget`](/docs/rest_api/policy_apis/poddisruptionbudget-policy-v1#poddisruptionbudget-policy-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`PodDisruptionBudget`](/docs/rest_api/policy_apis/poddisruptionbudget-policy-v1#poddisruptionbudget-policy-v1) schema |
| 201 - Created | [`PodDisruptionBudget`](/docs/rest_api/policy_apis/poddisruptionbudget-policy-v1#poddisruptionbudget-policy-v1) schema |
| 401 - Unauthorized | Empty |
