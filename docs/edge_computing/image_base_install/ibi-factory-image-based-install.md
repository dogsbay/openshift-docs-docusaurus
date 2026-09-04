---
title: Preinstalling single-node OpenShift using an image-based installation
sidebar_position: 3
---

# Preinstalling single-node OpenShift using an image-based installation {#ibi-factory-image-based-install}

<a id="ibi-factory-image-based-install"></a>

Use the `openshift-install` program to create a live installation ISO for preinstalling single-node OpenShift on bare-metal hosts. For more information about downloading the installation program, see "Installation process" in the "Additional resources" section.

The installation program takes a seed image URL and other inputs, such as the release version of the seed image and the disk to use for the installation process, and creates a live installation ISO. You can then start the host using the live installation ISO to begin preinstallation. When preinstallation is complete, the host is ready to ship to a remote site for the final site-specific configuration and deployment.

The following are the high-level steps to preinstall a single-node OpenShift cluster using an image-based installation:

- Generate a seed image.
- Create a live installation ISO using the `openshift-install` installation program.
- Boot the host using the live installation ISO to preinstall the host.

**Additional resources**

- [Installation process](/docs/installing/overview#installation-process_ocp-installation-overview)

## Creating a live installation ISO for a single-node OpenShift image-based installation {#ibi-create-iso-for-bmh_ibi-factory-image-based-install}

You can embed your single-node OpenShift seed image URL, and other installation artifacts, in a live installation ISO by using the `openshift-install` program.

:::note

For more information about the specification for the `image-based-installation-config.yaml` manifest, see the section "Reference specifications for the `image-based-installation-config.yaml` manifest".

:::

**Prerequisites**

- You generated a seed image from a single-node OpenShift seed cluster.
- You downloaded the `openshift-install` program. The version of the `openshift-install` program must match the OpenShift Container Platform version in your seed image.
- The target host has network access to the seed image URL and all other installation artifacts.
- If you require static networking, you must install the `nmstatectl` library on the host that creates the live installation ISO.

**Procedure**

1. Create a live installation ISO and embed your single-node OpenShift seed image URL and other installation artifacts:
   1. Create a working directory by running the following:
      ```terminal
      $ mkdir <working_directory>
      ```

      where `<working_directory>` is the name of your working directory, for example `ibi-iso-workdir`.
   2. Optional. Create an installation configuration template to use as a reference when configuring the `ImageBasedInstallationConfig` resource:
      ```terminal
      $ openshift-install image-based create image-config-template --dir <working_directory>
      ```

      where `<working_directory>` is the name of your working directory, for example `ibi-iso-workdir`. If you do not specify a working directory, the command uses the current directory.

      Example output:

      ```terminal
      INFO Image-Config-Template created in: ibi-iso-workdir
      ```

      The command creates the `image-based-installation-config.yaml` installation configuration template in your target directory:

      ```yaml
      #
      # Note: This is a sample ImageBasedInstallationConfig file showing
      # which fields are available to aid you in creating your
      # own image-based-installation-config.yaml file.
      #
      apiVersion: v1beta1
      kind: ImageBasedInstallationConfig
      metadata:
        name: example-image-based-installation-config
      # The following fields are required
      seedImage: quay.io/openshift-kni/seed-image:4.22.0
      seedVersion: 4.22.0
      installationDisk: /dev/vda
      pullSecret: '<your_pull_secret>'
      # networkConfig is optional and contains the network configuration for the host in NMState format.
      # See https://nmstate.io/examples.html for examples.
      # networkConfig:
      #   interfaces:
      #     - name: eth0
      #       type: ethernet
      #       state: up
      #       mac-address: 00:00:00:00:00:00
      #       ipv4:
      #         enabled: true
      #         address:
      #           - ip: 192.168.122.2
      #             prefix-length: 23
      #         dhcp: false
      ```
   3. Edit your installation configuration file:
      Example `image-based-installation-config.yaml` file:

      ```yaml
      apiVersion: v1beta1
      kind: ImageBasedInstallationConfig
      metadata:
        name: example-image-based-installation-config
      seedImage: quay.io/repo-id/seed:latest
      seedVersion: "4.22.0"
      extraPartitionStart: "-240G"
      installationDisk: /dev/disk/by-id/wwn-0x62c...
      sshKey: 'ssh-ed25519 AAAA...'
      pullSecret: '{"auths": ...}'
      networkConfig:
          interfaces:
            - name: ens1f0
              type: ethernet
              state: up
              ipv4:
                enabled: true
                dhcp: false
                auto-dns: false
                address:
                  - ip: 192.168.200.25
                    prefix-length: 24
              ipv6:
                enabled: false
          dns-resolver:
            config:
              server:
                - 192.168.15.47
                - 192.168.15.48
          routes:
            config:
            - destination: 0.0.0.0/0
              metric: 150
              next-hop-address: 192.168.200.254
              next-hop-interface: ens1f0
      ```
   4. Create the live installation ISO by running the following command:
      ```terminal
      $ openshift-install image-based create image --dir ibi-iso-workdir
      ```

      Example output:

      ```terminal
      INFO Consuming Image-based Installation ISO Config from target directory
      INFO Creating Image-based Installation ISO with embedded ignition
      ```

**Verification**

- View the output in the working directory:
  ```text
  ibi-iso-workdir/
    └── rhcos-ibi.iso
  ```

**Additional resources**

- [Reference specifications for the `image-based-installation-config.yaml` manifest](/docs/edge_computing/image_base_install/ibi_deploying_sno_clusters/ibi-edge-image-based-install-standalone#ibi-installer-configuration-config_ibi-edge-image-based-install)

### Configuring additional partitions on the target host {#ibi-extra-partition-ibi-install-iso_ibi-factory-image-based-install}

The installation ISO creates a partition for the `/var/lib/containers` directory as part of the image-based installation process.

You can create additional partitions by using the `coreosInstallerArgs` specification. For example, in hard disks with adequate storage, you might need an additional partition for storage options, such as Logical Volume Manager (LVM) Storage.

:::note

The `/var/lib/containers` partition requires at least 500 GB to ensure adequate disk space for precached images. You must create additional partitions with a starting position larger than the partition for `/var/lib/containers`.

:::

**Procedure**

1. Edit the `image-based-installation-config.yaml` file to configure additional partitions:
   Example `image-based-installation-config.yaml` file:

   ```yaml
   apiVersion: v1beta1
   kind: ImageBasedInstallationConfig
   metadata:
     name: example-extra-partition
   seedImage: quay.io/repo-id/seed:latest
   seedVersion: "4.22.0"
   installationDisk: /dev/sda
   pullSecret: '{"auths": ...}'
   # ...
   skipDiskCleanup: <skip_disk_cleanup>
   coreosInstallerArgs:
      - "--save-partindex"
      - "<partition_index>"
   ignitionConfigOverride: |
     {
       "ignition": {
         "version": "3.2.0"
       },
       "storage": {
         "disks": [
           {
             "device": "<installation_disk>",
             "partitions": [
               {
                 "label": "<partition_label>",
                 "number": <partition_number>,
                 "sizeMiB": <partition_size>,
                 "startMiB": <starting_position>
               }
             ]
           }
         ]
       }
     }
   ```

   where:

   <dl>
   <dt><code>&lt;skip_disk_cleanup&gt;</code></dt>
   <dd>Specifies whether to skip disk formatting during the installation process. Set to <code>true</code> to skip.</dd>
   <dt><code>"--save-partindex"</code></dt>
   <dd>Specifies the argument to preserve a partition.</dd>
   <dt><code>&lt;partition_index&gt;</code></dt>
   <dd>Specifies the additional partition to preserve. The live installation ISO requires five partitions. Set a number greater than five, for example <code>6</code>.</dd>
   <dt><code>&lt;installation_disk&gt;</code></dt>
   <dd>Specifies the installation disk on the target host, for example <code>/dev/sda</code>.</dd>
   <dt><code>&lt;partition_label&gt;</code></dt>
   <dd>Specifies the label for the partition, for example <code>storage</code>.</dd>
   <dt><code>&lt;partition_number&gt;</code></dt>
   <dd>Specifies the number for the partition, for example <code>6</code>.</dd>
   <dt><code>&lt;partition_size&gt;</code></dt>
   <dd>Specifies the size of partition in MiB, for example <code>380000</code>.</dd>
   <dt><code>&lt;starting_position&gt;</code></dt>
   <dd>Specifies the starting position on the disk in MiB for the additional partition. You must specify a starting point larger than the partition for <code>/var/lib/containers</code>, for example <code>500000</code>.</dd>
   </dl>

**Verification**

- When you complete the preinstallation of the host with the live installation ISO, login to the target host and run the following command to view the partitions:
  ```terminal
  $ lsblk
  ```

  Example output:

  ```terminal
  sda    8:0    0  140G  0 disk
  ├─sda1 8:1    0    1M  0 part
  ├─sda2 8:2    0  127M  0 part
  ├─sda3 8:3    0  384M  0 part /var/mnt/boot
  ├─sda4 8:4    0  120G  0 part /var/mnt
  ├─sda5 8:5    0  500G  0 part /var/lib/containers
  └─sda6 8:6    0  380G  0 part
  ```

## Provisioning the live installation ISO to a host {#ibi-provision-install-iso-to-bmh_ibi-factory-image-based-install}

You can provision a live installation ISO to a bare-metal host to preinstall single-node OpenShift.

**Procedure**

- Using your preferred method, boot the target bare-metal host from the `rhcos-ibi.iso` live installation ISO to preinstall single-node OpenShift.

**Verification**

1. Login to the target host.
2. View the system logs by running the following command:
   ```terminal
   $ journalctl -b
   ```

   Example output:

   ```terminal
   Aug 13 17:01:44 10.46.26.129 install-rhcos-and-restore-seed.sh[2876]: time="2024-08-13T17:01:44Z" level=info msg="All the precaching threads have finished."
   Aug 13 17:01:44 10.46.26.129 install-rhcos-and-restore-seed.sh[2876]: time="2024-08-13T17:01:44Z" level=info msg="Total Images: 125"
   Aug 13 17:01:44 10.46.26.129 install-rhcos-and-restore-seed.sh[2876]: time="2024-08-13T17:01:44Z" level=info msg="Images Pulled Successfully: 125"
   Aug 13 17:01:44 10.46.26.129 install-rhcos-and-restore-seed.sh[2876]: time="2024-08-13T17:01:44Z" level=info msg="Images Failed to Pull: 0"
   Aug 13 17:01:44 10.46.26.129 install-rhcos-and-restore-seed.sh[2876]: time="2024-08-13T17:01:44Z" level=info msg="Completed executing pre-caching"
   Aug 13 17:01:44 10.46.26.129 install-rhcos-and-restore-seed.sh[2876]: time="2024-08-13T17:01:44Z" level=info msg="Pre-cached images successfully."
   Aug 13 17:01:44 10.46.26.129 install-rhcos-and-restore-seed.sh[2876]: time="2024-08-13 17:01:44" level=info msg="Skipping shutdown"
   Aug 13 17:01:44 10.46.26.129 install-rhcos-and-restore-seed.sh[2876]: time="2024-08-13 17:01:44" level=info msg="IBI preparation process finished successfully!"
   Aug 13 17:01:44 10.46.26.129 systemd[1]: var-lib-containers-storage-overlay.mount: Deactivated successfully.
   Aug 13 17:01:44 10.46.26.129 systemd[1]: Finished SNO Image-based Installation.
   Aug 13 17:01:44 10.46.26.129 systemd[1]: Reached target Multi-User System.
   Aug 13 17:01:44 10.46.26.129 systemd[1]: Reached target Graphical Interface.
   ```

## Reference specifications for the image-based-installation-config.yaml manifest {#ibi-installer-installation-config_ibi-factory-image-based-install}

The following content describes the specifications for the `image-based-installation-config.yaml` manifest.

The `openshift-install` program uses the `image-based-installation-config.yaml` manifest to create a live installation ISO for image-based installations of single-node OpenShift.

**Required specifications**

<table>
<tbody>
<tr>
  <td>Specification</td>
  <td>Type</td>
  <td>Description</td>
</tr>
<tr>
  <td><code>seedImage</code></td>
  <td><code>string</code></td>
  <td>Specifies the seed image to use in the ISO generation process.</td>
</tr>
<tr>
  <td><code>seedVersion</code></td>
  <td><code>string</code></td>
  <td>Specifies the OpenShift Container Platform release version of the seed image. The release version in the seed image must match the release version that you specify in the <code>seedVersion</code> field.</td>
</tr>
<tr>
  <td><code>installationDisk</code></td>
  <td><code>string</code></td>
  <td>Specifies the disk that will be used for the installation process.<br><br>Because the disk discovery order is not guaranteed, the kernel name of the disk can change across booting options for machines with multiple disks. For example, <code>/dev/sda</code> becomes <code>/dev/sdb</code> and vice versa. To avoid this issue, you must use a persistent disk attribute, such as the disk World Wide Name (WWN), for example: <code>/dev/disk/by-id/wwn-&lt;disk-id&gt;</code>.</td>
</tr>
<tr>
  <td><code>pullSecret</code></td>
  <td><code>string</code></td>
  <td>Specifies the pull secret to use during the precache process. The pull secret contains authentication credentials for pulling the release payload images from the container registry.<br><br>If the seed image requires a separate private registry authentication, add the authentication details to the pull secret.</td>
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
  <td><code>shutdown</code></td>
  <td><code>boolean</code></td>
  <td>Specifies if the host shuts down after the installation process completes. The default value is <code>false</code>.</td>
</tr>
<tr>
  <td><code>extraPartitionStart</code></td>
  <td><code>string</code></td>
  <td>Specifies the start of the extra partition used for <code>/var/lib/containers</code>. The default value is <code>-40G</code>, which means that the partition will be exactly 40GiB in size and uses the space 40GiB from the end of the disk. If you specify a positive value, the partition will start at that position of the disk and extend to the end of the disk.</td>
</tr>
<tr>
  <td><code>extraPartitionLabel</code></td>
  <td><code>string</code></td>
  <td>The label of the extra partition you use for <code>/var/lib/containers</code>. The default partition label is <code>var-lib-containers</code>.<br><br>[NOTE] ==== You must ensure that the partition label in the installation ISO matches the partition label set in the machine configuration for the seed image. If the partition labels are different, the partition mount fails during installation on the host. For more information, see "Configuring a shared container partition between ostree stateroots". ====</td>
</tr>
<tr>
  <td><code>extraPartitionNumber</code></td>
  <td><code>unsigned integer</code></td>
  <td>The number of the extra partition you use for <code>/var/lib/containers</code>. The default number is <code>5</code>.</td>
</tr>
<tr>
  <td><code>skipDiskCleanup</code></td>
  <td><code>boolean</code></td>
  <td>The installation process formats the disk on the host. Set this specification to 'true' to skip this step. The default is <code>false</code>.</td>
</tr>
<tr>
  <td><code>networkConfig</code></td>
  <td><code>string</code></td>
  <td>Specifies networking configurations for the host, for example: [source,yaml] ---- networkConfig: interfaces: - name: ens1f0 type: ethernet state: up ... ---- If you require static networking, you must install the <code>nmstatectl</code> library on the host that creates the live installation ISO. For further information about defining network configurations by using <code>nmstate</code>, see <a href="https://nmstate.io/">nmstate.io</a>. [IMPORTANT] ==== The name of the interface must match the actual NIC name as shown in the operating system. ====</td>
</tr>
<tr>
  <td><code>proxy</code></td>
  <td><code>string</code></td>
  <td>Specifies proxy settings to use during the installation ISO generation, for example: [source,yaml] ---- proxy: httpProxy: "http://proxy.example.com:8080" httpsProxy: "http://proxy.example.com:8080" noProxy: "no_proxy.example.com" ----</td>
</tr>
<tr>
  <td><code>imageDigestSources</code></td>
  <td><code>string</code></td>
  <td>Specifies the sources or repositories for the release-image content, for example: [source,yaml] ---- imageDigestSources: - mirrors: - "registry.example.com:5000/ocp4/openshift4" source: "quay.io/openshift-release-dev/ocp-release" ----</td>
</tr>
<tr>
  <td><code>additionalTrustBundle</code></td>
  <td><code>string</code></td>
  <td>Specifies the PEM-encoded X.509 certificate bundle. The installation program adds this to the <code>/etc/pki/ca-trust/source/anchors/</code> directory in the installation ISO. [source,yaml] ---- additionalTrustBundle:  -----BEGIN CERTIFICATE----- MTICLDCCAdKgAwfBAgIBAGAKBggqhkjOPQRDAjB9MQswCQYRVEQGE ... l2wOuDwKQa+upc4GftXE7C//4mKBNBC6Ty01gUaTIpo= -----END CERTIFICATE----- ----</td>
</tr>
<tr>
  <td><code>sshKey</code></td>
  <td><code>string</code></td>
  <td>Specifies the SSH key to authenticate access to the host.</td>
</tr>
<tr>
  <td><code>ignitionConfigOverride</code></td>
  <td><code>string</code></td>
  <td>Specifies a JSON string containing the user overrides for the Ignition config. The configuration merges with the Ignition config file generated by the installation program. This feature requires Ignition version is 3.2 or later.</td>
</tr>
<tr>
  <td><code>coreosInstallerArgs</code></td>
  <td><code>string</code></td>
  <td>Specifies custom arguments for the <code>coreos-install</code> command that you can use to configure kernel arguments and disk partitioning options.</td>
</tr>
</tbody>
</table>

**Additional resources**

- [Configuring a shared container partition between ostree stateroots](/docs/edge_computing/image_base_install/ibi-preparing-for-image-based-install#cnf-image-based-upgrade-shared-container-partition_ibi-preparing-image-based-install)
