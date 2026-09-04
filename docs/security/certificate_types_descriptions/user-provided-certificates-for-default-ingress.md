---
title: User-provided certificates for default ingress
sidebar_position: 10
---

# User-provided certificates for default ingress {#cert-types-user-provided-certificates-for-default-ingress}

<a id="cert-types-user-provided-certificates-for-default-ingress"></a>

Review user-provided ingress certificates in OpenShift Container Platform, including transport layer security (TLS) secret storage, `IngressController` references, and replacing Operator-generated defaults.

Use user-provided certificates for the default `IngressController` CR to complete the following tasks:

- Replace Operator-generated default certificates before production use.
- Store TLS secrets in the correct namespace.
- Reference the secret in the `IngressController` CR.

## User-provided certificates for default ingress reference {#user-provided-certificates-ingress-ref_cert-types-user-provided-certificates-for-default-ingress}

Use user-provided default ingress certificates to allow applications on the default apps domain to present a custom TLS certificate to clients, so clients do not need to have cluster-managed certificate authority (CA) certificates installed.

### Purpose {#cert-types-user-provided-default-ingress-purpose_cert-types-user-provided-certificates-for-default-ingress}

Applications are usually exposed at `<route_name>.apps.<cluster_name>.<base_domain>`. The `<cluster_name>` and `<base_domain>` come from the installation config file. `<route_name>` is the host field of the route, if specified, or the route name. For example, `hello-openshift-default.apps.username.devcluster.openshift.com`. `hello-openshift` is the name of the route and the route is in the default namespace. You might want clients to access the applications without distributing cluster-managed CA certificates to the clients. Cluster administrators must set a custom default certificate when serving application content.

:::warning

The Ingress Operator generates a default certificate for an `IngressController` CR to serve as a placeholder until you configure a custom default certificate. Do not use Operator-generated default certificates in production clusters.

:::

### Location {#cert-types-user-provided-default-ingress-location_cert-types-user-provided-certificates-for-default-ingress}

Store user-provided certificates in a `tls` type `Secret` resource in the `openshift-ingress` namespace. Update the `IngressController` CR in the `openshift-ingress-operator` namespace to enable the use of the user-provided certificate. For more information, see "Setting a custom default certificate".

### Management {#cert-types-user-provided-default-ingress-management_cert-types-user-provided-certificates-for-default-ingress}

User-provided certificates are managed by the user.

### Expiration {#cert-types-user-provided-default-ingress-expiration_cert-types-user-provided-certificates-for-default-ingress}

Expiration and renewal are managed by the user.

### Services {#cert-types-user-provided-default-ingress-services_cert-types-user-provided-certificates-for-default-ingress}

Applications deployed on the cluster use user-provided certificates for default ingress.

### Customization {#cert-types-user-provided-default-ingress-customization_cert-types-user-provided-certificates-for-default-ingress}

Update the secret containing the user-managed certificate as needed.

**Additional resources**

- [Replacing the default ingress certificate](/docs/security/certificates/replacing-default-ingress-certificate#replacing-default-ingress)
- [Setting a custom default certificate](/docs/networking/networking_operators/ingress-operator#nw-ingress-setting-a-custom-default-certificate_configuring-ingress)
