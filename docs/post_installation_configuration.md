---
title: Postinstallation configuration overview
sidebar_position: 1
---

# Postinstallation configuration overview {#post-install-configuration-overview}

<a id="post-install-configuration-overview"></a>

After installing OpenShift Container Platform, you can configure machines, cluster services, nodes, networking, storage, users, and alert notifications to meet your operational requirements.

After installing OpenShift Container Platform, a cluster administrator can configure and customize the following components:

- Machine
- Bare metal
- Cluster
- Node
- Network
- Storage
- Users
- Alerts and notifications

## Postinstallation configuration tasks {#post-install-tasks_post-installation-configuration-overview}

You can perform the postinstallation configuration tasks to configure your environment to meet your needs.

The following lists details these configurations:

- Configure operating system features: The Machine Config Operator (MCO) manages `MachineConfig` objects. By using the MCO, you can configure nodes and custom resources.
- Configure cluster features. You can modify the following features of an OpenShift Container Platform cluster:
  - Image registry
  - Networking configuration
  - Image build behavior
  - Identity provider
  - The etcd configuration
  - Machine set creation to handle the workloads
  - Cloud provider credential management
- Configuring a private cluster: By default, the installation program provisions OpenShift Container Platform by using a publicly accessible DNS and endpoints. To make your cluster accessible only from within an internal network, configure the following components to make them private:
  - DNS
  - Ingress Controller
  - API server
- Perform node operations: By default, OpenShift Container Platform uses Red Hat Enterprise Linux CoreOS (RHCOS) compute machines. You can perform the following node operations:
  - Add and remove compute machines.
  - Add and remove taints and tolerations.
  - Configure the maximum number of pods per node.
  - Enable Device Manager.
- Configure users: Users can authenticate themselves to the API by using OAuth access tokens. You can configure OAuth to perform the following tasks:
  - Specify an identity provider.
  - Use role-based access control to define and grant permissions to users.
  - Install an Operator from the software catalog.
- Configuring alert notifications: By default, firing alerts are displayed on the Alerting UI of the web console. You can also configure OpenShift Container Platform to send alert notifications to external systems.

**Additional resources**

- [Configure operating system features](/docs/machine_configuration#machine-config-overview)
- [Configure cluster features](/docs/post_installation_configuration/cluster-tasks#post-install-cluster-tasks)
- [Configuring a private cluster](/docs/post_installation_configuration/configuring-private-cluster#configuring-private-cluster)
- [Perform node operations](/docs/post_installation_configuration/node-tasks#post-install-node-tasks)
- [Configure users](/docs/post_installation_configuration/preparing-for-users#post-install-preparing-for-users)
- [Configuring alert notifications](/docs/post_installation_configuration/configuring-alert-notifications#configuring-alert-notifications)
