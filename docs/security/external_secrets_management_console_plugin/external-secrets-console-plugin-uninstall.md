---
title: Uninstalling the External Secrets Management Console Plug-in
sidebar_position: 5
---

# Uninstalling the External Secrets Management Console Plug-in {#external-secrets-console-plugin-uninstall}

<a id="external-secrets-console-plugin-uninstall"></a>

Uninstall the External Secrets Management Console Plug-in from your OpenShift Container Platform cluster using the web console **Installed Operators** page.

:::warning

External Secrets Management Console Plug-in is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.

For more information about the support scope of Red Hat Technology Preview features, see [Technology Preview Features Support Scope](https://access.redhat.com/support/offerings/techpreview/).

:::

## Uninstalling the External Secrets Management Console Plug-in {#external-secrets-console-plugin-uninstall-web-console_external-secrets-console-plugin-uninstall}

You can uninstall the External Secrets Management Console Plug-in from your cluster using the OpenShift Container Platform web console.

**Prerequisites**

- You have access to the OpenShift Container Platform web console.
- The External Secrets Management Console Plug-in is installed.

**Procedure**

1. Log in to the OpenShift Container Platform web console.
2. Navigate to **Ecosystem** → **Installed Operators**.
3. Click the Options menu ![](/images/kebab.png "Options menu") next to the **External Secrets Management Console** entry, and then click **Uninstall Operator**.
4. In the confirmation dialog, select the **Delete all operand instances for this operator** checkbox and then click **Uninstall**.

**Verification**

- Verify that **Secrets Management** no longer appears under **Plugins**.
