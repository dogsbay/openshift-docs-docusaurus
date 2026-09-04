---
title: "ResourceSlice [resource.k8s.io/v1]"
sidebar_position: 12
---

# ResourceSlice [resource.k8s.io/v1] {#resourceslice-resource-k8s-io-v1}

<a id="resourceslice-resource-k8s-io-v1"></a>

Description
: ResourceSlice represents one or more resources in a pool of similar resources, managed by a common driver. A pool may span more than one ResourceSlice, and exactly how many ResourceSlices comprise a pool is determined by the driver.

  At the moment, the only supported resources are devices with attributes and capacities. Each device in a given pool, regardless of how many ResourceSlices, must have a unique name. The ResourceSlice in which a device gets published may change over time. The unique identifier for a device is the tuple <driver name>, <pool name>, <device name>.

  Whenever a driver needs to update a pool, it increments the pool.Spec.Pool.Generation number and updates all ResourceSlices with that new number and new resource definitions. A consumer must only use ResourceSlices with the highest generation number and ignore all others.

  When allocating all resources in a pool matching certain criteria or when looking for the best solution among several different alternatives, a consumer should check the number of ResourceSlices in a pool (included in each ResourceSlice) to determine whether its view of a pool is complete and if not, should wait until the driver has completed updating the pool.

  For resources that are not local to a node, the node name is not set. Instead, the driver may use a node selector to specify where the devices are available.

  This is an alpha type and requires enabling the DynamicResourceAllocation feature gate.

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
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | Standard object metadata |
| `spec` | `object` | ResourceSliceSpec contains the information published by the driver in one ResourceSlice. |

### .spec {#_spec}

Description
: ResourceSliceSpec contains the information published by the driver in one ResourceSlice.

Type
: ```
  `object`
  ```

Required
: - `driver`
  - `pool`

