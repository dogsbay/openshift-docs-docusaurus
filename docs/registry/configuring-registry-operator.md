---
title: Image Registry Operator in OpenShift Container Platform
sidebar_position: 2
---

# Image Registry Operator in OpenShift Container Platform {#configuring-registry-operator}

<a id="configuring-registry-operator"></a>

## Image Registry on cloud platforms and OpenStack {#image-registry-on-cloud}

The Image Registry Operator installs a single instance of the OpenShift image registry and manages all registry configuration, including setting up registry storage.

:::note

Storage is only automatically configured when you install an installer-provisioned infrastructure cluster on AWS, Azure, Google Cloud, IBM(R), or RHOSP.

When you install or upgrade an installer-provisioned infrastructure cluster on AWS, Azure, Google Cloud, IBM(R), or RHOSP, the Image Registry Operator sets the `spec.storage.managementState` parameter to `Managed`. If the `spec.storage.managementState` parameter is set to `Unmanaged`, the Image Registry Operator takes no action related to storage.

:::

After the control plane deploys in the management cluster, the Operator creates a default `configs.imageregistry.operator.openshift.io` custom resource (CR) instance based on configuration detected in the cluster.

If insufficient information is available to define a complete `configs.imageregistry.operator.openshift.io` CR, the incomplete resource is defined and the Operator updates the resource status with information about what is missing.

:::warning

The Image Registry Operator’s behavior for managing the pruner is orthogonal to the `managementState` specified on the `ClusterOperator` object for the Image Registry Operator. If the Image Registry Operator is not in the `Managed` state, the image pruner can still be configured and managed by the `Pruning` custom resource.

However, the `managementState` of the Image Registry Operator alters the behavior of the deployed image pruner job:

- `Managed`: the `--prune-registry` flag for the image pruner is set to `true`.
- `Removed`: the `--prune-registry` flag for the image pruner is set to `false`, meaning the image pruner job only prunes image metadata in etcd.

:::

## Image Registry on bare metal, Nutanix, and vSphere {#image-registry-on-bare-metal-vsphere}

### Image registry removed during installation {#registry-removed_configuring-registry-operator}

On platforms that do not provide shareable object storage, the OpenShift Image Registry Operator bootstraps itself as `Removed`. This allows `openshift-installer` to complete installations on these platform types.

After installation, you must edit the Image Registry Operator configuration to switch the `managementState` from `Removed` to `Managed`. When this has completed, you must configure storage.

## Image Registry Operator distribution across availability zones {#registry-operator-distribution-across-availability-zones_configuring-registry-operator}

The default configuration of the Image Registry Operator spreads image registry pods across topology zones to prevent delayed recovery times in case of a complete zone failure where all pods are impacted. Reference the example YAML to understand the default parameter values that the Image Registry Operator uses when the Operator deploys with a zone-related topology constraint:

```yaml
  topologySpreadConstraints:
  - labelSelector:
      matchLabels:
        docker-registry: default
    maxSkew: 1
    topologyKey: kubernetes.io/hostname
    whenUnsatisfiable: DoNotSchedule
  - labelSelector:
      matchLabels:
        docker-registry: default
    maxSkew: 1
    topologyKey: node-role.kubernetes.io/worker
    whenUnsatisfiable: DoNotSchedule
  - labelSelector:
      matchLabels:
        docker-registry: default
    maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule
```

Reference the following YAML to understand the default parameter value that the Image Registry Operator uses when the Operator deploys with a zone-related topology constraint, which applies to bare metal and vSphere instances:

```yaml
 topologySpreadConstraints:
  - labelSelector:
      matchLabels:
        docker-registry: default
    maxSkew: 1
    topologyKey: kubernetes.io/hostname
    whenUnsatisfiable: DoNotSchedule
  - labelSelector:
      matchLabels:
        docker-registry: default
    maxSkew: 1
    topologyKey: node-role.kubernetes.io/worker
    whenUnsatisfiable: DoNotSchedule
```

As a cluster administrator. you can override the default `topologySpreadConstraints` section values by configuring the `configs.imageregistry.operator.openshift.io/cluster` spec file.

**Additional resources**

