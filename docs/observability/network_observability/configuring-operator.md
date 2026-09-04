---
title: Configuring the Network Observability Operator
sidebar_position: 6
---

# Configuring the Network Observability Operator {#configuring-network-observability-operators}

<a id="configuring-network-observability-operators"></a>

Configure the Network Observability Operator by updating the cluster-wide `FlowCollector` API resource (cluster) to manage component configurations and flow collection settings.

The `FlowCollector` is explicitly created during installation. Since this resource operates cluster-wide, only a single `FlowCollector` is allowed, and it must be named `cluster`. For more information, see the "FlowCollector API reference".

## View the FlowCollector resource {#network-observability-flowcollector-view_network_observability}

View and modify the `FlowCollector` resource in the OpenShift Container Platform web console through the integrated setup, advanced form, or by editing the YAML directly to configure the Network Observability Operator.

**Procedure**

1. In the web console, navigate to **Ecosystem** → **Installed Operators**.
2. Under the **Provided APIs** heading for the **NetObserv Operator**, select **Flow Collector**.
3. Select **cluster** then select the **YAML** tab. There, you can modify the `FlowCollector` resource to configure the Network Observability Operator.

### Example of a FlowCollector resource {#network-observability-flowcollector-example_network_observability}

Review a comprehensive, annotated example of the `FlowCollector` custom resource that demonstrates configurations for `eBPF` sampling, conversation tracking, Loki integration, and console quick filters.

#### Sample `FlowCollector` resource {#network-observability-flowcollector-configuring-about-sample_network_observability}

```yaml
apiVersion: flows.netobserv.io/v1beta2
kind: FlowCollector
metadata:
  name: cluster
spec:
  namespace: netobserv
  deploymentModel: Service
  networkPolicy:
    enable: true
  agent:
    type: eBPF
    ebpf:
      sampling: 50
      privileged: false
      features: []
  processor:
    addZone: false
    subnetLabels:
      openShiftAutoDetect: true
      customLabels: []
    consumerReplicas: 3
  loki:
    enable: true
    mode: LokiStack
    lokiStack:
      name: loki
      namespace: netobserv-loki
  consolePlugin:
    enable: true
  exporters: []
```

where:

`spec.agent.type`
: Must be `eBPF` as eBPF is the only OpenShift Container Platform supported option.

`spec.agent.ebpf.sampling`
: Specifies the sampling interval. By default, eBPF sampling is set to `50`, so a packet has a 1 in 50 chance of being sampled. A lower sampling interval value requires more computational, memory, and storage resources. A value of `0` or `1` means all packets are sampled. It is recommended to start with the default value and refine it empirically to determine the optimal setting for your cluster.

`spec.agent.ebpf.privileged`
: Specifies if the eBPF agent pods should run as privileged. Running as privileged is required for several features, such as monitoring non-default networks and tracking packet drops. For security, in accordance with the principle of least privilege, it should only be enabled when some of those features are desired. A warning will be displayed if you enabled a feature requiring privileged mode without setting it to true explicitly.

`spec.processor.addZone`
: Used to inject cloud availability zones in network flows.

`spec.processor.subnetLabels`
: Specifies a list of customized labels to inject in network flows, based on CIDR matching.

`spec.processor.consumerReplicas`
: Specifies the number of replicas for the processor pods (flowlogs-pipeline). Refer to the Resource management and performance considerations section for recommendations based on the cluster size.

`spec.loki.mode`
: Specifies how to configure the connection to Loki, depending on its installation mode. If you use the install paths described in "Installing the Loki Operator", the mode must be set to `LokiStack`, and `spec.loki.lokiStack` should refer to the installed `LokiStack` resource name and namespace.

`spec.loki.lokistack.namespace`
: Specifies the namespace for the `LokiStack` resource. This value must match the `metadata.namespace` defined in the `LokiStack` custom resource. While this example uses `netobserv-loki`, you can use a different namespace for different components.

**Additional resources**

