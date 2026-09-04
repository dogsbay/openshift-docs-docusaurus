---
title: Operator Lifecycle Manager concepts and resources
sidebar_position: 1
---

# Operator Lifecycle Manager concepts and resources {#olm-understanding-olm}

<a id="olm-understanding-olm"></a>

Key concepts for understanding Operator Lifecycle Manager (OLM) include cluster service versions (CSVs), catalog sources, subscriptions, and Operator groups.

## About Operator Lifecycle Manager (OLM) Classic {#olm-overview_olm-understanding-olm}

Operator Lifecycle Manager (OLM) Classic helps users install, update, and manage the lifecycle of Kubernetes native applications (Operators) and their associated services running across their OpenShift Container Platform clusters. Operator Lifecycle Manager (OLM) Classic forms part of the Operator Framework, an open source toolkit designed to manage Operators in an effective, automated, and scalable way.

**Figure 1. OLM (Classic) workflow**

![olm-workflow](/images/olm-workflow.png)

OLM runs by default in OpenShift Container Platform 4.22, which aids cluster administrators

in installing, upgrading, and granting access to Operators running on their cluster. The OpenShift Container Platform web console provides management screens for cluster administrators to install Operators, as well as grant specific projects access to use the catalog of Operators available on the cluster.

For developers, a self-service experience allows provisioning and configuring instances of databases, monitoring, and big data services without having to be subject matter experts, because the Operator has that knowledge baked into it.

## OLM resources {#olm-resources_olm-understanding-olm}

The following custom resource definitions (CRDs) are defined and managed by Operator Lifecycle Manager (OLM) in OpenShift Container Platform. Use these resources to configure catalog sources, subscriptions, install plans, cluster service versions (CSVs), and Operator groups.

**CRDs managed by OLM and Catalog Operators**

<table>
<thead>
<tr>
  <th>Resource</th>
  <th>Short name</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>ClusterServiceVersion</code> (CSV)</td>
  <td><code>csv</code></td>
  <td>Application metadata. For example: name, version, icon, required resources.</td>
</tr>
<tr>
  <td><code>CatalogSource</code></td>
  <td><code>catsrc</code></td>
  <td>A repository of CSVs, CRDs, and packages that define an application.</td>
</tr>
<tr>
  <td><code>Subscription</code></td>
  <td><code>sub</code></td>
  <td>Keeps CSVs up to date by tracking a channel in a package.</td>
</tr>
<tr>
  <td><code>InstallPlan</code></td>
  <td><code>ip</code></td>
  <td>Calculated list of resources to be created to automatically install or upgrade a CSV.</td>
</tr>
<tr>
  <td><code>OperatorGroup</code></td>
  <td><code>og</code></td>
  <td>Configures all Operators deployed in the same namespace as the <code>OperatorGroup</code> object to watch for their custom resource (CR) in a list of namespaces or cluster-wide.</td>
</tr>
<tr>
  <td><code>OperatorConditions</code></td>
  <td>-</td>
  <td>Creates a communication channel between OLM and an Operator it manages. Operators can write to the <code>Status.Conditions</code> array to communicate complex states to OLM.</td>
</tr>
</tbody>
</table>

### Cluster service version {#olm-csv_olm-understanding-olm}

A cluster service version (CSV) is a YAML manifest that represents a specific version of a running Operator on your OpenShift Container Platform cluster. OLM uses CSV metadata to run Operators safely and determine how to apply upgrades when new versions are published.

A CSV includes the metadata that accompanies an Operator container image, used to populate user interfaces with information such as its name, version, description, labels, repository link, and logo.

A CSV is also a source of technical information required to run the Operator, such as which custom resources (CRs) it manages or depends on, RBAC rules, cluster requirements, and install strategies. This information tells OLM how to create required resources and set up the Operator as a deployment.

### Catalog source {#olm-catalogsource_olm-understanding-olm}

A *catalog source* represents a store of metadata, typically by referencing an *index image* stored in a container registry. Operator Lifecycle Manager (OLM) queries catalog sources to discover and install Operators and their dependencies. The software catalog in the OpenShift Container Platform web console also displays the Operators provided by catalog sources.

:::tip

Cluster administrators can view the full list of Operators provided by an enabled catalog source on a cluster by using the **Administration** → **Cluster Settings** → **Configuration** → **OperatorHub** page in the web console.

