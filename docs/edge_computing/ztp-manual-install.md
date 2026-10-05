---
title: Manually installing a single-node OpenShift cluster with GitOps ZTP
sidebar_position: 5
---

# Manually installing a single-node OpenShift cluster with GitOps ZTP {#ztp-manual-install}

<a id="ztp-manual-install"></a>

You can deploy a managed single-node OpenShift cluster by using Red Hat Advanced Cluster Management (RHACM) and the assisted service.

:::note

If you are creating multiple managed clusters, use the `ClusterInstance` method described in [Deploying far edge sites with ZTP](/docs/edge_computing/ztp-deploying-far-edge-sites#ztp-deploying-far-edge-sites).

:::

:::warning

The target bare-metal host must meet the networking, firmware, and hardware requirements listed in [Recommended cluster configuration for vDU application workloads](/docs/edge_computing/ztp-reference-cluster-configuration-for-vdu#sno-configure-for-vdu).

:::

## Extracting reference and example CRs from the ztp-site-generate container {#ztp-generating-install-and-config-crs-manually_ztp-manual-install}

Use the `ztp-site-generate` container to extract reference custom resources (CRs) and example `ClusterInstance` CRs to prepare for cluster installation and Day 2 configuration.

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).
- You have logged in to the hub cluster as a user with `cluster-admin` privileges.
- You installed `podman`.

**Procedure**

1. Create an output folder by running the following command:
   ```terminal
   $ mkdir -p ./out
   ```
2. Log in to the Ecosystem container registry with your credentials by running the following command:
   ```terminal
   $ podman login registry.redhat.io
   ```
3. Extract the reference and example CRs from the `ztp-site-generate` container image by running the following command:
   ```terminal
   $ podman run --log-driver=none --rm registry.redhat.io/openshift4/ztp-site-generate-rhel8:v4.22 extract /home/ztp --tar | tar x -C ./out
   ```

   The `./out` directory contains the reference `PolicyGenerator` and `ClusterInstance` CRs in the `out/argocd/example/` folder.

   ```terminal title="Example output"
   out
    └── argocd
         └── example
              ├── acmpolicygenerator
              │     ├── {policy-prefix}common-ranGen.yaml
              │     ├── {policy-prefix}example-sno-site.yaml
              │     ├── {policy-prefix}group-du-sno-ranGen.yaml
              │     ├── ...
              │     ├── kustomization.yaml
              │     └── ns.yaml
              └── clusterinstance
                    ├── example-sno.yaml
                    ├── example-3node.yaml
                    ├── example-standard.yaml
                    └── ...
   ```
4. Create a `ClusterInstance` CR for your cluster.
   Use the example `ClusterInstance` CRs in the `out/argocd/example/clusterinstance/` folder that you previously extracted from the `ztp-site-generate` container as a reference. The folder includes example files for single node, three-node, and standard clusters:

   - `example-sno.yaml`
   - `example-3node.yaml`
   - `example-standard.yaml`
     Change the cluster and host details in the example file to match the type of cluster you want to install. For example:

     ```yaml title="Example single-node OpenShift ClusterInstance CR"
     # example-node1-bmh-secret & assisted-deployment-pull-secret need to be created under same namespace example-ai-sno
     ---
     apiVersion: siteconfig.open-cluster-management.io/v1alpha1
     kind: ClusterInstance
     metadata:
       name: "example-ai-sno"
       namespace: "example-ai-sno"
     spec:
       baseDomain: "example.com"
       pullSecretRef:
         name: "assisted-deployment-pull-secret"
       clusterImageSetNameRef: "openshift-4.22"
       sshPublicKey: "ssh-rsa AAAA..."
       clusterName: "example-ai-sno"
       networkType: "OVNKubernetes"
       # installConfigOverrides is a generic way of passing install-config
       # parameters through the siteConfig.  The 'capabilities' field configures
       # the composable openshift feature.  In this 'capabilities' setting, we
       # remove all the optional set of components.
       # Notes:
       # - OperatorLifecycleManager is needed for 4.15 and later
       # - NodeTuning is needed for 4.13 and later, not for 4.12 and earlier
       # - Ingress is needed for 4.16 and later
       installConfigOverrides: |
         {
           "capabilities": {
             "baselineCapabilitySet": "None",
             "additionalEnabledCapabilities": [
               "NodeTuning",
               "OperatorLifecycleManager",
               "Ingress"
             ]
           }
         }
       # Include references to extraManifest ConfigMaps.
       extraManifestsRefs:
         - name: sno-extra-manifest-configmap
       extraLabels:
         ManagedCluster:
           # These example cluster labels correspond to the bindingRules in the PolicyGenTemplate examples
           du-profile: "latest"
           # These example cluster labels correspond to the bindingRules in the PolicyGenTemplate examples in ../policygentemplates:
           # ../policygentemplates/common-ranGen.yaml will apply to all clusters with 'common: true'
           common: "true"
           # ../policygentemplates/group-du-sno-ranGen.yaml will apply to all clusters with 'group-du-sno: ""'
           group-du-sno: ""
           # ../policygentemplates/example-sno-site.yaml will apply to all clusters with 'sites: "example-sno"'
           # Normally this should match or contain the cluster name so it only applies to a single cluster
           sites : "example-sno"
       clusterNetwork:
         - cidr: 1001:1::/48
           hostPrefix: 64
       machineNetwork:
         - cidr: 1111:2222:3333:4444::/64
       serviceNetwork:
         - cidr: 1001:2::/112
       additionalNTPSources:
         - 1111:2222:3333:4444::2
       # Initiates the cluster for workload partitioning. Setting specific reserved/isolated CPUSets is done via PolicyTemplate
       # please see Workload Partitioning Feature for a complete guide.
       cpuPartitioningMode: AllNodes
       templateRefs:
         - name: ai-cluster-templates-v1
           namespace: open-cluster-management
       nodes:
         - hostName: "example-node1.example.com"
           role: "master"
           bmcAddress: "idrac-virtualmedia+https://[1111:2222:3333:4444::bbbb:1]/redfish/v1/Systems/System.Embedded.1"
           bmcCredentialsName:
             name: "example-node1-bmh-secret"
           bootMACAddress: "AA:BB:CC:DD:EE:11"
           # Use UEFISecureBoot to enable secure boot, UEFI to disable.
           bootMode: "UEFISecureBoot"
           rootDeviceHints:
             deviceName: "/dev/disk/by-path/pci-0000:01:00.0-scsi-0:2:0:0"
           # disk partition at `/var/lib/containers` with ignitionConfigOverride. Some values must be updated. See DiskPartitionContainer.md in argocd folder for more details
           ignitionConfigOverride: |
             {
               "ignition": {
                 "version": "3.2.0"
               },
               "storage": {
                 "disks": [
                   {
                     "device": "/dev/disk/by-path/pci-0000:01:00.0-scsi-0:2:0:0",
                     "partitions": [
                       {
                         "label": "var-lib-containers",
                         "sizeMiB": 0,
                         "startMiB": 250000
                       }
                     ],
                     "wipeTable": false
                   }
                 ],
                 "filesystems": [
                   {
                     "device": "/dev/disk/by-partlabel/var-lib-containers",
                     "format": "xfs",
                     "mountOptions": [
                       "defaults",
                       "prjquota"
                     ],
                     "path": "/var/lib/containers",
                     "wipeFilesystem": true
                   }
                 ]
               },
               "systemd": {
                 "units": [
                   {
                     "contents": "# Generated by Butane\n[Unit]\nRequires=systemd-fsck@dev-disk-by\\x2dpartlabel-var\\x2dlib\\x2dcontainers.service\nAfter=systemd-fsck@dev-disk-by\\x2dpartlabel-var\\x2dlib\\x2dcontainers.service\n\n[Mount]\nWhere=/var/lib/containers\nWhat=/dev/disk/by-partlabel/var-lib-containers\nType=xfs\nOptions=defaults,prjquota\n\n[Install]\nRequiredBy=local-fs.target",
                     "enabled": true,
                     "name": "var-lib-containers.mount"
                   }
                 ]
               }
             }
           nodeNetwork:
             interfaces:
               - name: eno1
                 macAddress: "AA:BB:CC:DD:EE:11"
             config:
               interfaces:
                 - name: eno1
                   type: ethernet
                   state: up
                   ipv4:
                     enabled: false
                   ipv6:
                     enabled: true
                     address:
                     # For SNO sites with static IP addresses, the node-specific,
                     # API and Ingress IPs should all be the same and configured on
                     # the interface
                     - ip: 1111:2222:3333:4444::aaaa:1
                       prefix-length: 64
               dns-resolver:
                 config:
                   search:
                   - example.com
                   server:
                   - 1111:2222:3333:4444::2
               routes:
                 config:
                 - destination: ::/0
                   next-hop-interface: eno1
                   next-hop-address: 1111:2222:3333:4444::1
                   table-id: 254
           templateRefs:
             - name: ai-node-templates-v1
               namespace: open-cluster-management
     ```

     :::note

     Optional: To provision additional install-time manifests on the provisioned cluster, create the extra manifest CRs and apply them to the hub cluster. Then reference them in the `extraManifestsRefs` field of the `ClusterInstance` CR. For more information, see "Customizing extra installation manifests in the GitOps ZTP pipeline".

     :::
5. Optional: Generate Day 2 configuration CRs from the reference `PolicyGenerator` CRs:
   1. Create an output folder for the configuration CRs by running the following command:
      ```terminal
      $ mkdir -p ./ref
      ```
   2. Generate the configuration CRs by running the following command:
      ```terminal
      $ podman run -it --rm -v `pwd`/out/argocd/example/policygentemplates:/resources:Z -v `pwd`/ref:/output:Z,U registry.redhat.io/openshift4/ztp-site-generate-rhel8:v4.22 generator config -N . /output
      ```

      The command generates example group and cluster-specific configuration CRs in the `./ref` folder. You can apply these CRs to the cluster after installation is complete.

## Create the managed bare-metal host secrets {#ztp-creating-the-site-secrets_ztp-manual-install}

Add the required `Secret` custom resources (CRs) for the managed bare-metal host to the hub cluster. You need a secret for the GitOps Zero Touch Provisioning (ZTP) pipeline to access the Baseboard Management Controller (BMC) and a secret for the assisted installer service to pull cluster installation images from the registry.

:::note

The secrets are referenced from the `ClusterInstance` CR by name. The namespace must match the `ClusterInstance` namespace.

:::

**Procedure**

1. Create a YAML secret file containing credentials for the host Baseboard Management Controller (BMC) and a pull secret required for installing OpenShift and all add-on cluster Operators:
   1. Save the following YAML as the file `example-sno-secret.yaml`:
      ```yaml
      apiVersion: v1
      kind: Secret
      metadata:
        name: example-sno-bmc-secret
        namespace: example-sno
      data:
        password: <base64_password>
        username: <base64_username>
      type: Opaque
      ---
      apiVersion: v1
      kind: Secret
      metadata:
        name: pull-secret
        namespace: example-sno
      data:
        .dockerconfigjson: <pull_secret>
      type: kubernetes.io/dockerconfigjson
      ```

      where:

      <dl>
      <dt><code>namespace</code></dt>
      <dd>Must match the namespace configured in the related <code>ClusterInstance</code> CR.</dd>
      <dt><code>password</code>, <code>username</code></dt>
      <dd>Base64-encoded values for <code>password</code> and <code>username</code>.</dd>
      <dt><code>.dockerconfigjson</code></dt>
      <dd>Base64-encoded pull secret.</dd>
      </dl>
2. Add the relative path to `example-sno-secret.yaml` to the `kustomization.yaml` file that you use to install the cluster.

## Configuring Discovery ISO kernel arguments for manual installations using GitOps ZTP {#setting-managed-bare-metal-host-kernel-arguments_ztp-manual-install}

The GitOps Zero Touch Provisioning (ZTP) workflow uses the Discovery ISO as part of the OpenShift Container Platform installation process on managed bare-metal hosts. You can edit the `InfraEnv` resource to specify kernel arguments for the Discovery ISO. This is useful for cluster installations with specific environmental requirements. For example, configure the `rd.net.timeout.carrier` kernel argument for the Discovery ISO to facilitate static networking for the cluster or to receive a DHCP address before downloading the root file system during installation. In OpenShift Container Platform 4.22, you can only add kernel arguments. You can not replace or delete kernel arguments.

**Prerequisites**

- You have installed the OpenShift CLI (oc).
- You have logged in to the hub cluster as a user with cluster-admin privileges.
- You have applied a `ClusterInstance` CR to the hub cluster.

**Procedure**

1. Edit the `spec.kernelArguments` specification in the `InfraEnv` CR to configure kernel arguments:
   ```yaml
   apiVersion: agent-install.openshift.io/v1beta1
   kind: InfraEnv
   metadata:
     name: <cluster_name>
     namespace: <cluster_name>
   spec:
     kernelArguments:
       - operation: append
         value: audit=0
       - operation: append
         value: trace=1
     clusterRef:
       name: <cluster_name>
       namespace: <cluster_name>
     pullSecretRef:
       name: pull-secret
   ```

   where:

   <dl>
   <dt><code>operation</code></dt>
   <dd>Specify the <code>append</code> operation to add a kernel argument.</dd>
   <dt><code>value</code></dt>
   <dd>Specify the kernel argument you want to configure. This example configures the <code>audit</code> kernel argument and the <code>trace</code> kernel argument.</dd>
   </dl>

   :::note

   The `ClusterInstance` CR generates the `InfraEnv` resource as part of the day-0 installation CRs.

   :::

**Verification**

To verify that the kernel arguments are applied, after the Discovery image verifies that OpenShift Container Platform is ready for installation, you can SSH to the target host before the installation process begins. At that point, you can view the kernel arguments for the Discovery ISO in the `/proc/cmdline` file.

1. Begin an SSH session with the target host:
   ```terminal
   $ ssh -i /path/to/privatekey core@<host_name>
   ```
2. View the system’s kernel arguments by using the following command:
   ```terminal
   $ cat /proc/cmdline
   ```

## Installing a single managed cluster {#ztp-manually-install-a-single-managed-cluster_ztp-manual-install}

You can manually deploy a single managed cluster using the assisted service and Red Hat Advanced Cluster Management (RHACM).

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).
- You have logged in to the hub cluster as a user with `cluster-admin` privileges.
- You have extracted the reference and example CRs from the `ztp-site-generate` container and you configured the `ClusterInstance` CR.
- You have created the baseboard management controller (BMC) `Secret` and the image pull-secret `Secret` custom resources (CRs). See "Creating the managed bare-metal host secrets" for details.
- Your target bare-metal host meets the networking and hardware requirements for managed clusters.

