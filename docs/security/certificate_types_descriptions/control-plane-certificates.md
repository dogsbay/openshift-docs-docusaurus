---
title: Control plane certificates
sidebar_position: 13
---

# Control plane certificates {#cert-types-control-plane-certificates}

<a id="cert-types-control-plane-certificates"></a>

Review control plane certificate namespaces and automatic rotation in OpenShift Container Platform to plan maintenance and recover from expiration.

## Location {#control-plane-certs-location_cert-types-control-plane-certificates}

Control plane certificates are included in these namespaces:

- `openshift-config-managed`
- `openshift-kube-apiserver`
- `openshift-kube-apiserver-operator`
- `openshift-kube-controller-manager`
- `openshift-kube-controller-manager-operator`
- `openshift-kube-scheduler`

## Management {#control-plane-certs-management_cert-types-control-plane-certificates}

Control plane certificates are managed by the system and rotated automatically.

If control plane certificates expire, see "Recovering from expired control plane certificates".

**Additional resources**

- [Recovering from expired control plane certificates](/docs/backup_and_restore/control_plane_backup_and_restore/disaster_recovery/scenario-3-expired-certs#dr-recovering-expired-certs)
