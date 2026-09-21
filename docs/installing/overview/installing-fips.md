---
title: Support for FIPS cryptography
sidebar_position: 4
---

# Support for FIPS cryptography {#installing-fips}

<a id="installing-fips"></a>

You can install an OpenShift Container Platform cluster in FIPS mode.

OpenShift Container Platform is designed for FIPS. When running Red Hat Enterprise Linux (RHEL) or Red Hat Enterprise Linux CoreOS (RHCOS) booted in FIPS mode, OpenShift Container Platform core components use the RHEL cryptographic libraries that have been submitted to NIST for FIPS 140-2/140-3 Validation on only the x86_64, ppc64le, and s390x architectures.

For more information about the NIST validation program, see "Cryptographic Module Validation Program" in the *Additional resources* section. For the latest NIST status for the individual versions of RHEL cryptographic libraries that have been submitted for validation, see "Compliance Activities and Government Standards" in the *Additional resources* section.

:::warning

To enable FIPS mode for your cluster, you must run the installation program from a RHEL 9 computer that is configured to operate in FIPS mode, and you must use a FIPS-capable version of the installation program. See the section titled *Obtaining a FIPS-capable installation program using *`oc adm extract`.

For more information about configuring FIPS mode on RHEL, see "Installing the system in FIPS mode" in the *Additional resources* section.

:::

For the Red Hat Enterprise Linux CoreOS (RHCOS) machines in your cluster, this change is applied when the machines are deployed based on the status of an option in the `install-config.yaml` file, which governs the cluster options that a user can change during cluster deployment. With Red Hat Enterprise Linux (RHEL) machines, you must enable FIPS mode when you install the operating system on the machines that you plan to use as worker machines.

Because FIPS must be enabled before the operating system that your cluster uses boots for the first time, you cannot enable FIPS after you deploy a cluster.

## Obtain a FIPS-capable installation program using `oc adm extract` {#installation-obtaining-fips-installer-oc_installing-fips}

You must get a FIPS-capable installation binary to install a OpenShift Container Platform cluster in FIPS mode. Extract the binary from the release image by using the OpenShift CLI (`oc`). After you get the binary, you must proceed with the cluster installation, replacing all instances of the `openshift-install` command with `openshift-install-fips`.

**Prerequisites**

- You have installed the OpenShift CLI (`oc`) with version 4.16 or newer.

**Procedure**

1. Extract the FIPS-capable binary from the installation program by running the following command:
   ```terminal
   $ oc adm release extract --registry-config "${pullsecret_file}" --command=openshift-install-fips --to "${extract_dir}" ${RELEASE_IMAGE}
   ```

   where:

   <dl>
   <dt><code>&lt;pullsecret_file&gt;</code></dt>
   <dd>Specifies the name of a file that contains your pull secret.</dd>
   <dt><code>&lt;extract_dir&gt;</code></dt>
   <dd>Specifies the directory where you want to extract the binary.</dd>
   <dt><code>&lt;RELEASE_IMAGE&gt;</code></dt>
   <dd>Specifies the Quay.io URL of the OpenShift Container Platform release you are using. For more information on finding the release image, see <em>Extracting the OpenShift Container Platform installation program</em>.</dd>
   </dl>
2. Proceed with cluster installation, replacing all instances of the `openshift-install` command with `openshift-install-fips`.

**Additional resources**

