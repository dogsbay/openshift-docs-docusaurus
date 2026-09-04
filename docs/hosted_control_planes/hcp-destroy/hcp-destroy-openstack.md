---
title: Destroying a hosted control plane on OpenStack
sidebar_position: 6
---

# Destroying a hosted control plane on OpenStack {#hcp-destroy-openstack}

<a id="hcp-destroy-openstack"></a>

You might want to remove a hosted cluster if you are no longer using it, you are trying to reduce resources, or the hosted cluster is experiencing issues that are difficult to resolve.

## Destroying a hosted cluster by using the CLI {#hosted-clusters-openstack-destroy_hcp-destroy-openstack}

You can destroy a hosted cluster and its associated resources on Red Hat OpenStack Platform (RHOSP) by using the `hcp` CLI tool.

**Prerequisites**

- You installed the hosted control planes CLI, `hcp`.

**Procedure**

- To destroy the cluster and its associated resources, run the following command:
  ```terminal
  $ hcp destroy cluster openstack --name=<cluster_name>
  ```

  Replace `<cluster_name>` with the name of the hosted cluster.

  After the process completes, your cluster and all resources that are associated with it are destroyed.
