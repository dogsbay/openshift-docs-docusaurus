---
title: Admission plugins
sidebar_position: 9
---

# Admission plugins {#admission-plug-ins}

<a id="admission-plug-ins"></a>

You can use admission plugins to regulate how OpenShift Container Platform functions. The default set of admission plugins for OpenShift Container Platform ensures proper functioning for your cluster.

## About admission plugins {#admission-plug-ins-about_admission-plug-ins}

Admission plugins enforce security standards, resource limits, and configuration requirements across your cluster. These plugins intercept and validate authenticated API requests to ensure compliance with cluster policies.

After a request is authenticated and authorized, the admission plugins ensure that any associated policies are followed.

Admission plugins run in sequence as an admission chain. If any admission plugin in the sequence rejects a request, the whole chain is aborted and an error is returned.

OpenShift Container Platform has a default set of admission plugins enabled for each resource type. These are required for proper functioning of the cluster. Admission plugins ignore resources that they are not responsible for.

In addition to the defaults, the admission chain can be extended dynamically through webhook admission plugins that call out to custom webhook servers. There are two types of webhook admission plugins: a mutating admission plugin and a validating admission plugin. The mutating admission plugin runs first and can both modify resources and validate requests. The validating admission plugin validates requests and runs after the mutating admission plugin so that modifications triggered by the mutating admission plugin can also be validated.

Calling webhook servers through a mutating admission plugin can produce side effects on resources related to the target object. In such situations, you must take steps to validate that the end result is as expected.

:::warning

Dynamic admission should be used cautiously because it impacts cluster control plane operations. When calling webhook servers through webhook admission plugins in OpenShift Container Platform 4.22, ensure that you have read the documentation fully and tested for side effects of mutations. Include steps to restore resources back to their original state prior to mutation, in the event that a request does not pass through the entire admission chain.

:::

## Default admission plugins {#admission-plug-ins-default_admission-plug-ins}

Default validating and mutating admission plugins are enabled in OpenShift Container Platform. These default plugins contribute to fundamental control plane functionality, such as ingress policy, cluster resource limit override, and quota policy.

:::warning

Do not run workloads in or share access to default projects. Default projects are reserved for running core cluster components.

The following default projects are considered highly privileged: `default`, `kube-public`, `kube-system`, `openshift`, `openshift-infra`, `openshift-node`, and other system-created projects that have the `openshift.io/run-level` label set to `0` or `1`. Functionality that relies on admission plugins, such as pod security admission, security context constraints, cluster resource quotas, and image reference resolution, does not work in highly privileged projects.

:::

The following list includes the default validating admission plugins:

- `LimitRanger`
- `ServiceAccount`
- `PodNodeSelector`
- `Priority`
- `PodTolerationRestriction`
- `OwnerReferencesPermissionEnforcement`
- `PersistentVolumeClaimResize`
- `RuntimeClass`
- `CertificateApproval`
- `CertificateSigning`
- `CertificateSubjectRestriction`
- `autoscaling.openshift.io/ManagementCPUsOverride`
- `authorization.openshift.io/RestrictSubjectBindings`
- `scheduling.openshift.io/OriginPodNodeEnvironment`
- `network.openshift.io/ExternalIPRanger`
- `network.openshift.io/RestrictedEndpointsAdmission`
- `image.openshift.io/ImagePolicy`
- `security.openshift.io/SecurityContextConstraint`
- `security.openshift.io/SCCExecRestrictions`
- `route.openshift.io/IngressAdmission`
- `config.openshift.io/ValidateAPIServer`
- `config.openshift.io/ValidateAuthentication`
- `config.openshift.io/ValidateFeatureGate`
- `config.openshift.io/ValidateConsole`
- `operator.openshift.io/ValidateDNS`
- `config.openshift.io/ValidateImage`
- `config.openshift.io/ValidateOAuth`
- `config.openshift.io/ValidateProject`
- `config.openshift.io/DenyDeleteClusterConfiguration`
- `config.openshift.io/ValidateScheduler`
- `quota.openshift.io/ValidateClusterResourceQuota`
- `security.openshift.io/ValidateSecurityContextConstraints`
- `authorization.openshift.io/ValidateRoleBindingRestriction`
- `config.openshift.io/ValidateNetwork`
- `operator.openshift.io/ValidateKubeControllerManager`
- `ValidatingAdmissionWebhook`
- `ResourceQuota`
- `quota.openshift.io/ClusterResourceQuota`

