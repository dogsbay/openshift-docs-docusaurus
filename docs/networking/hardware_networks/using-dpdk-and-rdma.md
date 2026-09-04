---
title: Using DPDK and RDMA
sidebar_position: 10
---

# Using DPDK and RDMA

<a id="using-dpdk-and-rdma"></a>

The containerized Data Plane Development Kit (DPDK) application is supported on OpenShift Container Platform. You can use Single Root I/O Virtualization (SR-IOV) network hardware with the Data Plane Development Kit (DPDK) and with remote direct memory access (RDMA).

Before you perform any tasks in the following documentation, ensure that you [installed the SR-IOV Network Operator](/docs/networking/networking_operators/sr-iov-operator/installing-sriov-operator#installing-sriov-operator).

## Example use of a virtual function in a pod {#example-vf-use-in-pod_using-dpdk-and-rdma}

You can run a remote direct memory access (RDMA) or a Data Plane Development Kit (DPDK) application in a pod with SR-IOV VF attached.

This example shows a pod using a virtual function (VF) in RDMA mode:

```yaml title="Pod spec that uses RDMA mode"
apiVersion: v1
kind: Pod
metadata:
  name: rdma-app
  annotations:
    k8s.v1.cni.cncf.io/networks: sriov-rdma-mlnx
spec:
  containers:
  - name: testpmd
    image: <RDMA_image>
    imagePullPolicy: IfNotPresent
    securityContext:
      runAsUser: 0
      capabilities:
        add: ["IPC_LOCK","SYS_RESOURCE","NET_RAW"]
    command: ["sleep", "infinity"]
```

The following example shows a pod with a VF in DPDK mode:

```yaml title="Pod spec that uses DPDK mode"
apiVersion: v1
kind: Pod
metadata:
  name: dpdk-app
  annotations:
    k8s.v1.cni.cncf.io/networks: sriov-dpdk-net
spec:
  containers:
  - name: testpmd
    image: <DPDK_image>
    securityContext:
      runAsUser: 0
      capabilities:
        add: ["IPC_LOCK","SYS_RESOURCE","NET_RAW"]
    volumeMounts:
    - mountPath: /dev/hugepages
      name: hugepage
    resources:
      limits:
        memory: "1Gi"
        cpu: "2"
        hugepages-1Gi: "4Gi"
      requests:
        memory: "1Gi"
        cpu: "2"
        hugepages-1Gi: "4Gi"
    command: ["sleep", "infinity"]
  volumes:
  - name: hugepage
    emptyDir:
      medium: HugePages
```

## Using a virtual function in DPDK mode with an Intel NIC {#example-vf-use-in-dpdk-mode-intel_using-dpdk-and-rdma}

You can use a virtual function (VF) in Data Plane Development Kit (DPDK) mode with an Intel NIC by creating a `SriovNetworkNodePolicy` object and then deploying a pod.

**Prerequisites**

- Install the OpenShift CLI (`oc`).
- Install the SR-IOV Network Operator.
- Log in as a user with `cluster-admin` privileges.

**Procedure**

1. Create the following `SriovNetworkNodePolicy` object, and then save the YAML in the `intel-dpdk-node-policy.yaml` file.
   ```yaml
   apiVersion: sriovnetwork.openshift.io/v1
   kind: SriovNetworkNodePolicy
   metadata:
     name: intel-dpdk-node-policy
     namespace: openshift-sriov-network-operator
   spec:
     resourceName: intelnics
     nodeSelector:
       feature.node.kubernetes.io/network-sriov.capable: "true"
     priority: <priority>
     numVfs: <num>
     nicSelector:
       vendor: "8086"
       deviceID: "158b"
       pfNames: ["<pf_name>", ...]
       rootDevices: ["<pci_bus_id>", "..."]
     deviceType: vfio-pci
   ```

   where:

   <dl>
   <dt><code>spec.deviceType</code></dt>
   <dd>Specifies the driver type for the virtual functions. Set to <code>vfio-pci</code>.</dd>
   </dl>

   :::note

   See the `Configuring SR-IOV network devices` section for a detailed explanation on each option in `SriovNetworkNodePolicy`.

   When applying the configuration specified in a `SriovNetworkNodePolicy` object, the SR-IOV Operator might drain the nodes, and in some cases, reboot nodes. It might take several minutes for a configuration change to apply. Ensure that there are enough available nodes in your cluster to handle the evicted workload beforehand.

   After the configuration update is applied, all the pods in `openshift-sriov-network-operator` namespace will change to a `Running` status.

   :::
2. Create the `SriovNetworkNodePolicy` object by running the following command:
   ```terminal
   $ oc create -f intel-dpdk-node-policy.yaml
   ```
3. Create the following `SriovNetwork` object, and then save the YAML in the `intel-dpdk-network.yaml` file.
   ```yaml
   apiVersion: sriovnetwork.openshift.io/v1
   kind: SriovNetwork
   metadata:
     name: intel-dpdk-network
     namespace: openshift-sriov-network-operator
   spec:
     networkNamespace: <target_namespace>
     ipam: |-
   # ...
     vlan: <vlan>
     resourceName: intelnics
   ```

   where:

   <dl>
   <dt><code>spec.ipam</code></dt>
   <dd>Specifies a configuration object for the IPAM CNI plugin as a YAML block scalar. The plugin manages IP address assignment for the attachment definition.</dd>
   </dl>

   :::note

   See the "Configuring SR-IOV additional network" section for a detailed explanation on each option in `SriovNetwork`.

   :::

   An optional library, app-netutil, provides several API methods for gathering network information about a container’s parent pod.
4. Create the `SriovNetwork` object by running the following command:
   ```terminal
   $ oc create -f intel-dpdk-network.yaml
   ```
5. Create the following `Pod` spec, and then save the YAML in the `intel-dpdk-pod.yaml` file.
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: dpdk-app
     namespace: <target_namespace>
     annotations:
       k8s.v1.cni.cncf.io/networks: intel-dpdk-network
   spec:
     containers:
     - name: testpmd
       image: <DPDK_image>
       securityContext:
         runAsUser: 0
         capabilities:
           add: ["IPC_LOCK","SYS_RESOURCE","NET_RAW"]
       volumeMounts:
       - mountPath: /mnt/huge
         name: hugepage
       resources:
         limits:
           openshift.io/intelnics: "1"
           memory: "1Gi"
           cpu: "4"
           hugepages-1Gi: "4Gi"
         requests:
           openshift.io/intelnics: "1"
           memory: "1Gi"
           cpu: "4"
           hugepages-1Gi: "4Gi"
       command: ["sleep", "infinity"]
     volumes:
     - name: hugepage
       emptyDir:
         medium: HugePages
   ```

   where:

   <dl>
   <dt><code>metadata.namespace</code></dt>
   <dd>Specifies the same namespace where the <code>SriovNetwork</code> object <code>intel-dpdk-network</code> is created. If you want to create the pod in a different namespace, change <code>target_namespace</code> in both the <code>Pod</code> spec and the <code>SriovNetwork</code> object.</dd>
   <dt><code>spec.containers.image</code></dt>
   <dd>Specifies the DPDK image which includes your application and the DPDK library used by application.</dd>
   <dt><code>spec.containers.securityContext.capabilities.add</code></dt>
   <dd>Specifies additional capabilities required by the application inside the container for hugepage allocation, system resource allocation, and network interface access.</dd>
   <dt><code>spec.containers.volumeMounts.mountPath</code></dt>
   <dd>Specifies the path where a hugepage volume is mounted in the DPDK pod. The hugepage volume is backed by the <code>emptyDir</code> volume type with the medium being <code>Hugepages</code>.</dd>
   <dt><code>spec.containers.resources.limits.openshift.io/intelnics</code></dt>
   <dd>Optional: Specifies the number of DPDK devices allocated to DPDK pod. If not explicitly specified, this resource request and limit is automatically added by the SR-IOV network resource injector. The SR-IOV network resource injector is an admission controller component managed by the SR-IOV Operator. It is enabled by default and can be disabled by setting <code>enableInjector</code> option to <code>false</code> in the default <code>SriovOperatorConfig</code> CR.</dd>
   <dt><code>spec.containers.resources.limits.cpu</code></dt>
   <dd>Specifies the number of CPUs. The DPDK pod usually requires exclusive CPUs to be allocated from the kubelet. This is achieved by setting CPU Manager policy to <code>static</code> and creating a pod with <code>Guaranteed</code> QoS.</dd>
   <dt><code>spec.containers.resources.limits.hugepages-1Gi</code></dt>
   <dd>Specifies the hugepage size <code>hugepages-1Gi</code> or <code>hugepages-2Mi</code> and the quantity of hugepages that will be allocated to the DPDK pod. Configure <code>2Mi</code> and <code>1Gi</code> hugepages separately. Configuring <code>1Gi</code> hugepage requires adding kernel arguments to Nodes. For example, adding kernel arguments <code>default_hugepagesz=1GB</code>, <code>hugepagesz=1G</code> and <code>hugepages=16</code> will result in <code>16*1Gi</code> hugepages be allocated during system boot.</dd>
   </dl>
6. Create the DPDK pod by running the following command:
   ```terminal
   $ oc create -f intel-dpdk-pod.yaml
   ```

## Using a virtual function in DPDK mode with a Mellanox NIC {#example-vf-use-in-dpdk-mode-mellanox_using-dpdk-and-rdma}

You can create a network node policy and create a Data Plane Development Kit (DPDK) pod by using a virtual function in DPDK mode with a Mellanox NIC.

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).
- You have installed the Single Root I/O Virtualization (SR-IOV) Network Operator.
- You have logged in as a user with `cluster-admin` privileges.

**Procedure**

1. Save the following `SriovNetworkNodePolicy` YAML configuration to an `mlx-dpdk-node-policy.yaml` file:
   ```yaml
   apiVersion: sriovnetwork.openshift.io/v1
   kind: SriovNetworkNodePolicy
   metadata:
     name: mlx-dpdk-node-policy
     namespace: openshift-sriov-network-operator
   spec:
     resourceName: mlxnics
     nodeSelector:
       feature.node.kubernetes.io/network-sriov.capable: "true"
     priority: <priority>
     numVfs: <num>
     nicSelector:
       vendor: "15b3"
       deviceID: "1015"
       pfNames: ["<pf_name>", ...]
       rootDevices: ["<pci_bus_id>", "..."]
     deviceType: netdevice
     isRdma: true
   ```

   where:

   <dl>
   <dt><code>spec.nicSelector.deviceID</code></dt>
   <dd>Specifies the device hex code of the SR-IOV network device. The value <code>"1015"</code> is associated with a Mellanox NIC.</dd>
   <dt><code>spec.deviceType</code></dt>
   <dd>Specifies the driver type for the virtual functions. A Mellanox SR-IOV Virtual Function (VF) can work in DPDK mode without using the <code>vfio-pci</code> device type. Set to <code>netdevice</code>. The VF device is displayed as a kernel network interface inside a container.</dd>
   <dt><code>spec.isRdma</code></dt>
   <dd>Setting to <code>true</code> enables Remote Direct Memory Access (RDMA) mode. This is required for Mellanox cards to work in DPDK mode.</dd>
   </dl>

   :::note

   See *Configuring an SR-IOV network device* for a detailed explanation of each option in the `SriovNetworkNodePolicy` object.

   When applying the configuration specified in an `SriovNetworkNodePolicy` object, the SR-IOV Operator might drain the nodes, and in some cases, reboot nodes. It might take several minutes for a configuration change to apply. Ensure that there are enough available nodes in your cluster to handle the evicted workload beforehand.

   After the configuration update is applied, all the pods in the `openshift-sriov-network-operator` namespace will change to a `Running` status.

   :::
2. Create the `SriovNetworkNodePolicy` object by running the following command:
   ```terminal
   $ oc create -f mlx-dpdk-node-policy.yaml
   ```
3. Save the following `SriovNetwork` YAML configuration to an `mlx-dpdk-network.yaml` file:
   ```yaml
   apiVersion: sriovnetwork.openshift.io/v1
   kind: SriovNetwork
   metadata:
     name: mlx-dpdk-network
     namespace: openshift-sriov-network-operator
   spec:
     networkNamespace: <target_namespace>
     ipam: |-
   ...
     vlan: <vlan>
     resourceName: mlxnics
   ```

   where:

   <dl>
   <dt><code>spec.ipam</code></dt>
   <dd>Specifies a configuration object for the IP Address Management (IPAM) Container Network Interface (CNI) plugin as a YAML block scalar. The plugin manages IP address assignment for the attachment definition.</dd>
   </dl>

   :::note

   See *Configuring an SR-IOV network device* for a detailed explanation on each option in the `SriovNetwork` object.

   :::

   The `app-netutil` option library provides several API methods for gathering network information about the parent pod of a container.
4. Create the `SriovNetwork` object by running the following command:
   ```terminal
   $ oc create -f mlx-dpdk-network.yaml
   ```
5. Save the following `Pod` YAML configuration to an `mlx-dpdk-pod.yaml` file:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: dpdk-app
     namespace: <target_namespace>
     annotations:
       k8s.v1.cni.cncf.io/networks: mlx-dpdk-network
   spec:
     containers:
     - name: testpmd
       image: <DPDK_image>
       securityContext:
         runAsUser: 0
         capabilities:
           add: ["IPC_LOCK","SYS_RESOURCE","NET_RAW"]
       volumeMounts:
       - mountPath: /mnt/huge
         name: hugepage
       resources:
         limits:
           openshift.io/mlxnics: "1"
           memory: "1Gi"
           cpu: "4"
           hugepages-1Gi: "4Gi"
         requests:
           openshift.io/mlxnics: "1"
           memory: "1Gi"
           cpu: "4"
           hugepages-1Gi: "4Gi"
       command: ["sleep", "infinity"]
     volumes:
     - name: hugepage
       emptyDir:
         medium: HugePages
   ```

   where:

   <dl>
   <dt><code>metadata.namespace</code></dt>
   <dd>Specifies the same namespace where <code>SriovNetwork</code> object <code>mlx-dpdk-network</code> is created. To create the pod in a different namespace, change <code>target_namespace</code> in both the <code>Pod</code> spec and <code>SriovNetwork</code> object.</dd>
   <dt><code>spec.containers.image</code></dt>
   <dd>Specifies the DPDK image which includes your application and the DPDK library used by the application.</dd>
   <dt><code>spec.containers.securityContext.capabilities.add</code></dt>
   <dd>Specifies additional capabilities required by the application inside the container for hugepage allocation, system resource allocation, and network interface access.</dd>
   <dt><code>spec.containers.volumeMounts.mountPath</code></dt>
   <dd>Specifies the path where the hugepage volume is mounted in the DPDK pod. The hugepage volume is backed by the <code>emptyDir</code> volume type with the medium being <code>Hugepages</code>.</dd>
   <dt><code>spec.containers.resources.limits.openshift.io/mlxnics</code></dt>
   <dd>Optional: Specifies the number of DPDK devices allocated for the DPDK pod. If not explicitly specified, this resource request and limit is automatically added by the SR-IOV network resource injector. The SR-IOV network resource injector is an admission controller component managed by SR-IOV Operator. It is enabled by default and can be disabled by setting the <code>enableInjector</code> option to <code>false</code> in the default <code>SriovOperatorConfig</code> CR.</dd>
   <dt><code>spec.containers.resources.limits.cpu</code></dt>
   <dd>Specifies the number of CPUs. The DPDK pod usually requires that exclusive CPUs be allocated from the kubelet. To do this, set the CPU Manager policy to <code>static</code> and create a pod with <code>Guaranteed</code> Quality of Service (QoS).</dd>
   <dt><code>spec.containers.resources.limits.hugepages-1Gi</code></dt>
   <dd>Specifies the hugepage size <code>hugepages-1Gi</code> or <code>hugepages-2Mi</code> and the quantity of hugepages that will be allocated to the DPDK pod. Configure <code>2Mi</code> and <code>1Gi</code> hugepages separately. Configuring <code>1Gi</code> hugepages requires adding kernel arguments to Nodes.</dd>
   </dl>
6. Create the DPDK pod by running the following command:
   ```terminal
   $ oc create -f mlx-dpdk-pod.yaml
   ```

## Using the TAP CNI to run a rootless DPDK workload with kernel access {#nw-running-dpdk-rootless-tap_using-dpdk-and-rdma}

DPDK applications can use `virtio-user` as an exception path to inject certain types of packets, such as log messages, into the kernel for processing. For more information about this feature, see [Virtio_user as Exception Path](https://doc.dpdk.org/guides/howto/virtio_user_as_exception_path.html).

In OpenShift Container Platform version 4.14 and later, you can use non-privileged pods to run DPDK applications alongside the tap CNI plugin. To enable this functionality, you need to mount the `vhost-net` device by setting the `needVhostNet` parameter to `true` within the `SriovNetworkNodePolicy` object.

**Figure 1. DPDK and TAP example configuration**

![DPDK and TAP plugin](/images/348_OpenShift_rootless_DPDK_0923.png)

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).
- You have installed the SR-IOV Network Operator.
- You are logged in as a user with `cluster-admin` privileges.
- Ensure that `setsebools container_use_devices=on` is set as root on all nodes.
  :::note

  Use the Machine Config Operator to set this SELinux boolean.

  :::

**Procedure**

1. Create a file, such as `test-namespace.yaml`, with content such as the following example:
   ```yaml
   apiVersion: v1
   kind: Namespace
   metadata:
     name: test-namespace
     labels:
       pod-security.kubernetes.io/enforce: privileged
       pod-security.kubernetes.io/audit: privileged
       pod-security.kubernetes.io/warn: privileged
       security.openshift.io/scc.podSecurityLabelSync: "false"
   ```
2. Create the new `Namespace` object by running the following command:
   ```terminal
   $ oc apply -f test-namespace.yaml
   ```
3. Create a file, such as `sriov-node-network-policy.yaml`, with content such as the following example:
   ```yaml
   apiVersion: sriovnetwork.openshift.io/v1
   kind: SriovNetworkNodePolicy
   metadata:
    name: sriovnic
    namespace: openshift-sriov-network-operator
   spec:
    deviceType: netdevice
    isRdma: true
    needVhostNet: true
    nicSelector:
      vendor: "15b3"
      deviceID: "101b"
      rootDevices: ["00:05.0"]
    numVfs: 10
    priority: 99
    resourceName: sriovnic
    nodeSelector:
       feature.node.kubernetes.io/network-sriov.capable: "true"
   ```

   where:

   <dl>
   <dt><code>spec.deviceType</code></dt>
   <dd>Specifies that the profile is tailored specifically for Mellanox Network Interface Controllers (NICs). Set to <code>netdevice</code>.</dd>
   <dt><code>spec.isRdma</code></dt>
   <dd>Setting to <code>true</code> is only required for a Mellanox NIC.</dd>
   <dt><code>spec.needVhostNet</code></dt>
   <dd>Setting to <code>true</code> mounts the <code>/dev/net/tun</code> and <code>/dev/vhost-net</code> devices into the container so the application can create a tap device and connect the tap device to the DPDK workload.</dd>
   <dt><code>spec.nicSelector.vendor</code></dt>
   <dd>Specifies the vendor hexadecimal code of the SR-IOV network device. The value <code>"15b3"</code> is associated with a Mellanox NIC.</dd>
   <dt><code>spec.nicSelector.deviceID</code></dt>
   <dd>Specifies the device hexadecimal code of the SR-IOV network device.</dd>
   </dl>
4. Create the `SriovNetworkNodePolicy` object by running the following command:
   ```terminal
   $ oc create -f sriov-node-network-policy.yaml
   ```
5. Create the following `SriovNetwork` object, and then save the YAML in the `sriov-network-attachment.yaml` file:
   ```yaml
   apiVersion: sriovnetwork.openshift.io/v1
   kind: SriovNetwork
   metadata:
    name: sriov-network
    namespace: openshift-sriov-network-operator
   spec:
    networkNamespace: test-namespace
    resourceName: sriovnic
    spoofChk: "off"
    trust: "on"
   ```

   :::note

   See the "Configuring SR-IOV additional network" section for a detailed explanation on each option in `SriovNetwork`.

   :::

   An optional library, `app-netutil`, provides several API methods for gathering network information about a container’s parent pod.
6. Create the `SriovNetwork` object by running the following command:
   ```terminal
   $ oc create -f sriov-network-attachment.yaml
   ```
7. Create a file, such as `tap-example.yaml`, that defines a network attachment definition, with content such as the following example:
   ```yaml
   apiVersion: "k8s.cni.cncf.io/v1"
   kind: NetworkAttachmentDefinition
   metadata:
    name: tap-one
    namespace: test-namespace
   spec:
    config: '{
      "cniVersion": "0.4.0",
      "name": "tap",
      "plugins": [
        {
           "type": "tap",
           "multiQueue": true,
           "selinuxcontext": "system_u:system_r:container_t:s0"
        },
        {
          "type":"tuning",
          "capabilities":{
            "mac":true
          }
        }
      ]
    }'
   ```

   where:

   <dl>
   <dt><code>metadata.namespace</code></dt>
   <dd>Specifies the same <code>target_namespace</code> where the <code>SriovNetwork</code> object is created.</dd>
   </dl>
8. Create the `NetworkAttachmentDefinition` object by running the following command:
   ```terminal
   $ oc apply -f tap-example.yaml
   ```
9. Create a file, such as `dpdk-pod-rootless.yaml`, with content such as the following example:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: dpdk-app
     namespace: test-namespace
     annotations:
       k8s.v1.cni.cncf.io/networks: '[
         {"name": "sriov-network", "namespace": "test-namespace"},
         {"name": "tap-one", "interface": "ext0", "namespace": "test-namespace"}]'
   spec:
     nodeSelector:
       kubernetes.io/hostname: "worker-0"
     securityContext:
         fsGroup: 1001
         runAsGroup: 1001
         seccompProfile:
           type: RuntimeDefault
     containers:
     - name: testpmd
       image: <DPDK_image>
       securityContext:
         capabilities:
           drop: ["ALL"]
           add:
             - IPC_LOCK
             - NET_RAW #for mlx only
         runAsUser: 1001
         privileged: false
         allowPrivilegeEscalation: true
         runAsNonRoot: true
       volumeMounts:
       - mountPath: /mnt/huge
         name: hugepages
       resources:
         limits:
           openshift.io/sriovnic: "1"
           memory: "1Gi"
           cpu: "4"
           hugepages-1Gi: "4Gi"
         requests:
           openshift.io/sriovnic: "1"
           memory: "1Gi"
           cpu: "4"
           hugepages-1Gi: "4Gi"
       command: ["sleep", "infinity"]
     runtimeClassName: performance-cnf-performanceprofile
     volumes:
     - name: hugepages
       emptyDir:
         medium: HugePages
   ```

   where:

   <dl>
   <dt><code>metadata.namespace</code></dt>
   <dd>Specifies the same <code>target_namespace</code> in which the <code>SriovNetwork</code> object is created. If you want to create the pod in a different namespace, change <code>target_namespace</code> in both the <code>Pod</code> spec and the <code>SriovNetwork</code> object.</dd>
   <dt><code>spec.securityContext.fsGroup</code></dt>
   <dd>Sets the group ownership of volume-mounted directories and files created in those volumes.</dd>
   <dt><code>spec.securityContext.runAsGroup</code></dt>
   <dd>Specifies the primary group ID used for running the container.</dd>
   <dt><code>spec.containers.image</code></dt>
   <dd>Specifies the DPDK image that contains your application and the DPDK library used by application.</dd>
   <dt><code>spec.containers.securityContext.capabilities.drop</code></dt>
   <dd>Removing all capabilities (<code>ALL</code>) from the container’s <code>securityContext</code> means that the container has no special privileges beyond what is necessary for normal operation.</dd>
   <dt><code>spec.containers.securityContext.capabilities.add</code></dt>
   <dd>Specifies additional capabilities required by the application inside the container for hugepage allocation, system resource allocation, and network interface access. These capabilities must also be set in the binary file by using the <code>setcap</code> command. Mellanox network interface controller (NIC) requires the <code>NET_RAW</code> capability.</dd>
   <dt><code>spec.containers.securityContext.runAsUser</code></dt>
   <dd>Specifies the user ID used for running the container.</dd>
   <dt><code>spec.containers.securityContext.privileged</code></dt>
   <dd>Setting to <code>false</code> indicates that the container or containers within the pod should not be granted privileged access to the host system.</dd>
   <dt><code>spec.containers.securityContext.allowPrivilegeEscalation</code></dt>
   <dd>Setting to <code>true</code> allows a container to escalate its privileges beyond the initial non-root privileges it might have been assigned.</dd>
   <dt><code>spec.containers.securityContext.runAsNonRoot</code></dt>
   <dd>Setting to <code>true</code> ensures that the container runs with a non-root user. This helps enforce the principle of least privilege, limiting the potential impact of compromising the container and reducing the attack surface.</dd>
   <dt><code>spec.containers.volumeMounts.mountPath</code></dt>
   <dd>Specifies the path where a hugepage volume is mounted in the DPDK pod. The hugepage volume is backed by the <code>emptyDir</code> volume type with the medium being <code>Hugepages</code>.</dd>
   <dt><code>spec.containers.resources.limits.openshift.io/sriovnic</code></dt>
   <dd>Optional: Specifies the number of DPDK devices allocated for the DPDK pod. If not explicitly specified, this resource request and limit is automatically added by the SR-IOV network resource injector. The SR-IOV network resource injector is an admission controller component managed by SR-IOV Operator. It is enabled by default and can be disabled by setting the <code>enableInjector</code> option to <code>false</code> in the default <code>SriovOperatorConfig</code> CR.</dd>
   <dt><code>spec.containers.resources.limits.cpu</code></dt>
   <dd>Specifies the number of CPUs. The DPDK pod usually requires exclusive CPUs to be allocated from the kubelet. This is achieved by setting CPU Manager policy to <code>static</code> and creating a pod with <code>Guaranteed</code> QoS.</dd>
   <dt><code>spec.containers.resources.limits.hugepages-1Gi</code></dt>
   <dd>Specifies the hugepage size <code>hugepages-1Gi</code> or <code>hugepages-2Mi</code> and the quantity of hugepages that will be allocated to the DPDK pod. Configure <code>2Mi</code> and <code>1Gi</code> hugepages separately. Configuring <code>1Gi</code> hugepage requires adding kernel arguments to Nodes. For example, adding kernel arguments <code>default_hugepagesz=1GB</code>, <code>hugepagesz=1G</code> and <code>hugepages=16</code> will result in <code>16*1Gi</code> hugepages be allocated during system boot.</dd>
   <dt><code>spec.runtimeClassName</code></dt>
   <dd>Specifies the performance profile runtime class. If your performance profile is not named <code>cnf-performance profile</code>, replace that string with the correct performance profile name.</dd>
   </dl>
10. Create the DPDK pod by running the following command:
    ```terminal
    $ oc create -f dpdk-pod-rootless.yaml
    ```

**Additional resources**

- [Creating a performance profile](/docs/scalability_and_performance/cnf-tuning-low-latency-nodes-with-perf-profile#cnf-create-performance-profiles_cnf-tuning-low-latency-nodes-with-perf-profile)
- [Configuring an SR-IOV network device](/docs/networking/hardware_networks/configuring-sriov-device#configuring-sriov-device)

## Overview of achieving a specific DPDK line rate {#nw-sriov-example-dpdk-line-rate_using-dpdk-and-rdma}

To achieve a specific Data Plane Development Kit (DPDK) line rate, deploy a Node Tuning Operator and configure Single Root I/O Virtualization (SR-IOV). You must also tune the DPDK settings for the following resources:

- Isolated CPUs
- Hugepages
- The topology scheduler

:::note

In previous versions of OpenShift Container Platform, the Performance Addon Operator was used to implement automatic tuning to achieve low latency performance for OpenShift Container Platform applications. In OpenShift Container Platform 4.11 and later, this functionality is part of the Node Tuning Operator.

:::

The following diagram shows the components of a DPDK test environment:

![DPDK test environment](/images/261_OpenShift_DPDK_0722.png)

- **Traffic generator**: An application that can generate high-volume packet traffic.
- **SR-IOV-supporting NIC**: A network interface controller (NIC) compatible with SR-IOV. The card runs several virtual functions on a physical interface.
- **Physical Function (PF)**: A PCI Express (PCIe) function of a network adapter that supports the SR-IOV interface.
- **Virtual Function (VF)**:  A lightweight PCIe function on a network adapter that supports SR-IOV. The VF is associated with the PCIe PF on the network adapter. The VF represents a virtualized instance of the network adapter.
- **Switch**: A network switch. Nodes can also be connected back-to-back.
- `testpmd`: An example application included with DPDK. The `testpmd` application can be used to test the DPDK in a packet-forwarding mode. The `testpmd` application is also an example of how to build a fully-fledged application using the DPDK Software Development Kit (SDK).
- **worker 0** and **worker 1**: OpenShift Container Platform nodes.

## Using SR-IOV and the Node Tuning Operator to achieve a DPDK line rate {#nw-example-dpdk-line-rate_using-dpdk-and-rdma}

You can use the Node Tuning Operator to configure isolated CPUs, hugepages, and a topology scheduler. You can then use the Node Tuning Operator with Single Root I/O Virtualization (SR-IOV) to achieve a specific Data Plane Development Kit (DPDK) line rate.

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).
- You have installed the SR-IOV Network Operator.
- You have logged in as a user with `cluster-admin` privileges.
- You have deployed a standalone Node Tuning Operator.
  :::note

  In previous versions of OpenShift Container Platform, the Performance Addon Operator was used to implement automatic tuning to achieve low latency performance for OpenShift applications. In OpenShift Container Platform 4.11 and later, this functionality is part of the Node Tuning Operator.

  :::

**Procedure**

1. Create a `PerformanceProfile` object based on the following example:
   ```yaml
   apiVersion: performance.openshift.io/v2
   kind: PerformanceProfile
   metadata:
     name: performance
   spec:
     globallyDisableIrqLoadBalancing: true
     cpu:
       isolated: 21-51,73-103
       reserved: 0-20,52-72
     hugepages:
       defaultHugepagesSize: 1G
       pages:
         - count: 32
           size: 1G
     net:
       userLevelNetworking: true
     numa:
       topologyPolicy: "single-numa-node"
     nodeSelector:
       node-role.kubernetes.io/worker-cnf: ""
   ```

   where:

   <dl>
   <dt><code>metadata.name</code></dt>
   <dd>Specifies the name of the performance profile.</dd>
   <dt><code>spec.cpu.isolated</code></dt>
   <dd>Specifies the CPUs that are isolated for the application workloads. If Hyper-Threading is enabled on the system, allocate the relevant symbolic links to the <code>isolated</code> and <code>reserved</code> CPU groups. If the system has multiple non-uniform memory access (NUMA) nodes, allocate CPUs from both NUMAs to both groups. You can also use the Performance Profile Creator for this task. For more information, see <em>Creating a performance profile</em>.</dd>
   <dt><code>spec.cpu.reserved</code></dt>
   <dd>Specifies the CPUs that are reserved for the operating system and Kubernetes system daemons. You can also specify a list of devices that will have their queues set to the reserved CPU count. For more information, see <em>Reducing NIC queues using the Node Tuning Operator</em>.</dd>
   <dt><code>spec.hugepages.defaultHugepagesSize</code></dt>
   <dd>Specifies the default size of hugepages.</dd>
   <dt><code>spec.hugepages.pages</code></dt>
   <dd>Specifies the number and size of hugepages to allocate. You can specify the NUMA configuration for the hugepages. By default, the system allocates an even number to every NUMA node on the system.</dd>
   <dt><code>spec.net.userLevelNetworking</code></dt>
   <dd>Specifies whether to enable user-level networking. Set to <code>true</code> for DPDK workloads.</dd>
   <dt><code>spec.numa.topologyPolicy</code></dt>
   <dd>Specifies the NUMA topology policy. Set to <code>single-numa-node</code> to ensure that all resources are allocated from the same NUMA node.</dd>
   <dt><code>spec.nodeSelector</code></dt>
   <dd>Specifies the node selector label for nodes that this performance profile applies to.</dd>
   </dl>
2. Save the `yaml` file as `mlx-dpdk-perfprofile-policy.yaml`.
3. Apply the performance profile using the following command:
   ```terminal
   $ oc create -f mlx-dpdk-perfprofile-policy.yaml
   ```

### DPDK library for use with container applications {#nw-sriov-app-netutil_using-dpdk-and-rdma}

An [optional library](https://github.com/openshift/app-netutil), `app-netutil`, provides several API methods for gathering network information about a pod from within a container running within that pod.

This library can assist with integrating SR-IOV virtual functions (VFs) in Data Plane Development Kit (DPDK) mode into the container. The library provides both a Golang API and a C API.

Currently there are three API methods implemented:

`GetCPUInfo()`
: This function determines which CPUs are available to the container and returns the list.

`GetHugepages()`
: This function determines the amount of huge page memory requested in the `Pod` spec for each container and returns the values.

`GetInterfaces()`
: This function determines the set of interfaces in the container and returns the list. The return value includes the interface type and type-specific data for each interface.

The repository for the library includes a sample Dockerfile to build a container image, `dpdk-app-centos`. The container image can run one of the following DPDK sample applications, depending on an environment variable in the pod specification: `l2fwd`, `l3wd` or `testpmd`. The container image provides an example of integrating the `app-netutil` library into the container image itself. The library can also integrate into an init container. The init container can collect the required data and pass the data to an existing DPDK workload.

### Example SR-IOV Network Operator for virtual functions {#nw-sriov-network-operator_using-dpdk-and-rdma}

You can use the Single Root I/O Virtualization (SR-IOV) Network Operator to allocate and configure Virtual Functions (VFs) from SR-IOV-supporting Physical Function NICs on the nodes.

For more information on deploying the Operator, see *Installing the SR-IOV Network Operator*. For more information on configuring an SR-IOV network device, see *Configuring an SR-IOV network device*.

There are some differences between running Data Plane Development Kit (DPDK) workloads on Intel VFs and Mellanox VFs. This section provides object configuration examples for both VF types. The following is an example of an `sriovNetworkNodePolicy` object used to run DPDK applications on Intel NICs:

```yaml
apiVersion: sriovnetwork.openshift.io/v1
kind: SriovNetworkNodePolicy
metadata:
  name: dpdk-nic-1
  namespace: openshift-sriov-network-operator
