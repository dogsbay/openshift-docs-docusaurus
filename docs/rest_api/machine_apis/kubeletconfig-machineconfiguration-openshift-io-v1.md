---
title: "KubeletConfig [machineconfiguration.openshift.io/v1]"
sidebar_position: 5
---

# KubeletConfig [machineconfiguration.openshift.io/v1] {#kubeletconfig-machineconfiguration-openshift-io-v1}

<a id="kubeletconfig-machineconfiguration-openshift-io-v1"></a>

Description
: KubeletConfig describes a customized Kubelet configuration.

  Compatibility level 1: Stable within a major release for a minimum of 12 months or 3 minor releases (whichever is longer).

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
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | Standard object’s metadata. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata |
| `spec` | `object` | spec contains the desired kubelet configuration. |
| `status` | `object` | status contains observed information about the kubelet configuration. |

### .spec {#_spec}

Description
: spec contains the desired kubelet configuration.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `autoSizingReserved` | `boolean` | autoSizingReserved controls whether system-reserved CPU and memory are automatically calculated based on each node’s installed capacity. When set to true, this prevents node failure from resource starvation of system components (kubelet, CRI-O) without manual configuration. When omitted, this means the user has no opinion and the platform is left to choose a reasonable default, which is subject to change over time. The current default is true for worker nodes and false for control plane nodes. When set to false, automatic resource reservation is disabled and manual settings must be configured. |
| `kubeletConfig` | `` | kubeletConfig contains upstream Kubernetes kubelet configuration fields. Values are validated by the kubelet itself. Invalid values may render nodes unusable. Refer to OpenShift documentation for the Kubernetes version corresponding to your OpenShift release to find valid kubelet configuration options. |
| `logLevel` | `integer` | logLevel sets the kubelet log verbosity, controlling the amount of detail in kubelet logs. Valid values range from 0 (minimal logging) to 10 (maximum verbosity with trace-level detail). Higher log levels may impact node performance. When omitted, the platform chooses a reasonable default, which is subject to change over time. The current default is 2 (standard informational logging). |
| `machineConfigPoolSelector` | `object` | machineConfigPoolSelector selects which pools the KubeletConfig should apply to. When omitted or set to an empty selector {}, no pools are selected, which is equivalent to not matching any MachineConfigPool. |
| `tlsSecurityProfile` | `object` | tlsSecurityProfile configures TLS settings for the kubelet. When omitted, the TLS configuration defaults to the value from apiservers.config.openshift.io/cluster. When specified, the type field can be set to either "Old", "Intermediate", "Modern", "Custom" or omitted for backward compatibility. |

### .spec.machineConfigPoolSelector {#_specmachineconfigpoolselector}

Description
: machineConfigPoolSelector selects which pools the KubeletConfig should apply to. When omitted or set to an empty selector {}, no pools are selected, which is equivalent to not matching any MachineConfigPool.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `matchExpressions` | `array` | matchExpressions is a list of label selector requirements. The requirements are ANDed. |
| `matchExpressions[]` | `object` | A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values. |
| `matchLabels` | `object (string)` | matchLabels is a map of {key,value} pairs. A single {key,value} in the matchLabels map is equivalent to an element of matchExpressions, whose key field is "key", the operator is "In", and the values array contains only "value". The requirements are ANDed. |

### .spec.machineConfigPoolSelector.matchExpressions {#_specmachineconfigpoolselectormatchexpressions}

Description
: matchExpressions is a list of label selector requirements. The requirements are ANDed.

Type
: ```
  `array`
  ```

### .spec.machineConfigPoolSelector.matchExpressions[] {#_specmachineconfigpoolselectormatchexpressions}

Description
: A label selector requirement is a selector that contains values, a key, and an operator that relates the key and values.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `operator`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key is the label key that the selector applies to. |
| `operator` | `string` | operator represents a key’s relationship to a set of values. Valid operators are In, NotIn, Exists and DoesNotExist. |
| `values` | `array (string)` | values is an array of string values. If the operator is In or NotIn, the values array must be non-empty. If the operator is Exists or DoesNotExist, the values array must be empty. This array is replaced during a strategic merge patch. |

### .spec.tlsSecurityProfile {#_spectlssecurityprofile}

