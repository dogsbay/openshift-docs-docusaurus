---
title: Boot image skew enforcement
sidebar_position: 7
---

# Boot image skew enforcement {#mco-update-boot-skew-mgmt}

<a id="mco-update-boot-skew-mgmt"></a>

You can use boot image skew enforcement to help ensure that the boot images in a cluster are up-to-date with the OpenShift Container Platform and RHCOS version being used in the cluster. Using an older boot image could cause issues when scaling new nodes. If the images are older than a predetermined version, the MCO disables cluster upgrades until it deems the boot images to be compliant.

:::note

Boot image skew enforcement is not supported for single-node OpenShift clusters.

:::

## About boot image skew enforcement {#mco-update-boot-skew-mgmt-about_mco-update-boot-skew-mgmt}

Using boot image skew enforcement, you can ensure that the boot images in a cluster are up-to-date with the OpenShift Container Platform and RHCOS version being used in the cluster. Making sure that your boot images are current can help you avoid the problems associated with running older images.

When boot image skew enforcement is active in a cluster, the Machine Config Operator (MCO) examines the boot image version reported in the `MachineConfiguration` object to determine if that boot image is appropriate for the cluster. If the boot image version is too old, the Operator reports that *boot image version skew* is detected and blocks cluster updates until you manually update the boot image or disable boot image skew enforcement by setting the `None` mode, as described in this section.

The limit for boot image version skew is set within the MCO and cannot be modified.

For information on manually configuring the boot image in your cluster, see "Manually updating the boot image".

### About boot image skew enforcement modes {#mco-update-boot-skew-mgmt-about-modes_mco-update-boot-skew-mgmt}

Review the following information to learn about the boot image skew enforcement modes. Use the information to determine the best method for your cluster.

Boot image skew enforcement operates in one of the following modes:

Automatic
: When set to `Automatic`, with boot image management also enabled, if the cluster is updated from one OpenShift Container Platform version to the next, the MCO automatically updates the boot image version in the `MachineConfiguration` object and tests the boot image version for skew.

  :::note

  In OpenShift Container Platform 4.22, the automatic mode is available only for AWS, Google Cloud, Azure, and vSphere clusters and is the default for these platforms.

  :::

  The MCO automatically configures this mode when the following conditions are met:

  - Boot image management is available for the platform that your cluster uses. Currently boot image management is available for only AWS, Google Cloud, Azure, and vSphere clusters.
  - You have enabled boot image management for compute machine sets.
  - You have not set skew enforcement to the manual or none mode.

  For information on boot image management, see "Boot image management".

  ```yaml title="Example MachineConfiguration object with automatic skew enforcement"
  apiVersion: operator.openshift.io/v1
  kind: MachineConfiguration
  metadata:
    name: cluster
  status:
  # ...
    bootImageSkewEnforcementStatus:
      automatic:
        ocpVersion: 4.22.0
      mode: Automatic
  ```

  The MCO examines the boot image reported in the `ocpVersion` parameter to determine if the cluster is violating the boot image version skew limits.

Manual
: When set to `Manual`, if the boot image version is updated, a cluster administrator is responsible for manually updating the `MachineConfiguration` object with the RHCOS version of the new boot image or the OpenShift Container Platform version associated with the new boot image. The MCO then tests the boot image version for skew.

  ```yaml title="Example MachineConfiguration object with skew enforcement based on an RHCOS version"
  apiVersion: operator.openshift.io/v1
  kind: MachineConfiguration
  metadata:
    name: cluster
  # ...
  spec:
    bootImageSkewEnforcement:
      mode: Manual
      manual:
        mode: RHCOSVersion
        rhcosVersion: 9.2.20251023-0
  # ...
  status:
    bootImageSkewEnforcementStatus:
      manual:
        mode: RHCOSVersion
        rhcosVersion: 9.2.20251023-0
      mode: Manual
  ```

  ```yaml title="Example MachineConfiguration object with skew enforcement based on an OpenShift Container Platform version"
  apiVersion: operator.openshift.io/v1
  kind: MachineConfiguration
  metadata:
    name: cluster
  # ...
  spec:
    bootImageSkewEnforcement:
      manual:
        mode: OCPVersion
        ocpVersion: 4.22.0
      mode: Manual
  # ...
  status:
    bootImageSkewEnforcementStatus:
      manual:
        mode: OCPVersion
        ocpVersion: 4.22.0
      mode: Manual
  ```

  The MCO examines the boot image reported in the `rhcosVersion` or `ocpVersion` parameter to determine if the cluster is violating the boot image version skew limits.

