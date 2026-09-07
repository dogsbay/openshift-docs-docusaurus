---
title: Configuring an htpasswd identity provider
sidebar_position: 1
---

# Configuring an htpasswd identity provider {#configuring-htpasswd-identity-provider}

<a id="configuring-htpasswd-identity-provider"></a>

Configure the `htpasswd` identity provider so users can log in to OpenShift Container Platform with credentials from an `htpasswd` file.

To define an `htpasswd` identity provider, complete these tasks:

1. Create an `htpasswd` file to store the user and password information.
2. Create a secret to represent the `htpasswd` file.
3. Define an `htpasswd` identity provider resource that references the secret.
4. Apply the resource to the default OAuth configuration to add the identity provider.

## Identity providers in OpenShift Container Platform {#identity-provider-overview_configuring-htpasswd-identity-provider}

You can configure identity providers by creating a custom resource (CR) that describes the provider and adding it to the cluster. Identity providers enable user authentication in OpenShift Container Platform beyond the default `kubeadmin` user.

:::note

OpenShift Container Platform usernames containing `/`, `:`, and `%` are not supported.

:::

## About htpasswd authentication {#identity-provider-htpasswd-about_configuring-htpasswd-identity-provider}

Configure `htpasswd` authentication to use a flat password file for login to OpenShift Container Platform. The file stores hashed credentials for each user and enables local authentication without an external identity provider.

:::warning

Do not use `htpasswd` authentication in OpenShift Container Platform for production environments. Use `htpasswd` authentication only for development environments.

:::

## Creating the htpasswd file {#identity-provider-creating-htpasswd-file_configuring-htpasswd-identity-provider}

To configure the `htpasswd` identity provider, create an `htpasswd` file so usernames and hashed passwords are available for the cluster secret. The following procedures describe how to create the file on Linux and Windows Operating Systems.

- Creating an `htpasswd` file using Linux
- Creating an `htpasswd` file using Windows

### Creating an htpasswd file using Linux {#identity-provider-creating-htpasswd-file-linux_configuring-htpasswd-identity-provider}

Create a flat `htpasswd` file on Red Hat Enterprise Linux (RHEL) with the `htpasswd` utility to store usernames and hashed passwords for your cluster. The file enables the `htpasswd` identity provider to authenticate users in OpenShift Container Platform from locally stored credentials.

**Prerequisites**

- You have access to the `htpasswd` utility. On Red Hat Enterprise Linux (RHEL), this is available by installing the `httpd-tools` package.

**Procedure**

1. Create or update your `htpasswd` file with a username and hashed password by running the following command:
   ```terminal
   $ htpasswd -c -B -b </path/to/users.htpasswd> <username> <password>
   ```

   The command generates a hashed version of the password.

   For example:

   ```terminal
   $ htpasswd -c -B -b users.htpasswd <username> <password>
   ```

   ```terminal title="Example output"
   Adding password for user user1
   ```
2. Continue to add or update credentials to the file by running the following command:
   ```terminal
   $ htpasswd -B -b </path/to/users.htpasswd> <user_name> <password>
   ```

### Creating an htpasswd file using Windows {#identity-provider-creating-htpasswd-file-windows_configuring-htpasswd-identity-provider}

Create a flat `htpasswd` file on Windows with the `htpasswd.exe` utility to store usernames and hashed passwords for your cluster. The file enables the `htpasswd` identity provider to authenticate users in OpenShift Container Platform from locally stored credentials.

**Prerequisites**

- You have access to the `htpasswd.exe` utility. On Windows, this utility is included in the `\bin` subdirectory of many Apache httpd distributions.

**Procedure**

1. Create or update your `htpasswd` file with a username and hashed password by running the following command:
   ```terminal
   $ htpasswd.exe -c -B -b <\path\to\users.htpasswd> <username> <password>
   ```

   The command generates a hashed version of the password.

   For example:

   ```terminal
   $ htpasswd.exe -c -B -b users.htpasswd <username> <password>
   ```

   ```terminal title="Example output"
   Adding password for user user1
   ```
2. Continue to add or update credentials to the file by running the following command:
   ```terminal
   $ htpasswd.exe -b <\path\to\users.htpasswd> <username> <password>
   ```

## Creating the htpasswd secret {#identity-provider-creating-htpasswd-secret_configuring-htpasswd-identity-provider}

Create an OpenShift Container Platform secret from your `htpasswd` file so the `htpasswd` identity provider can read user credentials for cluster login.

**Prerequisites**

- You created an `htpasswd` file.

**Procedure**

