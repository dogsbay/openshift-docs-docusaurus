---
title: Managing user-owned OAuth access tokens
sidebar_position: 5
---

# Managing user-owned OAuth access tokens {#managing-oauth-access-tokens}

<a id="managing-oauth-access-tokens"></a>

Review and manage your user-owned OAuth access tokens to monitor active sessions, verify token scopes, and revoke tokens that are no longer needed.

## Listing user-owned OAuth access tokens {#oauth-list-tokens_managing-oauth-access-tokens}

List your user-owned OAuth access tokens to review active sessions, check token expiration, and identify tokens associated with specific OAuth clients.

Token names are not sensitive and cannot be used to log in.

**Procedure**

- List all user-owned OAuth access tokens:
  ```terminal
  $ oc get useroauthaccesstokens
  ```

  ```terminal title="Example output"
  NAME       CLIENT NAME                    CREATED                EXPIRES                         REDIRECT URI                                                       SCOPES
  <token1>   openshift-challenging-client   2021-01-11T19:25:35Z   2021-01-12 19:25:35 +0000 UTC   https://oauth-openshift.apps.example.com/oauth/token/implicit      user:full
  <token2>   openshift-browser-client       2021-01-11T19:27:06Z   2021-01-12 19:27:06 +0000 UTC   https://oauth-openshift.apps.example.com/oauth/token/display       user:full
  <token3>   console                        2021-01-11T19:26:29Z   2021-01-12 19:26:29 +0000 UTC   https://console-openshift-console.apps.example.com/auth/callback   user:full
  ```
- List user-owned OAuth access tokens for a particular OAuth client:
  ```terminal
  $ oc get useroauthaccesstokens --field-selector=clientName="console"
  ```

  ```terminal title="Example output"
  NAME       CLIENT NAME                    CREATED                EXPIRES                         REDIRECT URI                                                       SCOPES
  <token3>   console                        2021-01-11T19:26:29Z   2021-01-12 19:26:29 +0000 UTC   https://console-openshift-console.apps.example.com/auth/callback   user:full
  ```

## Viewing the details of a user-owned OAuth access token {#oauth-view-details-tokens_managing-oauth-access-tokens}

View details of a user-owned OAuth access token to identify the associated client application, check expiration and inactivity timeouts, verify scopes, and see other information fields.

**Procedure**

- Describe the details of a user-owned OAuth access token:
  ```terminal
  $ oc describe useroauthaccesstokens <token_name>
  ```

  ```terminal title="Example output"
  Name:                        <token_name>
  Namespace:
  Labels:                      <none>
  Annotations:                 <none>
  API Version:                 oauth.openshift.io/v1
  Authorize Token:             sha256~Ksckkug-9Fg_RWn_AUysPoIg-_HqmFI9zUL_CgD8wr8
  Client Name:                 openshift-browser-client
  Expires In:                  86400
  Inactivity Timeout Seconds:  317
  Kind:                        UserOAuthAccessToken
  Metadata:
    Creation Timestamp:  2021-01-11T19:27:06Z
    Managed Fields:
      API Version:  oauth.openshift.io/v1
      Fields Type:  FieldsV1
      fieldsV1:
        f:authorizeToken:
        f:clientName:
        f:expiresIn:
        f:redirectURI:
        f:scopes:
        f:userName:
        f:userUID:
      Manager:         oauth-server
      Operation:       Update
      Time:            2021-01-11T19:27:06Z
    Resource Version:  30535
    Self Link:         /apis/oauth.openshift.io/v1/useroauthaccesstokens/<token_name>
    UID:               f9d00b67-ab65-489b-8080-e427fa3c6181
  Redirect URI:        https://oauth-openshift.apps.example.com/oauth/token/display
  Scopes:
    user:full
  User Name:  <user_name>
  User UID:   82356ab0-95f9-4fb3-9bc0-10f1d6a6a345
  Events:     <none>
  ```

  where:

  <dl>
  <dt><code>Name</code></dt>
  <dd>Specifies the token name, which is the sha256 hash of the token. Token names are not sensitive and cannot be used to log in.</dd>
  <dt><code>Client Name</code></dt>
  <dd>Specifies the client name, which describes where the token originated from.</dd>
  <dt><code>Expires In</code></dt>
  <dd>Specifies the value in seconds from the creation time before this token expires.</dd>
  <dt><code>Inactivity Timeout Seconds</code></dt>
  <dd>If there is a token inactivity timeout set for the OAuth server, this specifies the value in seconds from the creation time before this token can no longer be used.</dd>
  <dt><code>Scopes</code></dt>
  <dd>Specifies the scopes for this token.</dd>
  <dt><code>User Name</code></dt>
  <dd>Specifies the user name associated with this token.</dd>
  </dl>

## Deleting user-owned OAuth access tokens {#oauth-delete-tokens_managing-oauth-access-tokens}

You can use the following procedure to delete any user-owned OAuth tokens that are no longer needed.

The `oc logout` command only invalidates the OAuth token for the active session. Deleting an OAuth access token logs out the user from all sessions that use the token.

**Procedure**

- Delete the user-owned OAuth access token:
  ```terminal
  $ oc delete useroauthaccesstokens <token_name>
  ```

  ```terminal title="Example output"
  useroauthaccesstoken.oauth.openshift.io "<token_name>" deleted
  ```

## Adding unauthenticated groups to cluster roles {#unauthenticated-users-cluster-role-bindings_managing-oauth-access-tokens}

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
