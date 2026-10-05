---
title: About disconnected environments
sidebar_position: 1
---

# About disconnected environments

<a id="about-disconnected-environments"></a>

A disconnected environment is an environment that does not have full access to the internet.

OpenShift Container Platform is designed to perform many automatic functions that depend on an internet connection, such as retrieving release images from a registry or retrieving update paths and recommendations for the cluster. Without a direct internet connection, you must perform additional setup and configuration for your cluster to maintain full functionality in the disconnected environment.

## Glossary of disconnected environment terms {#glossary-disconnected_about-disconnected-environments}

Although it is used throughout the OpenShift Container Platform documentation, *disconnected environment* is a broad term that can refer to environments with various levels of internet connectivity.

Other terms are sometimes used to refer to a specific level of internet connectivity, and these environments might require additional unique configurations.

The following table describes the different terms used to refer to environments without a full internet connection:

**Disconnected environment terms**

| Term | Description |
| --- | --- |
| Air-gapped network | An environment or network that is completely isolated from an external network.<br>This isolation depends on a physical separation, or an "air gap", between machines on the internal network and any other part of an external network. Air-gapped environments are often used in industries with strict security or regulatory requirements. |
| Disconnected environment | An environment or network that has some level of isolation from an external network.<br>This isolation could be enabled by physical or logical separation between machines on the internal network and an external network. Regardless of the level of isolation from the external network, a cluster in a disconnected environment does not have access to public services hosted by Red Hat and requires additional setup to maintain full cluster functionality. |
| Restricted Network | An environment or network with limited connection to an external network.<br>A physical connection may exist between machines on the internal network and an external network, but network traffic is limited by additional configurations, such as with firewalls and proxies. |

## Preferred methods for working with disconnected environments {#preferred-methods_about-disconnected-environments}

You can choose between multiple options for most aspects of managing a cluster in a disconnected environment. For example, when mirroring images you can choose between using the oc-mirror OpenShift CLI (`oc`) plugin or using the `oc adm` command.

However, some options provide a simpler and more convenient user experience for disconnected environments, and are the preferred method over their alternatives.

Unless your organizational needs require you to choose another option, use the following methods for mirroring images, installing your cluster, and updating your cluster:

- Mirror your images using the oc-mirror plugin v2. For more information, see "Mirroring images for a disconnected installation by using the oc-mirror plugin v2".
- Install your cluster using the Agent-based Installer. For more information, see "Installing a cluster with customizations".
- Update your cluster using a local OpenShift Update Service instance. For more information, see "Updating a cluster in a disconnected environment using the OpenShift Update Service".

**Additional resources**

- [Mirroring images for a disconnected installation by using the oc-mirror plugin v2](/docs/disconnected/about-installing-oc-mirror-v2#about-installing-oc-mirror-v2)
- [Installing a cluster with customizations](/docs/installing/installing_with_agent_based_installer/installing-with-agent-based-installer#installing-with-agent-based-installer)
- [Updating a cluster in a disconnected environment using the OpenShift Update Service](/docs/disconnected/updating/disconnected-update-osus#updating-disconnected-cluster-osus)
