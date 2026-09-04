---
title: "Overriding Kopia hashing, encryption, and splitter algorithms"
sidebar_position: 4
---

# Overriding Kopia hashing, encryption, and splitter algorithms {#overriding-kopia-algorithms}

<a id="overriding-kopia-algorithms"></a>

Override the default values of Kopia hashing, encryption, and splitter algorithms by using specific environment variables in the Data Protection Application (DPA).

## Configuring the DPA to override Kopia hashing, encryption, and splitter algorithms {#oadp-kopia-configuring-algorithms_overriding-kopia-algorithms}

Configure the Data Protection Application (DPA) to override the default Kopia hashing, encryption, and splitter algorithms by setting environment variables in the Velero pod configuration. This helps you improve Kopia performance and compare performance metrics for your backup operations.

:::note

The configuration of the Kopia algorithms for splitting, hashing, and encryption in the Data Protection Application (DPA) apply only during the initial Kopia repository creation, and cannot be changed later.

To use different Kopia algorithms, ensure that the object storage does not contain any previous Kopia repositories of backups. Configure a new object storage in the Backup Storage Location (BSL) or specify a unique prefix for the object storage in the BSL configuration.

:::

**Prerequisites**

- You have installed the OADP Operator.
- You have created the secret by using the credentials provided by the cloud provider.

**Procedure**

- Configure the DPA with the environment variables for hashing, encryption, and splitter as shown in the following example.
  ```yaml
  apiVersion: oadp.openshift.io/v1alpha1
  kind: DataProtectionApplication
  #...
  configuration:
    nodeAgent:
      enable: true
      uploaderType: kopia
    velero:
      defaultPlugins:
      - openshift
      - aws
      - csi
      defaultSnapshotMoveData: true
      podConfig:
        env:
          - name: KOPIA_HASHING_ALGORITHM
            value: <hashing_algorithm_name>
          - name: KOPIA_ENCRYPTION_ALGORITHM
            value: <encryption_algorithm_name>
          - name: KOPIA_SPLITTER_ALGORITHM
            value: <splitter_algorithm_name>
  ```

  where:

  <dl>
  <dt><code>enable</code></dt>
  <dd>Set to <code>true</code> to enable the <code>nodeAgent</code>.</dd>
  <dt><code>uploaderType</code></dt>
  <dd>Specifies the uploader type as <code>kopia</code>.</dd>
  <dt><code>csi</code></dt>
  <dd>Include the <code>csi</code> plugin.</dd>
  <dt><code>&lt;hashing_algorithm_name&gt;</code></dt>
  <dd>Specifies a hashing algorithm. For example, <code>BLAKE3-256</code>.</dd>
  <dt><code>&lt;encryption_algorithm_name&gt;</code></dt>
  <dd>Specifies an encryption algorithm. For example, <code>CHACHA20-POLY1305-HMAC-SHA256</code>.</dd>
  <dt><code>&lt;splitter_algorithm_name&gt;</code></dt>
  <dd>Specifies a splitter algorithm. For example, <code>DYNAMIC-8M-RABINKARP</code>.</dd>
  </dl>

## Use case for overriding Kopia hashing, encryption, and splitter algorithms {#oadp-usecase-kopia-override-algorithms_overriding-kopia-algorithms}

Back up an application by using Kopia environment variables for hashing, encryption, and splitter. Store the backup in an AWS S3 bucket and verify the environment variables by connecting to the Kopia repository.

**Prerequisites**

- You have installed the OADP Operator.
- You have an AWS S3 bucket configured as the backup storage location.
- You have created the secret by using the credentials provided by the cloud provider.
- You have installed the Kopia client.
- You have an application with persistent volumes running in a separate namespace.

**Procedure**

