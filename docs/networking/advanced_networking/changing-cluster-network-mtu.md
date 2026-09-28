---
title: Changing the MTU for the cluster network
sidebar_position: 2
---

# Changing the MTU for the cluster network {#changing-cluster-network-mtu}

<a id="changing-cluster-network-mtu"></a>

As a cluster administrator, you can change the maximum transmission unit (MTU) for the cluster network after cluster installation. This change is disruptive as cluster nodes must be rebooted to finalize the MTU change.

## About the cluster MTU {#nw-cluster-mtu-change-about_changing-cluster-network-mtu}

During installation, the cluster network maximum transmission unit (MTU) is set automatically based on the primary network interface MTU of cluster nodes. Typically, you do not need to override the detected MTU, but in some instances you must override it.

You might want to change the MTU of the cluster network for one of the following reasons:

- The MTU detected during cluster installation is not correct for your infrastructure.
- Your cluster infrastructure now requires a different MTU, such as from the addition of nodes that need a different MTU for optimal performance.

Only the OVN-Kubernetes network plugin supports changing the MTU value.

### Service interruption considerations {#service-interruption-considerations_changing-cluster-network-mtu}

When you initiate a maximum transmission unit (MTU) change on your cluster the following effects might impact service availability:

- At least two rolling reboots are required to complete the migration to a new MTU. During this time, some nodes are not available as they restart.
- Specific applications deployed to the cluster with shorter timeout intervals than the absolute TCP timeout interval might experience disruption during the MTU change.

### MTU value selection {#mtu-value-selection_changing-cluster-network-mtu}

When planning your maximum transmission unit (MTU) migration there are two related but distinct MTU values to consider.

- **Hardware MTU**: This MTU value is set based on the specifics of your network infrastructure.
- **Cluster network MTU**: This MTU value is always less than your hardware MTU to account for the cluster network overlay overhead. The specific overhead is determined by your network plugin. For OVN-Kubernetes, the overhead is `100` bytes.

If your cluster requires different MTU values for different nodes, you must subtract the overhead value for your network plugin from the lowest MTU value that is used by any node in your cluster. For example, if some nodes in your cluster have an MTU of `9001`, and some have an MTU of `1500`, you must set this value to `1400`.

:::warning

To avoid selecting an MTU value that is not acceptable by a node, verify the maximum MTU value (`maxmtu`) that is accepted by the network interface by using the `ip -d link` command.

:::

### How the migration process works {#how-the-migration-process-works_changing-cluster-network-mtu}

The following table summarizes the migration process by segmenting between the user-initiated steps in the process and the actions that the migration performs in response.

**Live migration of the cluster MTU**

<table>
<thead>
<tr>
  <th>User-initiated steps</th>
  <th>OpenShift Container Platform activity</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Set the following values in the Cluster Network Operator configuration:<br><br><ul><li><code>spec.migration.mtu.machine.to</code></li><li><code>spec.migration.mtu.network.from</code></li><li><code>spec.migration.mtu.network.to</code></li></ul></td>
  <td><strong>Cluster Network Operator (CNO)</strong>: Confirms that each field is set to a valid value.<br><br><ul><li>The <code>mtu.machine.to</code> must be set to either the new hardware MTU or to the current hardware MTU if the MTU for the hardware is not changing. This value is transient and is used as part of the migration process. If you set a hardware MTU different from the current value, you must manually configure it to persist. Use methods such as a machine config, DHCP setting, or kernel command line.</li><li>The <code>mtu.network.from</code> field must equal the <code>network.status.clusterNetworkMTU</code> field, which is the current MTU of the cluster network.</li><li>The <code>mtu.network.to</code> field must be set to the target cluster network MTU. It must be lower than the hardware MTU to allow for the overlay overhead of the network plugin. For OVN-Kubernetes, the overhead is <code>100</code> bytes.</li></ul>If the values provided are valid, the CNO writes out a new temporary configuration with the MTU for the cluster network set to the value of the <code>mtu.network.to</code> field.<br><br><strong>Machine Config Operator (MCO)</strong>: Performs a rolling reboot of each node in the cluster.</td>
</tr>
<tr>
  <td>Reconfigure the MTU of the primary network interface for the nodes on the cluster. You can use one of the following methods to accomplish this:<br><br><ul><li>Deploying a new NetworkManager connection profile with the MTU change</li><li>Changing the MTU through a DHCP server setting</li><li>Changing the MTU through boot parameters</li></ul></td>
  <td>N/A</td>
</tr>
<tr>
  <td>Set the <code>mtu</code> value in the CNO configuration for the network plugin and set <code>spec.migration</code> to <code>null</code>.</td>
  <td><strong>Machine Config Operator (MCO)</strong>: Performs a rolling reboot of each node in the cluster with the new MTU configuration.</td>
</tr>
</tbody>
</table>

## Changing the cluster network MTU {#nw-cluster-mtu-change_changing-cluster-network-mtu}

As a cluster administrator, you can increase or decrease the maximum transmission unit (MTU) for your cluster.

:::warning

You cannot roll back an MTU value for nodes during the MTU migration process, but you can roll back the value after the MTU migration process completes.

The migration is disruptive and nodes in your cluster might be temporarily unavailable as the MTU update takes effect.

:::

The following procedures describe how to change the cluster network MTU by using machine configs, Dynamic Host Configuration Protocol (DHCP), or an ISO image. If you use either the DHCP or ISO approaches, you must refer to configuration artifacts that you kept after installing your cluster to complete the procedure.

## Prerequisites for changing the cluster network MTU {#nw-cluster-mtu-change-prerequisites_changing-cluster-network-mtu}

Before you change the cluster network maximum transmission unit (MTU), verify that you have the required access, tools, and network infrastructure to support the new MTU value.

Ensure that the following conditions are met before you begin:

- You have installed the OpenShift CLI (`oc`).
- You have access to the cluster using an account with `cluster-admin` permissions.
- You have identified the target MTU for your cluster. The MTU for the OVN-Kubernetes network plugin must be set to `100` less than the lowest hardware MTU value in your cluster.
- If your nodes are physical machines, ensure that the cluster network and the connected network switches support jumbo frames.
- If your nodes are virtual machines (VMs), ensure that the hypervisor and the connected network switches support jumbo frames.

### Check the current cluster MTU value {#nw-cluster-mtu-checking_changing-cluster-network-mtu}

To ensure network stability and performance in a hybrid environment where part of your cluster is in the cloud and part is an on-premise environment, you can obtain the current maximum transmission unit (MTU) for the cluster network.

**Procedure**

- To obtain the current MTU for the cluster network, enter the following command:
  ```terminal
  $ oc describe network.config cluster
  ```

  ```text title="Example output"
  ...
  Status:
    Cluster Network:
      Cidr:               10.217.0.0/22
      Host Prefix:        23
    Cluster Network MTU:  1400
    Network Type:         OVNKubernetes
    Service Network:
      10.217.4.0/23
  ...
  ```

### Prepare your hardware MTU configuration {#nw-cluster-mtu-preparing_changing-cluster-network-mtu}

To maintain network stability during an MTU change, you must prepare the configuration for your underlying hardware using a method such as DHCP, PXE, or NetworkManager. This preparation ensures that all cluster nodes are ready to accept the new MTU value before you apply the changes to the cluster network.

**Procedure**

- Prepare your configuration for the hardware MTU:
  - If your hardware MTU is specified with DHCP, update your DHCP configuration such as with the following dnsmasq configuration:
    ```text
    dhcp-option-force=26,<mtu>
    ```

    where:

    <dl>
    <dt><code>&lt;mtu&gt;</code></dt>
    <dd>Specifies the hardware MTU for the DHCP server to advertise.</dd>
    </dl>
  - If your hardware MTU is specified with a kernel command line with PXE, update that configuration accordingly.
  - If your hardware MTU is specified in a NetworkManager connection configuration, complete the following steps. This approach is the default for OpenShift Container Platform if you do not explicitly specify your network configuration with DHCP, a kernel command line, or some other method. Your cluster nodes must all use the same underlying network configuration for the following procedure to work unmodified.
    Find the primary network interface by entering the following command:

    ```terminal
    $ oc debug node/<node_name> -- chroot /host nmcli -g connection.interface-name c show ovs-if-phys0
    ```

    where:

    <dl>
    <dt><code>&lt;node_name&gt;</code></dt>
    <dd>Specifies the name of a node in your cluster.</dd>
    </dl>

    Create the following `NetworkManager` configuration in the `<interface>-mtu.conf` file:

    ```ini
    [connection-<interface>-mtu]
    match-device=interface-name:<interface>
    ethernet.mtu=<mtu>
    ```

    where:

    **`<interface>`**

    Specifies the primary network interface name or the bond if using bonding.

    :::warning

    Observe the following constraints when specifying the interface:

    - Do not specify subordinate interfaces of a bond, such as `eth0` or `eth1`. Subordinate interfaces automatically inherit the MTU value set on the parent bond.
    - The MTU of the VLAN interface cannot exceed the MTU of its parent physical or bonded interface. A parent interface with a larger MTU, such as `9000` for jumbo frames, can host VLANs with smaller MTUs, such as `1500`. However, raising the MTU of a VLAN interface beyond the MTU parent interface current limit requires updating the parent interface first.

    :::

    **`<mtu>`**

    Specifies the new hardware MTU value.

    If you used Kubernetes NMState to configure the `br-ex` bridge, use the Kubernetes NMState Operator to update the MTU for the `br-ex` bridge. Changing the MTU for this bridge in a `.nmconnection` file could lead to persistence issues as the Machine Config Operator (MCO) might overwrite the file.

