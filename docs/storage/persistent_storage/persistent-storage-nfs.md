---
title: Persistent storage using NFS
sidebar_position: 9
---

# Persistent storage using NFS

<a id="persistent-storage-using-nfs"></a>

You can provision OpenShift Container Platform clusters with persistent storage using NFS.

Persistent volumes (PVs) and persistent volume claims (PVCs) provide a convenient method for sharing a volume across a project. While the NFS-specific information contained in a PV definition could also be defined directly in a pod definition, doing so does not create the volume as a distinct cluster resource, making the volume more susceptible to conflicts.

:::note

The in-tree NFS provisioner does not support user namespaces.

:::

**Additional resources**

- [Mounting NFS shares](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/managing_file_systems/mounting-nfs-shares_managing-file-systems)

## Provisioning persistent storage using NFS {#persistent-storage-nfs-provisioning_persistent-storage-nfs}

You can provision persistent storage for OpenShift Container Platform by creating persistent volume (PV) and persistent volume claim (PVC) objects that reference your NFS servers and export paths.

**Prerequisites**

- You have NFS storage available in the underlying infrastructure with the appropriate export paths configured.

**Procedure**

1. Create an object definition for the PV:
   ```yaml
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: pv0001
   spec:
     capacity:
       storage: 5Gi
     accessModes:
     - ReadWriteOnce
     nfs:
       path: /tmp
       server: 172.17.0.2
     persistentVolumeReclaimPolicy: Retain
   ```

   where:

   <dl>
   <dt><code>metadata.name</code></dt>
   <dd>Specifies the name of the volume. This is the PV identity in various <code>oc</code> commands.</dd>
   <dt><code>spec.capacity.storage</code></dt>
   <dd>Specifies the amount of storage allocated to this volume.</dd>
   <dt><code>spec.accessModes.ReadWriteOnce</code></dt>
   <dd>Though this appears to be related to controlling access to the volume, it is actually used similarly to labels and used to match a PVC to a PV. Currently, no access rules are enforced based on the <code>accessModes</code>.</dd>
   <dt><code>spec.nfs</code></dt>
   <dd>Specifies the volume type being used, in this case the <code>nfs</code> plugin.</dd>
   <dt><code>spec.nfs.path</code></dt>
   <dd>Specifies the path that is exported by the NFS server.</dd>
   <dt><code>spec.nfs.server</code></dt>
   <dd>Specifies the hostname or IP address of the NFS server.</dd>
   <dt><code>spec.persistentVolumeReclaimPolicy</code></dt>
   <dd>Specifies the reclaim policy for the PV. This defines what happens to a volume when released.</dd>
   </dl>

   :::note

   Each NFS volume must be mountable by all schedulable nodes in the cluster.

   :::
2. Verify that the PV was created:
   ```terminal
   $ oc get pv
   ```

   ```terminal title="Example output"
   NAME     LABELS    CAPACITY     ACCESSMODES   STATUS      CLAIM  REASON    AGE
   pv0001   <none>    5Gi          RWO           Available                    31s
   ```