- [Configuring pod topology spread constraints](/docs/nodes/scheduling/nodes-scheduler-pod-topology-spread-constraints#nodes-scheduler-pod-topology-spread-constraints)

## Image Registry Operator configuration parameters {#registry-operator-configuration-resource-overview_configuring-registry-operator}

You can configure the Image Registry Operator using the `configs.imageregistry.operator.openshift.io` resource. The resource provides parameters for managing registry state, storage, logging, routing, and deployment settings.

<table>
<thead>
<tr>
  <th>Parameter</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>managementState</code></td>
  <td><code>Managed</code>: The Operator updates the registry as configuration resources are updated.<br><br><code>Unmanaged</code>: The Operator ignores changes to the configuration resources.<br><br><code>Removed</code>: The Operator removes the registry instance and tear down any storage that the Operator provisioned.</td>
</tr>
<tr>
  <td><code>logLevel</code></td>
  <td>Sets <code>logLevel</code> of the registry instance. Defaults to  <code>Normal</code>.<br><br>The following values for <code>logLevel</code> are supported:<br><br><ul><li><code>Normal</code></li><li><code>Debug</code></li><li><code>Trace</code></li><li><code>TraceAll</code></li></ul></td>
</tr>
<tr>
  <td><code>httpSecret</code></td>
  <td>Value needed by the registry to secure uploads, generated by default.</td>
</tr>
<tr>
  <td><code>operatorLogLevel</code></td>
  <td>The <code>operatorLogLevel</code> configuration parameter provides intent-based logging for the Operator itself and a simple way to manage coarse-grained logging choices that Operators must interpret for themselves. This configuration parameter defaults to <code>Normal</code>. It does not provide fine-grained control.<br><br>The following values for <code>operatorLogLevel</code> are supported:<br><br><ul><li><code>Normal</code></li><li><code>Debug</code></li><li><code>Trace</code></li><li><code>TraceAll</code></li></ul></td>
</tr>
<tr>
  <td><code>proxy</code></td>
  <td>Defines the Proxy to be used when calling master API and upstream registries.</td>
</tr>
<tr>
  <td><code>affinity</code></td>
  <td>You can use the <code>affinity</code> parameter to configure pod scheduling preferences and constraints for Image Registry Operator pods.<br><br>Affinity settings can use the <code>podAffinity</code> or <code>podAntiAffinity</code> spec. Both options can use either <code>preferredDuringSchedulingIgnoredDuringExecution</code> rules or <code>requiredDuringSchedulingIgnoredDuringExecution</code> rules.</td>
</tr>
<tr>
  <td><code>storage</code></td>
  <td><code>Storagetype</code>: Details for configuring registry storage, for example S3 bucket coordinates. Normally configured by default.</td>
</tr>
<tr>
  <td><code>readOnly</code></td>
  <td>Indicates whether the registry instance should reject attempts to push new images or delete existing ones.</td>
</tr>
<tr>
  <td><code>requests</code></td>
  <td>API Request Limit details. Controls how many parallel requests a given registry instance will handle before queuing additional requests.</td>
</tr>
<tr>
  <td><code>defaultRoute</code></td>
  <td>Determines whether or not an external route is defined using the default hostname. If enabled, the route uses re-encrypt encryption. Defaults to <code>false</code>.</td>
</tr>
<tr>
  <td><code>routes</code></td>
  <td>Array of additional routes to create. You provide the hostname and certificate for the route.</td>
</tr>
<tr>
  <td><code>rolloutStrategy</code></td>
  <td>Defines rollout strategy for the image registry deployment. Defaults to <code>RollingUpdate</code>.</td>
</tr>
<tr>
  <td><code>replicas</code></td>
  <td>Replica count for the registry.</td>
</tr>
<tr>
  <td><code>disableRedirect</code></td>
  <td>Controls whether to route all data through the registry, rather than redirecting to the back end. Defaults to <code>false</code>.</td>
</tr>
<tr>
  <td><code>spec.storage.managementState</code></td>
  <td> The Image Registry Operator sets the <code>spec.storage.managementState</code> parameter to <code>Managed</code> on new installations or upgrades of clusters using installer-provisioned infrastructure on AWS or Azure.  <br><br>  <br><br><ul><li><code>Managed</code>: Determines that the Image Registry Operator manages underlying storage. If the Image Registry Operator's <code>managementState</code> is set to <code>Removed</code>, then the storage is deleted.<ul><li>If the <code>managementState</code> is set to <code>Managed</code>, the Image Registry Operator attempts to apply some default configuration on the underlying storage unit. For example, if set to <code>Managed</code>, the Operator tries to enable encryption on the S3 bucket before making it available to the registry. If you do not want the default settings to be applied on the storage you are providing, make sure the <code>managementState</code> is set to <code>Unmanaged</code>.</li></ul></li><li><code>Unmanaged</code>: Determines that the Image Registry Operator ignores the storage settings. If the Image Registry Operator's <code>managementState</code> is set to <code>Removed</code>, then the storage is not deleted. If you provided an underlying storage unit configuration, such as a bucket or container name, and the <code>spec.storage.managementState</code> is not yet set to any value, then the Image Registry Operator configures it to <code>Unmanaged</code>.</li></ul></td>
</tr>
</tbody>
</table>

## Enabling the Image Registry default route by using a CRD {#registry-operator-default-crd_configuring-registry-operator}

In OpenShift Container Platform, the `Registry` Operator controls the OpenShift image registry feature and you define this Operator in the `configs.imageregistry.operator.openshift.io` Custom Resource Definition (CRD). If you need to automatically enable the Image Registry default route, patch the Image Registry Operator CRD.

**Procedure**

- Patch the Image Registry Operator CRD:
  ```terminal
  $ oc patch configs.imageregistry.operator.openshift.io/cluster --type merge -p '{"spec":{"defaultRoute":true}}'
  ```

## Configuring additional trust stores for image registry access {#images-configuration-cas_configuring-registry-operator}

You can add references to a config map that has additional certificate authorities (CAs) to be trusted during image registry access to the `image.config.openshift.io/cluster` custom resource (CR).

**Prerequisites**

- The certificate authorities (CAs) must be PEM-encoded.

**Procedure**

1. Create a config map in the `openshift-config` namespace, then and use the config map name in the `AdditionalTrustedCA` parameter of the `image.config.openshift.io` CR. This adds CAs that should be trusted when the cluster contacts external image registries.
   ```yaml title="Image registry CA config map example"
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: my-registry-ca
   data:
     registry.example.com: |
       -----BEGIN CERTIFICATE-----
       ...
       -----END CERTIFICATE-----
     registry-with-port.example.com..5000: |
       -----BEGIN CERTIFICATE-----
       ...
       -----END CERTIFICATE-----
   ```

   where:

   <dl>
   <dt><code>data:registry.example.com:</code></dt>
   <dd>An example hostname of a registry for which this CA is to be trusted.</dd>
   <dt><code>data:registry-with-port.example.com..5000:</code></dt>
   <dd>An example hostname of a registry with the port for which this CA is to be trusted. If the registry has a port, such as <code>registry-with-port.example.com:5000</code>, <code>:</code> must be replaced with <code>..</code>. The PEM certificate content is the value for each additional registry CA to trust.</dd>
   </dl>
2. Optional. Configure an additional CA by running the following command:
   ```terminal
   $ oc create configmap registry-config --from-file=<external_registry_address>=ca.crt -n openshift-config
   ```

   ```terminal
   $ oc edit image.config.openshift.io cluster
   ```

   ```yaml
   spec:
     additionalTrustedCA:
       name: registry-config
   ```

## Configuring storage credentials for the Image Registry Operator {#registry-operator-config-resources-storage-credentials_configuring-registry-operator}

In addition to the `configs.imageregistry.operator.openshift.io` Custom Resource (CR) and ConfigMap resources, storage credential configuration is provided to the Operator by a separate secret resource. This resource is located within the `openshift-image-registry` namespace.

You can create an `image-registry-private-configuration-user` secret that in turn creates custom credentials needed for storage access and management. If default credentials exist, the custom credentials override the default credentials used by the Operator.

**Procedure**

- Create an OpenShift Container Platform secret that contains the required keys.
  ```terminal
  $ oc create secret generic image-registry-private-configuration-user --from-literal=KEY1=value1 --from-literal=KEY2=value2 --namespace openshift-image-registry
  ```

**Additional resources**

- [Configuring the registry for AWS user-provisioned infrastructure](/docs/registry/configuring_registry_storage/configuring-registry-storage-aws-user-infrastructure#configuring-registry-storage-aws-user-infrastructure)
- [Configuring the registry for Google Cloud user-provisioned infrastructure](/docs/registry/configuring_registry_storage/configuring-registry-storage-gcp-user-infrastructure#configuring-registry-storage-gcp-user-infrastructure)
- [Configuring the registry for Azure user-provisioned infrastructure](/docs/registry/configuring_registry_storage/configuring-registry-storage-azure-user-infrastructure#configuring-registry-storage-azure-user-infrastructure)
- [Configuring the registry for bare metal](/docs/registry/configuring_registry_storage/configuring-registry-storage-baremetal#configuring-registry-storage-baremetal)
- [Configuring the registry for vSphere](/docs/registry/configuring_registry_storage/configuring-registry-storage-vsphere#configuring-registry-storage-vsphere)
- [Configuring the registry for RHOSP](/docs/registry/configuring_registry_storage/configuring-registry-storage-osp#configuring-registry-storage-openstack)
- [Configuring the registry for Red Hat OpenShift Data Foundation](/docs/registry/configuring_registry_storage/configuring-registry-storage-rhodf#configuring-registry-storage-rhodf)
- [Configuring the registry for Nutanix](/docs/registry/configuring_registry_storage/configuring-registry-storage-nutanix#configuring-registry-storage-nutanix)
