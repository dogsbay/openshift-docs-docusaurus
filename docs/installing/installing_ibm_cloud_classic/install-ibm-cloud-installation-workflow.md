---
title: Setting up the environment for an OpenShift Container Platform installation on IBM Cloud(R) Bare Metal (Classic)
sidebar_position: 2
---

# Setting up the environment for an OpenShift Container Platform installation on IBM Cloud(R) Bare Metal (Classic) {#install-ibm-cloud-installation-workflow}

<a id="install-ibm-cloud-installation-workflow"></a>

After you complete the prerequisites, set up the environment for an OpenShift Container Platform installation on IBM Cloud(R) Bare Metal (Classic) by preparing the provisioner node, configuring the network, and deploying the cluster.

## Preparing the provisioner node on IBM Cloud(R) Bare Metal (Classic) infrastructure {#preparing-the-provisioner-node-for-openshift-install-on-ibm-cloud_install-ibm-cloud-installation-workflow}

Before you install OpenShift Container Platform on IBM Cloud(R) Bare Metal (Classic) infrastructure, prepare the provisioner node by creating a non-root user, configuring network bridges, registering the node, installing required packages, and downloading the pull secret.

**Procedure**

1. Log in to the provisioner node via `ssh`.
2. Create a non-root user (`kni`) and give that user `sudo` privileges:
   ```terminal
   # useradd kni
   ```

   ```terminal
   # passwd kni
   ```

   ```terminal
   # echo "kni ALL=(root) NOPASSWD:ALL" | tee -a /etc/sudoers.d/kni
   ```

   ```terminal
   # chmod 0440 /etc/sudoers.d/kni
   ```
3. Create an `ssh` key for the new user:
   ```terminal
   # su - kni -c "ssh-keygen -f /home/kni/.ssh/id_rsa -N ''"
   ```
4. Log in as the new user on the provisioner node:
   ```terminal
   # su - kni
   ```