1. Configure the Data Protection Application (DPA) as shown in the following example:
   ```yaml
   apiVersion: oadp.openshift.io/v1alpha1
   kind: DataProtectionApplication
   metadata:
   name: <dpa_name>
   namespace: openshift-adp
   spec:
   backupLocations:
   - name: aws
     velero:
       config:
         profile: default
         region: <region_name>
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
       - csi
       defaultSnapshotMoveData: true
       podConfig:
         env:
           - name: KOPIA_HASHING_ALGORITHM
             value: BLAKE3-256
           - name: KOPIA_ENCRYPTION_ALGORITHM
             value: CHACHA20-POLY1305-HMAC-SHA256
           - name: KOPIA_SPLITTER_ALGORITHM
             value: DYNAMIC-8M-RABINKARP
   ```

   where:

   <dl>
   <dt><code>&lt;dpa_name&gt;</code></dt>
   <dd>Specifies a name for the DPA.</dd>
   <dt><code>&lt;region_name&gt;</code></dt>
   <dd>Specifies the region for the backup storage location.</dd>
   <dt><code>cloud-credentials</code></dt>
   <dd>Specifies the name of the default <code>Secret</code> object.</dd>
   <dt><code>&lt;bucket_name&gt;</code></dt>
   <dd>Specifies the AWS S3 bucket name.</dd>
   <dt><code>csi</code></dt>
   <dd>Include the <code>csi</code> plugin.</dd>
   <dt><code>BLAKE3-256</code></dt>
   <dd>Specifies the hashing algorithm as <code>BLAKE3-256</code>.</dd>
   <dt><code>CHACHA20-POLY1305-HMAC-SHA256</code></dt>
   <dd>Specifies the encryption algorithm as <code>CHACHA20-POLY1305-HMAC-SHA256</code>.</dd>
   <dt><code>DYNAMIC-8M-RABINKARP</code></dt>
   <dd>Specifies the splitter algorithm as <code>DYNAMIC-8M-RABINKARP</code>.</dd>
   </dl>
2. Create the DPA by running the following command:
   ```terminal
   $ oc create -f <dpa_file_name>
   ```

   Replace `<dpa_file_name>` with the file name of the DPA you configured.
3. Verify that the DPA has reconciled by running the following command:
   ```terminal
   $ oc get dpa -o yaml
   ```
4. Create a backup CR as shown in the following example:
   ```yaml
   apiVersion: velero.io/v1
   kind: Backup
   metadata:
     name: test-backup
     namespace: openshift-adp
   spec:
     includedNamespaces:
     - <application_namespace>
     defaultVolumesToFsBackup: true
   ```

   Replace `<application_namespace>` with the namespace for the application installed in the cluster.
5. Create a backup by running the following command:
   ```terminal
   $ oc apply -f <backup_file_name>
   ```

   Replace `<backup_file_name>` with the name of the backup CR file.
6. Verify that the backup completed by running the following command:
   ```terminal
   $ oc get backups.velero.io <backup_name> -o yaml
   ```

   Replace `<backup_name>` with the name of the backup.

**Verification**

1. Connect to the Kopia repository by running the following command:
   ```terminal
   $ kopia repository connect s3 \
     --bucket=<bucket_name> \
     --prefix=velero/kopia/<application_namespace> \
     --password=static-passw0rd \
     --access-key="<aws_s3_access_key>" \
     --secret-access-key="<aws_s3_secret_access_key>"
   ```

   where:

   <dl>
   <dt><code>&lt;bucket_name&gt;</code></dt>
   <dd>Specifies the AWS S3 bucket name.</dd>
   <dt><code>&lt;application_namespace&gt;</code></dt>
   <dd>Specifies the namespace for the application.</dd>
   <dt><code>static-passw0rd</code></dt>
   <dd>This is the Kopia password to connect to the repository.</dd>
   <dt><code>&lt;aws_s3_access_key&gt;</code></dt>
   <dd>Specifies the AWS S3 access key.</dd>
   <dt><code>&lt;aws_s3_secret_access_key&gt;</code></dt>
   <dd>Specifies the AWS S3 storage provider secret access key.</dd>
   </dl>

If you are using a storage provider other than AWS S3, you will need to add `--endpoint`, the bucket endpoint URL parameter, to the command.

1. Verify that Kopia uses the environment variables that are configured in the DPA for the backup by running the following command:
   ```terminal
   $ kopia repository status
   ```

**Example output**

```terminal

Hash:                BLAKE3-256
Encryption:          CHACHA20-POLY1305-HMAC-SHA256
Splitter:            DYNAMIC-8M-RABINKARP
Format version:      3
```

## Benchmarking Kopia hashing, encryption, and splitter algorithms {#oadp-kopia-algorithms-benchmarking_overriding-kopia-algorithms}

Run Kopia commands to benchmark the hashing, encryption, and splitter algorithms. Based on the benchmarking results, you can select the most suitable algorithm for your workload. You run the Kopia benchmarking commands from a pod on the cluster. The benchmarking results can vary depending on CPU speed, available RAM, disk speed, current I/O load, and so on.

:::note

The configuration of the Kopia algorithms for splitting, hashing, and encryption in the Data Protection Application (DPA) apply only during the initial Kopia repository creation, and cannot be changed later.