- [Cryptographic Module Validation Program](https://csrc.nist.gov/Projects/cryptographic-module-validation-program/validated-modules)
- [Compliance Activities and Government Standards](https://access.redhat.com/articles/2918071#fips-140-2-and-fips-140-3-2)
- [Installing the system in FIPS mode](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/security_hardening/assembly_installing-the-system-in-fips-mode_security-hardening)
- [Extracting the OpenShift Container Platform installation program](/docs/installing/installing_bare_metal/ipi/ipi-install-installation-workflow#retrieving-the-openshift-installer_ipi-install-installation-workflow)

## Obtain a FIPS-capable installation program using the public OpenShift mirror {#installation-obtaining-fips-installer-mirror_installing-fips}

OpenShift Container Platform requires the use of a FIPS-capable installation binary to install a cluster in FIPS mode. You can obtain this binary by downloading it from the public OpenShift mirror. After you have obtained the binary, proceed with the cluster installation, replacing all instances of the `openshift-install` binary with `openshift-install-fips`.

**Prerequisites**

- You have access to the internet.

**Procedure**

1. Download the installation program from https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest-4.18/openshift-install-rhel9-amd64.tar.gz.
2. Extract the installation program. For example, on a computer that uses a Linux operating system, run the following command:
   ```terminal
   $ tar -xvf openshift-install-rhel9-amd64.tar.gz
   ```
3. Proceed with cluster installation, replacing all instances of the `openshift-install` command with `openshift-install-fips`.

## FIPS validation in OpenShift Container Platform {#installation-about-fips-validation_installing-fips}

OpenShift Container Platform uses certain FIPS validated or Modules In Process modules within RHEL and RHCOS for the operating system components that it uses.

For more information, see "RHEL core crypto components" in the *Additional resources* section. For example, when users use SSH to connect to OpenShift Container Platform clusters and containers, those connections are properly encrypted.

OpenShift Container Platform components are written in Go and built with Red Hat’s golang compiler. When you enable FIPS mode for your cluster, all OpenShift Container Platform components that require cryptographic signing call RHEL and RHCOS cryptographic libraries.

**FIPS mode attributes and limitations in OpenShift Container Platform 4.22**

<table>
<thead>
<tr>
  <th>Attributes</th>
  <th>Limitations</th>
</tr>
</thead>
<tbody>
<tr>
  <td>FIPS support in RHEL 9 and RHCOS operating systems.</td>
  <td rowspan="4">The FIPS implementation does not use a function that performs hash computation and signature generation or validation in a single step. This limitation will continue to be evaluated and improved in future OpenShift Container Platform releases.</td>
</tr>
<tr>
  <td>FIPS support in CRI-O runtimes.</td>
</tr>
<tr>
  <td>FIPS support in OpenShift Container Platform services.</td>
</tr>
<tr>
  <td>FIPS validated or Modules In Process cryptographic module and algorithms that are obtained from RHEL 9 and RHCOS binaries and images.</td>
</tr>
<tr>
  <td>Use of FIPS compatible golang compiler.</td>
  <td>TLS FIPS support is not complete but is planned for future OpenShift Container Platform releases.</td>
</tr>
<tr>
  <td>FIPS support across multiple architectures.</td>
  <td>FIPS is currently only supported on OpenShift Container Platform deployments using <code>x86_64</code>, <code>ppc64le</code>, and <code>s390x</code> architectures.</td>
</tr>
</tbody>
</table>

**Additional resources**

- [RHEL core crypto components](https://access.redhat.com/articles/3655361)

## FIPS support in components that the cluster uses {#installation-about-fips-components_installing-fips}

Although the OpenShift Container Platform cluster itself uses FIPS validated or Modules In Process modules, ensure that the systems that support your OpenShift Container Platform cluster use FIPS validated or Modules In Process modules for cryptography.

etcd
: To ensure that the secrets that are stored in etcd use FIPS validated or Modules In Process encryption, boot the node in FIPS mode. After you install the cluster in FIPS mode, you can encrypt the etcd data by using the FIPS-approved `aes cbc` cryptographic algorithm.

Storage
: For local storage, use RHEL-provided disk encryption or Container Native Storage that uses RHEL-provided disk encryption. By storing all data in volumes that use RHEL-provided disk encryption and enabling FIPS mode for your cluster, both data at rest and data in motion, or network data, are protected by FIPS validated or Modules In Process encryption. You can configure your cluster to encrypt the root filesystem of each node. For more information, see "Customizing nodes" in the *Additional resources* section.

Runtimes
: To ensure that containers know that they are running on a host that is using FIPS validated or Modules In Process cryptography modules, use CRI-O to manage your runtimes.

**Additional resources**

- [Encrypt the etcd data](/docs/etcd/etcd-encrypt#etcd-encrypt)
- [Customizing nodes](/docs/installing/install_config/installing-customizing#installing-customizing)

## Installation of a cluster in FIPS mode {#installing-fips-mode_installing-fips}

To install a cluster in FIPS mode, follow the instructions to install a customized cluster on your preferred infrastructure. Ensure that you set `fips: true` in the `install-config.yaml` file before you deploy your cluster.

:::warning

To enable FIPS mode for your cluster, you must run the installation program from a RHEL computer configured to operate in FIPS mode. For more information about configuring FIPS mode on RHEL, see [Installing the system in FIPS mode](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/security_hardening/assembly_installing-the-system-in-fips-mode_security-hardening).

:::

- [Amazon Web Services](/docs/installing/installing_aws/ipi/installing-aws-customizations#installing-aws-customizations)
- [Microsoft Azure](/docs/installing/installing_azure/ipi/installing-azure-customizations#installing-azure-customizations)
- [Bare metal](/docs/installing/installing_bare_metal/upi/installing-bare-metal#installing-bare-metal)
- [Google Cloud](/docs/installing/installing_gcp/installing-gcp-customizations#installing-gcp-customizations)
- [IBM Cloud(R)](/docs/installing/installing_ibm_cloud/installing-ibm-cloud-customizations#installing-ibm-cloud-customizations)
- [IBM Power(R)](/docs/installing/installing_ibm_power/installing-ibm-power#installing-ibm-power)
- [IBM Z(R) and IBM(R) LinuxONE](/docs/installing/installing_ibm_z/upi/installing-ibm-z#installing-ibm-z)
- [IBM Z(R) and IBM(R) LinuxONE with RHEL KVM](/docs/installing/installing_ibm_z/upi/installing-ibm-z-kvm#installing-ibm-z-kvm)
- [IBM Z(R) and IBM(R) LinuxONE in an LPAR](/docs/installing/installing_ibm_z/upi/installing-ibm-z-lpar#installing-ibm-z-lpar)
- [Red Hat OpenStack Platform (RHOSP)](/docs/installing/installing_openstack/installing-openstack-installer-custom#installing-openstack-installer-custom)
- [VMware vSphere](/docs/installing/installing_vsphere/upi/installing-vsphere#installing-vsphere)

:::note

If you are using Azure File storage, you cannot enable FIPS mode.

:::

To apply `AES CBC` encryption to your etcd data store, follow the "Encrypting etcd data" process after you install your cluster.

**Additional resources**

- [Encrypting etcd data](/docs/etcd/etcd-encrypt#etcd-encrypt)
