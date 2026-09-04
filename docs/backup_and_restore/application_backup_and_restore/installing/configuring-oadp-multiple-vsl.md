---
title: Configuring the OpenShift API for Data Protection (OADP) with more than one Volume Snapshot Location
sidebar_position: 1
---

# Configuring the OpenShift API for Data Protection (OADP) with more than one Volume Snapshot Location {#configuring-oadp-multiple-vsl}

<a id="configuring-oadp-multiple-vsl"></a>

Configure multiple Volume Snapshot Locations (VSLs) in the Data Protection Application (DPA) to store volume snapshots across different cloud provider regions. This provides geographic redundancy and regional disaster recovery capabilities.

## Configuring the DPA with more than one VSL {#oadp-configuring-dpa-multiple-vsl_configuring-oadp-multiple-vsl}

Configure the `DataProtectionApplication` (DPA) custom resource (CR) with multiple Volume Snapshot Locations (VSLs) using provider-specific credentials in the same region as your persistent volumes. This provides volume snapshot distribution across different storage targets.

**Procedure**

- Configure the DPA CR with more than one VSL as shown in the following example:
  ```yaml
  apiVersion: oadp.openshift.io/v1alpha1
  kind: DataProtectionApplication
  #...
  snapshotLocations:
    - velero:
        config:
          profile: default
          region: <region>
        credential:
          key: cloud
          name: cloud-credentials
        provider: aws
    - velero:
        config:
          profile: default
          region: <region>
        credential:
          key: cloud
          name: <custom_credential>
        provider: aws
  #...
  ```

  where:

  <dl>
  <dt><code>&lt;region&gt;</code></dt>
  <dd>Specifies the region. The snapshot location must be in the same region as the persistent volumes.</dd>
  <dt><code>&lt;custom_credential&gt;</code></dt>
  <dd>Specifies the custom credential name.</dd>
  </dl>
