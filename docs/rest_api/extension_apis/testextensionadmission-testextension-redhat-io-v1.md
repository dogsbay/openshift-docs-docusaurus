---
title: "TestExtensionAdmission [testextension.redhat.io/v1]"
sidebar_position: 5
---

# TestExtensionAdmission [testextension.redhat.io/v1] {#testextensionadmission-testextension-redhat-io-v1}

<a id="testextensionadmission-testextension-redhat-io-v1"></a>

Description
: TestExtensionAdmission controls which ImageStreams are permitted to provide extension test binaries

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
| `spec` | `object` | Specification of permitted ImageStreams |
| `status` | `` | Status of the TestExtensionAdmission |

### .spec {#_spec}

Description
: Specification of permitted ImageStreams

Type
: ```
  `object`
  ```

Required
: - `permit`

| Property | Type | Description |
| --- | --- | --- |
| `permit` | `array (string)` | List of permitted ImageStream patterns in format "namespace/imagestream". Each segment must be either "**" (wildcard) or a valid name (no embedded wildcards). Examples - "openshift/**", "**/**", "namespace/stream" |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/testextension.redhat.io/v1/testextensionadmissions`
  - `DELETE`: delete collection of TestExtensionAdmission
  - `GET`: list objects of kind TestExtensionAdmission
  - `POST`: create a TestExtensionAdmission
- `/apis/testextension.redhat.io/v1/testextensionadmissions/{name}`
  - `DELETE`: delete a TestExtensionAdmission
  - `GET`: read the specified TestExtensionAdmission
  - `PATCH`: partially update the specified TestExtensionAdmission
  - `PUT`: replace the specified TestExtensionAdmission
- `/apis/testextension.redhat.io/v1/testextensionadmissions/{name}/status`
  - `GET`: read status of the specified TestExtensionAdmission
  - `PATCH`: partially update status of the specified TestExtensionAdmission
  - `PUT`: replace status of the specified TestExtensionAdmission

### /apis/testextension.redhat.io/v1/testextensionadmissions {#_apistestextensionredhatiov1testextensionadmissions}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of TestExtensionAdmission
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
  list objects of kind TestExtensionAdmission
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`TestExtensionAdmissionList`](/docs/rest_api/objects#io-redhat-testextension-v1-TestExtensionAdmissionList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a TestExtensionAdmission
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`TestExtensionAdmission`](/docs/rest_api/extension_apis/testextensionadmission-testextension-redhat-io-v1#testextensionadmission-testextension-redhat-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`TestExtensionAdmission`](/docs/rest_api/extension_apis/testextensionadmission-testextension-redhat-io-v1#testextensionadmission-testextension-redhat-io-v1) schema |
| 201 - Created | [`TestExtensionAdmission`](/docs/rest_api/extension_apis/testextensionadmission-testextension-redhat-io-v1#testextensionadmission-testextension-redhat-io-v1) schema |
| 202 - Accepted | [`TestExtensionAdmission`](/docs/rest_api/extension_apis/testextensionadmission-testextension-redhat-io-v1#testextensionadmission-testextension-redhat-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/testextension.redhat.io/v1/testextensionadmissions/{name} {#_apistestextensionredhatiov1testextensionadmissions_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the TestExtensionAdmission |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete a TestExtensionAdmission
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
  read the specified TestExtensionAdmission
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`TestExtensionAdmission`](/docs/rest_api/extension_apis/testextensionadmission-testextension-redhat-io-v1#testextensionadmission-testextension-redhat-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified TestExtensionAdmission
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`TestExtensionAdmission`](/docs/rest_api/extension_apis/testextensionadmission-testextension-redhat-io-v1#testextensionadmission-testextension-redhat-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified TestExtensionAdmission
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`TestExtensionAdmission`](/docs/rest_api/extension_apis/testextensionadmission-testextension-redhat-io-v1#testextensionadmission-testextension-redhat-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`TestExtensionAdmission`](/docs/rest_api/extension_apis/testextensionadmission-testextension-redhat-io-v1#testextensionadmission-testextension-redhat-io-v1) schema |
| 201 - Created | [`TestExtensionAdmission`](/docs/rest_api/extension_apis/testextensionadmission-testextension-redhat-io-v1#testextensionadmission-testextension-redhat-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/testextension.redhat.io/v1/testextensionadmissions/{name}/status {#_apistestextensionredhatiov1testextensionadmissions_name_status}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the TestExtensionAdmission |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read status of the specified TestExtensionAdmission
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`TestExtensionAdmission`](/docs/rest_api/extension_apis/testextensionadmission-testextension-redhat-io-v1#testextensionadmission-testextension-redhat-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update status of the specified TestExtensionAdmission
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`TestExtensionAdmission`](/docs/rest_api/extension_apis/testextensionadmission-testextension-redhat-io-v1#testextensionadmission-testextension-redhat-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace status of the specified TestExtensionAdmission
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`TestExtensionAdmission`](/docs/rest_api/extension_apis/testextensionadmission-testextension-redhat-io-v1#testextensionadmission-testextension-redhat-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`TestExtensionAdmission`](/docs/rest_api/extension_apis/testextensionadmission-testextension-redhat-io-v1#testextensionadmission-testextension-redhat-io-v1) schema |
| 201 - Created | [`TestExtensionAdmission`](/docs/rest_api/extension_apis/testextensionadmission-testextension-redhat-io-v1#testextensionadmission-testextension-redhat-io-v1) schema |
| 401 - Unauthorized | Empty |
