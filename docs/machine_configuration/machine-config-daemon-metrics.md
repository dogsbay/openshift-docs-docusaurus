---
title: Machine Config Daemon metrics overview
sidebar_position: 13
---

# Machine Config Daemon metrics overview {#machine-config-daemon-metrics}

<a id="machine-config-daemon-metrics"></a>

The Machine Config Daemon, part of the Machine Config Operator, runs on every node in the cluster to manage configuration changes and updates on each of the nodes.

## Understand Machine Config Daemon metrics {#machine-config-daemon-metrics-understanding_machine-config-operator}

You can access the metrics provided by the Machine Config Daemon by using the Prometheus Cluster Monitoring stack.

The following table describes this set of metrics. Some entries contain commands for getting specific logs. However, the most comprehensive set of logs is available using the `oc adm must-gather` command.

:::note

Metrics marked with `*` in the **Name** and **Description** columns represent serious errors that might cause performance problems. Such problems might prevent updates and upgrades from proceeding.

:::

**MCO metrics**

<table>
<thead>
<tr>
  <th>Name</th>
  <th>Format</th>
  <th>Description</th>
  <th>Notes</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>mcd_host_os_and_version</code></td>
  <td><code>[]string{"os", "version"}</code></td>
  <td>Shows the OS that MCD is running on, such as RHCOS or RHEL. In case of RHCOS, the version is provided.</td>
  <td></td>
</tr>
<tr>
  <td><code>mcd_drain_err*</code></td>
  <td></td>
  <td>Logs errors received during failed drain. *</td>
  <td>While drains might need multiple tries to succeed, terminal failed drains prevent updates from proceeding. The <code>drain_time</code> metric, which shows how much time the drain took, might help with troubleshooting.<br><br>For further investigation, see the logs by running:<br><br><code>$ oc logs -f -n openshift-machine-config-operator machine-config-daemon-&lt;hash&gt; -c machine-config-daemon</code></td>
</tr>
<tr>
  <td><code>mcd_pivot_err*</code></td>
  <td><code>[]string{"err", "node", "pivot_target"}</code></td>
  <td>Logs errors encountered during pivot. *</td>
  <td>Pivot errors might prevent OS upgrades from proceeding.<br><br>For further investigation, run this command to see the logs from the <code>machine-config-daemon</code> container:<br><br><code>$ oc logs -f -n openshift-machine-config-operator machine-config-daemon-&lt;hash&gt; -c machine-config-daemon</code></td>
</tr>
<tr>
  <td><code>mcd_state</code></td>
  <td><code>[]string{"state", "reason"}</code></td>
  <td>State of Machine Config Daemon for the indicated node. Possible states are "Done", "Working", and "Degraded". In case of "Degraded", the reason is included.</td>
  <td>For further investigation, see the logs by running:<br><br><code>$ oc logs -f -n openshift-machine-config-operator machine-config-daemon-&lt;hash&gt; -c machine-config-daemon</code></td>
</tr>
<tr>
  <td><code>mcd_kubelet_state*</code></td>
  <td></td>
  <td>Logs kubelet health failures.  *</td>
  <td>This is expected to be empty, with failure count of 0. If failure count exceeds 2, the error indicating threshold is exceeded. This indicates a possible issue with the health of the kubelet.<br><br>For further investigation, run this command to access the node and see all its logs:<br><br><code>$ oc debug node/&lt;node&gt; -- chroot /host journalctl -u kubelet</code></td>
</tr>
<tr>
  <td><code>mcd_reboot_err*</code></td>
  <td><code>[]string{"message", "err", "node"}</code></td>
  <td>Logs the failed reboots and the corresponding errors. *</td>
  <td>This is expected to be empty, which indicates a successful reboot.<br><br>For further investigation, see the logs by running:<br><br><code>$ oc logs -f -n openshift-machine-config-operator machine-config-daemon-&lt;hash&gt; -c machine-config-daemon</code></td>
</tr>
<tr>
  <td><code>mcd_update_state</code></td>
  <td><code>[]string{"config", "err"}</code></td>
  <td>Logs success or failure of configuration updates and the corresponding errors.</td>
  <td>The expected value is <code>rendered-master/rendered-worker-XXXX</code>. If the update fails, an error is present.<br><br>For further investigation, see the logs by running:<br><br><code>$ oc logs -f -n openshift-machine-config-operator machine-config-daemon-&lt;hash&gt; -c machine-config-daemon</code></td>
</tr>
</tbody>
</table>

**Additional resources**

- [About OpenShift Container Platform monitoring](https://docs.redhat.com/en/documentation/monitoring_stack_for_red_hat_openshift/latest/html/about_monitoring/about-ocp-monitoring)
- [Gathering data about your cluster](/docs/support/gathering-cluster-data#gathering-cluster-data)