spec:
  deviceType: vfio-pci
  needVhostNet: true
  nicSelector:
    pfNames: ["ens3f0"]
  nodeSelector:
    node-role.kubernetes.io/worker-cnf: ""
  numVfs: 10
  priority: 99
  resourceName: dpdk_nic_1
---
apiVersion: sriovnetwork.openshift.io/v1
kind: SriovNetworkNodePolicy
metadata:
  name: dpdk-nic-1
  namespace: openshift-sriov-network-operator
spec:
  deviceType: vfio-pci
  needVhostNet: true
  nicSelector:
    pfNames: ["ens3f1"]
  nodeSelector:
  node-role.kubernetes.io/worker-cnf: ""
  numVfs: 10
  priority: 99
  resourceName: dpdk_nic_2
```

where:

`spec.deviceType`
: For Intel NICs, `deviceType` must be `vfio-pci`.

`spec.needVhostNet`
: If kernel communication with DPDK workloads is required, set to `true`. This mounts the `/dev/net/tun` and `/dev/vhost-net` devices into the container so the application can create a tap device and connect the tap device to the DPDK workload.

The following is an example of an `sriovNetworkNodePolicy` object for Mellanox NICs:

```yaml
apiVersion: sriovnetwork.openshift.io/v1
kind: SriovNetworkNodePolicy
metadata:
  name: dpdk-nic-1
  namespace: openshift-sriov-network-operator
