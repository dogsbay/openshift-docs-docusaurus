---
title: OLM certificates
sidebar_position: 7
---

# OLM certificates {#cert-types-olm-certificates}

<a id="cert-types-olm-certificates"></a>

Understand how Operator Lifecycle Manager (OLM) manages certificates for OLM components and creates and rotates certificates when installing Operators that include webhooks or API services. In proxy environments, you must manage Operator certificates yourself because OLM does not update them.

## Management {#_management}

All certificates for Operator Lifecycle Manager (OLM) components, such as `olm-operator`, `catalog-operator`, `packageserver`, and `marketplace-operator`, are managed by the system.

When installing Operators that include webhooks or API services in their `ClusterServiceVersion` (CSV) object, OLM creates and rotates the certificates for these resources. Certificates for resources in the `openshift-operator-lifecycle-manager` namespace are managed by OLM.

OLM does not update the certificates of Operators that it manages in proxy environments. These certificates must be managed by the user using the subscription config.

**Additional resources**

- [Configuring proxy support in Operator Lifecycle Manager](/docs/operators/admin/olm-configuring-proxy-support#olm-configuring-proxy-support)
- [Proxy certificates](/docs/security/certificate_types_descriptions/proxy-certificates#proxy-certificates)
- [Replacing the default ingress certificate](/docs/security/certificates/replacing-default-ingress-certificate#replacing-default-ingress)
- [Updating the CA bundle](/docs/security/certificates/updating-ca-bundle#updating-ca-bundle)
