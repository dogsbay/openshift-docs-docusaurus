---
title: Backing up 3scale API Management by using OADP
sidebar_position: 2
---

# Backing up 3scale API Management by using OADP

<a id="backing-up-3scale-api-management-by-using-oadp"></a>

Back up Red Hat 3scale API Management components, including the 3scale Operator, MySQL database, and Redis database, by using OpenShift API for Data Protection (OADP). This helps you protect your API management infrastructure and provides recovery in case of data loss.

For more information about installing and configuring Red Hat 3scale API Management, see *Installing 3scale API Management on OpenShift* and *Red Hat 3scale API Management*.

## Creating the Data Protection Application {#creating-the-data-protection-application_backing-up-3scale-api-management-by-using-oadp}

Create a Data Protection Application (DPA) custom resource (CR) to configure backup storage and Velero settings for Red Hat 3scale API Management. This helps you set up the backup infrastructure required for protecting your 3scale components.

**Procedure**

1. Create a YAML file with the following configuration:
   ```yaml
   apiVersion: oadp.openshift.io/v1alpha1
   kind: DataProtectionApplication
   metadata:
     name: dpa-sample
     namespace: openshift-adp
   spec:
     configuration:
       velero:
         defaultPlugins:
           - openshift
           - aws
           - csi
         resourceTimeout: 10m
       nodeAgent:
         enable: true
         uploaderType: kopia
     backupLocations:
       - name: default
         velero:
           provider: aws
           default: true
           objectStorage:
             bucket: <bucket_name>
             prefix: <prefix>
           config:
             region: <region>
             profile: "default"
             s3ForcePathStyle: "true"
             s3Url: <s3_url>
           credential:
             key: cloud
             name: cloud-credentials
   ```

   where:

   <dl>
   <dt><code>&lt;bucket_name&gt;</code></dt>
   <dd>Specifies a bucket as the backup storage location. If the bucket is not a dedicated bucket for Velero backups, you must specify a prefix.</dd>
   <dt><code>&lt;prefix&gt;</code></dt>
   <dd>Specifies a prefix for Velero backups, for example, <code>velero</code>, if the bucket is used for multiple purposes.</dd>
   <dt><code>&lt;region&gt;</code></dt>
   <dd>Specifies a region for backup storage location.</dd>
   <dt><code>&lt;s3_url&gt;</code></dt>
   <dd>Specifies the URL of the object store that you are using to store backups.</dd>
   </dl>
2. Create the DPA CR by running the following command:
   ```terminal
   $ oc create -f dpa.yaml
   ```

## Backing up the 3scale API Management operator, secret, and APIManager {#backing-up-the-3scale-operator-secret-apimanager_backing-up-3scale-api-management-by-using-oadp}

Back up the Red Hat 3scale API Management operator resources, including the `Secret` and APIManager custom resources (CRs), by creating backup CRs. This helps you preserve your 3scale operator configuration for recovery scenarios.

**Prerequisites**

- You created the Data Protection Application (DPA).

**Procedure**

1. Back up your 3scale operator CRs, such as `operatorgroup`, `namespaces`, and `subscriptions`, by creating a YAML file with the following configuration:
   ```yaml
   apiVersion: velero.io/v1
   kind: Backup
   metadata:
     name: operator-install-backup
     namespace: openshift-adp
   spec:
     csiSnapshotTimeout: 10m0s
     defaultVolumesToFsBackup: false
     includedNamespaces:
     - threescale
     includedResources:
     - operatorgroups
     - subscriptions
     - namespaces
     itemOperationTimeout: 1h0m0s
     snapshotMoveData: false
     ttl: 720h0m0s
   ```

   where:

   <dl>
   <dt><code>operator-install-backup</code></dt>
   <dd>Specifies the value of the <code>metadata.name</code> parameter in the backup. This is the same value used in the <code>metadata.backupName</code> parameter used when restoring the 3scale operator.</dd>
   <dt><code>threescale</code></dt>
   <dd>Specifies the namespace where the 3scale operator is installed.</dd>
   </dl>

   :::note

   You can also back up and restore `ReplicationControllers`, `Deployment`, and `Pod` objects to ensure that all manually set environments are backed up and restored. This does not affect the flow of restoration.

   :::
2. Create a backup CR by running the following command:
   ```terminal
   $ oc create -f backup.yaml
   ```

   ```terminal title="Example output"
   backup.velero.io/operator-install-backup created
   ```
