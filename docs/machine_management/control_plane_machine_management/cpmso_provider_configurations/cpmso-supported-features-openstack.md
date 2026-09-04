---
title: "Configuring Red&#160;Hat OpenStack Platform (RHOSP) features for control plane machines"
sidebar_position: 9
---

# Configuring Red Hat OpenStack Platform (RHOSP) features for control plane machines {#cpmso-supported-features-openstack}

<a id="cpmso-supported-features-openstack"></a>

You can enable or change the configuration of features for your control plane machines by editing values in the control plane machine set specification.

When you save an update to the control plane machine set, the Control Plane Machine Set Operator updates the control plane machines according to your configured update strategy. For more information, see "Updating the control plane configuration".

## Changing the RHOSP compute flavor by using a control plane machine set {#cpms-changing-openstack-flavor-type_cpmso-supported-features-openstack}

You can change the Red Hat OpenStack Platform (RHOSP) compute service (Nova) flavor that your control plane machines use by updating the specification in the control plane machine set custom resource.

In RHOSP, flavors define the compute, memory, and storage capacity of computing instances. By increasing or decreasing the flavor size, you can scale your control plane vertically.

**Prerequisites**

- Your RHOSP cluster uses a control plane machine set.

**Procedure**

1. Edit the following line under the `providerSpec` field:
   ```yaml
   providerSpec:
     value:
   # ...
       flavor: m1.xlarge
   ```

   where:

   <dl>
   <dt>providerSpec.value.flavor</dt>
   <dd>Specify a RHOSP flavor type that has the same base as the existing selection. For example, you can change <code>m6i.xlarge</code> to <code>m6i.2xlarge</code> or <code>m6i.4xlarge</code>. You can choose larger or smaller flavors depending on your vertical scaling needs.</dd>
   </dl>
2. Save your changes.
   After you save your changes, machines are replaced with ones that use the flavor you chose.

**Additional resources**

- [Updating the control plane configuration](/docs/machine_management/control_plane_machine_management/cpmso-managing-machines#cpmso-feat-config-update_cpmso-managing-machines)
- [Control plane configuration options for {rh_openstack_full}](/docs/machine_management/control_plane_machine_management/cpmso_provider_configurations/cpmso-config-options-openstack#cpmso-config-options-openstack)
