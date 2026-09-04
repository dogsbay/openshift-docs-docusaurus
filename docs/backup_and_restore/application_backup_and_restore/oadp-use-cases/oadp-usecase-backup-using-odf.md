---
title: "Backup using OpenShift API for Data Protection and Red&#160;Hat OpenShift Data Foundation (ODF)"
sidebar_position: 1
---

# Backup using OpenShift API for Data Protection and Red Hat OpenShift Data Foundation (ODF)  {#oadp-use-cases-backup}

<a id="oadp-use-cases-backup"></a>

Following is a use case for using OADP and ODF to back up an application.

## Backing up an application using OADP and ODF {#oadp-usecase-backup-odf_oadp-use-cases-backup}

In this use case, you back up an application by using OADP and store the backup in an object storage provided by Red Hat OpenShift Data Foundation (ODF).

- You create an object bucket claim (OBC) to configure the backup storage location. You use ODF to configure an Amazon S3-compatible object storage bucket. ODF provides MultiCloud Object Gateway (NooBaa MCG) and Ceph Object Gateway, also known as RADOS Gateway (RGW), object storage service. In this use case, you use NooBaa MCG as the backup storage location.
- You use the NooBaa MCG service with OADP by using the `aws` provider plugin.
- You configure the Data Protection Application (DPA) with the backup storage location (BSL).
- You create a backup custom resource (CR) and specify the application namespace to back up.
- You create and verify the backup.

**Prerequisites**

- You installed the OADP Operator.
- You installed the ODF Operator.
- You have an application with a database running in a separate namespace.

**Procedure**

1. Create an OBC manifest file to request a NooBaa MCG bucket as shown in the following example:
   ```yaml
   apiVersion: objectbucket.io/v1alpha1
   kind: ObjectBucketClaim
   metadata:
     name: test-obc
     namespace: openshift-adp
   spec:
     storageClassName: openshift-storage.noobaa.io
     generateBucketName: test-backup-bucket
   ```

   where:

   <dl>
   <dt><code>test-obc</code></dt>
   <dd>Specifies the name of the object bucket claim.</dd>
   <dt><code>test-backup-bucket</code></dt>
   <dd>Specifies the name of the bucket.</dd>
   </dl>
2. Create the OBC by running the following command:
   ```terminal
   $ oc create -f <obc_file_name>
   ```

   where:

   <dl>
   <dt><code>&lt;obc_file_name&gt;</code></dt>
   <dd>Specifies the file name of the object bucket claim manifest.</dd>
   </dl>
3. When you create an OBC, ODF creates a `secret` and a `config map` with the same name as the object bucket claim. The `secret` has the bucket credentials, and the `config map` has information to access the bucket. To get the bucket name and bucket host from the generated config map, run the following command:
   ```terminal
   $ oc extract --to=- cm/test-obc
   ```

   `test-obc` is the name of the OBC.

   ```terminal title="Example output"
   # BUCKET_NAME
   backup-c20...41fd
   # BUCKET_PORT
   443
   # BUCKET_REGION

   # BUCKET_SUBREGION

   # BUCKET_HOST
   s3.openshift-storage.svc
   ```
4. To get the bucket credentials from the generated `secret`, run the following command:
   ```terminal
   $ oc extract --to=- secret/test-obc
   ```

   ```terminal title="Example output"
   # AWS_ACCESS_KEY_ID
   ebYR....xLNMc
   # AWS_SECRET_ACCESS_KEY
   YXf...+NaCkdyC3QPym
   ```
5. Get the public URL for the S3 endpoint from the s3 route in the `openshift-storage` namespace by running the following command:
   ```terminal
   $ oc get route s3 -n openshift-storage
   ```
6. Create a `cloud-credentials` file with the object bucket credentials as shown in the following command:
   ```terminal
   [default]
   aws_access_key_id=<AWS_ACCESS_KEY_ID>
   aws_secret_access_key=<AWS_SECRET_ACCESS_KEY>
   ```
