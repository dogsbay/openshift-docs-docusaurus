---
title: Persistent storage using Cinder
sidebar_position: 4
---

# Persistent storage using Cinder {#persistent-storage-cinder}

<a id="persistent-storage-cinder"></a>

OpenShift Container Platform supports OpenStack Cinder volumes. You can provision your OpenShift Container Platform cluster with persistent storage using OpenStack Cinder. Some familiarity with Kubernetes and OpenStack is assumed.

Persistent volumes are not bound to a single project or namespace; they can be shared across the OpenShift Container Platform cluster. Persistent volume claims are specific to a project or namespace and can be requested by users.

:::warning

OpenShift Container Platform 4.11 and later provides automatic migration for the Cinder in-tree volume plugin to its equivalent CSI driver.

CSI automatic migration should be seamless. Migration does not change how you use all existing API objects, such as persistent volumes, persistent volume claims, and storage classes. For more information about migration, see "CSI automatic migration".

:::

**Additional resources**

- [CSI automatic migration](/docs/storage/container_storage_interface/persistent-storage-csi-migration#persistent-storage-csi-migration)
- [OpenStack Cinder](https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/8/html-single/architecture_guide/index#comp-cinder)

## Manual provisioning with Cinder {#persistent-storage-cinder-provisioning_persistent-storage-cinder}

Storage must exist in the underlying infrastructure before it can be mounted as a volume in OpenShift Container Platform.

Manual provisioning requires that OpenShift Container Platform is configured for Red Hat OpenStack Platform (RHOSP) and that you have the Cinder volume ID.

### Creating the persistent volume {#persistent-storage-cinder-creating-pv_persistent-storage-cinder}

You can create a persistent volume (PV) that provisions storage from an Red Hat OpenStack Platform (RHOSP) Cinder volume for use with OpenShift Container Platform.

**Prerequisites**

- You have defined your PV in an object definition before creating it in OpenShift Container Platform.

**Procedure**

1. Save your object definition to a file.
   ```yaml title="cinder-persistentvolume.yaml"
   apiVersion: "v1"
   kind: "PersistentVolume"
   metadata:
     name: "pv0001"
   spec:
     capacity:
       storage: "5Gi"
     accessModes:
       - "ReadWriteOnce"
     cinder:
       fsType: "ext3"
       volumeID: "f37a03aa-6212-4c62-a805-9ce139fab180"
   ```

   where:

   <dl>
   <dt><code>metadata.name</code></dt>
   <dd>Specifies the name of the volume that is used by persistent volume claims or pods.</dd>
   <dt><code>spec.capacity.storage</code></dt>
   <dd>Specifies the amount of storage allocated to this volume.</dd>
   <dt><code>spec.cinder</code></dt>
   <dd>Indicates <code>cinder</code> for Red Hat OpenStack Platform (RHOSP) Cinder volumes.</dd>
   <dt><code>spec.cinder.fsType</code></dt>
   <dd>Specifies the file system that is created when the volume is mounted for the first time.</dd>
   <dt><code>spec.cinder.volumeID</code></dt>
   <dd>Specifies the Cinder volume to use.</dd>
   </dl>

:::warning

Do not change the `fstype` parameter value after the volume is formatted and provisioned. Changing this value can result in data loss and pod failure.

:::

1. Create the object definition file you saved in the previous step.
   ```terminal
   $ oc create -f cinder-persistentvolume.yaml
   ```

### Persistent volume formatting {#persistent-storage-cinder-pv-format_persistent-storage-cinder}

You can use unformatted Cinder volumes as PVs because OpenShift Container Platform formats them before the first use.

Before OpenShift Container Platform mounts the volume and passes it to a container, the system checks that it contains a file system as specified by the `fsType` parameter in the PV definition. If the device is not formatted with the file system, all data from the device is erased and the device is automatically formatted with the given file system.

### Configuring Cinder volume security {#persistent-storage-cinder-volume-security_persistent-storage-cinder}

If you use Cinder PVs in your application, configure security for their deployment resources.

**Prerequisites**

- An SCC must be created that uses the appropriate `fsGroup` strategy.

**Procedure**

1. Create a service account and add it to the SCC:
   ```terminal
   $ oc create serviceaccount <service_account>
   ```

   ```terminal
   $ oc adm policy add-scc-to-user <new_scc> -z <service_account> -n <project>
   ```
2. In your application’s deployment resource, provide the service account name and `securityContext`:
   ```yaml
   apiVersion: v1
   kind: ReplicationController
   metadata:
     name: frontend-1
   spec:
     replicas: 1
     selector:
       name: frontend
     template:
       metadata:
         labels:
           name: frontend
       spec:
         containers:
         - image: openshift/hello-openshift
           name: helloworld
           ports:
           - containerPort: 8080
             protocol: TCP
         restartPolicy: Always
         serviceAccountName: <service_account>
         securityContext:
           fsGroup: 7777
   ```

   where:

   <dl>
   <dt><code>spec.replicas</code></dt>
   <dd>Specifies the number of copies of the pod to run.</dd>
   <dt><code>spec.selector</code></dt>
   <dd>Specifies the label selector of the pod to run.</dd>
   <dt><code>spec.template</code></dt>
   <dd>Specifies a template for the pod that the controller creates.</dd>
   <dt><code>spec.template.metadata.labels</code></dt>
   <dd>Specifies the labels on the pod. They must include labels from the label selector.</dd>
   <dt><code>spec.template.metadata.labels.name</code></dt>
   <dd>Specifies the maximum name length after expanding any parameters is 63 characters.</dd>
   <dt><code>spec.template.spec.serviceAccountName</code></dt>
   <dd>Specifies the service account you created.</dd>
   <dt><code>spec.template.spec.securityContext.fsGroup</code></dt>
   <dd>Specifies an <code>fsGroup</code> for the pods.</dd>
   </dl>
