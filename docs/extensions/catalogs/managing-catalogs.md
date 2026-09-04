---
title: Managing catalogs
sidebar_position: 3
---

# Managing catalogs

<a id="managing-catalogs"></a>

Cluster administrators can add *catalogs*, or curated collections of Operators and Kubernetes extensions, to their clusters. Operator authors publish their products to these catalogs.

When you add a catalog to your cluster, you have access to the versions, patches, and over-the-air updates of the Operators and extensions that are published to the catalog.

You can manage catalogs and extensions declaratively from the CLI by using custom resources (CRs).

*File-based catalogs* are the latest iteration of the catalog format in Operator Lifecycle Manager (OLM). It is a plain text-based (JSON or YAML) and declarative config evolution of the earlier SQLite database format, and it is fully compatible with earlier versions.

:::warning

Kubernetes periodically deprecates certain APIs that are removed in subsequent releases. As a result, Operators are unable to use removed APIs starting with the version of OpenShift Container Platform that uses the Kubernetes version that removed the API.

:::

## About catalogs in OLM v1 {#olmv1-about-catalogs_managing-catalogs}

You can discover installable content by querying a catalog for Kubernetes extensions, such as Operators and controllers, by using the catalogd component.

Catalogd is a Kubernetes extension that unpacks catalog content for on-cluster clients and is part of the Operator Lifecycle Manager (OLM) v1 suite of microservices. Currently, catalogd unpacks catalog content that is packaged and distributed as container images.

**Additional resources**