### Create MachineConfig objects {#nw-cluster-mtu-creating-mc-objects_changing-cluster-network-mtu}

To prepare your nodes for a hardware MTU change, you must create `MachineConfig` objects for both control plane and compute nodes. Creating these objects ensures that the updated network interface settings are ready for deployment without causing immediate cluster instability.

**Procedure**

1. Create two `MachineConfig` objects, one for the control plane nodes and another for the worker nodes in your cluster:
   1. Create the following Butane config in the `control-plane-interface.bu` file:
      :::note

      The [Butane version](https://coreos.github.io/butane/specs/) you specify in the config file should match the OpenShift Container Platform version and always ends in `0`. For example, `4.22.0`. See "Creating machine configs with Butane" for information about Butane.

      :::

      ```yaml
      variant: openshift
      version: 4.22.0
      metadata:
        name: 01-control-plane-interface
        labels:
          machineconfiguration.openshift.io/role: master
      storage:
        files:
          - path: /etc/NetworkManager/conf.d/99-<interface>-mtu.conf
            contents:
              local: <interface>-mtu.conf
            mode: 0600
      ```

      where:

      <dl>
      <dt><code>storage.files.path</code></dt>
      <dd>Specifies the <code>NetworkManager</code> connection name for the primary network interface.</dd>
      <dt><code>storage.files.local</code></dt>
      <dd>Specifies the local filename for the updated <code>NetworkManager</code> configuration file from an earlier step.</dd>
      </dl>
   2. Create the following Butane config in the `worker-interface.bu` file:
      :::note

      The [Butane version](https://coreos.github.io/butane/specs/) you specify in the config file should match the OpenShift Container Platform version and always ends in `0`. For example, `4.22.0`. See "Creating machine configs with Butane" for information about Butane.

      :::

      ```yaml
      variant: openshift
      version: 4.22.0
      metadata:
        name: 01-worker-interface
        labels:
          machineconfiguration.openshift.io/role: worker
      storage:
        files:
          - path: /etc/NetworkManager/conf.d/99-<interface>-mtu.conf
            contents:
              local: <interface>-mtu.conf
            mode: 0600
      ```

      where:

      <dl>
      <dt><code>storage.files.path</code></dt>
      <dd>Specifies the <code>NetworkManager</code> connection name for the primary network interface.</dd>
      <dt><code>storage.files.local</code></dt>
      <dd>Specifies the local filename for the updated <code>NetworkManager</code> configuration file from an earlier step.</dd>
      </dl>
2. Create `MachineConfig` objects from the Butane configs by running the following command:
   ```terminal
   $ for manifest in control-plane-interface worker-interface; do
       butane --files-dir . $manifest.bu > $manifest.yaml
     done
   ```

   :::warning

   Do not apply these machine configs until explicitly instructed later in this procedure. Applying these machine configs now causes a loss of stability for the cluster.

   :::

### Begin the MTU migration {#nw-cluster-mtu-migration_changing-cluster-network-mtu}

Start the maximum transmission unit (MTU) migration by specifying the migration configuration for the cluster network and machine interfaces. The Machine Config Operator performs a rolling reboot of the nodes to prepare the cluster for the MTU change.

**Procedure**

1. To begin the MTU migration, specify the migration configuration by entering the following command. The Machine Config Operator performs a rolling reboot of the nodes in the cluster in preparation for the MTU change.
   ```terminal
   $ oc patch Network.operator.openshift.io cluster --type=merge --patch \
     '{"spec": { "migration": { "mtu": { "network": { "from": <overlay_from>, "to": <overlay_to> } , "machine": { "to" : <machine_to> } } } } }'
   ```

   where:

   <dl>
   <dt><code>&lt;overlay_from&gt;</code></dt>
   <dd>Specifies the current cluster network MTU value.</dd>
   <dt><code>&lt;overlay_to&gt;</code></dt>
   <dd>Specifies the target MTU for the cluster network. This value is set relative to the value of <code>&lt;machine_to&gt;</code>. For OVN-Kubernetes, this value must be <code>100</code> less than the value of <code>&lt;machine_to&gt;</code>.</dd>
   <dt><code>&lt;machine_to&gt;</code></dt>
   <dd>Specifies the MTU for the primary network interface on the underlying host network.</dd>
   </dl>

   ```terminal
   $ oc patch Network.operator.openshift.io cluster --type=merge --patch \
     '{"spec": { "migration": { "mtu": { "network": { "from": 1400, "to": 9000 } , "machine": { "to" : 9100} } } } }'
   ```
2. As the Machine Config Operator updates machines in each machine config pool, the Operator reboots each node one by one. You must wait until all the nodes are updated. Check the machine config pool status by entering the following command:
   ```terminal
   $ oc get machineconfigpools
   ```

   A successfully updated node has the following status: `UPDATED=true`, `UPDATING=false`, `DEGRADED=false`.

   :::note

   By default, the Machine Config Operator updates one machine per pool at a time, causing the total time the migration takes to increase with the size of the cluster.

   :::

### Verify the machine configuration {#nw-cluster-mtu-verifying-configuration_changing-cluster-network-mtu}

Verify the machine configuration on your hosts to confirm that the maximum transmission unit (MTU) migration applied successfully. Checking the configuration state and system settings help ensures that the nodes use the correct migration script.

**Procedure**

- Confirm the status of the new machine configuration on the hosts:
  1. To list the machine configuration state and the name of the applied machine configuration, enter the following command:
     ```terminal
     $ oc describe node | egrep "hostname|machineconfig"
     ```

     ```text title="Example output"
     kubernetes.io/hostname=master-0
     machineconfiguration.openshift.io/currentConfig: rendered-master-c53e221d9d24e1c8bb6ee89dd3d8ad7b
     machineconfiguration.openshift.io/desiredConfig: rendered-master-c53e221d9d24e1c8bb6ee89dd3d8ad7b
     machineconfiguration.openshift.io/reason:
     machineconfiguration.openshift.io/state: Done
     ```
  2. Verify that the following statements are true:
     - The value of `machineconfiguration.openshift.io/state` field is `Done`.
     - The value of the `machineconfiguration.openshift.io/currentConfig` field is equal to the value of the `machineconfiguration.openshift.io/desiredConfig` field.
  3. To confirm that the machine config is correct, enter the following command:
     ```terminal
     $ oc get machineconfig <config_name> -o yaml | grep ExecStart
     ```

     where:

     <dl>
     <dt><code>&lt;config_name&gt;</code></dt>
     <dd>Specifies the name of the machine config from the <code>machineconfiguration.openshift.io/currentConfig</code> field.</dd>
     </dl>

     The machine config must include the following update to the systemd configuration:

     ```plain
     ExecStart=/usr/local/bin/mtu-migration.sh
     ```

### Apply the new hardware MTU value {#nw-cluster-mtu-applying-mtu-value_changing-cluster-network-mtu}

To ensure consistent network communication across your cluster, you must apply the new hardware maximum transmission unit (MTU) value to your nodes. This process involves updating the underlying network interfaces and verifying that the Machine Config Operator successfully reboots and updates each node.

**Procedure**

1. Update the underlying network interface MTU value:
   - If you are specifying the new MTU with a NetworkManager connection configuration, enter the following command. The MachineConfig Operator automatically performs a rolling reboot of the nodes in your cluster.
     ```terminal
     $ for manifest in control-plane-interface worker-interface; do
         oc create -f $manifest.yaml
       done
     ```
   - If you are specifying the new MTU with a DHCP server option or a kernel command line and PXE, make the necessary changes for your infrastructure.
2. As the Machine Config Operator updates machines in each machine config pool, the Operator reboots each node one by one. You must wait until all the nodes are updated. Check the machine config pool status by entering the following command:
   ```terminal
   $ oc get machineconfigpools
   ```

   A successfully updated node has the following status: `UPDATED=true`, `UPDATING=false`, `DEGRADED=false`.

   :::note

   By default, the Machine Config Operator updates one machine per pool at a time, causing the total time the migration takes to increase with the size of the cluster.

   :::
3. Confirm the status of the new machine configuration on the hosts:
   1. To list the machine configuration state and the name of the applied machine configuration, enter the following command:
      ```terminal
      $ oc describe node | egrep "hostname|machineconfig"
      ```

      ```text title="Example output"
      kubernetes.io/hostname=master-0
      machineconfiguration.openshift.io/currentConfig: rendered-master-c53e221d9d24e1c8bb6ee89dd3d8ad7b
      machineconfiguration.openshift.io/desiredConfig: rendered-master-c53e221d9d24e1c8bb6ee89dd3d8ad7b
      machineconfiguration.openshift.io/reason:
      machineconfiguration.openshift.io/state: Done
      ```

      Verify that the following statements are true:

      - The value of `machineconfiguration.openshift.io/state` field is `Done`.
      - The value of the `machineconfiguration.openshift.io/currentConfig` field is equal to the value of the `machineconfiguration.openshift.io/desiredConfig` field.
   2. To confirm that the machine config is correct, enter the following command:
      ```terminal
      $ oc get machineconfig <config_name> -o yaml | grep path:
      ```

      where:

      <dl>
      <dt><code>&lt;config_name&gt;</code></dt>
      <dd>Specifies the name of the machine config from the <code>machineconfiguration.openshift.io/currentConfig</code> field.</dd>
      </dl>

      If the machine config is successfully deployed, the previous output contains the `/etc/NetworkManager/conf.d/99-<interface>-mtu.conf` file path and the `ExecStart=/usr/local/bin/mtu-migration.sh` line.

### Finalize the MTU migration {#nw-cluster-mtu-finalizing-migration_changing-cluster-network-mtu}

Finalize the MTU migration to apply the new maximum transmission unit (MTU) settings to the OVN-Kubernetes network plugin. This updates the cluster configuration and triggers a rolling reboot of the nodes to complete the process.

**Procedure**

1. To finalize the MTU migration, enter the following command for the OVN-Kubernetes network plugin:
   ```terminal
   $ oc patch Network.operator.openshift.io cluster --type=merge --patch \
     '{"spec": { "migration": null, "defaultNetwork":{ "ovnKubernetesConfig": { "mtu": <mtu> }}}}'
   ```

   where:

   <dl>
   <dt><code>&lt;mtu&gt;</code></dt>
   <dd>Specifies the new cluster network MTU that you specified with <code>&lt;overlay_to&gt;</code>.</dd>
   </dl>
2. After finalizing the MTU migration, each machine config pool node is rebooted one by one. You must wait until all the nodes are updated. Check the machine config pool status by entering the following command:
   ```terminal
   $ oc get machineconfigpools
   ```

   A successfully updated node has the following status: `UPDATED=true`, `UPDATING=false`, `DEGRADED=false`.

**Verification**

1. To get the current MTU for the cluster network, enter the following command:
   ```terminal
   $ oc describe network.config cluster
   ```
2. Get the current MTU for the primary network interface of a node:
   1. To list the nodes in your cluster, enter the following command:
      ```terminal
      $ oc get nodes
      ```
   2. To obtain the current MTU setting for the primary network interface on a node, enter the following command:
      ```terminal
      $ oc adm node-logs <node> -u ovs-configuration | grep configure-ovs.sh | grep mtu | grep <interface> | head -1
      ```

      where:

      <dl>
      <dt><code>&lt;node&gt;</code></dt>
      <dd>Specifies a node from the output from the previous step.</dd>
      <dt><code>&lt;interface&gt;</code></dt>
      <dd>Specifies the primary network interface name for the node.</dd>
      </dl>

      ```text title="Example output"
      ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 8051
      ```

**Additional resources**

- [Using advanced networking options for PXE and ISO installations](/docs/installing/installing_bare_metal/upi/installing-bare-metal#installation-user-infra-machines-advanced_network_installing-bare-metal)
- [Manually creating NetworkManager profiles in key file format](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html-single/configuring_and_managing_networking/index#proc_manually-creating-a-networkmanager-profile-in-keyfile-format_assembly_networkmanager-connection-profiles-in-keyfile-format)
- [Configuring a dynamic Ethernet connection using nmcli](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html-single/configuring_and_managing_networking/index#configuring-a-dynamic-ethernet-connection-using-nmcli_configuring-an-ethernet-connection)
