---
title: "CSINode [storage.k8s.io/v1]"
sidebar_position: 3
---

# CSINode [storage.k8s.io/v1] {#csinode-storage-k8s-io-v1}

<a id="csinode-storage-k8s-io-v1"></a>

Description
: CSINode holds information about all CSI drivers installed on a node. CSI drivers do not need to create the CSINode object directly. As long as they use the node-driver-registrar sidecar container, the kubelet will automatically populate the CSINode object for the CSI driver as part of kubelet plugin registration. CSINode has the same name as a node. If the object is missing, it means either there are no CSI Drivers available on the node, or the Kubelet version is low enough that it doesn’t create this object. CSINode has an OwnerReference that points to the corresponding node object.

Type
: ```
  `object`
  ```

Required
: - `spec`

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | Standard object’s metadata. metadata.name must be the Kubernetes node name. |
| `spec` | `object` | CSINodeSpec holds information about the specification of all CSI drivers installed on a node |

### .spec {#_spec}

Description
: CSINodeSpec holds information about the specification of all CSI drivers installed on a node

Type
: ```
  `object`
  ```

Required
: - `drivers`

| Property | Type | Description |
| --- | --- | --- |
| `drivers` | `array` | drivers is a list of information of all CSI Drivers existing on a node. If all drivers in the list are uninstalled, this can become empty. |
| `drivers[]` | `object` | CSINodeDriver holds information about the specification of one CSI driver installed on a node |

### .spec.drivers {#_specdrivers}

Description
: drivers is a list of information of all CSI Drivers existing on a node. If all drivers in the list are uninstalled, this can become empty.

Type
: ```
  `array`
  ```

### .spec.drivers[] {#_specdrivers}

Description
: CSINodeDriver holds information about the specification of one CSI driver installed on a node

Type
: ```
  `object`
  ```

Required
: - `name`
  - `nodeID`

| Property | Type | Description |
| --- | --- | --- |
| `allocatable` | `object` | VolumeNodeResources is a set of resource limits for scheduling of volumes. |
| `name` | `string` | name represents the name of the CSI driver that this object refers to. This MUST be the same name returned by the CSI GetPluginName() call for that driver. |
| `nodeID` | `string` | nodeID of the node from the driver point of view. This field enables Kubernetes to communicate with storage systems that do not share the same nomenclature for nodes. For example, Kubernetes may refer to a given node as "node1", but the storage system may refer to the same node as "nodeA". When Kubernetes issues a command to the storage system to attach a volume to a specific node, it can use this field to refer to the node name using the ID that the storage system will understand, e.g. "nodeA" instead of "node1". This field is required. |
| `topologyKeys` | `array (string)` | topologyKeys is the list of keys supported by the driver. When a driver is initialized on a cluster, it provides a set of topology keys that it understands (e.g. "company.com/zone", "company.com/region"). When a driver is initialized on a node, it provides the same topology keys along with values. Kubelet will expose these topology keys as labels on its own node object. When Kubernetes does topology aware provisioning, it can use this list to determine which labels it should retrieve from the node object and pass back to the driver. It is possible for different nodes to use different topology keys. This can be empty if driver does not support topology. |

### .spec.drivers[].allocatable {#_specdriversallocatable}

Description
: VolumeNodeResources is a set of resource limits for scheduling of volumes.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `count` | `integer` | count indicates the maximum number of unique volumes managed by the CSI driver that can be used on a node. A volume that is both attached and mounted on a node is considered to be used once, not twice. The same rule applies for a unique volume that is shared among multiple pods on the same node. If this field is not specified, then the supported number of volumes on this node is unbounded. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/storage.k8s.io/v1/csinodes`
  - `DELETE`: delete collection of CSINode
  - `GET`: list or watch objects of kind CSINode
  - `POST`: create a CSINode
- `/apis/storage.k8s.io/v1/watch/csinodes`
  - `GET`: watch individual changes to a list of CSINode. deprecated: use the 'watch' parameter with a list operation instead.
- `/apis/storage.k8s.io/v1/csinodes/{name}`
  - `DELETE`: delete a CSINode
  - `GET`: read the specified CSINode
  - `PATCH`: partially update the specified CSINode
  - `PUT`: replace the specified CSINode
- `/apis/storage.k8s.io/v1/watch/csinodes/{name}`
  - `GET`: watch changes to an object of kind CSINode. deprecated: use the 'watch' parameter with a list operation instead, filtered to a single item with the 'fieldSelector' parameter.

### /apis/storage.k8s.io/v1/csinodes {#_apisstoragek8siov1csinodes}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of CSINode
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
  list or watch objects of kind CSINode
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`CSINodeList`](/docs/rest_api/objects#io-k8s-api-storage-v1-CSINodeList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a CSINode
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`CSINode`](/docs/rest_api/storage_apis/csinode-storage-k8s-io-v1#csinode-storage-k8s-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`CSINode`](/docs/rest_api/storage_apis/csinode-storage-k8s-io-v1#csinode-storage-k8s-io-v1) schema |
| 201 - Created | [`CSINode`](/docs/rest_api/storage_apis/csinode-storage-k8s-io-v1#csinode-storage-k8s-io-v1) schema |
| 202 - Accepted | [`CSINode`](/docs/rest_api/storage_apis/csinode-storage-k8s-io-v1#csinode-storage-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/storage.k8s.io/v1/watch/csinodes {#_apisstoragek8siov1watchcsinodes}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch individual changes to a list of CSINode. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |

### /apis/storage.k8s.io/v1/csinodes/{name} {#_apisstoragek8siov1csinodes_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the CSINode |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete a CSINode
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`CSINode`](/docs/rest_api/storage_apis/csinode-storage-k8s-io-v1#csinode-storage-k8s-io-v1) schema |
| 202 - Accepted | [`CSINode`](/docs/rest_api/storage_apis/csinode-storage-k8s-io-v1#csinode-storage-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read the specified CSINode
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`CSINode`](/docs/rest_api/storage_apis/csinode-storage-k8s-io-v1#csinode-storage-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified CSINode
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`CSINode`](/docs/rest_api/storage_apis/csinode-storage-k8s-io-v1#csinode-storage-k8s-io-v1) schema |
| 201 - Created | [`CSINode`](/docs/rest_api/storage_apis/csinode-storage-k8s-io-v1#csinode-storage-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified CSINode
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`CSINode`](/docs/rest_api/storage_apis/csinode-storage-k8s-io-v1#csinode-storage-k8s-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`CSINode`](/docs/rest_api/storage_apis/csinode-storage-k8s-io-v1#csinode-storage-k8s-io-v1) schema |
| 201 - Created | [`CSINode`](/docs/rest_api/storage_apis/csinode-storage-k8s-io-v1#csinode-storage-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/storage.k8s.io/v1/watch/csinodes/{name} {#_apisstoragek8siov1watchcsinodes_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the CSINode |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch changes to an object of kind CSINode. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead, filtered to a single item with the &#x27;fieldSelector&#x27; parameter.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |
