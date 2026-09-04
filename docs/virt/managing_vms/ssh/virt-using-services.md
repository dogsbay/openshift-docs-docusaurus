---
title: Create a service to connect with SSH
sidebar_position: 4
---

# Create a service to connect with SSH {#virt-using-services}

<a id="virt-using-services"></a>

You can create a service for a virtual machine (VM) and connect to the IP address and port exposed by the service. Services provide excellent performance and are recommended for applications that are accessed from outside the cluster or within the cluster. Ingress traffic is protected by firewalls.

After you create a service with `virtctl`, you must add `special: key` to the `spec.template.metadata.labels` stanza of the `VirtualMachine` manifest. If the cluster network cannot handle the traffic load, consider using a secondary network for VM access.

## About services {#virt-about-services_virt-using-services}

A Kubernetes service exposes network access for clients to an application running on a set of pods. Services offer abstraction, load balancing, and, in the case of the `NodePort` and `LoadBalancer` types, exposure to the outside world.

`ClusterIP`
: Exposes the service on an internal IP address and as a DNS name to other applications within the cluster. A single service can map to multiple virtual machines. When a client tries to connect to the service, the client’s request is load balanced among available backends. `ClusterIP` is the default service type.

`NodePort`
: Exposes the service on the same port of each selected node in the cluster. `NodePort` makes a port accessible from outside the cluster, provided that the node itself is externally accessible to the client.

`LoadBalancer`
: Creates an external load balancer in the current cloud (if supported) and assigns a fixed, external IP address to the service.

:::note

For on-premise clusters, you can configure a load balancing service by deploying the MetalLB Operator.

:::

## Enabling load balancer service creation by using the web console {#virt-enabling-load-balancer-service-web_virt-using-services}

You can enable the creation of load balancer services for a virtual machine (VM) by using the OpenShift Container Platform web console.

**Prerequisites**

- You have configured a load balancer for the cluster.
- You have logged in as a user with the `cluster-admin` role.
- You created a network attachment definition for the network.

**Procedure**

1. Go to **Virtualization** → **Settings**.
2. Click **Cluster**.
3. Expand **General settings** and **SSH configuration**.
4. Set **SSH over LoadBalancer service** to on.

## Create a service with the web console {#virt-creating-service-web_virt-using-services}

You can create a node port or load balancer service for a virtual machine (VM) by using the OpenShift Container Platform web console.

**Prerequisites**

- You configured the cluster network to support either a load balancer or a node port.
- To create a load balancer service, you enabled the creation of load balancer services.

**Procedure**

1. Navigate to **VirtualMachines** and select a virtual machine to view the **VirtualMachine details** page.
2. On the **Details** tab, select **SSH over LoadBalancer** from the **SSH service type** list.
3. Optional: Click the copy icon to copy the `SSH` command to your clipboard.

**Verification**

- Check the **Services** pane on the **Details** tab to view the new service.

## Create a service with virtctl {#virt-creating-service-virtctl_virt-using-services}

You can create a service for a virtual machine (VM) by using the `virtctl` command-line tool.

**Prerequisites**

- You installed the `virtctl` command-line tool.
- You configured the cluster network to support the service.
- The environment where you installed `virtctl` has the cluster permissions required to access the VM. For example, you ran `oc login` or you set the `KUBECONFIG` environment variable.

**Procedure**

- Create a service by running the following command:
  ```terminal
  $ virtctl expose vm <vm_name> --name <service_name> --type <service_type> --port <port>
  ```

  where:

  <dl>
  <dt><code>&lt;vm_name&gt;</code></dt>
  <dd>Specifies the name of the VM you are exposing.</dd>
  <dt><code>&lt;service_name&gt;</code></dt>
  <dd>Specifies a user-defined name for the service you are creating.</dd>
  <dt><code>&lt;service_type&gt;</code></dt>
  <dd>Specifies one of <code>ClusterIP</code>, <code>NodePort</code>, or <code>LoadBalancer</code>.</dd>
  <dt><code>&lt;port&gt;</code></dt>
  <dd>Specifies the network port on the VM that the service will expose. Example:</dd>
  </dl>

  ```terminal
  $ virtctl expose vm example-vm --name example-service --type NodePort --port 22
  ```

**Verification**

- Verify the service by running the following command:
  ```terminal
  $ oc get service
  ```

## Creating a service by using the CLI {#virt-creating-service-cli_virt-using-services}

You can create a service and associate it with a virtual machine (VM) by using the command line.

**Prerequisites**

- You configured the cluster network to support the service.
- You have installed the OpenShift CLI (`oc`).

**Procedure**

1. Edit the `VirtualMachine` manifest to add the label for service creation. Add `special: key` to the `spec.template.metadata.labels` stanza:
   ```yaml
   apiVersion: kubevirt.io/v1
   kind: VirtualMachine
   metadata:
     name: example-vm
     namespace: example-namespace
   spec:
     runStrategy: Halted
     template:
       metadata:
         labels:
           special: key
   # ...
   ```

   :::note

   Labels on a virtual machine pass through to the pod. The `special: key` label must match the label in the `spec.selector` attribute of the `Service` manifest.

   :::
2. Save the `VirtualMachine` manifest file to apply your changes.
3. Create a `Service` manifest to expose the VM:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: example-service
     namespace: example-namespace
   spec:
   # ...
     selector:
       special: key
     type: NodePort
     ports:
       protocol: TCP
       port: 80
       targetPort: 9376
       nodePort: 30000
   ```

   - `spec.selector` defines the label that you added to the `spec.template.metadata.labels` stanza of the `VirtualMachine` manifest.
   - `spec.type` defines the type of service by the way it is exposed. Choose one of `ClusterIP`, `NodePort`, or `LoadBalancer`.
   - `spec.ports` defines a collection of network ports and protocols to expose from the virtual machine.
4. Save the `Service` manifest file.
5. Create the service by running the following command:
   ```terminal
   $ oc create -f example-service.yaml
   ```
6. Restart the VM to apply the changes.

**Verification**

- Query the `Service` object to verify that it is available:
  ```terminal
  $ oc get service -n example-namespace
  ```

## Connecting to a VM exposed by a service by using SSH {#virt-connecting-service-ssh_virt-using-services}

You can connect to a virtual machine (VM) that a service exposes by using SSH.

**Prerequisites**

- You created a service to expose the VM.
- You have an SSH client installed.
- You are logged in to the cluster.

**Procedure**

- Run the following command to access the VM:
  ```terminal
  $ ssh <user_name>@<ip_address> -p <port>
  ```

  where:

  <dl>
  <dt><code>&lt;ip_address&gt;</code></dt>
  <dd>Specifies the cluster IP for a cluster IP service, the node IP for a node port service, or the external IP address for a load balancer service.</dd>
  </dl>