The following list includes the default mutating admission plugins:

- `NamespaceLifecycle`
- `LimitRanger`
- `ServiceAccount`
- `NodeRestriction`
- `TaintNodesByCondition`
- `PodNodeSelector`
- `Priority`
- `DefaultTolerationSeconds`
- `PodTolerationRestriction`
- `DefaultStorageClass`
- `StorageObjectInUseProtection`
- `RuntimeClass`
- `DefaultIngressClass`
- `autoscaling.openshift.io/ManagementCPUsOverride`
- `scheduling.openshift.io/OriginPodNodeEnvironment`
- `image.openshift.io/ImagePolicy`
- `security.openshift.io/SecurityContextConstraint`
- `security.openshift.io/DefaultSecurityContextConstraints`
- `MutatingAdmissionWebhook`

## Webhook admission plugins {#admission-webhooks-about_admission-plug-ins}

In addition to OpenShift Container Platform default admission plugins, you can implement dynamic admission through webhook admission plugins that call webhook servers. This approach extends the functionality of the admission chain.

Webhook servers are called over HTTP at defined endpoints.

There are two types of webhook admission plugins in OpenShift Container Platform:

- During the admission process, the *mutating admission plugin* can perform tasks, such as injecting affinity labels.
- At the end of the admission process, the *validating admission plugin* validates that an object is configured properly, for example ensuring affinity labels are as expected. If the validation passes, OpenShift Container Platform schedules the object as configured.

When an API request comes in, mutating or validating admission plugins use the list of external webhooks in the configuration and call them in parallel:

- If all of the webhooks approve the request, the admission chain continues.
- If any of the webhooks deny the request, the admission request is denied and the reason for doing so is based on the first denial.
- If more than one webhook denies the admission request, only the first denial reason is returned to the user.
- If an error is encountered when calling a webhook, the request is either denied or the webhook is ignored depending on the error policy set. If the error policy is set to `Ignore`, the request is unconditionally accepted in the event of a failure. If the policy is set to `Fail`, failed requests are denied. Using `Ignore` can result in unpredictable behavior for all clients.

Communication between the webhook admission plugin and the webhook server must use TLS. Generate a CA certificate and use the certificate to sign the server certificate that is used by your webhook admission server. The PEM-encoded CA certificate is supplied to the webhook admission plugin using a mechanism, such as service serving certificate secrets.

The following diagram illustrates the sequential admission chain process within which multiple webhook servers are called.

**Figure 1. API admission chain with mutating and validating admission plugins**

![API admission stage](/images/api-admission-chain.png)

An example webhook admission plugin use case is where all pods must have a common set of labels. In this example, the mutating admission plugin can inject labels and the validating admission plugin can check that labels are as expected. OpenShift Container Platform would subsequently schedule pods that include required labels and reject those that do not.

Some common webhook admission plugin use cases include:

- Namespace reservation.
- Limiting custom network resources managed by the SR-IOV network device plugin.
- Defining tolerations that enable taints to qualify which pods should be scheduled on a node.
- Pod priority class validation.

:::note

The maximum default webhook timeout value in OpenShift Container Platform is 13 seconds, and it cannot be changed.

:::

## Types of webhook admission plugins {#admission-webhook-types_admission-plug-ins}

As a cluster administrator, you can call webhook servers through the mutating admission plugin or the validating admission plugin in the API server admission chain.

