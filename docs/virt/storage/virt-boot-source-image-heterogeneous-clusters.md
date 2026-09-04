---
title: Heterogeneous cluster support
sidebar_position: 4
---

# Heterogeneous cluster support {#virt-boot-source-image-heterogeneous-clusters}

<a id="virt-boot-source-image-heterogeneous-clusters"></a>

A heterogeneous cluster is a cluster where nodes have differing architectures. Heterogeneous clusters promote optimal compute resource usage by mixing different types of hardware in one cluster.

With heterogeneous clusters, you can match workloads to hardware intended for the workload task instead of general purpose compute platforms. For example, you can combine GPU and general purpose compute resources and assign workloads to the appropriate hardware.

If you have a heterogeneous cluster but do not want to enable multiple architecture support, you can modify the workloads node placement in the `HyperConverged` custom resource (CR) to include only nodes with a specific architecture.

With boot source image support, you can deploy persistent VMs with specific architectures and define custom boot images that support heterogeneous clusters.

:::warning

If you do not enable boot source image support in a heterogeneous cluster, images might not match the node architecture. As a result, virtual machines might fail to start or might not run as expected. OpenShift Virtualization raises the `HCOMultiArchGoldenImagesDisabled` alert when this feature is not enabled.

:::

The same image can be used with nodes of different architectures if the boot image supports the required architectures. For example, a boot image that supports both ARM and AMD architectures can be used with both types of nodes.

Boot source image support for heterogeneous clusters is not enabled by default. You can enable heterogeneous cluster support by setting the feature gate in the `HyperConverged` CR.

## Default architecture behavior in heterogeneous clusters {#virt-about-architecture-defaults-heterogeneous-clusters_virt-boot-source-image-heterogeneous-clusters}

In a heterogeneous cluster, worker nodes run different CPU architectures, such as amd64 and arm64. A boot source image built for one architecture cannot start on a node with a different architecture.

To prevent architecture mismatches, OpenShift Virtualization creates a separate boot source for each supported architecture, each represented by a `DataSource` object.

:::warning

If you have a heterogeneous cluster and do not enable the `enableMultiArchBootImageImport` feature gate, VMs might fail to start on nodes with a different architecture than the boot source image.

:::

### How data sources for boot source images work in heterogeneous clusters {#data-sources-boot-source-images-heterogeneous-clusters_virt-boot-source-image-heterogeneous-clusters}

After you enable the `enableMultiArchBootImageImport` feature gate, the Scheduling, Scale, and Performance (SSP) Operator creates a new separate boot source for each supported architecture, represented by a `DataSource` object. For example, the `rhel9` boot source image gets the following data sources:

- `rhel9-amd64`
- `rhel9-arm64`

The original name, such as `rhel9`, defaults to the boot source that matches the control-plane architecture. For example, if the control-plane nodes use `amd64`, then `rhel9` resolves to `rhel9-amd64`.

Existing VM manifests and templates that use the original name, without an architecture suffix, continue to work without changes.

For common boot sources that are provided by OpenShift Virtualization, the SSP Operator determines the supported architectures automatically. For custom boot sources that you add through the `HyperConverged` CR, you must specify the supported architectures by using the `ssp.kubevirt.io/dict.architectures` annotation.

### Default architecture for standalone VMs {#default-architecture-standalone-vms_virt-boot-source-image-heterogeneous-clusters}

A VM that is not based on a boot source image, for example a VM from a container disk, HTTP source, upload, or clone, defaults to the control-plane architecture. The `spec.template.spec.architecture` field in the `VirtualMachine` manifest controls which architecture the VM uses.

### Default architecture for standalone data volumes {#default-architecture-standalone-data-volumes_virt-boot-source-image-heterogeneous-clusters}

A `DataVolume` created directly from a registry source does not have a default architecture. Without an explicit architecture, CDI pulls whichever image variant the registry returns. The `spec.source.registry.platform.architecture` field in the `DataVolume` manifest controls which architecture to pull.

## Enabling heterogeneous cluster support {#virt-enabling-heterogeneous-clusters_virt-boot-source-image-heterogeneous-clusters}

You can enable boot source image support for heterogeneous clusters by setting the `enableMultiArchBootImageImport` feature gate to `true` in the `HyperConverged` custom resource (CR).

**Prerequisites**

- You have access to the cluster as a user with `cluster-admin` permissions.
- You have installed the OpenShift CLI (`oc`).

**Procedure**

- Enable the `enableMultiArchBootImageImport` feature gate by running the following command:
  ```terminal
  $ oc patch hyperconvergeds.v1beta1.hco.kubevirt.io kubevirt-hyperconverged -n openshift-cnv \
    --type json -p '[{"op":"replace","path":"/spec/featureGates/enableMultiArchBootImageImport", "value": true}]'
  ```

**Verification**

- Verify that the feature gate is enabled by running the following command:
  ```terminal
  $ oc get hyperconvergeds.v1beta1.hco.kubevirt.io kubevirt-hyperconverged -n openshift-cnv \
    -o jsonpath='{.spec.featureGates[*].name}'
  ```

  The output must include `enableMultiArchBootImageImport`.

## Modifying a common boot source image in a heterogeneous cluster {#virt-mod-boot-source-image-heterogeneous-clusters_virt-boot-source-image-heterogeneous-clusters}

You can modify the source of a common boot source image in a heterogeneous cluster by specifying the supported architectures in the `ssp.kubevirt.io/dict.architectures` annotation in the `HyperConverged` custom resource (CR).

**Prerequisites**

- You have access to the cluster as a user with `cluster-admin` permissions.
- You have installed the OpenShift CLI (`oc`).
- You have enabled the `enableMultiArchBootImageImport` feature gate in the `HyperConverged` CR.

**Procedure**

1. Open the `HyperConverged` CR in your default editor by running the following command:
   ```terminal
   $ oc edit hyperconvergeds.v1beta1.hco.kubevirt.io kubevirt-hyperconverged -n openshift-cnv
   ```
2. Edit the `HyperConverged` CR to add the appropriate values for the `ssp.kubevirt.io/dict.architectures` annotation in the `dataImportCronTemplates` section. For example:
   ```yaml
   #...
   spec:
     dataImportCronTemplates:
     - metadata:
         name: kubevirt-hyperconverged
         annotations:
           ssp.kubevirt.io/dict.architectures: "<architecture_list>"
       spec:
         schedule: "0 */12 * * *"
         template:
           spec:
             source:
               registry:
                   url: docker://my-private-registry/my-own-version-of-centos:8
         managedDataSource: centos-stream8
   #...
   ```

   where:

   <dl>
   <dt><code>ssp.kubevirt.io/dict.architectures</code></dt>
   <dd>Specifies a comma-separated list of supported architectures for this image. For example, if the image supports <code>amd64</code> and <code>arm64</code> architectures, the value would be <code>"amd64,arm64"</code>.</dd>
   </dl>
3. Save and exit the editor to update the `HyperConverged` CR.

**Verification**

- Verify that architecture-suffixed data sources are created by running the following command:
  ```terminal
  $ oc get datasources -n openshift-virtualization-os-images
  ```

  Architecture-suffixed data sources, such as `centos-stream8-amd64` and `centos-stream8-arm64`, should appear in the output.

## Adding a custom boot source image in a heterogeneous cluster {#virt-add-custom-boot-source-image-heterogeneous-cluster_virt-boot-source-image-heterogeneous-clusters}

Add a custom boot source image in a heterogeneous cluster by editing the `HyperConverged` custom resource (CR).

**Prerequisites**

- You have access to the cluster as a user with `cluster-admin` permissions.
- You have installed the OpenShift CLI (`oc`).
- You have enabled the `enableMultiArchBootImageImport` feature gate in the `HyperConverged` CR.

**Procedure**

1. Open the `HyperConverged` CR in your default editor by running the following command:
   ```terminal
   $ oc edit hyperconvergeds.v1beta1.hco.kubevirt.io kubevirt-hyperconverged -n openshift-cnv
   ```
2. Edit the `HyperConverged` CR to add the custom boot source image. You must add the appropriate values for the `ssp.kubevirt.io/dict.architectures` annotation in the `dataImportCronTemplates` section. For example:
   ```yaml
   apiVersion: hco.kubevirt.io/v1beta1
   kind: HyperConverged
   metadata:
     name: kubevirt-hyperconverged
   spec:
     dataImportCronTemplates:
     - metadata:
         name: custom-image1
         annotations:
           ssp.kubevirt.io/dict.architectures: "<architecture_list>"
       spec:
         schedule: "0 */12 * * *"
         template:
           spec:
             source:
               registry:
                 url: docker://myprivateregistry/custom1
         managedDataSource: custom1
         retentionPolicy: "All"
   #...
   ```

   where:

   <dl>
   <dt><code>&lt;architecture_list&gt;</code></dt>
   <dd>Specifies a comma-separated list of supported architectures for this image. For example, if the image supports <code>amd64</code> and <code>arm64</code> architectures, the value would be <code>"amd64,arm64"</code>.</dd>
   </dl>

   :::warning

   Only include architectures that are present on your worker nodes and supported by your registry image. You do not need to list every architecture an image supports. OpenShift Virtualization does not validate the declared architectures.

   The annotation is optional but strongly recommended. Without it, only one boot source is created, and missing annotations trigger `HCOGoldenImageWithNoArchitectureAnnotation`.

   :::
3. Save and exit the editor to update the `HyperConverged` CR.

**Verification**

- Verify that architecture-suffixed data sources are created for the custom image by running the following command:
  ```terminal
  $ oc get datasources -n openshift-virtualization-os-images
  ```

  Architecture-suffixed data sources for the custom image should appear in the output.

## Creating a data volume from a registry source in a heterogeneous cluster {#virt-creating-dv-registry-heterogeneous-cluster_virt-boot-source-image-heterogeneous-clusters}

To pull the correct architecture-specific image in a heterogeneous cluster, specify the architecture in the `DataVolume` manifest. This step is only required for data volumes that you create outside the boot source image pipeline.

:::note

Boot source images managed through the `HyperConverged` custom resource (CR) select the correct architecture automatically. You do not need to specify the architecture for those images.

:::

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).

**Procedure**

1. Create a `DataVolume` manifest and save it as a YAML file:
   ```yaml
   apiVersion: cdi.kubevirt.io/v1beta1
   kind: DataVolume
   metadata:
     name: <datavolume_name>
   spec:
     source:
       registry:
         url: <image_url>
         platform:
           architecture: <architecture>
     storage:
       resources:
         requests:
           storage: <storage_size>
   ```

   where:

   <dl>
   <dt><code>&lt;datavolume_name&gt;</code></dt>
   <dd>Specifies the name of the data volume.</dd>
   <dt><code>&lt;image_url&gt;</code></dt>
   <dd>Specifies the URL of the container image, for example <code>docker://quay.io/containerdisks/centos-stream:9</code>.</dd>
   <dt><code>&lt;architecture&gt;</code></dt>
   <dd>Specifies the architecture of the image to pull, for example <code>amd64</code>, <code>arm64</code>, or <code>s390x</code>.</dd>
   <dt><code>&lt;storage_size&gt;</code></dt>
   <dd>Specifies the size of the storage requested, for example <code>10Gi</code>.</dd>
   </dl>
2. Create the data volume:
   ```terminal
   $ oc create -f <datavolume_manifest>.yaml
   ```

**Verification**

- Verify that the data volume was created and is importing by running the following command:
  ```terminal
  $ oc get dv <datavolume_name>
  ```

  The `PHASE` column should show `ImportScheduled`, `ImportInProgress`, or `Succeeded`.

## Architecture field for standalone virtual machines {#virt-architecture-field-standalone-vms_virt-boot-source-image-heterogeneous-clusters}

To run a standalone VM on a specific architecture in a heterogeneous cluster, set the `spec.template.spec.architecture` field in the `VirtualMachine` manifest. If you do not set this field, the VM defaults to the control-plane architecture.

This field applies to VMs created from container disks, HTTP sources, uploads, or clones. For VMs based on boot source images, the `DataSource` object resolves the architecture automatically.

```yaml title="Example VirtualMachine manifest with the architecture field"
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: my-vm
spec:
  template:
    spec:
      architecture: <architecture>
      domain:
        devices: {}
        memory:
          guest: 512Mi
        resources: {}
      volumes:
      - name: my-volume
        containerDisk:
          image: <container_disk_image>
# ...
```

where:

`<architecture>`
: Specifies the target architecture for the VM, for example `arm64`. If not specified, defaults to the control-plane architecture.

`<container_disk_image>`
: Specifies the container disk image to use for the VM.

## Modifying workloads node placement in a heterogeneous cluster {#virt-modify-workload-node-heterogeneous-cluster_virt-boot-source-image-heterogeneous-clusters}

If you have a heterogeneous cluster but do not want to enable multiple architecture support, you can modify the workloads node placement in the `HyperConverged` custom resource (CR) to include only nodes with a specific architecture.

**Prerequisites**

- You have access to the cluster as a user with `cluster-admin` permissions.
- You have installed the OpenShift CLI (`oc`).

**Procedure**

1. Open the `HyperConverged` CR in your default editor by running the following command:
   ```terminal
   $ oc edit hyperconvergeds.v1beta1.hco.kubevirt.io kubevirt-hyperconverged -n openshift-cnv
   ```
2. Edit the `HyperConverged` CR, to modify the workloads node placement to include only nodes with a specific architecture. For example:
   ```yaml
   apiVersion: hco.kubevirt.io/v1beta1
   kind: HyperConverged
   metadata:
     name: kubevirt-hyperconverged
   spec:
   #...
     workloads:
       nodePlacement:
         affinity:
           nodeAffinity:
             requiredDuringSchedulingIgnoredDuringExecution:
               nodeSelectorTerms:
                 - matchExpressions:
                     - key: kubernetes.io/arch
                       operator: In
                       values:
                         - <node_architecture>
   ```

   where:

   <dl>
   <dt><code>&lt;node_architecture&gt;</code></dt>
   <dd>Specifies the target architecture. For example, to limit placement to AMD nodes, use <code>amd64</code>.</dd>
   </dl>
3. Save and exit the editor to update the `HyperConverged` CR.

**Verification**

- Verify that the node affinity is applied by running the following command:
  ```terminal
  $ oc get hyperconvergeds.v1beta1.hco.kubevirt.io kubevirt-hyperconverged -n openshift-cnv \
    -o jsonpath='{.spec.deployment.nodePlacements.workload}'
  ```

  The output should show the node affinity configuration with the architecture you specified.
