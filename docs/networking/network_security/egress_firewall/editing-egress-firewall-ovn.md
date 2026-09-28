---
title: Editing an egress firewall for a project
sidebar_position: 2
---

# Editing an egress firewall for a project {#editing-egress-firewall-ovn}

<a id="editing-egress-firewall-ovn"></a>

As a cluster administrator, you can modify network traffic rules for an existing egress firewall.

## Edit an EgressFirewall custom resource (CR) {#nw-egress-firewall-edit_editing-egress-firewall-ovn}

As a cluster administrator, you can update the egress firewall for a project.

**Prerequisites**

- A cluster using the OVN-Kubernetes network plugin.
- Install the OpenShift CLI (`oc`).
- You must log in to the cluster as a cluster administrator.

**Procedure**

1. Find the name of the `EgressFirewall` CR for the project. Replace `<project>` with the name of the project.
   ```terminal
   $ oc get -n <project> egressfirewall
   ```
2. Optional: If you did not save a copy of the `EgressFirewall` object when you created the egress network firewall, enter the following command to create a copy.
   ```terminal
   $ oc get -n <project> egressfirewall <name> -o yaml > <filename>.yaml
   ```

   Replace `<project>` with the name of the project. Replace `<name>` with the name of the object. Replace `<filename>` with the name of the file to save the YAML to.
3. After making changes to the policy rules, enter the following command to replace the `EgressFirewall` CR. Replace `<filename>` with the name of the file containing the updated `EgressFirewall` CR.
   ```terminal
   $ oc replace -f <filename>.yaml
   ```
