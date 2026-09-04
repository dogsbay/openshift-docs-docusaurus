---
title: Network Observability CLI (oc netobserv) reference
sidebar_position: 3
---

# Network Observability CLI (oc netobserv) reference {#netobserv-cli-reference}

<a id="netobserv-cli-reference"></a>

The Network Observability CLI (`oc netobserv`) provides feature and filtering parity with the Network Observability Operator. Use command-line arguments to dynamically toggle features and isolate specific cluster network traffic flows.

## Network Observability CLI usage {#network-observability-netobserv-cli-reference_netobserv-cli-reference}

You can use the Network Observability CLI (`oc netobserv`) to pass command line arguments to capture flows data, packets data, and metrics for further analysis and enable features supported by the Network Observability Operator.

### Syntax {#cli-syntax_netobserv-cli-reference}

The basic syntax for `oc netobserv` commands:

```terminal title="oc netobserv syntax"
$ oc netobserv [<command>] [<feature_option>] [<command_options>] (1)
```

1. Feature options can only be used with the `oc netobserv flows` command. They cannot be used with the `oc netobserv packets` command.

### Basic commands {#cli-basic-commands_netobserv-cli-reference}

**Basic commands**

| Command | Description |
| --- | --- |
| flows | Capture flows information. For subcommands, see the "Flows capture options" table. |
| packets | Capture packets data. For subcommands, see the "Packets capture options" table. |
| metrics | Capture metrics data. For subcommands, see the "Metrics capture options" table. |
| follow | Follow collector logs when running in background. |
| stop | Stop collection by removing agent daemonset. |
| copy | Copy collector generated files locally. |
| cleanup | Remove the Network Observability CLI components. |
| version | Print the software version. |
| help | Show help. |

### Flows capture options {#cli-reference-flows-capture-options_netobserv-cli-reference}

Flows capture has mandatory commands as well as additional options, such as enabling extra features about packet drops, DNS latencies, Round-trip time, and filtering.

```terminal title="oc netobserv flows syntax"
$ oc netobserv flows [<feature_option>] [<command_options>]
```

| Option | Description | Default |
| --- | --- | --- |
| --enable_all | enable all eBPF features | false |
| --enable_dns | enable DNS tracking | false |
| --enable_ipsec | enable IPsec tracking | false |
| --enable_network_events | enable network events monitoring | false |
| --enable_pkt_translation | enable packet translation | false |
| --enable_pkt_drop | enable packet drop | false |
| --enable_rtt | enable RTT tracking | false |
| --enable_udn_mapping | enable User Defined Network mapping | false |
| --get-subnets | get subnets information | false |
| --privileged | force eBPF agent privileged mode | auto |
| --sampling | packets sampling interval | 1 |
| --background | run in background | false |
| --copy | copy the output files locally | prompt |
| --log-level | components logs | info |
| --max-time | maximum capture time | 5m |
| --max-bytes | maximum capture bytes | 50000000 = 50MB |
| --action | filter action | Accept |
| --cidr | filter CIDR | 0.0.0.0/0 |
| --direction | filter direction | - |
| --dport | filter destination port | - |
| --dport_range | filter destination port range | - |
| --dports | filter on either of two destination ports | - |
| --drops | filter flows with only dropped packets | false |
| --icmp_code | filter ICMP code | - |
| --icmp_type | filter ICMP type | - |
| --node-selector | capture on specific nodes | - |
| --peer_ip | filter peer IP | - |
| --peer_cidr | filter peer CIDR | - |
| --port_range | filter port range | - |
| --port | filter port | - |
| --ports | filter on either of two ports | - |
| --protocol | filter protocol | - |
| --query | filter flows using a custom query | - |
| --sport_range | filter source port range | - |
| --sport | filter source port | - |
| --sports | filter on either of two source ports | - |
| --tcp_flags | filter TCP flags | - |
| --interfaces | list of interfaces to monitor, comma separated | - |
| --exclude_interfaces | list of interfaces to exclude, comma separated | lo |

```terminal title="Example running flows capture on TCP protocol and port 49051 with PacketDrop and RTT features enabled:"
$ oc netobserv flows --enable_pkt_drop  --enable_rtt --action=Accept --cidr=0.0.0.0/0 --protocol=TCP --port=49051
```

