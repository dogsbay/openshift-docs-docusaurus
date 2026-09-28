---
title: Creating basic routes
sidebar_position: 1
---

# Creating basic routes

<a id="creating-basic-routes"></a>

If you have unencrypted HTTP, you can create a basic route with a route object.

## Create an HTTP-based route {#nw-creating-a-route_creating-basic-routes}

You can use the following procedure to create a simple HTTP-based route to a web application, using the `hello-openshift` application as an example.

You can create a route to host your application at a public URL. The route can either be secure or unsecured, depending on the network security configuration of your application. An HTTP-based route is an unsecured route that uses the basic HTTP routing protocol and exposes a service on an unsecured application port.

**Prerequisites**

- You installed the OpenShift CLI (`oc`).
- You are logged in as an administrator.
- You have a web application that exposes a port and a TCP endpoint listening for traffic on the port.

**Procedure**

1. Create a project called `hello-openshift` by running the following command:
   ```terminal
   $ oc new-project hello-openshift
   ```
2. Create a pod in the project by running the following command:
   ```terminal
   $ oc create -f https://raw.githubusercontent.com/openshift/origin/master/examples/hello-openshift/hello-pod.json
   ```
3. Create a service called `hello-openshift` by running the following command:
   ```terminal
   $ oc expose pod/hello-openshift
   ```
4. Create an unsecured route to the `hello-openshift` application by running the following command:
   ```terminal
   $ oc expose svc hello-openshift
   ```

**Verification**

- To verify that the `route` resource that you created, run the following command:
  ```terminal
  $ oc get routes -o yaml hello-openshift
  ```

  ```yaml title="Example YAML definition of the created unsecured route"
  apiVersion: route.openshift.io/v1
  kind: Route
  metadata:
    name: hello-openshift
  spec:
    host: www.example.com
    port:
      targetPort: 8080
    to:
      kind: Service
      name: hello-openshift
  ```

  where:

  <dl>
  <dt><code>host</code></dt>
  <dd>Specifies an alias DNS record that points to the service. This field can be any valid DNS name, such as <code>www.example.com</code>. The DNS name must follow DNS952 subdomain conventions. If not specified, a route name is automatically generated.</dd>
  <dt><code>targetPort</code></dt>
  <dd>Specifies the target port on pods that is selected by the service that this route points to.</dd>
  </dl>

  :::note

  To display your default ingress domain, run the following command:

  ```terminal
  $ oc get ingresses.config/cluster -o jsonpath={.spec.domain}
  ```

  :::

## Path-based routes {#nw-path-based-routes_creating-basic-routes}

To serve multiple applications by using a single hostname, configure path-based routes. This HTTP-based configuration directs traffic to specific services by comparing the URL path component, ensuring requests match the most specific route defined.

The following table shows example routes and their accessibility:

**Route availability**

<table>
<thead>
<tr>
  <th>Route</th>
  <th>When compared to</th>
  <th>Accessible</th>
</tr>
</thead>
<tbody>
<tr>
  <td rowspan="2"><em>www.example.com/test</em></td>
  <td><em>www.example.com/test</em></td>
  <td>Yes</td>
</tr>
<tr>
  <td><em>www.example.com</em></td>
  <td>No</td>
</tr>
<tr>
  <td rowspan="2"><em>www.example.com/test</em> and <em>www.example.com</em></td>
  <td><em>www.example.com/test</em></td>
  <td>Yes</td>
</tr>
<tr>
  <td><em>www.example.com</em></td>
  <td>Yes</td>
</tr>
<tr>
  <td rowspan="2"><em>www.example.com</em></td>
  <td><em>www.example.com/text</em></td>
  <td>Yes (Matched by the host, not the route)</td>
</tr>
<tr>
  <td><em>www.example.com</em></td>
  <td>Yes</td>
</tr>
</tbody>
</table>

```yaml title="Example of an unsecured route with a path"
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: route-unsecured
spec:
  host: www.example.com
  path: "/test"
  to:
    kind: Service
    name: service-name
```

- `spec.host`: Specifies the path attribute for a path-based route.

:::note

Path-based routing is not available when using passthrough TLS, as the router does not terminate TLS in that case and cannot read the contents of the request.

:::

