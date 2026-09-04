---
title: Security policies
sidebar_position: 3
---

# Security policies {#virt-security-policies}

<a id="virt-security-policies"></a>

OpenShift Virtualization provides built-in security features and authorization policies to protect virtual machine workloads and ensure secure cluster operations across your environment.

**Key points**

- OpenShift Virtualization adheres to the `restricted` Kubernetes pod security standards profile, which aims to enforce the current best practices for pod security.
- Virtual machine (VM) workloads run as unprivileged pods.
- Security context constraints (SCCs) are defined for the `kubevirt-controller` service account. For more information about SSCs, see "Additional resources".
- TLS certificates for OpenShift Virtualization components are renewed and rotated automatically.

## About workload security {#virt-about-workload-security_virt-security-policies}

By default, virtual machine (VM) workloads do not run with root privileges in OpenShift Virtualization, and there are no supported OpenShift Virtualization features that require root privileges.

For each VM, a `virt-launcher` pod runs an instance of `libvirt` in *session mode* to manage the VM process. In session mode, the `libvirt` daemon runs as a non-root user account and only permits connections from clients that are running under the same user identifier (UID). Therefore, VMs run as unprivileged pods, adhering to the security principle of least privilege.

## TLS certificates {#virt-automatic-certificates-renewal_virt-security-policies}

TLS certificates for OpenShift Virtualization components are renewed and rotated automatically. You are not required to refresh them manually.

### Automatic renewal schedules {#_automatic_renewal_schedules}

TLS certificates are automatically deleted and replaced according to the following schedule:

- KubeVirt certificates are renewed daily.
- Containerized Data Importer controller (CDI) certificates are renewed every 15 days.
- MAC pool certificates are renewed every year. Automatic TLS certificate rotation does not disrupt any operations. For example, the following operations continue to function without any disruption:
- Migrations
- Image uploads
- VNC and console connections

## Authorization {#virt-security-policies-auth_virt-security-policies}

OpenShift Virtualization uses role-based access control (RBAC) to define permissions for human users and service accounts. The permissions defined for service accounts control the actions that OpenShift Virtualization components can perform.

You can also use RBAC roles to manage user access to virtualization features. For example, an administrator can create an RBAC role that provides the permissions required to launch a virtual machine. The administrator can then restrict access by binding the role to specific users.

### Default cluster roles for OpenShift Virtualization {#default-cluster-roles-for-virt_virt-security-policies}

By using cluster role aggregation, OpenShift Virtualization extends the default OpenShift Container Platform cluster roles to include permissions for accessing virtualization objects. Roles unique to OpenShift Virtualization are not aggregated with OpenShift Container Platform roles.

**OpenShift Virtualization cluster roles**

<table>
<thead>
<tr>
  <th>Default cluster role</th>
  <th>OpenShift Virtualization cluster role</th>
  <th>OpenShift Virtualization cluster role description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>view</code></td>
  <td><code>kubevirt.io:view</code></td>
  <td>A user that can view all OpenShift Virtualization resources in the cluster but cannot create, delete, modify, or access them. For example, the user can see that a virtual machine (VM) is running but cannot shut it down or gain access to its console.</td>
</tr>
<tr>
  <td><code>edit</code></td>
  <td><code>kubevirt.io:edit</code></td>
  <td>A user that can modify all OpenShift Virtualization resources in the cluster. For example, the user can create VMs, access VM consoles, and delete VMs.</td>
</tr>
<tr>
  <td><code>admin</code></td>
  <td><code>kubevirt.io:admin</code></td>
  <td>A user that has full permissions to all OpenShift Virtualization resources, including the ability to delete collections of resources. The user can also view and modify the OpenShift Virtualization runtime configuration, which is located in the <code>HyperConverged</code> custom resource in the <code>openshift-cnv</code> namespace.</td>
</tr>
<tr>
  <td><code>N/A</code></td>
  <td><code>kubevirt.io:migrate</code></td>
  <td>A user that can create, delete, and update VM live migration requests, which are represented by namespaced <code>VirtualMachineInstanceMigration</code> (VMIM) objects. This role is specific to OpenShift Virtualization.</td>
</tr>
</tbody>
</table>

### RBAC roles for storage features in OpenShift Virtualization {#virt-storage-rbac-roles_virt-security-policies}

