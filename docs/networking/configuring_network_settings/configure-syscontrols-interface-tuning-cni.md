---
title: Configuring system controls and interface attributes using the tuning plugin
sidebar_position: 1
---

# Configuring system controls and interface attributes using the tuning plugin {#configure-syscontrols-interface-tuning-cni}

<a id="configure-syscontrols-interface-tuning-cni"></a>

To modify kernel parameters and interface attributes at runtime in OpenShift Container Platform, you can use the tuning Container Network Interface (CNI) meta plugin. The plugin operates in a chain with a main CNI plugin and allows you to change sysctls and interface attributes such as promiscuous mode, all-multicast mode, MTU, and MAC address.

![CNI plugin](/images/264_OpenShift_CNI_plugin_chain_0722.png)

## Configure system controls by using the tuning CNI {#nw-configuring-tuning-cni_configure-syscontrols-interface-tuning-cni}

To configure interface-level network sysctls in OpenShift Container Platform, you can use the tuning CNI meta plugin in a network attachment definition. Configure the `net.ipv4.conf.IFNAME.accept_redirects` sysctl to enable accepting and sending ICMP-redirected packets.

**Procedure**

1. Create a network attachment definition, such as `tuning-example.yaml`, with the following content:
   ```yaml
   apiVersion: "k8s.cni.cncf.io/v1"
   kind: NetworkAttachmentDefinition
   metadata:
     name: <name>
     namespace: default
   spec:
     config: '{
       "cniVersion": "0.4.0",
       "name": "<name>",
       "plugins": [{
          "type": "<main_CNI_plugin>"
         },
         {
          "type": "tuning",
          "sysctl": {
               "net.ipv4.conf.IFNAME.accept_redirects": "1"
           }
         }
        ]
   }
   ```

   where:

   <dl>
   <dt><code>metadata.name</code></dt>
   <dd>Specifies the name for the additional network attachment to create. The name must be unique within the specified namespace.</dd>
   <dt><code>metadata.namespace</code></dt>
   <dd>Specifies the namespace that the object is associated with.</dd>
   <dt><code>spec.config.cniVersion</code></dt>
   <dd>Specifies the CNI specification version.</dd>
   <dt><code>spec.config.name</code></dt>
   <dd>Specifies the name for the configuration. It is recommended to match the configuration name to the name value of the network attachment definition.</dd>
   <dt><code>spec.config.plugins.type</code></dt>
   <dd>Specifies the name of the main CNI plugin to configure.</dd>
   <dt><code>spec.config.plugins.tuning.sysctl</code></dt>
   <dd>Specifies the sysctl to set. The interface name is represented by the <code>IFNAME</code> token and is replaced with the actual name of the interface at runtime.</dd>
   </dl>

   ```yaml title="Example network attachment definition"
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
            "net.ipv4.conf.IFNAME.accept_redirects": "1"
           }
       }
     ]
   }'
   ```
2. Apply the YAML by running the following command:
   ```terminal
   $ oc apply -f tuning-example.yaml
   ```

   ```terminal title="Example output"
   networkattachmentdefinition.k8.cni.cncf.io/tuningnad created
   ```
