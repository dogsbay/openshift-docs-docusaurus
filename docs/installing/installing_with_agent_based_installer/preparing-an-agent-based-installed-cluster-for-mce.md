---
title: Preparing an Agent-based installed cluster for the multicluster engine for Kubernetes Operator
sidebar_position: 8
---

# Preparing an Agent-based installed cluster for the multicluster engine for Kubernetes Operator {#preparing-an-agent-based-installed-cluster-for-mce}

<a id="preparing-an-agent-based-installed-cluster-for-mce"></a>

You can install the multicluster engine Operator and deploy a hub cluster with the Agent-based Installer.

The following procedure is partially automated and requires manual steps after the initial cluster is deployed.

## Prerequisites {#prerequisites_preparing-an-agent-based-installed-cluster-for-mce}

Before installing the multicluster engine Operator and deploying a hub cluster with the Agent-based Installer, you must complete several prerequisites.

The following prerequisites must be completed:

- You have read the following documentation:
  - "Cluster lifecycle with multicluster engine operator overview"
  - "Persistent storage using local volumes"
  - "Using GitOps ZTP to provision clusters at the network far edge"
  - "Preparing to install with the Agent-based Installer"
  - "About disconnected installation mirroring"
- You have access to the internet to obtain the necessary container images.
- You have installed the OpenShift CLI (`oc`).
- If you are installing in a disconnected environment, you must have a configured local mirror registry for disconnected installation mirroring.

**Additional resources**