:::

The `spec` of a `CatalogSource` object indicates how to construct a pod or how to communicate with a service that serves the Operator Registry gRPC API.

```yaml title="Example CatalogSource object"
﻿apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  generation: 1
  name: example-catalog
  namespace: openshift-marketplace
  annotations:
    olm.catalogImageTemplate:
      "quay.io/example-org/example-catalog:v{kube_major_version}.{kube_minor_version}.{kube_patch_version}"
spec:
  displayName: Example Catalog
  image: quay.io/example-org/example-catalog:v1
  priority: -400
  publisher: Example Org
  sourceType: grpc
  grpcPodConfig:
    securityContextConfig: <security_mode>
    nodeSelector:
      custom_label: <label>
    priorityClassName: system-cluster-critical
    tolerations:
      - key: "key1"
        operator: "Equal"
        value: "value1"
        effect: "NoSchedule"
  updateStrategy:
    registryPoll:
      interval: 30m0s
status:
  connectionState:
    address: example-catalog.openshift-marketplace.svc:50051
    lastConnect: 2021-08-26T18:14:31Z
    lastObservedState: READY
  latestImageRegistryPoll: 2021-08-26T18:46:25Z
  registryService:
    createdAt: 2021-08-26T16:16:37Z
    port: 50051
    protocol: grpc
    serviceName: example-catalog
    serviceNamespace: openshift-marketplace
```

where:

`metadata.name`
: Specifies the name for the `CatalogSource` object. This value is also used as part of the name for the related pod that is created in the requested namespace.

`metadata.namespace`
: Specifies the namespace to create the catalog in. To make the catalog available cluster-wide in all namespaces, set this value to `openshift-marketplace`. The default Red Hat-provided catalog sources also use the `openshift-marketplace` namespace. Otherwise, set the value to a specific namespace to make the Operator only available in that namespace.

`metadata.annotations.olm.catalogImageTemplate`
: To avoid cluster upgrades potentially leaving Operator installations in an unsupported state or without a continued update path, you can enable automatically changing your Operator catalog’s index image version as part of cluster upgrades. This field is optional.

  Set the `olm.catalogImageTemplate` annotation to your index image name and use one or more of the Kubernetes cluster version variables as shown when constructing the template for the image tag. The annotation overwrites the `spec.image` field at run time. See the "Image template for custom catalog sources" section for more details.

`spec.displayName`
: Specifies the display name for the catalog in the web console and CLI.

`spec.image`
: Specifies the index image for the catalog. Optionally, this spec can be omitted when using the `olm.catalogImageTemplate` annotation, which sets the pull spec at run time.

`spec.priority`
: Specifies the weight for the catalog source. OLM uses the weight for prioritization during dependency resolution. A higher weight indicates the catalog is preferred over lower-weighted catalogs.

`spec.sourceType`
: Specifies one of the following source types:

  - `grpc` with an `image` reference: OLM pulls the image and runs the pod, which is expected to serve a compliant API.
  - `grpc` with an `address` field: OLM attempts to contact the gRPC API at the given address. This should not be used in most cases.
  - `configmap`: OLM parses config map data and runs a pod that can serve the gRPC API over it.

`spec.grpcPodConfig.securityContextConfig`
: Specifies the pod security admissions (PSA) policy for the catalog source pod. It accepts the values of `legacy` or `restricted`. If the field is not set, the default value is `legacy`. In a future OpenShift Container Platform release, it is planned that the default value will be `restricted`.

  :::note

  If your catalog cannot run with `restricted` permissions, it is recommended that you manually set this field to `legacy`.

  :::

`spec.grpcPodConfig.nodeSelector`
: Overrides the default node selector for the pod serving the content in `spec.image` for `grpc` type catalog sources. This field is optional.

`spec.grpcPodConfig.priorityClassName`
: Overrides the default priority class name for the pod serving the content in `spec.image` for `grpc` type catalog sources. Kubernetes provides `system-cluster-critical` and `system-node-critical` priority classes by default. Setting the field to empty (`""`) assigns the pod the default priority. Other priority classes can be defined manually. This field is optional.

`spec.grpcPodConfig.tolerations`
: Overrides the default tolerations for the pod serving the content in `spec.image` for `grpc` type catalog sources. This field is optional.

`spec.updateStrategy.registryPoll`
: Specifies how often OLM polls the container registry for updates.

