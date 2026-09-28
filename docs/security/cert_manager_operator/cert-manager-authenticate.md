---
title: Authenticating the cert-manager Operator for Red Hat OpenShift
sidebar_position: 6
---

# Authenticating the cert-manager Operator for Red Hat OpenShift {#cert-manager-authenticate}

<a id="cert-manager-authenticate"></a>

To enable the operator to manage components on your cloud provider, authenticate the cert-manager Operator for Red Hat OpenShift by configuring cloud credentials. You can grant the Operator access to external services required for certificate issuance, such as DNS providers.

## Authenticate on AWS {#cert-manager-configure-cloud-credentials-aws-non-sts_cert-manager-authenticate}

To securely access AWS resources from your applications, authenticate your workloads on AWS by using the cert-manager Operator for Red Hat OpenShift.

**Prerequisites**

- You have installed version 1.11.1 or later of the cert-manager Operator for Red Hat OpenShift.
- You have configured the Cloud Credential Operator to operate in *mint* or *passthrough* mode.

**Procedure**

1. Create a `CredentialsRequest` resource YAML file, for example, `sample-credential-request.yaml`, as follows:
   ```yaml
   apiVersion: cloudcredential.openshift.io/v1
   kind: CredentialsRequest
   metadata:
     name: cert-manager
     namespace: openshift-cloud-credential-operator
   spec:
     providerSpec:
       apiVersion: cloudcredential.openshift.io/v1
       kind: AWSProviderSpec
       statementEntries:
       - action:
         - "route53:GetChange"
         effect: Allow
         resource: "arn:aws:route53:::change/*"
       - action:
         - "route53:ChangeResourceRecordSets"
         - "route53:ListResourceRecordSets"
         effect: Allow
         resource: "arn:aws:route53:::hostedzone/*"
       - action:
         - "route53:ListHostedZonesByName"
         effect: Allow
         resource: "*"
     secretRef:
       name: aws-creds
       namespace: cert-manager
     serviceAccountNames:
     - cert-manager
   ```
2. Create a `CredentialsRequest` resource by running the following command:
   ```terminal
   $ oc create -f sample-credential-request.yaml
   ```
3. Update the subscription object for cert-manager Operator for Red Hat OpenShift by running the following command:
   ```terminal
   $ oc -n cert-manager-operator patch subscription openshift-cert-manager-operator --type=merge -p '{"spec":{"config":{"env":[{"name":"CLOUD_CREDENTIALS_SECRET_NAME","value":"aws-creds"}]}}}'
   ```

**Verification**

1. Get the name of the redeployed cert-manager controller pod by running the following command:
   ```terminal
   $ oc get pods -l app.kubernetes.io/name=cert-manager -n cert-manager
   ```

   ```terminal title="Example output"
   NAME                          READY   STATUS    RESTARTS   AGE
   cert-manager-bd7fbb9fc-wvbbt  1/1     Running   0          15m39s
   ```
2. Verify that the cert-manager controller pod is updated with AWS credential volumes that are mounted under the path specified in `mountPath` by running the following command:
   ```terminal
   $ oc get -n cert-manager pod/<cert-manager_controller_pod_name> -o yaml
   ```

   ```terminal title="Example output"
   ...
   spec:
     containers:
     - args:
       ...
       - mountPath: /.aws
         name: cloud-credentials
     ...
     volumes:
     ...
     - name: cloud-credentials
       secret:
         ...
         secretName: aws-creds
   ```

## Authenticate with AWS Security Token Service {#cert-manager-configure-cloud-credentials-aws-sts_cert-manager-authenticate}

To securely access AWS resources from your applications without managing long-lived keys, authenticate your workloads by using the AWS Security Token Service (STS).

**Prerequisites**

- You have extracted and prepared the `ccoctl` binary.
- You have configured an OpenShift Container Platform cluster with AWS STS by using the Cloud Credential Operator in manual mode.

**Procedure**

