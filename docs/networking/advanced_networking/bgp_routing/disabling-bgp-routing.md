---
title: Disabling BGP routing
sidebar_position: 3
---

# Disabling BGP routing

<a id="disabling-bgp-routing"></a>

To stop external route advertisement and restore standard cluster networking behavior, disable OVN-Kubernetes Border Gateway Protocol (BGP) routing.

As a cluster administrator, you can disable OVN-Kubernetes BGP routing support for your cluster.

## Disabling Border Gateway Protocol (BGP) routing {#nw-bgp-routing-config_disable-bgp-routing}

Disable Border Gateway Protocol (BGP) routing for your cluster by removing additional routing capabilities from the network configuration.

As a cluster administrator, you can disable BGP routing support for your cluster on bare-metal infrastructure.

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).
- You are logged in to the cluster as a user with the `cluster-admin` role.
- The cluster is installed on compatible infrastructure.

**Procedure**

- To disable dynamic routing, enter the following command:
  ```terminal
  $ oc patch Network.operator.openshift.io/cluster --type=merge -p '{
    "spec": { "additionalRoutingCapabilities": null }
  }'
  ```
