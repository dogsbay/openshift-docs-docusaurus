---
title: Using preallocation for data volumes
sidebar_position: 10
---

# Using preallocation for data volumes {#virt-using-preallocation-for-datavolumes}

<a id="virt-using-preallocation-for-datavolumes"></a>

The Containerized Data Importer (CDI) can preallocate disk space to improve write performance when creating data volumes. You can enable preallocation for specific data volumes.

## About preallocation {#virt-about-preallocation_virt-using-preallocation-for-datavolumes}

The Containerized Data Importer (CDI) can use the QEMU preallocate mode for data volumes to improve write performance. You can use preallocation mode for importing and uploading operations and when creating blank data volumes.

If preallocation is enabled, CDI uses the better preallocation method depending on the underlying file system and device type:

`fallocate`
: If the file system supports it, CDI uses the operating system’s `fallocate` call to preallocate space by using the `posix_fallocate` function, which allocates blocks and marks them as uninitialized.

`full`
: If `fallocate` mode cannot be used, `full` mode allocates space for the image by writing data to the underlying storage. Depending on the storage location, all the empty allocated space might be zeroed.

## Enabling preallocation for a data volume {#virt-enabling-preallocation-for-dv_virt-using-preallocation-for-datavolumes}

You can enable preallocation for specific data volumes by including the `spec.preallocation` field in the data volume manifest. You can enable preallocation mode in either the web console or by using the OpenShift CLI (`oc`).

Preallocation mode is supported for all CDI source types. However, preallocation is ignored for cloning operations.

**Procedure**

- Specify the `spec.preallocation` field in the data volume manifest:
  ```yaml
  apiVersion: cdi.kubevirt.io/v1beta1
  kind: DataVolume
  metadata:
    name: preallocated-datavolume
  spec:
    source:
      registry:
        url: <image_url>
    storage:
      resources:
        requests:
          storage: 1Gi
    preallocation: true
  # ...
  ```

  where:

  <dl>
  <dt><code>&lt;image_url&gt;</code></dt>
  <dd>Specifies the URL of the data source in your registry.</dd>
  </dl>
