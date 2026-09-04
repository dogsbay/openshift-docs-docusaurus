---
title: Configuring certificates with an issuer
sidebar_position: 8
---

# Configuring certificates with an issuer {#cert-manager-creating-certificate}

<a id="cert-manager-creating-certificate"></a>

By using the cert-manager Operator for Red Hat OpenShift, you can manage certificates, handling tasks such as renewal and issuance, for workloads within the cluster, as well as components interacting externally to the cluster.

## Creating certificates for user workloads {#cert-manager-certificate-mgmt_cert-manager-creating-certificate}

To secure communications for your applications, create and manage TLS certificates for your workloads by using the cert-manager Operator for Red Hat OpenShift

**Prerequisites**

- You have access to the cluster with `cluster-admin` privileges.
- You have installed the cert-manager Operator for Red Hat OpenShift.

**Procedure**

1. Create an issuer. For more information, see "Configuring an issuer" in the "Additional resources" section.
2. Create a certificate:
   1. Create a YAML file, for example, `certificate.yaml`, that defines the `Certificate` object:
      ```yaml
      apiVersion: cert-manager.io/v1
      kind: Certificate
      metadata:
        name: <tls_cert>
        namespace: <issuer_namespace>
      spec:
        isCA: false
        commonName: '<common_name>'
        secretName: <secret_name>
        dnsNames:
        - "<domain_name>"
        issuerRef:
          name: <issuer_name>
          kind: Issuer
      ```

      where:

      <dl>
      <dt><code>&lt;tls_cert&gt;</code></dt>
      <dd>Specifies a name for the certificate.</dd>
      <dt><code>&lt;issuer_namespace&gt;</code></dt>
      <dd>Specifies the namespace of the issuer.</dd>
      <dt><code>&lt;common_name&gt;</code></dt>
      <dd>Specifies the common name (CN).</dd>
      <dt><code>&lt;secret_name&gt;</code></dt>
      <dd>Specifies the name of the secret to create that contains the certificate.</dd>
      <dt><code>&lt;domain_name&gt;</code></dt>
      <dd>Specifies the domain name.</dd>
      <dt><code>&lt;issuer_name&gt;</code></dt>
      <dd>Specifies the name of the issuer.</dd>
      </dl>
   2. Create the `Certificate` object by running the following command:
      ```terminal
      $ oc create -f certificate.yaml
      ```

**Verification**

- Verify that the certificate is created and ready to use by running the following command:
  ```terminal
  $ oc get certificate -w -n <issuer_namespace>
  ```

  Once certificate is in `Ready` status, workloads on your cluster can start using the generated certificate secret.

## Creating certificates for the API server {#cert-manager-certificate-api-server_cert-manager-creating-certificate}

To secure interactions with the cluster control plane, create TLS certificates for the API server by using the cert-manager Operator for Red Hat OpenShift.

**Prerequisites**

- You have access to the cluster with `cluster-admin` privileges.
- You have installed version 1.13.0 or later of the cert-manager Operator for Red Hat OpenShift.

**Procedure**

1. Create an issuer. For more information, see "Configuring an issuer" in the "Additional resources" section.
2. Create a certificate:
   1. Create a YAML file, for example, `certificate.yaml`, that defines the `Certificate` object:
      ```yaml
      apiVersion: cert-manager.io/v1
      kind: Certificate
      metadata:
        name: <tls_cert>
        namespace: openshift-config
      spec:
        isCA: false
        commonName: "api.<cluster_base_domain>"
        secretName: <secret_name>
        dnsNames:
        - "api.<cluster_base_domain>"
        issuerRef:
          name: <issuer_name>
          kind: Issuer
      ```

      where:

      <dl>
      <dt><code>&lt;tls_cert&gt;</code></dt>
      <dd>Specifies a name for the certificate.</dd>
      <dt><code>&lt;cluster_base_domain&gt;</code></dt>
      <dd>Specifies the common name (CN).</dd>
      <dt><code>&lt;secret_name&gt;</code></dt>
      <dd>Specifies the name of the secret to create that contains the certificate.</dd>
      <dt><code>&lt;issuer_name&gt;</code></dt>
      <dd>Specifies the name of the issuer.</dd>
      </dl>
   2. Create the `Certificate` object by running the following command:
      ```terminal
      $ oc create -f certificate.yaml
      ```
