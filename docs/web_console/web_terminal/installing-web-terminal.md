---
title: Installing the web terminal
sidebar_position: 1
---

# Installing the web terminal {#installing-web-terminal}

<a id="installing-web-terminal"></a>

You can install the web terminal by using the Web Terminal Operator listed in the OpenShift Container Platform software catalog. When you install the Web Terminal Operator, the custom resource definitions (CRDs) that are required for the command line configuration, such as the `DevWorkspace` CRD, are automatically installed. The web console creates the required resources when you open the web terminal.

## Prerequisites {#prerequisites_installing-web-terminal}

- You are logged into the OpenShift Container Platform web console.
- You have cluster administrator permissions.

## Procedure {#installing-web-terminal-procedure}

1. In the **Administrator** perspective of the web console, navigate to **Ecosystem** → **Software Catalog**.
2. Use the **Filter by keyword** box to search for the Web Terminal Operator in the catalog, and then click the **Web Terminal** tile.
3. Read the brief description about the Operator on the **Web Terminal**  page, and then click **Install**.
4. On the **Install Operator** page, retain the default values for all fields.
   - The **fast** option in the **Update Channel** menu enables installation of the latest release of the Web Terminal Operator.
   - The **All namespaces on the cluster** option in the **Installation Mode** menu  enables the Operator to watch and be available to all namespaces in the cluster.
   - The **openshift-operators** option in the **Installed Namespace** menu installs the Operator in the default `openshift-operators` namespace.
   - The **Automatic** option in the **Approval Strategy** menu ensures that the future upgrades to the Operator are handled automatically by the Operator Lifecycle Manager.
5. Click **Install**.
6. In the **Installed Operators** page, click the **View Operator** to verify that the Operator is listed on the **Installed Operators** page.
   :::note

   The Web Terminal Operator installs the DevWorkspace Operator as a dependency.

   :::
7. After the Operator is installed, refresh your page to see the command-line terminal icon (![odc-wto-icon](/images/odc-wto-icon.png "web terminal icon")) in the masthead of the console.
