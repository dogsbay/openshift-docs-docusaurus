---
title: Preparing an AWS STS cluster for the AWS Load Balancer Operator
sidebar_position: 3
---

# Preparing an AWS STS cluster for the AWS Load Balancer Operator {#albo-sts-cluster}

<a id="albo-sts-cluster"></a>

To install the Amazon Web Services (AWS) Load Balancer Operator on a cluster that uses the Security Token Service (STS), prepare the cluster by configuring the `CredentialsRequest` object. This ensures the Operator can bootstrap the AWS Load Balancer Controller and access the required secrets.

The AWS Load Balancer Operator waits until the required secrets are created and available.

Before you start any Security Token Service (STS) procedures, ensure that you meet the following prerequisites:

- You installed the OpenShift CLI (`oc`).
- You know the infrastructure ID of your cluster. To show this ID, run the following command in your CLI:
  ```terminal
  $ oc get infrastructure cluster -o=jsonpath="{.status.infrastructureName}"
  ```
- You know the OpenID Connect (OIDC) DNS information for your cluster. To show this information, enter the following command in your CLI:
  ```terminal
  $ oc get authentication.config cluster -o=jsonpath="{.spec.serviceAccountIssuer}"
  ```

  where:

  <dl>
  <dt><code>{.spec.serviceAccountIssuer}</code></dt>
  <dd>Specifies an OIDC DNS URL. An example URL is <code>https://rh-oidc.s3.us-east-1.amazonaws.com/28292va7ad7mr9r4he1fb09b14t59t4f</code>.</dd>
  </dl>
- You logged into the AWS management console, navigated to **IAM** → **Access management** → **Identity providers**, and located the OIDC Amazon Resource Name (ARN) information. An OIDC ARN example is `arn:aws:iam::777777777777:oidc-provider/<oidc_dns_url>`.

**Additional resources**