## Creating a route for Ingress Controller sharding {#nw-ingress-sharding-route-configuration_creating-basic-routes}

You can use a route to host your application at a URL. Ingress Controller sharding helps balance incoming traffic load among a set of Ingress Controllers. Ingress Controller sharding can also isolate traffic to a specific Ingress Controller. For example, company A goes to one Ingress Controller and company B to another.

The following procedure describes how to create a route for Ingress Controller sharding, using the `hello-openshift` application as an example.

**Prerequisites**

- You installed the OpenShift CLI (`oc`).
- You are logged in as a project administrator.
- You have a web application that exposes a port and an HTTP or TLS endpoint listening for traffic on the port.
- You have configured the Ingress Controller for sharding.

**Procedure**

1. Create a project called `hello-openshift` by running the following command:
   ```terminal
   $ oc new-project hello-openshift
   ```
2. Create a pod in the project by running the following command:
   ```terminal
   $ oc create -f https://raw.githubusercontent.com/openshift/origin/master/examples/hello-openshift/hello-pod.json
   ```
3. Create a service called `hello-openshift` by running the following command:
   ```terminal
   $ oc expose pod/hello-openshift
   ```
4. Create a route definition called `hello-openshift-route.yaml`:
   ```yaml title="YAML definition of the created route for sharding"
   apiVersion: route.openshift.io/v1
   kind: Route
   metadata:
     labels:
       type: sharded
     name: hello-openshift-edge
     namespace: hello-openshift
   spec:
     subdomain: hello-openshift
     tls:
       termination: edge
     to:
       kind: Service
       name: hello-openshift
   ```

   where:

   <dl>
   <dt><code>type</code></dt>
   <dd>Specifies both the label key and its corresponding label value must match the ones specified in the Ingress Controller. In this example, the Ingress Controller has the label key and value <code>type: sharded</code>.</dd>
   <dt><code>subdomain</code></dt>
   <dd>Specifies the route gets exposed by using the value of the <code>subdomain</code> field. When you specify the <code>subdomain</code> field, you must leave the hostname unset. If you specify both the <code>host</code> and <code>subdomain</code> fields, then the route uses the value of the <code>host</code> field, and ignore the <code>subdomain</code> field.</dd>
   </dl>
5. Use `hello-openshift-route.yaml` to create a route to the `hello-openshift` application by running the following command:
   ```terminal
   $ oc -n hello-openshift create -f hello-openshift-route.yaml
   ```

**Verification**

- Get the status of the route with the following command:
  ```terminal
  $ oc -n hello-openshift get routes/hello-openshift-edge -o yaml
  ```

  The resulting `Route` resource should look similar to the following:

  ```yaml title="Example output"
  apiVersion: route.openshift.io/v1
  kind: Route
  metadata:
    labels:
      type: sharded
    name: hello-openshift-edge
    namespace: hello-openshift
  spec:
    subdomain: hello-openshift
    tls:
      termination: edge
    to:
      kind: Service
      name: hello-openshift
  status:
    ingress:
    - host: hello-openshift.<apps-sharded.basedomain.example.net>
      routerCanonicalHostname: router-sharded.<apps-sharded.basedomain.example.net>
      routerName: sharded
  ```

  where:

  <dl>
  <dt><code>host</code></dt>
  <dd>Specifies the hostname the Ingress Controller, or router, uses to expose the route. The value of the <code>host</code> field is automatically determined by the Ingress Controller, and uses its domain. In this example, the domain of the Ingress Controller is <code>&lt;apps-sharded.basedomain.example.net&gt;</code>.</dd>
  <dt><code>&lt;apps-sharded.basedomain.example.net&gt;</code></dt>
  <dd>Specifies the hostname of the Ingress Controller. If the hostname is not set, the route can use a subdomain instead. When you specify a subdomain, you automatically use the domain of the Ingress Controller that exposes the route. When a route is exposed by multiple Ingress Controllers, the route is hosted at multiple URLs.</dd>
  <dt><code>routerName</code></dt>
  <dd>Specifies the name of the Ingress Controller. In this example, the Ingress Controller has the name <code>sharded</code>.</dd>
  </dl>

## Create a route through an Ingress object {#nw-ingress-creating-a-route-via-an-ingress_creating-basic-routes}

