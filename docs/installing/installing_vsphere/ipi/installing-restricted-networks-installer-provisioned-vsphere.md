---
title: Installing a cluster on vSphere in a disconnected environment
sidebar_position: 5
---

# Installing a cluster on vSphere in a disconnected environment {#installing-restricted-networks-installer-provisioned-vsphere}

<a id="installing-restricted-networks-installer-provisioned-vsphere"></a>

In OpenShift Container Platform 4.22, you can install a cluster on VMware vSphere infrastructure in a restricted network by creating an internal mirror of the installation release content.

## Prerequisites {#prerequisites_installing-restricted-networks-installer-provisioned-vsphere}

- You have completed the tasks in "Preparing to install a cluster using installer-provisioned infrastructure".
- You reviewed your VMware platform licenses. Red Hat does not place any restrictions on your VMware licenses, but some VMware infrastructure components require licensing.
- You reviewed details about the OpenShift Container Platform installation and update processes.
- You read the documentation on selecting a cluster installation method and preparing it for users.
- You created a registry on your mirror host and obtained the `imageContentSources` data for your version of OpenShift Container Platform.
  :::warning

  Because the installation media is on the mirror host, you can use that computer to complete all installation steps.

  :::
- You provisioned persistent storage for your cluster. To deploy a private image registry, your storage must provide the ReadWriteMany access mode.
- The OpenShift Container Platform installer requires access to port 443 on the vCenter and ESXi hosts. You verified that port 443 is accessible.
- If you use a firewall, you confirmed with the administrator that port 443 is accessible. Control plane nodes must be able to reach vCenter and ESXi hosts on port 443 for the installation to succeed.
- If you use a firewall and plan to use the Telemetry service, you configured the firewall to allow the sites that your cluster requires access to.
  :::note

  If you are configuring a proxy, be sure to also review this site list.

  :::

## About installations in restricted networks {#installation-about-restricted-networks_installing-restricted-networks-installer-provisioned-vsphere}

You can install OpenShift Container Platform 4.22 in a restricted network without an active internet connection to obtain software components. Restricted network installations can use installer-provisioned or user-provisioned infrastructure, depending on the cloud platform to which you are installing the cluster.

If you choose to perform a restricted network installation on a cloud platform, you still require access to its cloud APIs. Some cloud functions, such as Amazon Web Service’s Route 53 DNS and IAM services, require internet access. Depending on your network, you might require less internet access for an installation on bare-metal hardware, Nutanix, or on VMware vSphere.

To complete a restricted network installation, you must create a registry that mirrors the contents of the OpenShift image registry and contains the installation media. You can create this registry on a mirror host, which can access both the internet and your closed network, or by using other methods that meet your restrictions.

### Additional limits {#installation-restricted-network-limits_installing-restricted-networks-installer-provisioned-vsphere}

Clusters in restricted networks have the following additional limitations and restrictions:

- The `ClusterVersion` status includes an `Unable to retrieve available updates` error.
- By default, you cannot use the contents of the Developer Catalog because you cannot access the required image stream tags.

## Internet access for OpenShift Container Platform {#cluster-entitlements_installing-restricted-networks-installer-provisioned-vsphere}

In OpenShift Container Platform 4.22, you require access to the internet to obtain the images that are necessary to install your cluster.

You must have internet access to perform the following actions:

- Access Red Hat Hybrid Cloud Console to download the installation program and perform subscription management. If the cluster has internet access and you do not disable Telemetry, that service automatically entitles your cluster.
- Access Quay.io to obtain the packages that are required to install your cluster.
- Obtain the packages that are required to perform cluster updates.

## Creating the RHCOS image for restricted network installations {#installation-creating-image-restricted_installing-restricted-networks-installer-provisioned-vsphere}

Download the Red Hat Enterprise Linux CoreOS (RHCOS) image to install OpenShift Container Platform on a restricted network VMware vSphere environment.

**Prerequisites**

- Obtain the OpenShift Container Platform installation program. For a restricted network installation, the program is on your mirror registry host.

**Procedure**

