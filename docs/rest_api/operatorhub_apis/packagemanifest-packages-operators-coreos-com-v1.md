---
title: "PackageManifest [packages.operators.coreos.com/v1]"
sidebar_position: 12
---

# PackageManifest [packages.operators.coreos.com/v1] {#packagemanifest-packages-operators-coreos-com-v1}

<a id="packagemanifest-packages-operators-coreos-com-v1"></a>

Description
: PackageManifest holds information about a package, which is a reference to one (or more) channels under a single package.

Type
: ```
  `object`
  ```

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) |  |
| `spec` | `object` | PackageManifestSpec defines the desired state of PackageManifest |
| `status` | `object` | PackageManifestStatus represents the current status of the PackageManifest |

### .spec {#_spec}

Description
: PackageManifestSpec defines the desired state of PackageManifest

Type
: ```
  `object`
  ```

### .status {#_status}

Description
: PackageManifestStatus represents the current status of the PackageManifest

Type
: ```
  `object`
  ```

Required
: - `catalogSource`
  - `catalogSourceDisplayName`
  - `catalogSourcePublisher`
  - `catalogSourceNamespace`
  - `packageName`
  - `channels`
  - `defaultChannel`

| Property | Type | Description |
| --- | --- | --- |
| `catalogSource` | `string` | CatalogSource is the name of the CatalogSource this package belongs to |
| `catalogSourceDisplayName` | `string` |  |
| `catalogSourceNamespace` | `string` | CatalogSourceNamespace is the namespace of the owning CatalogSource |
| `catalogSourcePublisher` | `string` |  |
| `channels` | `array` | Channels are the declared channels for the package, ala `stable` or `alpha`. |
| `channels[]` | `object` | PackageChannel defines a single channel under a package, pointing to a version of that package. |
| `defaultChannel` | `string` | DefaultChannel is, if specified, the name of the default channel for the package. The default channel will be installed if no other channel is explicitly given. If the package has a single channel, then that channel is implicitly the default. |
| `deprecation` | `object` | Deprecation conveys information regarding a deprecated resource. |
| `packageName` | `string` | PackageName is the name of the overall package, ala `etcd`. |
| `provider` | `object` | AppLink defines a link to an application |

### .status.channels {#_statuschannels}

Description
: Channels are the declared channels for the package, ala `stable` or `alpha`.

Type
: ```
  `array`
  ```

### .status.channels[] {#_statuschannels}

Description
: PackageChannel defines a single channel under a package, pointing to a version of that package.

Type
: ```
  `object`
  ```

Required
: - `name`
  - `currentCSV`
  - `entries`

| Property | Type | Description |
| --- | --- | --- |
| `currentCSV` | `string` | CurrentCSV defines a reference to the CSV holding the version of this package currently for the channel. |
| `currentCSVDesc` | `object` | CSVDescription defines a description of a CSV |
| `deprecation` | `object` | Deprecation conveys information regarding a deprecated resource. |
| `entries` | `array` | Entries lists all CSVs in the channel, with their upgrade edges. |
| `entries[]` | `object` | ChannelEntry defines a member of a package channel. |
| `name` | `string` | Name is the name of the channel, e.g. `alpha` or `stable` |

### .status.channels[].currentCSVDesc {#_statuschannelscurrentcsvdesc}

Description
: CSVDescription defines a description of a CSV

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `annotations` | `object (string)` |  |
| `apiservicedefinitions` | [`APIServiceDefinitions`](/docs/rest_api/objects#com-github-operator-framework-api-pkg-operators-v1alpha1-APIServiceDefinitions) |  |
| `customresourcedefinitions` | [`CustomResourceDefinitions`](/docs/rest_api/objects#com-github-operator-framework-api-pkg-operators-v1alpha1-CustomResourceDefinitions) |  |
| `description` | `string` | LongDescription is the CSV’s description |
| `displayName` | `string` | DisplayName is the CSV’s display name |
| `icon` | `array` | Icon is the CSV’s base64 encoded icon |
| `icon[]` | `object` | Icon defines a base64 encoded icon and media type |
| `installModes` | [`array (InstallMode)`](/docs/rest_api/objects#com-github-operator-framework-api-pkg-operators-v1alpha1-InstallMode) | InstallModes specify supported installation types |
| `keywords` | `array (string)` |  |
| `links` | `array` |  |
| `links[]` | `object` | AppLink defines a link to an application |
| `maintainers` | `array` |  |
| `maintainers[]` | `object` | Maintainer defines a project maintainer |
| `maturity` | `string` |  |
| `minKubeVersion` | `string` | Minimum Kubernetes version for operator installation |
| `nativeApis` | [`array (GroupVersionKind)`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-GroupVersionKind) |  |
| `provider` | `object` | AppLink defines a link to an application |
| `relatedImages` | `array (string)` | List of related images |
| `version` | [`OperatorVersion`](/docs/rest_api/objects#com-github-operator-framework-api-pkg-operators-lib-version-OperatorVersion) | Version is the CSV’s semantic version |

### .status.channels[].currentCSVDesc.icon {#_statuschannelscurrentcsvdescicon}

Description
: Icon is the CSV’s base64 encoded icon

Type
: ```
  `array`
  ```

### .status.channels[].currentCSVDesc.icon[] {#_statuschannelscurrentcsvdescicon}

Description
: Icon defines a base64 encoded icon and media type

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `base64data` | `string` |  |
| `mediatype` | `string` |  |

### .status.channels[].currentCSVDesc.links {#_statuschannelscurrentcsvdesclinks}

Description

Type
: ```
  `array`
  ```

### .status.channels[].currentCSVDesc.links[] {#_statuschannelscurrentcsvdesclinks}

Description
: AppLink defines a link to an application

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `name` | `string` |  |
| `url` | `string` |  |

### .status.channels[].currentCSVDesc.maintainers {#_statuschannelscurrentcsvdescmaintainers}

Description

Type
: ```
  `array`
  ```

### .status.channels[].currentCSVDesc.maintainers[] {#_statuschannelscurrentcsvdescmaintainers}

Description
: Maintainer defines a project maintainer

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `email` | `string` |  |
| `name` | `string` |  |

### .status.channels[].currentCSVDesc.provider {#_statuschannelscurrentcsvdescprovider}

Description
: AppLink defines a link to an application

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `name` | `string` |  |
| `url` | `string` |  |

### .status.channels[].deprecation {#_statuschannelsdeprecation}

Description
: Deprecation conveys information regarding a deprecated resource.

Type
: ```
  `object`
  ```

Required
: - `message`

| Property | Type | Description |
| --- | --- | --- |
| `message` | `string` | Message is a human readable message describing the deprecation. |

### .status.channels[].entries {#_statuschannelsentries}

Description
: Entries lists all CSVs in the channel, with their upgrade edges.

Type
: ```
  `array`
  ```

### .status.channels[].entries[] {#_statuschannelsentries}

Description
: ChannelEntry defines a member of a package channel.

Type
: ```
  `object`
  ```

Required
: - `name`

| Property | Type | Description |
| --- | --- | --- |
| `deprecation` | `object` | Deprecation conveys information regarding a deprecated resource. |
| `name` | `string` | Name is the name of the bundle for this entry. |
| `version` | `string` | Version is the version of the bundle for this entry. |

### .status.channels[].entries[].deprecation {#_statuschannelsentriesdeprecation}

Description
: Deprecation conveys information regarding a deprecated resource.

Type
: ```
  `object`
  ```

Required
: - `message`

| Property | Type | Description |
| --- | --- | --- |
| `message` | `string` | Message is a human readable message describing the deprecation. |

### .status.deprecation {#_statusdeprecation}

Description
: Deprecation conveys information regarding a deprecated resource.

Type
: ```
  `object`
  ```

Required
: - `message`

| Property | Type | Description |
| --- | --- | --- |
| `message` | `string` | Message is a human readable message describing the deprecation. |

### .status.provider {#_statusprovider}

Description
: AppLink defines a link to an application

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `name` | `string` |  |
| `url` | `string` |  |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/packages.operators.coreos.com/v1/packagemanifests`
  - `GET`: list objects of kind PackageManifest
- `/apis/packages.operators.coreos.com/v1/namespaces/{namespace}/packagemanifests`
  - `GET`: list objects of kind PackageManifest
- `/apis/packages.operators.coreos.com/v1/namespaces/{namespace}/packagemanifests/{name}`
  - `GET`: read the specified PackageManifest
- `/apis/packages.operators.coreos.com/v1/namespaces/{namespace}/packagemanifests/{name}/icon`
  - `GET`: connect GET requests to icon of PackageManifest

### /apis/packages.operators.coreos.com/v1/packagemanifests {#_apispackagesoperatorscoreoscomv1packagemanifests}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  list objects of kind PackageManifest
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`PackageManifestList`](/docs/rest_api/objects#com-github-operator-framework-operator-lifecycle-manager-pkg-package-server-apis-operators-v1-PackageManifestList) schema |

### /apis/packages.operators.coreos.com/v1/namespaces/{namespace}/packagemanifests {#_apispackagesoperatorscoreoscomv1namespaces_namespace_packagemanifests}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  list objects of kind PackageManifest
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`PackageManifestList`](/docs/rest_api/objects#com-github-operator-framework-operator-lifecycle-manager-pkg-package-server-apis-operators-v1-PackageManifestList) schema |

### /apis/packages.operators.coreos.com/v1/namespaces/{namespace}/packagemanifests/{name} {#_apispackagesoperatorscoreoscomv1namespaces_namespace_packagemanifests_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the PackageManifest |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read the specified PackageManifest
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`PackageManifest`](/docs/rest_api/operatorhub_apis/packagemanifest-packages-operators-coreos-com-v1#packagemanifest-packages-operators-coreos-com-v1) schema |

### /apis/packages.operators.coreos.com/v1/namespaces/{namespace}/packagemanifests/{name}/icon {#_apispackagesoperatorscoreoscomv1namespaces_namespace_packagemanifests_name_icon}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the PackageManifest |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  connect GET requests to icon of PackageManifest
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | `string` |
