---
title: Integrating the cert-manager Operator for Red Hat OpenShift with Istio-CSR
sidebar_position: 10
---

# Integrating the cert-manager Operator for Red Hat OpenShift with Istio-CSR {#cert-manager-operator-integrating-istio}

<a id="cert-manager-operator-integrating-istio"></a>

The cert-manager Operator for Red Hat OpenShift provides enhanced support for securing workloads and control plane components in Red Hat OpenShift Service Mesh or Istio. This includes support for certificates enabling mutual TLS (mTLS), which are signed, delivered, and renewed using cert-manager issuers. You can secure Istio workloads and control plane components by using the cert-manager Operator for Red Hat OpenShift managed Istio-CSR agent.

With this Istio-CSR integration, Istio can now obtain certificates from the cert-manager Operator for Red Hat OpenShift, simplifying security and certificate management.

## Creating a root CA issuer for the Istio-CSR agent {#cert-manager-istio-creating-issuer_cert-manager-operator-integrating-istio}

To enable certificate signing for the Istio-CSR agent, configure a root CA issuer using the cert-manager Operator for Red Hat OpenShift. You can establish a trusted root by using the cert-manager Operator for Red Hat OpenShift to ensure secure communication between workloads.

:::note

Other supported issuers can be used, except for the ACME issuer, which is not supported. For more information, see "cert-manager Operator for Red Hat OpenShift issuer providers".

:::

**Procedure**

1. Create a YAML file that defines the `Issuer` and `Certificate` objects by using the following example configuration:
   ```yaml
   apiVersion: cert-manager.io/v1
   kind: Issuer
   metadata:
     name: selfsigned
     namespace: <istio_project_name>
   spec:
     selfSigned: {}
   ---
   apiVersion: cert-manager.io/v1
   kind: Certificate
   metadata:
     name: istio-ca
     namespace: <istio_project_name>
   spec:
     isCA: true
     duration: 87600h # 10 years
     secretName: istio-ca
     commonName: istio-ca
     privateKey:
       algorithm: ECDSA
       size: 256
     subject:
       organizations:
         - cluster.local
         - cert-manager
     issuerRef:
       name: selfsigned
       kind: Issuer
       group: cert-manager.io
   ---
   apiVersion: cert-manager.io/v1
   kind: Issuer
   metadata:
     name: istio-ca
     namespace: <istio_project_name>
   spec:
     ca:
       secretName: istio-ca
   ```

   where:

   <dl>
   <dt><code>Issuer</code></dt>
   <dd>Specifies the <code>Issuer</code> or <code>ClusterIssuer</code>.</dd>
   <dt><code>&lt;istio_project_name&gt;</code></dt>
   <dd>Specifies the name of the Istio project.</dd>
   </dl>

**Verification**

- Verify that the Issuer is created and ready to use by running the following command:
  ```terminal
  $ oc get issuer istio-ca -n <istio_project_name>
  ```

  ```terminal title="Example output"
  NAME       READY   AGE
  istio-ca   True    3m
  ```

**Additional resources**

