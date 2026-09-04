---
title: Configuring the automatic image cleanup of the container storage disk
sidebar_position: 6
---

# Configuring the automatic image cleanup of the container storage disk {#cnf-image-based-upgrade-configure-auto-image-cleanup}

<a id="cnf-image-based-upgrade-configure-auto-image-cleanup"></a>

Configure when the Lifecycle Agent cleans up unpinned images in the `Prep` stage by setting a minimum threshold for available storage space through annotations. The default container storage disk usage threshold is 50%.

The Lifecycle Agent does not delete images that are pinned in CRI-O or are currently used. The Operator selects the images for deletion by starting with dangling images and then sorting the images from oldest to newest that is determined by the image `Created` timestamp.

## Configuring the automatic image cleanup of the container storage disk {#ztp-image-based-upgrade-configure-threshold_auto-cleanup}

Configure the minimum threshold for available storage space through annotations.

**Prerequisites**

- You have created an `ImageBasedUpgrade` CR.

**Procedure**

1. Increase the threshold to 65% by running the following command:
   ```terminal
   $ oc -n openshift-lifecycle-agent annotate ibu upgrade image-cleanup.lca.openshift.io/disk-usage-threshold-percent='65'
   ```
2. (Optional) Remove the threshold override by running the following command:
   ```terminal
   $ oc -n  openshift-lifecycle-agent annotate ibu upgrade image-cleanup.lca.openshift.io/disk-usage-threshold-percent-
   ```

## Disable the automatic image cleanup of the container storage disk {#ztp-image-based-upgrade-disable-container-storage-image-cleanup_auto-cleanup}

Disable the automatic image cleanup threshold.

**Procedure**

1. Disable the automatic image cleanup by running the following command:
   ```terminal
   $ oc -n openshift-lifecycle-agent annotate ibu upgrade image-cleanup.lca.openshift.io/on-prep='Disabled'
   ```
2. (Optional) Enable automatic image cleanup again by running the following command:
   ```terminal
   $ oc -n  openshift-lifecycle-agent annotate ibu upgrade image-cleanup.lca.openshift.io/on-prep-
   ```
