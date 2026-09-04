---
title: Monitoring Transport Layer Security traffic
sidebar_position: 12
---

# Monitoring Transport Layer Security traffic {#network-observability-monitoring-tls-traffic}

<a id="network-observability-monitoring-tls-traffic"></a>

Monitor TLS traffic to identify insecure protocols, detect security risks, and maintain compliance without decrypting traffic.

## Transport Layer Security traffic monitoring {#network-observability-tls-monitoring-overview_network-observability-monitoring-tls-traffic}

Transport Layer Security (TLS) traffic monitoring identifies security risks and maintains compliance by analyzing encrypted traffic metadata without decryption.

As a network administrator or security practitioner, you must verify that encrypted traffic uses secure protocols and cipher suites. Monitoring TLS usage identifies security risks, such as workloads that use deprecated TLS versions, and helps maintain compliance with cluster security policies.

### Security improvements through metadata analysis {#tls-monitoring-security-benefits_network-observability-monitoring-tls-traffic}

The Network Observability Operator captures TLS metadata from handshake messages without decrypting traffic, providing visibility into encryption protocols while maintaining data privacy. This approach enables the following improvements:

Security risk detection
: Identifies workloads using deprecated TLS versions (1.0, 1.1) or weak cipher suites by capturing TLS version, cipher suite, and group information. You can configure Prometheus alerts to automatically report deprecated TLS configurations.

Compliance auditing
: Audits TLS configurations to meet regulatory requirements through metric aggregation in dashboard charts and overview panels. You can filter flows by TLS fields to isolate specific protocol versions or cipher suites for compliance reporting.

Security posture assessment
: Visualizes encrypted network traffic with lock icons in the topology view and identifies unencrypted communications across your cluster. You can analyze TLS usage patterns to evaluate your overall security posture.

Remediation prioritization
: Targets workloads using deprecated protocols for updates by filtering and analyzing TLS fields to isolate problematic connections requiring immediate attention.

### TLS traffic monitoring workflow phases {#monitoring-tls-traffic-workflow_network-observability-monitoring-tls-traffic}

To monitor TLS traffic effectively, complete the following phases:

- Enable the TLS tracking feature in the eBPF agent configuration.
- Analyze TLS traffic details in the **Network Traffic** view.
- Visualize secure connections in the **Topology** view.

## Enable Transport Layer Security tracking {#network-observability-enable-tls-tracking_network-observability-monitoring-tls-traffic}

Enable Transport Layer Security (TLS) tracking to monitor encryption protocols and identify security risks in the cluster.

:::note

TLS fields only appear in flows for connections that perform a TLS handshake after the feature is enabled.

:::

**Prerequisites**

- The Network Observability Operator is installed.
- The `FlowCollector` custom resource (CR) is configured with `spec.agent.type: eBPF`.
- Access to the cluster with `cluster-admin` privileges.

**Procedure**

1. Edit the `FlowCollector` CR by running the following command:
   ```terminal
   $ oc edit flowcollector cluster
   ```
2. Add `TLSTracking` to the `spec.agent.ebpf.features` list:
   ```yaml
   apiVersion: flows.netobserv.io/v1beta2
   kind: FlowCollector
   metadata:
     name: cluster
   spec:
     agent:
       type: eBPF
       ebpf:
         features:
         - TLSTracking
   # ...
   ```

   where:

   <dl>
   <dt><code>spec.agent.ebpf.features</code></dt>
   <dd>Specifies the list of eBPF agent features to enable. Add <code>TLSTracking</code> to this array to enable TLS metadata capture from handshake messages.</dd>
   </dl>
3. Save and exit your editor.

**Verification**

1. Confirm that the eBPF agent pods have restarted by running the following command:
   ```terminal
   $ oc get pods -n netobserv-privileged
   ```

   ```terminal title="Example output"
   NAME                                    READY   STATUS    RESTARTS   AGE
   netobserv-ebpf-agent-abc12              1/1     Running   0          2m
   ```
2. Verify the TLS tracking feature is active by running the following command:
   ```terminal
   $ oc logs -n netobserv-privileged ds/netobserv-ebpf-agent | grep "EnableTLSTracking"
   ```

   ```terminal title="Example output"
   EnableTLSTracking:true
   ```

   The output confirms that the TLS tracking feature has been initialized in the eBPF agent.

## Analyze Transport Layer Security traffic data {#network-observability-analyze-tls-traffic_network-observability-monitoring-tls-traffic}

View and filter Transport Layer Security (TLS) metadata to identify deprecated configurations and verify encryption compliance in the cluster.

**Prerequisites**

- The Network Observability Operator is installed.
- TLS tracking is enabled in the `FlowCollector` custom resource (CR).
- Access to the OpenShift Container Platform web console.

**Procedure**

1. Navigate to **Observe** → **Network Traffic** in the OpenShift Container Platform web console and click the **Traffic flows** tab.
   :::note

   The **TLS Version** column is enabled by default. If the default TLS version column is not visible after enabling TLS tracking, click **Restore default columns** in **Manage columns** to refresh the table.

   :::
2. Add TLS-specific columns to the traffic table:
   1. Click **Manage columns**.
   2. Select the **TLS Cipher Suite**, **TLS Group**, and **TLS Types** checkboxes.
   3. Click **Save**.
3. Filter traffic by message type to view complete TLS metadata:
   1. In the filter bar, select **TLS Types** and choose **ServerHello** from the dropdown menu.
      `ServerHello` messages contain negotiated TLS metadata such as cipher suite and cryptographic group information.
4. Filter traffic by TLS version to identify deprecated configurations:
   1. In the filter bar, select **TLS Version**.
   2. Select the versions you want to review:
      - **1.0**: Deprecated
      - **1.1**: Deprecated
      - **1.2**: Legacy
      - **1.3**: Current standard
        To identify all deprecated connections, filter for TLS versions 1.0 and 1.1.
5. Analyze TLS metrics in the overview panel:
   1. Click the **Overview** tab.
   2. Review the default TLS panels, which include **TLS usage (network flows per second)** and **TLS per version (network flows per second)**.
   3. Optional: To view additional TLS metrics, click **Manage panels** to select and display additional panels, such as **TLS per group (network flows per second)** or **TLS per cipher suite (network flows per second)**.
6. Identify secure connections in the **Topology** view:
   1. Click the **Topology** tab.
      Connections secured with TLS are marked with a lock icon. The color of the lock icon indicates the security level:

      - **Red**: Deprecated TLS versions (1.0 or 1.1)
      - **Yellow**: Legacy configurations (TLS 1.2)
      - **Green**: Secure connections (TLS 1.3)
      - **Blue**: Post-Quantum Cryptography (PQC) compliant
        Select a connection node to view its specific TLS version and cipher suite details.
7. View TLS metrics in the Network Observability dashboard:
   1. Navigate to **Observe** → **Dashboards**.
   2. Search for **NetObserv** and review the available metrics:
      - **TLS Traffic**: Displays overall TLS traffic metrics.
      - **Flows rate per TLS version**: Displays traffic trends by TLS version over time.
      - **Flows rate per TLS group**: Displays traffic by TLS group over time.

## Transport Layer Security tracking fields reference {#tls-tracking-fields_network-observability-monitoring-tls-traffic}

Transport Layer Security (TLS) metadata fields track and define encryption protocols, protocol versions, and cipher suite data to help you analyze secure network flows.

**TLS tracking fields**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Description</th>
  <th>Possible values</th>
  <th>Availability</th>
</tr>
</thead>
<tbody>
<tr>
  <td><strong>TLS Version</strong></td>
  <td>Negotiated TLS protocol version.</td>
  <td><ul><li><code>1.0</code>: Deprecated</li><li><code>1.1</code>: Deprecated</li><li><code>1.2</code>: Secure</li><li><code>1.3</code>: Current standard</li></ul></td>
  <td><code>ClientHello</code>, <code>ServerHello</code><br><br><code>ClientHello</code> displays the version requested by the client. <code>ServerHello</code> displays the negotiated version selected by the server.</td>
</tr>
<tr>
  <td><strong>TLS Cipher Suite</strong></td>
  <td>Cryptographic algorithm suite negotiated between the client and server.</td>
  <td>Examples:<br><br><ul><li><code>TLS_AES_256_GCM_SHA384</code></li><li><code>TLS_CHACHA20_POLY1305_SHA256</code></li></ul></td>
  <td><code>ServerHello</code> only<br><br>Displays as <code>n/a</code> in <code>ClientHello</code> messages.</td>
</tr>
<tr>
  <td><strong>TLS Group</strong></td>
  <td>Elliptic curve used for key exchange.</td>
  <td>Examples:<br><br><ul><li><code>X25519</code>: Recommended for TLS 1.3</li><li><code>secp256r1</code> (P-256)</li></ul></td>
  <td><code>ServerHello</code> (TLS 1.3 only)<br><br>Displays as <code>n/a</code> in <code>ClientHello</code> messages and TLS 1.2 connections.</td>
</tr>
<tr>
  <td><strong>TLS Types</strong></td>
  <td>Type of TLS handshake message captured.</td>
  <td><ul><li><code>ClientHello</code>: Initial client request</li><li><code>ServerHello</code>: Server response</li></ul></td>
  <td>All TLS flows</td>
</tr>
</tbody>
</table>