To use different Kopia algorithms, ensure that the object storage does not contain any previous Kopia repositories of backups. Configure a new object storage in the Backup Storage Location (BSL) or specify a unique prefix for the object storage in the BSL configuration.

:::

**Prerequisites**

- You have installed the OADP Operator.
- You have an application with persistent volumes running in a separate namespace.
- You have run a backup of the application with Container Storage Interface (CSI) snapshots.

**Procedure**

1. Configure the `must-gather` pod as shown in the following example. Make sure you are using the `oadp-mustgather` image for OADP version 1.3 and later.
   ```yaml title="Example pod configuration"
   apiVersion: v1
   kind: Pod
   metadata:
     name: oadp-mustgather-pod
     labels:
       purpose: user-interaction
   spec:
     containers:
     - name: oadp-mustgather-container
       image: registry.redhat.io/oadp/oadp-mustgather-rhel9:v1.3
       command: ["sleep"]
       args: ["infinity"]
   ```

   The Kopia client is available in the `oadp-mustgather` image.
2. Create the pod by running the following command:
   ```terminal
   $ oc apply -f <pod_config_file_name>
   ```

   Replace `<pod_config_file_name>` with the name of the YAML file for the pod configuration.
3. Verify that the Security Context Constraints (SCC) on the pod is `anyuid`, so that Kopia can connect to the repository.
   ```terminal
   $ oc describe pod/oadp-mustgather-pod | grep scc
   ```

   ```terminal title="Example output"
   openshift.io/scc: anyuid
   ```
4. Connect to the pod via SSH by running the following command:
   ```terminal
   $ oc -n openshift-adp rsh pod/oadp-mustgather-pod
   ```
5. Connect to the Kopia repository by running the following command:
   ```terminal
   sh-5.1# kopia repository connect s3 \
     --bucket=<bucket_name> \
     --prefix=velero/kopia/<application_namespace> \
     --password=static-passw0rd \
     --access-key="<access_key>" \
     --secret-access-key="<secret_access_key>" \
     --endpoint=<bucket_endpoint>
   ```

   where:

   <dl>
   <dt><code>&lt;bucket_name&gt;</code></dt>
   <dd>Specifies the object storage provider bucket name.</dd>
   <dt><code>&lt;application_namespace&gt;</code></dt>
   <dd>Specifies the namespace for the application.</dd>
   <dt><code>static-passw0rd</code></dt>
   <dd>This is the Kopia password to connect to the repository.</dd>
   <dt><code>&lt;access_key&gt;</code></dt>
   <dd>Specifies the object storage provider access key.</dd>
   <dt><code>&lt;secret_access_key&gt;</code></dt>
   <dd>Specifies the object storage provider secret access key.</dd>
   <dt><code>&lt;bucket_endpoint&gt;</code></dt>
   <dd>Specifies the bucket endpoint. You do not need to specify the bucket endpoint, if you are using AWS S3 as the storage provider.</dd>
   </dl>

This is an example command. The command can vary based on the object storage provider.

1. To benchmark the hashing algorithm, run the following command:
   ```terminal
   sh-5.1# kopia benchmark hashing
   ```

   ```terminal title="Example output"
   Benchmarking hash 'BLAKE2B-256' (100 x 1048576 bytes, parallelism 1)
   Benchmarking hash 'BLAKE2B-256-128' (100 x 1048576 bytes, parallelism 1)

   Fastest option for this machine is: --block-hash=BLAKE3-256
   ```
2. To benchmark the encryption algorithm, run the following command:
   ```terminal
   sh-5.1# kopia benchmark encryption
   ```

   ```terminal title="Example output"
   Benchmarking encryption 'AES256-GCM-HMAC-SHA256'
   Benchmarking encryption 'CHACHA20-POLY1305-HMAC-SHA256'

   Fastest option for this machine is: --encryption=AES256-GCM-HMAC-SHA256
   ```
3. To benchmark the splitter algorithm, run the following command:
   ```terminal
   sh-5.1# kopia benchmark splitter
   ```

   ```terminal title="Example output"
   splitting 16 blocks of 32MiB each, parallelism 1
   DYNAMIC                     747.6 MB/s count:107 min:9467 10th:2277562 25th:2971794 50th:4747177 75th:7603998 90th:8388608 max:8388608
   DYNAMIC-128K-BUZHASH        718.5 MB/s count:3183 min:3076 10th:80896 25th:104312 50th:157621 75th:249115 90th:262144 max:262144
   DYNAMIC-128K-RABINKARP      164.4 MB/s count:3160 min:9667 10th:80098 25th:106626 50th:162269 75th:250655 90th:262144 max:262144
   ```
