---
title: Monitoring and OpenShift Logging Operator component certificates
sidebar_position: 12
---

# Monitoring and OpenShift Logging Operator component certificates {#cert-types-monitoring-and-cluster-logging-operator-component-certificates}

<a id="cert-types-monitoring-and-cluster-logging-operator-component-certificates"></a>

Review service certificate authority (CA) certificates for monitoring and Red Hat OpenShift Logging Operator components in OpenShift Container Platform, including validity, automatic rotation, and system-managed namespaces.

## Expiration {#monitor-logging-component-certificates-expiration_cert-types-monitoring-and-cluster-logging-operator-component-certificates}

Monitoring components secure their traffic with service CA certificates. These certificates are valid for 2 years and are replaced automatically on rotation of the service CA, which is every 13 months.

If the certificate is present in the `openshift-monitoring` or `openshift-logging` namespace, it is system managed and rotated automatically.

## Management {#monitor-logging-component-certificates-management_cert-types-monitoring-and-cluster-logging-operator-component-certificates}

These certificates are managed by the system and not the user.