spec:
  deviceType: netdevice
  isRdma: true
  nicSelector:
    rootDevices:
      - "0000:5e:00.1"
  nodeSelector:
    node-role.kubernetes.io/worker-cnf: ""
  numVfs: 5
  priority: 99
  resourceName: dpdk_nic_1
---
apiVersion: sriovnetwork.openshift.io/v1
kind: SriovNetworkNodePolicy
metadata:
  name: dpdk-nic-2
  namespace: openshift-sriov-network-operator
spec:
  deviceType: netdevice
  isRdma: true
  nicSelector:
    rootDevices:
      - "0000:5e:00.0"
  nodeSelector:
    node-role.kubernetes.io/worker-cnf: ""
  numVfs: 5
  priority: 99
  resourceName: dpdk_nic_2
```

where:

`spec.deviceType`
: For Mellanox devices the `deviceType` must be `netdevice`.

`spec.isRdma`
: For Mellanox devices `isRdma` must be `true`. Mellanox cards are connected to DPDK applications using Flow Bifurcation. This mechanism splits traffic between Linux user space and kernel space, and can enhance line rate processing capability.

### Example SR-IOV network operator {#nw-sriov-create-object_using-dpdk-and-rdma}

The following is an example definition of an `sriovNetwork` object. In this case, Intel and Mellanox configurations are identical:

```yaml
apiVersion: sriovnetwork.openshift.io/v1
kind: SriovNetwork
metadata:
  name: dpdk-network-1
  namespace: openshift-sriov-network-operator
