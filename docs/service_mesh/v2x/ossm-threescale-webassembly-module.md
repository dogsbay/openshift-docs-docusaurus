---
title: Using the 3scale WebAssembly module
sidebar_position: 23
---

# Using the 3scale WebAssembly module {#ossm-threescale-webassembly-module}

<a id="ossm-threescale-webassembly-module"></a>

:::note

The `threescale-wasm-auth` module runs on integrations of 3scale API Management 2.11 or later with Red Hat OpenShift Service Mesh 2.1.0 or later.

:::

The `threescale-wasm-auth` module is a [WebAssembly](https://webassembly.org) module that uses a set of interfaces, known as an application binary interfaces (*ABI*). This is defined by the [*Proxy-WASM*](https://github.com/proxy-wasm/spec) specification to drive any piece of software that implements the ABI so it can authorize HTTP requests against 3scale.

As an ABI specification, Proxy-WASM defines the interaction between a piece of software named *host* and another named *module*, *program*, or *extension*. The host exposes a set of services used by the module to perform a task, and in this case, to process proxy requests.

The host environment is composed of a WebAssembly virtual machine interacting with a piece of software, in this case, an HTTP proxy.

The module itself runs in isolation to the outside world except for the instructions it runs on the virtual machine and the ABI specified by Proxy-WASM. This is a safe way to provide extension points to software: the extension can only interact in well-defined ways with the virtual machine and the host. The interaction provides a computing model and a connection to the outside world the proxy is meant to have.

## Compatibility {#compatibility_ossm-threescale-webassembly-module}

The `threescale-wasm-auth` module is designed to be fully compatible with all implementations of the *Proxy-WASM ABI* specification. At this point, however, it has only been thoroughly tested to work with the [Envoy](https://www.envoyproxy.io) reverse proxy.

## Usage as a stand-alone module {#usage-as-a-stand-alone-module_ossm-threescale-webassembly-module}

Because of its self-contained design, it is possible to configure this module to work with Proxy-WASM proxies independently of Service Mesh, as well as 3scale Istio adapter deployments.

## Prerequisites {#prerequisites_ossm-threescale-webassembly-module}

- The module works with all supported 3scale releases, except when configuring a service to use [OpenID connect (OIDC)](/docs/authentication/identity_providers/configuring-oidc-identity-provider#configuring-oidc-identity-provider), which requires 3scale 2.11 or later.

## Configuring the threescale-wasm-auth module {#ossm-configuring-the-threescale-wasm-auth-module_ossm-threescale-webassembly-module}

Cluster administrators on OpenShift Container Platform can configure the `threescale-wasm-auth` module to authorize HTTP requests to 3scale API Management through an application binary interface (ABI). The ABI defines the interaction between host and the module, exposing the hosts services, and allows you to use the module to process proxy requests.

### The WasmPlugin API extension {#the-wasmplugin-api-extension_ossm-threescale-webassembly-module}

Service Mesh provides a custom resource definition to specify and apply Proxy-WASM extensions to sidecar proxies, known as [`WasmPlugin`](/docs/service_mesh/v2x/ossm-extensions#ossm-extensions-wasmplugin-format_ossm-extensions). Service Mesh applies this custom resource to the set of workloads that require HTTP API management with 3scale.

See [custom resource definition](/docs/operators/understanding/crds/crd-extending-api-with-crds#crd-extending-api-with-crds) for more information.

:::note

Configuring the WebAssembly extension is currently a manual process. Support for fetching the configuration for services from the 3scale system will be available in a future release.

:::

**Prerequisites**

- Identify a Kubernetes workload and namespace on your Service Mesh deployment that you will apply this module.
- You must have a 3scale tenant account. See [SaaS](https://www.3scale.net/signup) or [3scale 2.11 On-Premises](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.11/html-single/installing_3scale/index#install-threescale-on-openshift-guide) with a matching service and relevant applications and metrics defined.
- If you apply the module to the `<product_page>` microservice in the `bookinfo` namespace, see the [Bookinfo sample application](/docs/service_mesh/v2x/ossm-create-mesh#ossm-tutorial-bookinfo-overview_ossm-create-mesh).
  - The following example is the YAML format for the custom resource for `threescale-wasm-auth` module. This example refers to the upstream Maistra version of Service Mesh, `WasmPlugin` API. You must declare the namespace where the `threescale-wasm-auth` module is deployed, alongside a `selector` to identify the set of applications the module will apply to:
    ```yaml
    apiVersion: extensions.istio.io/v1alpha1
    kind: WasmPlugin
    metadata:
      name: <threescale_wasm_plugin_name>
      namespace: <bookinfo> (1)
    spec:
      selector: (2)
        labels:
          app: <product_page>
      pluginConfig: <yaml_configuration>
      url: oci://registry.redhat.io/3scale-amp2/3scale-auth-wasm-rhel8:0.0.3
      phase: AUTHZ
      priority: 100
    ```

    1. The `namespace`.
    2. The `selector`.
- The `spec.pluginConfig` field depends on the module configuration and it is not populated in the previous example. Instead, the example uses the `<yaml_configuration>` placeholder value. You can use the format of this custom resource example.
  - The `spec.pluginConfig` field varies depending on the application. All other fields persist across multiple instances of this custom resource. As examples:
    - `url`:  Only changes when newer versions of the module are deployed.
    - `phase`:  Remains the same, since this module needs to be invoked after the proxy has done any local authorization, such as validating OpenID Connect (OIDC) tokens.
- After you have the module configuration in `spec.pluginConfig` and the rest of the custom resource, apply it with the `oc apply` command:
  ```terminal
  $ oc apply -f threescale-wasm-auth-bookinfo.yaml
  ```

**Additional resources**

- [Migrating from `ServiceMeshExtension` to `WasmPlugin` resources](/docs/service_mesh/v2x/ossm-extensions#ossm-extensions-migration-overview_ossm-extensions)
- [Custom Resources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources)

## Applying 3scale external ServiceEntry objects {#ossm-threescale-applying-external-service-entry-objects_ossm-threescale-webassembly-module}

To have the `threescale-wasm-auth` module authorize requests against 3scale, the module must have access to 3scale services. You can do this within Red Hat OpenShift Service Mesh by applying an external `ServiceEntry` object and a corresponding `DestinationRule` object for TLS configuration to use the HTTPS protocol.

The custom resources (CRs) set up the service entries and destination rules for secure access from within Service Mesh to 3scale Hosted (SaaS) for the backend and system components of the Service Management API and the Account Management API. The Service Management API receives queries for the authorization status of each request. The Account Management API provides API management configuration settings for your services.

**Procedure**

1. Apply the following external `ServiceEntry` CR and related `DestinationRule` CR for 3scale Hosted **backend** to your cluster:
   1. Add the `ServiceEntry` CR to a file called `service-entry-threescale-saas-backend.yml`:
      ```yaml title="ServiceEntry CR"
      apiVersion: networking.istio.io/v1beta1
      kind: ServiceEntry
      metadata:
        name: service-entry-threescale-saas-backend
      spec:
        hosts:
        - su1.3scale.net
        ports:
        - number: 443
          name: https
          protocol: HTTPS
        location: MESH_EXTERNAL
        resolution: DNS
      ```
   2. Add the `DestinationRule` CR to a file called `destination-rule-threescale-saas-backend.yml`:
      ```yaml title="DestinationRule CR"
      apiVersion: networking.istio.io/v1beta1
      kind: DestinationRule
      metadata:
        name: destination-rule-threescale-saas-backend
      spec:
        host: su1.3scale.net
        trafficPolicy:
          tls:
            mode: SIMPLE
            sni: su1.3scale.net
      ```
   3. Apply and save the external `ServiceEntry` CR for the 3scale Hosted backend to your cluster, by running the following command:
      ```terminal
      $ oc apply -f service-entry-threescale-saas-backend.yml
      ```
   4. Apply and save the external `DestinationRule` CR for the 3scale Hosted backend to your cluster, by running the following command:
      ```terminal
      $ oc apply -f destination-rule-threescale-saas-backend.yml
      ```
2. Apply the following external `ServiceEntry` CR and related `DestinationRule` CR for 3scale Hosted **system** to your cluster:
   1. Add the `ServiceEntry` CR to a file called `service-entry-threescale-saas-system.yml`:
      ```terminal title="ServiceEntry CR"
      apiVersion: networking.istio.io/v1beta1
      kind: ServiceEntry
      metadata:
        name: service-entry-threescale-saas-system
      spec:
        hosts:
        - multitenant.3scale.net
        ports:
        - number: 443
          name: https
          protocol: HTTPS
        location: MESH_EXTERNAL
        resolution: DNS
      ```
   2. Add the `DestinationRule` CR to a file called `destination-rule-threescale-saas-system.yml`:
      ```terminal title="DestinationRule CR"
      apiVersion: networking.istio.io/v1beta1
      kind: DestinationRule
      metadata:
        name: destination-rule-threescale-saas-system
      spec:
        host: multitenant.3scale.net
        trafficPolicy:
          tls:
            mode: SIMPLE
            sni: multitenant.3scale.net
      ```
   3. Apply and save the external `ServiceEntry` CR for the 3scale Hosted system to your cluster, by running the following command:
      ```terminal
      $ oc apply -f service-entry-threescale-saas-system.yml
      ```
   4. Apply and save the external `DestinationRule` CR for the 3scale Hosted system to your cluster, by running the following command:
      ```terminal
      $ oc apply -f <destination-rule-threescale-saas-system.yml>
      ```

Alternatively, you can deploy an in-mesh 3scale service. To deploy an in-mesh 3scale service, change the location of the services in the CR by deploying 3scale and linking to the deployment.

**Additional resources**

- [Service entry and destination rule documentation](/docs/service_mesh/v2x/ossm-traffic-manage#ossm-routing-service-entries_traffic-management)

## The 3scale WebAssembly module configuration {#ossm-threescale-webassembly-module-configuration_ossm-threescale-webassembly-module}

The `WasmPlugin` custom resource spec provides the configuration that the `Proxy-WASM` module reads from.

The spec is embedded in the host and read by the `Proxy-WASM` module. Typically, the configurations are in the JSON file format for the modules to parse, however the `WasmPlugin` resource can interpret the spec value as YAML and convert it to JSON for consumption by the module.

If you use the `Proxy-WASM` module in stand-alone mode, you must write the configuration using the JSON format. Using the JSON format means using escaping and quoting where needed within the `host` configuration files, for example `Envoy`. When you use the WebAssembly module with the `WasmPlugin` resource, the configuration is in the YAML format. In this case, an invalid configuration forces the module to show diagnostics based on its JSON representation to a sidecar’s logging stream.

:::warning

The `EnvoyFilter` custom resource is not a supported API, although it can be used in some 3scale Istio adapter or Service Mesh releases. Using the `EnvoyFilter` custom resource is not recommended. Use the `WasmPlugin` API instead of the `EnvoyFilter` custom resource. If you must use the `EnvoyFilter` custom resource, you must specify the spec in JSON format.

:::

### Configuring the 3scale WebAssembly module {#threescale-configuring-the-threescale-webassembly-module_ossm-threescale-webassembly-module}

The architecture of the 3scale WebAssembly module configuration depends on the 3scale account and authorization service, and the list of services to handle.

**Prerequisites**

The prerequisites are a set of minimum mandatory fields in all cases:

- For the 3scale account and authorization service: the `backend-listener` URL.
- For the list of services to handle: the service IDs and at least one credential look up method and where to find it.
- You will find examples for dealing with `userkey`, `appid` with `appkey`, and OpenID Connect (OIDC) patterns.
- The WebAssembly module uses the settings you specified in the static configuration. For example, if you add a mapping rule configuration to the module, it will always apply, even when the 3scale Admin Portal has no such mapping rule. The rest of the `WasmPlugin` resource exists around the `spec.pluginConfig` YAML entry.

### The 3scale WebAssembly module api object {#ossm-threescale-webassembly-module-api-object_ossm-threescale-webassembly-module}

The `api` top-level string from the 3scale WebAssembly module defines which version of the configuration the module will use.

:::note

A non-existent or unsupported version of the `api` object renders the 3scale WebAssembly module inoperable.

:::

```yaml title="The api top-level string example"
apiVersion: extensions.istio.io/v1alpha1
kind: WasmPlugin
metadata:
  name: <threescale_wasm_plugin_name>
  namespace: <bookinfo>
spec:
  pluginConfig:
    api: v1
# ...
```

The `api` entry defines the rest of the values for the configuration. The only accepted value is `v1`. New settings that break compatibility with the current configuration or need more logic that modules using `v1` cannot handle, will require different values.

### The 3scale WebAssembly module system object {#ossm-threescale-webassembly-module-system-object_ossm-threescale-webassembly-module}

The `system` top-level object specifies how to access the 3scale Account Management API for a specific account. The `upstream` field is the most important part of the object. The `system` object is optional, but recommended unless you are providing a fully static configuration for the 3scale WebAssembly module, which is an option if you do not want to provide connectivity to the *system* component of 3scale.

When you provide static configuration objects in addition to the `system` object, the static ones always take precedence.

```yaml
apiVersion: extensions.istio.io/v1alpha1
kind: WasmPlugin
metadata:
  name: <threescale_wasm_plugin_name>
spec:
  pluginConfig:
    system:
      name: <saas_porta>
      upstream: <object>
      token: <my_account_token>
      ttl: 300
# ...
```

`system`** object fields**

<table>
<thead>
<tr>
  <th>Name</th>
  <th>Description</th>
  <th>Required</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>name</code></td>
  <td>An identifier for the 3scale service, currently not referenced elsewhere.</td>
  <td>Optional</td>
</tr>
<tr>
  <td><code>upstream</code></td>
  <td>The details about a network host to be contacted. <code>upstream</code> refers to the 3scale Account Management API host known as system.</td>
  <td>Yes</td>
</tr>
<tr>
  <td><code>token</code></td>
  <td>A 3scale personal access token with read permissions.</td>
  <td>Yes</td>
</tr>
<tr>
  <td><code>ttl</code></td>
  <td>The minimum amount of seconds to consider a configuration retrieved from this host as valid before trying to fetch new changes. The default is 600 seconds (10 minutes). <strong>Note:</strong> there is no maximum amount, but the module will generally fetch any configuration within a reasonable amount of time after this TTL elapses.</td>
  <td>Optional</td>
</tr>
</tbody>
</table>

### The 3scale WebAssembly module upstream object {#ossm-threescale-webassembly-module-upstream-object_ossm-threescale-webassembly-module}

The `upstream` object describes an external host to which the proxy can perform calls.

```yaml
apiVersion: maistra.io/v1
upstream:
  name: outbound|443||multitenant.3scale.net
  url: "https://myaccount-admin.3scale.net/"
  timeout: 5000
# ...
```

`upstream`** object fields**

<table>
<thead>
<tr>
  <th>Name</th>
  <th>Description</th>
  <th>Required</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>name</code></td>
  <td><code>name</code> is not a free-form identifier. It is the identifier for the external host as defined by the proxy configuration. In the case of stand-alone <code>Envoy</code> configurations, it maps to the name of a <a href="https://www.envoyproxy.io/docs/envoy/v1.19.0/api-v3/config/cluster/v3/cluster.proto#config-cluster-v3-cluster">Cluster</a>, also known as <code>upstream</code> in other proxies. <strong>Note:</strong> the value of this field, because the Service Mesh and 3scale Istio adapter control plane configure the name according to a format using a vertical bar (|) as the separator of multiple fields. For the purposes of this integration, always use the format: <code>outbound|&lt;port&gt;||&lt;hostname&gt;</code>.</td>
  <td>Yes</td>
</tr>
<tr>
  <td><code>url</code></td>
  <td>The complete URL to access the described service. Unless implied by the scheme, you must include the TCP port.</td>
  <td>Yes</td>
</tr>
<tr>
  <td><code>Timeout</code></td>
  <td>Timeout in milliseconds so that connections to this service that take more than the amount of time to respond will be considered errors. Default is 1000 seconds.</td>
  <td>Optional</td>
</tr>
</tbody>
</table>

### The 3scale WebAssembly module backend object {#ossm-threescale-webassembly-module-backend-object_ossm-threescale-webassembly-module}

The `backend` top-level object specifies how to access the 3scale Service Management API for authorizing and reporting HTTP requests. This service is provided by the *Backend* component of 3scale.

```yaml
apiVersion: extensions.istio.io/v1alpha1
kind: WasmPlugin
metadata:
  name: <threescale_wasm_plugin_name>
spec:
  pluginConfig:
# ...
    backend:
      name: backend
      upstream: <object>
# ...
```

`backend`** object fields**

<table>
<thead>
<tr>
  <th>Name</th>
  <th>Description</th>
  <th>Required</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>name</code></td>
  <td>An identifier for the 3scale backend, currently not referenced elsewhere.</td>
  <td>Optional</td>
</tr>
<tr>
  <td><code>upstream</code></td>
  <td>The details about a network host to be contacted. This must refer to the 3scale Account Management API host, known, system.</td>
  <td>Yes. The most important and required field.</td>
</tr>
</tbody>
</table>

### The 3scale WebAssembly module services object {#ossm-threescale-webassembly-module-services-object_ossm-threescale-webassembly-module}

The `services` top-level object specifies which service identifiers are handled by this particular instance of the `module`.

Since accounts have multiple services, you must specify which ones are handled. The rest of the configuration revolves around how to configure services.

The `services` field is required. It is an array that must contain at least one service to be useful.

```yaml
apiVersion: extensions.istio.io/v1alpha1
kind: WasmPlugin
metadata:
  name: <threescale_wasm_plugin_name>
spec:
  pluginConfig:
# ...
    services:
    - id: "2555417834789"
      token: service_token
      authorities:
        - "*.app"
        - 0.0.0.0
        - "0.0.0.0:8443"
      credentials: <object>
      mapping_rules: <object>
# ...
```

Each element in the `services` array represents a 3scale service.

`services`** object fields**

<table>
<thead>
<tr>
  <th>Name</th>
  <th>Description</th>
  <th>Required</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>ID</code></td>
  <td>An identifier for this 3scale service, currently not referenced elsewhere.</td>
  <td>Yes</td>
</tr>
<tr>
  <td><code>token</code></td>
  <td>This <code>token</code> can be found in the proxy configuration for your service in System or you can retrieve the it from System with following <code>curl</code> command:<br><br><code>curl \https://&lt;system_host&gt;/admin/api/services/&lt;service_id&gt;/proxy/configs/production/latest.json?access_token=&lt;access_token&gt;" | jq '.proxy_config.content.backend_authentication_value</code></td>
  <td>Optional</td>
</tr>
<tr>
  <td><code>authorities</code></td>
  <td>An array of strings, each one representing the <em>Authority</em> of a <em>URL</em> to match. These strings accept glob patterns supporting the asterisk (<em>*</em>), plus sign (<em>+</em>), and question mark (<em>?</em>) matchers.</td>
  <td>Yes</td>
</tr>
<tr>
  <td><code>credentials</code></td>
  <td>An object defining which kind of credentials to look for and where.</td>
  <td>Yes</td>
</tr>
<tr>
  <td><code>mapping_rules</code></td>
  <td>An array of objects representing mapping rules and 3scale methods to hit.</td>
  <td>Optional</td>
</tr>
</tbody>
</table>

### The 3scale WebAssembly module credentials object {#ossm-threescale-webassembly-module-credentials-object_ossm-threescale-webassembly-module}

The `credentials` object is a component of the `service` object. `credentials` specifies which kind of credentials to be looked up and the steps to perform this action.

All fields are optional, but you must specify at least one, `user_key` or `app_id`. The order in which you specify each credential is irrelevant because it is pre-established by the module. Only specify one instance of each credential.

```yaml
apiVersion: extensions.istio.io/v1alpha1
kind: WasmPlugin
metadata:
  name: <threescale_wasm_plugin_name>
spec:
  pluginConfig:
# ...
    services:
    - credentials:
        user_key: <array_of_lookup_queries>
        app_id: <array_of_lookup_queries>
        app_key: <array_of_lookup_queries>
# ...
```

`credentials`** object fields**

<table>
<thead>
<tr>
  <th>Name</th>
  <th>Description</th>
  <th>Required</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>user_key</code></td>
  <td>This is an array of lookup queries that defines a 3scale user key. A user key is commonly known as an API key.</td>
  <td>Optional</td>
</tr>
<tr>
  <td><code>app_id</code></td>
  <td>This is an array of lookup queries that define a 3scale application identifier. Application identifiers are provided by 3scale or by using an identity provider like <a href="https://access.redhat.com/products/red-hat-single-sign-on">Red Hat Single Sign-On (RH-SS0)</a>, or OpenID Connect (OIDC). The resolution of the lookup queries specified here, whenever it is successful and resolves to two values, it sets up the <code>app_id</code> and the <code>app_key</code>.</td>
  <td>Optional</td>
</tr>
<tr>
  <td><code>app_key</code></td>
  <td>This is an array of lookup queries that define a 3scale application key. Application keys without a resolved <code>app_id</code> are useless, so only specify this field when <code>app_id</code> has been specified.</td>
  <td>Optional</td>
</tr>
</tbody>
</table>

### The 3scale WebAssembly module lookup queries {#ossm-threescale-webassembly-module-lookup-queries_ossm-threescale-webassembly-module}

The `lookup query` object is part of any of the fields in the `credentials` object. It specifies how a given credential field should be found and processed. When evaluated, a successful resolution means that one or more values were found. A failed resolution means that no values were found.

Arrays of `lookup queries` describe a short-circuit or relationship: a successful resolution of one of the queries stops the evaluation of any remaining queries and assigns the value or values to the specified credential-type. Each query in the array is independent of each other.

A `lookup query` is made up of a single field, a source object, which can be one of a number of source types. See the following example:

```yaml
apiVersion: extensions.istio.io/v1alpha1
kind: WasmPlugin
metadata:
  name: <threescale_wasm_plugin_name>
spec:
  pluginConfig:
# ...
    services:
    - credentials:
        user_key:
          - <source_type>: <object>
          - <source_type>: <object>
# ...
        app_id:
          - <source_type>: <object>
# ...
        app_key:
          - <source_type>: <object>
# ...
```

### The 3scale WebAssembly module source object {#ossm-threescale-webassembly-module-source-object_ossm-threescale-webassembly-module}

A `source` object exists as part of an array of sources within any of the `credentials` object fields. The object field name, referred to as a `source`-type is any one of the following:

- `header`: The lookup query receives HTTP request headers as input.
- `query_string`: The `lookup query` receives the URL query string parameters as input.
- `filter`: The `lookup query` receives filter metadata as input.

All `source`-type objects have at least the following two fields:

`source`**-type object fields**

<table>
<thead>
<tr>
  <th>Name</th>
  <th>Description</th>
  <th>Required</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>keys</code></td>
  <td>An array of strings, each one a <code>key</code>, referring to entries found in the input data.</td>
  <td>Yes</td>
</tr>
<tr>
  <td><code>ops</code></td>
  <td>An array of <code>operations</code> that perform a <code>key</code> entry match. The array is a pipeline where operations receive inputs and generate outputs on the next operation. An <code>operation</code> failing to provide an output resolves the <code>lookup query</code> as failed. The pipeline order of the operations determines the evaluation order.</td>
  <td>Optional</td>
</tr>
</tbody>
</table>

The `filter` field name has a required `path` entry to show the path in the metadata you use to look up data.

When a `key` matches the input data, the rest of the keys are not evaluated and the source resolution algorithm jumps to executing the `operations` (`ops`) specified, if any. If no `ops` are specified, the result value of the matching `key`, if any, is returned.

`Operations` provide a way to specify certain conditions and transformations for inputs you have after the first phase looks up a `key`. Use `operations` when you need to transform, decode, and assert properties, however they do not provide a mature language to deal with all needs and lack *Turing-completeness*.

A stack stored the outputs of `operations`. When evaluated, the `lookup query` finishes by assigning the value or values at the bottom of the stack, depending on how many values the credential consumes.

### The 3scale WebAssembly module operations object {#ossm-threescale-webassembly-module-operations-object_ossm-threescale-webassembly-module}

Each element in the `ops` array belonging to a specific `source type` is an `operation` object that either applies transformations to values or performs tests. The field name to use for such an object is the name of the `operation` itself, and any values are the parameters to the `operation`, which could be structure objects, for example, maps with fields and values, lists, or strings.

Most `operations` attend to one or more inputs, and produce one or more outputs. When they consume inputs or produce outputs, they work with a stack of values: each value consumed by the operations is popped from the stack of values and initially populated with any `source` matches. The values outputted by them are pushed to the stack. Other `operations` do not consume or produce outputs other than asserting certain properties, but they inspect a stack of values.

:::note

When resolution finishes, the values picked up by the next step, such as assigning the values to be an `app_id`, `app_key`, or `user_key`, are taken from the bottom values of the stack.

:::

There are a few different `operations` categories:

- `decode`: These transform an input value by decoding it to get a different format.
- `string`: These take a string value as input and perform transformations and checks on it.
- `stack`: These take a set of values in the input and perform multiple stack transformations and selection of specific positions in the stack.
- `check`: These assert properties about sets of operations in a side-effect free way.
- `control`: These perform operations that allow for modifying the evaluation flow.
- `format`: These parse the format-specific structure of input values and look up values in it.

All operations are specified by the name identifiers as strings.

**Additional resources**

- Available [operations](https://github.com/3scale/threescale-wasm-auth/blob/main/docs/operations.md)

### The 3scale WebAssembly module mapping_rules object {#ossm-threescale-webassembly-module-mapping-rules-object_ossm-threescale-webassembly-module}

The `mapping_rules` object is part of the `service` object. It specifies a set of REST path patterns and related 3scale metrics and count increments to use when the patterns match.

You need the value if no dynamic configuration is provided in the `system` top-level object. If the object is provided in addition to the `system` top-level entry, then the `mapping_rules` object is evaluated first.

`mapping_rules` is an array object. Each element of that array is a `mapping_rule` object. The evaluated matching mapping rules on an incoming request provide the set of 3scale `methods` for authorization and reporting to the *APIManager*. When multiple matching rules refer to the same `methods`, there is a summation of `deltas` when calling into 3scale. For example, if two rules increase the *Hits* method twice with `deltas` of 1 and 3, a single method entry for Hits reporting to 3scale has a `delta` of 4.

### The 3scale WebAssembly module mapping_rule object {#ossm-threescale-webassembly-module-mapping-rule-object_ossm-threescale-webassembly-module}

The `mapping_rule` object is part of an array in the `mapping_rules` object.

The `mapping_rule` object fields specify the following information:

- The *HTTP request method* to match.
- A pattern to match the path against.
- The 3scale methods to report along with the amount to report. The order in which you specify the fields determines the evaluation order.

`mapping_rule`** object fields**

<table>
<thead>
<tr>
  <th>Name</th>
  <th>Description</th>
  <th>Required</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>method</code></td>
  <td>Specifies a string representing an HTTP request method, also known as verb. Values accepted match the any one of the accepted HTTP method names, case-insensitive. A special value of any matches any method.</td>
  <td>Yes</td>
</tr>
<tr>
  <td><code>pattern</code></td>
  <td>The pattern to match the HTTP request's URI path component. This pattern follows the same syntax as documented by 3scale. It allows wildcards (use of the asterisk (*) character) using any sequence of characters between braces such as <code>{this}</code>.</td>
  <td>Yes</td>
</tr>
<tr>
  <td><code>usages</code></td>
  <td>A list of <code>usage</code> objects. When the rule matches, all methods with their <code>deltas</code> are added to the list of methods sent to 3scale for authorization and reporting.<br><br>Embed the <code>usages</code> object with the following required fields:<br><br><ul><li><code>name</code>: The <code>method</code> system name to report.</li><li><code>delta</code>: For how much to increase that <code>method</code> by.</li></ul></td>
  <td>Yes</td>
</tr>
<tr>
  <td><code>last</code></td>
  <td>Whether the successful matching of this rule should stop the evaluation of more mapping rules.</td>
  <td>Optional Boolean. The default is <code>false</code></td>
</tr>
</tbody>
</table>

The following example is independent of existing hierarchies between methods in 3scale. That is, anything run on the 3scale side will not affect this. For example, the *Hits* metric might be a parent of them all, so it stores 4 hits due to the sum of all reported methods in the authorized request and calls the 3scale `Authrep` API endpoint.

The example below uses a `GET` request to a path, `/products/1/sold`, that matches all the rules.

```yaml title="mapping_rules GET request example"
apiVersion: extensions.istio.io/v1alpha1
kind: WasmPlugin
metadata:
  name: <threescale_wasm_plugin_name>
spec:
  pluginConfig:
# ...
    mapping_rules:
      - method: GET
        pattern: /
        usages:
          - name: hits
            delta: 1
      - method: GET
        pattern: /products/
        usages:
          - name: products
            delta: 1
      - method: ANY
        pattern: /products/{id}/sold
        usages:
          - name: sales
            delta: 1
          - name: products
            delta: 1
# ...
```

All `usages` get added to the request the module performs to 3scale with usage data as follows:

- Hits: 1
- products: 2
- sales: 1

## The 3scale WebAssembly module examples for credentials use cases {#ossm-threescale-webassembly-module-examples-for-credentials-use-cases_ossm-threescale-webassembly-module}

You will spend most of your time applying configuration steps to obtain credentials in the requests to your services.

The following are `credentials` examples, which you can modify to adapt to specific use cases.

You can combine them all, although when you specify multiple source objects with their own `lookup queries`, they are evaluated in order until one of them successfully resolves.

### API key (user_key) in query string parameters {#api-key-in-query-string-parameters_ossm-threescale-webassembly-module}

The following example looks up a `user_key` in a query string parameter or header of the same name:

```yaml
apiVersion: extensions.istio.io/v1alpha1
kind: WasmPlugin
metadata:
  name: <threescale_wasm_plugin_name>
spec:
# ...
    services:
# ...
      credentials:
        user_key:
          - query_string:
              keys:
                - <user_key>
          - header:
              keys:
                - <user_key>
# ...
```

### Application ID and key {#application-id-and-key_ossm-threescale-webassembly-module}

The following example looks up `app_key` and `app_id` credentials in a query or headers.

```yaml
apiVersion: extensions.istio.io/v1alpha1
kind: WasmPlugin
metadata:
  name: <threescale_wasm_plugin_name>
spec:
# ...
    services:
# ...
      credentials:
        app_id:
          - query_string:
              keys:
                - <app_id>
          - header:
              keys:
                - <app_id>
        app_key:
          - query_string:
              keys:
                - <app_key>
          - header:
              keys:
                - <app_key>
# ...
```

### Authorization header {#authorization-header_ossm-threescale-webassembly-module}

A request includes an `app_id` and `app_key` in an `authorization` header. If there is at least one or two values outputted at the end, then you can assign the `app_key`.

The resolution here assigns the `app_key` if there is one or two outputted at the end.

The `authorization` header specifies a value with the type of authorization and its value is encoded as `Base64`. This means you can split the value by a space character, take the second output and then split it again using a colon (:) as the separator. For example, if you use this format `app_id:app_key`, the header looks like the following example for `credential`:

```
aladdin:opensesame:  Authorization: Basic YWxhZGRpbjpvcGVuc2VzYW1l
```

You must use lower case header field names as shown in the following example:

```yaml
apiVersion: extensions.istio.io/v1alpha1
kind: WasmPlugin
metadata:
  name: <threescale_wasm_plugin_name>
spec:
# ...
    services:
# ...
      credentials:
        app_id:
          - header:
              keys:
                - authorization
              ops:
                - split:
                    separator: " "
                    max: 2
                - length:
                    min: 2
                - drop:
                    head: 1
                - base64_urlsafe
                - split:
                    max: 2
        app_key:
          - header:
              keys:
                - app_key
# ...
```

The previous example use case looks at the headers for an `authorization`:

1. It takes its string value and split it by a space, checking that it generates at least two values of a `credential`-type and the `credential` itself, then dropping the `credential`-type.
2. It then decodes the second value containing the data it needs, and splits it by using a colon (:) character to have an operations stack including first the `app_id`, then the `app_key`, if it exists.
   1. If `app_key` does not exist in the authorization header then its specific sources are checked, for example, the header with the key `app_key` in this case.
3. To add extra conditions to `credentials`, allow `Basic` authorizations, where `app_id` is either `aladdin` or `admin`, or any `app_id` being at least 8 characters in length.
4. `app_key` must contain a value and have a minimum of 64 characters as shown in the following example:
   ```yaml
   apiVersion: extensions.istio.io/v1alpha1
   kind: WasmPlugin
   metadata:
     name: <threescale_wasm_plugin_name>
   spec:
   # ...
       services:
   # ...
         credentials:
           app_id:
             - header:
                 keys:
                   - authorization
                 ops:
                   - split:
                       separator: " "
                       max: 2
                   - length:
                       min: 2
                   - reverse
                   - glob:
                     - Basic
                   - drop:
                       tail: 1
                   - base64_urlsafe
                   - split:
                       max: 2
                    - test:
                       if:
                         length:
                           min: 2
                      then:
                         - strlen:
                             max: 63
                         - or:
                             - strlen:
                                 min: 1
                             - drop:
                                 tail: 1
                   - assert:
                     - and:
                       - reverse
                       - or:
                         - strlen:
                             min: 8
                         - glob:
                           - aladdin
                           - admin
   # ...
   ```
5. After picking up the `authorization` header value, you get a `Basic` `credential`-type by reversing the stack so that the type is placed on top.
6. Run a glob match on it. When it validates, and the credential is decoded and split, you get the `app_id` at the bottom of the stack, and potentially the `app_key` at the top.
7. Run a `test:` if there are two values in the stack, meaning an `app_key` was acquired.
   1. Ensure the string length is between 1 and 63, including `app_id` and `app_key`. If the key’s length is zero, drop it and continue as if no key exists. If there was only an `app_id` and no `app_key`, the missing else branch indicates a successful test and evaluation continues.

The last operation, `assert`, indicates that no side-effects make it into the stack. You can then modify the stack:

1. Reverse the stack to have the `app_id` at the top.
   1. Whether or not an `app_key` is present, reversing the stack ensures `app_id` is at the top.
2. Use `and` to preserve the contents of the stack across tests.
   Then use one of the following possibilities:

   - Make sure `app_id` has a string length of at least 8.
   - Make sure `app_id` matches either `aladdin` or `admin`.

### OpenID Connect (OIDC) use case {#openid-connect-use-case_ossm-threescale-webassembly-module}

For Service Mesh and the 3scale Istio adapter, you must deploy a `RequestAuthentication` as shown in the following example, filling in your own workload data and `jwtRules`:

```yaml
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: jwt-example
  namespace: bookinfo
spec:
  selector:
    matchLabels:
      app: productpage
  jwtRules:
  - issuer: >-
      http://keycloak-keycloak.34.242.107.254.nip.io/auth/realms/3scale-keycloak
    jwksUri: >-
      http://keycloak-keycloak.34.242.107.254.nip.io/auth/realms/3scale-keycloak/protocol/openid-connect/certs
```

When you apply the `RequestAuthentication`, it configures `Envoy` with a [native plugin](https://www.envoyproxy.io/docs/envoy/v1.19.0/api-v3/extensions/filters/http/jwt_authn/v3/config.proto.html) to validate `JWT` tokens. The proxy validates everything before running the module so any requests that fail do not make it to the 3scale WebAssembly module.

When a `JWT` token is validated, the proxy stores its contents in an internal metadata object, with an entry whose key depends on the specific configuration of the plugin. This use case gives you the ability to look up structure objects with a single entry containing an unknown key name.

The 3scale `app_id` for OIDC matches the OAuth `client_id`. This is found in the `azp` or `aud` fields of `JWT` tokens.

To get `app_id` field from Envoy’s native `JWT` authentication filter, see the following example:

```yaml
apiVersion: extensions.istio.io/v1alpha1
kind: WasmPlugin
metadata:
  name: <threescale_wasm_plugin_name>
spec:
# ...
    services:
# ...
      credentials:
        app_id:
          - filter:
              path:
                - envoy.filters.http.jwt_authn
                - "0"
              keys:
                - azp
                - aud
              ops:
                - take:
                    head: 1
# ...
```

The example instructs the module to use the `filter` source type to look up filter metadata for an object from the `Envoy`-specific `JWT` authentication native plugin. This plugin includes the `JWT` token as part of a structure object with a single entry and a preconfigured name. Use `0` to specify that you will only access the single entry.

The resulting value is a structure for which you will resolve two fields:

- `azp`: The value where `app_id` is found.
- `aud`: The value where this information can also be found.

The operation ensures only one value is held for assignment.

### Picking up the JWT token from a header {#picking-up-the-jwt-token-from-a-header_ossm-threescale-webassembly-module}

Some setups might have validation processes for `JWT` tokens where the validated token would reach this module via a header in JSON format.

To get the `app_id`, see the following example:

```yaml
apiVersion: extensions.istio.io/v1alpha1
kind: WasmPlugin
metadata:
  name: <threescale_wasm_plugin_name>
spec:
# ...
    services:
# ...
      credentials:
        app_id:
          - header:
              keys:
                - x-jwt-payload
              ops:
                - base64_urlsafe
                - json:
                  - keys:
                    - azp
                    - aud
                - take:
                    head: 1
# ,,,
```

## 3scale WebAssembly module minimal working configuration {#ossm-threescale-webassembly-module-minimal-working-configuration_ossm-threescale-webassembly-module}

The following is an example of a 3scale WebAssembly module minimal working configuration. You can copy and paste this and edit it to work with your own configuration.

```yaml
apiVersion: extensions.istio.io/v1alpha1
kind: WasmPlugin
metadata:
  name: <threescale_wasm_plugin_name>
spec:
  url: oci://registry.redhat.io/3scale-amp2/3scale-auth-wasm-rhel8:0.0.3
  imagePullSecret: <optional_pull_secret_resource>
  phase: AUTHZ
  priority: 100
  selector:
    labels:
      app: <product_page>
  pluginConfig:
    api: v1
    system:
      name: <system_name>
      upstream:
        name: outbound|443||multitenant.3scale.net
        url: https://istiodevel-admin.3scale.net/
        timeout: 5000
      token: <token>
    backend:
      name: <backend_name>
      upstream:
        name: outbound|443||su1.3scale.net
        url: https://su1.3scale.net/
        timeout: 5000
      extensions:
      - no_body
    services:
    - id: '2555417834780'
      authorities:
      - "*"
      credentials:
        user_key:
          - query_string:
              keys:
                - <user_key>
          - header:
              keys:
                - <user_key>
        app_id:
          - query_string:
              keys:
                - <app_id>
          - header:
              keys:
                - <app_id>
        app_key:
          - query_string:
              keys:
                - <app_key>
          - header:
              keys:
                - <app_key>
```
