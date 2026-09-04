---
title: Preparing for users
sidebar_position: 9
---

# Preparing for users {#post-install-preparing-for-users}

<a id="post-install-preparing-for-users"></a>

You can prepare your cluster for users by configuring authentication and permissions, managing initial administrative access, and making Operators available through the software catalog.

After installing OpenShift Container Platform, you can further expand and customize your cluster to your requirements, including taking steps to prepare for users.

## Understanding identity provider configuration {#post-install-understanding-identity-provider}

The OpenShift Container Platform control plane includes a built-in OAuth server. Developers and administrators obtain OAuth access tokens to authenticate themselves to the API.

As an administrator, you can configure OAuth to specify an identity provider after you install your cluster.

### Identity providers in OpenShift Container Platform {#identity-provider-overview_post-install-preparing-for-users}

You can configure identity providers by creating a custom resource (CR) that describes the provider and adding it to the cluster. Identity providers enable user authentication in OpenShift Container Platform beyond the default `kubeadmin` user.

:::note

OpenShift Container Platform usernames containing `/`, `:`, and `%` are not supported.

:::

### Supported identity providers {#post-install-supported-identity-providers_post-install-preparing-for-users}

You can configure the following types of identity providers:

<table>
<thead>
<tr>
  <th>Identity provider</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><a href="/docs/authentication/identity_providers/configuring-htpasswd-identity-provider#configuring-htpasswd-identity-provider">htpasswd</a></td>
  <td>Configure the <code>htpasswd</code> identity provider to validate user names and passwords against a flat file generated using <a href="http://httpd.apache.org/docs/2.4/programs/htpasswd.html"><code>htpasswd</code></a>.</td>
</tr>
<tr>
  <td><a href="/docs/authentication/identity_providers/configuring-keystone-identity-provider#configuring-keystone-identity-provider">Keystone</a></td>
  <td>Configure the <code>keystone</code> identity provider to integrate your OpenShift Container Platform cluster with Keystone to enable shared authentication with an OpenStack Keystone v3 server configured to store users in an internal database.</td>
</tr>
<tr>
  <td><a href="/docs/authentication/identity_providers/configuring-ldap-identity-provider#configuring-ldap-identity-provider">LDAP</a></td>
  <td>Configure the <code>ldap</code> identity provider to validate user names and passwords against an LDAPv3 server, using simple bind authentication.</td>
</tr>
<tr>
  <td><a href="/docs/authentication/identity_providers/configuring-basic-authentication-identity-provider#configuring-basic-authentication-identity-provider">Basic authentication</a></td>
  <td>Configure a <code>basic-authentication</code> identity provider for users to log in to OpenShift Container Platform with credentials validated against a remote identity provider. Basic authentication is a generic backend integration mechanism.</td>
</tr>
<tr>
  <td><a href="/docs/authentication/identity_providers/configuring-request-header-identity-provider#configuring-request-header-identity-provider">Request header</a></td>
  <td>Configure a <code>request-header</code> identity provider to identify users from request header values, such as <code>X-Remote-User</code>. It is typically used in combination with an authenticating proxy, which sets the request header value.</td>
</tr>
<tr>
  <td><a href="/docs/authentication/identity_providers/configuring-github-identity-provider#configuring-github-identity-provider">GitHub or GitHub Enterprise</a></td>
  <td>Configure a <code>github</code> identity provider to validate user names and passwords against GitHub or GitHub Enterprise's OAuth authentication server.</td>
</tr>
<tr>
  <td><a href="/docs/authentication/identity_providers/configuring-gitlab-identity-provider#configuring-gitlab-identity-provider">GitLab</a></td>
  <td>Configure a <code>gitlab</code> identity provider to use <a href="https://gitlab.com/">GitLab.com</a> or any other GitLab instance as an identity provider.</td>
</tr>
<tr>
  <td><a href="/docs/authentication/identity_providers/configuring-google-identity-provider#configuring-google-identity-provider">Google</a></td>
  <td>Configure a <code>google</code> identity provider using <a href="https://developers.google.com/identity/protocols/OpenIDConnect">Google's OpenID Connect integration</a>.</td>
</tr>
<tr>
  <td><a href="/docs/authentication/identity_providers/configuring-oidc-identity-provider#configuring-oidc-identity-provider">OpenID Connect</a></td>
  <td>Configure an <code>oidc</code> identity provider to integrate with an OpenID Connect identity provider using an <a href="http://openid.net/specs/openid-connect-core-1_0.html#CodeFlowAuth">Authorization Code Flow</a>.</td>
</tr>
</tbody>
</table>

