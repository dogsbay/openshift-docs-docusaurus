---
title: Destroying a hosted cluster on non-bare-metal agent machines
sidebar_position: 7
---

# Destroying a hosted cluster on non-bare-metal agent machines {#hcp-destroy-non-bm}

<a id="hcp-destroy-non-bm"></a>

You might want to remove a hosted cluster if you are no longer using it, you are trying to reduce resources, or the hosted cluster is experiencing issues that are difficult to resolve.

## Destroying a hosted cluster on non-bare-metal agent machines {#destroy-hc-non-bm-cli_hcp-destroy-non-bm}

You can use the `hcp` command-line interface (CLI) to destroy a hosted cluster on non-bare-metal agent machines.

**Procedure**

- Delete the hosted cluster and its backend resources by running the following command:
  ```terminal
  $ hcp destroy cluster agent --name <hosted_cluster_name>
  ```

  Replace `<hosted_cluster_name>` with the name of your hosted cluster.

## Destroying a hosted cluster on non-bare-metal agent machines by using the web console {#destroy-hc-non-bm-console_hcp-destroy-non-bm}

You can use the multicluster engine Operator web console to destroy a hosted cluster on non-bare-metal agent machines.

**Procedure**

1. In the console, click **Infrastructure** → **Clusters**.
2. On the **Clusters** page, select the cluster that you want to destroy.
3. In the **Actions** menu, select **Destroy clusters** to remove the cluster.