3. Create a pod such as `examplepod.yaml` with the network attachment definition similar to the following:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: tunepod
     namespace: default
     annotations:
       k8s.v1.cni.cncf.io/networks: tuningnad
   spec:
     containers:
     - name: podexample
       image: centos
       command: ["/bin/bash", "-c", "sleep INF"]
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
   ```

   where:

   <dl>
   <dt><code>metadata.annotations.k8s.v1.cni.cncf.io/networks</code></dt>
   <dd>Specifies the name of the configured <code>NetworkAttachmentDefinition</code>.</dd>
   <dt><code>spec.containers.securityContext.runAsUser</code></dt>
   <dd>Specifies which user ID the container is run with.</dd>
   <dt><code>spec.containers.securityContext.runAsGroup</code></dt>
   <dd>Specifies which primary group ID the containers is run with.</dd>
   <dt><code>spec.containers.securityContext.allowPrivilegeEscalation</code></dt>
   <dd>Specifies if a pod can request to allow privilege escalation. If unspecified, it defaults to true. This boolean directly controls whether the <code>no_new_privs</code> flag gets set on the container process.</dd>
   <dt><code>spec.containers.securityContext.capabilities</code></dt>
   <dd>Specifies privileged actions without giving full root access. This policy ensures all capabilities are dropped from the pod.</dd>
   <dt><code>spec.securityContext.runAsNonRoot: true</code></dt>
   <dd>Specifies that the container will run with a user with any UID other than 0.</dd>
   <dt><code>spec.securityContext.seccompProfile</code></dt>
   <dd>Specifies the default seccomp profile for a pod or container workload.</dd>
   </dl>
4. Apply the yaml by running the following command:
   ```terminal
   $ oc apply -f examplepod.yaml
   ```
5. Verify that the pod is created by running the following command:
   ```terminal
   $ oc get pod
   ```

   ```terminal title="Example output"
   NAME      READY   STATUS    RESTARTS   AGE
   tunepod   1/1     Running   0          47s
   ```
6. Log in to the pod by running the following command:
   ```terminal
   $ oc rsh tunepod
   ```
7. Verify the values of the configured sysctl flags. For example, find the value `net.ipv4.conf.net1.accept_redirects` by running the following command:
   ```terminal
   sh-4.4# sysctl net.ipv4.conf.net1.accept_redirects
   ```

   ```terminal title="Expected output"
   net.ipv4.conf.net1.accept_redirects = 1
   ```

## Enable all-multicast mode by using the tuning CNI {#nw-enabling-all-multi-cni_configure-syscontrols-interface-tuning-cni}

To enable all-multicast mode on network interfaces in OpenShift Container Platform, you can use the tuning Container Network Interface (CNI) meta plugin in a network attachment definition. When enabled, the interface receives all multicast packets on the network.

**Procedure**

1. Create a network attachment definition, such as `tuning-example.yaml`, with the following content:
   ```yaml
   apiVersion: "k8s.cni.cncf.io/v1"
   kind: NetworkAttachmentDefinition
   metadata:
     name: <name>
     namespace: default
   spec:
     config: '{
       "cniVersion": "0.4.0",
       "name": "<name>",
       "plugins": [{
          "type": "<main_CNI_plugin>"
         },
         {
          "type": "tuning",
          "allmulti": true
           }
         }
        ]
   }
   ```

   where:

   <dl>
   <dt><code>&lt;name&gt;</code></dt>
   <dd>Specifies the name for the additional network attachment to create. The name must be unique within the specified namespace.</dd>
   <dt><code>default</code></dt>
   <dd>Specifies the namespace that the object is associated with.</dd>
   <dt><code>"0.4.0"</code></dt>
   <dd>Specifies the CNI specification version.</dd>
   <dt><code>"&lt;name&gt;"</code></dt>
   <dd>Specifies the name for the configuration. Match the configuration name to the name value of the network attachment definition.</dd>
   <dt><code>"&lt;main_CNI_plugin&gt;"</code></dt>
   <dd>Specifies the name of the main CNI plugin to configure.</dd>
   <dt><code>"tuning"</code></dt>
   <dd>Specifies the name of the CNI meta plugin.</dd>
   <dt><code>"true"</code></dt>
   <dd>Specifies the all-multicast mode of interface. If enabled, all multicast packets on the network will be received by the interface.</dd>
   </dl>

   ```yaml title="Example network attachment definition"
   apiVersion: "k8s.cni.cncf.io/v1"
   kind: NetworkAttachmentDefinition
   metadata:
     name: setallmulti
     namespace: default
   spec:
     config: '{
       "cniVersion": "0.4.0",
       "name": "setallmulti",
       "plugins": [
         {
           "type": "bridge"
         },
         {
           "type": "tuning",
           "allmulti": true
         }
       ]
     }'
   ```
2. Apply the settings specified in the YAML file by running the following command:
   ```terminal
   $ oc apply -f tuning-allmulti.yaml
   ```

   ```terminal title="Example output"
   networkattachmentdefinition.k8s.cni.cncf.io/setallmulti created
   ```
3. Create a pod with a network attachment definition similar to that specified in the following `examplepod.yaml` sample file:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: allmultipod
     namespace: default
     annotations:
       k8s.v1.cni.cncf.io/networks: setallmulti
   spec:
     containers:
     - name: podexample
       image: centos
       command: ["/bin/bash", "-c", "sleep INF"]
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
   ```

   where:

   <dl>
   <dt><code>metadata.annotations.k8s.v1.cni.cncf.io/networks</code></dt>
   <dd>Specifies the name of the configured <code>NetworkAttachmentDefinition</code>.</dd>
   <dt><code>spec.containers.securityContext.runAsUser</code></dt>
   <dd>Specifies which user ID the container is run with.</dd>
   <dt><code>spec.containers.securityContext.runAsGroup</code></dt>
   <dd>Specifies which primary group ID the containers is run with.</dd>
   <dt><code>spec.containers.securityContext.allowPrivilegeEscalation</code></dt>
   <dd>Specifies if a pod can request to allow privilege escalation. If unspecified, it defaults to true. This boolean directly controls whether the <code>no_new_privs</code> flag gets set on the container process.</dd>
   <dt><code>spec.containers.securityContext.capabilities</code></dt>
   <dd>Specifies privileged actions without giving full root access. This policy ensures all capabilities are dropped from the pod.</dd>
   <dt><code>spec.containers.securityContext.runAsNonRoot: true</code></dt>
   <dd>Specifies that the container will run with a user with any UID other than 0.</dd>
   <dt><code>spec.containers.securityContext.seccompProfile</code></dt>
   <dd>Specifies the default seccomp profile for a pod or container workload.</dd>
   </dl>
