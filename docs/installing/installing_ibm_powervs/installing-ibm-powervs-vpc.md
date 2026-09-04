---
title: Installing a cluster on IBM Power Virtual Server into an existing VPC
sidebar_position: 5
---

# Installing a cluster on IBM Power Virtual Server into an existing VPC {#installing-ibm-powervs-vpc}

<a id="installing-ibm-powervs-vpc"></a>

To install a OpenShift Container Platform cluster on IBM Power(R) Virtual Server into an existing Virtual Private Cloud (VPC), use installer-provisioned infrastructure. The installation program provisions the remaining required infrastructure, which you can then customize.

## Prerequisites {#prereqs-ibm-power-vs_installing-ibm-powervs-vpc}

Before you install a OpenShift Container Platform cluster on IBM Power(R) Virtual Server, complete the prerequisite tasks to configure your IBM Cloud(R) account, firewall, and credential utility.

- You reviewed details about the OpenShift Container Platform installation and update processes.
- You read the documentation on selecting a cluster installation method and preparing it for users.
- You configured an IBM Cloud(R) account to host the cluster.
- If you use a firewall, you configured it to allow the sites that your cluster requires access to.
- You configured the `ccoctl` utility before you installed the cluster.

**Additional resources**

- [OpenShift Container Platform installation and update](/docs/architecture/architecture-installation#architecture-installation)
- [Selecting a cluster installation method and preparing it for users](/docs/installing/overview/installing-preparing#installing-preparing)
- [Configuring an IBM Cloud(R) account](/docs/installing/installing_ibm_powervs/installing-ibm-cloud-account-power-vs#installing-ibm-cloud-account-power-vs)
- [Configuring your firewall](/docs/installing/install_config/configuring-firewall#configuring-firewall-module_configuring-firewall)
- [Configuring the Cloud Credential Operator utility](/docs/installing/installing_ibm_powervs/preparing-to-install-on-ibm-power-vs#cco-ccoctl-configuring_preparing-to-install-on-ibm-power-vs)

## About using a custom Virtual Private Cloud (VPC) {#installation-custom-ibm-powervs-vpc_installing-ibm-powervs-vpc}

In OpenShift Container Platform 4.22, you can deploy a cluster using an existing IBM(R) Virtual Private Cloud (VPC).

Because the installation program cannot know what other components are in your existing subnets, it cannot choose subnet CIDRs and so forth. You must configure networking for the subnets to which you will install the cluster.

### Requirements for using your Virtual Private Cloud (VPC) {#_requirements_for_using_your_virtual_private_cloud_vpc}

You must correctly configure the existing VPC and its subnets before you install the cluster. The installation program does not create a VPC or VPC subnet in this scenario.

The installation program cannot:

- Subdivide network ranges for the cluster to use
- Set route tables for the subnets
- Set VPC options such as DHCP

:::note

The installation program requires that you use the cloud-provided DNS server. Using a custom DNS server is not supported and causes the installation to fail.

:::

### Virtual Private Cloud (VPC) validation {#installation-custom-ibm-power-vs-validation_installing-ibm-powervs-vpc}

The VPC and all of the subnets must be in an existing resource group. The cluster is deployed to this resource group.

As part of the installation, specify the following in the `install-config.yaml` file:

- The name of the resource group
- The name of VPC
- The name of the VPC subnet

To ensure that the subnets that you give are suitable, the installation program confirms that all of the subnets you specify exist.

:::note

Subnet IDs are not supported.

:::

### Isolation between clusters {#installation-custom-ibm-power-vs-isolation_installing-ibm-powervs-vpc}

If you deploy OpenShift Container Platform to an existing network, cluster service isolation decreases in the following ways:

- ICMP Ingress is allowed to the entire network.
- TCP port 22 Ingress (SSH) is allowed to the entire network.
- Control plane TCP 6443 Ingress (Kubernetes API) is allowed to the entire network.
- Control plane TCP 22623 Ingress (MCS) is allowed to the entire network.

## Internet access for OpenShift Container Platform {#cluster-entitlements_installing-ibm-powervs-vpc}

In OpenShift Container Platform 4.22, you require access to the internet to install your cluster.

You must have internet access to perform the following actions:

- Access Red Hat Hybrid Cloud Console to download the installation program and perform subscription management. If the cluster has internet access and you do not disable Telemetry, that service automatically entitles your cluster.
- Access Quay.io to obtain the packages that are required to install your cluster.
- Obtain the packages that are required to perform cluster updates.

:::warning

If your cluster cannot have direct internet access, you can perform a restricted network installation on some types of infrastructure that you provision. During that process, you download the required content and use it to populate a mirror registry with the installation packages. With some installation types, the environment that you install your cluster in will not require internet access. Before you update the cluster, you update the content of the mirror registry.

:::

## Generating a key pair for cluster node SSH access {#ssh-agent-using_installing-ibm-powervs-vpc}

During an OpenShift Container Platform installation, you can provide an SSH public key to the installation program. The key is passed to the Red Hat Enterprise Linux CoreOS (RHCOS) nodes through their Ignition config files and is used to authenticate SSH access to the nodes. The key is added to the `~/.ssh/authorized_keys` list for the `core` user on each node, which enables password-less authentication.

The key is added to the `~/.ssh/authorized_keys` list for the `core` user on each node, which enables password-less authentication. After the key is passed to the nodes, you can use the key pair to SSH in to the RHCOS nodes as the user `core`. To access the nodes through SSH, the private key identity must be managed by SSH for your local user.

If you want to SSH in to your cluster nodes to perform installation debugging or disaster recovery, you must provide the SSH public key during the installation process. The `./openshift-install gather` command also requires the SSH public key to be in place on the cluster nodes.

:::warning

Do not skip this procedure in production environments, where disaster recovery and debugging is required.

:::

:::note

You must use a local key, not one that you configured with platform-specific approaches.

:::

**Procedure**

1. If you do not have an existing SSH key pair on your local machine to use for authentication onto your cluster nodes, create one. For example, on a computer that uses a Linux operating system, run the following command:
   ```terminal
   $ ssh-keygen -t ed25519 -N '' -f <path>/<file_name>
   ```

   Specifies the path and file name, such as `~/.ssh/id_ed25519`, of the new SSH key. If you have an existing key pair, ensure your public key is in the your `~/.ssh` directory.

   :::note

   If you plan to install an OpenShift Container Platform cluster that uses the RHEL cryptographic libraries that have been submitted to NIST for FIPS 140-2/140-3 Validation on only the `x86_64`, `ppc64le`, and `s390x` architectures, do not create a key that uses the `ed25519` algorithm. Instead, create a key that uses the `rsa` or `ecdsa` algorithm.

   :::
2. View the public SSH key:
   ```terminal
   $ cat <path>/<file_name>.pub
   ```

   For example, run the following to view the `~/.ssh/id_ed25519.pub` public key:

   ```terminal
   $ cat ~/.ssh/id_ed25519.pub
   ```
3. Add the SSH private key identity to the SSH agent for your local user, if it has not already been added. SSH agent management of the key is required for password-less SSH authentication onto your cluster nodes, or if you want to use the `./openshift-install gather` command.
   :::note

   On some distributions, default SSH private key identities such as `~/.ssh/id_rsa` and `~/.ssh/id_dsa` are managed automatically.

   :::

   1. If the `ssh-agent` process is not already running for your local user, start it as a background task:
      ```terminal
      $ eval "$(ssh-agent -s)"
      ```

      ```terminal title="Example output"
      Agent pid 31874
      ```

      :::note

      If your cluster is in FIPS mode, only use FIPS-compliant algorithms to generate the SSH key. The key must be either RSA or ECDSA.

      :::
4. Add your SSH private key to the `ssh-agent`:
   ```terminal
   $ ssh-add <path>/<file_name>
   ```

   Specifies the path and file name for your SSH private key, such as `~/.ssh/id_ed25519`

   ```terminal title="Example output"
   Identity added: /home/<you>/<path>/<file_name> (<computer_name>)
   ```

**Next steps**

- When you install OpenShift Container Platform, provide the SSH public key to the installation program.

## Obtaining the installation program {#installation-obtaining-installer_installing-ibm-powervs-vpc}

Before you install OpenShift Container Platform, download the installation file on

the host you are using for installation.

**Prerequisites**

- You have a computer that runs Linux or macOS, with 500 MB of local disk space.

**Procedure**

1. Go to the [Cluster Type](https://console.redhat.com/openshift/install) page on the Red Hat Hybrid Cloud Console. If you have a Red Hat account, log in with your credentials. If you do not, create an account.
   :::tip

   You can also [download the binaries for a specific OpenShift Container Platform release](https://mirror.openshift.com/pub/openshift-v4/clients/ocp/).

   :::
2. Select your infrastructure provider from the **Run it yourself** section of the page.
3. Select your host operating system and architecture from the dropdown menus under **OpenShift Installer** and click **Download Installer**.
4. Place the downloaded file in the directory where you want to store the installation configuration files.
   :::warning

   - The installation program creates several files on the computer that you use to install your cluster. You must keep the installation program and the files that the installation program creates after you finish installing the cluster. Both of the files are required to delete the cluster.
   - Deleting the files created by the installation program does not remove your cluster, even if the cluster failed during installation. To remove your cluster, complete the OpenShift Container Platform uninstallation procedures for your specific cloud provider.

   :::
5. Extract the installation program. For example, on a computer that uses a Linux operating system, run the following command:
   ```terminal
   $ tar -xvf openshift-install-linux.tar.gz
   ```
6. Download your installation [pull secret from Red Hat OpenShift Cluster Manager](https://console.redhat.com/openshift/install/pull-secret). This pull secret allows you to authenticate with the services that are provided by the included authorities, including Quay.io, which serves the container images for OpenShift Container Platform components.
   :::tip

   Alternatively, you can retrieve the installation program from the [Red Hat Customer Portal](https://access.redhat.com/downloads/content/290/), where you can specify a version of the installation program to download. However, you must have an active subscription to access this page.

   :::

## Exporting the API key {#installation-ibm-cloud-export-variables_installing-ibm-powervs-vpc}

You must set the API key you created as a global variable; the installation program ingests the variable during startup to set the API key.

**Prerequisites**

- You have created either a user API key or service ID API key for your IBM Cloud(R) account.

**Procedure**

- Export your API key for your account as a global variable:
  ```terminal
  $ export IBMCLOUD_API_KEY=<api_key>
  ```

  :::warning

  You must set the variable name exactly as specified; the installation program expects the variable name to be present during startup.

  :::

## Creating the installation configuration file {#installation-initializing_installing-ibm-powervs-vpc}

You can customize the OpenShift Container Platform cluster you install on

**Prerequisites**

- You have the OpenShift Container Platform installation program and the pull secret for your cluster.

**Procedure**

1. Create the `install-config.yaml` file.
   1. Change to the directory that contains the installation program and run the following command:
      ```terminal
      $ ./openshift-install create install-config --dir <installation_directory>
      ```

      - `<installation_directory>`: For `<installation_directory>`, specify the directory name to store the files that the installation program creates.
        When specifying the directory:
      - Use an empty directory. Some installation assets, such as bootstrap X.509 certificates, have short expiration intervals, therefore you must not reuse an installation directory. If you want to reuse individual files from another cluster installation, you can copy them into your directory. However, the file names for the installation assets might change between releases. Use caution when copying installation files from an earlier OpenShift Container Platform version.
        :::note

        Always delete the `~/.powervs` directory to avoid reusing a stale configuration. Run the following command:

        ```terminal
        $ rm -rf ~/.powervs
        ```

        :::
   2. At the prompts, provide the configuration details for your cloud:
      1. Optional: Select an SSH key to use to access your cluster machines.
         :::note

         For production OpenShift Container Platform clusters on which you want to perform installation debugging or disaster recovery, specify an SSH key that your `ssh-agent` process uses.

         :::
      2. Select **powervs** as the platform to target.
      3. Select the region to deploy the cluster to.
      4. Select the zone to deploy the cluster to.
      5. Select the base domain to deploy the cluster to. The base domain corresponds to the public DNS zone that you created for your cluster.
      6. Enter a descriptive name for your cluster.
2. Modify the `install-config.yaml` file. You can find more information about the available parameters in the "Installation configuration parameters" section.
3. Back up the `install-config.yaml` file so that you can use it to install multiple clusters.
   :::warning

   The `install-config.yaml` file is consumed during the installation process. If you want to reuse the file, you must back it up now.

   :::

**Additional resources**

- [Installation configuration parameters for IBM Power(R) Virtual Server](/docs/installing/installing_ibm_powervs/installation-config-parameters-ibm-power-vs#installation-config-parameters-ibm-power-vs)

### Minimum resource requirements for cluster installation {#installation-minimum-resource-requirements_installing-ibm-powervs-vpc}

To ensure that your OpenShift Container Platform cluster runs as expected, each cluster machine must meet minimum CPU, memory, and storage requirements.

**Minimum resource requirements**

<table>
<thead>
<tr>
  <th>Machine</th>
  <th>Operating system</th>
  <th>vCPU</th>
  <th>Virtual RAM</th>
  <th>Storage</th>
  <th>Input/Output Per Second (IOPS)</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Bootstrap</td>
  <td>RHCOS</td>
  <td>4</td>
  <td>16 GB</td>
  <td>100 GB</td>
  <td>300</td>
</tr>
<tr>
  <td>Control plane</td>
  <td>RHCOS</td>
  <td>4</td>
  <td>16 GB</td>
  <td>100 GB</td>
  <td>300</td>
</tr>
<tr>
  <td>Compute</td>
  <td>RHCOS</td>
  <td>2</td>
  <td>8 GB</td>
  <td>100 GB</td>
  <td>300</td>
</tr>
</tbody>
</table>

- One vCPU is equal to one physical core when simultaneous multithreading (SMT), or Hyper-Threading, is not enabled. When enabled, use the following formula to calculate the corresponding ratio: (threads per core × cores) × sockets = vCPUs.
- OpenShift Container Platform and Kubernetes are sensitive to disk performance, and Red Hat recommends faster storage, particularly for etcd on the control plane nodes which require a 10 ms p99 fsync duration. On many cloud platforms, storage size and IOPS scale together, so you might need to provision more storage to get enough performance.
- As with all user-provisioned installations, if you choose to use RHEL compute machines in your cluster, you take responsibility for all operating system life cycle management and maintenance, including performing system updates, applying patches, and completing all other required tasks. OpenShift Container Platform 4.10 and later do not support RHEL 7 compute machines.

:::note

In OpenShift Container Platform version 4.22, RHCOS uses RHEL version 9.8, which updates the micro-architecture requirements. Each architecture requires the following minimum instruction set architectures (ISA):

- x86-64 architecture requires x86-64-v2 ISA
- ARM64 architecture requires ARMv8.0-A ISA
- ppc64le architecture requires IBM(R) Power9 ISA
- s390x architecture requires IBM(R) z14 ISA

For more information, see [Architectures](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html-single/9.8_release_notes/index#architectures) in the RHEL documentation.

:::

If an instance type for your platform meets the minimum requirements for cluster machines, it is supported to use in OpenShift Container Platform.

**Additional resources**

- [Optimizing storage](/docs/scalability_and_performance/optimization/optimizing-storage#optimizing-storage)

### Sample customized install-config.yaml file for IBM Power Virtual Server {#installation-ibm-power-vs-config-yaml_installing-ibm-powervs-vpc}

You can customize the `install-config.yaml` file to specify more details about your OpenShift Container Platform cluster’s platform or change the values of the required parameters.

:::warning

This sample YAML file is for reference only. You must obtain your `install-config.yaml` file by using the installation program and change it.

:::

```yaml
apiVersion: v1
baseDomain: example.com
compute:
- architecture: ppc64le
  hyperthreading: Enabled
  name: worker
  platform:
    powervs:
      smtLevel: 8
  replicas: 3
controlPlane:
  architecture: ppc64le
  hyperthreading: Enabled
  name: master
  platform:
    powervs:
      smtLevel: 8
  replicas: 3
metadata:
  creationTimestamp: null
  name: example-cluster-existing-vpc
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  machineNetwork:
  - cidr: 192.168.0.0/24
  networkType: OVNKubernetes
  serviceNetwork:
  - 172.30.0.0/16
platform:
  powervs:
    userID: ibm-user-id
    powervsResourceGroup: "ibmcloud-resource-group"
    region: powervs-region
    vpcRegion: vpc-region
    vpcName: name-of-existing-vpc
    zone: powervs-zone
    serviceInstanceGUID: "powervs-region-service-instance-guid"
credentialsMode: Manual
publish: External
pullSecret: '{"auths": ...}'
fips: false
sshKey: ssh-ed25519 AAAA...
```

where:

`compute`
: Specifies parameters where, if you do not provide values, the installation program provides the default value. The first line of the `compute` section must begin with a hyphen, `-`. Both sections currently define a single machine pool.

`compute.hyperthreading`
: Specifies whether to enable or disable simultaneous multithreading, or `hyperthreading`. By default, simultaneous multithreading is enabled to increase the performance of your machines' cores. You can disable it by setting the parameter value to `Disabled`. If you disable simultaneous multithreading in some cluster machines, you must disable it in all cluster machines.

`compute.platform.powervs.smtLevel`
: Specifies the level of SMT to set to the compute machines. The supported values are 1, 2, 4, 8, `'off'`, and `'on'`. The default value is 8. The smtLevel `'off'` sets SMT to off, and smtLevel `'on'` sets SMT to the default value 8 on the cluster nodes.

`controlPlane`
: Specifies parameters where, if you do not provide values, the installation program provides the default value. The `controlPlane` section is a single mapping, and its first line must not begin with a hyphen. Only one control plane pool is used.

`controlPlane.hyperthreading`
: Specifies whether to enable or disable simultaneous multithreading, or `hyperthreading`. By default, simultaneous multithreading is enabled to increase the performance of your machines' cores. You can disable it by setting the parameter value to `Disabled`. If you disable simultaneous multithreading in some cluster machines, you must disable it in all cluster machines.

  :::warning

  When multithreading (SMT) is disabled, each vCPU is equivalent to one physical core. Disable SMT only if you have specific requirements.

  :::

`controlPlane.platform.powervs.smtLevel`
: Specifies the level of SMT to set to the control plane. The supported values are 1, 2, 4, 8, `'off'`, and `'on'`. The default value is 8. The smtLevel `'off'` sets SMT to off, and smtLevel `'on'` sets SMT to the default value 8 on the cluster nodes.

  :::note

  When simultaneous multithreading (SMT) is not enabled, one vCPU is equivalent to one physical core. When enabled, total vCPUs is computed as (Thread(s) per core * Core(s) per socket) * Socket(s). The smtLevel controls the threads per core. Lower SMT levels may require additional assigned cores when deploying the cluster nodes. You can do this by setting the `'processors'` parameter in the `install-config.yaml` file to an appropriate value to meet the requirements for deploying OpenShift Container Platform successfully.

  :::

`networking.clusterNetwork.cidr`
: Specifies the CIDR. The machine CIDR must contain the subnets for the compute machines and control plane machines.

`networking.networkType`
: Specifies the cluster network plugin for installation. The supported value is `OVNKubernetes`.

`platform.powervs.vpcName`
: Specifies the name of an existing VPC.

`publish`
: Specifies how to publish the user-facing endpoints of your cluster.

`pullSecret`
: Specifies your pull secret. The installation program prompts you for this value. This value is required.

`sshKey`
: Specifies the SSH key to use to access the machines in your cluster. This value is optional.

  :::note

  For production OpenShift Container Platform clusters on which you want to perform installation debugging or disaster recovery, specify an SSH key that your `ssh-agent` process uses.

  :::

### Configuring the cluster-wide proxy during installation {#installation-configure-proxy_installing-ibm-powervs-vpc}

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
   <dd>Specifies a comma-separated list of destination domain names, IP addresses, or other network CIDRs to exclude from proxying. Preface a domain with <code>.</code> to match subdomains only. For example, <code>.y.com</code> matches <code>x.y.com</code>, but not <code>y.com</code>. Use <code>*</code> to bypass the proxy for all destinations.</dd>
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

## Manually creating IAM {#manually-create-iam-ibm-cloud_installing-ibm-powervs-vpc}

To install OpenShift Container Platform, the Cloud Credential Operator (CCO) must operate in manual mode. While the installation program configures the CCO for manual mode, you must specify the identity and access management secrets for your cloud provider.

You can use the Cloud Credential Operator (CCO) utility (`ccoctl`) to create the required IBM Cloud(R) resources.

**Prerequisites**

- You have configured the `ccoctl` binary.
- You have an existing `install-config.yaml` file.

**Procedure**

1. Edit the `install-config.yaml` configuration file so that the file includes the `credentialsMode` parameter set to `Manual`.
   ```yaml title="Example install-config.yaml configuration file"
   apiVersion: v1
   baseDomain: cluster1.example.com
   credentialsMode: Manual
   compute:
   - architecture: ppc64le
     hyperthreading: Enabled
   ```

   where:

   <dl>
   <dt><code>credentialsMode</code></dt>
   <dd>Specifies the CCO credentials mode. Set the value to <code>Manual</code>.</dd>
   </dl>
2. To generate the manifests, run the following command from the directory that includes the installation program:
   ```terminal
   $ ./openshift-install create manifests --dir <installation_directory>
   ```
3. From the directory that includes the installation program, set a `$RELEASE_IMAGE` variable with the release image from your installation file by running the following command:
   ```terminal
   $ RELEASE_IMAGE=$(./openshift-install version | awk '/release image/ {print $3}')
   ```
4. Extract the list of `CredentialsRequest` custom resources (CRs) from the OpenShift Container Platform release image by running the following command:
   ```terminal
   $ oc adm release extract \
     --from=$RELEASE_IMAGE \
     --credentials-requests \
     --included \
     --install-config=<path_to_directory_with_installation_configuration>/install-config.yaml \
     --to=<path_to_directory_for_credentials_requests>
   ```

   where:

   <dl>
   <dt><code>--included</code></dt>
   <dd>Specifies that only the manifests that your specific cluster configuration requires are included.</dd>
   <dt><code>--install-config</code></dt>
   <dd>Specifies the location of the <code>install-config.yaml</code> file.</dd>
   <dt><code>--to</code></dt>
   <dd>Specifies the path to the directory where you want to store the <code>CredentialsRequest</code> objects. If the specified directory does not exist, this command creates it. This command creates a YAML file for each <code>CredentialsRequest</code> object.</dd>
   </dl>

   ```yaml title="Sample CredentialsRequest object"
     apiVersion: cloudcredential.openshift.io/v1
     kind: CredentialsRequest
     metadata:
       labels:
         controller-tools.k8s.io: "1.0"
       name: openshift-image-registry-ibmcos
       namespace: openshift-cloud-credential-operator
     spec:
       secretRef:
         name: installer-cloud-credentials
         namespace: openshift-image-registry
       providerSpec:
         apiVersion: cloudcredential.openshift.io/v1
         kind: IBMCloudProviderSpec
         policies:
         - attributes:
           - name: serviceName
             value: cloud-object-storage
           roles:
           - crn:v1:bluemix:public:iam::::role:Viewer
           - crn:v1:bluemix:public:iam::::role:Operator
           - crn:v1:bluemix:public:iam::::role:Editor
           - crn:v1:bluemix:public:iam::::serviceRole:Reader
           - crn:v1:bluemix:public:iam::::serviceRole:Writer
         - attributes:
           - name: resourceType
             value: resource-group
           roles:
           - crn:v1:bluemix:public:iam::::role:Viewer
   ```
5. Create the service ID for each credential request, assign the policies defined, create an API key, and generate the secret:
   ```terminal
   $ ccoctl ibmcloud create-service-id \
     --credentials-requests-dir=<path_to_credential_requests_directory> \
     --name=<cluster_name> \
     --output-dir=<installation_directory> \
     --resource-group-name=<resource_group_name>
   ```

   where:

   <dl>
   <dt><code>&lt;path_to_credential_requests_directory&gt;</code></dt>
   <dd>Specifies the directory that has the files for the <code>CredentialsRequest</code> objects.</dd>
   <dt><code>&lt;cluster_name&gt;</code></dt>
   <dd>Specifies the name of the OpenShift Container Platform cluster.</dd>
   <dt><code>&lt;installation_directory&gt;</code></dt>
   <dd>Specifies the directory in which you want the <code>ccoctl</code> utility to create objects. By default, the utility creates objects in the directory in which you run the commands. This parameter is optional.</dd>
   <dt><code>&lt;resource_group_name&gt;</code></dt>
   <dd>Specifies the name of the resource group used for scoping the access policies. This parameter is optional.</dd>
   </dl>

   :::note

   If you enabled Technology Preview features by using the `TechPreviewNoUpgrade` feature set for your cluster, you must include the `--enable-tech-preview` parameter in the configuration for the `CredentialsRequest` object.

   If you provided a wrong resource group name, the installation fails during the bootstrap phase. To find the correct resource group name, run the following command:

   ```terminal
   $ grep resourceGroup <installation_directory>/manifests/cluster-infrastructure-02-config.yml
   ```

   :::

**Verification**

- Check that the appropriate secrets exist in the `manifests` directory of your cluster.

## Deploying the cluster {#installation-launching-installer_installing-ibm-powervs-vpc}

To deploy your OpenShift Container Platform cluster, you can initialize installation by running the `openshift-install create cluster` command from the directory that contains the installation program. The installation program provisions infrastructure and completes cluster setup.

:::warning

You can run the `create cluster` command of the installation program only once, during initial installation.

:::

**Prerequisites**

- You have configured an account with the cloud platform that hosts your cluster.
- You have the OpenShift Container Platform installation program and the pull secret for your cluster.
- You have verified that the cloud provider account on your host has the correct permissions to deploy the cluster. An account with incorrect permissions causes the installation process to fail with an error message that displays the missing permissions.

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

## Installing the OpenShift CLI on Linux {#cli-installing-cli-linux_installing-ibm-powervs-vpc}

To manage your cluster and deploy applications from the command line on Linux, install the OpenShift CLI (`oc`) binary. You can download the OpenShift CLI (`oc`) from the Red  Customer Portal.

:::warning

If you installed an earlier version of `oc`, you cannot use it to complete all of the commands in OpenShift Container Platform.

Download and install the new version of `oc`.

:::

**Procedure**

1. Navigate to the [Download OpenShift Container Platform](https://access.redhat.com/downloads/content/290) page on the Red Hat Customer Portal.
2. Select the architecture from the **Product Variant** list.
3. Select the appropriate version from the **Version** list.
4. Click **Download Now** next to the **OpenShift v4.22 Linux Clients** entry and save the file.
5. Unpack the archive:
   ```terminal
   $ tar xvf <file>
   ```
6. Place the `oc` binary in a directory that is on your `PATH`.
   To check your `PATH`, execute the following command:

   ```terminal
   $ echo $PATH
   ```

**Verification**

- After you install the OpenShift CLI, it is available using the `oc` command:
  ```terminal
  $ oc <command>
  ```

## Installing the OpenShift CLI on Windows {#cli-installing-cli-windows_installing-ibm-powervs-vpc}

To manage your cluster and deploy applications from the command line on Windows, install the OpenShift CLI (`oc`) binary. You can download the OpenShift CLI (`oc`) from the Red  Customer Portal.

:::warning

If you installed an earlier version of `oc`, you cannot use it to complete all of the commands in OpenShift Container Platform.

Download and install the new version of `oc`.

:::

**Procedure**

1. Navigate to the [Download OpenShift Container Platform](https://access.redhat.com/downloads/content/290) page on the Red Hat Customer Portal.
2. Select the appropriate version from the **Version** list.
3. Click **Download Now** next to the **OpenShift v4.22 Windows Client** entry and save the file.
4. Extract the archive with a ZIP program.
5. Move the `oc` binary to a directory that is on your `PATH` variable.
   To check your `PATH` variable, open the command prompt and execute the following command:

   ```terminal
   C:\> path
   ```

**Verification**

- After you install the OpenShift CLI, it is available using the `oc` command:
  ```terminal
  C:\> oc <command>
  ```

## Installing the OpenShift CLI on macOS {#cli-installing-cli-macos_installing-ibm-powervs-vpc}

To manage your cluster and deploy applications from the command line on macOS, install the OpenShift CLI (`oc`) binary. You can download the OpenShift CLI (`oc`) from the Red  Customer Portal.

:::warning

If you installed an earlier version of `oc`, you cannot use it to complete all of the commands in OpenShift Container Platform.

Download and install the new version of `oc`.

:::

**Procedure**

1. Navigate to the [Download OpenShift Container Platform](https://access.redhat.com/downloads/content/290) page on the Red Hat Customer Portal.
2. Select the architecture from the **Product Variant** list.
3. Select the appropriate version from the **Version** list.
4. Click **Download Now** next to the **OpenShift v4.22 macOS Clients** entry and save the file.
   :::note

   For macOS arm64, choose the **OpenShift v4.22 macOS arm64 Client** entry.

   :::
5. Unpack and unzip the archive.
6. Move the `oc` binary to a directory on your `PATH` variable.
   To check your `PATH` variable, open a terminal and execute the following command:

   ```terminal
   $ echo $PATH
   ```

**Verification**

- Verify your installation by using an `oc` command:
  ```terminal
  $ oc <command>
  ```

## Logging in to the cluster by using the CLI {#cli-logging-in-kubeadmin_installing-ibm-powervs-vpc}

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

**Additional resources**

- [Accessing the web console](/docs/web_console/web-console#web-console)

## Telemetry access for OpenShift Container Platform {#cluster-telemetry_installing-ibm-powervs-vpc}

To provide metrics about cluster health and the success of updates, the Telemetry service requires internet access. When connected, this service runs automatically by default and registers your cluster to [OpenShift Cluster Manager](https://console.redhat.com/openshift).

After you confirm that your [OpenShift Cluster Manager](https://console.redhat.com/openshift) inventory is correct, either maintained automatically by Telemetry or manually by using OpenShift Cluster Manager,use subscription watch to track your OpenShift Container Platform subscriptions at the account or multi-cluster level. For more information about subscription watch, see "Data Gathered and Used by Red Hat’s subscription services" in the *Additional resources* section.

**Additional resources**

- [About remote health monitoring](/docs/support/remote_health_monitoring/about-remote-health-monitoring#about-remote-health-monitoring)
- [Customize your cluster](/docs/post_installation_configuration/cluster-tasks#available_cluster_customizations)
- [Remote health reporting](/docs/support/remote_health_monitoring/remote-health-reporting#remote-health-reporting)
