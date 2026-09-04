---
title: Configuring the registry for RHOSP
sidebar_position: 5
---

# Configuring the registry for RHOSP {#configuring-registry-storage-openstack}

<a id="configuring-registry-storage-openstack"></a>

You configure the image registry to use custom storage on clusters that run on RHOSP. You must configure custom storage to use a Cinder volume in a specific availability zone.

## Configuring an image registry with custom storage on clusters that run on RHOSP {#installation-registry-osp-creating-custom-pvc_configuring-registry-storage-openstack}

After you install a cluster on Red Hat OpenStack Platform (RHOSP), you can use a Cinder volume that is in a specific availability zone for registry storage.

**Procedure**

1. Create a YAML file that specifies the storage class and availability zone to use. For example:
   ```yaml
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: custom-csi-storageclass
   provisioner: cinder.csi.openstack.org
   volumeBindingMode: WaitForFirstConsumer
   allowVolumeExpansion: true
   parameters:
     availability: <availability_zone_name>
   ```

   :::note

   OpenShift Container Platform does not verify the existence of the availability zone you choose. Verify the name of the availability zone before you apply the configuration.

   :::
2. From a command line, apply the configuration:
   ```terminal
   $ oc apply -f <storage_class_file_name>
   ```

   ```terminal title="Example output"
   storageclass.storage.k8s.io/custom-csi-storageclass created
   ```
3. Create a YAML file that specifies a persistent volume claim (PVC) that uses your storage class and the `openshift-image-registry` namespace. For example:
   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: csi-pvc-imageregistry
     namespace: openshift-image-registry
     annotations:
       imageregistry.openshift.io: "true"
   spec:
     accessModes:
     - ReadWriteOnce
     volumeMode: Filesystem
     resources:
       requests:
         storage: 100Gi
     storageClassName: <your_custom_storage_class>
   ```

   where:

   <dl>
   <dt><code>metadata.namespace</code></dt>
   <dd>Specifying the <code>openshift-image-registry</code> namespace allows the Cluster Image Registry Operator to consume the PVC.</dd>
   <dt><code>spec.resources.requests.storage</code></dt>
   <dd>This optional field adjusts the volume size.</dd>
   <dt><code>spec.storageClassName</code></dt>
   <dd>Specifies the name of the storage class that you created.</dd>
   </dl>
4. From a command line, apply the configuration:
   ```terminal
   $ oc apply -f <pvc_file_name>
   ```

   ```terminal title="Example output"
   persistentvolumeclaim/csi-pvc-imageregistry created
   ```
5. Replace the original persistent volume claim in the image registry configuration with the new claim:
   ```terminal
   $ oc patch configs.imageregistry.operator.openshift.io/cluster --type 'json' -p='[{"op": "replace", "path": "/spec/storage/pvc/claim", "value": "csi-pvc-imageregistry"}]'
   ```

   ```terminal title="Example output"
   config.imageregistry.operator.openshift.io/cluster patched
   ```

   Over the next several minutes, the configuration is updated.

**Verification**

To confirm that the registry is using the resources that you defined:

1. Verify that the PVC claim value is identical to the name that you provided in your PVC definition:
   ```terminal
   $ oc get configs.imageregistry.operator.openshift.io/cluster -o yaml
   ```

   ```terminal title="Example output"
   ...
   status:
       ...
       managementState: Managed
       pvc:
         claim: csi-pvc-imageregistry
   ...
   ```
2. Verify that the status of the PVC is `Bound`:
   ```terminal
   $ oc get pvc -n openshift-image-registry csi-pvc-imageregistry
   ```

   ```terminal title="Example output"
   NAME                   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS             AGE
   csi-pvc-imageregistry  Bound    pvc-72a8f9c9-f462-11e8-b6b6-fa163e18b7b5   100Gi      RWO            custom-csi-storageclass  11m
   ```
