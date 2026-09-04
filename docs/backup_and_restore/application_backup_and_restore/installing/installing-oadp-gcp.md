---
title: Configuring the OpenShift API for Data Protection with Google Cloud
sidebar_position: 1
---

# Configuring the OpenShift API for Data Protection with Google Cloud {#installing-oadp-gcp}

<a id="installing-oadp-gcp"></a>

You install the OpenShift API for Data Protection (OADP) with Google Cloud by installing the OADP Operator. The Operator installs Velero 1.16.

You configure Google Cloud for Velero, create a default `Secret`, and then install the Data Protection Application.

To install the OADP Operator in a restricted network environment, you must first disable the default software catalog sources and mirror the Operator catalog.

## Configuring Google Cloud {#migration-configuring-gcp_installing-oadp-gcp}

You configure Google Cloud for the OpenShift API for Data Protection (OADP).

**Prerequisites**

- You must have the `gcloud` and `gsutil` CLI tools installed. See the [Google cloud documentation](https://cloud.google.com/sdk/docs/) for details.

**Procedure**

1. Log in to Google Cloud:
   ```terminal
   $ gcloud auth login
   ```
2. Set the `BUCKET` variable:
   ```terminal
   $ BUCKET=<bucket>
   ```

   where:

   <dl>
   <dt><code>bucket</code></dt>
   <dd>Specifies the bucket name.</dd>
   </dl>
3. Create the storage bucket:
   ```terminal
   $ gsutil mb gs://$BUCKET/
   ```
4. Set the `PROJECT_ID` variable to your active project:
   ```terminal
   $ PROJECT_ID=$(gcloud config get-value project)
   ```
5. Create a service account:
   ```terminal
   $ gcloud iam service-accounts create velero \
       --display-name "Velero service account"
   ```
6. List your service accounts:
   ```terminal
   $ gcloud iam service-accounts list
   ```
7. Set the `SERVICE_ACCOUNT_EMAIL` variable to match its `email` value:
   ```terminal
   $ SERVICE_ACCOUNT_EMAIL=$(gcloud iam service-accounts list \
       --filter="displayName:Velero service account" \
       --format 'value(email)')
   ```
8. Attach the policies to give the `velero` user the minimum necessary permissions:
   ```terminal
   $ ROLE_PERMISSIONS=(
       compute.disks.get
       compute.disks.create
       compute.disks.createSnapshot
       compute.snapshots.get
       compute.snapshots.create
       compute.snapshots.useReadOnly
       compute.snapshots.delete
       compute.zones.get
       storage.objects.create
       storage.objects.delete
       storage.objects.get
       storage.objects.list
       iam.serviceAccounts.signBlob
   )
   ```
9. Create the `velero.server` custom role:
   ```terminal
   $ gcloud iam roles create velero.server \
       --project $PROJECT_ID \
       --title "Velero Server" \
       --permissions "$(IFS=","; echo "${ROLE_PERMISSIONS[*]}")"
   ```
10. Add IAM policy binding to the project:
    ```terminal
    $ gcloud projects add-iam-policy-binding $PROJECT_ID \
        --member serviceAccount:$SERVICE_ACCOUNT_EMAIL \
        --role projects/$PROJECT_ID/roles/velero.server
    ```
11. Update the IAM service account:
    ```terminal
    $ gsutil iam ch serviceAccount:$SERVICE_ACCOUNT_EMAIL:objectAdmin gs://${BUCKET}
    ```
12. Save the IAM service account keys to the `credentials-velero` file in the current directory:
    ```terminal
    $ gcloud iam service-accounts keys create credentials-velero \
        --iam-account $SERVICE_ACCOUNT_EMAIL
    ```

    You use the `credentials-velero` file to create a `Secret` object for Google Cloud before you install the Data Protection Application.

## About backup and snapshot locations and their secrets {#oadp-about-backup-snapshot-locations_installing-oadp-gcp}

Review backup location, snapshot location, and secret configuration requirements for the `DataProtectionApplication` custom resource (CR). This helps you understand storage options and credential management for data protection operations.

### Backup locations {#backup-locations_installing-oadp-gcp}

You can specify one of the following AWS S3-compatible object storage solutions as a backup location:

- Multicloud Object Gateway (MCG)
- Red Hat Container Storage
- Ceph RADOS Gateway; also known as Ceph Object Gateway
- Red Hat OpenShift Data Foundation
- MinIO

Velero backs up OpenShift Container Platform resources, Kubernetes objects, and internal images as an archive file on object storage.

### Snapshot locations {#snapshot-locations_installing-oadp-gcp}

If you use your cloud provider’s native snapshot API to back up persistent volumes, you must specify the cloud provider as the snapshot location.

If you use Container Storage Interface (CSI) snapshots, you do not need to specify a snapshot location because you will create a `VolumeSnapshotClass` CR to register the CSI driver.

If you use File System Backup (FSB), you do not need to specify a snapshot location because FSB backs up the file system on object storage.

### Secrets {#secrets_installing-oadp-gcp}

If the backup and snapshot locations use the same credentials or if you do not require a snapshot location, you create a default `Secret`.

If the backup and snapshot locations use different credentials, you create two secret objects:

- Custom `Secret` for the backup location, which you specify in the `DataProtectionApplication` CR.
- Default `Secret` for the snapshot location, which is not referenced in the `DataProtectionApplication` CR.

:::warning

The Data Protection Application requires a default `Secret`. Otherwise, the installation will fail.

If you do not want to specify backup or snapshot locations during the installation, you can create a default `Secret` with an empty `credentials-velero` file.

:::

### Creating a default Secret {#oadp-creating-default-secret_installing-oadp-gcp}

You create a default `Secret` if your backup and snapshot locations use the same credentials or if you do not require a snapshot location.

The default name of the `Secret` is `cloud-credentials-gcp`.

:::note

The `DataProtectionApplication` custom resource (CR) requires a default `Secret`.  Otherwise, the installation will fail. If the name of the backup location `Secret` is not specified, the default name is used.

If you do not want to use the backup location credentials during the installation, you can create a `Secret` with the default name by using an empty `credentials-velero` file.

:::

**Prerequisites**

- Your object storage and cloud storage, if any, must use the same credentials.
- You must configure object storage for Velero.

**Procedure**

1. Create a `credentials-velero` file for the backup storage location in the appropriate format for your cloud provider.
2. Create a `Secret` custom resource (CR) with the default name:
   ```terminal
   $ oc create secret generic cloud-credentials-gcp -n openshift-adp --from-file cloud=credentials-velero
   ```

   The `Secret` is referenced in the `spec.backupLocations.credential` block of the `DataProtectionApplication` CR when you install the Data Protection Application.

### Creating secrets for different credentials {#oadp-secrets-for-different-credentials_installing-oadp-gcp}

Create separate `Secret` objects when your backup and snapshot locations require different credentials. This allows you to configure distinct authentication for each storage location while maintaining secure credential management.

**Procedure**

1. Create a `credentials-velero` file for the snapshot location in the appropriate format for your cloud provider.
2. Create a `Secret` for the snapshot location with the default name:
   ```terminal
   $ oc create secret generic cloud-credentials-gcp -n openshift-adp --from-file cloud=credentials-velero
   ```
3. Create a `credentials-velero` file for the backup location in the appropriate format for your object storage.
4. Create a `Secret` for the backup location with a custom name:
   ```terminal
   $ oc create secret generic <custom_secret> -n openshift-adp --from-file cloud=credentials-velero
   ```
5. Add the `Secret` with the custom name to the `DataProtectionApplication` CR, as in the following example:
   ```yaml
   apiVersion: oadp.openshift.io/v1alpha1
   kind: DataProtectionApplication
   metadata:
     name: <dpa_sample>
     namespace: openshift-adp
   spec:
   ...
     backupLocations:
       - velero:
           provider: gcp
           default: true
           credential:
             key: cloud
             name: <custom_secret>
           objectStorage:
             bucket: <bucket_name>
             prefix: <prefix>
     snapshotLocations:
       - velero:
           provider: gcp
           default: true
           config:
             project: <project>
             snapshotLocation: us-west1
   ```

   where:

   <dl>
   <dt><code>custom_secret</code></dt>
   <dd>Specifies the backup location <code>Secret</code> with custom name.</dd>
   </dl>

### Setting Velero CPU and memory resource allocations {#oadp-setting-resource-limits-and-requests_installing-oadp-gcp}

You set the CPU and memory resource allocations for the `Velero` pod by editing the  `DataProtectionApplication` custom resource (CR) manifest.

**Prerequisites**

- You must have the OpenShift API for Data Protection (OADP) Operator installed.

**Procedure**

- Edit the values in the `spec.configuration.velero.podConfig.ResourceAllocations` block of the `DataProtectionApplication` CR manifest, as in the following example:
  ```yaml
  apiVersion: oadp.openshift.io/v1alpha1
  kind: DataProtectionApplication
  metadata:
    name: <dpa_sample>
  spec:
  # ...
    configuration:
      velero:
        podConfig:
          nodeSelector: <node_selector>
          resourceAllocations:
            limits:
              cpu: "1"
              memory: 1024Mi
            requests:
              cpu: 200m
              memory: 256Mi
  ```

  where:

  <dl>
  <dt><code>nodeSelector</code></dt>
  <dd>Specifies the node selector to be supplied to Velero podSpec.</dd>
  <dt><code>resourceAllocations</code></dt>
  <dd>Specifies the resource allocations listed for average usage.</dd>
  </dl>

  :::note

  Kopia is an option in OADP 1.3 and later releases. You can use Kopia for file system backups, and Kopia is your only option for Data Mover cases with the built-in Data Mover.

  Kopia is more resource intensive than Restic, and you might need to adjust the CPU and memory requirements accordingly.

  :::

Use the `nodeSelector` field to select which nodes can run the node agent. The `nodeSelector` field is the simplest recommended form of node selection constraint. Any label specified must match the labels on each node.

### Enabling self-signed CA certificates {#oadp-self-signed-certificate_installing-oadp-gcp}

You must enable a self-signed CA certificate for object storage by editing the `DataProtectionApplication` custom resource (CR) manifest to prevent a `certificate signed by unknown authority` error.

**Prerequisites**

- You must have the OpenShift API for Data Protection (OADP) Operator installed.

**Procedure**

- Edit the `spec.backupLocations.velero.objectStorage.caCert` parameter and `spec.backupLocations.velero.config` parameters of the `DataProtectionApplication` CR manifest:
  ```yaml
  apiVersion: oadp.openshift.io/v1alpha1
  kind: DataProtectionApplication
  metadata:
    name: <dpa_sample>
  spec:
  # ...
    backupLocations:
      - name: default
        velero:
          provider: aws
          default: true
          objectStorage:
            bucket: <bucket>
            prefix: <prefix>
            caCert: <base64_encoded_cert_string>
          config:
            insecureSkipTLSVerify: "false"
  # ...
  ```

  where:

  <dl>
  <dt><code>caCert</code></dt>
  <dd>Specifies the Base64-encoded CA certificate string.</dd>
  <dt><code>insecureSkipTLSVerify</code></dt>
  <dd>Specifies the <code>insecureSkipTLSVerify</code> configuration. The configuration can be set to either <code>"true"</code> or <code>"false"</code>. If set to <code>"true"</code>, SSL/TLS security is disabled. If set to <code>"false"</code>, SSL/TLS security is enabled.</dd>
  </dl>

### Using CA certificates with the velero command aliased for Velero deployment {#oadp-using-ca-certificates-with-velero-command-aliased-for-velero-deployment_installing-oadp-gcp}

You might want to use the Velero CLI without installing it locally on your system by creating an alias for it.

**Prerequisites**

- You must be logged in to the OpenShift Container Platform cluster as a user with the `cluster-admin` role.
- You must have the OpenShift CLI (`oc`) installed.

**Procedure**

1. To use an aliased Velero command, run the following command:
   ```terminal
   $ alias velero='oc -n openshift-adp exec deployment/velero -c velero -it -- ./velero'
   ```
2. Check that the alias is working by running the following command:
   ```terminal
   $ velero version
   ```

   ```terminal
   Client:
   	Version: v1.12.1-OADP
   	Git commit: -
   Server:
   	Version: v1.12.1-OADP
   ```
3. To use a CA certificate with this command, you can add a certificate to the Velero deployment by running the following commands:
   ```terminal
   $ CA_CERT=$(oc -n openshift-adp get dataprotectionapplications.oadp.openshift.io <dpa-name> -o jsonpath='{.spec.backupLocations[0].velero.objectStorage.caCert}')
   ```

   ```terminal
   $ [[ -n $CA_CERT ]] && echo "$CA_CERT" | base64 -d | oc exec -n openshift-adp -i deploy/velero -c velero -- bash -c "cat > /tmp/your-cacert.txt" || echo "DPA BSL has no caCert"
   ```

   ```terminal
   $ velero describe backup <backup_name> --details --cacert /tmp/<your_cacert>.txt
   ```
4. To fetch the backup logs, run the following command:
   ```terminal
   $ velero backup logs  <backup_name>  --cacert /tmp/<your_cacert.txt>
   ```

   You can use these logs to view failures and warnings for the resources that you cannot back up.
5. If the Velero pod restarts, the `/tmp/your-cacert.txt` file disappears, and you must re-create the `/tmp/your-cacert.txt` file by re-running the commands from the previous step.
6. You can check if the `/tmp/your-cacert.txt` file still exists, in the file location where you stored it, by running the following command:
   ```terminal
   $ oc exec -n openshift-adp -i deploy/velero -c velero -- bash -c "ls /tmp/your-cacert.txt"
   /tmp/your-cacert.txt
   ```

   In a future release of OpenShift API for Data Protection (OADP), we plan to mount the certificate to the Velero pod so that this step is not required.

## Google workload identity federation cloud authentication {#oadp-gcp-wif-cloud-authentication_installing-oadp-gcp}

Applications running outside Google Cloud use service account keys, such as usernames and passwords, to gain access to Google Cloud resources. These service account keys might become a security risk if they are not properly managed.

With Google’s workload identity federation, you can use Identity and Access Management (IAM) to offer IAM roles, including the ability to impersonate service accounts, to external identities. This eliminates the maintenance and security risks associated with service account keys.

Workload identity federation handles encrypting and decrypting certificates, extracting user attributes, and validation. Identity federation externalizes authentication, passing it over to Security Token Services (STS), and reduces the demands on individual developers. Authorization and controlling access to resources remain the responsibility of the application.

:::note

Google workload identity federation is available for OADP 1.3.x and later.

:::

When backing up volumes, OADP on Google Cloud with Google workload identity federation authentication only supports CSI snapshots.

OADP on Google Cloud with Google workload identity federation authentication does not support Volume Snapshot Locations (VSL) backups. VSL backups finish with a `PartiallyFailed` phase when Google Cloud workload identity federation is configured.

If you do not use Google workload identity federation cloud authentication, continue to *Installing the Data Protection Application*.

**Prerequisites**

- You have installed a cluster in manual mode with [Google Cloud Workload Identity configured](https://docs.openshift.com/container-platform/latest/installing/installing_gcp/installing-gcp-customizations.html#installing-gcp-with-short-term-creds_installing-gcp-customizations).
- You have access to the Cloud Credential Operator utility (`ccoctl`) and to the associated workload identity pool.

**Procedure**

1. Create an `oadp-credrequest` directory by running the following command:
   ```terminal
   $ mkdir -p oadp-credrequest
   ```
2. Create a `CredentialsRequest.yaml` file as following:
   ```yaml
   echo 'apiVersion: cloudcredential.openshift.io/v1
   kind: CredentialsRequest
   metadata:
     name: oadp-operator-credentials
     namespace: openshift-cloud-credential-operator
   spec:
     providerSpec:
       apiVersion: cloudcredential.openshift.io/v1
       kind: GCPProviderSpec
       permissions:
       - compute.disks.get
       - compute.disks.create
       - compute.disks.createSnapshot
       - compute.snapshots.get
       - compute.snapshots.create
       - compute.snapshots.useReadOnly
       - compute.snapshots.delete
       - compute.zones.get
       - storage.objects.create
       - storage.objects.delete
       - storage.objects.get
       - storage.objects.list
       - iam.serviceAccounts.signBlob
       skipServiceCheck: true
     secretRef:
       name: cloud-credentials-gcp
       namespace: <OPERATOR_INSTALL_NS>
     serviceAccountNames:
     - velero
   ' > oadp-credrequest/credrequest.yaml
   ```
3. Use the `ccoctl` utility to process the `CredentialsRequest` objects in the `oadp-credrequest` directory by running the following command:
   ```terminal
   $ ccoctl gcp create-service-accounts \
       --name=<name> \
       --project=<gcp_project_id> \
       --credentials-requests-dir=oadp-credrequest \
       --workload-identity-pool=<pool_id> \
       --workload-identity-provider=<provider_id>
   ```

   The `manifests/openshift-adp-cloud-credentials-gcp-credentials.yaml` file is now available to use in the following steps.
4. Create a namespace by running the following command:
   ```terminal
   $ oc create namespace <OPERATOR_INSTALL_NS>
   ```
5. Apply the credentials to the namespace by running the following command:
   ```terminal
   $ oc apply -f manifests/openshift-adp-cloud-credentials-gcp-credentials.yaml
   ```

## Installing the Data Protection Application {#oadp-installing-dpa_installing-oadp-gcp}

You install the Data Protection Application (DPA) by creating an instance of the `DataProtectionApplication` API.

**Prerequisites**

- You must install the OADP Operator.
- You must configure object storage as a backup location.
- If you use snapshots to back up PVs, your cloud provider must support either a native snapshot API or Container Storage Interface (CSI) snapshots.
- If the backup and snapshot locations use the same credentials, you must create a `Secret` with the default name, `cloud-credentials-gcp`.
- If the backup and snapshot locations use different credentials, you must create two `Secrets`:
  - `Secret` with a custom name for the backup location. You add this `Secret` to the `DataProtectionApplication` CR.
  - `Secret` with another custom name for the snapshot location. You add this `Secret` to the `DataProtectionApplication` CR.

  :::note

  If you do not want to specify backup or snapshot locations during the installation, you can create a default `Secret` with an empty `credentials-velero` file. If there is no default `Secret`, the installation will fail.

  :::

**Procedure**

1. Click **Ecosystem** → **Installed Operators** and select the OADP Operator.
2. Under **Provided APIs**, click **Create instance** in the **DataProtectionApplication** box.
3. Click **YAML View** and update the parameters of the `DataProtectionApplication` manifest:
   ```yaml
   apiVersion: oadp.openshift.io/v1alpha1
   kind: DataProtectionApplication
   metadata:
     name: <dpa_sample>
     namespace: <OPERATOR_INSTALL_NS>
   spec:
     configuration:
       velero:
         defaultPlugins:
           - gcp
           - openshift
         resourceTimeout: 10m
       nodeAgent:
         enable: true
         uploaderType: kopia
         podConfig:
           nodeSelector: <node_selector>
     backupLocations:
       - velero:
           provider: gcp
           default: true
           credential:
             key: cloud
             name: cloud-credentials-gcp
           objectStorage:
             bucket: <bucket_name>
             prefix: <prefix>
     snapshotLocations:
       - velero:
           provider: gcp
           default: true
           config:
             project: <project>
             snapshotLocation: us-west1
           credential:
             key: cloud
             name: cloud-credentials-gcp
     backupImages: true
   ```

   where:

   <dl>
   <dt><code>namespace</code></dt>
   <dd>Specifies the default namespace for OADP which is <code>openshift-adp</code>. The namespace is a variable and is configurable.</dd>
   <dt><code>openshift</code></dt>
   <dd>Specifies that the <code>openshift</code> plugin is mandatory.</dd>
   <dt><code>resourceTimeout</code></dt>
   <dd>Specifies how many minutes to wait for several Velero resources such as Velero CRD availability, volumeSnapshot deletion, and backup repository availability, before timeout occurs. The default is 10m.</dd>
   <dt><code>nodeAgent</code></dt>
   <dd>Specifies the administrative agent that routes the administrative requests to servers.</dd>
   <dt><code>enable</code></dt>
   <dd>Set this value to <code>true</code> if you want to enable <code>nodeAgent</code> and perform File System Backup.</dd>
   <dt><code>uploaderType</code></dt>
   <dd>Specifies the uploader type. Enter <code>kopia</code> or <code>restic</code> as your uploader. You cannot change the selection after the installation. For the Built-in DataMover you must use Kopia. The <code>nodeAgent</code> deploys a daemon set, which means that the <code>nodeAgent</code> pods run on each working node. You can configure File System Backup by adding <code>spec.defaultVolumesToFsBackup: true</code> to the <code>Backup</code> CR.</dd>
   <dt><code>nodeSelector</code></dt>
   <dd>Specifies the nodes on which Kopia or Restic are available. By default, Kopia or Restic run on all nodes.</dd>
   <dt><code>key</code></dt>
   <dd>Specifies the secret key that contains credentials. For Google workload identity federation cloud authentication use <code>service_account.json</code>.</dd>
   <dt><code>name</code></dt>
   <dd>Specifies the secret name that contains credentials. If you do not specify this value, the default name, <code>cloud-credentials-gcp</code>, is used.</dd>
   <dt><code>bucket</code></dt>
   <dd>Specifies a bucket as the backup storage location. If the bucket is not a dedicated bucket for Velero backups, you must specify a prefix.</dd>
   <dt><code>prefix</code></dt>
   <dd>Specifies a prefix for Velero backups, for example, <code>velero</code>, if the bucket is used for multiple purposes.</dd>
   <dt><code>snapshotLocations</code></dt>
   <dd>Specifies a snapshot location, unless you use CSI snapshots or Restic to back up PVs.</dd>
   <dt><code>snapshotLocation</code></dt>
   <dd>Specifies that the snapshot location must be in the same region as the PVs.</dd>
   <dt><code>name</code></dt>
   <dd>Specifies the name of the <code>Secret</code> object that you created. If you do not specify this value, the default name, <code>cloud-credentials-gcp</code>, is used. If you specify a custom name, the custom name is used for the backup location.</dd>
   <dt><code>backupImages</code></dt>
   <dd>Specifies that Google workload identity federation supports internal image backup. Set this field to <code>false</code> if you do not want to use image backup.</dd>
   </dl>
4. Click **Create**.

**Verification**

1. Verify the installation by viewing the OpenShift API for Data Protection (OADP) resources by running the following command:
   ```terminal
   $ oc get all -n openshift-adp
   ```

   ```
   NAME                                                     READY   STATUS    RESTARTS   AGE
   pod/oadp-operator-controller-manager-67d9494d47-6l8z8    2/2     Running   0          2m8s
   pod/node-agent-9cq4q                                     1/1     Running   0          94s
   pod/node-agent-m4lts                                     1/1     Running   0          94s
   pod/node-agent-pv4kr                                     1/1     Running   0          95s
   pod/velero-588db7f655-n842v                              1/1     Running   0          95s

   NAME                                                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
   service/oadp-operator-controller-manager-metrics-service   ClusterIP   172.30.70.140    <none>        8443/TCP   2m8s
   service/openshift-adp-velero-metrics-svc                   ClusterIP   172.30.10.0      <none>        8085/TCP   8h

   NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
   daemonset.apps/node-agent    3         3         3       3            3           <none>          96s

   NAME                                                READY   UP-TO-DATE   AVAILABLE   AGE
   deployment.apps/oadp-operator-controller-manager    1/1     1            1           2m9s
   deployment.apps/velero                              1/1     1            1           96s

   NAME                                                           DESIRED   CURRENT   READY   AGE
   replicaset.apps/oadp-operator-controller-manager-67d9494d47    1         1         1       2m9s
   replicaset.apps/velero-588db7f655                              1         1         1       96s
   ```
2. Verify that the `DataProtectionApplication` (DPA) is reconciled by running the following command:
   ```terminal
   $ oc get dpa dpa-sample -n openshift-adp -o jsonpath='{.status}'
   ```

   ```yaml
   {"conditions":[{"lastTransitionTime":"2023-10-27T01:23:57Z","message":"Reconcile complete","reason":"Complete","status":"True","type":"Reconciled"}]}
   ```
3. Verify the `type` is set to `Reconciled`.
4. Verify the backup storage location and confirm that the `PHASE` is `Available` by running the following command:
   ```terminal
   $ oc get backupstoragelocations.velero.io -n openshift-adp
   ```

   ```yaml
   NAME           PHASE       LAST VALIDATED   AGE     DEFAULT
   dpa-sample-1   Available   1s               3d16h   true
   ```

## Configuring the DPA with client burst and QPS settings {#oadp-configuring-client-burst-qps_installing-oadp-gcp}

The burst setting determines how many requests can be sent to the `velero` server before the limit is applied. After the burst limit is reached, the queries per second (QPS) setting determines how many additional requests can be sent per second.

You can set the burst and QPS values of the `velero` server by configuring the Data Protection Application (DPA) with the burst and QPS values. You can use the `dpa.configuration.velero.client-burst` and `dpa.configuration.velero.client-qps` fields of the DPA to set the burst and QPS values.

**Prerequisites**

- You have installed the OADP Operator.

**Procedure**

- Configure the `client-burst` and the `client-qps` fields in the DPA as shown in the following example:
  ```yaml title="Example Data Protection Application"
  apiVersion: oadp.openshift.io/v1alpha1
  kind: DataProtectionApplication
  metadata:
    name: test-dpa
    namespace: openshift-adp
  spec:
    backupLocations:
      - name: default
        velero:
          config:
            insecureSkipTLSVerify: "true"
            profile: "default"
            region: <bucket_region>
            s3ForcePathStyle: "true"
            s3Url: <bucket_url>
          credential:
            key: cloud
            name: cloud-credentials
          default: true
          objectStorage:
            bucket: <bucket_name>
            prefix: velero
          provider: aws
    configuration:
      nodeAgent:
        enable: true
        uploaderType: restic
      velero:
        client-burst: 500
        client-qps: 300
        defaultPlugins:
          - openshift
          - aws
          - kubevirt
  ```

  where:

  <dl>
  <dt><code>client-burst</code></dt>
  <dd>Specifies the <code>client-burst</code> value. In this example, the <code>client-burst</code> field is set to 500.</dd>
  <dt><code>client-qps</code></dt>
  <dd>Specifies the <code>client-qps</code> value. In this example, the <code>client-qps</code> field is set to 300.</dd>
  </dl>

## Configuring node agents and node labels {#oadp-configuring-node-agents_installing-oadp-gcp}

The Data Protection Application (DPA) uses the `nodeSelector` field to select which nodes can run the node agent. The `nodeSelector` field is the recommended form of node selection constraint.

**Procedure**

1. Run the node agent on any node that you choose by adding a custom label:
   ```terminal
   $ oc label node/<node_name> node-role.kubernetes.io/nodeAgent=""
   ```

   :::note

   Any label specified must match the labels on each node.

   :::
2. Use the same custom label in the `DPA.spec.configuration.nodeAgent.podConfig.nodeSelector` field, which you used for labeling nodes:
   ```terminal
   configuration:
     nodeAgent:
       enable: true
       podConfig:
         nodeSelector:
           node-role.kubernetes.io/nodeAgent: ""
   ```

   The following example is an anti-pattern of `nodeSelector` and does not work unless both labels, `node-role.kubernetes.io/infra: ""` and `node-role.kubernetes.io/worker: ""`, are on the node:

   ```terminal
       configuration:
         nodeAgent:
           enable: true
           podConfig:
             nodeSelector:
               node-role.kubernetes.io/infra: ""
               node-role.kubernetes.io/worker: ""
   ```

## Configuring node agent load affinity {#oadp-configuring-node-agent-load-affinity_installing-oadp-gcp}

You can schedule the node agent pods on specific nodes by using the `spec.podConfig.nodeSelector` object of the `DataProtectionApplication` (DPA) custom resource (CR).

See the following example in which you can schedule the node agent pods on nodes with the label `label.io/role: cpu-1` and `other-label.io/other-role: cpu-2`.

```yaml
...
spec:
  configuration:
    nodeAgent:
      enable: true
      uploaderType: kopia
      podConfig:
        nodeSelector:
          label.io/role: cpu-1
          other-label.io/other-role: cpu-2
        ...
```

You can add more restrictions on the node agent pods scheduling by using the `nodeagent.loadAffinity` object in the DPA spec.

**Prerequisites**

- You must be logged in as a user with `cluster-admin` privileges.
- You have installed the OADP Operator.
- You have configured the DPA CR.

**Procedure**

- Configure the DPA spec `nodegent.loadAffinity` object as shown in the following example.
  In the example, you ensure that the node agent pods are scheduled only on nodes with the label `label.io/role: cpu-1` and the label `label.io/hostname` matching with either `node1` or `node2`.

  ```yaml
  ...
  spec:
    configuration:
      nodeAgent:
        enable: true
        loadAffinity:
          - nodeSelector:
              matchLabels:
                label.io/role: cpu-1
              matchExpressions:
                - key: label.io/hostname
                  operator: In
                  values:
                    - node1
                    - node2
                    ...
  ```

  where:

  <dl>
  <dt><code>loadAffinity</code></dt>
  <dd>Specifies the <code>loadAffinity</code> object by adding the <code>matchLabels</code> and <code>matchExpressions</code> objects.</dd>
  <dt><code>matchExpressions</code></dt>
  <dd>Specifies the <code>matchExpressions</code> object to add restrictions on the node agent pods scheduling.</dd>
  </dl>

## Node agent load affinity guidelines {#oadp-node-agent-load-affinity-guidelines_installing-oadp-gcp}

Use the following guidelines to configure the node agent `loadAffinity` object in the `DataProtectionApplication` (DPA) custom resource (CR).

- Use the `spec.nodeagent.podConfig.nodeSelector` object for simple node matching.
- Use the `loadAffinity.nodeSelector` object without the `podConfig.nodeSelector` object for more complex scenarios.
- You can use both `podConfig.nodeSelector` and `loadAffinity.nodeSelector` objects, but the `loadAffinity` object must be equal or more restrictive as compared to the `podConfig` object. In this scenario, the `podConfig.nodeSelector` labels must be a subset of the labels used in the `loadAffinity.nodeSelector` object.
- You cannot use the `matchExpressions` and `matchLabels` fields if you have configured both `podConfig.nodeSelector` and `loadAffinity.nodeSelector` objects in the DPA.
- See the following example to configure both `podConfig.nodeSelector` and `loadAffinity.nodeSelector` objects in the DPA.
  ```yaml
  ...
  spec:
    configuration:
      nodeAgent:
        enable: true
        uploaderType: kopia
        loadAffinity:
          - nodeSelector:
              matchLabels:
                label.io/location: 'US'
                label.io/gpu: 'no'
        podConfig:
          nodeSelector:
            label.io/gpu: 'no'
  ```

## Configuring node agent load concurrency {#oadp-configuring-node-agent-load-concurrency_installing-oadp-gcp}

You can control the maximum number of node agent operations that can run simultaneously on each node within your cluster.

You can configure it using one of the following fields of the Data Protection Application (DPA):

- `globalConfig`: Defines a default concurrency limit for the node agent across all nodes.
- `perNodeConfig`: Specifies different concurrency limits for specific nodes based on `nodeSelector` labels. This provides flexibility for environments where certain nodes might have different resource capacities or roles.

**Prerequisites**

- You must be logged in as a user with `cluster-admin` privileges.

**Procedure**

1. If you want to use load concurrency for specific nodes, add labels to those nodes:
   ```terminal
   $ oc label node/<node_name> label.io/instance-type='large'
   ```
2. Configure the load concurrency fields for your DPA instance:
   ```yaml
     configuration:
       nodeAgent:
         enable: true
         uploaderType: kopia
         loadConcurrency:
           globalConfig: 1
           perNodeConfig:
           - nodeSelector:
                 matchLabels:
                    label.io/instance-type: large
             number: 3
   ```

   where:

   <dl>
   <dt><code>globalConfig</code></dt>
   <dd>Specifies the global concurrent number. The default value is 1, which means there is no concurrency and only one load is allowed. The <code>globalConfig</code> value does not have a limit.</dd>
   <dt><code>label.io/instance-type</code></dt>
   <dd>Specifies the label for per-node concurrency.</dd>
   <dt><code>number</code></dt>
   <dd>Specifies the per-node concurrent number. You can specify many per-node concurrent numbers, for example, based on the instance type and size. The range of per-node concurrent number is the same as the global concurrent number. If the configuration file contains a per-node concurrent number and a global concurrent number, the per-node concurrent number takes precedence.</dd>
   </dl>

## Configuring the node agent as a non-root and non-privileged user {#oadp-configuring-node-agent-non-root_installing-oadp-gcp}

To enhance the node agent security, you can configure the OADP Operator node agent daemonset to run as a non-root and non-privileged user by using the `spec.configuration.velero.disableFsBackup` setting in the `DataProtectionApplication` (DPA) custom resource (CR).

By setting the `spec.configuration.velero.disableFsBackup` setting to `true`, the node agent security context sets the root file system to read-only and sets the `privileged` flag to `false`.

:::note

Setting `spec.configuration.velero.disableFsBackup` to `true` enhances the node agent security by removing the need for privileged containers and enforcing a read-only root file system.

However, it also disables File System Backup (FSB) with Kopia. If your workloads rely on FSB for backing up volumes that do not support native snapshots, then you should evaluate whether the `disableFsBackup` configuration fits your use case.

:::

**Prerequisites**

- You have installed the OADP Operator.

**Procedure**

- Configure the `disableFsBackup` field in the DPA as shown in the following example:
  ```yaml
  apiVersion: oadp.openshift.io/v1alpha1
  kind: DataProtectionApplication
  metadata:
    name: ts-dpa
    namespace: openshift-adp
  spec:
    backupLocations:
    - velero:
        credential:
          key: cloud
          name: cloud-credentials
        default: true
        objectStorage:
          bucket: <bucket_name>
          prefix: velero
        provider: gcp
    configuration:
      nodeAgent:
        enable: true
        uploaderType: kopia
      velero:
        defaultPlugins:
        - csi
        - gcp
        - openshift
        disableFsBackup: true
  ```

  where:

  <dl>
  <dt><code>nodeAgent</code></dt>
  <dd>Specifies to enable the node agent in the DPA.</dd>
  <dt><code>disableFsBackup</code></dt>
  <dd>Specifies to set the <code>disableFsBackup</code> field to <code>true</code>.</dd>
  </dl>

**Verification**

1. Verify that the node agent security context is set to run as non-root and the root file system is `readOnly` by running the following command:
   ```terminal
   $ oc get daemonset node-agent -o yaml
   ```

   The example output is as following:

   ```yaml
   apiVersion: apps/v1
   kind: DaemonSet
   metadata:
     ...
     name: node-agent
     namespace: openshift-adp
     ...
   spec:
     ...
     template:
       metadata:
         ...
       spec:
         containers:
         ...
           securityContext:
             allowPrivilegeEscalation: false
             capabilities:
               drop:
               - ALL
             privileged: false
             readOnlyRootFilesystem: true
           ...
         nodeSelector:
           kubernetes.io/os: linux
         os:
           name: linux
         restartPolicy: Always
         schedulerName: default-scheduler
         securityContext:
           runAsNonRoot: true
           seccompProfile:
             type: RuntimeDefault
         serviceAccount: velero
         serviceAccountName: velero
         ....
   ```

   where:

   <dl>
   <dt><code>allowPrivilegeEscalation</code></dt>
   <dd>Specifies that the <code>allowPrivilegeEscalation</code> field is false.</dd>
   <dt><code>privileged</code></dt>
   <dd>Specifies that the <code>privileged</code> field is false.</dd>
   <dt><code>readOnlyRootFilesystem</code></dt>
   <dd>Specifies that the root file system is read-only.</dd>
   <dt><code>runAsNonRoot</code></dt>
   <dd>Specifies that the node agent is run as a non-root user.</dd>
   </dl>

## Configuring repository maintenance {#oadp-configuring-repository-maintenance_installing-oadp-gcp}

OADP repository maintenance is a background job, you can configure it independently of the node agent pods. This means that you can schedule the repository maintenance pod on a node where the node agent is or is not running.

You can use the repository maintenance job affinity configurations in the `DataProtectionApplication` (DPA) custom resource (CR) only if you use Kopia as the backup repository.

You have the option to configure the load affinity at the global level affecting all repositories. Or you can configure the load affinity for each repository. You can also use a combination of global and per-repository configuration.

**Prerequisites**

- You must be logged in as a user with `cluster-admin` privileges.
- You have installed the OADP Operator.
- You have configured the DPA CR.

**Procedure**

- Configure the `loadAffinity` object in the DPA spec by using either one or both of the following methods:
  - Global configuration: Configure load affinity for all repositories as shown in the following example:
    ```yaml
    ...
    spec:
      configuration:
        repositoryMaintenance:
          global:
            podResources:
              cpuRequest: "100m"
              cpuLimit: "200m"
              memoryRequest: "100Mi"
              memoryLimit: "200Mi"
            loadAffinity:
              - nodeSelector:
                  matchLabels:
                    label.io/gpu: 'no'
                  matchExpressions:
                    - key: label.io/location
                      operator: In
                      values:
                        - US
                        - EU
    ```

    where:

    <dl>
    <dt><code>repositoryMaintenance</code></dt>
    <dd>Specifies the <code>repositoryMaintenance</code> object as shown in the example.</dd>
    <dt><code>global</code></dt>
    <dd>Specifies the <code>global</code> object to configure load affinity for all repositories.</dd>
    </dl>
  - Per-repository configuration: Configure load affinity per repository as shown in the following example:
    ```yaml
    ...
    spec:
      configuration:
        repositoryMaintenance:
          myrepositoryname:
            loadAffinity:
              - nodeSelector:
                  matchLabels:
                    label.io/cpu: 'yes'
    ```

    where:

    <dl>
    <dt><code>myrepositoryname</code></dt>
    <dd>Specifies the <code>repositoryMaintenance</code> object for each repository.</dd>
    </dl>

## Configuring Velero load affinity {#oadp-configuring-velero-load-affinity_installing-oadp-gcp}

With each OADP deployment, there is one Velero pod and its main purpose is to schedule Velero workloads. To schedule the Velero pod, you can use the `velero.podConfig.nodeSelector` and the `velero.loadAffinity` objects in the `DataProtectionApplication` (DPA) custom resource (CR) spec.

Use the `podConfig.nodeSelector` object to assign the Velero pod to specific nodes. You can also configure the `velero.loadAffinity` object for pod-level affinity and anti-affinity.

The OpenShift scheduler applies the rules and performs the scheduling of the Velero pod deployment.

**Prerequisites**

- You must be logged in as a user with `cluster-admin` privileges.
- You have installed the OADP Operator.
- You have configured the DPA CR.

**Procedure**

- Configure the `velero.podConfig.nodeSelector` and the `velero.loadAffinity` objects in the DPA spec as shown in the following examples:
  - `velero.podConfig.nodeSelector` object configuration:
    ```yaml
    ...
    spec:
      configuration:
        velero:
          podConfig:
            nodeSelector:
              some-label.io/custom-node-role: backup-core
    ```
  - `velero.loadAffinity` object configuration:
    ```yaml
    ...
    spec:
      configuration:
        velero:
          loadAffinity:
            - nodeSelector:
                matchLabels:
                  label.io/gpu: 'no'
                matchExpressions:
                  - key: label.io/location
                    operator: In
                    values:
                      - US
                      - EU
    ```

## Configuring a priority class for node agent and Velero pods {#oadp-configuring-priority-class_installing-oadp-gcp}

Configure a `priorityClassName` field for the node agent and Velero pods by editing the `DataProtectionApplication` (DPA) custom resource (CR). This helps you ensure that the Kubernetes scheduler prioritizes OADP pods during resource contention.

By setting a priority class, you ensure that critical OADP pods are scheduled first after events such as worker node outages, when user workloads might otherwise consume available resources.

**Prerequisites**

- The OADP Operator is installed.
- A DPA CR is configured.
- A `PriorityClass` object is created in the cluster.

**Procedure**

1. In your DPA CR, configure the `priorityClassName` field for the node agent, Velero, or both, in the `podConfig` object as shown in the following example:
   ```yaml
   apiVersion: oadp.openshift.io/v1alpha1
   kind: DataProtectionApplication
   metadata:
     name: <dpa_name>
     namespace: openshift-adp
   spec:
   # ...
     configuration:
       nodeAgent:
         enable: true
         uploaderType: kopia
         podConfig:
           priorityClassName: <priority_class_name>
       velero:
         podConfig:
           priorityClassName: <priority_class_name>
   ```

   where:

   <dl>
   <dt><code>&lt;priority_class_name&gt;</code></dt>
   <dd>Specifies the name of an existing <code>PriorityClass</code> to apply to the pods managed by this <code>podConfig</code> object. For example, <code>system-cluster-critical</code>.</dd>
   </dl>
2. Apply the `DataProtectionApplication` CR by running the following command:
   ```terminal
   $ oc apply -f <dpa_file_name>
   ```

**Verification**

1. Verify that the node agent daemon set pods have the correct priority class by running the following command:
   ```terminal
   $ oc get pods -n openshift-adp -l name=node-agent -o jsonpath='{range .items[*]}{.metadata.name}{" "}{.spec.priorityClassName}{"\n"}{end}'
   ```

   ```terminal
   node-agent-xxxxx <priority_class_name>
   node-agent-yyyyy <priority_class_name>
   ```
2. Verify that the Velero deployment pods have the correct priority class by running the following command:
   ```terminal
   $ oc get pods -n openshift-adp -l app.kubernetes.io/name=velero -o jsonpath='{range .items[*]}{.metadata.name}{" "}{.spec.priorityClassName}{"\n"}{end}'
   ```

   ```terminal
   velero-xxxxx <priority_class_name>
   ```

## Overriding the imagePullPolicy setting in the DPA  {#oadp-configuring-imagepullpolicy_installing-oadp-gcp}

In OADP 1.4.0 or earlier, the Operator sets the `imagePullPolicy` field of the Velero and node agent pods to `Always` for all images.

In OADP 1.4.1 or later, the Operator first checks if each image has the `sha256` or `sha512` digest and sets the `imagePullPolicy` field accordingly:

- If the image has the digest, the Operator sets `imagePullPolicy` to `IfNotPresent`.
- If the image does not have the digest, the Operator sets `imagePullPolicy` to `Always`.

You can also override the `imagePullPolicy` field by using the `spec.imagePullPolicy` field in the Data Protection Application (DPA).

**Prerequisites**

- You have installed the OADP Operator.

**Procedure**

- Configure the `spec.imagePullPolicy` field in the DPA as shown in the following example:
  ```yaml title="Example Data Protection Application"
  apiVersion: oadp.openshift.io/v1alpha1
  kind: DataProtectionApplication
  metadata:
    name: test-dpa
    namespace: openshift-adp
  spec:
    backupLocations:
      - name: default
        velero:
          config:
            insecureSkipTLSVerify: "true"
            profile: "default"
            region: <bucket_region>
            s3ForcePathStyle: "true"
            s3Url: <bucket_url>
          credential:
            key: cloud
            name: cloud-credentials
          default: true
          objectStorage:
            bucket: <bucket_name>
            prefix: velero
          provider: aws
    configuration:
      nodeAgent:
        enable: true
        uploaderType: kopia
      velero:
        defaultPlugins:
          - openshift
          - aws
          - kubevirt
          - csi
    imagePullPolicy: Never
  ```

  where:

  <dl>
  <dt><code>imagePullPolicy</code></dt>
  <dd>Specifies the value for <code>imagePullPolicy</code>. In this example, the <code>imagePullPolicy</code> field is set to <code>Never</code>.</dd>
  </dl>

### Enabling CSI in the DataProtectionApplication CR {#oadp-enabling-csi-dpa_installing-oadp-gcp}

You enable the Container Storage Interface (CSI) in the `DataProtectionApplication` custom resource (CR) in order to back up persistent volumes with CSI snapshots.

**Prerequisites**

- The cloud provider must support CSI snapshots.

**Procedure**

- Edit the `DataProtectionApplication` CR, as in the following example:
  ```yaml
  apiVersion: oadp.openshift.io/v1alpha1
  kind: DataProtectionApplication
  ...
  spec:
    configuration:
      velero:
        defaultPlugins:
        - openshift
        - csi
  ```

  where:

  <dl>
  <dt><code>csi</code></dt>
  <dd>Specifies the <code>csi</code> default plugin.</dd>
  </dl>

### Disabling the node agent in DataProtectionApplication {#oadp-about-disable-node-agent-dpa_installing-oadp-gcp}

If you are not using `Restic`, `Kopia`, or `DataMover` for your backups, you can disable the `nodeAgent` field in the `DataProtectionApplication` custom resource (CR). Before you disable `nodeAgent`, ensure the OADP Operator is idle and not running any backups.

**Procedure**

1. To disable the `nodeAgent`, set the `enable` flag to `false`. See the following example:
   ```yaml title="Example DataProtectionApplication CR"
   # ...
   configuration:
     nodeAgent:
       enable: false
       uploaderType: kopia
   # ...
   ```

   where:

   <dl>
   <dt><code>enable</code></dt>
   <dd>Enables the node agent.</dd>
   </dl>
2. To enable the `nodeAgent`, set the `enable` flag to `true`. See the following example:
   ```yaml title="Example DataProtectionApplication CR"
   # ...
   configuration:
     nodeAgent:
       enable: true
       uploaderType: kopia
   # ...
   ```

   where:

   <dl>
   <dt><code>enable</code></dt>
   <dd>Enables the node agent. You can set up a job to enable and disable the <code>nodeAgent</code> field in the <code>DataProtectionApplication</code> CR. For more information, see "Running tasks in pods using jobs".</dd>
   </dl>

**Additional resources**

- [Velero 1.16](https://velero.io/docs/v1.16/)
- [Installing the OADP Operator](/docs/backup_and_restore/application_backup_and_restore/installing/oadp-installing-operator#oadp-installing-operator-doc)
- [Using Operator Lifecycle Manager in disconnected environments](/docs/disconnected/using-olm#olm-restricted-networks)
- [Installing the Data Protection Application with the `kubevirt` and `openshift` plugins](/docs/backup_and_restore/application_backup_and_restore/installing/installing-oadp-kubevirt#oadp-installing-dpa_installing-oadp-kubevirt)
- [Running tasks in pods using jobs](/docs/nodes/jobs/nodes-nodes-jobs#nodes-nodes-jobs)
- [Configuring the OpenShift API for Data Protection (OADP) with multiple backup storage locations](/docs/backup_and_restore/application_backup_and_restore/installing/configuring-oadp-multiple-bsl#configuring-oadp-multiple-bsl)
