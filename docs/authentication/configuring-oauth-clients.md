---
title: Configuring OAuth clients
sidebar_position: 4
---

# Configuring OAuth clients

<a id="configuring-oauth-clients"></a>

OpenShift Container Platform includes default OAuth clients for platform authentication. You can register additional OAuth clients to integrate third-party applications and configure token inactivity timeouts to enhance security.

## Default OAuth clients {#oauth-default-clients_configuring-oauth-clients}

OpenShift Container Platform automatically creates OAuth clients for browser-based logins, CLI authentication, and challenge-based authentication when the API starts.

The following OAuth clients are created:

| OAuth client | Usage |
| --- | --- |
| `openshift-browser-client` | Requests tokens at `<namespace_route>/oauth/token/request` with a user-agent that can handle interactive logins. |
| `openshift-challenging-client` | Requests tokens with a user-agent that can handle `WWW-Authenticate` challenges. |
| `openshift-cli-client` | Requests tokens by using a local HTTP server fetching an authorization code grant. |

where:

`<namespace_route>`
: Specifies the namespace route. Find this value by running the following command:

  ```terminal
  $ oc get route oauth-openshift -n openshift-authentication -o json | jq .spec.host
  ```

## Registering an additional OAuth client {#oauth-register-additional-client_configuring-oauth-clients}

Register additional OAuth clients to manage authentication for applications that need to interact with your OpenShift Container Platform cluster.

**Procedure**

- To register additional OAuth clients:
  ```terminal
  $ oc create -f <(echo '
  kind: OAuthClient
  apiVersion: oauth.openshift.io/v1
  metadata:
   name: demo
  secret: "..."
  redirectURIs:
   - "http://www.example.com/"
  grantMethod: prompt
  ')
  ```

  where:

  <dl>
  <dt><code>metadata.name</code></dt>
  <dd>Specifies the OAuth client name. This value is used as the <code>client_id</code> parameter when making requests to <code>&lt;namespace_route&gt;/oauth/authorize</code> and <code>&lt;namespace_route&gt;/oauth/token</code>.</dd>
  <dt><code>secret</code></dt>
  <dd>Specifies the secret value used as the <code>client_secret</code> parameter when making requests to <code>&lt;namespace_route&gt;/oauth/token</code>.</dd>
  <dt><code>redirectURIs</code></dt>
  <dd>Specifies the list of valid redirect URIs. The <code>redirect_uri</code> parameter specified in requests to <code>&lt;namespace_route&gt;/oauth/authorize</code> and <code>&lt;namespace_route&gt;/oauth/token</code> must be equal to or prefixed by one of these URIs.</dd>
  <dt><code>grantMethod</code></dt>
  <dd>Specifies the action to take when this client requests tokens and has not yet been granted access by the user. Use <code>auto</code> to automatically approve the grant and retry the request, or <code>prompt</code> to prompt the user to approve or deny the grant.</dd>
  </dl>

## Configuring token inactivity timeout for an OAuth client {#oauth-token-inactivity-timeout_configuring-oauth-clients}

Configure OAuth clients to expire tokens after a set period of inactivity, improving security by automatically invalidating idle sessions.

By default, no token inactivity timeout is set.

:::note

If the token inactivity timeout is also configured in the internal OAuth server configuration, the timeout that is set in the OAuth client overrides that value.

:::

**Prerequisites**

- You have access to the cluster as a user with the `cluster-admin` role.
- You have configured an identity provider (IDP).

**Procedure**

- Update the `OAuthClient` configuration to set a token inactivity timeout.
  1. Edit the `OAuthClient` object:
     ```terminal
     $ oc edit oauthclient <oauth_client>
     ```

     Replace `<oauth_client>` with the OAuth client to configure, for example, `console`.

     Add the `accessTokenInactivityTimeoutSeconds` field and set your timeout value:

     ```yaml
     apiVersion: oauth.openshift.io/v1
     grantMethod: auto
     kind: OAuthClient
     metadata:
     ...
     accessTokenInactivityTimeoutSeconds: 600
     ```

     where:

     <dl>
     <dt><code>accessTokenInactivityTimeoutSeconds</code></dt>
     <dd>Specifies the token inactivity timeout in seconds. The minimum allowed value is <code>300</code>.</dd>
     </dl>
  2. Save the file to apply the changes.

**Verification**

1. Log in to the cluster with an identity from your IDP. Be sure to use the OAuth client that you just configured.
2. Perform an action and verify that it was successful.
3. Wait longer than the configured timeout without using the identity. In this procedure’s example, wait longer than 600 seconds.
4. Try to perform an action from the same identity’s session.
   This attempt should fail because the token should have expired due to inactivity longer than the configured timeout.

**Additional resources**

- [OAuthClient [oauth.openshift.io/v1](/docs/rest_api/oauth_apis/oauthclient-oauth-openshift-io-v1#oauthclient-oauth-openshift-io-v1)]
