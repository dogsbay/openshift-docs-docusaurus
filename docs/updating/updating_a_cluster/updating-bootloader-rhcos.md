---
title: Updating the boot loader on RHCOS nodes using bootupd
sidebar_position: 8
---

# Updating the boot loader on RHCOS nodes using bootupd {#updating-bootloader-rhcos}

<a id="updating-bootloader-rhcos"></a>

To update the boot loader on RHCOS nodes using `bootupd`, you must either run the `bootupctl update` command on RHCOS machines manually or provide a machine config with a `systemd` unit.

Unlike `grubby` or other boot loader tools, `bootupd` does not manage kernel space configuration such as passing kernel arguments. To configure kernel arguments, see [Adding kernel arguments to nodes](/docs/nodes/nodes/nodes-nodes-managing#nodes-nodes-kernel-arguments_nodes-nodes-managing).

:::note

You can use `bootupd` to update the boot loader to protect against the BootHole vulnerability.

:::

## Updating the boot loader manually {#updating-bootloader-manual_updating-bootloader-rhcos}

You can manually inspect the status of the system and update the boot loader by using the `bootupctl` command-line tool.

**Procedure**

1. Inspect the system status by running the following command:
   ```terminal
   # bootupctl status
   ```

   ```terminal title="Example output for x86_64"
   Component EFI
     Installed: grub2-efi-x64-1:2.04-31.el8_4.1.x86_64,shim-x64-15-8.el8_1.x86_64
     Update: At latest version
   ```

   ```terminal title="Example output for aarch64"
   Component EFI
     Installed: grub2-efi-aa64-1:2.02-99.el8_4.1.aarch64,shim-aa64-15.4-2.el8_1.aarch64
     Update: At latest version
   ```
2. OpenShift Container Platform clusters initially installed on version 4.4 and older require an explicit adoption phase.
   If the system status is `Adoptable`, perform the adoption by running the following command:

   ```terminal
   # bootupctl adopt-and-update
   ```

   ```terminal title="Example output"
   Updated: grub2-efi-x64-1:2.04-31.el8_4.1.x86_64,shim-x64-15-8.el8_1.x86_64
   ```
3. If an update is available, apply the update so that the changes take effect on the next reboot by running the following command:
   ```terminal
   # bootupctl update
   ```

   ```terminal title="Example output"
   Updated: grub2-efi-x64-1:2.04-31.el8_4.1.x86_64,shim-x64-15-8.el8_1.x86_64
   ```

## Updating the boot loader automatically by using a machine config {#updating-bootloader-auto_updating-bootloader-rhcos}

You can automatically update the boot loader with `bootupd` by creating a systemd service unit that will update the boot loader as needed on every boot. This unit will run the `bootupctl update` command during the boot process and will be installed on the nodes via a machine config.

:::note

This configuration is not enabled by default because unexpected interruptions of the update operation might lead to unbootable nodes. If you enable this configuration, make sure to avoid interrupting nodes during the boot process while the boot loader update is in progress. The boot loader update operation generally completes quickly thus the risk is low.

:::

**Procedure**

1. Create a Butane config file, `99-worker-bootupctl-update.bu`, including the contents of the `bootupctl-update.service` systemd unit.
   :::note

   The [Butane version](https://coreos.github.io/butane/specs/) you specify in the config file should match the OpenShift Container Platform version and always ends in `0`. For example, `4.22.0`. See "Creating machine configs with Butane" for information about Butane.

   :::

   ```yaml title="Example output"
   variant: openshift
   version: 4.22.0
   metadata:
     name: 99-worker-chrony
     labels:
       machineconfiguration.openshift.io/role: worker
   systemd:
     units:
     - name: bootupctl-update.service
       enabled: true
       contents: |
         [Unit]
         Description=Bootupd automatic update

         [Service]
         ExecStart=/usr/bin/bootupctl update
         RemainAfterExit=yes

         [Install]
         WantedBy=multi-user.target
   ```

   On control plane nodes, substitute `master` for `worker` in `metadata.name` and `metadata.labels.machineconfiguration.openshift.io/role`.
2. Generate a `MachineConfig` object file, `99-worker-bootupctl-update.yaml`, containing the configuration to be delivered to the nodes by running the following command:
   ```terminal
   $ butane 99-worker-bootupctl-update.bu -o 99-worker-bootupctl-update.yaml
   ```
3. Apply the configurations in one of two ways:
   - If the cluster is not running yet, after you generate manifest files, add the `MachineConfig` object file to the `<installation_directory>/openshift` directory, and then continue to create the cluster.
   - If the cluster is already running, apply the file by running the following command:
     ```terminal
     $ oc apply -f ./99-worker-bootupctl-update.yaml
     ```
