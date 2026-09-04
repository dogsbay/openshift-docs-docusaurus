---
title: Installation methods
sidebar_position: 1
---

# Installation methods {#preparing-to-install-on-azure}

<a id="preparing-to-install-on-azure"></a>

When you plan an OpenShift Container Platform deployment on Microsoft Azure, you can select installer-provisioned or user-provisioned infrastructure. Compare installation methods to find the path that matches your network, security, and operational requirements.

The default installation type uses installer-provisioned infrastructure, where the installation program provisions the underlying infrastructure for the cluster.

You can also install OpenShift Container Platform on infrastructure that you provision. If you do not use infrastructure that the installation program provisions, you must manage and support the cluster resources yourself.

## Installing a cluster on installer-provisioned infrastructure {#_installing_a_cluster_on_installer-provisioned_infrastructure}

You can install a cluster on Microsoft Azure infrastructure that is provisioned by the OpenShift Container Platform installation program, by using one of the following methods:

- You can install OpenShift Container Platform on Azure infrastructure that the installation program provisions and use default configuration options for a quick deployment. For more information, see "Installing a cluster quickly on Azure".
- You can install a customized cluster on Azure infrastructure that the installation program provisions. The installation program supports some customization during installation, and many other options are available postinstallation. For more information, see "Installing a customized cluster on Azure".
- You can customize your OpenShift Container Platform configuration during installation so that your cluster coexists with existing IP address allocations and meets customized network requirements. For more information, see "Installing a cluster on Azure with customizations".
- You can install a cluster on Azure in a restricted network by creating an internal mirror of the installation release content on an existing Azure Virtual Network (`VNet`). For more information, see "Installing a cluster on Azure in a restricted network".
- You can install OpenShift Container Platform on an existing Azure `VNet` when company guidelines limit new accounts or infrastructure. For more information, see "Installing a cluster on Azure into an existing `VNet`".
- You can install a private cluster into an existing Azure `VNet` on Azure and deploy OpenShift Container Platform on an internal network that is not visible to the internet. For more information, see "Installing a private cluster on Azure".
- You can deploy OpenShift Container Platform into Microsoft Azure Government (MAG) regions for US government agencies, contractors, educational institutions, and other US customers that must run sensitive workloads on Azure. For more information, see "Installing a cluster on Azure into a government region".

**Additional resources**

- [Installing a cluster quickly on Azure](/docs/installing/installing_azure/ipi/installing-azure-default#installing-azure-default)
- [Installing a cluster on Azure with customizations](/docs/installing/installing_azure/ipi/installing-azure-customizations#installing-azure-customizations)
- [Installing a cluster on Azure in a restricted network](/docs/installing/installing_azure/ipi/installing-restricted-networks-azure-installer-provisioned#installing-restricted-networks-azure-installer-provisioned)
- [Installing a cluster on Azure into an existing `VNet`](/docs/installing/installing_azure/ipi/installing-azure-vnet#installing-azure-vnet)
- [Installing a private cluster on Azure](/docs/installing/installing_azure/ipi/installing-azure-private#installing-azure-private)
- [Installing a cluster on Azure into a government region](/docs/installing/installing_azure/ipi/installing-azure-government-region#installing-azure-government-region)

## Installing a cluster on user-provisioned infrastructure {#choosing-an-method-to-install-ocp-on-azure-user-provisioned}

You can install a cluster on Azure infrastructure that you provision, by using one of the following methods:

- You can install a cluster on Azure in a restricted network with user-provisioned infrastructure when you do not require an active internet connection to obtain software components. For more information, see "Installing a cluster on Azure in a restricted network with user-provisioned infrastructure".
- You can install OpenShift Container Platform on Azure by using infrastructure that you manage and Azure Resource Manager (ARM) templates to assist with the installation. For more information, see "Installing a cluster on Azure using ARM templates".

**Additional resources**

- [Installing a cluster on Azure in a restricted network with user-provisioned infrastructure](/docs/installing/installing_azure/upi/installing-restricted-networks-azure-user-provisioned#installing-restricted-networks-azure-user-provisioned)
- [Installing a cluster on Azure using ARM templates](/docs/installing/installing_azure/upi/installing-azure-user-infra#installing-azure-user-infra)

**Additional resources**

- [Configuring an Azure account](/docs/installing/installing_azure/installing-azure-account#installing-azure-account)
- [Installation process](/docs/architecture/architecture-installation#installation-process_architecture-installation)
