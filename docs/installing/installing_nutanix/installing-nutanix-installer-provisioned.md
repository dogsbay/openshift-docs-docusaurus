---
title: Installing a cluster on Nutanix
sidebar_position: 3
---

# Installing a cluster on Nutanix {#installing-nutanix-installer-provisioned}

<a id="installing-nutanix-installer-provisioned"></a>

In OpenShift Container Platform version 4.22, you can choose one of the following options to install a cluster on your Nutanix instance:

**Using installer-provisioned infrastructure**: Use the procedures in the following sections to use installer-provisioned infrastructure. Installer-provisioned infrastructure is ideal for installing in connected or disconnected network environments. The installer-provisioned infrastructure includes an installation program that provisions the underlying infrastructure for the cluster.

**Using the Assisted Installer**: The Assisted Installer is hosted at console.redhat.com. The Assisted Installer cannot be used in disconnected environments. The Assisted Installer does not provision the underlying infrastructure for the cluster, so you must provision the infrastructure before you run the Assisted Installer. Installing with the Assisted Installer also provides integration with Nutanix, enabling autoscaling.

**Using user-provisioned infrastructure**: You provision the underlying infrastructure yourself and then complete the relevant installation steps.

**Additional resources**

- [Assisted Installer](https://access.redhat.com/documentation/en-us/assisted_installer_for_openshift_container_platform)
- [Installing an on-premise cluster using the Assisted Installer](/docs/installing/installing_on_prem_assisted/installing-on-prem-assisted#installing-on-prem-assisted)
- [Installing a cluster on any platform](/docs/installing/installing_platform_agnostic/installing-platform-agnostic#installing-platform-agnostic)

**Prerequisites**

- You have reviewed details about the OpenShift Container Platform installation and update processes.
- The installation program requires access to port 9440 on Prism Central and Prism Element. You verified that port 9440 is accessible.
- If you use a firewall, you have met these prerequisites:
  - You confirmed that port 9440 is accessible. Control plane nodes must be able to reach Prism Central and Prism Element on port 9440 for the installation to succeed.
  - You configured the firewall to grant access to the sites that OpenShift Container Platform requires. This includes the use of Telemetry.
- If your Nutanix environment is using the default self-signed SSL certificate, replace it with a certificate that is signed by a CA. The installation program requires a valid CA-signed certificate to access to the Prism Central API. For more information about replacing the self-signed certificate, see the Nutanix AOS Security Guide.
  If your Nutanix environment uses an internal CA to issue certificates, you must configure a cluster-wide proxy as part of the installation process. For more information, see "Configuring a custom PKI".

  :::warning

  Use 2048-bit certificates. The installation fails if you use 4096-bit certificates with Prism Central 2022.x.

  :::

## Internet access for OpenShift Container Platform {#cluster-entitlements_installing-nutanix-installer-provisioned}

In OpenShift Container Platform 4.22, you require access to the internet to install your cluster.

You must have internet access to perform the following actions:

- Access Red Hat Hybrid Cloud Console to download the installation program and perform subscription management. If the cluster has internet access and you do not disable Telemetry, that service automatically entitles your cluster.
- Access Quay.io to obtain the packages that are required to install your cluster.
- Obtain the packages that are required to perform cluster updates.

:::warning

If your cluster cannot have direct internet access, you can perform a restricted network installation on some types of infrastructure that you provision. During that process, you download the required content and use it to populate a mirror registry with the installation packages. With some installation types, the environment that you install your cluster in will not require internet access. Before you update the cluster, you update the content of the mirror registry.

:::

## Internet access for Prism Central {#nutanix-entitlements_installing-nutanix-installer-provisioned}

Prism Central requires internet access to obtain the Red Hat Enterprise Linux CoreOS (RHCOS) image that is required to install the cluster. The RHCOS image for Nutanix is available at `rhcos.mirror.openshift.com`.

## Generating a key pair for cluster node SSH access {#ssh-agent-using_installing-nutanix-installer-provisioned}

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

## Obtaining the installation program {#installation-obtaining-installer_installing-nutanix-installer-provisioned}

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

## Adding Nutanix root CA certificates to your system trust {#installation-adding-nutanix-root-certificates_installing-nutanix-installer-provisioned}

Because the installation program requires access to the Prism Central API, you must add your Nutanix trusted root CA certificates to your system trust before you install an OpenShift Container Platform cluster.

**Procedure**

1. From the Prism Central web console, download the Nutanix root CA certificates.
2. Extract the compressed file that contains the Nutanix root CA certificates.
3. Add the files for your operating system to the system trust. For example, on a Fedora operating system, run the following command:
   ```terminal
   # cp certs/lin/* /etc/pki/ca-trust/source/anchors
   ```
4. Update your system trust. For example, on a Fedora operating system, run the following command:
   ```terminal
   # update-ca-trust extract
   ```

## Creating the installation configuration file {#installation-initializing_installing-nutanix-installer-provisioned}

You can customize the OpenShift Container Platform cluster you install on Nutanix.

**Prerequisites**

- You have the OpenShift Container Platform installation program and the pull secret for your cluster.
- You have verified that you have met the Nutanix networking requirements. For more information, see "Preparing to install on Nutanix".

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
      2. Select **nutanix** as the platform to target.
      3. Enter the Prism Central domain name or IP address.
      4. Enter the port that is used to log into Prism Central.
      5. Enter the credentials that are used to log into Prism Central.
         The installation program connects to Prism Central.
      6. Select the Prism Element that will manage the OpenShift Container Platform cluster.
      7. Select the network subnet to use.
      8. Enter the virtual IP address that you configured for control plane API access.
      9. Enter the virtual IP address that you configured for cluster ingress.
      10. Enter the base domain. This base domain must be the same one that you configured in the DNS records.
      11. Enter a descriptive name for your cluster.

      The cluster name you enter must match the cluster name you specified when configuring the DNS records.
2. Optional: Update one or more of the default configuration parameters in the `install.config.yaml` file to customize the installation.
   For more information about the parameters, see "Installation configuration parameters".

   :::note

   If you are installing a three-node cluster, be sure to set the `compute.replicas` parameter to `0`. This ensures that cluster’s control planes are schedulable. For more information, see "Installing a three-node cluster on Nutanix".

   :::
3. Back up the `install-config.yaml` file so that you can use it to install multiple clusters.
   :::warning

   The `install-config.yaml` file is consumed during the installation process. If you want to reuse the file, you must back it up now.

   :::

**Additional resources**

- [Installation configuration parameters for Nutanix](/docs/installing/installing_nutanix/installation-config-parameters-nutanix#installation-config-parameters-nutanix)

### Sample customized install-config.yaml file for Nutanix {#installation-nutanix-config-yaml_installing-nutanix-installer-provisioned}

You can customize the `install-config.yaml` file to specify more details about your OpenShift Container Platform cluster’s platform or modify the values of the required parameters.

:::warning

This sample YAML file is provided for reference only. You must obtain your `install-config.yaml` file by using the installation program and modify it.

:::

```yaml
apiVersion: v1
baseDomain: example.com
compute:
- hyperthreading: Enabled
  name: worker
  replicas: 3
  platform:
    nutanix:
      cpus: 2
      coresPerSocket: 2
      memoryMiB: 8196
      osDisk:
        diskSizeGiB: 120
      categories:
      - key: <category_key_name>
        value: <category_value>
controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 3
  platform:
    nutanix:
      cpus: 4
      coresPerSocket: 2
      memoryMiB: 16384
      osDisk:
        diskSizeGiB: 120
      categories:
      - key: <category_key_name>
        value: <category_value>
metadata:
  creationTimestamp: null
  name: test-cluster
networking:
  clusterNetwork:
    - cidr: 10.128.0.0/14
      hostPrefix: 23
  machineNetwork:
    - cidr: 10.0.0.0/16
  networkType: OVNKubernetes
  serviceNetwork:
    - 172.30.0.0/16
platform:
  nutanix:
    apiVIPs:
      - 10.40.142.7
    defaultMachinePlatform:
      bootType: Legacy
      categories:
      - key: <category_key_name>
        value: <category_value>
      project:
        type: name
        name: <project_name>
    ingressVIPs:
      - 10.40.142.8
    prismCentral:
      endpoint:
        address: your.prismcentral.domainname
        port: 9440
      password: <password>
      username: <username>
    prismElements:
    - endpoint:
        address: your.prismelement.domainname
        port: 9440
      uuid: 0005b0f1-8f43-a0f2-02b7-3cecef193712
    subnetUUIDs:
    - c7938dc6-7659-453e-a688-e26020c68e43
    clusterOSImage: http://example.com/images/rhcos-47.83.202103221318-0-nutanix.x86_64.qcow2
credentialsMode: Manual
publish: External
pullSecret: '{"auths": ...}'
fips: false
sshKey: ssh-ed25519 AAAA...
```

The installation program prompts you for the values of `baseDomain`, `metadata.name`, `platform.nutanix.apiVIPs`, `platform.nutanix.ingressVIPs`, `platform.nutanix.prismCentral.endpoint.address`, `platform.nutanix.prismCentral.endpoint.port`, `platform.nutanix.prismCentral.password`, `platform.nutanix.prismCentral.username`, and `pullSecret`.

where:

`compute`
: The `compute` section is a sequence of mappings. The first line of the `compute` section must begin with a hyphen, `-`. Although this section currently defines a single machine pool, it is possible that future versions of OpenShift Container Platform will support defining multiple compute pools during installation.

`hyperthreading`
: Whether to enable or disable simultaneous multithreading, or `hyperthreading`. By default, simultaneous multithreading is enabled to increase the performance of your machines' cores. You can disable it by setting the parameter value to `Disabled`. If you disable simultaneous multithreading in some cluster machines, you must disable it in all cluster machines.

  :::warning

  If you disable simultaneous multithreading, ensure that your capacity planning accounts for the dramatically decreased machine performance.

  :::

`platform.nutanix`
: Optional: Provide additional configuration for the machine pool parameters for the compute and control plane machines.

`categories`
: Optional: Provide one or more pairs of a prism category key and a prism category value. These category key-value pairs must exist in Prism Central. You can provide separate categories to compute machines, control plane machines, or all machines.

`controlPlane`
: The `controlPlane` section is a single mapping. The first line of the `controlPlane` section must not begin with a hyphen. Only one control plane pool is used.

`networkType`
: The cluster network plugin to install. The default value `OVNKubernetes` is the only supported value.

`project`
: Optional: Specify a project with which VMs are associated. Specify either `name` or `uuid` for the project type, and then provide the corresponding UUID or project name. You can associate projects to compute machines, control plane machines, or all machines.

`<password>`
: Required. The installation program prompts you for this value.

`<username>`
: Required. The installation program prompts you for this value.

`clusterOSImage`
: Optional: By default, the installation program downloads and installs the Red Hat Enterprise Linux CoreOS (RHCOS) image. If Prism Central does not have internet access, you can override the default behavior by hosting the RHCOS image on any HTTP server and pointing the installation program to the image.

`fips`
: Whether to enable or disable FIPS mode. By default, FIPS mode is not enabled. If FIPS mode is enabled, the Red Hat Enterprise Linux CoreOS (RHCOS) machines that OpenShift Container Platform runs on bypass the default Kubernetes cryptography suite and use the cryptography modules that are provided with RHCOS instead.

  :::warning

  When running Red Hat Enterprise Linux (RHEL) or Red Hat Enterprise Linux CoreOS (RHCOS) booted in FIPS mode, OpenShift Container Platform core components use the RHEL cryptographic libraries that have been submitted to NIST for FIPS 140-2/140-3 Validation on only the x86_64, ppc64le, and s390x architectures.

  :::

`sshKey`
: Optional: You can provide the `sshKey` value that you use to access the machines in your cluster.

  :::note

  For production OpenShift Container Platform clusters on which you want to perform installation debugging or disaster recovery, specify an SSH key that your `ssh-agent` process uses.

  :::

### Configuring failure domains {#installation-configuring-nutanix-failure-domains_installing-nutanix-installer-provisioned}

Failure domains improve the fault tolerance of an OpenShift Container Platform cluster by distributing control plane and compute machines across multiple Nutanix Prism Elements (clusters).

:::tip

It is recommended that you configure three failure domains to ensure high-availability.

:::

**Prerequisites**

- You have an installation configuration file (`install-config.yaml`).

**Procedure**

1. Edit the `install-config.yaml` file and add the following stanza to configure the first failure domain:
   ```yaml
   apiVersion: v1
   baseDomain: example.com
   compute:
   # ...
   platform:
     nutanix:
       failureDomains:
       - name: <failure_domain_name>
         prismElement:
           name: <prism_element_name>
           uuid: <prism_element_uuid>
         subnetUUIDs:
         - <network_uuid>
   # ...
   ```

   where:

   <dl>
   <dt><code>&lt;failure_domain_name&gt;</code></dt>
   <dd>Specifies a unique name for the failure domain. The name is limited to 64 or fewer characters, which can include lower-case letters, digits, and a dash (<code>-</code>). The dash cannot be in the leading or ending position of the name.</dd>
   <dt><code>&lt;prism_element_name&gt;</code></dt>
   <dd>Optional. Specifies the name of the Prism Element.</dd>
   <dt><code>&lt;prism_element_uuid</code>&gt;</dt>
   <dd>Specifies the UUID of the Prism Element.</dd>
   <dt><code>&lt;network_uuid</code>&gt;</dt>
   <dd>Specifies the one or more UUIDs of the Prism Element subnet objects. Among them, one of the subnet’s IP address prefixes (CIDRs) must contain the virtual IP addresses that the OpenShift Container Platform cluster uses. A maximum of 32 subnets for each failure domain (Prism Element) in an OpenShift Container Platform cluster is supported. All <code>subnetUUID</code> values must be unique.</dd>
   </dl>
2. As required, configure additional failure domains.
3. To distribute control plane and compute machines across the failure domains, do one of the following:
   - If compute and control plane machines can share the same set of failure domains, add the failure domain names under the cluster’s default machine configuration.
     ```yaml title="Example of control plane and compute machines sharing a set of failure domains"
     apiVersion: v1
     baseDomain: example.com
     compute:
     # ...
     platform:
       nutanix:
         defaultMachinePlatform:
           failureDomains:
             - failure-domain-1
             - failure-domain-2
             - failure-domain-3
     # ...
     ```
   - If compute and control plane machines must use different failure domains, add the failure domain names under the respective machine pools.
     ```yaml title="Example of control plane and compute machines using different failure domains"
     apiVersion: v1
     baseDomain: example.com
     compute:
     # ...
     controlPlane:
       platform:
         nutanix:
           failureDomains:
             - failure-domain-1
             - failure-domain-2
             - failure-domain-3
     # ...
     compute:
       platform:
         nutanix:
           failureDomains:
             - failure-domain-1
             - failure-domain-2
     # ...
     ```
4. Save the file.

### Configuring the cluster-wide proxy during installation {#installation-configure-proxy_installing-nutanix-installer-provisioned}

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

## Installing the OpenShift CLI on Linux {#cli-installing-cli-linux_installing-nutanix-installer-provisioned}

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

## Installing the OpenShift CLI on Windows {#cli-installing-cli-windows_installing-nutanix-installer-provisioned}

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

## Installing the OpenShift CLI on macOS {#cli-installing-cli-macos_installing-nutanix-installer-provisioned}

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

## Configuring IAM for Nutanix {#manually-create-iam-nutanix_installing-nutanix-installer-provisioned}

Installing the cluster requires that the Cloud Credential Operator (CCO) operate in manual mode. While the installation program configures the CCO for manual mode, you must specify the identity and access management secrets.

**Prerequisites**

- You have configured the `ccoctl` binary.
- You have an `install-config.yaml` file.

**Procedure**

1. Create a YAML file that contains the credentials data in the following format:
   ```yaml title="Credentials data format"
   credentials:
   - type: basic_auth
     data:
       prismCentral:
         username: <username_for_prism_central>
         password: <password_for_prism_central>
       prismElements:
       - name: <name_of_prism_element>
         username: <username_for_prism_element>
         password: <password_for_prism_element>
   ```

   where:

   <dl>
   <dt><code>type</code></dt>
   <dd>Specifies the authentication type. Only basic authentication is supported.</dd>
   <dt><code>prismCentral</code></dt>
   <dd>Specifies the Prism Central credentials.</dd>
   <dt><code>prismElements</code></dt>
   <dd>Optional: Specifies the Prism Element credentials.</dd>
   </dl>
2. Set a `$RELEASE_IMAGE` variable with the release image from your installation file by running the following command:
   ```terminal
   $ RELEASE_IMAGE=$(./openshift-install version | awk '/release image/ {print $3}')
   ```
3. Extract the list of `CredentialsRequest` custom resources (CRs) from the OpenShift Container Platform release image by running the following command:
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
   <dd>Includes only the manifests that your specific cluster configuration requires.</dd>
   <dt><code>&lt;path_to_directory_with_installation_configuration&gt;</code></dt>
   <dd>Specifies the location of the <code>install-config.yaml</code> file.</dd>
   <dt><code>&lt;path_to_directory_for_credentials_requests&gt;</code></dt>
   <dd>Specifies the path to the directory where you want to store the <code>CredentialsRequest</code> objects. If the specified directory does not exist, this command creates it.</dd>
   </dl>

   ```yaml title="Sample CredentialsRequest object"
     apiVersion: cloudcredential.openshift.io/v1
     kind: CredentialsRequest
     metadata:
       annotations:
         include.release.openshift.io/self-managed-high-availability: "true"
       labels:
         controller-tools.k8s.io: "1.0"
       name: openshift-machine-api-nutanix
       namespace: openshift-cloud-credential-operator
     spec:
       providerSpec:
         apiVersion: cloudcredential.openshift.io/v1
         kind: NutanixProviderSpec
       secretRef:
         name: nutanix-credentials
         namespace: openshift-machine-api
   ```
4. Use the `ccoctl` tool to process all `CredentialsRequest` objects by running the following command:
   ```terminal
   $ ccoctl nutanix create-shared-secrets \
     --credentials-requests-dir=<path_to_credentials_requests_directory> \
     --output-dir=<ccoctl_output_dir> \
     --credentials-source-filepath=<path_to_credentials_file>
   ```

   where:

   <dl>
   <dt><code>&lt;path_to_credentials_requests_directory&gt;</code></dt>
   <dd>Specifies the path to the directory that contains the files for the component <code>CredentialsRequests</code> objects.</dd>
   <dt><code>&lt;ccoctl_output_dir&gt;</code></dt>
   <dd>Optional: Specifies the directory in which you want the <code>ccoctl</code> utility to create objects. By default, the utility creates objects in the directory in which the commands are run.</dd>
   <dt><code>&lt;path_to_credentials_file&gt;</code></dt>
   <dd>Optional: Specifies the directory that contains the credentials data YAML file. By default, <code>ccoctl</code> expects this file to be in <code>&lt;home_directory&gt;/.nutanix/credentials</code>.</dd>
   </dl>
5. Edit the `install-config.yaml` configuration file so that the `credentialsMode` parameter is set to `Manual`.
   ```yaml title="Example install-config.yaml configuration file"
   apiVersion: v1
   baseDomain: cluster1.example.com
   credentialsMode: Manual
   ...
   ```

   Add the `credentialsMode` line to set the parameter to `Manual`.
6. Create the installation manifests by running the following command:
   ```terminal
   $ openshift-install create manifests --dir <installation_directory>
   ```

   For `<installation_directory>`, specify the path to the directory that contains the `install-config.yaml` file for your cluster.
7. Copy the generated credential files to the target manifests directory by running the following command:
   ```terminal
   $ cp <ccoctl_output_dir>/manifests/*credentials.yaml ./<installation_directory>/manifests
   ```

**Verification**

- Ensure that the appropriate secrets exist in the `manifests` directory.
  ```terminal
  $ ls ./<installation_directory>/manifests
  ```

  ```text title="Example output"
  cluster-config.yaml
  cluster-dns-02-config.yml
  cluster-infrastructure-02-config.yml
  cluster-ingress-02-config.yml
  cluster-network-01-crd.yml
  cluster-network-02-config.yml
  cluster-proxy-01-config.yaml
  cluster-scheduler-02-config.yml
  cvo-overrides.yaml
  kube-cloud-config.yaml
  kube-system-configmap-root-ca.yaml
  machine-config-server-tls-secret.yaml
  openshift-config-secret-pull-secret.yaml
  openshift-cloud-controller-manager-nutanix-credentials-credentials.yaml
  openshift-machine-api-nutanix-credentials-credentials.yaml
  ```

## Adding config map and secret resources required for Nutanix CCM {#nutanix-ccm-config_installing-nutanix-installer-provisioned}

Installations on Nutanix require additional `ConfigMap` and `Secret` resources to integrate with the Nutanix Cloud Controller Manager (CCM).

**Prerequisites**

- You have created a `manifests` directory within your installation directory.

**Procedure**

1. Navigate to the `manifests` directory:
   ```terminal
   $ cd <path_to_installation_directory>/manifests
   ```
2. Create the `cloud-conf` `ConfigMap` file with the name `openshift-cloud-controller-manager-cloud-config.yaml` and add the following information:
   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: cloud-conf
     namespace: openshift-cloud-controller-manager
   data:
     cloud.conf: "{
         \"prismCentral\": {
             \"address\": \"<prism_central_FQDN/IP>\",
             \"port\": 9440,
               \"credentialRef\": {
                   \"kind\": \"Secret\",
                   \"name\": \"nutanix-credentials\",
                   \"namespace\": \"openshift-cloud-controller-manager\"
               }
          },
          \"topologyDiscovery\": {
              \"type\": \"Prism\",
              \"topologyCategories\": null
          },
          \"enableCustomLabeling\": true
        }"
   ```

   For `<prism_central_FQDN/IP>`, specify the Prism Central FQDN or IP address.
3. Verify that the file `cluster-infrastructure-02-config.yml` exists and has the following information:
   ```yaml
   spec:
     cloudConfig:
       key: config
       name: cloud-provider-config
   ```

## Services for a user-managed load balancer {#nw-osp-services-external-load-balancer_installing-nutanix-installer-provisioned}

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

### Configuring a user-managed load balancer {#nw-osp-configuring-external-load-balancer_installing-nutanix-installer-provisioned}

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
     nutanix:
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

## Deploying the cluster {#installation-launching-installer_installing-nutanix-installer-provisioned}

To deploy your OpenShift Container Platform cluster, you can initialize installation by running the `openshift-install create cluster` command from the directory that contains the installation program. The installation program provisions infrastructure and completes cluster setup.

:::warning

You can run the `create cluster` command of the installation program only once, during initial installation.

:::

**Prerequisites**

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

## Configuring the default storage container {#registry-configuring-storage-nutanix_installing-nutanix-installer-provisioned}

After you install the cluster, you must install the Nutanix CSI Operator and configure the default storage container for the cluster.

**Additional resources**

- [Installing the CSI Operator](https://opendocs.nutanix.com/openshift/operators/csi/)
- [Configuring registry storage](https://opendocs.nutanix.com/openshift/post-install/)

## Telemetry access for OpenShift Container Platform {#cluster-telemetry_installing-nutanix-installer-provisioned}

To provide metrics about cluster health and the success of updates, the Telemetry service requires internet access. When connected, this service runs automatically by default and registers your cluster to [OpenShift Cluster Manager](https://console.redhat.com/openshift).

After you confirm that your [OpenShift Cluster Manager](https://console.redhat.com/openshift) inventory is correct, either maintained automatically by Telemetry or manually by using OpenShift Cluster Manager,use subscription watch to track your OpenShift Container Platform subscriptions at the account or multi-cluster level. For more information about subscription watch, see "Data Gathered and Used by Red Hat’s subscription services" in the *Additional resources* section.

**Additional resources**

- [OpenShift Container Platform installation and update processes](/docs/architecture/architecture-installation#architecture-installation)
- [Configuring your firewall to grant required access](/docs/installing/install_config/configuring-firewall#configuring-firewall-module_configuring-firewall)
- [Nutanix AOS Security Guide](https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Security-Guide-v6_1:mul-security-ssl-certificate-pc-t.html)
- [Configuring a custom PKI](/docs/networking/configuring_network_settings/configuring-a-custom-pki#configuring-a-custom-pki)
- [About remote health monitoring](/docs/support/remote_health_monitoring/about-remote-health-monitoring#about-remote-health-monitoring)
- [Remote health reporting](/docs/support/remote_health_monitoring/remote-health-reporting#remote-health-reporting)
- [Customize your cluster](/docs/post_installation_configuration/cluster-tasks#available_cluster_customizations)
