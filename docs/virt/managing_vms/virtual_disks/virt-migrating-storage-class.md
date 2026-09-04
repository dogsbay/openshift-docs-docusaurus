---
title: Migrate a VM disk to a different storage class
sidebar_position: 4
---

# Migrate a VM disk to a different storage class {#virt-migrating-storage-class}

<a id="virt-migrating-storage-class"></a>

You can migrate one or more virtual disks to a different storage class to optimize storage performance or reduce costs without stopping your virtual machine (VM) or virtual machine instance (VMI).

## Assign storage migration permissions {#virt-migrating-storage-permissions_virt-migrating-storage-class}

Cluster administrators must grant users permission to perform storage migrations. Permissions to perform storage migrations are not part of the administrative or editing roles in the cluster by default.

**Prerequisites**

- You have cluster administrator privileges.

**Procedure**

1. (Optional) To assign the user single namespace storage migration permissions, run the following command:
   ```terminal
   $ kubectl create rolebinding <role_binding_name> \
       --clusterrole=migrations.kubevirt.io:storagemigrate \
       --user=<user_name> -n <namespace>
   ```

   where:

   <dl>
   <dt>&lt;role_binding_name&gt;</dt>
   <dd>The name to assign to this role binding instance.</dd>
   <dt>&lt;user_name&gt;</dt>
   <dd>The user to assign the storage migration permission.</dd>
   <dt>&lt;namespace&gt;</dt>
   <dd>The applicable namespace for this role binding instance.</dd>
   </dl>
2. (Optional) To assign the user multiple namespace storage migration permissions, run the following command:
   ```terminal
   $ kubectl create clusterrolebinding <role_binding_name> \
       --clusterrole=migrations.kubevirt.io:storagemigrate-multins \
       --user=<user_name>
   ```

   where:

   <dl>
   <dt>&lt;role_binding_name&gt;</dt>
   <dd>The name to assign to this role binding instance.</dd>
   <dt>&lt;user_name&gt;</dt>
   <dd>The user to assign the storage migration permission.</dd>
   </dl>

## Migrating VM disks to a different storage class by using the web console {#virt-migrating-storage-class-ui_virt-migrating-storage-class}

You can migrate one or more disks attached to a virtual machine (VM) to a different storage class by using the OpenShift Container Platform web console. When performing this action on a running VM, the operation of the VM is not interrupted and the data on the migrated disks remains accessible.

**Prerequisites**

- You must have a data volume or a persistent volume claim (PVC) available for storage class migration.
- The cluster must have a node available for live migration. As part of the storage class migration, the VM is live migrated to a different node.
- The VM must be running.

**Procedure**

1. Navigate to **Virtualization** → **VirtualMachines** in the web console.
2. Click the Options menu ![](/images/kebab.png "Options menu") beside the virtual machine and select **Migration** → **Storage**.
   You can also access this option from the **VirtualMachine details** page by selecting **Actions** → **Migration** → **Storage**.

   Alternatively, right-click the VM in the tree view and select **Migration** from the menu.
3. On the **Migration details** page, choose whether to migrate the entire VM storage or selected volumes only. If you click **Selected volumes**, select any disks that you intend to migrate. Click **Next** to proceed.
4. From the list of available options on the **Destination StorageClass** page, select the storage class to migrate to. Click **Next** to proceed.
5. On the **Review** page, review the list of affected disks and the target storage class. To start the migration, click **Migrate VirtualMachine storage**.
6. Stay on the **Migrate VirtualMachine storage** page to watch the progress and wait for the confirmation that the migration completed successfully.

**Verification**

1. From the **VirtualMachine details** page, navigate to **Configuration** → **Storage**.
2. Verify that all disks have the expected storage class listed in the **Storage class** column.
