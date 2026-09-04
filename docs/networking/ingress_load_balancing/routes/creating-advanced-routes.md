---
title: Securing routes through ingress objects
sidebar_position: 4
---

# Securing routes through ingress objects {#creating-advanced-routes}

<a id="creating-advanced-routes"></a>

You can secure your application traffic by managing certificates directly through ingress objects. This includes creating routes using the destination CA certificate in an ingress annotation or using the default certificate.

:::warning

The Ingress Controller maintains a one-way sync for certificates managed through ingress objects. Do not manually apply changes directly to the generated route’s TLS configuration. Any manual modifications are silently overwritten the next time the parent ingress object is updated or reconciled. This is particularly important to note if you operate a GitOps-managed cluster.

:::

## Create a route using the destination CA certificate in the Ingress annotation {#nw-ingress-re-encrypt-route-custom-cert_creating-advanced-routes}

To define a route with a custom destination CA certificate, apply the `route.openshift.io/destination-ca-certificate-secret` annotation to an Ingress object. This configuration ensures the Ingress Controller uses the specified secret to verify the identity of the destination service.

**Prerequisites**

- You have a certificate/key pair in PEM-encoded files, where the certificate is valid for the route host.
- You have a separate CA certificate in a PEM-encoded file that completes the certificate chain.
- You have a separate destination CA certificate in a PEM-encoded file.
- You have a service that you want to expose.

**Procedure**

1. Create a secret for the destination CA certificate by entering the following command:
   ```terminal
   $ oc create secret generic dest-ca-cert --from-file=tls.crt=<file_path>
   ```

   For example:

   ```terminal
   $ oc -n test-ns create secret generic dest-ca-cert --from-file=tls.crt=tls.crt
   ```

   ```terminal title="Example output"
   secret/dest-ca-cert created
   ```
2. Add the `route.openshift.io/destination-ca-certificate-secret` to the Ingress annotations:
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: frontend
     annotations:
       route.openshift.io/termination: "reencrypt"
       route.openshift.io/destination-ca-certificate-secret: secret-ca-cert
   ...
   ```

   where:

   <dl>
   <dt><code>destination-ca-certificate-secret</code></dt>
   <dd>Specifies the <code>route.openshift.io/destination-ca-certificate-secret</code> annotation. The annotation references a Kubernetes secret. The Ingress Controller inserts a secret that is referenced in the annotation into the generated route.</dd>
   </dl>

   ```yaml title="Example output"
   apiVersion: route.openshift.io/v1
   kind: Route
   metadata:
     name: frontend
     annotations:
       route.openshift.io/termination: reencrypt
       route.openshift.io/destination-ca-certificate-secret: secret-ca-cert
   spec:
   ...
     tls:
       insecureEdgeTerminationPolicy: Redirect
       termination: reencrypt
       destinationCACertificate: |
         -----BEGIN CERTIFICATE-----
         [...]
         -----END CERTIFICATE-----
   ...
   ```

## Create a route using the default certificate through an Ingress object {#nw-ingress-edge-route-default-certificate_creating-advanced-routes}

To generate a secure, edge-terminated route that uses the default ingress certificate, specify an empty TLS configuration in the Ingress object. This configuration overrides the default behavior, preventing the creation of an insecure route.

**Prerequisites**

- You have a service that you want to expose.
- You have access to the OpenShift CLI (`oc`).

**Procedure**

1. Create a YAML file for the Ingress object. In the following example, the file is called `example-ingress.yaml`:
   ```yaml title="YAML definition of an Ingress object"
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: frontend
     ...
   spec:
     rules:
       ...
     tls:
     - {}
   ```

   where:

   <dl>
   <dt><code>spec.tls</code></dt>
   <dd>Specifies the TLS configuration. Use the exact syntax shown to specify TLS without specifying a custom certificate.</dd>
   </dl>
2. Create the Ingress object by running the following command:
   ```terminal
   $ oc create -f example-ingress.yaml
   ```

**Verification**

- Verify that OpenShift Container Platform has created the expected route for the Ingress object by running the following command:
  ```terminal
  $ oc get routes -o yaml
  ```

  ```yaml title="Example output"
  apiVersion: v1
  items:
  - apiVersion: route.openshift.io/v1
    kind: Route
    metadata:
      name: frontend-j9sdd
  # ...
    spec:
    ...
      tls:
        insecureEdgeTerminationPolicy: Redirect
        termination: edge
  # ...
  ```

  where:

  <dl>
  <dt><code>metadata.name</code></dt>
  <dd>Specifies the name of the route, which includes the name of the Ingress object followed by a random suffix.</dd>
  <dt><code>spec.tls</code></dt>
  <dd>To use the default certificate, the route should not specify <code>spec.certificate</code>.</dd>
  <dt><code>tls.termination</code></dt>
  <dd>Specifies the termination policy for the route. The route should specify the <code>edge</code> termination policy.</dd>
  </dl>
