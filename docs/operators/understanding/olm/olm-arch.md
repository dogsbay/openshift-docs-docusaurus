---
title: Operator Lifecycle Manager architecture
sidebar_position: 2
---

# Operator Lifecycle Manager architecture {#olm-arch}

<a id="olm-arch"></a>

You can learn how Operator Lifecycle Manager (OLM) components interact to manage Operators in OpenShift Container Platform. The architecture includes the OLM Operator, Catalog Operator, and Catalog Registry.

## CRDs {#olm-architecture_olm-arch}

Operator Lifecycle Manager (OLM) and the Catalog Operator manage the following custom resource definitions (CRDs) that form the basis of the Operator Framework.

**CRDs managed by OLM and Catalog Operators**

<table>
<thead>
<tr>
  <th>Resource</th>
  <th>Short name</th>
  <th>Owner</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>ClusterServiceVersion</code> (CSV)</td>
  <td><code>csv</code></td>
  <td>OLM</td>
  <td>Application metadata: name, version, icon, required resources, installation, and so on.</td>
</tr>
<tr>
  <td><code>InstallPlan</code></td>
  <td><code>ip</code></td>
  <td>Catalog</td>
  <td>Calculated list of resources to be created to automatically install or upgrade a CSV.</td>
</tr>
<tr>
  <td><code>CatalogSource</code></td>
  <td><code>catsrc</code></td>
  <td>Catalog</td>
  <td>A repository of CSVs, CRDs, and packages that define an application.</td>
</tr>
<tr>
  <td><code>Subscription</code></td>
  <td><code>sub</code></td>
  <td>Catalog</td>
  <td>Used to keep CSVs up to date by tracking a channel in a package.</td>
</tr>
<tr>
  <td><code>OperatorGroup</code></td>
  <td><code>og</code></td>
  <td>OLM</td>
  <td>Configures all Operators deployed in the same namespace as the <code>OperatorGroup</code> object to watch for their custom resource (CR) in a list of namespaces or cluster-wide.</td>
</tr>
</tbody>
</table>

Each of these Operators is also responsible for creating the following resources:

**Resources created by OLM and Catalog Operators**

<table>
<thead>
<tr>
  <th>Resource</th>
  <th>Owner</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>Deployments</code></td>
  <td rowspan="4">OLM</td>
</tr>
<tr>
  <td><code>ServiceAccounts</code></td>
</tr>
<tr>
  <td><code>(Cluster)Roles</code></td>
</tr>
<tr>
  <td><code>(Cluster)RoleBindings</code></td>
</tr>
<tr>
  <td><code>CustomResourceDefinitions</code> (CRDs)</td>
  <td rowspan="2">Catalog</td>
</tr>
<tr>
  <td><code>ClusterServiceVersions</code></td>
</tr>
</tbody>
</table>

## OLM Operator {#olm-arch-olm-operator_olm-arch}

The OLM Operator deploys applications defined by cluster service versions (CSVs) after their required resources are present in the cluster. It watches CSVs in a namespace, verifies requirements, and runs the install strategy when conditions are met.

The OLM Operator is not concerned with the creation of the required resources; you can choose to manually create these resources using the CLI or using the Catalog Operator. This separation of concern allows users incremental buy-in in terms of how much of the OLM framework they choose to leverage for their application.

The OLM Operator uses the following workflow:

1. Watch for cluster service versions (CSVs) in a namespace and check that requirements are met.
2. If requirements are met, run the install strategy for the CSV.
   :::note

   A CSV must be an active member of an Operator group for the install strategy to run.

   :::

## Catalog Operator {#olm-arch-catalog-operator_olm-arch}

The Catalog Operator in OpenShift Container Platform resolves and installs cluster service versions (CSVs) and their required resources from catalog sources. It watches subscriptions and catalog sources to create install plans and upgrade packages in channels.

To track a package in a channel, you can create a `Subscription` object configuring the desired package, channel, and the `CatalogSource` object you want to use for pulling updates. When updates are found, an appropriate `InstallPlan` object is written into the namespace on behalf of the user.

The Catalog Operator uses the following workflow:

1. Connect to each catalog source in the cluster.
2. Watch for unresolved install plans created by a user, and if found:
   1. Find the CSV matching the name requested and add the CSV as a resolved resource.
   2. For each managed or required CRD, add the CRD as a resolved resource.
   3. For each required CRD, find the CSV that manages it.
3. Watch for resolved install plans and create all of the discovered resources for it, if approved by a user or automatically.
4. Watch for catalog sources and subscriptions and create install plans based on them.

## Catalog Registry {#olm-arch-catalog-registry_olm-arch}

The Catalog Registry stores cluster service versions (CSVs), custom resource definitions (CRDs), and metadata about packages and channels for Operator installation in OpenShift Container Platform. Package manifests link package identities to CSVs so the Catalog Operator can step through channel upgrade paths.

A *package manifest* is an entry in the Catalog Registry that associates a package identity with sets of CSVs. Within a package, channels point to a particular CSV. Because CSVs explicitly reference the CSV that they replace, a package manifest provides the Catalog Operator with all of the information that is required to update a CSV to the latest version in a channel, stepping through each intermediate version.