None
: When set to `None`, boot image skew enforcement is disabled. When disabled, the MCO does not monitor for boot image skew and does not report if new nodes are provisioned with older boot images, which could introduce issues when scaling new nodes.

  ```yaml title="Example MachineConfiguration object with skew enforcement disabled"
  apiVersion: operator.openshift.io/v1
  kind: MachineConfiguration
  metadata:
    name: cluster
  # ...
  spec:
    bootImageSkewEnforcement:
      mode: None
  # ...
  status:
    bootImageSkewEnforcementStatus:
      mode: None
  ```

  When in the none mode, the MCO reports a Prometheus alert that skew enforcement is disabled and that scale-up might run into issues due to old boot images. The alert does not cause any functional issues for the cluster.

  Single-node OpenShift clusters default to the none mode regardless of platform, because they do not scale. The skew enforcement Prometheus alert is not reported for single-node OpenShift clusters.

  Bare-metal clusters running OpenShift Container Platform version 4.10 and later do not use the MCO to keep their boot images up-to-date. Skew enforcement defaults to the none mode and the skew enforcement Prometheus alert mentioned is not reported. For bare-metal clusters running OpenShift Container Platform version 4.9 and earlier, you need to perform a one-time action to migrate to the 4.10 system, this is explained further in the bare metal boot image update docs. For information, see "Manually updating the boot image".

## Configuring boot image skew enforcement {#mco-update-boot-skew-mgmt-configuring_mco-update-boot-skew-mgmt}

You can configure the current boot image skew enforcement mode that the Machine Config Operator (MCO) uses. By configuring the boot image skew enforcement mode, you can determine if the boot image version in the `MachineConfiguration` object is updated automatically or manually.

Alternatively, you can disable boot image skew enforcement by setting the `mode` to `None`. When disabled, the MCO does not monitor for boot image skew, and older boot images could be used, possibly introducing issues when scaling new nodes.

In OpenShift Container Platform 4.22, the automatic mode is available only for AWS, Google Cloud, Azure, and vSphere clusters and is the default for these platforms. If you modify a cluster from the automatic mode to the manual or none mode, you can revert a cluster back to automatic mode only by removing the `bootImageSkewEnforcement` stanza from the `MachineConfiguration` object.

All other platforms default to manual mode with the OpenShift Container Platform version set as the boot image version in the `MachineConfiguration` object. In manual mode, you are expected to manually update the `MachineConfiguration` object with new boot image version whenever you update the boot image.

**Procedure**

1. For manual mode, you can obtain the current boot image on a node by using the following command:
   ```terminal
   $ oc debug node/<new-node> -- chroot /host cat /sysroot/.coreos-aleph-version.json
   ```

   ```terminal title="Example output"
   # ...
       "ref": "docker://ostree-image-signed:oci-archive:/rhcos-9.6.20251023-0-ostree.x86_64.ociarchive",
       "version": "9.6.20251023-0"
   ```

   You should use the newest node on the cluster, because the boot image might have been updated after the older nodes were created. Ideally, test the newest node from each machine set and use the oldest boot image among them.
2. Specify the boot image skew enforcement mode and set the boot image version as needed:
   ```yaml
   apiVersion: operator.openshift.io/v1
   kind: MachineConfiguration
   metadata:
     name: cluster
   spec:
   # ...
     bootImageSkewEnforcement:
       mode: Manual
       manual:
         mode: RHCOSVersion
         rhcosVersion: 9.6.20251023-0
   # ...
   ```

   where:

   <dl>
   <dt><code>spec.bootImageSkewEnforcement.mode</code></dt>
   <dd>Specifies the boot image enforcement mode, one of the following values: *   <code>Manual</code>. Specifies that boot image skew management is in manual mode. You must specify the <code>spec.bootImageSkewEnforcement.manual</code> parameters. *   <code>None</code>. Specifies that boot image skew management is disabled. You do not need to specify the <code>spec.bootImageSkewEnforcement.manual</code> parameters.</dd>
   </dl>