- [the Cloud Credential Operator utility (`ccoctl`)](/docs/installing/installing_aws/ipi/installing-aws-customizations#cco-ccoctl-configuring_installing-aws-customizations)

## The IAM role for the AWS Load Balancer Operator {#the-iam-role-albo-operator_albo-sts-cluster}

To install the Amazon Web Services (AWS) Load Balancer Operator on a cluster by using STS, configure an additional Identity and Access Management (IAM) role.

You can create the IAM role by using the following options:

- Using the Cloud Credential Operator utility (`ccoctl`) and a predefined `CredentialsRequest` object.
- Using the AWS CLI and predefined AWS manifests.

Use the AWS CLI if your environment does not support the `ccoctl` command.

### Create an AWS IAM role by using the Cloud Credential Operator utility {#using-ccoctl-create-iam-role-alb-operator_albo-sts-cluster}

To enable the AWS Load Balancer Operator to interact with subnets and VPCs, create an AWS IAM role by using the Cloud Credential Operator utility (`ccoctl`).

**Prerequisites**

- You must extract and prepare the `ccoctl` binary.

**Procedure**

1. Download the `CredentialsRequest` custom resource (CR) and store it in a directory by running the following command:
   ```terminal
   $ curl --create-dirs -o <credentials_requests_dir>/operator.yaml https://raw.githubusercontent.com/openshift/aws-load-balancer-operator/main/hack/operator-credentials-request.yaml
   ```
2. Use the `ccoctl` utility to create an AWS IAM role by running the following command:
   ```terminal
   $ ccoctl aws create-iam-roles \
       --name <name> \
       --region=<aws_region> \
       --credentials-requests-dir=<credentials_requests_dir> \
       --identity-provider-arn <oidc_arn>
   ```

   ```terminal title="Example output"
   2023/09/12 11:38:57 Role arn:aws:iam::777777777777:role/<name>-aws-load-balancer-operator-aws-load-balancer-operator created
   2023/09/12 11:38:57 Saved credentials configuration to: /home/user/<credentials_requests_dir>/manifests/aws-load-balancer-operator-aws-load-balancer-operator-credentials.yaml
   2023/09/12 11:38:58 Updated Role policy for Role <name>-aws-load-balancer-operator-aws-load-balancer-operator created
   ```

   where:

   <dl>
   <dt><code>&lt;name&gt;</code></dt>
   <dd>Specifies the Amazon Resource Name (ARN) for an AWS IAM role that was created for the AWS Load Balancer Operator, such as <code>arn:aws:iam::777777777777:role/&lt;name&gt;-aws-load-balancer-operator-aws-load-balancer-operator</code>.</dd>
   </dl>

   :::note

   The length of an AWS IAM role name must be less than or equal to 12 characters.

   :::

### Create an AWS IAM role by using the AWS CLI {#using-aws-cli-create-iam-role-alb-operator_albo-sts-cluster}

To enable the AWS Load Balancer Operator to interact with subnets and VPCs, create an AWS IAM role by using the AWS CLI. This enables the Operator to access and manage the necessary network resources within the cluster.

**Prerequisites**

- You must have access to the AWS Command Line Interface (`aws`).

**Procedure**

1. Generate a trust policy file by using your identity provider by running the following command:
   ```terminal
   $ cat <<EOF > albo-operator-trust-policy.json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Principal": {
                   "Federated": "<oidc_arn>"
               },
               "Action": "sts:AssumeRoleWithWebIdentity",
               "Condition": {
                   "StringEquals": {
                       "<cluster_oidc_endpoint>:sub": "system:serviceaccount:aws-load-balancer-operator:aws-load-balancer-operator-controller-manager"
                   }
               }
           }
       ]
   }
   EOF
   ```

   where:

   <dl>
   <dt><code>&lt;oidc_arn&gt;</code></dt>
   <dd>Specifies the Amazon Resource Name (ARN) of the OIDC identity provider, such as <code>arn:aws:iam::777777777777:oidc-provider/rh-oidc.s3.us-east-1.amazonaws.com/28292va7ad7mr9r4he1fb09b14t59t4f</code>.</dd>
   <dt><code>serviceaccount</code></dt>
   <dd>Specifies the service account for the AWS Load Balancer Controller. An example of <code>&lt;cluster_oidc_endpoint&gt;</code> is <code>rh-oidc.s3.us-east-1.amazonaws.com/28292va7ad7mr9r4he1fb09b14t59t4f</code>.</dd>
   </dl>
2. Create the IAM role with the generated trust policy by running the following command:
   ```terminal
   $ aws iam create-role --role-name albo-operator --assume-role-policy-document file://albo-operator-trust-policy.json
   ```

   ```terminal title="Example output"
   ROLE	arn:aws:iam::<aws_account_number>:role/albo-operator	2023-08-02T12:13:22Z (1)
   ASSUMEROLEPOLICYDOCUMENT	2012-10-17
   STATEMENT	sts:AssumeRoleWithWebIdentity	Allow
   STRINGEQUALS	system:serviceaccount:aws-load-balancer-operator:aws-load-balancer-controller-manager
   PRINCIPAL	arn:aws:iam:<aws_account_number>:oidc-provider/<cluster_oidc_endpoint>
   ```

   where:

   <dl>
   <dt><code>&lt;aws_account_number&gt;</code></dt>
   <dd>Specifies the ARN of the created AWS IAM role for the AWS Load Balancer Operator, such as <code>arn:aws:iam::777777777777:role/albo-operator</code>.</dd>
   </dl>
3. Download the permission policy for the AWS Load Balancer Operator by running the following command:
   ```terminal
   $ curl -o albo-operator-permission-policy.json https://raw.githubusercontent.com/openshift/aws-load-balancer-operator/main/hack/operator-permission-policy.json
   ```
4. Attach the permission policy for the AWS Load Balancer Controller to the IAM role by running the following command:
   ```terminal
   $ aws iam put-role-policy --role-name albo-operator --policy-name perms-policy-albo-operator --policy-document file://albo-operator-permission-policy.json
   ```

## Configure the ARN role for the AWS Load Balancer Operator {#specifying-role-arn-albo-sts_albo-sts-cluster}

You can configure the Amazon Resource Name (ARN) role for the AWS Load Balancer Operator as an environment variable. You can configure the ARN role by using the CLI.

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).

**Procedure**

1. Create the `aws-load-balancer-operator` project by running the following command:
   ```terminal
   $ oc new-project aws-load-balancer-operator
   ```
2. Create the `OperatorGroup` object by running the following command:
   ```terminal
   $ cat <<EOF | oc apply -f -
   apiVersion: operators.coreos.com/v1
   kind: OperatorGroup
   metadata:
     name: aws-load-balancer-operator
     namespace: aws-load-balancer-operator
   spec:
     targetNamespaces: []
   EOF
   ```