- Create a `Secret` object that contains the `htpasswd` users file by running the following command:
  ```terminal
  $ oc create secret generic htpass-secret --from-file=htpasswd=<path_to_users.htpasswd> -n openshift-config
  ```

  The `--from-file` key must be named `htpasswd`.

  :::tip

  You can alternatively apply the following YAML to create the secret:

  ```yaml
  apiVersion: v1
  kind: Secret
  metadata:
    name: htpass-secret
    namespace: openshift-config
  type: Opaque
  data:
    htpasswd: <base64_encoded_htpasswd_file_contents>
  ```

  :::

## Sample htpasswd CR {#identity-provider-htpasswd-CR_configuring-htpasswd-identity-provider}

Review the custom resource fields and acceptable values for configuring an `htpasswd` identity provider in OpenShift Container Platform.

```yaml title="htpasswd CR"
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: my_htpasswd_provider
    mappingMethod: claim
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpass-secret
```

where:

`spec.identityProviders.name`
: Specifies the provider name, which is prefixed to provider usernames to form an identity name.

`spec.identityProviders.mappingMethod`
: Specifies how mappings are established between identities from this provider and `User` objects.

`spec.identityProviders.htpasswd.fileData.name`
: Specifies an existing secret containing a file generated using `htpasswd`. For more information, see "htpasswd".

**Additional resources**

- [Identity provider parameters](/docs/authentication/understanding-identity-provider#identity-provider-parameters_understanding-identity-provider)

## Adding an identity provider to your cluster {#add-identity-provider_configuring-htpasswd-identity-provider}

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
2. Log in to the cluster as a user from your identity provider, entering the password when prompted.
   ```terminal
   $ oc login -u <username>
   ```
3. Confirm that the user logged in successfully and that the username displays by running the following command:
   ```terminal
   $ oc whoami
   ```

## Updating users for an htpasswd identity provider {#identity-provider-htpasswd-update-users_configuring-htpasswd-identity-provider}

Update users in the `htpasswd` identity provider so login credentials in OpenShift Container Platform stay in sync when you add or remove accounts.

**Prerequisites**

- You have created a `Secret` object named `htpass-secret` that contains the `htpasswd` user file.
- You have configured an `htpasswd` identity provider named `my_htpasswd_provider`.
- You have access to the `htpasswd` utility. On Red Hat Enterprise Linux (RHEL), this is available by installing the `httpd-tools` package.
- You have cluster administrator privileges.

**Procedure**

1. Retrieve the `htpasswd` file from the `htpass-secret` `Secret` object and save it to your local machine by running the following command:
   ```terminal
   $ oc get secret htpass-secret -ojsonpath={.data.htpasswd} -n openshift-config | base64 --decode > users.htpasswd
   ```
2. Add or remove users from the `users.htpasswd` file by running the following commands:
   1. To add a new user:
      ```terminal
      $ htpasswd -bB users.htpasswd <username> <password>
      ```

      ```terminal title="Example output"
      Adding password for user <username>
      ```
   2. To remove an existing user:
      ```terminal
      $ htpasswd -D users.htpasswd <username>
      ```

      ```terminal title="Example output"
      Deleting password for user <username>
      ```
3. Replace the `htpass-secret` `Secret` object with the updated users in the `users.htpasswd` file by running the following command:
   ```terminal
   $ oc create secret generic htpass-secret --from-file=htpasswd=users.htpasswd --dry-run=client -o yaml -n openshift-config | oc replace -f -
   ```

   :::tip

   You can also apply the following YAML to replace the secret:

   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: htpass-secret
     namespace: openshift-config
   type: Opaque
   data:
     htpasswd: <base64_encoded_htpasswd_file_contents>
   ```

   :::
4. If you removed one or more users, you must remove the existing resources for each user by running the following commands:
   1. Delete the `User` object:
      ```terminal
      $ oc delete user <username>
      ```

      ```terminal title="Example output"
      user.user.openshift.io "<username>" deleted
      ```

      Be sure to remove the user, otherwise the user can continue using their token as long as it has not expired.
   2. Delete the `Identity` object for the user:
      ```terminal
      $ oc delete identity my_htpasswd_provider:<username>
      ```

      ```terminal title="Example output"
      identity.user.openshift.io "my_htpasswd_provider:<username>" deleted
      ```

## Configuring identity providers using the web console {#identity-provider-configuring-using-the-web-console_configuring-htpasswd-identity-provider}

You can configure identity providers on your OpenShift Container Platform cluster through the web console by updating the **OAuth** settings in the **Cluster Settings**.

**Prerequisites**

- You are logged in to the web console as a cluster administrator.

**Procedure**

1. Navigate to **Administration** → **Cluster Settings**.
2. Under the **Configuration** tab, click **OAuth**.
3. Under the **Identity Providers** section, select your identity provider from the **Add** drop-down list.
   :::note

   You can specify multiple identity providers through the web console without overwriting existing identity providers.

   :::

**Additional resources**

- [htpasswd utility (Apache HTTP Server documentation)](http://httpd.apache.org/docs/2.4/programs/htpasswd.html)