spec:
  ipam: '{"type": "host-local","ranges": [[{"subnet": "10.0.1.0/24"}]],"dataDir":
   "/run/my-orchestrator/container-ipam-state-1"}'
  networkNamespace: dpdk-test
  spoofChk: "off"
  trust: "on"
  resourceName: dpdk_nic_1
---
apiVersion: sriovnetwork.openshift.io/v1
kind: SriovNetwork
metadata:
  name: dpdk-network-2
  namespace: openshift-sriov-network-operator
spec:
  ipam: '{"type": "host-local","ranges": [[{"subnet": "10.0.2.0/24"}]],"dataDir":
   "/run/my-orchestrator/container-ipam-state-1"}'
  networkNamespace: dpdk-test
  spoofChk: "off"
  trust: "on"
  resourceName: dpdk_nic_2
```

- You can use a different IP Address Management (IPAM) implementation, such as Whereabouts. For more information, see *Dynamic IP address assignment configuration with Whereabouts*.
- You must request the `networkNamespace` where the network attachment definition will be created. You must create the `sriovNetwork` CR under the `openshift-sriov-network-operator` namespace.
- The `resourceName` value must match that of the `resourceName` created under the `sriovNetworkNodePolicy`.

### Example DPDK base workload {#nw-sriov-dpdk-base-workload_using-dpdk-and-rdma}

The following is an example of a Data Plane Development Kit (DPDK) container:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dpdk-test
---
apiVersion: v1
kind: Pod
metadata:
  annotations:
    k8s.v1.cni.cncf.io/networks: '[
     {
      "name": "dpdk-network-1",
      "namespace": "dpdk-test"
     },
     {
      "name": "dpdk-network-2",
      "namespace": "dpdk-test"
     }
   ]'
    irq-load-balancing.crio.io: "disable"
    cpu-load-balancing.crio.io: "disable"
    cpu-quota.crio.io: "disable"
  labels:
    app: dpdk
  name: testpmd
  namespace: dpdk-test
spec:
  runtimeClassName: performance-performance
  containers:
    - command:
        - /bin/bash
        - -c
        - sleep INF
      image: registry.redhat.io/openshift4/dpdk-base-rhel8
      imagePullPolicy: Always
      name: dpdk
      resources:
        limits:
          cpu: "16"
          hugepages-1Gi: 8Gi
          memory: 2Gi
        requests:
          cpu: "16"
          hugepages-1Gi: 8Gi
          memory: 2Gi
      securityContext:
        capabilities:
          add:
            - IPC_LOCK
            - SYS_RESOURCE
            - NET_RAW
            - NET_ADMIN
        runAsUser: 0
      volumeMounts:
        - mountPath: /mnt/huge
          name: hugepages
  terminationGracePeriodSeconds: 5
  volumes:
    - emptyDir:
        medium: HugePages
      name: hugepages
```