3. Create the `Subscription` object by running the following command:
   ```terminal
   $ cat <<EOF | oc apply -f -
   apiVersion: operators.coreos.com/v1alpha1
   kind: Subscription
   metadata:
     name: aws-load-balancer-operator
     namespace: aws-load-balancer-operator
   spec:
     channel: stable-v1
     name: aws-load-balancer-operator
     source: redhat-operators
     sourceNamespace: openshift-marketplace
     config:
       env:
       - name: ROLEARN
         value: "<albo_role_arn>"
   EOF
   ```

   where:

   <dl>
   <dt><code>&lt;albo_role_arn&gt;</code></dt>
   <dd>Specifies the ARN role to be used in the <code>CredentialsRequest</code> to provision the AWS credentials for the AWS Load Balancer Operator. An example for <code>&lt;albo_role_arn&gt;</code> is <code>arn:aws:iam::&lt;aws_account_number&gt;:role/albo-operator</code>.</dd>
   </dl>

   :::note

   The AWS Load Balancer Operator waits until the secret is created before moving to the `Available` status.

   :::

## The IAM role for the AWS Load Balancer Controller {#the-iam-role-albo-controller.adoc_albo-sts-cluster}

The `CredentialsRequest` object for the AWS Load Balancer Controller must be set with a manually provisioned Identity and Access Management (IAM) role.

You can create the IAM role by using the following options:

- Using the Cloud Credential Operator utility (`ccoctl`) and a predefined `CredentialsRequest` object.
- Using the AWS CLI and predefined AWS manifests.

If your environment does not support the `ccoctl` `command.ws-short` CLI, use the AWS CLI.

**Additional resources**