5. Use Red Hat Subscription Manager to register the provisioner node:
   ```terminal
   $ sudo subscription-manager register --username=<user> --password=<pass> --auto-attach
   ```

   ```terminal
   $ sudo subscription-manager repos --enable=rhel-8-for-x86_64-appstream-rpms \
                                     --enable=rhel-8-for-x86_64-baseos-rpms
   ```

   :::note

   For more information about Red Hat Subscription Manager, see [Registering a RHEL system with command-line tools](https://docs.redhat.com/en/documentation/subscription_central/1-latest/html/getting_started_with_rhel_system_registration/basic-reg-rhel-cli).

   :::
6. Install the following packages:
   ```terminal
   $ sudo dnf install -y libvirt qemu-kvm mkisofs python3-devel jq ipmitool
   ```
7. Change the user to add the `libvirt` group to the newly created user:
   ```terminal
   $ sudo usermod --append --groups libvirt kni
   ```
8. Start `firewalld`:
   ```terminal
   $ sudo systemctl start firewalld
   ```
9. Enable `firewalld`:
   ```terminal
   $ sudo systemctl enable firewalld
   ```
10. Start the `http` service:
    ```terminal
    $ sudo firewall-cmd --zone=public --add-service=http --permanent
    ```

    ```terminal
    $ sudo firewall-cmd --reload
    ```
11. Start and enable the `libvirtd` service:
    ```terminal
    $ sudo systemctl enable libvirtd --now
    ```
12. Set the ID of the provisioner node:
    ```terminal
    $ PRVN_HOST_ID=<ID>
    ```

    You can view the ID with the following `ibmcloud` command:

    ```terminal
    $ ibmcloud sl hardware list
    ```
13. Set the ID of the public subnet:
    ```terminal
    $ PUBLICSUBNETID=<ID>
    ```

    You can view the ID with the following `ibmcloud` command:

    ```terminal
    $ ibmcloud sl subnet list
    ```
14. Set the ID of the private subnet:
    ```terminal
    $ PRIVSUBNETID=<ID>
    ```

    You can view the ID with the following `ibmcloud` command:

    ```terminal
    $ ibmcloud sl subnet list
    ```
15. Set the provisioner node public IP address:
    ```terminal
    $ PRVN_PUB_IP=$(ibmcloud sl hardware detail $PRVN_HOST_ID --output JSON | jq .primaryIpAddress -r)
    ```
16. Set the CIDR for the public network:
    ```terminal
    $ PUBLICCIDR=$(ibmcloud sl subnet detail $PUBLICSUBNETID --output JSON | jq .cidr)
    ```
17. Set the IP address and CIDR for the public network:
    ```terminal
    $ PUB_IP_CIDR=$PRVN_PUB_IP/$PUBLICCIDR
    ```
18. Set the gateway for the public network:
    ```terminal
    $ PUB_GATEWAY=$(ibmcloud sl subnet detail $PUBLICSUBNETID --output JSON | jq .gateway -r)
    ```
19. Set the private IP address of the provisioner node:
    ```terminal
    $ PRVN_PRIV_IP=$(ibmcloud sl hardware detail $PRVN_HOST_ID --output JSON | \
                     jq .primaryBackendIpAddress -r)
    ```
20. Set the CIDR for the private network:
    ```terminal
    $ PRIVCIDR=$(ibmcloud sl subnet detail $PRIVSUBNETID --output JSON | jq .cidr)
    ```
21. Set the IP address and CIDR for the private network:
    ```terminal
    $ PRIV_IP_CIDR=$PRVN_PRIV_IP/$PRIVCIDR
    ```
22. Set the gateway for the private network:
    ```terminal
    $ PRIV_GATEWAY=$(ibmcloud sl subnet detail $PRIVSUBNETID --output JSON | jq .gateway -r)
    ```
23. Set up the bridges for the `baremetal` and `provisioning` networks:
    ```terminal
    $ sudo nohup bash -c "
        nmcli --get-values UUID con show | xargs -n 1 nmcli con delete
        nmcli connection add ifname provisioning type bridge con-name provisioning
        nmcli con add type bridge-slave ifname eth1 master provisioning
        nmcli connection add ifname baremetal type bridge con-name baremetal
        nmcli con add type bridge-slave ifname eth2 master baremetal
        nmcli connection modify baremetal ipv4.addresses $PUB_IP_CIDR ipv4.method manual ipv4.gateway $PUB_GATEWAY
        nmcli connection modify provisioning ipv4.addresses 172.22.0.1/24,$PRIV_IP_CIDR ipv4.method manual
        nmcli connection modify provisioning +ipv4.routes \"10.0.0.0/8 $PRIV_GATEWAY\"
        nmcli con down baremetal
        nmcli con up baremetal
        nmcli con down provisioning
        nmcli con up provisioning
        init 6
    "
    ```

    :::note

    For `eth1` and `eth2`, substitute the appropriate interface name, as needed.

    :::
24. If required, SSH back into the `provisioner` node:
    ```terminal
    # ssh kni@provisioner.<cluster_name>.<domain>
    ```
25. Verify the connection bridges have been properly created:
    ```terminal
    $ sudo nmcli con show
    ```

    ```terminal title="Example output"
    NAME               UUID                                  TYPE      DEVICE
    baremetal          4d5133a5-8351-4bb9-bfd4-3af264801530  bridge    baremetal
    provisioning       43942805-017f-4d7d-a2c2-7cb3324482ed  bridge    provisioning
    virbr0             d9bca40f-eee1-410b-8879-a2d4bb0465e7  bridge    virbr0
    bridge-slave-eth1  76a8ed50-c7e5-4999-b4f6-6d9014dd0812  ethernet  eth1
    bridge-slave-eth2  f31c3353-54b7-48de-893a-02d2b34c4736  ethernet  eth2
    ```
26. Create a `pull-secret.txt` file:
    ```terminal
    $ vim pull-secret.txt
    ```

    Go to [Install on Bare Metal with user-provisioned infrastructure](https://console.redhat.com/openshift/install/metal/user-provisioned). In step 1, click **Download pull secret**. Paste the contents into the `pull-secret.txt` file and save the contents in the `kni` user’s home directory.

## Configuring the public subnet {#configuring-the-public-subnet_install-ibm-cloud-installation-workflow}

To offer DHCP services for OpenShift Container Platform cluster nodes on the public subnet in IBM Cloud(R) Bare Metal (Classic), you can install and configure `dnsmasq` on the provisioner node.

All of the OpenShift Container Platform cluster nodes must be on the public subnet. IBM Cloud(R) Bare Metal (Classic) does not offer a DHCP server on the subnet. Set it up separately on the provisioner node.

You must reset the BASH variables defined when preparing the provisioner node. Rebooting the provisioner node after preparing it will delete the BASH variables set before.

**Procedure**

1. Install `dnsmasq`:
   ```terminal
   $ sudo dnf install dnsmasq
   ```
2. Open the `dnsmasq` configuration file:
   ```terminal
   $ sudo vi /etc/dnsmasq.conf
   ```
3. Add the following configuration to the `dnsmasq` configuration file:
   ```text
   interface=baremetal
   except-interface=lo
   bind-dynamic
   log-dhcp

   dhcp-range=<ip_addr>,<ip_addr>,<pub_cidr>
   dhcp-option=baremetal,121,0.0.0.0/0,<pub_gateway>,<prvn_priv_ip>,<prvn_pub_ip>

   dhcp-hostsfile=/var/lib/dnsmasq/dnsmasq.hostsfile
   ```

   where:

   <dl>
   <dt><code>dhcp-range</code></dt>
   <dd>Specifies the DHCP range for the <code>baremetal</code> network. Replace both instances of <code>&lt;ip_addr&gt;</code> with one unused IP address from the public subnet so that the range begins and ends with the same IP address. Replace <code>&lt;pub_cidr&gt;</code> with the CIDR of the public subnet.</dd>
   </dl>

`dhcp-option`
: Specifies the DHCP option for the `baremetal` network. Replace `<pub_gateway>` with the IP address of the gateway for the `baremetal` network. Replace `<prvn_priv_ip>` with the private IP address of the provisioner node on the `provisioning` network. Replace `<prvn_pub_ip>` with the public IP address of the provisioner node on the `baremetal` network.

1. To retrieve the value for `<pub_cidr>`, run the following command:
   ```terminal
   $ ibmcloud sl subnet detail <publicsubnetid> --output JSON | jq .cidr
   ```

   Replace `<publicsubnetid>` with the ID of the public subnet.
2. To retrieve the value for `<pub_gateway>`, run the following command:
   ```terminal
   $ ibmcloud sl subnet detail <publicsubnetid> --output JSON | jq .gateway -r
   ```

   Replace `<publicsubnetid>` with the ID of the public subnet.
3. To retrieve the value for `<prvn_priv_ip>`, run the following command:
   ```terminal
   $ ibmcloud  sl hardware detail <id> --output JSON | \
               jq .primaryBackendIpAddress -r
   ```

   Replace `<id>` with the ID of the provisioner node.
4. To retrieve the value for `<prvn_pub_ip>`, run the following command:
   ```terminal
   $ ibmcloud sl hardware detail <id> --output JSON | jq .primaryIpAddress -r
   ```

   Replace `<id>` with the ID of the provisioner node.
5. Obtain the list of hardware for the cluster:
   ```terminal
   $ ibmcloud sl hardware list
   ```
6. Obtain the MAC addresses and IP addresses for each node:
   ```terminal
   $ ibmcloud sl hardware detail <id> --output JSON | \
     jq '.networkComponents[] | \
     "\(.primaryIpAddress) \(.macAddress)"' | grep -v null
   ```

   Replace `<id>` with the ID of the node.

   ```terminal title="Example output"
   "10.196.130.144 00:e0:ed:6a:ca:b4"
   "141.125.65.215 00:e0:ed:6a:ca:b5"
   ```

   Make a note of the MAC address and IP address of the public network. Make a separate note of the MAC address of the private network, which you will use later in the `install-config.yaml` file. Repeat this procedure for each node until you have all the public MAC and IP addresses for the public `baremetal` network, and the MAC addresses of the private `provisioning` network.
7. Add the MAC and IP address pair of the public `baremetal` network for each node into the `dnsmasq.hostsfile` file:
   ```terminal
   $ sudo vim /var/lib/dnsmasq/dnsmasq.hostsfile
   ```

   ```text title="Example input"
   00:e0:ed:6a:ca:b5,141.125.65.215,master-0
   <mac>,<ip>,master-1
   <mac>,<ip>,master-2
   <mac>,<ip>,worker-0
   <mac>,<ip>,worker-1
   ...
   ```

   Replace `<mac>,<ip>` with the public MAC address and public IP address of the corresponding node name.
8. Start `dnsmasq`:
   ```terminal
   $ sudo systemctl start dnsmasq
   ```
9. Enable `dnsmasq` so that it starts when booting the node:
   ```terminal
   $ sudo systemctl enable dnsmasq
   ```
10. Verify `dnsmasq` is running:
    ```terminal
    $ sudo systemctl status dnsmasq
    ```

    ```terminal title="Example output"
    ● dnsmasq.service - DNS caching server.
    Loaded: loaded (/usr/lib/systemd/system/dnsmasq.service; enabled; vendor preset: disabled)
    Active: active (running) since Tue 2021-10-05 05:04:14 CDT; 49s ago
    Main PID: 3101 (dnsmasq)
    Tasks: 1 (limit: 204038)
    Memory: 732.0K
    CGroup: /system.slice/dnsmasq.service
    └─3101 /usr/sbin/dnsmasq -k
    ```
11. Open ports `53` and `67` with UDP protocol:
    ```terminal
    $ sudo firewall-cmd --add-port 53/udp --permanent
    ```

    ```terminal
    $ sudo firewall-cmd --add-port 67/udp --permanent
    ```
12. Add `provisioning` to the external zone with masquerade:
    ```terminal
    $ sudo firewall-cmd --change-zone=provisioning --zone=external --permanent
    ```

    This step ensures network address translation for IPMI calls to the management subnet.
13. Reload the `firewalld` configuration:
    ```terminal
    $ sudo firewall-cmd --reload
    ```

## Retrieving the OpenShift Container Platform installer {#retrieving-the-openshift-installer_install-ibm-cloud-installation-workflow}

Use the `stable-4.x` version of the installation program and your selected architecture to deploy the generally available stable version of OpenShift Container Platform.

**Procedure**

- Retrieve the installation program by running one of the following commands:
  ```terminal
  $ export VERSION=stable-4.22
  ```

  ```terminal
  $ export RELEASE_ARCH=<architecture>
  ```

  ```terminal
  $ export RELEASE_IMAGE=$(curl -s https://mirror.openshift.com/pub/openshift-v4/$RELEASE_ARCH/clients/ocp/$VERSION/release.txt | grep 'Pull From: quay.io' | awk -F ' ' '{print $3}')
  ```

## Extracting the OpenShift Container Platform installer {#extracting-the-openshift-installer_install-ibm-cloud-installation-workflow}

Extract the OpenShift Container Platform installer after retrieving it to prepare for the installation of the cluster.

**Procedure**

1. Set the environment variables:
   ```terminal
   $ export cmd=openshift-baremetal-install
   ```

   ```terminal
   $ export pullsecret_file=~/pull-secret.txt
   ```

   ```terminal
   $ export extract_dir=$(pwd)
   ```
2. Get the `oc` binary:
   ```terminal
   $ curl -s https://mirror.openshift.com/pub/openshift-v4/clients/ocp/$VERSION/openshift-client-linux.tar.gz | tar zxvf - oc
   ```
3. Extract the installer:
   ```terminal
   $ sudo cp oc /usr/local/bin
   ```

   ```terminal
   $ oc adm release extract --registry-config "${pullsecret_file}" --command=$cmd --to "${extract_dir}" ${RELEASE_IMAGE}
   ```

   ```terminal
   $ sudo cp openshift-baremetal-install /usr/local/bin
   ```

## Configuring the install-config.yaml file {#configuring-the-install-config-file_install-ibm-cloud-installation-workflow}

To configure OpenShift Container Platform for IBM Cloud(R) Bare Metal (Classic) infrastructure, you can edit the `install-config.yaml` file to set the required IPMI privilege level and hardware parameters for your bare-metal nodes.

The `install-config.yaml` file requires some additional details. Most of the information is teaching the installation program and the resulting cluster enough about the available IBM Cloud(R) Bare Metal (Classic) hardware so that it is able to fully manage it. The material difference between installing on bare metal and installing on IBM Cloud(R) Bare Metal (Classic) is that you must explicitly set the privilege level for IPMI in the BMC section of the `install-config.yaml` file.

**Procedure**

1. Configure `install-config.yaml`. Change the appropriate variables to match the environment, including `pullSecret` and `sshKey`.
   ```yaml
   apiVersion: v1
   baseDomain: <domain>
   metadata:
     name: <cluster_name>
   networking:
     machineNetwork:
     - cidr: <public-cidr>
     networkType: OVNKubernetes
   compute:
   - name: worker
     replicas: 2
   controlPlane:
     name: master
     replicas: 3
     platform:
       baremetal: {}
   platform:
     baremetal:
       apiVIP: <api_ip>
       ingressVIP: <wildcard_ip>
       provisioningNetworkInterface: <NIC1>
       provisioningNetworkCIDR: <CIDR>
       hosts:
         - name: openshift-master-0
           role: master
           bmc:
             address: ipmi://10.196.130.145?privilegelevel=OPERATOR
             username: root
             password: <password>
           bootMACAddress: 00:e0:ed:6a:ca:b4
           rootDeviceHints:
             deviceName: "/dev/sda"
         - name: openshift-worker-0
           role: worker
           bmc:
             address: ipmi://<out-of-band-ip>?privilegelevel=OPERATOR
             username: <user>
             password: <password>
           bootMACAddress: <NIC1_mac_address>
           rootDeviceHints:
             deviceName: "/dev/sda"
   pullSecret: '<pull_secret>'
   sshKey: '<ssh_pub_key>'
   ```

   where:

   <dl>
   <dt><code>bmc.address</code></dt>
   <dd>Specifies the IPMI address with <code>privilegelevel=OPERATOR</code>. IBM Cloud(R) Bare Metal (Classic) infrastructure requires this privilege level.</dd>
   </dl>

`bootMACAddress`
: Specifies the MAC address of the private `provisioning` network NIC for the corresponding node.

  :::note

  You can use the `ibmcloud` command-line utility to retrieve the password.

  ```terminal
  $ ibmcloud sl hardware detail <id> --output JSON | \
    jq '"(.networkManagementIpAddress) (.remoteManagementAccounts[0].password)"'
  ```

  Replace `<id>` with the ID of the node.

  :::

1. Create a directory to store the cluster configuration:
   ```terminal
   $ mkdir ~/clusterconfigs
   ```
2. Copy the `install-config.yaml` file into the directory:
   ```terminal
   $ cp install-config.yaml ~/clusterconfigs
   ```
3. Power off all bare-metal nodes before installing the OpenShift Container Platform cluster:
   ```terminal
   $ ipmitool -I lanplus -U <user> -P <password> -H <management_server_ip> power off
   ```
4. Remove old bootstrap resources if any remain from an earlier deployment try:
   ```bash
   for i in $(sudo virsh list | tail -n +3 | grep bootstrap | awk {'print $2'});
   do
     sudo virsh destroy $i;
     sudo virsh undefine $i;
     sudo virsh vol-delete $i --pool $i;
     sudo virsh vol-delete $i.ign --pool $i;
     sudo virsh pool-destroy $i;
     sudo virsh pool-undefine $i;
   done
   ```

## Additional installation configuration parameters {#additional-install-config-parameters_install-ibm-cloud-installation-workflow}

Some parameters, such as the cluster domain name, are required in the `install-config.yaml` file when installing a cluster on bare metal. Others, such as the provisioning network CIDR, are optional.

**Required parameters**

<table>
<thead>
<tr>
  <th>Parameters</th>
  <th>Default</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>baseDomain</code></td>
  <td></td>
  <td>The domain name for the cluster. For example, <code>example.com</code>.</td>
</tr>
<tr>
  <td><code>bootMode</code></td>
  <td><code>UEFI</code></td>
  <td>The boot mode for a node. Options are <code>legacy</code>, <code>UEFI</code>, and <code>UEFISecureBoot</code>. If <code>bootMode</code> is not set, Ironic sets it while inspecting the node.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>For hardware that implements <code>BootMode</code> read-only, such as HP or Cisco, do not leave this parameter blank. You must manually set the system to UEFI mode before installation and explicitly set this parameter to UEFI.</p></div></td>
</tr>
<tr>
  <td><pre><code>platform:&#10;  baremetal:&#10;    bootstrapExternalStaticDNS</code></pre></td>
  <td></td>
  <td>The static network DNS of the bootstrap node. You must set this value when deploying a cluster with static IP addresses when there is no Dynamic Host Configuration Protocol (DHCP) server on the bare-metal network. If you do not set this value, the installation program will use the value from <code>bootstrapExternalStaticGateway</code>, which causes problems when the IP address values of the gateway and DNS are different.</td>
</tr>
<tr>
  <td><pre><code>platform:&#10;  baremetal:&#10;    bootstrapExternalStaticIP</code></pre></td>
  <td></td>
  <td>The static IP address for the bootstrap VM. You must set this value when deploying a cluster with static IP addresses when there is no DHCP server on the bare metal network.</td>
</tr>
<tr>
  <td><pre><code>platform:&#10;  baremetal:&#10;    bootstrapExternalStaticGateway</code></pre></td>
  <td></td>
  <td>The static IP address of the gateway for the bootstrap VM. You must set this value when deploying a cluster with static IP addresses when there is no DHCP server on the bare metal network.</td>
</tr>
<tr>
  <td><code>sshKey</code></td>
  <td></td>
  <td>The <code>sshKey</code> parameter sets the key in the <code>~/.ssh/id_rsa.pub</code> file required to access the control plane nodes and compute nodes. Typically, this key is from the <code>provisioner</code> node.</td>
</tr>
<tr>
  <td><code>pullSecret</code></td>
  <td></td>
  <td>The <code>pullSecret</code> parameter sets a copy of the pull secret downloaded from the <a href="https://console.redhat.com/openshift/install/metal/user-provisioned">Install OpenShift on Bare Metal</a> page when preparing the provisioner node.</td>
</tr>
<tr>
  <td><pre><code>metadata:&#10;    name:</code></pre></td>
  <td></td>
  <td>The OpenShift Container Platform cluster name. For example, <code>openshift</code>.</td>
</tr>
<tr>
  <td><pre><code>networking:&#10;    machineNetwork:&#10;    - cidr:</code></pre></td>
  <td></td>
  <td>The public CIDR (Classless Inter-Domain Routing) of the external network. For example, <code>10.0.0.0/24</code>.</td>
</tr>
<tr>
  <td><pre><code>compute:&#10;  - name: worker</code></pre></td>
  <td></td>
  <td>The OpenShift Container Platform cluster requires a name for each compute node even if there are zero nodes.</td>
</tr>
<tr>
  <td><pre><code>compute:&#10;    replicas: 2</code></pre></td>
  <td></td>
  <td>Replicas sets the number of compute nodes in the OpenShift Container Platform cluster.</td>
</tr>
<tr>
  <td><pre><code>controlPlane:&#10;    name: master</code></pre></td>
  <td></td>
  <td>The OpenShift Container Platform cluster requires a name for control plane nodes.</td>
</tr>
<tr>
  <td><pre><code>controlPlane:&#10;    replicas: 3</code></pre></td>
  <td></td>
  <td>Replicas sets the number of control plane nodes included as part of the OpenShift Container Platform cluster.</td>
</tr>
<tr>
  <td><code>provisioningNetworkInterface</code></td>
  <td></td>
  <td>The name of the network interface on nodes connected to the provisioning network. For OpenShift Container Platform 4.9 and later releases, use the <code>bootMACAddress</code> parameter to enable Ironic to identify the IP address of the NIC instead of using the <code>provisioningNetworkInterface</code> parameter to identify the name of the NIC.</td>
</tr>
<tr>
  <td><code>defaultMachinePlatform</code></td>
  <td></td>
  <td>The default configuration used for machine pools without a platform configuration.</td>
</tr>
<tr>
  <td><code>apiVIPs</code></td>
  <td>(Optional) The virtual IP address for Kubernetes API communication.<br><br>You must either provide this setting in the <code>install-config.yaml</code> file as a reserved IP from the <code>MachineNetwork</code> parameter or preconfigured in the DNS so that the default name resolves correctly. Use the virtual IP address and not the FQDN when adding a value to the <code>apiVIPs</code> configuration setting in the <code>install-config.yaml</code> file. For dual-stack networking, the primary IP address can be either an IPv4 network or an IPv6 network. If not set, the installation program uses <code>api.&lt;cluster_name&gt;.&lt;base_domain&gt;</code> to derive the IP address from the DNS.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>Before OpenShift Container Platform 4.12, the cluster installation program only accepted an IPv4 address or an IPv6 address for the <code>apiVIP</code> parameter. From OpenShift Container Platform 4.12 or later, the <code>apiVIP</code> parameter is deprecated. Instead, use a list format for the <code>apiVIPs</code> parameter to specify an IPv4 address, an IPv6 address or both IP address formats.</p></div></td>
  <td><code>bmcCACert</code></td>
</tr>
<tr>
  <td></td>
  <td><code>redfish</code> and <code>redfish-virtualmedia</code> need this parameter to manage BMC addresses when using self-signed certificates with <code>disableCertificateVerification</code> set to <code>False</code>.</td>
  <td><code>ingressVIPs</code></td>
</tr>
<tr>
  <td>(Optional) The virtual IP address for ingress traffic.<br><br>You must either provide this setting in the <code>install-config.yaml</code> file as a reserved IP from the <code>MachineNetwork</code> parameter or preconfigured in the DNS so that the default name resolves correctly. Use the virtual IP address and not the FQDN when adding a value to the <code>ingressVIPs</code> configuration setting in the <code>install-config.yaml</code> file. For dual-stack networking, the primary IP address can be either an IPv4 network or an IPv6 network. If not set, the installation program uses <code>test.apps.&lt;cluster_name&gt;.&lt;base_domain&gt;</code> to derive the IP address from the DNS.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>Before OpenShift Container Platform 4.12, the cluster installation program only accepted an IPv4 address or an IPv6 address for the <code>ingressVIP</code> parameter. In OpenShift Container Platform 4.12 and later, the <code>ingressVIP</code> parameter is deprecated. Instead, use a list format for the <code>ingressVIPs</code> parameter to specify an IPv4 addresses, an IPv6 addresses or both IP address formats.</p></div></td>
</tr>
</tbody>
</table>

**Optional Parameters**

<table>
<thead>
<tr>
  <th>Parameters</th>
  <th>Default</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><pre><code>platform:&#10;  baremetal:&#10;    additionalNTPServers:&#10;    - &lt;ip_address_or_domain_name&gt;</code></pre></td>
  <td></td>
  <td>An optional list of additional NTP servers to add to each host. You can use an IP address or a domain name to specify each NTP server. Additional NTP servers are user-defined NTP servers that enable preinstallation clock synchronization when the cluster host clocks are out of synchronization.</td>
</tr>
<tr>
  <td><code>provisioningDHCPRange</code></td>
  <td><code>172.22.0.10,172.22.0.100</code></td>
  <td>Defines the IP range for nodes on the provisioning network.</td>
</tr>
<tr>
  <td><code>provisioningNetworkCIDR</code></td>
  <td><code>172.22.0.0/24</code></td>
  <td>The CIDR for the network to use for provisioning. When not using the default address range on the provisioning network, you must set this configuration parameter.</td>
</tr>
<tr>
  <td><code>clusterProvisioningIP</code></td>
  <td>The third IP address of the <code>provisioningNetworkCIDR</code>.</td>
  <td>The IP address within the cluster where the provisioning services run. Defaults to the third IP address of the provisioning subnet. For example, <code>172.22.0.3</code>.</td>
</tr>
<tr>
  <td><code>bootstrapProvisioningIP</code></td>
  <td>The second IP address of the <code>provisioningNetworkCIDR</code>.</td>
  <td>The IP address on the bootstrap VM where the provisioning services run while the installation program is deploying the control plane nodes. Defaults to the second IP address of the provisioning subnet. For example, <code>172.22.0.2</code> or <code>2620:52:0:1307::2</code>.</td>
</tr>
<tr>
  <td><code>externalBridge</code></td>
  <td><code>baremetal</code></td>
  <td>The name of the bare metal bridge of the hypervisor attached to the bare metal network.</td>
</tr>
<tr>
  <td><code>provisioningBridge</code></td>
  <td><code>provisioning</code></td>
  <td>The name of the provisioning bridge on the <code>provisioner</code> host attached to the provisioning network.</td>
</tr>
<tr>
  <td><code>architecture</code></td>
  <td></td>
  <td>Defines the host architecture for your cluster. Valid values are <code>amd64</code> or <code>arm64</code>.</td>
</tr>
<tr>
  <td><code>defaultMachinePlatform</code></td>
  <td></td>
  <td>The default configuration used for machine pools without a platform configuration.</td>
</tr>
<tr>
  <td><code>bootstrapOSImage</code></td>
  <td></td>
  <td>A URL to override the default operating system image for the bootstrap node. The URL must contain a SHA-256 hash of the image. For example: <code>https://mirror.openshift.com/rhcos-&lt;version&gt;-qemu.qcow2.gz?sha256=&lt;uncompressed_sha256&gt;</code>.</td>
</tr>
<tr>
  <td><code>provisioningNetwork</code></td>
  <td></td>
  <td>The <code>provisioningNetwork</code> parameter determines whether the cluster uses the provisioning network. If it does, the parameter also determines if the cluster manages the network.<br><br><code>Disabled</code>: Set this parameter to <code>Disabled</code> to disable the requirement for a provisioning network. When set to <code>Disabled</code>, you must only use virtual media based provisioning, or install the cluster by using the Assisted Installer. If <code>Disabled</code> and using power management, BMCs must be accessible from the bare metal network. If <code>Disabled</code>, you must provide two IP addresses on the bare metal network for the provisioning services to use.<br><br><code>Managed</code>: Set this parameter to <code>Managed</code>, which is the default, to fully manage the provisioning network, including DHCP, TFTP, and so on.<br><br><code>Unmanaged</code>: Set this parameter to <code>Unmanaged</code> to enable the provisioning network but take care of manual configuration of DHCP. Virtual media provisioning is recommended but PXE is still available if required.</td>
</tr>
<tr>
  <td><code>httpProxy</code></td>
  <td></td>
  <td>Set this parameter to the appropriate HTTP proxy used within your environment.</td>
</tr>
<tr>
  <td><code>httpsProxy</code></td>
  <td></td>
  <td>Set this parameter to the appropriate HTTPS proxy used within your environment.</td>
</tr>
<tr>
  <td><code>noProxy</code></td>
  <td></td>
  <td>Set this parameter to the appropriate list of exclusions for proxy usage within your environment.</td>
</tr>
</tbody>
</table>

### Hosts {#_hosts}

The `hosts` parameter is a list of separate bare metal assets used to build the cluster.

**Hosts**

<table>
<thead>
<tr>
  <th>Name</th>
  <th>Default</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>name</code></td>
  <td></td>
  <td>The name of the <code>BareMetalHost</code> resource to associate with the details. For example, <code>openshift-master-0</code>.</td>
</tr>
<tr>
  <td><code>role</code></td>
  <td></td>
  <td>The role of the bare metal node. Either <code>master</code> (control plane node) or <code>worker</code> (compute node).</td>
</tr>
<tr>
  <td><code>bmc</code></td>
  <td></td>
  <td>Connection details for the baseboard management controller. See the BMC addressing section for additional details.</td>
</tr>
<tr>
  <td><pre><code>bmc:&#10;    address:</code></pre></td>
  <td></td>
  <td>The protocol and address of the BMC as a URL.</td>
</tr>
<tr>
  <td><pre><code>bmc:&#10;    username:</code></pre></td>
  <td></td>
  <td>The username of the BMC.</td>
</tr>
<tr>
  <td><pre><code>bmc:&#10;    password:</code></pre></td>
  <td></td>
  <td>The password of the BMC.</td>
</tr>
<tr>
  <td><pre><code>bmc:&#10;    disableCertificateVerification:</code></pre></td>
  <td><code>False</code></td>
  <td><code>redfish</code> and <code>redfish-virtualmedia</code> need this parameter to manage BMC addresses. For OpenShift Container Platform 4.16 and earlier, the value should be <code>True</code> when using a self-signed certificate. OpenShift Container Platform supports self-signed certificates with certificate verification when used with the <code>bmcVerifyCA</code> parameter.</td>
</tr>
<tr>
  <td><pre><code>platform:&#10;  baremetal:&#10;    bmcVerifyCA:</code></pre></td>
  <td></td>
  <td>A local or self-signed CA certificate that the installation program will use to secure communication with the BMC. If you specify your own CA certificate, ensure that <code>disableCertificateVerification</code> is set to <code>False</code> so that the user-provided CA certificate is validated.</td>
</tr>
<tr>
  <td><code>bootMACAddress</code></td>
  <td></td>
  <td>The MAC address of the NIC that the host uses for the provisioning network. Ironic retrieves the IP address by using the <code>bootMACAddress</code> parameter. Then, it binds to the host.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>You must provide a valid MAC address from the host if you disabled the provisioning network.</p></div></td>
</tr>
<tr>
  <td><code>networkConfig</code></td>
  <td></td>
  <td>Set this optional parameter to configure the network interface of a host. See "(Optional) Configuring host network interfaces" for additional details.</td>
</tr>
</tbody>
</table>

## Root device hints {#root-device-hints_install-ibm-cloud-installation-workflow}

The `rootDeviceHints` parameter enables the installer to provision the Red Hat Enterprise Linux CoreOS (RHCOS) image to a particular device. The installer examines the devices in the order it discovers them, and compares the discovered values with the hint values. The installer uses the first discovered device that matches the hint value. The configuration can combine multiple hints, but a device must match all hints for the installer to select it.

**Subfields**

<table>
<thead>
<tr>
  <th>Subfield</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>deviceName</code></td>
  <td>A string containing a Linux device name such as <code>/dev/vda</code> or <code>/dev/disk/by-path/</code>.<div class="alert alert--secondary"><strong>Note</strong><p>It is recommended to use the <code>/dev/disk/by-path/&lt;device_path&gt;</code> link to the storage location.</p></div><br><br>The hint must match the actual value exactly.</td>
</tr>
<tr>
  <td><code>hctl</code></td>
  <td>A string containing a SCSI bus address like <code>0:0:0:0</code>. The hint must match the actual value exactly.</td>
</tr>
<tr>
  <td><code>model</code></td>
  <td>A string containing a vendor-specific device identifier. The hint can be a substring of the actual value.</td>
</tr>
<tr>
  <td><code>vendor</code></td>
  <td>A string containing the name of the vendor or manufacturer of the device. The hint can be a sub-string of the actual value.</td>
</tr>
<tr>
  <td><code>serialNumber</code></td>
  <td>A string containing the device serial number. The hint must match the actual value exactly.</td>
</tr>
<tr>
  <td><code>minSizeGigabytes</code></td>
  <td>An integer representing the minimum size of the device in gigabytes.</td>
</tr>
<tr>
  <td><code>wwn</code></td>
  <td>A string containing the unique storage identifier. The hint must match the actual value exactly.</td>
</tr>
<tr>
  <td><code>wwnWithExtension</code></td>
  <td>A string containing the unique storage identifier with the vendor extension appended. The hint must match the actual value exactly.</td>
</tr>
<tr>
  <td><code>wwnVendorExtension</code></td>
  <td>A string containing the unique vendor storage identifier. The hint must match the actual value exactly.</td>
</tr>
<tr>
  <td><code>rotational</code></td>
  <td>A boolean indicating whether the device should be a rotating disk (true) or not (false).</td>
</tr>
</tbody>
</table>

```yaml title="Example usage"
     - name: master-0
       role: master
       bmc:
         address: ipmi://10.10.0.3:6203
         username: admin
         password: redhat
       bootMACAddress: de:ad:be:ef:00:40
       rootDeviceHints:
         deviceName: "/dev/sda"
```

## Creating the OpenShift Container Platform manifests {#creating-the-openshift-manifests_install-ibm-cloud-installation-workflow}

Create manifest files to begin customizing your cluster installation.

**Procedure**

- Create the OpenShift Container Platform manifests by running the following command:
  ```terminal
  $ ./openshift-baremetal-install --dir ~/clusterconfigs create manifests
  ```

  ```terminal title="Example output"
  INFO Consuming Install Config from target directory
  WARNING Making control-plane schedulable by setting MastersSchedulable to true for Scheduler cluster settings
  WARNING Discarding the OpenShift Manifest that was provided in the target directory because its dependencies are dirty and it needs to be regenerated
  ```

## Deploying the cluster via the OpenShift Container Platform installer {#deploying-the-cluster-via-the-openshift-installer_install-ibm-cloud-installation-workflow}

You can deploy the cluster by running the OpenShift Container Platform installer.

**Procedure**

- Run the OpenShift Container Platform installer:
  ```terminal
  $ ./openshift-baremetal-install --dir ~/clusterconfigs --log-level debug create cluster
  ```

## Following the progress of the installation {#ipi-install-following-the-progress-of-the-installation_install-ibm-cloud-installation-workflow}

During the deployment process, you can check the installation’s overall status by issuing the `tail` command to the `.openshift_install.log` log file in the install directory folder.

**Procedure**

- Track installation progress by running the following command:
  ```terminal
  $ tail -f /path/to/install-dir/.openshift_install.log
  ```
