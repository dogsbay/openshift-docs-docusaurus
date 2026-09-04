---
title: Installing Operators in your namespace
sidebar_position: 2
---

# Installing Operators in your namespace {#olm-installing-operators-in-namespace}

<a id="olm-installing-operators-in-namespace"></a>

If a cluster administrator has delegated Operator installation permissions to your account, you can install and subscribe an Operator to your namespace in a self-service manner.

## Prerequisites {#olm-installing-operators-in-namespace-prereqs}

- A cluster administrator must add certain permissions to your OpenShift Container Platform user account to allow self-service Operator installation to a namespace.

## About Operator installation from the software catalog {#olm-installing-operators-from-software-catalog_olm-installing-operators-in-namespace}

The software catalog in OpenShift Container Platform is the interface for discovering Operators that Operator Lifecycle Manager (OLM) installs and manages on your cluster. You can choose installation settings such as install mode, namespace, and approval strategy during subscription.

As a user with the proper permissions, you can install an Operator from the software catalog by using the OpenShift Container Platform web console or CLI.

During installation, you must determine the following initial settings for the Operator:

Installation Mode:: Choose a specific namespace in which to install the Operator.

Update Channel
: If an Operator is available through multiple channels, you can choose which channel you want to subscribe to. For example, to deploy from the **stable** channel, if available, select it from the list.

Approval Strategy
: You can choose automatic or manual updates.

  If you choose automatic updates for an installed Operator, when a new version of that Operator is available in the selected channel, Operator Lifecycle Manager (OLM) automatically upgrades the running instance of your Operator without human intervention.

  If you select manual updates, when a newer version of an Operator is available, OLM creates an update request. As a cluster administrator, you must then manually approve that update request to have the Operator updated to the new version.

**Additional resources**

- [Allowing non-cluster administrators to install Operators](/docs/operators/admin/olm-creating-policy#olm-creating-policy)
- [Understanding the software catalog](/docs/operators/understanding/olm-understanding-software-catalog#olm-understanding-software-catalog)

## Installing from the software catalog by using the web console {#olm-installing-from-software-catalog-using-web-console_olm-installing-operators-in-namespace}

To install and subscribe to an Operator from the software catalog, you can use the OpenShift Container Platform web console. The console guides you through selecting an install mode, namespace, and approval strategy.

**Prerequisites**

- Access to an OpenShift Container Platform cluster using an account with `cluster-admin` permissions.
- Access to an OpenShift Container Platform cluster using an account with Operator installation permissions.

**Procedure**

1. Navigate in the web console to the **Ecosystem** → **Software Catalog** page.
2. Scroll or type a keyword into the **Filter by keyword** box to find the Operator you want. For example, type `advanced` to find the Advanced Cluster Management for Kubernetes Operator.
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
   3. Choose a specific, single namespace in which to install the Operator. The Operator will only watch and be made available for use in this single namespace.
   4. For clusters on cloud providers with token authentication enabled:
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

## Installing from the software catalog by using the CLI {#olm-installing-operator-from-software-catalog-using-cli_olm-installing-operators-in-namespace}

To install an Operator from the software catalog without using the web console, you can create or update a `Subscription` object by using the `oc` command in OpenShift Container Platform.

For `SingleNamespace` install mode, you must also ensure an appropriate Operator group exists in the related namespace. An Operator group, defined by an `OperatorGroup` object, selects target namespaces in which to generate required RBAC access for all Operators in the same namespace as the Operator group.

:::tip

In most cases, the web console method of this procedure is preferred because it automates tasks in the background, such as handling the creation of `OperatorGroup` and `Subscription` objects automatically when choosing `SingleNamespace` mode.

:::

**Prerequisites**

- Access to your OpenShift Container Platform cluster using an account with Operator installation permissions.
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

- [Operator groups](/docs/operators/understanding/olm/olm-understanding-olm#olm-operatorgroups-about_olm-understanding-olm)
- [Channel names](/docs/operators/understanding/olm/olm-understanding-olm#olm-subscription_olm-understanding-olm)

**Additional resources**

- [Manually approving a pending Operator update](/docs/operators/admin/olm-upgrading-operators#olm-approving-pending-upgrade_olm-upgrading-operators)
