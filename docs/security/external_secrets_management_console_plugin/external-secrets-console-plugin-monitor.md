---
title: Monitoring the External Secrets Management Console Plug-in
sidebar_position: 4
---

# Monitoring the External Secrets Management Console Plug-in {#external-secrets-console-plugin-monitor}

<a id="external-secrets-console-plugin-monitor"></a>

Monitor certificates, issuers, external secrets, and secret stores from installed secrets management Operators using the External Secrets Management Console Plug-in in the OpenShift Container Platform web console.

:::warning

External Secrets Management Console Plug-in is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.

For more information about the support scope of Red Hat Technology Preview features, see [Technology Preview Features Support Scope](https://access.redhat.com/support/offerings/techpreview/).

:::

## Monitoring secrets management Operators by using External Secrets Management Console Plug-in {#external-secrets-console-plugin-monitor-custom-resources_external-secrets-console-plugin-monitor}

You can inspect custom resources for installed secrets management Operators in the OpenShift Container Platform web console.

**Prerequisites**

- You have installed the External Secrets Management Console Plug-in.
- You have installed at least one secrets management Operator.

**Procedure**

1. Log in to the OpenShift Container Platform web console.
2. Navigate to **Plugins** → **Secrets Management**.
3. Click ![](/images/kebab.png "Options menu") for the custom resource that you want to monitor, and then click **Inspect**.

**Verification**

- You must be able to view the following sections:
  - **Metadata**
  - **Labels**
  - **Annotations**
  - **Specifications**
  - **Status**
  - **Events**
