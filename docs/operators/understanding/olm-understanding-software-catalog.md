---
title: Understanding the software catalog
sidebar_position: 5
---

# Understanding the software catalog {#olm-understanding-software-catalog}

<a id="olm-understanding-software-catalog"></a>

The software catalog in OpenShift Container Platform provides a web console interface for discovering, installing, and managing Operators through Operator Lifecycle Manager (OLM).

## About the software catalog {#olm-software-catalog-overview_olm-understanding-software-catalog}

The *software catalog* is the web console interface in OpenShift Container Platform that cluster administrators use to discover and install Operators. With one click, an Operator can be pulled from its off-cluster source, installed and subscribed on the cluster, and made ready for engineering teams to self-service manage the product across deployment environments using Operator Lifecycle Manager (OLM).

Cluster administrators can choose from catalogs grouped into the following categories:

<table>
<thead>
<tr>
  <th>Category</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Red Hat Operators</td>
  <td>Red Hat products packaged and shipped by Red Hat. Supported by Red Hat.</td>
</tr>
<tr>
  <td>Certified Operators</td>
  <td>Products from leading independent software vendors (ISVs). Red Hat partners with ISVs to package and ship. Supported by the ISV.</td>
</tr>
<tr>
  <td>Community Operators</td>
  <td>Optionally-visible software maintained by relevant representatives in the community Operators GitHub repository. No official support.</td>
</tr>
<tr>
  <td>Custom Operators</td>
  <td>Operators you add to the cluster yourself. If you have not added any custom Operators, the <strong>Custom</strong> category does not appear in the web console software catalog.</td>
</tr>
</tbody>
</table>

Operators in the software catalog are packaged to run on OLM. This includes a YAML file called a cluster service version (CSV) containing all of the CRDs, RBAC rules, deployments, and container images required to install and securely run the Operator. It also contains user-visible information like a description of its features and supported Kubernetes versions.

## Software catalog architecture {#olm-software-catalog-arch_olm-understanding-software-catalog}

The software catalog UI component is driven by the Marketplace Operator by default on OpenShift Container Platform in the `openshift-marketplace` namespace.

### OperatorHub custom resource {#olm-software-catalog-arch-operatorhub-crd_olm-understanding-software-catalog}

The Marketplace Operator manages an `OperatorHub` custom resource (CR) named `cluster` that manages the default `CatalogSource` objects provided with the software catalog. You can modify this resource to enable or disable the default catalogs, which is useful when configuring OpenShift Container Platform in restricted network environments.

```yaml title="Example OperatorHub custom resource"
apiVersion: config.openshift.io/v1
kind: OperatorHub
metadata:
  name: cluster
spec:
  disableAllDefaultSources: true (1)
  sources: [ (2)
    {
      name: "community-operators",
      disabled: false
    }
  ]
```

1. `disableAllDefaultSources` is an override that controls availability of all default catalogs that are configured by default during an OpenShift Container Platform installation.
2. Disable default catalogs individually by changing the `disabled` parameter value per source.

**Additional resources**

- [Catalog source](/docs/operators/understanding/olm/olm-understanding-olm#olm-catalogsource_olm-understanding-olm)
- [Operator installation and upgrade workflow in OLM](/docs/operators/understanding/olm/olm-workflow#olm-upgrades_olm-workflow)
- [Red Hat Partner Connect](https://connect.redhat.com)