3. Back up the `Secret` CR by creating a YAML file with the following configuration:
   ```yaml
   apiVersion: velero.io/v1
   kind: Backup
   metadata:
     name: operator-resources-secrets
     namespace: openshift-adp
   spec:
     csiSnapshotTimeout: 10m0s
     defaultVolumesToFsBackup: false
     includedNamespaces:
     - threescale
     includedResources:
     - secrets
     itemOperationTimeout: 1h0m0s
     labelSelector:
       matchLabels:
         app: 3scale-api-management
     snapshotMoveData: false
     snapshotVolumes: false
     ttl: 720h0m0s
   ```

   <dl>
   <dt><code>name</code></dt>
   <dd>Specifies the value of the <code>metadata.name</code> parameter in the backup. Use this value in the <code>metadata.backupName</code> parameter when restoring the <code>Secret</code>.</dd>
   </dl>
4. Create the `Secret` backup CR by running the following command:
   ```terminal
   $ oc create -f backup-secret.yaml
   ```

   ```terminal title="Example output"
   backup.velero.io/operator-resources-secrets created
   ```
5. Back up the APIManager CR by creating a YAML file with the following configuration:
   ```yaml
   apiVersion: velero.io/v1
   kind: Backup
   metadata:
     name: operator-resources-apim
     namespace: openshift-adp
   spec:
     csiSnapshotTimeout: 10m0s
     defaultVolumesToFsBackup: false
     includedNamespaces:
     - threescale
     includedResources:
     - apimanagers
     itemOperationTimeout: 1h0m0s
     snapshotMoveData: false
     snapshotVolumes: false
     storageLocation: ts-dpa-1
     ttl: 720h0m0s
     volumeSnapshotLocations:
     - ts-dpa-1
   ```

   <dl>
   <dt><code>name</code></dt>
   <dd>Specifies the value of the <code>metadata.name</code> parameter in the backup. Use this value in the <code>metadata.backupName</code> parameter when restoring the APIManager.</dd>
   </dl>
6. Create the APIManager CR by running the following command:
   ```terminal
   $ oc create -f backup-apimanager.yaml
   ```

   ```terminal title="Example output"
   backup.velero.io/operator-resources-apim created
   ```

## Backing up a MySQL database {#backing-up-the-mysql-database_backing-up-3scale-api-management-by-using-oadp}

Back up a MySQL database by creating a persistent volume claim (PVC) to store the database dump. This helps you preserve your 3scale system database data for recovery scenarios.

**Prerequisites**

- You have backed up the Red Hat 3scale API Management operator.

**Procedure**

1. Create a YAML file with the following configuration for adding an additional PVC:
   ```yaml
   kind: PersistentVolumeClaim
   apiVersion: v1
   metadata:
     name: example-claim
     namespace: threescale
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 1Gi
     storageClassName: gp3-csi
     volumeMode: Filesystem
   ```
2. Create the additional PVC by running the following command:
   ```terminal
   $ oc create -f ts_pvc.yml
   ```
3. Attach the PVC to the system database pod by editing the `system-mysql` deployment to use the MySQL dump:
   ```terminal
   $ oc edit deployment system-mysql -n threescale
   ```

   ```yaml
     volumeMounts:
       - name: example-claim
         mountPath: /var/lib/mysqldump/data
       - name: mysql-storage
         mountPath: /var/lib/mysql/data
       - name: mysql-extra-conf
         mountPath: /etc/my-extra.d
       - name: mysql-main-conf
         mountPath: /etc/my-extra
       ...
         serviceAccount: amp
     volumes:
           - name: example-claim
             persistentVolumeClaim:
               claimName: example-claim
       ...
   ```

   <dl>
   <dt><code>claimName</code></dt>
   <dd>Specifies the PVC that contains the dumped data.</dd>
   </dl>
4. Create a YAML file with following configuration to back up the MySQL database:
   ```yaml
   apiVersion: velero.io/v1
   kind: Backup
   metadata:
     name: mysql-backup
     namespace: openshift-adp
   spec:
     csiSnapshotTimeout: 10m0s
     defaultVolumesToFsBackup: true
     hooks:
       resources:
       - name: dumpdb
         pre:
         - exec:
             command:
             - /bin/sh
             - -c
             - mysqldump -u $MYSQL_USER --password=$MYSQL_PASSWORD system --no-tablespaces
               > /var/lib/mysqldump/data/dump.sql
             container: system-mysql
             onError: Fail
             timeout: 5m
     includedNamespaces:
     - threescale
     includedResources:
     - deployment
     - pods
     - replicationControllers
     - persistentvolumeclaims
     - persistentvolumes
     itemOperationTimeout: 1h0m0s
     labelSelector:
       matchLabels:
         app: 3scale-api-management
         threescale_component_element: mysql
     snapshotMoveData: false
     ttl: 720h0m0s
   ```

   where:

   <dl>
   <dt><code>mysql-backup</code></dt>
   <dd>Specifies the value of the <code>metadata.name</code> parameter in the backup. Use this value in the <code>metadata.backupName</code> parameter when restoring the MySQL database.</dd>
   <dt><code>/var/lib/mysqldump/data/dump.sql</code></dt>
   <dd>Specifies the directory where the data is backed up.</dd>
   <dt><code>includedResources</code></dt>
   <dd>Specifies the resources to back up.</dd>
   </dl>
