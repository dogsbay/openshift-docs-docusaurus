---
title: "ConsoleSample [console.openshift.io/v1]"
sidebar_position: 8
---

# ConsoleSample [console.openshift.io/v1] {#consolesample-console-openshift-io-v1}

<a id="consolesample-console-openshift-io-v1"></a>

Description
: ConsoleSample is an extension to customizing OpenShift web console by adding samples.

  Compatibility level 1: Stable within a major release for a minimum of 12 months or 3 minor releases (whichever is longer).

Type
: ```
  `object`
  ```

Required
: - `metadata`
  - `spec`

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | Standard object’s metadata. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata |
| `spec` | `object` | spec contains configuration for a console sample. |

### .spec {#_spec}

Description
: spec contains configuration for a console sample.

Type
: ```
  `object`
  ```

Required
: - `abstract`
  - `description`
  - `source`
  - `title`

| Property | Type | Description |
| --- | --- | --- |
| `abstract` | `string` | abstract is a short introduction to the sample.<br>It is required and must be no more than 100 characters in length.<br>The abstract is shown on the sample card tile below the title and provider and is limited to three lines of content. |
| `description` | `string` | description is a long form explanation of the sample.<br>It is required and can have a maximum length of **4096** characters.<br>It is a README.md-like content for additional information, links, pre-conditions, and other instructions. It will be rendered as Markdown so that it can contain line breaks, links, and other simple formatting. |
| `icon` | `string` | icon is an optional base64 encoded image and shown beside the sample title.<br>The format must follow the data: URL format and can have a maximum size of **10 KB**.<br>  data:[<mediatype>][;base64],<base64 encoded image><br>For example:<br>  data:image;base64,             plus the base64 encoded image.<br>Vector images can also be used. SVG icons must start with:<br>  data:image/svg+xml;base64,     plus the base64 encoded SVG image.<br>All sample catalog icons will be shown on a white background (also when the dark theme is used). The web console ensures that different aspect ratios work correctly. Currently, the surface of the icon is at most 40x100px.<br>For more information on the data URL format, please visit https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URLs. |
| `provider` | `string` | provider is an optional label to honor who provides the sample.<br>It is optional and must be no more than 50 characters in length.<br>A provider can be a company like "Red Hat" or an organization like "CNCF" or "Knative".<br>Currently, the provider is only shown on the sample card tile below the title with the prefix "Provided by " |
| `source` | `object` | source defines where to deploy the sample service from. The sample may be sourced from an external git repository or container image. |
| `tags` | `array (string)` | tags are optional string values that can be used to find samples in the samples catalog.<br>Examples of common tags may be "Java", "Quarkus", etc.<br>They will be displayed on the samples details page. |
| `title` | `string` | title is the display name of the sample.<br>It is required and must be no more than 50 characters in length. |
| `type` | `string` | type is an optional label to group multiple samples.<br>It is optional and must be no more than 20 characters in length.<br>Recommendation is a singular term like "Builder Image", "Devfile" or "Serverless Function".<br>Currently, the type is shown a badge on the sample card tile in the top right corner. |

### .spec.source {#_specsource}

Description
: source defines where to deploy the sample service from. The sample may be sourced from an external git repository or container image.

Type
: ```
  `object`
  ```

Required
: - `type`

| Property | Type | Description |
| --- | --- | --- |
| `containerImport` | `object` | containerImport allows the user import a container image. |
| `gitImport` | `object` | gitImport allows the user to import code from a git repository. |
| `type` | `string` | type of the sample, currently supported: "GitImport";"ContainerImport" |

### .spec.source.containerImport {#_specsourcecontainerimport}

Description
: containerImport allows the user import a container image.

Type
: ```
  `object`
  ```

Required
: - `image`

| Property | Type | Description |
| --- | --- | --- |
| `image` | `string` | reference to a container image that provides a HTTP service. The service must be exposed on the default port (8080) unless otherwise configured with the port field.<br>Supported formats:   - <repository-name>/<image-name>   - docker.io/<repository-name>/<image-name>   - quay.io/<repository-name>/<image-name>   - quay.io/<repository-name>/<image-name>@sha256:<image hash>   - quay.io/<repository-name>/<image-name>:<tag> |
| `service` | `object` | service contains configuration for the Service resource created for this sample. |

### .spec.source.containerImport.service {#_specsourcecontainerimportservice}

