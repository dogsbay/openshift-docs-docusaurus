---
title: Deprecated features
sidebar_position: 2
---

# Deprecated features

<a id="deprecated-features"></a>

Large changes to the underlying architecture and installation process are applied in OpenShift Container Platform v4, and many features from OpenShift Container Platform v3 are now deprecated.

## Features deprecated in OpenShift Container Platform v4 {#deprecated}

**Features Deprecated in 4.22**

<table>
<thead>
<tr>
  <th>Feature</th>
  <th>Justification</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Hawkular</td>
  <td>Replaced by cluster monitoring.</td>
</tr>
<tr>
  <td>Cassandra</td>
  <td>Replaced by cluster monitoring.</td>
</tr>
<tr>
  <td>Heapster</td>
  <td>Replaced by Prometheus adapter.</td>
</tr>
<tr>
  <td>Atomic Host</td>
  <td>Replaced by Red&#160;Hat Enterprise Linux CoreOS (RHCOS).</td>
</tr>
<tr>
  <td>System containers</td>
  <td>Replaced by Red&#160;Hat Enterprise Linux CoreOS (RHCOS).</td>
</tr>
<tr>
  <td><code>projectatomic/docker-1.13</code> additional search registries</td>
  <td>CRI-O is the default container runtime for OpenShift Container Platform v4 on Fedora.</td>
</tr>
<tr>
  <td><code>oc adm diagnostics</code></td>
  <td>Operator-based diagnostics.</td>
</tr>
<tr>
  <td><code>oc adm registry</code></td>
  <td>Replaced by the Image Registry Operator.</td>
</tr>
<tr>
  <td>Custom strategy builds using Docker</td>
  <td>If you want to continue using custom builds, you should replace your Docker invocations with Podman or Buildah. The custom build strategy will not be removed, but the functionality changed significantly in OpenShift v4.</td>
</tr>
<tr>
  <td>Cockpit</td>
  <td>Improved OpenShift v4 web console.</td>
</tr>
<tr>
  <td>Stand-alone registry installations</td>
  <td>Quay is Red Hat's container image registry.</td>
</tr>
<tr>
  <td>DNSmasq</td>
  <td>CoreDNS is the default.</td>
</tr>
<tr>
  <td>External etcd nodes</td>
  <td>etcd is always on the cluster in OpenShift v4.</td>
</tr>
<tr>
  <td>CloudForms OpenShift Provider and Podified CloudForms</td>
  <td>Replaced by built-in management tooling.</td>
</tr>
<tr>
  <td>Volume Provisioning via installer</td>
  <td>Replaced by dynamic volumes or, if NFS is required, NFS provisioner.</td>
</tr>
<tr>
  <td>Blue-green installation method</td>
  <td>Ease of upgrade is a core value of OpenShift v4.</td>
</tr>
<tr>
  <td>OpenShift Service Broker and Service Catalog</td>
  <td>The Service Catalog and the OpenShift service brokers are being replaced over the course of several future OpenShift v4 releases. Reference the Operator Framework and Operator Lifecycle Manager (OLM) to continue providing your applications to OpenShift v4 clusters. These new technologies provide many benefits around complete management of the lifecycle of your application.</td>
</tr>
<tr>
  <td><code>oc adm ca</code></td>
  <td>Certificates are managed by Operators internally.</td>
</tr>
<tr>
  <td><code>oc adm create-api-client-config</code></td>
  <td rowspan="2">Functions are managed by Operators internally.</td>
</tr>
<tr>
  <td><code>oc adm create-bootstrap-policy-file</code></td>
</tr>
<tr>
  <td><code>oc adm policy reconcile-sccs</code></td>
  <td>Functions are managed by <code>openshift-apiserver</code> internally.</td>
</tr>
<tr>
  <td>Web console</td>
  <td>The web console from OpenShift v3 has been replaced by a new web console in OpenShift v4.</td>
</tr>
</tbody>
</table>
