---
title: About cluster updates in a disconnected environment
sidebar_position: 1
---

# About cluster updates in a disconnected environment {#about-disconnected-updates}

<a id="about-disconnected-updates"></a>

You can update a OpenShift Container Platform cluster in a disconnected environment where the cluster nodes cannot access the internet or where you want to manage update recommendations and release images locally for policy or performance purposes.

## Mirroring OpenShift Container Platform images {#about-disconnected-updates-mirroring_about-disconnected-updates}

You can mirror OpenShift Container Platform images to a local container image registry to provide your disconnected cluster with the resources necessary for targeted updates. A single container image registry is sufficient to host mirrored images for several clusters in the disconnected network.

For more information about mirroring images onto a repository in your disconnected cluster, see the "Mirroring OpenShift Container Platform images" section.

**Additional resources**

- [Mirroring OpenShift Container Platform images](/docs/disconnected/updating/mirroring-image-repository#mirroring-ocp-image-repository)

## Performing a cluster update in a disconnected environment {#about-disconnected-updates-update_about-disconnected-updates}

You can keep your disconnected OpenShift Container Platform environment up to date by performing a cluster update. This process can be managed locally either with or without the OpenShift Update Service (OSUS).

For more information about performing a cluster update in a disconnected environment with OSUS, see the "Updating a cluster in a disconnected environment using the OpenShift Update Service" section.

For more information about performing a cluster update in a disconnected environment without OSUS, see the "Updating a cluster in a disconnected environment without the OpenShift Update Service" section.

**Additional resources**

- [Updating a cluster in a disconnected environment using the OpenShift Update Service](/docs/disconnected/updating/disconnected-update-osus#updating-disconnected-cluster-osus)
- [Updating a cluster in a disconnected environment without the OpenShift Update Service](/docs/disconnected/updating/disconnected-update#updating-disconnected-cluster)

## Uninstalling the OpenShift Update Service from a cluster {#about-disconnected-updates-uninstalling-osus_about-disconnected-updates}

You can uninstall a local copy of the OpenShift Update Service (OSUS) from your OpenShift Container Platform cluster when you no longer need to manage updates locally:

For more information about uninstalling the OpenShift Update Service from a cluster, see the "Uninstalling the OpenShift Update Service from a cluster" section.

**Additional resources**

- [Uninstalling the OpenShift Update Service from a cluster](/docs/disconnected/updating/uninstalling-osus#uninstalling-osus)
