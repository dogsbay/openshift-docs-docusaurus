---
title: "GRPCRoute [gateway.networking.k8s.io/v1]"
sidebar_position: 15
---

# GRPCRoute [gateway.networking.k8s.io/v1] {#grpcroute-gateway-networking-k8s-io-v1}

<a id="grpcroute-gateway-networking-k8s-io-v1"></a>

Description
: GRPCRoute provides a way to route gRPC requests. This includes the capability to match requests by hostname, gRPC service, gRPC method, or HTTP/2 header. Filters can be used to specify additional processing steps. Backends specify where matching requests will be routed.

  GRPCRoute falls under extended support within the Gateway API. Within the following specification, the word "MUST" indicates that an implementation supporting GRPCRoute must conform to the indicated requirement, but an implementation not supporting this route type need not follow the requirement unless explicitly indicated.

  Implementations supporting `GRPCRoute` with the `HTTPS` `ProtocolType` MUST accept HTTP/2 connections without an initial upgrade from HTTP/1.1, i.e. via ALPN. If the implementation does not support this, then it MUST set the "Accepted" condition to "False" for the affected listener with a reason of "UnsupportedProtocol".  Implementations MAY also accept HTTP/2 connections with an upgrade from HTTP/1.

  Implementations supporting `GRPCRoute` with the `HTTP` `ProtocolType` MUST support HTTP/2 over cleartext TCP (h2c, https://www.rfc-editor.org/rfc/rfc7540#section-3.1) without an initial upgrade from HTTP/1.1, i.e. with prior knowledge (https://www.rfc-editor.org/rfc/rfc7540#section-3.4). If the implementation does not support this, then it MUST set the "Accepted" condition to "False" for the affected listener with a reason of "UnsupportedProtocol". Implementations MAY also accept HTTP/2 connections with an upgrade from HTTP/1, i.e. without prior knowledge.

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
| `spec` | `object` | Spec defines the desired state of GRPCRoute. |
| `status` | `object` | Status defines the current state of GRPCRoute. |

### .spec {#_spec}

Description
: Spec defines the desired state of GRPCRoute.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `hostnames` | `array (string)` | Hostnames defines a set of hostnames to match against the GRPC Host header to select a GRPCRoute to process the request. This matches the RFC 1123 definition of a hostname with 2 notable exceptions:<br>1. IPs are not allowed. 2. A hostname may be prefixed with a wildcard label (`\*.`). The wildcard    label MUST appear by itself as the first label.<br>If a hostname is specified by both the Listener and GRPCRoute, there MUST be at least one intersecting hostname for the GRPCRoute to be attached to the Listener. For example:<br>* A Listener with `test.example.com` as the hostname matches GRPCRoutes   that have either not specified any hostnames, or have specified at   least one of `test.example.com` or `*.example.com`. * A Listener with `\*.example.com` as the hostname matches GRPCRoutes   that have either not specified any hostnames or have specified at least   one hostname that matches the Listener hostname. For example,   `test.example.com` and `\*.example.com` would both match. On the other   hand, `example.com` and `test.example.net` would not match.<br>Hostnames that are prefixed with a wildcard label (`\*.`) are interpreted as a suffix match. That means that a match for `*.example.com` would match both `test.example.com`, and `foo.test.example.com`, but not `example.com`.<br>If both the Listener and GRPCRoute have specified hostnames, any GRPCRoute hostnames that do not match the Listener hostname MUST be ignored. For example, if a Listener specified `*.example.com`, and the GRPCRoute specified `test.example.com` and `test.example.net`, `test.example.net` MUST NOT be considered for a match.<br>If both the Listener and GRPCRoute have specified hostnames, and none match with the criteria above, then the GRPCRoute MUST NOT be accepted by the implementation. The implementation MUST raise an 'Accepted' Condition with a status of `False` in the corresponding RouteParentStatus.<br>If a Route (A) of type HTTPRoute or GRPCRoute is attached to a Listener and that listener already has another Route (B) of the other type attached and the intersection of the hostnames of A and B is non-empty, then the implementation MUST accept exactly one of these two routes, determined by the following criteria, in order:<br>* The oldest Route based on creation timestamp. * The Route appearing first in alphabetical order by   "{namespace}/{name}".<br>The rejected Route MUST raise an 'Accepted' condition with a status of 'False' in the corresponding RouteParentStatus.<br>Support: Core |
| `parentRefs` | `array` | ParentRefs references the resources (usually Gateways) that a Route wants to be attached to. Note that the referenced parent resource needs to allow this for the attachment to be complete. For Gateways, that means the Gateway needs to allow attachment from Routes of this kind and namespace. For Services, that means the Service must either be in the same namespace for a "producer" route, or the mesh implementation must support and allow "consumer" routes for the referenced Service. ReferenceGrant is not applicable for governing ParentRefs to Services - it is not possible to create a "producer" route for a Service in a different namespace from the Route.<br>There are two kinds of parent resources with "Core" support:<br>* Gateway (Gateway conformance profile) * Service (Mesh conformance profile, ClusterIP Services only)<br>This API may be extended in the future to support additional kinds of parent resources.<br>ParentRefs must be *distinct*. This means either that:<br>* They select different objects.  If this is the case, then parentRef   entries are distinct. In terms of fields, this means that the   multi-part key defined by `group`, `kind`, `namespace`, and `name` must   be unique across all parentRef entries in the Route. * They do not select different objects, but for each optional field used,   each ParentRef that selects the same object must set the same set of   optional fields to different values. If one ParentRef sets a   combination of optional fields, all must set the same combination.<br>Some examples:<br>* If one ParentRef sets `sectionName`, all ParentRefs referencing the   same object must also set `sectionName`. * If one ParentRef sets `port`, all ParentRefs referencing the same   object must also set `port`. * If one ParentRef sets `sectionName` and `port`, all ParentRefs   referencing the same object must also set `sectionName` and `port`.<br>It is possible to separately reference multiple distinct objects that may be collapsed by an implementation. For example, some implementations may choose to merge compatible Gateway Listeners together. If that is the case, the list of routes attached to those resources should also be merged.<br>Note that for ParentRefs that cross namespace boundaries, there are specific rules. Cross-namespace references are only valid if they are explicitly allowed by something in the namespace they are referring to. For example, Gateway has the AllowedRoutes field, and ReferenceGrant provides a generic way to enable other kinds of cross-namespace reference. |
| `parentRefs[]` | `object` | ParentReference identifies an API object (usually a Gateway) that can be considered a parent of this resource (usually a route). There are two kinds of parent resources with "Core" support:<br>* Gateway (Gateway conformance profile) * Service (Mesh conformance profile, ClusterIP Services only)<br>This API may be extended in the future to support additional kinds of parent resources.<br>The API object must be valid in the cluster; the Group and Kind must be registered in the cluster for this reference to be valid. |
| `rules` | `array` | Rules are a list of GRPC matchers, filters and actions. |
| `rules[]` | `object` | GRPCRouteRule defines the semantics for matching a gRPC request based on conditions (matches), processing it (filters), and forwarding the request to an API object (backendRefs). |

### .spec.parentRefs {#_specparentrefs}

Description
: ParentRefs references the resources (usually Gateways) that a Route wants to be attached to. Note that the referenced parent resource needs to allow this for the attachment to be complete. For Gateways, that means the Gateway needs to allow attachment from Routes of this kind and namespace. For Services, that means the Service must either be in the same namespace for a "producer" route, or the mesh implementation must support and allow "consumer" routes for the referenced Service. ReferenceGrant is not applicable for governing ParentRefs to Services - it is not possible to create a "producer" route for a Service in a different namespace from the Route.

  There are two kinds of parent resources with "Core" support:

  - Gateway (Gateway conformance profile)
  - Service (Mesh conformance profile, ClusterIP Services only)

  This API may be extended in the future to support additional kinds of parent resources.

  ParentRefs must be *distinct*. This means either that:

  - They select different objects.  If this is the case, then parentRef entries are distinct. In terms of fields, this means that the multi-part key defined by `group`, `kind`, `namespace`, and `name` must be unique across all parentRef entries in the Route.
  - They do not select different objects, but for each optional field used, each ParentRef that selects the same object must set the same set of optional fields to different values. If one ParentRef sets a combination of optional fields, all must set the same combination.

  Some examples:

  - If one ParentRef sets `sectionName`, all ParentRefs referencing the same object must also set `sectionName`.
  - If one ParentRef sets `port`, all ParentRefs referencing the same object must also set `port`.
  - If one ParentRef sets `sectionName` and `port`, all ParentRefs referencing the same object must also set `sectionName` and `port`.

  It is possible to separately reference multiple distinct objects that may be collapsed by an implementation. For example, some implementations may choose to merge compatible Gateway Listeners together. If that is the case, the list of routes attached to those resources should also be merged.

  Note that for ParentRefs that cross namespace boundaries, there are specific rules. Cross-namespace references are only valid if they are explicitly allowed by something in the namespace they are referring to. For example, Gateway has the AllowedRoutes field, and ReferenceGrant provides a generic way to enable other kinds of cross-namespace reference.

Type
: ```
  `array`
  ```

### .spec.parentRefs[] {#_specparentrefs}

Description
: ParentReference identifies an API object (usually a Gateway) that can be considered a parent of this resource (usually a route). There are two kinds of parent resources with "Core" support:

  - Gateway (Gateway conformance profile)
  - Service (Mesh conformance profile, ClusterIP Services only)

  This API may be extended in the future to support additional kinds of parent resources.

  The API object must be valid in the cluster; the Group and Kind must be registered in the cluster for this reference to be valid.

Type
: ```
  `object`
  ```

Required
: - `name`

| Property | Type | Description |
| --- | --- | --- |
| `group` | `string` | Group is the group of the referent. When unspecified, "gateway.networking.k8s.io" is inferred. To set the core API group (such as for a "Service" kind referent), Group must be explicitly set to "" (empty string).<br>Support: Core |
| `kind` | `string` | Kind is kind of the referent.<br>There are two kinds of parent resources with "Core" support:<br>* Gateway (Gateway conformance profile) * Service (Mesh conformance profile, ClusterIP Services only)<br>Support for other resources is Implementation-Specific. |
| `name` | `string` | Name is the name of the referent.<br>Support: Core |
| `namespace` | `string` | Namespace is the namespace of the referent. When unspecified, this refers to the local namespace of the Route.<br>Note that there are specific rules for ParentRefs which cross namespace boundaries. Cross-namespace references are only valid if they are explicitly allowed by something in the namespace they are referring to. For example: Gateway has the AllowedRoutes field, and ReferenceGrant provides a generic way to enable any other kind of cross-namespace reference.<br>Support: Core |
| `port` | `integer` | Port is the network port this Route targets. It can be interpreted differently based on the type of parent resource.<br>When the parent resource is a Gateway, this targets all listeners listening on the specified port that also support this kind of Route(and select this Route). It’s not recommended to set `Port` unless the networking behaviors specified in a Route must apply to a specific port as opposed to a listener(s) whose port(s) may be changed. When both Port and SectionName are specified, the name and port of the selected listener must match both specified values.<br>Implementations MAY choose to support other parent resources. Implementations supporting other types of parent resources MUST clearly document how/if Port is interpreted.<br>For the purpose of status, an attachment is considered successful as long as the parent resource accepts it partially. For example, Gateway listeners can restrict which Routes can attach to them by Route kind, namespace, or hostname. If 1 of 2 Gateway listeners accept attachment from the referencing Route, the Route MUST be considered successfully attached. If no Gateway listeners accept attachment from this Route, the Route MUST be considered detached from the Gateway.<br>Support: Extended |
| `sectionName` | `string` | SectionName is the name of a section within the target resource. In the following resources, SectionName is interpreted as the following:<br>* Gateway: Listener name. When both Port (experimental) and SectionName are specified, the name and port of the selected listener must match both specified values. * Service: Port name. When both Port (experimental) and SectionName are specified, the name and port of the selected listener must match both specified values.<br>Implementations MAY choose to support attaching Routes to other resources. If that is the case, they MUST clearly document how SectionName is interpreted.<br>When unspecified (empty string), this will reference the entire resource. For the purpose of status, an attachment is considered successful if at least one section in the parent resource accepts it. For example, Gateway listeners can restrict which Routes can attach to them by Route kind, namespace, or hostname. If 1 of 2 Gateway listeners accept attachment from the referencing Route, the Route MUST be considered successfully attached. If no Gateway listeners accept attachment from this Route, the Route MUST be considered detached from the Gateway.<br>Support: Core |

### .spec.rules {#_specrules}

Description
: Rules are a list of GRPC matchers, filters and actions.

Type
: ```
  `array`
  ```

### .spec.rules[] {#_specrules}

Description
: GRPCRouteRule defines the semantics for matching a gRPC request based on conditions (matches), processing it (filters), and forwarding the request to an API object (backendRefs).

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `backendRefs` | `array` | BackendRefs defines the backend(s) where matching requests should be sent.<br>Failure behavior here depends on how many BackendRefs are specified and how many are invalid.<br>If **all** entries in BackendRefs are invalid, and there are also no filters specified in this route rule, **all** traffic which matches this rule MUST receive an `UNAVAILABLE` status.<br>See the GRPCBackendRef definition for the rules about what makes a single GRPCBackendRef invalid.<br>When a GRPCBackendRef is invalid, `UNAVAILABLE` statuses MUST be returned for requests that would have otherwise been routed to an invalid backend. If multiple backends are specified, and some are invalid, the proportion of requests that would otherwise have been routed to an invalid backend MUST receive an `UNAVAILABLE` status.<br>For example, if two backends are specified with equal weights, and one is invalid, 50 percent of traffic MUST receive an `UNAVAILABLE` status. Implementations may choose how that 50 percent is determined.<br>Support: Core for Kubernetes Service<br>Support: Implementation-specific for any other resource<br>Support for weight: Core |
| `backendRefs[]` | `object` | GRPCBackendRef defines how a GRPCRoute forwards a gRPC request.<br>Note that when a namespace different than the local namespace is specified, a ReferenceGrant object is required in the referent namespace to allow that namespace’s owner to accept the reference. See the ReferenceGrant documentation for details. |
| `filters` | `array` | Filters define the filters that are applied to requests that match this rule.<br>The effects of ordering of multiple behaviors are currently unspecified. This can change in the future based on feedback during the alpha stage.<br>Conformance-levels at this level are defined based on the type of filter:<br>- ALL core filters MUST be supported by all implementations that support   GRPCRoute. - Implementers are encouraged to support extended filters. - Implementation-specific custom filters have no API guarantees across   implementations.<br>Specifying the same filter multiple times is not supported unless explicitly indicated in the filter.<br>If an implementation cannot support a combination of filters, it must clearly document that limitation. In cases where incompatible or unsupported filters are specified and cause the `Accepted` condition to be set to status `False`, implementations may use the `IncompatibleFilters` reason to specify this configuration error.<br>Support: Core |
| `filters[]` | `object` | GRPCRouteFilter defines processing steps that must be completed during the request or response lifecycle. GRPCRouteFilters are meant as an extension point to express processing that may be done in Gateway implementations. Some examples include request or response modification, implementing authentication strategies, rate-limiting, and traffic shaping. API guarantee/conformance is defined based on the type of the filter. |
| `matches` | `array` | Matches define conditions used for matching the rule against incoming gRPC requests. Each match is independent, i.e. this rule will be matched if **any** one of the matches is satisfied.<br>For example, take the following matches configuration:<br>matches: - method:     service: foo.bar   headers:     values:       version: 2 - method:     service: foo.bar.v2<br>For a request to match against this rule, it MUST satisfy EITHER of the two conditions:<br>- service of foo.bar AND contains the header `version: 2` - service of foo.bar.v2<br>See the documentation for GRPCRouteMatch on how to specify multiple match conditions to be ANDed together.<br>If no matches are specified, the implementation MUST match every gRPC request.<br>Proxy or Load Balancer routing configuration generated from GRPCRoutes MUST prioritize rules based on the following criteria, continuing on ties. Merging MUST not be done between GRPCRoutes and HTTPRoutes. Precedence MUST be given to the rule with the largest number of:<br>* Characters in a matching non-wildcard hostname. * Characters in a matching hostname. * Characters in a matching service. * Characters in a matching method. * Header matches.<br>If ties still exist across multiple Routes, matching precedence MUST be determined in order of the following criteria, continuing on ties:<br>* The oldest Route based on creation timestamp. * The Route appearing first in alphabetical order by   "{namespace}/{name}".<br>If ties still exist within the Route that has been given precedence, matching precedence MUST be granted to the first matching rule meeting the above criteria. |
| `matches[]` | `object` | GRPCRouteMatch defines the predicate used to match requests to a given action. Multiple match types are ANDed together, i.e. the match will evaluate to true only if all conditions are satisfied.<br>For example, the match below will match a gRPC request only if its service is `foo` AND it contains the `version: v1` header:<br>matches:   - method:     type: Exact     service: "foo"     headers:   - name: "version"     value "v1" |
| `name` | `string` | Name is the name of the route rule. This name MUST be unique within a Route if it is set.<br>Support: Extended |

### .spec.rules[].backendRefs {#_specrulesbackendrefs}

Description
: BackendRefs defines the backend(s) where matching requests should be sent.

  Failure behavior here depends on how many BackendRefs are specified and how many are invalid.

  If **all** entries in BackendRefs are invalid, and there are also no filters specified in this route rule, **all** traffic which matches this rule MUST receive an `UNAVAILABLE` status.

  See the GRPCBackendRef definition for the rules about what makes a single GRPCBackendRef invalid.

  When a GRPCBackendRef is invalid, `UNAVAILABLE` statuses MUST be returned for requests that would have otherwise been routed to an invalid backend. If multiple backends are specified, and some are invalid, the proportion of requests that would otherwise have been routed to an invalid backend MUST receive an `UNAVAILABLE` status.

  For example, if two backends are specified with equal weights, and one is invalid, 50 percent of traffic MUST receive an `UNAVAILABLE` status. Implementations may choose how that 50 percent is determined.

  Support: Core for Kubernetes Service

  Support: Implementation-specific for any other resource

  Support for weight: Core

Type
: ```
  `array`
  ```

### .spec.rules[].backendRefs[] {#_specrulesbackendrefs}

Description
: GRPCBackendRef defines how a GRPCRoute forwards a gRPC request.

  Note that when a namespace different than the local namespace is specified, a ReferenceGrant object is required in the referent namespace to allow that namespace’s owner to accept the reference. See the ReferenceGrant documentation for details.

Type
: ```
  `object`
  ```

Required
: - `name`

| Property | Type | Description |
| --- | --- | --- |
| `filters` | `array` | Filters defined at this level MUST be executed if and only if the request is being forwarded to the backend defined here.<br>Support: Implementation-specific (For broader support of filters, use the Filters field in GRPCRouteRule.) |
| `filters[]` | `object` | GRPCRouteFilter defines processing steps that must be completed during the request or response lifecycle. GRPCRouteFilters are meant as an extension point to express processing that may be done in Gateway implementations. Some examples include request or response modification, implementing authentication strategies, rate-limiting, and traffic shaping. API guarantee/conformance is defined based on the type of the filter. |
| `group` | `string` | Group is the group of the referent. For example, "gateway.networking.k8s.io". When unspecified or empty string, core API group is inferred. |
| `kind` | `string` | Kind is the Kubernetes resource kind of the referent. For example "Service".<br>Defaults to "Service" when not specified.<br>ExternalName services can refer to CNAME DNS records that may live outside of the cluster and as such are difficult to reason about in terms of conformance. They also may not be safe to forward to (see CVE-2021-25740 for more information). Implementations SHOULD NOT support ExternalName Services.<br>Support: Core (Services with a type other than ExternalName)<br>Support: Implementation-specific (Services with type ExternalName) |
| `name` | `string` | Name is the name of the referent. |
| `namespace` | `string` | Namespace is the namespace of the backend. When unspecified, the local namespace is inferred.<br>Note that when a namespace different than the local namespace is specified, a ReferenceGrant object is required in the referent namespace to allow that namespace’s owner to accept the reference. See the ReferenceGrant documentation for details.<br>Support: Core |
| `port` | `integer` | Port specifies the destination port number to use for this resource. Port is required when the referent is a Kubernetes Service. In this case, the port number is the service port number, not the target port. For other resources, destination port might be derived from the referent resource or this field. |
| `weight` | `integer` | Weight specifies the proportion of requests forwarded to the referenced backend. This is computed as weight/(sum of all weights in this BackendRefs list). For non-zero values, there may be some epsilon from the exact proportion defined here depending on the precision an implementation supports. Weight is not a percentage and the sum of weights does not need to equal 100.<br>If only one backend is specified and it has a weight greater than 0, 100% of the traffic is forwarded to that backend. If weight is set to 0, no traffic should be forwarded for this entry. If unspecified, weight defaults to 1.<br>Support for this field varies based on the context where used. |

### .spec.rules[].backendRefs[].filters {#_specrulesbackendrefsfilters}

Description
: Filters defined at this level MUST be executed if and only if the request is being forwarded to the backend defined here.

  Support: Implementation-specific (For broader support of filters, use the Filters field in GRPCRouteRule.)

Type
: ```
  `array`
  ```

### .spec.rules[].backendRefs[].filters[] {#_specrulesbackendrefsfilters}

Description
: GRPCRouteFilter defines processing steps that must be completed during the request or response lifecycle. GRPCRouteFilters are meant as an extension point to express processing that may be done in Gateway implementations. Some examples include request or response modification, implementing authentication strategies, rate-limiting, and traffic shaping. API guarantee/conformance is defined based on the type of the filter.

Type
: ```
  `object`
  ```

Required
: - `type`

| Property | Type | Description |
| --- | --- | --- |
| `extensionRef` | `object` | ExtensionRef is an optional, implementation-specific extension to the "filter" behavior.  For example, resource "myroutefilter" in group "networking.example.net"). ExtensionRef MUST NOT be used for core and extended filters.<br>Support: Implementation-specific<br>This filter can be used multiple times within the same rule. |
| `requestHeaderModifier` | `object` | RequestHeaderModifier defines a schema for a filter that modifies request headers.<br>Support: Core |
| `requestMirror` | `object` | RequestMirror defines a schema for a filter that mirrors requests. Requests are sent to the specified destination, but responses from that destination are ignored.<br>This filter can be used multiple times within the same rule. Note that not all implementations will be able to support mirroring to multiple backends.<br>Support: Extended |
| `responseHeaderModifier` | `object` | ResponseHeaderModifier defines a schema for a filter that modifies response headers.<br>Support: Extended |
| `type` | `string` | Type identifies the type of filter to apply. As with other API fields, types are classified into three conformance levels:<br>- Core: Filter types and their corresponding configuration defined by   "Support: Core" in this package, e.g. "RequestHeaderModifier". All   implementations supporting GRPCRoute MUST support core filters.<br>- Extended: Filter types and their corresponding configuration defined by   "Support: Extended" in this package, e.g. "RequestMirror". Implementers   are encouraged to support extended filters.<br>- Implementation-specific: Filters that are defined and supported by specific vendors.   In the future, filters showing convergence in behavior across multiple   implementations will be considered for inclusion in extended or core   conformance levels. Filter-specific configuration for such filters   is specified using the ExtensionRef field. `Type` MUST be set to   "ExtensionRef" for custom filters.<br>Implementers are encouraged to define custom implementation types to extend the core API with implementation-specific behavior.<br>If a reference to a custom filter type cannot be resolved, the filter MUST NOT be skipped. Instead, requests that would have been processed by that filter MUST receive a HTTP error response. |

