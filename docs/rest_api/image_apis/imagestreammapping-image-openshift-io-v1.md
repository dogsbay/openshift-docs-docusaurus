---
title: "ImageStreamMapping [image.openshift.io/v1]"
sidebar_position: 7
---

# ImageStreamMapping [image.openshift.io/v1] {#imagestreammapping-image-openshift-io-v1}

<a id="imagestreammapping-image-openshift-io-v1"></a>

Description
: ImageStreamMapping represents a mapping from a single image stream tag to a container image as well as the reference to the container image stream the image came from. This resource is used by privileged integrators to create an image resource and to associate it with an image stream in the status tags field. Creating an ImageStreamMapping will allow any user who can view the image stream to tag or pull that image, so only create mappings where the user has proven they have access to the image contents directly. The only operation supported for this resource is create and the metadata name and namespace should be set to the image stream containing the tag that should be updated.

  Compatibility level 1: Stable within a major release for a minimum of 12 months or 3 minor releases (whichever is longer).

Type
: ```
  `object`
  ```

Required
: - `image`
  - `tag`

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `image` | `object` | Image is an immutable representation of a container image and its metadata at a point in time. Images are named by taking a hash of their contents (metadata and content) and any change in format, content, or metadata results in a new name. The images resource is primarily for use by cluster administrators and integrations like the cluster image registry - end users, instead, access images via the imagestreamtags or imagestreamimages resources. While image metadata is stored in the API, any integration that implements the container image registry API must provide its own storage for the raw manifest data, image config, and layer contents.<br>Compatibility level 1: Stable within a major release for a minimum of 12 months or 3 minor releases (whichever is longer). |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | metadata is the standard object’s metadata. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata |
| `tag` | `string` | tag is a string value this image can be located with inside the stream. |

### .image {#_image}

Description
: Image is an immutable representation of a container image and its metadata at a point in time. Images are named by taking a hash of their contents (metadata and content) and any change in format, content, or metadata results in a new name. The images resource is primarily for use by cluster administrators and integrations like the cluster image registry - end users, instead, access images via the imagestreamtags or imagestreamimages resources. While image metadata is stored in the API, any integration that implements the container image registry API must provide its own storage for the raw manifest data, image config, and layer contents.

  Compatibility level 1: Stable within a major release for a minimum of 12 months or 3 minor releases (whichever is longer).

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `dockerImageConfig` | `string` | dockerImageConfig is a JSON blob that the runtime uses to set up the container. This is a part of manifest schema v2. Will not be set when the image represents a manifest list. |
| `dockerImageLayers` | `array` | dockerImageLayers represents the layers in the image. May not be set if the image does not define that data or if the image represents a manifest list. |
| `dockerImageLayers[]` | `object` | ImageLayer represents a single layer of the image. Some images may have multiple layers. Some may have none. |
| `dockerImageManifest` | `string` | dockerImageManifest is the raw JSON of the manifest |
| `dockerImageManifestMediaType` | `string` | dockerImageManifestMediaType specifies the mediaType of manifest. This is a part of manifest schema v2. |
| `dockerImageManifests` | `array` | dockerImageManifests holds information about sub-manifests when the image represents a manifest list. When this field is present, no DockerImageLayers should be specified. |
| `dockerImageManifests[]` | `object` | ImageManifest represents sub-manifests of a manifest list. The Digest field points to a regular Image object. |
| `dockerImageMetadata` | [`RawExtension`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-runtime-RawExtension) | dockerImageMetadata contains metadata about this image |
| `dockerImageMetadataVersion` | `string` | dockerImageMetadataVersion conveys the version of the object, which if empty defaults to "1.0" |
| `dockerImageReference` | `string` | dockerImageReference is the string that can be used to pull this image. |
| `dockerImageSignatures` | `array (string)` | dockerImageSignatures provides the signatures as opaque blobs. This is a part of manifest schema v1. |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | metadata is the standard object’s metadata. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata |
| `signatures` | `array` | signatures holds all signatures of the image. |
| `signatures[]` | `object` | ImageSignature holds a signature of an image. It allows to verify image identity and possibly other claims as long as the signature is trusted. Based on this information it is possible to restrict runnable images to those matching cluster-wide policy. Mandatory fields should be parsed by clients doing image verification. The others are parsed from signature’s content by the server. They serve just an informative purpose.<br>Compatibility level 1: Stable within a major release for a minimum of 12 months or 3 minor releases (whichever is longer). |

### .image.dockerImageLayers {#_imagedockerimagelayers}

Description
: dockerImageLayers represents the layers in the image. May not be set if the image does not define that data or if the image represents a manifest list.

Type
: ```
  `array`
  ```

### .image.dockerImageLayers[] {#_imagedockerimagelayers}

Description
: ImageLayer represents a single layer of the image. Some images may have multiple layers. Some may have none.

Type
: ```
  `object`
  ```

Required
: - `name`
  - `size`
  - `mediaType`

| Property | Type | Description |
| --- | --- | --- |
| `mediaType` | `string` | mediaType of the referenced object. |
| `name` | `string` | name of the layer as defined by the underlying store. |
| `size` | `integer` | size of the layer in bytes as defined by the underlying store. |

### .image.dockerImageManifests {#_imagedockerimagemanifests}

Description
: dockerImageManifests holds information about sub-manifests when the image represents a manifest list. When this field is present, no DockerImageLayers should be specified.

Type
: ```
  `array`
  ```

### .image.dockerImageManifests[] {#_imagedockerimagemanifests}

