---
title: About the Cloud Credential Operator in mint mode
sidebar_position: 2
---

# About the Cloud Credential Operator in mint mode {#cco-mode-mint}

<a id="cco-mode-mint"></a>

You can use the Cloud Credential Operator (CCO) in mint mode to create and reconcile credentials for components in the cluster.

Mint mode is the default CCO credentials mode for OpenShift Container Platform on platforms that support it. Mint mode supports Amazon Web Services (AWS) and Google Cloud clusters.

## About mint mode credentials management {#mint-mode-credentials-management_cco-mode-mint}

When using the Cloud Credential Operator (CCO) in mint mode, you should be familiar with how the CCO uses provider credentials.

For clusters that use the CCO in mint mode, the administrator-level credential is stored in the `kube-system` namespace. The CCO uses the `admin` credential to process the `CredentialsRequest` objects in the cluster and create users for components with limited permissions.

With mint mode, each cluster component has only the specific permissions it requires. Cloud credential reconciliation is automatic and continuous so that components can perform actions that require additional credentials or permissions.

For example, a minor version cluster update (such as updating from OpenShift Container Platform 4.21 to 4.22) might include an updated `CredentialsRequest` resource for a cluster component. The CCO, operating in mint mode, uses the `admin` credential to process the `CredentialsRequest` resource and create users with limited permissions to satisfy the updated authentication requirements.

:::note

By default, mint mode requires storing the `admin` credential in the cluster `kube-system` namespace. If this approach does not meet the security requirements of your organization, you can remove the credential after installing the cluster. For more information, see "Removing cloud provider credentials".

:::

### About mint mode permissions requirements {#mint-mode-permission-requirements_cco-mode-mint}

When using the Cloud Credential Operator (CCO) in mint mode, ensure that the credential you provide meets the requirements of the cloud on which you are running or installing OpenShift Container Platform. If the provided credentials are not sufficient for mint mode, the CCO cannot create an IAM user.

#### Required AWS permissions {#mint-mode-permission-requirements_aws_cco-mode-mint}

The credential you provide for mint mode in Amazon Web Services (AWS) must have the following permissions:

- `iam:CreateAccessKey`
- `iam:CreateUser`
- `iam:DeleteAccessKey`
- `iam:DeleteUser`
- `iam:DeleteUserPolicy`
- `iam:GetUser`
- `iam:GetUserPolicy`
- `iam:ListAccessKeys`
- `iam:PutUserPolicy`
- `iam:TagUser`
- `iam:SimulatePrincipalPolicy`

#### Required Google Cloud permissions {#mint-mode-permission-requirements_gcp_cco-mode-mint}

The credential you provide for mint mode in Google Cloud must have the following permissions:

- `resourcemanager.projects.get`
- `serviceusage.services.list`
- `iam.serviceAccountKeys.create`
- `iam.serviceAccountKeys.delete`
- `iam.serviceAccountKeys.list`
- `iam.serviceAccounts.create`
- `iam.serviceAccounts.delete`
- `iam.serviceAccounts.get`
- `iam.roles.create`
- `iam.roles.get`
- `iam.roles.list`
- `iam.roles.undelete`
- `iam.roles.update`
- `resourcemanager.projects.getIamPolicy`
- `resourcemanager.projects.setIamPolicy`

### Admin credentials root secret format {#admin-credentials-root-secret-formats_cco-mode-mint}

The Cloud Credential Operator (CCO) creates a credentials root secret by minting new credentials with *mint mode* or by copying the credentials root secret with *passthrough mode*.

Each cloud provider uses a credentials root secret in the `kube-system` namespace by convention, which is then used to satisfy all credentials requests and create their respective secrets.

The format for the secret varies by cloud, and is also used for each `CredentialsRequest` secret.

```yaml title="Amazon Web Services (AWS) secret format"
apiVersion: v1
kind: Secret
metadata:
  namespace: kube-system
  name: aws-creds
stringData:
  aws_access_key_id: <base64-encoded_access_key_id>
  aws_secret_access_key: <base64-encoded_secret_access_key>
```

