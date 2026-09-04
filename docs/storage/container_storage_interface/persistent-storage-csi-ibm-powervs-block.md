---
title: IBM Power Virtual Server Block CSI Driver Operator
sidebar_position: 17
---

# IBM Power Virtual Server Block CSI Driver Operator {#persistent-storage-csi-ibm-powervs-block}

<a id="persistent-storage-csi-ibm-powervs-block"></a>

You can provision and manage IBM Power(R) Virtual Server Block storage in OpenShift Container Platform by using the Container Storage Interface (CSI) Driver Operator and driver, which provide dynamic volume provisioning.

## Introduction to IBM Power(R) Virtual Server Block CSI Driver Operator  {#persistent-storage-csi-ibm-powervs-block-intro_persistent-storage-csi-ibm-powervs-block}

The IBM Power(R) Virtual Server Block CSI Driver is installed through the IBM Power(R) Virtual Server Block CSI Driver Operator and the operator is based on `library-go`.

The OpenShift Container Platform `library-go` framework is a collection of functions that allows users to build OpenShift operators easily. Most of the functionality of a CSI Driver Operator is already available there. The IBM Power(R) Virtual Server Block CSI Driver Operator is installed by the Cluster Storage Operator. The Cluster Storage Operator installs the IBM Power(R) Virtual Server Block CSI Driver Operator if the platform type is Power Virtual Servers.

## IBM Power(R) Virtual Server Block CSI Driver Operator overview {#persistent-storage-csi-ibm-powervs-block-overview_persistent-storage-csi-ibm-powervs-block}

OpenShift Container Platform can provision persistent volumes (PVs) by using the Container Storage Interface (CSI) driver for IBM Power(R) Virtual Server Block Storage.

Familiarity with persistent storage and configuring CSI volumes is helpful when working with a CSI Operator and driver. For more information, see "Understanding persistent storage" and "Configuring CSI volumes".

To create CSI-provisioned PVs that mount to IBM Power(R) Virtual Server Block storage assets, OpenShift Container Platform installs the IBM Power(R) Virtual Server Block CSI Driver Operator and the IBM Power(R) Virtual Server Block CSI driver by default in the `openshift-cluster-csi-drivers` namespace.

- The *IBM Power(R) Virtual Server Block CSI Driver Operator* provides two storage classes named `ibm-powervs-tier1` (default), and `ibm-powervs-tier3` for different tiers that you can use to create persistent volume claims (PVCs). The IBM Power(R) Virtual Server Block CSI Driver Operator supports dynamic volume provisioning by allowing storage volumes to be created on-demand, eliminating the need for cluster administrators to pre-provision storage.
- With the *IBM Power(R) Virtual Server Block CSI driver* you can create and mount IBM Power(R) Virtual Server Block PVs.

**Additional resources**

- [Understanding persistent storage](/docs/storage/understanding-persistent-storage#understanding-persistent-storage)
- [Configuring CSI volumes](/docs/storage/container_storage_interface/persistent-storage-csi#persistent-storage-csi)

## About CSI {#csi-about_persistent-storage-csi-ibm-powervs-block}

The Container Storage Interface (CSI) enables storage vendors to deliver plugins through a standard interface without modifying Kubernetes core code, replacing traditional embedded storage drivers.

CSI Operators give OpenShift Container Platform users storage options, such as volume snapshots, that are not possible with in-tree volume plugins.
