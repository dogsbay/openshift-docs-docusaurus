---
title: Tune the control plane
sidebar_position: 13
---

# Tune the control plane {#virt-vm-control-plane-tuning}

<a id="virt-vm-control-plane-tuning"></a>

You can configure how the control plane handles concurrency when you create or migrate virtual machines (VMs). For example, set the `QPS` or `burst` rates to batch create virtual machines (VMs) in a batch, or tune migration settings in the `HyperConverged` custom resource (CR).

## Configuring a highBurst profile {#virt-configuring-highburst-profile_virt-control-plane-tuning}

You can use the `highBurst` profile to create and maintain a large number of virtual machines (VMs) in one cluster.

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).

**Procedure**

- Apply the following patch to enable the `highBurst` tuning policy profile:
  ```terminal
  $ oc patch hyperconvergeds.v1beta1.hco.kubevirt.io kubevirt-hyperconverged -n openshift-cnv \
    --type=json -p='[{"op": "add", "path": "/spec/tuningPolicy", \
    "value": "highBurst"}]'
  ```

**Verification**

- Run the following command to verify the `highBurst` tuning policy profile is enabled:
  ```terminal
  $ oc get kubevirt.kubevirt.io/kubevirt-kubevirt-hyperconverged \
    -n {CNVNamespace} -o go-template --template='{{range $config, \
    $value := .spec.configuration}} {{if eq $config "apiConfiguration" \
    "webhookConfiguration" "controllerConfiguration" "handlerConfiguration"}} \
    {{"\n"}} {{$config}} = {{$value}} {{end}} {{end}} {{"\n"}}
  ```
