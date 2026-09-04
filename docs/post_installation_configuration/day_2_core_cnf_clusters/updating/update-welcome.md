---
title: Upgrading an OpenShift Container Platform cluster
sidebar_position: 1
---

# Upgrading an OpenShift Container Platform cluster {#update-welcome}

<a id="update-welcome"></a>

OpenShift Container Platform has long term support or extended update support (EUS) on all even releases and update paths between EUS releases. You can update from one EUS version to the next EUS version. It is also possible to update between y-stream and z-stream versions.

## Cluster updates for OpenShift clusters {#update-introduction_update-welcome}

Updating your cluster is a critical task that ensures that bugs and potential security vulnerabilities are patched. Often, updates to cloud-native applications require additional functionality from the platform that comes when you update the cluster version. You also must update the cluster periodically to ensure that the cluster platform version is supported.

You can minimize the effort required to stay current with updates by keeping up-to-date with EUS releases and upgrading to select important z-stream releases only.

:::note

The update path for the cluster can vary depending on the size and topology of the cluster.

:::

The following update scenarios are described:

- Control Plane Only updates
- Y-stream updates
- Z-stream updates

:::warning

Control Plane Only updates were previously known as EUS-to-EUS updates. Control Plane Only updates are only viable between even-numbered minor versions of OpenShift Container Platform.

:::
