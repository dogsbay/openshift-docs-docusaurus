---
title: Network Observability API reference
sidebar_position: 19
---

# Network Observability API reference

<a id="network-observability-api-reference"></a>

The Network Observability Operator provides custom resources for configuring flow collection, defining custom metrics, and managing per-tenant configurations. See the included links to the API specifications for these resources and related documentation.

## Network observability API specifications {#network-observability-api-specifications-reference_network_observability}

Reference documentation for Network Observability custom resources and flow data formats.

### FlowCollector API {#flowcollector-api_network_observability}

The `FlowCollector` resource configures the Network Observability Operator.

- [FlowCollector API for network observability 2.0](https://github.com/openshift/network-observability-operator/blob/release-2.0/docs/flowcollector-flows-netobserv-io-v1beta2.adoc)
- [FlowCollector API for network observability 1.12](https://github.com/openshift/network-observability-operator/blob/release-1.12/docs/flowcollector-flows-netobserv-io-v1beta2.adoc)

### FlowMetric API {#flowmetric-api_network_observability}

The `FlowMetric` resource creates custom metrics from collected flow logs.

- [FlowMetric API for network observability 2.0](https://github.com/openshift/network-observability-operator/blob/release-2.0/docs/flowmetric-flows-netobserv-io-v1alpha1.adoc)
- [FlowMetric API for network observability 1.12](https://github.com/openshift/network-observability-operator/blob/release-1.12/docs/flowmetric-flows-netobserv-io-v1alpha1.adoc)

### FlowCollectorSlice API {#flowcollector-slice-api_network_observability}

The `FlowCollectorSlice` resource enables per-tenant flow collection configuration.

- [FlowCollectorSlice API for network observability 2.0](https://github.com/openshift/network-observability-operator/blob/release-2.0/docs/FlowCollectorSlice.md)
- [FlowCollectorSlice API for network observability 1.12](https://github.com/openshift/network-observability-operator/blob/release-1.12/docs/FlowCollectorSlice.md)

### Network flows format {#network-flows-format_network_observability}

The network flows format reference describes the structure and fields of flow data.

- [Network flows format reference for network observability 2.0](https://github.com/openshift/network-observability-operator/blob/release-2.0/docs/flows-format.adoc)
- [Network flows format reference for network observability 1.12](https://github.com/openshift/network-observability-operator/blob/release-1.12/docs/flows-format.adoc)

### Network Observability CLI usage {#network-observability-cli-usage_network_observability}

The Network Observability CLI (`oc netobserv`) provides feature and filtering parity with the Network Observability Operator. Use command-line arguments to dynamically toggle features and isolate specific cluster network traffic flows.

- [Network Observability CLI usage](https://github.com/netobserv/netobserv-cli/blob/main/docs/netobserv_cli.adoc)