1. Create a directory to store a `CredentialsRequest` resource YAML file by running the following command:
   ```terminal
   $ mkdir credentials-request
   ```
2. Create a `CredentialsRequest` resource YAML file under the `credentials-request` directory, such as, `sample-credential-request.yaml`, by applying the following yaml:
   ```yaml
   apiVersion: cloudcredential.openshift.io/v1
   kind: CredentialsRequest
   metadata:
     name: cert-manager
     namespace: openshift-cloud-credential-operator
   spec:
     providerSpec:
       apiVersion: cloudcredential.openshift.io/v1
       kind: AWSProviderSpec
       statementEntries:
       - action:
         - "route53:GetChange"
         effect: Allow
         resource: "arn:aws:route53:::change/*"
       - action:
         - "route53:ChangeResourceRecordSets"
         - "route53:ListResourceRecordSets"
         effect: Allow
         resource: "arn:aws:route53:::hostedzone/*"
       - action:
         - "route53:ListHostedZonesByName"
         effect: Allow
         resource: "*"
     secretRef:
       name: aws-creds
       namespace: cert-manager
     serviceAccountNames:
     - cert-manager
   ```
3. Use the `ccoctl` tool to process `CredentialsRequest` objects by running the following command:
   ```terminal
   $ ccoctl aws create-iam-roles \
       --name <user_defined_name> --region=<aws_region> \
       --credentials-requests-dir=<path_to_credrequests_dir> \
       --identity-provider-arn <oidc_provider_arn> --output-dir=<path_to_output_dir>
   ```

   ```terminal title="Example output"
   2023/05/15 18:10:34 Role arn:aws:iam::XXXXXXXXXXXX:role/<user_defined_name>-cert-manager-aws-creds created
   2023/05/15 18:10:34 Saved credentials configuration to: <path_to_output_dir>/manifests/cert-manager-aws-creds-credentials.yaml
   2023/05/15 18:10:35 Updated Role policy for Role <user_defined_name>-cert-manager-aws-creds
   ```

   Copy the `<aws_role_arn>` from the output to use in the next step. For example, `"arn:aws:iam::XXXXXXXXXXXX:role/<user_defined_name>-cert-manager-aws-creds"`
4. Add the `eks.amazonaws.com/role-arn="<aws_role_arn>"` annotation to the service account by running the following command:
   ```terminal
   $ oc -n cert-manager annotate serviceaccount cert-manager eks.amazonaws.com/role-arn="<aws_role_arn>"
   ```
5. To create a new pod, delete the existing cert-manager controller pod by running the following command:
   ```terminal
   $ oc delete pods -l app.kubernetes.io/name=cert-manager -n cert-manager
   ```

   The AWS credentials are applied to a new cert-manager controller pod within a minute.

**Verification**

1. Get the name of the updated cert-manager controller pod by running the following command:
   ```terminal
   $ oc get pods -l app.kubernetes.io/name=cert-manager -n cert-manager
   ```

   ```terminal title="Example output"
   NAME                          READY   STATUS    RESTARTS   AGE
   cert-manager-bd7fbb9fc-wvbbt  1/1     Running   0          39s
   ```
2. Verify that AWS credentials are updated by running the following command:
   ```terminal
   $ oc set env -n cert-manager po/<cert_manager_controller_pod_name> --list
   ```

   ```terminal title="Example output"
   # pods/cert-manager-57f9555c54-vbcpg, container cert-manager-controller
   # POD_NAMESPACE from field path metadata.namespace
   AWS_ROLE_ARN=XXXXXXXXXXXX
   AWS_WEB_IDENTITY_TOKEN_FILE=/var/run/secrets/eks.amazonaws.com/serviceaccount/token
   ```

**Additional resources**

