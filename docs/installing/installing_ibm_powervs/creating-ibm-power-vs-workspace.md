---
title: Creating an IBM Power Virtual Server workspace
sidebar_position: 3
---

# Creating an IBM Power Virtual Server workspace {#creating-ibm-power-vs-workspace}

<a id="creating-ibm-power-vs-workspace"></a>

To host your OpenShift Container Platform cluster on IBM Power(R) Virtual Server, you can create a dedicated workspace and retrieve its identifier for use during installation.

## Creating an IBM Power Virtual Server workspace {#creating-ibm-power-vs-workspace-procedure_creating-ibm-power-vs-workspace}

To set up the infrastructure needed for your OpenShift Container Platform cluster, you can create an IBM Power(R) Virtual Server workspace and retrieve its GUID for use during installation.

**Procedure**

1. To create an IBM Power(R) Virtual Server workspace, complete step 1 to step 5 from the IBM Cloud(R) documentation for [Creating an IBM Power(R) Virtual Server](https://cloud.ibm.com/docs/power-iaas?topic=power-iaas-creating-power-virtual-server).
2. After it has finished provisioning, retrieve the 32-character alphanumeric Globally Unique Identifier (GUID) of your new workspace by entering the following command:
   ```terminal
   $ ibmcloud resource service-instance <workspace name>
   ```

**Additional resources**

- [Installing a cluster on IBM Power(R) Virtual Server with customizations](/docs/installing/installing_ibm_powervs/installing-ibm-power-vs-customizations#installing-ibm-power-vs-customizations)