Mutating admission plugin
: The mutating admission plugin is invoked during the mutation phase of the admission process, which allows modification of resource content before it is persisted. One example webhook that can be called through the mutating admission plugin is the Pod Node Selector feature, which uses an annotation on a namespace to find a label selector and add it to the pod specification.

  ```yaml title="Sample mutating admission plugin configuration"
  apiVersion: admissionregistration.k8s.io/v1beta1
  kind: MutatingWebhookConfiguration
  metadata:
    name: <webhook_name>
  webhooks:
  - name: <webhook_name>
    clientConfig:
      service:
        namespace: default
        name: kubernetes
        path: <webhook_url>
      caBundle: <ca_signing_certificate>
    rules:
    - operations:
      - <operation>
      apiGroups:
      - ""
      apiVersions:
      - "*"
      resources:
      - <resource>
    failurePolicy: <policy>
    sideEffects: None
  ```

  where:

`kind`
: Specifies a mutating admission plugin configuration.

`metadata.name`
: Specifies the name for the `MutatingWebhookConfiguration` object. Replace `<webhook_name>` with the appropriate value.

`webhooks.name`
: Specifies the name of the webhook to call. Replace `<webhook_name>` with the appropriate value.

`webhooks.clientConfig`
: Specifies information about how to connect to, trust, and send data to the webhook server.

`webhooks.clientConfig.service.namespace`
: Specifies the namespace where the front-end service is created.

`webhooks.clientConfig.service.name`
: Specifies the name of the front-end service.

`webhooks.clientConfig.service.path`
: Specifies the webhook URL used for admission requests. Replace `<webhook_url>` with the appropriate value.

`webhooks.clientConfig.caBundle`
: Specifies a PEM-encoded CA certificate that signs the server certificate that is used by the webhook server. Replace `<ca_signing_certificate>` with the appropriate certificate in base64 format.

`webhooks.rules`
: Specifies rules that define when the API server should use this webhook admission plugin.

`webhooks.rules.operations`
: Specifies one or more operations that trigger the API server to call this webhook admission plugin. Possible values are `create`, `update`, `delete`, or `connect`. Replace `<operation>` and `<resource>` with the appropriate values.

`webhooks.failurePolicy`
: Specifies how the policy should proceed if the webhook server is unavailable. Replace `<policy>` with either `Ignore` (to unconditionally accept the request in case of a failure) or `Fail` (to deny the failed request). Using `Ignore` can result in unpredictable behavior for all clients.

  :::warning

  In OpenShift Container Platform 4.22, objects created by users or control loops through a mutating admission plugin might return unexpected results, especially if values set in an initial request are overwritten, which is not recommended.

  :::

Validating admission plugin
: A validating admission plugin is invoked during the validation phase of the admission process. This phase allows the enforcement of invariants on particular API resources to ensure that the resource does not change again. The Pod Node Selector is also an example of a webhook which is called by the validating admission plugin, to ensure that all `nodeSelector` fields are constrained by the node selector restrictions on the namespace.

  ```yaml title="Sample validating admission plugin configuration"
  apiVersion: admissionregistration.k8s.io/v1beta1
  kind: ValidatingWebhookConfiguration
  metadata:
    name: <webhook_name>
  webhooks:
  - name: <webhook_name>
    clientConfig:
      service:
        namespace: default
        name: kubernetes
        path: <webhook_url>
      caBundle: <ca_signing_certificate>
    rules:
    - operations:
      - <operation>
      apiGroups:
      - ""
      apiVersions:
      - "*"
      resources:
      - <resource>
    failurePolicy: <policy>
    sideEffects: Unknown
  ```

  where:

`kind`
: Specifies a validating admission plugin configuration.

`metadata.name`
: Specifies the name for the `ValidatingWebhookConfiguration` object. Replace `<webhook_name>` with the appropriate value.

`webhooks.name`
: Specifies the name of the webhook to call. Replace `<webhook_name>` with the appropriate value.

`webhooks.clientConfig`
: Specifies information about how to connect to, trust, and send data to the webhook server.