- [Configuring the Cloud Credential Operator utility](/docs/installing/installing_aws/ipi/installing-aws-customizations#cco-ccoctl-configuring_installing-aws-customizations)

## Authenticate on Google Cloud {#cert-manager-configure-cloud-credentials-gcp-non-sts_cert-manager-authenticate}

To securely access Google Cloud resources, authenticate your workloads on Google Cloud by using the cert-manager Operator for Red Hat OpenShift.

**Prerequisites**

- You have installed version 1.11.1 or later of the cert-manager Operator for Red Hat OpenShift.
- You have configured the Cloud Credential Operator to operate in *mint* or *passthrough* mode.

**Procedure**

1. Create a `CredentialsRequest` resource YAML file, such as, `sample-credential-request.yaml` by applying the following yaml:
   ```yaml
   apiVersion: cloudcredential.openshift.io/v1
   kind: CredentialsRequest
   metadata:
     name: cert-manager
     namespace: openshift-cloud-credential-operator
   spec:
     providerSpec:
       apiVersion: cloudcredential.openshift.io/v1
       kind: GCPProviderSpec
       predefinedRoles:
       - roles/dns.admin
     secretRef:
       name: gcp-credentials
       namespace: cert-manager
     serviceAccountNames:
     - cert-manager
   ```

   :::note

   The `dns.admin` role provides admin privileges to the service account for managing Google Cloud DNS resources. To ensure that the cert-manager runs with the service account that has the least privilege, you can create a custom role with the following permissions:

   - `dns.resourceRecordSets.*`
   - `dns.changes.*`
   - `dns.managedZones.list`

   :::
2. Create a `CredentialsRequest` resource by running the following command:
   ```terminal
   $ oc create -f sample-credential-request.yaml
   ```
3. Update the subscription object for cert-manager Operator for Red Hat OpenShift by running the following command:
   ```terminal
   $ oc -n cert-manager-operator patch subscription openshift-cert-manager-operator --type=merge -p '{"spec":{"config":{"env":[{"name":"CLOUD_CREDENTIALS_SECRET_NAME","value":"gcp-credentials"}]}}}'
   ```

**Verification**

1. Get the name of the redeployed cert-manager controller pod by running the following command:
   ```terminal
   $ oc get pods -l app.kubernetes.io/name=cert-manager -n cert-manager
   ```

   ```terminal title="Example output"
   NAME                                       READY   STATUS    RESTARTS   AGE
   cert-manager-bd7fbb9fc-wvbbt               1/1     Running   0          15m39s
   ```
2. Verify that the cert-manager controller pod is updated with Google Cloud credential volumes that are mounted under the path specified in `mountPath` by running the following command:
   ```terminal
   $ oc get -n cert-manager pod/<cert-manager_controller_pod_name> -o yaml
   ```

   ```terminal title="Example output"
   spec:
     containers:
     - args:
       ...
       volumeMounts:
       ...
       - mountPath: /.config/gcloud
         name: cloud-credentials
       ....
     volumes:
     ...
     - name: cloud-credentials
       secret:
         ...
         items:
         - key: service_account.json
           path: application_default_credentials.json
         secretName: gcp-credentials
   ```

## Authenticate with Google Cloud Workload Identity {#cert-manager-configure-cloud-credentials-gcp-sts_cert-manager-authenticate}

To securely access Google Cloud resources from your applications without managing long-lived keys, authenticate your workloads by using Google Cloud Workload Identity.

**Prerequisites**

- You extracted and prepared the `ccoctl` binary.
- You have installed version 1.11.1 or later of the cert-manager Operator for Red Hat OpenShift.
- You have configured an OpenShift Container Platform cluster with Google Cloud Workload Identity by using the Cloud Credential Operator in a manual mode.

**Procedure**

1. Create a directory to store a `CredentialsRequest` resource YAML file by running the following command:
   ```terminal
   $ mkdir credentials-request
   ```
2. In the `credentials-request` directory, create a YAML file that contains the following `CredentialsRequest` manifest:
   ```yaml
   apiVersion: cloudcredential.openshift.io/v1
   kind: CredentialsRequest
   metadata:
     name: cert-manager
     namespace: openshift-cloud-credential-operator
   spec:
     providerSpec:
       apiVersion: cloudcredential.openshift.io/v1
       kind: GCPProviderSpec
       predefinedRoles:
       - roles/dns.admin
     secretRef:
       name: gcp-credentials
       namespace: cert-manager
     serviceAccountNames:
     - cert-manager
   ```

   :::note

   The `dns.admin` role provides admin privileges to the service account for managing Google Cloud DNS resources. To ensure that the cert-manager runs with the service account that has the least privilege, you can create a custom role with the following permissions:

   - `dns.resourceRecordSets.*`
   - `dns.changes.*`
   - `dns.managedZones.list`

   :::
3. Use the `ccoctl` tool to process `CredentialsRequest` objects by running the following command:
   ```terminal
   $ ccoctl gcp create-service-accounts \
       --name <user_defined_name> --output-dir=<path_to_output_dir> \
       --credentials-requests-dir=<path_to_credrequests_dir> \
       --workload-identity-pool <workload_identity_pool> \
       --workload-identity-provider <workload_identity_provider> \
       --project <gcp_project_id>
   ```

   ```terminal title="Example command"
   $ ccoctl gcp create-service-accounts \
       --name abcde-20230525-4bac2781 --output-dir=/home/outputdir \
       --credentials-requests-dir=/home/credentials-requests \
       --workload-identity-pool abcde-20230525-4bac2781 \
       --workload-identity-provider abcde-20230525-4bac2781 \
       --project openshift-gcp-devel
   ```
4. Apply the secrets generated in the manifests directory of your cluster by running the following command:
   ```terminal
   $ ls <path_to_output_dir>/manifests/*-credentials.yaml | xargs -I{} oc apply -f {}
   ```
5. Update the subscription object for cert-manager Operator for Red Hat OpenShift by running the following command:
   ```terminal
   $ oc -n cert-manager-operator patch subscription openshift-cert-manager-operator --type=merge -p '{"spec":{"config":{"env":[{"name":"CLOUD_CREDENTIALS_SECRET_NAME","value":"gcp-credentials"}]}}}'
   ```

**Verification**

1. Get the name of the redeployed cert-manager controller pod by running the following command:
   ```terminal
   $ oc get pods -l app.kubernetes.io/name=cert-manager -n cert-manager
   ```

   ```terminal title="Example output"
   NAME                          READY   STATUS    RESTARTS   AGE
   cert-manager-bd7fbb9fc-wvbbt  1/1     Running   0          15m39s
   ```
2. Verify that the cert-manager controller pod is updated with Google Cloud workload identity credential volumes that are mounted under the path specified in `mountPath` by running the following command:
   ```terminal
   $ oc get -n cert-manager pod/<cert-manager_controller_pod_name> -o yaml
   ```

   ```terminal title="Example output"
   spec:
     containers:
     - args:
       ...
       volumeMounts:
       - mountPath: /var/run/secrets/openshift/serviceaccount
         name: bound-sa-token
         ...
       - mountPath: /.config/gcloud
         name: cloud-credentials
     ...
     volumes:
     - name: bound-sa-token
       projected:
         ...
         sources:
         - serviceAccountToken:
             audience: openshift
             ...
             path: token
     - name: cloud-credentials
       secret:
         ...
         items:
         - key: service_account.json
           path: application_default_credentials.json
         secretName: gcp-credentials
   ```

**Additional resources**

- [Configuring the Cloud Credential Operator utility](/docs/installing/installing_gcp/installing-gcp-customizations#cco-ccoctl-configuring_installing-gcp-customizations)
- [Manual mode with short-term credentials for components](/docs/authentication/managing_cloud_provider_credentials/cco-short-term-creds#cco-short-term-creds)
- [Default behavior of the Cloud Credential Operator](/docs/authentication/managing_cloud_provider_credentials/about-cloud-credential-operator#about-cloud-credential-operator-default_about-cloud-credential-operator)
