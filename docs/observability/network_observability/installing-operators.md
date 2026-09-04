---
title: Installing the Network Observability Operator
sidebar_position: 3
---

# Installing the Network Observability Operator {#installing-network-observability-operators}

<a id="installing-network-observability-operators"></a>

Installing the Loki Operator is recommended before using the Network Observability Operator. You can use network observability without Loki, but special considerations apply if you only need metrics or external exporters.

The Loki Operator integrates a gateway that implements multi-tenancy and authentication with Loki for data flow storage. The `LokiStack` resource manages Loki, which is a scalable, highly-available, multi-tenant log aggregation system, and a web proxy with OpenShift Container Platform authentication. The `LokiStack` proxy uses OpenShift Container Platform authentication to enforce multi-tenancy and facilitate the saving and indexing of data in Loki log stores.

## Network observability without Loki {#network-observability-without-loki_network_observability}

Compare the features available with network observability with and without installing the Loki Operator.

If you only want to export flows to a Kafka consumer or IPFIX collector, or you only need dashboard metrics, then you do not need to install Loki or provide storage for Loki. The following table compares available features with and without Loki.

**Comparison of feature availability with and without Loki**

|  | **With Loki** | **Without Loki** |
| --- | --- | --- |
| **Exporters** | X | X |
| **Multi-tenancy** | X | X |
| **Complete filtering and aggregations capabilities** <sup>[1]</sup> | X |  |
| **Partial filtering and aggregations capabilities** <sup>[2]</sup> | X | X |
| **Flow-based metrics and dashboards** | X | X |
| **Traffic flows view overview** <sup>[3]</sup> | X | X |
| **Traffic flows view table** | X |  |
| **Topology view** | X | X |
| **OpenShift Container Platform console Network Traffic tab integration** | X | X |

1. Such as per pod.
2. Such as per workload or namespace.
3. Statistics on packet drops are only available with Loki.

**Additional resources**

