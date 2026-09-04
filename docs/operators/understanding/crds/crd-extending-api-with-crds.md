---
title: Extending the Kubernetes API with custom resource definitions
sidebar_position: 1
---

# Extending the Kubernetes API with custom resource definitions {#crd-extending-api-with-crds}

<a id="crd-extending-api-with-crds"></a>

To extend the Kubernetes API with custom object types that behave like built-in Kubernetes objects, cluster administrators can create and manage custom resource definitions (CRDs) on their OpenShift Container Platform cluster.

## Custom resource definitions {#crd-custom-resource-definitions_crd-extending-api-with-crds}

In the Kubernetes API, a *resource* is an endpoint that stores a collection of API objects of a certain kind. For example, the built-in `Pods` resource contains a collection of `Pod` objects.

A *custom resource definition* (CRD) object defines a new, unique object type, called a *kind*, in the cluster and lets the Kubernetes API server handle its entire lifecycle.

*Custom resource* (CR) objects are created from CRDs that have been added to the cluster by a cluster administrator, allowing all cluster users to add the new resource type into projects.

When a cluster administrator adds a new CRD to the cluster, the Kubernetes API server reacts by creating a new RESTful resource path that can be accessed by the entire cluster or a single project (namespace) and begins serving the specified CR.

Cluster administrators that want to grant access to the CRD to other users can use cluster role aggregation to grant access to users with the `admin`, `edit`, or `view` default cluster roles. Cluster role aggregation allows the insertion of custom policy rules into these cluster roles. This behavior integrates the new resource into the RBAC policy of the cluster as if it was a built-in resource.

Operators in particular make use of CRDs by packaging them with any required RBAC policy and other software-specific logic. Cluster administrators can also add CRDs manually to the cluster outside of the lifecycle of an Operator, making them available to all users.

:::note

While only cluster administrators can create CRDs, developers can create the CR from an existing CRD if they have read and write permission to it.

:::

## Creating a custom resource definition {#crd-creating-custom-resources-definition_crd-extending-api-with-crds}

To create custom resource (CR) objects, cluster administrators must first create a custom resource definition (CRD).

**Prerequisites**

- Access to an OpenShift Container Platform cluster with `cluster-admin` user privileges.

**Procedure**

1. Create a YAML file that contains the following field types:
   ```yaml title="Example YAML file for a CRD"
   apiVersion: apiextensions.k8s.io/v1
   kind: CustomResourceDefinition
   metadata:
     name: crontabs.stable.example.com
   spec:
     group: stable.example.com
     versions:
       - name: v1
         served: true
         storage: true
         schema:
           openAPIV3Schema:
             type: object
             properties:
               spec:
                 type: object
                 properties:
                   cronSpec:
                     type: string
                   image:
                     type: string
                   replicas:
                     type: integer
     scope: Namespaced
     names:
       plural: crontabs
       singular: crontab
       kind: CronTab
       shortNames:
       - ct
   ```

   where:

   <dl>
   <dt><code>apiVersion</code></dt>
   <dd>Specifies the <code>apiextensions.k8s.io/v1</code> API parameter.</dd>
   <dt><code>metadata.name</code></dt>
   <dd>Specifies a name for the definition. This must be in the <code>&lt;plural-name&gt;.&lt;group&gt;</code> format using the values from the <code>group</code> and <code>plural</code> fields.</dd>
   <dt><code>spec.group</code></dt>
   <dd>Specifies a group name for the API. An API group is a collection of objects that are logically related. For example, all batch objects like <code>Job</code> or <code>ScheduledJob</code> could be in the batch API group (such as <code>batch.api.example.com</code>). A good practice is to use a fully-qualified-domain name (FQDN) of your organization.</dd>
   <dt><code>spec.versions.name</code></dt>
   <dd>Specifies a version name to be used in the URL. Each API group can exist in multiple versions, for example <code>v1alpha</code>, <code>v1beta</code>, <code>v1</code>.</dd>
   <dt><code>spec.scope</code></dt>
   <dd>Specifies whether the custom objects are available to a project (<code>Namespaced</code>) or all projects in the cluster (<code>Cluster</code>).</dd>
   <dt><code>spec.names.plural</code></dt>
   <dd>Specifies the plural name to use in the URL. The <code>plural</code> field is the same as a resource in an API URL.</dd>
   <dt><code>spec.names.singular</code></dt>
   <dd>Specifies a singular name to use as an alias on the CLI and for display.</dd>
   <dt><code>spec.names.kind</code></dt>
   <dd>Specifies the kind of objects that can be created. The type can be in camel case.</dd>
   <dt><code>spec.names.shortNames</code></dt>
   <dd>Specifies a shorter string to match your resource on the CLI.</dd>
   </dl>

   :::note

   By default, a CRD is cluster-scoped and available to all projects.

   :::
