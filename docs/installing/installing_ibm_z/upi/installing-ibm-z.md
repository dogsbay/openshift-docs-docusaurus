---
title: Installing a cluster with z/VM on IBM Z and IBM LinuxONE
sidebar_position: 3
---

# Installing a cluster with z/VM on IBM Z and IBM LinuxONE {#installing-ibm-z}

<a id="installing-ibm-z"></a>

You can install OpenShift Container Platform on IBM Z(R) or IBM(R) LinuxONE by using z/VM on infrastructure that you provision, giving you full control over networking, storage, and compute resources.

:::note

While this document refers only to IBM Z(R), all information in it also applies to IBM(R) LinuxONE.

:::

## Prerequisites for installing a cluster on IBM Z {#prereqs-ibm-z-upi_installing-ibm-z}

Before you install OpenShift Container Platform on IBM Z(R) using user-provisioned infrastructure, you must complete prerequisite tasks that prepare your hardware, storage, and network environment.

- You have completed the tasks in preparing to install a cluster on IBM Z(R) using user-provisioned infrastructure.
- You reviewed details about the OpenShift Container Platform installation and update processes.
- You read the documentation on selecting a cluster installation method and preparing it for users.
- Before you begin the installation process, you must clean the installation directory. This ensures that the required installation files are created and updated during the installation process.
- You provisioned persistent storage by using OpenShift Data Foundation or other supported storage protocols for your cluster. To deploy a private image registry, you must set up persistent storage with `ReadWriteMany` access.
- If you use a firewall, you configured it to allow the sites that your cluster requires access to.
  :::note

  Be sure to also review this site list if you are configuring a proxy.

  :::

**Additional resources**