To integrate ecosystem components that require Ingress resources, configure an Ingress object. OpenShift Container Platform automatically manages the lifecycle of the corresponding route objects, creating and deleting them to ensure seamless connectivity.

**Prerequisites**

- If clients must receive a full certificate chain, you must combine the PEM-encoded leaf certificate and intermediates into a single file. Place the leaf certificate first, followed by each issuer in chain order.
- You confirmed the private key matches the leaf certificate in the `tls.crt` key.
- You confirmed the `tls.key` key has only the private key for the leaf certificate.
- The certificate Subject Alternative Name (SAN), or the subject CN if no SAN is present, covers every hostname set in `spec.rules[].host` and `spec.tls[].hosts`. These values must match for the same host.
- The private key is not password-encrypted. You must decrypt the key before you create the TLS secret so that OpenShift Container Platform can read the key material.
- You created a `Secret` of type `kubernetes.io/tls` in the same namespace as the `Ingress`. The `secretName` must match the `spec.tls[].secretName` field. If you have not created the secret, you must do so before you apply the `Ingress` object.

**Procedure**

1. Define an Ingress object in the OpenShift Container Platform console or by entering the `oc create` command:
   ```yaml title="YAML Definition of an Ingress"
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: frontend
     annotations:
       route.openshift.io/termination: "reencrypt"
       route.openshift.io/destination-ca-certificate-secret: secret-ca-cert
   spec:
     rules:
     - host: www.example.com
       http:
         paths:
         - backend:
             service:
               name: frontend
               port:
                 number: 443
           path: /
           pathType: Prefix
     tls:
     - hosts:
       - www.example.com
       secretName: example-com-tls-certificate
   # ...
   ```

   where:

   <dl>
   <dt><code>route.openshift.io/termination</code></dt>
   <dd>Specifies the <code>route.openshift.io/termination</code> annotation. You can configure the <code>spec.tls.termination</code> parameter of the <code>Route</code> because <code>Ingress</code> does not have this parameter. The accepted values are <code>edge</code>, <code>passthrough</code>, and <code>reencrypt</code>. All other values are silently ignored. When  the annotation value is unset, <code>edge</code> is the default route. The TLS certificate details must be defined in the template file to implement the default edge route.</dd>
   <dt><code>rules.host</code></dt>
   <dd>Specifies an explicit hostname for the <code>Ingress</code> object. Mandatory parameter. You can use the <code>&lt;host_name&gt;.&lt;cluster_ingress_domain&gt;</code> syntax, for example <code>apps.openshiftdemos.com</code>, to take advantage of the <code>*.&lt;cluster_ingress_domain&gt;</code> wildcard DNS record and serving certificate for the cluster. Otherwise, you must ensure that there is a DNS record for the chosen hostname.</dd>
   <dt><code>destination-ca-certificate-secret</code></dt>
   <dd>Specifies the <code>route.openshift.io/destination-ca-certificate-secret</code> annotation. The annotation can be used on an Ingress object to define a route with a custom destination certificate (CA). The annotation references a kubernetes secret, <code>secret-ca-cert</code> that will be inserted into the generated route.</dd>
   </dl>
2. If you specify the `passthrough` value in the `route.openshift.io/termination` annotation, set `path` to `''` and `pathType` to `ImplementationSpecific` in the spec:
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   # ...
     spec:
       rules:
       - host: www.example.com
         http:
           paths:
           - path: ''
             pathType: ImplementationSpecific
             backend:
               service:
                 name: frontend
                 port:
                   number: 443
   # ...
   ```

   ```terminal
   $ oc apply -f ingress.yaml
   ```
3. To specify a route object with a destination CA from an ingress object, you must create a `kubernetes.io/tls` or `Opaque` type secret with a certificate in PEM-encoded format in the `data.tls.crt` specifier of the secret.
4. List your routes:
   ```terminal
   $ oc get routes
   ```

   The result includes an autogenerated route whose name starts with `frontend-`:

   ```terminal
   NAME             HOST/PORT         PATH    SERVICES    PORT    TERMINATION          WILDCARD
   frontend-gnztq   www.example.com           frontend    443     reencrypt/Redirect   None
   ```

   ```yaml title="YAML definition example of an autogenerated route"
   apiVersion: route.openshift.io/v1
   kind: Route
   metadata:
     name: frontend-gnztq
     ownerReferences:
     - apiVersion: networking.k8s.io/v1
       controller: true
       kind: Ingress
       name: frontend
       uid: 4e6c59cc-704d-4f44-b390-617d879033b6
   spec:
     host: www.example.com
     path: /
     port:
       targetPort: https
     tls:
       certificate: |
         -----BEGIN CERTIFICATE-----
         [...]
         -----END CERTIFICATE-----
       insecureEdgeTerminationPolicy: Redirect
       key: |
         -----BEGIN RSA PRIVATE KEY-----
         [...]
         -----END RSA PRIVATE KEY-----
       termination: reencrypt
       destinationCACertificate: |
         -----BEGIN CERTIFICATE-----
         [...]
         -----END CERTIFICATE-----
     to:
       kind: Service
       name: frontend
   ```

## About label propagation from Ingress to Route resources {#networking-ingress-label-propagation-about_creating-basic-routes}

You can opt-in to a feature that enables the Ingress Operator to automatically propagate labels. This allows you to add metadata that helps track or manage resources, or to control specific behaviors that depend on labels.

By default, the managed `Route` object does not inherit labels from the `Ingress` resource. When you enable the propagation feature, the Operator actively reconciles the labels on the generated `Route` resource to match the labels on the parent `Ingress` resource.

:::note

When label propagation is enabled, the Ingress Operator replaces all labels on the managed `Route` resource with the exact set of labels from the parent `Ingress` resource. Any labels that were manually added to the `Route` resource are removed.

:::

The propagation behavior is controlled by the `route.openshift.io/reconcile-labels` annotation on the `Ingress` resource. The Operator’s behavior changes depending on the state of this annotation:

- Annotation not present (default): The Operator does not sync labels from the `Ingress` resource to the `Route` resource. Any existing labels on the `Route` are preserved.
- Annotation enabled (`route.openshift.io/reconcile-labels: "true"`): The Operator enables label propagation. On the next reconciliation (triggered by the `Ingress` create or update event), the Operator replaces all labels on the generated `Route` resource with the labels from the `Ingress` resource.
- Annotation disabled (removed or value set to non-"true"): The Operator disables label propagation. The labels that currently exist on the `Route` resource are kept as-is, but the Operator no longer syncs them with the `Ingress` resource.
- Annotation re-enabled: The Operator resumes propagation. It will again replace all labels on the `Route` resource with the current labels from the `Ingress` resource.

## Enable label propagation from Ingress to Route resources {#networking-ingress-label-propagation-enabling_creating-basic-routes}

You can enable the Ingress Operator to automatically propagate labels from an `Ingress` resource to the `Route` resource it manages. To enable this, you must add the `reconcile-labels` annotation to an `Ingress` resource.

**Prerequisites**

- You have access to an OpenShift Container Platform cluster.
- You have the `cluster-admin` role or permissions to create and edit `Ingress` resources in a project.

**Procedure**

1. Create or edit an `Ingress` resource manifest.
2. In the `metadata.annotations` section, add `route.openshift.io/reconcile-labels: "true"`.
3. In the `metadata.labels` section, add the labels you want to propagate.
   Example `Ingress` resource with label propagation enabled:

   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: example-ingress
     annotations:
       route.openshift.io/reconcile-labels: "true"
     labels:
       app: my-app
       owner: dev-team
   spec:
     ingressClassName: openshift-default
     rules:
     - host: example.com
       http:
         paths:
         - backend:
             service:
               name: example-service
               port:
                 number: 27017
           path: "/"
           pathType: "Prefix"
   ```
4. Apply the manifest to your cluster:
   ```terminal
   $ oc apply -f <example-ingress-manifest.yaml>
   ```

   Replace `<example-ingress-manifest.yaml>` with the name of your specific manifest file.
5. Verify that the labels from the `Ingress` resource have propagated to the generated `Route` resource:
   ```terminal
   $ oc get route -l app=my-app --show-labels
   ```

   Example output:

   ```terminal
   NAME          HOST/PORT     PATH   SERVICES          PORT    TERMINATION   WILDCARD   LABELS
   example-rt    example.com   /      example-service   8080                  None       app=my-app,owner=dev-team
   ```