7. Create the `cloud-credentials` secret with the `cloud-credentials` file content as shown in the following command:
   ```terminal
   $ oc create secret generic \
     cloud-credentials \
     -n openshift-adp \
     --from-file cloud=cloud-credentials
   ```
8. Configure the Data Protection Application (DPA) as shown in the following example:
   ```yaml
   apiVersion: oadp.openshift.io/v1alpha1
   kind: DataProtectionApplication
   metadata:
     name: oadp-backup
     namespace: openshift-adp
   spec:
     configuration:
       nodeAgent:
         enable: true
         uploaderType: kopia
       velero:
         defaultPlugins:
           - aws
           - openshift
           - csi
         defaultSnapshotMoveData: true
     backupLocations:
       - velero:
           config:
             profile: "default"
             region: noobaa
             s3Url: https://s3.openshift-storage.svc
             s3ForcePathStyle: "true"
             insecureSkipTLSVerify: "true"
           provider: aws
           default: true
           credential:
             key: cloud
             name:  cloud-credentials
           objectStorage:
             bucket: <bucket_name>
             prefix: oadp
   ```

   where:

   <dl>
   <dt><code>defaultSnapshotMoveData</code></dt>
   <dd>Set to <code>true</code> to use the OADP Data Mover to enable movement of Container Storage Interface (CSI) snapshots to a remote object storage.</dd>
   <dt><code>s3Url</code></dt>
   <dd>Specifies the S3 URL of ODF storage.</dd>
   <dt><code>&lt;bucket_name&gt;</code></dt>
   <dd>Specifies the bucket name.</dd>
   </dl>
9. Create the DPA by running the following command:
   ```terminal
   $ oc apply -f <dpa_filename>
   ```
10. Verify that the DPA is created successfully by running the following command. In the example output, you can see the `status` object has `type` field set to `Reconciled`. This means, the DPA is successfully created.
    ```terminal
    $ oc get dpa -o yaml
    ```

    ```yaml title="Example output"
    apiVersion: v1
    items:
    - apiVersion: oadp.openshift.io/v1alpha1
      kind: DataProtectionApplication
      metadata:
        namespace: openshift-adp
        #...#
      spec:
        backupLocations:
        - velero:
            config:
              #...#
      status:
        conditions:
        - lastTransitionTime: "20....9:54:02Z"
          message: Reconcile complete
          reason: Complete
          status: "True"
          type: Reconciled
    kind: List
    metadata:
      resourceVersion: ""
    ```
11. Verify that the backup storage location (BSL) is available by running the following command:
    ```terminal
    $ oc get backupstoragelocations.velero.io -n openshift-adp
    ```

    ```terminal title="Example output"
    NAME           PHASE       LAST VALIDATED   AGE   DEFAULT
    dpa-sample-1   Available   3s               15s   true
    ```
12. Configure a backup CR as shown in the following example:
    ```yaml
    apiVersion: velero.io/v1
    kind: Backup
    metadata:
      name: test-backup
      namespace: openshift-adp
    spec:
      includedNamespaces:
      - <application_namespace>
    ```

    where:

    <dl>
    <dt><code>&lt;application_namespace&gt;</code></dt>
    <dd>Specifies the namespace for the application to back up.</dd>
    </dl>
13. Create the backup CR by running the following command:
    ```terminal
    $ oc apply -f <backup_cr_filename>
    ```

**Verification**

- Verify that the backup object is in the `Completed` phase by running the following command. For more details, see the example output.
  ```terminal
  $ oc describe backup test-backup -n openshift-adp
  ```

  ```terminal title="Example output"
  Name:         test-backup
  Namespace:    openshift-adp
  # ....#
  Status:
    Backup Item Operations Attempted:  1
    Backup Item Operations Completed:  1
    Completion Timestamp:              2024-09-25T10:17:01Z
    Expiration:                        2024-10-25T10:16:31Z
    Format Version:                    1.1.0
    Hook Status:
    Phase:  Completed
    Progress:
      Items Backed Up:  34
      Total Items:      34
    Start Timestamp:    2024-09-25T10:16:31Z
    Version:            1
  Events:               <none>
  ```
