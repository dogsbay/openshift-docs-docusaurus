---
title: Destroying a hosted cluster on IBM Power
sidebar_position: 5
---

# Destroying a hosted cluster on IBM Power {#hcp-destroy-ibm-power}

<a id="hcp-destroy-ibm-power"></a>

You might want to remove a hosted cluster if you are no longer using it, you are trying to reduce resources, or the hosted cluster is experiencing issues that are difficult to resolve.

## Destroying a hosted cluster on IBM Power by using the CLI {#destroy-hc-ibm-power-cli_hcp-destroy-ibm-power}

To destroy a hosted cluster on IBM Power, you can use the hcp command-line interface (CLI).

**Procedure**

- Delete the hosted cluster by running the following command:
  ```terminal
  $ hcp destroy cluster agent
   --name=<hosted_cluster_name> \
   --namespace=<hosted_cluster_namespace> \
   --cluster-grace-period <duration>
  ```

  - `<hosted_cluster_name>` specifies the name of your hosted cluster.
  - `<hosted_cluster_namespace>` specifies the name of your hosted cluster namespace.
  - `<duration>` specifies the duration to destroy the hosted cluster completely, for example, `20m0s`.