- Request the SR-IOV networks you need. Resources for the devices are injected automatically.
- Disable the CPU and IRQ load balancing base. See *Disabling interrupt processing for individual pods* for more information.
- Set the `runtimeClass` to `performance-performance`. Do not set the `runtimeClass` to `HostNetwork` or `privileged`.
- Request an equal number of resources for requests and limits to start the pod with `Guaranteed` Quality of Service (QoS).

:::note

Do not start the pod with `SLEEP` and then exec into the pod to start the testpmd or the DPDK workload. This can add additional interrupts as the `exec` process is not pinned to any CPU.

:::

### Example testpmd script {#nw-sriov-dpdk-running-testpmd_using-dpdk-and-rdma}

The following is an example script for running `testpmd`:

```terminal
#!/bin/bash
set -ex
export CPU=$(cat /sys/fs/cgroup/cpuset/cpuset.cpus)
echo ${CPU}

dpdk-testpmd -l ${CPU} -a ${PCIDEVICE_OPENSHIFT_IO_DPDK_NIC_1} -a ${PCIDEVICE_OPENSHIFT_IO_DPDK_NIC_2} -n 4 -- -i --nb-cores=15 --rxd=4096 --txd=4096 --rxq=7 --txq=7 --forward-mode=mac --eth-peer=0,50:00:00:00:00:01 --eth-peer=1,50:00:00:00:00:02
```