`webhooks.clientConfig.service.namespace`
: Specifies the namespace where the front-end service is created.

`webhooks.clientConfig.service.name`
: Specifies the name of the front-end service.

`webhooks.clientConfig.service.path`
: Specifies the webhook URL used for admission requests. Replace `<webhook_url>` with the appropriate value.

`webhooks.caBundle`
: Specifies a PEM-encoded CA certificate that signs the server certificate that is used by the webhook server. Replace `<ca_signing_certificate>` with the appropriate certificate in base64 format.

`webhooks.rules`
: Specifies rules that define when the API server should use this webhook admission plugin.

`webhooks.rules.operations`
: Specifies one or more operations that trigger the API server to call this webhook admission plugin. Possible values are `create`, `update`, `delete`, or `connect`. Replace `<operation>` and `<resource>` with the appropriate values.

`webhooks.failurePolicy`
: Specifies how the policy should proceed if the webhook server is unavailable. Replace `<policy>` with either `Ignore` (to unconditionally accept the request in case of a failure) or `Fail` (to deny the failed request). Using `Ignore` can result in unpredictable behavior for all clients.

## Configuring dynamic admission {#configuring-dynamic-admission_admission-plug-ins}

You can complete high-level steps to configure dynamic admission. These steps extend the admission chain by configuring a webhook admission plugin to call out to a webhook server.

The webhook server is also configured as an aggregated API server. This allows other OpenShift Container Platform components to communicate with the webhook that uses internal credentials and facilitates testing that uses the `oc` command. Additionally, this enables role-based access control (RBAC) into the webhook and prevents token information from other API servers from being disclosed to the webhook.

**Prerequisites**

- An OpenShift Container Platform account with cluster administrator access.
- The OpenShift Container Platform OpenShift CLI (`oc`) installed.
- A published webhook server container image.

**Procedure**

1. Build a webhook server container image and make it available to the cluster by using an image registry.
2. Create a local CA key and certificate and use them to sign the webhook server’s certificate signing request (CSR).
3. Create a new project for webhook resources:
   ```terminal
   $ oc new-project my-webhook-namespace
   ```

   - Note that the webhook server might expect a specific name.
