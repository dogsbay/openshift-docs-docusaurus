---
title: Pods crash or restart due to lack of memory or CPU
sidebar_position: 3
---

# Pods crash or restart due to lack of memory or CPU

<a id="pods-crash-or-restart-due-to-lack-of-memory-or-cpu"></a>

Resolve Velero or Restic pod crashes caused by insufficient memory or CPU by configuring resource requests in the `DataProtectionApplication` custom resource (CR). This helps you allocate adequate CPU and memory resources to prevent pod restarts and ensure stable backup and restore operations.

Ensure that the values for the resource request fields follow the same format as Kubernetes resource requirements.

If you do not specify `configuration.velero.podConfig.resourceAllocations` or `configuration.restic.podConfig.resourceAllocations`, see the following default `resources` specification configuration for a Velero or Restic pod:

```yaml
requests:
  cpu: 500m
  memory: 128Mi
```

**Additional resources**

- [Velero CPU and memory requirements based on collected data](/docs/backup_and_restore/application_backup_and_restore/installing/about-installing-oadp#oadp-velero-cpu-memory-requirements_about-installing-oadp)

## Setting resource requests for a Velero pod {#oadp-pod-crash-resource-request-velero_pods-crash-or-restart-due-to-lack-of-memory-or-cpu}

Use the `configuration.velero.podConfig.resourceAllocations` specification field in the `oadp_v1alpha1_dpa.yaml` file to set specific resource requests for a `Velero` pod.

**Procedure**

- Set the `cpu` and `memory` resource requests as shown in the following example:
  ```yaml
  apiVersion: oadp.openshift.io/v1alpha1
  kind: DataProtectionApplication
  ...
  configuration:
    velero:
      podConfig:
        resourceAllocations:
          requests:
            cpu: 200m
            memory: 256Mi
  ```

  The `resourceAllocations` listed are for average usage.

## Velero server terminates with OOMKilled when accessing large Kopia repositories {#velero-oomkilled-large-kopia-repos_pods-crash-or-restart-due-to-lack-of-memory-or-cpu}

If the Velero server accesses large Kopia repositories, the Velero pod might crash and report an `OOMKilled` event.

During operations that access large Kopia repositories, such as backup deletion or checking repository existence, the Velero pod crashes and reports an `OOMKilled` event.

This issue is typically caused by high memory usage when the Velero server loads many Kopia index blobs and related repository metadata. When the Backup Repository controller accesses a large-scale repository, the required memory can exceed the default limits.

To resolve this issue, identify the peak memory usage and increase the memory limits in the `DataProtectionApplication` (DPA) custom resource (CR) so that the Velero server has enough memory to load the repository metadata.

**Procedure**

1. Check the Velero logs around the time of the `OOMKilled` event for repository-open or index-loading activity.
2. Note the approximate maximum memory usage observed for the Velero pod by using a command such as `oc top pods`.
3. Edit the `DataProtectionApplication` CR:
   ```terminal
   $ oc edit dpa __<dpa_name>__ -n openshift-adp
   ```

   where:

   <dl>
   <dt><code>&lt;dpa_name&gt;</code></dt>
   <dd>Specifies the name of your <code>DataProtectionApplication</code> CR.</dd>
   </dl>
4. Increase the Velero container memory request or limit in the `spec.configuration.velero.podConfig` block to exceed the observed peak. For example:
   ```yaml
   spec:
     configuration:
       velero:
         podConfig:
           resourceAllocations:
             limits:
               memory: __<memory_limit>__
             requests:
               memory: 512Mi
   ```

   where:

   <dl>
   <dt><code>&lt;memory_limit&gt;</code></dt>
   <dd>Specifies a value that exceeds the maximum memory usage you observed in the logs, for example, <code>2Gi</code>.</dd>
   </dl>
5. Save your changes.
   The Velero pod redeploys automatically with the new limits.
6. Monitor the environment and repeat the process if necessary until `OOMKilled` events no longer occur.
7. Record the final stable memory values for your environment to use for future sizing guidance.

## Setting resource requests for a Restic pod {#oadp-pod-crash-resource-request-retics_pods-crash-or-restart-due-to-lack-of-memory-or-cpu}

Use the `configuration.restic.podConfig.resourceAllocations` specification field to set specific resource requests for a `Restic` pod.

:::note

With OADP 1.5.0, the `configuration.restic.podConfig.resourceAllocations` specification field is removed from Data Protection Application (DPA). Use the `nodeAgent` section with the `uploaderType` field set to `Kopia` instead of `Restic` .

:::

**Procedure**

- Set the `cpu` and `memory` resource requests as shown in the following example:
  ```yaml
  apiVersion: oadp.openshift.io/v1alpha1
  kind: DataProtectionApplication
  ...
  configuration:
    restic:
      podConfig:
        resourceAllocations:
          requests:
            cpu: 1000m
            memory: 16Gi
  ```

  The `resourceAllocations` listed are for average usage.

## Setting resource requests for a nodeAgent pod {#setting-resource-requests-for-a-nodeagent-pod_pods-crash-or-restart-due-to-lack-of-memory-or-cpu}

Use the `configuration.nodeAgent.podConfig.resourceAllocations` specification field to set specific resource requests for a `nodeAgent` pod.

:::note

With OADP 1.5.0, the `configuration.restic.podConfig.resourceAllocations` specification field is removed from Data Protection Application (DPA). Use the `nodeAgent` section with the `uploaderType` field set to `Kopia` instead of `Restic` .

:::

**Procedure**

1. Set the `cpu` and `memory` resource requests in the YAML file:
   ```yaml
   apiVersion: oadp.openshift.io/v1alpha1
   kind: DataProtectionApplication
   metadata:
     name: ts-dpa
   spec:
     backupLocations:
     - velero:
         default: true
         objectStorage:
           bucket: oadp.....njph
           prefix: velero
         credential:
           key: cloud
           name: cloud-credentials-gcp
         provider: gcp
     configuration:
       velero:
         defaultPlugins:
         - gcp
         - openshift
         - csi
       nodeAgent:
         enable: true
         uploaderType: kopia
         podConfig:
           resourceAllocations:
             requests:
               cpu: 1000m
               memory: 16Gi
   ```

   where:

   <dl>
   <dt><code>resourceAllocations</code></dt>
   <dd>The resource allocation examples shown are for average usage.</dd>
   <dt><code>memory</code></dt>
   <dd>You can modify this parameter depending on your infrastructure and usage.</dd>
   </dl>
2. Create the DPA CR by running the following command:
   ```terminal
   $ oc create -f nodeAgent.yaml
   ```

**Verification**

1. Verify that the `nodeAgent` pods are running by using the following command:
   ```terminal
   $ oc get pods
   ```

   ```terminal title="Example output"
   NAME                                                        READY   STATUS      RESTARTS   AGE
   node-agent-hbj9l                                            1/1     Running     0          97s
   node-agent-wmwgz                                            1/1     Running     0          95s
   node-agent-zvc7k                                            1/1     Running     0          98s
   openshift-adp-controller-manager-7f9db86d96-4lhgq           1/1     Running     0          137m
   velero-7b6c7fb8d7-ppc8m                                     1/1     Running     0          4m2s
   ```
2. Check the resource requests by describing one of the `nodeAgent` pod:
   ```terminal
   $ oc describe pod node-agent-hbj9l | grep -C 5 Requests
   ```

   ```terminal title="Example output"
         --log-format=text
       State:          Running
         Started:      Mon, 09 Jun 2025 16:22:15 +0530
       Ready:          True
       Restart Count:  0
       Requests:
         cpu:     1
         memory:  1Gi
       Environment:
         NODE_NAME:            (v1:spec.nodeName)
         VELERO_NAMESPACE:    openshift-adp (v1:metadata.namespace)
   ```