**Procedure**

1. Create a `ClusterImageSet` for each specific cluster version to be deployed, for example `clusterImageSet-4.22.yaml`. A `ClusterImageSet` has the following format:
   ```yaml
   apiVersion: hive.openshift.io/v1
   kind: ClusterImageSet
   metadata:
     name: openshift-4.22.0
   spec:
      releaseImage: quay.io/openshift-release-dev/ocp-release:4.22.0-x86_64
   ```

   where:

   <dl>
   <dt><code>name</code></dt>
   <dd>The descriptive version that you want to deploy.</dd>
   <dt><code>releaseImage</code></dt>
   <dd>Specifies the <code>releaseImage</code> to deploy and determines the operating system image version. The discovery ISO is based on the image version as set by <code>releaseImage</code>, or the latest version if the exact version is unavailable.</dd>
   </dl>
2. Apply the `clusterImageSet` CR:
   ```terminal
   $ oc apply -f clusterImageSet-4.22.yaml
   ```
3. Create the `Namespace` CR in the `cluster-namespace.yaml` file:
   ```yaml
   apiVersion: v1
   kind: Namespace
   metadata:
        name: <cluster_name>
        labels:
           name: <cluster_name>
   ```

   where:

   <dl>
   <dt><code>name</code></dt>
   <dd>The name of the managed cluster to provision.</dd>
   </dl>