1. Log in to the Red Hat Customer Portal’s [Product Downloads page](https://access.redhat.com/downloads/content/290).
2. Under **Version**, select the most recent release of OpenShift Container Platform 4.22 for RHEL 8.
   :::warning

   The RHCOS images might not change with every release of OpenShift Container Platform. You must download images with the highest version that is less than or equal to the OpenShift Container Platform version that you install. Use the image versions that match your OpenShift Container Platform version if they are available.

   :::
3. Download the **Red Hat Enterprise Linux CoreOS (RHCOS) - vSphere** image.
4. Upload the image you downloaded to a location that is accessible from the bastion server.

## VMware vSphere region and zone enablement {#installation-vsphere-regions-zones_installing-restricted-networks-installer-provisioned-vsphere}

You can deploy an OpenShift Container Platform cluster to multiple vSphere data centers. Each data center can run multiple clusters. This configuration reduces the risk of a hardware failure or network outage that can cause your cluster to fail.

To enable regions and zones, you must define multiple failure domains for your OpenShift Container Platform cluster.

:::warning

The VMware vSphere region and zone enablement feature requires the vSphere Container Storage Interface (CSI) driver as the default storage driver in the cluster. As a result, the feature is only available on a newly installed cluster.

For a cluster that was upgraded from a previous release, you must enable CSI automatic migration for the cluster. You can then configure multiple regions and zones for the upgraded cluster.

:::

The default installation configuration deploys a cluster to a single vSphere data center. If you want to deploy a cluster to multiple vSphere data centers, you must create an installation configuration file that enables the region and zone feature.

The default `install-config.yaml` file includes `vcenters` and `failureDomains` fields, where you can specify multiple vSphere data centers and clusters for your OpenShift Container Platform cluster. You can use the default `failureDomains` from `install-config.yaml` if you want to install an OpenShift Container Platform cluster in a vSphere environment that consists of single data center.

The following list describes terms associated with defining zones and regions for your cluster:

- Failure domain: Establishes the relationships between a region and zone. You define a failure domain by using vCenter objects, such as a `datastore` object. A failure domain defines the vCenter location for OpenShift Container Platform cluster nodes.
- Region: Specifies a vCenter data center. You define a region by using a tag from the  `openshift-region` tag category.
- Zone: Specifies a vCenter cluster. You define a zone by using a tag from the `openshift-zone` tag category.

:::note

If you plan on specifying more than one failure domain in your `install-config.yaml` file, you must create tag categories, zone tags, and region tags in advance of creating the configuration file.

:::

You must create a vCenter tag for each vCenter data center, which represents a region. Additionally, you must create a vCenter tag for each cluster than runs in a data center, which represents a zone. After you create the tags, you must attach each tag to their respective data centers and clusters.

The following table outlines an example of the relationship among regions, zones, and tags for a configuration with multiple vSphere data centers running in a single VMware vCenter.

<table>
<thead>
<tr>
  <th>Data center (region)</th>
  <th>Cluster (zone)</th>
  <th>Tags</th>
</tr>
</thead>
<tbody>
<tr>
  <td rowspan="4">us-east</td>
  <td rowspan="2">us-east-1</td>
  <td>us-east-1a</td>
</tr>
<tr>
  <td>us-east-1b</td>
</tr>
<tr>
  <td rowspan="2">us-east-2</td>
  <td>us-east-2a</td>
</tr>
<tr>
  <td>us-east-2b</td>
</tr>
<tr>
  <td rowspan="4">us-west</td>
  <td rowspan="2">us-west-1</td>
  <td>us-west-1a</td>
</tr>
<tr>
  <td>us-west-1b</td>
</tr>
<tr>
  <td rowspan="2">us-west-2</td>
  <td>us-west-2a</td>
</tr>
<tr>
  <td>us-west-2b</td>
</tr>
</tbody>
</table>

## VMware vSphere host group enablement {#installation-vsphere-regions-zones-host-groups_installing-restricted-networks-installer-provisioned-vsphere}

When deploying an OpenShift Container Platform cluster to VMware vSphere, you can map your vSphere host groups onto OpenShift Container Platform failure domains. This is useful if you are using a stretched cluster configuration, where ESXi hosts are grouped into host groups by physical location.

To enable this feature, you must meet the following requirements:

- You must arrange your ESXi hosts into host groups.
- You must create a vCenter tag in the `openshift-region` tag category for your cluster. After you create the tag, you must attach the tag to the cluster.
- You must create a vCenter tag in the `openshift-zone` tag category for each host group and then attach the correct tag to each ESXi host.
- You must define multiple failure domains for your OpenShift Container Platform cluster in the `install-config.yaml` file.
- You must grant the `Host.Inventory.EditCluster` privilege on the vSphere vCenter cluster object.

Review the following key terms, which correspond to parameters in your `install-config.yaml` file that you must configure to enable this feature:

- Failure domain: Specifies the relationships between regions and zones in OpenShift Container Platform, and clusters and host groups in vSphere. You define a failure domain by using vCenter objects, such as a `datastore` object. A failure domain defines the vCenter location for OpenShift Container Platform cluster nodes.
- Region: Specifies a vCenter cluster. You define a region by using a tag from the `openshift-region` tag category.
- Zone: Specifies a vCenter host group. You define a zone by using a tag from the `openshift-zone` tag category.
- Region type: Specifies the `ComputeCluster` region type to enable this feature.
- Zone type: Specifies the `HostGroup` zone type to enable this feature.

**Additional resources**

- [Additional VMware vSphere configuration parameters](/docs/installing/installing_vsphere/installation-config-parameters-vsphere#installation-configuration-parameters-additional-vsphere_installation-config-parameters-vsphere)
- [Deprecated VMware vSphere configuration parameters](/docs/installing/installing_vsphere/installation-config-parameters-vsphere#deprecated-parameters-vsphere_installation-config-parameters-vsphere)
- [vSphere automatic migration](/docs/storage/container_storage_interface/persistent-storage-csi-migration#persistent-storage-csi-migration-sc-vsphere_persistent-storage-csi-migration)
- [VMware vSphere CSI Driver Operator](/docs/storage/container_storage_interface/persistent-storage-csi-vsphere#persistent-storage-csi-vsphere-top-aware_persistent-storage-csi-vsphere)

## Creating the installation configuration file {#installation-initializing_installing-restricted-networks-installer-provisioned-vsphere}

You can customize the OpenShift Container Platform cluster you install on VMware vSphere.

**Prerequisites**

- You have the OpenShift Container Platform installation program and the pull secret for your cluster. For a restricted network installation, these files are on your mirror host.
- You have the `imageContentSources` values that were generated during mirror registry creation.
- You have obtained the contents of the certificate for your mirror registry.
- You have retrieved a Red Hat Enterprise Linux CoreOS (RHCOS) image and uploaded it to an accessible location.

**Procedure**

1. Create the `install-config.yaml` file.
   1. Change to the directory that contains the installation program and run the following command:
      ```terminal
      $ ./openshift-install create install-config --dir <installation_directory>
      ```

      - `<installation_directory>`: For `<installation_directory>`, specify the directory name to store the files that the installation program creates.
        When specifying the directory:
      - Verify that the directory has the `execute` permission. This permission is required to run Terraform binaries under the installation directory.
      - Use an empty directory. Some installation assets, such as bootstrap X.509 certificates, have short expiration intervals, therefore you must not reuse an installation directory. If you want to reuse individual files from another cluster installation, you can copy them into your directory. However, the file names for the installation assets might change between releases. Use caution when copying installation files from an earlier OpenShift Container Platform version.
   2. At the prompts, provide the configuration details for your cloud:
      1. Optional: Select an SSH key to use to access your cluster machines.
         :::note

         For production OpenShift Container Platform clusters on which you want to perform installation debugging or disaster recovery, specify an SSH key that your `ssh-agent` process uses.

         :::
      2. Select **vsphere** as the platform to target.
      3. Specify the name of your vCenter instance.
      4. Specify the user name and password for the vCenter account that has the required permissions to create the cluster.
         The installation program connects to your vCenter instance.
      5. Select the data center in your vCenter instance to connect to.
         :::note

         After you create the installation configuration file, you can modify the file to create a multiple vSphere data center environment. This means that you can deploy an OpenShift Container Platform cluster to multiple vSphere data centers. For more information about creating this environment, see the section named *VMware vSphere region and zone enablement*.

         :::
      6. Select the default vCenter datastore to use.
         :::warning

         You can specify the path of any datastore that exists in a datastore cluster. By default, Storage Distributed Resource Scheduler (SDRS), which uses Storage vMotion, is automatically enabled for a datastore cluster. Red Hat does not support Storage vMotion, so you must disable Storage DRS to avoid data loss issues for your OpenShift Container Platform cluster.

         You cannot specify more than one datastore path. If you must specify VMs across multiple datastores, use a `datastore` object to specify a failure domain in your cluster’s `install-config.yaml` configuration file. For more information, see "VMware vSphere region and zone enablement".

         :::
      7. Select the vCenter cluster to install the OpenShift Container Platform cluster in. The installation program uses the root resource pool of the vSphere cluster as the default resource pool.
      8. Select the network in the vCenter instance that contains the virtual IP addresses and DNS records that you configured.
      9. Enter the virtual IP address that you configured for control plane API access.
      10. Enter the virtual IP address that you configured for cluster ingress.
      11. Enter the base domain. This base domain must be the same one that you used in the DNS records that you configured.
      12. Enter a descriptive name for your cluster.

      The cluster name you enter must match the cluster name you specified when configuring the DNS records.
2. Choose one of the following methods to speficy an RHCOS image for your cluster than runs in a VMware vSphere vCenter environment.
   1. The `clusterOSImage` parameter method: In the `install-config.yaml` file, set the value of `platform.vsphere.clusterOSImage` to the image location or name. For example:
      ```yaml
      platform:
        vsphere:
            clusterOSImage: http://mirror.example.com/images/rhcos-43.81.201912131630.0-vmware.x86_64.ova?sha256=ffebbd68e8a1f2a245ca19522c16c86f67f9ac8e4e0c1f0a812b068b16f7265d
      ```
   2. The `topology.template` parameter method:
      1. Download the **Red Hat Enterprise Linux CoreOS (RHCOS) - vSphere** image in Open Virtual Appliance (OVA) format to your local system. For more information, see "Creating the RHCOS image for restricted network installations".
      2. From the **Hosts and Clusters** tab on the vSphere Client, right-click your cluster name and select **Deploy OVF Template**.
      3. On the **Select an OVF** tab, specify the name of the RHCOS OVA file that you downloaded.
      4. On the **Select a name and folder** tab, set a **Virtual machine name** for your template, such as `Template-RHCOS`.
      5. Click the name of your vSphere cluster and select the folder you created in the previous step.
      6. On the **Select a compute resource** tab, click the name of your vSphere cluster.
      7. On the **Select storage** tab, configure the storage options for your VM.
      8. When creating the OVF template, do not specify values on the **Customize template** tab or configure the template any further.
      9. In the `install-config.yaml` file, set the value of `topology.template` to the path where you imported the image to your vSphere vCenter instance.
3. Edit the `install-config.yaml` file to give the additional information that is required for an installation in a restricted network.
   1. Update the `pullSecret` value to contain the authentication information for your registry:
      ```yaml
      pullSecret: '{"auths":{"<mirror_host_name>:5000": {"auth": "<credentials>","email": "you@example.com"}}}'
      ```

      For `<mirror_host_name>`, specify the registry domain name that you specified in the certificate for your mirror registry, and for `<credentials>`, specify the base64-encoded user name and password for your mirror registry.
   2. Add the `additionalTrustBundle` parameter and value.
      ```yaml
      additionalTrustBundle: |
        -----BEGIN CERTIFICATE-----
        ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ
        -----END CERTIFICATE-----
      ```

      The value must be the contents of the certificate file that you used for your mirror registry. The certificate file can be an existing, trusted certificate authority, or the self-signed certificate that you generated for the mirror registry.
   3. Add the image content resources, which resemble the following YAML excerpt:
      ```yaml
      imageContentSources:
      - mirrors:
        - <mirror_host_name>:5000/<repo_name>/release
        source: quay.io/openshift-release-dev/ocp-release
      - mirrors:
        - <mirror_host_name>:5000/<repo_name>/release
        source: registry.redhat.io/ocp/release
      ```

      For these values, use the `imageContentSources` that you recorded during mirror registry creation.
4. Make any other modifications to the `install-config.yaml` file that you require.
   For more information about the parameters, see "Installation configuration parameters".
5. Back up the `install-config.yaml` file so that you can use it to install multiple clusters.
   :::warning

   The `install-config.yaml` file is consumed during the installation process. If you want to reuse the file, you must back it up now.

   :::

**Additional resources**

- [Installation configuration parameters](/docs/installing/installing_vsphere/installation-config-parameters-vsphere#installation-config-parameters-vsphere)

### Sample install-config.yaml file for a VMware vSphere cluster {#installation-vsphere-config-yaml_installing-restricted-networks-installer-provisioned-vsphere}

You can customize the `install-config.yaml` file to specify more details about your OpenShift Container Platform cluster’s platform or change the values of the required parameters.

:::warning

Carefully review the "Installation configuration parameters for vSphere" page for detailed parameter explanations.

:::

```yaml
apiVersion: v1
baseDomain: example.com
metadata:
  name: test
sshKey: ssh-ed25519 AAAA...
compute:
- name:  <worker_name>
  platform: {}
  replicas: 3
controlPlane:
  name: <control_plane_name>
  platform: {}
  replicas: 3
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
platform:
  vsphere:
    apiVIPs:
    - 10.0.0.1
    ingressVIPs:
    - 10.0.0.2
    failureDomains:
    - name: <failure_domain_name>
      region: <default_region_name>
      server: <fully_qualified_domain_name>
      topology:
        computeCluster: "/<data_center>/host/<cluster>"
        datacenter: <data_center>
        datastore: "/<data_center>/datastore/<datastore>"
        networks:
        - <VM_Network_name>
      zone: <default_zone_name>
    vcenters:
    - datacenters:
      - <data_center>
      server: <fully_qualified_domain_name>
      user: administrator@vsphere.local
```

where:

`compute`
: Specifes the parameters that apply to compute nodes.

`controlPlane`
: Specifies the parameters that apply to control plane nodes.

`networking`
: Specifies the parameters that apply to cluster networking configuration.

`platform`
: Specifies the parameters that apply to the configuration of the platform hosting the cluster.

### Configuring the cluster-wide proxy during installation {#installation-configure-proxy_installing-restricted-networks-installer-provisioned-vsphere}

Production environments can deny direct access to the internet and instead have an HTTP or HTTPS proxy available. You can configure a new OpenShift Container Platform cluster to use a proxy by configuring the proxy settings in the `install-config.yaml` file.

**Prerequisites**

- You have an existing `install-config.yaml` file.
- You have reviewed the sites that your cluster requires access to and determined whether any of them need to bypass the proxy. By default, the proxy handles all cluster egress traffic, including calls to hosting cloud provider APIs. You added sites to the `Proxy` object’s `spec.noProxy` field to bypass the proxy if necessary.
  :::note

  The `Proxy` object `status.noProxy` field includes the values of the `networking.machineNetwork[].cidr`, `networking.clusterNetwork[].cidr`, and `networking.serviceNetwork[]` fields from your installation configuration.

  For installations on Amazon Web Services (AWS), Google Cloud, Microsoft Azure, and Red Hat OpenStack Platform (RHOSP), the `Proxy` object `status.noProxy` field also includes the instance metadata endpoint (`169.254.169.254`).

  :::

**Procedure**

1. Edit your `install-config.yaml` file and add the proxy settings. For example:
   ```yaml
   apiVersion: v1
   baseDomain: my.domain.com
   proxy:
     httpProxy: http://<username>:<pswd>@<ip>:<port>
     httpsProxy: https://<username>:<pswd>@<ip>:<port>
     noProxy: example.com
   additionalTrustBundle: |
       -----BEGIN CERTIFICATE-----
       <MY_TRUSTED_CA_CERT>
       -----END CERTIFICATE-----
   additionalTrustBundlePolicy: <policy_to_add_additionalTrustBundle>
   # ...
   ```

   where:

   <dl>
   <dt><code>proxy.httpProxy</code></dt>
   <dd>Specifies a proxy URL to use for creating HTTP connections outside the cluster. The URL scheme must be <code>http</code>.</dd>
   <dt><code>proxy.httpsProxy</code></dt>
   <dd>Specifies a proxy URL to use for creating HTTPS connections outside the cluster.</dd>
   <dt><code>proxy.noProxy</code></dt>
   <dd>Specifies a comma-separated list of destination domain names, IP addresses, or other network CIDRs to exclude from proxying. Preface a domain with <code>.</code> to match subdomains only. For example, <code>.y.com</code> matches <code>x.y.com</code>, but not <code>y.com</code>. Use <code>*</code> to bypass the proxy for all destinations. You must include vCenter’s IP address and the IP range that you use for its machines.</dd>
   <dt><code>additionalTrustBundle</code></dt>
   <dd>If you specify this value, the installation program generates a config map named <code>user-ca-bundle</code> in the <code>openshift-config</code> namespace to hold the additional CA certificates. If you specify <code>additionalTrustBundle</code> and at least one proxy setting, the <code>Proxy</code> object references the <code>user-ca-bundle</code> config map in the <code>trustedCA</code> field. The Cluster Network Operator then creates a <code>trusted-ca-bundle</code> config map that merges the contents specified for the <code>trustedCA</code> parameter with the RHCOS trust bundle. You must set the <code>additionalTrustBundle</code> field unless an authority from the RHCOS trust bundle signs the proxy’s identity certificate.</dd>
   <dt><code>additionalTrustBundlePolicy</code></dt>
   <dd>Specifies the policy that determines the configuration of the <code>Proxy</code> object to reference the <code>user-ca-bundle</code> config map in the <code>trustedCA</code> field. The allowed values are <code>Proxyonly</code> and <code>Always</code>. Use <code>Proxyonly</code> to reference the <code>user-ca-bundle</code> config map only when you configure an <code>http/https</code> proxy. Use <code>Always</code> to always reference the <code>user-ca-bundle</code> config map. The default value is <code>Proxyonly</code>. Optional parameter.</dd>
   </dl>

   :::note

   The installation program does not support the proxy `readinessEndpoints` field.

   :::

   :::note

   If the installation program times out, restart and then complete the deployment by using the `wait-for` command of the installation program. For example:

   ```terminal
   $ ./openshift-install wait-for install-complete --log-level debug
   ```

   :::
2. Save the file and reference it when installing OpenShift Container Platform.
   The installation program creates a cluster-wide proxy named `cluster` that uses the proxy settings in the `install-config.yaml` file. If you do not give proxy settings, the installation program still creates a `cluster` `Proxy` object, but it has a nil `spec`.

   :::note

   Only the `Proxy` object named `cluster` is supported, and you cannot create additional proxies.

   :::

### Configuring regions and zones for a VMware vCenter {#configuring-vsphere-regions-zones_installing-restricted-networks-installer-provisioned-vsphere}

You can modify the default installation configuration file, so that you can deploy an OpenShift Container Platform cluster to multiple vSphere data centers.

The default `install-config.yaml` file configuration from the previous release of OpenShift Container Platform is deprecated. You can continue to use the deprecated default configuration, but the `openshift-installer` will prompt you with a warning message that indicates the use of deprecated fields in the configuration file.

**Prerequisites**

- You have an existing `install-config.yaml` installation configuration file.
  :::warning

  You must specify at least one failure domain for your OpenShift Container Platform cluster, so that you can provision data center objects for your VMware vCenter server. Consider specifying multiple failure domains if you need to provision virtual machine nodes in different data centers, clusters, datastores, and other components. To enable regions and zones, you must define multiple failure domains for your OpenShift Container Platform cluster.

  :::
- You have installed the `govc` command line tool.
  :::warning

  The example uses the `govc` command. The `govc` command is an open source command available from VMware; it is not available from Red Hat. The Red Hat support team does not maintain the `govc` command. Instructions for downloading and installing `govc` are found on the VMware documentation website.

  :::

**Procedure**

1. Create the `openshift-region` and `openshift-zone` vCenter tag categories by running the following commands:
   :::warning

   If you specify different names for the `openshift-region` and `openshift-zone` vCenter tag categories, the installation of the OpenShift Container Platform cluster fails.

   :::

   ```terminal
   $ govc tags.category.create -d "OpenShift region" openshift-region
   ```

   ```terminal
   $ govc tags.category.create -d "OpenShift zone" openshift-zone
   ```
2. For each region where you want to deploy your cluster, create a region tag by running the following command:
   ```terminal
   $ govc tags.create -c <region_tag_category> <region_tag>
   ```
3. For each zone where you want to deploy your cluster, create a zone tag by running the following command:
   ```terminal
   $ govc tags.create -c <zone_tag_category> <zone_tag>
   ```
4. Attach region tags to each vCenter data center object by running the following command:
   ```terminal
   $ govc tags.attach -c <region_tag_category> <region_tag_1> /<data_center_1>
   ```
5. Attach the zone tags to each vCenter cluster object by running the following command:
   ```terminal
   $ govc tags.attach -c <zone_tag_category> <zone_tag_1> /<data_center_1>/host/<cluster1>
   ```
6. Change to the directory that contains the installation program and initialize the cluster deployment according to your chosen installation requirements.
   ```yaml title="Sample install-config.yaml file with multiple data centers defined in a vSphere center"
   # ...
   compute:
   ---
     vsphere:
         zones:
           - "<machine_pool_zone_1>"
           - "<machine_pool_zone_2>"
   # ...
   controlPlane:
   # ...
   vsphere:
         zones:
           - "<machine_pool_zone_1>"
           - "<machine_pool_zone_2>"
   # ...
   platform:
     vsphere:
       vcenters:
   # ...
       datacenters:
         - <data_center_1_name>
         - <data_center_2_name>
       failureDomains:
       - name: <machine_pool_zone_1>
         region: <region_tag_1>
         zone: <zone_tag_1>
         server: <fully_qualified_domain_name>
         topology:
           datacenter: <data_center_1>
           computeCluster: "/<data_center_1>/host/<cluster1>"
           networks:
           - <VM_Network1_name>
           datastore: "/<data_center_1>/datastore/<datastore1>"
           resourcePool: "/<data_center_1>/host/<cluster1>/Resources/<resourcePool1>"
           folder: "/<data_center_1>/vm/<folder1>"
       - name: <machine_pool_zone_2>
         region: <region_tag_2>
         zone: <zone_tag_2>
         server: <fully_qualified_domain_name>
         topology:
           datacenter: <data_center_2>
           computeCluster: "/<data_center_2>/host/<cluster2>"
           networks:
           - <VM_Network2_name>
           datastore: "/<data_center_2>/datastore/<datastore2>"
           resourcePool: "/<data_center_2>/host/<cluster2>/Resources/<resourcePool2>"
           folder: "/<data_center_2>/vm/<folder2>"
   # ...
   ```

### Configuring host groups for a VMware vCenter {#configuring-vsphere-host-groups_installing-restricted-networks-installer-provisioned-vsphere}

You can modify the default installation configuration file to deploy an OpenShift Container Platform cluster on a VMware vSphere stretched cluster, where ESXi hosts are grouped into host groups by physical location.

The default `install-config.yaml` file configuration from previous releases of OpenShift Container Platform is deprecated. Though you can still use it, the OpenShift Container Platform installer will display a warning message that indicates the use of deprecated fields in the configuration file.

**Prerequisites**

- You have an existing `install-config.yaml` installation configuration file.
- You have arranged your ESXi hosts into host groups.
- You have granted the `Host.Inventory.EditCluster` privilege on the vSphere vCenter cluster object.
- You have downloaded and installed the `govc` command line tool. Instructions can be found on the VMware documentation website. Note that `govc` is an open-source tool that is not maintained by the Red Hat support team.
  :::warning

  To enable host group support, you must define multiple failure domains for your OpenShift Container Platform cluster.

  :::

**Procedure**

1. Create the `openshift-region` and `openshift-zone` vCenter tag categories by running the following commands:
   :::warning

   If you specify different names for the `openshift-region` and `openshift-zone` vCenter tag categories, the installation of the OpenShift Container Platform cluster fails.

   :::

   ```terminal
   $ govc tags.category.create -d "OpenShift region" openshift-region
   ```

   ```terminal
   $ govc tags.category.create -d "OpenShift zone" openshift-zone
   ```
2. Create a region tag for the vSphere cluster where you want to deploy your OpenShift Container Platform cluster by entering the following command:
   ```terminal
   $ govc tags.create -c <region_tag_category> <region_tag>
   ```
3. Create a zone tag for each host group by entering the following command as needed:
   ```terminal
   $ govc tags.create -c <zone_tag_category> <zone_tag>
   ```
4. Attach the region tag to the vCenter cluster object by entering the following command:
   ```terminal
   $ govc tags.attach -c <region_tag_category> <region_tag_1> /<datacenter_1>/host/<cluster_1>
   ```
5. Use zone tags to associate each ESXi host with its host group, by entering the following command for each ESXi host:
   ```terminal
   $ govc tags.attach -c <zone_tag_category> <zone_tag_for_host_group_1> /<datacenter_1>/host/<cluster_1>/<esxi_host_in_host_group_1>
   ```
6. Change to the directory that contains the installation program and initialize the cluster deployment according to your chosen installation requirements.
   ```yaml title="Sample install-config.yaml file with multiple host groups"
   platform:
     vsphere:
       vcenters:
   # ...
       datacenters:
         - <data_center_1_name>
       failureDomains:
       - name: <host_group_1>
         region: <cluster_1_region_tag>
         zone: <host_group_1_zone_tag>
         regionType: "ComputeCluster"
         zoneType: "HostGroup"
         server: <fully_qualified_domain_name>
         topology:
           datacenter: <data_center_1>
           computeCluster: "/<data_center_1>/host/<cluster_1>"
           networks:
           - <VM_Network1_name>
           hostGroup: <host_group_1_name>
           datastore: "/<data_center_1>/datastore/<datastore_1>"
           resourcePool: "/<data_center_1>/host/<cluster_1>/Resources/<resourcePool_1>"
           folder: "/<data_center_1>/vm/<folder_1>"
       - name: <host_group_2>
         region: <cluster_1_region_tag>
         zone: <host_group_2_zone_tag>
         regionType: "ComputeCluster"
         zoneType: "HostGroup"
         server: <fully_qualified_domain_name>
         topology:
           datacenter: <data_center_1>
           computeCluster: "/<data_center_1>/host/<cluster_1>"
           networks:
           - <VM_Network1_name>
           hostGroup: <host_group_2_name>
           datastore: "/<data_center_1>/datastore/<datastore_1>"
           resourcePool: "/<data_center_1>/host/<cluster_1>/Resources/<resourcePool_1>"
           folder: "/<data_center_1>/vm/<folder_1>"
   ```

## Services for a user-managed load balancer {#nw-osp-services-external-load-balancer_installing-restricted-networks-installer-provisioned-vsphere}

You can configure an OpenShift Container Platform cluster to use a user-managed load balancer in place of the default load balancer.

:::warning

Configuring a user-managed load balancer depends on your vendor’s load balancer.

The information and examples in this section are for guideline purposes only. Consult the vendor documentation for more specific information about the vendor’s load balancer.

:::

Red Hat supports the following services for a user-managed load balancer:

- Ingress Controller
- OpenShift API
- OpenShift MachineConfig API

You can choose whether you want to configure one or all of these services for a user-managed load balancer. Configuring only the Ingress Controller service is a common configuration option. To better understand each service, view the following diagrams:

**Figure 1. Example network workflow that shows an Ingress Controller operating in an OpenShift Container Platform environment**

![An image that shows an example network workflow of an Ingress Controller operating in an OpenShift Container Platform environment.](/images/external-load-balancer-default.png)

**Figure 2. Example network workflow that shows an OpenShift API operating in an OpenShift Container Platform environment**

![An image that shows an example network workflow of an OpenShift API operating in an OpenShift Container Platform environment.](/images/external-load-balancer-openshift-api.png)

**Figure 3. Example network workflow that shows an OpenShift **`MachineConfig`** API operating in an OpenShift Container Platform environment**

![An image that shows an example network workflow of an OpenShift `MachineConfig` API operating in an OpenShift Container Platform environment.](/images/external-load-balancer-machine-config-api.png)

The following configuration options are supported for user-managed load balancers:

- Use a node selector to map the Ingress Controller to a specific set of nodes. You must assign a static IP address to each node in this set, or configure each node to receive the same IP address from the Dynamic Host Configuration Protocol (DHCP). Infrastructure nodes commonly receive this type of configuration.
- Target all IP addresses on a subnet. This configuration can reduce the effort required to maintain the load balancer, because you can create and destroy nodes within those networks without reconfiguring the load balancer targets. If you deploy your ingress pods by using a machine set on a smaller network, such as a `/27` or `/28`, you can simplify your load balancer targets.
  :::tip

  You can list all IP addresses that exist in a network by checking the machine config pool’s resources.

  :::

Before you configure a user-managed load balancer for your OpenShift Container Platform cluster, consider the following information:

- For a front-end IP address, you can use the same IP address for the front-end IP address, the Ingress Controller load balancer, and API load balancer. Check the vendor’s documentation for this capability.
- For a back-end IP address, ensure that an IP address for an OpenShift Container Platform control plane node does not change during the lifetime of the user-managed load balancer. You can achieve this by completing one of the following actions:
  - Assign a static IP address to each control plane node.
  - Configure each node to receive the same IP address from the DHCP every time the node requests a DHCP lease. Depending on the vendor, the DHCP lease might be in the form of an IP reservation or a static DHCP assignment.
- Manually define each node that runs the Ingress Controller in the user-managed load balancer for the Ingress Controller back-end service. For example, if the Ingress Controller moves to an undefined node, a connection outage can occur.

### Configuring a user-managed load balancer {#nw-osp-configuring-external-load-balancer_installing-restricted-networks-installer-provisioned-vsphere}

You can configure an OpenShift Container Platform cluster to use a user-managed load balancer in place of the default load balancer.

:::warning

Before you configure a user-managed load balancer, ensure that you read the "Services for a user-managed load balancer" section.

:::

Read the following prerequisites that apply to the service that you want to configure for your user-managed load balancer.

:::note

MetalLB, which runs on a cluster, functions as a user-managed load balancer.

:::

**Prerequisites**

The following list details OpenShift API prerequisites:

- You defined a front-end IP address.
- TCP ports 6443 and 22623 are exposed on the front-end IP address of your load balancer. Check the following items:
  - Port 6443 provides access to the OpenShift API service.
  - Port 22623 can provide ignition startup configurations to nodes.
- The front-end IP address and port 6443 are reachable by all users of your system with a location external to your OpenShift Container Platform cluster.
- The front-end IP address and port 22623 are reachable only by OpenShift Container Platform nodes.
- The load balancer backend can communicate with OpenShift Container Platform control plane nodes on port 6443 and 22623.

The following list details Ingress Controller prerequisites:

- You defined a front-end IP address.
- TCP port 443 and port 80 are exposed on the front-end IP address of your load balancer.
- The front-end IP address, port 80 and port 443 are reachable by all users of your system with a location external to your OpenShift Container Platform cluster.
- The front-end IP address, port 80 and port 443 are reachable by all nodes that operate in your OpenShift Container Platform cluster.
- The load balancer backend can communicate with OpenShift Container Platform nodes that run the Ingress Controller on ports 80, 443, and 1936.

The following list details prerequisites for health check URL specifications:

You can configure most load balancers by setting health check URLs that determine if a service is available or unavailable. OpenShift Container Platform provides these health checks for the OpenShift API, Machine Configuration API, and Ingress Controller backend services.

The following example shows a Kubernetes API health check specification for a backend service:

```terminal
Path: HTTPS:6443/readyz
Healthy threshold: 2
Unhealthy threshold: 2
Timeout: 10
Interval: 10
```

The following example shows a Machine Config API health check specification for a backend service:

```terminal
Path: HTTPS:22623/healthz
Healthy threshold: 2
Unhealthy threshold: 2
Timeout: 10
Interval: 10
```

The following example shows a Ingress Controller health check specification for a backend service:

```terminal
Path: HTTP:1936/healthz/ready
Healthy threshold: 2
Unhealthy threshold: 2
Timeout: 5
Interval: 10
```

**Procedure**

1. Configure the HAProxy Ingress Controller, so that you can enable access to the cluster from your load balancer on ports 6443, 22623, 443, and 80. Depending on your needs, you can specify the IP address of a single subnet or IP addresses from multiple subnets in your HAProxy configuration.
   ```terminal title="Example HAProxy configuration with one listed subnet"
   # ...
   listen my-cluster-api-6443
       bind 192.168.1.100:6443
       mode tcp
       balance roundrobin
     option httpchk
     http-check connect
     http-check send meth GET uri /readyz
     http-check expect status 200
       server my-cluster-master-2 192.168.1.101:6443 check inter 10s rise 2 fall 2
       server my-cluster-master-0 192.168.1.102:6443 check inter 10s rise 2 fall 2
       server my-cluster-master-1 192.168.1.103:6443 check inter 10s rise 2 fall 2

   listen my-cluster-machine-config-api-22623
       bind 192.168.1.100:22623
       mode tcp
       balance roundrobin
     option httpchk
     http-check connect
     http-check send meth GET uri /healthz
     http-check expect status 200
       server my-cluster-master-2 192.168.1.101:22623 check inter 10s rise 2 fall 2
       server my-cluster-master-0 192.168.1.102:22623 check inter 10s rise 2 fall 2
       server my-cluster-master-1 192.168.1.103:22623 check inter 10s rise 2 fall 2

   listen my-cluster-apps-443
       bind 192.168.1.100:443
       mode tcp
       balance roundrobin
     option httpchk
     http-check connect
     http-check send meth GET uri /healthz/ready
     http-check expect status 200
       server my-cluster-worker-0 192.168.1.111:443 check port 1936 inter 10s rise 2 fall 2
       server my-cluster-worker-1 192.168.1.112:443 check port 1936 inter 10s rise 2 fall 2
       server my-cluster-worker-2 192.168.1.113:443 check port 1936 inter 10s rise 2 fall 2

   listen my-cluster-apps-80
      bind 192.168.1.100:80
      mode tcp
      balance roundrobin
     option httpchk
     http-check connect
     http-check send meth GET uri /healthz/ready
     http-check expect status 200
       server my-cluster-worker-0 192.168.1.111:80 check port 1936 inter 10s rise 2 fall 2
       server my-cluster-worker-1 192.168.1.112:80 check port 1936 inter 10s rise 2 fall 2
       server my-cluster-worker-2 192.168.1.113:80 check port 1936 inter 10s rise 2 fall 2
   # ...
   ```

   ```terminal title="Example HAProxy configuration with multiple listed subnets"
   # ...
   listen api-server-6443
       bind *:6443
       mode tcp
         server master-00 192.168.83.89:6443 check inter 1s
         server master-01 192.168.84.90:6443 check inter 1s
         server master-02 192.168.85.99:6443 check inter 1s
         server bootstrap 192.168.80.89:6443 check inter 1s

   listen machine-config-server-22623
       bind *:22623
       mode tcp
         server master-00 192.168.83.89:22623 check inter 1s
         server master-01 192.168.84.90:22623 check inter 1s
         server master-02 192.168.85.99:22623 check inter 1s
         server bootstrap 192.168.80.89:22623 check inter 1s

   listen ingress-router-80
       bind *:80
       mode tcp
       balance source
         server worker-00 192.168.83.100:80 check inter 1s
         server worker-01 192.168.83.101:80 check inter 1s

   listen ingress-router-443
       bind *:443
       mode tcp
       balance source
         server worker-00 192.168.83.100:443 check inter 1s
         server worker-01 192.168.83.101:443 check inter 1s

   listen ironic-api-6385
       bind *:6385
       mode tcp
       balance source
         server master-00 192.168.83.89:6385 check inter 1s
         server master-01 192.168.84.90:6385 check inter 1s
         server master-02 192.168.85.99:6385 check inter 1s
         server bootstrap 192.168.80.89:6385 check inter 1s

   listen inspector-api-5050
       bind *:5050
       mode tcp
       balance source
         server master-00 192.168.83.89:5050 check inter 1s
         server master-01 192.168.84.90:5050 check inter 1s
         server master-02 192.168.85.99:5050 check inter 1s
         server bootstrap 192.168.80.89:5050 check inter 1s
   # ...
   ```
2. Use the `curl` CLI command to verify that the user-managed load balancer and its resources are operational:
   1. Verify that the cluster machine configuration API is accessible to the Kubernetes API server resource, by running the following command and observing the response:
      ```terminal
      $ curl https://<loadbalancer_ip_address>:6443/version --insecure
      ```

      If the configuration is correct, you receive a JSON object in response:

      ```json
      {
        "major": "1",
        "minor": "11+",
        "gitVersion": "v1.11.0+ad103ed",
        "gitCommit": "ad103ed",
        "gitTreeState": "clean",
        "buildDate": "2019-01-09T06:44:10Z",
        "goVersion": "go1.10.3",
        "compiler": "gc",
        "platform": "linux/amd64"
      }
      ```
   2. Verify that the cluster machine configuration API is accessible to the Machine config server resource, by running the following command and observing the output:
      ```terminal
      $ curl -v https://<loadbalancer_ip_address>:22623/healthz --insecure
      ```

      If the configuration is correct, the output from the command shows the following response:

      ```terminal
      HTTP/1.1 200 OK
      Content-Length: 0
      ```
   3. Verify that the controller is accessible to the Ingress Controller resource on port 80, by running the following command and observing the output:
      ```terminal
      $ curl -I -L -H "Host: console-openshift-console.apps.<cluster_name>.<base_domain>" http://<load_balancer_front_end_IP_address>
      ```

      If the configuration is correct, the output from the command shows the following response:

      ```terminal
      HTTP/1.1 302 Found
      content-length: 0
      location: https://console-openshift-console.apps.ocp4.private.opequon.net/
      cache-control: no-cache
      ```
   4. Verify that the controller is accessible to the Ingress Controller resource on port 443, by running the following command and observing the output:
      ```terminal
      $ curl -I -L --insecure --resolve console-openshift-console.apps.<cluster_name>.<base_domain>:443:<Load Balancer Front End IP Address> https://console-openshift-console.apps.<cluster_name>.<base_domain>
      ```

      If the configuration is correct, the output from the command shows the following response:

      ```terminal
      HTTP/1.1 200 OK
      referrer-policy: strict-origin-when-cross-origin
      set-cookie: csrf-token=UlYWOyQ62LWjw2h003xtYSKlh1a0Py2hhctw0WmV2YEdhJjFyQwWcGBsja261dGLgaYO0nxzVErhiXt6QepA7g==; Path=/; Secure; SameSite=Lax
      x-content-type-options: nosniff
      x-dns-prefetch-control: off
      x-frame-options: DENY
      x-xss-protection: 1; mode=block
      date: Wed, 04 Oct 2023 16:29:38 GMT
      content-type: text/html; charset=utf-8
      set-cookie: 1e2670d92730b515ce3a1bb65da45062=1bf5e9573c9a2760c964ed1659cc1673; path=/; HttpOnly; Secure; SameSite=None
      cache-control: private
      ```
3. Configure the DNS records for your cluster to target the front-end IP addresses of the user-managed load balancer. You must update records to your DNS server for the cluster API and applications over the load balancer. The following examples shows modified DNS records:
   ```dns
   <load_balancer_ip_address>  A  api.<cluster_name>.<base_domain>
   A record pointing to Load Balancer Front End
   ```

   ```dns
   <load_balancer_ip_address>   A apps.<cluster_name>.<base_domain>
   A record pointing to Load Balancer Front End
   ```

   :::warning

   DNS propagation might take some time for each DNS record to become available. Ensure that each DNS record propagates before validating each record.

   :::
4. For your OpenShift Container Platform cluster to use the user-managed load balancer, you must specify the following configuration in your cluster’s `install-config.yaml` file:
   ```yaml
   # ...
   platform:
     vsphere:
       loadBalancer:
         type: <loadBalancer_type>
       apiVIPs:
       - <api_ip>
       ingressVIPs:
       - <ingress_ip>
   # ...
   ```

   where:

   <dl>
   <dt><code>&lt;loadBalancer_type&gt;</code></dt>
   <dd>Specifies the load balancer type. Set to <code>UserManaged</code> to specify a user-managed load balancer for your cluster. The parameter defaults to <code>OpenShiftManagedDefault</code>, which denotes the default internal load balancer. For services defined in an <code>openshift-kni-infra</code> namespace, a user-managed load balancer can deploy the <code>coredns</code> service to pods in your cluster but ignores <code>keepalived</code> and <code>haproxy</code> services.</dd>
   <dt><code>&lt;api_ip&gt;</code></dt>
   <dd>Specifies the user-managed load balancer’s public IP address for the Kubernetes API. Mandatory parameter.</dd>
   <dt><code>&lt;ingress_ip&gt;</code></dt>
   <dd>Specifies the user-managed load balancer’s public IP address for ingress traffic. Mandatory parameter.</dd>
   </dl>

**Verification**

1. Use the `curl` CLI command to verify that the user-managed load balancer and DNS record configuration are operational:
   1. Verify that you can access the cluster API, by running the following command and observing the output:
      ```terminal
      $ curl https://api.<cluster_name>.<base_domain>:6443/version --insecure
      ```

      If the configuration is correct, you receive a JSON object in response:

      ```json
      {
        "major": "1",
        "minor": "11+",
        "gitVersion": "v1.11.0+ad103ed",
        "gitCommit": "ad103ed",
        "gitTreeState": "clean",
        "buildDate": "2019-01-09T06:44:10Z",
        "goVersion": "go1.10.3",
        "compiler": "gc",
        "platform": "linux/amd64"
        }
      ```
   2. Verify that you can access the cluster machine configuration, by running the following command and observing the output:
      ```terminal
      $ curl -v https://api.<cluster_name>.<base_domain>:22623/healthz --insecure
      ```

      If the configuration is correct, the output from the command shows the following response:

      ```terminal
      HTTP/1.1 200 OK
      Content-Length: 0
      ```
   3. Verify that you can access each cluster application on port 80, by running the following command and observing the output:
      ```terminal
      $ curl http://console-openshift-console.apps.<cluster_name>.<base_domain> -I -L --insecure
      ```

      If the configuration is correct, the output from the command shows the following response:

      ```terminal
      HTTP/1.1 302 Found
      content-length: 0
      location: https://console-openshift-console.apps.<cluster-name>.<base domain>/
      cache-control: no-cacheHTTP/1.1 200 OK
      referrer-policy: strict-origin-when-cross-origin
      set-cookie: csrf-token=39HoZgztDnzjJkq/JuLJMeoKNXlfiVv2YgZc09c3TBOBU4NI6kDXaJH1LdicNhN1UsQWzon4Dor9GWGfopaTEQ==; Path=/; Secure
      x-content-type-options: nosniff
      x-dns-prefetch-control: off
      x-frame-options: DENY
      x-xss-protection: 1; mode=block
      date: Tue, 17 Nov 2020 08:42:10 GMT
      content-type: text/html; charset=utf-8
      set-cookie: 1e2670d92730b515ce3a1bb65da45062=9b714eb87e93cf34853e87a92d6894be; path=/; HttpOnly; Secure; SameSite=None
      cache-control: private
      ```
   4. Verify that you can access each cluster application on port 443, by running the following command and observing the output:
      ```terminal
      $ curl https://console-openshift-console.apps.<cluster_name>.<base_domain> -I -L --insecure
      ```

      If the configuration is correct, the output from the command shows the following response:

      ```terminal
      HTTP/1.1 200 OK
      referrer-policy: strict-origin-when-cross-origin
      set-cookie: csrf-token=UlYWOyQ62LWjw2h003xtYSKlh1a0Py2hhctw0WmV2YEdhJjFyQwWcGBsja261dGLgaYO0nxzVErhiXt6QepA7g==; Path=/; Secure; SameSite=Lax
      x-content-type-options: nosniff
      x-dns-prefetch-control: off
      x-frame-options: DENY
      x-xss-protection: 1; mode=block
      date: Wed, 04 Oct 2023 16:29:38 GMT
      content-type: text/html; charset=utf-8
      set-cookie: 1e2670d92730b515ce3a1bb65da45062=1bf5e9573c9a2760c964ed1659cc1673; path=/; HttpOnly; Secure; SameSite=None
      cache-control: private
      ```

## Deploying the cluster {#installation-launching-installer_installing-restricted-networks-installer-provisioned-vsphere}

To deploy your OpenShift Container Platform cluster, you can initialize installation by running the `openshift-install create cluster` command from the directory that contains the installation program. The installation program provisions infrastructure and completes cluster setup.

:::warning

You can run the `create cluster` command of the installation program only once, during initial installation.

:::

**Prerequisites**

- You have the OpenShift Container Platform installation program and the pull secret for your cluster.
- You have verified that the cloud provider account on your host has the correct permissions to deploy the cluster. An account with incorrect permissions causes the installation process to fail with an error message that displays the missing permissions.
- Optional: Before you create the cluster, you configured an external load balancer in place of the default load balancer.
  :::warning

  You do not need to specify API and Ingress static addresses for your installation program. If you choose this configuration, you must take additional actions to define network targets that accept an IP address from each referenced vSphere subnet. See the section "Configuring a user-managed load balancer".

  :::

**Procedure**

- In the directory that contains the installation program, initialize the cluster deployment by running the following command:

```terminal
$ ./openshift-install create cluster --dir <installation_directory> \
    --log-level=info
```

- For `<installation_directory>`, specify the location of your customized `./install-config.yaml` file.
- To view different installation details, specify `warn`, `debug`, or `error` instead of `info`.

**Verification**

When the cluster deployment completes successfully:

- The terminal displays directions for accessing your cluster, including a link to the web console and credentials for the `kubeadmin` user.
- Credential information also outputs to `<installation_directory>/.openshift_install.log`.
  :::warning

  Do not delete the installation program or the files that the installation program creates. Both are required to delete the cluster.

  :::

  ```terminal title="Example output"
  ...
  INFO Install complete!
  INFO To access the cluster as the system:admin user when using 'oc', run 'export KUBECONFIG=/home/myuser/install_dir/auth/kubeconfig'
  INFO Access the OpenShift web-console here: https://console-openshift-console.apps.mycluster.example.com
  INFO Login to the console with user: "kubeadmin", and password: "password"
  INFO Time elapsed: 36m22s
  ```

  :::warning

  - The Ignition config files that the installation program generates contain certificates that expire after 24 hours, which are then renewed at that time. If the cluster is shut down before renewing the certificates and the cluster is later restarted after the 24 hours have elapsed, the cluster automatically recovers the expired certificates. The exception is that you must manually approve the pending `node-bootstrapper` certificate signing requests (CSRs) to recover kubelet certificates. See the documentation for *Recovering from expired control plane certificates* for more information.
  - It is recommended that you use Ignition config files within 12 hours after they are generated because the 24-hour certificate rotates from 16 to 22 hours after the cluster is installed. By using the Ignition config files within 12 hours, you can avoid installation failure if the certificate update runs during installation.

  :::

## Logging in to the cluster by using the CLI {#cli-logging-in-kubeadmin_installing-restricted-networks-installer-provisioned-vsphere}

To log in to your cluster as the default system user, export the `kubeconfig` file. This configuration enables the CLI to authenticate and connect to the specific API server created during OpenShift Container Platform installation.

The `kubeconfig` file is specific to a cluster and OpenShift Container Platform generates it during installation.

**Prerequisites**

- You deployed an OpenShift Container Platform cluster.
- You installed the OpenShift CLI (`oc`).

**Procedure**

1. Export the `kubeadmin` credentials by running the following command:
   ```terminal
   $ export KUBECONFIG=<installation_directory>/auth/kubeconfig
   ```

   where:

   <dl>
   <dt><code>&lt;installation_directory&gt;</code></dt>
   <dd>Specifies the path to the directory that stores the installation files.</dd>
   </dl>
2. Verify you can run `oc` commands successfully using the exported configuration by running the following command:
   ```terminal
   $ oc whoami
   ```

   ```terminal title="Example output"
   system:admin
   ```

**Next steps**

- "Customize your cluster"
- "Remote health reporting"

## Disabling the default software catalog sources {#olm-restricted-networks-operatorhub_installing-restricted-networks-installer-provisioned-vsphere}

To use only trusted or locally available Operator catalogs, disable the default software catalog sources that OpenShift Container Platform configures during installation. In a restricted network environment, you must disable the default catalogs as a cluster administrator.

**Procedure**

- Disable the sources for the default catalogs by adding `disableAllDefaultSources: true` to the `OperatorHub` object:
  ```terminal
  $ oc patch OperatorHub cluster --type json \
      -p '[{"op": "add", "path": "/spec/disableAllDefaultSources", "value": true}]'
  ```

  :::tip

  Or, you can use the web console to manage catalog sources. From the **Administration** → **Cluster Settings** → **Configuration** → **OperatorHub** page, click the **Sources** tab, where you can create, update, delete, disable, and enable individual sources.

  :::

## Image registry removed during installation {#registry-removed_installing-restricted-networks-installer-provisioned-vsphere}

On platforms that do not provide shareable object storage, the OpenShift Image Registry Operator bootstraps itself as `Removed`. This allows `openshift-installer` to complete installations on these platform types.

After installation, you must edit the Image Registry Operator configuration to switch the `managementState` from `Removed` to `Managed`. When this has completed, you must configure storage.

## Image registry storage configuration {#installation-registry-storage-config_installing-restricted-networks-installer-provisioned-vsphere}

The Image Registry Operator is not initially available for platforms that do not provide default storage. After installation, you must configure your registry to use storage so that the Registry Operator is made available.

Configure a persistent volume, which is required for production clusters. Where applicable, you can configure an empty directory as the storage location for non-production clusters.

You can also allow the image registry to use block storage types by using the `Recreate` rollout strategy during upgrades.

### Configuring registry storage for VMware vSphere {#registry-configuring-storage-vsphere_installing-restricted-networks-installer-provisioned-vsphere}

As a cluster administrator, following installation you must configure your registry to use storage.

**Prerequisites**

- Cluster administrator permissions.
- A cluster on VMware vSphere.
- Persistent storage provisioned for your cluster, such as Red Hat OpenShift Data Foundation.
  :::warning

  OpenShift Container Platform supports `ReadWriteOnce` access for image registry storage when you have only one replica. `ReadWriteOnce` access also requires that the registry uses the `Recreate` rollout strategy. To deploy an image registry that supports high availability with two or more replicas, `ReadWriteMany` access is required.

  :::
- Must have "100Gi" capacity.

:::warning

Testing shows issues with using the NFS server on RHEL as storage backend for core services. This includes the OpenShift Container Registry and Quay, Prometheus for monitoring storage, and Elasticsearch for logging storage. Therefore, using RHEL NFS to back PVs used by core services is not recommended.

Other NFS implementations on the marketplace might not have these issues. Contact the individual NFS implementation vendor for more information on any testing that was possibly completed against these OpenShift Container Platform core components.

:::

**Procedure**

1. Change the `spec.storage.pvc` field in the `configs.imageregistry/cluster` resource.
   :::note

   When you use shared storage, review your security settings to prevent outside access.

   :::
2. Verify that you do not have a registry pod by running the following command:
   ```terminal
   $ oc get pod -n openshift-image-registry -l docker-registry=default
   ```

   ```terminal title="Example output"
   No resourses found in openshift-image-registry namespace
   ```

   :::note

   If you do have a registry pod in your output, you do not need to continue with this procedure.

   :::
3. Check the registry configuration by running the following command:
   ```terminal
   $ oc edit configs.imageregistry.operator.openshift.io
   ```

   ```yaml title="Example output"
   storage:
     pvc:
       claim:
   ```

   Leave the `claim` field blank to allow the automatic creation of an `image-registry-storage` persistent volume claim (PVC). The PVC is generated based on the default storage class. However, be aware that the default storage class might provide ReadWriteOnce (RWO) volumes, such as a RADOS Block Device (RBD), which can cause issues when you replicate to more than one replica.
4. Check the `clusteroperator` status by running the following command:
   ```terminal
   $ oc get clusteroperator image-registry
   ```

   ```terminal title="Example output"
   NAME             VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE   MESSAGE
   image-registry   4.7       True        False         False      6h50m
   ```

## Telemetry access for OpenShift Container Platform {#cluster-telemetry_installing-restricted-networks-installer-provisioned-vsphere}

To provide metrics about cluster health and the success of updates, the Telemetry service requires internet access. When connected, this service runs automatically by default and registers your cluster to [OpenShift Cluster Manager](https://console.redhat.com/openshift).

After you confirm that your [OpenShift Cluster Manager](https://console.redhat.com/openshift) inventory is correct, either maintained automatically by Telemetry or manually by using OpenShift Cluster Manager,use subscription watch to track your OpenShift Container Platform subscriptions at the account or multi-cluster level. For more information about subscription watch, see "Data Gathered and Used by Red Hat’s subscription services" in the *Additional resources* section.

**Additional resources**

- [Preparing to install a cluster using installer-provisioned infrastructure](/docs/installing/installing_vsphere/ipi/ipi-vsphere-preparing-to-install#ipi-vsphere-preparing-to-install)
- [OpenShift Container Platform installation and update processes](/docs/architecture/architecture-installation#architecture-installation)
- [Selecting a cluster installation method and preparing it for users](/docs/installing/overview/installing-preparing#installing-preparing)
- [Creating a registry on your mirror host](/docs/disconnected/installing-mirroring-installation-images#installing-mirroring-installation-images)
- [Persistent storage](/docs/storage/understanding-persistent-storage#understanding-persistent-storage)
- [Configuring the firewall to allow required sites](/docs/installing/install_config/configuring-firewall#configuring-firewall-module_configuring-firewall)
- [About remote health monitoring](/docs/support/remote_health_monitoring/about-remote-health-monitoring#about-remote-health-monitoring)
- [Customize your cluster](/docs/installing/install_config/installing-customizing#installing-customizing)
- [Remote health reporting](/docs/support/remote_health_monitoring/remote-health-reporting#remote-health-reporting)
- [Registering your disconnected cluster](/docs/support/remote_health_monitoring/remote-health-reporting#insights-operator-register-disconnected-cluster_remote-health-reporting)
- [Set up your registry and configure registry storage](/docs/registry/configuring_registry_storage/configuring-registry-storage-vsphere#configuring-registry-storage-vsphere)
