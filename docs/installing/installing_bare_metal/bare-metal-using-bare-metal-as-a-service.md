---
title: "Using Red&#160;Hat Bare Metal as a Service for OpenShift"
sidebar_position: 6
---

# Using Red Hat Bare Metal as a Service for OpenShift {#bare-metal-using-bare-metal-as-a-service}

<a id="bare-metal-using-bare-metal-as-a-service"></a>

You can provision and manage bare-metal hosts by using the Metal^3^ API and the Bare Metal Operator (BMO). These hosts, external to the OpenShift Container Platform cluster, can run workloads that might not be suitable for containerization or virtualization, such as legacy applications or applications that require direct hardware access.

Red Hat Bare Metal as a Service for OpenShift has the following capabilities:

- Provisioning of bare-metal hosts, including initial configuration.
- Lifecycle management such as power management, firmware updates, and decommissioning by using the BMO.

As standalone systems, these hosts operate independently of the OpenShift Container Platform cluster and support diverse workloads by integrating bare metal resources with containerized and virtualized applications. Red Hat Bare Metal as a Service for OpenShift can run other operating systems, but only Red Hat Enterprise Linux (RHEL) and CentOS Stream 9 were tested.

## Prerequisites for using Red Hat Bare Metal as a Service for OpenShift {#bmaas-prerequisites_bare-metal-using-bmaas}

Red Hat Bare Metal as a Service for OpenShift lets you apply cloud-native management practices to bare-metal infrastructure, enabling automated provisioning and lifecycle management for workloads that require physical hardware. To use Red Hat Bare Metal as a Service for OpenShift, complete the following prerequisites:

BareMetalHost Configuration
: All bare-metal hosts must use a Baseboard Management Controller (BMC) configured with the Redfish protocol and virtual media (`redfish-virtualmedia`) driver. Each bare-metal host requires a boot interface with a MAC address configured to receive an IP address lease.

Network Requirements
: A DHCP server, separate from the OpenShift Container Platform and Metal^3^ infrastructure, must be operational on the same Layer 2 network as the bare-metal hosts. The DHCP server must be configured to match the MAC addresses of the boot interfaces on the bare-metal hosts, enabling IP address assignment for communication with Metal^3^ components.

Cluster Privileges
: You must have `cluster-admin` privileges on the OpenShift Container Platform cluster to perform configuration tasks.

Web server with images
: Red Hat Bare Metal as a Service for OpenShift does not provide images for deployment on hardware. You must configure a web server with the images and checksums you want to use. The `image` field of the `BareMetalHost` spec references these images during deployment. Ensure that the bare-metal hosts can reach the web server URL. Alternatively, you can access images from an OCI registry as a Technology Preview. This can be done by accessing a public registry such as Quay.io, or by hosting OCI images from the built-in registry in your cluster. The following is an example of images and checksums you might include:

  - `http://example.com/rhel9.qcow2`
  - `http://example.com/rhel9.qcow2.sha512sum`
  - `http://example.com/stream9.qcow2`
  - `http://example.com/stream9.qcow2.sha512sum`
  - `oci://quay.io/example/image:version`

These prerequisites ensure that Red Hat Bare Metal as a Service for OpenShift can provision and manage bare-metal hosts effectively.

:::warning

Accessing images from an OCI registry is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.