4. Define RBAC rules for the aggregated API service in a file called `rbac.yaml`:
   ```yaml
   apiVersion: v1
   kind: List
   items:

   - apiVersion: rbac.authorization.k8s.io/v1
     kind: ClusterRoleBinding
     metadata:
       name: auth-delegator-my-webhook-namespace
     roleRef:
       kind: ClusterRole
       apiGroup: rbac.authorization.k8s.io
       name: system:auth-delegator
     subjects:
     - kind: ServiceAccount
       namespace: my-webhook-namespace
       name: server

   - apiVersion: rbac.authorization.k8s.io/v1
     kind: ClusterRole
     metadata:
       annotations:
       name: system:openshift:online:my-webhook-server
     rules:
     - apiGroups:
       - online.openshift.io
       resources:
       - namespacereservations
       verbs:
       - get
       - list
       - watch

   - apiVersion: rbac.authorization.k8s.io/v1
     kind: ClusterRole
     metadata:
       name: system:openshift:online:my-webhook-requester
     rules:
     - apiGroups:
       - admission.online.openshift.io
       resources:
       - namespacereservations
       verbs:
       - create

   - apiVersion: rbac.authorization.k8s.io/v1
     kind: ClusterRoleBinding
     metadata:
       name: my-webhook-server-my-webhook-namespace
     roleRef:
       kind: ClusterRole
       apiGroup: rbac.authorization.k8s.io
       name: system:openshift:online:my-webhook-server
     subjects:
     - kind: ServiceAccount
       namespace: my-webhook-namespace
       name: server

   - apiVersion: rbac.authorization.k8s.io/v1
     kind: RoleBinding
     metadata:
       namespace: kube-system
       name: extension-server-authentication-reader-my-webhook-namespace
     roleRef:
       kind: Role
       apiGroup: rbac.authorization.k8s.io
       name: extension-apiserver-authentication-reader
     subjects:
     - kind: ServiceAccount
       namespace: my-webhook-namespace
       name: server

   - apiVersion: rbac.authorization.k8s.io/v1
     kind: ClusterRole
     metadata:
       name: my-cluster-role
     rules:
     - apiGroups:
       - admissionregistration.k8s.io
       resources:
       - validatingwebhookconfigurations
       - mutatingwebhookconfigurations
       verbs:
       - get
       - list
       - watch
     - apiGroups:
       - ""
       resources:
       - namespaces
       verbs:
       - get
       - list
       - watch

   - apiVersion: rbac.authorization.k8s.io/v1
     kind: ClusterRoleBinding
     metadata:
       name: my-cluster-role
     roleRef:
       kind: ClusterRole
       apiGroup: rbac.authorization.k8s.io
       name: my-cluster-role
     subjects:
     - kind: ServiceAccount
       namespace: my-webhook-namespace
       name: server
   ```

   where:

   <dl>
   <dt><code>apiVersion: rbac.authorization.k8s.io/v1</code></dt>
   <dd>For <code>ClusterRoleBinding</code>, specifies authentication and authorization metadata to the webhook server API.</dd>
   <dt><code>apiVersion: rbac.authorization.k8s.io/v1</code></dt>
   <dd>For <code>ClusterRole</code>, specifies the webhook server that is allowed access cluster resources.</dd>
   <dt><code>rules.resources</code></dt>
   <dd>For <code>ClusterRole</code>, specifies the location to resources. This example points to the <code>namespacereservations</code> resource.</dd>
   <dt><code>apiVersion: rbac.authorization.k8s.io/v1</code></dt>
   <dd>For <code>ClusterRole</code>, specifies the enablement of the aggregated API server to create admission reviews.</dd>
   <dt><code>apiVersion: rbac.authorization.k8s.io/v1</code></dt>
   <dd>For <code>ClusterRoleBinding</code>, specifies the enablement of the webhook server to access cluster resources.</dd>
   <dt><code>apiVersion: rbac.authorization.k8s.io/v1</code></dt>
   <dd>For <code>RoleBinding</code>, specifies role binding to read the configuration for terminating authentication.</dd>
   <dt><code>apiVersion: rbac.authorization.k8s.io/v1</code></dt>
   <dd>For <code>ClusterRole</code>, specifies the default cluster role and cluster role bindings for an aggregated API server.</dd>
   </dl>
5. Apply those RBAC rules to the cluster:
   ```terminal
   $ oc auth reconcile -f rbac.yaml
   ```
6. Create a YAML file called `webhook-daemonset.yaml` that is used to deploy a webhook as a daemon set server in a namespace:
   ```yaml
   apiVersion: apps/v1
   kind: DaemonSet
   metadata:
     namespace: my-webhook-namespace
     name: server
     labels:
       server: "true"
   spec:
     selector:
       matchLabels:
         server: "true"
     template:
       metadata:
         name: server
         labels:
           server: "true"
       spec:
         serviceAccountName: server
         containers:
         - name: my-webhook-container
           image: <image_registry_username>/<image_path>:<tag>
           imagePullPolicy: IfNotPresent
           command:
           - <container_commands>
           ports:
           - containerPort: 8443
           volumeMounts:
           - mountPath: /var/serving-cert
             name: serving-cert
           readinessProbe:
             httpGet:
               path: /healthz
               port: 8443
               scheme: HTTPS
         volumes:
         - name: serving-cert
           secret:
             defaultMode: 420
             secretName: server-serving-cert
   ```

   where:

   <dl>
   <dt><code>spec.template.spec.name</code></dt>
   <dd>Specifies the container name. Note that the webhook server might expect a specific container name.</dd>
   <dt><code>spec.template.spec.image</code></dt>
   <dd>Specifies the path to a webhook server container image. Replace <code>&lt;image_registry_username&gt;/&lt;image_path&gt;:&lt;tag&gt;</code> with the appropriate value.</dd>
   <dt><code>spec.template.spec.command</code></dt>
   <dd>Specifies webhook container run commands. Replace <code>&lt;container_commands&gt;</code> with the appropriate value.</dd>
   <dt><code>spec.template.spec.ports.containerPort</code></dt>
   <dd>Specifies the target port within pods. This example uses port 8443.</dd>
   <dt><code>spec.template.spec.readinessProbe.port</code></dt>
   <dd>Specifies the port used by the readiness probe. This example uses port 8443.</dd>
   </dl>
