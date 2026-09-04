---
title: Kiali configuration reference
sidebar_position: 27
---

# Kiali configuration reference {#kiali-config-ref}

<a id="kiali-config-ref"></a>

When the Service Mesh Operator creates the `ServiceMeshControlPlane` it also processes the Kiali resource. The Kiali Operator then uses this object when creating Kiali instances.

## Specifying Kiali configuration in the SMCP {#ossm-smcp-kiali_kiali-config-ref}

You can configure Kiali under the `addons` section of the `ServiceMeshControlPlane` resource. Kiali is enabled by default. To disable Kiali, set `spec.addons.kiali.enabled` to `false`.

You can specify your Kiali configuration in either of two ways:

- Specify the Kiali configuration in the `ServiceMeshControlPlane` resource under `spec.addons.kiali.install`. This approach has some limitations, because the complete list of Kiali configurations is not available in the SMCP.
- Configure and deploy a Kiali instance and specify the name of the Kiali resource as the value for `spec.addons.kiali.name` in the `ServiceMeshControlPlane` resource. You must create the CR in the same namespace as the Service Mesh control plane, for example, `istio-system`. If a Kiali resource matching the value of `name` exists, the control plane will configure that Kiali resource for use with the control plane. This approach lets you fully customize your Kiali configuration in the Kiali resource. Note that with this approach, various fields in the Kiali resource are overwritten by the Service Mesh Operator, specifically, the `accessible_namespaces` list, as well as the endpoints for Grafana, Prometheus, and tracing.

```yaml title="Example SMCP parameters for Kiali"
apiVersion: maistra.io/v2
kind: ServiceMeshControlPlane
metadata:
  name: basic
spec:
  addons:
    kiali:
      name: kiali
      enabled: true
      install:
        dashboard:
          viewOnly: false
          enableGrafana: true
          enableTracing: true
          enablePrometheus: true
        service:
          ingress:
            contextPath: /kiali
```

`ServiceMeshControlPlane`** Kiali parameters**

<table>
<tbody>
<tr>
  <td>Parameter</td>
  <td>Description</td>
  <td>Values</td>
  <td>Default value</td>
</tr>
<tr>
  <td>spec: addons: kiali: name:</td>
  <td>Name of Kiali custom resource. If a Kiali CR matching the value of <code>name</code> exists, the {SMProductShortname} Operator will use that CR for the installation. If no Kiali CR exists, the Operator will create one using this <code>name</code> and the configuration options specified in the SMCP.</td>
  <td>string</td>
  <td><code>kiali</code></td>
</tr>
<tr>
  <td>kiali: enabled:</td>
  <td>This parameter enables or disables Kiali. Kiali is enabled by default.</td>
  <td><code>true</code>/<code>false</code></td>
  <td><code>true</code></td>
</tr>
<tr>
  <td>kiali: install:</td>
  <td>Install a Kiali resource if the named Kiali resource is not present. The <code>install</code> section is ignored if <code>addons.kiali.enabled</code> is set to <code>false</code>.</td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td>kiali: install: dashboard:</td>
  <td>Configuration parameters for the dashboards shipped with Kiali.</td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td>kiali: install: dashboard: viewOnly:</td>
  <td>This parameter enables or disables view-only mode for the Kiali console. When view-only mode is enabled, users cannot use the Kiali console to make changes to the {SMProductShortname}.</td>
  <td><code>true</code>/<code>false</code></td>
  <td><code>false</code></td>
</tr>
<tr>
  <td>kiali: install: dashboard: enableGrafana:</td>
  <td>Grafana endpoint configured based on <code>spec.addons.grafana</code> configuration.</td>
  <td><code>true</code>/<code>false</code></td>
  <td><code>true</code></td>
</tr>
<tr>
  <td>kiali: install: dashboard: enablePrometheus:</td>
  <td>Prometheus endpoint configured based on <code>spec.addons.prometheus</code> configuration.</td>
  <td><code>true</code>/<code>false</code></td>
  <td><code>true</code></td>
</tr>
<tr>
  <td>kiali: install: dashboard: enableTracing:</td>
  <td>Tracing endpoint configured based on Jaeger custom resource configuration.</td>
  <td><code>true</code>/<code>false</code></td>
  <td><code>true</code></td>
</tr>
<tr>
  <td>kiali: install: service:</td>
  <td>Configuration parameters for the Kubernetes service associated with the Kiali installation.</td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td>kiali: install: service: metadata:</td>
  <td>Use to specify additional metadata to apply to resources.</td>
  <td>N/A</td>
  <td>N/A</td>
</tr>
<tr>
  <td>kiali: install: service: metadata: annotations:</td>
  <td>Use to specify additional annotations to apply to the component's service.</td>
  <td>string</td>
  <td>N/A</td>
</tr>
<tr>
  <td>kiali: install: service: metadata: labels:</td>
  <td>Use to specify additional labels to apply to the component's service.</td>
  <td>string</td>
  <td>N/A</td>
</tr>
<tr>
  <td>kiali: install: service: ingress:</td>
  <td>Use to specify details for accessing the component’s service through an OpenShift Route.</td>
  <td>N/A</td>
  <td>N/A</td>
</tr>
<tr>
  <td>kiali: install: service: ingress: metadata: annotations:</td>
  <td>Use to specify additional annotations to apply to the component's service ingress.</td>
  <td>string</td>
  <td>N/A</td>
</tr>
<tr>
  <td>kiali: install: service: ingress: metadata: labels:</td>
  <td>Use to specify additional labels to apply to the component's service ingress.</td>
  <td>string</td>
  <td>N/A</td>
</tr>
<tr>
  <td>kiali: install: service: ingress: enabled:</td>
  <td>Use to customize an OpenShift Route for the service associated with a component.</td>
  <td><code>true</code>/<code>false</code></td>
  <td><code>true</code></td>
</tr>
<tr>
  <td>kiali: install: service: ingress: contextPath:</td>
  <td>Use to specify the context path to the service.</td>
  <td>string</td>
  <td>N/A</td>
</tr>
<tr>
  <td>install: service: ingress: hosts:</td>
  <td>Use to specify a single hostname per OpenShift route. An empty hostname implies a default hostname for the Route.</td>
  <td>string</td>
  <td>N/A</td>
</tr>
<tr>
  <td>install: service: ingress: tls:</td>
  <td>Use to configure the TLS for the OpenShift route.</td>
  <td></td>
  <td>N/A</td>
</tr>
<tr>
  <td>kiali: install: service: nodePort:</td>
  <td>Use to specify the <code>nodePort</code> for the component's service <code>Values.&lt;component&gt;.service.nodePort.port</code></td>
  <td>integer</td>
  <td>N/A</td>
</tr>
</tbody>
</table>

## Specifying Kiali configuration in a Kiali custom resource {#ossm-specifying-external-kiali_kiali-config-ref}

You can fully customize your Kiali deployment by configuring Kiali in the Kiali custom resource (CR) rather than in the `ServiceMeshControlPlane` (SMCP) resource. This configuration is sometimes called an "external Kiali" since the configuration is specified outside of the SMCP.

:::note

You must deploy the `ServiceMeshControlPlane` and Kiali custom resources in the same namespace. For example, `istio-system`.

:::

You can configure and deploy a Kiali instance and then specify the `name` of the Kiali resource as the value for `spec.addons.kiali.name` in the SMCP resource. If a Kiali CR matching the value of `name` exists, the Service Mesh control plane will use the existing installation. This approach lets you fully customize your Kiali configuration.
