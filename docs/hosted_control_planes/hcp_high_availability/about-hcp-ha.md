---
title: About high availability for hosted control planes
sidebar_position: 1
---

# About high availability for hosted control planes {#about-hcp-ha}

<a id="about-hcp-ha"></a>

You can maintain high availability (HA) for hosted control planes by recovering etcd members for a hosted cluster, backing up and restoring etcd for a hosted cluster, and completing a disaster recovery process for a hosted cluster.

## Impact of the failed management cluster component {#hcp-mgmt-component-loss-impact_about-hcp-ha}

If the management cluster component fails, your workload remains unaffected. In the OpenShift Container Platform management cluster, the control plane is decoupled from the data plane to provide resiliency.

The following table covers the impact of a failed management cluster component on the control plane and the data plane. However, the table does not cover all scenarios for the management cluster component failures.

**Impact of the failed component on hosted control planes**

| Name of the failed component | Hosted control plane API status | Hosted cluster data plane status |
| --- | --- | --- |
| Worker node | Available | Available |
| Availability zone | Available | Available |
| Management cluster control plane | Available | Available |
| Management cluster control plane and worker nodes | Not available | Available |