| Property | Type | Description |
| --- | --- | --- |
| `allNodes` | `boolean` | AllNodes indicates that all nodes have access to the resources in the pool.<br>Exactly one of NodeName, NodeSelector, AllNodes, and PerDeviceNodeSelection must be set. |
| `devices` | `array` | Devices lists some or all of the devices in this pool.<br>Must not have more than 128 entries. If any device uses taints or consumes counters the limit is 64.<br>Only one of Devices and SharedCounters can be set in a ResourceSlice. |
| `devices[]` | `object` | Device represents one individual hardware instance that can be selected based on its attributes. Besides the name, exactly one field must be set. |
| `driver` | `string` | Driver identifies the DRA driver providing the capacity information. A field selector can be used to list only ResourceSlice objects with a certain driver name.<br>Must be a DNS subdomain and should end with a DNS domain owned by the vendor of the driver. It should use only lower case characters. This field is immutable. |
| `nodeName` | `string` | NodeName identifies the node which provides the resources in this pool. A field selector can be used to list only ResourceSlice objects belonging to a certain node.<br>This field can be used to limit access from nodes to ResourceSlices with the same node name. It also indicates to autoscalers that adding new nodes of the same type as some old node might also make new resources available.<br>Exactly one of NodeName, NodeSelector, AllNodes, and PerDeviceNodeSelection must be set. This field is immutable. |
| `nodeSelector` | [`NodeSelector`](/docs/rest_api/objects#io-k8s-api-core-v1-NodeSelector) | NodeSelector defines which nodes have access to the resources in the pool, when that pool is not limited to a single node.<br>Must use exactly one term.<br>Exactly one of NodeName, NodeSelector, AllNodes, and PerDeviceNodeSelection must be set. |
| `perDeviceNodeSelection` | `boolean` | PerDeviceNodeSelection defines whether the access from nodes to resources in the pool is set on the ResourceSlice level or on each device. If it is set to true, every device defined the ResourceSlice must specify this individually.<br>Exactly one of NodeName, NodeSelector, AllNodes, and PerDeviceNodeSelection must be set. |
| `pool` | `object` | ResourcePool describes the pool that ResourceSlices belong to. |
| `sharedCounters` | `array` | SharedCounters defines a list of counter sets, each of which has a name and a list of counters available.<br>The names of the counter sets must be unique in the ResourcePool.<br>Only one of Devices and SharedCounters can be set in a ResourceSlice.<br>The maximum number of counter sets is 8. |
| `sharedCounters[]` | `object` | CounterSet defines a named set of counters that are available to be used by devices defined in the ResourcePool.<br>The counters are not allocatable by themselves, but can be referenced by devices. When a device is allocated, the portion of counters it uses will no longer be available for use by other devices. |

### .spec.devices {#_specdevices}

Description
: Devices lists some or all of the devices in this pool.

  Must not have more than 128 entries. If any device uses taints or consumes counters the limit is 64.

  Only one of Devices and SharedCounters can be set in a ResourceSlice.

Type
: ```
  `array`
  ```

### .spec.devices[] {#_specdevices}

Description
: Device represents one individual hardware instance that can be selected based on its attributes. Besides the name, exactly one field must be set.

Type
: ```
  `object`
  ```

Required
: - `name`

| Property | Type | Description |
| --- | --- | --- |
| `allNodes` | `boolean` | AllNodes indicates that all nodes have access to the device.<br>Must only be set if Spec.PerDeviceNodeSelection is set to true. At most one of NodeName, NodeSelector and AllNodes can be set. |
| `allowMultipleAllocations` | `boolean` | AllowMultipleAllocations marks whether the device is allowed to be allocated to multiple DeviceRequests.<br>If AllowMultipleAllocations is set to true, the device can be allocated more than once, and all of its capacity is consumable, regardless of whether the requestPolicy is defined or not. |
| `attributes` | `object` | Attributes defines the set of attributes for this device. The name of each attribute must be unique in that set.<br>The maximum number of attributes and capacities combined is 32. |
| `attributes{}` | `object` | DeviceAttribute must have exactly one field set. |
| `bindingConditions` | `array (string)` | BindingConditions defines the conditions for proceeding with binding. All of these conditions must be set in the per-device status conditions with a value of True to proceed with binding the pod to the node while scheduling the pod.<br>The maximum number of binding conditions is 4.<br>The conditions must be a valid condition type string.<br>This is an alpha field and requires enabling the DRADeviceBindingConditions and DRAResourceClaimDeviceStatus feature gates. |
| `bindingFailureConditions` | `array (string)` | BindingFailureConditions defines the conditions for binding failure. They may be set in the per-device status conditions. If any is set to "True", a binding failure occurred.<br>The maximum number of binding failure conditions is 4.<br>The conditions must be a valid condition type string.<br>This is an alpha field and requires enabling the DRADeviceBindingConditions and DRAResourceClaimDeviceStatus feature gates. |
| `bindsToNode` | `boolean` | BindsToNode indicates if the usage of an allocation involving this device has to be limited to exactly the node that was chosen when allocating the claim. If set to true, the scheduler will set the ResourceClaim.Status.Allocation.NodeSelector to match the node where the allocation was made.<br>This is an alpha field and requires enabling the DRADeviceBindingConditions and DRAResourceClaimDeviceStatus feature gates. |
| `capacity` | `object` | Capacity defines the set of capacities for this device. The name of each capacity must be unique in that set.<br>The maximum number of attributes and capacities combined is 32. |
| `capacity{}` | `object` | DeviceCapacity describes a quantity associated with a device. |
| `consumesCounters` | `array` | ConsumesCounters defines a list of references to sharedCounters and the set of counters that the device will consume from those counter sets.<br>There can only be a single entry per counterSet.<br>The maximum number of device counter consumptions per device is 2. |
| `consumesCounters[]` | `object` | DeviceCounterConsumption defines a set of counters that a device will consume from a CounterSet. |
| `name` | `string` | Name is unique identifier among all devices managed by the driver in the pool. It must be a DNS label. |
| `nodeName` | `string` | NodeName identifies the node where the device is available.<br>Must only be set if Spec.PerDeviceNodeSelection is set to true. At most one of NodeName, NodeSelector and AllNodes can be set. |
| `nodeSelector` | [`NodeSelector`](/docs/rest_api/objects#io-k8s-api-core-v1-NodeSelector) | NodeSelector defines the nodes where the device is available.<br>Must use exactly one term.<br>Must only be set if Spec.PerDeviceNodeSelection is set to true. At most one of NodeName, NodeSelector and AllNodes can be set. |
| `taints` | `array` | If specified, these are the driver-defined taints.<br>The maximum number of taints is 16. If taints are set for any device in a ResourceSlice, then the maximum number of allowed devices per ResourceSlice is 64 instead of 128.<br>This is an alpha field and requires enabling the DRADeviceTaints feature gate. |
| `taints[]` | `object` | The device this taint is attached to has the "effect" on any claim which does not tolerate the taint and, through the claim, to pods using the claim. |

### .spec.devices[].attributes {#_specdevicesattributes}

Description
: Attributes defines the set of attributes for this device. The name of each attribute must be unique in that set.

  The maximum number of attributes and capacities combined is 32.

Type
: ```
  `object`
  ```

### .spec.devices[].attributes{} {#_specdevicesattributes}

Description
: DeviceAttribute must have exactly one field set.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `bool` | `boolean` | BoolValue is a true/false value. |
| `int` | `integer` | IntValue is a number. |
| `string` | `string` | StringValue is a string. Must not be longer than 64 characters. |
| `version` | `string` | VersionValue is a semantic version according to semver.org spec 2.0.0. Must not be longer than 64 characters. |

### .spec.devices[].capacity {#_specdevicescapacity}

Description
: Capacity defines the set of capacities for this device. The name of each capacity must be unique in that set.

  The maximum number of attributes and capacities combined is 32.

Type
: ```
  `object`
  ```

### .spec.devices[].capacity{} {#_specdevicescapacity}

Description
: DeviceCapacity describes a quantity associated with a device.

Type
: ```
  `object`
  ```

Required
: - `value`

| Property | Type | Description |
| --- | --- | --- |
| `requestPolicy` | `object` | CapacityRequestPolicy defines how requests consume device capacity.<br>Must not set more than one ValidRequestValues. |
| `value` | [`Quantity`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-api-resource-Quantity) | Value defines how much of a certain capacity that device has.<br>This field reflects the fixed total capacity and does not change. The consumed amount is tracked separately by scheduler and does not affect this value. |

### .spec.devices[].capacity{}.requestPolicy {#_specdevicescapacityrequestpolicy}

Description
: CapacityRequestPolicy defines how requests consume device capacity.

  Must not set more than one ValidRequestValues.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `default` | [`Quantity`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-api-resource-Quantity) | Default specifies how much of this capacity is consumed by a request that does not contain an entry for it in DeviceRequest’s Capacity. |
| `validRange` | `object` | CapacityRequestPolicyRange defines a valid range for consumable capacity values.<br>  - If the requested amount is less than Min, it is rounded up to the Min value.   - If Step is set and the requested amount is between Min and Max but not aligned with Step,     it will be rounded up to the next value equal to Min + (n * Step).   - If Step is not set, the requested amount is used as-is if it falls within the range Min to Max (if set).   - If the requested or rounded amount exceeds Max (if set), the request does not satisfy the policy,     and the device cannot be allocated. |
| `validValues` | [`array (Quantity)`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-api-resource-Quantity) | ValidValues defines a set of acceptable quantity values in consuming requests.<br>Must not contain more than 10 entries. Must be sorted in ascending order.<br>If this field is set, Default must be defined and it must be included in ValidValues list.<br>If the requested amount does not match any valid value but smaller than some valid values, the scheduler calculates the smallest valid value that is greater than or equal to the request. That is: min(ceil(requestedValue) ∈ validValues), where requestedValue ≤ max(validValues).<br>If the requested amount exceeds all valid values, the request violates the policy, and this device cannot be allocated. |

### .spec.devices[].capacity{}.requestPolicy.validRange {#_specdevicescapacityrequestpolicyvalidrange}

Description
: CapacityRequestPolicyRange defines a valid range for consumable capacity values.

  - If the requested amount is less than Min, it is rounded up to the Min value.
  - If Step is set and the requested amount is between Min and Max but not aligned with Step, it will be rounded up to the next value equal to Min + (n * Step).
  - If Step is not set, the requested amount is used as-is if it falls within the range Min to Max (if set).
  - If the requested or rounded amount exceeds Max (if set), the request does not satisfy the policy, and the device cannot be allocated.

Type
: ```
  `object`
  ```

Required
: - `min`

| Property | Type | Description |
| --- | --- | --- |
| `max` | [`Quantity`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-api-resource-Quantity) | Max defines the upper limit for capacity that can be requested.<br>Max must be less than or equal to the capacity value. Min and requestPolicy.default must be less than or equal to the maximum. |
| `min` | [`Quantity`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-api-resource-Quantity) | Min specifies the minimum capacity allowed for a consumption request.<br>Min must be greater than or equal to zero, and less than or equal to the capacity value. requestPolicy.default must be more than or equal to the minimum. |
| `step` | [`Quantity`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-api-resource-Quantity) | Step defines the step size between valid capacity amounts within the range.<br>Max (if set) and requestPolicy.default must be a multiple of Step. Min + Step must be less than or equal to the capacity value. |

### .spec.devices[].consumesCounters {#_specdevicesconsumescounters}

Description
: ConsumesCounters defines a list of references to sharedCounters and the set of counters that the device will consume from those counter sets.

  There can only be a single entry per counterSet.

  The maximum number of device counter consumptions per device is 2.

Type
: ```
  `array`
  ```

### .spec.devices[].consumesCounters[] {#_specdevicesconsumescounters}

Description
: DeviceCounterConsumption defines a set of counters that a device will consume from a CounterSet.

Type
: ```
  `object`
  ```

Required
: - `counterSet`
  - `counters`

| Property | Type | Description |
| --- | --- | --- |
| `counterSet` | `string` | CounterSet is the name of the set from which the counters defined will be consumed. |
| `counters` | `object` | Counters defines the counters that will be consumed by the device.<br>The maximum number of counters is 32. |
| `counters{}` | `object` | Counter describes a quantity associated with a device. |

### .spec.devices[].consumesCounters[].counters {#_specdevicesconsumescounterscounters}

Description
: Counters defines the counters that will be consumed by the device.

  The maximum number of counters is 32.

Type
: ```
  `object`
  ```

### .spec.devices[].consumesCounters[].counters{} {#_specdevicesconsumescounterscounters}

Description
: Counter describes a quantity associated with a device.

Type
: ```
  `object`
  ```

Required
: - `value`

| Property | Type | Description |
| --- | --- | --- |
| `value` | [`Quantity`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-api-resource-Quantity) | Value defines how much of a certain device counter is available. |

### .spec.devices[].taints {#_specdevicestaints}

Description
: If specified, these are the driver-defined taints.

  The maximum number of taints is 16. If taints are set for any device in a ResourceSlice, then the maximum number of allowed devices per ResourceSlice is 64 instead of 128.

  This is an alpha field and requires enabling the DRADeviceTaints feature gate.

Type
: ```
  `array`
  ```

### .spec.devices[].taints[] {#_specdevicestaints}

Description
: The device this taint is attached to has the "effect" on any claim which does not tolerate the taint and, through the claim, to pods using the claim.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `effect`

| Property | Type | Description |
| --- | --- | --- |
| `effect` | `string` | The effect of the taint on claims that do not tolerate the taint and through such claims on the pods using them.<br>Valid effects are None, NoSchedule and NoExecute. PreferNoSchedule as used for nodes is not valid here. More effects may get added in the future. Consumers must treat unknown effects like None.<br>Possible enum values:  - `"NoExecute"` Evict any already-running pods that do not tolerate the device taint.  - `"NoSchedule"` Do not allow new pods to schedule which use a tainted device unless they tolerate the taint, but allow all pods submitted to Kubelet without going through the scheduler to start, and allow all already-running pods to continue running.  - `"None"` No effect, the taint is purely informational. |
| `key` | `string` | The taint key to be applied to a device. Must be a label name. |
| `timeAdded` | [`Time`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-Time) | TimeAdded represents the time at which the taint was added. Added automatically during create or update if not set. |
| `value` | `string` | The taint value corresponding to the taint key. Must be a label value. |

### .spec.pool {#_specpool}

Description
: ResourcePool describes the pool that ResourceSlices belong to.

Type
: ```
  `object`
  ```

Required
: - `name`
  - `generation`
  - `resourceSliceCount`

| Property | Type | Description |
| --- | --- | --- |
| `generation` | `integer` | Generation tracks the change in a pool over time. Whenever a driver changes something about one or more of the resources in a pool, it must change the generation in all ResourceSlices which are part of that pool. Consumers of ResourceSlices should only consider resources from the pool with the highest generation number. The generation may be reset by drivers, which should be fine for consumers, assuming that all ResourceSlices in a pool are updated to match or deleted.<br>Combined with ResourceSliceCount, this mechanism enables consumers to detect pools which are comprised of multiple ResourceSlices and are in an incomplete state. |
| `name` | `string` | Name is used to identify the pool. For node-local devices, this is often the node name, but this is not required.<br>It must not be longer than 253 characters and must consist of one or more DNS sub-domains separated by slashes. This field is immutable. |
| `resourceSliceCount` | `integer` | ResourceSliceCount is the total number of ResourceSlices in the pool at this generation number. Must be greater than zero.<br>Consumers can use this to check whether they have seen all ResourceSlices belonging to the same pool. |

### .spec.sharedCounters {#_specsharedcounters}

Description
: SharedCounters defines a list of counter sets, each of which has a name and a list of counters available.

  The names of the counter sets must be unique in the ResourcePool.

  Only one of Devices and SharedCounters can be set in a ResourceSlice.

  The maximum number of counter sets is 8.

Type
: ```
  `array`
  ```

### .spec.sharedCounters[] {#_specsharedcounters}

Description
: CounterSet defines a named set of counters that are available to be used by devices defined in the ResourcePool.

  The counters are not allocatable by themselves, but can be referenced by devices. When a device is allocated, the portion of counters it uses will no longer be available for use by other devices.

Type
: ```
  `object`
  ```

Required
: - `name`
  - `counters`

| Property | Type | Description |
| --- | --- | --- |
| `counters` | `object` | Counters defines the set of counters for this CounterSet The name of each counter must be unique in that set and must be a DNS label.<br>The maximum number of counters is 32. |
| `counters{}` | `object` | Counter describes a quantity associated with a device. |
| `name` | `string` | Name defines the name of the counter set. It must be a DNS label. |

### .spec.sharedCounters[].counters {#_specsharedcounterscounters}

Description
: Counters defines the set of counters for this CounterSet The name of each counter must be unique in that set and must be a DNS label.

  The maximum number of counters is 32.

Type
: ```
  `object`
  ```

### .spec.sharedCounters[].counters{} {#_specsharedcounterscounters}

Description
: Counter describes a quantity associated with a device.

Type
: ```
  `object`
  ```

Required
: - `value`

| Property | Type | Description |
| --- | --- | --- |
| `value` | [`Quantity`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-api-resource-Quantity) | Value defines how much of a certain device counter is available. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/resource.k8s.io/v1/resourceslices`
  - `DELETE`: delete collection of ResourceSlice
  - `GET`: list or watch objects of kind ResourceSlice
  - `POST`: create a ResourceSlice
- `/apis/resource.k8s.io/v1/watch/resourceslices`
  - `GET`: watch individual changes to a list of ResourceSlice. deprecated: use the 'watch' parameter with a list operation instead.
- `/apis/resource.k8s.io/v1/resourceslices/{name}`
  - `DELETE`: delete a ResourceSlice
  - `GET`: read the specified ResourceSlice
  - `PATCH`: partially update the specified ResourceSlice
  - `PUT`: replace the specified ResourceSlice
- `/apis/resource.k8s.io/v1/watch/resourceslices/{name}`
  - `GET`: watch changes to an object of kind ResourceSlice. deprecated: use the 'watch' parameter with a list operation instead, filtered to a single item with the 'fieldSelector' parameter.

### /apis/resource.k8s.io/v1/resourceslices {#_apisresourcek8siov1resourceslices}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of ResourceSlice
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
  list or watch objects of kind ResourceSlice
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ResourceSliceList`](/docs/rest_api/objects#io-k8s-api-resource-v1-ResourceSliceList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a ResourceSlice
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`ResourceSlice`](/docs/rest_api/schedule_and_quota_apis/resourceslice-resource-k8s-io-v1#resourceslice-resource-k8s-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ResourceSlice`](/docs/rest_api/schedule_and_quota_apis/resourceslice-resource-k8s-io-v1#resourceslice-resource-k8s-io-v1) schema |
| 201 - Created | [`ResourceSlice`](/docs/rest_api/schedule_and_quota_apis/resourceslice-resource-k8s-io-v1#resourceslice-resource-k8s-io-v1) schema |
| 202 - Accepted | [`ResourceSlice`](/docs/rest_api/schedule_and_quota_apis/resourceslice-resource-k8s-io-v1#resourceslice-resource-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/resource.k8s.io/v1/watch/resourceslices {#_apisresourcek8siov1watchresourceslices}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch individual changes to a list of ResourceSlice. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |

### /apis/resource.k8s.io/v1/resourceslices/{name} {#_apisresourcek8siov1resourceslices_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the ResourceSlice |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete a ResourceSlice
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ResourceSlice`](/docs/rest_api/schedule_and_quota_apis/resourceslice-resource-k8s-io-v1#resourceslice-resource-k8s-io-v1) schema |
| 202 - Accepted | [`ResourceSlice`](/docs/rest_api/schedule_and_quota_apis/resourceslice-resource-k8s-io-v1#resourceslice-resource-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read the specified ResourceSlice
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ResourceSlice`](/docs/rest_api/schedule_and_quota_apis/resourceslice-resource-k8s-io-v1#resourceslice-resource-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified ResourceSlice
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ResourceSlice`](/docs/rest_api/schedule_and_quota_apis/resourceslice-resource-k8s-io-v1#resourceslice-resource-k8s-io-v1) schema |
| 201 - Created | [`ResourceSlice`](/docs/rest_api/schedule_and_quota_apis/resourceslice-resource-k8s-io-v1#resourceslice-resource-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified ResourceSlice
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`ResourceSlice`](/docs/rest_api/schedule_and_quota_apis/resourceslice-resource-k8s-io-v1#resourceslice-resource-k8s-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`ResourceSlice`](/docs/rest_api/schedule_and_quota_apis/resourceslice-resource-k8s-io-v1#resourceslice-resource-k8s-io-v1) schema |
| 201 - Created | [`ResourceSlice`](/docs/rest_api/schedule_and_quota_apis/resourceslice-resource-k8s-io-v1#resourceslice-resource-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/resource.k8s.io/v1/watch/resourceslices/{name} {#_apisresourcek8siov1watchresourceslices_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the ResourceSlice |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  watch changes to an object of kind ResourceSlice. deprecated: use the &#x27;watch&#x27; parameter with a list operation instead, filtered to a single item with the &#x27;fieldSelector&#x27; parameter.
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`WatchEvent`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-WatchEvent) schema |
| 401 - Unauthorized | Empty |