Cluster-wide and namespaced RBAC roles enable the Containerized Data Importer (CDI) to manage storage resources, data volumes, and virtual machine disk operations.

#### Cluster-wide RBAC roles {#cluster-wide-rbac-roles-cdi}

**Aggregated cluster roles for the **`cdi.kubevirt.io`** API group**

<table>
<thead>
<tr>
  <th>CDI cluster role</th>
  <th>Resources</th>
  <th>Verbs</th>
</tr>
</thead>
<tbody>
<tr>
  <td rowspan="2"><code>cdi.kubevirt.io:admin</code></td>
  <td><code>datavolumes</code>, <code>uploadtokenrequests</code></td>
  <td><code>*</code> (all)</td>
</tr>
<tr>
  <td><code>datavolumes/source</code></td>
  <td><code>create</code></td>
</tr>
<tr>
  <td rowspan="2"><code>cdi.kubevirt.io:edit</code></td>
  <td><code>datavolumes</code>, <code>uploadtokenrequests</code></td>
  <td><code>*</code></td>
</tr>
<tr>
  <td><code>datavolumes/source</code></td>
  <td><code>create</code></td>
</tr>
<tr>
  <td rowspan="2"><code>cdi.kubevirt.io:view</code></td>
  <td><code>cdiconfigs</code>, <code>dataimportcrons</code>, <code>datasources</code>, <code>datavolumes</code>, <code>objecttransfers</code>, <code>storageprofiles</code>, <code>volumeimportsources</code>, <code>volumeuploadsources</code>, <code>volumeclonesources</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code></td>
</tr>
<tr>
  <td><code>datavolumes/source</code></td>
  <td><code>create</code></td>
</tr>
<tr>
  <td><code>cdi.kubevirt.io:config-reader</code></td>
  <td><code>cdiconfigs</code>, <code>storageprofiles</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code></td>
</tr>
</tbody>
</table>

**Cluster-wide roles for the **`cdi-operator`** service account**

<table>
<thead>
<tr>
  <th>API group</th>
  <th>Resources</th>
  <th>Verbs</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>rbac.authorization.k8s.io</code></td>
  <td><code>clusterrolebindings</code>, <code>clusterroles</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code>, <code>create</code>, <code>update</code>, <code>delete</code></td>
</tr>
<tr>
  <td><code>security.openshift.io</code></td>
  <td><code>securitycontextconstraints</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code>, <code>update</code>, <code>create</code></td>
</tr>
<tr>
  <td><code>apiextensions.k8s.io</code></td>
  <td><code>customresourcedefinitions</code>, <code>customresourcedefinitions/status</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code>, <code>create</code>, <code>update</code>, <code>delete</code></td>
</tr>
<tr>
  <td><code>cdi.kubevirt.io</code></td>
  <td><code>*</code></td>
  <td><code>*</code></td>
</tr>
<tr>
  <td><code>upload.cdi.kubevirt.io</code></td>
  <td><code>*</code></td>
  <td><code>*</code></td>
</tr>
<tr>
  <td><code>admissionregistration.k8s.io</code></td>
  <td><code>validatingwebhookconfigurations</code>, <code>mutatingwebhookconfigurations</code></td>
  <td><code>create</code>, <code>list</code>, <code>watch</code></td>
</tr>
<tr>
  <td><code>admissionregistration.k8s.io</code></td>
  <td><code>validatingwebhookconfigurations</code> Allow list: <code>cdi-api-dataimportcron-validate, cdi-api-populator-validate, cdi-api-datavolume-validate, cdi-api-validate, objecttransfer-api-validate</code></td>
  <td><code>get</code>, <code>update</code>, <code>delete</code></td>
</tr>
<tr>
  <td><code>admissionregistration.k8s.io</code></td>
  <td><code>mutatingwebhookconfigurations</code> Allow list: <code>cdi-api-datavolume-mutate</code></td>
  <td><code>get</code>, <code>update</code>, <code>delete</code></td>
</tr>
<tr>
  <td><code>apiregistration.k8s.io</code></td>
  <td><code>apiservices</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code>, <code>create</code>, <code>update</code>, <code>delete</code></td>
</tr>
</tbody>
</table>

**Cluster-wide roles for the **`cdi-controller`** service account**