Description
: tlsSecurityProfile configures TLS settings for the kubelet. When omitted, the TLS configuration defaults to the value from apiservers.config.openshift.io/cluster. When specified, the type field can be set to either "Old", "Intermediate", "Modern", "Custom" or omitted for backward compatibility.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `custom` | `` | custom is a user-defined TLS security profile. Be extremely careful using a custom profile as invalid configurations can be catastrophic. An example custom profile looks like this:<br>  minTLSVersion: VersionTLS11   ciphers:     - ECDHE-ECDSA-CHACHA20-POLY1305     - ECDHE-RSA-CHACHA20-POLY1305     - ECDHE-RSA-AES128-GCM-SHA256     - ECDHE-ECDSA-AES128-GCM-SHA256 |
| `intermediate` | `` | intermediate is a TLS profile for use when you do not need compatibility with legacy clients and want to remain highly secure while being compatible with most clients currently in use.<br>This profile is equivalent to a Custom profile specified as:   minTLSVersion: VersionTLS12   ciphers:     - TLS_AES_128_GCM_SHA256     - TLS_AES_256_GCM_SHA384     - TLS_CHACHA20_POLY1305_SHA256     - ECDHE-ECDSA-AES128-GCM-SHA256     - ECDHE-RSA-AES128-GCM-SHA256     - ECDHE-ECDSA-AES256-GCM-SHA384     - ECDHE-RSA-AES256-GCM-SHA384     - ECDHE-ECDSA-CHACHA20-POLY1305     - ECDHE-RSA-CHACHA20-POLY1305 |
| `modern` | `` | modern is a TLS security profile for use with clients that support TLS 1.3 and do not need backward compatibility for older clients.<br>This profile is equivalent to a Custom profile specified as:   minTLSVersion: VersionTLS13   ciphers:     - TLS_AES_128_GCM_SHA256     - TLS_AES_256_GCM_SHA384     - TLS_CHACHA20_POLY1305_SHA256 |
| `old` | `` | old is a TLS profile for use when services need to be accessed by very old clients or libraries and should be used only as a last resort.<br>This profile is equivalent to a Custom profile specified as:   minTLSVersion: VersionTLS10   ciphers:     - TLS_AES_128_GCM_SHA256     - TLS_AES_256_GCM_SHA384     - TLS_CHACHA20_POLY1305_SHA256     - ECDHE-ECDSA-AES128-GCM-SHA256     - ECDHE-RSA-AES128-GCM-SHA256     - ECDHE-ECDSA-AES256-GCM-SHA384     - ECDHE-RSA-AES256-GCM-SHA384     - ECDHE-ECDSA-CHACHA20-POLY1305     - ECDHE-RSA-CHACHA20-POLY1305     - ECDHE-ECDSA-AES128-SHA256     - ECDHE-RSA-AES128-SHA256     - ECDHE-ECDSA-AES128-SHA     - ECDHE-RSA-AES128-SHA     - ECDHE-ECDSA-AES256-SHA     - ECDHE-RSA-AES256-SHA     - AES128-GCM-SHA256     - AES256-GCM-SHA384     - AES128-SHA256     - AES128-SHA     - AES256-SHA     - DES-CBC3-SHA |
| `type` | `string` | type is one of Old, Intermediate, Modern or Custom. Custom provides the ability to specify individual TLS security profile parameters.<br>The profiles are based on version 5.7 of the Mozilla Server Side TLS configuration guidelines. The cipher lists consist of the configuration’s "ciphersuites" followed by the Go-specific "ciphers" from the guidelines. See: https://ssl-config.mozilla.org/guidelines/5.7.json<br>The profiles are intent based, so they may change over time as new ciphers are developed and existing ciphers are found to be insecure. Depending on precisely which ciphers are available to a process, the list may be reduced. |

### .status {#_status}

Description
: status contains observed information about the kubelet configuration.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `conditions` | `array` | conditions represents the latest available observations of current state. |
| `conditions[]` | `object` | KubeletConfigCondition defines the state of the KubeletConfig |
| `observedGeneration` | `integer` | observedGeneration represents the generation observed by the controller. |

### .status.conditions {#_statusconditions}

Description
: conditions represents the latest available observations of current state.

Type
: ```
  `array`
  ```

### .status.conditions[] {#_statusconditions}

