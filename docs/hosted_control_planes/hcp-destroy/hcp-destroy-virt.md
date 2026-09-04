---
title: Destroying a hosted cluster on OpenShift Virtualization
sidebar_position: 3
---

# Destroying a hosted cluster on OpenShift Virtualization {#hcp-destroy-virt}

<a id="hcp-destroy-virt"></a>

You might want to remove a hosted cluster if you are no longer using it, you are trying to reduce resources, or the hosted cluster is experiencing issues that are difficult to resolve.

## Destroying a hosted cluster on OpenShift Virtualization by using the CLI {#destroy-hc-virt-cli_hcp-destroy-virt}

You can use the command-line interface (CLI) to destroy a hosted cluster and its managed cluster resource on OpenShift Virtualization.

**Procedure**

1. Delete the managed cluster resource on multicluster engine Operator by running the following command:
   ```terminal
   $ oc delete managedcluster <hosted_cluster_name>
   ```
2. Delete the hosted cluster and its backend resources by running the following command:
   ```terminal
   $ hcp destroy cluster kubevirt --name <hosted_cluster_name>
   ```