`status.connectionState.lastObservedState`
: Displays the last observed state of the catalog connection. For example:

  - `READY`: A connection is successfully established.
  - `CONNECTING`: A connection is attempting to establish.
  - `TRANSIENT_FAILURE`: A temporary problem has occurred while attempting to establish a connection, such as a timeout. The state will eventually switch back to `CONNECTING` and try again.

`status.latestImageRegistryPoll`
: Displays the last time OLM polled the container registry for catalog image updates.

`status.registryService`
: Displays status information for the catalog’s Operator Registry service.

Referencing the `name` of a `CatalogSource` object in a subscription instructs OLM where to search to find a requested Operator:

```yaml title="Example Subscription object referencing a catalog source"
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: example-operator
  namespace: example-namespace
spec:
  channel: stable
  name: example-operator
  source: example-catalog
  sourceNamespace: openshift-marketplace
```

**Additional resources**

- [Understanding the software catalog](/docs/operators/understanding/olm-understanding-software-catalog#olm-understanding-software-catalog)
- [Red Hat-provided Operator catalogs](/docs/operators/understanding/olm-rh-catalogs#olm-rh-catalogs)
- [Adding a catalog source to a cluster](/docs/operators/admin/olm-managing-custom-catalogs#olm-creating-catalog-from-index_olm-managing-custom-catalogs)
- [Catalog priority](/docs/operators/understanding/olm/olm-understanding-dependency-resolution#olm-dependency-catalog-priority_olm-understanding-dependency-resolution)
- [Viewing Operator catalog source status by using the CLI](/docs/operators/admin/olm-status#olm-cs-status-cli_olm-status)
- [States of Connectivity (gRPC documentation)](https://grpc.github.io/grpc/core/md_doc_connectivity-semantics-and-api.html)
- [Understanding and managing pod security admission](/docs/authentication/understanding-and-managing-pod-security-admission#understanding-and-managing-pod-security-admission)
- [Catalog source pod scheduling](/docs/operators/admin/olm-cs-podsched#olm-cs-podsched)

#### Image template for custom catalog sources {#olm-catalogsource-image-template_olm-understanding-olm}

Operator compatibility with the underlying cluster can be expressed by a catalog source in various ways. One way, which is used for the default Red Hat-provided catalog sources, is to identify image tags for index images that are specifically created for a particular platform release, for example OpenShift Container Platform 4.22.

During a cluster upgrade, the index image tag for the default Red Hat-provided catalog sources are updated automatically by the Cluster Version Operator (CVO) so that Operator Lifecycle Manager (OLM) pulls the updated version of the catalog. For example during an upgrade from OpenShift Container Platform 4.21 to 4.22, the `spec.image` field in the `CatalogSource` object for the `redhat-operators` catalog is updated from:

```terminal
registry.redhat.io/redhat/redhat-operator-index:v4.22
```

to:

```terminal
registry.redhat.io/redhat/redhat-operator-index:v4.22
```

However, the CVO does not automatically update image tags for custom catalogs. To ensure users are left with a compatible and supported Operator installation after a cluster upgrade, custom catalogs should also be kept updated to reference an updated index image.

Starting in OpenShift Container Platform 4.9, cluster administrators can add the `olm.catalogImageTemplate` annotation in the `CatalogSource` object for custom catalogs to an image reference that includes a template. The following Kubernetes version variables are supported for use in the template:

- `kube_major_version`
- `kube_minor_version`
- `kube_patch_version`

:::note

You must specify the Kubernetes cluster version and not the OpenShift Container Platform cluster version, as the latter is not currently available for templating.

:::

Provided that you have created and pushed an index image with a tag specifying the updated Kubernetes version, setting this annotation enables the index image versions in custom catalogs to be automatically changed after a cluster upgrade. The annotation value is used to set or update the image reference in the `spec.image` field of the `CatalogSource` object. This helps avoid cluster upgrades leaving Operator installations in unsupported states or without a continued update path.

:::warning

You must ensure that the index image with the updated tag, in whichever registry it is stored in, is accessible by the cluster at the time of the cluster upgrade.

:::

<details>
<summary>Example catalog source with an image template</summary>

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  generation: 1
  name: example-catalog
  namespace: openshift-marketplace
  annotations:
    olm.catalogImageTemplate:
      "quay.io/example-org/example-catalog:v{kube_major_version}.{kube_minor_version}"
spec:
  displayName: Example Catalog
  image: quay.io/example-org/example-catalog:v1.35
  priority: -400
  publisher: Example Org
```

</details>

:::note

If the `spec.image` field and the `olm.catalogImageTemplate` annotation are both set, the `spec.image` field is overwritten by the resolved value from the annotation. If the annotation does not resolve to a usable pull spec, the catalog source falls back to the set `spec.image` value.

If the `spec.image` field is not set and the annotation does not resolve to a usable pull spec, OLM stops reconciliation of the catalog source and sets it into a human-readable error condition.

:::

For an OpenShift Container Platform 4.22 cluster, which uses Kubernetes 1.35, the `olm.catalogImageTemplate` annotation in the preceding example resolves to the following image reference:

```terminal
quay.io/example-org/example-catalog:v1.35
```

For future releases of OpenShift Container Platform, you can create updated index images for your custom catalogs that target the later Kubernetes version that is used by the later OpenShift Container Platform version. With the `olm.catalogImageTemplate` annotation set before the upgrade, upgrading the cluster to the later OpenShift Container Platform version would then automatically update the catalog’s index image as well.

#### Catalog health requirements {#olm-cs-health_olm-understanding-olm}

Operator Lifecycle Manager (OLM) requires that all Operator catalogs in a shared global namespace are healthy. When a catalog is unhealthy, Operator installation and update operations in that namespace fail with a `CatalogSourcesUnhealthy` condition.

Operator catalogs on a cluster are interchangeable from the perspective of installation resolution; a `Subscription` object might reference a specific catalog, but dependencies are resolved using all catalogs on the cluster.

For example, if Catalog A is unhealthy, a subscription referencing Catalog A could resolve a dependency in Catalog B, which the cluster administrator might not have been expecting, because B normally had a lower catalog priority than A.

As a cluster administrator, if you observe an unhealthy catalog and want to consider the catalog as invalid and resume Operator installations, see the "Removing custom catalogs" or "Disabling the default software catalog sources" sections for information about removing the unhealthy catalog.

**Additional resources**

- [Removing custom catalogs](/docs/operators/admin/olm-managing-custom-catalogs#olm-removing-catalogs_olm-managing-custom-catalogs)
- [Disabling the default OperatorHub catalog sources](/docs/operators/admin/olm-managing-custom-catalogs#olm-restricted-networks-operatorhub_olm-managing-custom-catalogs)

### Subscription {#olm-subscription_olm-understanding-olm}

A *subscription*, defined by a `Subscription` object, represents an intention to install an Operator. It is the custom resource that relates an Operator to a catalog source.

Subscriptions describe which channel of an Operator package to subscribe to, and whether to perform updates automatically or manually. If set to automatic, the subscription ensures Operator Lifecycle Manager (OLM) manages and upgrades the Operator to ensure that the latest version is always running in the cluster.

```yaml title="Example Subscription object"
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: example-operator
  namespace: example-namespace
spec:
  channel: stable
  name: example-operator
  source: example-catalog
  sourceNamespace: openshift-marketplace
```

This `Subscription` object defines the name and namespace of the Operator, as well as the catalog from which the Operator data can be found. The channel, such as `alpha`, `beta`, or `stable`, helps determine which Operator stream should be installed from the catalog source.

The names of channels in a subscription can differ between Operators, but the naming scheme should follow a common convention within a given Operator. For example, channel names might follow a minor release update stream for the application provided by the Operator (`1.2`, `1.3`) or a release frequency (`stable`, `fast`).

In addition to being easily visible from the OpenShift Container Platform web console, it is possible to identify when there is a newer version of an Operator available by inspecting the status of the related subscription. The value associated with the `currentCSV` field is the newest version that is known to OLM, and `installedCSV` is the version that is installed on the cluster.

**Additional resources**

- [Multitenancy and Operator colocation](/docs/operators/understanding/olm/olm-colocation#olm-colocation)
- [Viewing Operator subscription status by using the CLI](/docs/operators/admin/olm-status#olm-status-viewing-cli_olm-status)

### Install plan {#olm-installplan_olm-understanding-olm}

An *install plan*, defined by an `InstallPlan` object, describes a set of resources that Operator Lifecycle Manager (OLM) creates to install or upgrade to a specific version of an Operator. The version is defined by a cluster service version (CSV).

To install an Operator, a cluster administrator, or a user who has been granted Operator installation permissions, must first create a `Subscription` object. A subscription represents the intent to subscribe to a stream of available versions of an Operator from a catalog source. The subscription then creates an `InstallPlan` object to facilitate the installation of the resources for the Operator.

The install plan must then be approved according to one of the following approval strategies:

- If the subscription’s `spec.installPlanApproval` field is set to `Automatic`, the install plan is approved automatically.
- If the subscription’s `spec.installPlanApproval` field is set to `Manual`, the install plan must be manually approved by a cluster administrator or user with proper permissions.

After the install plan is approved, OLM creates the specified resources and installs the Operator in the namespace that is specified by the subscription.

<details>
<summary>Example `InstallPlan` object</summary>

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: InstallPlan
metadata:
  name: install-abcde
  namespace: operators
spec:
  approval: Automatic
  approved: true
  clusterServiceVersionNames:
    - my-operator.v1.0.1
  generation: 1
status:
  ...
  catalogSources: []
  conditions:
    - lastTransitionTime: '2021-01-01T20:17:27Z'
      lastUpdateTime: '2021-01-01T20:17:27Z'
      status: 'True'
      type: Installed
  phase: Complete
  plan:
    - resolving: my-operator.v1.0.1
      resource:
        group: operators.coreos.com
        kind: ClusterServiceVersion
        manifest: >-
        ...
        name: my-operator.v1.0.1
        sourceName: redhat-operators
        sourceNamespace: openshift-marketplace
        version: v1alpha1
      status: Created
    - resolving: my-operator.v1.0.1
      resource:
        group: apiextensions.k8s.io
        kind: CustomResourceDefinition
        manifest: >-
        ...
        name: webservers.web.servers.org
        sourceName: redhat-operators
        sourceNamespace: openshift-marketplace
        version: v1beta1
      status: Created
    - resolving: my-operator.v1.0.1
      resource:
        group: ''
        kind: ServiceAccount
        manifest: >-
        ...
        name: my-operator
        sourceName: redhat-operators
        sourceNamespace: openshift-marketplace
        version: v1
      status: Created
    - resolving: my-operator.v1.0.1
      resource:
        group: rbac.authorization.k8s.io
        kind: Role
        manifest: >-
        ...
        name: my-operator.v1.0.1-my-operator-6d7cbc6f57
        sourceName: redhat-operators
        sourceNamespace: openshift-marketplace
        version: v1
      status: Created
    - resolving: my-operator.v1.0.1
      resource:
        group: rbac.authorization.k8s.io
        kind: RoleBinding
        manifest: >-
        ...
        name: my-operator.v1.0.1-my-operator-6d7cbc6f57
        sourceName: redhat-operators
        sourceNamespace: openshift-marketplace
        version: v1
      status: Created
      ...
```

</details>

**Additional resources**

- [Multitenancy and Operator colocation](/docs/operators/understanding/olm/olm-colocation#olm-colocation)
- [Allowing non-cluster administrators to install Operators](/docs/operators/admin/olm-creating-policy#olm-creating-policy)

### Operator groups {#olm-operatorgroups-about_olm-understanding-olm}

An Operator group defines multitenant configuration for OLM-installed Operators through an `OperatorGroup` resource. An Operator group selects target namespaces where the required RBAC access is generated for member Operators .

The set of target namespaces is provided by a comma-delimited string stored in the `olm.targetNamespaces` annotation of a cluster service version (CSV). This annotation is applied to the CSV instances of member Operators and is projected into their deployments.

**Additional resources**

- [Operator groups](/docs/operators/understanding/olm/olm-understanding-operatorgroups#olm-understanding-operatorgroups)

### Operator conditions {#olm-about-operatorconditions_olm-understanding-olm}

Operator Lifecycle Manager (OLM) infers Operator state from Kubernetes resources, but some conditions require explicit communication. You can use the `OperatorCondition` custom resource definition (CRD) to tell OLM about supported conditions that affect lifecycle management.

:::note

By default, the `Spec.Conditions` array is not present in an `OperatorCondition` object until it is either added by a user or as a result of custom Operator logic.

:::

**Additional resources**

- [Operator conditions](/docs/operators/understanding/olm/olm-operatorconditions#olm-operatorconditions)