- [cert-manager Operator for Red Hat OpenShift issuer providers](/docs/security/cert_manager_operator#cert-manager-issuer-types_cert-manager-operator-about)

## Creating the `IstioCSR` custom resource {#cert-manager-istio-csr-installing_cert-manager-operator-integrating-istio}

To secure your communications, install the Istio-CSR agent by creating the `IstioCSR` custom resource through the cert-manager Operator for Red Hat OpenShift.

**Prerequisites**

- You have access to the cluster with `cluster-admin` privileges.
- You have enabled the Istio-CSR feature.
- You have created the `Issuer` or `ClusterIssuer` resources required for generating certificates for the Istio-CSR agent.
  :::note

  If you are using `Issuer` resource, create the `Issuer` and `Certificate` resources in the Red Hat OpenShift Service Mesh or `Istiod` namespace. Certificate requests are generated in the same namespace, and role-based access control (RBAC) is configured accordingly.

  :::

**Procedure**

1. Create a new project for installing Istio-CSR by running the following command. If you have an existing project for installing Istio-CSR, skip this step.
   ```terminal
   $ oc new-project <istio_csr_project_name>
   ```
2. Create the `IstioCSR` custom resource to enable Istio-CSR agent managed by the cert-manager Operator for Red Hat OpenShift for processing Istio workload and control plane certificate signing requests.
   :::note

   Only one `IstioCSR` custom resource (CR) is supported at a time. If multiple `IstioCSR` CRs are created, only one will be active. Use the `status` sub-resource of `IstioCSR` to check if a resource is unprocessed.

   - If multiple `IstioCSR` CRs are created simultaneously, none will be processed.
   - If multiple `IstioCSR` CRs are created sequentially, only the first one will be processed.
   - To prevent new requests from being rejected, delete any unprocessed `IstioCSR` CRs.
   - The Operator does not automatically remove objects created for `IstioCSR`. If an active `IstioCSR` resource is deleted and a new one is created in a different namespace without removing the previous deployments, multiple `istio-csr` deployments may remain active. This behavior is not recommended and is not supported.

   :::

   1. Create a YAML file that defines the `IstioCSR` object by using the following example:
      ```yaml
      apiVersion: operator.openshift.io/v1alpha1
      kind: IstioCSR
      metadata:
        name: default
        namespace: <istio_csr_project_name>
      spec:
        istioCSRConfig:
          certManager:
            issuerRef:
              name: istio-ca
              kind: Issuer
              group: cert-manager.io
          istiodTLSConfig:
            trustDomain: cluster.local
          istio:
            namespace: <istio_project_name>
      ```

      where:

      <dl>
      <dt><code>name</code></dt>
      <dd>Specifies the <code>Issuer</code> or <code>ClusterIssuer</code> name. It should be the same name as the CA issuer defined in the <code>issuer.yaml</code> file.</dd>
      <dt><code>kind</code></dt>
      <dd>Specifies the <code>Issuer</code> or <code>ClusterIssuer</code> kind. It should be the same kind as the CA issuer defined in the <code>issuer.yaml</code> file.</dd>
      </dl>
   2. Create the `IstioCSR` custom resource by running the following command:
      ```terminal
      $ oc create -f IstioCSR.yaml
      ```

**Verification**

1. Verify that the Istio-CSR deployment is ready by running the following command:
   ```terminal
   $ oc get deployment -n <istio_csr_project_name>
   ```

   ```terminal title="Example output"
   NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
   cert-manager-istio-csr   1/1     1            1           24s
   ```
2. Verify that the Istio-CSR pods are running by running the following command:
   ```terminal
   $ oc get pod -n <istio_csr_project_name>
   ```

   ```terminal title="Example output"
   NAME                                  	 READY   STATUS	  RESTARTS    AGE
   cert-manager-istio-csr-5c979f9b7c-bv57w  1/1     Running  0           45s
   ```

   - Verify that the Istio-CSR pod is not reporting any errors in the logs by running the following command:
     ```terminal
     $ oc -n <istio_csr_project_name> logs <istio_csr_pod_name>
     ```
   - Verify that the cert-manager Operator for Red Hat OpenShift pod is not reporting any errors by running the following command:
     ```terminal
     $ oc -n cert-manager-operator logs <cert_manager_operator_pod_name>
     ```

## Setting the log level for the istio-csr component {#cert-manager-istio-csr-setting-log-level_cert-manager-operator-integrating-istio}

You can set the log level for the istio-csr component to control the verbosity and format of its log messages.

**Prerequisites**

- You have access to the cluster with `cluster-admin` privileges.
- You have created the `IstioCSR` custom resource (CR).

**Procedure**

1. Edit the `IstioCSR` CR by running the following command:
   ```terminal
   $ oc edit istiocsrs.operator.openshift.io default -n <istio_csr_project_name> (1)
   ```

   Replace `<istio_csr_project_name>` with the namespace where you created the `IstioCSR` CR.
2. Configure the log level and format in the `spec.istioCSRConfig` section by using the following example configuration:
   ```yaml
   apiVersion: operator.openshift.io/v1alpha1
   kind: IstioCSR
   ...
   spec:
     istioCSRConfig:
       logFormat: text
       logLevel: 2
   # ...
   ```

   where:

   <dl>
   <dt><code>istioCSRConfig.logFormat</code></dt>
   <dd>Specifies the log output format. You can set this field to either <code>text</code> or <code>json</code>.</dd>
   <dt><code>istioCSRConfig.logLevel</code></dt>
   <dd>Specifies the log level. Supported values are in the range <code>1</code> through <code>5</code>, as defined by Kubernetes logging guidelines. The default value is <code>1</code>.</dd>
   </dl>
3. Save and close the editor to apply your changes. After the changes are applied, the cert-manager Operator updates the log configuration for the istio-csr operand.

## Configuring the namespace selector for CA bundle distribution {#cert-manager-istio-csr-config-namespace-sel_cert-manager-operator-integrating-istio}

The Istio-CSR agent creates and updates the `istio-ca-root-cert` `ConfigMap`, which contains the CA bundle. Workloads in the service mesh use this CA bundle to validate connections to the Istio control plane. You can configure a namespace selector to specify the namespaces in which the Istio-CSR agent creates this `ConfigMap`. If you do not configure a selector, the Istio-CSR agent creates the `ConfigMap` in all namespaces.

**Prerequisites**

- You have access to the cluster with `cluster-admin` privileges.
- You have created the `IstioCSR` custom resource (CR).

**Procedure**

1. Edit the `IstioCSR` CR by running the following command:
   ```terminal
   oc edit istiocsrs.operator.openshift.io default -n <istio_csr_project_name>
   ```

   Replace `<istio_csr_project_name>` with the namespace where you created the `IstioCSR` CR.
2. Configure the `spec.istioCSRConfig.istioDataPlaneNamespaceSelector` section to set the namespace selector. See the following example:
   ```yaml
   apiVersion: operator.openshift.io/v1alpha1
   kind: IstioCSR
   ...
   spec:
     istioCSRConfig:
       istioDataPlaneNamespaceSelector: maistra.io/member-of=istio-system
   # ...
   ```

   The `maistra.io/member-of=istio-system` namespace selector defines the label key and value that identify the namespaces in your service mesh. Use the `<key>=<value>` format.

   :::note

   The istio-csr component does not delete or manage `ConfigMap` objects in namespaces that do not match the configured selector. If you create or update the selector after deploying the `IstioCSR` CR, or if you remove a label from a namespace, you must manually delete these `ConfigMap` objects to avoid conflicts.

   You can run the following command to list `ConfigMap` objects that are not in namespaces matching the selector. In this example, the selector is `maistra.io/member-of=istio-system`:

   ```terminal
   printf "%-25s %10s\n" "ConfigMap" "Namespace"; \
   for ns in $(oc get namespaces -l "maistra.io/member-of!=istio-system" -o=jsonpath='{.items[*].metadata.name}'); do \
     oc get configmaps -l "istio.io/config=true" -n $ns --no-headers -o jsonpath='{.items[*].metadata.name}{"\t"}{.items[*].metadata.namespace}{"\n"}' --ignore-not-found; \
   done
   ```

   :::
3. Save and close the editor to apply your changes. After the changes are applied, the cert-manager Operator for Red Hat OpenShift updates the namespace selector configuration for the istio-csr operand.

## Configuring the CA certificate for the Istio server {#cert-manager-istio-csr-config-ca-cert_cert-manager-operator-integrating-istio}

You can configure the `ConfigMap` that contains the CA bundle used by Istio workloads to verify the Istio server certificate. If not configured, the cert-manager Operator for Red Hat OpenShift looks for the CA certificate in the configured issuer and in the Kubernetes Secret that contains the Istio certificates.

**Prerequisites**

- You have access to the cluster with `cluster-admin` privileges.
- You have created the `IstioCSR` custom resource (CR).

**Procedure**

1. Edit the `IstioCSR` CR by running the following command:
   ```terminal
   $ oc edit istiocsrs.operator.openshift.io default -n <istio_csr_project_name>
   ```

   Replace `<istio_csr_project_name>` with the namespace where you created the `IstioCSR` CR.
2. Configure the CA bundle by editing the `spec.istioCSRConfig.certManager` section. See the following example:
   ```yaml
   apiVersion: operator.openshift.io/v1alpha1
   kind: IstioCSR
   ...
   spec:
     istioCSRConfig:
       certManager:
         istioCACertificate:
           key: <key_in_the_configmap>
           name: <configmap_name>
           namespace: <configmap_namespace>
   ```

   where:

   <dl>
   <dt><code>&lt;key_in_the_configmap&gt;</code></dt>
   <dd>Specifies the key name in the <code>ConfigMap</code> that contains the CA bundle.</dd>
   <dt><code>&lt;configmap_name&gt;</code></dt>
   <dd>Specifies the name of the <code>ConfigMap</code>. Ensure that the referenced <code>ConfigMap</code> and key exist before you update this field.</dd>
   <dt><code>&lt;configmap_namespace&gt;</code></dt>
   <dd>Optional. Specifies the namespace where the <code>ConfigMap</code> exists. If you do not set this field, the cert-manager Operator for Red Hat OpenShift searches for the <code>ConfigMap</code> in the namespace where you have installed the <code>IstioCSR</code> CR.</dd>
   </dl>

   :::note

   Whenever the CA certificate is rotated, you must manually update the `ConfigMap` with the latest certificate.

   :::
3. Save and close the editor to apply your changes. After the changes are applied, the cert-manager Operator updates the CA bundle for the `istio-csr` operand.

## Uninstalling the Istio-CSR agent managed by cert-manager Operator for Red Hat OpenShift {#cert-manager-istio-csr-uninstalling_cert-manager-operator-integrating-istio}

You can uninstall the Istio-CSR agent managed by the cert-manager Operator for Red Hat OpenShift to remove the agent and its associated resources.

**Prerequisites**

- You have access to the cluster with `cluster-admin` privileges.
- You have enabled the Istio-CSR feature.
- You have created the `IstioCSR` custom resource.

**Procedure**

1. Remove the `IstioCSR` custom resource by running the following command:
   ```terminal
   $ oc -n <istio_csr_project_name> delete istiocsrs.operator.openshift.io default
   ```
2. Remove related resources:
   :::warning

   To avoid disrupting any Red Hat OpenShift Service Mesh or Istio components, ensure that no component is referencing the Istio-CSR service or the certificates issued for Istio before removing the following resources.

   :::

   1. List the cluster scoped-resources by running the following command and save the names of the listed resources for later reference:
      ```terminal
      $ oc get clusterrolebindings,clusterroles -l "app=cert-manager-istio-csr,app.kubernetes.io/name=cert-manager-istio-csr"
      ```
   2. List the resources in Istio-csr deployed namespace by running the following command and save the names of the listed resources for later reference:
      ```terminal
      $ oc get certificate,deployments,services,serviceaccounts -l "app=cert-manager-istio-csr,app.kubernetes.io/name=cert-manager-istio-csr" -n <istio_csr_project_name>
      ```
   3. List the resources in Red Hat OpenShift Service Mesh or Istio deployed namespaces by running the following command and save the names of the listed resources for later reference:
      ```terminal
      $ oc get roles,rolebindings -l "app=cert-manager-istio-csr,app.kubernetes.io/name=cert-manager-istio-csr" -n <istio_csr_project_name>
      ```
   4. For each resource listed in previous steps, delete the resource by running the following command:
      ```terminal
      $ oc -n <istio_csr_project_name> delete <resource_type>/<resource_name>
      ```

      Repeat this process until all of the related resources have been deleted.
