---
title: "ImageStreamLayers [image.openshift.io/v1]"
sidebar_position: 6
---

# ImageStreamLayers [image.openshift.io/v1] {#imagestreamlayers-image-openshift-io-v1}

<a id="imagestreamlayers-image-openshift-io-v1"></a>

Description
: ImageStreamLayers describes information about the layers referenced by images in this image stream.

  Compatibility level 1: Stable within a major release for a minimum of 12 months or 3 minor releases (whichever is longer).

Type
: ```
  `object`
  ```

Required
: - `blobs`
  - `images`

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `blobs` | `object` | blobs is a map of blob name to metadata about the blob. |
| `blobs{}` | `object` | ImageLayerData contains metadata about an image layer. |
| `images` | `object` | images is a map between an image name and the names of the blobs and config that comprise the image. |
| `images{}` | `object` | ImageBlobReferences describes the blob references within an image. |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | metadata is the standard object’s metadata. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata |

### .blobs {#_blobs}

Description
: blobs is a map of blob name to metadata about the blob.

Type
: ```
  `object`
  ```

### .blobs{} {#_blobs}

Description
: ImageLayerData contains metadata about an image layer.

Type
: ```
  `object`
  ```

Required
: - `size`
  - `mediaType`

| Property | Type | Description |
| --- | --- | --- |
| `mediaType` | `string` | mediaType of the referenced object. |
| `size` | `integer` | size of the layer in bytes as defined by the underlying store. This field is optional if the necessary information about size is not available. |

### .images {#_images}

Description
: images is a map between an image name and the names of the blobs and config that comprise the image.

Type
: ```
  `object`
  ```

### .images{} {#_images}

Description
: ImageBlobReferences describes the blob references within an image.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `config` | `string` | config, if set, is the blob that contains the image config. Some images do not have separate config blobs and this field will be set to nil if so. |
| `imageMissing` | `boolean` | imageMissing is true if the image is referenced by the image stream but the image object has been deleted from the API by an administrator. When this field is set, layers and config fields may be empty and callers that depend on the image metadata should consider the image to be unavailable for download or viewing. |
| `layers` | `array (string)` | layers is the list of blobs that compose this image, from base layer to top layer. All layers referenced by this array will be defined in the blobs map. Some images may have zero layers. |
| `manifests` | `array (string)` | manifests is the list of other image names that this image points to. For a single architecture image, it is empty. For a multi-arch image, it consists of the digests of single architecture images, such images shouldn’t have layers nor config. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/image.openshift.io/v1/namespaces/{namespace}/imagestreams/{name}/layers`
  - `GET`: read layers of the specified ImageStream

### /apis/image.openshift.io/v1/namespaces/{namespace}/imagestreams/{name}/layers {#_apisimageopenshiftiov1namespaces_namespace_imagestreams_name_layers}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the ImageStreamLayers |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read layers of the specified ImageStream
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ImageStreamLayers`](/docs/rest_api/image_apis/imagestreamlayers-image-openshift-io-v1#imagestreamlayers-image-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |
