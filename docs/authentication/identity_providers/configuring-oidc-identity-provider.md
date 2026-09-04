---
title: Configuring an OpenID Connect identity provider
sidebar_position: 9
---

# Configuring an OpenID Connect identity provider {#configuring-oidc-identity-provider}

<a id="configuring-oidc-identity-provider"></a>

Configure the `oidc` identity provider to integrate with an OpenID Connect identity provider using an [Authorization Code Flow](http://openid.net/specs/openid-connect-core-1_0.html#CodeFlowAuth).

## Identity providers in OpenShift Container Platform {#identity-provider-overview_configuring-oidc-identity-provider}

You can configure identity providers by creating a custom resource (CR) that describes the provider and adding it to the cluster. Identity providers enable user authentication in OpenShift Container Platform beyond the default `kubeadmin` user.

:::note

OpenShift Container Platform usernames containing `/`, `:`, and `%` are not supported.

:::

## About OpenID Connect authentication {#identity-provider-oidc-about_configuring-oidc-identity-provider}

The Authentication Operator in OpenShift Container Platform requires that the configured OpenID Connect identity provider implements the [OpenID Connect Discovery](https://openid.net/specs/openid-connect-discovery-1_0.html) specification.

:::note

`ID Token` and `UserInfo` decryptions are not supported.

:::

By default, the `openid` scope is requested. If required, extra scopes can be specified in the `extraScopes` field.

Claims are read from the JWT `id_token` returned from the OpenID identity provider and, if specified, from the JSON returned by the `UserInfo` URL.

At least one claim must be configured to use as the user’s identity. The standard identity claim is `sub`.

You can also indicate which claims to use as the user’s preferred user name, display name, and email address. If multiple claims are specified, the first one with a non-empty value is used. The following table lists the standard claims:

| Claim | Description |
| --- | --- |
| `sub` | Short for "subject identifier." The remote identity for the user at the issuer. |
| `preferred_username` | The preferred user name when provisioning a user. A shorthand name that the user wants to be referred to as, such as `janedoe`. Typically a value that corresponding to the user’s login or username in the authentication system, such as username or email. |
| `email` | Email address. |
| `name` | Display name. |

See the [OpenID claims documentation](http://openid.net/specs/openid-connect-core-1_0.html#StandardClaims) for more information.

:::note

Unless your OpenID Connect identity provider supports the resource owner password credentials (ROPC) grant flow, users must get a token from `<namespace_route>/oauth/token/request` to use with command-line tools.

:::

## Supported OIDC providers {#identity-provider-oidc-supported_configuring-oidc-identity-provider}

Red Hat tests and supports specific OpenID Connect (OIDC) providers with OpenShift Container Platform. The following OpenID Connect (OIDC) providers are tested and supported with OpenShift Container Platform. Using an OIDC provider that is not on the following list might work with OpenShift Container Platform, but the provider was not tested by Red Hat and therefore is not supported by Red Hat.

- Active Directory Federation Services for Windows Server
  :::note

  Currently, it is not supported to use Active Directory Federation Services for Windows Server with OpenShift Container Platform when custom claims are used.

  :::
- GitLab
- Google
- Keycloak
- Microsoft Entra ID
  :::note

  Currently, it is not supported to use Microsoft Entra ID when group names are required to be synced.

  :::
- Okta
- Ping Identity
- Red Hat Single Sign-On

## Creating the secret {#identity-provider-creating-secret_configuring-oidc-identity-provider}

Create a `Secret` object in the `openshift-config` namespace to store the client secret and related credentials for the identity provider configuration.

**Procedure**

1. Create a `Secret` object containing the client secret by running the following command:
   ```terminal
   $ oc create secret generic <secret_name> --from-literal=clientSecret=<secret> -n openshift-config
   ```
2. Optional: Apply the following YAML to create the secret:
   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: <secret_name>
     namespace: openshift-config
   type: Opaque
   data:
     clientSecret: <base64_encoded_client_secret>
   ```
3. Create a `Secret` object from a file by running the following command:
   ```terminal
   $ oc create secret generic <secret_name> --from-file=<path_to_file> -n openshift-config
   ```

## Creating a 'ConfigMap' {#identity-provider-creating-configmap_configuring-oidc-identity-provider}

Create a `ConfigMap` object in the `openshift-config` namespace to store the certificate authority bundle that identity providers use to validate secure connections to the remote authentication service.

**Procedure**

1. Define an OpenShift Container Platform `ConfigMap` object containing the certificate authority by running the following command:
   ```terminal
   $ oc create configmap ca-config-map --from-file=ca.crt=/path/to/ca -n openshift-config
   ```
2. Optional: Apply the following YAML to create the config map:
   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: ca-config-map
     namespace: openshift-config
   data:
     ca.crt: |
       <CA_certificate_PEM>
   ```

   The certificate authority must be stored in the `ca.crt` key of the `ConfigMap` object.

## Sample OpenID Connect CRs {#identity-provider-oidc-CR_configuring-oidc-identity-provider}

The following custom resources (CRs) show the parameters and acceptable values for an OpenID Connect identity provider.

If you must specify a custom certificate bundle, extra scopes, extra authorization request parameters, or a `userInfo` URL, use the full OpenID Connect CR.

```yaml title="Standard OpenID Connect CR"
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: oidcidp (1)
    mappingMethod: claim (2)
    type: OpenID
    openID:
      clientID: ... (3)
      clientSecret: (4)
        name: idp-secret
      claims: (5)
        preferredUsername:
        - preferred_username
        name:
        - name
        email:
        - email
        groups:
        - groups
      issuer: https://www.idp-issuer.com (6)
```

1. This provider name is prefixed to the value of the identity claim to form an identity name. It is also used to build the redirect URL.
2. Controls how mappings are established between this provider’s identities and `User` objects.
3. The client ID of a client registered with the OpenID provider. The client must be allowed to redirect to `https://oauth-openshift.apps.<cluster_name>.<cluster_domain>/oauth2callback/<idp_provider_name>`.
4. A reference to an OpenShift Container Platform `Secret` object containing the client secret.
5. The list of claims to use as the identity. The first non-empty claim is used.
6. The [Issuer Identifier](https://openid.net/specs/openid-connect-core-1_0.html#IssuerIdentifier) described in the OpenID spec. Must use `https` without query or fragment component.

```yaml title="Full OpenID Connect CR"
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: oidcidp
    mappingMethod: claim
    type: OpenID
    openID:
      clientID: ...
      clientSecret:
        name: idp-secret
      ca: (1)
        name: ca-config-map
      extraScopes: (2)
      - email
      - profile
      extraAuthorizeParameters: (3)
        include_granted_scopes: "true"
      claims:
        preferredUsername: (4)
        - preferred_username
        - email
        name: (5)
        - nickname
        - given_name
        - name
        email: (6)
        - custom_email_claim
        - email
        groups: (7)
        - groups
      issuer: https://www.idp-issuer.com
```

1. Optional: Reference to an OpenShift Container Platform config map containing the PEM-encoded certificate authority bundle to use in validating server certificates for the configured URL.
2. Optional: The list of scopes to request, in addition to the `openid` scope, during the authorization token request.
3. Optional: A map of extra parameters to add to the authorization token request.
4. The list of claims to use as the preferred user name when provisioning a user for this identity. The first non-empty claim is used.
5. The list of claims to use as the display name. The first non-empty claim is used.
6. The list of claims to use as the email address. The first non-empty claim is used.
7. The list of claims to use to synchronize groups from the OpenID Connect provider to OpenShift Container Platform upon user login. The first non-empty claim is used.

**Additional resources**

- See [Identity provider parameters](/docs/authentication/understanding-identity-provider#identity-provider-parameters_understanding-identity-provider) for information on parameters, such as `mappingMethod`, that are common to all identity providers.

## Adding an identity provider to your cluster {#add-identity-provider_configuring-oidc-identity-provider}

Apply the identity provider custom resource (CR) to your cluster so users can authenticate with the configured identity provider.

**Prerequisites**

- You installed an OpenShift Container Platform cluster.
- You defined the CR for your identity provider.
- You are logged in as an administrator.

**Procedure**

1. Apply the defined CR by running the following command:
   ```terminal
   $ oc apply -f </path/to/CR>
   ```

   :::note

   If a CR does not exist, `oc apply` creates a new CR and might trigger the following warning: `Warning: oc apply should be used on resources created by either oc create --save-config or oc apply`. In this case you can safely ignore this warning.

   :::
2. Obtain a token from the OAuth server.
   As long as the `kubeadmin` user has been removed, the `oc login` command provides instructions on how to access a web page where you can retrieve the token.

   You can also access this page from the web console by navigating to **(?) Help** → **Command Line Tools** → **Copy Login Command**.
3. Log in to the cluster, passing in the token to authenticate, by running the following command:
   ```terminal
   $ oc login --token=<token>
   ```

   :::note

   After the OIDC identity provider is configured in OpenShift Container Platform, you can also log in by running the following command, which prompts for your username and password:

   ```terminal
   $ oc login -u <identity_provider_username> --server=<api_server_url_and_port>
   ```

   If your OpenID Connect identity provider supports the resource owner password credentials (ROPC) grant flow, you might need to take steps to enable the ROPC grant flow for your identity provider.

   :::
4. Confirm that the user logged in successfully and that the username displays by running the following command:
   ```terminal
   $ oc whoami
   ```

## Configuring identity providers using the web console {#identity-provider-configuring-using-the-web-console_configuring-oidc-identity-provider}

Configure your identity provider (IDP) through the web console instead of the CLI.

**Prerequisites**

- You must be logged in to the web console as a cluster administrator.

**Procedure**

1. Navigate to **Administration** → **Cluster Settings**.
2. Under the **Configuration** tab, click **OAuth**.
3. Under the **Identity Providers** section, select your identity provider from the **Add** drop-down menu.

:::note

You can specify multiple IDPs through the web console without overwriting existing IDPs.

:::
