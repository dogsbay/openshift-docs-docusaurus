---
title: Customizing the External Secrets Operator for Red Hat OpenShift
sidebar_position: 7
---

# Customizing the External Secrets Operator for Red Hat OpenShift {#external-secrets-log-levels}

<a id="external-secrets-log-levels"></a>

You can customize the behavior of the External Secrets Operator for Red Hat OpenShift operand components by configuring custom annotations, deployment lifecycle settings, and environment variables through the `ExternalSecretsConfig` custom resource (CR).

These configurations provide administrators with fine-grained control over the external-secrets deployment.

You can customize the External Secrets Operator for Red Hat OpenShift operand by using the `ExternalSecretsConfig` custom resource (CR). The CR supports a set of deployment and runtime options, such as custom annotations, revision history limits, environment variables, resource limits, tolerations, and proxy settings—so you can control how the operand is deployed and run without editing the operand resources directly.

All supported options are defined in the `ExternalSecretsConfig` CR (for example under the `spec.controllerConfig` for controller-related settings). The Operator reconciles the operand from this CR. Changes made directly to operand resources are overwritten. Use the `ExternalSecretsConfig` CR as the only supported way to customize the operand.

For the complete list of fields and allowed values, see the `ExternalSecretsConfig` API reference in the External Secrets Operator for Red Hat OpenShift documentation.

**Additional resources**