`spec.bootImageSkewEnforcement.manual.mode`
: Specifies the version you want to represent the current boot image, either `OCPVersion` or `RHCOSVersion`. You must include one of the following parameters:

  - For `RHCOSVersion`, use `spec.bootImageSkewEnforcement.manual.rhcosVersion` to specify the RHCOS version that is being used as a boot image in the `[major].[minor].[datestamp(YYYYMMDD)]-[buildnumber]` or `[major].[minor].[timestamp(YYYYMMDDHHmm)]-[buildnumber]` format. This field must be between 14 and 21 characters.
  - For `OCPVersion`, use `spec.bootImageSkewEnforcement.manual.ocpVersion` to specify the OpenShift Container Platform version associated with the boot image that is being used in the `x.y.z` format. This field must be between 5 and 10 characters.

## Updating the boot image skew enforcement version {#mco-update-boot-skew-mgmt-updating.adoc_mco-update-boot-skew-mgmt}

If you are running boot image skew enforcement in the manual mode, you must manually update the boot image version in the `MachineConfiguration` object each time you update the boot image in your cluster. With the boot image updated in the `MachineConfiguration` object, the Machine Config Operator (MCO) can properly perform boot image skew enforcement to ensure that your nodes are up-to-date.

**Procedure**

1. If necessary, obtain the RHCOS or OpenShift Container Platform version of the current boot image on an updated node by using one of the following commands:
   - Obtain the RHCOS version by running the following command:
     ```terminal
     $ oc debug node/<new-node> -- chroot /host cat /sysroot/.coreos-aleph-version.json
     ```

     ```terminal title="Example output"
     # ...
         "ref": "docker://ostree-image-signed:oci-archive:/rhcos-9.6.20251023-0-ostree.x86_64.ociarchive",
         "version": "9.6.20251023-0"
     ```
   - Obtain the OpenShift Container Platform version by running the following command:
     ```terminal
     $ openshift-install version
     ```

     Ensure that you use the same `openshift-install` binary that you used when updating the boot image.

     ```terminal title="Example output"
     openshift-install 4.22.0
     ```
2. Specify the boot image version in the `MachineConfiguration` object with either the RHCOS or OpenShift Container Platform version:
   - Update the `MachineConfiguration` object with the RHCOS version:
     ```yaml
     apiVersion: operator.openshift.io/v1
     kind: MachineConfiguration
     metadata:
       name: cluster
     # ...
     spec:
       bootImageSkewEnforcement:
         mode: Manual
         manual:
           mode: RHCOSVersion
           rhcosVersion: 9.2.20251023-0
     # ...
     ```

     If the `spec.bootImageSkewEnforcement.manual.mode` is `RHCOSVersion`, specify the RHCOS version of the boot image with the `rhcosVersion` parameter, as shown in the example.
   - Update the `MachineConfiguration` object with the OpenShift Container Platform version
     ```yaml
     apiVersion: operator.openshift.io/v1
     kind: MachineConfiguration
     metadata:
       name: cluster
     # ...
     spec:
       bootImageSkewEnforcement:
         mode: Manual
         manual:
           mode: OCPVersion
           ocpVersion: 4.22.0
     # ...
     ```

     If the `spec.bootImageSkewEnforcement.manual.mode` is `OCPVersion`, specify the OpenShift Container Platform version of the boot image with the `ocpVersion` parameter, as shown in the example.

**Additional resources**

- [Boot image management](/docs/machine_configuration/mco-update-boot-images#mco-update-boot-images)
- [Manually updating the boot image](/docs/machine_configuration/mco-update-boot-images-manual#mco-update-boot-images-manual)
