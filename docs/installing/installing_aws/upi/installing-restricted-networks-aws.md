---
title: Installing a cluster on AWS in a disconnected environment with user-provisioned infrastructure
sidebar_position: 4
---

# Installing a cluster on AWS in a disconnected environment with user-provisioned infrastructure {#installing-restricted-networks-aws}

<a id="installing-restricted-networks-aws"></a>

In OpenShift Container Platform version 4.22, you can install a cluster on Amazon Web Services (AWS) using infrastructure that you provide and an internal mirror of the installation release content.

:::warning

While you can install an OpenShift Container Platform cluster by using mirrored installation release content, your cluster still requires internet access to use the AWS APIs.

:::

One way to create this infrastructure is to use the provided CloudFormation templates. You can change the templates to customize your infrastructure or use the information that they contain to create AWS objects according to your company’s policies.

:::warning

The steps for performing a user-provisioned infrastructure installation are an example only. Installing a cluster with infrastructure you provide requires knowledge of the cloud provider and the installation process of OpenShift Container Platform. Several CloudFormation templates help you complete these steps or model your own. You are also free to create the required resources through other methods; the templates are just an example.

:::

## Prerequisites {#aws-upi-restricted_prerequisites_installing-restricted-networks-aws}

Before you install OpenShift Container Platform on Amazon Web Services (AWS) in a restricted network, ensure that you have prepared your user-provisioned infrastructure, configured mirror registries for disconnected installation, and met all account and networking requirements.

The following list outlines the prerequisites to complete:

- You reviewed details about the OpenShift Container Platform installation and update processes.
- You read the documentation on selecting a cluster installation method and preparing it for users.
- You created a mirror registry on your mirror host and obtained the `imageContentSources` data for your version of OpenShift Container Platform.
  :::warning

  Because the installation media is on the mirror host, you can use that computer to complete all installation steps.

  :::
- You configured an AWS account to host the cluster.
  :::warning

  If you have an AWS profile stored on your computer, it must not use a temporary session token that you generated while using a multi-factor authentication device. The cluster continues to use your current AWS credentials to create AWS resources for the entire life of the cluster, so you must use key-based, long-term credentials. You must generate appropriate keys. You can supply the keys when you run the installation program.

  :::
- You prepared the user-provisioned infrastructure.
- You downloaded the AWS CLI and installed it on your computer.
- If you use a firewall and plan to use the Telemetry service, you configured the firewall to allow the sites that your cluster requires access to.
  :::note

  Be sure to also review this site list if you are configuring a proxy.

  :::
- If the cloud identity and access management (IAM) APIs are not accessible in your environment, or if you do not want to store an administrator-level credential secret in the `kube-system` namespace, you can manually create and maintain long-term credentials.

**Additional resources**