```yaml title="Google Cloud secret format"
apiVersion: v1
kind: Secret
metadata:
  namespace: kube-system
  name: gcp-credentials
stringData:
  service_account.json: <base64-encoded_service_account>
```

## Maintaining cloud provider credentials {#_maintaining_cloud_provider_credentials}

If your cloud provider credentials are changed for any reason, you must manually update the secret that the Cloud Credential Operator (CCO) uses to manage cloud provider credentials.

The process for rotating cloud credentials depends on the mode that the CCO is configured to use. After you rotate credentials for a cluster that is using mint mode, you must manually remove the component credentials that were created by the removed credential.

**Prerequisites**

- Your cluster is installed on a platform that supports rotating cloud credentials manually with the CCO mode that you are using:
  - For mint mode, Amazon Web Services (AWS) and Google Cloud are supported.
- You have changed the credentials that are used to interface with your cloud provider.
- The new credentials have sufficient permissions for the mode CCO is configured to use in your cluster.

**Procedure**

1. In the **Administrator** perspective of the web console, navigate to **Workloads** → **Secrets**.
2. In the table on the **Secrets** page, find the root secret for your cloud provider.

<table>
<thead>
<tr>
  <th>Platform</th>
  <th>Secret name</th>
</tr>
</thead>
<tbody>
<tr>
  <td>AWS</td>
  <td><code>aws-creds</code></td>
</tr>
<tr>
  <td>Google Cloud</td>
  <td><code>gcp-credentials</code></td>
</tr>
</tbody>
</table>

1. Click the Options menu ![](/images/kebab.png "Options menu") in the same row as the secret and select **Edit Secret**.
2. Record the contents of the **Value** field or fields. You can use this information to verify that the value is different after updating the credentials.
3. Update the text in the **Value** field or fields with the new authentication information for your cloud provider, and then click **Save**.
4. Delete each component secret that is referenced by the individual `CredentialsRequest` objects.
   1. Log in to the OpenShift Container Platform CLI as a user with the `cluster-admin` role.
   2. Get the names and namespaces of all referenced component secrets:
      ```terminal
      $ oc -n openshift-cloud-credential-operator get CredentialsRequest \
        -o json | jq -r '.items[] | select (.spec.providerSpec.kind=="<provider_spec>") | .spec.secretRef'
      ```

      where `<provider_spec>` is the corresponding value for your cloud provider:

      - AWS: `AWSProviderSpec`
      - Google Cloud: `GCPProviderSpec`

      The following example is partial output for the command on an AWS cluster:

      ```json
      {
        "name": "ebs-cloud-credentials",
        "namespace": "openshift-cluster-csi-drivers"
      }
      {
        "name": "cloud-credential-operator-iam-ro-creds",
        "namespace": "openshift-cloud-credential-operator"
      }
      ```
   3. Delete each of the referenced component secrets:
      ```terminal
      $ oc delete secret <secret_name> \ (1)
        -n <secret_namespace> (2)
      ```

      where:

      <dl>
      <dt><code>&lt;secret_name&gt;</code></dt>
      <dd>Specifies the name of a secret.</dd>
      <dt><code>&lt;secret_namespace&gt;</code></dt>
      <dd>Specifies the namespace that contains the secret.</dd>
      </dl>

      The following example is a command to delete an AWS secret:

      ```terminal
      $ oc delete secret ebs-cloud-credentials -n openshift-cluster-csi-drivers
      ```

      You do not need to manually delete the credentials from your provider console. Deleting the referenced component secrets will cause the CCO to delete the existing credentials from the platform and create new ones.

**Verification**

1. In the **Administrator** perspective of the web console, navigate to **Workloads** → **Secrets**.
2. Verify that the contents of the **Value** field or fields have changed.

**Additional resources**

- [Removing cloud provider credentials](/docs/post_installation_configuration/changing-cloud-credentials-configuration#manually-removing-cloud-creds_changing-cloud-credentials-configuration)
