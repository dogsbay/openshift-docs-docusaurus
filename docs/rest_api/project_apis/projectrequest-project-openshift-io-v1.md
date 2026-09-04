---
title: "ProjectRequest [project.openshift.io/v1]"
sidebar_position: 3
---

# ProjectRequest [project.openshift.io/v1] {#projectrequest-project-openshift-io-v1}

<a id="projectrequest-project-openshift-io-v1"></a>

Description
: ProjectRequest is the set of options necessary to fully qualify a project request

  Compatibility level 1: Stable within a major release for a minimum of 12 months or 3 minor releases (whichever is longer).

Type
: ```
  `object`
  ```

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `description` | `string` | description is the description to apply to a project |
| `displayName` | `string` | displayName is the display name to apply to a project |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | metadata is the standard object’s metadata. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/project.openshift.io/v1/projectrequests`
  - `GET`: list objects of kind ProjectRequest
  - `POST`: create a ProjectRequest

### /apis/project.openshift.io/v1/projectrequests {#_apisprojectopenshiftiov1projectrequests}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  list objects of kind ProjectRequest
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Status_v2`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-Status_v2) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a ProjectRequest
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`ProjectRequest`](/docs/rest_api/project_apis/projectrequest-project-openshift-io-v1#projectrequest-project-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ProjectRequest`](/docs/rest_api/project_apis/projectrequest-project-openshift-io-v1#projectrequest-project-openshift-io-v1) schema |
| 201 - Created | [`ProjectRequest`](/docs/rest_api/project_apis/projectrequest-project-openshift-io-v1#projectrequest-project-openshift-io-v1) schema |
| 202 - Accepted | [`ProjectRequest`](/docs/rest_api/project_apis/projectrequest-project-openshift-io-v1#projectrequest-project-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |
