---
title: "HardwareData [metal3.io/v1alpha1]"
sidebar_position: 6
---

# HardwareData [metal3.io/v1alpha1] {#hardwaredata-metal3-io-v1alpha1}

<a id="hardwaredata-metal3-io-v1alpha1"></a>

Description
: HardwareData is the Schema for the hardwaredata API.

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
| `spec` | `object` | HardwareDataSpec defines the desired state of HardwareData. |

### .spec {#_spec}

Description
: HardwareDataSpec defines the desired state of HardwareData.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `hardware` | `object` | The hardware discovered on the host during its inspection. |

### .spec.hardware {#_spechardware}

Description
: The hardware discovered on the host during its inspection.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `cpu` | `object` | Details of the CPU(s) in the system. |
| `firmware` | `object` | System firmware information. |
| `hostname` | `string` | Name of the host at the inspection time. |
| `nics` | `array` | List of network interfaces for the host. |
| `nics[]` | `object` | NIC describes one network interface on the host. |
| `ramMebibytes` | `integer` | The host’s amount of memory in Mebibytes. |
| `storage` | `array` | List of storage (disk, SSD, etc.) available to the host. |
| `storage[]` | `object` | Storage describes one storage device (disk, SSD, etc.) on the host. |
| `systemVendor` | `object` | System vendor information. |

### .spec.hardware.cpu {#_spechardwarecpu}

Description
: Details of the CPU(s) in the system.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `arch` | `string` |  |
| `clockMegahertz` | `number` | ClockSpeed is a clock speed in MHz |
| `count` | `integer` |  |
| `flags` | `array (string)` |  |
| `model` | `string` |  |

### .spec.hardware.firmware {#_spechardwarefirmware}

Description
: System firmware information.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `bios` | `object` | The BIOS for this firmware |

### .spec.hardware.firmware.bios {#_spechardwarefirmwarebios}

Description
: The BIOS for this firmware

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `date` | `string` | The release/build date for this BIOS |
| `vendor` | `string` | The vendor name for this BIOS |
| `version` | `string` | The version of the BIOS |

### .spec.hardware.nics {#_spechardwarenics}

Description
: List of network interfaces for the host.

Type
: ```
  `array`
  ```

### .spec.hardware.nics[] {#_spechardwarenics}

Description
: NIC describes one network interface on the host.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ip` | `string` | The IP address of the interface. This will be an IPv4 or IPv6 address if one is present.  If both IPv4 and IPv6 addresses are present in a dual-stack environment, two nics will be output, one with each IP. |
| `lldp` | `object` | LLDP data for this interface |
| `mac` | `string` | The device MAC address |
| `model` | `string` | The vendor and product IDs of the NIC, e.g. "0x8086 0x1572" |
| `name` | `string` | The name of the network interface, e.g. "en0" |
| `pciAddress` | `string` | The NIC PCI address |
| `pxe` | `boolean` | Whether the NIC is PXE Bootable |
| `speedGbps` | `integer` | The speed of the device in Gigabits per second |
| `vlanId` | `integer` | The untagged VLAN ID |
| `vlans` | `array` | The VLANs available |
| `vlans[]` | `object` | VLAN represents the name and ID of a VLAN. |

### .spec.hardware.nics[].lldp {#_spechardwarenicslldp}

Description
: LLDP data for this interface

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `portID` | `string` | The switch port ID from LLDP |
| `switchID` | `string` | The switch chassis ID from LLDP |
| `switchSystemName` | `string` | The switch system name from LLDP |

### .spec.hardware.nics[].vlans {#_spechardwarenicsvlans}

Description
: The VLANs available

Type
: ```
  `array`
  ```

### .spec.hardware.nics[].vlans[] {#_spechardwarenicsvlans}

Description
: VLAN represents the name and ID of a VLAN.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `id` | `integer` | VLANID is a 12-bit 802.1Q VLAN identifier |
| `name` | `string` |  |

### .spec.hardware.storage {#_spechardwarestorage}

Description
: List of storage (disk, SSD, etc.) available to the host.

Type
: ```
  `array`
  ```

### .spec.hardware.storage[] {#_spechardwarestorage}

Description
: Storage describes one storage device (disk, SSD, etc.) on the host.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `alternateNames` | `array (string)` | A list of alternate Linux device names of the disk, e.g. "/dev/sda". Note that this list is not exhaustive, and names may not be stable across reboots. |
| `hctl` | `string` | The SCSI location of the device |
| `model` | `string` | Hardware model |
| `name` | `string` | A Linux device name of the disk, e.g. "/dev/disk/by-path/pci-0000:01:00.0-scsi-0:2:0:0". This will be a name that is stable across reboots if one is available. |
| `rotational` | `boolean` | Whether this disk represents rotational storage. This field is not recommended for usage, please prefer using 'Type' field instead, this field will be deprecated eventually. |
| `serialNumber` | `string` | The serial number of the device |
| `sizeBytes` | `integer` | The size of the disk in Bytes |
| `type` | `string` | Device type, one of: HDD, SSD, NVME. |
| `vendor` | `string` | The name of the vendor of the device |
| `wwn` | `string` | The WWN of the device |
| `wwnVendorExtension` | `string` | The WWN Vendor extension of the device |
| `wwnWithExtension` | `string` | The WWN with the extension |

### .spec.hardware.systemVendor {#_spechardwaresystemvendor}

Description
: System vendor information.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `manufacturer` | `string` |  |
| `productName` | `string` |  |
| `serialNumber` | `string` |  |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/metal3.io/v1alpha1/hardwaredata`
  - `GET`: list objects of kind HardwareData
- `/apis/metal3.io/v1alpha1/namespaces/{namespace}/hardwaredata`
  - `DELETE`: delete collection of HardwareData
  - `GET`: list objects of kind HardwareData
  - `POST`: create a HardwareData
- `/apis/metal3.io/v1alpha1/namespaces/{namespace}/hardwaredata/{name}`
  - `DELETE`: delete a HardwareData
  - `GET`: read the specified HardwareData
  - `PATCH`: partially update the specified HardwareData
  - `PUT`: replace the specified HardwareData

### /apis/metal3.io/v1alpha1/hardwaredata {#_apismetal3iov1alpha1hardwaredata}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  list objects of kind HardwareData
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`HardwareDataList`](/docs/rest_api/objects#io-metal3-v1alpha1-HardwareDataList) schema |
| 401 - Unauthorized | Empty |

### /apis/metal3.io/v1alpha1/namespaces/{namespace}/hardwaredata {#_apismetal3iov1alpha1namespaces_namespace_hardwaredata}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of HardwareData
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
  list objects of kind HardwareData
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`HardwareDataList`](/docs/rest_api/objects#io-metal3-v1alpha1-HardwareDataList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a HardwareData
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`HardwareData`](/docs/rest_api/provisioning_apis/hardwaredata-metal3-io-v1alpha1#hardwaredata-metal3-io-v1alpha1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`HardwareData`](/docs/rest_api/provisioning_apis/hardwaredata-metal3-io-v1alpha1#hardwaredata-metal3-io-v1alpha1) schema |
| 201 - Created | [`HardwareData`](/docs/rest_api/provisioning_apis/hardwaredata-metal3-io-v1alpha1#hardwaredata-metal3-io-v1alpha1) schema |
| 202 - Accepted | [`HardwareData`](/docs/rest_api/provisioning_apis/hardwaredata-metal3-io-v1alpha1#hardwaredata-metal3-io-v1alpha1) schema |
| 401 - Unauthorized | Empty |

### /apis/metal3.io/v1alpha1/namespaces/{namespace}/hardwaredata/{name} {#_apismetal3iov1alpha1namespaces_namespace_hardwaredata_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the HardwareData |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete a HardwareData
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
  read the specified HardwareData
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`HardwareData`](/docs/rest_api/provisioning_apis/hardwaredata-metal3-io-v1alpha1#hardwaredata-metal3-io-v1alpha1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified HardwareData
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`HardwareData`](/docs/rest_api/provisioning_apis/hardwaredata-metal3-io-v1alpha1#hardwaredata-metal3-io-v1alpha1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified HardwareData
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`HardwareData`](/docs/rest_api/provisioning_apis/hardwaredata-metal3-io-v1alpha1#hardwaredata-metal3-io-v1alpha1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`HardwareData`](/docs/rest_api/provisioning_apis/hardwaredata-metal3-io-v1alpha1#hardwaredata-metal3-io-v1alpha1) schema |
| 201 - Created | [`HardwareData`](/docs/rest_api/provisioning_apis/hardwaredata-metal3-io-v1alpha1#hardwaredata-metal3-io-v1alpha1) schema |
| 401 - Unauthorized | Empty |