Description
: KubeletConfigCondition defines the state of the KubeletConfig

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `lastTransitionTime` | `` | lastTransitionTime is the time of the last update to the current status object. |
| `message` | `string` | message provides additional information about the current condition. This is only to be consumed by humans. |
| `reason` | `string` | reason is the reason for the condition’s last transition.  Reasons are PascalCase |
| `status` | `string` | status of the condition, one of True, False, Unknown. |
| `type` | `string` | type specifies the state of the operator’s reconciliation functionality. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/machineconfiguration.openshift.io/v1/kubeletconfigs`
  - `DELETE`: delete collection of KubeletConfig
  - `GET`: list objects of kind KubeletConfig
  - `POST`: create a KubeletConfig
- `/apis/machineconfiguration.openshift.io/v1/kubeletconfigs/{name}`
  - `DELETE`: delete a KubeletConfig
  - `GET`: read the specified KubeletConfig
  - `PATCH`: partially update the specified KubeletConfig
  - `PUT`: replace the specified KubeletConfig
- `/apis/machineconfiguration.openshift.io/v1/kubeletconfigs/{name}/status`
  - `GET`: read status of the specified KubeletConfig
  - `PATCH`: partially update status of the specified KubeletConfig
  - `PUT`: replace status of the specified KubeletConfig

### /apis/machineconfiguration.openshift.io/v1/kubeletconfigs {#_apismachineconfigurationopenshiftiov1kubeletconfigs}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of KubeletConfig
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
  list objects of kind KubeletConfig
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`KubeletConfigList`](/docs/rest_api/objects#io-openshift-machineconfiguration-v1-KubeletConfigList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a KubeletConfig
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`KubeletConfig`](/docs/rest_api/machine_apis/kubeletconfig-machineconfiguration-openshift-io-v1#kubeletconfig-machineconfiguration-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`KubeletConfig`](/docs/rest_api/machine_apis/kubeletconfig-machineconfiguration-openshift-io-v1#kubeletconfig-machineconfiguration-openshift-io-v1) schema |
| 201 - Created | [`KubeletConfig`](/docs/rest_api/machine_apis/kubeletconfig-machineconfiguration-openshift-io-v1#kubeletconfig-machineconfiguration-openshift-io-v1) schema |
| 202 - Accepted | [`KubeletConfig`](/docs/rest_api/machine_apis/kubeletconfig-machineconfiguration-openshift-io-v1#kubeletconfig-machineconfiguration-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/machineconfiguration.openshift.io/v1/kubeletconfigs/{name} {#_apismachineconfigurationopenshiftiov1kubeletconfigs_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the KubeletConfig |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete a KubeletConfig
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
  read the specified KubeletConfig
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`KubeletConfig`](/docs/rest_api/machine_apis/kubeletconfig-machineconfiguration-openshift-io-v1#kubeletconfig-machineconfiguration-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified KubeletConfig
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`KubeletConfig`](/docs/rest_api/machine_apis/kubeletconfig-machineconfiguration-openshift-io-v1#kubeletconfig-machineconfiguration-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified KubeletConfig
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`KubeletConfig`](/docs/rest_api/machine_apis/kubeletconfig-machineconfiguration-openshift-io-v1#kubeletconfig-machineconfiguration-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`KubeletConfig`](/docs/rest_api/machine_apis/kubeletconfig-machineconfiguration-openshift-io-v1#kubeletconfig-machineconfiguration-openshift-io-v1) schema |
| 201 - Created | [`KubeletConfig`](/docs/rest_api/machine_apis/kubeletconfig-machineconfiguration-openshift-io-v1#kubeletconfig-machineconfiguration-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/machineconfiguration.openshift.io/v1/kubeletconfigs/{name}/status {#_apismachineconfigurationopenshiftiov1kubeletconfigs_name_status}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the KubeletConfig |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read status of the specified KubeletConfig
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`KubeletConfig`](/docs/rest_api/machine_apis/kubeletconfig-machineconfiguration-openshift-io-v1#kubeletconfig-machineconfiguration-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update status of the specified KubeletConfig
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`KubeletConfig`](/docs/rest_api/machine_apis/kubeletconfig-machineconfiguration-openshift-io-v1#kubeletconfig-machineconfiguration-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace status of the specified KubeletConfig
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`KubeletConfig`](/docs/rest_api/machine_apis/kubeletconfig-machineconfiguration-openshift-io-v1#kubeletconfig-machineconfiguration-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`KubeletConfig`](/docs/rest_api/machine_apis/kubeletconfig-machineconfiguration-openshift-io-v1#kubeletconfig-machineconfiguration-openshift-io-v1) schema |
| 201 - Created | [`KubeletConfig`](/docs/rest_api/machine_apis/kubeletconfig-machineconfiguration-openshift-io-v1#kubeletconfig-machineconfiguration-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |
