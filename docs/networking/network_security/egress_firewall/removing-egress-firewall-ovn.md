---
title: Removing an egress firewall from a project
sidebar_position: 3
---

# Removing an egress firewall from a project {#removing-egress-firewall-ovn}

<a id="removing-egress-firewall-ovn"></a>

As a cluster administrator, you can remove an egress firewall from a project to remove all restrictions on network traffic from the project that leaves the OpenShift Container Platform cluster.

## Removing an EgressFirewall CR {#nw-egress-firewall-delete_removing-egress-firewall-ovn}

As a cluster administrator, you can remove an egress firewall from a project.

**Prerequisites**

- A cluster using the OVN-Kubernetes network plugin.
- Install the OpenShift CLI (`oc`).
- You must log in to the cluster as a cluster administrator.

**Procedure**

1. Find the name of the `EgressFirewall` CR for the project. Replace `<project>` with the name of the project.
   ```terminal
   $ oc get egressfirewall -n <project>
   ```
2. Delete the `EgressFirewall` CR by entering the following command. Replace `<project>` with the name of the project and `<name>` with the name of the object.
   ```terminal
   $ oc delete -n <project> egressfirewall <name>
   ```
