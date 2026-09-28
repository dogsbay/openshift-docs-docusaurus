---
title: Replacing the default ingress certificate
sidebar_position: 1
---

# Replacing the default ingress certificate {#replacing-default-ingress}

<a id="replacing-default-ingress"></a>

To allow external clients to connect securely to applications under the .apps subdomain in OpenShift Container Platform, you can replace the default wildcard ingress certificate with one issued by a trusted public CA.

## Understand the default ingress certificate {#understanding-default-ingress_replacing-default-ingress}

You can replace the default ingress certificate with a certificate from a public CA so that external clients connect securely to your applications.

The default ingress certificate in OpenShift Container Platform is a wildcard certificate that the Ingress Operator issues from an internal CA for the web console, CLI, and applications under the `.apps` subdomain.

## Replace the default ingress certificate {#replacing-default-ingress_replacing-default-ingress}

To secure the web console, CLI, and all applications under the `.apps` subdomain in OpenShift Container Platform, you can replace the default ingress certificate by creating a TLS secret with your wildcard certificate and updating the Ingress Controller and cluster proxy configuration.

:::note

Before using the procedure, ensure you understand the following Ingress Controller behaviors:

- When certificates are renewed or rotated by using external certificate management tools, only the contents of the secret, such as the certificate and key, are updated. The secret name remains unchanged. Kubelet automatically propagates these updates to the mounted volume, allowing the router to detect the file changes and hot-reload the new certificate and key. As a result, no rolling update of the router deployment is triggered or required.
- For secret renewal or rotation, the cert-manager Operator changes the secret content, such as a cert/key pair, but does not change the secret name. This happens because kubelet automatically propagates changes to the secret in the volume mount. The router pod detects the file change and then hot reloads the new cert/key pair. Updating the secret content does not trigger rolling update.

:::

**Prerequisites**

- You must have a wildcard certificate for the fully qualified `.apps` subdomain and its corresponding private key. Each should be in a separate PEM format file.
- The private key must be unencrypted. If your key is encrypted, decrypt it before importing it into OpenShift Container Platform.
- The certificate must include the `subjectAltName` extension showing `*.apps.<clustername>.<domain>`.
- The certificate file can contain one or more certificates in a chain. The file must list the wildcard certificate as the first certificate, followed by other intermediate certificates, and then ending with the root CA certificate.
- Copy the root CA certificate into an additional PEM format file.
- Verify that all certificates which include `-----END CERTIFICATE-----` also end with one carriage return after that line.

**Procedure**

1. Create a config map that includes only the root CA certificate that is used to sign the wildcard certificate:
   ```terminal
   $ oc create configmap custom-ca \
        --from-file=ca-bundle.crt=</path/to/example-ca.crt> \
        -n openshift-config
   ```

   where

   <dl>
   <dt><code>&lt;/path/to/example-ca.crt&gt;</code></dt>
   <dd>The path to the root CA certificate file on your local file system. For example, <code>/etc/pki/ca-trust/source/anchors</code>.</dd>
   </dl>
2. Update the cluster-wide proxy configuration with the newly created config map:
   ```terminal
   $ oc patch proxy/cluster \
        --type=merge \
        --patch='{"spec":{"trustedCA":{"name":"custom-ca"}}}'
   ```

   :::note

   If you update only the trusted CA for your cluster, the MCO updates the `/etc/pki/ca-trust/source/anchors/openshift-config-user-ca-bundle.crt` file and the Machine Config Controller (MCC) applies the trusted CA update to each node so that a node reboot is not required. However, with these changes, the Machine Config Daemon (MCD) restarts critical services on each node, such as kubelet and CRI-O. These service restarts cause each node to briefly enter the `NotReady` state until the service is fully restarted.

   If you change any other parameter in the `openshift-config-user-ca-bundle.crt` file, such as `noproxy`, the MCO reboots each node in your cluster.

   :::
3. Create a secret that contains the wildcard certificate chain and key:
   ```terminal
   $ oc create secret tls <secret> \
        --cert=</path/to/cert.crt> \
        --key=</path/to/cert.key> \
        -n openshift-ingress
   ```

   where:

   <dl>
   <dt><code>&lt;secret&gt;</code></dt>
   <dd>Specifies the name of the secret that will contain the certificate chain and private key.</dd>
   <dt><code>&lt;/path/to/cert.crt&gt;</code></dt>
   <dd>Specifies the path to the certificate chain on your local file system.</dd>
   <dt><code>&lt;/path/to/cert.key&gt;</code></dt>
   <dd>Specifies the path to the private key associated with this certificate.</dd>
   </dl>
4. Update the Ingress Controller configuration with the newly created secret:
   ```terminal
   $ oc patch ingresscontroller.operator default \
        --type=merge -p \
        '{"spec":{"defaultCertificate": {"name": "<secret>"}}}' \
        -n openshift-ingress-operator
   ```

   - `<secret>`:: Specifies the name used for the secret. Replace `<secret>` with the name used for the secret.

**Additional resources**

- [Replacing the CA Bundle certificate](/docs/security/certificates/updating-ca-bundle#ca-bundle-understanding_updating-ca-bundle)
- [Proxy certificate customization](/docs/security/certificate_types_descriptions/proxy-certificates#proxy-cert-customization_proxy-certificates)
