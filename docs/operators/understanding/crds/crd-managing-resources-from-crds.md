---
title: Managing resources from custom resource definitions
sidebar_position: 2
---

# Managing resources from custom resource definitions {#crd-managing-resources-from-crds}

<a id="crd-managing-resources-from-crds"></a>

As a developer, you can manage custom resources (CRs) that come from custom resource definitions (CRDs) to work with the custom object types available in your cluster.

## Custom resource definitions {#crd-custom-resource-definitions_crd-managing-resources-from-crds}

In the Kubernetes API, a *resource* is an endpoint that stores a collection of API objects of a certain kind. For example, the built-in `Pods` resource contains a collection of `Pod` objects.

A *custom resource definition* (CRD) object defines a new, unique object type, called a *kind*, in the cluster and lets the Kubernetes API server handle its entire lifecycle.

*Custom resource* (CR) objects are created from CRDs that have been added to the cluster by a cluster administrator, allowing all cluster users to add the new resource type into projects.

Operators in particular make use of CRDs by packaging them with any required RBAC policy and other software-specific logic. Cluster administrators can also add CRDs manually to the cluster outside of the lifecycle of an Operator, making them available to all users.

:::note

While only cluster administrators can create CRDs, developers can create the CR from an existing CRD if they have read and write permission to it.

:::

## Creating custom resources from a file {#crd-creating-custom-resources-from-file_crd-managing-resources-from-crds}

After you add a custom resource definition (CRD) to the cluster, you can create custom resources (CRs) from a file by using the CLI.

**Prerequisites**

- CRD added to the cluster by a cluster administrator.

**Procedure**

1. Create a YAML file for the CR. In the following example definition, the `cronSpec` and `image` custom fields are set in a CR of `Kind: CronTab`. The `Kind` comes from the `spec.kind` field of the CRD object:
   ```yaml title="Example YAML file for a CR"
   apiVersion: "stable.example.com/v1"
   kind: CronTab
   metadata:
     name: my-new-cron-object
     finalizers:
     - finalizer.stable.example.com
   spec:
     cronSpec: "* * * * /5"
     image: my-awesome-cron-image
   ```

   where:

   <dl>
   <dt><code>apiVersion</code></dt>
   <dd>Specifies the group name and API version (name/version) from the CRD.</dd>
   <dt><code>kind</code></dt>
   <dd>Specifies the type in the CRD.</dd>
   <dt><code>metadata.name</code></dt>
   <dd>Specifies a name for the object.</dd>
   <dt><code>metadata.finalizers</code></dt>
   <dd>Specifies the finalizers for the object, if any. Finalizers allow controllers to implement conditions that must be completed before the object can be deleted.</dd>
   <dt><code>spec</code></dt>
   <dd>Specifies conditions specific to the type of object.</dd>
   </dl>
2. After you create the file, create the object:
   ```terminal
   $ oc create -f <file_name>.yaml
   ```

## Inspecting custom resources {#crd-inspecting-custom-resources_crd-managing-resources-from-crds}

You can inspect custom resource (CR) objects that exist in your cluster using the CLI.

**Prerequisites**

- A CR object exists in a namespace to which you have access.

**Procedure**

1. To get information on a specific kind of a CR, run:
   ```terminal
   $ oc get <kind>
   ```

   For example:

   ```terminal
   $ oc get crontab
   ```

   ```terminal title="Example output"
   NAME                 KIND
   my-new-cron-object   CronTab.v1.stable.example.com
   ```

   Resource names are not case-sensitive, and you can use either the singular or plural forms defined in the CRD, as well as any short name. For example:

   ```terminal
   $ oc get crontabs
   ```

   ```terminal
   $ oc get crontab
   ```

   ```terminal
   $ oc get ct
   ```
2. You can also view the raw YAML data for a CR:
   ```terminal
   $ oc get <kind> -o yaml
   ```

   For example:

   ```terminal
   $ oc get ct -o yaml
   ```

   ```terminal title="Example output"
   apiVersion: v1
   items:
   - apiVersion: stable.example.com/v1
     kind: CronTab
     metadata:
       clusterName: ""
       creationTimestamp: 2017-05-31T12:56:35Z
       deletionGracePeriodSeconds: null
       deletionTimestamp: null
       name: my-new-cron-object
       namespace: default
       resourceVersion: "285"
       selfLink: /apis/stable.example.com/v1/namespaces/default/crontabs/my-new-cron-object
       uid: 9423255b-4600-11e7-af6a-28d2447dc82b
     spec:
       cronSpec: '* * * * /5'
       image: my-awesome-cron-image
   ```

   The `spec` section in the output displays the custom configuration settings, such as `cronSpec` and `image`, that you defined when creating the object.