7. Deploy the daemon set:
   ```terminal
   $ oc apply -f webhook-daemonset.yaml
   ```
8. Define a secret for the service serving certificate signer, within a YAML file called `webhook-secret.yaml`:
   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     namespace: my-webhook-namespace
     name: server-serving-cert
   type: kubernetes.io/tls
   data:
     tls.crt: <server_certificate>
     tls.key: <server_key>
   ```

   where:

   <dl>
   <dt><code>data.tls.crt</code></dt>
   <dd>References the signed webhook server certificate. Replace <code>&lt;server_certificate&gt;</code> with the appropriate certificate in base64 format.</dd>
   <dt><code>data.tls.key</code></dt>
   <dd>References the signed webhook server key. Replace <code>&lt;server_key&gt;</code> with the appropriate key in base64 format.</dd>
   </dl>
9. Create the secret:
   ```terminal
   $ oc apply -f webhook-secret.yaml
   ```
10. Define a service account and service, within a YAML file called `webhook-service.yaml`:
    ```yaml
    apiVersion: v1
    kind: List
    items:

    - apiVersion: v1
      kind: ServiceAccount
      metadata:
        namespace: my-webhook-namespace
        name: server

    - apiVersion: v1
      kind: Service
      metadata:
        namespace: my-webhook-namespace
        name: server
        annotations:
          service.beta.openshift.io/serving-cert-secret-name: server-serving-cert
      spec:
        selector:
          server: "true"
        ports:
        - port: 443
          targetPort: 8443
    ```

    where:

    <dl>
    <dt><code>spec.ports.port</code></dt>
    <dd>Specifies the port that the service listens on. This example uses port 443.</dd>
    <dt><code>spec.ports.targetPort</code></dt>
    <dd>Specifies the target port within pods that the service forwards connections to. This example uses port 8443.</dd>
    </dl>
11. Expose the webhook server within the cluster:
    ```terminal
    $ oc apply -f webhook-service.yaml
    ```
12. Define a custom resource definition for the webhook server, in a file called `webhook-crd.yaml`:
    ```yaml
    apiVersion: apiextensions.k8s.io/v1beta1
    kind: CustomResourceDefinition
    metadata:
      name: namespacereservations.online.openshift.io
    spec:
      group: online.openshift.io
      version: v1alpha1
      scope: Cluster
      names:
        plural: namespacereservations
        singular: namespacereservation
        kind: NamespaceReservation
    ```

    where:

    <dl>
    <dt><code>metadata.name</code></dt>
    <dd>Reflects <code>CustomResourceDefinition</code> <code>spec</code> values and is in the format <code>&lt;plural&gt;.&lt;group&gt;</code>. This example uses the <code>namespacereservations</code> resource.</dd>
    <dt><code>spec.group</code></dt>
    <dd>Specifies the REST API group name.</dd>
    <dt><code>spec.version</code></dt>
    <dd>Specifies the REST API version name.</dd>
    <dt><code>spec.scope</code></dt>
    <dd>Specifies accepted values are <code>Namespaced</code> or <code>Cluster</code>.</dd>
    <dt><code>spec.names.plural</code></dt>
    <dd>Specifies the plural name to be included in URL.</dd>
    <dt><code>spec.names.singular</code></dt>
    <dd>Specifies the alias seen in <code>oc</code> output.</dd>
    <dt><code>spec.names.kind</code></dt>
    <dd>Specifies the reference for resource manifests.</dd>
    </dl>
13. Apply the custom resource definition:
    ```terminal
    $ oc apply -f webhook-crd.yaml
    ```
14. Configure the webhook server also as an aggregated API server, within a file called `webhook-api-service.yaml`:
    ```yaml
    apiVersion: apiregistration.k8s.io/v1beta1
    kind: APIService
    metadata:
      name: v1beta1.admission.online.openshift.io
    spec:
      caBundle: <ca_signing_certificate>
      group: admission.online.openshift.io
      groupPriorityMinimum: 1000
      versionPriority: 15
      service:
        name: server
        namespace: my-webhook-namespace
      version: v1beta1
    ```

    - The `spec.caBundle` field specifies a PEM-encoded CA certificate that signs the server certificate. This certificate is used by the webhook server. Replace `<ca_signing_certificate>` with the appropriate certificate in base64 format.
15. Deploy the aggregated API service:
    ```terminal
    $ oc apply -f webhook-api-service.yaml
    ```
16. Define the webhook admission plugin configuration within a file called `webhook-config.yaml`. This example uses the validating admission plugin:
    ```yaml
    apiVersion: admissionregistration.k8s.io/v1beta1
    kind: ValidatingWebhookConfiguration
    metadata:
      name: namespacereservations.admission.online.openshift.io
    webhooks:
    - name: namespacereservations.admission.online.openshift.io
      clientConfig:
        service:
          namespace: default
          name: kubernetes
          path: /apis/admission.online.openshift.io/v1beta1/namespacereservations
        caBundle: <ca_signing_certificate>
      rules:
      - operations:
        - CREATE
        apiGroups:
        - project.openshift.io
        apiVersions:
        - "*"
        resources:
        - projectrequests
      - operations:
        - CREATE
        apiGroups:
        - ""
        apiVersions:
        - "*"
        resources:
        - namespaces
      failurePolicy: Fail
    ```

    where:

    <dl>
    <dt><code>metadata.name</code></dt>
    <dd>Specifies the name for the <code>ValidatingWebhookConfiguration</code> object. This example uses the <code>namespacereservations</code> resource.</dd>
    <dt><code>webhooks.name</code></dt>
    <dd>Specifies the name of the webhook to call. This example uses the <code>namespacereservations</code> resource.</dd>
    <dt><code>name.clientConfig.service</code></dt>
    <dd>Enables access to the webhook server through the aggregated API.</dd>
    <dt><code>name.clientConfig.service.path</code></dt>
    <dd>Specifies the webhook URL used for admission requests. This example uses the <code>namespacereservation</code> resource.</dd>
    <dt><code>name.clientConfig.caBundle</code></dt>
    <dd>Specifies a PEM-encoded CA certificate that signs the server certificate that is used by the webhook server. Replace <code>&lt;ca_signing_certificate&gt;</code> with the appropriate certificate in base64 format.</dd>
    </dl>
17. Deploy the webhook:
    ```terminal
    $ oc apply -f webhook-config.yaml
    ```
18. Verify that the webhook is functioning as expected. For example, if you have configured dynamic admission to reserve specific namespaces, confirm that requests to create those namespaces are rejected and that requests to create non-reserved namespaces succeed.

**Additional resources**

- [Configuring the SR-IOV Network Operator](/docs/networking/networking_operators/sr-iov-operator/configuring-sriov-operator#configuring-sriov-operator_configuring-sriov-operator)
- [Controlling pod placement using node taints](/docs/nodes/scheduling/nodes-scheduler-taints-tolerations#nodes-scheduler-taints-tolerations_dedicating_nodes-scheduler-taints-tolerations)
- [Including pod priority in pod scheduling decisions](/docs/nodes/pods/nodes-pods-priority#admin-guide-priority-preemption-names_nodes-pods-priority)