Description
: service contains configuration for the Service resource created for this sample.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `targetPort` | `integer` | targetPort is the port that the service listens on for HTTP requests. This port will be used for Service and Route created for this sample. Port must be in the range 1 to 65535. Default port is 8080. |

### .spec.source.gitImport {#_specsourcegitimport}

Description
: gitImport allows the user to import code from a git repository.

Type
: ```
  `object`
  ```

Required
: - `repository`

| Property | Type | Description |
| --- | --- | --- |
| `repository` | `object` | repository contains the reference to the actual Git repository. |
| `service` | `object` | service contains configuration for the Service resource created for this sample. |

### .spec.source.gitImport.repository {#_specsourcegitimportrepository}

Description
: repository contains the reference to the actual Git repository.

Type
: ```
  `object`
  ```

Required
: - `url`

| Property | Type | Description |
| --- | --- | --- |
| `contextDir` | `string` | contextDir is used to specify a directory within the repository to build the component. Must start with `/` and have a maximum length of 256 characters. When omitted, the default value is to build from the root of the repository. |
| `revision` | `string` | revision is the git revision at which to clone the git repository Can be used to clone a specific branch, tag or commit SHA. Must be at most 256 characters in length. When omitted the repository’s default branch is used. |
| `url` | `string` | url of the Git repository that contains a HTTP service. The HTTP service must be exposed on the default port (8080) unless otherwise configured with the port field.<br>Only public repositories on GitHub, GitLab and Bitbucket are currently supported:<br>  - https://github.com/<org>/<repository>   - https://gitlab.com/<org>/<repository>   - https://bitbucket.org/<org>/<repository><br>The url must have a maximum length of 256 characters. |

### .spec.source.gitImport.service {#_specsourcegitimportservice}

Description
: service contains configuration for the Service resource created for this sample.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `targetPort` | `integer` | targetPort is the port that the service listens on for HTTP requests. This port will be used for Service created for this sample. Port must be in the range 1 to 65535. Default port is 8080. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/console.openshift.io/v1/consolesamples`
  - `DELETE`: delete collection of ConsoleSample
  - `GET`: list objects of kind ConsoleSample
  - `POST`: create a ConsoleSample
- `/apis/console.openshift.io/v1/consolesamples/{name}`
  - `DELETE`: delete a ConsoleSample
  - `GET`: read the specified ConsoleSample
  - `PATCH`: partially update the specified ConsoleSample
  - `PUT`: replace the specified ConsoleSample

### /apis/console.openshift.io/v1/consolesamples {#_apisconsoleopenshiftiov1consolesamples}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of ConsoleSample
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
  list objects of kind ConsoleSample
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ConsoleSampleList`](/docs/rest_api/objects#io-openshift-console-v1-ConsoleSampleList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a ConsoleSample
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`ConsoleSample`](/docs/rest_api/console_apis/consolesample-console-openshift-io-v1#consolesample-console-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ConsoleSample`](/docs/rest_api/console_apis/consolesample-console-openshift-io-v1#consolesample-console-openshift-io-v1) schema |
| 201 - Created | [`ConsoleSample`](/docs/rest_api/console_apis/consolesample-console-openshift-io-v1#consolesample-console-openshift-io-v1) schema |
| 202 - Accepted | [`ConsoleSample`](/docs/rest_api/console_apis/consolesample-console-openshift-io-v1#consolesample-console-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/console.openshift.io/v1/consolesamples/{name} {#_apisconsoleopenshiftiov1consolesamples_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the ConsoleSample |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete a ConsoleSample
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
  read the specified ConsoleSample
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ConsoleSample`](/docs/rest_api/console_apis/consolesample-console-openshift-io-v1#consolesample-console-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified ConsoleSample
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ConsoleSample`](/docs/rest_api/console_apis/consolesample-console-openshift-io-v1#consolesample-console-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified ConsoleSample
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`ConsoleSample`](/docs/rest_api/console_apis/consolesample-console-openshift-io-v1#consolesample-console-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ConsoleSample`](/docs/rest_api/console_apis/consolesample-console-openshift-io-v1#consolesample-console-openshift-io-v1) schema |
| 201 - Created | [`ConsoleSample`](/docs/rest_api/console_apis/consolesample-console-openshift-io-v1#consolesample-console-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |
