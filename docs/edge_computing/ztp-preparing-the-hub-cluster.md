---
title: Preparing the hub cluster for GitOps ZTP
sidebar_position: 2
---

# Preparing the hub cluster for GitOps ZTP {#ztp-preparing-the-hub-cluster}

<a id="ztp-preparing-the-hub-cluster"></a>

To use RHACM in a disconnected environment, create a mirror registry that mirrors the OpenShift Container Platform release images and Operator Lifecycle Manager (OLM) catalog that contains the required Operator images. OLM manages, installs, and upgrades Operators and their dependencies in the cluster. You can also use a disconnected mirror host to serve the RHCOS ISO and RootFS disk images that are used to provision the bare-metal hosts.

## Telco RAN DU 4.22 validated software components {#ztp-telco-ran-software-versions_ztp-preparing-the-hub-cluster}

The Red Hat telco RAN DU 4.22 solution has been validated using the following Red Hat software products for OpenShift Container Platform managed clusters.

**Telco RAN DU managed cluster validated software components**

| Component | Software version |
| --- | --- |
| OpenShift Container Platform | 4.22 |
| Cluster Logging Operator | 6.5 |
| Local Storage Operator | 4.22 |
| OpenShift API for Data Protection (OADP) | 1.6 |
| PTP Operator | 4.22 |
| SR-IOV Operator | 4.22 |
| SRIOV-FEC Operator | 2.12 |
| Lifecycle Agent Operator | 4.22 |

- Cluster Logging Operator will be updated to 6.6 when the aligned Cluster Logging Operator version is released.

## Recommended hub cluster specifications and managed cluster limits for GitOps ZTP {#ztp-gitops-ztp-max-spoke-clusters_ztp-preparing-the-hub-cluster}

With GitOps Zero Touch Provisioning (ZTP), you can manage thousands of clusters in geographically dispersed regions and networks. The Red Hat Performance and Scale lab successfully created and managed 3500 virtual single-node OpenShift clusters with a reduced DU profile from a single Red Hat Advanced Cluster Management (RHACM) hub cluster in a lab environment.

In real-world situations, the scaling limits for the number of clusters that you can manage will vary depending on various factors affecting the hub cluster. For example:

Hub cluster resources
: Available hub cluster host resources (CPU, memory, storage) are an important factor in determining how many clusters the hub cluster can manage. The more resources allocated to the hub cluster, the more managed clusters it can accommodate.

Hub cluster storage
: The hub cluster host storage IOPS rating and whether the hub cluster hosts use NVMe storage can affect hub cluster performance and the number of clusters it can manage.

Network bandwidth and latency
: Slow or high-latency network connections between the hub cluster and managed clusters can impact how the hub cluster manages multiple clusters.

Managed cluster size and complexity
: The size and complexity of the managed clusters also affects the capacity of the hub cluster. Larger managed clusters with more nodes, namespaces, and resources require additional processing and management resources. Similarly, clusters with complex configurations such as the RAN DU profile or diverse workloads can require more resources from the hub cluster.

Number of managed policies
: The number of policies managed by the hub cluster scaled over the number of managed clusters bound to those policies is an important factor that determines how many clusters can be managed.

Monitoring and management workloads
: RHACM continuously monitors and manages the managed clusters. The number and complexity of monitoring and management workloads running on the hub cluster can affect its capacity. Intensive monitoring or frequent reconciliation operations can require additional resources, potentially limiting the number of manageable clusters.

RHACM version and configuration
: Different versions of RHACM can have varying performance characteristics and resource requirements. Additionally, the configuration settings of RHACM, such as the number of concurrent reconciliations or the frequency of health checks, can affect the managed cluster capacity of the hub cluster.

Use the following representative configuration and network specifications to develop your own Hub cluster and network specifications.

:::warning

The following guidelines are based on internal lab benchmark testing only and do not represent complete bare-metal host specifications.

:::

**Representative three-node hub cluster machine specifications**

<table>
<thead>
<tr>
  <th>Requirement</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Server hardware</td>
  <td>3 x Dell PowerEdge R650 rack servers</td>
</tr>
<tr>
  <td>NVMe hard disks</td>
  <td><ul><li>50 GB disk for <code>/var/lib/etcd</code></li><li>2.9 TB disk for <code>/var/lib/containers</code></li></ul></td>
</tr>
<tr>
  <td>SSD hard disks</td>
  <td><ul><li>1 SSD split into 15 200GB thin-provisioned logical volumes provisioned as <code>PV</code> CRs</li><li>1 SSD serving as an extra large <code>PV</code> resource</li></ul></td>
</tr>
<tr>
  <td>Number of applied DU profile policies</td>
  <td>5</td>
</tr>
</tbody>
</table>

:::warning

The following network specifications are representative of a typical real-world RAN network and were applied to the scale lab environment during testing.

:::

**Simulated lab environment network specifications**

| Specification | Description |
| --- | --- |
| Round-trip time (RTT) latency | 50 ms |
| Packet loss | 0.02% packet loss |
| Network bandwidth limit | 20 Mbps |

**Additional resources**