- [Preparing to install a cluster on IBM Z using user-provisioned infrastructure](/docs/installing/installing_ibm_z/upi/upi-ibm-z-preparing-to-install#upi-ibm-z-preparing-to-install)
- [OpenShift Container Platform installation and update](/docs/architecture/architecture-installation#architecture-installation)
- [Selecting a cluster installation method and preparing it for users](/docs/installing/overview/installing-preparing#installing-preparing)
- [Persistent storage using OpenShift Data Foundation](/docs/storage/persistent_storage/persistent-storage-ocs#persistent-storage-ocs)
- [Configuring your firewall](/docs/installing/install_config/configuring-firewall#configuring-firewall-module_configuring-firewall)

## Preparing the user-provisioned infrastructure {#installation-infrastructure-user-infra_installing-ibm-z}

Before you install OpenShift Container Platform on user-provisioned infrastructure, you must prepare the underlying infrastructure.

This section provides details about the high-level steps required to set up your cluster infrastructure in preparation for an OpenShift Container Platform installation. This includes configuring IP networking and network connectivity for your cluster nodes, preparing a web server for the Ignition files, enabling the required ports through your firewall, and setting up the required DNS and load balancing infrastructure.

After preparation, your cluster infrastructure must meet the requirements outlined in the *Requirements for a cluster with user-provisioned infrastructure* section.

**Prerequisites**

- You have reviewed the [OpenShift Container Platform 4.x Tested Integrations](https://access.redhat.com/articles/4128421) page.
- You have reviewed the infrastructure requirements detailed in the *Requirements for a cluster with user-provisioned infrastructure* section.

**Procedure**

1. Set up static IP addresses.
2. Set up an HTTP or HTTPS server to provide Ignition files to the cluster nodes.
3. Ensure that your network infrastructure provides the required network connectivity between the cluster components. See the *Networking requirements for user-provisioned infrastructure* section for details about the requirements.
4. Configure your firewall to enable the ports required for the OpenShift Container Platform cluster components to communicate. See *Networking requirements for user-provisioned infrastructure* section for details about the ports that are required.
   :::warning

   By default, port `1936` is accessible for an OpenShift Container Platform cluster, because each control plane node needs access to this port.

   For ingress health check probes, the `/healthz/ready` endpoint is available on this port.

   Avoid using the Ingress load balancer to expose this port, because doing so might result in the exposure of sensitive information, such as statistics and metrics, related to Ingress Controllers.

   :::
5. Setup the required DNS infrastructure for your cluster.
   1. Configure DNS name resolution for the Kubernetes API, the application wildcard, the bootstrap machine, the control plane machines, and the compute machines.
   2. Configure reverse DNS resolution for the Kubernetes API, the bootstrap machine, the control plane machines, and the compute machines.
      See the *User-provisioned DNS requirements* section for more information about the OpenShift Container Platform DNS requirements.
6. Validate your DNS configuration.
   1. From your installation node, run DNS lookups against the record names of the Kubernetes API, the wildcard routes, and the cluster nodes. Validate that the IP addresses in the responses correspond to the correct components.
   2. From your installation node, run reverse DNS lookups against the IP addresses of the load balancer and the cluster nodes. Validate that the record names in the responses correspond to the correct components.
      See the *Validating DNS resolution for user-provisioned infrastructure* section for detailed DNS validation steps.
7. Provision the required API and application ingress load balancing infrastructure. See the *Load balancing requirements for user-provisioned infrastructure* section for more information about the requirements.
   :::note

   Some load balancing solutions require the DNS name resolution for the cluster nodes to be in place before the load balancing is initialized.

   :::

### Example load balancer configuration for user-provisioned clusters {#installation-load-balancing-user-infra-example_installing-ibm-z}

Reference the example API and application Ingress load balancer configuration so that you can understand how to meet the load balancing requirements for user-provisioned clusters.

The sample is an `/etc/haproxy/haproxy.cfg` configuration for an HAProxy load balancer. The example is not meant to provide advice for choosing one load balancing solution over another.

:::tip

If you are using HAProxy as a load balancer, you can check that the `haproxy` process is listening on ports `6443`, `22623`, `443`, and `80` by running `netstat -nltupe` on the HAProxy node.

:::

In the example, the same load balancer is used for the Kubernetes API and application ingress traffic. In production scenarios, you can deploy the API and application ingress load balancers separately so that you can scale the load balancer infrastructure for each in isolation.

:::note

If you are using HAProxy as a load balancer and SELinux is set to `enforcing`, you must ensure that the HAProxy service can bind to the configured TCP port by running `setsebool -P haproxy_connect_any=1`.

:::

```text title="Sample API and application Ingress load balancer configuration"
global
  log         127.0.0.1 local2
  pidfile     /var/run/haproxy.pid
  maxconn     4000
  daemon
defaults
  mode                    http
  log                     global
  option                  dontlognull
  option http-server-close
  option                  redispatch
  retries                 3
  timeout http-request    10s
  timeout queue           1m
  timeout connect         10s
  timeout client          1m
  timeout server          1m
  timeout http-keep-alive 10s
  timeout check           10s
  maxconn                 3000
listen api-server-6443
  bind *:6443
  mode tcp
  option  httpchk GET /readyz HTTP/1.0
  option  log-health-checks
  balance roundrobin
  server bootstrap bootstrap.ocp4.example.com:6443 verify none check check-ssl inter 10s fall 2 rise 3 backup
  server master0 master0.ocp4.example.com:6443 weight 1 verify none check check-ssl inter 10s fall 2 rise 3
  server master1 master1.ocp4.example.com:6443 weight 1 verify none check check-ssl inter 10s fall 2 rise 3
  server master2 master2.ocp4.example.com:6443 weight 1 verify none check check-ssl inter 10s fall 2 rise 3
listen machine-config-server-22623
  bind *:22623
  mode tcp
  server bootstrap bootstrap.ocp4.example.com:22623 check inter 1s backup
  server master0 master0.ocp4.example.com:22623 check inter 1s
  server master1 master1.ocp4.example.com:22623 check inter 1s
  server master2 master2.ocp4.example.com:22623 check inter 1s
listen ingress-router-443
  bind *:443
  mode tcp
  balance source
  server compute0 compute0.ocp4.example.com:443 check inter 1s
  server compute1 compute1.ocp4.example.com:443 check inter 1s
listen ingress-router-80
  bind *:80
  mode tcp
  balance source
  server compute0 compute0.ocp4.example.com:80 check inter 1s
  server compute1 compute1.ocp4.example.com:80 check inter 1s
```

where:

`listen api-server-6443`
: Port `6443` handles the Kubernetes API traffic and points to the control plane machines. You must configure health checks on this port to ensure that the API server is available before routing traffic.

`server bootstrap bootstrap.ocp4.example.com`
: The bootstrap entries must be in place before the OpenShift Container Platform cluster installation and they must be removed after the bootstrap process is complete.

`listen machine-config-server`
: Port `22623` handles the machine config server traffic and points to the control plane machines.

`listen ingress-router-443`
: Port `443` handles the HTTPS traffic and points to the machines that run the Ingress Controller pods. The Ingress Controller pods run on the compute machines by default.

`listen ingress-router-80`
: Port `80` handles the HTTP traffic and points to the machines that run the Ingress Controller pods. The Ingress Controller pods run on the compute machines by default.

  :::note

  If you are deploying a compact three-node cluster with zero compute nodes, the Ingress Controller pods run on the control plane nodes. In three-node cluster deployments, you must configure your application Ingress load balancer to route HTTP and HTTPS traffic to the control plane nodes.

  :::

## Manually creating the installation configuration file {#installation-initializing-manual_installing-ibm-z}

Installing the cluster requires that you manually create the installation configuration file.

**Prerequisites**

- You have an SSH public key on your local machine for use with the installation program. You can use the key for SSH authentication onto your cluster nodes for debugging and disaster recovery.
- You have obtained the OpenShift Container Platform installation program and the pull secret for your cluster.

**Procedure**

1. Create an installation directory to store your required installation assets in:
   ```terminal
   $ mkdir <installation_directory>
   ```

   :::warning

   You must create a directory. Some installation assets, such as bootstrap X.509 certificates have short expiration intervals, so you must not reuse an installation directory. If you want to reuse individual files from another cluster installation, you can copy them into your directory. However, the file names for the installation assets might change between releases. Use caution when copying installation files from an earlier OpenShift Container Platform version.

   :::
2. Customize the provided sample `install-config.yaml` file template and save the file in the `<installation_directory>`.
   :::note

   You must name this configuration file `install-config.yaml`.

   :::
3. Back up the `install-config.yaml` file so that you can use it to install many clusters.
   :::warning

   Back up the `install-config.yaml` file now, because the installation process consumes the file in the next step.

   :::

**Additional resources**

- [Installation configuration parameters for IBM Z(R)](/docs/installing/installing_ibm_z/installation-config-parameters-ibm-z#installation-config-parameters-ibm-z)

### Sample install-config.yaml file for IBM Z {#_sample_install-configyaml_file_for_ibm_z_title}

You can customize the `install-config.yaml` file to specify more details about your OpenShift Container Platform cluster platform or modify the values of the required parameters.

```yaml
apiVersion: v1
baseDomain: example.com
compute:
- hyperthreading: Enabled
  name: worker
  replicas: 0
  architecture: s390x
controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 3
  architecture: s390x
metadata:
  name: test
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  networkType: OVNKubernetes
  machineNetwork:
  - cidr: 192.168.0.0/16
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {}
fips: false

pullSecret: '{"auths": ...}'

sshKey: 'ssh-ed25519 AAAA...'

```

where:

`baseDomain`
: Specifies the base domain of the cluster. All DNS records must be sub-domains of this base and include the cluster name.

`compute`
: Specifies the `compute` node configurations, which is a sequence of mappings. To meet the requirements of the different data structures, the first line of the `compute` section must begin with a hyphen, `-`.

`controlPlane`
: Specifies the `controlPlane` node configurations, which is a single mapping. To meet the requirements of the different data structures, the first line of the `controlPlane` section must not. Only one control plane pool is used.

`hyperthreading`
: Specifies whether to enable or disable simultaneous multithreading (SMT), or hyperthreading. By default, SMT is enabled to increase the performance of the cores in your machines. You can disable it by setting the parameter value to `Disabled`. If you disable SMT, you must disable it in all cluster machines; this includes both control plane and compute machines.

:::note

Simultaneous multithreading (SMT) is enabled by default. If SMT is not available on your OpenShift Container Platform nodes, the `hyperthreading` parameter has no effect.

:::

:::warning

If you disable `hyperthreading`, whether on your OpenShift Container Platform nodes or in the `install-config.yaml` file, ensure that your capacity planning accounts for the dramatically decreased machine performance.

:::

`compute.replicas`
: Specifies the number of compute machines that the cluster creates and manages for you on installer-provisioned installations. You must set this value to `0` when you install OpenShift Container Platform on user-provisioned infrastructure. Additionally for user-provisioned installations, you must manually deploy the compute machines before you finish installing the cluster.

:::note

If you are installing a three-node cluster, do not deploy any compute machines when you install the Red Hat Enterprise Linux CoreOS (RHCOS) machines.

:::

`controlPlane.replicas`
: Specifies the number of control plane machines that you add to the cluster. Because the cluster uses these values as the number of etcd endpoints in the cluster, the value must match the number of control plane machines that you deploy.

`metadata.name`
: Specifies the cluster name that you specified in your DNS records.

`networking.clusterNetwork.cidr`
: Specifies a block of IP addresses from which pod IP addresses are allocated. This block must not overlap with existing physical networks. These IP addresses are used for the pod network. If you need to access the pods from an external network, you must configure load balancers and routers to manage the traffic.

:::note

Class E CIDR range is reserved for a future use. To use the Class E CIDR range, you must ensure your networking environment accepts the IP addresses within the Class E CIDR range.

:::

`networking.cidr.hostPrefix`
: Specifies the subnet prefix length to assign to each individual node. For example, if `hostPrefix` is set to `23`, then each node is assigned a `/23` subnet out of the given `cidr`, which allows for 510 (2^(32 - 23) - 2) pod IP addresses. If you are required to provide access to nodes from an external network, configure load balancers and routers to manage the traffic.

`networking.networkType`
: Specifies the cluster network plugin to install. The default value `OVNKubernetes` is the only supported value.

`networking.serviceNetwork`
: Specifies the IP address pool to use for service IP addresses. You can enter only one IP address pool. This block must not overlap with existing physical networks. If you need to access the services from an external network, configure load balancers and routers to manage the traffic.

`platform`
: Specifies the platform. You must set the platform to `none`. You cannot provide additional platform configuration variables for IBM Z(R) infrastructure.

:::warning

Clusters that are installed with the platform type `none` are unable to use some features, such as managing compute machines with the Machine API. This limitation applies even if the compute machines that are attached to the cluster are installed on a platform that would normally support the feature. This parameter cannot be changed after installation.

:::

`fips`
: Specifies either enabling or disabling FIPS mode. By default, FIPS mode is not enabled. If FIPS mode is enabled, the Red Hat Enterprise Linux CoreOS (RHCOS) machines that OpenShift Container Platform runs on bypass the default Kubernetes cryptography suite and use the cryptography modules that are provided with RHCOS instead.

:::warning

To enable FIPS mode for your cluster, you must run the installation program from a Red Hat Enterprise Linux (RHEL) computer configured to operate in FIPS mode. For more information about configuring FIPS mode on RHEL, see [Switching RHEL to FIPS mode](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/security_hardening/switching-rhel-to-fips-mode_security-hardening).

When running Red Hat Enterprise Linux (RHEL) or Red Hat Enterprise Linux CoreOS (RHCOS) booted in FIPS mode, OpenShift Container Platform core components use the RHEL cryptographic libraries that have been submitted to NIST for FIPS 140-2/140-3 Validation on only the x86_64, ppc64le, and s390x architectures.

:::

`pullSecret`
: Specifies the [pull secret from Red Hat OpenShift Cluster Manager](https://console.redhat.com/openshift/install/pull-secret). This pull secret allows you to authenticate with the services that are provided by the included authorities, including Quay.io, which serves the container images for OpenShift Container Platform components.

`sshKey`:: Specifies the SSH public key for the `core` user in Red Hat Enterprise Linux CoreOS (RHCOS).

:::note

For production OpenShift Container Platform clusters on which you want to perform installation debugging or disaster recovery, specify an SSH key that your `ssh-agent` process uses.

:::

### Configuring the cluster-wide proxy during installation {#installation-configure-proxy_installing-ibm-z}

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

### Configuring a three-node cluster {#installation-three-node-cluster_installing-ibm-z}

To create smaller, resource-efficient clusters for testing and production, deploy a bare-metal cluster with zero compute machines in a minimal three-node cluster. This optional configuration uses only three control plane machines, optimizing infrastructure resources for testing, development, and production purposes.

In three-node OpenShift Container Platform environments, the three control plane machines are schedulable, which means that your application workloads run on them.

**Prerequisites**

- You have an existing `install-config.yaml` file.

**Procedure**

- Ensure that the number of compute replicas is set to `0` in your `install-config.yaml` file, as shown in the following `compute` stanza:
  ```yaml
  compute:
  - name: worker
    platform: {}
    replicas: 0
  # ...
  ```

  :::note

  You must set the value of the `replicas` parameter for the compute machines to `0` when you install OpenShift Container Platform on user-provisioned infrastructure, regardless of the number of compute machines you deploy. In installer-provisioned installations, the parameter controls the number of compute machines that the cluster creates and manages for you. This does not apply to user-provisioned installations, where you deploy the compute machines manually.

  :::

  :::note

  The preferred resource for control plane nodes is six vCPUs and 21 GB. For three control plane nodes this is the memory + vCPU equivalent of a minimum five-node cluster. Back the three nodes, each installed on a 120 GB disk, with three IFLs that are SMT2 enabled. The minimum tested setup is three vCPUs and 10 GB on a 120 GB disk for each control plane node.

  :::

  **Next steps**

For three-node cluster installations, follow these next steps:

- If you are deploying a three-node cluster with zero compute nodes, the Ingress Controller pods run on the control plane nodes. In three-node cluster deployments, you must configure your application ingress load balancer to route HTTP and HTTPS traffic to the control plane nodes. See the *Load balancing requirements for user-provisioned infrastructure* section for more information.
- When you create the Kubernetes manifest files in the following procedure, ensure that the `mastersSchedulable` parameter in the `<installation_directory>/manifests/cluster-scheduler-02-config.yml` file is set to `true`. This enables your application workloads to run on the control plane nodes.
- Do not deploy any compute nodes when you create the Red Hat Enterprise Linux CoreOS (RHCOS) machines.

## Cluster Network Operator configuration {#nw-operator-cr_installing-ibm-z}

To manage cluster networking, configure the Cluster Network Operator (CNO) `Network` custom resource (CR) named `cluster` so the cluster uses the correct IP ranges and network plugin settings for reliable pod and service connectivity. Some settings and fields are inherited at the time of install or by the `default.Network.type` plugin, OVN-Kubernetes.

The CNO configuration inherits the following fields during cluster installation from the `Network` API in the `Network.config.openshift.io` API group:

`clusterNetwork`
: IP address pools from which pod IP addresses are allocated.

`serviceNetwork`
: IP address pool for services.

`defaultNetwork.type`
: Cluster network plugin. `OVNKubernetes` is the only supported plugin during installation.

You can specify the cluster network plugin configuration for your cluster by setting the fields for the `defaultNetwork` object in the CNO object named `cluster`.

### Cluster Network Operator configuration object {#nw-operator-cr-cno-object_installing-ibm-z}

The fields for the Cluster Network Operator (CNO) are described in the following table:

**Cluster Network Operator configuration object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>metadata.name</code></td>
  <td><code>string</code></td>
  <td>The name of the CNO object. This name is always <code>cluster</code>.</td>
</tr>
<tr>
  <td><code>spec.clusterNetwork</code></td>
  <td><code>array</code></td>
  <td>A list specifying the blocks of IP addresses from which pod IP addresses are allocated and the subnet prefix length assigned to each individual node in the cluster. If you use dual-stack networking, specify IPv4 and IPv6 address families. For example:<br><br><pre><code>spec:&#10;  clusterNetwork:&#10;  - cidr: 10.128.0.0/19&#10;    hostPrefix: 23&#10;  - cidr: fd01::/48&#10;    hostPrefix: 64</code></pre><br><br>If you install a cluster on AWS with dual-stack networking, the order of addresses must match the dual-stack configuration you selected. For example, if you specified the <code>DualStackIPv4Primary</code>, list the IPv4 address first.</td>
</tr>
<tr>
  <td><code>spec.serviceNetwork</code></td>
  <td><code>array</code></td>
  <td>A block of IP addresses for services. If you use dual-stack networking, specify IPv4 and IPv6 address families. For example:<br><br><pre><code>spec:&#10;  serviceNetwork:&#10;  - 172.30.0.0/14&#10;  - fd02::/112</code></pre><br><br>If you install a cluster on AWS with dual-stack networking, the order of addresses must match the dual-stack configuration you selected. For example, if you specified the <code>DualStackIPv4Primary</code>, list the IPv4 address first.<br><br>   You can customize this field only in the <code>install-config.yaml</code> file before you create the manifests. The value is read-only in the manifest file. </td>
</tr>
<tr>
  <td><code>spec.defaultNetwork</code></td>
  <td><code>object</code></td>
  <td>Configures the network plugin for the cluster network.</td>
</tr>
<tr>
  <td><code>spec.additionalRoutingCapabilities.providers</code></td>
  <td><code>array</code></td>
  <td>This setting enables a dynamic routing provider. The FRR routing capability provider is required for the route advertisement feature. The only supported value is <code>FRR</code>.<br><br><ul><li><code>FRR</code>: The FRR routing provider</li></ul><br><br><pre><code>spec:&#10;  additionalRoutingCapabilities:&#10;    providers:&#10;    - FRR</code></pre></td>
</tr>
</tbody>
</table>

:::warning

For a cluster that needs to deploy objects across multiple networks, ensure that you specify the same value for the `clusterNetwork.hostPrefix` parameter for each network type that is defined in the `install-config.yaml` file. Setting a different value for each `clusterNetwork.hostPrefix` parameter can impact the OVN-Kubernetes network plugin, where the plugin cannot effectively route object traffic among different nodes.

:::

### defaultNetwork object configuration {#nw-operator-cr-defaultnetwork_installing-ibm-z}

The values for the `defaultNetwork` object are defined in the following table:

`defaultNetwork`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>type</code></td>
  <td><code>string</code></td>
  <td><code>OVNKubernetes</code>. The Red Hat OpenShift Networking network plugin is selected during installation. This value cannot be changed after cluster installation.<div class="alert alert--secondary"><strong>Note</strong><p>OpenShift Container Platform uses the OVN-Kubernetes network plugin by default.</p></div></td>
</tr>
<tr>
  <td><code>ovnKubernetesConfig</code></td>
  <td><code>object</code></td>
  <td>This object is only valid for the OVN-Kubernetes network plugin.</td>
</tr>
</tbody>
</table>

### Configuration for the OVN-Kubernetes network plugin {#nw-operator-configuration-parameters-for-ovn-sdn_installing-ibm-z}

The following table describes the configuration fields for the OVN-Kubernetes network plugin:

`ovnKubernetesConfig`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>mtu</code></td>
  <td><code>integer</code></td>
  <td> The maximum transmission unit (MTU) for the Geneve (Generic Network Virtualization Encapsulation) overlay network. This is detected automatically based on the MTU of the primary network interface. You do not normally need to override the detected MTU.<br><br>If the auto-detected value is not what you expect it to be, confirm that the MTU on the primary network interface on your nodes is correct. You cannot use this option to change the MTU value of the primary network interface on the nodes.<br><br>If your cluster requires different MTU values for different nodes, you must set this value to <code>100</code> less than the lowest MTU value in your cluster. For example, if some nodes in your cluster have an MTU of <code>9001</code>, and some have an MTU of <code>1500</code>, you must set this value to <code>1400</code>.  </td>
</tr>
<tr>
  <td><code>genevePort</code></td>
  <td><code>integer</code></td>
  <td> The port to use for all Geneve packets. The default value is <code>6081</code>. This value cannot be changed after cluster installation.  </td>
</tr>
<tr>
  <td><code>ipsecConfig</code></td>
  <td><code>object</code></td>
  <td> Specify a configuration object for customizing the IPsec configuration.  </td>
</tr>
<tr>
  <td><code>ipv4</code></td>
  <td><code>object</code></td>
  <td>Specifies a configuration object for IPv4 settings.</td>
</tr>
<tr>
  <td><code>ipv6</code></td>
  <td><code>object</code></td>
  <td>Specifies a configuration object for IPv6 settings.</td>
</tr>
<tr>
  <td><code>policyAuditConfig</code></td>
  <td><code>object</code></td>
  <td>Specify a configuration object for customizing network policy audit logging. If unset, the defaults audit log settings are used.</td>
</tr>
<tr>
  <td><code>routeAdvertisements</code></td>
  <td><code>string</code></td>
  <td>Specifies whether to advertise cluster network routes. The default value is <code>Disabled</code>.<ul><li><code>Enabled</code>: Import routes to the cluster network and advertise cluster network routes as configured in <code>RouteAdvertisements</code> objects.</li><li><code>Disabled</code>: Do not import routes to the cluster network or advertise cluster network routes.</li></ul></td>
</tr>
<tr>
  <td><code>gatewayConfig</code></td>
  <td><code>object</code></td>
  <td>Optional: Specify a configuration object for customizing how egress traffic is sent to the node gateway. Valid values are <code>Shared</code> and <code>Local</code>. The default value is <code>Shared</code>. In the default setting, the Open vSwitch (OVS) outputs traffic directly to the node IP interface. If you are using hardware offloading, Red Hat recommends to use the default <code>Shared</code> gateway mode to bypass the host routing plane. In the <code>Local</code> setting, it traverses the host network; consequently, it gets applied to the routing table of the host.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>While migrating egress traffic, you can expect some disruption to workloads and service traffic until the Cluster Network Operator (CNO) successfully rolls out the changes.</p></div></td>
</tr>
</tbody>
</table>

`ovnKubernetesConfig.ipv4`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>internalTransitSwitchSubnet</code></td>
  <td>string</td>
  <td>If your existing network infrastructure overlaps with the <code>100.88.0.0/16</code> IPv4 subnet, you can specify a different IP address range for internal use by OVN-Kubernetes. The subnet for the distributed transit switch that enables east-west traffic. This subnet cannot overlap with any other subnets used by OVN-Kubernetes or on the host itself. It must be large enough to accommodate one IP address per node in your cluster.<br><br>The default value is <code>100.88.0.0/16</code>.</td>
</tr>
<tr>
  <td><code>internalJoinSubnet</code></td>
  <td>string</td>
  <td>If your existing network infrastructure overlaps with the <code>100.64.0.0/16</code> IPv4 subnet, you can specify a different IP address range for internal use by OVN-Kubernetes. You must ensure that the IP address range does not overlap with any other subnet used by your OpenShift Container Platform installation. The IP address range must be larger than the maximum number of nodes that can be added to the cluster. For example, if the <code>clusterNetwork.cidr</code> value is <code>10.128.0.0/14</code> and the <code>clusterNetwork.hostPrefix</code> value is <code>/23</code>, then the maximum number of nodes is <code>2^(23-14)=512</code>.<br><br>The default value is <code>100.64.0.0/16</code>.</td>
</tr>
</tbody>
</table>

`ovnKubernetesConfig.ipv6`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>internalTransitSwitchSubnet</code></td>
  <td>string</td>
  <td>If your existing network infrastructure overlaps with the <code>fd97::/64</code> IPv6 subnet, you can specify a different IP address range for internal use by OVN-Kubernetes. The subnet for the distributed transit switch that enables east-west traffic. This subnet cannot overlap with any other subnets used by OVN-Kubernetes or on the host itself. It must be large enough to accommodate one IP address per node in your cluster.<br><br>The default value is <code>fd97::/64</code>.</td>
</tr>
<tr>
  <td><code>internalJoinSubnet</code></td>
  <td>string</td>
  <td>If your existing network infrastructure overlaps with the <code>fd98::/64</code> IPv6 subnet, you can specify a different IP address range for internal use by OVN-Kubernetes. You must ensure that the IP address range does not overlap with any other subnet used by your OpenShift Container Platform installation. The IP address range must be larger than the maximum number of nodes that can be added to the cluster.<br><br>The default value is <code>fd98::/64</code>.</td>
</tr>
</tbody>
</table>

`policyAuditConfig`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>rateLimit</code></td>
  <td>integer</td>
  <td>The maximum number of messages to generate every second per node. The default value is <code>20</code> messages per second.</td>
</tr>
<tr>
  <td><code>maxFileSize</code></td>
  <td>integer</td>
  <td>The maximum size for the audit log in bytes. The default value is <code>50000000</code> or 50 MB.</td>
</tr>
<tr>
  <td><code>maxLogFiles</code></td>
  <td>integer</td>
  <td>The maximum number of log files that are retained.</td>
</tr>
<tr>
  <td><code>destination</code></td>
  <td>string</td>
  <td>One of the following additional audit log targets:<br><br><dl><dt><code>libc</code></dt><dd>The libc <code>syslog()</code> function of the journald process on the host.</dd><dt><code>udp:&lt;host&gt;:&lt;port&gt;</code></dt><dd>A syslog server. Replace <code>&lt;host&gt;:&lt;port&gt;</code> with the host and port of the syslog server.</dd><dt><code>unix:&lt;file&gt;</code></dt><dd>A Unix Domain Socket file specified by <code>&lt;file&gt;</code>.</dd><dt><code>null</code></dt><dd>Do not send the audit logs to any additional target.</dd></dl></td>
</tr>
<tr>
  <td><code>syslogFacility</code></td>
  <td>string</td>
  <td>The syslog facility, such as <code>kern</code>, as defined by RFC5424. The default value is <code>local0</code>.</td>
</tr>
</tbody>
</table>

<a name="gatewayConfig-object_installing-ibm-z"></a>

`gatewayConfig`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>routingViaHost</code></td>
  <td><code>boolean</code></td>
  <td>Set this field to <code>true</code> to send egress traffic from pods to the host networking stack. For highly-specialized installations and applications that rely on manually configured routes in the kernel routing table, you might want to route egress traffic to the host networking stack. By default, egress traffic is processed in OVN to exit the cluster and is not affected by specialized routes in the kernel routing table. The default value is <code>false</code>.<br><br>This field has an interaction with the Open vSwitch hardware offloading feature. If you set this field to <code>true</code>, you do not receive the performance benefits of the offloading because egress traffic is processed by the host networking stack.</td>
</tr>
<tr>
  <td><code>ipForwarding</code></td>
  <td><code>object</code></td>
  <td>You can control IP forwarding for all traffic on OVN-Kubernetes managed interfaces by using the <code>ipForwarding</code> specification in the <code>Network</code> resource. Specify <code>Restricted</code> to only allow IP forwarding for Kubernetes related traffic. Specify <code>Global</code> to allow forwarding of all IP traffic. For new installations, the default is <code>Restricted</code>. For updates to OpenShift Container Platform 4.14 or later, the default is <code>Global</code>.<div class="alert alert--secondary"><strong>Note</strong><p>The default value of <code>Restricted</code> sets the IP forwarding to drop.</p></div></td>
</tr>
<tr>
  <td><code>ipv4</code></td>
  <td><code>object</code></td>
  <td>Optional: Specify an object to configure the internal OVN-Kubernetes masquerade address for host to service traffic for IPv4 addresses.</td>
</tr>
<tr>
  <td><code>ipv6</code></td>
  <td><code>object</code></td>
  <td>Optional: Specify an object to configure the internal OVN-Kubernetes masquerade address for host to service traffic for IPv6 addresses.</td>
</tr>
</tbody>
</table>

<a name="gatewayconfig-ipv4-object_installing-ibm-z"></a>

`gatewayConfig.ipv4`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>internalMasqueradeSubnet</code></td>
  <td><code>string</code></td>
  <td>The masquerade IPv4 addresses that are used internally to enable host to service traffic. The host is configured with these IP addresses as well as the shared gateway bridge interface. The default value is <code>169.254.169.0/29</code>.<div class="alert alert--warning"><strong>Important</strong><p>For OpenShift Container Platform 4.17 and later versions, clusters use <code>169.254.0.0/17</code> as the default masquerade subnet. For upgraded clusters, there is no change to the default masquerade subnet.</p></div></td>
</tr>
</tbody>
</table>

<a name="gatewayconfig-ipv6-object_installing-ibm-z"></a>

`gatewayConfig.ipv6`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>internalMasqueradeSubnet</code></td>
  <td><code>string</code></td>
  <td>The masquerade IPv6 addresses that are used internally to enable host to service traffic. The host is configured with these IP addresses as well as the shared gateway bridge interface. The default value is <code>fd69::/125</code>.<div class="alert alert--warning"><strong>Important</strong><p>For OpenShift Container Platform 4.17 and later versions, clusters use <code>fd69::/112</code> as the default masquerade subnet. For upgraded clusters, there is no change to the default masquerade subnet.</p></div></td>
</tr>
</tbody>
</table>

<a name="nw-operator-cr-ipsec_installing-ibm-z"></a>

`ipsecConfig`** object**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>mode</code></td>
  <td><code>string</code></td>
  <td>Specifies the behavior of the IPsec implementation. Must be one of the following values:<br><br><ul><li><code>Disabled</code>: IPsec is not enabled on cluster nodes.</li><li><code>External</code>: IPsec is enabled for network traffic with external hosts.</li><li><code>Full</code>: IPsec is enabled for pod traffic and network traffic with external hosts.</li></ul></td>
</tr>
</tbody>
</table>

```yaml title="Example OVN-Kubernetes configuration with IPSec enabled"
defaultNetwork:
  type: OVNKubernetes
  ovnKubernetesConfig:
    mtu: 1400
    genevePort: 6081
    ipsecConfig:
      mode: Full
```

## Creating the Kubernetes manifest and Ignition config files {#installation-user-infra-generate-k8s-manifest-ignition_installing-ibm-z}

Because you manually provision infrastructure, you must generate the Kubernetes manifest and Ignition config files that the cluster requires.

The installation program converts the installation configuration into Kubernetes manifests and then wraps them into Ignition configuration files. You use these Ignition files to configure the cluster machines.

:::warning

- The Ignition config files that the OpenShift Container Platform installation program generates contain certificates that expire after 24 hours, which the system then renews. If you shut down the cluster before the system renews the certificates and you later restart the cluster after the 24 hours have elapsed, the cluster automatically recovers the expired certificates. The exception is that you must manually approve the pending `node-bootstrapper` certificate signing requests (CSRs) to recover kubelet certificates. See the documentation for *Recovering from expired control plane certificates* for more information.
- Use Ignition config files within 12 hours after you generate them, because the 24-hour certificate rotates from 16 to 22 hours after you install the cluster. By using the Ignition config files within 12 hours, you can avoid installation failure if the certificate update runs during installation.

:::

:::note

The installation program that generates the manifest and Ignition files is architecture specific. You can obtain it from the [client image mirror](https://mirror.openshift.com/pub/openshift-v4/s390x/clients/ocp/latest/). The Linux version of the installation program runs on s390x only. This installation program is also available as a macOS version.

:::

**Prerequisites**

- You obtained the OpenShift Container Platform installation program.
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

   :::warning

   If you are installing a three-node cluster, skip the following step to allow the control plane nodes to be schedulable.

   :::

   :::warning

   When you configure control plane nodes from the default unschedulable to schedulable, you require additional subscriptions because control plane nodes then become compute nodes.

   :::
2. Verify that the `mastersSchedulable` parameter in the `<installation_directory>/manifests/cluster-scheduler-02-config.yml` Kubernetes manifest file is set to `false`. This setting prevents pods from being scheduled on the control plane machines:
   1. Open the `<installation_directory>/manifests/cluster-scheduler-02-config.yml` file.
   2. Locate the `mastersSchedulable` parameter and verify that it is set to `false`.
   3. Save and exit the file.
3. To create the Ignition configuration files, run the following command from the directory that contains the installation program:
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

## Configuring boot volume encryption in an IBM Z or IBM LinuxONE environment {#configuring-boot-volume-encryption-ibm-z-linuxone-environment_installing-ibm-z}

You can optionally encrypt the boot volumes of your OpenShift Container Platform control plane and compute nodes on IBM Z(R) or IBM(R) LinuxONE by using LUKS encryption via IBM(R) Crypto Express (CEX) or Network Bound Disk Encryption (NBDE).

- Linux Unified Key Setup (LUKS) encryption via IBM(R) Crypto Express (CEX)
- Network Bound Disk Encryption (NBDE)

### LUKS encryption via CEX in an IBM Z or IBM LinuxONE environment {#configuring-luks-encryption-via-cex-ibm-z-linuxone-environment_installing-ibm-z}

Enabling hardware-based Linux Unified Key Setup (LUKS) encryption via IBM(R) Crypto Express (CEX) in an IBM Z(R) or IBM(R) LinuxONE environment requires additional steps.

**Prerequisites**

- You have installed the `butane` utility.
- You have reviewed the instructions for how to create machine configs with Butane.

**Procedure**

````
    . Choose the appropriate method to create Butane configuration files for the control plane and compute nodes:
    ** For installations on DASD-type disks, create a file named `main-storage.bu` by using the following Butane configuration for a control plane node with disk encryption, for example:
    +
    ```yaml
    variant: openshift
    version: 4.22.0
    metadata:
      name: main-storage
      labels:
        machineconfiguration.openshift.io/role: master
    boot_device:
      layout: s390x-eckd
      luks:
        device: /dev/dasda
        cex:
          enabled: true
    openshift:
      fips: true
      kernel_arguments:
        - rd.luks.key=/etc/luks/cex.key
    ```
   \
    where:
````

`openshift.fips`
: Specifies whether to enable or disable FIPS mode. By default, FIPS mode is not enabled. If FIPS mode is enabled, the Red Hat Enterprise Linux CoreOS (RHCOS) machines that OpenShift Container Platform runs on bypass the default Kubernetes cryptography suite and use the cryptography modules that are provided with RHCOS instead.

`openshift.kernel_arguments`
: Specifies the location of the key that is required to decrypt the device. You cannot change this value.

  - For installations on FCP-type disks, create a file named `main-storage.bu` by using the following Butane configuration for a control plane node with disk encryption, for example:

  ```yaml
  variant: openshift
  version: 4.22.0
  metadata:
    name: main-storage
    labels:
      machineconfiguration.openshift.io/role: master
  storage:
    filesystems:
      - device: /dev/mapper/root
        format: xfs
        label: root
        wipe_filesystem: true
    luks:
      - device: /dev/disk/by-label/root
        label: luks-root
        name: root
        wipe_volume: true
        cex:
          enabled: true
  openshift:
    fips: true
    kernel_arguments:
      - rd.luks.key=/etc/luks/cex.key
  ```

  where:

`openshift.fips`
: Specifies whether to enable or disable FIPS mode. By default, FIPS mode is not enabled. If FIPS mode is enabled, the Red Hat Enterprise Linux CoreOS (RHCOS) machines that OpenShift Container Platform runs on bypass the default Kubernetes cryptography suite and use the cryptography modules that are provided with RHCOS instead.

`openshift.kernel_arguments`
: Specifies the location of the key that is required to decrypt the device. You cannot change this value.

1. Create a parameter file that includes `ignition.platform.id=metal` and `ignition.firstboot`.
   **Example kernel parameter file for the control plane machine**

   ```terminal
   cio_ignore=all,!condev rd.neednet=1 \
   console=ttysclp0 \
   coreos.inst.install_dev=/dev/disk/by-id/scsi-<serial_number> \
   ignition.firstboot ignition.platform.id=metal \
   coreos.inst.ignition_url=http://<http_server>/master.ign \
   coreos.live.rootfs_url=http://<http_server>/rhcos-<version>-live-rootfs.<architecture>.img \
   ip=<ip_address>::<gateway>:<netmask>:<hostname>::none nameserver=<dns> \
   rd.znet=qeth,0.0.bdd0,0.0.bdd1,0.0.bdd2,layer2=1 \
   rd.zfcp=0.0.5677,0x600606680g7f0056,0x034F000000000000
   ```

   where:

   <dl>
   <dt><code>coreos.inst.install_dev</code></dt>
   <dd>Specifies a unique fully qualified path depending on disk type. This can be DASD-type or FCP-type disks.</dd>
   <dt><code>coreos.inst.ignition_url</code></dt>
   <dd>Specifies the location of the Ignition configuration file. Use <code>master.ign</code> or <code>worker.ign</code>. You can only use the HTTP and HTTPS protocols.</dd>
   <dt><code>coreos.live.rootfs_url</code></dt>
   <dd>Specifies the location of the <code>rootfs</code> artifact for the <code>kernel</code> and <code>initramfs</code> that you want to boot. You can only use the HTTP and HTTPS protocols.</dd>
   <dt><code>rd.zfcp</code></dt>
   <dd>Specifies the FCP device. For installations on DASD-type disks, replace with <code>rd.dasd=0.0.xxxx</code> to specify the DASD device.</dd>
   </dl>

   :::note

   Write all options in the parameter file as a single line and make sure you have no newline characters.

   :::

### Configuring NBDE with static IP in an IBM Z or IBM LinuxONE environment {#configuring-nbde-static-ip-ibm-z-linuxone-environment_installing-ibm-z}

Enabling NBDE disk encryption in an IBM Z(R) or IBM(R) LinuxONE environment requires additional steps.

**Prerequisites**

- You have set up the External Tang Server. See [Network-bound disk encryption](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/security_hardening/configuring-automated-unlocking-of-encrypted-volumes-using-policy-based-decryption_security-hardening#network-bound-disk-encryption_configuring-automated-unlocking-of-encrypted-volumes-using-policy-based-decryption) for instructions.
- You have installed the `butane` utility.
- You have reviewed the instructions for how to create machine configs with Butane.

**Procedure**

1. Create Butane configuration files for the control plane and compute nodes.
   The following example of a Butane configuration for a control plane node creates a file named `master-storage.bu` for disk encryption:

   ```yaml
   variant: openshift
   version: 4.22.0
   metadata:
     name: master-storage
     labels:
       machineconfiguration.openshift.io/role: master
   storage:
     luks:
       - clevis:
           tang:
             - thumbprint: QcPr_NHFJammnRCA3fFMVdNBwjs
               url: http://clevis.example.com:7500
         device: /dev/disk/by-partlabel/root
         label: luks-root
         name: root
         wipe_volume: true
     filesystems:
       - device: /dev/mapper/root
         format: xfs
         label: root
         wipe_filesystem: true
   openshift:
     fips: true
   ```

   where:

   <dl>
   <dt><code>storage.luks.device</code></dt>
   <dd>Specifies the device to encrypt. For installations on DASD-type disks, replace with <code>device: /dev/disk/by-label/root</code>.</dd>
   <dt><code>openshift.fips</code></dt>
   <dd>Specifies whether to enable or disable FIPS mode. By default, FIPS mode is not enabled. If FIPS mode is enabled, the Red Hat Enterprise Linux CoreOS (RHCOS) machines that OpenShift Container Platform runs on bypass the default Kubernetes cryptography suite and use the cryptography modules that are provided with RHCOS instead.</dd>
   </dl>
2. Create a customized initramfs file to boot the machine, by running the following command:
   ```terminal
   $ coreos-installer pxe customize \
       /root/rhcos-bootfiles/rhcos-<release>-live-initramfs.s390x.img \
       --dest-device /dev/disk/by-id/scsi-<serial_number> --dest-karg-append \
       ip=<ip_address>::<gateway_ip>:<subnet_mask>::<network_device>:none \
       --dest-karg-append nameserver=<nameserver_ip> \
       --dest-karg-append rd.neednet=1 -o \
       /root/rhcos-bootfiles/<node_name>-initramfs.s390x.img
   ```

   :::note

   Before first boot, you must customize the initramfs for each node in the cluster, and add PXE kernel parameters.

   :::
3. Create a parameter file that includes `ignition.platform.id=metal` and `ignition.firstboot`.
   **Example kernel parameter file for the control plane machine**

   ```terminal
   cio_ignore=all,!condev rd.neednet=1 \
   console=ttysclp0 \
   coreos.inst.install_dev=/dev/<block_device> \
   ignition.firstboot ignition.platform.id=metal \
   coreos.inst.ignition_url=http://<http_server>/master.ign \
   coreos.live.rootfs_url=http://<http_server>/rhcos-<version>-live-rootfs.<architecture>.img \
   ip=<ip>::<gateway>:<netmask>:<hostname>::none nameserver=<dns> \
   rd.znet=qeth,0.0.bdd0,0.0.bdd1,0.0.bdd2,layer2=1 \
   rd.zfcp=0.0.5677,0x600606680g7f0056,0x034F000000000000 \
   zfcp.allow_lun_scan=0
   ```

   where:

   <dl>
   <dt><code>coreos.inst.install_dev</code></dt>
   <dd>Specifies the block device type. For installations on DASD-type disks, specify <code>/dev/dasda</code>. For installations on FCP-type disks, specify <code>/dev/sda</code>.</dd>
   <dt><code>coreos.inst.ignition_url</code></dt>
   <dd>Specifies the location of the Ignition config file. Use <code>master.ign</code> or <code>worker.ign</code>. Only HTTP and HTTPS protocols are supported.</dd>
   <dt><code>coreos.live.rootfs_url</code></dt>
   <dd>Specifies the location of the <code>rootfs</code> artifact for the <code>kernel</code> and <code>initramfs</code> you are booting. Only HTTP and HTTPS protocols are supported.</dd>
   <dt><code>rd.zfcp</code></dt>
   <dd>Specifies the FCP device. For installations on DASD-type disks, replace with <code>rd.dasd=0.0.xxxx</code> to specify the DASD device.</dd>
   </dl>

   :::note

   Write all options in the parameter file as a single line and make sure you have no newline characters.

   :::

**Additional resources**

- [Creating machine configs with Butane](/docs/installing/install_config/installing-customizing#installation-special-config-butane_installing-customizing)

## Installing RHCOS and starting the OpenShift Container Platform bootstrap process {#installation-user-infra-machines-iso-ibm-z_installing-ibm-z}

To install OpenShift Container Platform on IBM Z(R) infrastructure that you provision, you must install Red Hat Enterprise Linux CoreOS (RHCOS) on z/VM guest virtual machines.

When you install RHCOS, you must provide the Ignition config file that was generated by the OpenShift Container Platform installation program for the type of machine you are installing. If you have configured suitable networking, DNS, and load balancing infrastructure, the OpenShift Container Platform bootstrap process begins automatically after the RHCOS guest machines have rebooted.

Complete the following steps to create the machines.

**Prerequisites**

- An HTTP or HTTPS server running on your provisioning machine that is accessible to the machines you create.
- If you want to enable secure boot, you have obtained the appropriate Red Hat Product Signing Key and read [Secure boot on IBM Z and IBM LinuxONE](https://www.ibm.com/docs/en/linux-on-systems?topic=security-secure-boot-linux-onibm-z-linuxone) in IBM(R) documentation.

**Procedure**

1. Log in to Linux on your provisioning machine.
2. Obtain the Red Hat Enterprise Linux CoreOS (RHCOS) kernel, initramfs, and rootfs files from the [RHCOS image mirror](https://mirror.openshift.com/pub/openshift-v4/s390x/dependencies/rhcos/latest/).
   :::warning

   The RHCOS images might not change with every release of OpenShift Container Platform. You must download images with the highest version that is less than or equal to the OpenShift Container Platform version that you install. Only use the appropriate kernel, initramfs, and rootfs artifacts described in the following procedure.

   :::

   The file names contain the OpenShift Container Platform version number. They resemble the following examples:

   - kernel: `rhcos-<version>-live-kernel-<architecture>`
   - initramfs: `rhcos-<version>-live-initramfs.<architecture>.img`
   - rootfs: `rhcos-<version>-live-rootfs.<architecture>.img`
     :::note

     The rootfs image is the same for FCP and DASD.

     :::
3. Create parameter files. The following parameters are specific for a particular virtual machine:
   - For `ip=`, specify the following seven entries:
     1. The IP address for the machine.
     2. An empty string.
     3. The gateway.
     4. The netmask.
     5. The machine host and domain name in the form `hostname.domainname`. If you omit this value, RHCOS obtains the hostname through a reverse DNS lookup.
     6. The network interface name. If you omit this value, RHCOS applies the IP configuration to all available interfaces.
     7. If you use static IP addresses, specify `none`.
   - For `coreos.inst.ignition_url=`, specify the Ignition file for the machine role. Use `bootstrap.ign`, `master.ign`, or `worker.ign`. Only HTTP and HTTPS protocols are supported.
   - For `coreos.live.rootfs_url=`, specify the matching rootfs artifact for the kernel and initramfs you are booting. Only HTTP and HTTPS protocols are supported.
   - Optional: To enable secure boot, add `coreos.inst.secure_ipl`
   - For installations on DASD-type disks, complete the following tasks:
     1. For `coreos.inst.install_dev=`, specify `/dev/disk/by-path/ccw-<device_id>`. For <device_id> specify, for example, `0.0.1000`.
     2. Use `rd.dasd=` to specify the DASD where RHCOS is to be installed.
     3. Leave all other parameters unchanged.
        Example parameter file, `bootstrap-0.parm`, for the bootstrap machine:

        ```terminal
        cio_ignore=all,!condev rd.neednet=1 \
        console=ttysclp0 \
        coreos.inst.install_dev=/dev/disk/by-id/scsi-<serial_number> \
        coreos.inst.ignition_url=http://<http_server>/bootstrap.ign \
        coreos.live.rootfs_url=http://<http_server>/rhcos-<version>-live-rootfs.<architecture>.img \
        coreos.inst.secure_ipl \
        ip=<ip>::<gateway>:<netmask>:<hostname>::none nameserver=<dns> \
        rd.znet=qeth,0.0.bdf0,0.0.bdf1,0.0.bdf2,layer2=1,portno=0 \
        rd.dasd=0.0.3490
        ```

        where:

        <dl>
        <dt><code>coreos.inst.install_dev</code></dt>
        <dd>Specifies a unique fully qualified path depending on disk type. This can be either DASD-type or FCP-type disks.</dd>
        <dt><code>coreos.inst.ignition_url</code></dt>
        <dd>Specifies the location of the Ignition config file. Use <code>bootstrap.ign</code>, <code>master.ign</code>, or <code>worker.ign</code>. Only HTTP and HTTPS protocols are supported.</dd>
        <dt><code>coreos.live.rootfs_url</code></dt>
        <dd>Specifies the location of the <code>rootfs</code> artifact for the <code>kernel</code> and <code>initramfs</code> you are booting. Only HTTP and HTTPS protocols are supported.</dd>
        <dt><code>coreos.inst.secure_ipl</code></dt>
        <dd>Specifies the <code>coreos.inst.secure_ipl</code> artifact. Optional: To enable secure boot, add <code>coreos.inst.secure_ipl</code>. Write all options in the parameter file as a single line and make sure you have no newline characters.</dd>
        </dl>
   - For installations on FCP-type disks, complete the following tasks:
     1. Use `rd.zfcp=<adapter>,<wwpn>,<lun>` to specify the FCP disk where RHCOS is to be installed. For multipathing repeat this step for each additional path.
        :::note

        When you install with multiple paths, you must enable multipathing directly after the installation, not at a later point in time, as this can cause problems.

        :::
     2. Set the install device as: `coreos.inst.install_dev=/dev/disk/by-id/scsi-<serial_number>`.
4. Optional: Create a `generic.ins` file:
   Some installation methods also require a `generic.ins` file with a mapping of the location of the installation data in the file system of the Hardware Management Console (HMC), the DVD, or the FTP server and the memory locations where the data is to be copied. A sample `generic.ins` file is provided with the RHEL installation media. The file contains file names for the initial RAM disk (`initrd.img`), the kernel image (`kernel.img`), and the parameter (`generic.prm`) files and a memory location for each file.

   ```terminal title="Example generic.ins file"
   images/kernel.img 0x00000000
   images/initrd.img 0x02000000
   images/genericdvd.prm 0x00010480
   images/initrd.addrsize 0x00010408
   ```
5. Leave all other parameters unchanged.
   :::warning

   Additional postinstallation steps are required to fully enable multipathing. For more information, see “Enabling multipathing with kernel arguments on RHCOS" in *Machine configuration*.

   :::

   The following is an example parameter file `worker-1.parm` for a compute node with multipathing:

   ```terminal
   cio_ignore=all,!condev rd.neednet=1 \
   console=ttysclp0 \
   coreos.inst.install_dev=/dev/disk/by-id/scsi-<serial_number> \
   coreos.live.rootfs_url=http://<http_server>/rhcos-<version>-live-rootfs.<architecture>.img \
   coreos.inst.ignition_url=http://<http_server>/worker.ign \
   ip=<ip>::<gateway>:<netmask>:<hostname>::none nameserver=<dns> \
   rd.znet=qeth,0.0.bdf0,0.0.bdf1,0.0.bdf2,layer2=1,portno=0 \
   rd.zfcp=0.0.1987,0x50050763070bc5e3,0x4008400B00000000 \
   rd.zfcp=0.0.19C7,0x50050763070bc5e3,0x4008400B00000000 \
   rd.zfcp=0.0.1987,0x50050763071bc5e3,0x4008400B00000000 \
   rd.zfcp=0.0.19C7,0x50050763071bc5e3,0x4008400B00000000
   ```

   Write all options in the parameter file as a single line and make sure you have no newline characters.
6. Transfer the initramfs, kernel, parameter files, and RHCOS images to z/VM, for example with FTP. For details about how to transfer the files with FTP and boot from the virtual reader, see [Booting the installation on IBM Z(R) to install RHEL in z/VM](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html-single/interactively_installing_rhel_over_the_network/index#installing-under-z-vm_booting-the-installation-media).
7. Punch the files to the virtual reader of the z/VM guest virtual machine that is to become your bootstrap node.
   See [PUNCH](https://www.ibm.com/docs/en/zvm/latest?topic=commands-punch) (IBM(R) Documentation).

   :::tip

   You can use the CP PUNCH command or, if you use Linux, the **vmur** command to transfer files between two z/VM guest virtual machines.

   :::
8. Log in to CMS on the bootstrap machine.
9. IPL the bootstrap machine from the reader:
   ```
   $ ipl c
   ```

   See [IPL](https://www.ibm.com/docs/en/zvm/latest?topic=commands-ipl) (IBM(R) Documentation).
10. Repeat this procedure for the other machines in the cluster.

### Networking and bonding options for ISO installations {#installation-user-infra-machines-routing-bonding_installing-ibm-z}

You can configure advanced options so that you can modify the Red Hat Enterprise Linux CoreOS (RHCOS) manual installation process. The subsequent sections show examples of networking options for an ISO installation.

If you install RHCOS from an ISO image, you can add kernel arguments manually when you boot the image to configure networking for a node. If no networking arguments are specified, DHCP is activated in the initramfs when RHCOS detects that networking is required to fetch the Ignition config file.

:::warning

When adding networking arguments manually, you must also add the `rd.neednet=1` kernel argument to bring the network up in the initramfs.

:::

The following information provides examples for configuring networking and bonding on your RHCOS nodes for ISO installations. The examples describe how to use the `ip=`, `nameserver=`, and `bond=` kernel arguments.

:::note

Ordering is important when adding the kernel arguments: `ip=`, `nameserver=`, and then `bond=`.

:::

The networking options are passed to the `dracut` tool during system boot. For more information about the networking options supported by `dracut`, see `dracut.cmdline` manual page.

**Additional resources**

- [`dracut.cmdline` manual page](https://www.man7.org/linux/man-pages/man7/dracut.cmdline.7.html)

#### Configuring DHCP or static IP addresses {#configuring-dhcp-or-static-ip-addresses_installing-ibm-z}

You can configure an IP address by using either DHCP or an individual static IP address. If you set a static IP, you must then identify the DNS server IP address on each node.

The configuration examples in the procedure, update the IP addresses for the following components:

- The node’s IP address to `10.10.10.2`
- The gateway address to `10.10.10.254`
- The netmask to `255.255.255.0`
- The hostname to `core0.example.com`
- The DNS server address to `4.4.4.41`
- The auto-configuration value to `none`. No auto-configuration is required when IP networking is configured statically.

**Procedure**

1. Enter a command like the following command to configure a static IP address:
   ```terminal
   ip=10.10.10.2::10.10.10.254:255.255.255.0:core0.example.com:enp1s0:none
   nameserver=4.4.4.41
   ```
2. Enter a command like the following command to configure a DHCP IP address:
   ```terminal
   ip=enp1s0:dhcp
   ```

   :::note

   When you use DHCP to configure IP addressing for the RHCOS machines, the machines also obtain the DNS server information through DHCP. For DHCP-based deployments, you can define the DNS server address that is used by the RHCOS nodes through your DHCP server configuration.

   :::
3. If two or more network interfaces and only one interface exists, disable DHCP on a single interface. In the example, the `enp1s0` interface has a static networking configuration and DHCP is disabled for `enp2s0`, which is not used:
   ```terminal
   ip=10.10.10.2::10.10.10.254:255.255.255.0:core0.example.com:enp1s0:none
   ip=::::core0.example.com:enp2s0:none
   ```
4. If you need to combine DHCP and static IP configurations on systems with multiple network interfaces, run the following example command:
   ```terminal
   ip=enp1s0:dhcp
   ip=10.10.10.2::10.10.10.254:255.255.255.0:core0.example.com:enp2s0:none
   ```

#### Configuring an IP address without a static hostname {#configuring-ip-address-without-static-hostname_installing-ibm-z}

You can configure an IP address without assigning a static hostname. If a static hostname is not set by the user, the static hostname gets picked up and automatically set by a reverse DNS lookup.

The configuration examples in the procedure, update the IP addresses for the following components:

- The node’s IP address to `10.10.10.2`
- The gateway address to `10.10.10.254`
- The netmask to `255.255.255.0`
- The DNS server address to `4.4.4.41`
- The auto-configuration value to `none`. No auto-configuration is required when IP networking is configured statically.

**Procedure**

- To configure an IP address without a static hostname, enter a command like the following command:
  ```terminal
  ip=10.10.10.2::10.10.10.254:255.255.255.0::enp1s0:none
  nameserver=4.4.4.41
  ```

#### Specifying multiple network interfaces and DNS servers {#specifying-multiple-network-interfaces_installing-ibm-z}

You can specify multiple network interfaces by setting multiple `ip=` entries. You can provide multiple DNS servers by adding a `nameserver=` entry for each server,

**Procedure**

- To specify multiple network interfaces for your interfaces, you can enter a command like the following command:
  ```terminal
  ip=10.10.10.2::10.10.10.254:255.255.255.0:core0.example.com:enp1s0:none
  ip=10.10.10.3::10.10.10.254:255.255.255.0:core0.example.com:enp2s0:none
  ```
- To provide multiple DNS servers by adding a `nameserver=` entry for each server, enter a command like the following command:
  ```terminal
  nameserver=1.1.1.1
  nameserver=8.8.8.8
  ```

#### Configuring default gateway and route {#configuring-default-gateway-route_installing-ibm-z}

As an optional task, you can configure routes to additional networks by setting an `rd.route=` value.

:::note

When you configure one or multiple networks, one default gateway is required. If the additional network gateway is different from the primary network gateway, the default gateway must be the primary network gateway.

:::

**Procedure**

- To configure the default gateway, enter the following command:
  ```terminal
  ip=::10.10.10.254::::
  ```
- To configure the route for an additional network, enter the following command:
  ```terminal
  rd.route=20.20.20.0/24:20.20.20.254:enp2s0
  ```

#### Configuring VLANs on individual interfaces {#configuring-vlans-individual-interfaces_installing-ibm-z}

As an optional task, you can configure VLANs on individual interfaces by using the `vlan=` parameter.

**Procedure**

- To configure a VLAN on a network interface and use a static IP address, run the following command:
  ```terminal
  ip=10.10.10.2::10.10.10.254:255.255.255.0:core0.example.com:enp2s0.100:none
  vlan=enp2s0.100:enp2s0
  ```
- To configure a VLAN on a network interface and to use DHCP, run the following command:
  ```terminal
  ip=enp2s0.100:dhcp
  vlan=enp2s0.100:enp2s0
  ```

#### Bonding multiple network interfaces to a single interface {#bonding-multiple-network-interfaces-to-single-interface_installing-ibm-z}

As an optional task, you can bond multiple network interfaces to a single interface by using the `bond=` option.

The following example demonstrates editing the `/etc/config/network` file and specifying the following syntax for bonding multiple network interfaces to a single interface:

```terminal
bond=<name>[:<network_interfaces>][:<options>]
```

- `<name>`: Specifies the bonding device name, for example `bond0`.
- `<network_interfaces>`: Specifies a comma-separated list of physical (ethernet) interfaces, such as `em1,em2`.
- ` <options>: Specifies a comma-separated list of bonding options. Enter the  `modinfo bonding` command to see available options.

When you create a bonded interface using the `bond=` command, you must specify how the IP address is assigned and other information for the bonded interface.

**Procedure**

- To configure the bonded interface to use DHCP, edit the `/etc/config/network` file by setting the IP address for the bond to `dhcp`. For example:
  ```terminal
  ip=bond0:dhcp
  ```
- To configure the bonded interface to use a static IP address, edit the `/etc/config/network` file entering the specific IP address you want and related information. For example:
  ```terminal
  bond=bond0:em1,em2:mode=active-backup
  ip=10.10.10.2::10.10.10.254:255.255.255.0:core0.example.com:bond0:none::AA:BB:CC:DD:EE:FF ip=em1:none::AA:BB:CC:DD:EE:FF
  ip=em2:none::AA:BB:CC:DD:EE:FF
  ```

  IBM Z supports value `1` for the `fail_over_mac` parameter, so always set the `fail_over_mac=1` option in active-backup mode to avoid problems when shared OSA/RoCE cards are used.
- You can configure VLANs on bonded interfaces by editing the `/etc/config/network` file and specifying the `vlan=` parameter to use DHCP. For example:
  ```terminal
  ip=bond0.100:dhcp
  bond=bond0:em1,em2:mode=active-backup
  vlan=bond0.100:bond0
  ```
- To configure the bonded interface with a VLAN, edit the `/etc/config/network` file and specify a static IP address. For example:
  ```terminal
  ip=10.10.10.2::10.10.10.254:255.255.255.0:core0.example.com:bond0.100:none
  bond=bond0:em1,em2:mode=active-backup
  vlan=bond0.100:bond0
  ```

#### Using network teaming {#_using_network_teaming}

You can use network teaming as an alternative to bonding by using the `team=` parameter.

**Procedure**

1. Optional: You can use network teaming as an alternative to bonding by using the `team=` parameter.
   - The syntax for configuring a team interface is: `team=name[:network_interfaces]`
     *name* is the team device name (`team0`) and *network_interfaces* represents a comma-separated list of physical (ethernet) interfaces (`em1, em2`).

     :::note

     Teaming is planned to be deprecated when RHCOS switches to an upcoming version of RHEL. For more information, see this [Red Hat Knowledgebase Article](https://access.redhat.com/solutions/6509691).

     :::

     Use the following example to configure a network team:

     ```terminal
     team=team0:em1,em2
     ip=team0:dhcp
     ```

## Waiting for the bootstrap process to complete {#installation-installing-bare-metal_installing-ibm-z}

The OpenShift Container Platform bootstrap process begins after the cluster nodes first boot into the persistent RHCOS environment that has been installed to disk. The configuration information provided through the Ignition config files is used to initialize the bootstrap process and install OpenShift Container Platform on the machines. You must wait for the bootstrap process to complete.

**Prerequisites**

- You have created the Ignition config files for your cluster.
- You have configured suitable network, DNS, and load balancing infrastructure.
- You have obtained the installation program and generated the Ignition config files for your cluster.
- You installed RHCOS on your cluster machines and provided the Ignition config files that the OpenShift Container Platform installation program generated.
- Your machines have direct internet access or have an HTTP or HTTPS proxy available.

**Procedure**

1. Monitor the bootstrap process:
   ```terminal
   $ ./openshift-install --dir <installation_directory> wait-for bootstrap-complete \
       --log-level=info
   ```

   where:

   <dl>
   <dt><code>&lt;installation_directory&gt;</code></dt>
   <dd>Specifies the path to the directory that stores the installation files.</dd>
   <dt><code>--log-level=info</code></dt>
   <dd>Specifies <code>warn</code>, <code>debug</code>, or <code>error</code> instead of <code>info</code> to view different installation details.</dd>
   </dl>

   ```terminal title="Example output"
   INFO Waiting up to 20m0s for the Kubernetes API at https://api.test.example.com:6443...
   INFO API v1.35.4 up
   INFO Waiting up to 1h0m0s for bootstrapping to complete...
   INFO It is now safe to remove the bootstrap resources
   ```

   The command succeeds when the Kubernetes API server signals that it has been bootstrapped on the control plane machines.
2. After the bootstrap process is complete, remove the bootstrap machine from the load balancer.
   :::warning

   You must remove the bootstrap machine from the load balancer at this point. You can also remove or reformat the bootstrap machine itself.

   :::

## Logging in to the cluster by using the CLI {#cli-logging-in-kubeadmin_installing-ibm-z}

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

## Approving the certificate signing requests for your machines {#installation-approve-csrs_installing-ibm-z}

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
   NAME        AGE   REQUESTOR                                   CONDITION
   csr-mddf5   20m   system:node:master-01.example.com   Approved,Issued
   csr-z5rln   16m   system:node:worker-21.example.com   Approved,Issued
   ```
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

## Initial Operator configuration {#installation-operators-config_installing-ibm-z}

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

### Image registry storage configuration {#installation-registry-storage-config_installing-ibm-z}

The Image Registry Operator is not initially available for platforms that do not provide default storage. After installation, you must configure your registry to use storage so that the Registry Operator is made available.

Configure a persistent volume, which is required for production clusters. Where applicable, you can configure an empty directory as the storage location for non-production clusters.

You can also allow the image registry to use block storage types by using the `Recreate` rollout strategy during upgrades.

#### Configuring registry storage for IBM Z {#_configuring_registry_storage_for_ibm_z_title}

As a cluster administrator, following installation you must configure your registry to use storage.

**Prerequisites**

- You have access to the cluster as a user with the `cluster-admin` role.
- You have a cluster on IBM Z(R).
- You have provisioned persistent storage for your cluster, such as Red Hat OpenShift Data Foundation.
  :::warning

  OpenShift Container Platform supports `ReadWriteOnce` access for image registry storage when you have only one replica. `ReadWriteOnce` access also requires that the registry uses the `Recreate` rollout strategy. To deploy an image registry that supports high availability with two or more replicas, `ReadWriteMany` access is required.

  :::
- You must have a system with at least 100Gi capacity.

**Procedure**

1. To configure your registry to use storage, change the `spec.storage.pvc` in the `configs.imageregistry/cluster` resource.
   :::note

   When you use shared storage, review your security settings to prevent outside access.

   :::
2. Verify that you do not have a registry pod:
   ```terminal
   $ oc get pod -n openshift-image-registry -l docker-registry=default
   ```

   ```terminal title="Example output"
   No resources found in openshift-image-registry namespace
   ```

   :::note

   If you do have a registry pod in your output, you do not need to continue with this procedure.

   :::
3. Check the registry configuration:
   ```terminal
   $ oc edit configs.imageregistry.operator.openshift.io
   ```

   ```yaml title="Example output"
   storage:
     pvc:
       claim:
   ```

   Leave the `claim` field blank to allow the automatic creation of an `image-registry-storage` PVC.
4. Check the `clusteroperator` status:
   ```terminal
   $ oc get clusteroperator image-registry
   ```

   ```terminal title="Example output"
   NAME             VERSION              AVAILABLE   PROGRESSING   DEGRADED   SINCE   MESSAGE
   image-registry   4.22                 True        False         False      6h50m
   ```
5. Ensure that your registry is set to managed to enable building and pushing of images.
   - Run:
     ```
     $ oc edit configs.imageregistry/cluster
     ```

     Then, change the line

     ```
     managementState: Removed
     ```

     to

     ```
     managementState: Managed
     ```

#### Configuring storage for the image registry in non-production clusters {#installation-registry-storage-non-production_installing-ibm-z}

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

## Completing installation on user-provisioned infrastructure {#installation-complete-user-infra_installing-ibm-z}

To finalize the installation on user-provisioned infrastructure, complete the cluster deployment after configuring the Operators. This ensures the cluster is fully operational on the infrastructure that you provide.

**Prerequisites**

- Your control plane has initialized.
- You have completed the initial Operator configuration.

**Procedure**

1. Confirm that all the cluster components are online with the following command:
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
   marketplace                                4.22.0    True        False         False      37muser
   monitoring                                 4.22.0    True        False         False      29m
   network                                    4.22.0    True        False         False      38m
   node-tuning                                4.22.0    True        False         False      37m
   openshift-apiserver                        4.22.0    True        False         False      32muser
   openshift-controller-manager               4.22.0    True        False         False      30m
   openshift-samples                          4.22.0    True        False         False      32m
   operator-lifecycle-manager                 4.22.0    True        False         False      37m
   operator-lifecycle-manager-catalog         4.22.0    True        False         False      37m
   operator-lifecycle-manager-packageserver   4.22.0    True        False         False      32m
   service-ca                                 4.22.0    True        False         False      38m
   storage                                    4.22.0    True        False         False      37m
   ```

   Alternatively, the following command notifies you when all of the clusters are available. The command also retrieves and displays credentials:

   ```terminal
   $ ./openshift-install --dir <installation_directory> wait-for install-complete
   ```

   where:

   <dl>
   <dt><code>&lt;installation_directory&gt;</code></dt>
   <dd>Specifies the path to the directory that you stored the installation files in.</dd>
   </dl>

   ```terminal title="Example output"
   INFO Waiting up to 30m0s for the cluster to initialize...
   ```

   The command succeeds when the Cluster Version Operator finishes deploying the OpenShift Container Platform cluster from Kubernetes API server.

   :::warning

   - The Ignition config files that the installation program generates contain certificates that expire after 24 hours, which are then renewed at that time. If the cluster is shut down before renewing the certificates and the cluster is later restarted after the 24 hours have elapsed, the cluster automatically recovers the expired certificates. The exception is that you must manually approve the pending `node-bootstrapper` certificate signing requests (CSRs) to recover kubelet certificates. See the documentation for *Recovering from expired control plane certificates* for more information.
   - It is recommended that you use Ignition config files within 12 hours after they are generated because the 24-hour certificate rotates from 16 to 22 hours after the cluster is installed. By using the Ignition config files within 12 hours, you can avoid installation failure if the certificate update runs during installation.

   :::
2. Confirm that the Kubernetes API server is communicating with the pods.
   1. To view a list of all pods, use the following command:
      ```terminal
      $ oc get pods --all-namespaces
      ```

      ```terminal title="Example output"
      NAMESPACE                         NAME                                            READY   STATUS      RESTARTS   AGE
      openshift-apiserver-operator      openshift-apiserver-operator-85cb746d55-zqhs8   1/1     Running     1          9m
      openshift-apiserver               apiserver-67b9g                                 1/1     Running     0          3m
      openshift-apiserver               apiserver-ljcmx                                 1/1     Running     0          1m
      openshift-apiserver               apiserver-z25h4                                 1/1     Running     0          2m
      openshift-authentication-operator authentication-operator-69d5d8bf84-vh2n8        1/1     Running     0          5m
      ```
   2. View the logs for a pod that is listed in the output of the previous command by using the following command:
      ```terminal
      $ oc logs <pod_name> -n <namespace>
      ```

      where:

      <dl>
      <dt><code>&lt;namespace&gt;</code></dt>
      <dd>Specifies the pod name and namespace, as shown in the output of an earlier command. If the pod logs display, the Kubernetes API server can communicate with the cluster machines.</dd>
      </dl>
3. For an installation with Fibre Channel Protocol (FCP), additional steps are required to enable multipathing. Do not enable multipathing during installation.
   See "Enabling multipathing with kernel arguments on RHCOS" in the *Postinstallation machine configuration tasks* documentation for more information.

**Verification**

If you have enabled secure boot during the OpenShift Container Platform bootstrap process, the following verification steps are required:

1. Debug the node by running the following command:
   ```terminal
   $ oc debug node/<node_name>
   ```

   ```terminal title="Example output"
   chroot /host
   ```
2. Confirm that secure boot is enabled by running the following command. Example output states `1` if secure boot is enabled and `0` if secure boot is not enabled.
   ```terminal
   $ cat /sys/firmware/ipl/secure
   ```

## Telemetry access for OpenShift Container Platform {#cluster-telemetry_installing-ibm-z}

To provide metrics about cluster health and the success of updates, the Telemetry service requires internet access. When connected, this service runs automatically by default and registers your cluster to [OpenShift Cluster Manager](https://console.redhat.com/openshift).

After you confirm that your [OpenShift Cluster Manager](https://console.redhat.com/openshift) inventory is correct, either maintained automatically by Telemetry or manually by using OpenShift Cluster Manager,use subscription watch to track your OpenShift Container Platform subscriptions at the account or multi-cluster level. For more information about subscription watch, see "Data Gathered and Used by Red Hat’s subscription services" in the *Additional resources* section.

**Additional resources**

- [About remote health monitoring](/docs/support/remote_health_monitoring/about-remote-health-monitoring#about-remote-health-monitoring)
- [How to generate SOSREPORT within OpenShift Container Platform version 4 nodes without SSH](https://access.redhat.com/solutions/4387261)
- [Remote health reporting](/docs/support/remote_health_monitoring/remote-health-reporting#remote-health-reporting)
- [Enabling multipathing with kernel arguments on RHCOS](/docs/machine_configuration/machine-configs-configure#rhcos-enabling-multipath-day-2_machine-configs-configure)
- [Customize your cluster](/docs/post_installation_configuration/cluster-tasks#available_cluster_customizations)
