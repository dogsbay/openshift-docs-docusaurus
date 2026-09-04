---
title: Operator Framework glossary of common terms
sidebar_position: 3
---

# Operator Framework glossary of common terms {#of-terms}

<a id="of-terms"></a>

Review the Operator Framework terminology, including Operator Lifecycle Manager (OLM) v1 terms, used throughout the documentation.

## Bundle {#olm-common-terms-bundle_of-terms}

In the bundle format, a *bundle* is a collection of an Operator CSV, manifests, and metadata. Together, they form a unique version of an Operator that can be installed onto the cluster.

## Bundle image {#olm-common-terms-bundle-image_of-terms}

In the bundle format, a *bundle image* is a container image that is built from Operator manifests and that contains one bundle. Bundle images are stored and distributed by Open Container Initiative (OCI) spec container registries, such as Quay.io or DockerHub.

## Catalog source {#olm-common-terms-catalogsource_of-terms}

A *catalog source* represents a store of metadata that OLM can query to discover and install Operators and their dependencies.

## Channel {#olm-common-terms-channel_of-terms}

A *channel* defines a stream of updates for an Operator and is used to roll out updates for subscribers. The head points to the latest version of that channel. For example, a `stable` channel would have all stable versions of an Operator arranged from the earliest to the latest.

An Operator can have several channels, and a subscription binding to a certain channel would only look for updates in that channel.

## Channel head {#olm-common-terms-channel-head_of-terms}

A *channel head* refers to the latest known update in a particular channel.

## Cluster service version {#olm-common-terms-csv_of-terms}

A *cluster service version (CSV)* is a YAML manifest created from Operator metadata that assists OLM in running the Operator in a cluster. It is the metadata that accompanies an Operator container image, used to populate user interfaces with information such as its logo, description, and version.

It is also a source of technical information that is required to run the Operator, like the RBAC rules it requires and which custom resources (CRs) it manages or depends on.

## Dependency {#olm-common-terms-dependency_of-terms}

An Operator may have a *dependency* on another Operator being present in the cluster. For example, the Vault Operator has a dependency on the etcd Operator for its data persistence layer.

OLM resolves dependencies by ensuring that all specified versions of Operators and CRDs are installed on the cluster during the installation phase. This dependency is resolved by finding and installing an Operator in a catalog that satisfies the required CRD API, and is not related to packages or bundles.

## Extension {#olm-common-terms-extension_of-terms}

Extensions enable cluster administrators to extend capabilities for users on their OpenShift Container Platform cluster. Extensions are managed by Operator Lifecycle Manager (OLM) v1.

The `ClusterExtension` API streamlines management of installed extensions, which includes Operators via the `registry+v1` bundle format, by consolidating user-facing APIs into a single object. Administrators and SREs can use the API to automate processes and define desired states by using GitOps principles.

## Index image {#olm-common-terms-index-image_of-terms}

In the bundle format, an *index image* refers to an image of a database (a database snapshot) that contains information about Operator bundles including CSVs and CRDs of all versions. This index can host a history of Operators on a cluster and be maintained by adding or removing Operators using the `opm` CLI tool.

## Install plan {#olm-common-terms-installplan_of-terms}

An *install plan* is a calculated list of resources to be created to automatically install or upgrade a CSV.

## Multitenancy {#olm-common-terms-multitenancy_of-terms}

A *tenant* in OpenShift Container Platform is a user or group of users that share common access and privileges for a set of deployed workloads, typically represented by a namespace or project. You can use tenants to provide a level of isolation between different groups or teams.

When a cluster is shared by multiple users or groups, it is considered a *multitenant* cluster.

## Operator {#olm-common-terms-operator_of-terms}

Operators are a method of packaging, deploying, and managing a Kubernetes application. A Kubernetes application is an app that is both deployed on Kubernetes and managed using the Kubernetes APIs and `kubectl` or `oc` tooling.

In Operator Lifecycle Manager (OLM) v1, the `ClusterExtension` API streamlines management of installed extensions, which includes Operators via the `registry+v1` bundle format.

## Operator group {#olm-common-terms-operatorgroup_of-terms}

An *Operator group* configures all Operators deployed in the same namespace as the `OperatorGroup` object to watch for their CR in a list of namespaces or cluster-wide.

## Package {#olm-common-terms-package_of-terms}

In the bundle format, a *package* is a directory that encloses all released history of an Operator with each version. A released version of an Operator is described in a CSV manifest alongside the CRDs.

## Registry {#olm-common-terms-registry_of-terms}

A *registry* is a database that stores bundle images of Operators, each with all of its latest and historical versions in all channels.

## Subscription {#olm-common-terms-subscription_of-terms}

A *subscription* keeps CSVs up to date by tracking a channel in a package.

## Update graph {#olm-common-terms-update-graph_of-terms}

An *update graph* links versions of CSVs together, similar to the update graph of any other packaged software. Operators can be installed sequentially, or certain versions can be skipped. The update graph is expected to grow only at the head with newer versions being added.

Also known as *update edges* or *update paths*.
