---
title: Uninstalling a cluster on IBM Cloud
sidebar_position: 11
---

# Uninstalling a cluster on IBM Cloud {#uninstalling-cluster-ibm-cloud}

<a id="uninstalling-cluster-ibm-cloud"></a>

You can remove an OpenShift Container Platform cluster that you deployed to IBM Cloud(R). The uninstall process removes all cluster resources and infrastructure components that the installation program provisioned.

## Removing a cluster that uses installer-provisioned infrastructure {#installation-uninstall-clouds_uninstalling-cluster-ibm-cloud}

To remove an OpenShift Container Platform cluster that uses installer-provisioned infrastructure, you can use the installation program and the installation files from your original deployment to uninstall the cluster from your cloud platform.

:::note

After uninstallation, check your cloud provider for any resources that were not removed properly, especially with user-provisioned infrastructure clusters. Some resources might exist because either the installation program did not create the resource or could not access the resource.

:::

**Prerequisites**

- You have a copy of the installation program that you used to deploy the cluster.
- You have the files that the installation program generated when you created your cluster.
- You have configured the `ccoctl` binary.
- You have installed the IBM Cloud(R) CLI and installed or updated the VPC infrastructure service plugin. For more information see "Prerequisites" in the [IBM Cloud(R) CLI documentation](https://cloud.ibm.com/docs/vpc?topic=vpc-infrastructure-cli-plugin-vpc-reference&interface=ui#cli-ref-prereqs).

**Procedure**

1. If the following conditions are met, this step is required:
   - The installer created a resource group as part of the installation process.
   - You or one of your applications created persistent volume claims (PVCs) after the cluster was deployed.
     In which case, the PVCs are not removed when uninstalling the cluster, which might prevent the resource group from being successfully removed. To prevent a failure:

     1. Log in to the IBM Cloud(R) using the CLI.
     2. To list the PVCs, run the following command:
        ```terminal
        $ ibmcloud is volumes --resource-group-name <infrastructure_id>
        ```

        For more information about listing volumes, see the [IBM Cloud(R) CLI documentation](https://cloud.ibm.com/docs/vpc?topic=vpc-infrastructure-cli-plugin-vpc-reference&interface=ui#volume-cli).
     3. To delete the PVCs, run the following command:
        ```terminal
        $ ibmcloud is volume-delete --force <volume_id>
        ```

        For more information about deleting volumes, see the [IBM Cloud(R) CLI documentation](https://cloud.ibm.com/docs/vpc?topic=vpc-infrastructure-cli-plugin-vpc-reference&interface=ui#volume-delete).
2. Export the API key that was created as part of the installation process.

```terminal
$ export IC_API_KEY=<api_key>
```

:::note

You must set the variable name exactly as specified. The installation program expects the variable name to be present to remove the service IDs that were created when the cluster was installed.

:::

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
2. Remove the manual CCO credentials that were created for the cluster:
   ```terminal
   $ ccoctl ibmcloud delete-service-id \
       --credentials-requests-dir <path_to_credential_requests_directory> \
       --name <cluster_name>
   ```

   :::note

   If your cluster uses Technology Preview features that are enabled by the `TechPreviewNoUpgrade` feature set, you must include the `--enable-tech-preview` parameter.

   :::
3. Optional: Delete the `<installation_directory>` directory and the OpenShift Container Platform installation program.