5. Back up the MySQL database by running the following command:
   ```terminal
   $ oc create -f mysql.yaml
   ```

   ```terminal title="Example output"
   backup.velero.io/mysql-backup created
   ```

**Verification**

- Verify that the MySQL backup is completed by running the following command:
  ```terminal
  $ oc get backups.velero.io mysql-backup -o yaml
  ```

  ```terminal title="Example output"
  status:
  completionTimestamp: "2025-04-17T13:25:19Z"
  errors: 1
  expiration: "2025-05-17T13:25:16Z"
  formatVersion: 1.1.0
  hookStatus: {}
  phase: Completed
  progress: {}
  startTimestamp: "2025-04-17T13:25:16Z"
  version: 1
  ```

## Backing up the back-end Redis database {#backing-up-the-backend-redis-database_backing-up-3scale-api-management-by-using-oadp}

Back up the back-end Redis database by configuring Velero annotations and creating a backup CR with the required resources. This helps you preserve your 3scale back-end Redis data for recovery scenarios.

**Prerequisites**

- You backed up the Red Hat 3scale API Management operator.
- You backed up your MySQL database.
- The Redis queues have been drained before performing the backup.

**Procedure**

1. Edit the annotations on the `backend-redis` deployment by running the following command:
   ```terminal
   $ oc edit deployment backend-redis -n threescale
   ```

   ```yaml
   annotations:
   post.hook.backup.velero.io/command: >-
            ["/bin/bash", "-c", "redis-cli CONFIG SET auto-aof-rewrite-percentage
            100"]
          pre.hook.backup.velero.io/command: >-
            ["/bin/bash", "-c", "redis-cli CONFIG SET auto-aof-rewrite-percentage
            0"]
   ```
2. Create a YAML file with the following configuration to back up the Redis database:
   ```yaml
   apiVersion: velero.io/v1
   kind: Backup
   metadata:
     name: redis-backup
     namespace: openshift-adp
   spec:
     csiSnapshotTimeout: 10m0s
     defaultVolumesToFsBackup: true
     includedNamespaces:
     - threescale
     includedResources:
     - deployment
     - pods
     - replicationcontrollers
     - persistentvolumes
     - persistentvolumeclaims
     itemOperationTimeout: 1h0m0s
     labelSelector:
       matchLabels:
         app: 3scale-api-management
         threescale_component: backend
         threescale_component_element: redis
     snapshotMoveData: false
     snapshotVolumes: false
     ttl: 720h0m0s
   ```

   <dl>
   <dt><code>name</code></dt>
   <dd>Specifies the value of the <code>metadata.name</code> parameter in the backup. Use this value in the <code>metadata.backupName</code> parameter when restoring the Redis database.</dd>
   </dl>
3. Back up the Redis database by running the following command:
   ```terminal
   $ oc create -f redis-backup.yaml
   ```

   ```terminal title="Example output"
   backup.velero.io/redis-backup created
   ```

**Verification**

- Verify that the Redis backup is completed by running the following command:
  ```terminal
  $ oc get backups.velero.io redis-backup -o yaml
  ```

  ```terminal title="Example output"
  status:
  completionTimestamp: "2025-04-17T13:25:19Z"
  errors: 1
  expiration: "2025-05-17T13:25:16Z"
  formatVersion: 1.1.0
  hookStatus: {}
  phase: Completed
  progress: {}
  startTimestamp: "2025-04-17T13:25:16Z"
  version: 1
  ```

**Additional resources**

- [Installing 3scale API Management on OpenShift](https://docs.redhat.com/en/documentation/red_hat_3scale_api_management/2.15/html/installing_red_hat_3scale_api_management/install-threescale-on-openshift-guide)
- [Red Hat 3scale API Management](https://docs.redhat.com/en/documentation/red_hat_3scale_api_management)
- [Installing the Data Protection Application](/docs/backup_and_restore/application_backup_and_restore/installing/installing-oadp-aws#oadp-installing-dpa_installing-oadp-aws)
- [Creating a Backup CR](/docs/backup_and_restore/application_backup_and_restore/backing_up_and_restoring/oadp-creating-backup-cr#oadp-creating-backup-cr-doc)
