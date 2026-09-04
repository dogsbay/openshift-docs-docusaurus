---
title: Installing a cluster on vSphere using the Agent-based Installer
sidebar_position: 5
---

# Installing a cluster on vSphere using the Agent-based Installer {#installing-vsphere-agent-based-installer}

<a id="installing-vsphere-agent-based-installer"></a>

The Agent-based installation method provides the flexibility to boot your on-premise servers in any way that you choose. It combines the ease of use of the Assisted Installation service with the ability to run offline, including in air-gapped environments.

Agent-based installation is a subcommand of the OpenShift Container Platform installer. It generates a bootable ISO image containing all of the information required to deploy an OpenShift Container Platform cluster with an available release image.

:::warning

Your vSphere account must include privileges for reading and creating the resources required to install an OpenShift Container Platform cluster.

:::

**Additional resources**

- [Preparing to install with the Agent-based Installer](/docs/installing/installing_with_agent_based_installer/preparing-to-install-with-agent-based-installer#preparing-to-install-with-agent-based-installer)
- [vCenter requirements](/docs/installing/installing_vsphere/upi/upi-vsphere-installation-reqs#installation-vsphere-installer-infra-requirements_upi-vsphere-installation-reqs)