- [Cluster lifecycle with multicluster engine operator overview](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.9/html/clusters/cluster_mce_overview)
- [Persistent storage using local volumes](/docs/storage/persistent_storage_local/persistent-storage-local#persistent-storage-using-local-volume)
- [Using GitOps ZTP to provision clusters at the network far edge](/docs/edge_computing/ztp-deploying-far-edge-clusters-at-scale#about-ztp_ztp-deploying-far-edge-clusters-at-scale)
- [Preparing to install with the Agent-based Installer](/docs/installing/installing_with_agent_based_installer/preparing-to-install-with-agent-based-installer#preparing-to-install-with-agent-based-installer)
- [About disconnected installation mirroring](/docs/disconnected#installing-mirroring-disconnected-about)

## Prepare an Agent-based cluster deployment for the multicluster engine for Kubernetes Operator while disconnected {#preparing-an-initial-cluster-deployment-for-mce-disconnected_preparing-an-agent-based-installed-cluster-for-mce}

You can mirror the required OpenShift Container Platform container images, the multicluster engine Operator, and the Local Storage Operator (LSO) into your local mirror registry in a disconnected environment. Ensure that you note the local DNS hostname and port of your mirror registry.

:::note

To mirror your OpenShift Container Platform image repository to your mirror registry, you can use either the `oc adm release image` or `oc mirror` command. In this procedure, the `oc mirror` command is used as an example.

:::

**Procedure**

1. Create an `<assets_directory>` folder to contain valid `install-config.yaml` and `agent-config.yaml` files. This directory is used to store all the assets.
2. To mirror an OpenShift Container Platform image repository, the multicluster engine, and the LSO, create a `ImageSetConfiguration.yaml` file with the following settings:
   ```yaml title="Example ImageSetConfiguration.yaml"
     kind: ImageSetConfiguration
     apiVersion: mirror.openshift.io/v1alpha2
     archiveSize: 4
     storageConfig:
       imageURL: <your-local-registry-dns-name>:<your-local-registry-port>/mirror/oc-mirror-metadata
       skipTLS: true
     mirror:
       platform:
         architectures:
           - "amd64"
         channels:
           - name: stable-4.22
             type: ocp
       additionalImages:
         - name: registry.redhat.io/ubi9/ubi:latest
       operators:
         - catalog: registry.redhat.io/redhat/redhat-operator-index:v4.22
           packages:
             - name: multicluster-engine
             - name: local-storage-operator
   ```

   where:

   <dl>
   <dt><code>archiveSize</code></dt>
   <dd>Specifies the maximum size, in GiB, of each file within the image set.</dd>
   <dt><code>storageConfig</code></dt>
   <dd>Specifies the back-end location to receive the image set metadata. This location can be a registry or local directory. It is required to specify <code>storageConfig</code> values.</dd>
   <dt><code>storageConfig.imageURL</code></dt>
   <dd>Specifies the registry URL for the storage backend.</dd>
   <dt><code>channels.name</code></dt>
   <dd>Specifies the channel that contains the OpenShift Container Platform images for the version you are installing.</dd>
   <dt><code>operators.catalog</code></dt>
   <dd>Specifies the Operator catalog that contains the OpenShift Container Platform images that you are installing.</dd>
   <dt><code>packages</code></dt>
   <dd>Specifies only certain Operator packages and channels to include in the image set. Remove this field to retrieve all packages in the catalog. In this example, a <code>package.name</code> value of <code>multicluster-engine</code> includes the multicluster engine packages and channels, and <code>local-storage-operator</code> includes the LSO packages and channels.</dd>
   </dl>

   :::note

   This file is required by the `oc mirror` command when mirroring content.

   :::
3. To mirror a specific OpenShift Container Platform image repository, the multicluster engine, and the LSO, run the following command:
   ```terminal
   $ oc mirror --dest-skip-tls --config ocp-mce-imageset.yaml docker://<your-local-registry-dns-name>:<your-local-registry-port>
   ```
4. Update the registry and certificate in the `install-config.yaml` file:
   ```yaml title="Example imageContentSources.yaml"
     imageContentSources:
       - source: "quay.io/openshift-release-dev/ocp-release"
         mirrors:
           - "<your-local-registry-dns-name>:<your-local-registry-port>/openshift/release-images"
       - source: "quay.io/openshift-release-dev/ocp-v4.0-art-dev"
         mirrors:
           - "<your-local-registry-dns-name>:<your-local-registry-port>/openshift/release"
       - source: "registry.redhat.io/ubi9"
         mirrors:
           - "<your-local-registry-dns-name>:<your-local-registry-port>/ubi9"
       - source: "registry.redhat.io/multicluster-engine"
         mirrors:
           - "<your-local-registry-dns-name>:<your-local-registry-port>/multicluster-engine"
       - source: "registry.redhat.io/rhel8"
         mirrors:
           - "<your-local-registry-dns-name>:<your-local-registry-port>/rhel8"
       - source: "registry.redhat.io/redhat"
         mirrors:
           - "<your-local-registry-dns-name>:<your-local-registry-port>/redhat"
   ```

   Additionally, ensure your certificate is present in the `additionalTrustBundle` field of the `install-config.yaml`.

   ```yaml title="Example install-config.yaml"
   additionalTrustBundle: |
     -----BEGIN CERTIFICATE-----
     zzzzzzzzzzz
     -----END CERTIFICATE-------
   ```

   :::warning

   The `oc mirror` command  creates a folder called `oc-mirror-workspace` with several outputs. This includes the `imageContentSourcePolicy.yaml` file that identifies all the mirrors you need for OpenShift Container Platform and your selected Operators.

   :::
5. Generate the cluster manifests by running the following command:
   ```terminal
   $ openshift-install agent create cluster-manifests
   ```

   This command updates the cluster manifests folder to include a `mirror` folder that contains your mirror configuration.

## Preparing an Agent-based cluster deployment for the multicluster engine for Kubernetes Operator while connected {#preparing-an-initial-cluster-deployment-for-mce-connected_preparing-an-agent-based-installed-cluster-for-mce}

Create the required manifests for the multicluster engine Operator, the Local Storage Operator (LSO), and to deploy an agent-based OpenShift Container Platform cluster as a hub cluster.

**Procedure**

1. Create a sub-folder named  `openshift` in the `<assets_directory>` folder. This sub-folder is used to store the extra manifests that will be applied during the installation to further customize the deployed cluster. The `<assets_directory>` folder contains all the assets including the `install-config.yaml` and `agent-config.yaml` files.
   :::note

   The installer does not validate extra manifests.

   :::
2. For the multicluster engine, create the following manifests and save them in the `<assets_directory>/openshift` folder:
   ```yaml title="Example mce_namespace.yaml"
     apiVersion: v1
     kind: Namespace
     metadata:
       labels:
         openshift.io/cluster-monitoring: "true"
       name: multicluster-engine
   ```

   ```yaml title="Example mce_operatorgroup.yaml"
     apiVersion: operators.coreos.com/v1
     kind: OperatorGroup
     metadata:
       name: multicluster-engine-operatorgroup
       namespace: multicluster-engine
     spec:
       targetNamespaces:
       - multicluster-engine
   ```

   ```yaml title="Example mce_subscription.yaml"
     apiVersion: operators.coreos.com/v1alpha1
     kind: Subscription
     metadata:
       name: multicluster-engine
       namespace: multicluster-engine
     spec:
       channel: "stable-2.3"
       name: multicluster-engine
       source: redhat-operators
       sourceNamespace: openshift-marketplace
   ```

   :::note

   You can install a distributed unit (DU) at scale with the Red Hat Advanced Cluster Management (RHACM) using the assisted installer (AI). These distributed units must be enabled in the hub cluster. The AI service requires persistent volumes (PVs), which are manually created.

   :::
3. For the AI service, create the following manifests and save them in the `<assets_directory>/openshift` folder:
   ```yaml title="Example lso_namespace.yaml"
     apiVersion: v1
     kind: Namespace
     metadata:
       annotations:
         openshift.io/cluster-monitoring: "true"
       name: openshift-local-storage
   ```

   ```yaml title="Example lso_operatorgroup.yaml"
     apiVersion: operators.coreos.com/v1
     kind: OperatorGroup
     metadata:
       name: local-operator-group
       namespace: openshift-local-storage
     spec:
       targetNamespaces:
         - openshift-local-storage
   ```

   ```yaml title="Example lso_subscription.yaml"
     apiVersion: operators.coreos.com/v1alpha1
     kind: Subscription
     metadata:
       name: local-storage-operator
       namespace: openshift-local-storage
     spec:
       installPlanApproval: Automatic
       name: local-storage-operator
       source: redhat-operators
       sourceNamespace: openshift-marketplace
   ```

   :::note

   After creating all the manifests, your filesystem must display as follows:

   ```terminal title="Example Filesystem"
   <assets_directory>
       ├─ install-config.yaml
       ├─ agent-config.yaml
       └─ /openshift
           ├─ mce_namespace.yaml
           ├─ mce_operatorgroup.yaml
           ├─ mce_subscription.yaml
           ├─ lso_namespace.yaml
           ├─ lso_operatorgroup.yaml
           └─ lso_subscription.yaml
   ```

   :::
4. Create the agent ISO image by running the following command:
   ```terminal
   $ openshift-install agent create image --dir <assets_directory>
   ```
5. When the image is ready, boot the target machine and wait for the installation to complete.
6. To monitor the installation, run the following command:
   ```terminal
   $ openshift-install agent wait-for install-complete --dir <assets_directory>
   ```

   :::note

   To configure a fully functional hub cluster, you must create the following manifests and manually apply them by running the command `$ oc apply -f <manifest-name>`. The order of the manifest creation is important and where required, the waiting condition is displayed.

   :::
7. For the PVs that are required by the AI service, create the following manifests:
   ```yaml
     apiVersion: local.storage.openshift.io/v1
     kind: LocalVolume
     metadata:
      name: assisted-service
      namespace: openshift-local-storage
     spec:
      logLevel: Normal
      managementState: Managed
      storageClassDevices:
        - devicePaths:
            - /dev/vda
            - /dev/vdb
          storageClassName: assisted-service
          volumeMode: Filesystem
   ```
8. Use the following command to wait for the availability of the PVs, before applying the subsequent manifests:
   ```terminal
   $ oc wait localvolume -n openshift-local-storage assisted-service --for condition=Available --timeout 10m
   ```

   :::note

   ```
   The `devicePath` is an example and may vary depending on the actual hardware configuration used.
   ```

   :::
9. Create a manifest for a multicluster engine instance.
   ```yaml title="Example MultiClusterEngine.yaml"
     apiVersion: multicluster.openshift.io/v1
     kind: MultiClusterEngine
     metadata:
       name: multiclusterengine
     spec: {}
   ```
10. Create a manifest to enable the AI service.
    ```yaml title="Example agentserviceconfig.yaml"
      apiVersion: agent-install.openshift.io/v1beta1
      kind: AgentServiceConfig
      metadata:
        name: agent
        namespace: assisted-installer
      spec:
       databaseStorage:
        storageClassName: assisted-service
        accessModes:
        - ReadWriteOnce
        resources:
         requests:
          storage: 10Gi
       filesystemStorage:
        storageClassName: assisted-service
        accessModes:
        - ReadWriteOnce
        resources:
         requests:
          storage: 10Gi
    ```
11. Create a manifest to deploy subsequently spoke clusters.
    ```yaml title="Example clusterimageset.yaml"
      apiVersion: hive.openshift.io/v1
      kind: ClusterImageSet
      metadata:
        name: "4.22"
      spec:
        releaseImage: quay.io/openshift-release-dev/ocp-release:4.22.0-x86_64
    ```
12. Create a manifest to import the agent installed cluster (that hosts the multicluster engine and the Assisted Service) as the hub cluster.
    ```yaml title="Example autoimport.yaml"
      apiVersion: cluster.open-cluster-management.io/v1
      kind: ManagedCluster
      metadata:
       labels:
         local-cluster: "true"
         cloud: auto-detect
         vendor: auto-detect
       name: local-cluster
      spec:
       hubAcceptsClient: true
    ```
13. Wait for the managed cluster to be created.
    ```terminal
    $ oc wait -n multicluster-engine managedclusters local-cluster --for condition=ManagedClusterJoined=True --timeout 10m
    ```

**Verification**

- To confirm that the managed cluster installation is successful, run the following command:
  ```terminal
  $ oc get managedcluster
  NAME            HUB ACCEPTED   MANAGED CLUSTER URLS             JOINED   AVAILABLE  AGE
  local-cluster   true           https://<your cluster url>:6443   True     True       77m
  ```

**Additional resources**

- [The Local Storage Operator](/docs/storage/persistent_storage_local/persistent-storage-local#persistent-storage-using-local-volume)
