---
title: User-provided certificates for the API server
sidebar_position: 1
---

# User-provided certificates for the API server {#cert-types-user-provided-certificates-for-the-api-server}

<a id="cert-types-user-provided-certificates-for-the-api-server"></a>

Review user-provided TLS certificates for the API server in OpenShift Container Platform to understand their purpose, location, management, and expiration for external client access.

## Purpose {#user-provided-certificates-for-api-server-purpose_cert-types-user-provided-certificates-for-the-api-server}

The API server is accessible by clients external to the cluster at `api.<cluster_name>.<base_domain>`. You might want clients to access the API server at a different hostname or without the need to distribute the cluster-managed certificate authority (CA) certificates to the clients. The administrator must set a custom default certificate to be used by the API server when serving content.

## Location {#user-provided-certificates-for-api-server-location_cert-types-user-provided-certificates-for-the-api-server}

The user-provided certificates must be provided in a `kubernetes.io/tls` type `Secret` in the `openshift-config` namespace. Update the API server cluster configuration, the `apiserver/cluster` resource, to enable the use of the user-provided certificate.

## Management {#user-provided-certificates-for-api-server-management_cert-types-user-provided-certificates-for-the-api-server}

User-provided certificates are managed by the user.

## Expiration {#user-provided-certificates-for-api-server-expiration_cert-types-user-provided-certificates-for-the-api-server}

API server client certificate expiration is less than five minutes.

## Customization {#user-provided-certificates-for-api-server-customization_cert-types-user-provided-certificates-for-the-api-server}

Update the secret containing the user-managed certificate as needed.

**Additional resources**

- [Adding API server certificates](/docs/security/certificates/api-server#api-server-certificates)
