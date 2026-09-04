---
title: "PodSecurityPolicySubjectReview [security.openshift.io/v1]"
sidebar_position: 6
---

# PodSecurityPolicySubjectReview [security.openshift.io/v1] {#podsecuritypolicysubjectreview-security-openshift-io-v1}

<a id="podsecuritypolicysubjectreview-security-openshift-io-v1"></a>

Description
: PodSecurityPolicySubjectReview checks whether a particular user/SA tuple can create the PodTemplateSpec.

  Compatibility level 2: Stable within a major release for a minimum of 9 months or 3 minor releases (whichever is longer).

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
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | metadata is the standard object’s metadata. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata |
| `spec` | `object` | PodSecurityPolicySubjectReviewSpec defines specification for PodSecurityPolicySubjectReview |
| `status` | `object` | PodSecurityPolicySubjectReviewStatus contains information/status for PodSecurityPolicySubjectReview. |

### .spec {#_spec}

Description
: PodSecurityPolicySubjectReviewSpec defines specification for PodSecurityPolicySubjectReview

Type
: ```
  `object`
  ```

Required
: - `template`

| Property | Type | Description |
| --- | --- | --- |
| `groups` | `array (string)` | groups is the groups you’re testing for. |
| `template` | [`PodTemplateSpec`](/docs/rest_api/objects#io-k8s-api-core-v1-PodTemplateSpec) | template is the PodTemplateSpec to check. If template.spec.serviceAccountName is empty it will not be defaulted. If its non-empty, it will be checked. |
| `user` | `string` | user is the user you’re testing for. If you specify "user" but not "group", then is it interpreted as "What if user were not a member of any groups. If user and groups are empty, then the check is performed using **only** the serviceAccountName in the template. |

### .status {#_status}

Description
: PodSecurityPolicySubjectReviewStatus contains information/status for PodSecurityPolicySubjectReview.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `allowedBy` | [`ObjectReference`](/docs/rest_api/objects#io-k8s-api-core-v1-ObjectReference) | allowedBy is a reference to the rule that allows the PodTemplateSpec. A rule can be a SecurityContextConstraint or a PodSecurityPolicy A `nil`, indicates that it was denied. |
| `reason` | `string` | A machine-readable description of why this operation is in the "Failure" status. If this value is empty there is no information available. |
| `template` | [`PodTemplateSpec`](/docs/rest_api/objects#io-k8s-api-core-v1-PodTemplateSpec) | template is the PodTemplateSpec after the defaulting is applied. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/security.openshift.io/v1/namespaces/{namespace}/podsecuritypolicysubjectreviews`
  - `POST`: create a PodSecurityPolicySubjectReview

### /apis/security.openshift.io/v1/namespaces/{namespace}/podsecuritypolicysubjectreviews {#_apissecurityopenshiftiov1namespaces_namespace_podsecuritypolicysubjectreviews}

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
  create a PodSecurityPolicySubjectReview
  ```

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`PodSecurityPolicySubjectReview`](/docs/rest_api/security_apis/podsecuritypolicysubjectreview-security-openshift-io-v1#podsecuritypolicysubjectreview-security-openshift-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`PodSecurityPolicySubjectReview`](/docs/rest_api/security_apis/podsecuritypolicysubjectreview-security-openshift-io-v1#podsecuritypolicysubjectreview-security-openshift-io-v1) schema |
| 201 - Created | [`PodSecurityPolicySubjectReview`](/docs/rest_api/security_apis/podsecuritypolicysubjectreview-security-openshift-io-v1#podsecuritypolicysubjectreview-security-openshift-io-v1) schema |
| 202 - Accepted | [`PodSecurityPolicySubjectReview`](/docs/rest_api/security_apis/podsecuritypolicysubjectreview-security-openshift-io-v1#podsecuritypolicysubjectreview-security-openshift-io-v1) schema |
| 401 - Unauthorized | Empty |