<table>
<thead>
<tr>
  <th>API group</th>
  <th>Resources</th>
  <th>Verbs</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>""</code> (core)</td>
  <td><code>events</code></td>
  <td><code>create</code>, <code>patch</code></td>
</tr>
<tr>
  <td><code>""</code> (core)</td>
  <td><code>persistentvolumeclaims</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code>, <code>create</code>, <code>update</code>, <code>delete</code>, <code>deletecollection</code>, <code>patch</code></td>
</tr>
<tr>
  <td><code>""</code> (core)</td>
  <td><code>persistentvolumes</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code>, <code>update</code></td>
</tr>
<tr>
  <td><code>""</code> (core)</td>
  <td><code>persistentvolumeclaims/finalizers</code>, <code>pods/finalizers</code></td>
  <td><code>update</code></td>
</tr>
<tr>
  <td><code>""</code> (core)</td>
  <td><code>pods</code>, <code>services</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code>, <code>create</code>, <code>delete</code></td>
</tr>
<tr>
  <td><code>""</code> (core)</td>
  <td><code>configmaps</code></td>
  <td><code>get</code>, <code>create</code></td>
</tr>
<tr>
  <td><code>storage.k8s.io</code></td>
  <td><code>storageclasses</code>, <code>csidrivers</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code></td>
</tr>
<tr>
  <td><code>config.openshift.io</code></td>
  <td><code>proxies</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code></td>
</tr>
<tr>
  <td><code>cdi.kubevirt.io</code></td>
  <td><code>*</code></td>
  <td><code>*</code></td>
</tr>
<tr>
  <td><code>snapshot.storage.k8s.io</code></td>
  <td><code>volumesnapshots</code>, <code>volumesnapshotclasses</code>, <code>volumesnapshotcontents</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code>, <code>create</code>, <code>delete</code></td>
</tr>
<tr>
  <td><code>snapshot.storage.k8s.io</code></td>
  <td><code>volumesnapshots</code></td>
  <td><code>update</code>, <code>deletecollection</code></td>
</tr>
<tr>
  <td><code>apiextensions.k8s.io</code></td>
  <td><code>customresourcedefinitions</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code></td>
</tr>
<tr>
  <td><code>scheduling.k8s.io</code></td>
  <td><code>priorityclasses</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code></td>
</tr>
<tr>
  <td><code>image.openshift.io</code></td>
  <td><code>imagestreams</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code></td>
</tr>
<tr>
  <td><code>""</code> (core)</td>
  <td><code>secrets</code></td>
  <td><code>create</code></td>
</tr>
<tr>
  <td><code>kubevirt.io</code></td>
  <td><code>virtualmachines/finalizers</code></td>
  <td><code>update</code></td>
</tr>
</tbody>
</table>

#### Namespaced RBAC roles {#namespaced-rbac-roles-cdi}

**Namespaced roles for the **`cdi-operator`** service account**

<table>
<thead>
<tr>
  <th>API group</th>
  <th>Resources</th>
  <th>Verbs</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>rbac.authorization.k8s.io</code></td>
  <td><code>rolebindings</code>, <code>roles</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code>, <code>create</code>, <code>update</code>, <code>delete</code></td>
</tr>
<tr>
  <td><code>""</code> (core)</td>
  <td><code>serviceaccounts</code>, <code>configmaps</code>, <code>events</code>, <code>secrets</code>, <code>services</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code>, <code>create</code>, <code>update</code>, <code>patch</code>, <code>delete</code></td>
</tr>
<tr>
  <td><code>apps</code></td>
  <td><code>deployments</code>, <code>deployments/finalizers</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code>, <code>create</code>, <code>update</code>, <code>delete</code></td>
</tr>
<tr>
  <td><code>route.openshift.io</code></td>
  <td><code>routes</code>, <code>routes/custom-host</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code>, <code>create</code>, <code>update</code></td>
</tr>
<tr>
  <td><code>config.openshift.io</code></td>
  <td><code>proxies</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code></td>
</tr>
<tr>
  <td><code>monitoring.coreos.com</code></td>
  <td><code>servicemonitors</code>, <code>prometheusrules</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code>, <code>create</code>, <code>delete</code>, <code>update</code>, <code>patch</code></td>