3. Create a persistent volume claim that binds to the new PV:
   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: nfs-claim1
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 5Gi
     volumeName: pv0001
     storageClassName: ""
   ```

   where:

   <dl>
   <dt><code>spec.accessModes.ReadWriteOnce</code></dt>
   <dd>Specifies the access modes do not enforce security, but rather act as labels to match a PV to a PVC.</dd>
   <dt><code>spec.resources.requests.storage</code></dt>
   <dd>This claim looks for PVs offering <strong>5Gi</strong> or greater capacity.</dd>
   </dl>
4. Verify that the persistent volume claim was created:
   ```terminal
   $ oc get pvc
   ```

   ```terminal title="Example output"
   NAME         STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
   nfs-claim1   Bound    pv0001   5Gi        RWO                           2m
   ```

## Enforce disk quotas {#nfs-enforcing-disk-quota_persistent-storage-nfs}

You can enforce disk quotas for NFS volumes by allocating individual persistent volumes for each project, allowing you to control storage capacity per namespace.

You can use disk partitions to enforce disk quotas and size constraints. Each partition can be its own export. Each export is one PV. OpenShift Container Platform enforces unique names for PVs, but the uniqueness of the NFS volume’s server and path is up to the administrator.

Enforcing quotas in this way allows the developer to request persistent storage by a specific amount, such as 10Gi, and be matched with a corresponding volume of equal or greater capacity.

## NFS volume security {#nfs-volume-security_persistent-storage-nfs}

To understand NFS volume security, including matching permissions and SELinux considerations, you should understand the basics of POSIX permissions, process UIDs, supplemental groups, and SELinux.

Developers request NFS storage by referencing either a PVC by name or the NFS volume plugin directly in the `volumes` section of their `Pod` definition.

The `/etc/exports` file on the NFS server contains the accessible NFS directories. The target NFS directory has POSIX owner and group IDs. The OpenShift Container Platform NFS plugin mounts the container’s NFS directory with the same POSIX ownership and permissions found on the exported NFS directory. However, the container is not run with its effective UID equal to the owner of the NFS mount, which is the desired behavior.

As an example, if the target NFS directory appears on the NFS server as:

```terminal
$ ls -lZ /opt/nfs -d
```

```terminal title="Example output"
drwxrws---. nfsnobody 5555 unconfined_u:object_r:usr_t:s0   /opt/nfs
```

```terminal
$ id nfsnobody
```

```terminal title="Example output"
uid=65534(nfsnobody) gid=65534(nfsnobody) groups=65534(nfsnobody)
```

Then the container must match SELinux labels, and either run with a UID of `65534`, the `nfsnobody` owner, or with `5555` in its supplemental groups to access the directory.

:::note

The owner ID of `65534` is used as an example. Even though NFS’s `root_squash` maps `root`, uid `0`, to `nfsnobody`, uid `65534`, NFS exports can have arbitrary owner IDs. Owner `65534` is not required for NFS exports.

:::

### Group IDs {#storage-persistent-storage-nfs-group-ids_persistent-storage-nfs}

You can use supplemental groups to manage NFS access in OpenShift Container Platform when you cannot change permissions on the NFS export. Supplemental groups manage shared storage such as NFS, while block storage such as iSCSI uses the `fsGroup` SCC strategy and `fsGroup` value in the pod `securityContext`.

:::note

To gain access to persistent storage, it is generally preferable to use supplemental group IDs versus user IDs.

:::

Because the group ID on the example target NFS directory is `5555`, the pod can define that group ID using `supplementalGroups` under the `securityContext` definition of the pod. For example:

```yaml
spec:
  containers:
    - name:
    ...
  securityContext:
    supplementalGroups: [5555]
```

where:

`spec.securityContext`
: Must be defined at the pod level, not under a specific container.

`spec.securityContext.supplementalGroups`
: Specifies an array of GIDs defined for the pod. In this case, there is one element in the array. Additional GIDs would be comma-separated.

Assuming there are no custom SCCs that might satisfy the pod requirements, the pod likely matches the `restricted` SCC. This SCC has the `supplementalGroups` strategy set to `RunAsAny`, meaning that any supplied group ID is accepted without range checking.

As a result, the above pod passes admissions and is launched. However, if group ID range checking is desired, a custom SCC is the preferred solution. A custom SCC can be created such that minimum and maximum group IDs are defined, group ID range checking is enforced, and a group ID of `5555` is allowed.

:::note

To use a custom SCC, you must first add it to the appropriate service account. For example, use the `default` service account in the given project unless another has been specified on the `Pod` specification.

:::

### User IDs {#nfs-user-id_persistent-storage-nfs}

You can define user IDs in the container image or in the pod definition.

:::note

It is generally preferable to use supplemental group IDs to gain access to persistent storage versus using user IDs.

:::

In the example target NFS directory shown above, the container needs its UID set to `65534`, ignoring group IDs for the moment, so the following can be added to the `Pod` definition:

```yaml
spec:
  containers:
  - name:
  ...
    securityContext:
      runAsUser: 65534