- [External Secrets Operator for Red Hat OpenShift APIs](/docs/security/external_secrets_operator/external-secrets-operator-api#external-secrets-operator-api)

## Set a log level for the External Secrets Operator for Red Hat OpenShift {#external-secrets-enable-operator-log-level_external-secrets-log-levels}

You can configure the log verbosity for the lifecycle manager. You must adjust this setting to troubleshoot issues related to the installation, upgrade, or configuration of the operator itself, rather than secret synchronization.

**Prerequisites**

- You have access to the cluster with `cluster-admin` privileges.
- You have created the `ExternalSecretsConfig` custom resource.

**Procedure**

- Update the subscription object for the External Secrets Operator for Red Hat OpenShift to provide the verbosity level for the operator logs by running the following command:
  ```terminal
  $ oc -n <external_secrets_operator_namespace> patch subscription openshift-external-secrets-operator --type='merge' -p '{"spec":{"config":{"env":[{"name":"OPERATOR_LOG_LEVEL","value":"<log_level>"}]}}}'
  ```

  where:

  <dl>
  <dt>external_secrets_operator_namespace</dt>
  <dd>Specifies the namespace where the Operator is installed.</dd>
  </dl>

log_level
: Specifies the level of log detail. Values range from 1-5. The default is 2.

**Verification**

1. The External Secrets Operator pod is redeployed. Verify that the log level of the External Secrets Operator for Red Hat OpenShift is updated by running the following command:
   ```terminal
   $ oc set env deploy/external-secrets-operator-controller-manager -n external-secrets-operator --list | grep -e OPERATOR_LOG_LEVEL -e container
   ```

   The following example verifies that the log level of the External Secrets Operator for Red Hat OpenShift is updated.

   ```terminal
   # deployments/external-secrets-operator-controller-manager, container manager
   OPERATOR_LOG_LEVEL=2
   ```
2. Verify that the log level of the External Secrets Operator for Red Hat OpenShift is updated by running the `oc logs` command:
   ```terminal
   $ oc logs -n external-secrets-operator -f deployments/external-secrets-operator-controller-manager -c manager
   ```

## Set a log level for the External Secrets Operator for Red Hat OpenShift operand {#external-secrets-enable-operand-log-level_external-secrets-log-levels}

You can troubleshoot common issues, such as secret synchronization failures, provider authentication errors, or data formatting problems, by configuring the log verbosity for the core controller.

**Prerequisites**

- You have access to the cluster with `cluster-admin` privileges.
- You have created the `ExternalSecretsConfig` custom resource.

**Procedure**

1. Edit the `ExternalSecretsConfig` CR by running the following command:
   ```terminal
   $ oc edit externalsecretsconfigs.operator.openshift.io cluster
   ```
2. Set the log level value by editing the `spec.appConfig.logLevel` section:
   ```yaml
   apiVersion: operator.openshift.io/v1alpha1
   kind: ExternalSecretsConfig
   ...
   spec:
     appConfig:
       logLevel: <log_level>
   ```

   where:

   <dl>
   <dt>log_level</dt>
   <dd>Supports the value range of 1-5. The log level gets mapped to the following operand support levels: *   1 - warnings *   2 - error logs *   3 - info logs *   4 and 5 - debug logs</dd>
   </dl>
3. Save your changes and exit the editor.

## Configure cert-manager for the external-secrets certificate requirements {#external-secrets-cert-manager-config_external-secrets-log-levels}

You can optionally configure cert-manager to manage certificates for the External Secrets Operator for Red Hat OpenShift webhook and plugins. If you do not use cert-manager, the Operator automatically generates webhook certificates, but you must manually configure certificates for any plugins.

**Prerequisites**

- You have access to the cluster with `cluster-admin` privileges.
- You have created the `ExternalSecretsConfig` custom resource.
- You have installed the cert-manager Operator for Red Hat OpenShift. For more information, see "Installing the cert-manager Operator for Red Hat OpenShift"

**Procedure**

1. Edit the `ExternalSecretsConfig` custom resource by running the following command:
   ```terminal
   $  oc edit externalsecretsconfigs.operator.openshift.io cluster
   ```
2. Configure `cert-manager` by editing the `spec.controllerConfig.certProvider.certManager` section as follows:
   ```yaml
   apiVersion: operator.openshift.io/v1alpha1
   kind: ExternalSecretsConfig
   ...
   spec:
     controllerConfig:
       certProvider:
         certManager:
           injectAnnotations: "true"
           issuerRef:
             name: <issuer_name>
             kind: <issuer_kind>
             group: <issuer_group>
           mode: Enabled
   ```

   where:

   <dl>
   <dt>injectAnnotation</dt>
   <dd>Must be set to <code>true</code> when enabled.</dd>
   <dt>name</dt>
   <dd>Specifies the name of the issuer object referenced in <code>ExternalSecretsConfig</code>.</dd>
   <dt>kind</dt>
   <dd>Specifies the API issuer. Can be set to either <code>Issuer</code> or <code>ClusterIssuer</code>.</dd>
   <dt>group</dt>
   <dd>Specifies the API issuer group. The group name must be <code>cert-manager.io</code>.</dd>
   <dt>mode</dt>
   <dd>Must be set to <code>Enabled</code>. This is an immutable field and cannot be modified once it is configured.</dd>
   </dl>
3. Save your changes.
4. After you update the `cert-manager` configurations in the `externalsecretsconfig.operator.openshift.io` object, you must manually delete `external-secrets-cert-controller` deployment by running the following command. This prevents performance degradation of the `external-secrets` application.
   ```terminal
   $ oc delete deployments.apps external-secrets-cert-controller -n external-secrets
   ```
5. Optionally, you can delete other resources created for the `cert-controller` by running the following commands:
   ```terminal
   $ oc delete clusterrolebindings.rbac.authorization.k8s.io external-secrets-cert-controller
   ```

   ```terminal
   $ oc delete clusterroles.rbac.authorization.k8s.io external-secrets-cert-controller
   ```

   ```terminal
   $ oc delete serviceaccounts external-secrets-cert-controller -n external-secrets
   ```

   ```terminal
   $ oc delete secrets external-secrets-webhook -n external-secrets
   ```

**Additional resources**

- [External Secrets Operator for Red Hat OpenShift APIs](/docs/security/external_secrets_operator/external-secrets-operator-api#external-secrets-operator-api)
- [cert-manager Operator for Red Hat Openshift](/docs/security/cert_manager_operator#cert-manager-operator-about)
- [Installing the cert-manager-Operator for Red Hat Openshift](/docs/security/cert_manager_operator/cert-manager-operator-install#cert-manager-operator-install)

## Configure the bitwardenSecretManagerProvider plugin {#external-secrets-bit-warden-config_external-secrets-log-levels}

You must configure the `bitwardenSecretManagerProvider` plugin to enable communication with the Bitwarden API. This configuration enables the Operator to authenticate and fetch secrets for synchronization.

**Prerequisites**

- You have access to the cluster with `cluster-admin` privileges.
- You have created the `ExternalSecretsConfig` custom resource.

**Procedure**

1. Edit the `ExternalSecretsConfig` custom resource by running the following command:
   ```terminal
   $  oc edit externalsecretsconfigs.operator.openshift.io cluster
   ```
2. Edit the `spec.plugins.bitwardenSecretManagerProvider` section as follows to enable the Bitwarden Secrets Manager:
   ```yaml
   apiVersion: operator.openshift.io/v1alpha1
   kind: ExternalSecretsConfig
   ...
   spec:
     plugins:
       bitwardenSecretManagerProvider:
         mode: Enabled
         secretRef:
           name: <secret_object_name>
   ```

   where:

   <dl>
   <dt>name</dt>
   <dd>The name of the secret containing the certificate key pair for the plugin. The key name in the secret for the certificate must be <code>tls.crt</code>. The key name for the private key must be <code>tls.key</code>. The key name for the Certificate Authority (CA) certificate key name must be <code>ca.crt</code>. Configuring the secret is optional when the cert-manager certificate provider is configured.</dd>
   </dl>
3. Save your changes and exit the editor.
4. If you disable the plugin the following resources must be deleted manually by running the following commands:
   ```terminal
   $ oc delete deployments.apps bitwarden-sdk-server -n external-secrets
   ```

   ```terminal
   $ oc delete certificates.cert-manager.io bitwarden-tls-certs -n external-secrets
   ```

   ```terminal
   $ oc delete service bitwarden-sdk-server -n external-secrets
   ```

   ```terminal
   $ oc delete serviceaccounts bitwarden-sdk-server -n external-secrets
   ```

## Add custom annotations to external-secrets resources {#external-secrets-enable-operator-adding-custom-annotations_external-secrets-log-levels}

To customize your resources, you can define up to 20 custom annotations in the custom resource (CR). The Operator merges the annotations with the defaults, prioritizes them, and safely preserves annotations set by external systems.

When an annotation is removed from the CR, the Operator automatically removes it from all managed resources during the next reconciliation. Annotations set by external sources, such as Kubernetes system annotations or annotations added by other controllers, are preserved and are not affected by the Operator.

Annotation keys containing the following reserved domain prefixes are not allowed and are rejected by validation if applied:

- `kubernetes.io/` (including subdomains such as `*.kubernetes.io/`)
- `k8s.io/` (including subdomains such as `*.k8s.io/`)
- `openshift.io/` (including subdomains such as `*.openshift.io/`)
- `cert-manager.io/`

**Prerequisites**

- You have access to the cluster with `cluster-admin` privileges.
- You have created the `ExternalSecretsConfig` custom resource.

**Procedure**

1. Edit the `ExternalSecretsConfig` CR by running the following command:
   ```terminal
   $ oc edit externalsecretsconfigs.operator.openshift.io cluster
   ```
2. Add the `annotations` field under `spec.controllerConfig` as follows:
   ```yaml
   apiVersion: operator.openshift.io/v1alpha1
   kind: ExternalSecretsConfig
   metadata:
     name: cluster
   spec:
     controllerConfig:
       annotations:
         prometheus.io/scrape: "true"
         example.com/environment: "production"
   ```

**Verification**

1. Verify that annotations are applied to the external-secrets deployment by running the following command:
   ```terminal
   $ oc get deployment external-secrets -n external-secrets -o jsonpath='{.metadata.annotations}' | jq .
   ```

   The output should include the custom annotations you specified.
2. Verify that annotations are applied to the pod template by running the following command:
   ```terminal
   $ oc get deployment external-secrets -n external-secrets -o jsonpath='{.spec.template.metadata.annotations}' | jq .
   ```

   The output should include the custom annotations you specified.
3. Verify that annotations are applied to other managed resources such as Services by running the following command:
   ```terminal
   $ oc get service external-secrets-webhook -n external-secrets -o jsonpath='{.metadata.annotations}' | jq .
   ```

   The output should include the custom annotations you specified.

## Configure the revisionHistoryLimit for external-secrets components {#external-secrets-enable-operator-configure-history-limit_external-secrets-log-levels}

Configure the number of old `ReplicaSet` objects retained for rollback by setting the `revisionHistoryLimit` parameter for `external-secrets` components.

The following components can be configured:

| Component name | Description |
| --- | --- |
| `ExternalSecretsCoreController` | The main `external-secrets` controller. |
| `Webhook` | The `external-secrets` webhook server. |
| `CertController` | The certificate controller for webhook TLS. |
| `BitwardenSDKServer` | The Bitwarden SDK server plugin. |

Each component can only have one configuration entry. A maximum of 4 component configuration entries are allowed, one per component.

**Prerequisites**

- You have access to the cluster with `cluster-admin` privileges.
- You have created the `ExternalSecretsConfig` custom resource.

**Procedure**

1. Edit the `ExternalSecretsConfig` CR by running the following command:
   ```terminal
   $ oc edit externalsecretsconfigs.operator.openshift.io cluster
   ```
2. Add the `componentConfigs` field under `spec.controllerConfig` as follows:
   ```yaml
   apiVersion: operator.openshift.io/v1alpha1
   kind: ExternalSecretsConfig
   metadata:
     name: cluster
   spec:
     controllerConfig:
       componentConfigs:
         - componentName: ExternalSecretsCoreController
           deploymentConfigs:
             revisionHistoryLimit: 5
         - componentName: Webhook
           deploymentConfigs:
             revisionHistoryLimit: 3
   ```

   where

   <dl>
   <dt><code>spec.controllerConfig.componentConfigs.componentName.deploymentConfigs.revisionHistoryLimit</code></dt>
   <dd>Specifies the number of old <code>ReplicaSet</code> objects to retain for rollback. The value must be at least 1 to ensure rollback capability. The maximum value is 50. If not specified, the default is 10.</dd>
   </dl>

**Verification**

- Verify that the `revisionHistoryLimit` parameter is applied to the deployment by running the following command:
  ```terminal
  $ oc get deployment external-secrets -n external-secrets -o jsonpath='{.spec.revisionHistoryLimit}'
  ```

  The output should display the value you configured.

## Set custom environment variables for external-secrets components {#external-secrets-operator-set-custom-variables_external-secrets-log-levels}

To configure component behavior at runtime or integrate with external services, set custom environment variables for individual `external-secrets` components.

Custom environment variables are merged with the default environment variables set by the Operator. User-specified variables take precedence in case of conflicts with the Operator defaults. A maximum of 50 custom environment variables can be specified per component.

The environment variable names starting with the following prefixes are reserved:

- `HOSTNAME`
- `KUBERNETES_`
- `EXTERNAL_SECRETS_`

**Prerequisites**

- You have access to the cluster with `cluster-admin` privileges.
- You have created the `ExternalSecretsConfig` custom resource.

**Procedure**

1. Edit the `ExternalSecretsConfig` CR by running the following command:
   ```terminal
   $ oc edit externalsecretsconfigs.operator.openshift.io cluster
   ```
2. Add the `overrideEnv` field under the desired component in the `spec.controllerConfig.componentConfigs` stanza as follows:
   ```yaml
   apiVersion: operator.openshift.io/v1alpha1
   kind: ExternalSecretsConfig
   metadata:
     name: cluster
   spec:
     controllerConfig:
       componentConfigs:
         - componentName: ExternalSecretsCoreController
           overrideEnv:
             - name: Example
               value: "4"
   ```

   where

   <dl>
   <dt><code>spec.controllerConfig.componentConfigs.overrideEnv.name</code></dt>
   <dd>Specifies the name of the environment variable. Environment variable names starting with <code>HOSTNAME</code>, <code>KUBERNETES_</code>, or <code>EXTERNAL_SECRETS_</code> are reserved and are not allowed.</dd>
   </dl>

`spec.controllerConfig.componentConfigs.overrideEnv.value`
: Specifies the value of the environment variable.

**Verification**

- Verify that the environment variable is set on the deployment by running the following command:
  ```terminal
  $ oc set env deployment/external-secrets -n external-secrets --list
  ```

  The output should include the custom environment variable you specified.

## Enable optional features for External Secrets Operator for Red Hat OpenShift {#external-secrets-operator-enable-optional-features_external-secrets-log-levels}

The External Secrets Operator for Red Hat OpenShift supports optional capabilities that can be enabled cluster-wide through the `ExternalSecretsManager` custom resource (CR). Features are disabled by default and must be explicitly enabled.

You can enable or disable a feature at any time. The Operator reconciles the core controller deployment when the feature state changes, without requiring a restart or reinstallation.

:::warning

`UnsafeAllowGenericTargets` is a pre-release feature. It is not recommended for production use. Enabling this feature allows `ExternalSecret` resources to write secret data to arbitrary Kubernetes resource types beyond Secret objects. This might cause data managed by other controllers to be overwritten and can expose sensitive values through non-secret resources. This feature provides no additional access control beyond standard Kubernetes role-based access control (RBAC).

:::

When enabled, `ExternalSecret` resources can target arbitrary Kubernetes resource types as their sync destination, instead of being limited to `Secret` objects.

The Operator passes the `--unsafe-allow-generic-targets=true` flag to the core `external-secrets` controller. The webhook and cert-controller are not affected.

**Prerequisites**

- You have access to the cluster with `cluster-admin` privileges.
- You have installed the External Secrets Operator for Red Hat OpenShift and created the `ExternalSecretsConfig` CR.

**Procedure**

1. Edit the `ExternalSecretsManager` CR by running the following command:
   ```terminal
   $ oc edit externalsecretsmanagers.operator.openshift.io cluster
   ```
2. Add the `features` field under `spec` and set the desired feature mode:
   ```yaml
   apiVersion: operator.openshift.io/v1alpha1
   kind: ExternalSecretsManager
   metadata:
     name: cluster
   spec:
     features:
       - name: UnsafeAllowGenericTargets
         mode: Enabled
   ```

   To disable the feature, set `mode: Disabled` or remove the entry from the features list.

**Verification**

1. Verify that the feature flag is passed to the core controller by running the following command:
   ```terminal
   $ oc get deployment external-secrets \
     -n external-secrets \
     -o jsonpath='{.spec.template.spec.containers[0].args}' | jq .
   ```

   ```json title="Example output"
   [
     "--concurrent=1",
     "--metrics-addr=:8080",
     "--loglevel=warn",
     "--zap-time-encoding=epoch",
     "--enable-leader-election=true",
     "--enable-push-secret-reconciler=true",
     "--enable-cluster-store-reconciler=true",
     "--enable-cluster-external-secret-reconciler=true",
     "--unsafe-allow-generic-targets=true"
   ]
   ```

   When the feature is enabled, the output includes `--unsafe-allow-generic-targets=true`. When disabled or not configured, the flag is absent.
2. Verify that the `ExternalSecretsManager` CR reflects the configured feature by running the following command:
   ```terminal
   $ oc get externalsecretsmanagers.operator.openshift.io cluster -o jsonpath='{.spec.features}' | jq .
   ```

   ```json title="Example output"
   [
     {
       "mode": "Enabled",
       "name": "UnsafeAllowGenericTargets"
     }
   ]
   ```

## Mount a custom trusted certificate authority bundle for external-secrets {#external-secrets-operator-mounting-bundle_external-secrets-log-levels}

You can configure the External Secrets Operator for Red Hat OpenShift to trust a custom certificate authority (CA) bundle when the `external-secrets` core controller communicates with external secret backends over transport layer socket (TLS). This is required when your organization uses a private CA or a self-signed certificate that is not included in the default system truststore.

To enable mounting a custom trusted CA, you reference a `ConfigMap` that contains the Privacy Enhanced Mail (PEM)-encoded CA certificates in the `spec.controllerConfig.trustedCABundle` field of the `ExternalSecretsConfig` custom resource (CR). The Operator mounts the bundle into the core controller pod and configures the TLS library to use it alongside the default system trust stores.

The External Secrets Operator for Red Hat OpenShift applies the following rules to the CA bundle `ConfigMap`:

- The `ConfigMap` must reside in the `external-secrets` namespace and must contain only PEM-encoded X.509 CA certificates. Leaf certificates and private key PEM blocks are rejected.
- If the `ConfigMap` key contains an invalid bundle, the `ExternalSecretsConfig` CR enters a `Degraded` state. The Operator automatically recovers and mounts the bundle when the `ConfigMap` is corrected, without requiring manual intervention.
- If the referenced `ConfigMap` does not exist, the Operator removes any previously mounted CA bundle from the core controller deployment and sets the `ExternalSecretsConfig` CR to a `Degraded` state until the `ConfigMap` is created.
- The CA bundle is mounted only on the core `external-secrets` controller container. The webhook and cert-controller containers are not affected.
- If the `ConfigMap` has the `config.openshift.io/inject-trusted-cabundle: "true"` label and a cluster proxy is configured, the Operator skips the user-defined mount. The cluster-wide CA bundle injected by the Cluster Network Operator (CNO) is already available to the controller through the proxy CA bundle mechanism.

**Prerequisites**

- You have access to the cluster with `cluster-admin` privileges.
- You have installed the External Secrets Operator for Red Hat OpenShift and created the `ExternalSecretsConfig` CR.
- A `ConfigMap` containing PEM-encoded X.509 CA certificates exists in the `external-secrets` namespace.

**Procedure**

1. Create the `ConfigMap` containing your CA bundle by running the following command:
   ```terminal
   $ oc create configmap user-ca-bundle \
     --from-file=ca-bundle.crt=/path/to/ca.pem \
     -n external-secrets
   ```
2. Edit the `ExternalSecretsConfig` CR by running the following command:
   ```terminal
   $ oc edit externalsecretsconfigs.operator.openshift.io cluster
   ```
3. Add the `trustedCABundle` field under `spec.controllerConfig`:
   ```yaml
   apiVersion: operator.openshift.io/v1alpha1
   kind: ExternalSecretsConfig
   metadata:
     name: cluster
   spec:
     controllerConfig:
       trustedCABundle:
         name: user-ca-bundle
         key: ca-bundle.crt
   ```

   where:

   <dl>
   <dt><code>spec.controllerConfig.trustedCABundle.name</code></dt>
   <dd>Specifies the name of the <code>ConfigMap</code> in the <code>external-secrets</code> namespace that contains the CA certificate bundle.</dd>
   <dt><code>spec.controllerConfig.trustedCABundle.key</code></dt>
   <dd>Optional. Specifies the key within the <code>ConfigMap</code> that holds the PEM-encoded CA bundle. The default is <code>ca-bundle.crt</code>.</dd>
   </dl>

**Verification**

1. Verify that the CA bundle volume is mounted on the core controller deployment by running the following command:
   ```terminal
   $ oc get deployment external-secrets \
     -n external-secrets \
     -o jsonpath='{.spec.template.spec.volumes}' | jq '.[] | select(.name=="user-ca-bundle")'
   ```

   ```json title="Example output"
   {
     "configMap": {
       "defaultMode": 420,
       "items": [
         {
           "key": "ca-bundle.crt",
           "path": "ca-bundle.crt"
         }
       ],
       "name": "trusted-ca-bundle-for-es"
     },
     "name": "user-ca-bundle"
   }
   ```
2. Verify that the `SSL_CERT_DIR` is set on the core controller container by running the following command:
   ```terminal
   $ oc set env deployment/external-secrets \
     -n external-secrets \
     --list | grep SSL_CERT_DIR
   ```

   ```terminal title="Example output"
   SSL_CERT_DIR=/etc/pki/tls/user-certs:/etc/pki/tls/certs:/etc/ssl/certs
   ```
3. Verify that the `ExternalSecretsConfig` CR is not in a `Degraded` state by running the following command:
   ```terminal
   $ oc get externalsecretsconfigs.operator.openshift.io cluster \
     -o jsonpath='{.status.conditions[?(@.type=="Degraded")]}' | jq .
   ```

   ```json title="Example output"
   {
     "lastTransitionTime": "2026-06-22T10:29:11Z",
     "message": "",
     "observedGeneration": 5,
     "reason": "Ready",
     "status": "False",
     "type": "Degraded"
   }
   ```

   The `Degraded` condition should show `"status": "False"`. If the condition is `True,` review the message field for the specific validation error and correct the referenced `ConfigMap`.

## Override operand container arguments for the External Secrets Operator for Red Hat OpenShift {#external-secrets-operator-overriding-operand-arguments_external-secrets-log-levels}

You can override container arguments for the `external-secrets` operand deployments by setting environment variables on the External Secrets Operator for Red Hat OpenShift subscription. Use this method when you need to pass additional or replacement `--key=value` flags to operand containers.

:::note

This is a temporary feature available only in the External Secrets Operator for Red Hat OpenShift 1.1 and 1.2 z-streams. External Secrets Operator for Red Hat OpenShift 1.3.0 adds support in the `ExternalSecretsConfig` API to configure the same behavior. Consider migrating to the `ExternalSecretsConfig` API when upgrading to External Secrets Operator 1.3.0. This subscription-based method is scheduled to be deprecated and removed in External Secrets Operator for Red Hat OpenShift 1.4.0.

:::

**Prerequisites**

- Your installed External Secrets Operator for Red Hat OpenShift version is 1.1 or 1.2.
- You have access to the cluster as a user with the `cluster-admin` role.

**Procedure**

- Update the Subscription for the External Secrets Operator to set the operand argument overrides by running the following command:
  ```terminal
  $ oc -n <external_secrets_operator_namespace> patch subscription openshift-external-secrets-operator --type='merge' -p '{"spec":{"config":{"env":[{"name":"OPERAND_EXTERNAL_SECRETS_ARGS","value":"<controller_args>"},{"name":"OPERAND_WEBHOOK_ARGS","value":"<webhook_args>"},{"name":"OPERAND_CERT_CONTROLLER_ARGS","value":"<cert_controller_args>"},{"name":"OPERAND_BITWARDEN_SDK_SERVER_ARGS","value":"<bitwarden_args>"}]}}}'
  ```

  where:

  <dl>
  <dt><code>&lt;external_secrets_operator_namespace&gt;</code></dt>
  <dd>Specifies the namespace where the Operator is installed.</dd>
  <dt><code>&lt;controller_args&gt;</code></dt>
  <dd>Specifies a comma-separated list of <code>--key</code> or <code>--key=value</code> flags for the <code>external-secrets</code> core controller. For example, <code>--concurrent=2,--loglevel=debug</code>.</dd>
  <dt><code>&lt;webhook_args&gt;</code></dt>
  <dd>Specifies a comma-separated list of flags for the webhook. Commas inside a flag value are preserved when the next flag begins with <code>--</code>. For example, <code>--tls-ciphers=TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256,--loglevel=debug</code>.</dd>
  <dt><code>&lt;cert_controller_args&gt;</code></dt>
  <dd>Specifies a comma-separated list of flags for the <code>cert-controller</code>. For example, <code>--crd-requeue-interval=10m,--loglevel=debug</code>.</dd>
  <dt><code>&lt;bitwarden_args&gt;</code></dt>
  <dd>Specifies a comma-separated list of flags for the <code>bitwarden-sdk-server</code>. For example, <code>--key-file=/certs/key.pem,--cert-file=/certs/cert.pem</code>. Set only the environment variables you need. Omit any unused entries from the <code>env</code> list.</dd>
  </dl>

**Verification**

1. Verify that the environment variables are set on the Operator by running the following command:
   ```terminal
   $ oc set env deploy/external-secrets-operator-controller-manager -n <external_secrets_operator_namespace> --list | grep -e OPERAND_ -e container
   ```

   ```terminal title="Example output"
   $ deployments/external-secrets-operator-controller-manager, container manager
   OPERAND_EXTERNAL_SECRETS_ARGS=--concurrent=2,--loglevel=debug
   OPERAND_WEBHOOK_ARGS=--tls-ciphers=TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256,--loglevel=debug
   ```
2. Verify that the operand `Deployment` container arguments were updated by running the following command:
   ```terminal
   $ oc get deploy/external-secrets-webhook -n <operand_namespace> -o jsonpath='{.spec.template.spec.containers[0].args}'
   ```

   ```terminal title="Example output"
   $ ["webhook","--dns-name=external-secrets-webhook.external-secrets.svc","--port=10250","--cert-dir=/tmp/certs","--check-interval=15m0s","--metrics-addr=:8080","--healthz-addr=:8081","--loglevel=debug","--zap-time-encoding=epoch","--tls-ciphers=TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256"]
   ```