4. Apply the settings specified in the YAML file by running the following command:
   ```terminal
   $ oc apply -f examplepod.yaml
   ```
5. Verify that the pod is created by running the following command:
   ```terminal
   $ oc get pod
   ```

   ```terminal title="Example output"
   NAME          READY   STATUS    RESTARTS   AGE
   allmultipod   1/1     Running   0          23s
   ```
6. Log in to the pod by running the following command:
   ```terminal
   $ oc rsh allmultipod
   ```
7. List all the interfaces associated with the pod by running the following command:
   ```terminal
   sh-4.4# ip link
   ```

   ```terminal title="Example output"
   1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
   2: eth0@if22: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 8901 qdisc noqueue state UP mode DEFAULT group default
       link/ether 0a:58:0a:83:00:10 brd ff:ff:ff:ff:ff:ff link-netnsid 0
   3: net1@if24: <BROADCAST,MULTICAST,ALLMULTI,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
       link/ether ee:9b:66:a4:ec:1d brd ff:ff:ff:ff:ff:ff link-netnsid 0
   ```

   where:

   <dl>
   <dt><code>eth0@if22</code></dt>
   <dd>Specifies the primary interface.</dd>
   <dt><code>net1@if24</code></dt>
   <dd>Specifies the secondary interface configured with the network-attachment-definition that supports the all-multicast mode (ALLMULTI flag).</dd>
   </dl>

**Additional resources**

- [Using sysctls in containers](/docs/nodes/containers/nodes-containers-sysctls#nodes-containers-sysctls)
- [SR-IOV network node configuration object](/docs/networking/hardware_networks/configuring-sriov-device#nw-sriov-networknodepolicy-object_configuring-sriov-device)
- [Configuring interface-level network sysctl settings and all-multicast mode for SR-IOV networks](/docs/networking/hardware_networks/configuring-interface-sysctl-sriov-device#configuring-interface-level-sysctl-settings-sriov-device)
