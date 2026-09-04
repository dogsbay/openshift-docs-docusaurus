---
title: Extensions
sidebar_position: 21
---

# Extensions {#ossm-extensions}

<a id="ossm-extensions"></a>

You can use WebAssembly extensions to add new features directly into the Red Hat OpenShift Service Mesh proxies. This lets you move even more common functionality out of your applications, and implement them in a single language that compiles to WebAssembly bytecode.

:::note

WebAssembly extensions are not supported on IBM Z(R) and IBM Power(R).

:::

## WebAssembly modules overview {#ossm-extensions-overview_ossm-extensions}

WebAssembly modules can be run on many platforms, including proxies, and have broad language support, fast execution, and a sandboxed-by-default security model.

Red Hat OpenShift Service Mesh extensions are [Envoy HTTP Filters](https://www.envoyproxy.io/docs/envoy/v1.20.0/intro/arch_overview/http/http_filters#arch-overview-http-filters), giving them a wide range of capabilities:

- Manipulating the body and headers of requests and responses.
- Out-of-band HTTP requests to services not in the request path, such as authentication or policy checking.
- Side-channel data storage and queues for filters to communicate with each other.

:::note

When creating new WebAssembly extensions, use the `WasmPlugin` API. The `ServiceMeshExtension` API was deprecated in Red Hat OpenShift Service Mesh version 2.2 and was removed in Red Hat OpenShift Service Mesh version 2.3.

:::

There are two parts to writing a Red Hat OpenShift Service Mesh extension:

1. You must write your extension using an SDK that exposes the [proxy-wasm API](https://github.com/proxy-wasm/spec) and compile it to a WebAssembly module.
2. You must then package the module into a container.

**Supported languages**

You can use any language that compiles to WebAssembly bytecode to write a Red Hat OpenShift Service Mesh extension, but the following languages have existing SDKs that expose the proxy-wasm API so that it can be consumed directly.

**Supported languages**

| Language | Maintainer | Repository |
| --- | --- | --- |
| AssemblyScript | solo.io | [solo-io/proxy-runtime](https://github.com/solo-io/proxy-runtime) |
| C++ | proxy-wasm team (Istio Community) | [proxy-wasm/proxy-wasm-cpp-sdk](https://github.com/proxy-wasm/proxy-wasm-cpp-sdk) |
| Go | tetrate.io | [tetratelabs/proxy-wasm-go-sdk](https://github.com/tetratelabs/proxy-wasm-go-sdk) |
| Rust | proxy-wasm team (Istio Community) | [proxy-wasm/proxy-wasm-rust-sdk](https://github.com/proxy-wasm/proxy-wasm-rust-sdk) |

## `WasmPlugin` container format {#ossm-extensions-wasmplugin-format_ossm-extensions}

Istio supports Open Container Initiative (OCI) images in its Wasm Plugin mechanism. You can distribute your Wasm Plugins as a container image, and you can use the `spec.url` field to refer to a container registry location.  For example, `quay.io/my-username/my-plugin:latest`.

Because each execution environment (runtime) for a WASM module can have runtime-specific configuration parameters, a WASM image can be composed of two layers:

- **plugin.wasm** (Required) - Content layer. This layer consists of a `.wasm` binary containing the bytecode of your WebAssembly module, to be loaded by the runtime.  You must name this file `plugin.wasm`.
- **runtime-config.json** (Optional) - Configuration layer. This layer consists of a JSON-formatted string that describes metadata about the module for the target runtime. The config layer might also contain additional data, depending on the target runtime. For example, the config for a WASM Envoy Filter contains root_ids available on the filter.

## WasmPlugin API reference {#ossm-wasm-ref-wasmplugin_ossm-extensions}

The WasmPlugins API provides a mechanism to extend the functionality provided by the Istio proxy through WebAssembly filters.

You can deploy multiple WasmPlugins. The `phase` and `priority` settings determine the order of execution (as part of Envoy’s filter chain), allowing the configuration of complex interactions between user-supplied WasmPlugins and Istio’s internal filters.

In the following example, an authentication filter implements an OpenID flow and populates the Authorization header with a JSON Web Token (JWT). Istio authentication consumes this token and deploys it to the ingress gateway. The WasmPlugin file lives in the proxy sidecar filesystem. Note the field `url`.

```yaml
apiVersion: extensions.istio.io/v1alpha1
kind: WasmPlugin
metadata:
  name: openid-connect
  namespace: istio-ingress
spec:
  selector:
    matchLabels:
      istio: ingressgateway
  url: file:///opt/filters/openid.wasm
  sha256: 1ef0c9a92b0420cf25f7fe5d481b231464bc88f486ca3b9c83ed5cc21d2f6210
  phase: AUTHN
  pluginConfig:
    openid_server: authn
    openid_realm: ingress
```

Below is the same example, but this time an Open Container Initiative (OCI) image is used instead of a file in the filesystem. Note the fields `url`, `imagePullPolicy`, and `imagePullSecret`.

```yaml
apiVersion: extensions.istio.io/v1alpha1
kind: WasmPlugin
metadata:
  name: openid-connect
  namespace: istio-system
spec:
  selector:
    matchLabels:
      istio: ingressgateway
  url: oci://private-registry:5000/openid-connect/openid:latest
  imagePullPolicy: IfNotPresent
  imagePullSecret: private-registry-pull-secret
  phase: AUTHN
  pluginConfig:
    openid_server: authn
    openid_realm: ingress
```

**WasmPlugin Field Reference**

<table>
<tbody>
<tr>
  <td>Field</td>
  <td>Type</td>
  <td>Description</td>
  <td>Required</td>
</tr>
<tr>
  <td>spec.selector</td>
  <td>WorkloadSelector</td>
  <td>Criteria used to select the specific set of pods/VMs on which this plugin configuration should be applied. If omitted, this configuration will be applied to all workload instances in the same namespace. If the <code>WasmPlugin</code> field is present in the config root namespace, it will be applied to all applicable workloads in any namespace.</td>
  <td>No</td>
</tr>
<tr>
  <td>spec.url</td>
  <td>string</td>
  <td>URL of a Wasm module or OCI container. If no scheme is present, defaults to <code>oci://</code>, referencing an OCI image. Other valid schemes are <code>file://</code> for referencing .wasm module files present locally within the proxy container, and <code>http[s]://</code> for .wasm module files hosted remotely.</td>
  <td>No</td>
</tr>
<tr>
  <td>spec.sha256</td>
  <td>string</td>
  <td>SHA256 checksum that will be used to verify the Wasm module or OCI container. If the <code>url</code> field already references a SHA256 (using the <code>@sha256:</code> notation), it must match the value of this field. If an OCI image is referenced by tag and this field is set, its checksum will be verified against the contents of this field after pulling.</td>
  <td>No</td>
</tr>
<tr>
  <td>spec.imagePullPolicy</td>
  <td>PullPolicy</td>
  <td>The pull behavior to be applied when fetching an OCI image. Only relevant when images are referenced by tag instead of SHA. Defaults to the value <code>IfNotPresent</code>, except when an OCI image is referenced in the <code>url</code> field and the <code>latest</code> tag is used, in which case the value <code>Always</code> is the default, mirroring K8s behavior. Setting is ignored if the <code>url</code> field is referencing a Wasm module directly using <code>file://</code> or <code>http[s]://</code>.</td>
  <td>No</td>
</tr>
<tr>
  <td>spec.imagePullSecret</td>
  <td>string</td>
  <td>Credentials to use for OCI image pulling. The name of a secret in the same namespace as the <code>WasmPlugin</code> object that contains a pull secret for authenticating against the registry when pulling the image.</td>
  <td>No</td>
</tr>
<tr>
  <td>spec.phase</td>
  <td>PluginPhase</td>
  <td>Determines where in the filter chain this <code>WasmPlugin</code> object is injected.</td>
  <td>No</td>
</tr>
<tr>
  <td>spec.priority</td>
  <td><code>int64</code></td>
  <td>Determines the ordering of <code>WasmPlugins</code> objects that have the same <code>phase</code> value. When multiple <code>WasmPlugins</code> objects are applied to the same workload in the same phase, they will be applied by priority and in descending order. If the <code>priority</code> field is not set, or two <code>WasmPlugins</code> objects with the same value, the ordering will be determined from the name and namespace of the <code>WasmPlugins</code> objects. Defaults to the value <code>0</code>.</td>
  <td>No</td>
</tr>
<tr>
  <td>spec.pluginName</td>
  <td>string</td>
  <td>The plugin name used in the Envoy configuration. Some Wasm modules might require this value to select the Wasm plugin to execute.</td>
  <td>No</td>
</tr>
<tr>
  <td>spec.pluginConfig</td>
  <td>Struct</td>
  <td>The configuration that will be passed on to the plugin.</td>
  <td>No</td>
</tr>
<tr>
  <td>spec.pluginConfig.verificationKey</td>
  <td>string</td>
  <td>The public key used to verify signatures of signed OCI images or Wasm modules. Must be supplied in PEM format.</td>
  <td>No</td>
</tr>
</tbody>
</table>

The `WorkloadSelector` object specifies the criteria used to determine if a filter can be applied to a proxy. The matching criteria includes the metadata associated with a proxy, workload instance information such as labels attached to the pod/VM, or any other information that the proxy provides to Istio during the initial handshake. If multiple conditions are specified, all conditions need to match in order for the workload instance to be selected. Currently, only label based selection mechanism is supported.

**WorkloadSelector**

<table>
<tbody>
<tr>
  <td>Field</td>
  <td>Type</td>
  <td>Description</td>
  <td>Required</td>
</tr>
<tr>
  <td>matchLabels</td>
  <td>map&lt;string, string></td>
  <td>One or more labels that indicate a specific set of pods/VMs on which a policy should be applied. The scope of label search is restricted to the configuration namespace in which the resource is present.</td>
  <td>Yes</td>
</tr>
</tbody>
</table>

The `PullPolicy` object specifies the pull behavior to be applied when fetching an OCI image.

**PullPolicy**

<table>
<tbody>
<tr>
  <td>Value</td>
  <td>Description</td>
</tr>
<tr>
  <td><empty></td>
  <td>Defaults to the value <code>IfNotPresent</code>, except for OCI images with tag latest, for which the default will be the value <code>Always</code>.</td>
</tr>
<tr>
  <td>IfNotPresent</td>
  <td>If an existing version of the image has been pulled before, that will be used. If no version of the image is present locally, we will pull the latest version.</td>
</tr>
<tr>
  <td>Always</td>
  <td>Always pull the latest version of an image when applying this plugin.</td>
</tr>
</tbody>
</table>

`Struct` represents a structured data value, consisting of fields which map to dynamically typed values. In some languages, Struct might be supported by a native representation. For example, in scripting languages like JavaScript a struct is represented as an object.

**Struct**

<table>
<tbody>
<tr>
  <td>Field</td>
  <td>Type</td>
  <td>Description</td>
</tr>
<tr>
  <td>fields</td>
  <td>map&lt;string, Value></td>
  <td>Map of dynamically typed values.</td>
</tr>
</tbody>
</table>

`PluginPhase` specifies the phase in the filter chain where the plugin will be injected.

**PluginPhase**

<table>
<tbody>
<tr>
  <td>Field</td>
  <td>Description</td>
</tr>
<tr>
  <td><empty></td>
  <td>Control plane decides where to insert the plugin. This will generally be at the end of the filter chain, right before the Router. Do not specify PluginPhase if the plugin is independent of others.</td>
</tr>
<tr>
  <td>AUTHN</td>
  <td>Insert plugin before Istio authentication filters.</td>
</tr>
<tr>
  <td>AUTHZ</td>
  <td>Insert plugin before Istio authorization filters and after Istio authentication filters.</td>
</tr>
<tr>
  <td>STATS</td>
  <td>Insert plugin before Istio stats filters and after Istio authorization filters.</td>
</tr>
</tbody>
</table>

### Deploying `WasmPlugin` resources {#ossm-wasmplugin-deploy_ossm-extensions}

You can enable Red Hat OpenShift Service Mesh extensions using the `WasmPlugin` resource. In this example, `istio-system` is the name of the Service Mesh control plane project. The following example creates an `openid-connect` filter that performs an OpenID Connect flow to authenticate the user.

**Procedure**

1. Create the following example resource:
   ```yaml title="Example plugin.yaml"
   apiVersion: extensions.istio.io/v1alpha1
   kind: WasmPlugin
   metadata:
     name: openid-connect
     namespace: istio-system
   spec:
     selector:
       matchLabels:
         istio: ingressgateway
     url: oci://private-registry:5000/openid-connect/openid:latest
     imagePullPolicy: IfNotPresent
     imagePullSecret: private-registry-pull-secret
     phase: AUTHN
     pluginConfig:
       openid_server: authn
       openid_realm: ingress
   ```
2. Apply your `plugin.yaml` file with the following command:
   ```terminal
   $ oc apply -f plugin.yaml
   ```

## `ServiceMeshExtension` container format {#ossm-extensions-smextension-format_ossm-extensions}

You must have a `.wasm` file containing the bytecode of your WebAssembly module, and a `manifest.yaml` file in the root of the container filesystem to make your container image a valid extension image.

:::note

When creating new WebAssembly extensions, use the `WasmPlugin` API. The `ServiceMeshExtension` API was deprecated in Red Hat OpenShift Service Mesh version 2.2 and was removed in Red Hat OpenShift Service Mesh version 2.3.

:::

```yaml title="manifest.yaml"
schemaVersion: 1

name: <your-extension>
description: <description>
version: 1.0.0
phase: PreAuthZ
priority: 100
module: extension.wasm
```

**Field Reference for manifest.yml**

<table>
<tbody>
<tr>
  <td>Field</td>
  <td>Description</td>
  <td>Required</td>
</tr>
<tr>
  <td>schemaVersion</td>
  <td>Used for versioning of the manifest schema. Currently the only possible value is <code>1</code>.</td>
  <td>This is a required field.</td>
</tr>
<tr>
  <td>name</td>
  <td>The name of your extension.</td>
  <td>This field is just metadata and currently unused.</td>
</tr>
<tr>
  <td>description</td>
  <td>The description of your extension.</td>
  <td>This field is just metadata and currently unused.</td>
</tr>
<tr>
  <td>version</td>
  <td>The version of your extension.</td>
  <td>This field is just metadata and currently unused.</td>
</tr>
<tr>
  <td>phase</td>
  <td>The default execution phase of your extension.</td>
  <td>This is a required field.</td>
</tr>
<tr>
  <td>priority</td>
  <td>The default priority of your extension.</td>
  <td>This is a required field.</td>
</tr>
<tr>
  <td>module</td>
  <td>The relative path from the container filesystem's root to your WebAssembly module.</td>
  <td>This is a required field.</td>
</tr>
</tbody>
</table>

## ServiceMeshExtension reference {#ossm-wasm-ref-smextension_ossm-extensions}

The ServiceMeshExtension API provides a mechanism to extend the functionality provided by the Istio proxy through WebAssembly filters. There are two parts to writing a WebAssembly extension:

1. Write your extension using an SDK that exposes the proxy-wasm API and compile it to a WebAssembly module.
2. Package it into a container.

:::note

When creating new WebAssembly extensions, use the `WasmPlugin` API. The `ServiceMeshExtension` API, which was deprecated in Red Hat OpenShift Service Mesh version 2.2, was removed in Red Hat OpenShift Service Mesh version 2.3.

:::

**ServiceMeshExtension Field Reference**

<table>
<tbody>
<tr>
  <td>Field</td>
  <td>Description</td>
</tr>
<tr>
  <td>metadata.namespace</td>
  <td>The <code>metadata.namespace</code> field of a <code>ServiceMeshExtension</code> source has a special semantic: if it equals the Control Plane Namespace, the extension will be applied to all workloads in the Service Mesh that match its <code>workloadSelector</code> value. When deployed to any other Mesh Namespace, it will only be applied to workloads in that same Namespace.</td>
</tr>
<tr>
  <td>spec.workloadSelector</td>
  <td>The <code>spec.workloadSelector</code> field has the same semantic as the <code>spec.selector</code> field of the <a href="https://istio.io/v1.6/docs/reference/config/networking/gateway/#Gateway">Istio Gateway resource</a>. It will match a workload based on its Pod labels. If no <code>workloadSelector</code> value is specified, the extension will be applied to all workloads in the namespace.</td>
</tr>
<tr>
  <td>spec.config</td>
  <td>This is a structured field that will be handed over to the extension, with the semantics dependent on the extension you are deploying.</td>
</tr>
<tr>
  <td>spec.image</td>
  <td>A container image URI pointing to the image that holds the extension.</td>
</tr>
<tr>
  <td>spec.phase</td>
  <td>The phase determines where in the filter chain the extension is injected, in relation to existing Istio functionality like Authentication, Authorization and metrics generation. Valid values are: PreAuthN, PostAuthN, PreAuthZ, PostAuthZ, PreStats, PostStats. This field defaults to the value set in the <code>manifest.yaml</code> file of the extension, but can be overwritten by the user.</td>
</tr>
<tr>
  <td>spec.priority</td>
  <td>If multiple extensions with the same <code>spec.phase</code> value are applied to the same workload instance, the <code>spec.priority</code> value determines the ordering of execution. Extensions with higher priority will be executed first. This allows for inter-dependent extensions. This field defaults to the value set in the <code>manifest.yaml</code> file of the extension, but can be overwritten by the user.</td>
</tr>
</tbody>
</table>

### Deploying `ServiceMeshExtension` resources {#ossm-smextensions-deploy_ossm-extensions}

You can enable Red Hat OpenShift Service Mesh extensions using the `ServiceMeshExtension` resource. In this example, `istio-system` is the name of the Service Mesh control plane project.

:::note

When creating new WebAssembly extensions, use the `WasmPlugin` API. The `ServiceMeshExtension` API was deprecated in Red Hat OpenShift Service Mesh version 2.2 and removed in Red Hat OpenShift Service Mesh version 2.3.

:::

For a complete example that was built using the Rust SDK, take a look at the [header-append-filter](https://github.com/maistra/header-append-filter). It is a simple filter that appends one or more headers to the HTTP responses, with their names and values taken out from the `config` field of the extension. See a sample configuration in the snippet below.

**Procedure**

1. Create the following example resource:
   ```yaml title="Example ServiceMeshExtension resource extension.yaml"
   apiVersion: maistra.io/v1
   kind: ServiceMeshExtension
   metadata:
     name: header-append
     namespace: istio-system
   spec:
     workloadSelector:
       labels:
         app: httpbin
     config:
       first-header: some-value
       another-header: another-value
     image: quay.io/maistra-dev/header-append-filter:2.1
     phase: PostAuthZ
     priority: 100
   ```
2. Apply your `extension.yaml` file with the following command:
   ```terminal
   $ oc apply -f <extension>.yaml
   ```

## Migrating from `ServiceMeshExtension` to `WasmPlugin` resources {#ossm-extensions-migration-overview_ossm-extensions}

The `ServiceMeshExtension` API, which was deprecated in Red Hat OpenShift Service Mesh version 2.2, was removed in Red Hat OpenShift Service Mesh version 2.3. If you are using the `ServiceMeshExtension` API, you must migrate to the `WasmPlugin` API to continue using your WebAssembly extensions.

The APIs are very similar. The migration consists of two steps:

1. Renaming your plugin file and updating the module packaging.
2. Creating a `WasmPlugin` resource that references the updated container image.

### API changes {#ossm-extensions-migration-api-changes_ossm-extensions}

The new `WasmPlugin` API is similar to the `ServiceMeshExtension`, but with a few differences, especially in the field names:

**Field changes between **`ServiceMeshExtensions`** and **`WasmPlugin`

<table>
<tbody>
<tr>
  <td>ServiceMeshExtension</td>
  <td>WasmPlugin</td>
</tr>
<tr>
  <td><code>spec.config</code></td>
  <td><code>spec.pluginConfig</code></td>
</tr>
<tr>
  <td><code>spec.workloadSelector</code></td>
  <td><code>spec.selector</code></td>
</tr>
<tr>
  <td><code>spec.image</code></td>
  <td><code>spec.url</code></td>
</tr>
<tr>
  <td><code>spec.phase</code> valid values: PreAuthN, PostAuthN, PreAuthZ, PostAuthZ, PreStats, PostStats</td>
  <td><code>spec.phase</code> valid values: <empty>, AUTHN, AUTHZ, STATS</td>
</tr>
</tbody>
</table>

The following is an example of how a `ServiceMeshExtension` resource could be converted into a `WasmPlugin` resource.

```yaml title="ServiceMeshExtension resource"
apiVersion: maistra.io/v1
kind: ServiceMeshExtension
metadata:
  name: header-append
  namespace: istio-system
spec:
  workloadSelector:
    labels:
      app: httpbin
  config:
    first-header: some-value
    another-header: another-value
  image: quay.io/maistra-dev/header-append-filter:2.2
  phase: PostAuthZ
  priority: 100
```

```yaml title="New WasmPlugin resource equivalent to the ServiceMeshExtension above"
apiVersion: extensions.istio.io/v1alpha1
kind: WasmPlugin
metadata:
  name: header-append
  namespace: istio-system
spec:
  selector:
    matchLabels:
      app: httpbin
  url: oci://quay.io/maistra-dev/header-append-filter:2.2
  phase: STATS
  pluginConfig:
    first-header: some-value
    another-header: another-value
```

### Container image format changes {#ossm-extensions-migration-format-changes_ossm-extensions}

The new `WasmPlugin` container image format is similar to the `ServiceMeshExtensions`, with the following differences:

- The `ServiceMeshExtension` container format required a metadata file named `manifest.yaml` in the root directory of the container filesystem. The `WasmPlugin` container format does not require a `manifest.yaml` file.
- The `.wasm` file (the actual plugin) that previously could have any filename now must be named `plugin.wasm` and must be located in the root directory of the container filesystem.

### Migrating to `WasmPlugin` resources {#ossm-extensions-migrating-to-wasmplugin_ossm-extensions}

To upgrade your WebAssembly extensions from the `ServiceMeshExtension` API to the `WasmPlugin` API, you rename your plugin file.

**Prerequisites**

- `ServiceMeshControlPlane` is upgraded to version 2.2 or later.

**Procedure**

1. Update your container image. If the plugin is already in `/plugin.wasm` inside the container, skip to the next step.  If not:
   1. Ensure the plugin file is named `plugin.wasm`. You must name the extension file `plugin.wasm`.
   2. Ensure the plugin file is located in the root (/) directory. You must store extension files in the root of the container filesystem..
   3. Rebuild your container image and push it to a container registry.
2. Remove the `ServiceMeshExtension` resource and create a `WasmPlugin` resource that refers to the new container image you built.
