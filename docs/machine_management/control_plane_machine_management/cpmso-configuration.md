---
title: Control plane machine set configuration
sidebar_position: 4
---

# Control plane machine set configuration {#cpmso-configuration}

<a id="cpmso-configuration"></a>

Use a control plane machine set to automate management and recovery of control plane machines in your cluster.

## Sample YAML for a control plane machine set custom resource {#cpmso-yaml-sample-cr_cpmso-configuration}

Use this sample YAML as a starting point for creating or modifying control plane machine set configurations on any supported platform.

```yaml title="Sample ControlPlaneMachineSet CR YAML file"
apiVersion: machine.openshift.io/v1
kind: ControlPlaneMachineSet
metadata:
  name: cluster
  namespace: openshift-machine-api
spec:
  replicas: 3
  selector:
    matchLabels:
      machine.openshift.io/cluster-api-cluster: <cluster_id>
      machine.openshift.io/cluster-api-machine-role: master
      machine.openshift.io/cluster-api-machine-type: master
  state: Active
  strategy:
    type: RollingUpdate
  template:
    machineType: machines_v1beta1_machine_openshift_io
    machines_v1beta1_machine_openshift_io:
      failureDomains:
        platform: <platform>
        <platform_failure_domains>
      metadata:
        labels:
          machine.openshift.io/cluster-api-cluster: <cluster_id>
          machine.openshift.io/cluster-api-machine-role: master
          machine.openshift.io/cluster-api-machine-type: master
      spec:
        providerSpec:
          value:
            <platform_provider_spec>
```

where:

`name: cluster`
: Specifies the name of the `ControlPlaneMachineSet` CR, which is `cluster`. Do not change this value.

`replicas: 3`
: Specifies the number of control plane machines. Only clusters with three control plane machines are supported, so the `replicas` value is `3`. Horizontal scaling is not supported. Do not change this value.

`<cluster_id>`
: Specifies the infrastructure ID that is based on the cluster ID that you set when you provisioned the cluster. You must specify this value when you create a `ControlPlaneMachineSet` CR. If you have the OpenShift CLI (`oc`) installed, you can obtain the infrastructure ID by running the following command:

  ```terminal
  $ oc get -o jsonpath='{.status.infrastructureName}{"\n"}' infrastructure cluster
  ```

`state: Active`
: Specifies the state of the Operator. When the state is `Inactive`, the Operator is not operational. You can activate the Operator by setting the value to `Active`.

  :::warning

  Before you activate the Operator, you must ensure that the `ControlPlaneMachineSet` CR configuration is correct for your cluster requirements. For more information about activating the Control Plane Machine Set Operator, see "Getting started with control plane machine sets".

  :::

`type: RollingUpdate`
: Specifies the update strategy for the cluster. The allowed values are `OnDelete` and `RollingUpdate`. The default value is `RollingUpdate`. For more information about update strategies, see "Updating the control plane configuration".

`platform: <platform>`
: Specifies the cloud provider platform name. Do not change this value.

`<platform_failure_domains>`
: Specifies the failure domains configuration for the cluster. The format and values of this section are provider-specific. For more information, see the sample failure domain configuration for your cloud provider.

`<platform_provider_spec>`
: Specifies the provider spec configuration for the cluster. The format and values of this section are provider-specific. For more information, see the sample provider specification for your cloud provider.

**Additional resources**

- [Getting started with control plane machine sets](/docs/machine_management/control_plane_machine_management/cpmso-getting-started#cpmso-getting-started)
- [Updating the control plane configuration](/docs/machine_management/control_plane_machine_management/cpmso-managing-machines#cpmso-feat-config-update_cpmso-managing-machines)

## Control plane machine set configuration options {#cpmso-config-options-overview_cpmso-configuration}

Customize your control plane machine set to meet specific cluster requirements or naming conventions.

### Adding a custom prefix to control plane machine names {#cpmso-config-prefix_cpmso-configuration}

Customize the prefix of control plane machine names to distinguish machines across environments or match your naming conventions.

**Procedure**

1. Edit the `ControlPlaneMachineSet` CR by running the following command:
   ```terminal
   $ oc edit controlplanemachineset.machine.openshift.io cluster \
     -n openshift-machine-api
   ```
2. Edit the `.spec.machineNamePrefix` field of the `ControlPlaneMachineSet` CR:
   ```yaml
   apiVersion: machine.openshift.io/v1
   kind: ControlPlaneMachineSet
   metadata:
     name: cluster
     namespace: openshift-machine-api
   spec:
     machineNamePrefix: <machine_prefix>
   # ...
   ```

   where `<machine_prefix>` specifies a prefix name that follows the requirements for a lowercase RFC 1123 subdomain.

   :::warning

   A lowercase RFC 1123 subdomain must consist of only lowercase alphanumeric characters, hyphens ('-'), and periods ('.'). Each block, separated by periods, must start and end with an alphanumeric character. Hyphens are not allowed at the start or end of a block, and consecutive periods are not permitted.

   :::
3. Save your changes.

**Next steps**

- If you changed only the value of the `machineNamePrefix` parameter, clusters that use the default `RollingUpdate` update strategy are not automatically updated. To propagate this change, you must replace your control plane machines manually, regardless of the update strategy for the cluster. For more information, see "Replacing a control plane machine".

**Additional resources**

- [Replacing a control plane machine](/docs/machine_management/control_plane_machine_management/cpmso-managing-machines#cpmso-feat-replace_cpmso-managing-machines)

## Provider-specific configuration options {#cpmso-config-provider-specific_cpmso-configuration}

Configure the provider-specific sections of your control plane machine set manifest for your cloud platform.

The `<platform_provider_spec>` and `<platform_failure_domains>` sections of the control plane machine set manifests are provider specific. For provider-specific configuration options for your cluster, see the documentation for your cloud provider.

**Additional resources**

- [Control plane configuration options for Amazon Web Services](/docs/machine_management/control_plane_machine_management/cpmso_provider_configurations/cpmso-config-options-aws#cpmso-config-options-aws)
- [Control plane configuration options for Google Cloud](/docs/machine_management/control_plane_machine_management/cpmso_provider_configurations/cpmso-config-options-gcp#cpmso-config-options-gcp)
- [Control plane configuration options for Microsoft Azure](/docs/machine_management/control_plane_machine_management/cpmso_provider_configurations/cpmso-config-options-azure#cpmso-config-options-azure)
- [Control plane configuration options for Nutanix](/docs/machine_management/control_plane_machine_management/cpmso_provider_configurations/cpmso-config-options-nutanix#cpmso-config-options-nutanix)
- [Control plane configuration options for Red Hat OpenStack Platform (RHOSP)](/docs/machine_management/control_plane_machine_management/cpmso_provider_configurations/cpmso-config-options-openstack#cpmso-config-options-openstack)
- [Control plane configuration options for VMware vSphere](/docs/machine_management/control_plane_machine_management/cpmso_provider_configurations/cpmso-config-options-vsphere#cpmso-config-options-vsphere)