```

where:

`spec.containers`
: Pods contain a `securityContext` definition specific to each container and a pod’s `securityContext`, which applies to all containers defined in the pod.

`spec.securityContext.runAsUser`
: Specifies the user ID to run the container as. In this example, `65534` is the `nfsnobody` user.

Assuming that the project is `default` and the SCC is `restricted`, the user ID of `65534` as requested by the pod is not allowed. Therefore, the pod fails for the following reasons:

- It requests `65534` as its user ID.
- All SCCs available to the pod are examined to see which SCC allows a user ID of `65534`. While all policies of the SCCs are checked, the focus here is on user ID.
- Because all available SCCs use `MustRunAsRange` for their `runAsUser` strategy, UID range checking is required.
- `65534` is not included in the SCC or project’s user ID range.

It is generally considered a good practice not to modify the predefined SCCs. The preferred way to fix this situation is to create a custom SCC A custom SCC can be created such that minimum and maximum user IDs are defined, UID range checking is still enforced, and the UID of `65534` is allowed.

:::note

To use a custom SCC, you must first add it to the appropriate service account. For example, use the `default` service account in the given project unless another has been specified on the `Pod` specification.

:::

### SELinux {#nfs-selinux_persistent-storage-nfs}

For non-RHEL and non-RHCOS systems, SELinux does not allow writing from a pod to a remote NFS server. The NFS volume mounts correctly but it is read-only. You need to manually enable the correct SELinux permissions.

Red Hat Enterprise Linux (RHEL) and Red Hat Enterprise Linux CoreOS (RHCOS) systems are configured to use SELinux on remote NFS servers by default.

The following procedure shows how to enable the correct SELinux permissions.

**Prerequisites**

- The `container-selinux` package must be installed. This package provides the `virt_use_nfs` SELinux boolean.

**Procedure**

- Enable the `virt_use_nfs` boolean using the following command. The `-P` option makes this boolean persistent across reboots.
  ```terminal
  # setsebool -P virt_use_nfs 1
  ```

### Export settings {#persistent-storage-nfs-export-settings_persistent-storage-nfs}

Before you can enable arbitrary container users to read and write the volume, check that each exported volume on the NFS server meets the required conditions.

**Prerequisites**

- You have access to the NFS server with root permissions.
- You have installed and configured an NFS server on your system.

**Procedure**

- Every export must be exported using the following format:
  ```terminal
  /<example_fs> *(rw,root_squash)
  ```
- The firewall must be configured to allow traffic to the mount point.
  - For NFSv4, configure the default port `2049` (**nfs**).
    ```terminal title="NFSv4"
    # iptables -I INPUT 1 -p tcp --dport 2049 -j ACCEPT
    ```
  - For NFSv3, there are three ports to configure: `2049` (**nfs**), `20048` (**mountd**), and `111` (**portmapper**).
    ```terminal title="NFSv3"
    # iptables -I INPUT 1 -p tcp --dport 2049 -j ACCEPT
    ```

    ```terminal
    # iptables -I INPUT 1 -p tcp --dport 20048 -j ACCEPT
    ```

    ```terminal
    # iptables -I INPUT 1 -p tcp --dport 111 -j ACCEPT
    ```
- The NFS export and directory must be set up so that they are accessible by the target pods. Either set the export to be owned by the container’s primary UID, or supply the pod group access using `supplementalGroups`, as shown in the group IDs above.

## Resource reclamation {#nfs-reclaiming-resources_persistent-storage-nfs}

You can release NFS shares to allow them to be reclaimed.

NFS implements the OpenShift Container Platform `Recyclable` plugin interface. Automatic processes handle reclamation tasks based on policies set on each persistent volume.

By default, PVs are set to `Retain`.

Once claim to a PVC is deleted, and the PV is released, the PV object should not be reused. Instead, a new PV should be created with the same basic volume details as the original.

For example, the administrator creates a PV named `nfs1`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs1
spec:
  capacity:
    storage: 1Mi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 192.168.1.1
    path: "/"
```

The user creates `PVC1`, which binds to `nfs1`. The user then deletes `PVC1`, releasing claim to `nfs1`. This results in `nfs1` being `Released`. If the administrator wants to make the same NFS share available, they should create a new PV with the same NFS server details, but a different PV name:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs2
spec:
  capacity:
    storage: 1Mi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 192.168.1.1
    path: "/"
```

Deleting the original PV and re-creating it with the same name is discouraged. Attempting to manually change the status of a PV from `Released` to `Available` causes errors and potential data loss.

## Additional configuration and troubleshooting {#additional-config-troubleshooting_persistent-storage-nfs}

You can configure additional NFS mount options to customize volume behavior and optimize performance for your specific storage requirements.

Depending on what version of NFS is being used and how it is configured, there may be additional configuration steps needed for proper export and security mapping. The following are some that may apply:

<table>
<tbody>
<tr>
  <td>NFSv4 mount incorrectly shows all files with ownership of <code>nobody:nobody</code></td>
  <td><ul><li>Could be attributed to the ID mapping settings, found in <code>/etc/idmapd.conf</code> on your NFS.</li><li>See <a href="https://access.redhat.com/solutions/33455">this Red Hat Solution</a>.</li></ul></td>
</tr>
<tr>
  <td>Disabling ID mapping on NFSv4</td>
  <td><ul><li>On the NFS server, run the following command:</li></ul><pre><code># echo 'Y' &gt; /sys/module/nfsd/parameters/nfs4_disable_idmapping</code></pre></td>
</tr>
</tbody>
</table>
