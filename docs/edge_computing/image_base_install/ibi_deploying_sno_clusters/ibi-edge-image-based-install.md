---
title: About image-based deployments for managed single-node OpenShift
sidebar_position: 1
---

# About image-based deployments for managed single-node OpenShift {#ibi-edge-image-based-install}

<a id="ibi-edge-image-based-install"></a>

When a host preinstalled with single-node OpenShift using an image-based installation arrives at a remote site, a technician can easily reconfigure and deploy the host in a matter of minutes.

For clusters with a hub-and-spoke architecture, to complete the deployment of a preinstalled host, you must first define site-specific configuration resources on the hub cluster for each host. These resources contain configuration information such as the properties of the bare-metal host, authentication details, and other deployment and networking information.

The Image Based Install (IBI) Operator creates a configuration ISO from these resources, and then boots the host with the configuration ISO attached. The host mounts the configuration ISO and runs the reconfiguration process. When the reconfiguration completes, the single-node OpenShift cluster is ready.

:::note

You must create distinct configuration resources for each bare-metal host.

:::

See the following high-level steps to deploy a preinstalled host in a cluster with a hub-and-spoke architecture:

1. Install the IBI Operator on the hub cluster.
2. Create site-specific configuration resources in the hub cluster for each host.
3. The IBI Operator creates a configuration ISO from these resources and boots the target host with the configuration ISO attached.
4. The host mounts the configuration ISO and runs the reconfiguration process. When the reconfiguration completes, the single-node OpenShift cluster is ready.

:::note

Alternatively, you can manually deploy a preinstalled host for a cluster without using a hub cluster. You must define an `ImageBasedConfig` resource and an installation manifest, and provide these as inputs to the `openshift-install` installation program. For more information, see "Deploying a single-node OpenShift cluster using the `openshift-install` program".

:::

**Additional resources**

