---
title: Enabling TLS security profiles for the kubelet
sidebar_position: 15
---

# Enabling TLS security profiles for the kubelet {#nodes-nodes-tls}

<a id="nodes-nodes-tls"></a>

You can use a TLS (Transport Layer Security) security profile to define which TLS ciphers are required by the kubelet when it is acting as an HTTP server. The kubelet uses its HTTP/GRPC server to communicate with the Kubernetes API server, which sends commands to pods, gathers logs, and run exec commands on pods through the kubelet.

A TLS security profile defines the TLS ciphers that the Kubernetes API server must use when connecting with the kubelet to protect communication between the kubelet and the Kubernetes API server.

:::note

By default, when the kubelet acts as a client with the Kubernetes API server, it automatically negotiates the TLS parameters with the API server.

:::

## Understanding TLS security profiles {#tls-profiles-understanding_nodes-nodes-tls}

You can use a TLS (Transport Layer Security) security profile, as described in this section, to define which TLS ciphers are required by various OpenShift Container Platform components.

The OpenShift Container Platform TLS security profiles are based on [Mozilla recommended configurations](https://wiki.mozilla.org/Security/Server_Side_TLS).

You can specify one of the following TLS security profiles for each component:

**TLS security profiles**

<table>
<thead>
<tr>
  <th>Profile</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>Old</code></td>
  <td>This profile is intended for use with legacy clients or libraries. The profile is based on the <a href="https://wiki.mozilla.org/Security/Server_Side_TLS#Old_backward_compatibility">Old backward compatibility</a> recommended configuration.<br><br>The <code>Old</code> profile requires a minimum TLS version of 1.0.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>For the Ingress Controller, the minimum TLS version is converted from 1.0 to 1.1.</p></div></td>
</tr>
<tr>
  <td><code>Intermediate</code></td>
  <td>This profile is the default TLS security profile for the Ingress Controller, kubelet, and control plane. The profile is based on the <a href="https://wiki.mozilla.org/Security/Server_Side_TLS#Intermediate_compatibility_.28recommended.29">Intermediate compatibility</a> recommended configuration.<br><br>The <code>Intermediate</code> profile requires a minimum TLS version of 1.2.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>This profile is the recommended configuration for the majority of clients.</p></div></td>
</tr>
<tr>
  <td><code>Modern</code></td>
  <td>This profile is intended for use with modern clients that have no need for backwards compatibility. This profile is based on the <a href="https://wiki.mozilla.org/Security/Server_Side_TLS#Modern_compatibility">Modern compatibility</a> recommended configuration.<br><br>The <code>Modern</code> profile requires a minimum TLS version of 1.3.</td>
</tr>
<tr>
  <td><code>Custom</code></td>
  <td>This profile allows you to define the TLS version and ciphers to use.<br><br><div class="alert alert--warning"><strong>Warning</strong><p>Use caution when using a <code>Custom</code> profile, because invalid configurations can cause problems.</p></div></td>
</tr>
</tbody>
</table>

:::note

When using one of the predefined profile types, the effective profile configuration is subject to change between releases. For example, given a specification to use the Intermediate profile deployed on release X.Y.Z, an upgrade to release X.Y.Z+1 might cause a new profile configuration to be applied, resulting in a rollout.

:::

## Configuring the TLS security profile for the kubelet {#tls-profiles-kubelet-configuring_nodes-nodes-tls}

To configure TLS ciphers and minimum versions for the kubelet HTTP server in OpenShift Container Platform, apply a predefined or custom TLS security profile through a `KubeletConfig` custom resource (CR). Without a custom profile, the kubelet defaults to the `Intermediate` profile.

```yaml title="Sample KubeletConfig CR that configures the Old TLS security profile on worker nodes"
apiVersion: machineconfiguration.openshift.io/v1
kind: KubeletConfig
# ...
spec:
  tlsSecurityProfile:
    old: {}
    type: Old
  machineConfigPoolSelector:
    matchLabels:
      pools.operator.machineconfiguration.openshift.io/worker: ""
# ...
```

You can see the ciphers and the minimum TLS version of the configured TLS security profile in the `kubelet.conf` file on a configured node.

**Prerequisites**

- You are logged in to OpenShift Container Platform as a user with the `cluster-admin` role.

**Procedure**

1. Create a `KubeletConfig` CR to configure the TLS security profile:
   ```yaml title="Sample KubeletConfig CR for a Custom profile"
   apiVersion: machineconfiguration.openshift.io/v1
   kind: KubeletConfig
   metadata:
     name: set-kubelet-tls-security-profile
   spec:
     tlsSecurityProfile:
       type: Custom
       custom:
         ciphers:
         - ECDHE-ECDSA-CHACHA20-POLY1305
         - ECDHE-RSA-CHACHA20-POLY1305
         - ECDHE-RSA-AES128-GCM-SHA256
         - ECDHE-ECDSA-AES128-GCM-SHA256
         minTLSVersion: VersionTLS11
     machineConfigPoolSelector:
       matchLabels:
         pools.operator.machineconfiguration.openshift.io/worker: ""
   #...
   ```

   where:

   <dl>
   <dt><code>spec.tlsSecurityProfile.type</code></dt>
   <dd>Specifies the TLS security profile type (<code>Old</code>, <code>Intermediate</code>, or <code>Custom</code>). The default is <code>Intermediate</code>.</dd>
   <dt><code>spec.tlsSecurityProfile.type.custom</code></dt>
   <dd>Specifies the appropriate field for the selected type: *   <code>old: {}</code> *   <code>intermediate: {}</code> *   <code>modern: {}</code> *   <code>custom:</code></dd>
   <dt><code>spec.tlsSecurityProfile.type.custom</code></dt>
   <dd>For the <code>custom</code> type, specifies a list of TLS ciphers and the minimum accepted TLS version.</dd>
   <dt><code>spec.machineConfigPoolSelector.matchLabels.custom</code></dt>
   <dd>Specifies the machine config pool label for the nodes you want to apply the TLS security profile. This parameter is optional.</dd>
   </dl>
2. Create the `KubeletConfig` object:
   ```terminal
   $ oc create -f <filename>
   ```

   Depending on the number of worker nodes in the cluster, wait for the configured nodes to be rebooted one by one.

**Verification**

To verify that the profile is set,  perform the following steps after the nodes are in the `Ready` state:

1. Start a debug session for a configured node:
   ```terminal
   $ oc debug node/<node_name>
   ```
2. Set `/host` as the root directory within the debug shell:
   ```terminal
   sh-4.4# chroot /host
   ```
3. View the `kubelet.conf` file:
   ```terminal
   sh-4.4# cat /etc/kubernetes/kubelet.conf
   ```

   ```terminal title="Example output"
     "kind": "KubeletConfiguration",
     "apiVersion": "kubelet.config.k8s.io/v1beta1",
   #...
     "tlsCipherSuites": [
       "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256",
       "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256",
       "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384",
       "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384",
       "TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256",
       "TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256"
     ],
     "tlsMinVersion": "VersionTLS12",
   #...
   ```
