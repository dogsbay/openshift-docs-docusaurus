---
title: Configuring a Google identity provider
sidebar_position: 8
---

# Configuring a Google identity provider {#configuring-google-identity-provider}

<a id="configuring-google-identity-provider"></a>

Configure a Google identity provider so users can authenticate to OpenShift Container Platform with Google accounts. When configured, sign-in is permitted only for Google accounts in that hosted domain.

## Identity providers in OpenShift Container Platform {#identity-provider-overview_configuring-google-identity-provider}

You can configure identity providers by creating a custom resource (CR) that describes the provider and adding it to the cluster. Identity providers enable user authentication in OpenShift Container Platform beyond the default `kubeadmin` user.

:::note

OpenShift Container Platform usernames containing `/`, `:`, and `%` are not supported.

:::

## Google authentication {#identity-provider-google-about_configuring-google-identity-provider}

By using Google as an identity provider, you can authenticate to your server. You can use the `hostedDomain` configuration attribute to limit authentication to members of a specific hosted domain.

Google authentication uses OpenID Connect through the cluster OAuth server.

:::note

Using Google as an identity provider requires users to get a token using `<namespace_route>/oauth/token/request` to use with command-line tools.

:::

## Create the secret {#identity-provider-creating-secret_configuring-google-identity-provider}

Create a `Secret` object in the `openshift-config` namespace to store the client secret for your identity provider. The identity provider custom resource (CR) references this secret during configuration.

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

## Sample Google custom resource {#identity-provider-google-CR_configuring-google-identity-provider}

Review the custom resource (CR) fields and acceptable values for configuring a Google identity provider in OpenShift Container Platform. Use these definitions to set client credentials and hosted domain restrictions before applying the configuration to the cluster.

```yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: googleidp
    mappingMethod: claim
    type: Google
    google:
      clientID: {...}
      clientSecret:
        name: google-secret
      hostedDomain: "example.com"
```

where:

`spec.identityProviders.name`
: Specifies the provider name, which is prefixed to the Google numeric user ID to form an identity name. The provider name is also used to build the redirect URL.

`spec.identityProviders.mappingMethod`
: Specifies how mappings are established between identities from this provider and `User` objects.

`spec.identityProviders.google.clientID`
: Specifies the client ID from the Google Cloud project where you create the OAuth client. The project must be configured with a redirect URI of `https://oauth-openshift.apps.<cluster-name>.<cluster-domain>/oauth2callback/<idp-provider-name>`.

`spec.identityProviders.google.clientSecret`
: Specifies a reference to an OpenShift Container Platform `Secret` object containing the client secret issued by Google.

`spec.identityProviders.google.hostedDomain`
: Specifies a hosted domain used to restrict sign-in accounts. Optional if the `lookup` `mappingMethod` is used. If empty, any Google account is allowed to authenticate.

**Additional resources**

- [Identity provider parameters](/docs/authentication/understanding-identity-provider#identity-provider-parameters_understanding-identity-provider)

## Add an identity provider to your cluster {#add-identity-provider_configuring-google-identity-provider}

Apply the identity provider custom resource (CR) to your cluster after you define it. With this configuration, you can authenticate with the configured identity provider.

**Prerequisites**

- You have access to a OpenShift Container Platform cluster.
- You have created the CR for your identity providers.
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
3. Log in to the cluster by running the following command, passing in the token to authenticate:
   ```terminal
   $ oc login --token=<token>
   ```

   :::note

   This identity provider does not support logging in with a username and password.

   :::
4. Confirm that the user logged in successfully and that the username displays by running the following command:
   ```terminal
   $ oc whoami
   ```

**Additional resources**

- [OpenID Connect (Google Identity documentation)](https://developers.google.com/identity/protocols/OpenIDConnect)