Description
: ImageManifest represents sub-manifests of a manifest list. The Digest field points to a regular Image object.

Type
: ```
  `object`
  ```

Required
: - `digest`
  - `mediaType`
  - `manifestSize`
  - `architecture`
  - `os`

| Property | Type | Description |
| --- | --- | --- |
| `architecture` | `string` | architecture specifies the supported CPU architecture, for example `amd64` or `ppc64le`. |
| `digest` | `string` | digest is the unique identifier for the manifest. It refers to an Image object. |
| `manifestSize` | `integer` | manifestSize represents the size of the raw object contents, in bytes. |
| `mediaType` | `string` | mediaType defines the type of the manifest, possible values are application/vnd.oci.image.manifest.v1+json, application/vnd.docker.distribution.manifest.v2+json or application/vnd.docker.distribution.manifest.v1+json. |
| `os` | `string` | os specifies the operating system, for example `linux`. |
| `variant` | `string` | variant is an optional field repreenting a variant of the CPU, for example v6 to specify a particular CPU variant of the ARM CPU. |

### .image.signatures {#_imagesignatures}

Description
: signatures holds all signatures of the image.

Type
: ```
  `array`
  ```

### .image.signatures[] {#_imagesignatures}

Description
: ImageSignature holds a signature of an image. It allows to verify image identity and possibly other claims as long as the signature is trusted. Based on this information it is possible to restrict runnable images to those matching cluster-wide policy. Mandatory fields should be parsed by clients doing image verification. The others are parsed from signature’s content by the server. They serve just an informative purpose.

  Compatibility level 1: Stable within a major release for a minimum of 12 months or 3 minor releases (whichever is longer).

Type
: ```
  `object`
  ```

Required
: - `type`
  - `content`

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `conditions` | `array` | conditions represent the latest available observations of a signature’s current state. |
| `conditions[]` | `object` | SignatureCondition describes an image signature condition of particular kind at particular probe time. |
| `content` | `string` | Required: An opaque binary string which is an image’s signature. |
| `created` | [`Time`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-Time) | If specified, it is the time of signature’s creation. |
| `imageIdentity` | `string` | A human readable string representing image’s identity. It could be a product name and version, or an image pull spec (e.g. "registry.access.redhat.com/rhel7/rhel:7.2"). |
| `issuedBy` | `object` | SignatureIssuer holds information about an issuer of signing certificate or key. |
| `issuedTo` | `object` | SignatureSubject holds information about a person or entity who created the signature. |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | metadata is the standard object’s metadata. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata |
| `signedClaims` | `object (string)` | Contains claims from the signature. |
| `type` | `string` | Required: Describes a type of stored blob. |

### .image.signatures[].conditions {#_imagesignaturesconditions}

Description
: conditions represent the latest available observations of a signature’s current state.

Type
: ```
  `array`
  ```

### .image.signatures[].conditions[] {#_imagesignaturesconditions}

Description
: SignatureCondition describes an image signature condition of particular kind at particular probe time.

Type
: ```
  `object`
  ```

Required
: - `type`
  - `status`

| Property | Type | Description |
| --- | --- | --- |
| `lastProbeTime` | [`Time`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-Time) | Last time the condition was checked. |
| `lastTransitionTime` | [`Time`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-Time) | Last time the condition transit from one status to another. |
| `message` | `string` | Human readable message indicating details about last transition. |
| `reason` | `string` | (brief) reason for the condition’s last transition. |
| `status` | `string` | status of the condition, one of True, False, Unknown. |
| `type` | `string` | type of signature condition, Complete or Failed. |

### .image.signatures[].issuedBy {#_imagesignaturesissuedby}

Description
: SignatureIssuer holds information about an issuer of signing certificate or key.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `commonName` | `string` | Common name (e.g. openshift-signing-service). |
| `organization` | `string` | organization name. |

### .image.signatures[].issuedTo {#_imagesignaturesissuedto}

Description
: SignatureSubject holds information about a person or entity who created the signature.

Type
: ```
  `object`
  ```

Required
: - `publicKeyID`

| Property | Type | Description |
| --- | --- | --- |
| `commonName` | `string` | Common name (e.g. openshift-signing-service). |
| `organization` | `string` | organization name. |
| `publicKeyID` | `string` | If present, it is a human readable key id of public key belonging to the subject used to verify image signature. It should contain at least 64 lowest bits of public key’s fingerprint (e.g. 0x685ebe62bf278440). |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/image.openshift.io/v1/namespaces/{namespace}/imagestreammappings`
  - `POST`: create an ImageStreamMapping

### /apis/image.openshift.io/v1/namespaces/{namespace}/imagestreammappings {#_apisimageopenshiftiov1namespaces_namespace_imagestreammappings}

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
  create an ImageStreamMapping
  ```

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`ImageStreamMapping`](/docs/rest_api/image_apis/imagestreammapping-image-openshift-io-v1#imagestreammapping-image-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ImageStreamMapping`](/docs/rest_api/image_apis/imagestreammapping-image-openshift-io-v1#imagestreammapping-image-openshift-io-v1) schema |
| 201 - Created | [`ImageStreamMapping`](/docs/rest_api/image_apis/imagestreammapping-image-openshift-io-v1#imagestreammapping-image-openshift-io-v1) schema |
| 202 - Accepted | [`ImageStreamMapping`](/docs/rest_api/image_apis/imagestreammapping-image-openshift-io-v1#imagestreammapping-image-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |
