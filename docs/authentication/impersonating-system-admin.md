---
title: "Impersonating the system:admin user"
sidebar_position: 19
---

# Impersonating the system:admin user {#impersonating-system-admin}

<a id="impersonating-system-admin"></a>

You can configure API requests to impersonate users or groups to test permissions and troubleshoot access issues in OpenShift Container Platform.

## API impersonation {#authentication-api-impersonation_impersonating-system-admin}

You can configure API requests in OpenShift Container Platform to act as another user. Impersonation allows you to perform actions on behalf of another account without switching credentials.

## Impersonate the system:admin user {#impersonation-system-admin-user_impersonating-system-admin}

You can use the OpenShift web console to impersonate a user and select multiple group memberships at the same time to reproduce that user’s effective permissions.

**Procedure**

- To grant a user permission to impersonate `system:admin`, run the following command:
  ```terminal
  $ oc create clusterrolebinding <any_valid_name> --clusterrole=sudoer --user=<username>
  ```

  :::tip

  You can alternatively apply the following YAML to grant permission to impersonate `system:admin`:

  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: <any_valid_name>
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: sudoer
  subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: <username>
  ```

  :::

## Impersonate the system:admin group {#impersonation-system-admin-group_impersonating-system-admin}

To impersonate a user who has cluster administration privileges through group membership, you must specify both the user and the associated groups in the impersonation command.

**Procedure**

- To grant a user permission to impersonate a `system:admin` by impersonating the associated cluster administration groups, run the following command:
  ```terminal
  $ oc create clusterrolebinding <any_valid_name> --clusterrole=sudoer --as=<user> \
  --as-group=<group1> --as-group=<group2>
  ```

## Impersonate a user with multiple group memberships in the web console {#impersonating-user-multiple-group-memberships-web-console_impersonating-system-admin}

You can start user impersonation from multiple locations in the OpenShift Container Platform Console. Depending on where you start, you can impersonate a single user, a single group, or a user with one or more group memberships.

**Prerequisites**

- You must be logged in to the OpenShift Container Platform web console as a user with permission to impersonate other users.
- The user or group that you want to impersonate must already exist.

:::note

The impersonated user can belong to zero or more groups.

:::

**Procedure**

1. From the **Overview** page in the OpenShift Container Platform console, click your user name and select **Impersonate User**.
2. In the **Username** field in the **Impersonate** dialog, enter the name of the user you want to impersonate.
3. Optional: In the **Groups** field, choose one or more groups that are associated with the user.
   The dialog displays a warning message explaining that impersonation applies the effective permissions of the specified user and any selected groups.
4. Click **Impersonate** to impersonate your selected user, groups, or both.

:::note

Selecting one group uses the existing single-group impersonation behavior. Selecting no groups uses regular single-user impersonation.

:::

## Start impersonation from the Users or Groups pages {#starting-impersonation-users-groups-pages_impersonating-system-admin}

You can start impersonation for users or groups from the **Users** or **Groups** pages in the OpenShift Container Platform Console.

**Procedure**

1. From the **Overview** page in the OpenShift Container Platform console, click **User Management** → **Users**.
2. Open the menu for the user you want to impersonate and select **Impersonate User**.
3. Optional: To impersonate a group, click **User Management** → **Groups**, click the menu for that group, and select **Impersonate Group**.

## Stop impersonation {#stopping-impersonation_impersonating-system-admin}

You can stop impersonating a user or group at any time from the OpenShift Container Platform Console.

**Procedure**

1. On any page in the OpenShift Container Platform console, click **Stop impersonating** at the top of the page.
2. Alternatively, click your user name and select **Stop impersonating**.

## Add unauthenticated groups to cluster roles {#unauthenticated-users-cluster-role-bindings_impersonating-system-admin}

Grant unauthenticated users access to specific cluster roles to enable features that require cluster access without authentication, such as external webhooks or automated token management.

You can add unauthenticated users to the following cluster roles:

- `system:scope-impersonation`
- `system:webhook`
- `system:oauth-token-deleter`
- `self-access-reviewer`

:::warning

Always verify compliance with your organization’s security standards when modifying unauthenticated access.

:::

**Prerequisites**

- You have access to the cluster as a user with the `cluster-admin` role.
- You have installed the OpenShift CLI (`oc`).

**Procedure**

1. Create a YAML file named `add-<cluster_role>-unauth.yaml` and add the following content:
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRoleBinding
   metadata:
    annotations:
      rbac.authorization.kubernetes.io/autoupdate: "true"
    name: <cluster_role>access-unauthenticated
   roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: <cluster_role>
   subjects:
    - apiGroup: rbac.authorization.k8s.io
      kind: Group
      name: system:unauthenticated
   ```
2. Apply the configuration by running the following command:
   ```terminal
   $ oc apply -f add-<cluster_role>.yaml
   ```

**Additional resources**

- [User impersonation (Kubernetes documentation)](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#user-impersonation)