- [the Cloud Credential Operator utility (`ccoctl`)](/docs/installing/installing_aws/ipi/installing-aws-customizations#cco-ccoctl-configuring_installing-aws-customizations)

### Create an AWS IAM role for the controller by using the Cloud Credential Operator utility {#using-ccoctl-create-iam-role-alb-controller_albo-sts-cluster}

To enable the AWS Load Balancer Controller to interact with subnets and VPCs, create an IAM role by using the Cloud Credential Operator utility (`ccoctl`). This utility ensures the controller has the specific permissions required to manage network resources within the cluster.

**Prerequisites**

- You must extract and prepare the `ccoctl` binary.

**Procedure**

1. Download the `CredentialsRequest` custom resource (CR) and store it in a directory by running the following command:
   ```terminal
   $ curl --create-dirs -o <credentials_requests_dir>/controller.yaml https://raw.githubusercontent.com/openshift/aws-load-balancer-operator/main/hack/controller/controller-credentials-request.yaml
   ```
2. Use the `ccoctl` utility to create an AWS IAM role by running the following command:
   ```terminal
   $ ccoctl aws create-iam-roles \
       --name <name> \
       --region=<aws_region> \
       --credentials-requests-dir=<credentials_requests_dir> \
       --identity-provider-arn <oidc_arn>
   ```

   ```terminal title="Example output"
   2023/09/12 11:38:57 Role arn:aws:iam::777777777777:role/<name>-aws-load-balancer-operator-aws-load-balancer-controller created
   2023/09/12 11:38:57 Saved credentials configuration to: /home/user/<credentials_requests_dir>/manifests/aws-load-balancer-operator-aws-load-balancer-controller-credentials.yaml
   2023/09/12 11:38:58 Updated Role policy for Role <name>-aws-load-balancer-operator-aws-load-balancer-controller created
   ```

   where:

   <dl>
   <dt><code>&lt;name&gt;</code></dt>
   <dd>Specifies the Amazon Resource Name (ARN) for an AWS IAM role that was created for the AWS Load Balancer Controller, such as <code>arn:aws:iam::777777777777:role/&lt;name&gt;-aws-load-balancer-operator-aws-load-balancer-controller</code>.</dd>
   </dl>

   :::note

   The length of an AWS IAM role name must be less than or equal to 12 characters.

   :::

### Create an AWS IAM role for the controller by using the AWS CLI {#using-aws-cli-create-iam-role-alb-controller_albo-sts-cluster}

To enable the AWS Load Balancer Controller to interact with subnets and Virtual Private Clouds (VPCs), create an IAM role by using the AWS CLI. This ensures the controller has the specific permissions required to manage network resources within the cluster.

**Prerequisites**

- You must have access to the AWS command-line interface (`aws`).

**Procedure**

1. Generate a trust policy file using your identity provider by running the following command:
   ```terminal
   $ cat <<EOF > albo-controller-trust-policy.json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Principal": {
                   "Federated": "<oidc_arn>"
               },
               "Action": "sts:AssumeRoleWithWebIdentity",
               "Condition": {
                   "StringEquals": {
                       "<cluster_oidc_endpoint>:sub": "system:serviceaccount:aws-load-balancer-operator:aws-load-balancer-operator-controller-manager"
                   }
               }
           }
       ]
   }
   EOF
   ```

   where:

   <dl>
   <dt><code>&lt;oidc_arn&gt;</code></dt>
   <dd>Specifies the Amazon Resource Name (ARN) of the OIDC identity provider, such as <code>arn:aws:iam::777777777777:oidc-provider/rh-oidc.s3.us-east-1.amazonaws.com/28292va7ad7mr9r4he1fb09b14t59t4f</code>.</dd>
   <dt><code>serviceaccount</code></dt>
   <dd>Specifies the service account for the AWS Load Balancer Controller. An example of <code>&lt;cluster_oidc_endpoint&gt;</code> is <code>rh-oidc.s3.us-east-1.amazonaws.com/28292va7ad7mr9r4he1fb09b14t59t4f</code>.</dd>
   </dl>
2. Create an AWS IAM role with the generated trust policy by running the following command:
   ```terminal
   $ aws iam create-role --role-name albo-controller --assume-role-policy-document file://albo-controller-trust-policy.json
   ```

   ```terminal title="Example output"
   ROLE	arn:aws:iam::<aws_account_number>:role/albo-controller	2023-08-02T12:13:22Z (1)
   ASSUMEROLEPOLICYDOCUMENT	2012-10-17
   STATEMENT	sts:AssumeRoleWithWebIdentity	Allow
   STRINGEQUALS	system:serviceaccount:aws-load-balancer-operator:aws-load-balancer-operator-controller-manager
   PRINCIPAL	arn:aws:iam:<aws_account_number>:oidc-provider/<cluster_oidc_endpoint>
   ```

   where:

   <dl>
   <dt><code>&lt;aws_account_number&gt;</code></dt>
   <dd>Specifies the ARN for an AWS IAM role for the AWS Load Balancer Controller, such as <code>arn:aws:iam::777777777777:role/albo-controller</code>.</dd>
   </dl>
3. Download the permission policy for the AWS Load Balancer Controller by running the following command:
   ```terminal
   $ curl -o albo-controller-permission-policy.json https://raw.githubusercontent.com/openshift/aws-load-balancer-operator/main/assets/iam-policy.json
   ```
4. Attach the permission policy for the AWS Load Balancer Controller to an AWS IAM role by running the following command:
   ```terminal
   $ aws iam put-role-policy --role-name albo-controller --policy-name perms-policy-albo-controller --policy-document file://albo-controller-permission-policy.json
   ```
5. Create a YAML file that defines the `AWSLoadBalancerController` object:
   ```yaml title="Example sample-aws-lb-manual-creds.yaml file"
   apiVersion: networking.olm.openshift.io/v1
   kind: AWSLoadBalancerController
   metadata:
     name: cluster
   spec:
     credentialsRequestConfig:
       stsIAMRoleARN: <albc_role_arn>
   ```

   where:

   <dl>
   <dt><code>kind</code></dt>
   <dd>Specifies the <code>AWSLoadBalancerController</code> object.</dd>
   <dt><code>metatdata.name</code></dt>
   <dd>Specifies the AWS Load Balancer Controller name. All related resources use this instance name as a suffix.</dd>
   <dt><code>stsIAMRoleARN</code></dt>
   <dd>Specifies the ARN role for the AWS Load Balancer Controller. The <code>CredentialsRequest</code> object uses this ARN role to provision the AWS credentials. An example of <code>&lt;albc_role_arn&gt;</code> is <code>arn:aws:iam::777777777777:role/albo-controller</code>.</dd>
   </dl>

**Additional resources**

- [Configuring the Cloud Credential Operator utility](/docs/installing/installing_aws/ipi/installing-aws-customizations#cco-ccoctl-configuring_installing-aws-customizations)
