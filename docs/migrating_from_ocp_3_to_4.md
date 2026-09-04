---
title: Migration from OpenShift Container Platform 3 to 4 overview
sidebar_position: 1
---

# Migration from OpenShift Container Platform 3 to 4 overview {#migration-from-version-3-to-4-overview}

<a id="migration-from-version-3-to-4-overview"></a>

OpenShift Container Platform 4 clusters are different from OpenShift Container Platform 3 clusters. OpenShift Container Platform 4 clusters contain new technologies and functionality that result in a cluster that is self-managing, flexible, and automated. To learn more about migrating from OpenShift Container Platform 3 to 4 see [About migrating from OpenShift Container Platform 3 to 4](/docs/migrating_from_ocp_3_to_4/about-migrating-from-3-to-4#about-migrating-from-3-to-4).

## Differences between OpenShift Container Platform 3 and 4 {#mtc-3-to-4-overview-differences-mtc}

Before migrating from OpenShift Container Platform 3 to 4, you can check [differences between OpenShift Container Platform 3 and 4](/docs/migrating_from_ocp_3_to_4/planning-migration-3-4#planning-migration-3-4). Review the following information:

- [Architecture](/docs/architecture/#architecture)
- [Installation and update](/docs/architecture/architecture-installation#architecture-installation)
- [Storage](/docs/storage#index), [network](/docs/networking/networking_overview/understanding-networking#understanding-networking), [security](/docs/security#index), and [monitoring considerations](/docs/observability/monitoring/about-ocp-monitoring#about-ocp-monitoring)

## Planning network considerations {#mtc-3-to-4-overview-planning-network-considerations-mtc}

Before migrating from OpenShift Container Platform 3 to 4, review the [differences between OpenShift Container Platform 3 and 4](/docs/migrating_from_ocp_3_to_4/planning-migration-3-4#planning-migration-3-4) for information about the following areas:

- [DNS considerations](/docs/migrating_from_ocp_3_to_4/planning-considerations-3-4#dns-considerations_planning-considerations-3-4)
  - [Isolating the DNS domain of the target cluster from the clients](/docs/migrating_from_ocp_3_to_4/planning-considerations-3-4#migration-isolating-dns-domain-of-target-cluster-from-clients_planning-considerations-3-4).
  - [Setting up the target cluster to accept the source DNS domain](/docs/migrating_from_ocp_3_to_4/planning-considerations-3-4#migration-setting-up-target-cluster-to-accept-source-dns-domain_planning-considerations-3-4).
