---
title: Monitoring ANP and BANP
sidebar_position: 3
---

# Monitoring ANP and BANP {#ovn-k-anp-banp-metrics}

<a id="ovn-k-anp-banp-metrics"></a>

To monitor and troubleshoot `AdminNetworkPolicy` and `BaselineAdminNetworkPolicy` resources in OpenShift Container Platform, you can use OVN-Kubernetes metrics that report resource counts, rules, and northbound database objects.

## Metrics for AdminNetworkPolicy {#anp-banp-metrics_ovn-k-anp-banp-metrics}

You can use OVN-Kubernetes metrics to monitor `AdminNetworkPolicy` and `BaselineAdminNetworkPolicy` resources. These metrics report policy and rule counts by direction and action, and northbound database objects that the policies create.

<table>
<tbody>
<tr>
  <td>Name</td>
  <td>Description</td>
  <td>Explanation</td>
</tr>
<tr>
  <td><code>ovnkube_controller_admin_network_policies</code></td>
  <td>Not applicable</td>
  <td>The total number of <code>AdminNetworkPolicy</code> resources in the cluster.</td>
</tr>
<tr>
  <td><code>ovnkube_controller_baseline_admin_network_policies</code></td>
  <td>Not applicable</td>
  <td>The total number of <code>BaselineAdminNetworkPolicy</code> resources in the cluster. The value should be 0 or 1.</td>
</tr>
<tr>
  <td><code>ovnkube_controller_admin_network_policies_rules</code></td>
  <td><ul><li><code>direction</code>: specifies either <code>Ingress</code> or <code>Egress</code>.</li><li><code>action</code>: specifies either <code>Pass</code>, <code>Allow</code>, or <code>Deny</code>.</li></ul></td>
  <td>The total number of rules across all ANP policies in the cluster grouped by <code>direction</code> and <code>action</code>.</td>
</tr>
<tr>
  <td><code>ovnkube_controller_baseline_admin_network_policies_rules</code></td>
  <td><ul><li><code>direction</code>: specifies either <code>Ingress</code> or <code>Egress</code>.</li><li><code>action</code>: specifies either <code>Allow</code> or <code>Deny</code>.</li></ul></td>
  <td>The total number of rules across all BANP policies in the cluster grouped by <code>direction</code> and <code>action</code>.</td>
</tr>
<tr>
  <td><code>ovnkube_controller_admin_network_policies_db_objects</code></td>
  <td><code>table_name</code>: specifies either <code>ACL</code> or <code>Address_Set</code></td>
  <td>The total number of OVN Northbound database (nbdb) objects that are created by all the ANP in the cluster grouped by the <code>table_name</code>.</td>
</tr>
<tr>
  <td><code>ovnkube_controller_baseline_admin_network_policies_db_objects</code></td>
  <td><code>table_name</code>: specifies either <code>ACL</code> or <code>Address_Set</code></td>
  <td>The total number of OVN Northbound database (nbdb) objects that are created by all the BANP in the cluster grouped by the <code>table_name</code>.</td>
</tr>
</tbody>
</table>