- [Installation and update](/docs/architecture/architecture-installation#architecture-installation)
- [Selecting a cluster installation method and preparing it for users](/docs/installing/overview/installing-preparing#installing-preparing)
- [Mirroring images for a disconnected installation](/docs/disconnected/installing-mirroring-installation-images#installing-mirroring-installation-images)
- [Configuring an AWS account](/docs/installing/installing_aws/installing-aws-account#installing-aws-account)
- [Managing access keys for IAM users (AWS documentation)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html)
- [Requirements for a cluster with user-provisioned infrastructure on AWS](/docs/installing/installing_aws/upi/upi-aws-installation-reqs#upi-aws-installation-reqs)
- [Install the AWS CLI Using the Bundled Installer (Linux, macOS, or UNIX) (AWS documentation)](https://docs.aws.amazon.com/cli/latest/userguide/install-bundle.html)
- [Configuring your firewall](/docs/installing/install_config/configuring-firewall#configuring-firewall)
- [Manually creating long-term credentials](/docs/installing/installing_aws/ipi/installing-aws-customizations#manually-create-iam_installing-aws-customizations)

## About installations in restricted networks {#installation-about-restricted-networks_installing-restricted-networks-aws}

You can install OpenShift Container Platform 4.22 in a restricted network without an active internet connection to obtain software components. Restricted network installations can use installer-provisioned or user-provisioned infrastructure, depending on the cloud platform to which you are installing the cluster.

If you choose to perform a restricted network installation on a cloud platform, you still require access to its cloud APIs. Some cloud functions, such as Amazon Web Service’s Route 53 DNS and IAM services, require internet access. Depending on your network, you might require less internet access for an installation on bare-metal hardware, Nutanix, or on VMware vSphere.

To complete a restricted network installation, you must create a registry that mirrors the contents of the OpenShift image registry and contains the installation media. You can create this registry on a mirror host, which can access both the internet and your closed network, or by using other methods that meet your restrictions.

:::warning

Because of the complexity of the configuration for user-provisioned installations, consider completing a standard user-provisioned infrastructure installation before you try a restricted network installation using user-provisioned infrastructure. Completing this test installation might make it easier to isolate and troubleshoot any issues that might arise during your installation in a restricted network.

:::

### Additional limits {#installation-restricted-network-limits_installing-restricted-networks-aws}

Clusters in restricted networks have the following additional limitations and restrictions:

- The `ClusterVersion` status includes an `Unable to retrieve available updates` error.
- By default, you cannot use the contents of the Developer Catalog because you cannot access the required image stream tags.

## Creating the installation files for AWS {#installation-user-infra-generate_installing-restricted-networks-aws}

To install OpenShift Container Platform on Amazon Web Services by using user-provisioned infrastructure, you must generate the files that the installation program needs to deploy your cluster and modify them so that the cluster creates only the machines that it will use.

You generate and customize the `install-config.yaml` file, Kubernetes manifests, and Ignition config files. You also have the option to first set up a separate `var` partition during the preparation phases of installation.

### Creating a separate `/var` partition {#installation-disk-partitioning-upi-templates_installing-restricted-networks-aws}

To isolate growing storage for containers, etcd, or logs, you can optionally create a separate `/var` partition on worker nodes before you generate Ignition configs.

It is recommended that disk partitioning for OpenShift Container Platform be left to the installation program. However, there are cases where you might want to create separate partitions in a part of the filesystem that you expect to grow.

OpenShift Container Platform supports the addition of a single partition to attach storage to either the `/var` partition or a subdirectory of `/var`. For example:

- `/var/lib/containers`: Holds container-related content that can grow as more images and containers are added to a system.
- `/var/lib/etcd`: Holds data that you might want to keep separate for purposes such as performance optimization of etcd storage.
- `/var`: Holds data that you might want to keep separate for purposes such as auditing.

Storing the contents of a `/var` directory separately makes it easier to grow storage for those areas as needed and reinstall OpenShift Container Platform at a later date and keep that data intact. With this method, you will not have to pull all your containers again, nor will you have to copy massive log files when you update systems.

Because `/var` must be in place before a fresh installation of Red Hat Enterprise Linux CoreOS (RHCOS), the following procedure sets up the separate `/var` partition by creating a machine config manifest that is inserted during the `openshift-install` preparation phases of an OpenShift Container Platform installation.

:::warning

If you follow the steps to create a separate `/var` partition in this procedure, it is not necessary to create the Kubernetes manifest and Ignition config files again as described later in this section.

:::

**Procedure**

1. Create a directory to hold the OpenShift Container Platform installation files:
   ```terminal
   $ mkdir $HOME/clusterconfig
   ```
2. Run `openshift-install` to create a set of files in the `manifest` and `openshift` subdirectories. Answer the system questions as you are prompted:
   ```terminal
   $ openshift-install create manifests --dir $HOME/clusterconfig
   ```

   ```terminal title="Example output"
   ? SSH Public Key ...
   INFO Credentials loaded from the "myprofile" profile in file "/home/myuser/.aws/credentials"
   INFO Consuming Install Config from target directory
   INFO Manifests created in: $HOME/clusterconfig/manifests and $HOME/clusterconfig/openshift
   ```
3. Optional: Confirm that the installation program created manifests in the `clusterconfig/openshift` directory:
   ```terminal
   $ ls $HOME/clusterconfig/openshift/
   ```

   ```terminal title="Example output"
   99_kubeadmin-password-secret.yaml
   99_openshift-cluster-api_master-machines-0.yaml
   99_openshift-cluster-api_master-machines-1.yaml
   99_openshift-cluster-api_master-machines-2.yaml
   ...
   ```
4. Create a Butane config that configures the additional partition. For example, name the file `$HOME/clusterconfig/98-var-partition.bu`, change the disk device name to the name of the storage device on the `worker` systems, and set the storage size as appropriate. This example places the `/var` directory on a separate partition:
   ```yaml
   variant: openshift
   version: 4.22.0
   metadata:
     labels:
       machineconfiguration.openshift.io/role: worker
     name: 98-var-partition
   storage:
     disks:
     - device: /dev/disk/by-id/<device_name>
       partitions:
       - label: var
         start_mib: <partition_start_offset>
         size_mib: <partition_size>
         number: 5
     filesystems:
       - device: /dev/disk/by-partlabel/var
         path: /var
         format: xfs
         mount_options: [defaults, prjquota]
         with_mount_unit: true
   ```

   where:

   <dl>
   <dt><code>&lt;device_name&gt;</code></dt>
   <dd>Specifies the storage device name of the disk that you want to partition.</dd>
   <dt><code>&lt;partition_start_offset&gt;</code></dt>
   <dd>Specifies the <code>start_mib</code> parameter. When adding a data partition to the boot disk, a minimum value of 25000 MiB (Mebibytes) is recommended. The root file system is automatically resized to fill all available space up to the specified offset. If no value is specified, or if the specified value is smaller than the recommended minimum, the resulting root file system will be too small, and future reinstalls of RHCOS might overwrite the beginning of the data partition.</dd>
   <dt><code>&lt;partition_size&gt;</code></dt>
   <dd>Specifies the size of the data partition in mebibytes.</dd>
   <dt><code>storage.filesystems.mount_options</code></dt>
   <dd>The <code>prjquota</code> mount option must be enabled for filesystems used for container storage.</dd>
   </dl>

   :::note

   When creating a separate `/var` partition, you cannot use different instance types for worker nodes, if the different instance types do not have the same device name.

   :::
5. Create a manifest from the Butane config and save it to the `clusterconfig/openshift` directory. For example, run the following command:
   ```terminal
   $ butane $HOME/clusterconfig/98-var-partition.bu -o $HOME/clusterconfig/openshift/98-var-partition.yaml
   ```
6. Run `openshift-install` again to create Ignition configs from a set of files in the `manifest` and `openshift` subdirectories:
   ```terminal
   $ openshift-install create ignition-configs --dir $HOME/clusterconfig
   ```

   ```terminal
   $ ls $HOME/clusterconfig/
   auth  bootstrap.ign  master.ign  metadata.json  worker.ign
   ```

   You can now use the Ignition config files as input to the installation procedures to install Red Hat Enterprise Linux CoreOS (RHCOS) systems.

### Creating the installation configuration file {#installation-generate-aws-user-infra-install-config_installing-restricted-networks-aws}

Generate and customize the installation configuration file that the installation program needs to deploy your cluster.

**Prerequisites**

- You obtained the OpenShift Container Platform installation program for user-provisioned infrastructure and the pull secret for your cluster. For a restricted network installation, these files are on your mirror host.
- You checked that you are deploying your cluster to an Amazon Web Services (AWS) Region with an accompanying Red Hat Enterprise Linux CoreOS (RHCOS) AMI published by Red Hat. If you are deploying to an AWS Region that requires a custom AMI, such as an AWS GovCloud Region, you must create the `install-config.yaml` file manually.

**Procedure**

1. Create the `install-config.yaml` file.
   1. Change to the directory that contains the installation program and run the following command:
      ```terminal
      $ ./openshift-install create install-config --dir <installation_directory>
      ```

      For `<installation_directory>`, specify the directory name to store the files that the installation program creates.

      :::warning

      Specify an empty directory. Some installation assets, such as bootstrap X.509 certificates have short expiration intervals, so you must not reuse an installation directory. If you want to reuse individual files from another cluster installation, you can copy them into your directory. However, the file names for the installation assets might change between releases. Use caution when copying installation files from an earlier OpenShift Container Platform version.

      :::
   2. At the prompts, provide the configuration details for your cloud:
      1. Optional: Select an SSH key to use to access your cluster machines.
         :::note

         For production OpenShift Container Platform clusters on which you want to perform installation debugging or disaster recovery, specify an SSH key that your `ssh-agent` process uses.

         :::
      2. Select **aws** as the platform to target.
      3. If you do not have an AWS profile stored on your computer, enter the AWS access key ID and secret access key for the user that you configured to run the installation program.
         :::note

         The AWS access key ID and secret access key are stored in `~/.aws/credentials` in the home directory of the current user on the installation host. You are prompted for the credentials by the installation program if the credentials for the exported profile are not present in the file. Any credentials that you provide to the installation program are stored in the file.

         :::
      4. Select the AWS Region to deploy the cluster to.
      5. Select the base domain for the Route 53 service that you configured for your cluster.
      6. Enter a descriptive name for your cluster.
      7. Paste the [pull secret from Red Hat OpenShift Cluster Manager](https://console.redhat.com/openshift/install/pull-secret).
2. Edit the `install-config.yaml` file to give the additional information that is required for an installation in a restricted network.
   1. Update the `pullSecret` value to contain the authentication information for your registry:
      ```yaml
      pullSecret: '{"auths":{"<local_registry>": {"auth": "<credentials>","email": "you@example.com"}}}'
      ```

      For `<local_registry>`, specify the registry domain name, and optionally the port, that your mirror registry uses to serve content. For example `registry.example.com` or `registry.example.com:5000`. For `<credentials>`, specify the base64-encoded user name and password for your mirror registry.
   2. Add the `additionalTrustBundle` parameter and value. The value must be the contents of the certificate file that you used for your mirror registry. The certificate file can be an existing, trusted certificate authority or the self-signed certificate that you generated for the mirror registry.
      ```yaml
      additionalTrustBundle: |
        -----BEGIN CERTIFICATE-----
        ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ
        -----END CERTIFICATE-----
      ```
   3. Add the image content resources:
      ```yaml
      imageContentSources:
      - mirrors:
        - <local_registry>/<local_repository_name>/release
        source: quay.io/openshift-release-dev/ocp-release
      - mirrors:
        - <local_registry>/<local_repository_name>/release
        source: quay.io/openshift-release-dev/ocp-v4.0-art-dev
      ```

      Use the `imageContentSources` section from the output of the command to mirror the repository or the values that you used when you mirrored the content from the media that you brought into your restricted network.
   4. Optional: Set the publishing strategy to `Internal`:
      ```yaml
      publish: Internal
      ```

      By setting this option, you create an internal Ingress Controller and a private load balancer.
3. Optional: Back up the `install-config.yaml` file.
   :::warning

   The `install-config.yaml` file is consumed during the installation process. If you want to reuse the file, you must back it up now.

   :::

**Additional resources**

- [Configuration and credential file settings (AWS documentation)](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)

### Configuring the cluster-wide proxy during installation {#installation-configure-proxy_installing-restricted-networks-aws}

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
     noProxy: ec2.<aws_region>.amazonaws.com,elasticloadbalancing.<aws_region>.amazonaws.com,s3.<aws_region>.amazonaws.com
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
   <dd>Specifies a comma-separated list of destination domain names, IP addresses, or other network CIDRs to exclude from proxying. Preface a domain with <code>.</code> to match subdomains only. For example, <code>.y.com</code> matches <code>x.y.com</code>, but not <code>y.com</code>. Use <code>*</code> to bypass the proxy for all destinations. If you have added the Amazon <code>EC2</code>, <code>Elastic Load Balancing</code>, and <code>S3</code> VPC endpoints to your VPC, you must add these endpoints to the <code>noProxy</code> field.</dd>
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

### Creating the Kubernetes manifest and Ignition config files {#installation-user-infra-generate-k8s-manifest-ignition_installing-restricted-networks-aws}

Because you manually provision infrastructure, you must generate the Kubernetes manifest and Ignition config files that the cluster requires.

The installation program converts the installation configuration into Kubernetes manifests and then wraps them into Ignition configuration files. You use these Ignition files to configure the cluster machines.

:::warning

- The Ignition config files that the OpenShift Container Platform installation program generates contain certificates that expire after 24 hours, which the system then renews. If you shut down the cluster before the system renews the certificates and you later restart the cluster after the 24 hours have elapsed, the cluster automatically recovers the expired certificates. The exception is that you must manually approve the pending `node-bootstrapper` certificate signing requests (CSRs) to recover kubelet certificates. See the documentation for *Recovering from expired control plane certificates* for more information.
- Use Ignition config files within 12 hours after you generate them, because the 24-hour certificate rotates from 16 to 22 hours after you install the cluster. By using the Ignition config files within 12 hours, you can avoid installation failure if the certificate update runs during installation.

:::

**Prerequisites**

- You obtained the OpenShift Container Platform installation program. For a restricted network installation, these files are on your mirror host.
- You created the `install-config.yaml` installation configuration file.

**Procedure**

1. Change to the directory that contains the OpenShift Container Platform installation program and generate the Kubernetes manifests for the cluster:
   ```terminal
   $ ./openshift-install create manifests --dir <installation_directory>
   ```

   where:

   <dl>
   <dt><code>&lt;installation_directory&gt;</code></dt>
   <dd>Specifies the installation directory that contains the <code>install-config.yaml</code> file you created.</dd>
   </dl>
2. Remove the Kubernetes manifest files that define the control plane machines:
   ```terminal
   $ rm -f <installation_directory>/openshift/99_openshift-cluster-api_master-machines-*.yaml
   ```

   By removing these files, you prevent the cluster from automatically generating control plane machines.
3. Remove the Kubernetes manifest files that define the control plane machine set:
   ```terminal
   $ rm -f <installation_directory>/openshift/99_openshift-machine-api_master-control-plane-machine-set.yaml
   ```
4. Remove the Kubernetes manifest files that define the worker machines:
   ```terminal
   $ rm -f <installation_directory>/openshift/99_openshift-cluster-api_worker-machineset-*.yaml
   ```

   :::warning

   If you disabled the `MachineAPI` capability when installing a cluster on user-provisioned infrastructure, you must remove the Kubernetes manifest files that define the worker machines. Otherwise, your cluster fails to install.

   :::

   Because you create and manage the worker machines yourself, you do not need to initialize these machines.
5. Verify that the `mastersSchedulable` parameter in the `<installation_directory>/manifests/cluster-scheduler-02-config.yml` Kubernetes manifest file is set to `false`. This setting prevents pods from being scheduled on the control plane machines:
   1. Open the `<installation_directory>/manifests/cluster-scheduler-02-config.yml` file.
   2. Locate the `mastersSchedulable` parameter and verify that it is set to `false`.
   3. Save and exit the file.
6. Optional: If you do not want [the Ingress Operator](https://github.com/openshift/cluster-ingress-operator) to create DNS records on your behalf, remove the `privateZone` and `publicZone` sections from the `<installation_directory>/manifests/cluster-dns-02-config.yml` DNS configuration file:
   ```yaml
   apiVersion: config.openshift.io/v1
   kind: DNS
   metadata:
     creationTimestamp: null
     name: cluster
   spec:
     baseDomain: example.openshift.com
     privateZone:
       id: mycluster-100419-private-zone
     publicZone:
       id: example.openshift.com
   status: {}
   ```

   `spec.privateZone`: Remove this section completely.

   If you do so, you must add ingress DNS records manually in a later step.
7. To create the Ignition configuration files, run the following command from the directory that contains the installation program:
   ```terminal
   $ ./openshift-install create ignition-configs --dir <installation_directory>
   ```

   where:

   <dl>
   <dt><code>&lt;installation_directory&gt;</code></dt>
   <dd>Specifies the same installation directory. The installation program creates Ignition config files for the bootstrap, control plane, and compute nodes in the installation directory. The program also creates the <code>kubeadmin-password</code> and <code>kubeconfig</code> files in the <code>./&lt;installation_directory&gt;/auth</code> directory:</dd>
   </dl>

   ```
   .
   ├── auth
   │   ├── kubeadmin-password
   │   └── kubeconfig
   ├── bootstrap.ign
   ├── master.ign
   ├── metadata.json
   └── worker.ign
   ```

**Additional resources**

- [Manually creating long-term credentials](/docs/installing/installing_aws/ipi/installing-restricted-networks-aws-installer-provisioned#manually-create-iam_installing-restricted-networks-aws-installer-provisioned)

## Extracting the infrastructure name {#installation-extracting-infraid_installing-restricted-networks-aws}

To identify your cluster resources in Amazon Web Services, extract the unique infrastructure name from the Ignition config files.

The Ignition config files contain a unique cluster identifier that you can use to uniquely identify your cluster in Amazon Web Services. The infrastructure name is also used to locate the appropriate AWS resources during an OpenShift Container Platform installation. The provided CloudFormation templates contain references to this infrastructure name, so you must extract it.

:::warning

Do not run the `openshift-install create manifests` command again after creating any Google Cloud resources. Running the command again generates a new cluster identifier, which will cause errors in existing resources. If you need to regenerate the manifests because you modified the `install-config.yaml` file, delete any Google Cloud resources you created and re-create them with the new cluster identifier.

:::

**Prerequisites**

- You obtained the OpenShift Container Platform installation program and the pull secret for your cluster.
- You generated the Ignition config files for your cluster.
- You installed the `jq` package.

**Procedure**

- To extract and view the infrastructure name from the Ignition config file metadata, run the following command:
  ```terminal
  $ jq -r .infraID <installation_directory>/metadata.json
  ```

  where `<installation_directory>` is the path to the directory that you stored the installation files in.

  ```terminal title="Example output"
  openshift-vw9j6
  ```

  The output of this command is your cluster name and a random string.

## Creating a VPC in AWS {#installation-creating-aws-vpc_installing-restricted-networks-aws}

To provide the network foundation for your OpenShift Container Platform cluster, create a Virtual Private Cloud (VPC) in Amazon Web Services (AWS) by using the provided CloudFormation template.

You can customize the VPC to meet your requirements, including VPN and route tables. You can use the provided CloudFormation template and a custom parameter file to create a stack of AWS resources that represent the VPC.

:::note

If you do not use the provided CloudFormation template to create your AWS infrastructure, you must review the provided information and manually create the infrastructure. If your cluster does not initialize correctly, you might have to contact Red Hat support with your installation logs.

:::

**Prerequisites**

- You added your AWS keys and region to your local AWS profile by running `aws configure`.

**Procedure**

1. Create a JSON file that contains the parameter values that the template requires:
   ```json
   [
     {
       "ParameterKey": "VpcCidr",
       "ParameterValue": "10.0.0.0/16"
     },
     {
       "ParameterKey": "AvailabilityZoneCount",
       "ParameterValue": "1"
     },
     {
       "ParameterKey": "SubnetBits",
       "ParameterValue": "12"
     }
   ]
   ```

   where:

   <dl>
   <dt><code>VpcCidr</code></dt>
   <dd>Specifies the CIDR block for the VPC in the format <code>x.x.x.x/16-24</code>.</dd>
   <dt><code>AvailabilityZoneCount</code></dt>
   <dd>Specifies the number of availability zones to deploy the VPC in. Set the value to an integer between <code>1</code> and <code>3</code>.</dd>
   <dt><code>SubnetBits</code></dt>
   <dd>Specifies the size of each subnet in each availability zone. Set the value to an integer between <code>5</code> and <code>13</code>, where <code>5</code> is <code>/27</code> and <code>13</code> is <code>/19</code>.</dd>
   </dl>
2. Copy the template from the **CloudFormation template for the VPC** section of this topic and save it as a YAML file on your computer. This template describes the VPC that your cluster requires.
3. Launch the CloudFormation template to create a stack of AWS resources that represent the VPC:
   :::warning

   You must enter the command on a single line.

   :::

   ```terminal
   $ aws cloudformation create-stack --stack-name <name> \
        --template-body file://<template>.yaml \
        --parameters file://<parameters>.json
   ```

   where:

   <dl>
   <dt><code>&lt;name&gt;</code></dt>
   <dd>Specifies the name for the CloudFormation stack, such as <code>cluster-vpc</code>. You need the name of this stack if you remove the cluster.</dd>
   <dt><code>&lt;template&gt;</code></dt>
   <dd>Specifies the relative path to and name of the CloudFormation template YAML file that you saved.</dd>
   <dt><code>&lt;parameters&gt;</code></dt>
   <dd>Specifies the relative path to and name of the CloudFormation parameters JSON file.</dd>
   </dl>

   ```terminal title="Example output"
   arn:aws:cloudformation:us-east-1:269333783861:stack/cluster-vpc/dbedae40-2fd3-11eb-820e-12a48460849f
   ```
4. Confirm that the template components exist:
   ```terminal
   $ aws cloudformation describe-stacks --stack-name <name>
   ```

   After the `StackStatus` displays `CREATE_COMPLETE`, the output displays values for the following parameters. You must provide these parameter values to the other CloudFormation templates that you run to create your cluster:

   <dl>
   <dt><code>VpcId</code></dt>
   <dd>The ID of your VPC.</dd>
   <dt><code>PublicSubnetIds</code></dt>
   <dd>The IDs of the new public subnets.</dd>
   <dt><code>PrivateSubnetIds</code></dt>
   <dd>The IDs of the new private subnets.</dd>
   </dl>

### CloudFormation template for the VPC {#installation-cloudformation-vpc_installing-restricted-networks-aws}

The VPC `CloudFormation` template creates the Amazon Web Services (AWS) networking infrastructure, including the public and private subnets, that your OpenShift Container Platform cluster requires.

```yaml title="CloudFormation template for the VPC"
AWSTemplateFormatVersion: 2010-09-09
Description: Template for Best Practice VPC with 1-3 AZs

Parameters:
  VpcCidr:
    AllowedPattern: ^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])(\/(1[6-9]|2[0-4]))$
    ConstraintDescription: CIDR block parameter must be in the form x.x.x.x/16-24.
    Default: 10.0.0.0/16
    Description: CIDR block for VPC.
    Type: String
  AvailabilityZoneCount:
    ConstraintDescription: "The number of availability zones. (Min: 1, Max: 3)"
    MinValue: 1
    MaxValue: 3
    Default: 1
    Description: "How many AZs to create VPC subnets for. (Min: 1, Max: 3)"
    Type: Number
  SubnetBits:
    ConstraintDescription: CIDR block parameter must be in the form x.x.x.x/19-27.
    MinValue: 5
    MaxValue: 13
    Default: 12
    Description: "Size of each subnet to create within the availability zones. (Min: 5 = /27, Max: 13 = /19)"
    Type: Number

Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
    - Label:
        default: "Network Configuration"
      Parameters:
      - VpcCidr
      - SubnetBits
    - Label:
        default: "Availability Zones"
      Parameters:
      - AvailabilityZoneCount
    ParameterLabels:
      AvailabilityZoneCount:
        default: "Availability Zone Count"
      VpcCidr:
        default: "VPC CIDR"
      SubnetBits:
        default: "Bits Per Subnet"

Conditions:
  DoAz3: !Equals [3, !Ref AvailabilityZoneCount]
  DoAz2: !Or [!Equals [2, !Ref AvailabilityZoneCount], Condition: DoAz3]

Resources:
  VPC:
    Type: "AWS::EC2::VPC"
    Properties:
      EnableDnsSupport: "true"
      EnableDnsHostnames: "true"
      CidrBlock: !Ref VpcCidr
  PublicSubnet:
    Type: "AWS::EC2::Subnet"
    Properties:
      VpcId: !Ref VPC
      CidrBlock: !Select [0, !Cidr [!Ref VpcCidr, 6, !Ref SubnetBits]]
      AvailabilityZone: !Select
      - 0
      - Fn::GetAZs: !Ref "AWS::Region"
  PublicSubnet2:
    Type: "AWS::EC2::Subnet"
    Condition: DoAz2
    Properties:
      VpcId: !Ref VPC
      CidrBlock: !Select [1, !Cidr [!Ref VpcCidr, 6, !Ref SubnetBits]]
      AvailabilityZone: !Select
      - 1
      - Fn::GetAZs: !Ref "AWS::Region"
  PublicSubnet3:
    Type: "AWS::EC2::Subnet"
    Condition: DoAz3
    Properties:
      VpcId: !Ref VPC
      CidrBlock: !Select [2, !Cidr [!Ref VpcCidr, 6, !Ref SubnetBits]]
      AvailabilityZone: !Select
      - 2
      - Fn::GetAZs: !Ref "AWS::Region"
  InternetGateway:
    Type: "AWS::EC2::InternetGateway"
  GatewayToInternet:
    Type: "AWS::EC2::VPCGatewayAttachment"
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway
  PublicRouteTable:
    Type: "AWS::EC2::RouteTable"
    Properties:
      VpcId: !Ref VPC
  PublicRoute:
    Type: "AWS::EC2::Route"
    DependsOn: GatewayToInternet
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway
  PublicSubnetRouteTableAssociation:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Properties:
      SubnetId: !Ref PublicSubnet
      RouteTableId: !Ref PublicRouteTable
  PublicSubnetRouteTableAssociation2:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Condition: DoAz2
    Properties:
      SubnetId: !Ref PublicSubnet2
      RouteTableId: !Ref PublicRouteTable
  PublicSubnetRouteTableAssociation3:
    Condition: DoAz3
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Properties:
      SubnetId: !Ref PublicSubnet3
      RouteTableId: !Ref PublicRouteTable
  PrivateSubnet:
    Type: "AWS::EC2::Subnet"
    Properties:
      VpcId: !Ref VPC
      CidrBlock: !Select [3, !Cidr [!Ref VpcCidr, 6, !Ref SubnetBits]]
      AvailabilityZone: !Select
      - 0
      - Fn::GetAZs: !Ref "AWS::Region"
  PrivateRouteTable:
    Type: "AWS::EC2::RouteTable"
    Properties:
      VpcId: !Ref VPC
  PrivateSubnetRouteTableAssociation:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Properties:
      SubnetId: !Ref PrivateSubnet
      RouteTableId: !Ref PrivateRouteTable
  NAT:
    DependsOn:
    - GatewayToInternet
    Type: "AWS::EC2::NatGateway"
    Properties:
      AllocationId:
        "Fn::GetAtt":
        - EIP
        - AllocationId
      SubnetId: !Ref PublicSubnet
  EIP:
    Type: "AWS::EC2::EIP"
    Properties:
      Domain: vpc
  Route:
    Type: "AWS::EC2::Route"
    Properties:
      RouteTableId:
        Ref: PrivateRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId:
        Ref: NAT
  PrivateSubnet2:
    Type: "AWS::EC2::Subnet"
    Condition: DoAz2
    Properties:
      VpcId: !Ref VPC
      CidrBlock: !Select [4, !Cidr [!Ref VpcCidr, 6, !Ref SubnetBits]]
      AvailabilityZone: !Select
      - 1
      - Fn::GetAZs: !Ref "AWS::Region"
  PrivateRouteTable2:
    Type: "AWS::EC2::RouteTable"
    Condition: DoAz2
    Properties:
      VpcId: !Ref VPC
  PrivateSubnetRouteTableAssociation2:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Condition: DoAz2
    Properties:
      SubnetId: !Ref PrivateSubnet2
      RouteTableId: !Ref PrivateRouteTable2
  NAT2:
    DependsOn:
    - GatewayToInternet
    Type: "AWS::EC2::NatGateway"
    Condition: DoAz2
    Properties:
      AllocationId:
        "Fn::GetAtt":
        - EIP2
        - AllocationId
      SubnetId: !Ref PublicSubnet2
  EIP2:
    Type: "AWS::EC2::EIP"
    Condition: DoAz2
    Properties:
      Domain: vpc
  Route2:
    Type: "AWS::EC2::Route"
    Condition: DoAz2
    Properties:
      RouteTableId:
        Ref: PrivateRouteTable2
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId:
        Ref: NAT2
  PrivateSubnet3:
    Type: "AWS::EC2::Subnet"
    Condition: DoAz3
    Properties:
      VpcId: !Ref VPC
      CidrBlock: !Select [5, !Cidr [!Ref VpcCidr, 6, !Ref SubnetBits]]
      AvailabilityZone: !Select
      - 2
      - Fn::GetAZs: !Ref "AWS::Region"
  PrivateRouteTable3:
    Type: "AWS::EC2::RouteTable"
    Condition: DoAz3
    Properties:
      VpcId: !Ref VPC
  PrivateSubnetRouteTableAssociation3:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Condition: DoAz3
    Properties:
      SubnetId: !Ref PrivateSubnet3
      RouteTableId: !Ref PrivateRouteTable3
  NAT3:
    DependsOn:
    - GatewayToInternet
    Type: "AWS::EC2::NatGateway"
    Condition: DoAz3
    Properties:
      AllocationId:
        "Fn::GetAtt":
        - EIP3
        - AllocationId
      SubnetId: !Ref PublicSubnet3
  EIP3:
    Type: "AWS::EC2::EIP"
    Condition: DoAz3
    Properties:
      Domain: vpc
  Route3:
    Type: "AWS::EC2::Route"
    Condition: DoAz3
    Properties:
      RouteTableId:
        Ref: PrivateRouteTable3
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId:
        Ref: NAT3
  S3Endpoint:
    Type: AWS::EC2::VPCEndpoint
    Properties:
      PolicyDocument:
        Version: 2012-10-17
        Statement:
        - Effect: Allow
          Principal: '*'
          Action:
          - '*'
          Resource:
          - '*'
      RouteTableIds:
      - !Ref PublicRouteTable
      - !Ref PrivateRouteTable
      - !If [DoAz2, !Ref PrivateRouteTable2, !Ref "AWS::NoValue"]
      - !If [DoAz3, !Ref PrivateRouteTable3, !Ref "AWS::NoValue"]
      ServiceName: !Join
      - ''
      - - com.amazonaws.
        - !Ref 'AWS::Region'
        - .s3
      VpcId: !Ref VPC

Outputs:
  VpcId:
    Description: ID of the new VPC.
    Value: !Ref VPC
  PublicSubnetIds:
    Description: Subnet IDs of the public subnets.
    Value:
      !Join [
        ",",
        [!Ref PublicSubnet, !If [DoAz2, !Ref PublicSubnet2, !Ref "AWS::NoValue"], !If [DoAz3, !Ref PublicSubnet3, !Ref "AWS::NoValue"]]
      ]
  PrivateSubnetIds:
    Description: Subnet IDs of the private subnets.
    Value:
      !Join [
        ",",
        [!Ref PrivateSubnet, !If [DoAz2, !Ref PrivateSubnet2, !Ref "AWS::NoValue"], !If [DoAz3, !Ref PrivateSubnet3, !Ref "AWS::NoValue"]]
      ]
  PublicRouteTableId:
    Description: Public Route table ID
    Value: !Ref PublicRouteTable
  PrivateRouteTableIds:
    Description: Private Route table IDs
    Value:
      !Join [
        ",",
        [
          !Join ["=", [
            !Select [0, "Fn::GetAZs": !Ref "AWS::Region"],
            !Ref PrivateRouteTable
          ]],
          !If [DoAz2,
               !Join ["=", [!Select [1, "Fn::GetAZs": !Ref "AWS::Region"], !Ref PrivateRouteTable2]],
               !Ref "AWS::NoValue"
          ],
          !If [DoAz3,
               !Join ["=", [!Select [2, "Fn::GetAZs": !Ref "AWS::Region"], !Ref PrivateRouteTable3]],
               !Ref "AWS::NoValue"
          ]
        ]
      ]
```

## Creating networking and load balancing components in AWS {#installation-creating-aws-dns_installing-restricted-networks-aws}

To route traffic to your OpenShift Container Platform cluster, configure the networking and load balancing components in Amazon Web Services (AWS) by using the provided `CloudFormation` template.

You can use the provided `CloudFormation` template and a custom parameter file to create a stack of AWS resources. The stack represents the networking and load balancing components that your OpenShift Container Platform cluster requires. The template also creates a hosted zone and subnet tags.

You can run the template many times within a single Virtual Private Cloud (VPC).

:::note

If you do not use the provided `CloudFormation` template to create your AWS infrastructure, you must review the provided information and manually create the infrastructure. If your cluster does not initialize correctly, you might have to contact Red Hat support with your installation logs.

:::

**Prerequisites**

- You created and configured a VPC and associated subnets in AWS.

**Procedure**

1. Obtain the hosted zone ID for the Route 53 base domain that you specified in the `install-config.yaml` file for your cluster. You can obtain details about your hosted zone by running the following command:
   ```terminal
   $ aws route53 list-hosted-zones-by-name --dns-name <route53_domain>
   ```

   where `<route53_domain>` is the Route 53 base domain that you used when you generated the `install-config.yaml` file for the cluster.

   ```terminal title="Example output"
   mycluster.example.com.	False	100
   HOSTEDZONES	65F8F38E-2268-B835-E15C-AB55336FCBFA	/hostedzone/Z21IXYZABCZ2A4	mycluster.example.com.	10
   ```

   In the example output, the hosted zone ID is `Z21IXYZABCZ2A4`.
2. Create a JSON file that has the parameter values that the template requires:
   ```json
   [
     {
       "ParameterKey": "ClusterName",
       "ParameterValue": "mycluster"
     },
     {
       "ParameterKey": "InfrastructureName",
       "ParameterValue": "mycluster-<random_string>"
     },
     {
       "ParameterKey": "HostedZoneId",
       "ParameterValue": "<random_string>"
     },
     {
       "ParameterKey": "HostedZoneName",
       "ParameterValue": "example.com"
     },
     {
       "ParameterKey": "PublicSubnets",
       "ParameterValue": "subnet-<random_string>"
     },
     {
       "ParameterKey": "PrivateSubnets",
       "ParameterValue": "subnet-<random_string>"
     },
     {
       "ParameterKey": "VpcId",
       "ParameterValue": "vpc-<random_string>"
     }
   ]
   ```

   where:

   <dl>
   <dt><code>ClusterName</code></dt>
   <dd>Specifies a short, representative cluster name to use for hostnames, and so on. Set the value to the cluster name that you used when you generated the <code>install-config.yaml</code> file for the cluster.</dd>
   <dt><code>InfrastructureName</code></dt>
   <dd>Specifies the name for your cluster infrastructure that your Ignition config files encode for the cluster. Set the value to the infrastructure name that you extracted from the Ignition config file metadata, which has the format <code>&lt;cluster_name&gt;-&lt;random_string&gt;</code>.</dd>
   <dt><code>HostedZoneId</code></dt>
   <dd>Specifies the Route 53 public zone ID to register the targets with. Set the value to the Route 53 public zone ID, which has a format similar to <code>Z21IXYZABCZ2A4</code>. You can obtain this value from the AWS console.</dd>
   <dt><code>HostedZoneName</code></dt>
   <dd>Specifies the Route 53 zone to register the targets with. Set the value to the Route 53 base domain that you used when you generated the <code>install-config.yaml</code> file for the cluster. Do not include the trailing period (.) that is displayed in the AWS console.</dd>
   <dt><code>PublicSubnets</code></dt>
   <dd>Specifies the public subnets that you created for your VPC. Set the value to the <code>PublicSubnetIds</code> value from the output of the <code>CloudFormation</code> template for the VPC.</dd>
   <dt><code>PrivateSubnets</code></dt>
   <dd>Specifies the private subnets that you created for your VPC. Set the value to the <code>PrivateSubnetIds</code> value from the output of the <code>CloudFormation</code> template for the VPC.</dd>
   <dt><code>VpcId</code></dt>
   <dd>Specifies the VPC that you created for the cluster. Set the value to the <code>VpcId</code> value from the output of the <code>CloudFormation</code> template for the VPC.</dd>
   </dl>
3. Copy the template from the `CloudFormation`** template for the network and load balancers** section and save it as a YAML file on your computer. This template describes the networking and load balancing objects that your cluster requires.
   :::warning

   If you are deploying your cluster to an AWS government or secret region, you must update the `InternalApiServerRecord` in the `CloudFormation` template to use `CNAME` records. Records of type `ALIAS` are not supported for AWS government regions.

   :::
4. Launch the `CloudFormation` template to create a stack of AWS resources for the networking and load balancing components:
   :::warning

   You must enter the command on a single line.

   :::

   ```terminal
   $ aws cloudformation create-stack --stack-name <name> \
        --template-body file://<template>.yaml \
        --parameters file://<parameters>.json \
        --capabilities CAPABILITY_NAMED_IAM
   ```

   where:

   <dl>
   <dt><code>&lt;name&gt;</code></dt>
   <dd>Specifies the name for the <code>CloudFormation</code> stack, such as <code>cluster-dns</code>. You need the name of this stack if you remove the cluster.</dd>
   <dt><code>&lt;template&gt;</code></dt>
   <dd>Specifies the relative path to and name of the <code>CloudFormation</code> template YAML file that you saved.</dd>
   <dt><code>&lt;parameters&gt;</code></dt>
   <dd>Specifies the relative path to and name of the <code>CloudFormation</code> parameters JSON file.</dd>
   <dt><code>CAPABILITY_NAMED_IAM</code></dt>
   <dd>You must explicitly declare this capability because the provided template creates some <code>AWS::IAM::Role</code> resources.</dd>
   </dl>

   ```terminal title="Example output"
   arn:aws:cloudformation:us-east-1:269333783861:stack/cluster-dns/cd3e5de0-2fd4-11eb-5cf0-12be5c33a183
   ```
5. Confirm that the template components exist:
   ```terminal
   $ aws cloudformation describe-stacks --stack-name <name>
   ```

   After the `StackStatus` displays `CREATE_COMPLETE`, the output displays values for the following parameters. You must give these parameter values to the other `CloudFormation` templates that you run to create your cluster:

   <dl>
   <dt><code>PrivateHostedZoneId</code></dt>
   <dd>Hosted zone ID for the private DNS.</dd>
   <dt><code>ExternalApiLoadBalancerName</code></dt>
   <dd>Full name of the external API load balancer.</dd>
   <dt><code>InternalApiLoadBalancerName</code></dt>
   <dd>Full name of the internal API load balancer.</dd>
   <dt><code>ApiServerDnsName</code></dt>
   <dd>Full hostname of the API server.</dd>
   <dt><code>RegisterNlbIpTargetsLambda</code></dt>
   <dd>Lambda ARN useful to help register and unregister IP targets for these load balancers.</dd>
   <dt><code>ExternalApiTargetGroupArn</code></dt>
   <dd>ARN of external API target group.</dd>
   <dt><code>InternalApiTargetGroupArn</code></dt>
   <dd>ARN of internal API target group.</dd>
   <dt><code>InternalServiceTargetGroupArn</code></dt>
   <dd>ARN of internal service target group.</dd>
   </dl>

### CloudFormation template for the network and load balancers {#installation-cloudformation-dns_installing-restricted-networks-aws}

The networking `CloudFormation` template creates the Route 53 DNS entries and load balancers on Amazon Web Services (AWS) that route traffic to your OpenShift Container Platform control plane and applications.

```yaml title="CloudFormation template for the network and load balancers"
AWSTemplateFormatVersion: 2010-09-09
Description: Template for OpenShift Cluster Network Elements (Route53 & LBs)

Parameters:
  ClusterName:
    AllowedPattern: ^([a-zA-Z][a-zA-Z0-9\-]{0,26})$
    MaxLength: 27
    MinLength: 1
    ConstraintDescription: Cluster name must be alphanumeric, start with a letter, and have a maximum of 27 characters.
    Description: A short, representative cluster name to use for host names and other identifying names.
    Type: String
  InfrastructureName:
    AllowedPattern: ^([a-zA-Z][a-zA-Z0-9\-]{0,26})$
    MaxLength: 27
    MinLength: 1
    ConstraintDescription: Infrastructure name must be alphanumeric, start with a letter, and have a maximum of 27 characters.
    Description: A short, unique cluster ID used to tag cloud resources and identify items owned or used by the cluster.
    Type: String
  HostedZoneId:
    Description: The Route53 public zone ID to register the targets with, such as Z21IXYZABCZ2A4.
    Type: String
  HostedZoneName:
    Description: The Route53 zone to register the targets with, such as example.com. Omit the trailing period.
    Type: String
    Default: "example.com"
  PublicSubnets:
    Description: The internet-facing subnets.
    Type: List<AWS::EC2::Subnet::Id>
  PrivateSubnets:
    Description: The internal subnets.
    Type: List<AWS::EC2::Subnet::Id>
  VpcId:
    Description: The VPC-scoped resources will belong to this VPC.
    Type: AWS::EC2::VPC::Id

Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
    - Label:
        default: "Cluster Information"
      Parameters:
      - ClusterName
      - InfrastructureName
    - Label:
        default: "Network Configuration"
      Parameters:
      - VpcId
      - PublicSubnets
      - PrivateSubnets
    - Label:
        default: "DNS"
      Parameters:
      - HostedZoneName
      - HostedZoneId
    ParameterLabels:
      ClusterName:
        default: "Cluster Name"
      InfrastructureName:
        default: "Infrastructure Name"
      VpcId:
        default: "VPC ID"
      PublicSubnets:
        default: "Public Subnets"
      PrivateSubnets:
        default: "Private Subnets"
      HostedZoneName:
        default: "Public Hosted Zone Name"
      HostedZoneId:
        default: "Public Hosted Zone ID"

Resources:
  ExtApiElb:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Name: !Join ["-", [!Ref InfrastructureName, "ext"]]
      IpAddressType: ipv4
      Subnets: !Ref PublicSubnets
      Type: network

  IntApiElb:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Name: !Join ["-", [!Ref InfrastructureName, "int"]]
      Scheme: internal
      IpAddressType: ipv4
      Subnets: !Ref PrivateSubnets
      Type: network

  IntDns:
    Type: "AWS::Route53::HostedZone"
    Properties:
      HostedZoneConfig:
        Comment: "Managed by CloudFormation"
      Name: !Join [".", [!Ref ClusterName, !Ref HostedZoneName]]
      HostedZoneTags:
      - Key: Name
        Value: !Join ["-", [!Ref InfrastructureName, "int"]]
      - Key: !Join ["", ["kubernetes.io/cluster/", !Ref InfrastructureName]]
        Value: "owned"
      VPCs:
      - VPCId: !Ref VpcId
        VPCRegion: !Ref "AWS::Region"

  ExternalApiServerRecord:
    Type: AWS::Route53::RecordSetGroup
    Properties:
      Comment: Alias record for the API server
      HostedZoneId: !Ref HostedZoneId
      RecordSets:
      - Name:
          !Join [
            ".",
            ["api", !Ref ClusterName, !Join ["", [!Ref HostedZoneName, "."]]],
          ]
        Type: A
        AliasTarget:
          HostedZoneId: !GetAtt ExtApiElb.CanonicalHostedZoneID
          DNSName: !GetAtt ExtApiElb.DNSName

  InternalApiServerRecord:
    Type: AWS::Route53::RecordSetGroup
    Properties:
      Comment: Alias record for the API server
      HostedZoneId: !Ref IntDns
      RecordSets:
      - Name:
          !Join [
            ".",
            ["api", !Ref ClusterName, !Join ["", [!Ref HostedZoneName, "."]]],
          ]
        Type: A
        AliasTarget:
          HostedZoneId: !GetAtt IntApiElb.CanonicalHostedZoneID
          DNSName: !GetAtt IntApiElb.DNSName
      - Name:
          !Join [
            ".",
            ["api-int", !Ref ClusterName, !Join ["", [!Ref HostedZoneName, "."]]],
          ]
        Type: A
        AliasTarget:
          HostedZoneId: !GetAtt IntApiElb.CanonicalHostedZoneID
          DNSName: !GetAtt IntApiElb.DNSName

  ExternalApiListener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      DefaultActions:
      - Type: forward
        TargetGroupArn:
          Ref: ExternalApiTargetGroup
      LoadBalancerArn:
        Ref: ExtApiElb
      Port: 6443
      Protocol: TCP

  ExternalApiTargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      HealthCheckIntervalSeconds: 10
      HealthCheckPath: "/readyz"
      HealthCheckPort: 6443
      HealthCheckProtocol: HTTPS
      HealthyThresholdCount: 2
      UnhealthyThresholdCount: 2
      Port: 6443
      Protocol: TCP
      TargetType: ip
      VpcId:
        Ref: VpcId
      TargetGroupAttributes:
      - Key: deregistration_delay.timeout_seconds
        Value: 60

  InternalApiListener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      DefaultActions:
      - Type: forward
        TargetGroupArn:
          Ref: InternalApiTargetGroup
      LoadBalancerArn:
        Ref: IntApiElb
      Port: 6443
      Protocol: TCP

  InternalApiTargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      HealthCheckIntervalSeconds: 10
      HealthCheckPath: "/readyz"
      HealthCheckPort: 6443
      HealthCheckProtocol: HTTPS
      HealthyThresholdCount: 2
      UnhealthyThresholdCount: 2
      Port: 6443
      Protocol: TCP
      TargetType: ip
      VpcId:
        Ref: VpcId
      TargetGroupAttributes:
      - Key: deregistration_delay.timeout_seconds
        Value: 60

  InternalServiceInternalListener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      DefaultActions:
      - Type: forward
        TargetGroupArn:
          Ref: InternalServiceTargetGroup
      LoadBalancerArn:
        Ref: IntApiElb
      Port: 22623
      Protocol: TCP

  InternalServiceTargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      HealthCheckIntervalSeconds: 10
      HealthCheckPath: "/healthz"
      HealthCheckPort: 22623
      HealthCheckProtocol: HTTPS
      HealthyThresholdCount: 2
      UnhealthyThresholdCount: 2
      Port: 22623
      Protocol: TCP
      TargetType: ip
      VpcId:
        Ref: VpcId
      TargetGroupAttributes:
      - Key: deregistration_delay.timeout_seconds
        Value: 60

  RegisterTargetLambdaIamRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: !Join ["-", [!Ref InfrastructureName, "nlb", "lambda", "role"]]
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
        - Effect: "Allow"
          Principal:
            Service:
            - "lambda.amazonaws.com"
          Action:
          - "sts:AssumeRole"
      Path: "/"
      Policies:
      - PolicyName: !Join ["-", [!Ref InfrastructureName, "master", "policy"]]
        PolicyDocument:
          Version: "2012-10-17"
          Statement:
          - Effect: "Allow"
            Action:
              [
                "elasticloadbalancing:RegisterTargets",
                "elasticloadbalancing:DeregisterTargets",
              ]
            Resource: !Ref InternalApiTargetGroup
          - Effect: "Allow"
            Action:
              [
                "elasticloadbalancing:RegisterTargets",
                "elasticloadbalancing:DeregisterTargets",
              ]
            Resource: !Ref InternalServiceTargetGroup
          - Effect: "Allow"
            Action:
              [
                "elasticloadbalancing:RegisterTargets",
                "elasticloadbalancing:DeregisterTargets",
              ]
            Resource: !Ref ExternalApiTargetGroup

  RegisterNlbIpTargets:
    Type: "AWS::Lambda::Function"
    Properties:
      Handler: "index.handler"
      Role:
        Fn::GetAtt:
        - "RegisterTargetLambdaIamRole"
        - "Arn"
      Code:
        ZipFile: |
          import json
          import boto3
          import cfnresponse
          def handler(event, context):
            elb = boto3.client('elbv2')
            if event['RequestType'] == 'Delete':
              elb.deregister_targets(TargetGroupArn=event['ResourceProperties']['TargetArn'],Targets=[{'Id': event['ResourceProperties']['TargetIp']}])
            elif event['RequestType'] == 'Create':
              elb.register_targets(TargetGroupArn=event['ResourceProperties']['TargetArn'],Targets=[{'Id': event['ResourceProperties']['TargetIp']}])
            responseData = {}
            cfnresponse.send(event, context, cfnresponse.SUCCESS, responseData, event['ResourceProperties']['TargetArn']+event['ResourceProperties']['TargetIp'])
      Runtime: "python3.11"
      Timeout: 120

  RegisterSubnetTagsLambdaIamRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: !Join ["-", [!Ref InfrastructureName, "subnet-tags-lambda-role"]]
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
        - Effect: "Allow"
          Principal:
            Service:
            - "lambda.amazonaws.com"
          Action:
          - "sts:AssumeRole"
      Path: "/"
      Policies:
      - PolicyName: !Join ["-", [!Ref InfrastructureName, "subnet-tagging-policy"]]
        PolicyDocument:
          Version: "2012-10-17"
          Statement:
          - Effect: "Allow"
            Action:
              [
                "ec2:DeleteTags",
                "ec2:CreateTags"
              ]
            Resource: "arn:aws:ec2:*:*:subnet/*"
          - Effect: "Allow"
            Action:
              [
                "ec2:DescribeSubnets",
                "ec2:DescribeTags"
              ]
            Resource: "*"

  RegisterSubnetTags:
    Type: "AWS::Lambda::Function"
    Properties:
      Handler: "index.handler"
      Role:
        Fn::GetAtt:
        - "RegisterSubnetTagsLambdaIamRole"
        - "Arn"
      Code:
        ZipFile: |
          import json
          import boto3
          import cfnresponse
          def handler(event, context):
            ec2_client = boto3.client('ec2')
            if event['RequestType'] == 'Delete':
              for subnet_id in event['ResourceProperties']['Subnets']:
                ec2_client.delete_tags(Resources=[subnet_id], Tags=[{'Key': 'kubernetes.io/cluster/' + event['ResourceProperties']['InfrastructureName']}]);
            elif event['RequestType'] == 'Create':
              for subnet_id in event['ResourceProperties']['Subnets']:
                ec2_client.create_tags(Resources=[subnet_id], Tags=[{'Key': 'kubernetes.io/cluster/' + event['ResourceProperties']['InfrastructureName'], 'Value': 'shared'}]);
            responseData = {}
            cfnresponse.send(event, context, cfnresponse.SUCCESS, responseData, event['ResourceProperties']['InfrastructureName']+event['ResourceProperties']['Subnets'][0])
      Runtime: "python3.11"
      Timeout: 120

  RegisterPublicSubnetTags:
    Type: Custom::SubnetRegister
    Properties:
      ServiceToken: !GetAtt RegisterSubnetTags.Arn
      InfrastructureName: !Ref InfrastructureName
      Subnets: !Ref PublicSubnets

  RegisterPrivateSubnetTags:
    Type: Custom::SubnetRegister
    Properties:
      ServiceToken: !GetAtt RegisterSubnetTags.Arn
      InfrastructureName: !Ref InfrastructureName
      Subnets: !Ref PrivateSubnets

Outputs:
  PrivateHostedZoneId:
    Description: Hosted zone ID for the private DNS, which is required for private records.
    Value: !Ref IntDns
  ExternalApiLoadBalancerName:
    Description: Full name of the external API load balancer.
    Value: !GetAtt ExtApiElb.LoadBalancerFullName
  InternalApiLoadBalancerName:
    Description: Full name of the internal API load balancer.
    Value: !GetAtt IntApiElb.LoadBalancerFullName
  ApiServerDnsName:
    Description: Full hostname of the API server, which is required for the Ignition config files.
    Value: !Join [".", ["api-int", !Ref ClusterName, !Ref HostedZoneName]]
  RegisterNlbIpTargetsLambda:
    Description: Lambda ARN useful to help register or deregister IP targets for these load balancers.
    Value: !GetAtt RegisterNlbIpTargets.Arn
  ExternalApiTargetGroupArn:
    Description: ARN of the external API target group.
    Value: !Ref ExternalApiTargetGroup
  InternalApiTargetGroupArn:
    Description: ARN of the internal API target group.
    Value: !Ref InternalApiTargetGroup
  InternalServiceTargetGroupArn:
    Description: ARN of the internal service target group.
    Value: !Ref InternalServiceTargetGroup
```

:::warning

If you are deploying your cluster to an AWS government or secret region, you must update the `InternalApiServerRecord` to use `CNAME` records. Records of type `ALIAS` are not supported for AWS government regions. For example:

```yaml
Type: CNAME
TTL: 10
ResourceRecords:
- !GetAtt IntApiElb.DNSName
```

:::

**Additional resources**

- [Listing public hosted zones(AWS documentation)](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/ListInfoOnHostedZone.html)

## Creating security group and roles in AWS {#installation-creating-aws-security_installing-restricted-networks-aws}

To control access to your OpenShift Container Platform cluster resources, create the required security groups and IAM roles in Amazon Web Services (AWS) by using the provided `CloudFormation` template.

You can use the provided `CloudFormation` template and a custom parameter file to create a stack of AWS resources. The stack represents the security groups and roles that your OpenShift Container Platform cluster requires.

:::note

If you do not use the provided `CloudFormation` template to create your AWS infrastructure, you must review the provided information and manually create the infrastructure. If your cluster does not initialize correctly, you might have to contact Red Hat support with your installation logs.

:::

**Procedure**

1. Create a JSON file that has the parameter values that the template requires:
   ```json
   [
     {
       "ParameterKey": "InfrastructureName",
       "ParameterValue": "mycluster-<random_string>"
     },
     {
       "ParameterKey": "VpcCidr",
       "ParameterValue": "10.0.0.0/16"
     },
     {
       "ParameterKey": "PrivateSubnets",
       "ParameterValue": "subnet-<random_string>"
     },
     {
       "ParameterKey": "VpcId",
       "ParameterValue": "vpc-<random_string>"
     }
   ]
   ```

   where:

   <dl>
   <dt><code>InfrastructureName</code></dt>
   <dd>Specifies the name for your cluster infrastructure that your Ignition config files encode for the cluster. Set the value to the infrastructure name that you extracted from the Ignition config file metadata, which has the format <code>&lt;cluster_name&gt;-&lt;random_string&gt;</code>.</dd>
   <dt><code>VpcCidr</code></dt>
   <dd>Specifies the CIDR block for the VPC. Set the value to the CIDR block parameter that you used for the VPC that you defined in the form <code>x.x.x.x/16-24</code>.</dd>
   <dt><code>PrivateSubnets</code></dt>
   <dd>Specifies the private subnets that you created for your VPC. Set the value to the <code>PrivateSubnetIds</code> value from the output of the <code>CloudFormation</code> template for the VPC.</dd>
   <dt><code>VpcId</code></dt>
   <dd>Specifies the VPC that you created for the cluster. Set the value to the <code>VpcId</code> value from the output of the <code>CloudFormation</code> template for the VPC.</dd>
   </dl>
2. Copy the template from the `CloudFormation`** template for security objects** section and save it as a YAML file on your computer. This template describes the security groups and roles that your cluster requires.
3. Launch the `CloudFormation` template to create a stack of AWS resources that represent the security groups and roles:
   :::warning

   You must enter the command on a single line.

   :::

   ```terminal
   $ aws cloudformation create-stack --stack-name <name> \
        --template-body file://<template>.yaml \
        --parameters file://<parameters>.json \
        --capabilities CAPABILITY_NAMED_IAM
   ```

   where:

   <dl>
   <dt><code>&lt;name&gt;</code></dt>
   <dd>Specifies the name for the <code>CloudFormation</code> stack, such as <code>cluster-sec</code>. You need the name of this stack if you remove the cluster.</dd>
   <dt><code>&lt;template&gt;</code></dt>
   <dd>Specifies the relative path to and name of the <code>CloudFormation</code> template YAML file that you saved.</dd>
   <dt><code>&lt;parameters&gt;</code></dt>
   <dd>Specifies the relative path to and name of the <code>CloudFormation</code> parameters JSON file.</dd>
   <dt><code>CAPABILITY_NAMED_IAM</code></dt>
   <dd>You must explicitly declare this capability because the provided template creates some <code>AWS::IAM::Role</code> and <code>AWS::IAM::InstanceProfile</code> resources.</dd>
   </dl>

   ```terminal title="Example output"
   arn:aws:cloudformation:us-east-1:269333783861:stack/cluster-sec/03bd4210-2ed7-11eb-6d7a-13fc0b61e9db
   ```
4. Confirm that the template components exist:
   ```terminal
   $ aws cloudformation describe-stacks --stack-name <name>
   ```

   After the `StackStatus` displays `CREATE_COMPLETE`, the output displays values for the following parameters. You must give these parameter values to the other `CloudFormation` templates that you run to create your cluster:

   <dl>
   <dt><code>MasterSecurityGroupId</code></dt>
   <dd>Control plane security group ID</dd>
   <dt><code>WorkerSecurityGroupId</code></dt>
   <dd>Worker security group ID</dd>
   <dt><code>MasterInstanceProfile</code></dt>
   <dd>Control plane IAM instance profile</dd>
   <dt><code>WorkerInstanceProfile</code></dt>
   <dd>Worker IAM instance profile</dd>
   </dl>

### CloudFormation template for security objects {#installation-cloudformation-security_installing-restricted-networks-aws}

The security `CloudFormation` template creates the IAM roles and security groups on Amazon Web Services (AWS) that control access to your OpenShift Container Platform cluster resources.

```yaml title="CloudFormation template for security objects"
AWSTemplateFormatVersion: 2010-09-09
Description: Template for OpenShift Cluster Security Elements (Security Groups & IAM)

Parameters:
  InfrastructureName:
    AllowedPattern: ^([a-zA-Z][a-zA-Z0-9\-]{0,26})$
    MaxLength: 27
    MinLength: 1
    ConstraintDescription: Infrastructure name must be alphanumeric, start with a letter, and have a maximum of 27 characters.
    Description: A short, unique cluster ID used to tag cloud resources and identify items owned or used by the cluster.
    Type: String
  VpcCidr:
    AllowedPattern: ^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])(\/(1[6-9]|2[0-4]))$
    ConstraintDescription: CIDR block parameter must be in the form x.x.x.x/16-24.
    Default: 10.0.0.0/16
    Description: CIDR block for VPC.
    Type: String
  VpcId:
    Description: The VPC-scoped resources will belong to this VPC.
    Type: AWS::EC2::VPC::Id
  PrivateSubnets:
    Description: The internal subnets.
    Type: List<AWS::EC2::Subnet::Id>

Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
    - Label:
        default: "Cluster Information"
      Parameters:
      - InfrastructureName
    - Label:
        default: "Network Configuration"
      Parameters:
      - VpcId
      - VpcCidr
      - PrivateSubnets
    ParameterLabels:
      InfrastructureName:
        default: "Infrastructure Name"
      VpcId:
        default: "VPC ID"
      VpcCidr:
        default: "VPC CIDR"
      PrivateSubnets:
        default: "Private Subnets"

Resources:
  MasterSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Cluster Master Security Group
      SecurityGroupIngress:
      - IpProtocol: icmp
        FromPort: 0
        ToPort: 0
        CidrIp: !Ref VpcCidr
      - IpProtocol: tcp
        FromPort: 22
        ToPort: 22
        CidrIp: !Ref VpcCidr
      - IpProtocol: tcp
        ToPort: 6443
        FromPort: 6443
        CidrIp: !Ref VpcCidr
      - IpProtocol: tcp
        FromPort: 22623
        ToPort: 22623
        CidrIp: !Ref VpcCidr
      VpcId: !Ref VpcId

  WorkerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Cluster Worker Security Group
      SecurityGroupIngress:
      - IpProtocol: icmp
        FromPort: 0
        ToPort: 0
        CidrIp: !Ref VpcCidr
      - IpProtocol: tcp
        FromPort: 22
        ToPort: 22
        CidrIp: !Ref VpcCidr
      VpcId: !Ref VpcId

  MasterIngressEtcd:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt MasterSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt MasterSecurityGroup.GroupId
      Description: etcd
      FromPort: 2379
      ToPort: 2380
      IpProtocol: tcp

  MasterIngressVxlan:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt MasterSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt MasterSecurityGroup.GroupId
      Description: Vxlan packets
      FromPort: 4789
      ToPort: 4789
      IpProtocol: udp

  MasterIngressWorkerVxlan:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt MasterSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt WorkerSecurityGroup.GroupId
      Description: Vxlan packets
      FromPort: 4789
      ToPort: 4789
      IpProtocol: udp

  MasterIngressGeneve:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt MasterSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt MasterSecurityGroup.GroupId
      Description: Geneve packets
      FromPort: 6081
      ToPort: 6081
      IpProtocol: udp

  MasterIngressWorkerGeneve:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt MasterSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt WorkerSecurityGroup.GroupId
      Description: Geneve packets
      FromPort: 6081
      ToPort: 6081
      IpProtocol: udp

  MasterIngressIpsecIke:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt MasterSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt MasterSecurityGroup.GroupId
      Description: IPsec IKE packets
      FromPort: 500
      ToPort: 500
      IpProtocol: udp

  MasterIngressIpsecNat:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt MasterSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt MasterSecurityGroup.GroupId
      Description: IPsec NAT-T packets
      FromPort: 4500
      ToPort: 4500
      IpProtocol: udp

  MasterIngressIpsecEsp:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt MasterSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt MasterSecurityGroup.GroupId
      Description: IPsec ESP packets
      IpProtocol: 50

  MasterIngressWorkerIpsecIke:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt MasterSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt WorkerSecurityGroup.GroupId
      Description: IPsec IKE packets
      FromPort: 500
      ToPort: 500
      IpProtocol: udp

  MasterIngressWorkerIpsecNat:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt MasterSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt WorkerSecurityGroup.GroupId
      Description: IPsec NAT-T packets
      FromPort: 4500
      ToPort: 4500
      IpProtocol: udp

  MasterIngressWorkerIpsecEsp:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt MasterSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt WorkerSecurityGroup.GroupId
      Description: IPsec ESP packets
      IpProtocol: 50

  MasterIngressInternal:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt MasterSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt MasterSecurityGroup.GroupId
      Description: Internal cluster communication
      FromPort: 9000
      ToPort: 9999
      IpProtocol: tcp

  MasterIngressWorkerInternal:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt MasterSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt WorkerSecurityGroup.GroupId
      Description: Internal cluster communication
      FromPort: 9000
      ToPort: 9999
      IpProtocol: tcp

  MasterIngressInternalUDP:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt MasterSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt MasterSecurityGroup.GroupId
      Description: Internal cluster communication
      FromPort: 9000
      ToPort: 9999
      IpProtocol: udp

  MasterIngressWorkerInternalUDP:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt MasterSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt WorkerSecurityGroup.GroupId
      Description: Internal cluster communication
      FromPort: 9000
      ToPort: 9999
      IpProtocol: udp

  MasterIngressKube:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt MasterSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt MasterSecurityGroup.GroupId
      Description: Kubernetes kubelet, scheduler and controller manager
      FromPort: 10250
      ToPort: 10259
      IpProtocol: tcp

  MasterIngressWorkerKube:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt MasterSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt WorkerSecurityGroup.GroupId
      Description: Kubernetes kubelet, scheduler and controller manager
      FromPort: 10250
      ToPort: 10259
      IpProtocol: tcp

  MasterIngressIngressServices:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt MasterSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt MasterSecurityGroup.GroupId
      Description: Kubernetes ingress services
      FromPort: 30000
      ToPort: 32767
      IpProtocol: tcp

  MasterIngressWorkerIngressServices:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt MasterSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt WorkerSecurityGroup.GroupId
      Description: Kubernetes ingress services
      FromPort: 30000
      ToPort: 32767
      IpProtocol: tcp

  MasterIngressIngressServicesUDP:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt MasterSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt MasterSecurityGroup.GroupId
      Description: Kubernetes ingress services
      FromPort: 30000
      ToPort: 32767
      IpProtocol: udp

  MasterIngressWorkerIngressServicesUDP:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt MasterSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt WorkerSecurityGroup.GroupId
      Description: Kubernetes ingress services
      FromPort: 30000
      ToPort: 32767
      IpProtocol: udp

  WorkerIngressVxlan:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt WorkerSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt WorkerSecurityGroup.GroupId
      Description: Vxlan packets
      FromPort: 4789
      ToPort: 4789
      IpProtocol: udp

  WorkerIngressMasterVxlan:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt WorkerSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt MasterSecurityGroup.GroupId
      Description: Vxlan packets
      FromPort: 4789
      ToPort: 4789
      IpProtocol: udp

  WorkerIngressGeneve:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt WorkerSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt WorkerSecurityGroup.GroupId
      Description: Geneve packets
      FromPort: 6081
      ToPort: 6081
      IpProtocol: udp

  WorkerIngressMasterGeneve:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt WorkerSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt MasterSecurityGroup.GroupId
      Description: Geneve packets
      FromPort: 6081
      ToPort: 6081
      IpProtocol: udp

  WorkerIngressIpsecIke:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt WorkerSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt WorkerSecurityGroup.GroupId
      Description: IPsec IKE packets
      FromPort: 500
      ToPort: 500
      IpProtocol: udp

  WorkerIngressIpsecNat:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt WorkerSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt WorkerSecurityGroup.GroupId
      Description: IPsec NAT-T packets
      FromPort: 4500
      ToPort: 4500
      IpProtocol: udp

  WorkerIngressIpsecEsp:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt WorkerSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt WorkerSecurityGroup.GroupId
      Description: IPsec ESP packets
      IpProtocol: 50

  WorkerIngressMasterIpsecIke:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt WorkerSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt MasterSecurityGroup.GroupId
      Description: IPsec IKE packets
      FromPort: 500
      ToPort: 500
      IpProtocol: udp

  WorkerIngressMasterIpsecNat:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt WorkerSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt MasterSecurityGroup.GroupId
      Description: IPsec NAT-T packets
      FromPort: 4500
      ToPort: 4500
      IpProtocol: udp

  WorkerIngressMasterIpsecEsp:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt WorkerSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt MasterSecurityGroup.GroupId
      Description: IPsec ESP packets
      IpProtocol: 50

  WorkerIngressInternal:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt WorkerSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt WorkerSecurityGroup.GroupId
      Description: Internal cluster communication
      FromPort: 9000
      ToPort: 9999
      IpProtocol: tcp

  WorkerIngressMasterInternal:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt WorkerSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt MasterSecurityGroup.GroupId
      Description: Internal cluster communication
      FromPort: 9000
      ToPort: 9999
      IpProtocol: tcp

  WorkerIngressInternalUDP:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt WorkerSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt WorkerSecurityGroup.GroupId
      Description: Internal cluster communication
      FromPort: 9000
      ToPort: 9999
      IpProtocol: udp

  WorkerIngressMasterInternalUDP:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt WorkerSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt MasterSecurityGroup.GroupId
      Description: Internal cluster communication
      FromPort: 9000
      ToPort: 9999
      IpProtocol: udp

  WorkerIngressKube:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt WorkerSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt WorkerSecurityGroup.GroupId
      Description: Kubernetes secure kubelet port
      FromPort: 10250
      ToPort: 10250
      IpProtocol: tcp

  WorkerIngressWorkerKube:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt WorkerSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt MasterSecurityGroup.GroupId
      Description: Internal Kubernetes communication
      FromPort: 10250
      ToPort: 10250
      IpProtocol: tcp

  WorkerIngressIngressServices:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt WorkerSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt WorkerSecurityGroup.GroupId
      Description: Kubernetes ingress services
      FromPort: 30000
      ToPort: 32767
      IpProtocol: tcp

  WorkerIngressMasterIngressServices:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt WorkerSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt MasterSecurityGroup.GroupId
      Description: Kubernetes ingress services
      FromPort: 30000
      ToPort: 32767
      IpProtocol: tcp

  WorkerIngressIngressServicesUDP:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt WorkerSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt WorkerSecurityGroup.GroupId
      Description: Kubernetes ingress services
      FromPort: 30000
      ToPort: 32767
      IpProtocol: udp

  WorkerIngressMasterIngressServicesUDP:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !GetAtt WorkerSecurityGroup.GroupId
      SourceSecurityGroupId: !GetAtt MasterSecurityGroup.GroupId
      Description: Kubernetes ingress services
      FromPort: 30000
      ToPort: 32767
      IpProtocol: udp

  MasterIamRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
        - Effect: "Allow"
          Principal:
            Service:
            - "ec2.amazonaws.com"
          Action:
          - "sts:AssumeRole"
      Policies:
      - PolicyName: !Join ["-", [!Ref InfrastructureName, "master", "policy"]]
        PolicyDocument:
          Version: "2012-10-17"
          Statement:
          - Effect: "Allow"
            Action:
            - "ec2:AttachVolume"
            - "ec2:AuthorizeSecurityGroupIngress"
            - "ec2:CreateSecurityGroup"
            - "ec2:CreateTags"
            - "ec2:CreateVolume"
            - "ec2:DeleteSecurityGroup"
            - "ec2:DeleteVolume"
            - "ec2:Describe*"
            - "ec2:DetachVolume"
            - "ec2:ModifyInstanceAttribute"
            - "ec2:ModifyVolume"
            - "ec2:RevokeSecurityGroupIngress"
            - "elasticloadbalancing:AddTags"
            - "elasticloadbalancing:AttachLoadBalancerToSubnets"
            - "elasticloadbalancing:ApplySecurityGroupsToLoadBalancer"
            - "elasticloadbalancing:CreateListener"
            - "elasticloadbalancing:CreateLoadBalancer"
            - "elasticloadbalancing:CreateLoadBalancerPolicy"
            - "elasticloadbalancing:CreateLoadBalancerListeners"
            - "elasticloadbalancing:CreateTargetGroup"
            - "elasticloadbalancing:ConfigureHealthCheck"
            - "elasticloadbalancing:DeleteListener"
            - "elasticloadbalancing:DeleteLoadBalancer"
            - "elasticloadbalancing:DeleteLoadBalancerListeners"
            - "elasticloadbalancing:DeleteTargetGroup"
            - "elasticloadbalancing:DeregisterInstancesFromLoadBalancer"
            - "elasticloadbalancing:DeregisterTargets"
            - "elasticloadbalancing:Describe*"
            - "elasticloadbalancing:DetachLoadBalancerFromSubnets"
            - "elasticloadbalancing:ModifyListener"
            - "elasticloadbalancing:ModifyLoadBalancerAttributes"
            - "elasticloadbalancing:ModifyTargetGroup"
            - "elasticloadbalancing:ModifyTargetGroupAttributes"
            - "elasticloadbalancing:RegisterInstancesWithLoadBalancer"
            - "elasticloadbalancing:RegisterTargets"
            - "elasticloadbalancing:SetLoadBalancerPoliciesForBackendServer"
            - "elasticloadbalancing:SetLoadBalancerPoliciesOfListener"
            - "kms:DescribeKey"
            Resource: "*"

  MasterInstanceProfile:
    Type: "AWS::IAM::InstanceProfile"
    Properties:
      Roles:
      - Ref: "MasterIamRole"

  WorkerIamRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
        - Effect: "Allow"
          Principal:
            Service:
            - "ec2.amazonaws.com"
          Action:
          - "sts:AssumeRole"
      Policies:
      - PolicyName: !Join ["-", [!Ref InfrastructureName, "worker", "policy"]]
        PolicyDocument:
          Version: "2012-10-17"
          Statement:
          - Effect: "Allow"
            Action:
            - "ec2:DescribeInstances"
            - "ec2:DescribeRegions"
            Resource: "*"

  WorkerInstanceProfile:
    Type: "AWS::IAM::InstanceProfile"
    Properties:
      Roles:
      - Ref: "WorkerIamRole"

Outputs:
  MasterSecurityGroupId:
    Description: Master Security Group ID
    Value: !GetAtt MasterSecurityGroup.GroupId

  WorkerSecurityGroupId:
    Description: Worker Security Group ID
    Value: !GetAtt WorkerSecurityGroup.GroupId

  MasterInstanceProfile:
    Description: Master IAM Instance Profile
    Value: !Ref MasterInstanceProfile

  WorkerInstanceProfile:
    Description: Worker IAM Instance Profile
    Value: !Ref WorkerInstanceProfile
```

## Accessing RHCOS AMIs with stream metadata {#installation-aws-ami-stream-metadata_installing-restricted-networks-aws}

To find the correct RHCOS boot image for your cluster, you can use stream metadata, which provides standardized information about RHCOS in the JSON format.

You can use the `coreos print-stream-json` subcommand of `openshift-install` to access information about the boot images in the stream metadata format. This command provides a method for printing stream metadata in a scriptable, machine-readable format.

For user-provisioned installations, the `openshift-install` binary has references to the version of RHCOS boot images that are tested for use with OpenShift Container Platform, such as the Amazon Web Services (AWS) AMI.

To parse the stream metadata, use one of the following methods:

**Procedure**

- From a Go program, use the official `stream-metadata-go` library at https://github.com/coreos/stream-metadata-go. You can also view example code in the library.
- From another programming language, such as Python or Ruby, use the JSON library of your preferred programming language.
- From a command-line utility that handles JSON data, such as `jq`, print the current `x86_64` or `aarch64` AMI for an AWS region, such as `us-west-1`:
  ```terminal title="For x86_64"
  $ openshift-install coreos print-stream-json | jq -r '.architectures.x86_64.images.aws.regions["us-west-1"].image'
  ```

  ```terminal title="Example output"
  ami-0d3e625f84626bbda
  ```

  ```terminal title="For aarch64"
  $ openshift-install coreos print-stream-json | jq -r '.architectures.aarch64.images.aws.regions["us-west-1"].image'
  ```

  ```terminal title="Example output"
  ami-0af1d3b7fa5be2131
  ```

  The output of this command is the AWS AMI ID for your designated architecture and the `us-west-1` region. The AMI must belong to the same region as the cluster.

## RHCOS AMIs for the AWS infrastructure {#installation-aws-user-infra-rhcos-ami_installing-restricted-networks-aws}

To deploy OpenShift Container Platform nodes on Amazon Web Services (AWS), select from the valid Red Hat Enterprise Linux CoreOS (RHCOS) AMIs for your region and instance architecture.

:::note

By importing your own AMI, you can also install to regions that do not have published RHCOS AMIs.

:::

**x86_64 RHCOS AMIs**

<table>
<thead>
<tr>
  <th>AWS zone</th>
  <th>RHEL 9 AMI</th>
  <th>RHEL 10 AMI</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>af-south-1</code></td>
  <td><code>ami-0e09db1a117f89982</code></td>
  <td><code>ami-00b9419956a1b8301</code></td>
</tr>
<tr>
  <td><code>ap-east-1</code></td>
  <td><code>ami-0f3883046e2b590c4</code></td>
  <td><code>ami-093ce74a7831d5796</code></td>
</tr>
<tr>
  <td><code>ap-east-2</code></td>
  <td><code>ami-05fda30c28d357b97</code></td>
  <td><code>ami-0f1f5f78fad6126f3</code></td>
</tr>
<tr>
  <td><code>ap-northeast-1</code></td>
  <td><code>ami-0acebf7451fbed435</code></td>
  <td><code>ami-0b1305ab18da2b503</code></td>
</tr>
<tr>
  <td><code>ap-northeast-2</code></td>
  <td><code>ami-07e85fe3474ab6f53</code></td>
  <td><code>ami-089d5b21fb5472656</code></td>
</tr>
<tr>
  <td><code>ap-northeast-3</code></td>
  <td><code>ami-0bf06ecbd16316390</code></td>
  <td><code>ami-08b988fa11f0772c3</code></td>
</tr>
<tr>
  <td><code>ap-south-1</code></td>
  <td><code>ami-001b087fae6b102a2</code></td>
  <td><code>ami-030c986c15d7d21fe</code></td>
</tr>
<tr>
  <td><code>ap-south-2</code></td>
  <td><code>ami-02e59bb73395086de</code></td>
  <td><code>ami-0baf89a420726a0c9</code></td>
</tr>
<tr>
  <td><code>ap-southeast-1</code></td>
  <td><code>ami-07e0e4d66f0276e33</code></td>
  <td><code>ami-07db2a2569bf635d5</code></td>
</tr>
<tr>
  <td><code>ap-southeast-2</code></td>
  <td><code>ami-0656ee074d43ebeb9</code></td>
  <td><code>ami-0c66ca97b4cb72a96</code></td>
</tr>
<tr>
  <td><code>ap-southeast-3</code></td>
  <td><code>ami-0b8ac3107bf7b8091</code></td>
  <td><code>ami-0ea4633b5accce1cc</code></td>
</tr>
<tr>
  <td><code>ap-southeast-4</code></td>
  <td><code>ami-06a85e79ca82e97d3</code></td>
  <td><code>ami-0b9a19c6d404ebe82</code></td>
</tr>
<tr>
  <td><code>ap-southeast-5</code></td>
  <td><code>ami-068e811f466ce5eec</code></td>
  <td><code>ami-0860196faab6d36f5</code></td>
</tr>
<tr>
  <td><code>ap-southeast-6</code></td>
  <td><code>ami-01801dc800c336d1f</code></td>
  <td><code>ami-05391d944831d449c</code></td>
</tr>
<tr>
  <td><code>ap-southeast-7</code></td>
  <td><code>ami-01b449b1bf9c95caf</code></td>
  <td><code>ami-0ea7c99fe14478e31</code></td>
</tr>
<tr>
  <td><code>ca-central-1</code></td>
  <td><code>ami-016a214bc34aed24c</code></td>
  <td><code>ami-01a1c5433b11c6040</code></td>
</tr>
<tr>
  <td><code>ca-west-1</code></td>
  <td><code>ami-0279542db8d76fe7c</code></td>
  <td><code>ami-0aaec38c9c3c18973</code></td>
</tr>
<tr>
  <td><code>eu-central-1</code></td>
  <td><code>ami-02b4be39da643ac06</code></td>
  <td><code>ami-050a2036417aa85c9</code></td>
</tr>
<tr>
  <td><code>eu-central-2</code></td>
  <td><code>ami-09e9173753792f284</code></td>
  <td><code>ami-0dc1cab1a5a382089</code></td>
</tr>
<tr>
  <td><code>eu-north-1</code></td>
  <td><code>ami-0b4a484d5db49d4a5</code></td>
  <td><code>ami-0ebb900e33852ac20</code></td>
</tr>
<tr>
  <td><code>eu-south-1</code></td>
  <td><code>ami-02f2692568ca70d48</code></td>
  <td><code>ami-06794550da69b4d4f</code></td>
</tr>
<tr>
  <td><code>eu-south-2</code></td>
  <td><code>ami-0777de9170dd480a0</code></td>
  <td><code>ami-09b9b2363f8b9bf79</code></td>
</tr>
<tr>
  <td><code>eu-west-1</code></td>
  <td><code>ami-0754b5979bce4f62f</code></td>
  <td><code>ami-00277e2896ce030cd</code></td>
</tr>
<tr>
  <td><code>eu-west-2</code></td>
  <td><code>ami-05a2b3abb8cf0cc92</code></td>
  <td><code>ami-06c08d05f6a1085e5</code></td>
</tr>
<tr>
  <td><code>eu-west-3</code></td>
  <td><code>ami-01ba91ba1e67b52fa</code></td>
  <td><code>ami-0c94bd2324f9a7dc4</code></td>
</tr>
<tr>
  <td><code>il-central-1</code></td>
  <td><code>ami-0be1e841b9475abc2</code></td>
  <td><code>ami-090c5d273c266bcb1</code></td>
</tr>
<tr>
  <td><code>mx-central-1</code></td>
  <td><code>ami-04e5e190abb398aef</code></td>
  <td><code>ami-0127400b1a4f4d8a8</code></td>
</tr>
<tr>
  <td><code>sa-east-1</code></td>
  <td><code>ami-09b6c03d247ba3007</code></td>
  <td><code>ami-0d636038b33e48e74</code></td>
</tr>
<tr>
  <td><code>us-east-1</code></td>
  <td><code>ami-09a04cae40b5df1b1</code></td>
  <td><code>ami-06c799e44545e8040</code></td>
</tr>
<tr>
  <td><code>us-east-2</code></td>
  <td><code>ami-008f91aec6651d818</code></td>
  <td><code>ami-0b56c6461b8dfea32</code></td>
</tr>
<tr>
  <td><code>us-gov-east-1</code></td>
  <td><code>ami-083a079a4e93810d0</code></td>
  <td><code>ami-00a5a8f684bfe21a4</code></td>
</tr>
<tr>
  <td><code>us-gov-west-1</code></td>
  <td><code>ami-03c270b5f712d93c5</code></td>
  <td><code>ami-0ee3e9e7a587954c3</code></td>
</tr>
<tr>
  <td><code>us-west-1</code></td>
  <td><code>ami-000065c53330c76d2</code></td>
  <td><code>ami-0ceb35adb65ceb3ee</code></td>
</tr>
<tr>
  <td><code>us-west-2</code></td>
  <td><code>ami-0106a1d635d4a36c0</code></td>
  <td><code>ami-0a8a99e4004c7938d</code></td>
</tr>
</tbody>
</table>

**aarch64 RHCOS AMIs**

<table>
<thead>
<tr>
  <th>AWS zone</th>
  <th>RHEL 9 AMI</th>
  <th>RHEL 10 AMI</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>af-south-1</code></td>
  <td><code>ami-09b3b126662fe7a18</code></td>
  <td><code>ami-07c2492a6e610eb29</code></td>
</tr>
<tr>
  <td><code>ap-east-1</code></td>
  <td><code>ami-009fe8f4f06381d2e</code></td>
  <td><code>ami-0c081ca051d9066c3</code></td>
</tr>
<tr>
  <td><code>ap-east-2</code></td>
  <td><code>ami-0403657dcda8a5e9c</code></td>
  <td><code>ami-016834812d68d485e</code></td>
</tr>
<tr>
  <td><code>ap-northeast-1</code></td>
  <td><code>ami-0f9d02af671b8f84e</code></td>
  <td><code>ami-0a93317cde971c817</code></td>
</tr>
<tr>
  <td><code>ap-northeast-2</code></td>
  <td><code>ami-09fb79703d81dad43</code></td>
  <td><code>ami-008d018630379e1eb</code></td>
</tr>
<tr>
  <td><code>ap-northeast-3</code></td>
  <td><code>ami-038a507ec93b04ce1</code></td>
  <td><code>ami-016bf4359ca8f9ea2</code></td>
</tr>
<tr>
  <td><code>ap-south-1</code></td>
  <td><code>ami-0eb4f5b5dbaa33c62</code></td>
  <td><code>ami-01c3da87c9088e490</code></td>
</tr>
<tr>
  <td><code>ap-south-2</code></td>
  <td><code>ami-0d0f18aae857f459b</code></td>
  <td><code>ami-089e3dc824dfc53cc</code></td>
</tr>
<tr>
  <td><code>ap-southeast-1</code></td>
  <td><code>ami-0519530b4a949ac79</code></td>
  <td><code>ami-047501898db0e6004</code></td>
</tr>
<tr>
  <td><code>ap-southeast-2</code></td>
  <td><code>ami-029b0ef4d6d0872e6</code></td>
  <td><code>ami-00aa2f8c59143b0ae</code></td>
</tr>
<tr>
  <td><code>ap-southeast-3</code></td>
  <td><code>ami-0e04bab1932cc8079</code></td>
  <td><code>ami-001bd2512362e7b35</code></td>
</tr>
<tr>
  <td><code>ap-southeast-4</code></td>
  <td><code>ami-03b0fdc3fbc4a0fa4</code></td>
  <td><code>ami-0c3a562ba17fcc7fe</code></td>
</tr>
<tr>
  <td><code>ap-southeast-5</code></td>
  <td><code>ami-046fecd472297b7c4</code></td>
  <td><code>ami-0abc0ee6a009667b2</code></td>
</tr>
<tr>
  <td><code>ap-southeast-6</code></td>
  <td><code>ami-088024b57838dfd53</code></td>
  <td><code>ami-0c8b6c104987a0fc3</code></td>
</tr>
<tr>
  <td><code>ap-southeast-7</code></td>
  <td><code>ami-00c84a187abf62194</code></td>
  <td><code>ami-0fbf9de5c1828e872</code></td>
</tr>
<tr>
  <td><code>ca-central-1</code></td>
  <td><code>ami-0f65ba965f0cdf25b</code></td>
  <td><code>ami-06be00da14f45f988</code></td>
</tr>
<tr>
  <td><code>ca-west-1</code></td>
  <td><code>ami-0ce3bfdc385214b60</code></td>
  <td><code>ami-0260b4a668a59a922</code></td>
</tr>
<tr>
  <td><code>eu-central-1</code></td>
  <td><code>ami-077c9e69aa2a7442b</code></td>
  <td><code>ami-001fdc3025ce50006</code></td>
</tr>
<tr>
  <td><code>eu-central-2</code></td>
  <td><code>ami-0843ce8434ed947e0</code></td>
  <td><code>ami-0443b053f6e845524</code></td>
</tr>
<tr>
  <td><code>eu-north-1</code></td>
  <td><code>ami-047f81c57b0567e80</code></td>
  <td><code>ami-06bc091c0435adf6f</code></td>
</tr>
<tr>
  <td><code>eu-south-1</code></td>
  <td><code>ami-048742ddf9599b9a3</code></td>
  <td><code>ami-02ee91218bdc1bb3a</code></td>
</tr>
<tr>
  <td><code>eu-south-2</code></td>
  <td><code>ami-0385fbca30108a3a9</code></td>
  <td><code>ami-0b8943a7a26627b01</code></td>
</tr>
<tr>
  <td><code>eu-west-1</code></td>
  <td><code>ami-04631bbd6c1be5b55</code></td>
  <td><code>ami-064942d9b57521cf3</code></td>
</tr>
<tr>
  <td><code>eu-west-2</code></td>
  <td><code>ami-0915a41744ba40397</code></td>
  <td><code>ami-0e13b80ab624fc7d3</code></td>
</tr>
<tr>
  <td><code>eu-west-3</code></td>
  <td><code>ami-09fd8d0e79f45b71a</code></td>
  <td><code>ami-0b1bd601d3ecde37d</code></td>
</tr>
<tr>
  <td><code>il-central-1</code></td>
  <td><code>ami-0853f94ef8841751a</code></td>
  <td><code>ami-0073de64ca6a1189b</code></td>
</tr>
<tr>
  <td><code>mx-central-1</code></td>
  <td><code>ami-039d2c56cbe869df0</code></td>
  <td><code>ami-03bf73795d8dfac51</code></td>
</tr>
<tr>
  <td><code>sa-east-1</code></td>
  <td><code>ami-0915393860fee75df</code></td>
  <td><code>ami-0f8e239c3eb87df2b</code></td>
</tr>
<tr>
  <td><code>us-east-1</code></td>
  <td><code>ami-0e3af3b58f5710e43</code></td>
  <td><code>ami-04ec52f48c28d001d</code></td>
</tr>
<tr>
  <td><code>us-east-2</code></td>
  <td><code>ami-017020cb8aeeda203</code></td>
  <td><code>ami-0469df626c198243e</code></td>
</tr>
<tr>
  <td><code>us-gov-east-1</code></td>
  <td><code>ami-014a147dae2cf3359</code></td>
  <td><code>ami-03557a94deb16be46</code></td>
</tr>
<tr>
  <td><code>us-gov-west-1</code></td>
  <td><code>ami-07113f5ee8cde6fb3</code></td>
  <td><code>ami-06460c1920305cf08</code></td>
</tr>
<tr>
  <td><code>us-west-1</code></td>
  <td><code>ami-09ca10147735afd05</code></td>
  <td><code>ami-0cd45be3140b38916</code></td>
</tr>
<tr>
  <td><code>us-west-2</code></td>
  <td><code>ami-00e116f16409da3de</code></td>
  <td><code>ami-03206cc79683aa1a6</code></td>
</tr>
</tbody>
</table>

## Creating the bootstrap node in AWS {#installation-creating-aws-bootstrap_installing-restricted-networks-aws}

To initialize the OpenShift Container Platform control plane, create the bootstrap node in Amazon Web Services (AWS) by uploading the Ignition config to an S3 bucket and launching the `CloudFormation` template.

- Providing a location to serve the `bootstrap.ign` Ignition config file to your cluster. This file is in your installation directory. The provided `CloudFormation` template assumes that you serve the Ignition config files for your cluster from an S3 bucket. If you choose to serve the files from another location, you must change the templates.
- Using the provided `CloudFormation` template and a custom parameter file to create a stack of AWS resources. The stack represents the bootstrap node that your OpenShift Container Platform installation requires.

:::note

If you do not use the provided `CloudFormation` template to create your bootstrap node, you must review the provided information and manually create the infrastructure. If your cluster does not initialize correctly, you might have to contact Red Hat support with your installation logs.

:::

**Prerequisites**

- You created and configured DNS, load balancers, and listeners in AWS.
- You created the security groups and roles required for your cluster in AWS.

**Procedure**

1. Create the bucket by running the following command:
   ```terminal
   $ aws s3 mb s3://<cluster_name>-infra
   ```

   where `<cluster_name>-infra` is the bucket name. When creating the `install-config.yaml` file, replace `<cluster_name>` with the name specified for the cluster. You must use a presigned URL for your S3 bucket, instead of the `s3://` schema, if you are:

   - Deploying to a region that has endpoints that differ from the AWS SDK.
   - Deploying a proxy.
   - Providing your own custom endpoints.
2. Upload the `bootstrap.ign` Ignition config file to the bucket by running the following command:
   ```terminal
   $ aws s3 cp <installation_directory>/bootstrap.ign s3://<cluster_name>-infra/bootstrap.ign
   ```

   where `<installation_directory>` is the path to the directory that you stored the installation files in.
3. Verify that the file uploaded by running the following command:
   ```terminal
   $ aws s3 ls s3://<cluster_name>-infra/
   ```

   ```terminal title="Example output"
   2019-04-03 16:15:16     314878 bootstrap.ign
   ```

   :::note

   The bootstrap Ignition config file does have secrets, such as X.509 keys. The following steps give basic security for the S3 bucket. To give additional security, you can enable an S3 bucket policy to allow only certain users, such as the OpenShift IAM user, to access objects that the bucket has. You can avoid S3 entirely and serve your bootstrap Ignition config file from any address that the bootstrap machine can reach.

   :::
4. Create a JSON file that has the parameter values that the template requires:
   ```json
   [
     {
       "ParameterKey": "InfrastructureName",
       "ParameterValue": "mycluster-<random_string>"
     },
     {
       "ParameterKey": "RhcosAmi",
       "ParameterValue": "ami-<random_string>"
     },
     {
       "ParameterKey": "AllowedBootstrapSshCidr",
       "ParameterValue": "0.0.0.0/0"
     },
     {
       "ParameterKey": "PublicSubnet",
       "ParameterValue": "subnet-<random_string>"
     },
     {
       "ParameterKey": "MasterSecurityGroupId",
       "ParameterValue": "sg-<random_string>"
     },
     {
       "ParameterKey": "VpcId",
       "ParameterValue": "vpc-<random_string>"
     },
     {
       "ParameterKey": "BootstrapIgnitionLocation",
       "ParameterValue": "s3://<bucket_name>/bootstrap.ign"
     },
     {
       "ParameterKey": "AutoRegisterELB",
       "ParameterValue": "yes"
     },
     {
       "ParameterKey": "RegisterNlbIpTargetsLambdaArn",
       "ParameterValue": "arn:aws:lambda:<aws_region>:<account_number>:function:<dns_stack_name>-RegisterNlbIpTargets-<random_string>"
     },
     {
       "ParameterKey": "ExternalApiTargetGroupArn",
       "ParameterValue": "arn:aws:elasticloadbalancing:<aws_region>:<account_number>:targetgroup/<dns_stack_name>-Exter-<random_string>"
     },
     {
       "ParameterKey": "InternalApiTargetGroupArn",
       "ParameterValue": "arn:aws:elasticloadbalancing:<aws_region>:<account_number>:targetgroup/<dns_stack_name>-Inter-<random_string>"
     },
     {
       "ParameterKey": "InternalServiceTargetGroupArn",
       "ParameterValue": "arn:aws:elasticloadbalancing:<aws_region>:<account_number>:targetgroup/<dns_stack_name>-Inter-<random_string>"
     }
   ]
   ```

   where:

   <dl>
   <dt><code>InfrastructureName</code></dt>
   <dd>Specifies the name for your cluster infrastructure that your Ignition config files encode for the cluster. Specify the infrastructure name that you extracted from the Ignition config file metadata, which has the format <code>&lt;cluster_name&gt;-&lt;random_string&gt;</code>.</dd>
   <dt><code>RhcosAmi</code></dt>
   <dd>Specifies the current Red Hat Enterprise Linux CoreOS (RHCOS) AMI to use for the bootstrap node based on your selected architecture. Specify a valid <code>AWS::EC2::Image::Id</code> value.</dd>
   <dt><code>AllowedBootstrapSshCidr</code></dt>
   <dd>Specifies the CIDR block to allow SSH access to the bootstrap node. Specify a CIDR block in the format <code>x.x.x.x/16-24</code>.</dd>
   <dt><code>PublicSubnet</code></dt>
   <dd>Specifies the public subnet in your VPC to launch the bootstrap node into. Specify the <code>PublicSubnetIds</code> value from the output of the <code>CloudFormation</code> template for the VPC.</dd>
   <dt><code>MasterSecurityGroupId</code></dt>
   <dd>Specifies the control plane security group ID for registering temporary rules. Specify the <code>MasterSecurityGroupId</code> value from the output of the <code>CloudFormation</code> template for the security group and roles.</dd>
   <dt><code>VpcId</code></dt>
   <dd>Specifies the VPC that the created resources will belong to. Specify the <code>VpcId</code> value from the output of the <code>CloudFormation</code> template for the VPC.</dd>
   <dt><code>BootstrapIgnitionLocation</code></dt>
   <dd>Specifies the location to fetch the bootstrap Ignition config file from. Specify the S3 bucket and file name in the form <code>s3://&lt;bucket_name&gt;/bootstrap.ign</code>.</dd>
   <dt><code>AutoRegisterELB</code></dt>
   <dd>Specifies whether to register a network load balancer (NLB). Specify <code>yes</code> or <code>no</code>. If you specify <code>yes</code>, you must give a Lambda Amazon Resource Name (ARN) value.</dd>
   <dt><code>RegisterNlbIpTargetsLambdaArn</code></dt>
   <dd>Specifies the ARN for NLB IP target registration lambda group. Specify the <code>RegisterNlbIpTargetsLambda</code> value from the output of the <code>CloudFormation</code> template for DNS and load balancing. Use <code>arn:aws-us-gov</code> if deploying the cluster to an AWS <code>GovCloud</code> region.</dd>
   <dt><code>ExternalApiTargetGroupArn</code></dt>
   <dd>Specifies the ARN for external API load balancer target group. Specify the <code>ExternalApiTargetGroupArn</code> value from the output of the <code>CloudFormation</code> template for DNS and load balancing. Use <code>arn:aws-us-gov</code> if deploying the cluster to an AWS <code>GovCloud</code> region.</dd>
   <dt><code>InternalApiTargetGroupArn</code></dt>
   <dd>Specifies the ARN for internal API load balancer target group. Specify the <code>InternalApiTargetGroupArn</code> value from the output of the <code>CloudFormation</code> template for DNS and load balancing. Use <code>arn:aws-us-gov</code> if deploying the cluster to an AWS <code>GovCloud</code> region.</dd>
   <dt><code>InternalServiceTargetGroupArn</code></dt>
   <dd>Specifies the ARN for internal service load balancer target group. Specify the <code>InternalServiceTargetGroupArn</code> value from the output of the <code>CloudFormation</code> template for DNS and load balancing. Use <code>arn:aws-us-gov</code> if deploying the cluster to an AWS <code>GovCloud</code> region.</dd>
   </dl>
5. Copy the template from the `CloudFormation`** template for the bootstrap machine** section and save it as a YAML file on your computer. This template describes the bootstrap machine that your cluster requires.
6. Optional: If you are deploying the cluster with a proxy, you must update the ignition in the template to add the  `ignition.config.proxy` fields. Additionally, If you have added the Amazon EC2, Elastic Load Balancing, and S3 VPC endpoints to your VPC, you must add these endpoints to the `noProxy` field.
7. Launch the `CloudFormation` template to create a stack of AWS resources that represent the bootstrap node:
   :::warning

   You must enter the command on a single line.

   :::

   ```terminal
   $ aws cloudformation create-stack --stack-name <name> \
        --template-body file://<template>.yaml \
        --parameters file://<parameters>.json \
        --capabilities CAPABILITY_NAMED_IAM
   ```

   where:

   <dl>
   <dt><code>&lt;name&gt;</code></dt>
   <dd>Specifies the name for the <code>CloudFormation</code> stack, such as <code>cluster-bootstrap</code>. You need the name of this stack if you remove the cluster.</dd>
   <dt><code>&lt;template&gt;</code></dt>
   <dd>Specifies the relative path to and name of the <code>CloudFormation</code> template YAML file that you saved.</dd>
   <dt><code>&lt;parameters&gt;</code></dt>
   <dd>Specifies the relative path to and name of the <code>CloudFormation</code> parameters JSON file.</dd>
   <dt><code>CAPABILITY_NAMED_IAM</code></dt>
   <dd>You must explicitly declare this capability because the provided template creates some <code>AWS::IAM::Role</code> and <code>AWS::IAM::InstanceProfile</code> resources.</dd>
   </dl>

   ```terminal title="Example output"
   arn:aws:cloudformation:us-east-1:269333783861:stack/cluster-bootstrap/12944486-2add-11eb-9dee-12dace8e3a83
   ```
8. Confirm that the template components exist:
   ```terminal
   $ aws cloudformation describe-stacks --stack-name <name>
   ```

   After the `StackStatus` displays `CREATE_COMPLETE`, the output displays values for the following parameters. You must give these parameter values to the other `CloudFormation` templates that you run to create your cluster:

   <dl>
   <dt><code>BootstrapInstanceId</code></dt>
   <dd>The bootstrap Instance ID.</dd>
   <dt><code>BootstrapPublicIp</code></dt>
   <dd>The bootstrap node public IP address.</dd>
   <dt><code>BootstrapPrivateIp</code></dt>
   <dd>The bootstrap node private IP address.</dd>
   </dl>

### CloudFormation template for the bootstrap machine {#installation-cloudformation-bootstrap_installing-restricted-networks-aws}

The bootstrap machine `CloudFormation` template creates the temporary Amazon Web Services (AWS) resources that the OpenShift Container Platform bootstrap process requires to initialize the control plane.

```yaml title="CloudFormation template for the bootstrap machine"
AWSTemplateFormatVersion: 2010-09-09
Description: Template for OpenShift Cluster Bootstrap (EC2 Instance, Security Groups and IAM)

Parameters:
  InfrastructureName:
    AllowedPattern: ^([a-zA-Z][a-zA-Z0-9\-]{0,26})$
    MaxLength: 27
    MinLength: 1
    ConstraintDescription: Infrastructure name must be alphanumeric, start with a letter, and have a maximum of 27 characters.
    Description: A short, unique cluster ID used to tag cloud resources and identify items owned or used by the cluster.
    Type: String
  RhcosAmi:
    Description: Current Red Hat Enterprise Linux CoreOS AMI to use for bootstrap.
    Type: AWS::EC2::Image::Id
  AllowedBootstrapSshCidr:
    AllowedPattern: ^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])(\/([0-9]|1[0-9]|2[0-9]|3[0-2]))$
    ConstraintDescription: CIDR block parameter must be in the form x.x.x.x/0-32.
    Default: 0.0.0.0/0
    Description: CIDR block to allow SSH access to the bootstrap node.
    Type: String
  PublicSubnet:
    Description: The public subnet to launch the bootstrap node into.
    Type: AWS::EC2::Subnet::Id
  MasterSecurityGroupId:
    Description: The master security group ID for registering temporary rules.
    Type: AWS::EC2::SecurityGroup::Id
  VpcId:
    Description: The VPC-scoped resources will belong to this VPC.
    Type: AWS::EC2::VPC::Id
  BootstrapIgnitionLocation:
    Default: s3://my-s3-bucket/bootstrap.ign
    Description: Ignition config file location.
    Type: String
  AutoRegisterELB:
    Default: "yes"
    AllowedValues:
    - "yes"
    - "no"
    Description: Do you want to invoke NLB registration, which requires a Lambda ARN parameter?
    Type: String
  RegisterNlbIpTargetsLambdaArn:
    Description: ARN for NLB IP target registration lambda.
    Type: String
  ExternalApiTargetGroupArn:
    Description: ARN for external API load balancer target group.
    Type: String
  InternalApiTargetGroupArn:
    Description: ARN for internal API load balancer target group.
    Type: String
  InternalServiceTargetGroupArn:
    Description: ARN for internal service load balancer target group.
    Type: String
  BootstrapInstanceType:
    Description: Instance type for the bootstrap EC2 instance
    Default: "i3.large"
    Type: String

Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
    - Label:
        default: "Cluster Information"
      Parameters:
      - InfrastructureName
    - Label:
        default: "Host Information"
      Parameters:
      - RhcosAmi
      - BootstrapIgnitionLocation
      - MasterSecurityGroupId
    - Label:
        default: "Network Configuration"
      Parameters:
      - VpcId
      - AllowedBootstrapSshCidr
      - PublicSubnet
    - Label:
        default: "Load Balancer Automation"
      Parameters:
      - AutoRegisterELB
      - RegisterNlbIpTargetsLambdaArn
      - ExternalApiTargetGroupArn
      - InternalApiTargetGroupArn
      - InternalServiceTargetGroupArn
    ParameterLabels:
      InfrastructureName:
        default: "Infrastructure Name"
      VpcId:
        default: "VPC ID"
      AllowedBootstrapSshCidr:
        default: "Allowed SSH Source"
      PublicSubnet:
        default: "Public Subnet"
      RhcosAmi:
        default: "Red Hat Enterprise Linux CoreOS AMI ID"
      BootstrapIgnitionLocation:
        default: "Bootstrap Ignition Source"
      MasterSecurityGroupId:
        default: "Master Security Group ID"
      AutoRegisterELB:
        default: "Use Provided ELB Automation"

Conditions:
  DoRegistration: !Equals ["yes", !Ref AutoRegisterELB]

Resources:
  BootstrapIamRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
        - Effect: "Allow"
          Principal:
            Service:
            - "ec2.amazonaws.com"
          Action:
          - "sts:AssumeRole"
      Path: "/"
      Policies:
      - PolicyName: !Join ["-", [!Ref InfrastructureName, "bootstrap", "policy"]]
        PolicyDocument:
          Version: "2012-10-17"
          Statement:
          - Effect: "Allow"
            Action: "ec2:Describe*"
            Resource: "*"
          - Effect: "Allow"
            Action: "ec2:AttachVolume"
            Resource: "*"
          - Effect: "Allow"
            Action: "ec2:DetachVolume"
            Resource: "*"
          - Effect: "Allow"
            Action: "s3:GetObject"
            Resource: "*"

  BootstrapInstanceProfile:
    Type: "AWS::IAM::InstanceProfile"
    Properties:
      Path: "/"
      Roles:
      - Ref: "BootstrapIamRole"

  BootstrapSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Cluster Bootstrap Security Group
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 22
        ToPort: 22
        CidrIp: !Ref AllowedBootstrapSshCidr
      - IpProtocol: tcp
        ToPort: 19531
        FromPort: 19531
        CidrIp: 0.0.0.0/0
      VpcId: !Ref VpcId

  BootstrapInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref RhcosAmi
      IamInstanceProfile: !Ref BootstrapInstanceProfile
      InstanceType: !Ref BootstrapInstanceType
      NetworkInterfaces:
      - AssociatePublicIpAddress: "true"
        DeviceIndex: "0"
        GroupSet:
        - !Ref "BootstrapSecurityGroup"
        - !Ref "MasterSecurityGroupId"
        SubnetId: !Ref "PublicSubnet"
      UserData:
        Fn::Base64: !Sub
        - '{"ignition":{"config":{"replace":{"source":"${S3Loc}"}},"version":"3.1.0"}}'
        - {
          S3Loc: !Ref BootstrapIgnitionLocation
        }

  RegisterBootstrapApiTarget:
    Condition: DoRegistration
    Type: Custom::NLBRegister
    Properties:
      ServiceToken: !Ref RegisterNlbIpTargetsLambdaArn
      TargetArn: !Ref ExternalApiTargetGroupArn
      TargetIp: !GetAtt BootstrapInstance.PrivateIp

  RegisterBootstrapInternalApiTarget:
    Condition: DoRegistration
    Type: Custom::NLBRegister
    Properties:
      ServiceToken: !Ref RegisterNlbIpTargetsLambdaArn
      TargetArn: !Ref InternalApiTargetGroupArn
      TargetIp: !GetAtt BootstrapInstance.PrivateIp

  RegisterBootstrapInternalServiceTarget:
    Condition: DoRegistration
    Type: Custom::NLBRegister
    Properties:
      ServiceToken: !Ref RegisterNlbIpTargetsLambdaArn
      TargetArn: !Ref InternalServiceTargetGroupArn
      TargetIp: !GetAtt BootstrapInstance.PrivateIp

Outputs:
  BootstrapInstanceId:
    Description: Bootstrap Instance ID.
    Value: !Ref BootstrapInstance

  BootstrapPublicIp:
    Description: The bootstrap node public IP address.
    Value: !GetAtt BootstrapInstance.PublicIp

  BootstrapPrivateIp:
    Description: The bootstrap node private IP address.
    Value: !GetAtt BootstrapInstance.PrivateIp
```

**Additional resources**

- [RHCOS AMIs for the AWS infrastructure (AWS documentation)](/docs/installing/installing_aws/upi/installing-aws-user-infra#installation-aws-user-infra-rhcos-ami_installing-aws-user-infra)

### Creating the control plane machines in AWS {#installation-creating-aws-control-plane_installing-restricted-networks-aws}

To run the OpenShift Container Platform control plane, create the three control plane machines in Amazon Web Services (AWS) by using the provided `CloudFormation` template and a custom parameter file.

:::warning

The `CloudFormation` template creates a stack that represents three control plane nodes.

:::

:::note

If you do not use the provided `CloudFormation` template to create your control plane nodes, you must review the provided information and manually create the infrastructure. If your cluster does not initialize correctly, you might have to contact Red Hat support with your installation logs.

:::

**Prerequisites**

- You created the bootstrap machine.

**Procedure**

1. Create a JSON file that has the parameter values that the template requires:
   ```json
   [
     {
       "ParameterKey": "InfrastructureName",
       "ParameterValue": "mycluster-<random_string>"
     },
     {
       "ParameterKey": "RhcosAmi",
       "ParameterValue": "ami-<random_string>"
     },
     {
       "ParameterKey": "AutoRegisterDNS",
       "ParameterValue": "yes"
     },
     {
       "ParameterKey": "PrivateHostedZoneId",
       "ParameterValue": "<random_string>"
     },
     {
       "ParameterKey": "PrivateHostedZoneName",
       "ParameterValue": "mycluster.example.com"
     },
     {
       "ParameterKey": "Master0Subnet",
       "ParameterValue": "subnet-<random_string>"
     },
     {
       "ParameterKey": "Master1Subnet",
       "ParameterValue": "subnet-<random_string>"
     },
     {
       "ParameterKey": "Master2Subnet",
       "ParameterValue": "subnet-<random_string>"
     },
     {
       "ParameterKey": "MasterSecurityGroupId",
       "ParameterValue": "sg-<random_string>"
     },
     {
       "ParameterKey": "IgnitionLocation",
       "ParameterValue": "https://api-int.<cluster_name>.<domain_name>:22623/config/master"
     },
     {
       "ParameterKey": "CertificateAuthorities",
       "ParameterValue": "data:text/plain;charset=utf-8;base64,ABC...xYz=="
     },
     {
       "ParameterKey": "MasterInstanceProfileName",
       "ParameterValue": "<roles_stack>-MasterInstanceProfile-<random_string>"
     },
     {
       "ParameterKey": "MasterInstanceType",
       "ParameterValue": ""
     },
     {
       "ParameterKey": "AutoRegisterELB",
       "ParameterValue": "yes"
     },
     {
       "ParameterKey": "RegisterNlbIpTargetsLambdaArn",
       "ParameterValue": "arn:aws:lambda:<aws_region>:<account_number>:function:<dns_stack_name>-RegisterNlbIpTargets-<random_string>"
     },
     {
       "ParameterKey": "ExternalApiTargetGroupArn",
       "ParameterValue": "arn:aws:elasticloadbalancing:<aws_region>:<account_number>:targetgroup/<dns_stack_name>-Exter-<random_string>"
     },
     {
       "ParameterKey": "InternalApiTargetGroupArn",
       "ParameterValue": "arn:aws:elasticloadbalancing:<aws_region>:<account_number>:targetgroup/<dns_stack_name>-Inter-<random_string>"
     },
     {
       "ParameterKey": "InternalServiceTargetGroupArn",
       "ParameterValue": "arn:aws:elasticloadbalancing:<aws_region>:<account_number>:targetgroup/<dns_stack_name>-Inter-<random_string>"
     }
   ]
   ```

   where:

   <dl>
   <dt><code>InfrastructureName</code></dt>
   <dd>Specifies the name for your cluster infrastructure that your Ignition config files encode for the cluster. Specify the infrastructure name that you extracted from the Ignition config file metadata, which has the format <code>&lt;cluster_name&gt;-&lt;random_string&gt;</code>.</dd>
   <dt><code>RhcosAmi</code></dt>
   <dd>Specifies the current Red Hat Enterprise Linux CoreOS (RHCOS) AMI to use for the control plane machines based on your selected architecture. Specify an <code>AWS::EC2::Image::Id</code> value.</dd>
   <dt><code>AutoRegisterDNS</code></dt>
   <dd>Specifies whether to perform DNS etcd registration. Specify <code>yes</code> or <code>no</code>. If you specify <code>yes</code>, you must give hosted zone information.</dd>
   <dt><code>PrivateHostedZoneId</code></dt>
   <dd>Specifies the Route 53 private zone ID to register the etcd targets with. Specify the <code>PrivateHostedZoneId</code> value from the output of the <code>CloudFormation</code> template for DNS and load balancing.</dd>
   <dt><code>PrivateHostedZoneName</code></dt>
   <dd>Specifies the Route 53 zone to register the targets with. Specify <code>&lt;cluster_name&gt;.&lt;domain_name&gt;</code> where <code>&lt;domain_name&gt;</code> is the Route 53 base domain that you used when you generated the <code>install-config.yaml</code> file for the cluster. Do not include the trailing period (.) that is displayed in the AWS console.</dd>
   <dt><code>Master0Subnet</code>, <code>Master1Subnet</code>, <code>Master2Subnet</code></dt>
   <dd>Specifies a subnet, preferably private, to launch the control plane machines on. Specify a subnet from the <code>PrivateSubnets</code> value from the output of the <code>CloudFormation</code> template for DNS and load balancing.</dd>
   <dt><code>MasterSecurityGroupId</code></dt>
   <dd>Specifies the control plane security group ID to associate with control plane nodes. Specify the <code>MasterSecurityGroupId</code> value from the output of the <code>CloudFormation</code> template for the security group and roles.</dd>
   <dt><code>IgnitionLocation</code></dt>
   <dd>Specifies the location to fetch the control plane Ignition config file from. Specify the generated Ignition config file location, <code>https://api-int.&lt;cluster_name&gt;.&lt;domain_name&gt;:22623/config/master</code>.</dd>
   <dt><code>CertificateAuthorities</code></dt>
   <dd>Specifies the base64 encoded certificate authority string to use. Specify the value from the <code>master.ign</code> file that is in the installation directory. This value is the long string with the format <code>data:text/plain;charset=utf-8;base64,ABC...xYz==</code>.</dd>
   <dt><code>MasterInstanceProfileName</code></dt>
   <dd>Specifies the IAM profile to associate with control plane nodes. Specify the <code>MasterInstanceProfile</code> parameter value from the output of the <code>CloudFormation</code> template for the security group and roles.</dd>
   <dt><code>MasterInstanceType</code></dt>
   <dd>Specifies the type of AWS instance to use for the control plane machines based on your selected architecture. The instance type value corresponds to the minimum resource requirements for control plane machines. For example <code>m6i.xlarge</code> is a type for AMD64 and <code>m6g.xlarge</code> is a type for ARM64.</dd>
   <dt><code>AutoRegisterELB</code></dt>
   <dd>Specifies whether to register a network load balancer (NLB). Specify <code>yes</code> or <code>no</code>. If you specify <code>yes</code>, you must give a Lambda Amazon Resource Name (ARN) value.</dd>
   <dt><code>RegisterNlbIpTargetsLambdaArn</code></dt>
   <dd>Specifies the ARN for NLB IP target registration lambda group. Specify the <code>RegisterNlbIpTargetsLambda</code> value from the output of the <code>CloudFormation</code> template for DNS and load balancing. Use <code>arn:aws-us-gov</code> if deploying the cluster to an AWS <code>GovCloud</code> region.</dd>
   <dt><code>ExternalApiTargetGroupArn</code></dt>
   <dd>Specifies the ARN for external API load balancer target group. Specify the <code>ExternalApiTargetGroupArn</code> value from the output of the <code>CloudFormation</code> template for DNS and load balancing. Use <code>arn:aws-us-gov</code> if deploying the cluster to an AWS <code>GovCloud</code> region.</dd>
   <dt><code>InternalApiTargetGroupArn</code></dt>
   <dd>Specifies the ARN for internal API load balancer target group. Specify the <code>InternalApiTargetGroupArn</code> value from the output of the <code>CloudFormation</code> template for DNS and load balancing. Use <code>arn:aws-us-gov</code> if deploying the cluster to an AWS <code>GovCloud</code> region.</dd>
   <dt><code>InternalServiceTargetGroupArn</code></dt>
   <dd>Specifies the ARN for internal service load balancer target group. Specify the <code>InternalServiceTargetGroupArn</code> value from the output of the <code>CloudFormation</code> template for DNS and load balancing. Use <code>arn:aws-us-gov</code> if deploying the cluster to an AWS <code>GovCloud</code> region.</dd>
   </dl>
2. Copy the template from the `CloudFormation`** template for control plane machines** section and save it as a YAML file on your computer. This template describes the control plane machines that your cluster requires.
3. If you specified an `m5` instance type as the value for `MasterInstanceType`, add that instance type to the `MasterInstanceType.AllowedValues` parameter in the `CloudFormation` template.
4. Launch the `CloudFormation` template to create a stack of AWS resources that represent the control plane nodes:
   :::warning

   You must enter the command on a single line.

   :::

   ```terminal
   $ aws cloudformation create-stack --stack-name <name> \
        --template-body file://<template>.yaml \
        --parameters file://<parameters>.json
   ```

   where:

   <dl>
   <dt><code>&lt;name&gt;</code></dt>
   <dd>Specifies the name for the <code>CloudFormation</code> stack, such as <code>cluster-control-plane</code>. You need the name of this stack if you remove the cluster.</dd>
   <dt><code>&lt;template&gt;</code></dt>
   <dd>Specifies the relative path to and name of the <code>CloudFormation</code> template YAML file that you saved.</dd>
   <dt><code>&lt;parameters&gt;</code></dt>
   <dd>Specifies the relative path to and name of the <code>CloudFormation</code> parameters JSON file.</dd>
   </dl>

   ```terminal title="Example output"
   arn:aws:cloudformation:us-east-1:269333783861:stack/cluster-control-plane/21c7e2b0-2ee2-11eb-c6f6-0aa34627df4b
   ```

   :::note

   The `CloudFormation` template creates a stack that represents three control plane nodes.

   :::
5. Confirm that the template components exist:
   ```terminal
   $ aws cloudformation describe-stacks --stack-name <name>
   ```

### CloudFormation template for control plane machines {#installation-cloudformation-control-plane_installing-restricted-networks-aws}

The control plane `CloudFormation` template creates the Amazon Web Services (AWS) resources for the three control plane machines that manage your OpenShift Container Platform cluster.

```yaml title="CloudFormation template for control plane machines"
AWSTemplateFormatVersion: 2010-09-09
Description: Template for OpenShift Cluster Node Launch (EC2 master instances)

Parameters:
  InfrastructureName:
    AllowedPattern: ^([a-zA-Z][a-zA-Z0-9\-]{0,26})$
    MaxLength: 27
    MinLength: 1
    ConstraintDescription: Infrastructure name must be alphanumeric, start with a letter, and have a maximum of 27 characters.
    Description: A short, unique cluster ID used to tag nodes for the kubelet cloud provider.
    Type: String
  RhcosAmi:
    Description: Current Red Hat Enterprise Linux CoreOS AMI to use for bootstrap.
    Type: AWS::EC2::Image::Id
  AutoRegisterDNS:
    Default: ""
    Description: unused
    Type: String
  PrivateHostedZoneId:
    Default: ""
    Description: unused
    Type: String
  PrivateHostedZoneName:
    Default: ""
    Description: unused
    Type: String
  Master0Subnet:
    Description: The subnets, recommend private, to launch the master nodes into.
    Type: AWS::EC2::Subnet::Id
  Master1Subnet:
    Description: The subnets, recommend private, to launch the master nodes into.
    Type: AWS::EC2::Subnet::Id
  Master2Subnet:
    Description: The subnets, recommend private, to launch the master nodes into.
    Type: AWS::EC2::Subnet::Id
  MasterSecurityGroupId:
    Description: The master security group ID to associate with master nodes.
    Type: AWS::EC2::SecurityGroup::Id
  IgnitionLocation:
    Default: https://api-int.$CLUSTER_NAME.$DOMAIN:22623/config/master
    Description: Ignition config file location.
    Type: String
  CertificateAuthorities:
    Default: data:text/plain;charset=utf-8;base64,ABC...xYz==
    Description: Base64 encoded certificate authority string to use.
    Type: String
  MasterInstanceProfileName:
    Description: IAM profile to associate with master nodes.
    Type: String
  MasterInstanceType:
    Default: m5.xlarge
    Type: String

  AutoRegisterELB:
    Default: "yes"
    AllowedValues:
    - "yes"
    - "no"
    Description: Do you want to invoke NLB registration, which requires a Lambda ARN parameter?
    Type: String
  RegisterNlbIpTargetsLambdaArn:
    Description: ARN for NLB IP target registration lambda. Supply the value from the cluster infrastructure or select "no" for AutoRegisterELB.
    Type: String
  ExternalApiTargetGroupArn:
    Description: ARN for external API load balancer target group. Supply the value from the cluster infrastructure or select "no" for AutoRegisterELB.
    Type: String
  InternalApiTargetGroupArn:
    Description: ARN for internal API load balancer target group. Supply the value from the cluster infrastructure or select "no" for AutoRegisterELB.
    Type: String
  InternalServiceTargetGroupArn:
    Description: ARN for internal service load balancer target group. Supply the value from the cluster infrastructure or select "no" for AutoRegisterELB.
    Type: String

Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
    - Label:
        default: "Cluster Information"
      Parameters:
      - InfrastructureName
    - Label:
        default: "Host Information"
      Parameters:
      - MasterInstanceType
      - RhcosAmi
      - IgnitionLocation
      - CertificateAuthorities
      - MasterSecurityGroupId
      - MasterInstanceProfileName
    - Label:
        default: "Network Configuration"
      Parameters:
      - VpcId
      - AllowedBootstrapSshCidr
      - Master0Subnet
      - Master1Subnet
      - Master2Subnet
    - Label:
        default: "Load Balancer Automation"
      Parameters:
      - AutoRegisterELB
      - RegisterNlbIpTargetsLambdaArn
      - ExternalApiTargetGroupArn
      - InternalApiTargetGroupArn
      - InternalServiceTargetGroupArn
    ParameterLabels:
      InfrastructureName:
        default: "Infrastructure Name"
      VpcId:
        default: "VPC ID"
      Master0Subnet:
        default: "Master-0 Subnet"
      Master1Subnet:
        default: "Master-1 Subnet"
      Master2Subnet:
        default: "Master-2 Subnet"
      MasterInstanceType:
        default: "Master Instance Type"
      MasterInstanceProfileName:
        default: "Master Instance Profile Name"
      RhcosAmi:
        default: "Red Hat Enterprise Linux CoreOS AMI ID"
      BootstrapIgnitionLocation:
        default: "Master Ignition Source"
      CertificateAuthorities:
        default: "Ignition CA String"
      MasterSecurityGroupId:
        default: "Master Security Group ID"
      AutoRegisterELB:
        default: "Use Provided ELB Automation"

Conditions:
  DoRegistration: !Equals ["yes", !Ref AutoRegisterELB]

Resources:
  Master0:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref RhcosAmi
      BlockDeviceMappings:
      - DeviceName: /dev/xvda
        Ebs:
          VolumeSize: "120"
          VolumeType: "gp2"
      IamInstanceProfile: !Ref MasterInstanceProfileName
      InstanceType: !Ref MasterInstanceType
      NetworkInterfaces:
      - AssociatePublicIpAddress: "false"
        DeviceIndex: "0"
        GroupSet:
        - !Ref "MasterSecurityGroupId"
        SubnetId: !Ref "Master0Subnet"
      UserData:
        Fn::Base64: !Sub
        - '{"ignition":{"config":{"merge":[{"source":"${SOURCE}"}]},"security":{"tls":{"certificateAuthorities":[{"source":"${CA_BUNDLE}"}]}},"version":"3.1.0"}}'
        - {
          SOURCE: !Ref IgnitionLocation,
          CA_BUNDLE: !Ref CertificateAuthorities,
        }
      Tags:
      - Key: !Join ["", ["kubernetes.io/cluster/", !Ref InfrastructureName]]
        Value: "shared"

  RegisterMaster0:
    Condition: DoRegistration
    Type: Custom::NLBRegister
    Properties:
      ServiceToken: !Ref RegisterNlbIpTargetsLambdaArn
      TargetArn: !Ref ExternalApiTargetGroupArn
      TargetIp: !GetAtt Master0.PrivateIp

  RegisterMaster0InternalApiTarget:
    Condition: DoRegistration
    Type: Custom::NLBRegister
    Properties:
      ServiceToken: !Ref RegisterNlbIpTargetsLambdaArn
      TargetArn: !Ref InternalApiTargetGroupArn
      TargetIp: !GetAtt Master0.PrivateIp

  RegisterMaster0InternalServiceTarget:
    Condition: DoRegistration
    Type: Custom::NLBRegister
    Properties:
      ServiceToken: !Ref RegisterNlbIpTargetsLambdaArn
      TargetArn: !Ref InternalServiceTargetGroupArn
      TargetIp: !GetAtt Master0.PrivateIp

  Master1:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref RhcosAmi
      BlockDeviceMappings:
      - DeviceName: /dev/xvda
        Ebs:
          VolumeSize: "120"
          VolumeType: "gp2"
      IamInstanceProfile: !Ref MasterInstanceProfileName
      InstanceType: !Ref MasterInstanceType
      NetworkInterfaces:
      - AssociatePublicIpAddress: "false"
        DeviceIndex: "0"
        GroupSet:
        - !Ref "MasterSecurityGroupId"
        SubnetId: !Ref "Master1Subnet"
      UserData:
        Fn::Base64: !Sub
        - '{"ignition":{"config":{"merge":[{"source":"${SOURCE}"}]},"security":{"tls":{"certificateAuthorities":[{"source":"${CA_BUNDLE}"}]}},"version":"3.1.0"}}'
        - {
          SOURCE: !Ref IgnitionLocation,
          CA_BUNDLE: !Ref CertificateAuthorities,
        }
      Tags:
      - Key: !Join ["", ["kubernetes.io/cluster/", !Ref InfrastructureName]]
        Value: "shared"

  RegisterMaster1:
    Condition: DoRegistration
    Type: Custom::NLBRegister
    Properties:
      ServiceToken: !Ref RegisterNlbIpTargetsLambdaArn
      TargetArn: !Ref ExternalApiTargetGroupArn
      TargetIp: !GetAtt Master1.PrivateIp

  RegisterMaster1InternalApiTarget:
    Condition: DoRegistration
    Type: Custom::NLBRegister
    Properties:
      ServiceToken: !Ref RegisterNlbIpTargetsLambdaArn
      TargetArn: !Ref InternalApiTargetGroupArn
      TargetIp: !GetAtt Master1.PrivateIp

  RegisterMaster1InternalServiceTarget:
    Condition: DoRegistration
    Type: Custom::NLBRegister
    Properties:
      ServiceToken: !Ref RegisterNlbIpTargetsLambdaArn
      TargetArn: !Ref InternalServiceTargetGroupArn
      TargetIp: !GetAtt Master1.PrivateIp

  Master2:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref RhcosAmi
      BlockDeviceMappings:
      - DeviceName: /dev/xvda
        Ebs:
          VolumeSize: "120"
          VolumeType: "gp2"
      IamInstanceProfile: !Ref MasterInstanceProfileName
      InstanceType: !Ref MasterInstanceType
      NetworkInterfaces:
      - AssociatePublicIpAddress: "false"
        DeviceIndex: "0"
        GroupSet:
        - !Ref "MasterSecurityGroupId"
        SubnetId: !Ref "Master2Subnet"
      UserData:
        Fn::Base64: !Sub
        - '{"ignition":{"config":{"merge":[{"source":"${SOURCE}"}]},"security":{"tls":{"certificateAuthorities":[{"source":"${CA_BUNDLE}"}]}},"version":"3.1.0"}}'
        - {
          SOURCE: !Ref IgnitionLocation,
          CA_BUNDLE: !Ref CertificateAuthorities,
        }
      Tags:
      - Key: !Join ["", ["kubernetes.io/cluster/", !Ref InfrastructureName]]
        Value: "shared"

  RegisterMaster2:
    Condition: DoRegistration
    Type: Custom::NLBRegister
    Properties:
      ServiceToken: !Ref RegisterNlbIpTargetsLambdaArn
      TargetArn: !Ref ExternalApiTargetGroupArn
      TargetIp: !GetAtt Master2.PrivateIp

  RegisterMaster2InternalApiTarget:
    Condition: DoRegistration
    Type: Custom::NLBRegister
    Properties:
      ServiceToken: !Ref RegisterNlbIpTargetsLambdaArn
      TargetArn: !Ref InternalApiTargetGroupArn
      TargetIp: !GetAtt Master2.PrivateIp

  RegisterMaster2InternalServiceTarget:
    Condition: DoRegistration
    Type: Custom::NLBRegister
    Properties:
      ServiceToken: !Ref RegisterNlbIpTargetsLambdaArn
      TargetArn: !Ref InternalServiceTargetGroupArn
      TargetIp: !GetAtt Master2.PrivateIp

Outputs:
  PrivateIPs:
    Description: The control-plane node private IP addresses.
    Value:
      !Join [
        ",",
        [!GetAtt Master0.PrivateIp, !GetAtt Master1.PrivateIp, !GetAtt Master2.PrivateIp]
      ]
```

## Creating the worker nodes in AWS {#installation-creating-aws-worker_installing-restricted-networks-aws}

To run application workloads on your OpenShift Container Platform cluster, create worker nodes in Amazon Web Services (AWS) by using the provided CloudFormation template.

You can use the provided CloudFormation template and a custom parameter file to create a stack of AWS resources that represent a worker node.

:::warning

The CloudFormation template creates a stack that represents one worker node. You must create a stack for each worker node.

:::

:::note

If you do not use the provided CloudFormation template to create your worker nodes, you must review the provided information and manually create the infrastructure. If your cluster does not initialize correctly, you might have to contact Red Hat support with your installation logs.

:::

**Prerequisites**

- You created the control plane machines.

**Procedure**

1. Create a JSON file that contains the parameter values that the CloudFormation template requires:
   ```json
   [
     {
       "ParameterKey": "InfrastructureName",
       "ParameterValue": "mycluster-<random_string>"
     },
     {
       "ParameterKey": "RhcosAmi",
       "ParameterValue": "ami-<random_string>"
     },
     {
       "ParameterKey": "Subnet",
       "ParameterValue": "subnet-<random_string>"
     },
     {
       "ParameterKey": "WorkerSecurityGroupId",
       "ParameterValue": "sg-<random_string>"
     },
     {
       "ParameterKey": "IgnitionLocation",
       "ParameterValue": "https://api-int.<cluster_name>.<domain_name>:22623/config/worker"
     },
     {
       "ParameterKey": "CertificateAuthorities",
       "ParameterValue": "data:text/plain;charset=utf-8;base64,ABC...xYz=="
     },
     {
       "ParameterKey": "WorkerInstanceProfileName",
       "ParameterValue": "<roles_stack>-WorkerInstanceProfile-<random_string>"
     },
     {
       "ParameterKey": "WorkerInstanceType",
       "ParameterValue": ""
     }
   ]
   ```

   where:

   <dl>
   <dt><code>InfrastructureName</code></dt>
   <dd>Specifies the name for your cluster infrastructure that is encoded in your Ignition config files for the cluster. Set the value to the infrastructure name that you extracted from the Ignition config file metadata, which has the format <code>&lt;cluster-name&gt;-&lt;random-string&gt;</code>.</dd>
   <dt><code>RhcosAmi</code></dt>
   <dd>Specifies the current Red Hat Enterprise Linux CoreOS (RHCOS) AMI to use for the worker nodes based on your selected architecture. Set the value to a valid <code>AWS::EC2::Image::Id</code> value.</dd>
   <dt><code>Subnet</code></dt>
   <dd>Specifies a subnet, preferably private, to start the worker nodes on. Set the value to a subnet from the <code>PrivateSubnets</code> value from the output of the CloudFormation template for DNS and load balancing.</dd>
   <dt><code>WorkerSecurityGroupId</code></dt>
   <dd>Specifies the worker security group ID to associate with worker nodes. Set the value to the <code>WorkerSecurityGroupId</code> value from the output of the CloudFormation template for the security group and roles.</dd>
   <dt><code>IgnitionLocation</code></dt>
   <dd>Specifies the location to fetch the bootstrap Ignition config file from. Set the value to the generated Ignition config location, <code>https://api-int.&lt;cluster_name&gt;.&lt;domain_name&gt;:22623/config/worker</code>.</dd>
   <dt><code>CertificateAuthorities</code></dt>
   <dd>Specifies the base64 encoded certificate authority string to use. Set the value to the value from the <code>worker.ign</code> file that is in the installation directory. This value is the long string with the format <code>data:text/plain;charset=utf-8;base64,ABC...xYz==</code>.</dd>
   <dt><code>WorkerInstanceProfileName</code></dt>
   <dd>Specifies the IAM profile to associate with worker nodes. Set the value to the <code>WorkerInstanceProfile</code> parameter value from the output of the CloudFormation template for the security group and roles.</dd>
   <dt><code>WorkerInstanceType</code></dt>
   <dd>Specifies the type of AWS instance to use for the compute machines based on your selected architecture. The instance type value corresponds to the minimum resource requirements for compute machines. For example <code>m6i.large</code> is a type for AMD64 and <code>m6g.large</code> is a type for ARM64.</dd>
   </dl>
2. Copy the template from the **CloudFormation template for compute machines** section of this topic and save it as a YAML file on your computer. This template describes the compute machines that your cluster requires.
3. Optional: If you specified an `m5` instance type as the value for `WorkerInstanceType`, add that instance type to the `WorkerInstanceType.AllowedValues` parameter in the CloudFormation template.
4. Optional: If you are deploying with an AWS Marketplace image, update the `Worker0.type.properties.ImageID` parameter with the AMI ID that you obtained from your subscription.
5. Use the CloudFormation template to create a stack of AWS resources that represent a worker node:
   :::warning

   You must enter the command on a single line.

   :::

   ```terminal
   $ aws cloudformation create-stack --stack-name <name> \
        --template-body file://<template>.yaml \
        --parameters file://<parameters>.json
   ```

   where:

   <dl>
   <dt><code>&lt;name&gt;</code></dt>
   <dd>Specifies the name for the CloudFormation stack, such as <code>cluster-worker-1</code>. You need the name of this stack if you remove the cluster.</dd>
   <dt><code>&lt;template&gt;</code></dt>
   <dd>Specifies the relative path to and name of the CloudFormation template YAML file that you saved.</dd>
   <dt><code>&lt;parameters&gt;</code></dt>
   <dd>Specifies the relative path to and name of the CloudFormation parameters JSON file.</dd>
   </dl>

   ```terminal title="Example output"
   arn:aws:cloudformation:us-east-1:269333783861:stack/cluster-worker-1/729ee301-1c2a-11eb-348f-sd9888c65b59
   ```

   :::note

   The CloudFormation template creates a stack that represents one worker node.

   :::
6. Confirm that the template components exist:
   ```terminal
   $ aws cloudformation describe-stacks --stack-name <name>
   ```
7. Continue to create worker stacks until you have created enough worker machines for your cluster. You can create additional worker stacks by referencing the same template and parameter files and specifying a different stack name.
   :::warning

   You must create at least two worker machines, so you must create at least two stacks that use this CloudFormation template.

   :::

### CloudFormation template for compute machines {#installation-cloudformation-worker_installing-restricted-networks-aws}

The compute machine `CloudFormation` template creates the Amazon Web Services (AWS) resources for the worker nodes that run your OpenShift Container Platform application workloads.

```yaml title="CloudFormation template for compute machines"
AWSTemplateFormatVersion: 2010-09-09
Description: Template for OpenShift Cluster Node Launch (EC2 worker instance)

Parameters:
  InfrastructureName:
    AllowedPattern: ^([a-zA-Z][a-zA-Z0-9\-]{0,26})$
    MaxLength: 27
    MinLength: 1
    ConstraintDescription: Infrastructure name must be alphanumeric, start with a letter, and have a maximum of 27 characters.
    Description: A short, unique cluster ID used to tag nodes for the kubelet cloud provider.
    Type: String
  RhcosAmi:
    Description: Current Red Hat Enterprise Linux CoreOS AMI to use for bootstrap.
    Type: AWS::EC2::Image::Id
  Subnet:
    Description: The subnets, recommend private, to launch the worker nodes into.
    Type: AWS::EC2::Subnet::Id
  WorkerSecurityGroupId:
    Description: The worker security group ID to associate with worker nodes.
    Type: AWS::EC2::SecurityGroup::Id
  IgnitionLocation:
    Default: https://api-int.$CLUSTER_NAME.$DOMAIN:22623/config/worker
    Description: Ignition config file location.
    Type: String
  CertificateAuthorities:
    Default: data:text/plain;charset=utf-8;base64,ABC...xYz==
    Description: Base64 encoded certificate authority string to use.
    Type: String
  WorkerInstanceProfileName:
    Description: IAM profile to associate with worker nodes.
    Type: String
  WorkerInstanceType:
    Default: m5.large
    Type: String

Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
    - Label:
        default: "Cluster Information"
      Parameters:
      - InfrastructureName
    - Label:
        default: "Host Information"
      Parameters:
      - WorkerInstanceType
      - RhcosAmi
      - IgnitionLocation
      - CertificateAuthorities
      - WorkerSecurityGroupId
      - WorkerInstanceProfileName
    - Label:
        default: "Network Configuration"
      Parameters:
      - Subnet
    ParameterLabels:
      Subnet:
        default: "Subnet"
      InfrastructureName:
        default: "Infrastructure Name"
      WorkerInstanceType:
        default: "Worker Instance Type"
      WorkerInstanceProfileName:
        default: "Worker Instance Profile Name"
      RhcosAmi:
        default: "Red Hat Enterprise Linux CoreOS AMI ID"
      IgnitionLocation:
        default: "Worker Ignition Source"
      CertificateAuthorities:
        default: "Ignition CA String"
      WorkerSecurityGroupId:
        default: "Worker Security Group ID"

Resources:
  Worker0:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref RhcosAmi
      BlockDeviceMappings:
      - DeviceName: /dev/xvda
        Ebs:
          VolumeSize: "120"
          VolumeType: "gp2"
      IamInstanceProfile: !Ref WorkerInstanceProfileName
      InstanceType: !Ref WorkerInstanceType
      NetworkInterfaces:
      - AssociatePublicIpAddress: "false"
        DeviceIndex: "0"
        GroupSet:
        - !Ref "WorkerSecurityGroupId"
        SubnetId: !Ref "Subnet"
      UserData:
        Fn::Base64: !Sub
        - '{"ignition":{"config":{"merge":[{"source":"${SOURCE}"}]},"security":{"tls":{"certificateAuthorities":[{"source":"${CA_BUNDLE}"}]}},"version":"3.1.0"}}'
        - {
          SOURCE: !Ref IgnitionLocation,
          CA_BUNDLE: !Ref CertificateAuthorities,
        }
      Tags:
      - Key: !Join ["", ["kubernetes.io/cluster/", !Ref InfrastructureName]]
        Value: "shared"

Outputs:
  PrivateIP:
    Description: The compute node private IP address.
    Value: !GetAtt Worker0.PrivateIp
```

### Creating the CloudFormation stack for compute machines {#installation-aws-creating-cloudformation-stack_installing-restricted-networks-aws}

You can create a stack of Amazon Web Services (AWS) resources for the compute machines by using the provided `CloudFormation` template.

:::warning

When you use the `CloudFormation` template for the control plane machines, the template provisions all three control plane machines with a single stack; however, when you use the `CloudFormation` template to deploy the compute machines, you must create the number of stacks based on the number that you defined in the `install-config.yaml` file. You provision each stack once for each machine. To provision a new compute machine, you must change the stack name.

:::

**Procedure**

- To create the `CloudFormation` stack for compute machines, run the following command:
  ```terminal
  $ aws cloudformation create-stack --stack-name <name> \
       --template-body file://<template>.yaml \
       --parameters file://<parameters>.json
  ```

  where:

  <dl>
  <dt><code>&lt;name&gt;</code></dt>
  <dd>Specifies the <code>&lt;name&gt;</code> with the name for the <code>CloudFormation</code> stack, such as <code>cluster-worker-1</code>. You need the name of this stack if you remove the cluster.</dd>
  <dt><code>&lt;template&gt;</code></dt>
  <dd>Specifies the relative path and the name of the <code>CloudFormation</code> template YAML file that you saved.</dd>
  <dt><code>&lt;parameters&gt;</code></dt>
  <dd>Specifies the relative path and the name of the JSON file for the <code>CloudFormation</code> parameters.</dd>
  </dl>

  ```terminal title="Example output"
  arn:aws:cloudformation:us-east-1:269333783861:stack/cluster-worker-1/729ee301-1c2a-11eb-348f-sd9888c65b59
  ```

## Initializing the bootstrap sequence on AWS with user-provisioned infrastructure {#installation-aws-user-infra-bootstrap_installing-restricted-networks-aws}

After creating all required infrastructure in AWS, you can start the bootstrap sequence that initializes the OpenShift Container Platform control plane. Run the installation program to monitor the bootstrap process until the control plane is ready.

**Prerequisites**

- You created the worker nodes.

**Procedure**

1. Change to the directory that has the installation program and start the bootstrap process that initializes the OpenShift Container Platform control plane:
   ```terminal
   $ ./openshift-install wait-for bootstrap-complete --dir <installation_directory> \
       --log-level=info
   ```

   - For `<installation_directory>`, specify the path to the directory that you stored the installation files in.
   - To view different installation details, specify `warn`, `debug`, or `error` instead of `info`.
     ```terminal title="Example output"
     INFO Waiting up to 20m0s for the Kubernetes API at https://api.mycluster.example.com:6443...
     INFO API v1.35.4 up
     INFO Waiting up to 45m0s for bootstrapping to complete...
     INFO It is now safe to remove the bootstrap resources
     INFO Time elapsed: 1s
     ```

     The bootstrapping completion wait time varies per platform.

     If the command exits without a `FATAL` warning, your OpenShift Container Platform control plane has initialized.

     :::note

     After the control plane initializes, it sets up the compute nodes and installs additional services in the form of Operators.

     :::

**Additional resources**

- [Monitoring installation progress](/docs/support/troubleshooting/troubleshooting-installations#monitoring-installation-progress_troubleshooting-installations)
- [Gathering bootstrap node diagnostic data](/docs/support/troubleshooting/troubleshooting-installations#gathering-bootstrap-diagnostic-data_troubleshooting-installations)

## Approving the certificate signing requests for your machines {#installation-approve-csrs_installing-restricted-networks-aws}

To allow newly added machines to join your OpenShift Container Platform cluster, confirm that the cluster approves pending certificate signing requests (CSRs), or approve them yourself. Approve client requests first, then server requests.

**Prerequisites**

- You added machines to your cluster.

**Procedure**

1. Confirm that the cluster recognizes the machines:
   ```terminal
   $ oc get nodes
   ```

   ```terminal title="Example output"
   NAME      STATUS    ROLES   AGE  VERSION
   master-0  Ready     master  63m  v1.35.4
   master-1  Ready     master  63m  v1.35.4
   master-2  Ready     master  64m  v1.35.4
   ```

   The output lists all of the machines that you created.

   :::note

   The preceding output might not include the compute nodes until you approve some CSRs.

   :::
2. Review the pending CSRs and ensure that you see the client requests with the `Pending` or `Approved` status for each machine that you added to the cluster:
   ```terminal
   $ oc get csr
   ```

   ```terminal title="Example output"
   NAME        AGE     REQUESTOR                                                                   CONDITION
   csr-8b2br   15m     system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Pending
   csr-8vnps   15m     system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Pending
   ...
   ```

   In this example, two machines are joining the cluster. You might see more approved CSRs in the list.
3. If the CSRs were not approved, after all of the pending CSRs for the machines you added are in `Pending` status, approve the CSRs for your cluster machines:
   :::note

   You must approve your CSRs within an hour of adding the machines to the cluster. If you do not approve them within an hour, the certificates rotate, and more than two certificates are present for each node. You must approve all of these certificates. After you approve the client CSR, the kubelet creates a secondary CSR for the serving certificate, which requires manual approval. The `machine-approver` then automatically approves later serving certificate renewal requests if the kubelet requests a new certificate with the same parameters.

   :::

   :::note

   For clusters running on platforms that are not machine API enabled, such as bare metal and other user-provisioned infrastructure, you must implement a method of automatically approving the kubelet serving certificate requests (CSRs). If you do not approve a request, the `oc exec`, `oc rsh`, and `oc logs` commands cannot succeed, because the API server requires a serving certificate when it connects to the kubelet. Any operation that contacts the kubelet endpoint requires this certificate approval to be in place. The method must watch for new CSRs, confirm that the `node-bootstrapper` service account in the `system:node` or `system:admin` groups submitted the CSR, and confirm the identity of the node.

   :::

   - To approve them individually, run the following command for each valid CSR:
     ```terminal
     $ oc adm certificate approve <csr_name>
     ```

     where:

     <dl>
     <dt><code>&lt;csr_name&gt;</code></dt>
     <dd>Specifies the name of a CSR from the list of current CSRs.</dd>
     </dl>
   - To approve all pending CSRs, run the following command:
     ```terminal
     $ oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs --no-run-if-empty oc adm certificate approve
     ```

     :::note

     Some Operators might not become available until you approve some CSRs. Each node submits two CSRs, so you might need to run the command to approve CSRs many times.

     :::
4. After you approve your client requests, review the server requests for each machine that you added to the cluster:
   ```terminal
   $ oc get csr
   ```

   ```terminal title="Example output"
   NAME        AGE     REQUESTOR                                                                   CONDITION
   csr-bfd72   5m26s   system:node:ip-10-0-50-126.us-east-2.compute.internal                       Pending
   csr-c57lv   5m26s   system:node:ip-10-0-95-157.us-east-2.compute.internal                       Pending
   ...
   ```
5. If the remaining CSRs are not approved, and are in the `Pending` status, approve the CSRs for your cluster machines:
   - To approve them individually, run the following command for each valid CSR:
     ```terminal
     $ oc adm certificate approve <csr_name>
     ```

     where:

     <dl>
     <dt><code>&lt;csr_name&gt;</code></dt>
     <dd>Specifies the name of a CSR from the list of current CSRs.</dd>
     </dl>
   - To approve all pending CSRs, run the following command:
     ```terminal
     $ oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc adm certificate approve
     ```
6. After you approve all client and server CSRs, the machines have the `Ready` status. Verify this by running the following command:
   ```terminal
   $ oc get nodes
   ```

   ```terminal title="Example output"
   NAME      STATUS    ROLES   AGE  VERSION
   master-0  Ready     master  73m  v1.35.4
   master-1  Ready     master  73m  v1.35.4
   master-2  Ready     master  74m  v1.35.4
   worker-0  Ready     worker  11m  v1.35.4
   worker-1  Ready     worker  11m  v1.35.4
   ```

   :::note

   You might need to wait a few minutes after approval of the server CSRs for the machines to change to the `Ready` status.

   :::

## Initial Operator configuration {#installation-operators-config_installing-restricted-networks-aws}

After the control plane initializes, you must immediately configure some Operators so that they all become available.

**Prerequisites**

- Your control plane has initialized.

**Procedure**

1. Watch the cluster components come online:
   ```terminal
   $ watch -n5 oc get clusteroperators
   ```

   ```terminal title="Example output"
   NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE
   authentication                             4.22.0    True        False         False      19m
   baremetal                                  4.22.0    True        False         False      37m
   cloud-credential                           4.22.0    True        False         False      40m
   cluster-autoscaler                         4.22.0    True        False         False      37m
   config-operator                            4.22.0    True        False         False      38m
   console                                    4.22.0    True        False         False      26m
   csi-snapshot-controller                    4.22.0    True        False         False      37m
   dns                                        4.22.0    True        False         False      37m
   etcd                                       4.22.0    True        False         False      36m
   image-registry                             4.22.0    True        False         False      31m
   ingress                                    4.22.0    True        False         False      30m
   insights                                   4.22.0    True        False         False      31m
   kube-apiserver                             4.22.0    True        False         False      26m
   kube-controller-manager                    4.22.0    True        False         False      36m
   kube-scheduler                             4.22.0    True        False         False      36m
   kube-storage-version-migrator              4.22.0    True        False         False      37m
   machine-api                                4.22.0    True        False         False      29m
   machine-approver                           4.22.0    True        False         False      37m
   machine-config                             4.22.0    True        False         False      36m
   marketplace                                4.22.0    True        False         False      37m
   monitoring                                 4.22.0    True        False         False      29m
   network                                    4.22.0    True        False         False      38m
   node-tuning                                4.22.0    True        False         False      37m
   openshift-apiserver                        4.22.0    True        False         False      32m
   openshift-controller-manager               4.22.0    True        False         False      30m
   openshift-samples                          4.22.0    True        False         False      32m
   operator-lifecycle-manager                 4.22.0    True        False         False      37m
   operator-lifecycle-manager-catalog         4.22.0    True        False         False      37m
   operator-lifecycle-manager-packageserver   4.22.0    True        False         False      32m
   service-ca                                 4.22.0    True        False         False      38m
   storage                                    4.22.0    True        False         False      37m
   ```
2. Configure the Operators that are not available.

### Disabling the default software catalog sources {#olm-restricted-networks-operatorhub_installing-restricted-networks-aws}

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

### Image registry storage configuration {#installation-registry-storage-config_installing-restricted-networks-aws}

Amazon Web Services provides default storage, which means the Image Registry Operator is available after installation. However, if the Registry Operator cannot create an S3 bucket and automatically configure storage, you must manually configure registry storage.

Configure a persistent volume, which is required for production clusters. Where applicable, you can configure an empty directory as the storage location for non-production clusters.

You can also allow the image registry to use block storage types by using the `Recreate` rollout strategy during upgrades.

#### Configuring registry storage for AWS with user-provisioned infrastructure {#registry-configuring-storage-aws-user-infra_installing-restricted-networks-aws}

If the Registry Operator cannot automatically create and configure an Amazon S3 bucket during installation, you must manually configure registry storage for your cluster.

:::warning

To secure your registry images in Amazon Web Services (AWS), [block public access](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket-publicaccessblockconfiguration.html) to the S3 bucket.

:::

**Prerequisites**

- You have a cluster on AWS with user-provisioned infrastructure.
- For Amazon S3 storage, the secret must contain two keys:
  - `REGISTRY_STORAGE_S3_ACCESSKEY`
  - `REGISTRY_STORAGE_S3_SECRETKEY`

**Procedure**

1. Set up a [Bucket Lifecycle Policy](https://docs.aws.amazon.com/AmazonS3/latest/dev/mpuoverview.html#mpu-abort-incomplete-mpu-lifecycle-config) to cancel incomplete multipart uploads that are one day old.
2. Enter the storage configuration in `configs.imageregistry.operator.openshift.io/cluster`:
   ```terminal
   $ oc edit configs.imageregistry.operator.openshift.io/cluster
   ```

   ```yaml title="Example configuration"
   apiVersion: imageregistry.operator.openshift.io/v1
   kind: Config
   metadata:
     name: cluster
   spec:
     storage:
       s3:
         bucket: <bucket_name>
         region: <region_name>
   ```

#### Configuring storage for the image registry in non-production clusters {#installation-registry-storage-non-production_installing-restricted-networks-aws}

You must configure storage for the Image Registry Operator. For non-production clusters, you can set the image registry to an empty directory, but you lose all images if you restart the registry.

**Procedure**

- To set the image registry storage to an empty directory:
  ```terminal
  $ oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"storage":{"emptyDir":{}}}}'
  ```

  :::warning

  Configure this option only for non-production clusters.

  :::

  If you run this command before the Image Registry Operator initializes its components, the `oc patch` command fails with the following error:

  ```terminal title="Example output"
  Error from server (NotFound): configs.imageregistry.operator.openshift.io "cluster" not found
  ```

  Wait a few minutes and run the command again.

## Deleting the bootstrap resources {#installation-aws-user-infra-delete-bootstrap_installing-restricted-networks-aws}

After completing the initial Operator configuration for your OpenShift Container Platform cluster, you can delete the bootstrap resources from AWS to free up capacity and reduce costs.

**Prerequisites**

- You completed the initial Operator configuration for your cluster.

**Procedure**

1. Delete the bootstrap resources. If you used the `CloudFormation` template, [delete its stack](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-delete-stack.html):
   - Delete the stack by using the AWS CLI:
     ```terminal
     $ aws cloudformation delete-stack --stack-name <name>
     ```

     `<name>` is the name of your bootstrap stack.
   - Delete the stack by using the [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation/).

## Creating the Ingress DNS records {#installation-create-ingress-dns-records_installing-restricted-networks-aws}

If you removed the DNS zone configuration during installation, you must manually create DNS records that point to the Ingress load balancer so that your OpenShift Container Platform cluster routes are reachable.

You can create either a wildcard record or specific records. While the following procedure uses A records, you can use other record types that you require, such as CNAME or alias.

**Prerequisites**

- You deployed an OpenShift Container Platform cluster on Amazon Web Services (AWS) that uses infrastructure that you provisioned.
- You installed the OpenShift CLI (`oc`).
- You installed the `jq` package.
- You downloaded the AWS CLI and installed it on your computer. See [Install the AWS CLI Using the Bundled Installer (Linux, macOS, or Unix)](https://docs.aws.amazon.com/cli/latest/userguide/install-bundle.html).

**Procedure**

1. Find the routes to create.
   - To create a wildcard record, use `*.apps.<cluster_name>.<domain_name>`, where `<cluster_name>` is your cluster name, and `<domain_name>` is the Route 53 base domain for your OpenShift Container Platform cluster.
   - To create specific records, you must create a record for each route that your cluster uses, as shown in the output of the following command:
     ```terminal
     $ oc get --all-namespaces -o jsonpath='{range .items[*]}{range .status.ingress[*]}{.host}{"\n"}{end}{end}' routes
     ```

     ```terminal title="Example output"
     oauth-openshift.apps.<cluster_name>.<domain_name>
     console-openshift-console.apps.<cluster_name>.<domain_name>
     downloads-openshift-console.apps.<cluster_name>.<domain_name>
     alertmanager-main-openshift-monitoring.apps.<cluster_name>.<domain_name>
     prometheus-k8s-openshift-monitoring.apps.<cluster_name>.<domain_name>
     ```
2. Retrieve the Ingress Operator load balancer status and note the value of the external IP address that it uses, which the `EXTERNAL-IP` column displays:
   ```terminal
   $ oc -n openshift-ingress get service router-default
   ```

   ```terminal title="Example output"
   NAME             TYPE           CLUSTER-IP      EXTERNAL-IP                            PORT(S)                      AGE
   router-default   LoadBalancer   172.30.62.215   ab3...28.us-east-2.elb.amazonaws.com   80:31499/TCP,443:30693/TCP   5m
   ```
3. Locate the hosted zone ID for the load balancer:
   ```terminal
   $ aws elb describe-load-balancers | jq -r '.LoadBalancerDescriptions[] | select(.DNSName == "<external_ip>").CanonicalHostedZoneNameID'
   ```

   For `<external_ip>`, specify the value of the external IP address of the Ingress Operator load balancer that you obtained.

   ```terminal title="Example output"
   Z3AADJGX6KTTL2
   ```

   The output of this command is the load balancer hosted zone ID.
4. Obtain the public hosted zone ID for your cluster’s domain:
   ```terminal
   $ aws route53 list-hosted-zones-by-name \
               --dns-name "<domain_name>" \
               --query 'HostedZones[? Config.PrivateZone != `true` && Name == `<domain_name>.`].Id'
               --output text
   ```

   For `<domain_name>`, specify the Route 53 base domain for your OpenShift Container Platform cluster.

   ```terminal title="Example output"
   /hostedzone/Z3URY6TWQ91KVV
   ```

   The command output displays the public hosted zone ID for your domain. In this example, it is `Z3URY6TWQ91KVV`.
5. Add the alias records to your private zone:
   ```terminal
   $ aws route53 change-resource-record-sets --hosted-zone-id "<private_hosted_zone_id>" --change-batch '{
   >   "Changes": [
   >     {
   >       "Action": "CREATE",
   >       "ResourceRecordSet": {
   >         "Name": "\\052.apps.<cluster_domain>",
   >         "Type": "A",
   >         "AliasTarget":{
   >           "HostedZoneId": "<hosted_zone_id>",
   >           "DNSName": "<external_ip>.",
   >           "EvaluateTargetHealth": false
   >         }
   >       }
   >     }
   >   ]
   > }'
   ```

   where:

   <dl>
   <dt><code>&lt;private_hosted_zone_id&gt;</code></dt>
   <dd>Specifies the value from the output of the CloudFormation template for DNS and load balancing.</dd>
   <dt><code>&lt;cluster_domain&gt;</code></dt>
   <dd>Specifies the domain or subdomain that you use with your OpenShift Container Platform cluster.</dd>
   <dt><code>&lt;hosted_zone_id&gt;</code></dt>
   <dd>Specifies the public hosted zone ID for the load balancer that you obtained.</dd>
   <dt><code>&lt;external_ip&gt;</code></dt>
   <dd>Specifies the value of the external IP address of the Ingress Operator load balancer. Ensure that you include the trailing period (<code>.</code>) in this parameter value.</dd>
   </dl>
6. Add the records to your public zone:
   ```terminal
   $ aws route53 change-resource-record-sets --hosted-zone-id "<public_hosted_zone_id>"" --change-batch '{
   >   "Changes": [
   >     {
   >       "Action": "CREATE",
   >       "ResourceRecordSet": {
   >         "Name": "\\052.apps.<cluster_domain>",
   >         "Type": "A",
   >         "AliasTarget":{
   >           "HostedZoneId": "<hosted_zone_id>",
   >           "DNSName": "<external_ip>.",
   >           "EvaluateTargetHealth": false
   >         }
   >       }
   >     }
   >   ]
   > }'
   ```

   where:

   <dl>
   <dt><code>&lt;public_hosted_zone_id&gt;</code></dt>
   <dd>Specifies the public hosted zone for your domain.</dd>
   <dt><code>&lt;cluster_domain&gt;</code></dt>
   <dd>Specifies the domain or subdomain that you use with your OpenShift Container Platform cluster.</dd>
   <dt><code>&lt;hosted_zone_id&gt;</code></dt>
   <dd>Specifies the public hosted zone ID for the load balancer that you obtained.</dd>
   <dt><code>&lt;external_ip&gt;</code></dt>
   <dd>Specifies the value of the external IP address of the Ingress Operator load balancer. Ensure that you include the trailing period (<code>.</code>) in this parameter value.</dd>
   </dl>

## Completing an AWS installation on user-provisioned infrastructure {#installation-aws-user-infra-installation_installing-restricted-networks-aws}

To finish installing OpenShift Container Platform on user-provisioned AWS infrastructure, monitor the deployment until it completes successfully.

**Prerequisites**

- You removed the bootstrap node for an OpenShift Container Platform cluster on user-provisioned AWS infrastructure.
- You installed the `oc` CLI.

**Procedure**

1. From the directory that has the installation program, complete

the cluster installation:

```terminal
$ ./openshift-install --dir <installation_directory> wait-for install-complete
```

For `<installation_directory>`, specify the path to the directory that you stored the installation files in.

.Example output

```terminal
INFO Waiting up to 40m0s for the cluster at https://api.mycluster.example.com:6443 to initialize...
INFO Waiting up to 10m0s for the openshift-console route to be created...
INFO Install complete!
INFO To access the cluster as the system:admin user when using 'oc', run 'export KUBECONFIG=/home/myuser/install_dir/auth/kubeconfig'
INFO Access the OpenShift web-console here: https://console-openshift-console.apps.mycluster.example.com
INFO Login to the console with user: "kubeadmin", and password: "password"
INFO Time elapsed: 1s
```

:::warning

- The Ignition config files that the installation program generates contain certificates that expire after 24 hours, which are then renewed at that time. If you shut down the cluster before renewing the certificates and later restart it after the 24 hours have elapsed, the cluster automatically recovers the expired certificates. The exception is that you must manually approve the pending `node-bootstrapper` certificate signing requests (CSRs) to recover kubelet certificates. See the documentation for *Recovering from expired control plane certificates* for more information.
- Use Ignition config files within 12 hours after the installation program generates them because the 24-hour certificate rotates from 16 to 22 hours after you install the cluster. By using the Ignition config files within 12 hours, you can avoid installation failure if the certificate update runs during installation.

:::

1. Register your cluster on the [Cluster registration](https://console.redhat.com/openshift/register) page.

## Logging in to the cluster by using the CLI {#cli-logging-in-kubeadmin_installing-restricted-networks-aws}

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

## Logging in to the cluster by using the web console {#logging-in-by-using-the-web-console_installing-restricted-networks-aws}

To verify that your cluster deployed successfully and access its features, log in to the OpenShift Container Platform web console as the `kubeadmin` user.

**Prerequisites**

- You have access to the installation host.
- You completed a cluster installation and all cluster Operators are available.

**Procedure**

1. Obtain the password for the `kubeadmin` user from the `kubeadmin-password` file on the installation host:
   ```terminal
   $ cat <installation_directory>/auth/kubeadmin-password
   ```

   :::note

   Or, you can obtain the `kubeadmin` password from the `<installation_directory>/.openshift_install.log` log file on the installation host.

   :::
2. List the OpenShift Container Platform web console route:
   ```terminal
   $ oc get routes -n openshift-console | grep 'console-openshift'
   ```

   :::note

   Or, you can obtain the OpenShift Container Platform route from the `<installation_directory>/.openshift_install.log` log file on the installation host.

   :::

   ```terminal title="Example output"
   console     console-openshift-console.apps.<cluster_name>.<base_domain>            console     https   reencrypt/Redirect   None
   ```
3. Navigate to the route detailed in the output of the preceding command in a web browser and log in as the `kubeadmin` user.

**Additional resources**

- [Accessing the web console](/docs/web_console/web-console#web-console)
- [About remote health monitoring](/docs/support/remote_health_monitoring/about-remote-health-monitoring#about-remote-health-monitoring)
- [Managing AWS resources as a single unit with CloudFormation stacks (AWS documentation)](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacks.html)
- [Validating an installation](/docs/installing/validation_and_troubleshooting/validating-an-installation#validating-an-installation)
- [Customizing your cluster](/docs/post_installation_configuration/cluster-tasks#available_cluster_customizations)
- [Configuring image streams](/docs/post_installation_configuration/cluster-tasks#post-install-must-gather-disconnected)
- [Using Operator Lifecycle Manager in disconnected environments](/docs/disconnected/using-olm#olm-restricted-networks)
- [Image configuration resources](/docs/openshift_images/image-configuration#images-configuration-cas_image-configuration)
- [Remote health reporting](/docs/support/remote_health_monitoring/remote-health-reporting#remote-health-reporting)
- [Changing the cloud provider credentials configuration](/docs/post_installation_configuration/changing-cloud-credentials-configuration#manually-removing-cloud-creds_changing-cloud-credentials-configuration)
