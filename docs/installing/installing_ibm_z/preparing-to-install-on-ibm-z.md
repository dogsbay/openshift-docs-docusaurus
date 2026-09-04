---
title: Installation methods
sidebar_position: 1
---

# Installation methods {#preparing-to-install-on-ibm-z}

<a id="preparing-to-install-on-ibm-z"></a>

You can install an OpenShift Container Platform cluster on IBM Z(R) and IBM(R) LinuxONE by using a variety of different installation methods. Choose the method that best fits your environment, such as a disconnected deployment or a minimally configured installation.

:::note

While this document refers only to IBM Z(R), all information in it also applies to IBM(R) LinuxONE.

:::

## Choosing a method to install OpenShift Container Platform on IBM Z or IBM LinuxONE {#choosing-an-method-to-install-ocp-on-ibm-z_preparing-to-install-on-ibm-z}

OpenShift Container Platform supports many installation methods on IBM Z(R) and IBM(R) LinuxONE. The method you select depends on your network environment, the level of infrastructure control you require, and whether your deployment connects to the internet.

The OpenShift Container Platform installation program offers the following methods for deploying a cluster on IBM Z(R):

- **Interactive**: You can deploy a cluster with the web-based Assisted Installer. This method requires no setup for the installation program, and is ideal for connected environments such as IBM Z(R).
- **Local Agent-based**: You can deploy a cluster locally with the Agent-based Installer. It provides many of the benefits of the Assisted Installer, but you must download and configure the Agent-based Installer first. You complete the configuration with a command-line interface (CLI). This approach is ideal for disconnected networks.
- **Full control**: You can deploy a cluster on infrastructure that you prepare and support, which provides maximum customizability. You can deploy clusters in connected or disconnected environments.

**IBM Z(R) installation options**

|  | Assisted Installer | Agent-based Installer | User-provisioned installation | Installer-provisioned installation |
| --- | --- | --- | --- | --- |
| IBM Z(R) with z/VM | ✓ | ✓ | ✓ |  |
| Restricted network IBM Z(R) with z/VM |  | ✓ | ✓ |  |
| IBM Z(R) with RHEL KVM | ✓ | ✓ | ✓ |  |
| Restricted network IBM Z(R) with RHEL KVM |  | ✓ | ✓ |  |
| IBM Z(R) in an LPAR | ✓ | ✓ | ✓ |  |
| Restricted network IBM Z(R) in an LPAR |  | ✓ | ✓ |  |

For more information about the installation process, see Installation process in the Additional resources section.

## User-provisioned infrastructure installation of OpenShift Container Platform on IBM Z {#ibm-z-upi-installation-overview_preparing-to-install-on-ibm-z}

User-provisioned infrastructure requires you to provision and manage all resources that OpenShift Container Platform needs, including networking, load balancing, storage, and compute. This approach suits organizations that have specific infrastructure requirements or that operate in air-gapped or restricted networks.

:::warning

These steps for performing a user-provisioned infrastructure installation are an example only. Installing a cluster with infrastructure you offer requires knowledge of the IBM Z(R) platform and the installation process of OpenShift Container Platform. Use the user-provisioned infrastructure installation instructions as a guide; you are free to create the required resources through other methods.

:::

- **Installing a cluster with z/VM on IBM Z(R) and IBM(R) LinuxONE**: You can install OpenShift Container Platform with z/VM on IBM Z(R) or IBM(R) LinuxONE infrastructure that you provision.
- **Installing a cluster with z/VM on IBM Z and IBM LinuxONE in a disconnected environment**: You can install OpenShift Container Platform with z/VM on IBM Z(R) or IBM(R) LinuxONE infrastructure that you provision in a restricted or disconnected network by using an internal mirror of the installation release content. You can use this method to install a cluster that does not require an active internet connection to obtain the software components. You can also use this installation method to ensure that your clusters only use container images that satisfy your organizational controls on external content.
- **Installing a cluster with RHEL KVM on IBM Z(R) and IBM(R) LinuxONE**: You can install OpenShift Container Platform with KVM on IBM Z(R) or IBM(R) LinuxONE infrastructure that you provision.
- **Installing a cluster with RHEL KVM on IBM Z(R) and IBM(R) LinuxONE in a disconnected environment**: You can install OpenShift Container Platform with RHEL KVM on IBM Z(R) or IBM(R) LinuxONE infrastructure that you provision in a restricted or disconnected network by using an internal mirror of the installation release content. You can use this method to install a cluster that does not require an active internet connection to obtain the software components. You can also use this installation method to ensure that your clusters only use container images that satisfy your organizational controls on external content.
- **Installing a cluster in an LPAR on IBM Z(R) and IBM(R) LinuxONE**: You can install OpenShift Container Platform in a logical partition (LPAR) on IBM Z(R) or IBM(R) LinuxONE infrastructure that you provision.
- **Installing a cluster in an LPAR on IBM Z(R) and IBM(R) LinuxONE in a disconnected environment**: You can install OpenShift Container Platform in an LPAR on IBM Z(R) or IBM(R) LinuxONE infrastructure that you provision in a restricted or disconnected network by using an internal mirror of the installation release content. You can use this method to install a cluster that does not require an active internet connection to obtain the software components. You can also use this installation method to ensure that your clusters only use container images that satisfy your organizational controls on external content.

**Additional resources**

- [Assisted Installer](https://access.redhat.com/documentation/en-us/assisted_installer_for_openshift_container_platform)
- [Preparing to install with the Agent-based Installer](/docs/installing/installing_with_agent_based_installer/preparing-to-install-with-agent-based-installer#preparing-to-install-with-agent-based-installer)
- [Agent-based Installer for IBM Z(R)](https://console.redhat.com/openshift/install/ibmz/agent-based)
- [Installation process](/docs/architecture/architecture-installation#installation-process_architecture-installation)
- [Installing a cluster with z/VM on IBM Z(R) and IBM(R) LinuxONE](/docs/installing/installing_ibm_z/upi/installing-ibm-z#installing-ibm-z)
- [Installing a cluster with z/VM on IBM Z and IBM LinuxONE in a disconnected environment](/docs/installing/installing_ibm_z/upi/installing-restricted-networks-ibm-z#installing-restricted-networks-ibm-z)
- [Installing a cluster with RHEL KVM on IBM Z(R) and IBM(R) LinuxONE](/docs/installing/installing_ibm_z/upi/installing-ibm-z-kvm#installing-ibm-z-kvm)
- [Installing a cluster with RHEL KVM on IBM Z(R) and IBM(R) LinuxONE in a disconnected environment](/docs/installing/installing_ibm_z/upi/installing-restricted-networks-ibm-z-kvm#installing-restricted-networks-ibm-z-kvm)
- [Installing a cluster in an LPAR on IBM Z(R) and IBM(R) LinuxONE](/docs/installing/installing_ibm_z/upi/installing-ibm-z-lpar#installing-ibm-z-lpar)
- [Installing a cluster in an LPAR on IBM Z(R) and IBM(R) LinuxONE in a disconnected environment](/docs/installing/installing_ibm_z/upi/installing-restricted-networks-ibm-z-lpar#installing-restricted-networks-ibm-z-lpar)