For more information about the support scope of Red Hat Technology Preview features, see [Technology Preview Features Support Scope](https://access.redhat.com/support/offerings/techpreview/).

:::

## Using the Bare Metal Operator to manage resources across all namespaces {#bmaas-using-the-bmo-to-manage-resources-across-all-namespaces_bare-metal-using-bmaas}

For the Bare Metal Operator (BMO) to manage `BareMetalHost` resources across all namespaces in your OpenShift Container Platform cluster, you must configure the Operator to watch all namespaces. This configuration is important to avoid mixing non-OpenShift Container Platform workloads with other components in the same namespace.

**Prerequisites**

- If you are using user-provisioned installation and the Provisioning CR does not exist, you must create it manually. For instructions, see [Configuring a provisioning resource to scale user-provisioned clusters](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html-single/installing_on_bare_metal/index#scaling-a-user-provisioned-cluster-with-the-bare-metal-operator). For installer-provisioned installations, the installation program creates the Provisioning custom resource (CR) automatically.

**Procedure**

- Patch the provisioning configuration to enable watching all namespaces by running the following command:
  ```terminal
  $ oc patch provisioning/provisioning-configuration \
    --type merge -p '{"spec": {"watchAllNamespaces": true}}'
  ```

  The BMO applies this change automatically.

## Setting up a dedicated namespace {#bmaas-setting-up-a-dedicated-namespace_bare-metal-using-bmaas}

To prevent accidental interference between Red Hat Bare Metal as a Service for OpenShift workloads and the OpenShift Container Platform infrastructure, set up a dedicated namespace. Repeat this procedure for every project where you intend to use Red Hat Bare Metal as a Service for OpenShift.

**Prerequisites**

- You have [configured an identify provider](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html-single/authentication_and_authorization/index#configuring-identity-providers).

**Procedure**

1. Configure a `bmadmin` user in the identity provider and create a secret in OpenShift:
   1. Create the `bmadmin` user in the identity provider. For example, if using the `htpasswd` identity provider, run the following command:
      ```terminal
      $ htpasswd -c -B -b ./users_htpasswd <username> <password>
      ```

      <dl>
      <dt>&lt;username&gt;</dt>
      <dd>The user name for the identity provider. Replace <code>&lt;username&gt;</code> with your preferred user name. This example uses <code>bmadmin</code>.</dd>
      <dt>&lt;password&gt;</dt>
      <dd>The password for the user. Replace <code>&lt;password&gt;</code> with a secure password.</dd>
      </dl>
   2. Create a secret in the `openshift-config` namespace to store the identity provider configuration by running the following command:
      ```terminal
      $ oc create secret generic <identity_provider_arguments> -n openshift-config
      ```

      For example, when using the `htpasswd` identity provider, run the following command:

      ```terminal
      $ oc create secret generic htpass-secret --from-file=htpasswd=users_htpasswd -n openshift-config
      ```

      <dl>
      <dt>&lt;identity_provider_arguments&gt;</dt>
      <dd>The arguments specific to the identity provider secret. Replace <code>&lt;identity_provider_arguments&gt;</code> with the appropriate arguments for your identity provider.</dd>
      </dl>
2. Configure OAuth to use the identity provider:
   1. Edit the OAuth resource by running the following command:
      ```terminal
      $ oc edit oauth cluster
      ```

      The editor opens and displays the Oauth resource.
   2. Add the identity provider configuration to the `spec.identityProviders` list:
      **Identity provider configuration examples**

<table>
<thead>
<tr>
  <th>Type</th>
  <th>Example</th>
</tr>
</thead>
<tbody>
<tr>
  <td>htpasswd</td>
  <td><pre><code># ...&#10;- name: my_bmaas_provider&#10;  mappingMethod: claim&#10;  type: htpasswd&#10;  htpasswd:&#10;    fileData:&#10;      name: &lt;secret&gt;&#10;# ...</code></pre></td>
</tr>
<tr>
  <td>LDAP</td>
  <td><pre><code># ...&#10;- name: my_bmaas_provider&#10;  mappingMethod: claim&#10;  type: ldap&#10;  ldap:&#10;    attributes:&#10;      id:&#10;      - dn&#10;      email:&#10;      - mail&#10;      name:&#10;      - cn&#10;      preferredUsername:&#10;      - uid&#10;# ...</code></pre></td>
</tr>
<tr>
  <td>GitHub</td>
  <td><pre><code># ...&#10;- name: my_bmaas_provider&#10;  mappingMethod: claim&#10;  type: GitHub&#10;    github:&#10;      ca:&#10;        name: ca-config-map&#10;      clientID: {...}&#10;      clientSecret:&#10;        name: github-secret&#10;      hostname: ...&#10;      organizations:&#10;      - myorganization1&#10;      - myorganization2&#10;      teams:&#10;      - myorganization1/team-a&#10;      - myorganization2/team-b&#10;# ...</code></pre></td>
</tr>
</tbody>
</table>

```
    For more information about identify providers, see [Authentication and authorization](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html-single/authentication_and_authorization/index).
1.  Save and exit the editor.
```

1. Create a `bmadmin` user by running the following command:
   ```terminal
   $ oc create user <username>
   ```

   <dl>
   <dt>&lt;username&gt;</dt>
   <dd>The user name. Replace <code>&lt;username&gt;</code> with your username. The following examples use <code>bmadmin</code> as the username.</dd>
   </dl>
2. Create a dedicated `bmaas` namespace for Red Hat Bare Metal as a Service for OpenShift hosts by running the following command:
   ```terminal
   $ oc new-project <namespace>
   ```

   <dl>
   <dt><code>&lt;namespace&gt;</code></dt>
   <dd>Replace &lt;namespace&gt; with the namespace name that you want to use. This example uses <code>bmaas</code>.</dd>
   </dl>
3. Assign the `edit` role to the `bmadmin` user in the `bmaas` namespace by running the following command:
   ```terminal
   $ oc adm policy add-role-to-user edit <username> -n bmaas
   ```
4. Clone the `baremetal-operator` repository to obtain the role-based access control (RBAC) role definitions by running the following command:
   ```terminal
   $ git clone -b release-4.22 https://github.com/openshift/baremetal-operator.git
   ```
5. For each role you want to add, apply the appropriate RBAC role YAML file from the repository by running the following command:
   ```terminal
   $ oc apply -f baremetal-operator/config/base/rbac/<role_filename>.yaml
   ```
6. Assign the custom RBAC roles to the `bmadmin` user in the `bmaas` namespace by running the following command:
   ```terminal
   $ oc adm policy add-role-to-user <role_name> bmadmin -n bmaas
   ```
7. Login as the `bmadmin` user by running the following command:
   ```terminal
   $ oc login <api_server_url>:6443
   ```

   <dl>
   <dt><code>&lt;api_server_url&gt;</code></dt>
   <dd>The URL to the Kubernetes API.</dd>
   </dl>

## Creating a BMC secret {#bmo-creating-a-bmc-secret_bare-metal-using-bmaas}

To deploy a bare-metal host, you must create a secret to access the baseboard management controller (BMC). This means you can remotely provision or manage the physical hardware.

**Procedure**

1. Create a BMC secret file by running the following command:
   ```terminal
   $ vim bmaas-<name>-bmc-secret.yaml
   ```

   Replace `<name>` with the name of the bare-metal host.
2. Edit the secret:
   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: bmaas-<name>-bmc-secret
     namespace: bmaas
   type: Opaque
   data:
     username: <base64_of_uid>
     password: <base64_of_pwd>
   ```

   <dl>
   <dt>&lt;base64_of_uid&gt;</dt>
   <dd>Replace <code>&lt;base64_of_uid&gt;</code> with the BMC user name as a Base64-encoded string.</dd>
   <dt>&lt;base64_of_pwd&gt;</dt>
   <dd>Replace <code>&lt;base64_of_pwd&gt;</code> with the BMC password as a Base64-encoded string.</dd>
   </dl>
3. Apply the BMC secret by running the following command:
   ```terminal
   $ oc apply -f bmaas-<name>-bmc-secret.yaml
   ```

**Additional resources**

- [About BMC addressing](/docs/installing/installing_bare_metal/ipi/ipi-install-installation-workflow#bmc-addressing_ipi-install-installation-workflow)

## Creating a bare-metal host resource {#bmo-creating-a-bare-metal-host-resource_bare-metal-using-bmaas}

To deploy a bare-metal host, you must create a `BareMetalHost` resource.

**Procedure**

1. Create a `BareMetalHost` custom resource (CR) file by running the following command:
   ```terminal
   $ vim bmaas-<name>-bmh.yaml
   ```

   **<name>**

   ```
     Replace `<name>` with the name of the bare-metal host.
   ```
2. Edit the CR:
   ```yaml
   apiVersion: metal3.io/v1alpha1
   kind: BareMetalHost
   metadata:
     name: bmaas-<name>
     namespace:  bmaas
   spec:
     online: true
     bootMACAddress: <mac_addr>
     bmc:
       address: redfish-virtualmedia+<address>/redfish/v1/Systems/System.Embedded.1
       credentialsName: bmaas-<num>-bmc-secret
   ```

   **<mac_addr>**

   ```
     Replace `<mac_addr>` with the MAC address of the first NIC on the bare-metal host.
   ```

   **<address>**

   ```
     Replace `<address>` with IP address or FQDN of the host.
   ```
3. Apply the CR by running the following command:
   ```terminal
   $ oc apply -f bmaas-<name>-bmh.yaml
   ```

**Verification**

- Check the `BareMetalHost` state by running the following command:
  ```terminal
  $ oc get baremetalhost -n bmaas
  ```

  The state progresses from **registering**, to **inspecting**, and finally to **available**.

## Configuring users for Red Hat Bare Metal as a Service for OpenShift hosts {#bmo-configuring-users-for-bmaas-hosts_bare-metal-using-bmaas}

Configure bare-metal host users and add them to a Kubernetes secret. Then, create and apply the secret to customize the host. With configured users you can access or manage the bare-metal host after it is provisioned.

**Procedure**

1. Create a file named `<hostname>-user-data.yaml`, where `<hostname>` is the name of the bare-metal host, with the following content:
   ```yaml
   users:
     - name: <name>
       sudo: [<sudo_config>]
       ssh_authorized_keys:
         - <key_type>
         <key>
       shell: <shell_path>
       groups: [<groups>]
       lock_passwd: true|false
   ```

   <dl>
   <dt><code>users.name</code></dt>
   <dd>The user name.</dd>
   <dt><code>users.sudo</code></dt>
   <dd>The sudo configuration for the user.</dd>
   <dt><code>users.ssh_authorized_keys.&lt;key_type&gt;</code></dt>
   <dd>The SSH key type.</dd>
   <dt><code>users.ssh_authorized_keys.&lt;key&gt;</code></dt>
   <dd>The public SSH key to use when accessing this host as the <code>&lt;name&gt;</code> user.</dd>
   <dt><code>users.shell</code></dt>
   <dd>The shell to use when accessing the host.</dd>
   <dt><code>users.groups</code></dt>
   <dd>The groups the user belongs to.</dd>
   <dt><code>users.lock_passwd</code></dt>
   <dd>Whether the user password is locked. If <code>true</code>, the user cannot log in by using the password, but can still use SSH.</dd>
   </dl>

   ```yaml title="Example user"
   users:
     - name: sysadmin
       sudo: ["ALL=(ALL) NOPASSWD:ALL"]
       ssh_authorized_keys:
         - ssh-rsa AAAAB3NzaC1yc2E... sysadmin@workstation.example.com
       shell: /bin/bash
       groups: [adm, sudo]
       lock_passwd: true
   ```
2. Create a secret from the `<hostname>-user-data.yaml` file by running the following command:
   ```terminal
   $ oc create secret generic <hostname>-user-data \
     --from-file=userData=<hostname>-user-data.yaml -n bmaas
   ```

   **`<hostname>`**

   ```
   The name of the bare-metal host.
   ```
3. Configure the `BareMetalHost` to use the `<hostname>-user-data.yaml` file by running the following command:
   ```terminal
   $ oc patch baremetalhost <hostname> -n bmaas \
        --type merge -p '{"spec":{"userData":{"name":"<hostname>-user-data"}}}'
   ```

   **`<hostname>`**

   ```
   The name of the bare-metal host.
   ```

## Configuring the networkData parameter in the BareMetalHost resource {#bmo-configuring-the-networkdata-parameter-in-the-bmo-cr_bare-metal-using-bmaas}

The `networkData` field in the `BareMetalHost` custom resource (CR) allows you to control the network configuration of the bare-metal host at creation time. For most operating systems, this is achieved using a configuration file encapsulated in a Kubernetes secret. Then, the `cloud-init` service uses it to customize services.

**Procedure**

1. Create a file named `network-data.yaml` with the following content:
   ```yaml
   links:
     - id: <interface_id>
       type: phy
       ethernet_mac_address: <mac_address>
   networks:
     - id: <interface_id>
       link: <interface_id>
       type: ipv4_dhcp
   services:
     - type: dns
       address: <dns_server>
   ```

   <dl>
   <dt><code>&lt;interface_id&gt;</code></dt>
   <dd>The ID of the network interface, such as <code>enp2s0</code>.</dd>
   <dt><code>&lt;mac_address&gt;</code></dt>
   <dd>The MAC address of the network interface.</dd>
   <dt><code>&lt;dns_server&gt;</code></dt>
   <dd>The IP address of the DNS server.</dd>
   </dl>
2. Create a secret from the `networkData` file by running the following command:
   ```terminal
   $ oc create secret generic <hostname>-network-data \
     --from-file=networkData=network-data.yaml -n bmaas
   ```

   <dl>
   <dt><code>&lt;hostname&gt;</code></dt>
   <dd>The hostname of the bare-metal host.</dd>
   </dl>
3. Configure the `BareMetalHost` to use the `networkData` file by running the following command:
   ```terminal
   $ oc patch baremetalhost <hostname> -n bmaas \
     --type merge -p '{"spec":{"networkData":{"name":"<hostname>-network-data"}}}'
   ```

## Deploying an image to the bare-metal host {#bmo-deploying-an-image-to-the-bare-metal-host_bare-metal-using-bmaas}

To deploy the image to the host, update the `image` field in the `spec` section of the `BareMetalHost` resource. Once you update the `image` field, provisioning begins immediately. Deploying an image transforms bare hardware into a functional system ready to run your workloads, in an automated and repeatable way.

**Procedure**

- Update the `image` field in the `BareMetalHost` CR by running the following command:
  ```terminal
  $ oc patch baremetalhost <hostname> \
    --type merge -p '{"spec": {"image": {"url": "<image_url>", "checksum": "<checksum_url>", "checksumType": "auto"}}}'
  ```

  <dl>
  <dt><code>&lt;hostname&gt;</code></dt>
  <dd>The name of your <code>BareMetalHost</code> resource.</dd>
  <dt><code>&lt;image_url&gt;</code></dt>
  <dd>The URL of the image to deploy. You can access images using the HTTP and OCI protocols. Accessing images using the OCI protocol is available as a Technology Preview.</dd>
  <dt><code>&lt;checksum_url&gt;</code></dt>
  <dd>The URL of the checksum file for the image.</dd>
  </dl>