- [File-based catalogs](/docs/extensions/catalogs/fbc#fbc)

## Red Hat-provided Operator catalogs in OLM v1 {#olmv1-red-hat-catalogs_managing-catalogs}

Operator Lifecycle Manager (OLM) v1 includes several Red Hat-provided Operator catalogs on the cluster by default. If you want to add a catalog to your cluster, create a custom resource (CR) for the catalog and apply it to the cluster.

The following custom resource (CR) examples show the default catalogs installed on the cluster:

```yaml title="Red&#160;Hat Operators catalog"
apiVersion: olm.operatorframework.io/v1
kind: ClusterCatalog
metadata:
  name: openshift-redhat-operators
spec:
  priority: -100
  source:
    image:
      pollIntervalMinutes: <poll_interval_duration>
      ref: registry.redhat.io/redhat/redhat-operator-index:v4.22
    type: Image
```

Replace `<poll_interval_duration>` with the interval in minutes for polling the remote registry for newer image digests. To disable polling, do not set the field.

```yaml title="Certified Operators catalog"
apiVersion: olm.operatorframework.io/v1
kind: ClusterCatalog
metadata:
  name: openshift-certified-operators
spec:
priority: -200
  source:
    type: image
    image:
      pollIntervalMinutes: 10
      ref: registry.redhat.io/redhat/certified-operator-index:v4.22
    type: Image
```

```yaml title="Red&#160;Hat Marketplace catalog"
apiVersion: olm.operatorframework.io/v1
kind: ClusterCatalog
metadata:
  name: openshift-redhat-marketplace
spec:
  priority: -300
  source:
    image:
      pollIntervalMinutes: 10
      ref: registry.redhat.io/redhat/redhat-marketplace-index:v4.22
    type: Image
```

```yaml title="Community Operators catalog"
apiVersion: olm.operatorframework.io/v1
kind: ClusterCatalog
metadata:
  name: openshift-community-operators
spec:
  priority: -400
  source:
    image:
      pollIntervalMinutes: 10
      ref: registry.redhat.io/redhat/community-operator-index:v4.22
    type: Image
```

The following command adds a catalog to your cluster:

```terminal title="Command syntax"
$ oc apply -f <catalog_name>.yaml
```

Replace `<catalog_name>.yaml` with the catalog CR, such as `my-catalog.yaml`.

## Adding a catalog to a cluster {#olmv1-adding-a-catalog-to-a-cluster_managing-catalogs}

To add a catalog to a cluster for Operator Lifecycle Manager (OLM) v1 usage, create a `ClusterCatalog` custom resource (CR) and apply it to the cluster.

**Procedure**

1. Create a catalog custom resource (CR), similar to the following example:
   ```yaml title="Example my-redhat-operators.yaml file"
   apiVersion: olm.operatorframework.io/v1
   kind: ClusterCatalog
   metadata:
     name: my-redhat-operators
   spec:
     priority: 1000
     source:
       image:
         pollIntervalMinutes: 10
         ref: registry.redhat.io/redhat/community-operator-index:v4.22
       type: Image
   ```

   where:

   <dl>
   <dt><code>metadata.name</code></dt>
   <dd>The catalog is automatically labeled with the value of the <code>metadata.name</code> field when it is applied to the cluster. For more information about labels and catalog selection, see "Catalog content resolution".</dd>
   <dt><code>spec.priority</code></dt>
   <dd>Optional: Specifies the priority of the catalog in relation to the other catalogs on the cluster. For more information, see "Catalog selection by priority".</dd>
   <dt><code>spec.source.image.pollIntervalMinutes</code></dt>
   <dd>Specifies the interval in minutes for polling the remote registry for newer image digests. To disable polling, do not set the field.</dd>
   <dt><code>spec.source.image.ref</code></dt>
   <dd>Specifies the catalog image in the <code>spec.source.image.ref</code> field.</dd>
   </dl>
2. Add the catalog to your cluster by running the following command:
   ```terminal
   $ oc apply -f my-redhat-operators.yaml
   ```

   ```text title="Example output"
   clustercatalog.olm.operatorframework.io/my-redhat-operators created
   ```

**Verification**

- Run the following commands to verify the status of your catalog:
  1. Check if your catalog is available by running the following command:
     ```terminal
     $ oc get clustercatalog
     ```

     ```text title="Example output"
     NAME                            LASTUNPACKED   SERVING   AGE
     my-redhat-operators             55s            True      64s
     openshift-certified-operators   83m            True      84m
     openshift-community-operators   43m            True      84m
     openshift-redhat-marketplace    83m            True      84m
     openshift-redhat-operators      54m            True      84m
     ```
  2. Check the status of your catalog by running the following command:
     ```terminal
     $ oc describe clustercatalog my-redhat-operators
     ```

     ```text title="Example output"
     Name:         my-redhat-operators
     Namespace:
     Labels:       olm.operatorframework.io/metadata.name=my-redhat-operators
     Annotations:  <none>
     API Version:  olm.operatorframework.io/v1
     Kind:         ClusterCatalog
     Metadata:
       Creation Timestamp:  2025-02-18T20:28:50Z
       Finalizers:
         olm.operatorframework.io/delete-server-cache
       Generation:        1
       Resource Version:  50248
       UID:               86adf94f-d2a8-4e70-895b-31139f2eeab7
     Spec:
       Availability Mode:  Available
       Priority:           1000
       Source:
         Image:
           Poll Interval Minutes:  10
           Ref:                    registry.redhat.io/redhat/community-operator-index:v4.22
         Type:                     Image
     Status:
       Conditions:
         Last Transition Time:  2025-02-18T20:29:00Z
         Message:               Successfully unpacked and stored content from resolved source
         Observed Generation:   1
         Reason:                Succeeded
         Status:                True
         Type:                  Progressing
         Last Transition Time:  2025-02-18T20:29:00Z
         Message:               Serving desired content from resolved source
         Observed Generation:   1
         Reason:                Available
         Status:                True
         Type:                  Serving
       Last Unpacked:           2025-02-18T20:28:59Z
       Resolved Source:
         Image:
           Ref:  registry.redhat.io/redhat/community-operator-index@sha256:11627ea6fdd06b8092df815076e03cae9b7cede8b353c0b461328842d02896c5
         Type:   Image
       Urls:
         Base:  https://catalogd-service.openshift-catalogd.svc/catalogs/my-redhat-operators
     Events:    <none>
     ```

     In the output, the `Status` section describes the status of the catalog. In the `Status` section, the `Reason` field displays the reason the catalog is in the current state. In the `Resolved Source` section, the `Ref` field displays the image reference of the catalog.

## Deleting a catalog {#olmv1-deleting-catalog_managing-catalogs}

You can delete a catalog by deleting its custom resource (CR).

**Prerequisites**

- You have a catalog installed.

**Procedure**

- Delete a catalog by running the following command:
  ```terminal
  $ oc delete clustercatalog <catalog_name>
  ```

  ```text title="Example output"
  clustercatalog.olm.operatorframework.io "my-redhat-operators" deleted
  ```

**Verification**

- Verify the catalog is deleted by running the following command:
  ```terminal
  $ oc get clustercatalog
  ```

## Disabling a default catalog {#olmv1-disabling-a-default-catalog_managing-catalogs}

You can disable the Red Hat-provided catalogs that are included with OpenShift Container Platform by default.

**Procedure**

- Disable a default catalog by running the following command:
  ```terminal
  $ oc patch clustercatalog openshift-certified-operators -p \
    '{"spec": {"availabilityMode": "Unavailable"}}' --type=merge
  ```

  ```text title="Example output"
  clustercatalog.olm.operatorframework.io/openshift-certified-operators patched
  ```

**Verification**

- Verify the catalog is disabled by running the following command:
  ```terminal
  $ oc get clustercatalog openshift-certified-operators
  ```

  ```text title="Example output"
  NAME                            LASTUNPACKED   SERVING   AGE
  openshift-certified-operators                  False     6h54m
  ```
