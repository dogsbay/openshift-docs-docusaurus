---
title: Configuring the OpenShift API for Data Protection with IBM Cloud
sidebar_position: 1
---

# Configuring the OpenShift API for Data Protection with IBM Cloud {#installing-oadp-ibm-cloud}

<a id="installing-oadp-ibm-cloud"></a>

You install the OpenShift API for Data Protection (OADP) Operator on an IBM Cloud cluster to back up and restore applications on the cluster. You configure IBM Cloud Object Storage (COS) to store the backups.

## Configuring the COS instance {#configuring-ibm-cos_installing-oadp-ibm-cloud}

You create an IBM Cloud Object Storage (COS) instance to store the OADP backup data. After you create the COS instance, configure the `HMAC` service credentials.

**Prerequisites**

- You have an IBM Cloud Platform account.
- You installed the [IBM Cloud CLI](https://cloud.ibm.com/docs/cli?topic=cli-getting-started).
- You are logged in to IBM Cloud.

**Procedure**

1. Install the IBM Cloud Object Storage (COS) plugin by running the following command:
   ```terminal
   $ ibmcloud plugin install cos -f
   ```
2. Set a bucket name by running the following command:
   ```terminal
   $ BUCKET=<bucket_name>
   ```
3. Set a bucket region by running the following command:
   ```terminal
   $ REGION=<bucket_region>
   ```

   where:

   <dl>
   <dt><code>&lt;bucket_region&gt;</code></dt>
   <dd>Specifies the bucket region. For example, <code>eu-gb</code>.</dd>
   </dl>
4. Create a resource group by running the following command:
   ```terminal
   $ ibmcloud resource group-create <resource_group_name>
   ```
5. Set the target resource group by running the following command:
   ```terminal
   $ ibmcloud target -g <resource_group_name>
   ```
6. Verify that the target resource group is correctly set by running the following command:
   ```terminal
   $ ibmcloud target
   ```

   ```yaml title="Example output"
   API endpoint:     https://cloud.ibm.com
   Region:
   User:             test-user
   Account:          Test Account (fb6......e95) <-> 2...122
   Resource group:   Default
   ```

   In the example output, the resource group is set to `Default`.
7. Set a resource group name by running the following command:
   ```terminal
   $ RESOURCE_GROUP=<resource_group>
   ```

   where:

   <dl>
   <dt><code>&lt;resource_group&gt;</code></dt>
   <dd>Specifies the resource group name. For example, <code>"default"</code>.</dd>
   </dl>
8. Create an IBM Cloud `service-instance` resource  by running the following command:
   ```terminal
   $ ibmcloud resource service-instance-create \
   <service_instance_name> \
   <service_name> \
   <service_plan> \
   <region_name>
   ```

   where:

   <dl>
   <dt><code>&lt;service_instance_name&gt;</code></dt>
   <dd>Specifies a name for the <code>service-instance</code> resource.</dd>
   <dt><code>&lt;service_name&gt;</code></dt>
   <dd>Specifies the service name. Alternatively, you can specify a service ID.</dd>
   <dt><code>&lt;service_plan&gt;</code></dt>
   <dd>Specifies the service plan for your IBM Cloud account.</dd>
   <dt><code>&lt;region_name&gt;</code></dt>
   <dd>Specifies the region name.</dd>
   </dl>

   Refer to the following example command:

   ```terminal
   $ ibmcloud resource service-instance-create test-service-instance cloud-object-storage \
   standard \
   global \
   -d premium-global-deployment
   ```

   where:

   <dl>
   <dt><code>cloud-object-storage</code></dt>
   <dd>Specifies the service name.</dd>
   <dt><code>-d premium-global-deployment</code></dt>
   <dd>Specifies the deployment name.</dd>
   </dl>
9. Extract the service instance ID by running the following command:
   ```terminal
   $ SERVICE_INSTANCE_ID=$(ibmcloud resource service-instance test-service-instance --output json | jq -r '.[0].id')
   ```
10. Create a COS bucket by running the following command:
    ```terminal
    $ ibmcloud cos bucket-create \
    --bucket $BUCKET \
    --ibm-service-instance-id $SERVICE_INSTANCE_ID \
    --region $REGION
    ```

    Variables such as `$BUCKET`, `$SERVICE_INSTANCE_ID`, and `$REGION` are replaced by the values you set previously.
11. Create `HMAC` credentials by running the following command.
    ```terminal
    $ ibmcloud resource service-key-create test-key Writer --instance-name test-service-instance --parameters {\"HMAC\":true}
    ```
12. Extract the access key ID and the secret access key from the `HMAC` credentials and save them in the `credentials-velero` file. You can use the `credentials-velero` file to create a `secret` for the backup storage location. Run the following command:
    ```terminal
    $ cat > credentials-velero << __EOF__
    [default]
    aws_access_key_id=$(ibmcloud resource service-key test-key -o json  | jq -r '.[0].credentials.cos_hmac_keys.access_key_id')
    aws_secret_access_key=$(ibmcloud resource service-key test-key -o json  | jq -r '.[0].credentials.cos_hmac_keys.secret_access_key')
    __EOF__
    ```

## Creating a default Secret {#oadp-creating-default-secret_installing-oadp-ibm-cloud}

You create a default `Secret` if your backup and snapshot locations use the same credentials or if you do not require a snapshot location.

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
   $ oc create secret generic cloud-credentials -n openshift-adp --from-file cloud=credentials-velero
   ```

   The `Secret` is referenced in the `spec.backupLocations.credential` block of the `DataProtectionApplication` CR when you install the Data Protection Application.

## Creating secrets for different credentials {#oadp-secrets-for-different-credentials_installing-oadp-ibm-cloud}

Create separate `Secret` objects when your backup and snapshot locations require different credentials. This allows you to configure distinct authentication for each storage location while maintaining secure credential management.

**Procedure**

1. Create a `credentials-velero` file for the snapshot location in the appropriate format for your cloud provider.
2. Create a `Secret` for the snapshot location with the default name:
   ```terminal
   $ oc create secret generic cloud-credentials -n openshift-adp --from-file cloud=credentials-velero
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
           provider: <provider>
           default: true
           credential:
             key: cloud
             name: <custom_secret>
           objectStorage:
             bucket: <bucket_name>
             prefix: <prefix>
   ```

   where:

   <dl>
   <dt><code>custom_secret</code></dt>
   <dd>Specifies the backup location <code>Secret</code> with custom name.</dd>
   </dl>

## Installing the Data Protection Application {#oadp-installing-dpa_installing-oadp-ibm-cloud}

You install the Data Protection Application (DPA) by creating an instance of the `DataProtectionApplication` API.

**Prerequisites**

- You must install the OADP Operator.
- You must configure object storage as a backup location.
- If you use snapshots to back up PVs, your cloud provider must support either a native snapshot API or Container Storage Interface (CSI) snapshots.
- If the backup and snapshot locations use the same credentials, you must create a `Secret` with the default name, `cloud-credentials`.
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
     namespace: openshift-adp
     name: <dpa_name>
   spec:
     configuration:
       velero:
         defaultPlugins:
         - openshift
         - aws
         - csi
     backupLocations:
       - velero:
           provider: aws
           default: true
           objectStorage:
             bucket: <bucket_name>
             prefix: velero
           config:
             insecureSkipTLSVerify: 'true'
             profile: default
             region: <region_name>
             s3ForcePathStyle: 'true'
             s3Url: <s3_url>
           credential:
             key: cloud
             name: cloud-credentials
   ```

   where:

   <dl>
   <dt><code>provider</code></dt>
   <dd>Specifies that the provider is <code>aws</code> when you use IBM Cloud as a backup storage location.</dd>
   <dt><code>bucket</code></dt>
   <dd>Specifies the IBM Cloud Object Storage (COS) bucket name.</dd>
   <dt><code>region</code></dt>
   <dd>Specifies the COS region name, for example, <code>eu-gb</code>.</dd>
   <dt><code>s3Url</code></dt>
   <dd>Specifies the S3 URL of the COS bucket. For example, <code>http://s3.eu-gb.cloud-object-storage.appdomain.cloud</code>. Here, <code>eu-gb</code> is the region name. Replace the region name according to your bucket region.</dd>
   <dt><code>name</code></dt>
   <dd>Specifies the name of the secret you created by using the access key and the secret access key from the <code>HMAC</code> credentials.</dd>
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

## Setting Velero CPU and memory resource allocations {#oadp-setting-resource-limits-and-requests_installing-oadp-ibm-cloud}

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

## Configuring node agents and node labels {#oadp-configuring-node-agents_installing-oadp-ibm-cloud}

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

## Configuring the DPA with client burst and QPS settings {#oadp-configuring-client-burst-qps_installing-oadp-ibm-cloud}

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

## Configuring node agent load affinity {#oadp-configuring-node-agent-load-affinity_installing-oadp-ibm-cloud}

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

## Node agent load affinity guidelines {#oadp-node-agent-load-affinity-guidelines_installing-oadp-ibm-cloud}

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

## Configuring node agent load concurrency {#oadp-configuring-node-agent-load-concurrency_installing-oadp-ibm-cloud}

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

## Configuring repository maintenance {#oadp-configuring-repository-maintenance_installing-oadp-ibm-cloud}

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

## Configuring Velero load affinity {#oadp-configuring-velero-load-affinity_installing-oadp-ibm-cloud}

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

## Overriding the imagePullPolicy setting in the DPA  {#oadp-configuring-imagepullpolicy_installing-oadp-ibm-cloud}

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

## Configuring the DPA with more than one BSL {#oadp-configuring-dpa-multiple-bsl_installing-oadp-ibm-cloud}

Configure the `DataProtectionApplication` (DPA) custom resource (CR) with multiple `BackupStorageLocation` (BSL) resources to store backups across different locations using provider-specific credentials. This provides backup distribution and location-specific restore capabilities.

For example, you have configured the following two BSLs:

- Configured one BSL in the DPA and set it as the default BSL.
- Created another BSL independently by using the `BackupStorageLocation` CR.

As you have already set the BSL created through the DPA as the default, you cannot set the independently created BSL again as the default. This means, at any given time, you can set only one BSL as the default BSL.

**Prerequisites**

- You must install the OADP Operator.
- You must create the secrets by using the credentials provided by the cloud provider.

**Procedure**

1. Configure the `DataProtectionApplication` CR with more than one `BackupStorageLocation` CR. See the following example:
   ```yaml title="Example DPA"
   apiVersion: oadp.openshift.io/v1alpha1
   kind: DataProtectionApplication
   #...
   backupLocations:
     - name: aws
       velero:
         provider: aws
         default: true
         objectStorage:
           bucket: <bucket_name>
           prefix: <prefix>
         config:
           region: <region_name>
           profile: "default"
         credential:
           key: cloud
           name: cloud-credentials
     - name: odf
       velero:
         provider: aws
         default: false
         objectStorage:
           bucket: <bucket_name>
           prefix: <prefix>
         config:
           profile: "default"
           region: <region_name>
           s3Url: <url>
           insecureSkipTLSVerify: "true"
           s3ForcePathStyle: "true"
         credential:
           key: cloud
           name: <custom_secret_name_odf>
   #...
   ```

   where:

   <dl>
   <dt><code>name: aws</code></dt>
   <dd>Specifies a name for the first BSL.</dd>
   <dt><code>default: true</code></dt>
   <dd>Indicates that this BSL is the default BSL. If a BSL is not set in the <code>Backup CR</code>, the default BSL is used. You can set only one BSL as the default.</dd>
   <dt><code>&lt;bucket_name&gt;</code></dt>
   <dd>Specifies the bucket name.</dd>
   <dt><code>&lt;prefix&gt;</code></dt>
   <dd>Specifies a prefix for Velero backups. For example, <code>velero</code>.</dd>
   <dt><code>&lt;region_name&gt;</code></dt>
   <dd>Specifies the AWS region for the bucket.</dd>
   <dt><code>cloud-credentials</code></dt>
   <dd>Specifies the name of the default <code>Secret</code> object that you created.</dd>
   <dt><code>name: odf</code></dt>
   <dd>Specifies a name for the second BSL.</dd>
   <dt><code>&lt;url&gt;</code></dt>
   <dd>Specifies the URL of the S3 endpoint.</dd>
   <dt><code>&lt;custom_secret_name_odf&gt;</code></dt>
   <dd>Specifies the correct name for the <code>Secret</code>. For example, <code>custom_secret_name_odf</code>. If you do not specify a <code>Secret</code> name, the default name is used.</dd>
   </dl>
2. Specify the BSL to be used in the backup CR. See the following example.
   ```yaml title="Example backup CR"
   apiVersion: velero.io/v1
   kind: Backup
   # ...
   spec:
     includedNamespaces:
     - <namespace>
     storageLocation: <backup_storage_location>
     defaultVolumesToFsBackup: true
   ```

   where:

   <dl>
   <dt><code>&lt;namespace&gt;</code></dt>
   <dd>Specifies the namespace to back up.</dd>
   <dt><code>&lt;backup_storage_location&gt;</code></dt>
   <dd>Specifies the storage location.</dd>
   </dl>

## Disabling the node agent in DataProtectionApplication {#oadp-about-disable-node-agent-dpa_installing-oadp-ibm-cloud}

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