- [Export enriched network flow data](/docs/observability/network_observability/configuring-operator#network-observability-enriched-flows_network_observability)

## Installing the Loki Operator {#network-observability-loki-installation_network_observability}

Install the supported Loki Operator version from the software catalog to enable the secure `LokiStack` instance, which provides automatic in-cluster authentication and authorization for network observability.

The [Loki Operator versions 6.0+](https://catalog.redhat.com/software/containers/openshift-logging/loki-rhel9-operator/64479927e1820602a81cdf13) are the supported Loki Operator versions for network observability; these versions provide the ability to create a `LokiStack` instance using the `openshift-network` tenant configuration mode and provide fully-automatic, in-cluster authentication and authorization support for network observability.

**Prerequisites**

- You have administrator permissions.
- You have access to the OpenShift Container Platform web console.
- You have access to a supported object store. For example: AWS S3, Google Cloud Storage, Azure, Swift, Minio, or OpenShift Data Foundation.

**Procedure**

1. In the OpenShift Container Platform web console, click **Ecosystem** → **Software Catalog**.
2. Choose  **Loki Operator** from the list of available Operators, and click **Install**.
3. Under **Installation Mode**, select **All namespaces on the cluster**.

**Verification**

1. Verify that you installed the Loki Operator. Visit the **Ecosystem** → **Installed Operators** page and look for **Loki Operator**.
2. Verify that **Loki Operator** is listed with **Status** as **Succeeded** in all the projects.

:::warning

To uninstall Loki, refer to the uninstallation process that corresponds with the method you used to install Loki. You might have remaining `ClusterRoles` and `ClusterRoleBindings`, data stored in object store, and persistent volume that must be removed.

:::

### Creating a secret for Loki storage {#network-observability-loki-secret_network_observability}

Create a secret with cloud storage credentials, such as for Amazon Web Services (AWS), to allow the Loki Operator to access the necessary object store for log persistence.

The Loki Operator supports a few log storage options, such as AWS S3, Google Cloud Storage, Azure, Swift, Minio, OpenShift Data Foundation. The following example shows how to create a secret for AWS S3 storage. The secret created in this example, `loki-s3`, is referenced in "Creating a LokiStack custom resource". You can create this secret in the web console or CLI.

**Procedure**

1. Using the web console, navigate to the **Project** → **All Projects** dropdown and select **Create Project**.
2. Name the project `netobserv-loki` and click **Create**.
3. Navigate to the Import icon, **+**, in the top right corner. Paste your YAML file into the editor.
   The following shows an example secret YAML file for S3 storage:

   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: loki-s3
     namespace: netobserv-loki
   stringData:
     access_key_id: QUtJQUlPU0ZPRE5ON0VYQU1QTEUK
     access_key_secret: d0phbHJYVXRuRkVNSS9LN01ERU5HL2JQeFJmaUNZRVhBTVBMRUtFWQo=
     bucketnames: s3-bucket-name
     endpoint: https://s3.eu-central-1.amazonaws.com
     region: eu-central-1
   ```

   where:

   <dl>
   <dt><code>metadata.namespace</code></dt>
   <dd>Specifies the namespace for the Loki S3 secret. While this example uses <code>netobserv-loki</code>, you can use a different namespace for different components.</dd>
   <dt><code>stringData.access_key_id</code></dt>
   <dd>Specifies the access key ID for the S3 bucket.</dd>
   <dt><code>stringData.access_key_secret</code></dt>
   <dd>Specifies the secret access key for the S3 bucket.</dd>
   <dt><code>stringData.bucketnames</code></dt>
   <dd>Specifies the name of the S3 bucket.</dd>
   <dt><code>stringData.endpoint</code></dt>
   <dd>Specifies the endpoint URL for the S3 service.</dd>
   <dt><code>stringData.region</code></dt>
   <dd>Specifies the AWS region where the bucket is located.</dd>
   </dl>

**Verification**

- After you create the secret, you view the secret listed under **Workloads** → **Secrets** in the web console.

### Creating a LokiStack custom resource {#network-observability-lokistack-create_network_observability}

Deploy the `LokiStack` custom resource using the web console or OpenShift CLI (`oc`), ensuring you configure the correct namespace, deployment size, and secret name for Loki object storage.

You can deploy a `LokiStack` custom resource (CR) to create a namespace or new project.

**Procedure**

1. Navigate to **Ecosystem** → **Installed Operators**, viewing **All projects** from the **Project** dropdown.
2. Look for **Loki Operator**. In the details, under **Provided APIs**, select **LokiStack**.
3. Click **Create LokiStack**.
4. Ensure the following fields are specified in either **Form View** or **YAML view**:
   ```yaml
   apiVersion: loki.grafana.com/v1
   kind: LokiStack
   metadata:
     name: loki
     namespace: netobserv-loki
   spec:
     size: 1x.small
     storage:
       schemas:
       - version: v13
         effectiveDate: '2022-06-01'
       secret:
         name: loki-s3
         type: s3
     storageClassName: gp3
     tenants:
       mode: openshift-network
   ```

   where:

   <dl>
   <dt><code>metadata.namespace</code></dt>
   <dd>Specifies the namespace for the <code>LokiStack</code> resource. While this example uses <code>netobserv-loki</code>, you can use a different namespace for different components.</dd>
   <dt><code>spec.size</code></dt>
   <dd>Specifies the deployment size. In Loki Operator 5.8 and later versions, the supported size options for production instances of Loki are <code>1x.extra-small</code>, <code>1x.small</code>, or <code>1x.medium</code>.</dd>
   </dl>

   :::warning

   It is not possible to change the number `1x` for the deployment size.

   :::

   <dl>
   <dt><code>spec.storageClassName</code></dt>
   <dd>Specifies a storage class name that is available on the cluster for <code>ReadWriteOnce</code> access mode. For best performance, specify a storage class that allocates block storage. Use the <code>oc get storageclasses</code> command to see available storage classes on your cluster.</dd>
   </dl>

   :::warning

   You must not reuse the same `LokiStack` custom resource that is used for logging.

   :::
5. Click **Create**.

### Role-based access control for Loki logs {#network-observability-role-based-access-control-for-loki-logs_network_observability}

Configure role-based access control to grant users permission to view application, infrastructure, or audit logs in Loki.

By default, logging 5.8 and later does not grant users access to logs. You must configure role-based access control to grant users permission to view specific log types.

For more information on access control for Loki logs, see: "Fine grained access for Loki logs" in the Red Hat OpenShift Logging Operator documentation.

#### Grant non-admin users cluster-wide log access {#grant-non-admin-users-cluster-wide-log-access_network_observability}

Add users to a custom admin group to grant cluster-wide log access without making them cluster administrators. This is useful for senior engineers who need full log visibility but should not have cluster modification privileges.

Users who are members of any group specified in the `adminGroups` field of the `LokiStack` custom resource (CR) have the same read access to logs as administrators.

```yaml title="Example LokiStack CR"
apiVersion: loki.grafana.com/v1
kind: LokiStack
metadata:
  name: loki
  namespace: netobserv-loki
spec:
  tenants:
    mode: openshift-network
    openshift:
      adminGroups:
      - cluster-admin
      - custom-admin-group
```

where:

`spec.tenants.mode`
: Specifies the tenant mode. Must be `openshift-network` for network observability.

`spec.tenants.openshift.adminGroups`
: Specifies the list of groups whose members have cluster-wide log access. Defaults to `system:cluster-admins`, `cluster-admin`, and `dedicated-admin`. Set to `[]` to disable.

**Additional resources**

- [Fine grained access for Loki logs](https://docs.redhat.com/en/documentation/red_hat_openshift_logging/6.5/html/configuring_logging/configuring-lokistack-storage#logging-loki-log-access_configuring-the-log-store)

### LokiStack ingestion limits and health alerts {#network-observability-lokistack-configuring-ingestion_network_observability}

The `LokiStack` instance includes default ingestion and query limits that can be overridden by administrators to manage performance and prevent system alerts or errors.

:::note

You might want to update the ingestion and query limits if you get Loki errors showing up in the Console plugin, or in `flowlogs-pipeline` logs.

:::

Here is an example of configured limits:

```yaml
spec:
  limits:
    global:
      ingestion:
        ingestionBurstSize: 40
        ingestionRate: 20
        maxGlobalStreamsPerTenant: 25000
      queries:
        maxChunksPerQuery: 2000000
        maxEntriesLimitPerQuery: 10000
        maxQuerySeries: 3000
```

For more information about these settings, see "LokiStack API reference".

**Additional resources**

- [Creating a LokiStack custom resource](/docs/observability/network_observability/installing-operators#network-observability-lokistack-create_network_observability)
- [Loki object storage](https://docs.redhat.com/en/documentation/red_hat_openshift_logging/latest/html/configuring_logging/configuring-lokistack-storage#logging-loki-storage_configuring-the-log-store)
- [Loki deployment sizing](https://docs.redhat.com/en/documentation/red_hat_openshift_logging/latest/html/configuring_logging/configuring-lokistack-storage.html#loki-sizing_configuring-the-log-store)
- [LokiStack API reference](https://loki-operator.dev/docs/api.md/#loki-grafana-com-v1-IngestionLimitSpec)
- [Flow Collector API Reference](/docs/observability/network_observability/flowcollector-api#network-observability-flowcollector-api-specifications_network_observability)
- [Flow Collector sample resource](/docs/observability/network_observability/configuring-operator#network-observability-flowcollector-view_network_observability)

## Installing the Network Observability Operator {#network-observability-operator-installation_network_observability}

Install the Network Observability Operator and use the setup wizard to create the `FlowCollector` custom resource definition (CRD) to complete the initial configuration.

You can set specifications in the web console when you create the `FlowCollector`.

:::warning

The actual memory consumption of the Operator depends on your cluster size and the number of resources deployed. Memory consumption might need to be adjusted accordingly. For more information refer to "Network Observability controller manager pod runs out of memory" in the "Important Flow Collector configuration considerations" section.

:::

**Prerequisites**

- If you choose to use Loki, install the [Loki Operator version 5.7+](https://catalog.redhat.com/software/containers/openshift-logging/loki-rhel8-operator/622b46bcae289285d6fcda39).
- You must have `cluster-admin` privileges.
- One of the following supported architectures is required: `amd64`, `ppc64le`, `arm64`, or `s390x`.
- Any CPU supported by Red Hat Enterprise Linux (RHEL) 9.
- Must be configured with OVN-Kubernetes as the main network plugin, and optionally using secondary interfaces with Multus and SR-IOV.

:::note

Additionally, this installation example uses the `netobserv` namespace, which is used across all components. You can optionally use a different namespace.

:::

**Procedure**

1. In the OpenShift Container Platform web console, click **Ecosystem** → **Software Catalog**.
2. Choose  **Network Observability Operator** from the list of available Operators in the software catalog, and click **Install**.
3. Select the checkbox `Enable Operator recommended cluster monitoring on this Namespace`.
4. Navigate to **Operators** → **Installed Operators**. Under Provided APIs for Network Observability, select the **Flow Collector** link.
5. Follow the **Network Observability FlowCollector setup** wizard.
6. Click **Create**.

**Verification**

To confirm this was successful, when you navigate to **Observe** you should see **Network Traffic** listed in the options.

In the absence of **Application Traffic** within the OpenShift Container Platform cluster, default filters might show that there are "No results", which results in no visual flow. Beside the filter selections, select **Clear all filters** to see the flow.

### Important FlowCollector configuration considerations {#network-observability-important-flowcollector-configuration-considerations_network_observability}

Review essential `FlowCollector` configuration options before initial deployment to avoid pod disruptions caused by later reconfiguration. Key settings include Kafka integration, enriched flow data exports, SR-IOV traffic monitoring, and advanced tracking for DNS and packet drops.

Once you create the `FlowCollector` instance, you can reconfigure it, but the pods are terminated and recreated again, which can be disruptive.

Therefore, you can consider configuring the following options when creating the `FlowCollector` for the first time.

**Additional resources**

- [Configuring the Flow Collector resource with Kafka](/docs/observability/network_observability/configuring-operator#network-observability-flowcollector-kafka-config_network_observability)
- [Export enriched network flow data to Kafka or IPFIX](/docs/observability/network_observability/configuring-operator#network-observability-enriched-flows_network_observability)
- [Configuring monitoring for SR-IOV interface traffic](/docs/observability/network_observability/network-observability-secondary-networks#network-observability-SR-IOV-config_network-observability-secondary-networks)
- [Working with conversation tracking](/docs/observability/network_observability/observing-network-traffic#network-observability-working-with-conversations_nw-observe-network-traffic)
- [Working with DNS tracking](/docs/observability/network_observability/observing-network-traffic#network-observability-dns-tracking_nw-observe-network-traffic)
- [Working with packet drops](/docs/observability/network_observability/observing-network-traffic#network-observability-packet-drops_nw-observe-network-traffic)
- [Flow Collector API Reference](/docs/observability/network_observability/flowcollector-api#network-observability-flowcollector-api-specifications_network_observability)
- [Flow Collector sample resource](/docs/observability/network_observability/configuring-operator#network-observability-flowcollector-view_network_observability)
- [Resource considerations](/docs/observability/network_observability/configuring-operator#network-observability-resources-table_network_observability)
- [Troubleshooting network observability controller manager pod runs out of memory](/docs/observability/network_observability/troubleshooting-network-observability#controller-manager-pod-runs-out-of-memory_network-observability-troubleshooting)
- [Network observability architecture](/docs/observability/network_observability/understanding-network-observability-operator#network-observability-architecture_nw-network-observability-operator)

## Migrating removed stored versions of the FlowCollector CRD {#network-observability-updating-migrating_network_observability}

Manually remove the deprecated `v1alpha1` version from the `FlowCollector` custom resource definition (CRD) `storedVersion` list to prevent upgrade errors and successfully migrate to Network Observability Operator 1.6.

There are two options to remove stored versions:

1. Use the Storage Version Migrator Operator.
2. Uninstall and reinstall the Network Observability Operator, ensuring that the installation is in a clean state.

**Prerequisites**

- You have an older version of the Operator installed, and you want to prepare your cluster to install the latest version of the Operator. Or you have attempted to install the Network Observability Operator 1.6 and run into the error: `Failed risk of data loss updating "flowcollectors.flows.netobserv.io": new CRD removes version v1alpha1 that is listed as a stored version on the existing CRD`.

**Procedure**

1. Verify that the old `FlowCollector` CRD version is still referenced in the `storedVersion`:
   ```terminal
   $ oc get crd flowcollectors.flows.netobserv.io -ojsonpath='{.status.storedVersions}'
   ```
2. If `v1alpha1` appears in the list of results, proceed with **Step a** to use the Kubernetes Storage Version Migrator or **Step b** to uninstall and reinstall the CRD and the Operator.
   1. **Option 1: Kubernetes Storage Version Migrator**: Create a YAML to define the `StorageVersionMigration` object, for example `migrate-flowcollector-v1alpha1.yaml`:
      ```yaml
      apiVersion: migration.k8s.io/v1alpha1
      kind: StorageVersionMigration
      metadata:
        name: migrate-flowcollector-v1alpha1
      spec:
        resource:
          group: flows.netobserv.io
          resource: flowcollectors
          version: v1alpha1
      ```

      1. Save the file.
      2. Apply the `StorageVersionMigration` by running the following command:
         ```terminal
         $ oc apply -f migrate-flowcollector-v1alpha1.yaml
         ```
      3. Update the `FlowCollector` CRD to manually remove `v1alpha1` from the `storedVersion`:
         ```terminal
         $ oc edit crd flowcollectors.flows.netobserv.io
         ```
   2. **Option 2: Reinstall**: Save the Network Observability Operator 1.5 version of the `FlowCollector` CR to a file, for example `flowcollector-1.5.yaml`.
      ```terminal
      $ oc get flowcollector cluster -o yaml > flowcollector-1.5.yaml
      ```

      1. Follow the steps in "Uninstalling the Network Observability Operator", which uninstalls the Operator and removes the existing `FlowCollector` CRD.
      2. Install the Network Observability Operator latest version, 1.6.0.
      3. Create the `FlowCollector` using backup that was saved in Step b.

**Verification**

- Run the following command:
  ```terminal
  $ oc get crd flowcollectors.flows.netobserv.io -ojsonpath='{.status.storedVersions}'
  ```

  The list of results should no longer show `v1alpha1` and only show the latest version, `v1beta1`.

## Enabling multi-tenancy in network observability {#network-observability-multi-tenancy_network_observability}

Enable multi-tenancy in network observability by configuring cluster roles and namespace roles to grant project administrators and developers granular, restricted access to flows and metrics in Loki and Prometheus.

Access is enabled for project administrators. Project administrators who have limited access to some namespaces can access flows for only those namespaces.

For Developers, multi-tenancy is available for both Loki and Prometheus but requires different access rights.

**Prerequisite**

- If you are using Loki, you have installed at least [Loki Operator version 5.7](https://catalog.redhat.com/software/containers/openshift-logging/loki-rhel8-operator/622b46bcae289285d6fcda39).
- You must be logged in as a project administrator.

**Procedure**

- For per-tenant access, you must have the `netobserv-loki-reader` cluster role and the `netobserv-metrics-reader` namespace role to use the developer perspective. Run the following commands for this level of access:
  ```terminal
  $ oc adm policy add-cluster-role-to-user netobserv-loki-reader <user_group_or_name>
  ```

  ```terminal
  $ oc adm policy add-role-to-user netobserv-metrics-reader <user_group_or_name> -n <namespace>
  ```
- For cluster-wide access, non-cluster-administrators must have the `netobserv-loki-reader`, `cluster-monitoring-view`, and `netobserv-metrics-reader` cluster roles. In this scenario, you can use either the admin perspective or the developer perspective. Run the following commands for this level of access:
  ```terminal
  $ oc adm policy add-cluster-role-to-user netobserv-loki-reader <user_group_or_name>
  ```

  ```terminal
  $ oc adm policy add-cluster-role-to-user cluster-monitoring-view <user_group_or_name>
  ```

  ```terminal
  $ oc adm policy add-cluster-role-to-user netobserv-metrics-reader <user_group_or_name>
  ```

**Additional resources**

- [Kubernetes Storage Version Migrator Operator](/docs/operators/operator-reference#cluster-kube-storage-version-migrator-operator_operator-reference)

## Uninstalling the Network Observability Operator {#network-observability-operator-uninstall_network_observability}

Uninstall the Network Observability Operator using the OpenShift Container Platform web console Operator Hub, working in the **Ecosystem** → **Installed Operators** area.

**Procedure**

1. Remove the `FlowCollector` custom resource.
   1. Click **Flow Collector**, which is next to the **Network Observability Operator** in the **Provided APIs** column.
   2. Click the Options menu ![](/images/kebab.png "Options menu") for the **cluster** and select **Delete FlowCollector**.
2. Uninstall the Network Observability Operator.
   1. Navigate back to the **Ecosystem** → **Installed Operators** area.
   2. Click the Options menu ![](/images/kebab.png "Options menu") next to the  **Network Observability Operator** and select **Uninstall Operator**.
   3. **Home** → **Projects** and select `openshift-netobserv-operator`
   4. Navigate to **Actions** and select **Delete Project**
3. Remove the `FlowCollector` custom resource definition (CRD).
   1. Navigate to **Administration** → **CustomResourceDefinitions**.
   2. Look for **FlowCollector** and click the Options menu ![](/images/kebab.png "Options menu").
   3. Select **Delete CustomResourceDefinition**.
      :::warning

      The Loki Operator and Kafka remain if they were installed and must be removed separately. Additionally, you might have remaining data stored in an object store, and a persistent volume that must be removed.

      :::
