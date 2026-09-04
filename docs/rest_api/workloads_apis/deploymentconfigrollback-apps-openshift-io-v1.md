---
title: "DeploymentConfigRollback [apps.openshift.io/v1]"
sidebar_position: 10
---

# DeploymentConfigRollback [apps.openshift.io/v1] {#deploymentconfigrollback-apps-openshift-io-v1}

<a id="deploymentconfigrollback-apps-openshift-io-v1"></a>

Description
: DeploymentConfigRollback provides the input to rollback generation.

  Compatibility level 1: Stable within a major release for a minimum of 12 months or 3 minor releases (whichever is longer).

Type
: ```
  `object`
  ```

Required
: - `name`
  - `spec`

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `name` | `string` | name of the deployment config that will be rolled back. |
| `spec` | `object` | DeploymentConfigRollbackSpec represents the options for rollback generation. |
| `updatedAnnotations` | `object (string)` | updatedAnnotations is a set of new annotations that will be added in the deployment config. |

### .spec {#_spec}

Description
: DeploymentConfigRollbackSpec represents the options for rollback generation.

Type
: ```
  `object`
  ```

Required
: - `from`
  - `includeTriggers`
  - `includeTemplate`
  - `includeReplicationMeta`
  - `includeStrategy`

| Property | Type | Description |
| --- | --- | --- |
| `from` | [`ObjectReference`](/docs/rest_api/objects#io-k8s-api-core-v1-ObjectReference) | from points to a ReplicationController which is a deployment. |
| `includeReplicationMeta` | `boolean` | includeReplicationMeta specifies whether to include the replica count and selector. |
| `includeStrategy` | `boolean` | includeStrategy specifies whether to include the deployment Strategy. |
| `includeTemplate` | `boolean` | includeTemplate specifies whether to include the PodTemplateSpec. |
| `includeTriggers` | `boolean` | includeTriggers specifies whether to include config Triggers. |
| `revision` | `integer` | revision to rollback to. If set to 0, rollback to the last revision. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/apps.openshift.io/v1/namespaces/{namespace}/deploymentconfigs/{name}/rollback`
  - `POST`: create rollback of a DeploymentConfig

### /apis/apps.openshift.io/v1/namespaces/{namespace}/deploymentconfigs/{name}/rollback {#_apisappsopenshiftiov1namespaces_namespace_deploymentconfigs_name_rollback}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the DeploymentConfigRollback |

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
  create rollback of a DeploymentConfig
  ```

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`DeploymentConfigRollback`](/docs/rest_api/workloads_apis/deploymentconfigrollback-apps-openshift-io-v1#deploymentconfigrollback-apps-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`DeploymentConfigRollback`](/docs/rest_api/workloads_apis/deploymentconfigrollback-apps-openshift-io-v1#deploymentconfigrollback-apps-openshift-io-v1) schema |
| 201 - Created | [`DeploymentConfigRollback`](/docs/rest_api/workloads_apis/deploymentconfigrollback-apps-openshift-io-v1#deploymentconfigrollback-apps-openshift-io-v1) schema |
| 202 - Accepted | [`DeploymentConfigRollback`](/docs/rest_api/workloads_apis/deploymentconfigrollback-apps-openshift-io-v1#deploymentconfigrollback-apps-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |
