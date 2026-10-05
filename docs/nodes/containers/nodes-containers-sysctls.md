---
title: Using sysctls in containers
sidebar_position: 9
---

# Using sysctls in containers {#nodes-containers-sysctls}

<a id="nodes-containers-sysctls"></a>

You can configure namespaced sysctls to manage kernel parameters for specific pods or network interfaces. By enabling safe or unsafe namespaced sysctls, you can fine-tune your environment’s performance and networking behavior at the pod level.

Only sysctls that are namespaced can be set independently on pods. If a sysctl is not namespaced, it is known as a *node-level* sysctl. You must use another method of setting the sysctl, such as by using the Node Tuning Operator. To set node-level sysctls, see "Using the Node Tuning Operator".

Network sysctls are a special category of sysctl. Network sysctls include:

- System-wide sysctls, for example `net.ipv4.ip_local_port_range`, that are valid for all networking. You can set these independently for each pod on a node.
- Interface-specific sysctls, for example `net.ipv4.conf.IFNAME.accept_local`, that only apply to a specific additional network interface for a given pod. You can set these independently for each additional network configuration. You set these by using a configuration in the `tuning-cni` after the network interfaces are created.

:::warning

If the `net.ipv4.ip_local_port_range` safe sysctl parameter value and the default node port service range overlap, the OVN Kubernetes plugin might experience connection failures. For more information about this parameter, see the *System-wide safe sysctls* table in the "Safe and unsafe sysctls" section.

:::

Only those sysctls considered *safe* are enabled by default. A cluster administrator can manually enable *unsafe* sysctls on the node to be available to the user.

**Additional resources**

