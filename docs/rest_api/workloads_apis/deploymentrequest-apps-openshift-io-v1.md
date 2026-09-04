---
title: "DeploymentRequest [apps.openshift.io/v1]"
sidebar_position: 12
---

# DeploymentRequest [apps.openshift.io/v1] {#deploymentrequest-apps-openshift-io-v1}

<a id="deploymentrequest-apps-openshift-io-v1"></a>

Description
: DeploymentRequest is a request to a deployment config for a new deployment.

  Compatibility level 1: Stable within a major release for a minimum of 12 months or 3 minor releases (whichever is longer).

Type
: ```
  `object`
  ```

Required
: - `name`
  - `latest`
  - `force`

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `excludeTriggers` | `array (string)` | excludeTriggers instructs the instantiator to avoid processing the specified triggers. This field overrides the triggers from latest and allows clients to control specific logic. This field is ignored if not specified. |
| `force` | `boolean` | force will try to force a new deployment to run. If the deployment config is paused, then setting this to true will return an Invalid error. |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `latest` | `boolean` | latest will update the deployment config with the latest state from all triggers. |
| `name` | `string` | name of the deployment config for requesting a new deployment. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/apps.openshift.io/v1/namespaces/{namespace}/deploymentconfigs/{name}/instantiate`
  - `POST`: create instantiate of a DeploymentConfig

### /apis/apps.openshift.io/v1/namespaces/{namespace}/deploymentconfigs/{name}/instantiate {#_apisappsopenshiftiov1namespaces_namespace_deploymentconfigs_name_instantiate}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the DeploymentRequest |

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
  create instantiate of a DeploymentConfig
  ```

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`DeploymentRequest`](/docs/rest_api/workloads_apis/deploymentrequest-apps-openshift-io-v1#deploymentrequest-apps-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`DeploymentRequest`](/docs/rest_api/workloads_apis/deploymentrequest-apps-openshift-io-v1#deploymentrequest-apps-openshift-io-v1) schema |
| 201 - Created | [`DeploymentRequest`](/docs/rest_api/workloads_apis/deploymentrequest-apps-openshift-io-v1#deploymentrequest-apps-openshift-io-v1) schema |
| 202 - Accepted | [`DeploymentRequest`](/docs/rest_api/workloads_apis/deploymentrequest-apps-openshift-io-v1#deploymentrequest-apps-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |
