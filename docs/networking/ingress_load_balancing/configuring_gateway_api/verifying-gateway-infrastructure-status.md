---
title: Verifying Gateway infrastructure status
sidebar_position: 9
---

# Verifying Gateway infrastructure status

<a id="verifying-gateway-infrastructure-status"></a>

To ensure your gateway infrastructure is properly configured and functioning, review the `status` conditions of your `GatewayClass` and `Gateway` custom resources (CRs). Checking these conditions confirms that the controller has successfully programmed your underlying data plane without routing conflicts.

## GatewayClass status conditions reference {#gatewayclass-status-conditions_verifying-gateway-infrastructure-status}

To verify that your `GatewayClass` custom resource (CR) is valid and ready to provision gateways, review its `status` conditions. A healthy `GatewayClass` CR reports a status of `True` for core conditions like `Accepted` and `SupportedVersion`.

`GatewayClass`** CR status conditions**

<table>
<thead>
<tr>
  <th>Condition</th>
  <th>Status</th>
  <th>Description and common reasons</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>Accepted</code></td>
  <td><code>True</code></td>
  <td>The <code>GatewayClass</code> CR is valid and the controller has claimed it.</td>
</tr>
<tr>
  <td><code>Accepted</code></td>
  <td><code>False</code></td>
  <td>The configuration has errors or was rejected. Common reasons include:<br><br><ul><li><code>InvalidParameters</code>: Referenced parameters are invalid or not found.</li><li><code>Pending</code>: The controller has not processed the resource yet.</li><li><code>Unknown</code>: An unsupported <code>controllerName</code> was provided.</li></ul></td>
</tr>
<tr>
  <td><code>Accepted</code></td>
  <td><code>Unknown</code></td>
  <td>The <code>GatewayClass</code> CR is waiting for the controller to process it.</td>
</tr>
<tr>
  <td><code>SupportedVersion</code></td>
  <td><code>True</code></td>
  <td>The installed Gateway API version is compatible with the controller.</td>
</tr>
<tr>
  <td><code>SupportedVersion</code></td>
  <td><code>False</code></td>
  <td>There is a version mismatch. A common reason is <code>UnsupportedVersion</code>, which indicates that the custom resource definition (CRD) version does not match the controller requirements.</td>
</tr>
<tr>
  <td><code>ControllerInstalled</code></td>
  <td><code>True</code></td>
  <td>The Cluster Ingress Operator successfully installed the controller for Gateway API.</td>
</tr>
<tr>
  <td><code>ControllerInstalled</code></td>
  <td><code>False</code></td>
  <td>The installation failed.</td>
</tr>
<tr>
  <td><code>ControllerInstalled</code></td>
  <td><code>Unknown</code></td>
  <td>The controller has not started the installation yet.</td>
</tr>
<tr>
  <td><code>CRDsReady</code></td>
  <td><code>True</code></td>
  <td>The Istio CRDs are installed and actively managed by either the Cluster Ingress Operator or OLM.</td>
</tr>
<tr>
  <td><code>CRDsReady</code></td>
  <td><code>False</code></td>
  <td>The CRDs were installed by a third party or have mixed ownership, preventing the controller from managing them.</td>
</tr>
<tr>
  <td><code>CRDsReady</code></td>
  <td><code>Unknown</code></td>
  <td>The CRDs are not installed yet.</td>
</tr>
</tbody>
</table>

## Gateway and listener status conditions reference {#gateway-listener-status-conditions_verifying-gateway-infrastructure-status}

To verify that your gateway is configured in the data plane and ready to route traffic, review its gateway-level and listener-level `status` conditions. A healthy `Gateway` custom resource (CR) reports a status of `True` for its `Accepted` and `Programmed` conditions.

:::warning

The `Conflicted` listener condition uses negative polarity. This means that a status of `False` indicates a healthy state, while a status of `True` indicates an error.

:::

**Gateway-level status conditions**

<table>
<thead>
<tr>
  <th>Condition</th>
  <th>Status</th>
  <th>Description and common reasons</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>Accepted</code></td>
  <td><code>True</code></td>
  <td>The gateway configuration is valid and working properly.</td>
</tr>
<tr>
  <td><code>Accepted</code></td>
  <td><code>False</code></td>
  <td>The configuration has errors. Common reasons include:<br><br><ul><li><code>ListenersNotValid</code>: One or more listeners have issues.</li><li><code>InvalidParameters</code>: The configuration is invalid.</li></ul></td>
</tr>
<tr>
  <td><code>Accepted</code></td>
  <td><code>Unknown</code></td>
  <td>The controller has not evaluated the gateway yet.</td>
</tr>
<tr>
  <td><code>Programmed</code></td>
  <td><code>True</code></td>
  <td>The infrastructure is provisioned and the gateway is configured in the data plane, such as a load balancer or proxy.</td>
</tr>
<tr>
  <td><code>Programmed</code></td>
  <td><code>False</code></td>
  <td>Programming failed or the data plane is not ready. Common reasons include:<br><br><ul><li><code>NoResources</code>: Insufficient resources or pods unavailable.</li><li><code>Invalid</code>: Cannot apply to the data plane.</li><li><code>Pending</code></li></ul></td>
</tr>
<tr>
  <td><code>Programmed</code></td>
  <td><code>Unknown</code></td>
  <td>Programming is currently in progress.</td>
</tr>
<tr>
  <td><code>LoadBalancerReady</code></td>
  <td><code>True</code></td>
  <td>The cloud load balancer service for the gateway is successfully provisioned.</td>
</tr>
<tr>
  <td><code>LoadBalancerReady</code></td>
  <td><code>False</code></td>
  <td>The load balancer service failed to provision or is pending. Common reasons include <code>ServiceNotFound</code>, <code>LoadBalancerPending</code>, or <code>SyncLoadBalancerFailed</code>.</td>
</tr>
<tr>
  <td><code>DNSReady</code></td>
  <td><code>True</code></td>
  <td>DNS records for all listeners are functioning correctly.</td>
</tr>
<tr>
  <td><code>DNSReady</code></td>
  <td><code>False</code></td>
  <td>One or more listeners have DNS provisioning issues.</td>
</tr>
</tbody>
</table>

**Listener-level status conditions**

<table>
<thead>
<tr>
  <th>Condition</th>
  <th>Status</th>
  <th>Description and common reasons</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>Accepted</code></td>
  <td><code>True</code></td>
  <td>The listener configuration is valid and working properly.</td>
</tr>
<tr>
  <td><code>Accepted</code></td>
  <td><code>False</code></td>
  <td>The listener configuration has errors.</td>
</tr>
<tr>
  <td><code>Programmed</code></td>
  <td><code>True</code></td>
  <td>The listener is successfully configured in the data plane.</td>
</tr>
<tr>
  <td><code>Programmed</code></td>
  <td><code>False</code></td>
  <td>The listener configuration failed in the data plane.</td>
</tr>
<tr>
  <td><code>ResolvedRefs</code></td>
  <td><code>True</code></td>
  <td>All references, such as TLS certificates, are found and valid.</td>
</tr>
<tr>
  <td><code>ResolvedRefs</code></td>
  <td><code>False</code></td>
  <td>At least one reference is invalid. Common reasons include:<br><br><ul><li><code>InvalidCertificateRef</code>: A TLS certificate was not found or is invalid.</li><li><code>RefNotPermitted</code>: A cross-namespace reference is not allowed.</li></ul></td>
</tr>
<tr>
  <td><code>Conflicted</code> (Negative polarity)</td>
  <td><code>False</code></td>
  <td>Healthy state. There are no conflicts.</td>
</tr>
<tr>
  <td><code>Conflicted</code> (Negative polarity)</td>
  <td><code>True</code></td>
  <td>The listener conflicts with another listener. Common reasons include:<br><br><ul><li><code>ProtocolConflict</code>: Multiple listeners on the same port with incompatible protocols.</li><li><code>HostnameConflict</code>: Overlapping hostnames.</li></ul></td>
</tr>
<tr>
  <td><code>DNSReady</code></td>
  <td><code>True</code></td>
  <td>The DNS record for this listener's hostname is successfully provisioned in all reported zones.</td>
</tr>
<tr>
  <td><code>DNSReady</code></td>
  <td><code>False</code></td>
  <td>The DNS record failed to provision. Common reasons include <code>FailedZones</code>, <code>NoDNSZones</code>, or <code>RecordNotFound</code>.</td>