- [FlowCollector API reference](/docs/observability/network_observability/flowcollector-api#network-observability-flowcollector-api-specifications_network_observability)
- [Working with conversation tracking](/docs/observability/network_observability/observing-network-traffic#network-observability-working-with-conversations_nw-observe-network-traffic)

## Grant permissions for custom namespace and secret access {#network-observability-grant-permissions-custom-namespace-and-secret-access_network_observability}

Grant RBAC permissions to allow the Network Observability Operator to access secrets across custom or non-default namespaces.

For security, the Network Observability Operator does not have cluster-wide permissions to read secrets. You must explicitly grant the required permissions when any of the following conditions apply:

- You configured `spec.namespace` in the `FlowCollector` resource to a namespace other than `netobserv`.
- You configured the `FlowCollector` resource with `spec.deploymentModel: Kafka` and TLS or mTLS enabled.
- You use the Loki Operator with `LokiStack` installed in a namespace other than `netobserv`.

Complete only the procedures that apply to your configuration. If you do not meet any of these conditions, no additional permissions are required.

### Create cluster role bindings for a custom namespace {#network-observability-create-cluster-role-bindings-custom-namespace_network_observability}

Create cluster role bindings for the Network Observability Operator service accounts when you deploy in a namespace other than the default `netobserv` namespace.

:::warning

Do not modify the existing default bindings. They are overwritten during Operator upgrades. Create new bindings as shown.

:::

**Prerequisites**

- The Network Observability Operator is installed.
- You have `cluster-admin` privileges.
- You configured `spec.namespace` in the `FlowCollector` resource to a namespace other than `netobserv`.

**Procedure**

1. Replace `<namespace>` with the namespace you configured in `spec.namespace` of the `FlowCollector` resource.
   1. Create the `netobserv-informers-custom` cluster role binding by running the following command:
      ```terminal
      $ oc create clusterrolebinding netobserv-informers-custom \
        --clusterrole=netobserv-informers \
        --serviceaccount=<namespace>:flowlogs-pipeline \
        --serviceaccount=<namespace>:flowlogs-pipeline-transformer
      ```
   2. Create the `netobserv-lokiwriter-custom` cluster role binding by running the following command:
      ```terminal
      $ oc create clusterrolebinding netobserv-lokiwriter-custom \
        --clusterrole=netobserv-loki-writer \
        --serviceaccount=<namespace>:flowlogs-pipeline \
        --serviceaccount=<namespace>:flowlogs-pipeline-transformer
      ```
   3. Create the `netobserv-hostnetwork-custom` cluster role binding by running the following command:
      ```terminal
      $ oc create clusterrolebinding netobserv-hostnetwork-custom \
        --clusterrole=netobserv-hostnetwork \
        --serviceaccount=<namespace>:flowlogs-pipeline
      ```
   4. Create the token review cluster role binding by running the following command:
      ```terminal
      $ oc create clusterrolebinding netobserv-tokenreview-custom \
        --clusterrole=netobserv-token-review \
        --serviceaccount=<namespace>:netobserv-plugin
      ```

**Verification**

1. Check the `FlowCollector` status for errors by running the following command:
   ```terminal
   $ oc get flowcollector cluster -o jsonpath='{.status.conditions}'
   ```
2. Verify that no conditions report permission-related errors.
   :::note

   If the `FlowCollector` status continues to show permission errors after you grant the required permissions, restart the Network Observability Operator pod for faster reconciliation:

   ```terminal
   $ oc delete pods -n openshift-netobserv-operator -l app=netobserv-operator
   ```

   :::

### Grant access to Kafka secrets {#network-observability-grant-access-kafka-secrets_network_observability}

Grant the Network Observability Operator permission to access Kafka secrets when you use the Kafka deployment model with TLS or mTLS enabled.

**Prerequisites**

- The Network Observability Operator is installed.
- You have `cluster-admin` privileges.
- You configured the `FlowCollector` resource with `spec.deploymentModel: Kafka` and TLS or mTLS enabled.

**Procedure**

1. Replace `<namespace>` with the namespace configured in `spec.namespace` of the `FlowCollector` resource. The default value is `netobserv`.
   1. If Kafka is installed in the same namespace as the Network Observability components, create the secret watcher role binding by running the following command:
      ```terminal
      $ oc create rolebinding secret-watcher \
        -n <namespace> \
        --clusterrole=netobserv-secret-watcher \
        --serviceaccount=openshift-netobserv-operator:netobserv-controller-manager
      ```
   2. Create the secret creator role binding in the privileged namespace by running the following command:
      ```terminal
      $ oc create rolebinding secret-creator \
        -n <namespace>-privileged \
        --clusterrole=netobserv-secret-creator \
        --serviceaccount=openshift-netobserv-operator:netobserv-controller-manager
      ```
2. If Kafka is installed in a different namespace, create the secret watcher role binding in the Kafka namespace.
   1. Replace `<kafka_namespace>` with the namespace where Kafka is installed by running the following command:
      ```terminal
      $ oc create rolebinding secret-watcher \
        -n <kafka_namespace> \
        --clusterrole=netobserv-secret-watcher \
        --serviceaccount=openshift-netobserv-operator:netobserv-controller-manager
      ```
   2. Create the secret creator role binding by running the following command:
      ```terminal
      $ oc create rolebinding secret-creator \
        -n <namespace> \
        --clusterrole=netobserv-secret-creator \
        --serviceaccount=openshift-netobserv-operator:netobserv-controller-manager
      ```
   3. Create the secret creator role binding in the privileged namespace by running the following command:
      ```terminal
      $ oc create rolebinding secret-creator \
        -n <namespace>-privileged \
        --clusterrole=netobserv-secret-creator \
        --serviceaccount=openshift-netobserv-operator:netobserv-controller-manager
      ```

**Verification**

1. Check the `FlowCollector` status for errors by running the following command:
   ```terminal
   $ oc get flowcollector cluster -o jsonpath='{.status.conditions}'
   ```
2. Verify that no conditions report permission-related errors.
   :::note

   If the `FlowCollector` status continues to show permission errors after you grant the required permissions, restart the Network Observability Operator pod for faster reconciliation:

   ```terminal
   $ oc delete pods -n openshift-netobserv-operator -l app=netobserv-operator
   ```

   :::

### Grant access to LokiStack secrets {#network-observability-grant-access-lokistack-secrets_network_observability}

Grant the Network Observability Operator permission to access the `LokiStack` secret when `LokiStack` is installed in a namespace other than `netobserv`.

**Prerequisites**

- The Network Observability Operator is installed.
- You have `cluster-admin` privileges.
- You use the Loki Operator with `LokiStack` installed in a namespace other than `netobserv`.

**Procedure**

1. Replace `<lokistack_namespace>` with the namespace where `LokiStack` is installed and `<namespace>` with the namespace configured in `spec.namespace` of the `FlowCollector` resource.
   1. Create the `secret-watcher` role binding in the LokiStack namespace by running the following command:
      ```terminal
      $ oc create rolebinding secret-watcher \
        -n <lokistack_namespace> \
        --clusterrole=netobserv-secret-watcher \
        --serviceaccount=openshift-netobserv-operator:netobserv-controller-manager
      ```
   2. Create the `secret-creator` role binding by running the following command:
      ```terminal
      $ oc create rolebinding secret-creator \
        -n <namespace> \
        --clusterrole=netobserv-secret-creator \
        --serviceaccount=openshift-netobserv-operator:netobserv-controller-manager
      ```

**Verification**

1. Check the `FlowCollector` status for errors by running the following command:
   ```terminal
   $ oc get flowcollector cluster -o jsonpath='{.status.conditions}'
   ```
2. Verify that no conditions report permission-related errors.
   :::note

   If the `FlowCollector` status continues to show permission errors after you grant the required permissions, restart the Network Observability Operator pod for faster reconciliation:

   ```terminal
   $ oc delete pods -n openshift-netobserv-operator -l app=netobserv-operator
   ```

   :::

## Configuring the FlowCollector resource with Kafka {#network-observability-flowcollector-kafka-config_network_observability}

Configure the `FlowCollector` resource to use Kafka for high-throughput and low-latency data feeds.

You must have a running Kafka instance and create a Kafka topic in that instance dedicated to OpenShift Container Platform Network Observability. For more information, see [Kafka documentation with AMQ Streams](https://access.redhat.com/documentation/en-us/red_hat_amq/7.7/html/using_amq_streams_on_openshift/using-the-topic-operator-str).

**Prerequisites**

- You have installed Kafka. Red Hat supports Kafka with AMQ Streams Operator.

**Procedure**

1. In the web console, navigate to **Ecosystem** → **Installed Operators**.
2. Under the **Provided APIs** heading for the Network Observability Operator, select **Flow Collector**.
3. Select the cluster and then click the **YAML** tab.
4. Change the `FlowCollector` resource for OpenShift Container Platform Network Observability Operator to use Kafka, as shown in the following sample YAML:
   ```yaml title="Sample Kafka configuration in FlowCollector resource"
   apiVersion: flows.netobserv.io/v1beta2
   kind: FlowCollector
   metadata:
     name: cluster
   spec:
     deploymentModel: Kafka
     kafka:
       address: "kafka-cluster-kafka-bootstrap.netobserv"
       topic: network-flows
       tls:
         enable: false
   ```

   where:

   <dl>
   <dt><code>spec.deploymentModel</code></dt>
   <dd>Specifies the deployment model. Set to <code>Kafka</code> instead of <code>Service</code> to enable the Kafka deployment model.</dd>
   <dt><code>spec.kafka.address</code></dt>
   <dd>Specifies the Kafka bootstrap server address. You can specify a port if needed, for instance <code>kafka-cluster-kafka-bootstrap.netobserv:9093</code> for using TLS on port 9093.</dd>
   <dt><code>spec.kafka.topic</code></dt>
   <dd>Specifies the name of the topic created in Kafka. It should match the name of a topic created in Kafka.</dd>
   <dt><code>spec.kafka.tls</code></dt>
   <dd>Specifies communication encryption. Use this setting to encrypt all communications to and from Kafka with TLS or mTLS. When enabled, the Kafka CA certificate must be available as a ConfigMap or a Secret in both namespaces: the namespace where you deploy the <code>flowlogs-pipeline</code> processor component (default: <code>netobserv</code>) and the namespace where you deploy the eBPF agents (default: <code>netobserv-privileged</code>). Reference the certificate by using <code>spec.kafka.tls.caCert</code>. When you use mTLS, make the client secrets available in these namespaces as well. You can generate the secrets by using the Red Hat AMQ Streams User Operator. Reference the secrets by using <code>spec.kafka.tls.userCert</code>.</dd>
   </dl>

## Export enriched network flow data {#network-observability-enriched-flows_network_observability}

Configure the `FlowCollector` resource to export enriched network flow data simultaneously to Kafka, IPFIX, or an OpenTelemetry endpoint for external consumption by tools like Splunk or Prometheus.

For Kafka or IPFIX, any processor or storage that supports those inputs, such as Splunk, Elasticsearch, or Fluentd, can consume the enriched network flow data.

For OpenTelemetry, network flow data and metrics can be exported to a compatible OpenTelemetry endpoint, such as Red Hat build of OpenTelemetry or Prometheus.

After configuration, network flows data can be sent to an available output. For more information, see "Network flows format reference".

**Prerequisites**

- Your Kafka, IPFIX, or OpenTelemetry collector endpoints are available from Network Observability `flowlogs-pipeline` pods.

**Procedure**

1. In the web console, navigate to **Ecosystem** → **Installed Operators**.
2. Under the **Provided APIs** heading for the **NetObserv Operator**, select **Flow Collector**.
3. Select **cluster** and then select the **YAML** tab.
4. Edit the `FlowCollector` to configure `spec.exporters` as follows:
   ```yaml
   apiVersion: flows.netobserv.io/v1beta2
   kind: FlowCollector
   metadata:
     name: cluster
   spec:
     exporters:
     - type: Kafka
         kafka:
           address: "kafka-cluster-kafka-bootstrap.netobserv"
           topic: netobserv-flows-export
           tls:
             enable: false
     - type: IPFIX
         ipfix:
           targetHost: "ipfix-collector.ipfix.svc.cluster.local"
           targetPort: 4739
           transport: tcp
    -  type: OpenTelemetry
         openTelemetry:
           targetHost: my-otelcol-collector-headless.otlp.svc
           targetPort: 4317
           type: grpc
           logs:
             enable: true
           metrics:
             enable: true
             prefix: netobserv
             pushTimeInterval: 20s
             expiryTime: 2m
      #    fieldsMapping:
      #      input: SrcAddr
      #      output: source.address
   ```

   where:

   <dl>
   <dt><code>spec.exporters.type</code></dt>
   <dd>Specifies the export type. You can export flows to <code>IPFIX</code>, <code>OpenTelemetry</code>, and <code>Kafka</code> individually or concurrently.</dd>
   <dt><code>spec.exporters.kafka.topic</code></dt>
   <dd>Specifies the Kafka topic where the Network Observability Operator exports all flows.</dd>
   <dt><code>spec.exporters.kafka.tls.enable</code></dt>
   <dd>Specifies whether to encrypt communications to and from Kafka with SSL/TLS or mTLS. When enabled, the Kafka CA certificate must be available as a <code>ConfigMap</code> or a <code>Secret</code> in the namespace where the <code>flowlogs-pipeline</code> processor component is deployed (default: <code>netobserv</code>). Reference the certificate with <code>spec.exporters.tls.caCert</code>. For mTLS, client secrets must also be available in these namespaces and referenced with <code>spec.exporters.tls.userCert</code>.</dd>
   <dt><code>spec.exporters.ipfix.transport</code></dt>
   <dd>Specifies the transport protocol. The default value is <code>tcp</code>, but you can also specify <code>udp</code>.</dd>
   <dt><code>spec.exporters.openTelemetry.type</code></dt>
   <dd>Specifies the OpenTelemetry connection protocol. The available options are <code>http</code> and <code>grpc</code>.</dd>
   <dt><code>spec.exporters.openTelemetry.logs</code></dt>
   <dd>Specifies the OpenTelemetry configuration for exporting logs, which are identical to the logs created for Loki.</dd>
   <dt><code>spec.exporters.openTelemetry.metrics</code></dt>
   <dd>Specifies the OpenTelemetry configuration for exporting metrics, which are identical to the metrics created for Prometheus. These are defined in the <code>spec.processor.metrics.includeList</code> parameter of the <code>FlowCollector</code> resource or via the <code>FlowMetrics</code> resource.</dd>
   <dt><code>spec.exporters.openTelemetry.metrics.pushTimeInterval</code></dt>
   <dd>Specifies the time interval for sending metrics to the OpenTelemetry collector.</dd>
   <dt><code>spec.exporters.openTelemetry.fieldsMapping</code></dt>
   <dd>Specifies an optional mapping to customize the OpenTelemetry format output. Network Observability flow formats are automatically renamed to an OpenTelemetry-compliant format, but this parameter allows for custom overrides. For example in the YAML sample, <code>SrcAddr</code> is the Network Observability input field, and it is being renamed to <code>source.address</code> in OpenTelemetry output. You can see both Network Observability and OpenTelemetry formats in the "Network flows format reference".</dd>
   </dl>

**Additional resources**

- [Network flows format reference](/docs/observability/network_observability/json-flows-format-reference#network-observability-flows-format_json_reference)

## Updating the FlowCollector resource {#network-observability-config-FLP-sampling_network_observability}

As an alternative to using the web console, use the `oc patch` command with the `flowcollector` custom resource to quickly update specific specifications, such as eBPF sampling

**Procedure**

1. Run the following command to patch the `flowcollector` CR and update the `spec.agent.ebpf.sampling` value:
   ```terminal
   $ oc patch flowcollector cluster --type=json -p "[{"op": "replace", "path": "/spec/agent/ebpf/sampling", "value": <new value>}] -n netobserv"
   ```

## Filter network flows at ingestion {#network-observability-filter-network-flows-at-ingestion_network_observability}

Create filters to reduce the number of generated network flows. Filtering network flows can reduce the resource usage of the network observability components.

You can configure two kinds of filters:

- eBPF agent filters
- Flowlogs-pipeline filters

### eBPF agent filters {#ebpf-agent-filters_network_observability}

eBPF agent filters maximize performance because they take effect at the earliest stage of the network flows collection process.

To configure eBPF agent filters with the Network Observability Operator, see "Filtering eBPF flow data using multiple rules".

### Flowlogs-pipeline filters {#flowlogs-pipeline-filters_network_observability}

Flowlogs-pipeline filters provide greater control over traffic selection because they take effect later in the network flows collection process. They are primarily used to improve data storage.

Flowlogs-pipeline filters use a simple query language to filter network flow, as shown in the following example:

```terminal
(srcnamespace="netobserv" OR (srcnamespace="ingress" AND dstnamespace="netobserv")) AND srckind!="service"
```

The query language uses the following syntax:

**Query language syntax**

| Category | Operators |
| --- | --- |
| Logical boolean operators (not case-sensitive) | `and`, `or` |
| Comparison operators | `=` (equals),<br>`!=` (not equals),<br>`=~` (matches regexp),<br>`!~` (not matches regexp),<br>`<` / `\<=` (less than or equal to),<br>`>` / `>=` (greater than or equal to) |
| Unary operations | `with(field)` (field is present),<br>`without(field)` (field is absent) |

You can configure flowlogs-pipeline filters in the `spec.processor.filters` section of the `FlowCollector` resource. For example:

```yaml title="Example YAML Flowlogs-pipeline filter"
apiVersion: flows.netobserv.io/v1beta2
kind: FlowCollector
metadata:
  name: cluster
spec:
  namespace: netobserv
  agent:
  processor:
    filters:
      - query: |
          (SrcK8S_Namespace="netobserv" OR (SrcK8S_Namespace="openshift-ingress" AND DstK8S_Namespace="netobserv"))
        outputTarget: Loki
        sampling: 10
```

where:

`spec.processor.filters.outputTarget`
: Specifies the output destination for matching flows, such as `Loki`, `Prometheus`, or an external system. If you omit this parameter, the system sends the flows to all configured outputs.

`spec.processor.filters.sampling`
: Specifies an optional sampling interval to limit the number of matching flows stored or exported. For example, a value of `10` means there is a 1 in 10 chance that a flow is kept.

**Additional resources**

- [Filtering eBPF flow data using multiple rules](/docs/observability/network_observability/observing-network-traffic#network-observability-filtering-ebpf-rule_nw-observe-network-traffic)

## Configuring quick filters {#network-observability-config-quick-filters_network_observability}

Use the list of available source, destination, and universal filter keys to modify quick filters within the `FlowCollector` resource.

Exact matches are possible using double-quotes around values. Otherwise, partial matches are used for textual values. The bang (!) character, placed at the end of a key, means negation. See the sample `FlowCollector` resource for more context about modifying the YAML.

:::note

The filter matching types "all of" or "any of" is a UI setting that the users can modify from the query options. It is not part of this resource configuration.

:::

Here is a list of all available filter keys:

**Filter keys**

<table>
<thead>
<tr>
  <th>Universal*</th>
  <th>Source</th>
  <th>Destination</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>namespace</td>
  <td><code>src_namespace</code></td>
  <td><code>dst_namespace</code></td>
  <td>Filter traffic related to a specific namespace.</td>
</tr>
<tr>
  <td>name</td>
  <td><code>src_name</code></td>
  <td><code>dst_name</code></td>
  <td>Filter traffic related to a given leaf resource name, such as a specific pod, service, or node (for host-network traffic).</td>
</tr>
<tr>
  <td>kind</td>
  <td><code>src_kind</code></td>
  <td><code>dst_kind</code></td>
  <td>Filter traffic related to a given resource kind. The resource kinds include the leaf resource (Pod, Service or Node), or the owner resource (Deployment and StatefulSet).</td>
</tr>
<tr>
  <td>owner_name</td>
  <td><code>src_owner_name</code></td>
  <td><code>dst_owner_name</code></td>
  <td>Filter traffic related to a given resource owner; that is, a workload or a set of pods. For example, it can be a Deployment name, a StatefulSet name, etc.</td>
</tr>
<tr>
  <td>resource</td>
  <td><code>src_resource</code></td>
  <td><code>dst_resource</code></td>
  <td>Filter traffic related to a specific resource that is denoted by its canonical name, that identifies it uniquely. The canonical notation is <code>kind.namespace.name</code> for namespaced kinds, or <code>node.name</code> for nodes. For example, <code>Deployment.my-namespace.my-web-server</code>.</td>
</tr>
<tr>
  <td>address</td>
  <td><code>src_address</code></td>
  <td><code>dst_address</code></td>
  <td>Filter traffic related to an IP address. IPv4 and IPv6 are supported. CIDR ranges are also supported.</td>
</tr>
<tr>
  <td>mac</td>
  <td><code>src_mac</code></td>
  <td><code>dst_mac</code></td>
  <td>Filter traffic related to a MAC address.</td>
</tr>
<tr>
  <td>port</td>
  <td><code>src_port</code></td>
  <td><code>dst_port</code></td>
  <td>Filter traffic related to a specific port.</td>
</tr>
<tr>
  <td>host_address</td>
  <td><code>src_host_address</code></td>
  <td><code>dst_host_address</code></td>
  <td>Filter traffic related to the host IP address where the pods are running.</td>
</tr>
<tr>
  <td>protocol</td>
  <td>N/A</td>
  <td>N/A</td>
  <td>Filter traffic related to a protocol, such as TCP or UDP.</td>
</tr>
</tbody>
</table>

- Universal keys filter for any of source or destination. For example, filtering `name: 'my-pod'` means all traffic from `my-pod` and all traffic to `my-pod`, regardless of the matching type used, whether **Match all** or **Match any**.

## Resource management and performance considerations {#network-observability-resource-recommendations_network_observability}

Review the key configuration settings, including eBPF sampling, feature enablement, and resource limits, necessary to manage performance criteria and optimize resource consumption for network observability.

The amount of resources required by network observability depends on the size of your cluster and your requirements for the cluster to ingest and store observability data. To manage resources and set performance criteria for your cluster, consider configuring the following settings. Configuring these settings might meet your optimal setup and observability needs.

The following settings can help you manage resources and performance from the outset:

eBPF Sampling
: You can set the Sampling specification, `spec.agent.ebpf.sampling`, to manage resources. By default, eBPF sampling is set to `50`, so a flow has a 1 in 50 chance of being sampled. A lower sampling interval value requires more computational, memory, and storage resources. A value of `0` or `1` means all flows are sampled. It is recommended to start with the default value and refine it empirically to determine the optimal setting for your cluster.

eBPF features
: The more features that are enabled, the more CPU and memory are impacted. See "Observing the network traffic" for a complete list of these features.

Without Loki
: You can reduce the amount of resources that network observability requires by not using Loki and instead relying on Prometheus. For example, when network observability is configured without Loki, the total savings of memory usage are in the 20-65% range and CPU utilization is lower by 10-30%, depending upon the sampling interval value. See "Network observability without Loki" for more information.

Restricting or excluding interfaces
: Reduce the overall observed traffic by setting the values for `spec.agent.ebpf.interfaces` and `spec.agent.ebpf.excludeInterfaces`. By default, the agent fetches all the interfaces in the system, except the ones listed in `excludeInterfaces` and `lo` (local interface). Note that the interface names might vary according to the Container Network Interface (CNI) used.

Performance fine-tuning
: The following settings can be used to fine-tune performance after the Network Observability has been running for a while:

  - **Resource requirements and limits**: Adapt the resource requirements and limits to the load and memory usage you expect on your cluster by using the `spec.agent.ebpf.resources` and `spec.processor.resources` specifications. The default limits of 800MB might be sufficient for most medium-sized clusters.
  - **Cache max flows timeout**: Control how often flows are reported by the agents by using the eBPF agent’s `spec.agent.ebpf.cacheMaxFlows` and `spec.agent.ebpf.cacheActiveTimeout` specifications. A larger value results in less traffic being generated by the agents, which correlates with a lower CPU load. However, a larger value leads to a slightly higher memory consumption, and might generate more latency in the flow collection.

### Resource considerations {#network-observability-resources-table_network_observability}

The Network Observability Operator configuration can be adjusted based on the cluster workload size. Use the following baseline examples to determine the appropriate resource limits and configuration settings for the environment.

The examples outlined in the table demonstrate scenarios that are tailored to specific workloads. Consider each example only as a baseline from which adjustments can be made to accommodate your workload needs.

The test beds used for these recommendations are:

- Extra small: 10-node cluster, 4 vCPUs and 16 GiB memory per worker, `LokiStack` size `1x.extra-small`, tested on AWS M6i instances.
- Small: 25-node cluster, 16 vCPUs and 64 GiB memory per worker, `LokiStack` size `1x.small`, tested on AWS M6i instances.
- Large: 250-node cluster, 16 vCPUs and 64 GiB memory per worker, `LokiStack` size `1x.medium`, tested on AWS M6i instances. In addition to the worker and controller nodes, three infrastructure nodes (size `M6i.12xlarge`) and one workload node (size `M6i.8xlarge`) were tested.

**Resource recommendations for cluster sizes**

| Criterion | Extra small (10 nodes) | Small (25 nodes) | Large (250 nodes) |
| --- | --- | --- | --- |
| Operator memory limit: `Subscription` `spec.config.resources` | `400Mi` (default) | `400Mi` (default) | `400Mi` (default) |
| eBPF agent sampling interval: `FlowCollector` `spec.agent.ebpf.sampling` | `50` (default) | `50` (default) | `50` (default) |
| eBPF agent memory limit: `FlowCollector` `spec.agent.ebpf.resources` | `800Mi` (default) | `800Mi` (default) | `1600Mi` |
| eBPF agent cache size: `FlowCollector` `spec.agent.ebpf.cacheMaxSize` | `50,000` | `120,000` (default) | `120,000` (default) |
| Processor memory limit: `FlowCollector` `spec.processor.resources` | `800Mi` (default) | `800Mi` (default) | `800Mi` (default) |
| Processor replicas: `FlowCollector` `spec.processor.consumerReplicas` | `3` (default) | `6` | `18` |
| Deployment model: `FlowCollector` `spec.deploymentModel` | `Service` (default) | `Kafka` | `Kafka` |
| Kafka partitions: Kafka installation | N/A | `48` | `48` |
| Kafka brokers: Kafka installation | N/A | `3` (default) | `3` (default) |

### Total average memory and CPU usage {#network-observability-total-resource-usage-table_network_observability}

Review the table detailing the total average CPU and memory usage for network observability components under two distinct traffic scenarios (`Test 1` and `Test 2`) at different eBPF sampling values.

The following table outlines averages of total resource usage for clusters with a sampling value of `1` and `50` for two different tests: `Test 1` and `Test 2`. The tests differ in the following ways:

- `Test 1` takes into account high ingress traffic volume in addition to the total number of namespace, pods and services in an OpenShift Container Platform cluster, places load on the eBPF agent, and represents use cases with a high number of workloads for a given cluster size. For example, `Test 1` consists of 76 Namespaces, 5153 Pods, and 2305 Services with a network traffic scale of ~350 MB/s.
- `Test 2` takes into account high ingress traffic volume in addition to the total number of namespace, pods and services in an OpenShift Container Platform cluster and represents use cases with a high number of workloads for a given cluster size. For example, `Test 2` consists of 553 Namespaces, 6998 Pods, and 2508 Services with a network traffic scale of ~950 MB/s.

Since different types of cluster use cases are exemplified in the different tests, the numbers in this table do not scale linearly when compared side-by-side. Instead, they are intended to be used as a benchmark for evaluating your personal cluster usage. The examples outlined in the table demonstrate scenarios that are tailored to specific workloads. Consider each example only as a baseline from which adjustments can be made to accommodate your workload needs.

:::note

Metrics exported to Prometheus can impact the resource usage. Cardinality values for the metrics can help determine how much resources are impacted. For more information, see "Network Flows format" in the Additional resources section.

:::

**Total average resource usage**

<table>
<thead>
<tr>
  <th>Sampling value</th>
  <th>Resources used</th>
  <th>Test 1 (25 nodes)</th>
  <th>Test 2 (250 nodes)</th>
</tr>
</thead>
<tbody>
<tr>
  <td rowspan="2"><strong>Sampling = 50</strong></td>
  <td>Total NetObserv CPU Usage</td>
  <td>1.35</td>
  <td>5.39</td>
</tr>
<tr>
  <td>Total NetObserv RSS (Memory) Usage</td>
  <td>16 GB</td>
  <td>63 GB</td>
</tr>
<tr>
  <td rowspan="2"><strong>Sampling = 1</strong></td>
  <td>Total NetObserv CPU Usage</td>
  <td>1.82</td>
  <td>11.99</td>
</tr>
<tr>
  <td>Total NetObserv RSS (Memory) Usage</td>
  <td>22 GB</td>
  <td>87 GB</td>
</tr>
</tbody>
</table>

Summary: This table shows average total resource usage of Network Observability, which includes Agents, FLP, Kafka, and Loki with all features enabled. For details about what features are enabled, see the features covered in "Observing the network traffic", which comprises all the features that are enabled for this testing.

**Additional resources**

- [Observing the network traffic from the traffic flows view](/docs/observability/network_observability/observing-network-traffic#network-observability-trafficflow_nw-observe-network-traffic)
- [Network observability without Loki](/docs/observability/network_observability/installing-operators#network-observability-without-loki_network_observability)
- [Network Flows format reference](/docs/observability/network_observability/json-flows-format-reference#network-observability-flows-format_json_reference)
