---
title: Deploy your plugin on a cluster
sidebar_position: 3
---

# Deploy your plugin on a cluster {#deploy-plugin-cluster_{context}}

<a id="deploy-plugin-cluster_{context}"></a>

You can deploy the plugin to an OpenShift Container Platform cluster.

## Build an image with Docker {#build-image-with-docker_deploy-plugin-cluster}

To deploy your plugin on a cluster, you need to build an image and push it to an image registry first.

**Procedure**

1. Build the image with the following command:
   ```terminal
   $ docker build -t quay.io/my-repositroy/my-plugin:latest .
   ```
2. Optional: If you want to test your image, run the following command:
   ```terminal
   $ docker run -it --rm -d -p 9001:80 quay.io/my-repository/my-plugin:latest
   ```
3. Push the image by running the following command:
   ```terminal
   $ docker push quay.io/my-repository/my-plugin:latest
   ```

## Deploy your plugin on a cluster {#deploy-on-cluster_deploy-plugin-cluster}

After pushing an image with your changes to a registry, you can deploy the plugin to a cluster by using a Helm chart.

**Prerequisites**

- You must have the location of the image containing the plugin that was previously pushed.
  :::note

  You can specify additional parameters based on the needs of your plugin. The [`values.yaml`](https://github.com/openshift/console-plugin-template/blob/main/charts/openshift-console-plugin/values.yaml) file provides a full set of supported parameters.

  :::

**Procedure**

1. To deploy your plugin to a cluster, install a Helm chart with the name of the plugin as the Helm release name into a new namespace or an existing namespace as specified by the `-n` command-line option. Provide the location of the image within the `plugin.image` parameter by using the following command:
   ```terminal
   $ helm upgrade -i  my-plugin charts/openshift-console-plugin -n my-plugin-namespace --create-namespace --set plugin.image=my-plugin-image-location
   ```

   Where:

   <dl>
   <dt><code>n &lt;my-plugin-namespace&gt;</code></dt>
   <dd>Specifies an existing namespace to deploy your plugin into.</dd>
   <dt><code>--create-namespace</code></dt>
   <dd>Optional: If deploying to a new namespace, use this parameter.</dd>
   <dt><code>--set plugin.image=my-plugin-image-location</code></dt>
   <dd>Specifies the location of the image within the <code>plugin.image</code> parameter.</dd>
   </dl>

   :::note

   If you are deploying on OpenShift Container Platform 4.10 and later, it is recommended to exclude configurations related to pod security by adding the parameter `--set plugin.securityContext.enabled=false`.

   :::
2. Optional: You can specify any additional parameters by using the set of supported parameters in the `charts/openshift-console-plugin/values.yaml` file.
   ```yaml
   plugin:
     name: ""
     description: ""
     image: ""
     imagePullPolicy: IfNotPresent
     replicas: 2
     port: 9443
     securityContext:
       enabled: true
     podSecurityContext:
       enabled: true
       runAsNonRoot: true
       seccompProfile:
         type: RuntimeDefault
     containerSecurityContext:
       enabled: true
       allowPrivilegeEscalation: false
       capabilities:
         drop:
           - ALL
     resources:
       requests:
         cpu: 10m
         memory: 50Mi
     basePath: /
     certificateSecretName: ""
     serviceAccount:
       create: true
       annotations: {}
       name: ""
     patcherServiceAccount:
       create: true
       annotations: {}
       name: ""
     jobs:
       patchConsoles:
         enabled: true
         image: "registry.redhat.io/openshift4/ose-tools-rhel8@sha256:e44074f21e0cca6464e50cb6ff934747e0bd11162ea01d522433a1a1ae116103"
         podSecurityContext:
           enabled: true
           runAsNonRoot: true
           seccompProfile:
             type: RuntimeDefault
         containerSecurityContext:
           enabled: true
           allowPrivilegeEscalation: false
           capabilities:
             drop:
               - ALL
         resources:
           requests:
             cpu: 10m
             memory: 50Mi
   ```

**Verification**

- View the list of enabled plugins by navigating from **Administration** → **Cluster Settings** → **Configuration** → **Console** `operator.openshift.io` → **Console plugins** or by visiting the **Overview** page.

:::note

It can take a few minutes for the new plugin configuration to appear. If you do not see your plugin, you might need to refresh your browser if the plugin was recently enabled. If you receive any errors at runtime, check the JS console in browser developer tools to look for any errors in your plugin code.

:::

## Plugin service proxy {#dynamic-plugin-proxy-service_deploy-plugin-cluster}

If you need to make HTTP requests to an in-cluster service from your plugin, you can declare a service proxy in its `ConsolePlugin` resource by using the `spec.proxy` array field. The console backend exposes the `/api/proxy/plugin/<plugin-name>/<proxy-alias>/<request-path>++?++<optional-query-parameters>` endpoint to proxy the communication between the plugin and the service. A proxied request uses a *service CA bundle* by default. The service must use HTTPS.

:::note

The plugin must use the `consolefetch` API to make requests from its JavaScript code or some requests might fail. For more information, see "Dynamic plugin API".

:::

For each entry, you must specify an endpoint and alias of the proxy under the `endpoint` and `alias` fields. For the Service proxy type, you must set the endpoint `type` field to `Service` and the `service` must include values for the `name`, `namespace`, and `port` fields. For example, `/api/proxy/plugin/helm/helm-charts/releases++?++limit++=++10` is a proxy request path from the `helm` plugin with a `helm-charts` service that lists ten helm releases.

```YAML title="Example service proxy"
apiVersion: console.openshift.io/v1
kind: ConsolePlugin
metadata:
  name:<plugin-name>
spec:
  proxy:
  - alias: helm-charts
    authorization: UserToken
    caCertificate: +'-----BEGIN CERTIFICATE-----\nMIID....'en+
    endpoint:
      service:
        name: <service-name>
        namespace: <service-namespace>
        port: <service-port>
      type: Service
```

where:

`spec.proxy.alias.helm-charts`
: Alias of the proxy.

`spec.proxy.authorization.UserToken`
: If the service proxy request must contain the logged-in user’s OpenShift Container Platform access token, you must set the authorization field to `UserToken`.

  :::note

  If the service proxy request does not contain the logged-in user’s OpenShift Container Platform access token, set the authorization field to `None`.

  :::

`spec.proxy.caCertificate.+'-----BEGIN CERTIFICATE-----\nMIID....'en+`
: If the service uses a custom service CA, the `caCertificate` field must contain the certificate bundle.

`spec.proxy.endpoint`
: Endpoint of the proxy.

## Enable a dynamic plugin with the CLI {#enabling-a-dynamic-plugin-by-using-the-cli_deploy-plugin-cluster}

You can enable a dynamic plugin to extend the core web console with more features, such as additional pages, perspectives, or dashboard items. Use the OpenShift CLI (`oc`) after a scripted installation, such as an Operator or Helm-based install. Add the `ConsolePlugin` name to `spec.plugins` in the console Operator configuration (`console.operator.openshift.io/cluster`) so the web console loads it.

**Prerequisites**

- You logged in to the cluster as a user with `cluster-admin` privileges.
- You installed the dynamic plugin using a scripted installation, such as an Operator or Helm chart.
- A `ConsolePlugin` custom resource (CR) exists on the cluster.

**Procedure**

1. Confirm the name of the `ConsolePlugin` resource by running the following command:
   ```terminal
   $ oc get consoleplugin
   ```
2. Optional: View details for a specific `ConsolePlugin` resource by running the following commands:
   1. Set the plugin name as an environment variable:
      ```terminal
      $ PLUGIN_NAME="<plugin_name>"
      ```

      where `<plugin_name>` is the name of the `ConsolePlugin` resource.
   2. Verify the plugin details:
      ```terminal
      $ oc get consoleplugin "${PLUGIN_NAME}" -o yaml
      ```

      The following example shows a `ConsolePlugin` YAML with the plugin listed in `spec.plugins`:

      ```yaml
      apiVersion: operator.openshift.io/v1
      kind: Console
      metadata:
        name: cluster
      spec:
        plugins:
          - <plugin_name>
          # ...
      ```

      Replace `<plugin_name>` with the name of your plugin.
3. Enable the dynamic plugin by adding the `ConsolePlugin` name to the console Operator configuration.
   :::note

   Make sure the Operator finishes installing the dynamic plugin before you run the following patch command.

   :::

   1. Set the plugin patch as an environment variable:
      ```terminal
      $ PLUGIN_PATCH=$(cat <<EOF
      [
        {
          "op": "add",
          "path": "/spec/plugins/-",
          "value": "<plugin_name>"
        }
      ]
      EOF
      )
      ```
   2. Patch the console Operator configuration:
      ```terminal
      $ oc patch consoles.operator.openshift.io cluster --type=json -p "${PLUGIN_PATCH}"
      ```

**Verification**

1. Confirm that the console Operator configuration includes the `ConsolePlugin` name by running the following command:
   ```terminal
   $ oc get console.operator.openshift.io cluster -o jsonpath='{.spec.plugins}{"\n"}'
   ```
2. Refresh the OpenShift Container Platform web console.
   The console can take a few minutes to apply the updated configuration.

**Additional resources**

- [Upstream demo instructions for enabling a plugin](https://github.com/openshift/console/tree/main/dynamic-demo-plugin#enabling-the-plugin)
- [Helm template that patches the console Operator configuration](https://github.com/openshift/console-plugin-template/blob/3a06152cffdd10ad9654b6b607cb00a055f29733/charts/openshift-console-plugin/templates/patch-consoles-job.yaml)

**Additional resources**

- [Service CA certificates](/docs/security/certificate_types_descriptions/service-ca-certificates#service-ca-certificates)
- [Securing service traffic using service serving certificate secrets](/docs/security/certificates/service-serving-certificate#service-serving-certificate)
- [Dynamic plugin API](/docs/web_console/dynamic-plugin/dynamic-plugins-reference#dynamic-plugin-api_dynamic-plugins-reference)

## Disable your plugin in the browser {#disabling-your-plugin-browser_deploy-plugin-cluster}

Console users can use the `disable-plugins` query parameter to disable specific or all dynamic plugins that would normally get loaded at runtime.

**Procedure**

- To disable a specific plugin(s), remove the plugin you want to disable from the comma-separated list of plugin names.
- To disable all plugins, leave an empty string in the `disable-plugins` query parameter.
  :::note

  Cluster administrators can disable plugins in the **Cluster Settings** page of the web console.

  :::

**Additional resources**

- [Understanding Helm](/docs/applications/working_with_helm_charts/understanding-helm#understaning-helm)
