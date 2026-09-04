---
title: Understanding DNS management policies
sidebar_position: 11
---

# Understanding DNS management policies {#ingress-controller-dnsmgt}

<a id="ingress-controller-dnsmgt"></a>

As a cluster administrator, when you create an Ingress Controller, the Operator manages the DNS records automatically. This approach has some limitations when the required DNS zone is different from the cluster DNS zone or when the DNS zone is hosted outside the cloud provider.

The following list details key aspects for a managed DNS management policy:

- The Managed DNS management policy for Ingress Controllers ensures that the lifecycle of the wildcard DNS record on the cloud provider is automatically managed by the Operator. This is the default behavior.
- When you change an Ingress Controller from `Managed` to `Unmanaged` DNS management policy, the Operator does not clean up the previous wildcard DNS record provisioned on the cloud.
- When you change an Ingress Controller from `Unmanaged` to `Managed` DNS management policy, the Operator attempts to create the DNS record on the cloud provider if it does not exist or updates the DNS record if it already exists.

The following list details key aspects for a unmanaged DNS management policy:

- The Unmanaged DNS management policy for Ingress Controllers ensures that the lifecycle of the wildcard DNS record on the cloud provider is not automatically managed; instead, it becomes the responsibility of the cluster administrator.
  :::note

  For Google Cloud installations, you can use a custom DNS solution. Refer to the `DNSRecord` CR for information on what you need to include in the DNS record. For more information, see [Enabling a user-managed DNS](/docs/installing/installing_gcp/installing-gcp-customizations#installation-gcp-enabling-user-managed-DNS_installing-gcp-customizations) and [Provisioning your own DNS records](/docs/installing/installing_gcp/installing-gcp-customizations#installation-gcp-provisioning-own-dns-records_installing-gcp-customizations).

  :::

## Creating an Ingress Controller for manual DNS management {#creating-a-custom-ingress-controller_ingress-controller-dnsmgt}

As a cluster administrator, you can create a new custom Ingress Controller with the Unmanaged DNS management policy.

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).
- You are logged in as a user with `cluster-admin` privileges.

**Procedure**

1. Create an `IngressController` custom resource (CR) file named `sample-ingress.yaml` with the following content:
   ```yaml
   apiVersion: operator.openshift.io/v1
   kind: IngressController
   metadata:
     namespace: openshift-ingress-operator
     name: <name>
   spec:
     domain: <domain>
     endpointPublishingStrategy:
       type: LoadBalancerService
       loadBalancer:
         scope: External
         dnsManagementPolicy: Unmanaged
   ```

   where:

   <dl>
   <dt><code>metadata.name</code></dt>
   <dd>Specify the <code>&lt;name&gt;</code> with a name for the <code>IngressController</code> object.</dd>
   <dt><code>spec.domain</code></dt>
   <dd>Specify the <code>domain</code> based on the DNS record that was created as a prerequisite.</dd>
   <dt><code>loadBalancer.scope</code></dt>
   <dd>Specify the <code>scope</code> as <code>External</code> to expose the load balancer externally. <code>loadBalancer.dnsManagementPolicy</code>: Specifies if the Ingress Controller is managing the lifecycle of the wildcard DNS record associated with the load balancer. The valid values are <code>Managed</code> and <code>Unmanaged</code>. The default value is <code>Managed</code>.</dd>
   </dl>
2. Apply the manifest to create the `IngressController` object:
   ```terminal
   $ oc apply -f sample-ingress.yaml
   ```
3. Verify that the Ingress Controller was created with the correct policy by running the following command:
   ```terminal
   $ oc get ingresscontroller <name> -n openshift-ingress-operator -o=jsonpath={.spec.endpointPublishingStrategy.loadBalancer}
   ```

   Inspect the output and confirm that `dnsManagementPolicy` is set to `Unmanaged`.

## Modifying an existing Ingress Controller for manual DNS management {#modifying-an-existing-ingress-controller_ingress-controller-dnsmgt}

As a cluster administrator, you can modify an existing Ingress Controller to manually manage the DNS record lifecycle.

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).
- You are logged in as a user with `cluster-admin` privileges.

**Procedure**

1. Modify the chosen Ingress Controller to set the `dnsManagementPolicy` parameter:
   ```terminal
   $ SCOPE=$(oc -n openshift-ingress-operator get ingresscontroller <name> -o=jsonpath="{.status.endpointPublishingStrategy.loadBalancer.scope}")
   ```

   ```terminal
   $ oc -n openshift-ingress-operator patch ingresscontrollers/default --type=merge --patch="{\"spec\":{\"endpointPublishingStrategy\":{\"type\":\"LoadBalancerService\",\"loadBalancer\":{\"dnsManagementPolicy\":\"Unmanaged\", \"scope\":\"${SCOPE}\"}}}}"
   ingresscontroller.operator.openshift.io/default patched
   ```
2. Verify that the Ingress Controller was modified correctly by running the following command:
   ```terminal
   $ oc get ingresscontroller <name> -n openshift-ingress-operator -o=jsonpath={.spec.endpointPublishingStrategy.loadBalancer}
   ```

   Inspect the output and confirm that `dnsManagementPolicy` is set to `Unmanaged`.

**Additional resources**

- [Ingress Controller configuration parameters](/docs/networking/networking_operators/ingress-operator#nw-ingress-controller-configuration-parameters_configuring-ingress)