- [Configuring the node port service range](/docs/networking/configuring_network_settings/configuring-node-port-service-range#configuring-node-port-service-range)

## About sysctls {#nodes-containers-sysctls-about_nodes-containers-using}

The Linux sysctl interface allows you to modify kernel parameters at runtime to manage subsystems such as networking, virtual memory, and MDADM. By accessing the sysctl interface, you can view and adjust system configurations without rebooting the operating system.

You can modify the following subsystems by using sysctls:

- kernel (common prefix: `_kernel._`)
- networking (common prefix: `_net._`)
- virtual memory (common prefix: `_vm._`)
- MDADM (common prefix: `_dev._`)

For more information on the subsystems you can manage, see "Kernel.org documentation".

You can get a list of all parameters by running the following command:

```terminal
$ sudo sysctl -a
```

**Additional resources**

- [Kernel.org documentation](https://www.kernel.org/doc/Documentation/sysctl/README)

## Namespaced and node-level sysctls

In a OpenShift Container Platform cluster, you can use namespaced sysctls, which apply to specific pods on a node, or node-level sysctls, which affect an entire node.

Some sysctls are *namespaced* in the Linux kernels. This means that you can set them independently for each pod on a node. Being namespaced is a requirement for sysctls to be accessible in a pod context within Kubernetes.

The following sysctls are known to be namespaced:

- `_kernel.shm*_`
- `_kernel.msg*_`
- `_kernel.sem_`
- `_fs.mqueue.*_`

Additionally, most of the sysctls in the `net.*` group are known to be namespaced. Their namespace adoption differs based on the kernel version and distributor.

*Node-level* sysctls are not namespaced and must be set manually by a cluster administrator, either by using of the underlying Linux distribution of the nodes, such as by modifying the `_/etc/sysctls.conf_` file, or by using a daemon set with privileged containers. You can also use the Node Tuning Operator to set *node-level* sysctls.

:::note

Consider marking nodes with special sysctls as tainted. Only schedule pods onto them that need those sysctl settings. Use the taints and toleration feature to mark the nodes.

:::

## Safe and unsafe sysctls {#safe_and_unsafe_sysctls_nodes-containers-using}

In a OpenShift Container Platform cluster, you can use *safe* or *unsafe* sysctls.

For system-wide sysctls to be considered safe, they must be namespaced. A namespaced sysctl ensures there is isolation between namespaces and therefore pods. If you set a sysctl for one pod it must not take any of the following actions:

- Influence on any other pod on the node
- Harm the node health
- Gain CPU or memory resources outside of the resource limits of a pod

:::note

Being namespaced alone is not sufficient for the sysctl to be considered safe.

:::

Any sysctl that is not added to the allowed list on OpenShift Container Platform is considered unsafe for OpenShift Container Platform.

Unsafe sysctls are not allowed by default. For system-wide sysctls, a cluster administrator must manually enable them on a per-node basis. Pods with disabled unsafe sysctls are scheduled but do not launch.

:::note

You cannot manually enable interface-specific unsafe sysctls.

:::

OpenShift Container Platform adds the following system-wide and interface-specific safe sysctls to an allowed safe list:

**System-wide safe sysctls**

<table>
<thead>
<tr>
  <th>sysctl</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>kernel.shm_rmid_forced</code></td>
  <td>When set to <code>1</code>, all shared memory objects in current IPC namespace are automatically forced to use IPC_RMID. For more information, see <a href="https://docs.kernel.org/admin-guide/sysctl/kernel.html?highlight=shm_rmid_forced#shm-rmid-forced">shm_rmid_forced</a>.</td>
</tr>
<tr>
  <td><code>net.ipv4.ip_local_port_range</code></td>
  <td>Defines the local port range that is used by TCP and UDP to choose the local port. The first number is the first port number, and the second number is the last local port number. If possible, ensure these numbers have different parity, such as one even and one odd value. The numbers must be greater than or equal to <code>ip_unprivileged_port_start</code>. The default values are <code>32768</code> and <code>60999</code> respectively. For more information, see <a href="https://docs.kernel.org/networking/ip-sysctl.html?highlight=ip_local_port_range#ip-variables">ip_local_port_range (Kernel.org documentation)</a>.<br><br><div class="alert alert--warning"><strong>Important</strong><p>When specifying a range for the <code>net.ipv4.ip_local_port_range</code> sysctl parameter, ensure the range does not overlap with the range you set for the <code>serviceNodePortRange</code> parameter. For more information, see "Configuring the node port service range".</p></div></td>
</tr>
<tr>
  <td><code>net.ipv4.tcp_syncookies</code></td>
  <td>When <code>net.ipv4.tcp_syncookies</code> is set, the kernel handles TCP SYN packets normally until the half-open connection queue is full, at which time, the SYN cookie functionality kicks in. This functionality allows the system to keep accepting valid connections, even if under a denial-of-service attack. For more information, see <a href="https://docs.kernel.org/networking/ip-sysctl.html?highlight=tcp_syncookies#tcp-variables">tcp_syncookies (Kernel.org documentation)</a>.</td>
</tr>
<tr>
  <td><code>net.ipv4.ping_group_range</code></td>
  <td>Restricts <code>ICMP_PROTO</code> datagram sockets to users in the group range. The default is <code>1 0</code>, meaning that nobody, not even root, can create ping sockets. For more information, see <a href="https://docs.kernel.org/networking/ip-sysctl.html?highlight=ping_group_range#ip-variables">ping_group_range (Kernel.org documentation)</a>.</td>
</tr>
<tr>
  <td><code>net.ipv4.ip_unprivileged_port_start</code></td>
  <td>Defines the first unprivileged port in the network namespace. To disable all privileged ports, set to <code>0</code>. Privileged ports must not overlap with the <code>ip_local_port_range</code>. For more information, see <a href="https://docs.kernel.org/networking/ip-sysctl.html?highlight=ip_unprivileged_port_start#ip-variables#ip-variables">ip_unprivileged_port_start (Kernel.org documentation)</a>.</td>
</tr>
<tr>
  <td><code>net.ipv4.ip_local_reserved_ports</code></td>
  <td>Specifies a range of comma-separated local ports that you want to reserve for applications or services.</td>
</tr>
<tr>
  <td><code>net.ipv4.tcp_keepalive_time</code></td>
  <td>Specifies the interval in seconds before the first <code>keepalive</code> probe should be sent after a connection has become idle.</td>
</tr>
<tr>
  <td><code>net.ipv4.tcp_fin_timeout</code></td>
  <td>Specifies the time in seconds that a connection remains in the <code>FIN-WAIT-2</code> state before it is aborted.</td>
</tr>
<tr>
  <td><code>net.ipv4.tcp_keepalive_intvl</code></td>
  <td>Specifies the interval in seconds between the <code>keepalive</code> probes. This value is multiplied by the <code>tcp_keepalive_probes</code> value to determine the total time required before it is decided that the connection is broken.</td>
</tr>
<tr>
  <td><code>net.ipv4.tcp_keepalive_probes</code></td>
  <td>Specifies how many <code>keepalive</code> probes to send until it is determined that the connection is broken.</td>
</tr>
</tbody>
</table>

**Interface-specific safe sysctls**

<table>
<thead>
<tr>
  <th>sysctl</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>net.ipv4.conf.IFNAME.accept_redirects</code></td>
  <td>Accepts IPv4 ICMP redirect messages.</td>
</tr>
<tr>
  <td><code>net.ipv4.conf.IFNAME.accept_source_route</code></td>
  <td>Accepts IPv4 packets with strict source route (SRR) option.</td>
</tr>
<tr>
  <td><code>net.ipv4.conf.IFNAME.arp_accept</code></td>
  <td>Defines the behavior for gratuitous ARP frames with an IPv4 address that is not already present in the ARP table:<br><br><ul><li><code>0</code> - Do not create new entries in the ARP table.</li><li><code>1</code> - Create new entries in the ARP table.</li></ul></td>
</tr>
<tr>
  <td><code>net.ipv4.conf.IFNAME.arp_notify</code></td>
  <td>Defines the mode for notification of IPv4 address and device changes.</td>
</tr>
<tr>
  <td><code>net.ipv4.conf.IFNAME.disable_policy</code></td>
  <td>Disables IPSEC policy (SPD) for this IPv4 interface.</td>
</tr>
<tr>
  <td><code>net.ipv4.conf.IFNAME.secure_redirects</code></td>
  <td>Accepts ICMP redirect messages only to gateways listed in the interface’s current gateway list.</td>
</tr>
<tr>
  <td><code>net.ipv4.conf.IFNAME.send_redirects</code></td>
  <td>Sends redirects only if the node acts as a router. That is, a host should not send an ICMP redirect message. It is used by routers to notify the host about a better routing path that is available for a particular destination.</td>
</tr>
<tr>
  <td><code>net.ipv6.conf.IFNAME.accept_ra</code></td>
  <td>Accepts IPv6 Router advertisements; autoconfigure using them. It also determines whether or not to transmit router solicitations. Router solicitations are transmitted only if the functional setting is to accept router advertisements.</td>
</tr>
<tr>
  <td><code>net.ipv6.conf.IFNAME.accept_redirects</code></td>
  <td>Accepts IPv6 ICMP redirect messages.</td>
</tr>
<tr>
  <td><code>net.ipv6.conf.IFNAME.accept_source_route</code></td>
  <td>Accepts IPv6 packets with SRR option.</td>
</tr>
<tr>
  <td><code>net.ipv6.conf.IFNAME.arp_accept</code></td>
  <td>Defines the behavior for gratuitous ARP frames with an IPv6 address that is not already present in the ARP table:<br><br><ul><li><code>0</code> - Do not create new entries in the ARP table.</li><li><code>1</code> - Create new entries in the ARP table.</li></ul></td>
</tr>
<tr>
  <td><code>net.ipv6.conf.IFNAME.arp_notify</code></td>
  <td>Defines the mode for notification of IPv6 address and device changes.</td>
</tr>
<tr>
  <td><code>net.ipv6.neigh.IFNAME.base_reachable_time_ms</code></td>
  <td>Controls the hardware address to IP mapping lifetime in the neighbor table for IPv6.</td>
</tr>
<tr>
  <td><code>net.ipv6.neigh.IFNAME.retrans_time_ms</code></td>
  <td>Sets the retransmit timer for neighbor discovery messages.</td>
</tr>
</tbody>
</table>

:::note

When setting these values using the `tuning` CNI plugin, use the value `IFNAME` literally. The interface name is represented by the `IFNAME` token, and is replaced with the actual name of the interface at runtime.

:::

**Additional resources**

- [Configuring ingress cluster traffic using a NodePort](/docs/networking/ingress_load_balancing/configuring_ingress_cluster_traffic/configuring-ingress-cluster-traffic-nodeport#configuring-ingress-cluster-traffic-nodeport)

## Update the interface-specific safe sysctls list {#updating-interface-specific-safe-sysctls-list_nodes-containers-using}

You can modify the default list of safe interface-specific `sysctls` by updating the `cni-sysctl-allowlist` in the `openshift-multus` namespace.

:::warning

The support for updating the interface-specific safe sysctls list is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.

For more information about the support scope of Red Hat Technology Preview features, see [Technology Preview Features Support Scope](https://access.redhat.com/support/offerings/techpreview/).

:::

For example, the following procedure modifies the predefined list of safe `sysctls` to add sysctls that allow users to enforce stricter reverse path forwarding for IPv4. For more information on reverse path forwarding see Reverse Path Forwarding.

**Procedure**

1. View the existing predefined list by running the following command:
   ```terminal
   $ oc get cm -n openshift-multus cni-sysctl-allowlist -oyaml
   ```

   ```terminal title="Expected output"
   apiVersion: v1
   data:
     allowlist.conf: |-
       ^net.ipv4.conf.IFNAME.accept_redirects$
       ^net.ipv4.conf.IFNAME.accept_source_route$
       ^net.ipv4.conf.IFNAME.arp_accept$
       ^net.ipv4.conf.IFNAME.arp_notify$
       ^net.ipv4.conf.IFNAME.disable_policy$
       ^net.ipv4.conf.IFNAME.secure_redirects$
       ^net.ipv4.conf.IFNAME.send_redirects$
       ^net.ipv6.conf.IFNAME.accept_ra$
       ^net.ipv6.conf.IFNAME.accept_redirects$
       ^net.ipv6.conf.IFNAME.accept_source_route$
       ^net.ipv6.conf.IFNAME.arp_accept$
       ^net.ipv6.conf.IFNAME.arp_notify$
       ^net.ipv6.neigh.IFNAME.base_reachable_time_ms$
       ^net.ipv6.neigh.IFNAME.retrans_time_ms$
   kind: ConfigMap
   metadata:
     annotations:
       kubernetes.io/description: |
         Sysctl allowlist for nodes.
       release.openshift.io/version: 4.22.0-0.nightly-2022-11-16-003434
     creationTimestamp: "2022-11-17T14:09:27Z"
     name: cni-sysctl-allowlist
     namespace: openshift-multus
     resourceVersion: "2422"
     uid: 96d138a3-160e-4943-90ff-6108fa7c50c3
   ```
2. Edit the list by using the following command:
   ```terminal
   $ oc edit cm -n openshift-multus cni-sysctl-allowlist -oyaml
   ```
3. Add the `^net.ipv4.conf.IFNAME.rp_filter$` and `^net.ipv6.conf.IFNAME.rp_filter$` fields to the list of parameters to allow users to implement stricter reverse path forwarding.
   ```terminal
   # Please edit the object below. Lines beginning with a '#' will be ignored,
   # and an empty file will abort the edit. If an error occurs while saving this file will be
   # reopened with the relevant failures.
   #
   apiVersion: v1
   data:
     allowlist.conf: |-
       ^net.ipv4.conf.IFNAME.accept_redirects$
       ^net.ipv4.conf.IFNAME.accept_source_route$
       ^net.ipv4.conf.IFNAME.arp_accept$
       ^net.ipv4.conf.IFNAME.arp_notify$
       ^net.ipv4.conf.IFNAME.disable_policy$
       ^net.ipv4.conf.IFNAME.secure_redirects$
       ^net.ipv4.conf.IFNAME.send_redirects$
       ^net.ipv4.conf.IFNAME.rp_filter$
       ^net.ipv6.conf.IFNAME.accept_ra$
       ^net.ipv6.conf.IFNAME.accept_redirects$
       ^net.ipv6.conf.IFNAME.accept_source_route$
       ^net.ipv6.conf.IFNAME.arp_accept$
       ^net.ipv6.conf.IFNAME.arp_notify$
       ^net.ipv6.neigh.IFNAME.base_reachable_time_ms$
       ^net.ipv6.neigh.IFNAME.retrans_time_ms$
       ^net.ipv6.conf.IFNAME.rp_filter$
   ```
4. Save the changes to the file and exit.
   :::note

   The removal of `sysctls` is also supported. Edit the file, remove the `sysctl` or `sysctls` then save the changes and exit.

   :::

**Verification**

1. Create a network attachment definition, such as `reverse-path-fwd-example.yaml`, with the following content:
   ```yaml
   apiVersion: "k8s.cni.cncf.io/v1"
   kind: NetworkAttachmentDefinition
   metadata:
     name: tuningnad
     namespace: default
   spec:
     config: '{
       "cniVersion": "0.4.0",
       "name": "tuningnad",
       "plugins": [{
         "type": "bridge"
         },
         {
         "type": "tuning",
         "sysctl": {
            "net.ipv4.conf.IFNAME.rp_filter": "1"
           }
       }
     ]
   }'
   ```
2. Apply the yaml by running the following command:
   ```terminal
   $ oc apply -f reverse-path-fwd-example.yaml
   ```

   ```terminal title="Example output"
   networkattachmentdefinition.k8.cni.cncf.io/tuningnad created
   ```
3. Create a pod such as `examplepod.yaml` using the following YAML:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: example
     labels:
       app: httpd
     namespace: default
     annotations:
       k8s.v1.cni.cncf.io/networks: tuningnad
   spec:
     securityContext:
       runAsNonRoot: true
       seccompProfile:
         type: RuntimeDefault
     containers:
       - name: httpd
         image: 'image-registry.openshift-image-registry.svc:5000/openshift/httpd:latest'
         ports:
           - containerPort: 8080
         securityContext:
           allowPrivilegeEscalation: false
           capabilities:
             drop:
               - ALL
   ```

   where:

   <dl>
   <dt><code>metadata.annotations</code></dt>
   <dd>Specifies the name of the configured <code>NetworkAttachmentDefinition</code>.</dd>
   </dl>
4. Apply the YAML by running the following command:
   ```terminal
   $ oc apply -f examplepod.yaml
   ```
5. Verify that the pod is created by running the following command:
   ```terminal
   $ oc get pod
   ```

   ```terminal title="Example output"
   NAME      READY   STATUS    RESTARTS   AGE
   example   1/1     Running   0          47s
   ```
6. Log in to the pod by running the following command:
   ```terminal
   $ oc rsh example
   ```
7. Verify the value of the configured sysctl flag. For example, find the value `net.ipv4.conf.net1.rp_filter` by running the following command:
   ```terminal
   sh-4.4# sysctl net.ipv4.conf.net1.rp_filter
   ```

   ```terminal title="Expected output"
   net.ipv4.conf.net1.rp_filter = 1
   ```

## Start a pod with safe sysctls {#nodes-starting-pod-safe-sysctls_nodes-containers-using}

You can modify kernel parameters for all containers in a pod by adding the sysctls parameter to the `securityContext` parameter in a pod spec.

Safe sysctls are allowed by default.

This example uses the pod `securityContext` to set the following safe sysctls:

- `kernel.shm_rmid_forced`
- `net.ipv4.ip_local_port_range`
- `net.ipv4.tcp_syncookies`
- `net.ipv4.ping_group_range`

:::warning

To avoid destabilizing your operating system, modify sysctl parameters only after you understand their effects.

:::

The following procedure shows how to start a pod with the configured sysctl settings.

:::note

In most cases you modify an existing pod definition and add the `securityContext` spec.

:::

**Procedure**

1. Create a YAML file `sysctl_pod.yaml` that defines an example pod and add the `securityContext` spec, as shown in the following example:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: sysctl-example
     namespace: default
   spec:
     containers:
     - name: podexample
       image: centos
       command: ["bin/bash", "-c", "sleep INF"]
       securityContext:
         runAsUser: 2000
         runAsGroup: 3000
         allowPrivilegeEscalation: false
         capabilities:
           drop: ["ALL"]
     securityContext:
       runAsNonRoot: true
       seccompProfile:
         type: RuntimeDefault
       sysctls:
       - name: kernel.shm_rmid_forced
         value: "1"
       - name: net.ipv4.ip_local_port_range
         value: "32770       60666"
       - name: net.ipv4.tcp_syncookies
         value: "0"
       - name: net.ipv4.ping_group_range
         value: "0           200000000"
   ```

   where:

   <dl>
   <dt><code>spec.containers.securityContext.runAsUser</code></dt>
   <dd>Specifies which user ID the container is run with.</dd>
   <dt><code>spec.containers.securityContext.runAsGroup</code></dt>
   <dd>Specifies which primary group ID the containers is run with.</dd>
   <dt><code>spec.containers.securityContext.allowPrivilegeEscalation</code></dt>
   <dd>Specifies whether a pod can request privilege escalation. The default is <code>true</code>. This boolean directly controls whether the <code>no_new_privs</code> flag gets set on the container process.</dd>
   <dt><code>spec.containers.securityContext.capabilities</code></dt>
   <dd>Specifies permitted privileged actions without giving full root access. This policy ensures all capabilities are dropped from the pod.</dd>
   <dt><code>spec.securityContext.runAsNonRoot: true</code></dt>
   <dd>Specifies that the container runs with a user with any UID other than 0.</dd>
   <dt><code>spec.securityContext.seccompProfile.type: RuntimeDefault</code></dt>
   <dd>Specifies that the default seccomp profile is enabled for a pod or container workload.</dd>
   </dl>
2. Create the pod by running the following command:
   ```terminal
   $ oc apply -f sysctl_pod.yaml
   ```

**Verification**

1. Check that the pod is created by running the following command:
   ```terminal
   $ oc get pod
   ```

   ```terminal title="Example output"
   NAME              READY   STATUS            RESTARTS   AGE
   sysctl-example    1/1     Running           0          14s
   ```
2. Log in to the pod by running the following command:
   ```terminal
   $ oc rsh sysctl-example
   ```
3. Verify the values of the configured sysctl flags. For example, find the value `kernel.shm_rmid_forced` by running the following command:
   ```terminal
   sh-4.4# sysctl kernel.shm_rmid_forced
   ```

   ```terminal title="Expected output"
   kernel.shm_rmid_forced = 1
   ```

## Start a pod with unsafe sysctls {#nodes-containers-starting-pod-with-unsafe-sysctls_nodes-containers-using}

You can run a pod that is configured to use unsafe sysctls on a node where a cluster administrator explicitly enabled unsafe sysctls. You might use unsafe sysctls for situations such as high performance or real-time application tuning.

You can use the taints and toleration feature or labels on nodes to schedule those pods onto the right nodes.

The following example uses the pod `securityContext` to set a safe sysctl `kernel.shm_rmid_forced` and two unsafe sysctls, `net.core.somaxconn` and `kernel.msgmax`. There is no distinction between *safe* and *unsafe* sysctls in the specification.

:::warning

To avoid destabilizing your operating system, modify sysctl parameters only after you understand their effects.

:::

The following example illustrates what happens when you add safe and unsafe sysctls to a pod specification:

**Procedure**

1. Create a YAML file `sysctl-example-unsafe.yaml` that defines an example pod and add the `securityContext` specification, as shown in the following example:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: sysctl-example-unsafe
   spec:
     containers:
     - name: podexample
       image: centos
       command: ["bin/bash", "-c", "sleep INF"]
       securityContext:
         runAsUser: 2000
         runAsGroup: 3000
         allowPrivilegeEscalation: false
         capabilities:
           drop: ["ALL"]
     securityContext:
       runAsNonRoot: true
       seccompProfile:
         type: RuntimeDefault
       sysctls:
       - name: kernel.shm_rmid_forced
         value: "0"
       - name: net.core.somaxconn
         value: "1024"
       - name: kernel.msgmax
         value: "65536"
   ```
2. Create the pod by using the following command:
   ```terminal
   $ oc apply -f sysctl-example-unsafe.yaml
   ```
3. Verify that the pod is scheduled but does not deploy because unsafe sysctls are not allowed for the node using the following command:
   ```terminal
   $ oc get pod
   ```

   ```terminal title="Example output"
   NAME                       READY             STATUS            RESTARTS   AGE
   sysctl-example-unsafe      0/1               SysctlForbidden   0          14s
   ```

## Enable unsafe sysctls {#nodes-containers-sysctls-unsafe_nodes-containers-using}

As a cluster administrator, you can allow certain unsafe sysctls for very special situations such as high performance or real-time application tuning.

If you want to use unsafe sysctls, a cluster administrator must enable them individually for a specific type of node. The sysctls must be namespaced.

You can further control which sysctls are set in pods by specifying lists of sysctls or sysctl patterns in the `allowedUnsafeSysctls` field of the Security Context Constraints.

The `allowedUnsafeSysctls` option controls specific needs such as high performance or real-time application tuning.

:::warning

Because these sysctls are considered unsafe, the use of unsafe sysctls is at-your-own-risk and can lead to severe problems, such as improper behavior of containers, resource shortage, or node breakage.

:::

**Procedure**

1. List existing `MachineConfig` objects for your OpenShift Container Platform cluster to decide how to label your machine config by running the following command:
   ```terminal
   $ oc get machineconfigpool
   ```

   ```terminal title="Example output"
   NAME     CONFIG                                             UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT   AGE
   master   rendered-master-bfb92f0cd1684e54d8e234ab7423cc96   True      False      False      3              3                   3                     0                      42m
   worker   rendered-worker-21b6cb9a0f8919c88caf39db80ac1fce   True      False      False      3              3                   3                     0                      42m
   ```
2. Add a label to the machine config pool where the containers with the unsafe sysctls are to run by running the following command:
   ```terminal
   $ oc label machineconfigpool worker custom-kubelet=sysctl
   ```
3. Create a YAML file that defines a `KubeletConfig` custom resource (CR):
   ```yaml
   apiVersion: machineconfiguration.openshift.io/v1
   kind: KubeletConfig
   metadata:
     name: custom-kubelet
   spec:
     machineConfigPoolSelector:
       matchLabels:
         custom-kubelet: sysctl
     kubeletConfig:
       allowedUnsafeSysctls:
         - "kernel.msg*"
         - "net.core.somaxconn"
   ```

   where:

   <dl>
   <dt><code>spec.machineConfigPoolSelector.matchLabels</code></dt>
   <dd>Specifies the label from the machine config pool.</dd>
   <dt><code>spec.kubeletConfig.allowedUnsafeSysctls</code></dt>
   <dd>Specifies a list of unsafe sysctls to allow.</dd>
   </dl>
4. Create the object by running the following command:
   ```terminal
   $ oc apply -f set-sysctl-worker.yaml
   ```
5. Wait for the Machine Config Operator to generate the new rendered configuration and apply it to the machines by running the following command:
   ```terminal
   $ oc get machineconfigpool worker -w
   ```

   After some minutes the `UPDATING` status changes from True to False:

   ```terminal
   NAME     CONFIG                                             UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT   AGE
   worker   rendered-worker-f1704a00fc6f30d3a7de9a15fd68a800   False     True       False      3              2                   2                     0                      71m
   worker   rendered-worker-f1704a00fc6f30d3a7de9a15fd68a800   False     True       False      3              2                   3                     0                      72m
   worker   rendered-worker-0188658afe1f3a183ec8c4f14186f4d5   True      False      False      3              3                   3                     0                      72m
   ```
6. Create a YAML file that defines a pod and add the `securityContext` spec, as shown in the following example:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: sysctl-example-safe-unsafe
   spec:
     containers:
     - name: podexample
       image: centos
       command: ["bin/bash", "-c", "sleep INF"]
       securityContext:
         runAsUser: 2000
         runAsGroup: 3000
         allowPrivilegeEscalation: false
         capabilities:
           drop: ["ALL"]
     securityContext:
       runAsNonRoot: true
       seccompProfile:
         type: RuntimeDefault
       sysctls:
       - name: kernel.shm_rmid_forced
         value: "0"
       - name: net.core.somaxconn
         value: "1024"
       - name: kernel.msgmax
         value: "65536"
   ```
7. Create the pod by running the following command:
   ```terminal
   $ oc apply -f sysctl-example-safe-unsafe.yaml
   ```

   ```terminal title="Expected output"
   Warning: would violate PodSecurity "restricted:latest": forbidden sysctls (net.core.somaxconn, kernel.msgmax)
   pod/sysctl-example-safe-unsafe created
   ```
8. Verify that the pod is created by running the following command:
   ```terminal
   $ oc get pod
   ```

   ```terminal title="Example output"
   NAME                         READY   STATUS    RESTARTS   AGE
   sysctl-example-safe-unsafe   1/1     Running   0          19s
   ```
9. Log in to the pod by running the following command:
   ```terminal
   $ oc rsh sysctl-example-safe-unsafe
   ```
10. Verify the values of the configured sysctl flags. For example, find the value of the `net.core.somaxconn` parameter by running the following command:
    ```terminal
    sh-4.4# sysctl net.core.somaxconn
    ```

    ```terminal title="Expected output"
    net.core.somaxconn = 1024
    ```

    The unsafe sysctl is now allowed and the value is set as defined in the `securityContext` spec of the updated pod specification.

**Additional resources**

- [Linux networking documentation](https://docs.kernel.org/networking/ip-sysctl.html)
- [Configuring system controls by using the tuning CNI](/docs/networking/configuring_network_settings/configure-syscontrols-interface-tuning-cni#nw-configuring-tuning-cni_configure-syscontrols-interface-tuning-cni)
- [Using the Node Tuning Operator](/docs/scalability_and_performance/using-node-tuning-operator#using-node-tuning-operator)
- [Kernel.org documentation](https://www.kernel.org/doc/Documentation/sysctl/)