4. Apply the `Namespace` CR by running the following command:
   ```terminal
   $ oc apply -f cluster-namespace.yaml
   ```
5. Apply the `ClusterInstance` CR that you configured to the hub cluster by running the following command:
   ```terminal
   $ oc apply -f clusterinstance.yaml
   ```

   The SiteConfig Operator processes the `ClusterInstance` CR and automatically generates the required installation CRs, including `BareMetalHost`, `AgentClusterInstall`, `ClusterDeployment`, `InfraEnv`, and `NMStateConfig`. The assisted service then begins the cluster installation.

**Additional resources**

- [BMC addressing](/docs/installing/installing_bare_metal/ipi/ipi-install-installation-workflow#bmc-addressing_ipi-install-installation-workflow)
- [About root device hints](/docs/installing/installing_with_agent_based_installer/preparing-to-install-with-agent-based-installer#root-device-hints_preparing-to-install-with-agent-based-installer)
- [Single-node OpenShift ClusterInstance CR installation reference](/docs/edge_computing/ztp-deploying-far-edge-sites#ztp-clusterinstance-config-reference_ztp-deploying-far-edge-sites)
- [Connectivity prerequisites for managed cluster networks](/docs/edge_computing/ztp-reference-cluster-configuration-for-vdu#ztp-managed-cluster-network-prereqs_sno-configure-for-vdu)
- [Deploying LVM Storage on single-node OpenShift clusters](/docs/storage/persistent_storage_local/persistent-storage-using-lvms#lvms-preface-sno-ran_logical-volume-manager-storage)
- [Configuring LVM Storage using PolicyGenerator CRs](/docs/edge_computing/policygenerator_for_ztp/ztp-advanced-policygenerator-config#ztp-provisioning-lvm-storage_ztp-advanced-policy-config)
- [Configuring managed cluster policies by using PolicyGenerator resources](/docs/edge_computing/policygenerator_for_ztp/ztp-configuring-managed-clusters-policygenerator#ztp-configuring-managed-clusters-policygenerator)

## Late binding for bare-metal host pools in GitOps ZTP deployments {#ztp-late-binding-bare-metal-host-pools_ztp-manual-install}

In the standard GitOps ZTP flow, the `InfraEnv` custom resource (CR) references a specific `ClusterDeployment` CR, and all discovered hosts are automatically associated with that cluster. Late binding separates host discovery from cluster assignment, so you can manage a pool of bare-metal hosts independently from cluster lifecycle.

With late binding, you create an `InfraEnv` CR without a `ClusterDeployment` reference, so that all hosts booted from the discovery ISO remain unbound and available for assignment to any cluster. You then use the `bmac.agent-install.openshift.io/cluster-reference` annotation on `BareMetalHost` resources to declaratively bind individual hosts to specific `ClusterDeployment` CRs. This annotation-based binding is compatible with GitOps workflows such as ArgoCD.

Late binding in the GitOps ZTP workflow is designed for environments where hardware provisioning and cluster creation happen independently. Common scenarios include:

- An infrastructure team boots a set of bare-metal servers from a single discovery ISO, and a platform team later assigns subsets of those hosts to different clusters as demand arises.
- Hardware provisioning and cluster creation are handled by different teams at different times.
- Spare bare metal capacity must be allocated to clusters as workload requirements change.

If you are deploying a single cluster where all discovered hosts belong to that cluster, use the standard GitOps ZTP flow with a `ClusterDeployment` reference in the `InfraEnv` CR.

**Additional resources**

- [Bind bare-metal hosts to clusters using the cluster-reference annotation in GitOps ZTP deployments](/docs/edge_computing/ztp-manual-install#ztp-binding-bmh-to-cluster-using-annotation_ztp-manual-install)
- [`BareMetalHost` cluster-reference annotation reference](/docs/edge_computing/ztp-manual-install#ztp-bmh-cluster-reference-annotation-ref_ztp-manual-install)
- [Binding and unbinding hosts in the RHACM documentation](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.17/html/clusters/cluster_mce_overview#bind-unbind-hosts)

## Bind bare-metal hosts to clusters using the cluster-reference annotation in GitOps ZTP deployments {#ztp-binding-bmh-to-cluster-using-annotation_ztp-manual-install}

In GitOps ZTP deployments, you can declaratively bind individual `BareMetalHost` resources from a shared `InfraEnv` pool to specific `ClusterDeployment` CRs by using the `bmac.agent-install.openshift.io/cluster-reference` annotation. This approach is compatible with GitOps workflows because the binding is managed through annotations on the `BareMetalHost` resource rather than by patching `Agent` CRs directly.

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).
- You have logged in to the hub cluster as a user with `cluster-admin` privileges.
- You have created an `InfraEnv` CR without a `clusterRef` field. If you use the SiteConfig Operator, verify that the generated `InfraEnv` does not include a `clusterRef` field.
- You have `BareMetalHost` resources that reference the shared `InfraEnv`. These resources are either generated automatically by the SiteConfig Operator from a `ClusterInstance` CR, or created manually with the `infraenvs.agent-install.openshift.io` label set to the name of the `InfraEnv` CR.
- You have booted the hosts from the discovery ISO and `Agent` CRs have been created automatically for the discovered hosts.
- You have created the target `ClusterDeployment` and `AgentClusterInstall` CRs for the cluster that you want to bind hosts to.

**Procedure**

1. Verify that the `InfraEnv` CR does not have a `clusterRef` field set:
   ```terminal
   $ oc get infraenv <infraenv_name> -n <namespace> -o jsonpath='{.spec.clusterRef}'
   ```

   If the command returns an empty result, the `InfraEnv` is configured for late binding. If a `clusterRef` value is returned, the `BareMetalHost` cluster-reference annotation is ignored and you must use the standard binding flow.
2. Verify that the `Agent` CRs exist and are not bound to a cluster:
   ```terminal
   $ oc get agents -n <namespace>
   ```

   ```terminal title="Example output"
   NAME                                   CLUSTER   APPROVED   ROLE          STAGE
   aaaaaaaa-1111-2222-3333-bbbbbbbbbbbb             false      auto-assign
   cccccccc-4444-5555-6666-dddddddddddd             false      auto-assign
   ```

   Agents that are not bound to a cluster have an empty `CLUSTER` column.
3. Add the `bmac.agent-install.openshift.io/cluster-reference` annotation to the `BareMetalHost` resource to bind it to the target `ClusterDeployment` CR:
   ```terminal
   $ oc annotate bmh <bmh_name> -n <namespace> \
       bmac.agent-install.openshift.io/cluster-reference=<cluster_namespace>/<cluster_name>
   ```

   Replace `<cluster_namespace>/<cluster_name>` with the namespace and name of the target `ClusterDeployment` CR.

   Alternatively, you can set the annotation declaratively in the `BareMetalHost` YAML manifest:

   ```yaml
   apiVersion: metal3.io/v1alpha1
   kind: BareMetalHost
   metadata:
     name: worker-01
     namespace: my-infraenv-ns
     annotations:
       bmac.agent-install.openshift.io/cluster-reference: my-cluster-ns/my-cluster
     labels:
       infraenvs.agent-install.openshift.io: my-infraenv
   spec:
     online: true
     bootMACAddress: 00:00:5E:00:53:01
     bmc:
       address: redfish-virtualmedia://192.0.2.10/redfish/v1/Systems/1
       credentialsName: worker-01-bmc-secret
   ```

   The `bmac.agent-install.openshift.io/cluster-reference` annotation value uses the format `<namespace>/<name>`, referencing the target `ClusterDeployment`.
4. Verify that the `Agent` CR is bound to the correct `ClusterDeployment` CR by running the following command:
   ```terminal
   $ oc get agents -n <namespace>
   ```

   ```terminal title="Example output"
   NAME                                   CLUSTER      APPROVED   ROLE          STAGE
   aaaaaaaa-1111-2222-3333-bbbbbbbbbbbb   my-cluster   false      auto-assign
   ```

   The `CLUSTER` column shows the name of the target `ClusterDeployment`.
5. Verify the `Bound` condition on the `Agent` CR:
   ```terminal
   $ oc get agent <agent_name> -n <namespace> \
       -o jsonpath='{.status.conditions[?(@.type=="Bound")]}'
   ```

   A successfully bound agent shows `status: "True"` and `reason: "Bound"` in the output.

   :::note

   Agents discovered by using a `BareMetalHost` resource are automatically approved for installation when the `bootMACAddress` in the `BareMetalHost` matches a NIC in the inventory of the discovered host. You do not need to manually approve these agents.

   :::
6. Optional: To unbind a host from a cluster before installation starts, set the annotation value to an empty string:
   ```terminal
   $ oc annotate bmh <bmh_name> -n <namespace> \
       bmac.agent-install.openshift.io/cluster-reference="" --overwrite
   ```

   The bare metal agent controller (BMAC) clears the `spec.clusterDeploymentName` field on the corresponding `Agent` CR, and the host returns to the unbound pool.

   :::warning

   You cannot unbind a host after cluster installation has started. If you try to unbind a host that is already installed or in an `error` or `canceled` state, the `Agent` CR `Bound` condition is set to `False` with the reason `UnbindingPendingUserAction`. For hosts discovered by using a `BareMetalHost` resource, the assisted controllers automatically use the `BareMetalHost` to boot the host back into the discovery ISO to complete the unbinding process. For hosts discovered without a `BareMetalHost` resource, you must manually reboot the host with the discovery ISO.

   :::

**Verification**

- Verify that all agents are approved and bound to the correct cluster:
  ```terminal
  $ oc get agents -n <namespace> -o custom-columns=NAME:.metadata.name,CLUSTER:.spec.clusterDeploymentName.name,APPROVED:.spec.approved
  ```

**Additional resources**

- [Late binding for bare-metal host pools in GitOps ZTP deployments](/docs/edge_computing/ztp-manual-install#ztp-late-binding-bare-metal-host-pools_ztp-manual-install)
- [`BareMetalHost` cluster-reference annotation reference](/docs/edge_computing/ztp-manual-install#ztp-bmh-cluster-reference-annotation-ref_ztp-manual-install)
- [Binding and unbinding hosts in the RHACM documentation](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.16/html/clusters/cluster_mce_overview#binding-and-unbinding-hosts)

## BareMetalHost cluster-reference annotation {#ztp-bmh-cluster-reference-annotation-ref_ztp-manual-install}

The `bmac.agent-install.openshift.io/cluster-reference` annotation on `BareMetalHost` resources controls the declarative binding of discovered hosts to `ClusterDeployment` CRs. You can use this annotation to bind, unbind, or leave hosts unchanged in a late binding workflow.

**Cluster-reference annotation states**

| Annotation state | Value | Effect on Agent CR |
| --- | --- | --- |
| Set with cluster reference | `<namespace>/<name>` | Sets `spec.clusterDeploymentName` to the referenced `ClusterDeployment` CR. The `Agent` is bound to the specified cluster. |
| Set with empty string | `""` | Clears `spec.clusterDeploymentName`. The `Agent` is unbound from its current cluster and returns to the unbound pool. |
| Not set | N/A | No change to the `Agent` CR cluster reference. The host remains in its current state. |

**Constraints and precedence rules**

| Constraint | Description |
| --- | --- |
| `InfraEnv` `clusterRef` takes precedence | If the `InfraEnv` CR has a `clusterRef` field set, the `BareMetalHost` cluster-reference annotation is ignored. The `InfraEnv` CR must not have a `clusterRef` for late binding to work. |
| Unbinding is blocked after installation starts | You can only unbind a host before cluster installation begins. After the installation starts, setting the annotation to an empty string results in an `UnbindingPendingUserAction` state on the `Agent` CR. |
| `ClusterDeployment` CR must exist | The `ClusterDeployment` CR referenced by the annotation must exist in the specified namespace. If the `ClusterDeployment` CR does not exist, the binding fails. |
| One `InfraEnv` CR per host | Each `BareMetalHost` CR is associated with a single `InfraEnv` CR through the `infraenvs.agent-install.openshift.io` label. |

**Related BareMetalHost bare metal agent controller (BMAC) annotations**

| Annotation | Description |
| --- | --- |
| `bmac.agent-install.openshift.io/cluster-reference` | Binds the host to a specific `ClusterDeployment` CR. Value format: `<namespace>/<name>`. |
| `bmac.agent-install.openshift.io/hostname` | Sets the hostname for the agent during deployment. |
| `bmac.agent-install.openshift.io/role` | Sets the role of the host, such as `master` or `worker`. |
| `bmac.agent-install.openshift.io/installer-args` | Passes additional arguments to the OpenShift Container Platform installer. |
| `bmac.agent-install.openshift.io/ignition-config-overrides` | Provides ignition configuration overrides for the host. |

## Monitoring the managed cluster installation status {#ztp-checking-the-managed-cluster-status_ztp-manual-install}

Ensure that cluster provisioning was successful by checking the cluster status.

**Prerequisites**

- All of the custom resources have been configured and provisioned, and the `Agent` custom resource is created on the hub for the managed cluster.

**Procedure**

1. Check the status of the managed cluster:
   ```terminal
   $ oc get managedcluster
   ```

   `True` indicates the managed cluster is ready.
2. Check the agent status:
   ```terminal
   $ oc get agent -n <cluster_name>
   ```
3. Use the `describe` command to provide an in-depth description of the agent’s condition. Statuses to be aware of include `BackendError`, `InputError`, `ValidationsFailing`, `InstallationFailed`, and `AgentIsConnected`. These statuses are relevant to the `Agent` and `AgentClusterInstall` custom resources.
   ```terminal
   $ oc describe agent -n <cluster_name>
   ```
4. Check the cluster provisioning status:
   ```terminal
   $ oc get agentclusterinstall -n <cluster_name>
   ```
5. Use the `describe` command to provide an in-depth description of the cluster provisioning status:
   ```terminal
   $ oc describe agentclusterinstall -n <cluster_name>
   ```
6. Check the status of the managed cluster’s add-on services:
   ```terminal
   $ oc get managedclusteraddon -n <cluster_name>
   ```
7. Retrieve the authentication information of the `kubeconfig` file for the managed cluster:
   ```terminal
   $ oc get secret -n <cluster_name> <cluster_name>-admin-kubeconfig -o jsonpath={.data.kubeconfig} | base64 -d > <directory>/<cluster_name>-kubeconfig
   ```

## Troubleshooting the managed cluster {#ztp-troubleshooting-the-managed-cluster_ztp-manual-install}

Use this procedure to diagnose any installation issues that might occur with the managed cluster.

**Procedure**

1. Check the status of the managed cluster:
   ```terminal
   $ oc get managedcluster
   ```

   ```terminal title="Example output"
   NAME            HUB ACCEPTED   MANAGED CLUSTER URLS   JOINED   AVAILABLE   AGE
   SNO-cluster     true                                   True     True      2d19h
   ```

   If the status in the `AVAILABLE` column is `True`, the managed cluster is being managed by the hub.

   If the status in the `AVAILABLE` column is `Unknown`, the managed cluster is not being managed by the hub. Use the following steps to continue checking to get more information.
2. Check the `AgentClusterInstall` install status:
   ```terminal
   $ oc get clusterdeployment -n <cluster_name>
   ```

   ```terminal title="Example output"
   NAME        PLATFORM            REGION   CLUSTERTYPE   INSTALLED    INFRAID    VERSION  POWERSTATE AGE
   Sno0026    agent-baremetal                               false                          Initialized
   2d14h
   ```

   If the status in the `INSTALLED` column is `false`, the installation was unsuccessful.
3. If the installation failed, enter the following command to review the status of the `AgentClusterInstall` resource:
   ```terminal
   $ oc describe agentclusterinstall -n <cluster_name> <cluster_name>
   ```
4. Resolve the errors and reset the cluster:
   1. Remove the cluster’s managed cluster resource:
      ```terminal
      $ oc delete managedcluster <cluster_name>
      ```
   2. Remove the cluster’s namespace:
      ```terminal
      $ oc delete namespace <cluster_name>
      ```

      This deletes all of the namespace-scoped custom resources created for this cluster. You must wait for the `ManagedCluster` CR deletion to complete before proceeding.
   3. Recreate the custom resources for the managed cluster.

## RHACM generated cluster installation CRs {#ztp-installation-crs_ztp-manual-install}

Red Hat Advanced Cluster Management (RHACM) supports deploying OpenShift Container Platform on single-node clusters, three-node clusters, and standard clusters with a specific set of installation custom resources (CRs) that you generate by using `ClusterInstance` CRs for each cluster.

:::note

Every managed cluster has its own namespace, and all of the installation CRs except for `ManagedCluster` and `ClusterImageSet` are under that namespace. `ManagedCluster` and `ClusterImageSet` are cluster-scoped, not namespace-scoped. The namespace and the CR names match the cluster name.

:::

The following table lists the installation CRs that are automatically applied by the RHACM assisted service when it installs clusters using the `ClusterInstance` CRs that you configure.

**Cluster installation CRs generated by RHACM**

<table>
<thead>
<tr>
  <th>CR</th>
  <th>Description</th>
  <th>Usage</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>BareMetalHost</code></td>
  <td>Contains the connection information for the Baseboard Management Controller (BMC) of the target bare-metal host. In late binding scenarios where the <code>InfraEnv</code> CR is not bound to a <code>ClusterDeployment</code> CR, the optional <code>bmac.agent-install.openshift.io/cluster-reference</code> annotation on the <code>BareMetalHost</code> CR declaratively binds the host to a specific <code>ClusterDeployment</code> CR.</td>
  <td>Provides access to the BMC to load and start the discovery image on the target server by using the Redfish protocol.</td>
</tr>
<tr>
  <td><code>InfraEnv</code></td>
  <td>Contains information for installing OpenShift Container Platform on the target bare-metal host.</td>
  <td>Used with <code>ClusterDeployment</code> to generate the discovery ISO for the managed cluster. Can also be created without a <code>ClusterDeployment</code> reference to enable late binding, where hosts are bound to clusters individually by using the <code>BareMetalHost</code> cluster-reference annotation.</td>
</tr>
<tr>
  <td><code>AgentClusterInstall</code></td>
  <td>Specifies details of the managed cluster configuration such as networking and the number of control plane nodes. Displays the cluster <code>kubeconfig</code> and credentials when the installation is complete.</td>
  <td>Specifies the managed cluster configuration information and provides status during the installation of the cluster.</td>
</tr>
<tr>
  <td><code>ClusterDeployment</code></td>
  <td>References the <code>AgentClusterInstall</code> CR to use.</td>
  <td>Used with <code>InfraEnv</code> to generate the discovery ISO for the managed cluster.</td>
</tr>
<tr>
  <td><code>NMStateConfig</code></td>
  <td>Provides network configuration information such as <code>MAC</code> address to <code>IP</code> mapping, DNS server, default route, and other network settings.</td>
  <td>Sets up a static IP address for the managed cluster's Kube API server.</td>
</tr>
<tr>
  <td><code>Agent</code></td>
  <td>Contains hardware information about the target bare-metal host.</td>
  <td>Created automatically on the hub when the target machine's discovery image boots.</td>
</tr>
<tr>
  <td><code>ManagedCluster</code></td>
  <td>When a cluster is managed by the hub, it must be imported and known. This Kubernetes object provides that interface.</td>
  <td>The hub uses this resource to manage and show the status of managed clusters.</td>
</tr>
<tr>
  <td><code>KlusterletAddonConfig</code></td>
  <td>Contains the list of services provided by the hub to be deployed to the <code>ManagedCluster</code> resource.</td>
  <td>Tells the hub which add-on services to deploy to the <code>ManagedCluster</code> resource.</td>
</tr>
<tr>
  <td><code>Namespace</code></td>
  <td>Logical space for <code>ManagedCluster</code> resources existing on the hub. Unique per site.</td>
  <td>Propagates resources to the <code>ManagedCluster</code>.</td>
</tr>
<tr>
  <td><code>Secret</code></td>
  <td>Two CRs are created: <code>BMC Secret</code> and <code>Image Pull Secret</code>.</td>
  <td><ul><li><code>BMC Secret</code> authenticates into the target bare-metal host using its username and password.</li><li><code>Image Pull Secret</code> contains authentication information for the OpenShift Container Platform image installed on the target bare-metal host.</li></ul></td>
</tr>
<tr>
  <td><code>ClusterImageSet</code></td>
  <td>Contains OpenShift Container Platform image information such as the repository and image name.</td>
  <td>Passed into resources to provide OpenShift Container Platform images.</td>
</tr>
</tbody>
</table>
