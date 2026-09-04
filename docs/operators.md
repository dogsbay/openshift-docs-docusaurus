---
title: Operators overview
sidebar_position: 1
---

# Operators overview

<a id="operators-overview"></a>

Operators are the foundational control plane extensions of OpenShift Container Platform. Operators are the preferred method to package, deploy, and manage services on the control plane, and to support your applications.

Operators integrate with Kubernetes APIs and CLI tools such as `kubectl` and the OpenShift CLI (`oc`). They provide the means of monitoring applications, performing health checks, managing over-the-air (OTA) updates, and ensuring that applications remain in your specified state.

Operators are designed specifically for Kubernetes-native applications to implement and automate common Day 1 operations, such as installation and configuration. Operators can also automate Day 2 operations, such as autoscaling up or down and creating backups. All of these activities are directed by a piece of software running on your cluster.

While both follow similar Operator concepts and goals, Operators in OpenShift Container Platform are managed by two different systems, depending on their purpose:

Cluster Operators
: Managed by the Cluster Version Operator (CVO) and installed by default to perform cluster functions.

Optional add-on Operators
: Managed by Operator Lifecycle Manager (OLM) and can be made accessible for users to run in their applications. Also known as *OLM-based Operators*.

## OLM-based Operator tasks {#operators-overview-tasks_operators-overview}

To maintain cluster security and operational boundaries, your assigned user role, such as cluster administrator or application developer, determines whether you can install, manage, or use Operator Lifecycle Manager (OLM)-based Operators.

For developers
: As an Operator author, you can perform the following development tasks for OLM-based Operators:

  - Install and subscribe an Operator to your namespace.
  - Create an application from an installed Operator through the web console.

For administrators
: As a cluster administrator, you can perform the following administrative tasks for OLM-based Operators:

  - Manage custom catalogs.
  - Allow non-cluster administrators to install Operators
  - Install an Operator from the software catalog
  - View Operator status
  - Manage Operator conditions
  - Upgrade installed Operators
  - Delete installed Operators
  - Configure proxy support
  - Using Operator Lifecycle Manager in disconnected environments

For information about the cluster Operators that Red Hat provides, see "Cluster Operators reference".

**Additional resources**

- [Manage custom catalogs](/docs/operators/admin/olm-managing-custom-catalogs#olm-managing-custom-catalogs)
- [Machine deletion lifecycle hook examples for Operator developers](/docs/machine_management/deleting-machine#machine-lifecycle-hook-deletion-uses_deleting-machine)
- [Install and subscribe an Operator to your namespace](/docs/operators/user/olm-installing-operators-in-namespace#olm-installing-operators-in-namespace)
- [Allow non-cluster administrators to install Operators](/docs/operators/admin/olm-creating-policy#olm-creating-policy)
- [Install an Operator from the software catalog](/docs/operators/user/olm-installing-operators-in-namespace#olm-installing-operators-in-namespace)
- [Using Operator Lifecycle Manager in disconnected environments](/docs/disconnected/using-olm#olm-restricted-networks)
- [Cluster Operators reference](/docs/operators/operator-reference#operator-reference)
- [Create an application from an installed Operator through the web console](/docs/operators/user/olm-creating-apps-from-installed-operators#olm-creating-apps-from-installed-operators)
- [View Operator status](/docs/operators/admin/olm-status#olm-status)
- [Manage Operator conditions](/docs/operators/admin/olm-managing-operatorconditions#olm-managing-operatorconditions)
- [Upgrade installed Operators](/docs/operators/admin/olm-upgrading-operators#olm-upgrading-operators)
- [Delete installed Operators](/docs/operators/admin/olm-deleting-operators-from-cluster#olm-deleting-operators-from-a-cluster)
- [Configure proxy support](/docs/operators/admin/olm-configuring-proxy-support#olm-configuring-proxy-support)
- [What are Operators?](/docs/operators/understanding/olm-what-operators-are#olm-what-operators-are)
