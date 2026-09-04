---
title: Enabling user permissions to clone data volumes across namespaces
sidebar_position: 7
---

# Enabling user permissions to clone data volumes across namespaces {#virt-enabling-user-permissions-to-clone-datavolumes}

<a id="virt-enabling-user-permissions-to-clone-datavolumes"></a>

By default, users cannot clone resources between namespaces. To enable cloning, a user with the `cluster-admin` role must create and bind a cluster role that grants the required permissions.

To enable a user to clone a virtual machine to another namespace, a user with the `cluster-admin` role must create a new cluster role. Bind this cluster role to a user to enable them to clone virtual machines to the destination namespace.

## Creating RBAC resources for cloning data volumes {#virt-creating-rbac-cloning-dvs_virt-enabling-user-permissions-to-clone-datavolumes}

You can create a new cluster role that enables permissions for all actions for the `datavolumes` resource.

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).
- You must have cluster admin privileges.

:::note

If you are a non-admin user that is an administrator for both the source and target namespaces, you can create a `Role` instead of a `ClusterRole` where appropriate.

:::

**Procedure**

1. Create a `ClusterRole` manifest:
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRole
   metadata:
     name: <datavolume_cloner>
   rules:
   - apiGroups: ["cdi.kubevirt.io"]
     resources: ["datavolumes/source"]
     verbs: ["*"]
   # ...
   ```

   where:

   <dl>
   <dt><code>&lt;datavolume_cloner&gt;</code></dt>
   <dd>Specifies a unique name for the cluster role.</dd>
   </dl>
2. Create the cluster role in the cluster:
   ```terminal
   $ oc create -f <datavolume_cloner.yaml>
   ```

   where:

   <dl>
   <dt><code>&lt;datavolume_cloner.yaml&gt;</code></dt>
   <dd>Specifies the file name of the <code>ClusterRole</code> manifest created in the previous step.</dd>
   </dl>
3. Create a `RoleBinding` manifest that applies to both the source and destination namespaces and references the cluster role created in the previous step.
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: <allow_clone_to_user>
     namespace: <source_namespace>
   subjects:
   - kind: ServiceAccount
     name: default
     namespace: <destination_namespace>
   roleRef:
     kind: ClusterRole
     name: datavolume-cloner
     apiGroup: rbac.authorization.k8s.io
   ```

   - `metadata.name` specifies a unique name for the role binding.
   - `metadata.namespace` specifies the namespace for the source data volume.
   - `subjects.namespace` specifies the namespace to which the data volume is cloned.
   - `roleRef.name` specifies the name of the cluster role created in the previous step.
4. Create the role binding in the cluster:
   ```terminal
   $ oc create -f <datavolume_cloner.yaml>
   ```

   where:

   <dl>
   <dt><code>&lt;datavolume_cloner.yaml&gt;</code></dt>
   <dd>Specifies the file name of the <code>RoleBinding</code> manifest created in the previous step.</dd>
   </dl>
