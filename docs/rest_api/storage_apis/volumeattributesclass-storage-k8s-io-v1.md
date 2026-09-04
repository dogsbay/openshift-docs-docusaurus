---
title: "VolumeAttributesClass [storage.k8s.io/v1]"
sidebar_position: 11
---

# VolumeAttributesClass [storage.k8s.io/v1] {#volumeattributesclass-storage-k8s-io-v1}

<a id="volumeattributesclass-storage-k8s-io-v1"></a>

Description
: VolumeAttributesClass represents a specification of mutable volume attributes defined by the CSI driver. The class can be specified during dynamic provisioning of PersistentVolumeClaims, and changed in the PersistentVolumeClaim spec after provisioning.

Type
: ```
  `object`
  ```

Required
: - `driverName`

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `driverName` | `string` | Name of the CSI driver This field is immutable. |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | Standard object’s metadata. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata |
| `parameters` | `object (string)` | parameters hold volume attributes defined by the CSI driver. These values are opaque to the Kubernetes and are passed directly to the CSI driver. The underlying storage provider supports changing these attributes on an existing volume, however the parameters field itself is immutable. To invoke a volume update, a new VolumeAttributesClass should be created with new parameters, and the PersistentVolumeClaim should be updated to reference the new VolumeAttributesClass.<br>This field is required and must contain at least one key/value pair. The keys cannot be empty, and the maximum number of parameters is 512, with a cumulative max size of 256K. If the CSI driver rejects invalid parameters, the target PersistentVolumeClaim will be set to an "Infeasible" state in the modifyVolumeStatus field. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/storage.k8s.io/v1/volumeattributesclasses`
  - `DELETE`: delete collection of VolumeAttributesClass
  - `GET`: list or watch objects of kind VolumeAttributesClass
  - `POST`: create a VolumeAttributesClass
- `/apis/storage.k8s.io/v1/watch/volumeattributesclasses`
  - `GET`: watch individual changes to a list of VolumeAttributesClass. deprecated: use the 'watch' parameter with a list operation instead.
- `/apis/storage.k8s.io/v1/volumeattributesclasses/{name}`
  - `DELETE`: delete a VolumeAttributesClass
  - `GET`: read the specified VolumeAttributesClass
  - `PATCH`: partially update the specified VolumeAttributesClass
  - `PUT`: replace the specified VolumeAttributesClass
- `/apis/storage.k8s.io/v1/watch/volumeattributesclasses/{name}`
  - `GET`: watch changes to an object of kind VolumeAttributesClass. deprecated: use the 'watch' parameter with a list operation instead, filtered to a single item with the 'fieldSelector' parameter.

### /apis/storage.k8s.io/v1/volumeattributesclasses {#_apisstoragek8siov1volumeattributesclasses}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of VolumeAttributesClass
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
  list or watch objects of kind VolumeAttributesClass
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`VolumeAttributesClassList`](/docs/rest_api/objects#io-k8s-api-storage-v1-VolumeAttributesClassList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a VolumeAttributesClass
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`VolumeAttributesClass`](/docs/rest_api/storage_apis/volumeattributesclass-storage-k8s-io-v1#volumeattributesclass-storage-k8s-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`VolumeAttributesClass`](/docs/rest_api/storage_apis/volumeattributesclass-storage-k8s-io-v1#volumeattributesclass-storage-k8s-io-v1) schema |
| 201 - Created | [`VolumeAttributesClass`](/docs/rest_api/storage_apis/volumeattributesclass-storage-k8s-io-v1#volumeattributesclass-storage-k8s-io-v1) schema |
| 202 - Accepted | [`VolumeAttributesClass`](/docs/rest_api/storage_apis/volumeattributesclass-storage-k8s-io-v1#volumeattributesclass-storage-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/storage.k8s.io/v1/watch/volumeattributesclasses {#_apisstoragek8siov1watchvolumeattributesclasses}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch individual changes to a list of VolumeAttributesClass. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |

### /apis/storage.k8s.io/v1/volumeattributesclasses/{name} {#_apisstoragek8siov1volumeattributesclasses_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the VolumeAttributesClass |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete a VolumeAttributesClass
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`VolumeAttributesClass`](/docs/rest_api/storage_apis/volumeattributesclass-storage-k8s-io-v1#volumeattributesclass-storage-k8s-io-v1) schema |
| 202 - Accepted | [`VolumeAttributesClass`](/docs/rest_api/storage_apis/volumeattributesclass-storage-k8s-io-v1#volumeattributesclass-storage-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read the specified VolumeAttributesClass
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`VolumeAttributesClass`](/docs/rest_api/storage_apis/volumeattributesclass-storage-k8s-io-v1#volumeattributesclass-storage-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified VolumeAttributesClass
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`VolumeAttributesClass`](/docs/rest_api/storage_apis/volumeattributesclass-storage-k8s-io-v1#volumeattributesclass-storage-k8s-io-v1) schema |
| 201 - Created | [`VolumeAttributesClass`](/docs/rest_api/storage_apis/volumeattributesclass-storage-k8s-io-v1#volumeattributesclass-storage-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified VolumeAttributesClass
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`VolumeAttributesClass`](/docs/rest_api/storage_apis/volumeattributesclass-storage-k8s-io-v1#volumeattributesclass-storage-k8s-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`VolumeAttributesClass`](/docs/rest_api/storage_apis/volumeattributesclass-storage-k8s-io-v1#volumeattributesclass-storage-k8s-io-v1) schema |
| 201 - Created | [`VolumeAttributesClass`](/docs/rest_api/storage_apis/volumeattributesclass-storage-k8s-io-v1#volumeattributesclass-storage-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/storage.k8s.io/v1/watch/volumeattributesclasses/{name} {#_apisstoragek8siov1watchvolumeattributesclasses_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the VolumeAttributesClass |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch changes to an object of kind VolumeAttributesClass. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead, filtered to a single item with the &#x27;fieldSelector&#x27; parameter.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |
