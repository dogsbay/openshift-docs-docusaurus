---
title: "BuildLog [build.openshift.io/v1]"
sidebar_position: 4
---

# BuildLog [build.openshift.io/v1] {#buildlog-build-openshift-io-v1}

<a id="buildlog-build-openshift-io-v1"></a>

Description
: BuildLog is the (unused) resource associated with the build log redirector

  Compatibility level 1: Stable within a major release for a minimum of 12 months or 3 minor releases (whichever is longer).

Type
: ```
  `object`
  ```

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/build.openshift.io/v1/namespaces/{namespace}/builds/{name}/log`
  - `GET`: read log of the specified Build

### /apis/build.openshift.io/v1/namespaces/{namespace}/builds/{name}/log {#_apisbuildopenshiftiov1namespaces_namespace_builds_name_log}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the BuildLog |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read log of the specified Build
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`BuildLog`](/docs/rest_api/workloads_apis/buildlog-build-openshift-io-v1#buildlog-build-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |
