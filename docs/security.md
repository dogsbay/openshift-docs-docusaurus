---
title: OpenShift Container Platform security and compliance
sidebar_position: 1
---

# OpenShift Container Platform security and compliance {#security-compliance-overview}

<a id="security-compliance-overview"></a>

Review the security and compliance capabilities available in OpenShift Container Platform, and learn how to secure your cluster.

## Security overview {#security-overview_security-compliance-overview}

It is important to understand how to properly secure various aspects of your OpenShift Container Platform cluster.

### Container security {#container-security_security-compliance-overview}

A good starting point to understanding OpenShift Container Platform security is to review the concepts in [Understanding container security](/docs/security/container_security/security-understanding#security-understanding). This and subsequent sections provide a high-level walkthrough of the container security measures available in OpenShift Container Platform, including solutions for the host layer, the container and orchestration layer, and the build and application layer. These sections also include information on the following topics:

- Why container security is important and how it compares with existing security standards.
- Which container security measures are provided by the host (RHCOS and RHEL) layer and which are provided by OpenShift Container Platform.
- How to evaluate your container content and sources for vulnerabilities.
- How to design your build and deployment process to proactively check container content.
- How to control access to containers through authentication and authorization.
- How networking and attached storage are secured in OpenShift Container Platform.
- Containerized solutions for API management and SSO.

### Auditing {#auditing_security-compliance-overview}

OpenShift Container Platform auditing provides a security-relevant chronological set of records documenting the sequence of activities that have affected the system by individual users, administrators, or other components of the system. Administrators can [configure the audit log policy](/docs/security/audit-log-policy-config#audit-log-policy-config) and [view audit logs](/docs/security/audit-log-view#audit-log-view).

### Certificates {#certificates_security-compliance-overview}

Certificates are used by various components to validate access to the cluster. Administrators can [replace the default ingress certificate](/docs/security/certificates/replacing-default-ingress-certificate#replacing-default-ingress), [add API server certificates](/docs/security/certificates/api-server#api-server-certificates), or [add a service certificate](/docs/security/certificates/service-serving-certificate#add-service-serving).

You can also review more details about the types of certificates used by the cluster:

- [User-provided certificates for the API server](/docs/security/certificate_types_descriptions/user-provided-certificates-for-api-server#cert-types-user-provided-certificates-for-the-api-server)
- [Proxy certificates](/docs/security/certificate_types_descriptions/proxy-certificates#proxy-certificates)
- [Service CA certificates](/docs/security/certificate_types_descriptions/service-ca-certificates#cert-types-service-ca-certificates)
- [Node certificates](/docs/security/certificate_types_descriptions/node-certificates#cert-types-node-certificates)
- [Bootstrap certificates](/docs/security/certificate_types_descriptions/bootstrap-certificates#cert-types-bootstrap-certificates)
- [etcd certificates](/docs/security/certificate_types_descriptions/etcd-certificates#cert-types-etcd-certificates)
- [OLM certificates](/docs/security/certificate_types_descriptions/olm-certificates#cert-types-olm-certificates)
- [Aggregated API client certificates](/docs/security/certificate_types_descriptions/aggregated-api-client-certificates#cert-types-aggregated-api-client-certificates)
- [Machine Config Operator certificates](/docs/security/certificate_types_descriptions/machine-config-operator-certificates#cert-types-machine-config-operator-certificates)
- [User-provided certificates for default ingress](/docs/security/certificate_types_descriptions/user-provided-certificates-for-default-ingress#cert-types-user-provided-certificates-for-default-ingress)
- [Ingress certificates](/docs/security/certificate_types_descriptions/ingress-certificates#cert-types-ingress-certificates)
- [Monitoring and cluster logging Operator component certificates](/docs/security/certificate_types_descriptions/monitoring-and-cluster-logging-operator-component-certificates#cert-types-monitoring-and-cluster-logging-operator-component-certificates)
- [Control plane certificates](/docs/security/certificate_types_descriptions/control-plane-certificates#cert-types-control-plane-certificates)

### Encrypting data {#encrypting-data_security-compliance-overview}

You can [enable etcd encryption](/docs/etcd/etcd-encrypt#etcd-encrypt) for your cluster to provide an additional layer of data security. For example, it can help protect the loss of sensitive data if an etcd backup is exposed to the incorrect parties.

### Vulnerability scanning {#vulnerability-scanning_security-compliance-overview}

Administrators can use the Red Hat Quay Container Security Operator to run [vulnerability scans](/docs/security/pod-vulnerability-scan#pod-vulnerability-scan) and review information about detected vulnerabilities.

## Compliance overview {#compliance-overview_security-compliance-overview}

For many OpenShift Container Platform customers, regulatory readiness, or compliance, on some level is required before any systems can be put into production. That regulatory readiness can be imposed by national standards, industry standards, or the organization’s corporate governance framework.

### Compliance checking {#compliance-checking_security-compliance-overview}

Administrators can use the [Compliance Operator](/docs/security/compliance_operator/co-concepts/compliance-operator-understanding#understanding-compliance-operator) to run compliance scans and recommend remediations for any issues found. The [`oc-compliance` plugin](/docs/security/compliance_operator/co-scans/oc-compliance-plug-in-using#using-oc-compliance-plug-in) is an OpenShift CLI (`oc`) plugin that provides a set of utilities to easily interact with the Compliance Operator.

### File integrity checking {#file-integrity-checking_security-compliance-overview}

Administrators can use the [File Integrity Operator](/docs/security/file_integrity_operator/file-integrity-operator-understanding#understanding-file-integrity-operator) to continually run file integrity checks on cluster nodes and provide a log of files that have been modified.

**Additional resources**

- [Understanding authentication](/docs/authentication/understanding-authentication#understanding-authentication)
- [Configuring the internal OAuth server](/docs/authentication/configuring-internal-oauth#configuring-internal-oauth)
- [Understanding identity provider configuration](/docs/authentication/understanding-identity-provider#understanding-identity-provider)
- [Using RBAC to define and apply permissions](/docs/authentication/using-rbac#using-rbac)
- [Managing security context constraints](/docs/authentication/managing-security-context-constraints#managing-pod-security-policies)