</tr>
<tr>
  <td><code>DNSReady</code></td>
  <td><code>Unknown</code></td>
  <td>The DNS status cannot be determined or is unmanaged.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>Listeners without a configured hostname will not have DNS conditions added to their <code>status</code>.</p></div></td>
</tr>
</tbody>
</table>

```yaml title="Example Gateway CR status output showing a DNS failure on one listener"
# ...
status:
  # Gateway-level conditions (LoadBalancer and aggregate DNS status)
  conditions:
  - type: LoadBalancerReady
    status: "True"
    reason: LoadBalancerProvisioned
    message: "The LoadBalancer service is provisioned"
    observedGeneration: 1
    lastTransitionTime: "2025-01-12T10:00:00Z"
  - type: DNSReady
    status: "False"
    reason: SomeListenersNotReady
    message: "One or more listeners have DNS provisioning issues"
    observedGeneration: 1
    lastTransitionTime: "2025-01-12T10:00:00Z"

  # Listener-level conditions (DNS status per listener)
  listeners:
  - name: <stage_http>
    conditions:
    - type: DNSReady
      status: "True"
      reason: NoFailedZones
      message: "The record is provisioned in all reported zones."
      observedGeneration: 1
      lastTransitionTime: "2025-01-12T10:00:00Z"
  - name: <prod_https>
    conditions:
    - type: DNSReady
      status: "False"
      reason: FailedZones
      message: "The record failed to provision in some zones: [<prod.example.com>]"
      observedGeneration: 1
      lastTransitionTime: "2025-01-12T10:00:00Z"
```

:::note

For Google Cloud installations, you can use a custom DNS solution. You must manually create a DNS record for any gateways in Gateway API. For more information, see "Installing a cluster on Google Cloud with customizations".

:::

## Querying Gateway infrastructure status using the CLI {#querying-gateway-status-cli_verifying-gateway-infrastructure-status}

To quickly check the health of your gateway infrastructure, query specific `status` fields using the OpenShift Container Platform CLI. You can validate your deployment, check route attachments, and retrieve IP addresses without parsing lengthy YAML manifests.

**Prerequisites**

- You have access to the cluster as a user with the `cluster-admin` role.
- You have installed the OpenShift CLI (`oc`).
- Your gateway is deployed in the `openshift-ingress` namespace.
- Your gateway is managed by the gateway controller (`openshift.io/gateway-controller/v1`).

**Procedure**

- Run the relevant command for the status information you need to retrieve:
  - To list all `GatewayClass` custom resources (CRs) in your cluster, run the following command:
    ```terminal
    $ oc get gatewayclass
    ```
  - To check if a specific `GatewayClass` CR has been accepted by the controller, run the following command:
    ```terminal
    $ oc get gatewayclass <gatewayclass_name> -o jsonpath='{.status.conditions[?(@.type=="Accepted")].status}'
    ```

    - `<gatewayclass_name>`: Specify the name of your gateway class.
  - To list all `Gateway` custom resources (CRs) across all namespaces, run the following command:
    ```terminal
    $ oc get gateway -A
    ```
  - To check if a specific `Gateway` CR is successfully programmed in the data plane, run the following command:
    ```terminal
    $ oc get gateway <gateway_name> -n openshift-ingress -o jsonpath='{.status.conditions[?(@.type=="Programmed")].status}'
    ```

    - `<gateway_name>`: Specify the name of your gateway.
  - To retrieve the IP address assigned to a specific `Gateway` CR, run the following command:
    ```terminal
    $ oc get gateway <gateway_name> -n openshift-ingress -o jsonpath='{.status.addresses[0].value}'
    ```

    - `<gateway_name>`: Specify the name of your gateway.
  - To check the total number of routes attached to a specific `Gateway` CR, run the following command:
    ```terminal
    $ oc get gateway <gateway_name> -n openshift-ingress -o jsonpath='{.status.listeners[*].attachedRoutes}'
    ```

    - `<gateway_name>`: Specify the name of your gateway.
  - To watch a specific `Gateway` CR for real-time status changes, run the following command:
    ```terminal
    $ oc get gateway <gateway_name> -n openshift-ingress -w
    ```

    - `<gateway_name>`: Specify the name of your gateway.

**Additional resources**

- [Installing a cluster on Google Cloud with customizations](/docs/installing/installing_gcp/installing-gcp-customizations#installation-gcp-provisioning-dns-records_installing-gcp-customizations)