This example uses two different `sriovNetwork` CRs. The environment variable contains the Virtual Function (VF) PCI address that was allocated for the pod. If you use the same network in the pod definition, you must split the `pciAddress`. It is important to configure the correct MAC addresses of the traffic generator. This example uses custom MAC addresses.

## Using a virtual function in RDMA mode with a Mellanox NIC {#example-vf-use-in-rdma-mode-mellanox_using-dpdk-and-rdma}

:::warning

RDMA over Converged Ethernet (RoCE) is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.

For more information about the support scope of Red Hat Technology Preview features, see [Technology Preview Features Support Scope](https://access.redhat.com/support/offerings/techpreview/).

:::

RDMA over Converged Ethernet (RoCE) is the only supported mode when using RDMA on OpenShift Container Platform.

**Prerequisites**

- Install the OpenShift CLI (`oc`).
- Install the SR-IOV Network Operator.
- Log in as a user with `cluster-admin` privileges.

**Procedure**

1. Create the following `SriovNetworkNodePolicy` object, and then save the YAML in the `mlx-rdma-node-policy.yaml` file.
   ```yaml
   apiVersion: sriovnetwork.openshift.io/v1
   kind: SriovNetworkNodePolicy
   metadata:
     name: mlx-rdma-node-policy
     namespace: openshift-sriov-network-operator
   spec:
     resourceName: mlxnics
     nodeSelector:
       feature.node.kubernetes.io/network-sriov.capable: "true"
     priority: <priority>
     numVfs: <num>
     nicSelector:
       vendor: "15b3"
       deviceID: "1015"
       pfNames: ["<pf_name>", ...]
       rootDevices: ["<pci_bus_id>", "..."]
     deviceType: netdevice
     isRdma: true
   ```

   where:

   <dl>
   <dt><code>spec.nicSelector.deviceID</code></dt>
   <dd>Specifies the device hex code of the SR-IOV network device.</dd>
   <dt><code>spec.deviceType</code></dt>
   <dd>Specifies the driver type for the virtual functions. Set to <code>netdevice</code> for Mellanox NICs.</dd>
   <dt><code>spec.isRdma</code></dt>
   <dd>Set to <code>true</code> to enable RDMA mode.</dd>
   </dl>

   :::note

   See the `Configuring SR-IOV network devices` section for a detailed explanation on each option in `SriovNetworkNodePolicy`.

   When applying the configuration specified in a `SriovNetworkNodePolicy` object, the SR-IOV Operator might drain the nodes, and in some cases, reboot nodes. It might take several minutes for a configuration change to apply. Ensure that there are enough available nodes in your cluster to handle the evicted workload beforehand.

   After the configuration update is applied, all the pods in the `openshift-sriov-network-operator` namespace will change to a `Running` status.

   :::
2. Create the `SriovNetworkNodePolicy` object by running the following command:
   ```terminal
   $ oc create -f mlx-rdma-node-policy.yaml
   ```
3. Create the following `SriovNetwork` object, and then save the YAML in the `mlx-rdma-network.yaml` file.
   ```yaml
   apiVersion: sriovnetwork.openshift.io/v1
   kind: SriovNetwork
   metadata:
     name: mlx-rdma-network
     namespace: openshift-sriov-network-operator
   spec:
     networkNamespace: <target_namespace>
     ipam: |-
   # ...
     vlan: <vlan>
     resourceName: mlxnics
   ```

   where:

   <dl>
   <dt><code>spec.ipam</code></dt>
   <dd>Specifies a configuration object for the IPAM CNI plugin as a YAML block scalar. The plugin manages IP address assignment for the attachment definition.</dd>
   </dl>

   :::note

   See the "Configuring SR-IOV additional network" section for a detailed explanation on each option in `SriovNetwork`.

   :::

   An optional library, app-netutil, provides several API methods for gathering network information about a container’s parent pod.
4. Create the `SriovNetworkNodePolicy` object by running the following command:
   ```terminal
   $ oc create -f mlx-rdma-network.yaml
   ```
5. Create the following `Pod` spec, and then save the YAML in the `mlx-rdma-pod.yaml` file.
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: rdma-app
     namespace: <target_namespace>
     annotations:
       k8s.v1.cni.cncf.io/networks: mlx-rdma-network
   spec:
     containers:
     - name: testpmd
       image: <RDMA_image>
       securityContext:
         runAsUser: 0
         capabilities:
           add: ["IPC_LOCK","SYS_RESOURCE","NET_RAW"]
       volumeMounts:
       - mountPath: /mnt/huge
         name: hugepage
       resources:
         limits:
           memory: "1Gi"
           cpu: "4"
           hugepages-1Gi: "4Gi"
         requests:
           memory: "1Gi"
           cpu: "4"
           hugepages-1Gi: "4Gi"
       command: ["sleep", "infinity"]
     volumes:
     - name: hugepage
       emptyDir:
         medium: HugePages
   ```

   where:

   <dl>
   <dt><code>metadata.namespace</code></dt>
   <dd>Specifies the same namespace where <code>SriovNetwork</code> object <code>mlx-rdma-network</code> is created. If you want to create the pod in a different namespace, change <code>target_namespace</code> in both the <code>Pod</code> spec and the <code>SriovNetwork</code> object.</dd>
   <dt><code>spec.containers.image</code></dt>
   <dd>Specifies the RDMA image which includes your application and the RDMA library used by the application.</dd>
   <dt><code>spec.containers.securityContext.capabilities.add</code></dt>
   <dd>Specifies additional capabilities required by the application inside the container for hugepage allocation, system resource allocation, and network interface access.</dd>
   <dt><code>spec.containers.volumeMounts.mountPath</code></dt>
   <dd>Specifies the path where the hugepage volume is mounted in the RDMA pod. The hugepage volume is backed by the <code>emptyDir</code> volume type with the medium being <code>HugePages</code>.</dd>
   <dt><code>spec.containers.resources.limits.cpu</code></dt>
   <dd>Specifies the number of CPUs. The RDMA pod usually requires exclusive CPUs be allocated from the kubelet. This is achieved by setting CPU Manager policy to <code>static</code> and creating a pod with <code>Guaranteed</code> QoS.</dd>
   <dt><code>spec.containers.resources.limits.hugepages-1Gi</code></dt>
   <dd>Specifies the hugepage size (<code>hugepages-1Gi</code> or <code>hugepages-2Mi</code>) and the quantity of hugepages that will be allocated to the RDMA pod. Configure <code>2Mi</code> and <code>1Gi</code> hugepages separately. Configuring <code>1Gi</code> hugepage requires adding kernel arguments to Nodes.</dd>
   </dl>
6. Create the RDMA pod by running the following command:
   ```terminal
   $ oc create -f mlx-rdma-pod.yaml
   ```

## A test pod template for clusters that use OVS-DPDK on OpenStack {#nw-openstack-ovs-dpdk-testpmd-pod_using-dpdk-and-rdma}

The following `testpmd` pod demonstrates container creation with huge pages, reserved CPUs, and the SR-IOV port.

```yaml title="An example testpmd pod"
apiVersion: v1
kind: Pod
metadata:
  name: testpmd-dpdk
  namespace: mynamespace
  annotations:
    cpu-load-balancing.crio.io: "disable"
    cpu-quota.crio.io: "disable"
# ...
spec:
  containers:
  - name: testpmd
    command: ["sleep", "99999"]
    image: registry.redhat.io/openshift4/dpdk-base-rhel8:v4.9
    securityContext:
      capabilities:
        add: ["IPC_LOCK","SYS_ADMIN"]
      privileged: true
      runAsUser: 0
    resources:
      requests:
        memory: 1000Mi
        hugepages-1Gi: 1Gi
        cpu: '2'
        openshift.io/dpdk1: 1
      limits:
        hugepages-1Gi: 1Gi
        cpu: '2'
        memory: 1000Mi
        openshift.io/dpdk1: 1
    volumeMounts:
      - mountPath: /mnt/huge
        name: hugepage
        readOnly: False
  runtimeClassName: performance-cnf-performanceprofile
  volumes:
  - name: hugepage
    emptyDir:
      medium: HugePages
```

- The name `dpdk1` in this example is a user-created `SriovNetworkNodePolicy` resource. You can substitute this name for that of a resource that you create.
- If your performance profile is not named `cnf-performance profile`, replace that string with the correct performance profile name.

**Additional resources**

- [Red Hat certified hardware (Red Hat Ecosystem Catalog)](https://catalog.redhat.com/en/hardware)
- [Configuring a cluster for RDMA in Red Hat OpenShift AI](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/latest/html/managing_openshift_ai/managing-distributed-workloads_managing-rhoai#configuring-a-cluster-for-rdma_managing-rhoai)
- [Creating a performance profile](/docs/scalability_and_performance/cnf-tuning-low-latency-nodes-with-perf-profile#cnf-create-performance-profiles_cnf-tuning-low-latency-nodes-with-perf-profile)
- [Adjusting the NIC queues with the performance profile](/docs/scalability_and_performance/cnf-tuning-low-latency-nodes-with-perf-profile#adjusting-nic-queues-with-the-performance-profile_cnf-tuning-low-latency-nodes-with-perf-profile)
- [Provisioning real-time and low latency workloads](/docs/scalability_and_performance/cnf-provisioning-low-latency-workloads#cnf-provisioning-low-latency-workloads)
- [Installing the SR-IOV Network Operator](/docs/networking/networking_operators/sr-iov-operator/installing-sriov-operator#installing-sriov-operator)
- [Configuring an SR-IOV network device](/docs/networking/hardware_networks/configuring-sriov-device#nw-sriov-networknodepolicy-object_configuring-sriov-device)
- [Dynamic IP address assignment configuration with Whereabouts](/docs/networking/multiple_networks/secondary_networks/configuring-ip-secondary-nwt#nw-multus-whereabouts_configuring-additional-network)
- [Disabling interrupt processing for individual pods](/docs/scalability_and_performance/cnf-provisioning-low-latency-workloads#disabling-interrupt-processing-for-individual-pods_cnf-provisioning-low-latency)
- [Configuring an SR-IOV Ethernet network attachment](/docs/networking/hardware_networks/configuring-sriov-net-attach#configuring-sriov-net-attach)