2. Create the CRD object:
   ```terminal
   $ oc create -f <file_name>.yaml
   ```

   A new RESTful API endpoint is created at:

   ```terminal
   /apis/<spec:group>/<spec:version>/<scope>/*/<names-plural>/...
   ```

   For example, using the example file, the following endpoint is created:

   ```terminal
   /apis/stable.example.com/v1/namespaces/*/crontabs/...
   ```

   You can now use this endpoint URL to create and manage CRs. The object kind is based on the `spec.kind` field of the CRD object you created.

## Creating cluster roles for custom resource definitions {#crd-creating-aggregated-cluster-role_crd-extending-api-with-crds}

Cluster administrators can grant permissions to existing cluster-scoped custom resource definitions (CRDs). If you use the `admin`, `edit`, and `view` default cluster roles, you can take advantage of cluster role aggregation for their rules.

:::warning

You must explicitly assign permissions to each of these roles. The roles with more permissions do not inherit rules from roles with fewer permissions. If you assign a rule to a role, you must also assign that verb to roles that have more permissions. For example, if you grant the `get crontabs` permission to the view role, you must also grant it to the `edit` and `admin` roles. The `admin` or `edit` role is usually assigned to the user that created a project through the project template.

:::

**Prerequisites**

- Create a CRD.

**Procedure**

1. Create a cluster role definition file for the CRD. The cluster role definition is a YAML file that contains the rules that apply to each cluster role. An OpenShift Container Platform controller adds the rules that you specify to the default cluster roles.
   ```yaml title="Example YAML file for a cluster role definition"
   kind: ClusterRole
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     name: aggregate-cron-tabs-admin-edit
     labels:
       rbac.authorization.k8s.io/aggregate-to-admin: "true"
       rbac.authorization.k8s.io/aggregate-to-edit: "true"
   rules:
   - apiGroups: ["stable.example.com"]
     resources: ["crontabs"]
     verbs: ["get", "list", "watch", "create", "update", "patch", "delete", "deletecollection"]
   ---
   kind: ClusterRole
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     name: aggregate-cron-tabs-view
     labels:
       # Add these permissions to the "view" default role.
       rbac.authorization.k8s.io/aggregate-to-view: "true"
       rbac.authorization.k8s.io/aggregate-to-cluster-reader: "true"
   rules:
   - apiGroups: ["stable.example.com"]
     resources: ["crontabs"]
     verbs: ["get", "list", "watch"]
   ```

   where:

   <dl>
   <dt><code>apiVersion</code></dt>
   <dd>Specifies the <code>rbac.authorization.k8s.io/v1</code> API.</dd>
   <dt><code>metadata.name</code></dt>
   <dd>Specifies a name for the definition.</dd>
   <dt><code>metadata.labels.rbac.authorization.k8s.io/aggregate-to-admin</code></dt>
   <dd>Specifies <code>"true"</code> to enable cluster role aggregation to the admin role.</dd>
   <dt><code>metadata.labels.rbac.authorization.k8s.io/aggregate-to-edit</code></dt>
   <dd>Specifies <code>"true"</code> to grant permissions to the edit default role.</dd>
   <dt><code>rules.apiGroups</code></dt>
   <dd>Specifies the group name of the CRD.</dd>
   <dt><code>rules.resources</code></dt>
   <dd>Specifies the plural name of the CRD that these rules apply to.</dd>
   <dt><code>rules.verbs</code></dt>
   <dd>Specifies the verbs that represent the permissions that are granted to the role. For example, apply read and write permissions to the <code>admin</code> and <code>edit</code> roles and only read permission to the <code>view</code> role.</dd>
   <dt><code>metadata.labels.rbac.authorization.k8s.io/aggregate-to-view</code></dt>
   <dd>Specifies <code>"true"</code> to grant permissions to the <code>view</code> default role.</dd>
   <dt><code>metadata.labels."rbac.authorization.k8s.io/aggregate-to-cluster-reader"</code></dt>
   <dd>Specifies <code>"true"</code> to grant permissions to the <code>cluster-reader</code> default role.</dd>
   </dl>
2. Create the cluster role:
   ```terminal
   $ oc create -f <file_name>.yaml
   ```

## Creating custom resources from a file {#crd-creating-custom-resources-from-file_crd-extending-api-with-crds}

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

## Inspecting custom resources {#crd-inspecting-custom-resources_crd-extending-api-with-crds}

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
