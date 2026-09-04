---
title: About multi-queue functionality
sidebar_position: 15
---

# About multi-queue functionality {#virt-about-multi-queue}

<a id="virt-about-multi-queue"></a>

Use multi-queue functionality to scale network throughput and performance on virtual machines (VMs) with multiple vCPUs.

By default, the `queueCount` value, which is derived from the domain XML, is determined by the number of vCPUs allocated to a VM. Network performance does not scale as the number of vCPUs increases. Additionally, because `virtio-net` has only one transmit and receive queue, guests cannot send or receive packs in parallel.

:::note

Enabling `virtio-net` multi-queue does not offer significant improvements when the number of vNICs in a guest instance is proportional to the number of vCPUs.

:::

## Known limitations {#known-limitations_virt-about-multi-queue}

- Message signaled interrupt (MSI) vectors are still consumed if `virtio-net` multi-queue is enabled in the host but not enabled in the guest operating system by the administrator.
- Each `virtio-net` queue consumes 64 KiB of kernel memory for the `vhost` driver.

## Enabling multi-queue functionality {#virt-enabling-multi-queue_virt-about-multi-queue}

You can enable multi-queue functionality for interfaces configured with a VirtIO model.

**Procedure**

1. Set the `networkInterfaceMultiqueue` value to `true` in the `VirtualMachine` manifest file of your VM to enable multi-queue functionality:
   ```yaml
   apiVersion: kubevirt.io/v1
   kind: VM
   spec:
     domain:
       devices:
         networkInterfaceMultiqueue: true
   ```
2. Save the `VirtualMachine` manifest file to apply your changes.
