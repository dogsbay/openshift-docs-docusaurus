---
title: Installation methods
sidebar_position: 1
---

# Installation methods {#preparing-to-install-on-azure-stack-hub}

<a id="preparing-to-install-on-azure-stack-hub"></a>

You can install OpenShift Container Platform on Azure Stack Hub using installer-provisioned or user-provisioned infrastructure.

The default installation type uses installer-provisioned infrastructure, where the installation program provisions the underlying infrastructure for the cluster. You can also install OpenShift Container Platform on infrastructure that you provision. If you do not use infrastructure that the installation program provisions, you must manage and maintain the cluster resources yourself.

See "Installation process" for more information about installer-provisioned and user-provisioned installation processes.

## Installing a cluster on installer-provisioned infrastructure {#choosing-a-method-to-install-ocp-on-ash-installer-provisioned}

You can install a cluster on Azure Stack Hub infrastructure that is provisioned by the OpenShift Container Platform installation program, by using the following method:

- Installing a cluster: You can install OpenShift Container Platform on Azure Stack Hub infrastructure that is provisioned by the OpenShift Container Platform installation program. See "Installing a cluster".

## Installing a cluster on user-provisioned infrastructure {#choosing-a-method-to-install-ocp-on-ash-user-provisioned}

You can install a cluster on Azure Stack Hub infrastructure that you provision, by using the following method:

- Installing a cluster on Azure Stack Hub using ARM templates: You can install OpenShift Container Platform on Azure Stack Hub by using infrastructure that you provide. You can use the provided Azure Resource Manager (ARM) templates to assist with an installation. See "Installing a cluster on Azure Stack Hub using ARM templates".

**Additional resources**

- [Installation process](/docs/architecture/architecture-installation#installation-process_architecture-installation)
- [Installing a cluster](/docs/installing/installing_azure_stack_hub/ipi/installing-azure-stack-hub-default#installing-azure-stack-hub-default)
- [Installing a cluster on Azure Stack Hub using ARM templates](/docs/installing/installing_azure_stack_hub/upi/installing-azure-stack-hub-user-infra#installing-azure-stack-hub-user-infra)
- [Configuring an Azure Stack Hub account](/docs/installing/installing_azure_stack_hub/installing-azure-stack-hub-account#installing-azure-stack-hub-account)