- [Creating and managing single-node OpenShift clusters with RHACM](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.7/html/install/installing#single-node)

## Installing GitOps ZTP in a disconnected environment {#installing-disconnected-rhacm_ztp-preparing-the-hub-cluster}

Use Red Hat Advanced Cluster Management (RHACM), Red Hat OpenShift GitOps, and Topology Aware Lifecycle Manager (TALM) on the hub cluster in the disconnected environment to manage the deployment of multiple managed clusters.

**Prerequisites**

- You have installed the OpenShift Container Platform CLI (`oc`).
- You have logged in as a user with `cluster-admin` privileges.
- You have configured a disconnected mirror registry for use in the cluster.
  :::note

  The disconnected mirror registry that you create must contain a version of TALM backup and pre-cache images that matches the version of TALM running in the hub cluster. The spoke cluster must be able to resolve these images in the disconnected mirror registry.

  :::

**Procedure**

- Install RHACM in the hub cluster. See [Installing RHACM in a disconnected environment](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.17/html/install/installing#install-on-disconnected-networks).
- Install GitOps and TALM in the hub cluster.

**Additional resources**

- [Installing OpenShift GitOps](https://docs.openshift.com/gitops/latest/installing_gitops/installing-openshift-gitops.html#installing-openshift-gitops)
- [Installing TALM](/docs/edge_computing/cnf-talm-for-cluster-upgrades#installing-topology-aware-lifecycle-manager-using-cli_cnf-topology-aware-lifecycle-manager)
- [Mirroring an Operator catalog](/docs/disconnected/using-olm#olm-mirror-catalog_olm-restricted-networks)

## Adding RHCOS ISO and RootFS images to the disconnected mirror host {#ztp-acm-adding-images-to-mirror-registry_ztp-preparing-the-hub-cluster}

Before you begin installing clusters in the disconnected environment with Red Hat Advanced Cluster Management (RHACM), you must first host Red Hat Enterprise Linux CoreOS (RHCOS) images for it to use. Use a disconnected mirror to host the RHCOS images.

**Prerequisites**

- Deploy and configure an HTTP server to host the RHCOS image resources on the network. You must be able to access the HTTP server from your computer, and from the machines that you create.

:::warning

The RHCOS images might not change with every release of OpenShift Container Platform. You must download images with the highest version that is less than or equal to the version that you install. Use the image versions that match your OpenShift Container Platform version if they are available. You require ISO and RootFS images to install RHCOS on the hosts. RHCOS QCOW2 images are not supported for this installation type.

:::

**Procedure**

1. Log in to the mirror host.
2. Obtain the RHCOS ISO and RootFS images from [mirror.openshift.com](https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/), for example:
   1. Export the required image names and OpenShift Container Platform version as environment variables:
      ```terminal
      $ export ISO_IMAGE_NAME=<iso_image_name>
      ```

      ```terminal
      $ export ROOTFS_IMAGE_NAME=<rootfs_image_name>
      ```

      ```terminal
      $ export OCP_VERSION=<ocp_version>
      ```

      where:

      <dl>
      <dt><code>&lt;iso_image_name&gt;</code></dt>
      <dd>ISO image name, for example, <code>rhcos-4.22.1-x86_64-live.x86_64.iso</code></dd>
      <dt><code>&lt;rootfs_image_name&gt;</code></dt>
      <dd>RootFS image name, for example, <code>rhcos-4.22.1-x86_64-live-rootfs.x86_64.img</code></dd>
      <dt><code>&lt;ocp_version&gt;</code></dt>
      <dd>OpenShift Container Platform version, for example, <code>4.22.1</code></dd>
      </dl>
   2. Download the required images:
      ```terminal
      $ sudo wget https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.22/${OCP_VERSION}/${ISO_IMAGE_NAME} -O /var/www/html/${ISO_IMAGE_NAME}
      ```

      ```terminal
      $ sudo wget https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.22/${OCP_VERSION}/${ROOTFS_IMAGE_NAME} -O /var/www/html/${ROOTFS_IMAGE_NAME}
      ```

**Verification**

- Verify that the images downloaded successfully and are being served on the disconnected mirror host, for example:
  ```terminal
  $ wget http://$(hostname)/${ISO_IMAGE_NAME}
  ```

  Example output:

  ```terminal
  Saving to: rhcos-4.22.1-x86_64-live.x86_64.iso
  rhcos-4.22.1-x86_64-live.x86_64.iso-  11%[====>    ]  10.01M  4.71MB/s
  ```

**Additional resources**

- [Creating a mirror registry](/docs/disconnected/installing-mirroring-creating-registry#installing-mirroring-creating-registry)
- [Mirroring images for a disconnected installation](/docs/disconnected/installing-mirroring-installation-images#installing-mirroring-installation-images)

## Enabling the assisted service {#enabling-assisted-installer-service-on-bare-metal_ztp-preparing-the-hub-cluster}

Red Hat Advanced Cluster Management (RHACM) uses the assisted service to deploy OpenShift Container Platform clusters. The assisted service is deployed automatically when you enable the MultiClusterHub Operator on Red Hat Advanced Cluster Management (RHACM). After that, you need to configure the `Provisioning` resource to watch all namespaces and to update the `AgentServiceConfig` custom resource (CR) with references to the ISO and RootFS images that are hosted on the mirror registry HTTP server.

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).
- You have logged in to the hub cluster as a user with `cluster-admin` privileges.
- You have RHACM with `MultiClusterHub` enabled.

**Procedure**

1. Enable the `Provisioning` resource to watch all namespaces and configure mirrors for disconnected environments. For more information, see [Enabling the central infrastructure management service](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.9/html/clusters/cluster_mce_overview#enable-cim).
2. Open the `AgentServiceConfig` CR to update the `spec.osImages` field by running the following command:
   ```terminal
   $ oc edit AgentServiceConfig
   ```
3. Update the `spec.osImages` field in the `AgentServiceConfig` CR:
   ```yaml
   apiVersion: agent-install.openshift.io/v1beta1
   kind: AgentServiceConfig
   metadata:
    name: agent
   spec:
   # ...
     osImages:
       - cpuArchitecture: x86_64
         openshiftVersion: "4.22"
         rootFSUrl: https://<host>/<path>/rhcos-live-rootfs.x86_64.img
         url: https://<host>/<path>/rhcos-live.x86_64.iso
   ```

   where:

   <dl>
   <dt><code>&lt;host&gt;</code></dt>
   <dd>Specifies the fully qualified domain name (FQDN) for the target mirror registry HTTP server.</dd>
   <dt><code>&lt;path&gt;</code></dt>
   <dd>Specifies the path to the image on the target mirror registry.</dd>
   </dl>
4. Save and quit the editor to apply the changes.

## Configuring the hub cluster to use a disconnected mirror registry {#ztp-configuring-the-cluster-for-a-disconnected-environment_ztp-preparing-the-hub-cluster}

You can configure the hub cluster to use a disconnected mirror registry for a disconnected environment.

**Prerequisites**

- You have a disconnected hub cluster installation with Red Hat Advanced Cluster Management (RHACM) 2.17 installed.
- You have hosted the `rootfs` and `iso` images on an HTTP server. See the *Additional resources* section for guidance about *Mirroring the OpenShift Container Platform image repository*.

:::warning

If you enable TLS for the HTTP server, you must confirm the root certificate is signed by an authority trusted by the client and verify the trusted certificate chain between your OpenShift Container Platform hub and managed clusters and the HTTP server. Using a server configured with an untrusted certificate prevents the images from being downloaded to the image creation service. Using untrusted HTTPS servers is not supported.

:::

**Procedure**

1. Create a `ConfigMap` containing the mirror registry config:
   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: assisted-installer-mirror-config
     namespace: multicluster-engine
     labels:
       app: assisted-service
   data:
     ca-bundle.crt: |
       -----BEGIN CERTIFICATE-----
       <certificate_contents>
       -----END CERTIFICATE-----

     registries.conf: |
       unqualified-search-registries = ["registry.access.redhat.com", "docker.io"]

       [[registry]]
          prefix = ""
          location = "quay.io/example-repository"
          mirror-by-digest-only = true

          [[registry.mirror]]
          location = "mirror1.registry.corp.com:5000/example-repository"
   ```

   where:

   <dl>
   <dt><code>namespace: multicluster-engine</code></dt>
   <dd>The <code>ConfigMap</code> namespace must be set to <code>multicluster-engine</code>.</dd>
   <dt><code>ca-bundle.crt</code></dt>
   <dd>The mirror registry’s certificate that is used when creating the mirror registry.</dd>
   <dt><code>registries.conf</code></dt>
   <dd>The configuration file for the mirror registry. The mirror registry configuration adds mirror information to the <code>/etc/containers/registries.conf</code> file in the discovery image. The mirror information is stored in the <code>imageContentSources</code> section of the <code>install-config.yaml</code> file when the information is passed to the installation program. The Assisted Service pod that runs on the hub cluster fetches the container images from the configured mirror registry.</dd>
   <dt><code>location = "quay.io/example-repository"</code></dt>
   <dd>The URL of the mirror registry. You must use the URL from the <code>imageContentSources</code> section by running the <code>oc adm release mirror</code> command when you configure the mirror registry. For more information, see the <em>Mirroring the OpenShift Container Platform image repository</em> section.</dd>
   <dt><code>location = "mirror1.registry.corp.com:5000/example-repository"</code></dt>
   <dd>The registries defined in the <code>registries.conf</code> file must be scoped by repository, not by registry. In this example, both the <code>quay.io/example-repository</code> and the <code>mirror1.registry.corp.com:5000/example-repository</code> repositories are scoped by the <code>example-repository</code> repository.</dd>
   </dl>

   This updates `mirrorRegistryRef` in the `AgentServiceConfig` custom resource, as shown in this example output:

   ```yaml
   apiVersion: agent-install.openshift.io/v1beta1
   kind: AgentServiceConfig
   metadata:
     name: agent
     namespace: multicluster-engine
   spec:
     databaseStorage:
       volumeName: <db_pv_name>
       accessModes:
       - ReadWriteOnce
       resources:
         requests:
           storage: <db_storage_size>
     filesystemStorage:
       volumeName: <fs_pv_name>
       accessModes:
       - ReadWriteOnce
       resources:
         requests:
           storage: <fs_storage_size>
     mirrorRegistryRef:
       name: assisted-installer-mirror-config
     osImages:
       - openshiftVersion: <ocp_version>
         url: <iso_url>
   ```

   where:

   <dl>
   <dt><code>namespace: multicluster-engine</code></dt>
   <dd>Set the <code>AgentServiceConfig</code> namespace to <code>multicluster-engine</code> to match the <code>ConfigMap</code> namespace.</dd>
   <dt><code>assisted-installer-mirror-config</code></dt>
   <dd>Set <code>mirrorRegistryRef.name</code> to match the definition specified in the related <code>ConfigMap</code> CR.</dd>
   <dt><code>&lt;ocp_version&gt;</code></dt>
   <dd>Set the OpenShift Container Platform version to either the x.y or x.y.z format.</dd>
   <dt><code>&lt;iso_url&gt;</code></dt>
   <dd>Set the URL for the ISO hosted on the <code>httpd</code> server.</dd>
   </dl>

   :::warning

   A valid NTP server is required during cluster installation. Ensure that a suitable NTP server is available and can be reached from the installed clusters through the disconnected network.

   :::

**Additional resources**

- [Mirroring the OpenShift Container Platform repository](/docs/disconnected/installing-mirroring-installation-images#installation-mirror-repository_installing-mirroring-installation-images)

## Configuring the hub cluster to use unauthenticated registries {#ztp-configuring-the-hub-cluster-to-use-unauthenticated-registries_ztp-preparing-the-hub-cluster}

You can configure the hub cluster to use unauthenticated registries. Unauthenticated registries does not require authentication to access and download images.

**Prerequisites**

- You have installed and configured a hub cluster and installed Red Hat Advanced Cluster Management (RHACM) on the hub cluster.
- You have installed the OpenShift Container Platform CLI (oc).
- You have logged in as a user with `cluster-admin` privileges.
- You have configured an unauthenticated registry for use with the hub cluster.

**Procedure**

1. Update the `AgentServiceConfig` custom resource (CR) by running the following command:
   ```terminal
   $ oc edit AgentServiceConfig agent
   ```
2. Add the `unauthenticatedRegistries` field in the CR:
   ```yaml
   apiVersion: agent-install.openshift.io/v1beta1
   kind: AgentServiceConfig
   metadata:
     name: agent
   spec:
     unauthenticatedRegistries:
     - example.registry.com
     - example.registry2.com
     ...
   ```

   Unauthenticated registries are listed under `spec.unauthenticatedRegistries` in the `AgentServiceConfig` resource. Any registry on this list is not required to have an entry in the pull secret used for the spoke cluster installation. `assisted-service` validates the pull secret by making sure it contains the authentication information for every image registry used for installation.

   :::note

   Mirror registries are automatically added to the ignore list and do not need to be added under `spec.unauthenticatedRegistries`. Specifying the `PUBLIC_CONTAINER_REGISTRIES` environment variable in the `ConfigMap` overrides the default values with the specified value. The `PUBLIC_CONTAINER_REGISTRIES` defaults are [quay.io](https://quay.io) and [registry.svc.ci.openshift.org](https://registry.svc.ci.openshift.org).

   :::

**Verification**

Verify that you can access the newly added registry from the hub cluster by running the following commands:

1. Open a debug shell prompt to the hub cluster:
   ```terminal
   $ oc debug node/<node_name>
   ```
2. Test access to the unauthenticated registry by running the following command:
   ```terminal
   sh-4.4# podman login -u kubeadmin -p $(oc whoami -t) <unauthenticated_registry>
   ```

   where:

   <dl>
   <dt>&lt;unauthenticated_registry&gt;</dt>
   <dd>Is the new registry, for example, <code>unauthenticated-image-registry.openshift-image-registry.svc:5000</code>.</dd>
   </dl>

   ```terminal title="Example output"
   Login Succeeded!
   ```

## Configuring the hub cluster with ArgoCD {#ztp-configuring-hub-cluster-with-argocd_ztp-preparing-the-hub-cluster}

You can configure the hub cluster with a set of ArgoCD applications that generate the required installation and policy custom resources (CRs) for each site with GitOps Zero Touch Provisioning (ZTP).

:::note

Red Hat Advanced Cluster Management (RHACM) uses `ClusterInstance` CRs to generate the Day 1 managed cluster installation CRs for ArgoCD. Each ArgoCD application can manage a maximum of 1000 `ClusterInstance` CRs.

:::

**Prerequisites**

- You have a OpenShift Container Platform hub cluster with Red Hat Advanced Cluster Management (RHACM) and Red Hat OpenShift GitOps installed.
- You have extracted the reference deployment from the GitOps ZTP plugin container as described in the "Preparing the GitOps ZTP site configuration repository" section. Extracting the reference deployment creates the `out/argocd/deployment` directory referenced in the following procedure.

**Procedure**

1. Prepare the ArgoCD pipeline configuration:
   1. Create a Git repository with the directory structure similar to the example directory. For more information, see "Preparing the GitOps ZTP site configuration repository".
   2. Configure access to the repository using the ArgoCD UI. Under **Settings** configure the following:
      - **Repositories** - Add the connection information. The URL must end in `.git`, for example, `https://repo.example.com/repo.git` and credentials.
      - **Certificates** - Add the public certificate for the repository, if needed.
   3. Modify the two ArgoCD applications, `out/argocd/deployment/clusters-app.yaml` and `out/argocd/deployment/policies-app.yaml`, based on your Git repository:
      - Update the URL to point to the Git repository. The URL ends with `.git`, for example, `https://repo.example.com/repo.git`.
      - The `targetRevision` indicates which Git repository branch to monitor.
      - `path` specifies the path to the `ClusterInstance` and `PolicyGenerator` or `PolicyGentemplate` CRs, respectively.
2. To install the GitOps ZTP plugin, patch the ArgoCD instance in the hub cluster with the relevant multicluster engine (MCE) subscription image. Customize the patch file that you previously extracted into the `out/argocd/deployment/` directory for your environment.
   1. Select the `multicluster-operators-subscription` image that matches your RHACM version.
      - For RHACM 2.8 and 2.9, use the `registry.redhat.io/rhacm2/multicluster-operators-subscription-rhel8:v<rhacm_version>` image.
      - For RHACM 2.10 and later, use the `registry.redhat.io/rhacm2/multicluster-operators-subscription-rhel9:v<rhacm_version>` image.

      :::warning

      The version of the `multicluster-operators-subscription` image must match the RHACM version. Beginning with the MCE 2.10 release, RHEL 9 is the base image for `multicluster-operators-subscription` images.

      Click `[Expand for Operator list]` in the "Platform Aligned Operators" table in [OpenShift Operator Life Cycles](https://access.redhat.com/support/policy/updates/openshift_operators) to view the complete supported Operators matrix for OpenShift Container Platform.

      :::
   2. Modify the `out/argocd/deployment/argocd-openshift-gitops-patch.json` file with the `multicluster-operators-subscription` image that matches your RHACM version:
      ```json
      {
        "args": [
          "-c",
          "mkdir -p /.config/kustomize/plugin/policy.open-cluster-management.io/v1/policygenerator && cp /policy-generator/PolicyGenerator-not-fips-compliant /.config/kustomize/plugin/policy.open-cluster-management.io/v1/policygenerator/PolicyGenerator"
        ],
        "command": [
          "/bin/bash"
        ],
        "image": "registry.redhat.io/rhacm2/multicluster-operators-subscription-rhel9:v2.10",
        "name": "policy-generator-install",
        "imagePullPolicy": "Always",
        "volumeMounts": [
          {
            "mountPath": "/.config",
            "name": "kustomize"
          }
        ]
      }
      ```

      - Optional: For RHEL 9 images, in the `args` field, change the executable path from `/policy-generator/PolicyGenerator-not-fips-compliant` to match the required universal executable for your ArgoCD version.
      - Match the `multicluster-operators-subscription` image to your RHACM version. In disconnected environments, replace the URL with the disconnected registry equivalent for your environment.
   3. Patch the ArgoCD instance. Run the following command:
      ```terminal
      $ oc patch argocd openshift-gitops \
      -n openshift-gitops --type=merge \
      --patch-file out/argocd/deployment/argocd-openshift-gitops-patch.json
      ```
3. In RHACM 2.7 and later, the multicluster engine enables the `cluster-proxy-addon` feature by default. Apply the following patch to disable the `cluster-proxy-addon` feature and remove the relevant hub cluster and managed pods that are responsible for this add-on. Run the following command:
   ```terminal
   $ oc patch multiclusterengines.multicluster.openshift.io multiclusterengine --type=merge --patch-file out/argocd/deployment/disable-cluster-proxy-addon.json
   ```
4. Apply the pipeline configuration to your hub cluster by running the following command:
   ```terminal
   $ oc apply -k out/argocd/deployment
   ```
5. Optional: If you have existing ArgoCD applications, verify that the `PrunePropagationPolicy=background` policy is set in the `Application` resource by running the following command:
   ```terminal
   $ oc -n openshift-gitops get applications.argoproj.io  \
   clusters -o jsonpath='{.spec.syncPolicy.syncOptions}' |jq
   ```

   Example output for an existing policy:

   ```terminal
   [
     "CreateNamespace=true",
     "PrunePropagationPolicy=background",
     "RespectIgnoreDifferences=true"
   ]
   ```

   1. If the `spec.syncPolicy.syncOption` field does not contain a `PrunePropagationPolicy` parameter or `PrunePropagationPolicy` is set to the `foreground` value, set the policy to `background` in the `Application` resource. See the following example:
      ```yaml
      kind: Application
      spec:
        syncPolicy:
          syncOptions:
          - PrunePropagationPolicy=background
      ```

      Setting the `background` deletion policy ensures that the `ManagedCluster` CR and all its associated resources are deleted.

## Preparing the GitOps ZTP site configuration repository {#ztp-preparing-the-ztp-git-repository_ztp-preparing-the-hub-cluster}

Before you can use the GitOps Zero Touch Provisioning (ZTP) pipeline, you need to prepare the Git repository to host the site configuration data.

**Prerequisites**

- You have configured the hub cluster GitOps applications for generating the required installation and policy custom resources (CRs).
- You have deployed the managed clusters using GitOps ZTP.

**Procedure**

1. Create a directory structure with separate paths for the `ClusterInstance` and `PolicyGenerator` or `PolicyGentemplate` CRs.
   :::note

   Keep `ClusterInstance` and `PolicyGenerator` or `PolicyGentemplate` CRs in separate directories. Both the `ClusterInstance` and `PolicyGenerator` or `PolicyGentemplate` directories must contain a `kustomization.yaml` file that explicitly includes the files in that directory.

   :::
2. Export the `argocd` directory from the `ztp-site-generate` container image using the following commands:
   ```terminal
   $ podman pull registry.redhat.io/openshift4/ztp-site-generate-rhel8:v4.22
   ```

   ```terminal
   $ mkdir -p ./out
   ```

   ```terminal
   $ podman run --log-driver=none --rm registry.redhat.io/openshift4/ztp-site-generate-rhel8:v4.22 extract /home/ztp --tar | tar x -C ./out
   ```
3. Check that the `out` directory contains the following subdirectories:
   - `out/extra-manifest` contains the source CR files that you use to create extra manifest `ConfigMap` resources through the `configMapGenerator` in the `kustomization.yaml` file. The `ClusterInstance` CR references these `ConfigMap` resources using the `extraManifestsRefs` field.
   - `out/source-crs` contains the source CR files that `PolicyGenerator` uses to generate the Red Hat Advanced Cluster Management (RHACM) policies.
   - `out/argocd/deployment` contains patches and YAML files to apply on the hub cluster for use in the next step of this procedure.
   - `out/argocd/example/clusterinstance` contains the examples for `ClusterInstance` and `PolicyGenerator` or `PolicyGentemplate` files that represent the recommended configuration.
4. Copy the `out/source-crs` folder and contents to the `PolicyGenerator` or `PolicyGentemplate` directory.
5. The out/extra-manifests directory contains the reference manifests for a RAN DU cluster. Copy the `out/extra-manifests` directory into the `ClusterInstance` folder. This directory should contain CRs from the `ztp-site-generate` container only. Do not add user-provided CRs here. If you want to work with user-provided CRs you must create another directory for that content. For example:
   ```text
   example/
     ├── acmpolicygenerator
     │   ├── kustomization.yaml
     │   └── source-crs/
     ├── policygentemplates
     │   ├── kustomization.yaml
     │   └── source-crs/
     └── clusterinstance
           ├── extra-manifests
           └── kustomization.yaml
   ```

   :::note

   Using `PolicyGenTemplate` CRs to manage and deploy policies to manage clusters will be deprecated in a future OpenShift Container Platform release. Equivalent and improved functionality is available by using Red Hat Advanced Cluster Management (RHACM) and `PolicyGenerator` CRs.

   :::
6. Commit the directory structure and the `kustomization.yaml` files and push to your Git repository. The initial push to Git should include the `kustomization.yaml` files.
   You can use the directory structure under `out/argocd/example` as a reference for the structure and content of your Git repository. That structure includes `ClusterInstance` and `PolicyGenerator` or `PolicyGentemplate` reference CRs for single-node, three-node, and standard clusters. Remove references to cluster types that you are not using.

   For all cluster types, you must:

   - Add the `source-crs` subdirectory to the `acmpolicygenerator` or `policygentemplates` directory.
   - Add the `extra-manifests` directory to the `clusterinstance` directory.
     The following example describes a set of CRs for a network of single-node clusters:

     ```text
     example/
       ├── acmpolicygenerator
       │   ├── acm-common-ranGen.yaml
       │   ├── acm-example-sno-site.yaml
       │   ├── acm-group-du-sno-ranGen.yaml
       │   ├── group-du-sno-validator-ranGen.yaml
       │   ├── kustomization.yaml
       │   ├── source-crs/
       │   └── ns.yaml
       └── clusterinstance
             ├── example-sno.yaml
             ├── extra-manifests/
             ├── custom-manifests/
             ├── KlusterletAddonConfigOverride.yaml
             └── kustomization.yaml
     ```

     where:

     <dl>
     <dt><code>extra-manifests/</code></dt>
     <dd>Contains reference manifests from the <code>ztp-container</code>.</dd>
     <dt><code>custom-manifests/</code></dt>
     <dd>Contains custom manifests.</dd>
     </dl>

:::warning

Using `PolicyGenTemplate` CRs to manage and deploy policies to managed clusters will be deprecated in an upcoming OpenShift Container Platform release. Equivalent and improved functionality is available using Red Hat Advanced Cluster Management (RHACM) and `PolicyGenerator` CRs.

For more information about `PolicyGenerator` resources, see the RHACM [Integrating Policy Generator](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.17/html-single/governance/index#integrate-policy-generator) documentation.

:::

**Additional resources**

- [Configuring managed cluster policies by using PolicyGenerator resources](/docs/edge_computing/policygenerator_for_ztp/ztp-configuring-managed-clusters-policygenerator#ztp-configuring-managed-clusters-policygenerator)
- [Comparing RHACM PolicyGenerator and PolicyGenTemplate resource patching](/docs/edge_computing/policygenerator_for_ztp/ztp-configuring-managed-clusters-policygenerator#ztp-comparing-pgt-and-rhacm-pg-patching-strategies_ztp-configuring-managed-clusters-policygenerator)

## Preparing the GitOps ZTP site configuration repository for version independence {#ztp-preparing-the-ztp-git-repository-ver-ind_ztp-preparing-the-hub-cluster}

You can use GitOps ZTP to manage source custom resources (CRs) for managed clusters that are running different versions of OpenShift Container Platform. This means that the version of OpenShift Container Platform running on the hub cluster can be independent of the version running on the managed clusters.

:::note

The following procedure assumes you are using `PolicyGenerator` resources instead of `PolicyGentemplate` resources for cluster policies management.

:::

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).
- You have logged in as a user with `cluster-admin` privileges.

**Procedure**

1. Create a directory structure with separate paths for the `ClusterInstance` and `PolicyGenerator` CRs.
2. Within the `PolicyGenerator` directory, create a directory for each OpenShift Container Platform version you want to make available. For each version, create the following resources:
   - `kustomization.yaml` file that explicitly includes the files in that directory
   - `source-crs` directory to contain reference CR configuration files from the `ztp-site-generate` container
     If you want to work with user-provided CRs, you must create a separate directory for them.
3. In the `/clusterinstance` directory, create a subdirectory for each OpenShift Container Platform version you want to make available. For each version, create at least one directory for reference CRs to be copied from the container. There is no restriction on the naming of directories or on the number of reference directories. If you want to work with custom manifests, you must create a separate directory for them.
   The following example describes a structure using user-provided manifests and CRs for different versions of OpenShift Container Platform:

   ```text
   ├── acmpolicygenerator
   │   ├── kustomization.yaml
   │   ├── version_4.13
   │   │   ├── common-ranGen.yaml
   │   │   ├── group-du-sno-ranGen.yaml
   │   │   ├── group-du-sno-validator-ranGen.yaml
   │   │   ├── helix56-v413.yaml
   │   │   ├── kustomization.yaml
   │   │   ├── ns.yaml
   │   │   └── source-crs/
   │   │      └── reference-crs/
   │   │      └── custom-crs/
   │   └── version_4.14
   │       ├── common-ranGen.yaml
   │       ├── group-du-sno-ranGen.yaml
   │       ├── group-du-sno-validator-ranGen.yaml
   │       ├── helix56-v414.yaml
   │       ├── kustomization.yaml
   │       ├── ns.yaml
   │       └── source-crs/
   │         └── reference-crs/
   │         └── custom-crs/
   └── clusterinstance
       ├── kustomization.yaml
       ├── version_4.13
       │   ├── helix56-v413.yaml
       │   ├── kustomization.yaml
       │   ├── extra-manifest/
       │   └── custom-manifest/
       └── version_4.14
           ├── helix57-v414.yaml
           ├── kustomization.yaml
           ├── extra-manifest/
           └── custom-manifest/
   ```

   where:

   <dl>
   <dt><code>kustomization.yaml</code> (top-level)</dt>
   <dd>Create a top-level <code>kustomization</code> YAML file.</dd>
   <dt><code>version_4.13</code>, <code>version_4.14</code></dt>
   <dd>Create the version-specific directories within the custom <code>/acmpolicygenerator</code> directory.</dd>
   <dt><code>kustomization.yaml</code> (per-version)</dt>
   <dd>Create a <code>kustomization.yaml</code> file for each version.</dd>
   <dt><code>source-crs/</code></dt>
   <dd>Create a <code>source-crs</code> directory for each version to contain reference CRs from the <code>ztp-site-generate</code> container.</dd>
   <dt><code>reference-crs/</code></dt>
   <dd>Create the <code>reference-crs</code> directory for policy CRs that are extracted from the ZTP container.</dd>
   <dt><code>custom-crs/</code></dt>
   <dd>Optional: Create a <code>custom-crs</code> directory for user-provided CRs.</dd>
   <dt><code>extra-manifest/</code></dt>
   <dd>Create a directory within the custom <code>/clusterinstance</code> directory to contain extra manifests from the <code>ztp-site-generate</code> container.</dd>
   <dt><code>custom-manifest/</code></dt>
   <dd>Create a folder to hold user-provided manifests.</dd>
   </dl>

   :::note

   In the example directory structure, each version subdirectory in the custom `/clusterinstance` directory contains two further subdirectories, one containing the reference manifests copied from the container, the other for custom manifests that you provide. The names assigned to those directories are examples.

   :::
4. Create ConfigMaps from the manifest directories and reference them in the `ClusterInstance` CR using the `extraManifestsRefs` field.
   Example `kustomization.yaml` with `configMapGenerator`:

   ```yaml
   apiVersion: kustomize.config.k8s.io/v1beta1
   kind: Kustomization

   configMapGenerator:
   - name: extra-manifests-cm
     namespace: helix56-v413
     files:
     - extra-manifest/workload-partitioning.yaml
     - extra-manifest/enable-crun-master.yaml
     - custom-manifest/custom-config.yaml
     # ...

   generatorOptions:
     disableNameSuffixHash: true
   ```

   where:

   <dl>
   <dt><code>extra-manifest/</code></dt>
   <dd>Extra manifest files from the <code>ztp-site-generate</code> container.</dd>
   <dt><code>custom-manifest/</code></dt>
   <dd>User-provided custom manifest files.</dd>
   </dl>
5. Edit the `ClusterInstance` CR to reference the `ConfigMap` CR:
   Example `ClusterInstance` CR:

   ```yaml
   apiVersion: siteconfig.open-cluster-management.io/v1alpha1
   kind: ClusterInstance
   metadata:
     name: helix56-v413
     namespace: helix56-v413
   spec:
     # ...
     extraManifestsRefs:
     - name: extra-manifests-cm
   ```

   The `extra-manifests-cm` value references the `ConfigMap` containing the extra manifests.
6. Edit the top-level `kustomization.yaml` file to control which OpenShift Container Platform versions are active. The following is an example of a `kustomization.yaml` file at the top level:
   ```yaml
   resources:
   - version_4.13
   #- version_4.14
   ```

   where:

   <dl>
   <dt><code>version_4.13</code></dt>
   <dd>Activate version 4.13.</dd>
   <dt><code>#- version_4.14</code></dt>
   <dd>Use comments to deactivate a version.</dd>
   </dl>

## Configuring the hub cluster for backup and restore {#ztp-configuring-the-hub-cluster-for-backup-and-restore_ztp-preparing-the-hub-cluster}

You can use GitOps ZTP to configure a set of policies to back up `BareMetalHost` resources. This allows you to recover data from a failed hub cluster and deploy a replacement cluster using Red Hat Advanced Cluster Management (RHACM).

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).
- You have logged in as a user with `cluster-admin` privileges.

**Procedure**

1. Create a policy to add the `cluster.open-cluster-management.io/backup=cluster-activation` label to all `BareMetalHost` resources that have the `infraenvs.agent-install.openshift.io` label. Save the policy as `BareMetalHostBackupPolicy.yaml`.
   The following example adds the `cluster.open-cluster-management.io/backup` label to all `BareMetalHost` resources that have the `infraenvs.agent-install.openshift.io` label:

   ```yaml title="Example Policy"
   apiVersion: policy.open-cluster-management.io/v1
   kind: Policy
   metadata:
     name: bmh-cluster-activation-label
     annotations:
       policy.open-cluster-management.io/description: Policy used to add the cluster.open-cluster-management.io/backup=cluster-activation label to all BareMetalHost resources
   spec:
     disabled: false
     policy-templates:
       - objectDefinition:
           apiVersion: policy.open-cluster-management.io/v1
           kind: ConfigurationPolicy
           metadata:
             name: set-bmh-backup-label
           spec:
             object-templates-raw: |
               {{- /* Set cluster-activation label on all BMH resources */ -}}
               {{- $infra_label := "infraenvs.agent-install.openshift.io" }}
               {{- range $bmh := (lookup "metal3.io/v1alpha1" "BareMetalHost" "" "" $infra_label).items }}
                   - complianceType: musthave
                     objectDefinition:
                       kind: BareMetalHost
                       apiVersion: metal3.io/v1alpha1
                       metadata:
                         name: {{ $bmh.metadata.name }}
                         namespace: {{ $bmh.metadata.namespace }}
                         labels:
                           cluster.open-cluster-management.io/backup: cluster-activation
               {{- end }}
             remediationAction: enforce
             severity: high
   ---
   apiVersion: cluster.open-cluster-management.io/v1beta1
   kind: Placement
   metadata:
     name: bmh-cluster-activation-label-pr
   spec:
     predicates:
       - requiredClusterSelector:
           labelSelector:
             matchExpressions:
               - key: name
                 operator: In
                 values:
                   - local-cluster
   ---
   apiVersion: policy.open-cluster-management.io/v1
   kind: PlacementBinding
   metadata:
     name: bmh-cluster-activation-label-binding
   placementRef:
     name: bmh-cluster-activation-label-pr
     apiGroup: cluster.open-cluster-management.io
     kind: Placement
   subjects:
     - name: bmh-cluster-activation-label
       apiGroup: policy.open-cluster-management.io
       kind: Policy
   ---
   apiVersion: cluster.open-cluster-management.io/v1beta2
   kind: ManagedClusterSetBinding
   metadata:
     name: default
     namespace: default
   spec:
     clusterSet: default
   ```

   If you apply the `cluster.open-cluster-management.io/backup: cluster-activation` label to `BareMetalHost` resources, the RHACM cluster backs up those resources. You can restore the `BareMetalHost` resources if the active cluster becomes unavailable, when restoring the hub activation resources.
2. Apply the policy by running the following command:
   ```terminal
   $ oc apply -f BareMetalHostBackupPolicy.yaml
   ```

**Verification**

1. Find all `BareMetalHost` resources with the label `infraenvs.agent-install.openshift.io`  by running the following command:
   ```terminal
   $ oc get BareMetalHost -A -l infraenvs.agent-install.openshift.io
   ```

   ```yaml title="Example output"
   NAMESPACE      NAME             STATE   CONSUMER   ONLINE   ERROR   AGE
   baremetal-ns   baremetal-name                      false            50s
   ```
2. Verify that the policy has applied the label `cluster.open-cluster-management.io/backup=cluster-activation` to all these resources, by running the following command:
   ```terminal
   $ oc get BareMetalHost -A -l infraenvs.agent-install.openshift.io,cluster.open-cluster-management.io/backup=cluster-activation
   ```

   ```yaml title="Example output"
   NAMESPACE      NAME             STATE   CONSUMER   ONLINE   ERROR   AGE
   baremetal-ns   baremetal-name                      false            50s
   ```

   The output must show the same list as in the previous step, which listed all `BareMetalHost` resources with the label `infraenvs.agent-install.openshift.io`. This confirms that all the `BareMetalHost` resources with the `infraenvs.agent-install.openshift.io` label also have the `cluster.open-cluster-management.io/backup: cluster-activation` label.

   The following example shows a `BareMetalHost` resource with the `infraenvs.agent-install.openshift.io` label. The resource must also have the `cluster.open-cluster-management.io/backup: cluster-activation` label, which was added by the policy created in step 1.

   ```yaml
   apiVersion: metal3.io/v1alpha1
   kind: BareMetalHost
   metadata:
     labels:
       cluster.open-cluster-management.io/backup: cluster-activation
       infraenvs.agent-install.openshift.io: value
     name: baremetal-name
     namespace: baremetal-ns
   ```

You can now use Red Hat Advanced Cluster Management to restore a managed cluster.

:::warning

When you restore `BareMetalHost` resources as part of restoring the cluster activation data, you must restore the `BareMetalHost` status. The following RHACM `Restore` resource example restores activation resources, including `BareMetalHost`, and also restores the status for the `BareMetalHost` resources:

```yaml
apiVersion: cluster.open-cluster-management.io/v1beta1
kind: Restore
metadata:
  name: restore-acm-bmh
  namespace: open-cluster-management-backup
spec:
  cleanupBeforeRestore: CleanupRestored
  veleroManagedClustersBackupName: latest
  veleroCredentialsBackupName: latest
  veleroResourcesBackupName: latest
  restoreStatus:
    includedResources:
      - BareMetalHosts
```

- Set `veleroManagedClustersBackupName: latest` to restore activation resources.
- Restores the status for `BareMetalHost` resources.

:::

**Additional resources**

- [Restoring managed cluster activation data](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/latest/html/business_continuity/business-cont-overview#managed-cluster-activation-data)
- [Active-passive configuration](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/latest/html/business_continuity/business-cont-overview#active-passive-config)
- [Restoring activation resources](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/latest/html/business_continuity/business-cont-overview#restore-activation-resources)