### .spec.rules[].backendRefs[].filters[].extensionRef {#_specrulesbackendrefsfiltersextensionref}

Description
: ExtensionRef is an optional, implementation-specific extension to the "filter" behavior.  For example, resource "myroutefilter" in group "networking.example.net"). ExtensionRef MUST NOT be used for core and extended filters.

  Support: Implementation-specific

  This filter can be used multiple times within the same rule.

Type
: ```
  `object`
  ```

Required
: - `group`
  - `kind`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `group` | `string` | Group is the group of the referent. For example, "gateway.networking.k8s.io". When unspecified or empty string, core API group is inferred. |
| `kind` | `string` | Kind is kind of the referent. For example "HTTPRoute" or "Service". |
| `name` | `string` | Name is the name of the referent. |

### .spec.rules[].backendRefs[].filters[].requestHeaderModifier {#_specrulesbackendrefsfiltersrequestheadermodifier}

Description
: RequestHeaderModifier defines a schema for a filter that modifies request headers.

  Support: Core

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `add` | `array` | Add adds the given header(s) (name, value) to the request before the action. It appends to any existing values associated with the header name.<br>Input:   GET /foo HTTP/1.1   my-header: foo<br>Config:   add:   - name: "my-header"     value: "bar,baz"<br>Output:   GET /foo HTTP/1.1   my-header: foo,bar,baz |
| `add[]` | `object` | HTTPHeader represents an HTTP Header name and value as defined by RFC 7230. |
| `remove` | `array (string)` | Remove the given header(s) from the HTTP request before the action. The value of Remove is a list of HTTP header names. Note that the header names are case-insensitive (see https://datatracker.ietf.org/doc/html/rfc2616#section-4.2).<br>Input:   GET /foo HTTP/1.1   my-header1: foo   my-header2: bar   my-header3: baz<br>Config:   remove: ["my-header1", "my-header3"]<br>Output:   GET /foo HTTP/1.1   my-header2: bar |
| `set` | `array` | Set overwrites the request with the given header (name, value) before the action.<br>Input:   GET /foo HTTP/1.1   my-header: foo<br>Config:   set:   - name: "my-header"     value: "bar"<br>Output:   GET /foo HTTP/1.1   my-header: bar |
| `set[]` | `object` | HTTPHeader represents an HTTP Header name and value as defined by RFC 7230. |

### .spec.rules[].backendRefs[].filters[].requestHeaderModifier.add {#_specrulesbackendrefsfiltersrequestheadermodifieradd}

Description
: Add adds the given header(s) (name, value) to the request before the action. It appends to any existing values associated with the header name.

  Input: GET /foo HTTP/1.1 my-header: foo

  Config: add:

  - name: "my-header" value: "bar,baz"

  Output: GET /foo HTTP/1.1 my-header: foo,bar,baz

Type
: ```
  `array`
  ```

### .spec.rules[].backendRefs[].filters[].requestHeaderModifier.add[] {#_specrulesbackendrefsfiltersrequestheadermodifieradd}

Description
: HTTPHeader represents an HTTP Header name and value as defined by RFC 7230.

Type
: ```
  `object`
  ```

Required
: - `name`
  - `value`

| Property | Type | Description |
| --- | --- | --- |
| `name` | `string` | Name is the name of the HTTP Header to be matched. Name matching MUST be case-insensitive. (See https://tools.ietf.org/html/rfc7230#section-3.2).<br>If multiple entries specify equivalent header names, the first entry with an equivalent name MUST be considered for a match. Subsequent entries with an equivalent header name MUST be ignored. Due to the case-insensitivity of header names, "foo" and "Foo" are considered equivalent. |
| `value` | `string` | Value is the value of HTTP Header to be matched. |

### .spec.rules[].backendRefs[].filters[].requestHeaderModifier.set {#_specrulesbackendrefsfiltersrequestheadermodifierset}

Description
: Set overwrites the request with the given header (name, value) before the action.

  Input: GET /foo HTTP/1.1 my-header: foo

  Config: set:

  - name: "my-header" value: "bar"

  Output: GET /foo HTTP/1.1 my-header: bar

Type
: ```
  `array`
  ```

### .spec.rules[].backendRefs[].filters[].requestHeaderModifier.set[] {#_specrulesbackendrefsfiltersrequestheadermodifierset}

Description
: HTTPHeader represents an HTTP Header name and value as defined by RFC 7230.

Type
: ```
  `object`
  ```

Required
: - `name`
  - `value`

| Property | Type | Description |
| --- | --- | --- |
| `name` | `string` | Name is the name of the HTTP Header to be matched. Name matching MUST be case-insensitive. (See https://tools.ietf.org/html/rfc7230#section-3.2).<br>If multiple entries specify equivalent header names, the first entry with an equivalent name MUST be considered for a match. Subsequent entries with an equivalent header name MUST be ignored. Due to the case-insensitivity of header names, "foo" and "Foo" are considered equivalent. |
| `value` | `string` | Value is the value of HTTP Header to be matched. |

### .spec.rules[].backendRefs[].filters[].requestMirror {#_specrulesbackendrefsfiltersrequestmirror}

Description
: RequestMirror defines a schema for a filter that mirrors requests. Requests are sent to the specified destination, but responses from that destination are ignored.

  This filter can be used multiple times within the same rule. Note that not all implementations will be able to support mirroring to multiple backends.

  Support: Extended

Type
: ```
  `object`
  ```

Required
: - `backendRef`

| Property | Type | Description |
| --- | --- | --- |
| `backendRef` | `object` | BackendRef references a resource where mirrored requests are sent.<br>Mirrored requests must be sent only to a single destination endpoint within this BackendRef, irrespective of how many endpoints are present within this BackendRef.<br>If the referent cannot be found, this BackendRef is invalid and must be dropped from the Gateway. The controller must ensure the "ResolvedRefs" condition on the Route status is set to `status: False` and not configure this backend in the underlying implementation.<br>If there is a cross-namespace reference to an **existing** object that is not allowed by a ReferenceGrant, the controller must ensure the "ResolvedRefs"  condition on the Route is set to `status: False`, with the "RefNotPermitted" reason and not configure this backend in the underlying implementation.<br>In either error case, the Message of the `ResolvedRefs` Condition should be used to provide more detail about the problem.<br>Support: Extended for Kubernetes Service<br>Support: Implementation-specific for any other resource |
| `fraction` | `object` | Fraction represents the fraction of requests that should be mirrored to BackendRef.<br>Only one of Fraction or Percent may be specified. If neither field is specified, 100% of requests will be mirrored. |
| `percent` | `integer` | Percent represents the percentage of requests that should be mirrored to BackendRef. Its minimum value is 0 (indicating 0% of requests) and its maximum value is 100 (indicating 100% of requests).<br>Only one of Fraction or Percent may be specified. If neither field is specified, 100% of requests will be mirrored. |

### .spec.rules[].backendRefs[].filters[].requestMirror.backendRef {#_specrulesbackendrefsfiltersrequestmirrorbackendref}

Description
: BackendRef references a resource where mirrored requests are sent.

  Mirrored requests must be sent only to a single destination endpoint within this BackendRef, irrespective of how many endpoints are present within this BackendRef.

  If the referent cannot be found, this BackendRef is invalid and must be dropped from the Gateway. The controller must ensure the "ResolvedRefs" condition on the Route status is set to `status: False` and not configure this backend in the underlying implementation.

  If there is a cross-namespace reference to an **existing** object that is not allowed by a ReferenceGrant, the controller must ensure the "ResolvedRefs"  condition on the Route is set to `status: False`, with the "RefNotPermitted" reason and not configure this backend in the underlying implementation.

  In either error case, the Message of the `ResolvedRefs` Condition should be used to provide more detail about the problem.

  Support: Extended for Kubernetes Service

  Support: Implementation-specific for any other resource

Type
: ```
  `object`
  ```

Required
: - `name`

| Property | Type | Description |
| --- | --- | --- |
| `group` | `string` | Group is the group of the referent. For example, "gateway.networking.k8s.io". When unspecified or empty string, core API group is inferred. |
| `kind` | `string` | Kind is the Kubernetes resource kind of the referent. For example "Service".<br>Defaults to "Service" when not specified.<br>ExternalName services can refer to CNAME DNS records that may live outside of the cluster and as such are difficult to reason about in terms of conformance. They also may not be safe to forward to (see CVE-2021-25740 for more information). Implementations SHOULD NOT support ExternalName Services.<br>Support: Core (Services with a type other than ExternalName)<br>Support: Implementation-specific (Services with type ExternalName) |
| `name` | `string` | Name is the name of the referent. |
| `namespace` | `string` | Namespace is the namespace of the backend. When unspecified, the local namespace is inferred.<br>Note that when a namespace different than the local namespace is specified, a ReferenceGrant object is required in the referent namespace to allow that namespace’s owner to accept the reference. See the ReferenceGrant documentation for details.<br>Support: Core |
| `port` | `integer` | Port specifies the destination port number to use for this resource. Port is required when the referent is a Kubernetes Service. In this case, the port number is the service port number, not the target port. For other resources, destination port might be derived from the referent resource or this field. |

### .spec.rules[].backendRefs[].filters[].requestMirror.fraction {#_specrulesbackendrefsfiltersrequestmirrorfraction}

Description
: Fraction represents the fraction of requests that should be mirrored to BackendRef.

  Only one of Fraction or Percent may be specified. If neither field is specified, 100% of requests will be mirrored.

Type
: ```
  `object`
  ```

Required
: - `numerator`

| Property | Type | Description |
| --- | --- | --- |
| `denominator` | `integer` |  |
| `numerator` | `integer` |  |

### .spec.rules[].backendRefs[].filters[].responseHeaderModifier {#_specrulesbackendrefsfiltersresponseheadermodifier}

Description
: ResponseHeaderModifier defines a schema for a filter that modifies response headers.

  Support: Extended

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `add` | `array` | Add adds the given header(s) (name, value) to the request before the action. It appends to any existing values associated with the header name.<br>Input:   GET /foo HTTP/1.1   my-header: foo<br>Config:   add:   - name: "my-header"     value: "bar,baz"<br>Output:   GET /foo HTTP/1.1   my-header: foo,bar,baz |
| `add[]` | `object` | HTTPHeader represents an HTTP Header name and value as defined by RFC 7230. |
| `remove` | `array (string)` | Remove the given header(s) from the HTTP request before the action. The value of Remove is a list of HTTP header names. Note that the header names are case-insensitive (see https://datatracker.ietf.org/doc/html/rfc2616#section-4.2).<br>Input:   GET /foo HTTP/1.1   my-header1: foo   my-header2: bar   my-header3: baz<br>Config:   remove: ["my-header1", "my-header3"]<br>Output:   GET /foo HTTP/1.1   my-header2: bar |
| `set` | `array` | Set overwrites the request with the given header (name, value) before the action.<br>Input:   GET /foo HTTP/1.1   my-header: foo<br>Config:   set:   - name: "my-header"     value: "bar"<br>Output:   GET /foo HTTP/1.1   my-header: bar |
| `set[]` | `object` | HTTPHeader represents an HTTP Header name and value as defined by RFC 7230. |

### .spec.rules[].backendRefs[].filters[].responseHeaderModifier.add {#_specrulesbackendrefsfiltersresponseheadermodifieradd}

Description
: Add adds the given header(s) (name, value) to the request before the action. It appends to any existing values associated with the header name.

  Input: GET /foo HTTP/1.1 my-header: foo

  Config: add:

  - name: "my-header" value: "bar,baz"

  Output: GET /foo HTTP/1.1 my-header: foo,bar,baz

Type
: ```
  `array`
  ```

### .spec.rules[].backendRefs[].filters[].responseHeaderModifier.add[] {#_specrulesbackendrefsfiltersresponseheadermodifieradd}

Description
: HTTPHeader represents an HTTP Header name and value as defined by RFC 7230.

Type
: ```
  `object`
  ```

Required
: - `name`
  - `value`

| Property | Type | Description |
| --- | --- | --- |
| `name` | `string` | Name is the name of the HTTP Header to be matched. Name matching MUST be case-insensitive. (See https://tools.ietf.org/html/rfc7230#section-3.2).<br>If multiple entries specify equivalent header names, the first entry with an equivalent name MUST be considered for a match. Subsequent entries with an equivalent header name MUST be ignored. Due to the case-insensitivity of header names, "foo" and "Foo" are considered equivalent. |
| `value` | `string` | Value is the value of HTTP Header to be matched. |

### .spec.rules[].backendRefs[].filters[].responseHeaderModifier.set {#_specrulesbackendrefsfiltersresponseheadermodifierset}

Description
: Set overwrites the request with the given header (name, value) before the action.

  Input: GET /foo HTTP/1.1 my-header: foo

  Config: set:

  - name: "my-header" value: "bar"

  Output: GET /foo HTTP/1.1 my-header: bar

Type
: ```
  `array`
  ```

### .spec.rules[].backendRefs[].filters[].responseHeaderModifier.set[] {#_specrulesbackendrefsfiltersresponseheadermodifierset}

Description
: HTTPHeader represents an HTTP Header name and value as defined by RFC 7230.

Type
: ```
  `object`
  ```

Required
: - `name`
  - `value`

| Property | Type | Description |
| --- | --- | --- |
| `name` | `string` | Name is the name of the HTTP Header to be matched. Name matching MUST be case-insensitive. (See https://tools.ietf.org/html/rfc7230#section-3.2).<br>If multiple entries specify equivalent header names, the first entry with an equivalent name MUST be considered for a match. Subsequent entries with an equivalent header name MUST be ignored. Due to the case-insensitivity of header names, "foo" and "Foo" are considered equivalent. |
| `value` | `string` | Value is the value of HTTP Header to be matched. |

### .spec.rules[].filters {#_specrulesfilters}

Description
: Filters define the filters that are applied to requests that match this rule.

  The effects of ordering of multiple behaviors are currently unspecified. This can change in the future based on feedback during the alpha stage.

  Conformance-levels at this level are defined based on the type of filter:

  - ALL core filters MUST be supported by all implementations that support GRPCRoute.
  - Implementers are encouraged to support extended filters.
  - Implementation-specific custom filters have no API guarantees across implementations.

  Specifying the same filter multiple times is not supported unless explicitly indicated in the filter.

  If an implementation cannot support a combination of filters, it must clearly document that limitation. In cases where incompatible or unsupported filters are specified and cause the `Accepted` condition to be set to status `False`, implementations may use the `IncompatibleFilters` reason to specify this configuration error.

  Support: Core

Type
: ```
  `array`
  ```

### .spec.rules[].filters[] {#_specrulesfilters}

Description
: GRPCRouteFilter defines processing steps that must be completed during the request or response lifecycle. GRPCRouteFilters are meant as an extension point to express processing that may be done in Gateway implementations. Some examples include request or response modification, implementing authentication strategies, rate-limiting, and traffic shaping. API guarantee/conformance is defined based on the type of the filter.

Type
: ```
  `object`
  ```

Required
: - `type`

| Property | Type | Description |
| --- | --- | --- |
| `extensionRef` | `object` | ExtensionRef is an optional, implementation-specific extension to the "filter" behavior.  For example, resource "myroutefilter" in group "networking.example.net"). ExtensionRef MUST NOT be used for core and extended filters.<br>Support: Implementation-specific<br>This filter can be used multiple times within the same rule. |
| `requestHeaderModifier` | `object` | RequestHeaderModifier defines a schema for a filter that modifies request headers.<br>Support: Core |
| `requestMirror` | `object` | RequestMirror defines a schema for a filter that mirrors requests. Requests are sent to the specified destination, but responses from that destination are ignored.<br>This filter can be used multiple times within the same rule. Note that not all implementations will be able to support mirroring to multiple backends.<br>Support: Extended |
| `responseHeaderModifier` | `object` | ResponseHeaderModifier defines a schema for a filter that modifies response headers.<br>Support: Extended |
| `type` | `string` | Type identifies the type of filter to apply. As with other API fields, types are classified into three conformance levels:<br>- Core: Filter types and their corresponding configuration defined by   "Support: Core" in this package, e.g. "RequestHeaderModifier". All   implementations supporting GRPCRoute MUST support core filters.<br>- Extended: Filter types and their corresponding configuration defined by   "Support: Extended" in this package, e.g. "RequestMirror". Implementers   are encouraged to support extended filters.<br>- Implementation-specific: Filters that are defined and supported by specific vendors.   In the future, filters showing convergence in behavior across multiple   implementations will be considered for inclusion in extended or core   conformance levels. Filter-specific configuration for such filters   is specified using the ExtensionRef field. `Type` MUST be set to   "ExtensionRef" for custom filters.<br>Implementers are encouraged to define custom implementation types to extend the core API with implementation-specific behavior.<br>If a reference to a custom filter type cannot be resolved, the filter MUST NOT be skipped. Instead, requests that would have been processed by that filter MUST receive a HTTP error response. |

### .spec.rules[].filters[].extensionRef {#_specrulesfiltersextensionref}

Description
: ExtensionRef is an optional, implementation-specific extension to the "filter" behavior.  For example, resource "myroutefilter" in group "networking.example.net"). ExtensionRef MUST NOT be used for core and extended filters.

  Support: Implementation-specific

  This filter can be used multiple times within the same rule.

Type
: ```
  `object`
  ```

Required
: - `group`
  - `kind`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `group` | `string` | Group is the group of the referent. For example, "gateway.networking.k8s.io". When unspecified or empty string, core API group is inferred. |
| `kind` | `string` | Kind is kind of the referent. For example "HTTPRoute" or "Service". |
| `name` | `string` | Name is the name of the referent. |

### .spec.rules[].filters[].requestHeaderModifier {#_specrulesfiltersrequestheadermodifier}

Description
: RequestHeaderModifier defines a schema for a filter that modifies request headers.

  Support: Core

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `add` | `array` | Add adds the given header(s) (name, value) to the request before the action. It appends to any existing values associated with the header name.<br>Input:   GET /foo HTTP/1.1   my-header: foo<br>Config:   add:   - name: "my-header"     value: "bar,baz"<br>Output:   GET /foo HTTP/1.1   my-header: foo,bar,baz |
| `add[]` | `object` | HTTPHeader represents an HTTP Header name and value as defined by RFC 7230. |
| `remove` | `array (string)` | Remove the given header(s) from the HTTP request before the action. The value of Remove is a list of HTTP header names. Note that the header names are case-insensitive (see https://datatracker.ietf.org/doc/html/rfc2616#section-4.2).<br>Input:   GET /foo HTTP/1.1   my-header1: foo   my-header2: bar   my-header3: baz<br>Config:   remove: ["my-header1", "my-header3"]<br>Output:   GET /foo HTTP/1.1   my-header2: bar |
| `set` | `array` | Set overwrites the request with the given header (name, value) before the action.<br>Input:   GET /foo HTTP/1.1   my-header: foo<br>Config:   set:   - name: "my-header"     value: "bar"<br>Output:   GET /foo HTTP/1.1   my-header: bar |
| `set[]` | `object` | HTTPHeader represents an HTTP Header name and value as defined by RFC 7230. |

### .spec.rules[].filters[].requestHeaderModifier.add {#_specrulesfiltersrequestheadermodifieradd}

Description
: Add adds the given header(s) (name, value) to the request before the action. It appends to any existing values associated with the header name.

  Input: GET /foo HTTP/1.1 my-header: foo

  Config: add:

  - name: "my-header" value: "bar,baz"

  Output: GET /foo HTTP/1.1 my-header: foo,bar,baz

Type
: ```
  `array`
  ```

### .spec.rules[].filters[].requestHeaderModifier.add[] {#_specrulesfiltersrequestheadermodifieradd}

Description
: HTTPHeader represents an HTTP Header name and value as defined by RFC 7230.

Type
: ```
  `object`
  ```

Required
: - `name`
  - `value`

| Property | Type | Description |
| --- | --- | --- |
| `name` | `string` | Name is the name of the HTTP Header to be matched. Name matching MUST be case-insensitive. (See https://tools.ietf.org/html/rfc7230#section-3.2).<br>If multiple entries specify equivalent header names, the first entry with an equivalent name MUST be considered for a match. Subsequent entries with an equivalent header name MUST be ignored. Due to the case-insensitivity of header names, "foo" and "Foo" are considered equivalent. |
| `value` | `string` | Value is the value of HTTP Header to be matched. |

### .spec.rules[].filters[].requestHeaderModifier.set {#_specrulesfiltersrequestheadermodifierset}

Description
: Set overwrites the request with the given header (name, value) before the action.

  Input: GET /foo HTTP/1.1 my-header: foo

  Config: set:

  - name: "my-header" value: "bar"

  Output: GET /foo HTTP/1.1 my-header: bar

Type
: ```
  `array`
  ```

### .spec.rules[].filters[].requestHeaderModifier.set[] {#_specrulesfiltersrequestheadermodifierset}

Description
: HTTPHeader represents an HTTP Header name and value as defined by RFC 7230.

Type
: ```
  `object`
  ```

Required
: - `name`
  - `value`

| Property | Type | Description |
| --- | --- | --- |
| `name` | `string` | Name is the name of the HTTP Header to be matched. Name matching MUST be case-insensitive. (See https://tools.ietf.org/html/rfc7230#section-3.2).<br>If multiple entries specify equivalent header names, the first entry with an equivalent name MUST be considered for a match. Subsequent entries with an equivalent header name MUST be ignored. Due to the case-insensitivity of header names, "foo" and "Foo" are considered equivalent. |
| `value` | `string` | Value is the value of HTTP Header to be matched. |

### .spec.rules[].filters[].requestMirror {#_specrulesfiltersrequestmirror}

Description
: RequestMirror defines a schema for a filter that mirrors requests. Requests are sent to the specified destination, but responses from that destination are ignored.

  This filter can be used multiple times within the same rule. Note that not all implementations will be able to support mirroring to multiple backends.

  Support: Extended

Type
: ```
  `object`
  ```

Required
: - `backendRef`

| Property | Type | Description |
| --- | --- | --- |
| `backendRef` | `object` | BackendRef references a resource where mirrored requests are sent.<br>Mirrored requests must be sent only to a single destination endpoint within this BackendRef, irrespective of how many endpoints are present within this BackendRef.<br>If the referent cannot be found, this BackendRef is invalid and must be dropped from the Gateway. The controller must ensure the "ResolvedRefs" condition on the Route status is set to `status: False` and not configure this backend in the underlying implementation.<br>If there is a cross-namespace reference to an **existing** object that is not allowed by a ReferenceGrant, the controller must ensure the "ResolvedRefs"  condition on the Route is set to `status: False`, with the "RefNotPermitted" reason and not configure this backend in the underlying implementation.<br>In either error case, the Message of the `ResolvedRefs` Condition should be used to provide more detail about the problem.<br>Support: Extended for Kubernetes Service<br>Support: Implementation-specific for any other resource |
| `fraction` | `object` | Fraction represents the fraction of requests that should be mirrored to BackendRef.<br>Only one of Fraction or Percent may be specified. If neither field is specified, 100% of requests will be mirrored. |
| `percent` | `integer` | Percent represents the percentage of requests that should be mirrored to BackendRef. Its minimum value is 0 (indicating 0% of requests) and its maximum value is 100 (indicating 100% of requests).<br>Only one of Fraction or Percent may be specified. If neither field is specified, 100% of requests will be mirrored. |

### .spec.rules[].filters[].requestMirror.backendRef {#_specrulesfiltersrequestmirrorbackendref}

Description
: BackendRef references a resource where mirrored requests are sent.

  Mirrored requests must be sent only to a single destination endpoint within this BackendRef, irrespective of how many endpoints are present within this BackendRef.

  If the referent cannot be found, this BackendRef is invalid and must be dropped from the Gateway. The controller must ensure the "ResolvedRefs" condition on the Route status is set to `status: False` and not configure this backend in the underlying implementation.

  If there is a cross-namespace reference to an **existing** object that is not allowed by a ReferenceGrant, the controller must ensure the "ResolvedRefs"  condition on the Route is set to `status: False`, with the "RefNotPermitted" reason and not configure this backend in the underlying implementation.

  In either error case, the Message of the `ResolvedRefs` Condition should be used to provide more detail about the problem.

  Support: Extended for Kubernetes Service

  Support: Implementation-specific for any other resource

Type
: ```
  `object`
  ```

Required
: - `name`

| Property | Type | Description |
| --- | --- | --- |
| `group` | `string` | Group is the group of the referent. For example, "gateway.networking.k8s.io". When unspecified or empty string, core API group is inferred. |
| `kind` | `string` | Kind is the Kubernetes resource kind of the referent. For example "Service".<br>Defaults to "Service" when not specified.<br>ExternalName services can refer to CNAME DNS records that may live outside of the cluster and as such are difficult to reason about in terms of conformance. They also may not be safe to forward to (see CVE-2021-25740 for more information). Implementations SHOULD NOT support ExternalName Services.<br>Support: Core (Services with a type other than ExternalName)<br>Support: Implementation-specific (Services with type ExternalName) |
| `name` | `string` | Name is the name of the referent. |
| `namespace` | `string` | Namespace is the namespace of the backend. When unspecified, the local namespace is inferred.<br>Note that when a namespace different than the local namespace is specified, a ReferenceGrant object is required in the referent namespace to allow that namespace’s owner to accept the reference. See the ReferenceGrant documentation for details.<br>Support: Core |
| `port` | `integer` | Port specifies the destination port number to use for this resource. Port is required when the referent is a Kubernetes Service. In this case, the port number is the service port number, not the target port. For other resources, destination port might be derived from the referent resource or this field. |

### .spec.rules[].filters[].requestMirror.fraction {#_specrulesfiltersrequestmirrorfraction}

Description
: Fraction represents the fraction of requests that should be mirrored to BackendRef.

  Only one of Fraction or Percent may be specified. If neither field is specified, 100% of requests will be mirrored.

Type
: ```
  `object`
  ```

Required
: - `numerator`

| Property | Type | Description |
| --- | --- | --- |
| `denominator` | `integer` |  |
| `numerator` | `integer` |  |

### .spec.rules[].filters[].responseHeaderModifier {#_specrulesfiltersresponseheadermodifier}

Description
: ResponseHeaderModifier defines a schema for a filter that modifies response headers.

  Support: Extended

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `add` | `array` | Add adds the given header(s) (name, value) to the request before the action. It appends to any existing values associated with the header name.<br>Input:   GET /foo HTTP/1.1   my-header: foo<br>Config:   add:   - name: "my-header"     value: "bar,baz"<br>Output:   GET /foo HTTP/1.1   my-header: foo,bar,baz |
| `add[]` | `object` | HTTPHeader represents an HTTP Header name and value as defined by RFC 7230. |
| `remove` | `array (string)` | Remove the given header(s) from the HTTP request before the action. The value of Remove is a list of HTTP header names. Note that the header names are case-insensitive (see https://datatracker.ietf.org/doc/html/rfc2616#section-4.2).<br>Input:   GET /foo HTTP/1.1   my-header1: foo   my-header2: bar   my-header3: baz<br>Config:   remove: ["my-header1", "my-header3"]<br>Output:   GET /foo HTTP/1.1   my-header2: bar |
| `set` | `array` | Set overwrites the request with the given header (name, value) before the action.<br>Input:   GET /foo HTTP/1.1   my-header: foo<br>Config:   set:   - name: "my-header"     value: "bar"<br>Output:   GET /foo HTTP/1.1   my-header: bar |
| `set[]` | `object` | HTTPHeader represents an HTTP Header name and value as defined by RFC 7230. |

### .spec.rules[].filters[].responseHeaderModifier.add {#_specrulesfiltersresponseheadermodifieradd}

Description
: Add adds the given header(s) (name, value) to the request before the action. It appends to any existing values associated with the header name.

  Input: GET /foo HTTP/1.1 my-header: foo

  Config: add:

  - name: "my-header" value: "bar,baz"

  Output: GET /foo HTTP/1.1 my-header: foo,bar,baz

Type
: ```
  `array`
  ```

### .spec.rules[].filters[].responseHeaderModifier.add[] {#_specrulesfiltersresponseheadermodifieradd}

Description
: HTTPHeader represents an HTTP Header name and value as defined by RFC 7230.

Type
: ```
  `object`
  ```

Required
: - `name`
  - `value`

| Property | Type | Description |
| --- | --- | --- |
| `name` | `string` | Name is the name of the HTTP Header to be matched. Name matching MUST be case-insensitive. (See https://tools.ietf.org/html/rfc7230#section-3.2).<br>If multiple entries specify equivalent header names, the first entry with an equivalent name MUST be considered for a match. Subsequent entries with an equivalent header name MUST be ignored. Due to the case-insensitivity of header names, "foo" and "Foo" are considered equivalent. |
| `value` | `string` | Value is the value of HTTP Header to be matched. |

### .spec.rules[].filters[].responseHeaderModifier.set {#_specrulesfiltersresponseheadermodifierset}

Description
: Set overwrites the request with the given header (name, value) before the action.

  Input: GET /foo HTTP/1.1 my-header: foo

  Config: set:

  - name: "my-header" value: "bar"

  Output: GET /foo HTTP/1.1 my-header: bar

Type
: ```
  `array`
  ```

### .spec.rules[].filters[].responseHeaderModifier.set[] {#_specrulesfiltersresponseheadermodifierset}

Description
: HTTPHeader represents an HTTP Header name and value as defined by RFC 7230.

Type
: ```
  `object`
  ```

Required
: - `name`
  - `value`

| Property | Type | Description |
| --- | --- | --- |
| `name` | `string` | Name is the name of the HTTP Header to be matched. Name matching MUST be case-insensitive. (See https://tools.ietf.org/html/rfc7230#section-3.2).<br>If multiple entries specify equivalent header names, the first entry with an equivalent name MUST be considered for a match. Subsequent entries with an equivalent header name MUST be ignored. Due to the case-insensitivity of header names, "foo" and "Foo" are considered equivalent. |
| `value` | `string` | Value is the value of HTTP Header to be matched. |

### .spec.rules[].matches {#_specrulesmatches}

Description
: Matches define conditions used for matching the rule against incoming gRPC requests. Each match is independent, i.e. this rule will be matched if **any** one of the matches is satisfied.

  For example, take the following matches configuration:

  matches:

  - method: service: foo.bar headers: values: version: 2
  - method: service: foo.bar.v2

  For a request to match against this rule, it MUST satisfy EITHER of the two conditions:

  - service of foo.bar AND contains the header `version: 2`
  - service of foo.bar.v2

  See the documentation for GRPCRouteMatch on how to specify multiple match conditions to be ANDed together.

  If no matches are specified, the implementation MUST match every gRPC request.

  Proxy or Load Balancer routing configuration generated from GRPCRoutes MUST prioritize rules based on the following criteria, continuing on ties. Merging MUST not be done between GRPCRoutes and HTTPRoutes. Precedence MUST be given to the rule with the largest number of:

  - Characters in a matching non-wildcard hostname.
  - Characters in a matching hostname.
  - Characters in a matching service.
  - Characters in a matching method.
  - Header matches.

  If ties still exist across multiple Routes, matching precedence MUST be determined in order of the following criteria, continuing on ties:

  - The oldest Route based on creation timestamp.
  - The Route appearing first in alphabetical order by "{namespace}/{name}".

  If ties still exist within the Route that has been given precedence, matching precedence MUST be granted to the first matching rule meeting the above criteria.

Type
: ```
  `array`
  ```

### .spec.rules[].matches[] {#_specrulesmatches}

Description
: GRPCRouteMatch defines the predicate used to match requests to a given action. Multiple match types are ANDed together, i.e. the match will evaluate to true only if all conditions are satisfied.

  For example, the match below will match a gRPC request only if its service is `foo` AND it contains the `version: v1` header:

  matches:

  - method: type: Exact service: "foo" headers:
  - name: "version" value "v1"

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `headers` | `array` | Headers specifies gRPC request header matchers. Multiple match values are ANDed together, meaning, a request MUST match all the specified headers to select the route. |
| `headers[]` | `object` | GRPCHeaderMatch describes how to select a gRPC route by matching gRPC request headers. |
| `method` | `object` | Method specifies a gRPC request service/method matcher. If this field is not specified, all services and methods will match. |

### .spec.rules[].matches[].headers {#_specrulesmatchesheaders}

Description
: Headers specifies gRPC request header matchers. Multiple match values are ANDed together, meaning, a request MUST match all the specified headers to select the route.

Type
: ```
  `array`
  ```

### .spec.rules[].matches[].headers[] {#_specrulesmatchesheaders}

Description
: GRPCHeaderMatch describes how to select a gRPC route by matching gRPC request headers.

Type
: ```
  `object`
  ```

Required
: - `name`
  - `value`

| Property | Type | Description |
| --- | --- | --- |
| `name` | `string` | Name is the name of the gRPC Header to be matched.<br>If multiple entries specify equivalent header names, only the first entry with an equivalent name MUST be considered for a match. Subsequent entries with an equivalent header name MUST be ignored. Due to the case-insensitivity of header names, "foo" and "Foo" are considered equivalent. |
| `type` | `string` | Type specifies how to match against the value of the header. |
| `value` | `string` | Value is the value of the gRPC Header to be matched. |

### .spec.rules[].matches[].method {#_specrulesmatchesmethod}

Description
: Method specifies a gRPC request service/method matcher. If this field is not specified, all services and methods will match.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `method` | `string` | Value of the method to match against. If left empty or omitted, will match all services.<br>At least one of Service and Method MUST be a non-empty string. |
| `service` | `string` | Value of the service to match against. If left empty or omitted, will match any service.<br>At least one of Service and Method MUST be a non-empty string. |
| `type` | `string` | Type specifies how to match against the service and/or method. Support: Core (Exact with service and method specified)<br>Support: Implementation-specific (Exact with method specified but no service specified)<br>Support: Implementation-specific (RegularExpression) |

### .status {#_status}

Description
: Status defines the current state of GRPCRoute.

Type
: ```
  `object`
  ```

Required
: - `parents`

| Property | Type | Description |
| --- | --- | --- |
| `parents` | `array` | Parents is a list of parent resources (usually Gateways) that are associated with the route, and the status of the route with respect to each parent. When this route attaches to a parent, the controller that manages the parent must add an entry to this list when the controller first sees the route and should update the entry as appropriate when the route or gateway is modified.<br>Note that parent references that cannot be resolved by an implementation of this API will not be added to this list. Implementations of this API can only populate Route status for the Gateways/parent resources they are responsible for.<br>A maximum of 32 Gateways will be represented in this list. An empty list means the route has not been attached to any Gateway. |
| `parents[]` | `object` | RouteParentStatus describes the status of a route with respect to an associated Parent. |

### .status.parents {#_statusparents}

Description
: Parents is a list of parent resources (usually Gateways) that are associated with the route, and the status of the route with respect to each parent. When this route attaches to a parent, the controller that manages the parent must add an entry to this list when the controller first sees the route and should update the entry as appropriate when the route or gateway is modified.

  Note that parent references that cannot be resolved by an implementation of this API will not be added to this list. Implementations of this API can only populate Route status for the Gateways/parent resources they are responsible for.

  A maximum of 32 Gateways will be represented in this list. An empty list means the route has not been attached to any Gateway.

Type
: ```
  `array`
  ```

### .status.parents[] {#_statusparents}

Description
: RouteParentStatus describes the status of a route with respect to an associated Parent.

Type
: ```
  `object`
  ```

Required
: - `conditions`
  - `controllerName`
  - `parentRef`

| Property | Type | Description |
| --- | --- | --- |
| `conditions` | `array` | Conditions describes the status of the route with respect to the Gateway. Note that the route’s availability is also subject to the Gateway’s own status conditions and listener status.<br>If the Route’s ParentRef specifies an existing Gateway that supports Routes of this kind AND that Gateway’s controller has sufficient access, then that Gateway’s controller MUST set the "Accepted" condition on the Route, to indicate whether the route has been accepted or rejected by the Gateway, and why.<br>A Route MUST be considered "Accepted" if at least one of the Route’s rules is implemented by the Gateway.<br>There are a number of cases where the "Accepted" condition may not be set due to lack of controller visibility, that includes when:<br>* The Route refers to a nonexistent parent. * The Route is of a type that the controller does not support. * The Route is in a namespace the controller does not have access to. |
| `conditions[]` | `object` | Condition contains details for one aspect of the current state of this API Resource. |
| `controllerName` | `string` | ControllerName is a domain/path string that indicates the name of the controller that wrote this status. This corresponds with the controllerName field on GatewayClass.<br>Example: "example.net/gateway-controller".<br>The format of this field is DOMAIN "/" PATH, where DOMAIN and PATH are valid Kubernetes names (https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names).<br>Controllers MUST populate this field when writing status. Controllers should ensure that entries to status populated with their ControllerName are cleaned up when they are no longer necessary. |
| `parentRef` | `object` | ParentRef corresponds with a ParentRef in the spec that this RouteParentStatus struct describes the status of. |

### .status.parents[].conditions {#_statusparentsconditions}

Description
: Conditions describes the status of the route with respect to the Gateway. Note that the route’s availability is also subject to the Gateway’s own status conditions and listener status.

  If the Route’s ParentRef specifies an existing Gateway that supports Routes of this kind AND that Gateway’s controller has sufficient access, then that Gateway’s controller MUST set the "Accepted" condition on the Route, to indicate whether the route has been accepted or rejected by the Gateway, and why.

  A Route MUST be considered "Accepted" if at least one of the Route’s rules is implemented by the Gateway.

  There are a number of cases where the "Accepted" condition may not be set due to lack of controller visibility, that includes when:

  - The Route refers to a nonexistent parent.
  - The Route is of a type that the controller does not support.
  - The Route is in a namespace the controller does not have access to.

Type
: ```
  `array`
  ```

### .status.parents[].conditions[] {#_statusparentsconditions}

Description
: Condition contains details for one aspect of the current state of this API Resource.

Type
: ```
  `object`
  ```

Required
: - `lastTransitionTime`
  - `message`
  - `reason`
  - `status`
  - `type`

| Property | Type | Description |
| --- | --- | --- |
| `lastTransitionTime` | `string` | lastTransitionTime is the last time the condition transitioned from one status to another. This should be when the underlying condition changed.  If that is not known, then using the time when the API field changed is acceptable. |
| `message` | `string` | message is a human readable message indicating details about the transition. This may be an empty string. |
| `observedGeneration` | `integer` | observedGeneration represents the .metadata.generation that the condition was set based upon. For instance, if .metadata.generation is currently 12, but the .status.conditions[x].observedGeneration is 9, the condition is out of date with respect to the current state of the instance. |
| `reason` | `string` | reason contains a programmatic identifier indicating the reason for the condition’s last transition. Producers of specific condition types may define expected values and meanings for this field, and whether the values are considered a guaranteed API. The value should be a CamelCase string. This field may not be empty. |
| `status` | `string` | status of the condition, one of True, False, Unknown. |
| `type` | `string` | type of condition in CamelCase or in foo.example.com/CamelCase. |

### .status.parents[].parentRef {#_statusparentsparentref}

Description
: ParentRef corresponds with a ParentRef in the spec that this RouteParentStatus struct describes the status of.

Type
: ```
  `object`
  ```

Required
: - `name`

| Property | Type | Description |
| --- | --- | --- |
| `group` | `string` | Group is the group of the referent. When unspecified, "gateway.networking.k8s.io" is inferred. To set the core API group (such as for a "Service" kind referent), Group must be explicitly set to "" (empty string).<br>Support: Core |
| `kind` | `string` | Kind is kind of the referent.<br>There are two kinds of parent resources with "Core" support:<br>* Gateway (Gateway conformance profile) * Service (Mesh conformance profile, ClusterIP Services only)<br>Support for other resources is Implementation-Specific. |
| `name` | `string` | Name is the name of the referent.<br>Support: Core |
| `namespace` | `string` | Namespace is the namespace of the referent. When unspecified, this refers to the local namespace of the Route.<br>Note that there are specific rules for ParentRefs which cross namespace boundaries. Cross-namespace references are only valid if they are explicitly allowed by something in the namespace they are referring to. For example: Gateway has the AllowedRoutes field, and ReferenceGrant provides a generic way to enable any other kind of cross-namespace reference.<br>Support: Core |
| `port` | `integer` | Port is the network port this Route targets. It can be interpreted differently based on the type of parent resource.<br>When the parent resource is a Gateway, this targets all listeners listening on the specified port that also support this kind of Route(and select this Route). It’s not recommended to set `Port` unless the networking behaviors specified in a Route must apply to a specific port as opposed to a listener(s) whose port(s) may be changed. When both Port and SectionName are specified, the name and port of the selected listener must match both specified values.<br>Implementations MAY choose to support other parent resources. Implementations supporting other types of parent resources MUST clearly document how/if Port is interpreted.<br>For the purpose of status, an attachment is considered successful as long as the parent resource accepts it partially. For example, Gateway listeners can restrict which Routes can attach to them by Route kind, namespace, or hostname. If 1 of 2 Gateway listeners accept attachment from the referencing Route, the Route MUST be considered successfully attached. If no Gateway listeners accept attachment from this Route, the Route MUST be considered detached from the Gateway.<br>Support: Extended |
| `sectionName` | `string` | SectionName is the name of a section within the target resource. In the following resources, SectionName is interpreted as the following:<br>* Gateway: Listener name. When both Port (experimental) and SectionName are specified, the name and port of the selected listener must match both specified values. * Service: Port name. When both Port (experimental) and SectionName are specified, the name and port of the selected listener must match both specified values.<br>Implementations MAY choose to support attaching Routes to other resources. If that is the case, they MUST clearly document how SectionName is interpreted.<br>When unspecified (empty string), this will reference the entire resource. For the purpose of status, an attachment is considered successful if at least one section in the parent resource accepts it. For example, Gateway listeners can restrict which Routes can attach to them by Route kind, namespace, or hostname. If 1 of 2 Gateway listeners accept attachment from the referencing Route, the Route MUST be considered successfully attached. If no Gateway listeners accept attachment from this Route, the Route MUST be considered detached from the Gateway.<br>Support: Core |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/gateway.networking.k8s.io/v1/grpcroutes`
  - `GET`: list objects of kind GRPCRoute
- `/apis/gateway.networking.k8s.io/v1/namespaces/{namespace}/grpcroutes`
  - `DELETE`: delete collection of GRPCRoute
  - `GET`: list objects of kind GRPCRoute
  - `POST`: create a GRPCRoute
- `/apis/gateway.networking.k8s.io/v1/namespaces/{namespace}/grpcroutes/{name}`
  - `DELETE`: delete a GRPCRoute
  - `GET`: read the specified GRPCRoute
  - `PATCH`: partially update the specified GRPCRoute
  - `PUT`: replace the specified GRPCRoute
- `/apis/gateway.networking.k8s.io/v1/namespaces/{namespace}/grpcroutes/{name}/status`
  - `GET`: read status of the specified GRPCRoute
  - `PATCH`: partially update status of the specified GRPCRoute
  - `PUT`: replace status of the specified GRPCRoute

### /apis/gateway.networking.k8s.io/v1/grpcroutes {#_apisgatewaynetworkingk8siov1grpcroutes}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  list objects of kind GRPCRoute
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`GRPCRouteList`](/docs/rest_api/objects#io-k8s-networking-gateway-v1-GRPCRouteList) schema |
| 401 - Unauthorized | Empty |

### /apis/gateway.networking.k8s.io/v1/namespaces/{namespace}/grpcroutes {#_apisgatewaynetworkingk8siov1namespaces_namespace_grpcroutes}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of GRPCRoute
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
  list objects of kind GRPCRoute
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`GRPCRouteList`](/docs/rest_api/objects#io-k8s-networking-gateway-v1-GRPCRouteList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create a GRPCRoute
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`GRPCRoute`](/docs/rest_api/network_apis/grpcroute-gateway-networking-k8s-io-v1#grpcroute-gateway-networking-k8s-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`GRPCRoute`](/docs/rest_api/network_apis/grpcroute-gateway-networking-k8s-io-v1#grpcroute-gateway-networking-k8s-io-v1) schema |
| 201 - Created | [`GRPCRoute`](/docs/rest_api/network_apis/grpcroute-gateway-networking-k8s-io-v1#grpcroute-gateway-networking-k8s-io-v1) schema |
| 202 - Accepted | [`GRPCRoute`](/docs/rest_api/network_apis/grpcroute-gateway-networking-k8s-io-v1#grpcroute-gateway-networking-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/gateway.networking.k8s.io/v1/namespaces/{namespace}/grpcroutes/{name} {#_apisgatewaynetworkingk8siov1namespaces_namespace_grpcroutes_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the GRPCRoute |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete a GRPCRoute
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
  read the specified GRPCRoute
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`GRPCRoute`](/docs/rest_api/network_apis/grpcroute-gateway-networking-k8s-io-v1#grpcroute-gateway-networking-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified GRPCRoute
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`GRPCRoute`](/docs/rest_api/network_apis/grpcroute-gateway-networking-k8s-io-v1#grpcroute-gateway-networking-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified GRPCRoute
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`GRPCRoute`](/docs/rest_api/network_apis/grpcroute-gateway-networking-k8s-io-v1#grpcroute-gateway-networking-k8s-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`GRPCRoute`](/docs/rest_api/network_apis/grpcroute-gateway-networking-k8s-io-v1#grpcroute-gateway-networking-k8s-io-v1) schema |
| 201 - Created | [`GRPCRoute`](/docs/rest_api/network_apis/grpcroute-gateway-networking-k8s-io-v1#grpcroute-gateway-networking-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

### /apis/gateway.networking.k8s.io/v1/namespaces/{namespace}/grpcroutes/{name}/status {#_apisgatewaynetworkingk8siov1namespaces_namespace_grpcroutes_name_status}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the GRPCRoute |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read status of the specified GRPCRoute
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`GRPCRoute`](/docs/rest_api/network_apis/grpcroute-gateway-networking-k8s-io-v1#grpcroute-gateway-networking-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update status of the specified GRPCRoute
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`GRPCRoute`](/docs/rest_api/network_apis/grpcroute-gateway-networking-k8s-io-v1#grpcroute-gateway-networking-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace status of the specified GRPCRoute
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`GRPCRoute`](/docs/rest_api/network_apis/grpcroute-gateway-networking-k8s-io-v1#grpcroute-gateway-networking-k8s-io-v1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`GRPCRoute`](/docs/rest_api/network_apis/grpcroute-gateway-networking-k8s-io-v1#grpcroute-gateway-networking-k8s-io-v1) schema |
| 201 - Created | [`GRPCRoute`](/docs/rest_api/network_apis/grpcroute-gateway-networking-k8s-io-v1#grpcroute-gateway-networking-k8s-io-v1) schema |
| 401 - Unauthorized | Empty |
