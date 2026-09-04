---
title: Securing routes with the cert-manager Operator for Red Hat OpenShift
sidebar_position: 9
---

# Securing routes with the cert-manager Operator for Red Hat OpenShift {#cert-manager-securing-routes}

<a id="cert-manager-securing-routes"></a>

In the OpenShift Container Platform, the route API is extended to provide a configurable option to reference TLS certificates via secrets. With externally managed certificates enabled, you can minimize errors from manual intervention, streamline the certificate management process, and enable the OpenShift Container Platform router to promptly serve the referenced certificate.

## Configuring certificates to secure routes in your cluster {#cert-manager-configuring-routes_cert-manager-securing-routes}

To encrypt traffic between external clients and your applications, configure certificates for routes in your OpenShift Container Platform cluster. You can secure your routes by defining TLS termination types, such as edge, passthrough, or re-encrypt, to match your specific security policies.

**Prerequisites**

- You have installed version 1.14.0 or later of the cert-manager Operator for Red Hat OpenShift.
- You have `create` permission on the `routes/custom-host` sub-resource, which is used for both creating and updating routes.
- You have a `Service` resource that you want to expose.

**Procedure**

1. Create an `Issuer` to configure the HTTP-01 solver by running the following command. For other ACME issuer types, see "Configuring ACME an issuer".
   ```yaml
   $ oc create -f - << EOF
   apiVersion: cert-manager.io/v1
   kind: Issuer
   metadata:
     name: letsencrypt-acme
     namespace: <namespace>
   spec:
     acme:
       server: https://acme-v02.api.letsencrypt.org/directory
       privateKeySecretRef:
         name: letsencrypt-acme-account-key
       solvers:
         - http01:
             ingress:
               ingressClassName: openshift-default
   EOF
   ```

   where:

   <dl>
   <dt><code>&lt;namespace&gt;</code></dt>
   <dd>Specifies the namespace where the <code>Issuer</code> is located. It must be the same as the route namespace.</dd>
   </dl>
2. Create a `Certificate` object for the route by running the following command. The `secretName` specifies the TLS secret that is going to be issued and managed by cert-manager and will also be referenced in your route in the following steps.
   ```yaml
   $ oc create -f - << EOF
   apiVersion: cert-manager.io/v1
   kind: Certificate
   metadata:
     name: example-route-cert
     namespace: <namespace>
   spec:
     commonName: <common_host>
     dnsNames:
       - <hostname>
     usages:
       - server auth
     issuerRef:
       kind: Issuer
       name: letsencrypt-acme
     secretName: <secret_name>
   EOF
   ```

   where:

   <dl>
   <dt><code>&lt;namespace&gt;</code></dt>
   <dd>Specifies the <code>namespace</code> where the <code>Certificate</code> resource is located. It should be the same as the namespace of your route.</dd>
   <dt><code>&lt;common_host&gt;</code></dt>
   <dd>Specifies the common name of your certificate by using the hostname of the route.</dd>
   <dt><code>&lt;hostname&gt;</code></dt>
   <dd>Specifies the hostname of your route to the DNS names of your certificate.</dd>
   <dt><code>&lt;secret_name&gt;</code></dt>
   <dd>Specifies the name of the secret that contains the certificate.</dd>
   </dl>
3. Create a `Role` to provide the router service account permissions to read the referenced secret by using the following command:
   ```terminal
   $ oc create role secret-reader \
     --verb=get,list,watch \
     --resource=secrets \
     --resource-name=<secret_name> \
     --namespace=<namespace>
   ```

   where:

   <dl>
   <dt><code>&lt;secret_name&gt;</code></dt>
   <dd>Specifies the name of the secret that you want to grant access to. It should be consistent with your <code>secretName</code> specified in the <code>Certificate</code> resource.</dd>
   <dt><code>&lt;namespace&gt;</code></dt>
   <dd>Specifies the namespace where both your secret and route are located.</dd>
   </dl>
4. Create a `RoleBinding` resource to bind the router service account with the newly created `Role` resource by using the following command:
   ```terminal
   $ oc create rolebinding secret-reader-binding \
     --role=secret-reader \
     --serviceaccount=openshift-ingress:router \
     --namespace=<namespace>
   ```

   where:

   <dl>
   <dt><code>&lt;namespace&gt;</code></dt>
   <dd>Specifies the namespace where both your secret and route are located.</dd>
   </dl>
5. Create a route for your service resource, that uses edge TLS termination and a custom hostname, by running the following command. The hostname is used when creating a `Certificate` resource in the next step.
   ```terminal
   $ oc create route edge <route_name> \
     --service=<service_name> \
     --hostname=<hostname> \
     --namespace=<namespace>
   ```

   where:

   <dl>
   <dt><code>&lt;route_name&gt;</code></dt>
   <dd>Specifies the name of your route.</dd>
   <dt><code>&lt;service_name&gt;</code></dt>
   <dd>Specifies the service you want to expose.</dd>
   <dt><code>&lt;hostname&gt;</code></dt>
   <dd>Specifies the hostname of your route.</dd>
   <dt><code>&lt;namespace&gt;</code></dt>
   <dd>Specifies the namespace where your route is located.</dd>
   </dl>
6. To reference the secret and use the certificate issued by `cert-manager`, update the `.spec.tls.externalCertificate` field in the route by using the following command:
   ```terminal
   $ oc patch route <route_name> \
     -n <namespace> \
     --type=merge \
     -p '{"spec":{"tls":{"externalCertificate":{"name":"<secret_name>"}}}}'
   ```

   where:

   <dl>
   <dt><code>&lt;route_name&gt;</code></dt>
   <dd>Specifies the name of your route.</dd>
   <dt><code>&lt;namespace&gt;</code></dt>
   <dd>Specifies the namespace where both your secret and route are located.</dd>
   <dt><code>&lt;secret_name&gt;</code></dt>
   <dd>Specifies the name of the secret that contains the certificate.</dd>
   </dl>

**Verification**

1. Verify that the certificate is created and ready to use by running the following command:
   ```terminal
   $ oc get certificate -n <namespace>
   $ oc get secret -n <namespace>
   ```

   where:

   <dl>
   <dt><code>&lt;namespace&gt;</code></dt>
   <dd>Specifies the namespace where both your secret and route are located.</dd>
   </dl>
2. Verify that the router is using the referenced external certificate by running the following command. The command should return with the status code `200 OK`.
   ```terminal
   $ curl -IsS https://<hostname>
   ```

   where:

   <dl>
   <dt><code>&lt;hostname&gt;</code></dt>
   <dd>Specifies the hostname of your route.</dd>
   </dl>
3. Verify the `subject`, `subjectAltName`, and `issuer` fields of your server certificate are all as expected from the curl verbose outputs by running the following command:
   ```terminal
   $ curl -v https://<hostname>
   ```

   where:

   <dl>
   <dt><code>&lt;hostname&gt;</code></dt>
   <dd>Specifies the hostname of your route.</dd>
   </dl>

The certificate from the referenced secret secures the route. The `cert-manager` component issues the certificate and automatically manages the certificate lifecycle.

**Additional resources**

- [Creating a route with externally managed certificate](/docs/networking/ingress_load_balancing/routes/nw-configuring-routes#nw-ingress-route-secret-load-external-cert_secured-routes)
- [Configuring an ACME issuer](/docs/security/cert_manager_operator/cert-manager-operator-issuer-acme#cert-manager-operator-issuer-acme)
- [Externally managed certificates](/docs/networking/ingress_load_balancing/routes/securing-routes#nw-ingress-route-secret-load-external-cert_secured-routes)