### Packets capture options {#cli-reference-packet-capture-options_netobserv-cli-reference}

You can filter packets capture data the as same as flows capture by using the filters. Certain features, such as packets drop, DNS, RTT, and network events, are only available for flows and metrics capture.

```terminal title="oc netobserv packets syntax"
$ oc netobserv packets [<option>]
```

| Option | Description | Default |
| --- | --- | --- |
| --background | run in background | false |
| --copy | copy the output files locally | prompt |
| --log-level | components logs | info |
| --max-time | maximum capture time | 5m |
| --max-bytes | maximum capture bytes | 50000000 = 50MB |
| --action | filter action | Accept |
| --cidr | filter CIDR | 0.0.0.0/0 |
| --direction | filter direction | - |
| --dport | filter destination port | - |
| --dport_range | filter destination port range | - |
| --dports | filter on either of two destination ports | - |
| --drops | filter flows with only dropped packets | false |
| --icmp_code | filter ICMP code | - |
| --icmp_type | filter ICMP type | - |
| --node-selector | capture on specific nodes | - |
| --peer_ip | filter peer IP | - |
| --peer_cidr | filter peer CIDR | - |
| --port_range | filter port range | - |
| --port | filter port | - |
| --ports | filter on either of two ports | - |
| --protocol | filter protocol | - |
| --query | filter flows using a custom query | - |
| --sport_range | filter source port range | - |
| --sport | filter source port | - |
| --sports | filter on either of two source ports | - |
| --tcp_flags | filter TCP flags | - |

```terminal title="Example running packets capture on TCP protocol and port 49051:"
$ oc netobserv packets --action=Accept --cidr=0.0.0.0/0 --protocol=TCP --port=49051
```

### Metrics capture options {#cli-reference-metrics-capture-options_netobserv-cli-reference}

You can enable features and use filters on metrics capture, the same as flows capture. The generated graphs fill accordingly in the dashboard.

```terminal title="oc netobserv metrics syntax"
$ oc netobserv metrics [<option>]
```

| Option | Description | Default |
| --- | --- | --- |
| --enable_all | enable all eBPF features | false |
| --enable_dns | enable DNS tracking | false |
| --enable_ipsec | enable IPsec tracking | false |
| --enable_network_events | enable network events monitoring | false |
| --enable_pkt_translation | enable packet translation | false |
| --enable_pkt_drop | enable packet drop | false |
| --enable_rtt | enable RTT tracking | false |
| --enable_udn_mapping | enable User Defined Network mapping | false |
| --get-subnets | get subnets information | false |
| --privileged | force eBPF agent privileged mode | auto |
| --sampling | packets sampling interval | 1 |
| --background | run in background | false |
| --log-level | components logs | info |
| --max-time | maximum capture time | 1h |
| --action | filter action | Accept |
| --cidr | filter CIDR | 0.0.0.0/0 |
| --direction | filter direction | - |
| --dport | filter destination port | - |
| --dport_range | filter destination port range | - |
| --dports | filter on either of two destination ports | - |
| --drops | filter flows with only dropped packets | false |
| --icmp_code | filter ICMP code | - |
| --icmp_type | filter ICMP type | - |
| --node-selector | capture on specific nodes | - |
| --peer_ip | filter peer IP | - |
| --peer_cidr | filter peer CIDR | - |
| --port_range | filter port range | - |
| --port | filter port | - |
| --ports | filter on either of two ports | - |
| --protocol | filter protocol | - |
| --query | filter flows using a custom query | - |
| --sport_range | filter source port range | - |
| --sport | filter source port | - |
| --sports | filter on either of two source ports | - |
| --tcp_flags | filter TCP flags | - |
| --include_list | list of metric names to generate, comma separated | namespace_flows_total,node_ingress_bytes_total,node_egress_bytes_total,workload_ingress_bytes_total |
| --interfaces | list of interfaces to monitor, comma separated | - |
| --exclude_interfaces | list of interfaces to exclude, comma separated | lo |

```terminal title="Example running metrics capture for TCP drops"
$ oc netobserv metrics --enable_pkt_drop --protocol=TCP
```
