---
title: Installing the CLI Manager Operator
sidebar_position: 3
---

# Installing the CLI Manager Operator {#cli-manager-install}

<a id="cli-manager-install"></a>

You can simplify the installation and management of CLI plugins in connected and disconnected environments with the CLI Manager Operator. The CLI Manager Operator makes Krew compatible with the `oc` CLI, allowing cluster administrators to manage custom CLI plugin resources.

:::warning

Using the CLI Manager Operator to install and manage plugins for the OpenShift CLI is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.

For more information about the support scope of Red Hat Technology Preview features, see [Technology Preview Features Support Scope](https://access.redhat.com/support/offerings/techpreview/).

:::

## Installing the CLI Manager Operator {#cli-manager-installing_cli-manager-install}

You can install the CLI Manager Operator to facilitate adding CLI plugins in both connected and disconnected environments.

:::note

Krew always works with OpenShift CLI (`oc`) without the CLI Manager Operator installed. You can use the same commands outlined in this documentation to use Krew with `oc`. For more information, see [Krew documentation](https://krew.sigs.k8s.io/docs/).

:::

**Prerequisites**

- [Krew is installed](https://krew.sigs.k8s.io/docs/user-guide/setup/install).
- You are logged in to OpenShift Container Platform as a user with the `cluster-admin` role.
- You have access to the OpenShift Container Platform web console.

**Procedure**

1. Log in to the OpenShift Container Platform web console.
2. Create the required namespace for the CLI Manager Operator:
   1. Navigate to **Administration** → **Namespaces** and click **Create Namespace**.
   2. In the **Name** field, enter `openshift-cli-manager-operator` and click **Create**.
3. Install the CLI Manager Operator:
   1. Navigate to **Ecosystem** → **Software Catalog**.
   2. In the filter box, enter **CLI Manager Operator**.
   3. Select the **CLI Manager Operator** and click **Install**.
   4. On the **Install Operator** page, complete the following steps:
      1. Ensure that the **Update channel** is set to **tech preview**, which installs the latest Technology Preview release of the CLI Manager Operator.
      2. From the drop-down menu, select **A specific namespace on the cluster** and select **openshift-cli-manager-operator**.
      3. Click **Install**.
4. Create the `CliManager` resource by completing the following steps:
   1. Navigate to **Installed Operators**.
   2. Select **CLI Manager Operator**.
   3. Select the **CLI Manager** tab.
   4. Click **Create CliManager**.
   5. Use the default **Name**.
   6. Click **Create**.
      1. The new `CliManager` resource is listed in the **CLI Manager** tab.

**Verification**

1. Navigate to **Ecosystem** → **Installed Operators**.
2. Verify that **CLI Manager Operator** is listed with a **Status** of **Succeeded**.

## Adding the CLI Manager Operator custom index to Krew {#cli-manager-custom-index_cli-manager-install}

You can use the terminal to add the CLI Manager Operator custom index to Krew so that the CLI Manager Operator will work in disconnected environments. This procedure is required for the CLI Manager Operator to function correctly and needs to be done only once.

:::note

If you use self-signed certificates, mark the certificate as trusted on your local operating system to use Krew.

:::

**Prerequisites**

- [Krew is installed](https://krew.sigs.k8s.io/docs/user-guide/setup/install).
- The CLI Manager Operator is installed.

**Procedure**

1. To establish the `ROUTE` variable, enter the following command:
   ```terminal
   $ ROUTE=$(oc get route/openshift-cli-manager -n openshift-cli-manager-operator -o=jsonpath='{.spec.host}')
   ```
2. To add the custom index to Krew, enter the following command:
   ```terminal
   $ oc krew index add <custom_index_name> https://$ROUTE/cli-manager
   ```
3. To update Krew, enter the following command and check for any errors:
   ```terminal
   $ oc krew update
   ```

   ```terminal title="Example output"
   Updated the local copy of plugin index.
   Updated the local copy of plugin index <custom_index_name>.
   New plugins available:
   * ocp/<plugin_name>
   ```

## Adding a plugin to the CLI Manager Operator {#cli-manager-adding-plugin-yamls_cli-manager-install}

You can add a CLI plugin to the CLI Manager Operator by creating a new plugin resource in the OpenShift Container Platform web console’s YAML view.

**Prerequisites**

- You are logged in to OpenShift Container Platform as a user with the `cluster-admin` role.
- The CLI Manager Operator is installed.

**Procedure**

1. Log in to the OpenShift Container Platform web console.
2. Navigate to **Ecosystem** → **Installed Operators**.
3. From the list, select **CLI Manager Operator**.
4. Select the **CLI Plugin** tab.
5. Click **Create Plugin**.
6. In the text box, enter the information for the plugin you are installing. See the following example YAML file.
   ```yaml title="Example YAML file to add a plugin"
   apiVersion: config.openshift.io/v1alpha1
   kind: Plugin
   metadata:
     name: <plugin_name>
   spec:
     description: <description_of_plugin>
     homepage: <plugin_homepage>
     platforms:
     - bin:
       files:
       - from: <plugin_file_path>
         to: .
       image: <plugin_image>
       imagePullSecret:
       platform: <platform>
     shortDescription: <short_description_of_plugin>
     version: <version>
   ```

   where:

   <dl>
   <dt><code>&lt;plugin_name&gt;</code></dt>
   <dd>Specifies the name of the plugin you plan to use in commands.</dd>
   <dt><code>bin</code></dt>
   <dd>Specifies the path to the plugin executable.</dd>
   <dt><code>imagePullSecret</code></dt>
   <dd>Optional field if the registry is not public to add a pull secret to access your plugin image.</dd>
   <dt><code>&lt;platform&gt;</code></dt>
   <dd>Add the architecture for your system; for example, <code>linux/amd64</code>, <code>darwin/arm64</code>, <code>windows/amd64</code>, or another architecture.</dd>
   <dt><code>&lt;version&gt;</code></dt>
   <dd>The version must be in v0.0.0 format.</dd>
   </dl>
7. Click **Save**.

**Verification**

- Enter the following command to see if the plugin is listed and has been added successfully:

```terminal
$ oc get plugin/<plugin_name> -o yaml
```

- Example output:

```terminal
<plugin_name> ready to be served.
```
