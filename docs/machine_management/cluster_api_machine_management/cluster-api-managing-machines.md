---
title: Managing machines with the Cluster API
sidebar_position: 3
---

# Managing machines with the Cluster API {#cluster-api-managing-machines}

<a id="cluster-api-managing-machines"></a>

You can manage machines with the Cluster API by modifying a Cluster API machine template or a compute machine set by using the CLI.

:::warning

Managing machines with the Cluster API is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.

For more information about the support scope of Red Hat Technology Preview features, see [Technology Preview Features Support Scope](https://access.redhat.com/support/offerings/techpreview/).

:::

## Modifying a Cluster API machine template {#capi-modifying-machine-template_cluster-api-managing-machines}

You can update the machine template resource for your cluster by modifying the YAML manifest file and applying it with the OpenShift CLI (`oc`).

**Prerequisites**

- You have deployed an OpenShift Container Platform cluster that uses the Cluster API.
- You have access to the cluster using an account with `cluster-admin` permissions.
- You have installed the OpenShift CLI (`oc`).

**Procedure**

1. List the machine template resource for your cluster by running the following command:
   ```terminal
   $ oc get <machine_template_kind>
   ```

   Replace `<machine_template_kind>` with the value that corresponds to your platform. The following values are valid:

   | Cluster infrastructure provider | Value |
   | --- | --- |
   | Amazon Web Services | `AWSMachineTemplate` |
   | Google Cloud | `GCPMachineTemplate` |
   | Microsoft Azure | `AzureMachineTemplate` |
   | RHOSP | `OpenStackMachineTemplate` |
   | VMware vSphere | `VSphereMachineTemplate` |
   | Bare metal | `Metal3MachineTemplate` |

   ```text title="Example output"
   NAME              AGE
   <template_name>   77m
   ```
2. Write the machine template resource for your cluster to a file that you can edit by running the following command:
   ```terminal
   $ oc get <machine_template_kind> <template_name> -o yaml > <template_name>.yaml
   ```

   Replace `<template_name>` with the name of the machine template resource for your cluster.
3. Make a copy of the `<template_name>.yaml` file with a different name. This procedure uses `<modified_template_name>.yaml` as an example file name.
4. Use a text editor to make changes to the `<modified_template_name>.yaml` file that defines the updated machine template resource for your cluster. When editing the machine template resource, observe the following:
   - The parameters in the `spec` stanza are provider specific. For more information, see the sample Cluster API machine template YAML for your provider.
   - You must use a value for the `metadata.name` parameter that differs from any existing values.
     :::warning

     For any Cluster API compute machine sets that reference this template, you must update the `spec.template.spec.infrastructureRef.name` parameter to match the `metadata.name` value in the new machine template resource.

     :::
5. Apply the machine template CR by running the following command:
   ```terminal
   $ oc apply -f <modified_template_name>.yaml
   ```

   For `<modified_template_name>`, use the edited YAML file with a new name.

**Next steps**

- For any Cluster API compute machine sets that reference this template, update the `spec.template.spec.infrastructureRef.name` parameter to match the `metadata.name` value in the new machine template resource. For more information, see "Modifying a compute machine set by using the CLI."

**Additional resources**

- [Sample YAML for a Cluster API machine template resource on Amazon Web Services](/docs/machine_management/cluster_api_machine_management/cluster_api_provider_configurations/cluster-api-config-options-aws#capi-yaml-machine-template-aws_cluster-api-config-options-aws)
- [Sample YAML for a Cluster API machine template resource on Google Cloud](/docs/machine_management/cluster_api_machine_management/cluster_api_provider_configurations/cluster-api-config-options-gcp#capi-yaml-machine-template-gcp_cluster-api-config-options-gcp)
- [Sample YAML for a Cluster API machine template resource on Microsoft Azure](/docs/machine_management/cluster_api_machine_management/cluster_api_provider_configurations/cluster-api-config-options-azure#capi-yaml-machine-template-azure_cluster-api-config-options-azure)
- [Sample YAML for a Cluster API machine template resource on RHOSP](/docs/machine_management/cluster_api_machine_management/cluster_api_provider_configurations/cluster-api-config-options-rhosp#capi-yaml-machine-template-rhosp_cluster-api-config-options-rhosp)
- [Sample YAML for a Cluster API machine template resource on VMware vSphere](/docs/machine_management/cluster_api_machine_management/cluster_api_provider_configurations/cluster-api-config-options-vsphere#capi-yaml-machine-template-vsphere_cluster-api-config-options-vsphere)
- [Modifying a compute machine set by using the CLI](/docs/machine_management/cluster_api_machine_management/cluster-api-managing-machines#machineset-modifying_cluster-api-managing-machines)

## Modify a compute machine set by using the CLI {#machineset-modifying_cluster-api-managing-machines}

To enable features or change the properties of machines, you can modify the configuration of a compute machine set using the CLI. You can then propagate the changes to the machines in your cluster.

When you modify a compute machine set, your changes only apply to compute machines that are created after you save the updated `MachineSet` custom resource (CR). The changes do not affect existing machines.

:::note

Changes made in the underlying cloud provider are not reflected in the `Machine` or `MachineSet` CRs. To adjust instance configuration in cluster-managed infrastructure, use the cluster-side resources.

:::

You can replace the existing machines with new ones that reflect the updated configuration by scaling the compute machine set to create twice the number of replicas and then scaling it down to the original number of replicas.

If you need to scale a compute machine set without making other changes, you do not need to delete the machines.

:::note

By default, the OpenShift Container Platform router pods are deployed on compute machines. Because the router is required to access some cluster resources, including the web console, do not scale the compute machine set to `0` unless you first relocate the router pods.

:::

The output examples in this procedure use the values for an AWS cluster.

**Prerequisites**

tag:CAPI[][]

- Your OpenShift Container Platform cluster uses the Cluster API. end:CAPI[][] tag:MAPI[][]
- Your OpenShift Container Platform cluster uses the Machine API. end:MAPI[][]
- You are logged in to the cluster as an administrator by using the OpenShift CLI (`oc`).

**Procedure**

1. List the compute machine sets in your cluster by running the following command:
   tag:CAPI[][]

   ```terminal
   $ oc get machinesets.cluster.x-k8s.io -n openshift-cluster-api
   ```

   ```text title="Example output"
   NAME                          CLUSTER             REPLICAS   READY   AVAILABLE   AGE   VERSION
   <compute_machine_set_name_1>  <cluster_name>      1          1       1           26m
   <compute_machine_set_name_2>  <cluster_name>      1          1       1           26m
   ```

   end:CAPI[][] tag:MAPI[][]

   ```terminal
   $ oc get machinesets.machine.openshift.io -n openshift-machine-api
   ```

   ```text title="Example output"
   NAME                           DESIRED   CURRENT   READY   AVAILABLE   AGE
   <compute_machine_set_name_1>   1         1         1       1           55m
   <compute_machine_set_name_2>   1         1         1       1           55m
   ```

   end:MAPI[][]
2. Edit a compute machine set by running the following command:
   tag:CAPI[][]

   ```terminal
   $ oc edit machinesets.cluster.x-k8s.io <machine_set_name> \
     -n openshift-cluster-api
   ```

   end:CAPI[][] tag:MAPI[][]

   ```terminal
   $ oc edit machinesets.machine.openshift.io <machine_set_name> \
     -n openshift-machine-api
   ```

   end:MAPI[][]
3. Note the value of the `spec.replicas` field, because you need it when scaling the machine set to apply the changes.
   ```yaml
   #tag::CAPI[]
   apiVersion: cluster.x-k8s.io/v1beta1
   #end::CAPI[]
   #tag::MAPI[]
   apiVersion: machine.openshift.io/v1beta1
   #end::MAPI[]
   kind: MachineSet
   metadata:
     name: <machine_set_name>
   #tag::CAPI[]
     namespace: openshift-cluster-api
   #end::CAPI[]
   #tag::MAPI[]
     namespace: openshift-machine-api
   #end::MAPI[]
   spec:
     replicas: 2
   # ...
   ```

   The examples in this procedure show a compute machine set that has a `replicas` value of `2`.
4. Update the compute machine set CR with the configuration options that you want and save your changes.
5. List the machines that are managed by the updated compute machine set by running the following command:
   tag:CAPI[][]

   ```terminal
   $ oc get machines.cluster.x-k8s.io \
     -n openshift-cluster-api \
     -l cluster.x-k8s.io/set-name=<machine_set_name>
   ```

   ```text title="Example output for an AWS cluster"
   NAME                        CLUSTER          NODENAME                                    PROVIDERID                              PHASE           AGE     VERSION
   <machine_name_original_1>   <cluster_name>   <original_1_ip>.<region>.compute.internal   aws:///us-east-2a/i-04e7b2cbd61fd2075   Running         4h
   <machine_name_original_2>   <cluster_name>   <original_2_ip>.<region>.compute.internal   aws:///us-east-2a/i-04e7b2cbd61fd2075   Running         4h
   ```

   end:CAPI[][] tag:MAPI[][]

   ```terminal
   $ oc get machines.machine.openshift.io \
     -n openshift-machine-api \
     -l machine.openshift.io/cluster-api-machineset=<machine_set_name>
   ```

   ```text title="Example output for an AWS cluster"
   NAME                        PHASE     TYPE         REGION      ZONE         AGE
   <machine_name_original_1>   Running   m6i.xlarge   us-west-1   us-west-1a   4h
   <machine_name_original_2>   Running   m6i.xlarge   us-west-1   us-west-1a   4h
   ```

   end:MAPI[][]
6. For each machine that is managed by the updated compute machine set, set the `delete` annotation by running the following command:
   tag:CAPI[][]

   ```terminal
   $ oc annotate machines.cluster.x-k8s.io/<machine_name_original_1> \
     -n openshift-cluster-api \
     cluster.x-k8s.io/delete-machine="true"
   ```

   end:CAPI[][] tag:MAPI[][]

   ```terminal
   $ oc annotate machine.machine.openshift.io/<machine_name_original_1> \
     -n openshift-machine-api \
     machine.openshift.io/delete-machine="true"
   ```

   end:MAPI[][]
7. To create replacement machines with the new configuration, scale the compute machine set to twice the number of replicas by running the following command:
   tag:CAPI[][]

   ```terminal
   $ oc scale --replicas=4 \
     machinesets.cluster.x-k8s.io <machine_set_name> \
     -n openshift-cluster-api
   ```

   end:CAPI[][] tag:MAPI[][]

   ```terminal
   $ oc scale --replicas=4 \
     machineset.machine.openshift.io <machine_set_name> \
     -n openshift-machine-api
   ```

   end:MAPI[][]

   The original example value of `2` is doubled to `4`.
8. List the machines that are managed by the updated compute machine set by running the following command:
   tag:CAPI[][]

   ```terminal
   $ oc get machines.cluster.x-k8s.io \
     -n openshift-cluster-api \
     -l cluster.x-k8s.io/set-name=<machine_set_name>
   ```

   ```text title="Example output for an AWS cluster"
   NAME                        CLUSTER          NODENAME                                    PROVIDERID                              PHASE           AGE     VERSION
   <machine_name_original_1>   <cluster_name>   <original_1_ip>.<region>.compute.internal   aws:///us-east-2a/i-04e7b2cbd61fd2075   Running         4h
   <machine_name_original_2>   <cluster_name>   <original_2_ip>.<region>.compute.internal   aws:///us-east-2a/i-04e7b2cbd61fd2075   Running         4h
   <machine_name_updated_1>    <cluster_name>   <updated_1_ip>.<region>.compute.internal    aws:///us-east-2a/i-04e7b2cbd61fd2075   Provisioned     55s
   <machine_name_updated_2>    <cluster_name>   <updated_2_ip>.<region>.compute.internal    aws:///us-east-2a/i-04e7b2cbd61fd2075   Provisioning    55s
   ```

   end:CAPI[][] tag:MAPI[][]

   ```terminal
   $ oc get machines.machine.openshift.io \
     -n openshift-machine-api \
     -l machine.openshift.io/cluster-api-machineset=<machine_set_name>
   ```

   ```text title="Example output for an AWS cluster"
   NAME                        PHASE          TYPE         REGION      ZONE         AGE
   <machine_name_original_1>   Running        m6i.xlarge   us-west-1   us-west-1a   4h
   <machine_name_original_2>   Running        m6i.xlarge   us-west-1   us-west-1a   4h
   <machine_name_updated_1>    Provisioned    m6i.xlarge   us-west-1   us-west-1a   55s
   <machine_name_updated_2>    Provisioning   m6i.xlarge   us-west-1   us-west-1a   55s
   ```

   end:MAPI[][]

   When the new machines are in the `Running` phase, you can scale the compute machine set to the original number of replicas.
9. To remove the machines that were created with the old configuration, scale the compute machine set to the original number of replicas by running the following command:
   tag:CAPI[][]

   ```terminal
   $ oc scale --replicas=2 \
     machinesets.cluster.x-k8s.io <machine_set_name> \
     -n openshift-cluster-api
   ```

   end:CAPI[][] tag:MAPI[][]

   ```terminal
   $ oc scale --replicas=2 \
     machineset.machine.openshift.io <machine_set_name> \
     -n openshift-machine-api
   ```

   end:MAPI[][]

   The `replicas` parameter is set to the original example value of `2`.

**Verification**

- To verify that a machine created by the updated machine set has the correct configuration, examine the relevant fields in the CR for one of the new machines by running the following command:
  tag:CAPI[][]

  ```terminal
  $ oc describe machines.cluster.x-k8s.io <machine_name_updated_1> \
    -n openshift-cluster-api
  ```

  end:CAPI[][] tag:MAPI[][]

  ```terminal
  $ oc describe machine.machine.openshift.io <machine_name_updated_1> \
    -n openshift-machine-api
  ```

  end:MAPI[][]
- To verify that the compute machines without the updated configuration are deleted, list the machines that are managed by the updated compute machine set by running the following command:
  tag:CAPI[][]

  ```terminal
  $ oc get machines.cluster.x-k8s.io \
    -n openshift-cluster-api \
    cluster.x-k8s.io/set-name=<machine_set_name>
  ```

  ```text title="Example output while deletion is in progress for an AWS cluster"
  NAME                        CLUSTER          NODENAME                                    PROVIDERID                              PHASE      AGE     VERSION
  <machine_name_original_1>   <cluster_name>   <original_1_ip>.<region>.compute.internal   aws:///us-east-2a/i-04e7b2cbd61fd2075   Running    18m
  <machine_name_original_2>   <cluster_name>   <original_2_ip>.<region>.compute.internal   aws:///us-east-2a/i-04e7b2cbd61fd2075   Running    18m
  <machine_name_updated_1>    <cluster_name>   <updated_1_ip>.<region>.compute.internal    aws:///us-east-2a/i-04e7b2cbd61fd2075   Running    18m
  <machine_name_updated_2>    <cluster_name>   <updated_2_ip>.<region>.compute.internal    aws:///us-east-2a/i-04e7b2cbd61fd2075   Running    18m
  ```

  ```text title="Example output when deletion is complete for an AWS cluster"
  NAME                        CLUSTER          NODENAME                                    PROVIDERID                              PHASE      AGE     VERSION
  <machine_name_updated_1>    <cluster_name>   <updated_1_ip>.<region>.compute.internal    aws:///us-east-2a/i-04e7b2cbd61fd2075   Running    18m
  <machine_name_updated_2>    <cluster_name>   <updated_2_ip>.<region>.compute.internal    aws:///us-east-2a/i-04e7b2cbd61fd2075   Running    18m
  ```

  end:CAPI[][] tag:MAPI[][]

  ```terminal
  $ oc get machines.machine.openshift.io \
    -n openshift-machine-api \
    -l machine.openshift.io/cluster-api-machineset=<machine_set_name>
  ```

  ```text title="Example output while deletion is in progress for an AWS cluster"
  NAME                        PHASE           TYPE         REGION      ZONE         AGE
  <machine_name_original_1>   Deleting        m6i.xlarge   us-west-1   us-west-1a   4h
  <machine_name_original_2>   Deleting        m6i.xlarge   us-west-1   us-west-1a   4h
  <machine_name_updated_1>    Running         m6i.xlarge   us-west-1   us-west-1a   5m41s
  <machine_name_updated_2>    Running         m6i.xlarge   us-west-1   us-west-1a   5m41s
  ```

  ```text title="Example output when deletion is complete for an AWS cluster"
  NAME                        PHASE           TYPE         REGION      ZONE         AGE
  <machine_name_updated_1>    Running         m6i.xlarge   us-west-1   us-west-1a   6m30s
  <machine_name_updated_2>    Running         m6i.xlarge   us-west-1   us-west-1a   6m30s
  ```

  end:MAPI[][]

**Additional resources**

- [Sample YAML for a Cluster API compute machine set resource on Amazon Web Services](/docs/machine_management/cluster_api_machine_management/cluster_api_provider_configurations/cluster-api-config-options-aws#capi-yaml-machine-set-aws_cluster-api-config-options-aws)
- [Sample YAML for a Cluster API compute machine set resource on Google Cloud](/docs/machine_management/cluster_api_machine_management/cluster_api_provider_configurations/cluster-api-config-options-gcp#capi-yaml-machine-set-gcp_cluster-api-config-options-gcp)
- [Sample YAML for a Cluster API compute machine set resource on Microsoft Azure](/docs/machine_management/cluster_api_machine_management/cluster_api_provider_configurations/cluster-api-config-options-azure#capi-yaml-machine-set-azure_cluster-api-config-options-azure)
- [Sample YAML for a Cluster API compute machine set resource on RHOSP](/docs/machine_management/cluster_api_machine_management/cluster_api_provider_configurations/cluster-api-config-options-rhosp#capi-yaml-machine-set-rhosp_cluster-api-config-options-rhosp)
- [Sample YAML for a Cluster API compute machine set resource on VMware vSphere](/docs/machine_management/cluster_api_machine_management/cluster_api_provider_configurations/cluster-api-config-options-vsphere#capi-yaml-machine-set-vsphere_cluster-api-config-options-vsphere)
