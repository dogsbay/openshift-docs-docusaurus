---
title: Deleting certificates and secrets with External Secrets Management Console Plug-in
sidebar_position: 6
---

# Deleting certificates and secrets with External Secrets Management Console Plug-in {#external-secrets-console-plugin-delete}

<a id="external-secrets-console-plugin-delete"></a>

Delete certificates and secrets from installed secrets management Operators using the External Secrets Management Console Plug-in in the OpenShift Container Platform web console.

:::warning

External Secrets Management Console Plug-in is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.

For more information about the support scope of Red Hat Technology Preview features, see [Technology Preview Features Support Scope](https://access.redhat.com/support/offerings/techpreview/).

:::

## Deleting certificates and secrets by using External Secrets Management Console Plug-in {#external-secrets-console-plugin-delete-certificates-secrets_external-secrets-console-plugin-delete}

You can delete certificates and secrets from installed secrets management Operators across your clusters by using the OpenShift Container Platform web console.

**Prerequisites**

- You have access to the OpenShift Container Platform cluster as a user with the `cluster-admin` role.
- You have installed the External Secrets Management Console Plug-in.
- You have installed at least one secrets management Operator.

**Procedure**

1. Log in to the OpenShift Container Platform web console.
2. Navigate to **Plugins** → **Secrets Management**.
3. Click ![](/images/kebab.png "Options menu") for the custom resource that you want to delete, and then click **Delete**.
4. Enter the name of the custom resource.
5. Click **Delete**.

**Verification**

1. Navigate to **Ecosystem** → **Installed Operators**.
2. Select the secrets management Operator whose custom resource you deleted.
3. Verify that the deleted custom resource no longer appears in the Operator details view.
