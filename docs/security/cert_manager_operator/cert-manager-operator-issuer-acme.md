---
title: Configuring an ACME issuer
sidebar_position: 7
---

# Configuring an ACME issuer {#cert-manager-operator-issuer-acme}

<a id="cert-manager-operator-issuer-acme"></a>

The cert-manager Operator for Red Hat OpenShift supports using Automated Certificate Management Environment (ACME) CA servers, such as *Let’s Encrypt*, to issue certificates. Explicit credentials are configured by specifying the secret details in the `Issuer` API object. Ambient credentials are extracted from the environment, metadata services, or local files which are not explicitly configured in the `Issuer` API object.

The `Issuer` object is namespace scoped. It can only issue certificates from the same namespace. You can also use the `ClusterIssuer` object to issue certificates across all namespaces in the cluster.

```yaml title="Example YAML file that defines the ClusterIssuer object"
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: acme-cluster-issuer
spec:
  acme:
    ...
```

:::note

By default, you can use the `ClusterIssuer` object with ambient credentials. To use the `Issuer` object with ambient credentials, you must enable the `--issuer-ambient-credentials` setting for the cert-manager controller.

:::

## About ACME issuers {#cert-manager-acme-about_cert-manager-operator-issuer-acme}

The ACME issuer type for the cert-manager Operator for Red Hat OpenShift represents an Automated Certificate Management Environment (ACME) certificate authority (CA) server. ACME CA servers rely on a *challenge* to verify that a client owns the domain names that the certificate is being requested for. If the challenge is successful, the cert-manager Operator for Red Hat OpenShift can issue the certificate. If the challenge fails, the cert-manager Operator for Red Hat OpenShift does not issue the certificate.

:::note

Private DNS zones are not supported with *Let’s Encrypt* and internet ACME servers.

:::

### Supported ACME challenges types {#cert-manager-acme-challenges-types_cert-manager-operator-issuer-acme}

To validate domain ownership with ACME issuers, you can use the challenge types supported by the cert-manager Operator for Red Hat OpenShift.

The cert-manager Operator for Red Hat OpenShift supports the following challenge types for ACME issuers:

HTTP-01
: With the HTTP-01 challenge type, you provide a computed key at an HTTP URL endpoint in your domain. If the ACME CA server can get the key from the URL, it can validate you as the owner of the domain.

:::note

HTTP-01 requires that the Let’s Encrypt servers can access the route of the cluster. If an internal or private cluster is behind a proxy, the HTTP-01 validations for certificate issuance fail.

The HTTP-01 challenge is restricted to port 80.

:::

DNS-01
: With the DNS-01 challenge type, you provide a computed key at a DNS TXT record. If the ACME CA server can get the key by DNS lookup, it can validate you as the owner of the domain.

### Supported DNS-01 providers {#cert-manager-acme-dns-providers_cert-manager-operator-issuer-acme}

To configure DNS-01 challenges for ACME issuers, you can validate domain ownership by integrating with supported services, such as Amazon Route 53, Azure DNS, and Google Cloud DNS, or by using Webhooks.

The cert-manager Operator for Red Hat OpenShift supports the following DNS-01 providers for ACME issuers:

- Amazon Route 53
- Azure DNS
  :::note

  The cert-manager Operator for Red Hat OpenShift does not support using Microsoft Entra ID pod identities to assign a managed identity to a pod.

  :::
- Google Cloud DNS
- Webhook Red Hat tests and supports DNS providers using an external webhook with cert-manager on OpenShift Container Platform. The following DNS providers are tested and supported with OpenShift Container Platform:
  - [cert-manager-webhook-ibmcis](https://github.com/jb-dk/cert-manager-webhook-ibmcis)

  :::note

  Using a DNS provider that is not listed might work with OpenShift Container Platform, but the provider was not tested by Red Hat and therefore is not supported by Red Hat.

  :::

## Configuring an ACME issuer to solve HTTP-01 challenges {#cert-manager-acme-http01_cert-manager-operator-issuer-acme}

You can use cert-manager Operator for Red Hat OpenShift to set up an ACME issuer to solve HTTP-01 challenges. This procedure uses *Let’s Encrypt* as the ACME CA server.

**Prerequisites**

- You have access to the cluster as a user with the `cluster-admin` role.
- You have a service that you want to expose. In this procedure, the service is named `sample-workload`.

**Procedure**

1. Create an ACME cluster issuer.
   1. Create a YAML file, `acme-cluster-issuer.yaml`, that defines the `ClusterIssuer` object:
      ```yaml
      apiVersion: cert-manager.io/v1
      kind: ClusterIssuer
      metadata:
        name: <cluster_issuer_name>
      spec:
        acme:
          preferredChain: ""
          privateKeySecretRef:
            name: <secret_for_private_key>
          server: <url>
          solvers:
          - http01:
              ingress:
                ingressClassName: <ingress_class_name>
      ```

      where:

      <dl>
      <dt><code>&lt;cluster_issuer_name&gt;</code></dt>
      <dd>Specifies a name for the cluster issuer.</dd>
      <dt><code>&lt;secret_for_private_key&gt;</code></dt>
      <dd>Specifies the name of secret to store the ACME account private key in.</dd>
      <dt><code>&lt;url&gt;</code></dt>
      <dd>Specifies the URL to access the ACME server’s <code>directory</code> endpoint. This example uses the <em>Let’s Encrypt</em> staging environment.</dd>
      <dt><code>&lt;ingress_class_name&gt;</code></dt>
      <dd>Specifies the Ingress class, for example, <code>openshift-default</code>.</dd>
      </dl>
   2. Optional: If you create the object without specifying `ingressClassName`, use the following command to patch the existing ingress:
      ```terminal
      $ oc patch ingress/<ingress-name> --type=merge --patch '{"spec":{"ingressClassName":"openshift-default"}}' -n <namespace>
      ```
   3. Create the `ClusterIssuer` object by running the following command:
      ```terminal
      $ oc create -f acme-cluster-issuer.yaml
      ```
2. Create an Ingress to expose the service of the user workload.
   1. Create a YAML file, for example, `namespace.yaml`, that defines a `Namespace` object:
      ```yaml
      apiVersion: v1
      kind: Namespace
      metadata:
        name: <ingress_namespace>
      ```

      Replace `<ingress_namespace>` with the namespace for the Ingress.
   2. Create the `Namespace` object by running the following command:
      ```terminal
      $ oc create -f namespace.yaml
      ```
   3. Create a YAML file, for example, `ingress.yaml`, that defines the `Ingress` object:
      ```yaml
      apiVersion: networking.k8s.io/v1
      kind: Ingress
      metadata:
        name: <ingress_name>
        namespace: <ingress_namespace>
        annotations:
          cert-manager.io/cluster-issuer: <cluster_issuer_name>
      spec:
        ingressClassName: <ingress_class_name>
        tls:
        - hosts:
          - <tls_hostname>
          secretName: <secret_name>
        rules:
        - host: <hostname>
          http:
            paths:
            - path: /
              pathType: Prefix
              backend:
                service:
                  name: <service_name>
                  port:
                    number: 80
      ```

      where:

      <dl>
      <dt><code>&lt;ingress_name&gt;</code></dt>
      <dd>Specifies the name of the Ingress.</dd>
      <dt><code>&lt;ingress_namespace&gt;</code></dt>
      <dd>Specifies the namespace that you created for the Ingress.</dd>
      <dt><code>&lt;cluster_issuer_name&gt;</code></dt>
      <dd>Specifies the cluster issuer that you created.</dd>
      <dt><code>&lt;ingress_class_name&gt;</code></dt>
      <dd>Specifies the Ingress class name.</dd>
      <dt><code>&lt;tls_hostname&gt;</code></dt>
      <dd>Specifies the Subject Alternative Name (SAN) to be associated with the certificate. This name is used to add DNS names to the certificate.</dd>
      <dt><code>&lt;secret_name&gt;</code></dt>
      <dd>Specifies the secret that stores the certificate.</dd>
      <dt><code>&lt;hostname&gt;</code></dt>
      <dd>Specifies the host name. You can use the <code>&lt;host_name&gt;.&lt;cluster_ingress_domain&gt;</code> syntax to take advantage of the <code>*.&lt;cluster_ingress_domain&gt;</code> wildcard DNS record and serving certificate for the cluster. For example, you might use <code>apps.&lt;cluster_base_domain&gt;</code>. Otherwise, you must ensure that a DNS record exists for the chosen hostname.</dd>
      <dt><code>&lt;service_name&gt;</code></dt>
      <dd>Specifies the name of the service to expose. This example uses a service named <code>sample-workload</code>.</dd>
      </dl>
   4. Create the `Ingress` object by running the following command:
      ```terminal
      $ oc create -f ingress.yaml
      ```

## Configuring an ACME issuer by using explicit credentials for AWS Route53 {#cert-manager-acme-dns01-explicit-aws_cert-manager-operator-issuer-acme}

You can use cert-manager Operator for Red Hat OpenShift to set up an Automated Certificate Management Environment (ACME) issuer to solve DNS-01 challenges by using explicit credentials on AWS. This procedure uses *Let’s Encrypt* as the ACME certificate authority (CA) server and shows how to solve DNS-01 challenges with Amazon Route 53.

**Prerequisites**

- You must provide the explicit `accessKeyID` and `secretAccessKey` credentials. For more information, see [Route53](https://cert-manager.io/docs/configuration/acme/dns01/route53/) in the upstream cert-manager documentation.
  :::note

  You can use Amazon Route 53 with explicit credentials in an OpenShift Container Platform cluster that is not running on AWS.

  :::

**Procedure**

1. Optional: Override the name server settings for the DNS-01 self check.
   This step is required only when the target public-hosted zone overlaps with the cluster’s default private-hosted zone.

   1. Edit the `CertManager` resource by running the following command:
      ```terminal
      $ oc edit certmanager cluster
      ```
   2. Add a `spec.controllerConfig` section with the following override arguments:
      ```yaml
      apiVersion: operator.openshift.io/v1alpha1
      kind: CertManager
      metadata:
        name: cluster
        ...
      spec:
        ...
        controllerConfig:
          overrideArgs:
            - '--dns01-recursive-nameservers-only'
            - '--dns01-recursive-nameservers=1.1.1.1:53'
      ```

      where:

      <dl>
      <dt><code>--dns01-recursive-nameservers-only</code></dt>
      <dd>Specifies recursive name servers instead of checking the authoritative name servers associated with that domain.</dd>
      <dt><code>--dns01-recursive-nameservers=1.1.1.1:53</code></dt>
      <dd>Specifies a comma-separated list of <code>&lt;host&gt;:&lt;port&gt;</code> names servers to query for the DNS-01 self check. You must use a <code>1.1.1.1:53</code> value to avoid the public and private zones overlapping.</dd>
      </dl>
   3. Save the file to apply the changes.
2. Optional: Create a namespace for the issuer:
   ```terminal
   $ oc new-project <issuer_namespace>
   ```
3. Create a secret to store your AWS credentials in by running the following command:
   ```terminal
   $ oc create secret -n my-issuer-namespace generic aws-secret \
     --from-literal=awsSecretAccessKey=<aws_secret_access_key>
   ```

   Replace `<aws_secret_access_key>` with your AWS secret access key.
4. Create an issuer:
   1. Create a YAML file that defines the `Issuer` object:
      ```yaml title="Example issuer.yaml file"
      apiVersion: cert-manager.io/v1
      kind: Issuer
      metadata:
        name: <issuer_name>
        namespace: <issuer_namespace>
      spec:
        acme:
          server: <server>
          email: "<email_address>"
          privateKeySecretRef:
            name: <secret_private_key>
          solvers:
          - dns01:
              route53:
                accessKeyID: <aws_key_id>
                hostedZoneID: <hosted_zone_id>
                region: <region_name>
                secretAccessKeySecretRef:
                  name: "<aws_secret>"
                  key: "<aws_secret_access_key>"
      ```

      where:

      <dl>
      <dt><code>&lt;issuer_name&gt;</code></dt>
      <dd>Specifies a name for the issuer.</dd>
      <dt><code>&lt;issuer_namespace&gt;</code></dt>
      <dd>Specifies the namespace that you created for the issuer.</dd>
      <dt><code>server</code></dt>
      <dd>Specifies the URL to access the ACME server’s <code>directory</code> endpoint. This example uses the <em>Let’s Encrypt</em> staging environment.</dd>
      <dt><code>&lt;email_address&gt;</code></dt>
      <dd>Specifies your email address.</dd>
      <dt><code>&lt;secret_private_key&gt;</code></dt>
      <dd>Specifies the name of the secret to store the ACME account private key in.</dd>
      <dt><code>&lt;aws_key_id&gt;</code></dt>
      <dd>Specifies your AWS key ID.</dd>
      <dt><code>&lt;hosted_zone_id&gt;</code></dt>
      <dd>Specifies your hosted zone ID.</dd>
      <dt><code>&lt;region_name&gt;</code></dt>
      <dd>Specifies the AWS region name. For example, <code>us-east-1</code>.</dd>
      <dt><code>&lt;aws_secret&gt;</code></dt>
      <dd>Specifies the name of the secret you created.</dd>
      <dt><code>&lt;aws_secret_access_key&gt;</code></dt>
      <dd>Specifies the key in the secret you created that stores your AWS secret access key.</dd>
      </dl>
   2. Create the `Issuer` object by running the following command:
      ```terminal
      $ oc create -f issuer.yaml
      ```

## Configuring an ACME issuer by using ambient credentials on AWS {#cert-manager-acme-dns01-ambient-aws_cert-manager-operator-issuer-acme}

You can use cert-manager Operator for Red Hat OpenShift to set up an ACME issuer to solve DNS-01 challenges by using ambient credentials on AWS. This procedure uses *Let’s Encrypt* as the ACME CA server and shows how to solve DNS-01 challenges with Amazon Route 53.

**Prerequisites**

- If your cluster is configured to use the AWS Security Token Service (STS), you followed the instructions from the *Configuring cloud credentials for the cert-manager Operator for Red Hat OpenShift for the AWS Security Token Service cluster* section.
- If your cluster does not use the AWS STS, you followed the instructions from the *Configuring cloud credentials for the cert-manager Operator for Red Hat OpenShift on AWS* section.

**Procedure**

1. Optional: Override the name server settings for the DNS-01 self check.
   This step is required only when the target public-hosted zone overlaps with the cluster’s default private-hosted zone.

   1. Edit the `CertManager` resource by running the following command:
      ```terminal
      $ oc edit certmanager cluster
      ```
   2. Add a `spec.controllerConfig` section with the following override arguments:
      ```yaml
      apiVersion: operator.openshift.io/v1alpha1
      kind: CertManager
      metadata:
        name: cluster
        ...
      spec:
        ...
        controllerConfig:
          overrideArgs:
            - '--dns01-recursive-nameservers-only'
            - '--dns01-recursive-nameservers=1.1.1.1:53'
      ```

      where:

      <dl>
      <dt><code>--dns01-recursive-nameservers-only</code></dt>
      <dd>Specifies recursive name servers instead of checking the authoritative name servers associated with that domain.</dd>
      <dt><code>--dns01-recursive-nameservers=1.1.1.1:53</code></dt>
      <dd>Specifies a comma-separated list of <code>&lt;host&gt;:&lt;port&gt;</code> name servers to query for the DNS-01 self check. You must use a <code>1.1.1.1:53</code> value to avoid the public and private zones overlapping.</dd>
      </dl>
   3. Save the file to apply the changes.
2. Optional: Create a namespace for the issuer:
   ```terminal
   $ oc new-project <issuer_namespace>
   ```
3. Modify the `CertManager` resource to add the `--issuer-ambient-credentials` argument:
   ```terminal
   $ oc patch certmanager/cluster \
     --type=merge \
     -p='{"spec":{"controllerConfig":{"overrideArgs":["--issuer-ambient-credentials"]}}}'
   ```
4. Create an issuer:
   1. Create a YAML file that defines the `Issuer` object:
      ```yaml title="Example issuer.yaml file"
      apiVersion: cert-manager.io/v1
      kind: Issuer
      metadata:
        name: <issuer_name>
        namespace: <issuer_namespace>
      spec:
        acme:
          server: <server>
          email: "<email_address>"
          privateKeySecretRef:
            name: <secret_private_key>
          solvers:
          - dns01:
              route53:
                hostedZoneID: <hosted_zone_id>
                region: us-east-1
      ```

      where:

      <dl>
      <dt><code>&lt;issuer_name&gt;</code></dt>
      <dd>Specifies a name for the issuer.</dd>
      <dt><code>&lt;issuer_namespace&gt;</code></dt>
      <dd>Specifies the namespace that you created for the issuer.</dd>
      <dt><code>&lt;server&gt;</code></dt>
      <dd>Specifies the URL to access the ACME server’s <code>directory</code> endpoint. This example uses the <em>Let’s Encrypt</em> staging environment.</dd>
      <dt><code>&lt;email_address&gt;</code></dt>
      <dd>Specifies your email address.</dd>
      <dt><code>&lt;secret_private_key&gt;</code></dt>
      <dd>Specifies the name of the secret to store the ACME account private key in.</dd>
      <dt><code>&lt;hosted_zone_id&gt;</code></dt>
      <dd>Specifies your hosted zone ID.</dd>
      </dl>
   2. Create the `Issuer` object by running the following command:
      ```terminal
      $ oc create -f issuer.yaml
      ```

## Configuring an ACME issuer by using explicit credentials for Google Cloud DNS {#cert-manager-acme-dns01-explicit-gcp_cert-manager-operator-issuer-acme}

You can use the cert-manager Operator for Red Hat OpenShift to set up an ACME issuer to solve DNS-01 challenges by using explicit credentials on Google Cloud. This procedure uses *Let’s Encrypt* as the ACME CA server and shows how to solve DNS-01 challenges with Google Cloud DNS.

**Prerequisites**

- You have set up a Google Cloud service account with a desired role for Google Cloud DNS.
  :::note

  You can use Google Cloud DNS with explicit credentials in an OpenShift Container Platform cluster that is not running on Google Cloud.

  :::

**Procedure**

1. Optional: Override the name server settings for the DNS-01 self check.
   This step is required only when the target public-hosted zone overlaps with the cluster’s default private-hosted zone.

   1. Edit the `CertManager` resource by running the following command:
      ```terminal
      $ oc edit certmanager cluster
      ```
   2. Add a `spec.controllerConfig` section with the following override arguments:
      ```yaml
      apiVersion: operator.openshift.io/v1alpha1
      kind: CertManager
      metadata:
        name: cluster
        ...
      spec:
        ...
        controllerConfig:
          overrideArgs:
            - '--dns01-recursive-nameservers-only'
            - '--dns01-recursive-nameservers=1.1.1.1:53'
      ```

      where:

      <dl>
      <dt><code>--dns01-recursive-nameservers-only</code></dt>
      <dd>Specifies recursive name servers instead of checking the authoritative name servers associated with that domain.</dd>
      <dt><code>--dns01-recursive-nameservers=1.1.1.1:53</code></dt>
      <dd>Specifies a comma-separated list of <code>&lt;host&gt;:&lt;port&gt;</code> name servers to query for the DNS-01 self check. You must use a <code>1.1.1.1:53</code> value to avoid the public and private zones overlapping.</dd>
      </dl>
   3. Save the file to apply the changes.
2. Optional: Create a namespace for the issuer:
   ```terminal
   $ oc new-project my-issuer-namespace
   ```
3. Create a secret to store your Google Cloud credentials by running the following command:
   ```terminal
   $ oc create secret generic clouddns-dns01-solver-svc-acct --from-file=service_account.json=<path/to/gcp_service_account.json> -n my-issuer-namespace
   ```
4. Create an issuer:
   1. Create a YAML file, for example, `issuer.yaml`, that defines the `Issuer` object:
      ```yaml
      apiVersion: cert-manager.io/v1
      kind: Issuer
      metadata:
        name: <acme_dns01_clouddns_issuer>
        namespace: <issuer_namespace>
      spec:
        acme:
          preferredChain: ""
          privateKeySecretRef:
            name: <secret_private_key>
          server: <server>
          solvers:
          - dns01:
              cloudDNS:
                project: <project_id>
                serviceAccountSecretRef:
                  name: <secret>
                  key: <service_account.json>
      ```

      where:

      <dl>
      <dt><code>&lt;acme_dns01_clouddns_issuer&gt;</code></dt>
      <dd>Specifies a name for the issuer.</dd>
      <dt><code>&lt;issuer_namespace&gt;</code></dt>
      <dd>Specifies your issuer namespace.</dd>
      <dt><code>&lt;secret_private_key&gt;</code></dt>
      <dd>Specifies the name of the secret to store the ACME account private key in.</dd>
      <dt><code>&lt;server&gt;</code></dt>
      <dd>Specifies the URL to access the ACME server’s <code>directory</code> endpoint. This example uses the <em>Let’s Encrypt</em> staging environment.</dd>
      <dt><code>&lt;project_id&gt;</code></dt>
      <dd>Specifies the name of the Google Cloud project that contains the Cloud DNS zone.</dd>
      <dt><code>&lt;secret&gt;</code></dt>
      <dd>Specifies the name of the secret you created.</dd>
      <dt><code>&lt;service_account.json&gt;</code></dt>
      <dd>Specifies the key in the secret you created that stores your Google Cloud secret access key.</dd>
      </dl>
   2. Create the `Issuer` object by running the following command:
      ```terminal
      $ oc create -f issuer.yaml
      ```

## Configuring an ACME issuer by using ambient credentials on Google Cloud {#cert-manager-acme-dns01-ambient-gcp_cert-manager-operator-issuer-acme}

You can use the cert-manager Operator for Red Hat OpenShift to set up an ACME issuer to solve DNS-01 challenges by using ambient credentials on Google Cloud. This procedure uses *Let’s Encrypt* as the ACME CA server and shows how to solve DNS-01 challenges with Google Cloud DNS.

**Prerequisites**

- If your cluster is configured to use Google Cloud Workload Identity, you followed the instructions from the *Configuring cloud credentials for the cert-manager Operator for Red Hat OpenShift with Google Cloud Workload Identity* section.
- If your cluster does not use Google Cloud Workload Identity, you followed the instructions from the *Configuring cloud credentials for the cert-manager Operator for Red Hat OpenShift on Google Cloud* section.

**Procedure**

1. Optional: Override the name server settings for the DNS-01 self check.
   This step is required only when the target public-hosted zone overlaps with the cluster’s default private-hosted zone.

   1. Edit the `CertManager` resource by running the following command:
      ```terminal
      $ oc edit certmanager cluster
      ```
   2. Add a `spec.controllerConfig` section with the following override arguments:
      ```yaml
      apiVersion: operator.openshift.io/v1alpha1
      kind: CertManager
      metadata:
        name: cluster
        ...
      spec:
        ...
        controllerConfig:
          overrideArgs:
            - '--dns01-recursive-nameservers-only'
            - '--dns01-recursive-nameservers=1.1.1.1:53'
      ```

      where:

      <dl>
      <dt><code>--dns01-recursive-nameservers-only</code></dt>
      <dd>Specifies recursive name servers instead of checking the authoritative name servers associated with that domain.</dd>
      <dt><code>--dns01-recursive-nameservers=1.1.1.1:53</code></dt>
      <dd>Specifies a comma-separated list of <code>&lt;host&gt;:&lt;port&gt;</code> name servers to query for the DNS-01 self check. You must use a <code>1.1.1.1:53</code> value to avoid the public and private zones overlapping.</dd>
      </dl>
   3. Save the file to apply the changes.
2. Optional: Create a namespace for the issuer:
   ```terminal
   $ oc new-project <issuer_namespace>
   ```
3. Modify the `CertManager` resource to add the `--issuer-ambient-credentials` argument:
   ```terminal
   $ oc patch certmanager/cluster \
     --type=merge \
     -p='{"spec":{"controllerConfig":{"overrideArgs":["--issuer-ambient-credentials"]}}}'
   ```
4. Create an issuer:
   1. Create a YAML file that defines the `Issuer` object:
      ```yaml title="Example issuer.yaml file"
      apiVersion: cert-manager.io/v1
      kind: Issuer
      metadata:
        name: <issuer_name>
        namespace: <issuer_namespace>
      spec:
        acme:
          preferredChain: ""
          privateKeySecretRef:
            name: <secret_private_key>
          server: <server>
          solvers:
          - dns01:
              cloudDNS:
                project: <gcp_project_id>
      ```

      where:

      <dl>
      <dt><code>&lt;issuer_name&gt;</code></dt>
      <dd>Specifies a name for the issuer.</dd>
      <dt><code>&lt;issuer_namespace&gt;</code></dt>
      <dd>Specifies a namespace for the issuer.</dd>
      <dt><code>&lt;secret_private_key&gt;</code></dt>
      <dd>Specifies the name of the secret to store the ACME account private key in.</dd>
      <dt><code>&lt;server&gt;</code></dt>
      <dd>Specifies the URL to access the ACME server’s <code>directory</code> endpoint. This example uses the <em>Let’s Encrypt</em> staging environment.</dd>
      <dt><code>&lt;gcp_project_id&gt;</code></dt>
      <dd>Specifies the name of the Google Cloud project that contains the Cloud DNS zone.</dd>
      </dl>
   2. Create the `Issuer` object by running the following command:
      ```terminal
      $ oc create -f issuer.yaml
      ```

## Configuring an ACME issuer by using explicit credentials for Microsoft Azure DNS {#cert-manager-acme-dns01-explicit-azure_cert-manager-operator-issuer-acme}

You can use cert-manager Operator for Red Hat OpenShift to set up an ACME issuer to solve DNS-01 challenges by using explicit credentials on Microsoft Azure. This procedure uses *Let’s Encrypt* as the ACME CA server and shows how to solve DNS-01 challenges with Azure DNS.

**Prerequisites**

- You have set up a service principal with desired role for Azure DNS.
  :::note

  You can follow this procedure for an OpenShift Container Platform cluster that is not running on Microsoft Azure.

  :::

**Procedure**

1. Optional: Override the nameserver settings for the DNS-01 self check.
   This step is required only when the target public-hosted zone overlaps with the cluster’s default private-hosted zone.

   1. Edit the `CertManager` resource by running the following command:
      ```terminal
      $ oc edit certmanager cluster
      ```
   2. Add a `spec.controllerConfig` section with the following override arguments:
      ```yaml
      apiVersion: operator.openshift.io/v1alpha1
      kind: CertManager
      metadata:
        name: cluster
        ...
      spec:
        ...
        controllerConfig:
          overrideArgs:
            - '--dns01-recursive-nameservers-only'
            - '--dns01-recursive-nameservers=1.1.1.1:53'
      ```

      where:

      <dl>
      <dt><code>--dns01-recursive-nameservers-only</code></dt>
      <dd>Specifies recursive name servers instead of checking the authoritative name servers associated with that domain.</dd>
      <dt><code>--dns01-recursive-nameservers=1.1.1.1:53</code></dt>
      <dd>Specifies a comma-separated list of <code>&lt;host&gt;:&lt;port&gt;</code> name servers to query for the DNS-01 self check. You must use a <code>1.1.1.1:53</code> value to avoid the public and private zones overlapping.</dd>
      </dl>
   3. Save the file to apply the changes.
2. Optional: Create a namespace for the issuer:
   ```terminal
   $ oc new-project my-issuer-namespace
   ```
3. Create a secret to store your Azure credentials in by running the following command:
   ```terminal
   $ oc create secret generic <secret_name> --from-literal=<azure_secret_access_key_name>=<azure_secret_access_key_value> \
       -n my-issuer-namespace
   ```

   - Replace `<secret_name>` with your secret name.
   - Replace `<azure_secret_access_key_name>` with your Azure secret access key name.
   - Replace `<azure_secret_access_key_value>` with your Azure secret key.
4. Create an issuer:
   1. Create a YAML file, for example, `issuer.yaml`, that defines the `Issuer` object:
      ```yaml
      apiVersion: cert-manager.io/v1
      kind: Issuer
      metadata:
        name: <acme-dns01-azuredns-issuer>
        namespace: <issuer_namespace>
      spec:
        acme:
          preferredChain: ""
          privateKeySecretRef:
            name: <secret_private_key>
          server: <server>
          solvers:
          - dns01:
              azureDNS:
                clientID: <azure_client_id>
                clientSecretSecretRef:
                  name: <secret_name>
                  key: <azure_secret_access_key_name>
                subscriptionID: <azure_subscription_id>
                tenantID: <azure_tenant_id>
                resourceGroupName: <azure_dns_zone_resource_group>
                hostedZoneName: <azure_dns_zone>
                environment: AzurePublicCloud
      ```

      where:

      <dl>
      <dt><code>&lt;acme-dns01-azuredns-issuer&gt;</code></dt>
      <dd>Specifies a name for the issuer.</dd>
      <dt><code>&lt;issuer_namespace&gt;</code></dt>
      <dd>Specifies your issuer namespace.</dd>
      <dt><code>&lt;secret_private_key&gt;</code></dt>
      <dd>Specifies the name of the secret to store the ACME account private key in.</dd>
      <dt><code>&lt;server&gt;</code></dt>
      <dd>Specifies the URL to access the ACME server’s <code>directory</code> endpoint. This example uses the <em>Let’s Encrypt</em> staging environment.</dd>
      <dt><code>&lt;azure_client_id&gt;</code></dt>
      <dd>Specifies your Azure client ID.</dd>
      <dt><code>&lt;secret_name&gt;</code></dt>
      <dd>Specifies a name of the client secret.</dd>
      <dt><code>&lt;azure_secret_access_key_name&gt;</code></dt>
      <dd>Specifies the client secret key name.</dd>
      <dt><code>&lt;azure_subscription_id&gt;</code></dt>
      <dd>Specifies your Azure subscription ID.</dd>
      <dt><code>&lt;azure_tenant_id&gt;</code></dt>
      <dd>Specifies your Azure tenant ID.</dd>
      <dt><code>&lt;azure_dns_zone_resource_group&gt;</code></dt>
      <dd>Specifies the name of the Azure DNS zone resource group.</dd>
      <dt><code>&lt;azure_dns_zone&gt;</code></dt>
      <dd>Specifies the name of Azure DNS zone.</dd>
      </dl>
   2. Create the `Issuer` object by running the following command:
      ```terminal
      $ oc create -f issuer.yaml
      ```

**Additional resources**

- [Azure DNS](https://cert-manager.io/docs/configuration/acme/dns01/azuredns/)
- [Google Cloud DNS](https://cert-manager.io/docs/configuration/acme/dns01/google/)
- [Configuring cloud credentials for the cert-manager Operator for Red Hat OpenShift for the AWS Security Token Service cluster](/docs/security/cert_manager_operator/cert-manager-authenticate#cert-manager-configure-cloud-credentials-aws-sts_cert-manager-authenticate)
- [Configuring cloud credentials for the cert-manager Operator for Red Hat OpenShift on AWS](/docs/security/cert_manager_operator/cert-manager-authenticate#cert-manager-configure-cloud-credentials-aws-non-sts_cert-manager-authenticate)
- [Configuring cloud credentials for the cert-manager Operator for Red Hat OpenShift with Google Cloud Workload Identity](/docs/security/cert_manager_operator/cert-manager-authenticate#cert-manager-configure-cloud-credentials-gcp-sts_cert-manager-authenticate)
- [Configuring cloud credentials for the cert-manager Operator for Red Hat OpenShift on Google Cloud](/docs/security/cert_manager_operator/cert-manager-authenticate#cert-manager-configure-cloud-credentials-gcp-non-sts_cert-manager-authenticate)
- [HTTP01](https://cert-manager.io/docs/configuration/acme/http01/)
- [HTTP-01 challenge](https://letsencrypt.org/docs/challenge-types/#http-01-challenge)
- [DNS01](https://cert-manager.io/docs/configuration/acme/dns01/)