</tr>
<tr>
  <td><code>coordination.k8s.io</code></td>
  <td><code>leases</code></td>
  <td><code>get</code>, <code>create</code>, <code>update</code></td>
</tr>
</tbody>
</table>

**Namespaced roles for the **`cdi-controller`** service account**

<table>
<thead>
<tr>
  <th>API group</th>
  <th>Resources</th>
  <th>Verbs</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>""</code> (core)</td>
  <td><code>configmaps</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code>, <code>create</code>, <code>update</code>, <code>delete</code></td>
</tr>
<tr>
  <td><code>""</code> (core)</td>
  <td><code>secrets</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code></td>
</tr>
<tr>
  <td><code>batch</code></td>
  <td><code>cronjobs</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code>, <code>create</code>, <code>update</code>, <code>delete</code></td>
</tr>
<tr>
  <td><code>batch</code></td>
  <td><code>jobs</code></td>
  <td><code>create</code>, <code>delete</code>, <code>list</code>, <code>watch</code></td>
</tr>
<tr>
  <td><code>coordination.k8s.io</code></td>
  <td><code>leases</code></td>
  <td><code>get</code>, <code>create</code>, <code>update</code></td>
</tr>
<tr>
  <td><code>networking.k8s.io</code></td>
  <td><code>ingresses</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code></td>
</tr>
<tr>
  <td><code>route.openshift.io</code></td>
  <td><code>routes</code></td>
  <td><code>get</code>, <code>list</code>, <code>watch</code></td>
</tr>
</tbody>
</table>

### Additional SCCs and permissions for the kubevirt-controller service account {#virt-additional-scc-for-kubevirt-controller_virt-security-policies}

Security context constraints (SCCs) control permissions for pods. These permissions include actions that a pod, a collection of containers, can perform and what resources it can access. You can use SCCs to define a set of conditions that a pod must run with to be accepted into the system.

The `virt-controller` is a cluster controller that creates the `virt-launcher` pods for virtual machines in the cluster.

:::note

By default, `virt-launcher` pods run with the `default` service account in the namespace. If your compliance controls require a unique service account, assign one to the VM. The setting applies to the `VirtualMachineInstance` object and the `virt-launcher` pod.

:::

The `kubevirt-controller` service account is granted additional SCCs and Linux capabilities so that it can create `virt-launcher` pods with the appropriate permissions. These extended permissions allow virtual machines to use OpenShift Virtualization features that are beyond the scope of typical pods.

The `kubevirt-controller` service account is granted the following SCCs:

`scc.AllowHostDirVolumePlugin = true`
: This allows virtual machines to use the hostpath volume plugin.

`scc.AllowPrivilegedContainer = false`
: This ensures the `virt-launcher` pod is not run as a privileged container.

`scc.AllowedCapabilities = []corev1.Capability{"SYS_NICE", "NET_BIND_SERVICE"}`
: - `SYS_NICE` allows setting the CPU affinity.
  - `NET_BIND_SERVICE` allows DHCP and Slirp operations.

#### Viewing the SCC and RBAC definitions for the kubevirt-controller {#_viewing_the_scc_and_rbac_definitions_for_the_kubevirt-controller}

You can view the `SecurityContextConstraints` definition for the `kubevirt-controller` by using the `oc` tool:

```terminal
$ oc get scc kubevirt-controller -o yaml
```

You can view the RBAC definition for the `kubevirt-controller` clusterrole by using the `oc` tool:

```terminal
$ oc get clusterrole kubevirt-controller -o yaml
```

**Additional resources**

- [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/#restricted)
- [About Security context constraints](/docs/authentication/managing-security-context-constraints#security-context-constraints-about_configuring-internal-oauth)
- [Using RBAC to define and apply permissions](/docs/authentication/using-rbac#using-rbac)
- [Managing security context constraints](/docs/authentication/managing-security-context-constraints#security-context-constraints-about_configuring-internal-oauth)
- [Creating a cluster role](/docs/authentication/using-rbac#creating-cluster-role_using-rbac)
- [Cluster role binding commands](/docs/authentication/using-rbac#cluster-role-binding-commands_using-rbac)
- [Enabling user permissions to clone data volumes across namespaces](/docs/virt/storage/virt-enabling-user-permissions-to-clone-datavolumes#virt-enabling-user-permissions-to-clone-datavolumes)