3. Add the API server named certificate. For more information, see "Adding an API server named certificate" section in the "Additional resources" section.
   :::note

   To ensure the certificates are updated, run the `oc login` command again after the certificate is created.

   :::

**Verification**

- Verify that the certificate is created and ready to use by running the following command:
  ```terminal
  $ oc get certificate -w -n openshift-config
  ```

  Once certificate is in `Ready` status, API server on your cluster can start using the generated certificate secret.

## Creating certificates for the Ingress Controller {#cert-manager-certificate-ingress_cert-manager-creating-certificate}

You can create a certificate for the Ingress Controller and then replace bootstrapped default self-signed certificates with cert-manager-managed external certificates.

:::note

Before using the procedure, ensure you understand the following Ingress Controller behaviors:

- When certificates are renewed or rotated by using the cert-manager Operator, only the contents of the secret, such as the certificate and key, are updated. The secret name remains unchanged. Kubelet automatically propagates these updates to the mounted volume, allowing the router to detect the file changes and hot-reload the new certificate and key. As a result, no rolling update of the router deployment is triggered or required.
- The secret name is referenced in the Ingress Controller configuration. If you want to replace the default ingress certificate or use different secret name in Ingress Controller configuration, you must patch or edit the configuration to apply the change. This operation triggers a rolling update for router pods where new router pods load the new cert/key pair.

:::

**Prerequisites**

- You have access to the cluster with `cluster-admin` privileges.
- You have installed version 1.13.0 or later of the cert-manager Operator for Red Hat OpenShift.

**Procedure**

1. Create an issuer. For more information, see "Configuring an issuer" in the "Additional resources" section.
2. Create a certificate:
   1. Create a YAML file, for example, `certificate.yaml`, that defines the `Certificate` object:
      ```yaml title="Example certificate.yaml file"
      apiVersion: cert-manager.io/v1
      kind: Certificate
      metadata:
        name: <tls_cert>
        namespace: openshift-ingress
      spec:
        isCA: false
        commonName: "apps.<cluster_base_domain>"
        secretName: <secret_name>
        dnsNames:
        - "apps.<cluster_base_domain>"
        - "\*.apps.<cluster_base_domain>"
        issuerRef:
          name: <issuer_name>
          kind: Issuer
      ```

      where:

      <dl>
      <dt><code>&lt;tls_cert&gt;</code></dt>
      <dd>Specifies the name for the certificate.</dd>
      <dt><code>&lt;cluster_base_domain&gt;</code></dt>
      <dd>Specifies the common name (CN).</dd>
      <dt><code>&lt;secret_name&gt;</code></dt>
      <dd>Specifies the name of the secret to create that contains the certificate.</dd>
      <dt><code>&lt;cluster_base_domain&gt;</code></dt>
      <dd>Specifies the DNS name of the ingress.</dd>
      <dt><code>&lt;issuer_name&gt;</code></dt>
      <dd>Specifies the name of the issuer.</dd>
      </dl>
   2. Create the `Certificate` object by running the following command:
      ```terminal
      $ oc create -f certificate.yaml
      ```
3. Replace the default ingress certificate. For more information, see "Replacing the default ingress certificate" section in the "Additional resources" section.

**Verification**

1. Verify that the certificate is created and ready to use by running the following command:
   ```terminal
   $ oc get certificate -n openshift-ingress
   ```
2. Verify the definition and content of the secret object by running the following command:
   ```terminal
   $ oc get secret <secretName> -n openshift-ingress
   ```
3. Verify that the default TLS certificate has the correct configuration details for the Ingress Controller by running the following command:
   ```terminal
   $ oc get ingresscontroller default -n openshift-ingress-operator -o yaml | grep -A2 defaultCertificate
   ```

   After the certificate is in `Ready` status, the Ingress Controller on your cluster can start using the generated certificate secret.

**Additional resources**

- [Red Hat Knowledgebase solution](https://access.redhat.com/solutions/4542531)

**Additional resources**

- [Supported issuer types](/docs/security/cert_manager_operator#cert-manager-issuer-types_cert-manager-operator-about)
- [Configuring an ACME issuer](/docs/security/cert_manager_operator/cert-manager-operator-issuer-acme#cert-manager-operator-issuer-acme)
- [Adding an API server named certificate](/docs/security/certificates/api-server#customize-certificates-api-add-named_api-server-certificates)
- [Replacing the default ingress certificate](/docs/security/certificates/replacing-default-ingress-certificate#replacing-default-ingress)