Once an identity provider has been defined, you can [use RBAC to define and apply permissions](/docs/authentication/using-rbac#authorization-overview_using-rbac).

### Identity provider parameters {#identity-provider-parameters_post-install-preparing-for-users}

The following parameters are common to all identity providers:

<table>
<thead>
<tr>
  <th>Parameter</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>name</code></td>
  <td>The provider name is prefixed to provider user names to form an identity name.</td>
</tr>
<tr>
  <td><code>mappingMethod</code></td>
  <td>Defines how new identities are mapped to users when they log in. Enter one of the following values:<br><br>claim:: The default value. Provisions a user with the identity's preferred user name. Fails if a user with that user name is already mapped to another identity. lookup:: Looks up an existing identity, user identity mapping, and user, but does not automatically provision users or identities. This allows cluster administrators to set up identities and users manually, or using an external process. Using this method requires you to manually provision users. add:: Provisions a user with the identity's preferred user name. If a user with that user name already exists, the identity is mapped to the existing user, adding to any existing identity mappings for the user. Required when multiple identity providers are configured that identify the same set of users and map to the same user names.</td>
</tr>
</tbody>
</table>

:::note

When adding or changing identity providers, you can map identities from the new provider to existing users by setting the `mappingMethod` parameter to `add`.

:::

### Sample identity provider CR {#identity-provider-default-CR_post-install-preparing-for-users}

You can use a custom resource (CR) to see the parameters and default values that you use to configure an identity provider.

The following example uses the htpasswd identity provider.

```yaml title="Sample identity provider CR"
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: my_identity_provider
    mappingMethod: claim
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpass-secret
```

where:

`spec.identityProviders.name`
: Specifies the provider name, which is prefixed to provider user names to form an identity name.

`spec.identityProviders.mappingMethod`
: Specifies how mappings are established between this provider’s identities and `User` objects.

`spec.identityProviders.htpasswd.fileData.name`
: Specifies an existing secret containing a file generated using [`htpasswd`](http://httpd.apache.org/docs/2.4/programs/htpasswd.html).

## Using RBAC to define and apply permissions {#post-install-using-rbac-to-define-and-apply-permissions_post-install-preparing-for-users}

Understand and apply role-based access control (RBAC) to manage user and service account permissions in your cluster.

### RBAC overview {#authorization-overview_post-install-preparing-for-users}

You can use role-based access control to configure whether users and groups can perform specific actions on cluster or project resources by evaluating roles, rules, and bindings.

Role-based access control (RBAC) objects determine whether a user is allowed to perform a given action within a project.

Cluster administrators

can use the cluster roles and bindings to control who has various access levels to the OpenShift Container Platform platform itself and all projects.

Developers can use local roles and bindings to control who has access to their projects. Note that authorization is a separate step from authentication, which is more about determining the identity of who is taking the action.

Authorization is managed using:

| Authorization object | Description |
| --- | --- |
| Rules | Sets of permitted verbs on a set of objects. For example, whether a user or service account can `create` pods. |
| Roles | Collections of rules. You can associate, or bind, users and groups to multiple roles. |
| Bindings | Associations between users and/or groups with a role. |

There are two levels of RBAC roles and bindings that control authorization:

| RBAC level | Description |
| --- | --- |
| Cluster RBAC | Roles and bindings that are applicable across all projects. *Cluster roles* exist cluster-wide, and *cluster role bindings* can reference only cluster roles. |
| Local RBAC | Roles and bindings that are scoped to a given project. While *local roles* exist only in a single project, local role bindings can reference *both* cluster and local roles. |

A cluster role binding is a binding that exists at the cluster level. A role binding exists at the project level. The cluster role *view* must be bound to a user using a local role binding for that user to view the project. Create local roles only if a cluster role does not provide the set of permissions needed for a particular situation.

This two-level hierarchy allows reuse across multiple projects through the cluster roles while allowing customization inside of individual projects through local roles.

During evaluation, both the cluster role bindings and the local role bindings are used. For example:

1. Cluster-wide "allow" rules are checked.
2. Locally-bound "allow" rules are checked.
3. Deny by default.

#### Default cluster roles {#default-roles_post-install-preparing-for-users}

OpenShift Container Platform includes a set of default cluster roles that you can bind to users and groups cluster-wide or locally.

:::warning

It is not recommended to manually modify the default cluster roles. Modifications to these system roles can prevent a cluster from functioning properly.

:::

| Default cluster role | Description |
| --- | --- |
| `admin` | A project manager. If used in a local binding, an `admin` has rights to view any resource in the project and modify any resource in the project except for quota. |
| `basic-user` | A user that can get basic information about projects and users. |
| `cluster-admin` | A super-user that can perform any action in any project. When bound to a user with a local binding, they have full control over quota and every action on every resource in the project. |
| `cluster-status` | A user that can get basic cluster status information. |
| `cluster-reader` | A user that can get or view most of the objects but cannot modify them. |
| `edit` | A user that can modify most objects in a project but does not have the power to view or modify roles or bindings. |
| `self-provisioner` | A user that can create their own projects. |
| `view` | A user who cannot make any modifications, but can see most objects in a project. They cannot view or modify roles or bindings. |

Be mindful of the difference between local and cluster bindings. For example, if you bind the `cluster-admin` role to a user by using a local role binding, it might appear that this user has the privileges of a cluster administrator. This is not the case. Binding the `cluster-admin` to a user in a project grants super administrator privileges for only that project to the user. That user has the permissions of the cluster role `admin`, plus a few additional permissions like the ability to edit rate limits, for that project. This binding can be confusing via the web console UI, which does not list cluster role bindings that are bound to true cluster administrators. However, it does list local role bindings that you can use to locally bind `cluster-admin`.

The relationships between cluster roles, local roles, cluster role bindings, local role bindings, users, groups and service accounts are illustrated below.

![OpenShift Container Platform RBAC](/images/rbac.png)

:::warning

The `get pods/exec`, `get pods/*`, and `get *` rules grant execution privileges when they are applied to a role. Apply the principle of least privilege and assign only the minimal RBAC rights required for users and agents. For more information, see "RBAC rules allow execution privileges".

:::

#### Evaluating authorization {#evaluating-authorization_post-install-preparing-for-users}

OpenShift Container Platform evaluates authorization by using:

Identity
: The user name and list of groups that the user belongs to.

Action
: The action you perform. In most cases, this consists of:

  - **Project**: The project you access. A project is a Kubernetes namespace with additional annotations that allows a community of users to organize and manage their content in isolation from other communities.
  - **Verb** : The action itself:  `get`, `list`, `create`, `update`, `delete`, `deletecollection`, or `watch`.
  - **Resource name**: The API endpoint that you access.

Bindings
: The full list of bindings, the associations between users or groups with a role.

OpenShift Container Platform evaluates authorization by using the following steps:

1. The identity and the project-scoped action is used to find all bindings that apply to the user or their groups.
2. Bindings are used to locate all the roles that apply.
3. Roles are used to find all the rules that apply.
4. The action is checked against each rule to find a match.
5. If no matching rule is found, the action is then denied by default.

:::tip

Remember that users and groups can be associated with, or bound to, multiple roles at the same time.

:::

Project administrators can use the CLI to

view local roles and bindings, including a matrix of the verbs and resources each are associated with.

:::warning

The cluster role bound to the project administrator is limited in a project through a local binding. It is not bound cluster-wide like the cluster roles granted to the **cluster-admin** or **system:admin**.

Cluster roles are roles defined at the cluster level but can be bound either at the cluster level or at the project level.

:::

#### Cluster role aggregation {#cluster-role-aggregations_post-install-preparing-for-users}

The default admin, edit, view, and cluster-reader cluster roles support cluster role aggregation, where the cluster rules for each role are dynamically updated as new rules are created. This feature is relevant only if you extend the Kubernetes API by creating custom resources.

**Additional resources**

- [RBAC rules allow execution privileges](https://access.redhat.com/solutions/6989997)
- [Aggregated ClusterRoles (Kubernetes documentation)](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#aggregated-clusterroles)

### Projects and namespaces {#rbac-projects-namespaces_post-install-preparing-for-users}

You can use projects and namespaces to organize and isolate cluster resources. These resources provide boundaries for access control, policies, quotas, and service accounts.

A Kubernetes *namespace* provides a mechanism to scope resources in a cluster. The Kubernetes documentation has more information on namespaces.

Namespaces provide a unique scope for:

- Named resources to avoid basic naming collisions.
- Delegated management authority to trusted users.
- The ability to limit community resource consumption.

Most objects in the system are scoped by namespace, but some are excepted and have no namespace, including nodes and users.

A *project* is a Kubernetes namespace with additional annotations and is the central vehicle by which access to resources for regular users is managed. A project allows a community of users to organize and manage their content in isolation from other communities. Users must be given access to projects by administrators, or if allowed to create projects, automatically have access to their own projects.

Projects can have a separate `name`, `displayName`, and `description`.

- The mandatory `name` is a unique identifier for the project and is most visible when using the CLI tools or API. The maximum name length is 63 characters.
- The optional `displayName` is how the project is displayed in the web console (defaults to `name`).
- The optional `description` can be a more detailed description of the project and is also visible in the web console.

Each project scopes its own set of:

| Object | Description |
| --- | --- |
| `Objects` | Pods, services, replication controllers, etc. |
| `Policies` | Rules for which users can or cannot perform actions on objects. |
| `Constraints` | Quotas for each kind of object that can be limited. |
| `Service accounts` | Service accounts act automatically with designated access to objects in the project. |

Cluster administrators

can create projects and delegate administrative rights for the project to any member of the user community. Cluster administrators can also allow developers to create their own projects.

Developers and administrators can interact with projects by using the CLI or the web console.

**Additional resources**

- [Kubernetes documentation on namespaces](https://kubernetes.io/docs/tasks/administer-cluster/namespaces/)

### Default projects {#rbac-default-projects_post-install-preparing-for-users}

Default projects host critical cluster and infrastructure components. By understanding their purpose, you can avoid making changes that could disrupt essential cluster services.

OpenShift Container Platform includes several default projects, and projects starting with `openshift-` are the most essential to users. These projects host master components that run as pods and other infrastructure components. The pods created in these namespaces that have a critical pod annotation are considered critical, and they have guaranteed admission by kubelet. Pods created for master components in these namespaces are already marked as critical.

:::warning

Do not run workloads in or share access to default projects. Default projects are reserved for running core cluster components.

The following default projects are considered highly privileged: `default`, `kube-public`, `kube-system`, `openshift`, `openshift-infra`, `openshift-node`, and other system-created projects that have the `openshift.io/run-level` label set to `0` or `1`. Functionality that relies on admission plugins, such as pod security admission, security context constraints, cluster resource quotas, and image reference resolution, does not work in highly privileged projects.

:::

**Additional resources**

- [Guaranteed Scheduling For Critical Add-On Pods (Kubernetes documentation)](https://kubernetes.io/docs/tasks/administer-cluster/guaranteed-scheduling-critical-addon-pods/#rescheduler-guaranteed-scheduling-of-critical-add-ons)

### Viewing cluster roles and bindings {#viewing-cluster-roles_post-install-preparing-for-users}

You can view cluster roles and bindings by using the `oc` CLI to determine the permissions associated with roles and identify the users, groups, and service accounts assigned to them.

You can use the `oc` CLI to view cluster roles and bindings by using the `oc describe` command.

**Prerequisites**

- Install the `oc` CLI.
- Obtain permission to view the cluster roles and bindings.

Users with the `cluster-admin` default cluster role bound cluster-wide can perform any action on any resource, including viewing cluster roles and bindings.

**Procedure**

1. To view the cluster roles and their associated rule sets:
   ```terminal
   $ oc describe clusterrole.rbac
   ```

   ```terminal title="Example output"
   Name:         admin
   Labels:       kubernetes.io/bootstrapping=rbac-defaults
   Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
   PolicyRule:
     Resources                                                  Non-Resource URLs  Resource Names  Verbs
     ---------                                                  -----------------  --------------  -----
     .packages.apps.redhat.com                                  []                 []              [* create update patch delete get list watch]
     imagestreams                                               []                 []              [create delete deletecollection get list patch update watch create get list watch]
     imagestreams.image.openshift.io                            []                 []              [create delete deletecollection get list patch update watch create get list watch]
     secrets                                                    []                 []              [create delete deletecollection get list patch update watch get list watch create delete deletecollection patch update]
     buildconfigs/webhooks                                      []                 []              [create delete deletecollection get list patch update watch get list watch]
     buildconfigs                                               []                 []              [create delete deletecollection get list patch update watch get list watch]
     buildlogs                                                  []                 []              [create delete deletecollection get list patch update watch get list watch]
     deploymentconfigs/scale                                    []                 []              [create delete deletecollection get list patch update watch get list watch]
     deploymentconfigs                                          []                 []              [create delete deletecollection get list patch update watch get list watch]
     imagestreamimages                                          []                 []              [create delete deletecollection get list patch update watch get list watch]
     imagestreammappings                                        []                 []              [create delete deletecollection get list patch update watch get list watch]
     imagestreamtags                                            []                 []              [create delete deletecollection get list patch update watch get list watch]
     processedtemplates                                         []                 []              [create delete deletecollection get list patch update watch get list watch]
     routes                                                     []                 []              [create delete deletecollection get list patch update watch get list watch]
     templateconfigs                                            []                 []              [create delete deletecollection get list patch update watch get list watch]
     templateinstances                                          []                 []              [create delete deletecollection get list patch update watch get list watch]
     templates                                                  []                 []              [create delete deletecollection get list patch update watch get list watch]
     deploymentconfigs.apps.openshift.io/scale                  []                 []              [create delete deletecollection get list patch update watch get list watch]
     deploymentconfigs.apps.openshift.io                        []                 []              [create delete deletecollection get list patch update watch get list watch]
     buildconfigs.build.openshift.io/webhooks                   []                 []              [create delete deletecollection get list patch update watch get list watch]
     buildconfigs.build.openshift.io                            []                 []              [create delete deletecollection get list patch update watch get list watch]
     buildlogs.build.openshift.io                               []                 []              [create delete deletecollection get list patch update watch get list watch]
     imagestreamimages.image.openshift.io                       []                 []              [create delete deletecollection get list patch update watch get list watch]
     imagestreammappings.image.openshift.io                     []                 []              [create delete deletecollection get list patch update watch get list watch]
     imagestreamtags.image.openshift.io                         []                 []              [create delete deletecollection get list patch update watch get list watch]
     routes.route.openshift.io                                  []                 []              [create delete deletecollection get list patch update watch get list watch]
     processedtemplates.template.openshift.io                   []                 []              [create delete deletecollection get list patch update watch get list watch]
     templateconfigs.template.openshift.io                      []                 []              [create delete deletecollection get list patch update watch get list watch]
     templateinstances.template.openshift.io                    []                 []              [create delete deletecollection get list patch update watch get list watch]
     templates.template.openshift.io                            []                 []              [create delete deletecollection get list patch update watch get list watch]
     serviceaccounts                                            []                 []              [create delete deletecollection get list patch update watch impersonate create delete deletecollection patch update get list watch]
     imagestreams/secrets                                       []                 []              [create delete deletecollection get list patch update watch]
     rolebindings                                               []                 []              [create delete deletecollection get list patch update watch]
     roles                                                      []                 []              [create delete deletecollection get list patch update watch]
     rolebindings.authorization.openshift.io                    []                 []              [create delete deletecollection get list patch update watch]
     roles.authorization.openshift.io                           []                 []              [create delete deletecollection get list patch update watch]
     imagestreams.image.openshift.io/secrets                    []                 []              [create delete deletecollection get list patch update watch]
     rolebindings.rbac.authorization.k8s.io                     []                 []              [create delete deletecollection get list patch update watch]
     roles.rbac.authorization.k8s.io                            []                 []              [create delete deletecollection get list patch update watch]
     networkpolicies.extensions                                 []                 []              [create delete deletecollection patch update create delete deletecollection get list patch update watch get list watch]
     networkpolicies.networking.k8s.io                          []                 []              [create delete deletecollection patch update create delete deletecollection get list patch update watch get list watch]
     configmaps                                                 []                 []              [create delete deletecollection patch update get list watch]
     endpoints                                                  []                 []              [create delete deletecollection patch update get list watch]
     persistentvolumeclaims                                     []                 []              [create delete deletecollection patch update get list watch]
     pods                                                       []                 []              [create delete deletecollection patch update get list watch]
     replicationcontrollers/scale                               []                 []              [create delete deletecollection patch update get list watch]
     replicationcontrollers                                     []                 []              [create delete deletecollection patch update get list watch]
     services                                                   []                 []              [create delete deletecollection patch update get list watch]
     daemonsets.apps                                            []                 []              [create delete deletecollection patch update get list watch]
     deployments.apps/scale                                     []                 []              [create delete deletecollection patch update get list watch]
     deployments.apps                                           []                 []              [create delete deletecollection patch update get list watch]
     replicasets.apps/scale                                     []                 []              [create delete deletecollection patch update get list watch]
     replicasets.apps                                           []                 []              [create delete deletecollection patch update get list watch]
     statefulsets.apps/scale                                    []                 []              [create delete deletecollection patch update get list watch]
     statefulsets.apps                                          []                 []              [create delete deletecollection patch update get list watch]
     horizontalpodautoscalers.autoscaling                       []                 []              [create delete deletecollection patch update get list watch]
     cronjobs.batch                                             []                 []              [create delete deletecollection patch update get list watch]
     jobs.batch                                                 []                 []              [create delete deletecollection patch update get list watch]
     daemonsets.extensions                                      []                 []              [create delete deletecollection patch update get list watch]
     deployments.extensions/scale                               []                 []              [create delete deletecollection patch update get list watch]
     deployments.extensions                                     []                 []              [create delete deletecollection patch update get list watch]
     ingresses.extensions                                       []                 []              [create delete deletecollection patch update get list watch]
     replicasets.extensions/scale                               []                 []              [create delete deletecollection patch update get list watch]
     replicasets.extensions                                     []                 []              [create delete deletecollection patch update get list watch]
     replicationcontrollers.extensions/scale                    []                 []              [create delete deletecollection patch update get list watch]
     poddisruptionbudgets.policy                                []                 []              [create delete deletecollection patch update get list watch]
     deployments.apps/rollback                                  []                 []              [create delete deletecollection patch update]
     deployments.extensions/rollback                            []                 []              [create delete deletecollection patch update]
     catalogsources.operators.coreos.com                        []                 []              [create update patch delete get list watch]
     clusterserviceversions.operators.coreos.com                []                 []              [create update patch delete get list watch]
     installplans.operators.coreos.com                          []                 []              [create update patch delete get list watch]
     packagemanifests.operators.coreos.com                      []                 []              [create update patch delete get list watch]
     subscriptions.operators.coreos.com                         []                 []              [create update patch delete get list watch]
     buildconfigs/instantiate                                   []                 []              [create]
     buildconfigs/instantiatebinary                             []                 []              [create]
     builds/clone                                               []                 []              [create]
     deploymentconfigrollbacks                                  []                 []              [create]
     deploymentconfigs/instantiate                              []                 []              [create]
     deploymentconfigs/rollback                                 []                 []              [create]
     imagestreamimports                                         []                 []              [create]
     localresourceaccessreviews                                 []                 []              [create]
     localsubjectaccessreviews                                  []                 []              [create]
     podsecuritypolicyreviews                                   []                 []              [create]
     podsecuritypolicyselfsubjectreviews                        []                 []              [create]
     podsecuritypolicysubjectreviews                            []                 []              [create]
     resourceaccessreviews                                      []                 []              [create]
     routes/custom-host                                         []                 []              [create]
     subjectaccessreviews                                       []                 []              [create]
     subjectrulesreviews                                        []                 []              [create]
     deploymentconfigrollbacks.apps.openshift.io                []                 []              [create]
     deploymentconfigs.apps.openshift.io/instantiate            []                 []              [create]
     deploymentconfigs.apps.openshift.io/rollback               []                 []              [create]
     localsubjectaccessreviews.authorization.k8s.io             []                 []              [create]
     localresourceaccessreviews.authorization.openshift.io      []                 []              [create]
     localsubjectaccessreviews.authorization.openshift.io       []                 []              [create]
     resourceaccessreviews.authorization.openshift.io           []                 []              [create]
     subjectaccessreviews.authorization.openshift.io            []                 []              [create]
     subjectrulesreviews.authorization.openshift.io             []                 []              [create]
     buildconfigs.build.openshift.io/instantiate                []                 []              [create]
     buildconfigs.build.openshift.io/instantiatebinary          []                 []              [create]
     builds.build.openshift.io/clone                            []                 []              [create]
     imagestreamimports.image.openshift.io                      []                 []              [create]
     routes.route.openshift.io/custom-host                      []                 []              [create]
     podsecuritypolicyreviews.security.openshift.io             []                 []              [create]
     podsecuritypolicyselfsubjectreviews.security.openshift.io  []                 []              [create]
     podsecuritypolicysubjectreviews.security.openshift.io      []                 []              [create]
     jenkins.build.openshift.io                                 []                 []              [edit view view admin edit view]
     builds                                                     []                 []              [get create delete deletecollection get list patch update watch get list watch]
     builds.build.openshift.io                                  []                 []              [get create delete deletecollection get list patch update watch get list watch]
     projects                                                   []                 []              [get delete get delete get patch update]
     projects.project.openshift.io                              []                 []              [get delete get delete get patch update]
     namespaces                                                 []                 []              [get get list watch]
     pods/attach                                                []                 []              [get list watch create delete deletecollection patch update]
     pods/exec                                                  []                 []              [get list watch create delete deletecollection patch update]
     pods/portforward                                           []                 []              [get list watch create delete deletecollection patch update]
     pods/proxy                                                 []                 []              [get list watch create delete deletecollection patch update]
     services/proxy                                             []                 []              [get list watch create delete deletecollection patch update]
     routes/status                                              []                 []              [get list watch update]
     routes.route.openshift.io/status                           []                 []              [get list watch update]
     appliedclusterresourcequotas                               []                 []              [get list watch]
     bindings                                                   []                 []              [get list watch]
     builds/log                                                 []                 []              [get list watch]
     deploymentconfigs/log                                      []                 []              [get list watch]
     deploymentconfigs/status                                   []                 []              [get list watch]
     events                                                     []                 []              [get list watch]
     imagestreams/status                                        []                 []              [get list watch]
     limitranges                                                []                 []              [get list watch]
     namespaces/status                                          []                 []              [get list watch]
     pods/log                                                   []                 []              [get list watch]
     pods/status                                                []                 []              [get list watch]
     replicationcontrollers/status                              []                 []              [get list watch]
     resourcequotas/status                                      []                 []              [get list watch]
     resourcequotas                                             []                 []              [get list watch]
     resourcequotausages                                        []                 []              [get list watch]
     rolebindingrestrictions                                    []                 []              [get list watch]
     deploymentconfigs.apps.openshift.io/log                    []                 []              [get list watch]
     deploymentconfigs.apps.openshift.io/status                 []                 []              [get list watch]
     controllerrevisions.apps                                   []                 []              [get list watch]
     rolebindingrestrictions.authorization.openshift.io         []                 []              [get list watch]
     builds.build.openshift.io/log                              []                 []              [get list watch]
     imagestreams.image.openshift.io/status                     []                 []              [get list watch]
     appliedclusterresourcequotas.quota.openshift.io            []                 []              [get list watch]
     imagestreams/layers                                        []                 []              [get update get]
     imagestreams.image.openshift.io/layers                     []                 []              [get update get]
     builds/details                                             []                 []              [update]
     builds.build.openshift.io/details                          []                 []              [update]

   Name:         basic-user
   Labels:       <none>
   Annotations:  openshift.io/description: A user that can get basic information about projects.
   	              rbac.authorization.kubernetes.io/autoupdate: true
   PolicyRule:
   	Resources                                           Non-Resource URLs  Resource Names  Verbs
   	  ---------                                           -----------------  --------------  -----
   	  selfsubjectrulesreviews                             []                 []              [create]
   	  selfsubjectaccessreviews.authorization.k8s.io       []                 []              [create]
   	  selfsubjectrulesreviews.authorization.openshift.io  []                 []              [create]
   	  clusterroles.rbac.authorization.k8s.io              []                 []              [get list watch]
   	  clusterroles                                        []                 []              [get list]
   	  clusterroles.authorization.openshift.io             []                 []              [get list]
   	  storageclasses.storage.k8s.io                       []                 []              [get list]
   	  users                                               []                 [~]             [get]
   	  users.user.openshift.io                             []                 [~]             [get]
   	  projects                                            []                 []              [list watch]
   	  projects.project.openshift.io                       []                 []              [list watch]
   	  projectrequests                                     []                 []              [list]
   	  projectrequests.project.openshift.io                []                 []              [list]

   Name:         cluster-admin
   Labels:       kubernetes.io/bootstrapping=rbac-defaults
   Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
   PolicyRule:
   Resources  Non-Resource URLs  Resource Names  Verbs
   ---------  -----------------  --------------  -----
   *.*        []                 []              [*]
              [*]                []              [*]

   ...
   ```
2. To view the current set of cluster role bindings, which shows the users and groups that are bound to various roles:
   ```terminal
   $ oc describe clusterrolebinding.rbac
   ```

   ```terminal title="Example output"
   Name:         alertmanager-main
   Labels:       <none>
   Annotations:  <none>
   Role:
     Kind:  ClusterRole
     Name:  alertmanager-main
   Subjects:
     Kind            Name               Namespace
     ----            ----               ---------
     ServiceAccount  alertmanager-main  openshift-monitoring

   Name:         basic-users
   Labels:       <none>
   Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
   Role:
     Kind:  ClusterRole
     Name:  basic-user
   Subjects:
     Kind   Name                  Namespace
     ----   ----                  ---------
     Group  system:authenticated

   Name:         cloud-credential-operator-rolebinding
   Labels:       <none>
   Annotations:  <none>
   Role:
     Kind:  ClusterRole
     Name:  cloud-credential-operator-role
   Subjects:
     Kind            Name     Namespace
     ----            ----     ---------
     ServiceAccount  default  openshift-cloud-credential-operator

   Name:         cluster-admin
   Labels:       kubernetes.io/bootstrapping=rbac-defaults
   Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
   Role:
     Kind:  ClusterRole
     Name:  cluster-admin
   Subjects:
     Kind   Name            Namespace
     ----   ----            ---------
     Group  system:masters

   Name:         cluster-admins
   Labels:       <none>
   Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
   Role:
     Kind:  ClusterRole
     Name:  cluster-admin
   Subjects:
     Kind   Name                   Namespace
     ----   ----                   ---------
     Group  system:cluster-admins
     User   system:admin

   Name:         cluster-api-manager-rolebinding
   Labels:       <none>
   Annotations:  <none>
   Role:
     Kind:  ClusterRole
     Name:  cluster-api-manager-role
   Subjects:
     Kind            Name     Namespace
     ----            ----     ---------
     ServiceAccount  default  openshift-machine-api

   ...
   ```

### Viewing local roles and bindings {#viewing-local-roles_post-install-preparing-for-users}

You can view local role bindings by using the `oc` CLI to identify the users, groups, and service accounts that have roles within the current project or another project.

You can use the `oc` CLI to view local roles and bindings by using the `oc describe` command.

**Prerequisites**

- Install the `oc` CLI.
- Obtain permission to view the local roles and bindings:
  - Users with the `cluster-admin` default cluster role bound cluster-wide can perform any action on any resource, including viewing local roles and bindings.
  - Users with the `admin` default cluster role bound locally can view and manage roles and bindings in that project.

**Procedure**

1. To view the current set of local role bindings, which show the users and groups that are bound to various roles for the current project:
   ```terminal
   $ oc describe rolebinding.rbac
   ```
2. To view the local role bindings for a different project, add the `-n` flag to the command:
   ```terminal
   $ oc describe rolebinding.rbac -n joe-project
   ```

   ```terminal title="Example output"
   Name:         admin
   Labels:       <none>
   Annotations:  <none>
   Role:
     Kind:  ClusterRole
     Name:  admin
   Subjects:
     Kind  Name        Namespace
     ----  ----        ---------
     User  kube:admin

   Name:         system:deployers
   Labels:       <none>
   Annotations:  openshift.io/description:
                   Allows deploymentconfigs in this namespace to rollout pods in
                   this namespace.  It is auto-managed by a controller; remove
                   subjects to disa...
   Role:
     Kind:  ClusterRole
     Name:  system:deployer
   Subjects:
     Kind            Name      Namespace
     ----            ----      ---------
     ServiceAccount  deployer  joe-project

   Name:         system:image-builders
   Labels:       <none>
   Annotations:  openshift.io/description:
                   Allows builds in this namespace to push images to this
                   namespace.  It is auto-managed by a controller; remove subjects
                   to disable.
   Role:
     Kind:  ClusterRole
     Name:  system:image-builder
   Subjects:
     Kind            Name     Namespace
     ----            ----     ---------
     ServiceAccount  builder  joe-project

   Name:         system:image-pullers
   Labels:       <none>
   Annotations:  openshift.io/description:
                   Allows all pods in this namespace to pull images from this
                   namespace.  It is auto-managed by a controller; remove subjects
                   to disable.
   Role:
     Kind:  ClusterRole
     Name:  system:image-puller
   Subjects:
     Kind   Name                                Namespace
     ----   ----                                ---------
     Group  system:serviceaccounts:joe-project
   ```

### Adding roles to users {#adding-roles_post-install-preparing-for-users}

To grant a user access within a project, you can bind an appropriate role to the user and verify the resulting role binding.

You can use the `oc adm` administrator CLI to manage the roles and bindings.

Binding, or adding, a role to users or groups gives the user or group the access that is granted by the role. You can add and remove roles to and from users and groups using `oc adm policy` commands.

You can bind any of the default cluster roles to local users or groups in your project.

**Procedure**

1. Add a role to a user in a specific project:
   ```terminal
   $ oc adm policy add-role-to-user <role> <user> -n <project>
   ```

   For example, you can add the `admin` role to the `alice` user in `joe` project by running:

   ```terminal
   $ oc adm policy add-role-to-user admin alice -n joe
   ```

   :::tip

   You can alternatively apply the following YAML to add the role to the user:

   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: admin-0
     namespace: joe
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: ClusterRole
     name: admin
   subjects:
   - apiGroup: rbac.authorization.k8s.io
     kind: User
     name: alice
   ```

   :::
2. View the local role bindings and verify the addition in the output:
   ```terminal
   $ oc describe rolebinding.rbac -n <project>
   ```

   For example, to view the local role bindings for the `joe` project:

   ```terminal
   $ oc describe rolebinding.rbac -n joe
   ```

   ```terminal title="Example output"

   Name:         admin
   Labels:       <none>
   Annotations:  <none>
   Role:
     Kind:  ClusterRole
     Name:  admin
   Subjects:
     Kind  Name        Namespace
     ----  ----        ---------
     User  kube:admin

   Name:         admin-0
   Labels:       <none>
   Annotations:  <none>
   Role:
     Kind:  ClusterRole
     Name:  admin
   Subjects:
     Kind  Name   Namespace
     ----  ----   ---------
     User  alice

   Name:         system:deployers
   Labels:       <none>
   Annotations:  openshift.io/description:
                   Allows deploymentconfigs in this namespace to rollout pods in
                   this namespace.  It is auto-managed by a controller; remove
                   subjects to disa...
   Role:
     Kind:  ClusterRole
     Name:  system:deployer
   Subjects:
     Kind            Name      Namespace
     ----            ----      ---------
     ServiceAccount  deployer  joe

   Name:         system:image-builders
   Labels:       <none>
   Annotations:  openshift.io/description:
                   Allows builds in this namespace to push images to this
                   namespace.  It is auto-managed by a controller; remove subjects
                   to disable.
   Role:
     Kind:  ClusterRole
     Name:  system:image-builder
   Subjects:
     Kind            Name     Namespace
     ----            ----     ---------
     ServiceAccount  builder  joe

   Name:         system:image-pullers
   Labels:       <none>
   Annotations:  openshift.io/description:
                   Allows all pods in this namespace to pull images from this
                   namespace.  It is auto-managed by a controller; remove subjects
                   to disable.
   Role:
     Kind:  ClusterRole
     Name:  system:image-puller
   Subjects:
     Kind   Name                                Namespace
     ----   ----                                ---------
     Group  system:serviceaccounts:joe
   ```

   The `alice` user has been added to the `admins` `RoleBinding`.

### Creating a local role {#creating-local-role_post-install-preparing-for-users}

You can create a local role and bind it to a user to define custom permissions within a project.

**Procedure**

1. To create a local role for a project, run the following command:
   ```terminal
   $ oc create role <name> --verb=<verb> --resource=<resource> -n <project>
   ```

   In this command, specify:

   - `<name>`, the local role’s name
   - `<verb>`, a comma-separated list of the verbs to apply to the role
   - `<resource>`, the resources that the role applies to
   - `<project>`, the project name

   For example, to create a local role that allows a user to view pods in the `blue` project, run the following command:

   ```terminal
   $ oc create role podview --verb=get --resource=pod -n blue
   ```
2. To bind the new role to a user, run the following command:
   ```terminal
   $ oc adm policy add-role-to-user podview user2 --role-namespace=blue -n blue
   ```

### Creating a cluster role {#creating-cluster-role_post-install-preparing-for-users}

To define custom cluster-wide permissions, you can create a cluster role that specifies the verbs and resources users can access.

**Procedure**

- To create a cluster role, run the following command:
  ```terminal
  $ oc create clusterrole <name> --verb=<verb> --resource=<resource>
  ```

  In this command, specify:

  - `<name>`, the local role’s name
  - `<verb>`, a comma-separated list of the verbs to apply to the role
  - `<resource>`, the resources that the role applies to

  For example, to create a cluster role that allows a user to view pods, run the following command:

  ```terminal
  $ oc create clusterrole podviewonly --verb=get --resource=pod
  ```

### Local role binding commands {#local-role-binding-commands_post-install-preparing-for-users}

You can use local role binding commands to review, grant, or remove user and group permissions within the current or a specified project.

When you manage a user or group’s associated roles for local role bindings using the following operations, a project may be specified with the `-n` flag. If it is not specified, then the current project is used.

You can use the following commands for local RBAC management.

**Local role binding operations**

| Command | Description |
| --- | --- |
| `$ oc adm policy who-can _<verb>_ _<resource>_` | Indicates which users can perform an action on a resource. |
| `$ oc adm policy add-role-to-user _<role>_ _<username>_` | Binds a specified role to specified users in the current project. |
| `$ oc adm policy remove-role-from-user _<role>_ _<username>_` | Removes a given role from specified users in the current project. |
| `$ oc adm policy remove-user _<username>_` | Removes specified users and all of their roles in the current project. |
| `$ oc adm policy add-role-to-group _<role>_ _<groupname>_` | Binds a given role to specified groups in the current project. |
| `$ oc adm policy remove-role-from-group _<role>_ _<groupname>_` | Removes a given role from specified groups in the current project. |
| `$ oc adm policy remove-group _<groupname>_` | Removes specified groups and all of their roles in the current project. |

### Cluster role binding commands {#cluster-role-binding-commands_post-install-preparing-for-users}

You can use cluster role binding commands to grant or remove roles for users and groups across all projects in the cluster.

You can also manage cluster role bindings using the following operations. The `-n` flag is not used for these operations because cluster role bindings use non-namespaced resources.

**Cluster role binding operations**

| Command | Description |
| --- | --- |
| `$ oc adm policy add-cluster-role-to-user _<role>_ _<username>_` | Binds a given role to specified users for all projects in the cluster. |
| `$ oc adm policy remove-cluster-role-from-user _<role>_ _<username>_` | Removes a given role from specified users for all projects in the cluster. |
| `$ oc adm policy add-cluster-role-to-group _<role>_ _<groupname>_` | Binds a given role to specified groups for all projects in the cluster. |
| `$ oc adm policy remove-cluster-role-from-group _<role>_ _<groupname>_` | Removes a given role from specified groups for all projects in the cluster. |

### Creating a cluster admin {#creating-cluster-admin_post-install-preparing-for-users}

To grant a user full administrative access to the cluster, you can bind the `cluster-admin` cluster role to that user.

The `cluster-admin` role is required to perform administrator level tasks on the OpenShift Container Platform cluster, such as modifying cluster resources.

**Prerequisites**

- You must have created a user to define as the cluster admin.

**Procedure**

- Define the user as a cluster admin:
  ```terminal
  $ oc adm policy add-cluster-role-to-user cluster-admin <user>
  ```

### Cluster role bindings for unauthenticated groups {#unauthenticated-users-cluster-role-bindings-concept_post-install-preparing-for-users}

Unauthenticated groups do not have default access to cluster roles. As a cluster administrator, you can grant limited unauthenticated access when required, while ensuring that the change complies with organizational security standards.

:::note

Before OpenShift Container Platform 4.17, unauthenticated groups were allowed access to some cluster roles. Clusters updated from versions before OpenShift Container Platform 4.17 retain this access for unauthenticated groups.

:::

For security reasons OpenShift Container Platform 4.22 does not allow unauthenticated groups to have default access to cluster roles.

There are use cases where it might be necessary to add `system:unauthenticated` to a cluster role.

Cluster administrators can add unauthenticated users to the following cluster roles:

- `system:scope-impersonation`
- `system:webhook`
- `system:oauth-token-deleter`
- `self-access-reviewer`

:::warning

Always verify compliance with your organization’s security standards when modifying unauthenticated access.

:::

### Adding unauthenticated groups to cluster roles {#unauthenticated-users-cluster-role-bindings_post-install-preparing-for-users}

Grant unauthenticated users access to specific cluster roles to enable features that require cluster access without authentication, such as external webhooks or automated token management.

You can add unauthenticated users to the following cluster roles:

- `system:scope-impersonation`
- `system:webhook`
- `system:oauth-token-deleter`
- `self-access-reviewer`

:::warning

Always verify compliance with your organization’s security standards when modifying unauthenticated access.

:::

**Prerequisites**

- You have access to the cluster as a user with the `cluster-admin` role.
- You have installed the OpenShift CLI (`oc`).

**Procedure**

1. Create a YAML file named `add-<cluster_role>-unauth.yaml` and add the following content:
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRoleBinding
   metadata:
    annotations:
      rbac.authorization.kubernetes.io/autoupdate: "true"
    name: <cluster_role>access-unauthenticated
   roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: <cluster_role>
   subjects:
    - apiGroup: rbac.authorization.k8s.io
      kind: Group
      name: system:unauthenticated
   ```
2. Apply the configuration by running the following command:
   ```terminal
   $ oc apply -f add-<cluster_role>.yaml
   ```

## The kubeadmin user {#understanding-kubeadmin_post-install-preparing-for-users}

OpenShift Container Platform creates a cluster administrator, `kubeadmin`, after the installation process completes. This user has the `cluster-admin` role automatically applied and is treated as the root user for the cluster.

The password is dynamically generated and unique to your OpenShift Container Platform environment. After the installation completes, the password is provided in the installation program’s output. For example:

```terminal
INFO Install complete!
INFO Run 'export KUBECONFIG=<your working directory>/auth/kubeconfig' to manage the cluster with 'oc', the OpenShift CLI.
INFO The cluster is ready when 'oc login -u kubeadmin -p <provided>' succeeds (wait a few minutes).
INFO Access the OpenShift web-console here: https://console-openshift-console.apps.demo1.openshift4-beta-abcorp.com
INFO Login to the console with user: kubeadmin, password: <provided>
```

### Removing the kubeadmin user {#removing-kubeadmin_post-install-preparing-for-users}

After you define an identity provider and create a new `cluster-admin` user, you can remove the `kubeadmin` to improve cluster security.

:::warning

If you follow this procedure before another user is a `cluster-admin`, then OpenShift Container Platform must be reinstalled. It is not possible to undo this command.

:::

**Prerequisites**

- You must have configured at least one identity provider.
- You must have added the `cluster-admin` role to a user.
- You must be logged in as an administrator.

**Procedure**

- Remove the `kubeadmin` secrets:
  ```terminal
  $ oc delete secrets kubeadmin -n kube-system
  ```

## Populating the software catalog from mirrored Operator catalogs {#post-install-mirrored-catalogs_post-install-preparing-for-users}

If you mirrored Operator catalogs for use with disconnected clusters, you can populate the software catalog with the Operators from your mirrored catalogs. You can use the generated manifests from the mirroring process to create the required `ImageContentSourcePolicy` and `CatalogSource` objects.

**Prerequisites**

- You mirrored Operator catalogs for use with disconnected clusters.

**Additional resources**

- [Mirroring Operator catalogs for use with disconnected clusters](/docs/disconnected/installing-mirroring-installation-images#olm-mirror-catalog_installing-mirroring-installation-images)

### Creating the ImageContentSourcePolicy object {#olm-mirror-catalog-icsp_post-install-preparing-for-users}

To make mirrored Operator images available to a disconnected cluster, you can create an `ImageContentSourcePolicy` object that redirects image references to your mirror registry.

After mirroring Operator catalog content to your mirror registry, create the required `ImageContentSourcePolicy` (ICSP) object. The ICSP object configures nodes to translate between the image references stored in Operator manifests and the mirrored registry.

**Procedure**

- On a host with access to the disconnected cluster, create the ICSP by running the following command to specify the `imageContentSourcePolicy.yaml` file in your manifests directory:
  ```terminal
  $ oc create -f <path/to/manifests/dir>/imageContentSourcePolicy.yaml
  ```

  where `<path/to/manifests/dir>` is the path to the manifests directory for your mirrored content.

  You can now create a `CatalogSource` object to reference your mirrored index image and Operator content.

### Adding a catalog source to a cluster {#olm-creating-catalog-from-index_post-install-preparing-for-users}

To make Operators from a custom index image available for installation, create a catalog source that adds the catalog content to your cluster.

Cluster administrators

can create a `CatalogSource` object that references an index image. The software catalog uses catalog sources to populate the user interface.

:::tip

Alternatively, you can use the web console to manage catalog sources. From the **Administration** → **Cluster Settings** → **Configuration** → **OperatorHub** page, click the **Sources** tab, where you can create, update, delete, disable, and enable individual sources.

:::

**Prerequisites**

- You built and pushed an index image to a registry.
- You have access to the cluster as a user with the `cluster-admin` role.

**Procedure**

1. Create a `CatalogSource` object that references your index image. If you used the `oc adm catalog mirror` command to mirror your catalog to a target registry, you can use the generated `catalogSource.yaml` file in your manifests directory as a starting point.
   1. Modify the following to your specifications and save it as a `catalogSource.yaml` file:
      ```yaml
      apiVersion: operators.coreos.com/v1alpha1
      kind: CatalogSource
      metadata:
        name: my-operator-catalog
        namespace: openshift-marketplace
      spec:
        sourceType: grpc
        grpcPodConfig:
          securityContextConfig: <security_mode>
        image: <registry>/<namespace>/redhat-operator-index:v4.22
        displayName: My Operator Catalog
        publisher: <publisher_name>
        updateStrategy:
          registryPoll:
            interval: 30m
      ```

      where:

      **`metadata.name`**

      Specifies the value for the `metadata.name` parameter. If you mirrored content to local files before uploading to a registry, remove any backslash (`/`) characters from the `metadata.name` field to avoid an "invalid resource name" error when you create the object.

      **`metadata.namespace`**

      Specifies the value for the `metadata.namespace` parameter. If you want the catalog source to be available globally to users in all namespaces, specify the `openshift-marketplace` namespace. Otherwise, you can specify a different namespace for the catalog to be scoped and available only for that namespace.

      **`spec.grpcPodConfig.securityContextConfig`**

      Specifies the value of `legacy` or `restricted`. If the field is not set, the default value is `legacy`. In a future OpenShift Container Platform release, it is planned that the default value will be `restricted`.

      :::note

      If your catalog cannot run with `restricted` permissions, it is recommended that you manually set this field to `legacy`.

      :::

      **`spec.image`**

      Specifies your index image. If you specify a tag after the image name, for example `:v4.22`, the catalog source pod uses an image pull policy of `Always`, meaning the pod always pulls the image before starting the container. If you specify a digest, for example `@sha256:<id>`, the image pull policy is `IfNotPresent`, meaning the pod pulls the image only if it does not already exist on the node.

      **`spec.publisher`**

      Specifies your name or an organization name publishing the catalog.

      **`spec.updateStrategy.registryPoll`**

      Specifies the value for the `spec.updateStrategy.registryPoll` parameter. The catalog sources can automatically check for new versions to keep up to date.
   2. Use the file to create the `CatalogSource` object:
      ```terminal
      $ oc apply -f catalogSource.yaml
      ```
2. Verify the following resources are created successfully.
   1. Check the pods:
      ```terminal
      $ oc get pods -n openshift-marketplace
      ```

      The following is example output:

      ```terminal
      NAME                                    READY   STATUS    RESTARTS  AGE
      my-operator-catalog-6njx6               1/1     Running   0         28s
      marketplace-operator-d9f549946-96sgr    1/1     Running   0         26h
      ```
   2. Check the catalog source:
      ```terminal
      $ oc get catalogsource -n openshift-marketplace
      ```

      The following is example output:

      ```terminal
      NAME                  DISPLAY               TYPE PUBLISHER  AGE
      my-operator-catalog   My Operator Catalog   grpc            5s
      ```
   3. Check the package manifest:
      ```terminal
      $ oc get packagemanifest -n openshift-marketplace
      ```

      The following is example output:

      ```terminal
      NAME                          CATALOG               AGE
      jaeger-product                My Operator Catalog   93s
      ```

      You can now install the Operators from the **Software Catalog** page on your OpenShift Container Platform web console.

**Additional resources**

- [Accessing images for Operators from private registries](/docs/operators/admin/olm-managing-custom-catalogs#olm-accessing-images-private-registries_olm-managing-custom-catalogs)
- [Image template for custom catalog sources](/docs/operators/understanding/olm/olm-understanding-olm#olm-catalogsource-image-template_olm-understanding-olm)
- [Image pull policy](/docs/openshift_images/managing_images/image-pull-policy#image-pull-policy)

## About Operator installation from the software catalog {#olm-installing-operators-from-software-catalog_post-install-preparing-for-users}

The software catalog in OpenShift Container Platform is the interface for discovering Operators that Operator Lifecycle Manager (OLM) installs and manages on your cluster. You can choose installation settings such as install mode, namespace, and approval strategy during subscription.

As a cluster administrator, you can install an Operator from the software catalog by using the OpenShift Container Platform web console or CLI. Subscribing an Operator to one or more namespaces makes the Operator available to developers on your cluster.

During installation, you must determine the following initial settings for the Operator:

Installation Mode
: Choose **All namespaces on the cluster (default)** to have the Operator installed on all namespaces or choose individual namespaces, if available, to only install the Operator on selected namespaces. This example chooses **All namespaces...** to make the Operator available to all users and projects.

Update Channel
: If an Operator is available through multiple channels, you can choose which channel you want to subscribe to. For example, to deploy from the **stable** channel, if available, select it from the list.

Approval Strategy
: You can choose automatic or manual updates.

  If you choose automatic updates for an installed Operator, when a new version of that Operator is available in the selected channel, Operator Lifecycle Manager (OLM) automatically upgrades the running instance of your Operator without human intervention.

  If you select manual updates, when a newer version of an Operator is available, OLM creates an update request. As a cluster administrator, you must then manually approve that update request to have the Operator updated to the new version.

### Installing from the software catalog by using the web console {#olm-installing-from-software-catalog-using-web-console_post-install-preparing-for-users}

To install and subscribe to an Operator from the software catalog, you can use the OpenShift Container Platform web console. The console guides you through selecting an install mode, namespace, and approval strategy.

**Prerequisites**

- Access to an OpenShift Container Platform cluster using an account with `cluster-admin` permissions.

**Procedure**

1. Navigate in the web console to the **Ecosystem** → **Software Catalog** page.
2. Scroll or type a keyword into the **Filter by keyword** box to find the Operator you want. For example, type `jaeger` to find the Jaeger Operator.
   You can also filter options by **Infrastructure Features**. For example, select **Disconnected** if you want to see Operators that work in disconnected environments, also known as restricted network environments.
3. Select the Operator to display additional information.
   :::note

   Choosing a Community Operator warns that Red Hat does not certify Community Operators; you must acknowledge the warning before continuing.

   :::
4. Read the information about the Operator and click **Install**.
5. On the **Install Operator** page, configure your Operator installation:
   1. If you want to install a specific version of an Operator, select an **Update channel** and **Version** from the lists. You can browse the various versions of an Operator across any channels it might have, view the metadata for that channel and version, and select the exact version you want to install.
      :::note

      The version selection defaults to the latest version for the channel selected. If the latest version for the channel is selected, the **Automatic** approval strategy is enabled by default. Otherwise, **Manual** approval is required when not installing the latest version for the selected channel.

      Installing an Operator with **Manual** approval causes all Operators installed within the namespace to function with the **Manual** approval strategy and all Operators are updated together. If you want to update Operators independently, install Operators into separate namespaces.

      :::
   2. Confirm the installation mode for the Operator:
      - **All namespaces on the cluster (default)** installs the Operator in the default `openshift-operators` namespace to watch and be made available to all namespaces in the cluster. This option is not always available.
      - **A specific namespace on the cluster** allows you to choose a specific, single namespace in which to install the Operator. The Operator will only watch and be made available for use in this single namespace.
   3. For clusters on cloud providers with token authentication enabled:
      - If the cluster uses AWS Security Token Service (**STS Mode** in the web console), enter the Amazon Resource Name (ARN) of the AWS IAM role of your service account in the **role ARN** field. To create the role’s ARN, follow the procedure described in [Preparing AWS account](https://docs.redhat.com/en/documentation/red_hat_openshift_service_on_aws/4/html/tutorials/cloud-experts-deploy-api-data-protection#prepare-aws-account_cloud-experts-deploy-api-data-protection).
      - If the cluster uses Microsoft Entra Workload ID (**Workload Identity / Federated Identity Mode** in the web console), add the client ID, tenant ID, and subscription ID in the appropriate fields.
      - If the cluster uses Google Cloud Platform Workload Identity (**GCP Workload Identity / Federated Identity Mode** in the web console), add the project number, pool ID, provider ID, and service account email in the appropriate fields.
6. For **Update approval**, select either the **Automatic** or **Manual** approval strategy.
   :::warning

   If the web console shows that the cluster uses AWS STS, Microsoft Entra Workload ID, or GCP Workload Identity, you must set **Update approval** to **Manual**.

   Subscriptions with automatic approvals for updates are not recommended because there might be permission changes to make before updating. Subscriptions with manual approvals for updates ensure that administrators have the opportunity to verify the permissions of the later version, take any necessary steps, and then update.

   :::

   1. Click **Install** to make the Operator available to the selected namespaces on this OpenShift Container Platform cluster:
7. If you selected a **Manual** approval strategy, the upgrade status of the subscription remains **Upgrading** until you review and approve the install plan.
   After approving on the **Install Plan** page, the subscription upgrade status moves to **Up to date**.
8. If you selected an **Automatic** approval strategy, the upgrade status should resolve to **Up to date** without intervention.

**Verification**

- After the upgrade status of the subscription is **Up to date**, select **Ecosystem** → **Installed Operators** to verify that the cluster service version (CSV) of the installed Operator eventually shows up. The **Status** should eventually resolve to **Succeeded** in the relevant namespace.
  :::note

  For the **All namespaces...** installation mode, the status resolves to **Succeeded** in the `openshift-operators` namespace, but the status is **Copied** if you check in other namespaces.

  :::

  If it does not:

  - Check the logs in any pods in the `openshift-operators` project (or other relevant namespace if **A specific namespace...** installation mode was selected) on the **Workloads** → **Pods** page that are reporting issues to troubleshoot further.
- When the Operator is installed, the metadata indicates which channel and version are installed.
  :::note

  The **Channel** and **Version** dropdown menus are still available for viewing other version metadata in this catalog context.

  :::

### Installing from the software catalog by using the CLI {#olm-installing-operator-from-software-catalog-using-cli_post-install-preparing-for-users}

To install an Operator from the software catalog without using the web console, you can create or update a `Subscription` object by using the `oc` command in OpenShift Container Platform.

For `SingleNamespace` install mode, you must also ensure an appropriate Operator group exists in the related namespace. An Operator group, defined by an `OperatorGroup` object, selects target namespaces in which to generate required RBAC access for all Operators in the same namespace as the Operator group.

:::tip

In most cases, the web console method of this procedure is preferred because it automates tasks in the background, such as handling the creation of `OperatorGroup` and `Subscription` objects automatically when choosing `SingleNamespace` mode.

:::

**Prerequisites**

- Access to your OpenShift Container Platform cluster using an account with `cluster-admin` permissions.
- You have installed the OpenShift CLI (`oc`).

**Procedure**

1. View the list of Operators available to the cluster from the software catalog:
   ```terminal
   $ oc get packagemanifests -n openshift-marketplace
   ```

   ```terminal title="Example output"
   NAME                               CATALOG               AGE
   3scale-operator                    Red Hat Operators     91m
   advanced-cluster-management        Red Hat Operators     91m
   amq7-cert-manager                  Red Hat Operators     91m
   # ...
   couchbase-enterprise-certified     Certified Operators   91m
   crunchy-postgres-operator          Certified Operators   91m
   mongodb-enterprise                 Certified Operators   91m
   # ...
   etcd                               Community Operators   91m
   jaeger                             Community Operators   91m
   kubefed                            Community Operators   91m
   # ...
   ```

   Note the catalog for your desired Operator.
2. Inspect your desired Operator to verify its supported install modes and available channels:
   ```terminal
   $ oc describe packagemanifests <operator_name> -n openshift-marketplace
   ```

   ```terminal title="Example output"
   # ...
   Kind:         PackageManifest
   # ...
         Install Modes:
           Supported:  true
           Type:       OwnNamespace
           Supported:  true
           Type:       SingleNamespace
           Supported:  false
           Type:       MultiNamespace
           Supported:  true
           Type:       AllNamespaces
   # ...
       Entries:
         Name:       example-operator.v3.7.11
         Version:    3.7.11
         Name:       example-operator.v3.7.10
         Version:    3.7.10
       Name:         stable-3.7
   # ...
      Entries:
         Name:         example-operator.v3.8.5
         Version:      3.8.5
         Name:         example-operator.v3.8.4
         Version:      3.8.4
       Name:           stable-3.8
     Default Channel:  stable-3.8
   ```

   In the example output, `Install Modes` indicates which install modes are supported; `Name` shows example channel names; and `Default Channel` is the channel selected by default if one is not specified.

   :::tip

   You can print an Operator’s version and channel information in YAML format by running the following command:

   ```terminal
   $ oc get packagemanifests <operator_name> -n <catalog_namespace> -o yaml
   ```

   :::
3. If more than one catalog is installed in a namespace, run the following command to look up the available versions and channels of an Operator from a specific catalog:
   ```terminal
   $ oc get packagemanifest \
      --selector=catalog=<catalogsource_name> \
      --field-selector metadata.name=<operator_name> \
      -n <catalog_namespace> -o yaml
   ```

   :::warning

   If you do not specify the Operator’s catalog, running the `oc get packagemanifest` and `oc describe packagemanifest` commands might return a package from an unexpected catalog if the following conditions are met:

   - Multiple catalogs are installed in the same namespace.
   - The catalogs contain the same Operators or Operators with the same name.

   :::
4. If the Operator you intend to install supports the `AllNamespaces` install mode, and you choose to use this mode, skip this step, because the `openshift-operators` namespace already has an appropriate Operator group in place by default, called `global-operators`.
   If the Operator you intend to install supports the `SingleNamespace` install mode, and you choose to use this mode, you must ensure an appropriate Operator group exists in the related namespace. If one does not exist, you can create create one by following these steps:

   :::warning

   You can only have one Operator group per namespace. For more information, see "Operator groups".

   :::

   1. Create an `OperatorGroup` object YAML file, for example `operatorgroup.yaml`, for `SingleNamespace` install mode:
      ```yaml title="Example OperatorGroup object for SingleNamespace install mode"
      apiVersion: operators.coreos.com/v1
      kind: OperatorGroup
      metadata:
        name: <operatorgroup_name>
        namespace: <namespace>
      spec:
        targetNamespaces:
        - <namespace>
      ```

      For `SingleNamespace` install mode, use the same `<namespace>` value for both the `metadata.namespace` and `spec.targetNamespaces` fields.
   2. Create the `OperatorGroup` object:
      ```terminal
      $ oc apply -f operatorgroup.yaml
      ```
5. Create a `Subscription` object to subscribe a namespace to an Operator:
   1. Create a YAML file for the `Subscription` object, for example `subscription.yaml`:
      :::note

      If you want to subscribe to a specific version of an Operator, set the `startingCSV` field to the desired version and set the `installPlanApproval` field to `Manual` to prevent the Operator from automatically upgrading if a later version exists in the catalog. For details, see the following "Example `Subscription` object with a specific starting Operator version".

      :::

      ```yaml title="Example Subscription object"
      apiVersion: operators.coreos.com/v1alpha1
      kind: Subscription
      metadata:
        name: <subscription_name>
        namespace: <namespace_per_install_mode>
      spec:
        channel: <channel_name>
        name: <operator_name>
        source: <catalog_name>
        sourceNamespace: <catalog_source_namespace>
        config:
          env:
          - name: ARGS
            value: "-v=10"
          envFrom:
          - secretRef:
              name: license-secret
          volumes:
          - name: <volume_name>
            configMap:
              name: <configmap_name>
          volumeMounts:
          - mountPath: <directory_name>
            name: <volume_name>
          tolerations:
          - operator: "Exists"
          resources:
            requests:
              memory: "64Mi"
              cpu: "250m"
            limits:
              memory: "128Mi"
              cpu: "500m"
          nodeSelector:
            foo: bar
      ```

      where:

      <dl>
      <dt><code>&lt;namespace_per_install_mode&gt;</code></dt>
      <dd>Specifies the namespace for your chosen install mode. For default <code>AllNamespaces</code> install mode usage, specify the <code>openshift-operators</code> namespace. Alternatively, you can specify a custom global namespace, if you have created one. For <code>SingleNamespace</code> install mode usage, specify the relevant single namespace.</dd>
      <dt><code>&lt;channel_name&gt;</code></dt>
      <dd>Specifies the name of the channel to subscribe to.</dd>
      <dt><code>&lt;operator_name&gt;</code></dt>
      <dd>Specifies the name of the Operator to subscribe to.</dd>
      <dt><code>&lt;catalog_name&gt;</code></dt>
      <dd>Specifies the name of the catalog source that provides the Operator.</dd>
      <dt><code>&lt;catalog_source_namespace&gt;</code></dt>
      <dd>Specifies the namespace of the catalog source. Use <code>openshift-marketplace</code> for the default software catalog sources.</dd>
      <dt><code>config.env</code></dt>
      <dd>Specifies a list of environment variables that must exist in all containers in the pod created by OLM.</dd>
      <dt><code>config.envFrom</code></dt>
      <dd>Specifies a list of sources to populate environment variables in the container.</dd>
      <dt><code>config.volumes</code></dt>
      <dd>Specifies a list of volumes that must exist on the pod created by OLM.</dd>
      <dt><code>config.volumeMounts</code></dt>
      <dd>Specifies a list of volume mounts that must exist in all containers in the pod created by OLM. If a <code>volumeMount</code> references a <code>volume</code> that does not exist, OLM fails to deploy the Operator.</dd>
      <dt><code>config.tolerations</code></dt>
      <dd>Specifies a list of tolerations for the pod created by OLM.</dd>
      <dt><code>config.resources</code></dt>
      <dd>Specifies resource constraints for all the containers in the pod created by OLM.</dd>
      <dt><code>config.nodeSelector</code></dt>
      <dd>Specifies a <code>NodeSelector</code> for the pod created by OLM.</dd>
      </dl>

      ```yaml title="Example Subscription object with a specific starting Operator version"
      apiVersion: operators.coreos.com/v1alpha1
      kind: Subscription
      metadata:
        name: example-operator
        namespace: example-operator
      spec:
        channel: stable-3.7
        installPlanApproval: Manual
        name: example-operator
        source: custom-operators
        sourceNamespace: openshift-marketplace
        startingCSV: example-operator.v3.7.10
      ```

      where: `installPlanApproval`:: Specifies the approval strategy. Set to `Manual` in case your specified version is superseded by a later version in the catalog. This plan prevents an automatic upgrade to a later version and requires manual approval before the starting CSV can complete the installation. `startingCSV`:: Specifies a specific version of an Operator CSV.
   2. For clusters on cloud providers with token authentication enabled, such as Amazon Web Services (AWS) Security Token Service (STS), Microsoft Entra Workload ID, or Google Cloud Platform Workload Identity, configure your `Subscription` object by following these steps:
      1. Ensure the `Subscription` object is set to manual update approvals:
         ```yaml title="Example Subscription object with manual update approvals"
         kind: Subscription
         # ...
         spec:
           installPlanApproval: Manual
         ```

         Set the `spec.installPlanApproval` parameter to `Manual`. Subscriptions with manual approvals for updates ensure that administrators have the opportunity to verify the permissions of the later version, take any necessary steps, and then update. Subscriptions with automatic approvals for updates are not recommended because there might be permission changes to make before updating.
      2. Include the relevant cloud provider-specific fields in the `Subscription` object’s `config` section:
         If the cluster is in AWS STS mode, include the following fields:

         ```yaml title="Example Subscription object with AWS STS variables"
         kind: Subscription
         # ...
         spec:
           config:
             env:
             - name: ROLEARN
               value: "<role_arn>"
         ```

         - `ROLEARN` is the Amazon Resource Name (ARN) of the role that the Operator assumes.
           If the cluster is in Workload ID mode, include the following fields:

           ```yaml title="Example Subscription object with Workload ID variables"
           kind: Subscription
           # ...
           spec:
            config:
              env:
              - name: CLIENTID
                value: "<client_id>"
              - name: TENANTID
                value: "<tenant_id>"
              - name: SUBSCRIPTIONID
                value: "<subscription_id>"
           ```

           where:

           <dl>
           <dt><code>&lt;client_id&gt;</code></dt>
           <dd>Specifies the client ID.</dd>
           <dt><code>&lt;tenant_id&gt;</code></dt>
           <dd>Specifies the tenant ID.</dd>
           <dt><code>&lt;subscription_id&gt;</code></dt>
           <dd>Specifies the subscription ID. If the cluster is in GCP Workload Identity mode, include the following fields:</dd>
           </dl>

           ```yaml title="Example Subscription object with GCP Workload Identity variables"
           kind: Subscription
           # ...
           spec:
            config:
              env:
              - name: AUDIENCE
                value: "<audience_url>"
              - name: SERVICE_ACCOUNT_EMAIL
                value: "<service_account_email>"
           ```

           where: `<audience_url>`:: Created in Google Cloud by the administrator when they set up GCP Workload Identity, the `AUDIENCE` value must be a preformatted URL in the following format:

           ```text
           //iam.googleapis.com/projects/<project_number>/locations/global/workloadIdentityPools/<pool_id>/providers/<provider_id>
           ```

           <dl>
           <dt><code>&lt;service_account_email&gt;</code></dt>
           <dd>Specifies a Google Cloud service account email that is impersonated during Operator operation, for example:</dd>
           </dl>

           ```text
           <service_account_name>@<project_id>.iam.gserviceaccount.com
           ```
   3. Create the `Subscription` object by running the following command:
      ```terminal
      $ oc apply -f subscription.yaml
      ```
6. If you set the `installPlanApproval` field to `Manual`, manually approve the pending install plan to complete the Operator installation. For more information, see "Manually approving a pending Operator update".

**Verification**

At this point, OLM is now aware of the selected Operator. A cluster service version (CSV) for the Operator should appear in the target namespace, and APIs provided by the Operator should be available for creation.

1. Check the status of the `Subscription` object for your installed Operator by running the following command:
   ```terminal
   $ oc describe subscription <subscription_name> -n <namespace>
   ```
2. If you created an Operator group for `SingleNamespace` install mode, check the status of the `OperatorGroup` object by running the following command:
   ```terminal
   $ oc describe operatorgroup <operatorgroup_name> -n <namespace>
   ```

**Additional resources**

- [About OperatorGroups](/docs/operators/understanding/olm/olm-understanding-operatorgroups#olm-operatorgroups-about_olm-understanding-operatorgroups)
