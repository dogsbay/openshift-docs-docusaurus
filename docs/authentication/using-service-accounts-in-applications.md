---
title: Using service accounts in applications
sidebar_position: 13
---

# Using service accounts in applications {#using-service-accounts}

<a id="using-service-accounts"></a>

## Service accounts overview {#service-accounts-overview_using-service-accounts}

You can use OpenShift Container Platform service accounts to allow a OpenShift Container Platform component to directly access the API.

Service accounts are API objects that exist within each project that provide a flexible way to control API access without sharing a regular user’s credentials.

When you use the OpenShift Container Platform CLI or web console, your API token authenticates you to the API. You can associate a component with a service account so that they can access the API without using a regular user’s credentials.

For example, service accounts can allow:

- Replication controllers to make API calls to create or delete pods
- Applications inside containers to make API calls for discovery purposes
- External applications to make API calls for monitoring or integration purposes

Each service account’s user name is derived from its project and name:

```text
system:serviceaccount:<project>:<name>
```

Every service account is also a member of two groups:

| Group | Description |
| --- | --- |
| system:serviceaccounts | Includes all service accounts in the system. |
| system:serviceaccounts:<project> | Includes all service accounts in the specified project. |

## Default service accounts {#service-accounts-default_using-service-accounts}

OpenShift Container Platform automatically creates default service accounts for cluster management and in each project for build, deployment, and pod operations.

### Default cluster service accounts {#default-cluster-service-accounts_using-service-accounts}

Several infrastructure controllers run using service account credentials. The following service accounts are created in the OpenShift Container Platform infrastructure project (`openshift-infra`) at server start, and given the following roles cluster-wide:

| Service account | Description |
| --- | --- |
| `replication-controller` | Assigned the `system:replication-controller` role |
| `deployment-controller` | Assigned the `system:deployment-controller` role |
| `build-controller` | Assigned the `system:build-controller` role. Additionally, the `build-controller` service account is included in the privileged security context constraint to create privileged build pods. |

### Default project service accounts and roles {#default-service-accounts-and-roles_using-service-accounts}

Three service accounts are automatically created in each project:

<table>
<thead>
<tr>
  <th>Service account</th>
  <th>Usage</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>builder</code></td>
  <td>Used by build pods. It is given the <code>system:image-builder</code> role, which allows pushing images to any imagestream in the project using the internal Docker registry.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>The <code>builder</code> service account is not created if the <code>Build</code> cluster capability is not enabled.</p></div></td>
</tr>
<tr>
  <td><code>deployer</code></td>
  <td>Used by deployment pods and given the <code>system:deployer</code> role, which allows viewing and modifying replication controllers and pods in the project.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>The <code>deployer</code> service account is not created if the <code>DeploymentConfig</code> cluster capability is not enabled.</p></div></td>
</tr>
<tr>
  <td><code>default</code></td>
  <td>Used to run all other pods unless they specify a different service account.<br><br><div class="alert alert--warning"><strong>Important</strong><p>Access rights and security privileges tied to the <code>default</code> service account apply to every pod in the project that does not specify a different service account. To implement the principle of least privilege and improve auditability, create dedicated service accounts for your workloads instead of using the <code>default</code> service account.<br><br>While most OpenShift Container Platform platform components and Operators use dedicated service accounts, the following dynamic tools continue to use the <code>default</code> service account to ensure operational efficiency:<br><br><ul><li><code>oc debug</code>: Uses the <code>default</code> service account to avoid the performance overhead of creating and removing unique service accounts for short-lived troubleshooting sessions.</li><li><code>oc adm must-gather</code>: Uses the <code>default</code> service account to collect diagnostic data across the cluster without requiring extensive manual RBAC modifications.</li></ul></p></div></td>
</tr>
</tbody>
</table>

All service accounts in a project are given the `system:image-puller` role, which allows pulling images from any image stream in the project using the internal container image registry.

### Automatically generated image pull secrets {#auto-generated-sa-token-secrets_using-service-accounts}

OpenShift Container Platform automatically creates image pull secrets for each service account to integrate the internal image registry with user authentication.

:::note

Prior to OpenShift Container Platform 4.16, a long-lived service account API token secret was also generated for each service account that was created. Starting with OpenShift Container Platform 4.16, this service account API token secret is no longer created.

After upgrading to 4.22, any existing long-lived service account API token secrets are not deleted and will continue to function. For information about detecting long-lived API tokens that are in use in your cluster or deleting them if they are not needed, see "Long-lived service account API tokens in OpenShift Container Platform (Red Hat Knowledgebase)".

:::

This image pull secret is necessary to integrate the OpenShift image registry into the cluster’s user authentication and authorization system.

However, if you do not enable the `ImageRegistry` capability or if you disable the integrated OpenShift image registry in the Cluster Image Registry Operator’s configuration, an image pull secret is not generated for each service account.

When the integrated OpenShift image registry is disabled on a cluster that previously had it enabled, the previously generated image pull secrets are deleted automatically.

## Create service accounts {#service-accounts-managing_using-service-accounts}

You can create a service account in a project and grant it permissions by binding it to a role.

**Procedure**

1. Optional: To view the service accounts in the current project:
   ```terminal
   $ oc get sa
   ```

   ```terminal title="Example output"
   NAME       SECRETS   AGE
   builder    1         2d
   default    1         2d
   deployer   1         2d
   ```
2. To create a new service account in the current project:
   ```terminal
   $ oc create sa <service_account_name>
   ```

   To create a service account in a different project, specify `-n <project_name>`.

   ```terminal title="Example output"
   serviceaccount "robot" created
   ```

   :::tip

   You can alternatively apply the following YAML to create the service account:

   ```yaml
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: <service_account_name>
     namespace: <current_project>
   ```

   :::
3. Optional: View the secrets for the service account:
   ```terminal
   $ oc describe sa robot
   ```

   ```terminal title="Example output"
   Name:                robot
   Namespace:           project1
   Labels:              <none>
   Annotations:         openshift.io/internal-registry-pull-secret-ref: robot-dockercfg-qzbhb
   Image pull secrets:  robot-dockercfg-qzbhb
   Mountable secrets:   robot-dockercfg-qzbhb
   Tokens:              <none>
   Events:              <none>
   ```