- [Deploying a single-node OpenShift cluster using the `openshift-install` program](/docs/edge_computing/image_base_install/ibi_deploying_sno_clusters/ibi-edge-image-based-install-standalone#create-standalone-config-iso_ibi-edge-image-based-install)

## Installing the Image Based Install Operator {#ibi-install-ibi-operator_ibi-edge-image-based-install}

The Image Based Install (IBI) Operator is part of the image-based deployment workflow for preinstalled single-node OpenShift on bare-metal hosts.

:::note

The IBI Operator is part of the multicluster engine for Kubernetes Operator from MCE version 2.7.

:::

**Prerequisites**

- You logged in as a user with `cluster-admin` privileges.
- You deployed a Red Hat Advanced Cluster Management (RHACM) hub cluster or you deployed the multicluster engine for Kubernetes Operator.
- You reviewed the required versions of software components in the section "Software prerequisites for an image-based installation".

**Procedure**

- Set the `enabled` specification to `true` for the `image-based-install-operator` component in the `MultiClusterEngine` resource by running the following command:
  ```terminal
  $ oc patch multiclusterengines.multicluster.openshift.io multiclusterengine --type json \
  --patch '[{"op": "add", "path":"/spec/overrides/components/-", "value": {"name":"image-based-install-operator","enabled": true}}]'
  ```

**Verification**

- Check that the Image Based Install Operator pod is running by running the following command:
  ```terminal
  $ oc get pods -A | grep image-based
  ```

  Example output:

  ```terminal
  multicluster-engine             image-based-install-operator-57fb8sc423-bxdj8             2/2     Running     0               5m
  ```

## Deploying a managed single-node OpenShift cluster using the IBI Operator {#ibi-create-config-iso_ibi-edge-image-based-install}

Create the site-specific configuration resources in the hub cluster to initiate the image-based deployment of a preinstalled host.

When you create these configuration resources in the hub cluster, the Image Based Install (IBI) Operator generates a configuration ISO and attaches it to the target host to begin the site-specific configuration  process. When the configuration process completes, the single-node OpenShift cluster is ready.

:::note

For more information about the configuration resources that you must configure in the hub cluster, see "Cluster configuration resources for deploying a preinstalled host".

:::

**Prerequisites**

- You preinstalled a host with single-node OpenShift using an image-based installation.
- You logged in as a user with `cluster-admin` privileges.
- You deployed a Red Hat Advanced Cluster Management (RHACM) hub cluster or you deployed the multicluster engine for Kubernetes operator (MCE).
- You installed the IBI Operator on the hub cluster.
- You created a pull secret to authenticate pull requests. For more information, see "Using image pull secrets".

**Procedure**

1. Create the `ibi-ns` namespace by running the following command:
   ```terminal
   $ oc create namespace ibi-ns
   ```
2. Create the `Secret` resource for your image registry:
   1. Create a YAML file that defines the `Secret` resource for your image registry:
      Example `secret-image-registry.yaml` file:

      ```yaml
      apiVersion: v1
      kind: Secret
      metadata:
        name: ibi-image-pull-secret
        namespace: ibi-ns
      stringData:
        .dockerconfigjson: <base64_docker_auth_code>
      type: kubernetes.io/dockerconfigjson
      ```

      where:

      <dl>
      <dt><code>&lt;base64_docker_auth_code&gt;</code></dt>
      <dd>Specifies base64-encoded credential details. See the "Additional resources" section for more information about using image pull secrets.</dd>
      </dl>
   2. Create the `Secret` resource for your image registry by running the following command:
      ```terminal
      $ oc create -f secret-image-registry.yaml
      ```
3. Optional: Configure static networking for the host:
   1. Create a `Secret` resource containing the static network configuration in `nmstate` format:
      Example `host-network-config-secret.yaml` file:

      ```yaml
      apiVersion: v1
      kind: Secret
      metadata:
       name: <network_secret_name>
       namespace: ibi-ns
      type: Opaque
      stringData:
       nmstate: |
        interfaces:
          - name: <interface_name>
            type: ethernet
            state: up
            ipv4:
              enabled: true
              address:
                - ip: 192.168.200.25
                  prefix-length: 24
              dhcp: false
            ipv6:
              enabled: false
        dns-resolver:
          config:
            server:
              - <dns_server_1>
              - 192.168.15.48
        routes:
          config:
            - destination: 0.0.0.0/0
              metric: 150
              next-hop-address: 192.168.200.254
              next-hop-interface: <interface_name>
              table-id: 254
      ```

      where:

      <dl>
      <dt><code>&lt;network_secret_name&gt;</code></dt>
      <dd>Specifies the name for the <code>Secret</code> resource, for example <code>host-network-config-secret</code>.</dd>
      <dt><code>nmstate</code></dt>
      <dd>Specifies the static network configuration in <code>nmstate</code> format.</dd>
      <dt><code>&lt;interface_name&gt;</code></dt>
      <dd>Specifies the name of the interface on the host, for example <code>ens1f0</code>. The name of the interface must match the actual NIC name as shown in the operating system. To use your MAC address for NIC matching, set the <code>identifier</code> field to <code>mac-address</code>.</dd>
      <dt><code>dhcp: false</code></dt>
      <dd>Specifies that DHCP is disabled to ensure <code>nmstate</code> assigns the static IP address to the interface.</dd>
      <dt><code>&lt;dns_server_1&gt;</code></dt>
      <dd>Specifies one or more DNS servers that the system will use to resolve domain names, for example <code>192.168.15.47</code>.</dd>
      <dt><code>config</code></dt>
      <dd>Specifies the default route through the <code>ens1f0</code> interface to the next hop IP address <code>192.168.200.254</code>.</dd>
      </dl>
4. Create the `BareMetalHost` and `Secret` resources:
   1. Create a YAML file that defines the `BareMetalHost` and `Secret` resources:
      Example `ibi-bmh.yaml` file:

      ```yaml
      apiVersion: metal3.io/v1alpha1
      kind: BareMetalHost
      metadata:
        name: <baremetalhost_name>
        namespace: ibi-ns
      spec:
        online: <online_status>
        bootMACAddress: <boot_mac_address>
        bmc:
          address: <bmc_address>
          credentialsName: <bmh_secret_name>
        preprovisioningNetworkDataName: <network_secret_name>
        automatedCleaningMode: disabled
        externallyProvisioned: true
      ---
      apiVersion: v1
      kind: Secret
      metadata:
        name: <bmh_secret_name>
        namespace: ibi-ns
      type: Opaque
      data:
        username: <username>
        password: <password>
      ```

      where:

      <dl>
      <dt><code>&lt;baremetalhost_name&gt;</code></dt>
      <dd>Specifies the name for the <code>BareMetalHost</code> resource, for example <code>ibi-bmh</code>.</dd>
      <dt><code>&lt;online_status&gt;</code></dt>
      <dd>Specifies if the host should be online, for example <code>false</code>.</dd>
      <dt><code>&lt;boot_mac_address&gt;</code></dt>
      <dd>Specifies the host boot MAC address, for example <code>00:a5:12:55:62:64</code>.</dd>
      <dt><code>&lt;bmc_address&gt;</code></dt>
      <dd>Specifies the BMC address, for example <code>redfish-virtualmedia+http://192.168.111.1:8000/redfish/v1/Systems/8a5babac-94d0-4c20-b282-50dc3a0a32b5</code>. You can only use bare-metal host drivers that support virtual media networking booting, for example redfish-virtualmedia and idrac-virtualmedia.</dd>
      <dt><code>&lt;bmh_secret_name&gt;</code></dt>
      <dd>Specifies the name of the bare-metal host <code>Secret</code> resource, for example <code>ibi-bmh-bmc-secret</code>.</dd>
      <dt><code>&lt;network_secret_name&gt;</code></dt>
      <dd>(Optional) Specifies the name of the <code>Secret</code> resource containing the static network configuration for the host, for example <code>host-network-config-secret</code>.</dd>
      <dt><code>automatedCleaningMode: disabled</code></dt>
      <dd>Specifies that automated cleaning is disabled to prevent the provisioning service from deleting all preinstallation artifacts, such as the seed image, during disk inspection.</dd>
      <dt><code>externallyProvisioned: true</code></dt>
      <dd>Specifies that the host is externally provisioned to enable it to boot from the preinstalled disk, instead of the configuration ISO.</dd>
      <dt><code>&lt;username&gt;</code></dt>
      <dd>Specifies the username for BMC authentication.</dd>
      <dt><code>&lt;password&gt;</code></dt>
      <dd>Specifies the password for BMC authentication.</dd>
      </dl>
   2. Create the `BareMetalHost` and `Secret` resources by running the following command:
      ```terminal
      $ oc create -f ibi-bmh.yaml
      ```
5. Create the `ClusterImageSet` resource:
   1. Create a YAML file that defines the `ClusterImageSet` resource:
      Example `ibi-cluster-image-set.yaml` file:

      ```yaml
      apiVersion: hive.openshift.io/v1
      kind: ClusterImageSet
      metadata:
        name: <clusterimageset_name>
      spec:
        releaseImage: <release_image>
      ```

      where:

      <dl>
      <dt><code>&lt;clusterimageset_name&gt;</code></dt>
      <dd>Specifies the name for the <code>ClusterImageSet</code> resource, for example <code>ibi-img-version-arch</code>.</dd>
      <dt><code>&lt;release_image&gt;</code></dt>
      <dd>Specifies the address for the release image to use for the deployment, for example <code>ibi.example.com:path/to/release/images:version-arch</code>. If you use a different image registry compared to the image registry used during seed image generation, ensure that the OpenShift Container Platform version for the release image remains the same.</dd>
      </dl>
   2. Create the `ClusterImageSet` resource by running the following command:
      ```terminal
      $ oc apply -f ibi-cluster-image-set.yaml
      ```
6. Create the `ImageClusterInstall` resource:
   1. Create a YAML file that defines the `ImageClusterInstall` resource:
      Example `ibi-image-cluster-install.yaml` file:

      ```yaml
      apiVersion: extensions.hive.openshift.io/v1alpha1
      kind: ImageClusterInstall
      metadata:
        name: <imageclusterinstall_name>
        namespace: ibi-ns
      spec:
        bareMetalHostRef:
          name: <baremetalhost_name>
          namespace: ibi-ns
        clusterDeploymentRef:
          name: <clusterdeployment_name>
        hostname: <cluster_hostname>
        imageSetRef:
          name: <clusterimageset_name>
        machineNetworks:
        - cidr: 10.0.0.0/24
        #- cidr: fd01::/64
        proxy:
          httpProxy: "http://proxy.example.com:8080"
          #httpsProxy: "http://proxy.example.com:8080"
          #noProxy: "no_proxy.example.com"
      ```

      where:

      <dl>
      <dt><code>&lt;imageclusterinstall_name&gt;</code></dt>
      <dd>Specifies the name for the <code>ImageClusterInstall</code> resource, for example <code>ibi-image-install</code>.</dd>
      <dt><code>&lt;baremetalhost_name&gt;</code></dt>
      <dd>Specifies the <code>BareMetalHost</code> resource that you want to target for the image-based installation, for example <code>ibi-bmh</code>.</dd>
      <dt><code>&lt;clusterdeployment_name&gt;</code></dt>
      <dd>Specifies the name of the <code>ClusterDeployment</code> resource that you want to use for the image-based installation of the target host, for example <code>ibi-cluster-deployment</code>.</dd>
      <dt><code>&lt;cluster_hostname&gt;</code></dt>
      <dd>Specifies the hostname for the cluster, for example <code>ibi-host</code>.</dd>
      <dt><code>&lt;clusterimageset_name&gt;</code></dt>
      <dd>Specifies the name of the <code>ClusterImageSet</code> resource you used to define the container release images to use for deployment, for example <code>ibi-img-version-arch</code>.</dd>
      <dt><code>machineNetworks</code></dt>
      <dd>Specifies the public Classless Inter-Domain Routing (CIDR) of the external network. For dual-stack networking, you can specify both IPv4 and IPv6 CIDRs using a list format. The first CIDR in the list is the primary address family and must match the primary address family of the seed cluster.</dd>
      <dt><code>proxy</code></dt>
      <dd>(Optional) Specifies a proxy to use for the cluster deployment.</dd>
      </dl>

      :::warning

      If your cluster deployment requires a proxy configuration, you must do the following:

      - Create a seed image from a seed cluster featuring a proxy configuration. The proxy configurations do not have to match.
      - Configure the `machineNetwork` field in your installation manifest.

      :::
   2. Create the `ImageClusterInstall` resource by running the following command:
      ```terminal
      $ oc create -f ibi-image-cluster-install.yaml
      ```
7. Create the `ClusterDeployment` resource:
   1. Create a YAML file that defines the `ClusterDeployment` resource:
      Example `ibi-cluster-deployment.yaml` file:

      ```yaml
      apiVersion: hive.openshift.io/v1
      kind: ClusterDeployment
      metadata:
        name: <clusterdeployment_name>
        namespace: <namespace>
      spec:
        baseDomain: <base_domain>
        clusterInstallRef:
          group: extensions.hive.openshift.io
          kind: ImageClusterInstall
          name: <imageclusterinstall_name>
          version: v1alpha1
        clusterName: <cluster_name>
        platform:
          none: {}
        pullSecretRef:
          name: <pull_secret_name>
      ```

      where:

      <dl>
      <dt><code>&lt;clusterdeployment_name&gt;</code></dt>
      <dd>Specifies the name for the <code>ClusterDeployment</code> resource, for example <code>ibi-cluster-deployment</code>.</dd>
      <dt><code>&lt;namespace&gt;</code></dt>
      <dd>Specifies the namespace for the <code>ClusterDeployment</code> resource, for example <code>ibi-ns</code>.</dd>
      <dt><code>&lt;base_domain&gt;</code></dt>
      <dd>Specifies the base domain that the cluster should belong to, for example <code>example.com</code>.</dd>
      <dt><code>&lt;imageclusterinstall_name&gt;</code></dt>
      <dd>Specifies the name of the <code>ImageClusterInstall</code> in which you defined the container images to use for the image-based installation of the target host, for example <code>ibi-image-install</code>.</dd>
      <dt><code>&lt;cluster_name&gt;</code></dt>
      <dd>Specifies a name for the cluster, for example <code>ibi-cluster</code>.</dd>
      <dt><code>&lt;pull_secret_name&gt;</code></dt>
      <dd>Specifies the secret to use for pulling images from your image registry, for example <code>ibi-image-pull-secret</code>.</dd>
      </dl>
   2. Create the `ClusterDeployment` resource by running the following command:
      ```terminal
      $ oc apply -f ibi-cluster-deployment.yaml
      ```
8. Create the `ManagedCluster` resource:
   1. Create a YAML file that defines the `ManagedCluster` resource:
      Example `ibi-managed.yaml` file:

      ```yaml
      apiVersion: cluster.open-cluster-management.io/v1
      kind: ManagedCluster
      metadata:
        name: <managedcluster_name>
      spec:
        hubAcceptsClient: <hub_accepts_client>
      ```

      where:

      <dl>
      <dt><code>&lt;managedcluster_name&gt;</code></dt>
      <dd>Specifies the name for the <code>ManagedCluster</code> resource, for example <code>sno-ibi</code>.</dd>
      <dt><code>&lt;hub_accepts_client&gt;</code></dt>
      <dd>Specifies whether RHACM manages the cluster. Set to <code>true</code> to enable management.</dd>
      </dl>
   2. Create the `ManagedCluster` resource by running the following command:
      ```terminal
      $ oc apply -f ibi-managed.yaml
      ```

**Verification**

1. Check the status of the `ImageClusterInstall` in the hub cluster to monitor the progress of the target host installation by running the following command:
   ```terminal
   $ oc get imageclusterinstall
   ```

   Example output:

   ```terminal
   NAME       REQUIREMENTSMET           COMPLETED                     BAREMETALHOSTREF
   target-0   HostValidationSucceeded   ClusterInstallationSucceeded  ibi-bmh
   ```

   :::warning

   If the `ImageClusterInstall` resource is deleted, the IBI Operator reattaches the `BareMetalHost` resource and reboots the machine.

   :::
2. When the installation completes, you can retrieve the `kubeconfig` secret to log in to the managed cluster by running the following command:
   ```terminal
   $ oc extract secret/<cluster_name>-admin-kubeconfig -n <cluster_namespace>  --to - > <directory>/<cluster_name>-kubeconfig
   ```

   where:

   <dl>
   <dt><code>&lt;cluster_name&gt;</code></dt>
   <dd>Specifies the name of the cluster.</dd>
   <dt><code>&lt;cluster_namespace&gt;</code></dt>
   <dd>Specifies the namespace of the cluster.</dd>
   <dt><code>&lt;directory&gt;</code></dt>
   <dd>Specifies the directory in which to create the file.</dd>
   </dl>

**Additional resources**

- [Using image pull secrets](/docs/openshift_images/managing_images/using-image-pull-secrets)
- [Cluster configuration resources for deploying a preinstalled host](/docs/edge_computing/image_base_install/ibi_deploying_sno_clusters/ibi-edge-image-based-install#ibi-managed-cluster-config-resources_ibi-edge-image-based-install)

### Cluster configuration resources for deploying a preinstalled host {#ibi-managed-cluster-config-resources_ibi-edge-image-based-install}

To complete a deployment for a preinstalled host at a remote site, you must configure the following site-specifc cluster configuration resources in the hub cluster for each bare-metal host.

**Cluster configuration resources reference**

| Resource | Description |
| --- | --- |
| `Namespace` | Namespace for the managed single-node OpenShift cluster. |
| `BareMetalHost` | Describes the physical host and its properties, such as the provisioning and hardware configuration. |
| `Secret` for the bare-metal host | Credentials for the host BMC. |
| `Secret` for the bare-metal host static network configuration | Optional: Describes static network configuration for the target host. |
| `Secret` for the image registry | Credentials for the image registry. The secret for the image registry must be of type `kubernetes.io/dockerconfigjson`. |
| `ImageClusterInstall` | References the bare-metal host, deployment, and image set resources. |
| `ClusterImageSet` | Describes the release images to use for the cluster. |
| `ClusterDeployment` | Describes networking, authentication, and platform-specific settings. |
| `ManagedCluster` | Describes cluster details to enable Red Hat Advanced Cluster Management (RHACM) to register and manage. |
| `ConfigMap` | Optional: Describes additional configurations for the cluster deployment, such as adding a bundle of trusted certificates for the host to ensure trusted communications for cluster services. |

### ImageClusterInstall resource API specifications {#ibi-image-cluster-install-api-spec_ibi-edge-image-based-install}

The following content describes the API specifications for the `ImageClusterInstall` resource. This resource is the endpoint for the Image Based Install Operator.

**Required specifications**

<table>
<tbody>
<tr>
  <td>Specification</td>
  <td>Type</td>
  <td>Description</td>
</tr>
<tr>
  <td><code>imageSetRef</code></td>
  <td><code>string</code></td>
  <td>Specify the name of the <code>ClusterImageSet</code> resource that defines the release images for the deployment.</td>
</tr>
<tr>
  <td><code>hostname</code></td>
  <td><code>string</code></td>
  <td>Specify the hostname for the cluster.</td>
</tr>
<tr>
  <td><code>sshKey</code></td>
  <td><code>string</code></td>
  <td>Specify your SSH key to provide SSH access to the target host.</td>
</tr>
</tbody>
</table>

**Optional specifications**

<table>
<tbody>
<tr>
  <td>Specification</td>
  <td>Type</td>
  <td>Description</td>
</tr>
<tr>
  <td><code>clusterDeploymentRef</code></td>
  <td><code>string</code></td>
  <td>Specify the name of the <code>ClusterDeployment</code> resource that you want to use for the image-based installation of the target host.</td>
</tr>
<tr>
  <td><code>clusterMetadata</code></td>
  <td><code>string</code></td>
  <td>After the deployment completes, this specification is automatically populated with metadata information about the cluster, including the <code>cluster-admin</code> kubeconfig credentials for logging in to the cluster.</td>
</tr>
<tr>
  <td><code>imageDigestSources</code></td>
  <td><code>string</code></td>
  <td>Specifies the sources or repositories for the release-image content, for example: [source,yaml] ---- imageDigestSources: - mirrors: - "registry.example.com:5000/ocp4/openshift4" source: "quay.io/openshift-release-dev/ocp-release" ----</td>
</tr>
<tr>
  <td><code>extraManifestsRefs</code></td>
  <td><code>string</code></td>
  <td>Specify a <code>ConfigMap</code> resource containing additional manifests to be applied to the target cluster.</td>
</tr>
<tr>
  <td><code>bareMetalHostRef</code></td>
  <td><code>string</code></td>
  <td>Specify the <code>bareMetalHost</code> resource to use for the cluster deployment</td>
</tr>
<tr>
  <td><code>machineNetworks</code></td>
  <td><code>string</code></td>
  <td>Specify the public Classless Inter-Domain Routing (CIDR) of the external network. For dual-stack networking, you can specify both IPv4 and IPv6 CIDRs using a list format. The first CIDR in the list is the primary address family and must match the primary address family of the seed cluster.</td>
</tr>
<tr>
  <td><code>proxy</code></td>
  <td><code>string</code></td>
  <td>Specifies proxy settings for the cluster, for example: [source,yaml]<br><br>---- proxy: httpProxy: "http://proxy.example.com:8080" httpsProxy: "http://proxy.example.com:8080" noProxy: "no_proxy.example.com" ----</td>
</tr>
<tr>
  <td><code>caBundleRef</code></td>
  <td><code>string</code></td>
  <td>Specify a <code>ConfigMap</code> resource containing the new bundle of trusted certificates for the host.</td>
</tr>
</tbody>
</table>

## ConfigMap resources for extra manifests {#ibi-extra-manifests-configmap_ibi-edge-image-based-install}

You can optionally create a `ConfigMap` resource to define additional manifests in an image-based deployment for managed single-node OpenShift clusters.

After you create the `ConfigMap` resource, reference it in the `ImageClusterInstall` resource. During deployment, the IBI Operator includes the extra manifests in the deployment.

### Creating a ConfigMap resource to add extra manifests in an image-based deployment {#ibi-create-extra-manifest-configmap_ibi-edge-image-based-install}

You can use a `ConfigMap` resource to add extra manifests to the image-based deployment for single-node OpenShift clusters.

The following example adds an single-root I/O virtualization (SR-IOV) network to the deployment.

:::note

Filenames for extra manifests must not exceed 30 characters. Longer filenames might cause deployment failures.

:::

Before you begin, ensure that:

- You preinstalled a host with single-node OpenShift using an image-based installation.
- You logged in as a user with `cluster-admin` privileges.

To create the `ConfigMap` resource, complete the following steps:

1. Create the `SriovNetworkNodePolicy` and `SriovNetwork` resources:
   1. Create a YAML file that defines the resources, as in the following example:
      ```yaml
      apiVersion: sriovnetwork.openshift.io/v1
      kind: SriovNetworkNodePolicy
      metadata:
        name: "example-sriov-node-policy"
        namespace: openshift-sriov-network-operator
      spec:
        deviceType: vfio-pci
        isRdma: false
        nicSelector:
          pfNames: [ens1f0]
        nodeSelector:
          node-role.kubernetes.io/master: ""
        mtu: 1500
        numVfs: 8
        priority: 99
        resourceName: example-sriov-node-policy
      ---
      apiVersion: sriovnetwork.openshift.io/v1
      kind: SriovNetwork
      metadata:
        name: "example-sriov-network"
        namespace: openshift-sriov-network-operator
      spec:
        ipam: |-
          {
          }
        linkState: auto
        networkNamespace: sriov-namespace
        resourceName: example-sriov-node-policy
        spoofChk: "on"
        trust: "off"
      ```
   2. Create the `ConfigMap` resource by running the following command:
      ```terminal
      $ oc create configmap sr-iov-extra-manifest --from-file=sriov-extra-manifest.yaml -n <namespace>
      ```

      where:

      <dl>
      <dt><code>&lt;namespace&gt;</code></dt>
      <dd>Specifies the namespace that has the <code>ImageClusterInstall</code> resource, for example <code>ibi-ns</code>. Example output:</dd>
      </dl>

      ```terminal
      configmap/sr-iov-extra-manifest created
      ```

      :::note

      If you add more than one extra manifest, and the manifests must be applied in a specific order, you must prefix the filenames of the manifests with numbers that represent the required order. For example, `00-namespace.yaml`, `01-sriov-extra-manifest.yaml`, and so on.

      :::
2. Reference the `ConfigMap` resource in the `spec.extraManifestsRefs` field of the `ImageClusterInstall` resource:
   ```yaml
   #...
     spec:
       extraManifestsRefs:
       - name: sr-iov-extra-manifest
   #...
   ```

### Creating a ConfigMap resource to add a CA bundle in an image-based deployment {#ibi-create-ca-extra-manifest-configmap_ibi-edge-image-based-install}

You can use a `ConfigMap` resource to add a certificate authority (CA) bundle to the host to ensure trusted communications for cluster services.

After you create the `ConfigMap` resource, reference it in the `spec.caBundleRef` field of the `ImageClusterInstall` resource.

Before you begin, ensure that:

- You preinstalled a host with single-node OpenShift using an image-based installation.
- You logged in as a user with `cluster-admin` privileges.

To create the CA bundle `ConfigMap` resource, complete the following steps:

1. Create a CA bundle file called `tls-ca-bundle.pem`, as in the following example:
   ```text
   -----BEGIN CERTIFICATE-----
   MIIDXTCCAkWgAwIBAgIJAKmjYKJbIyz3MA0GCSqGSIb3DQEBCwUAMEUxCzAJBgNV
   ...Custom CA certificate bundle...
   4WPl0Qb27Sb1xZyAsy1ww6MYb98EovazUSfjYr2EVF6ThcAPu4/sMxUV7He2J6Jd
   cA8SMRwpUbz3LXY=
   -----END CERTIFICATE-----
   ```
2. Create the `ConfigMap` object by running the following command:
   ```terminal
   $ oc create configmap custom-ca --from-file=tls-ca-bundle.pem -n ibi-ns
   ```

   where:

   <dl>
   <dt><code>custom-ca</code></dt>
   <dd>Specifies the name for the <code>ConfigMap</code> resource.</dd>
   <dt><code>tls-ca-bundle.pem</code></dt>
   <dd>Specifies the key for the <code>data</code> entry in the <code>ConfigMap</code> resource. You must include a <code>data</code> entry with the <code>tls-ca-bundle.pem</code> key.</dd>
   <dt><code>ibi-ns</code></dt>
   <dd>Specifies the namespace that has the <code>ImageClusterInstall</code> resource. Example output:</dd>
   </dl>

   ```terminal
   configmap/custom-ca created
   ```
3. Reference the `ConfigMap` resource in the `spec.caBundleRef` field of the `ImageClusterInstall` resource:
   ```yaml
   #...
     spec:
       caBundleRef:
         name: custom-ca
   #...
   ```

**Additional resources**

- [About the BareMetalHost resource](/docs/installing/installing_bare_metal/bare-metal-postinstallation-configuration#bmo-about-the-baremetalhost-resource_bare-metal-postinstallation-configuration)
- [Using image pull secrets](/docs/openshift_images/managing_images/using-image-pull-secrets)
- [Reference specifications for the image-based-config.yaml manifest](/docs/edge_computing/image_base_install/ibi-factory-image-based-install#ibi-installer-installation-config_ibi-factory-image-based-install)
