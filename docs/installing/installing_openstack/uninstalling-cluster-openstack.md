---
title: Uninstalling a cluster on OpenStack
sidebar_position: 10
---

# Uninstalling a cluster on OpenStack {#uninstalling-cluster-openstack}

<a id="uninstalling-cluster-openstack"></a>

You can remove a cluster that you deployed to Red Hat OpenStack Platform (RHOSP).

## Removing a cluster that uses installer-provisioned infrastructure {#installation-uninstall-clouds_uninstalling-cluster-openstack}

To remove an OpenShift Container Platform cluster that uses installer-provisioned infrastructure, you can use the installation program and the installation files from your original deployment to uninstall the cluster from your cloud platform.

:::note

After uninstallation, check your cloud provider for any resources that were not removed properly, especially with user-provisioned infrastructure clusters. Some resources might exist because either the installation program did not create the resource or could not access the resource.

:::

**Prerequisites**

- You have a copy of the installation program that you used to deploy the cluster.
- You have the files that the installation program generated when you created your cluster.
- You installed the `core-installer` tool by entering the `sudo dnf install coreos-installer` command in your CLI.

**Procedure**

1. From the directory that has the installation program on the computer that you used to install the cluster, run the following command:
   ```terminal
   $ ./openshift-install destroy cluster \
   --dir <installation_directory> --log-level info
   ```

   where:

   <dl>
   <dt><code>&lt;installation_directory&gt;</code></dt>
   <dd>Specify the path to the directory that you stored the installation files in.</dd>
   <dt><code>--log-level info</code></dt>
   <dd>To view different details, specify <code>warn</code>, <code>debug</code>, or <code>error</code> instead of <code>info</code>.</dd>
   </dl>

   :::note

   You must specify the directory that includes the cluster definition files for your cluster. The installation program requires the `metadata.json` file in this directory to delete the cluster.

   :::
2. Optional: Use the `coreos-installer` tool to add the `coreos.inst.wipe=yes` flag to the Preboot Execution Environment (PXE) boot configuration. This operation wipes the disk on your system so that if you create a new cluster, you have a clean installation environment. For more detailed instructions, see [How to wipe OpenStack disks in OpenShift Container Platform 4 reinstallation](https://access.redhat.com/solutions/7128657) (Knowledgebase article).
3. Optional: Delete the `<installation_directory>` directory and the OpenShift Container Platform installation program.
