---
title: Using the cluster-compare plugin
sidebar_position: 3
---

# Using the cluster-compare plugin

<a id="using-the-cluster-compare-plugin"></a>

You can use the `cluster-compare` plugin to compare a reference configuration with a configuration from a live cluster or `must-gather` data.

## Using the cluster-compare plugin with a live cluster {#using-cluster-compare-live-cluster_using-cluster-compare-plugin}

You can use the `cluster-compare` plugin to compare a reference configuration with configuration custom resources (CRs) from a live cluster.

Validate live cluster configurations to ensure compliance with reference configurations during design, development, or testing scenarios.

:::note

Use the `cluster-compare` plugin with live clusters in non-production environments only. For production environments, use the plugin with `must-gather` data.

:::

**Prerequisites**

- You installed the OpenShift CLI (`oc`).
- You have access to the cluster as a user with the `cluster-admin` role.
- You downloaded the `cluster-compare` plugin and include it in your `PATH` environment variable.
- You have access to a reference configuration.

**Procedure**

- Run the `cluster-compare` plugin by using the following command:
  ```terminal
  $ oc cluster-compare -r <path_to_reference_config>/metadata.yaml
  ```

  - `-r` specifies a path to the `metadata.yaml` file of the reference configuration. You can specify a local directory or a URI.

The following is example output:

```terminal

...

**********************************

Cluster CR: operator.openshift.io/v1_Console_cluster
Reference File: optional/console-disable/ConsoleOperatorDisable.yaml
Diff Output: diff -u -N /tmp/MERGED-622469311/operator-openshift-io-v1_console_cluster /tmp/LIVE-2358803347/operator-openshift-io-v1_console_cluster
/tmp/MERGED-622469311/operator-openshift-io-v1_console_cluster	2024-11-20 15:43:42.888633602 +0000
+++ /tmp/LIVE-2358803347/operator-openshift-io-v1_console_cluster	2024-11-20 15:43:42.888633602 +0000
@@ -4,5 +4,5 @@
   name: cluster
 spec:
   logLevel: Normal
-  managementState: Removed
+  managementState: Managed
   operatorLogLevel: Normal

**********************************

…

Summary
CRs with diffs: 5/49
CRs in reference missing from the cluster: 1
required-cluster-tuning:
  cluster-tuning:
    Missing CRs:
    - required/cluster-tuning/disabling-network-diagnostics/DisableSnoNetworkDiag.yaml
No CRs are unmatched to reference CRs
Metadata Hash: 512a9bf2e57fd5a5c44bbdea7abb3ffd7739d4a1f14ef9021f6793d5cdf868f0
No patched CRs
```

where:

`Cluster CR`
: The CR under comparison. The plugin displays each CR with a difference from the corresponding template.

`Reference File`
: The template matching with the CR for comparison.

`Diff Output`
: The output in Linux diff format shows the difference between the template and the cluster CR.

`Summary`
: After the plugin reports the line diffs for each CR, the summary of differences are reported.

`CRs with diffs`
: The number of CRs in the comparison with differences from the corresponding templates.

`CRs in reference missing from the cluster`
: The number of CRs represented in the reference configuration, but missing from the live cluster.

`Missing CRs`
: The list of CRs represented in the reference configuration, but missing from the live cluster.

`No CRs are unmatched to reference CRs`
: The CRs that did not match to a corresponding template in the reference configuration.

`Metadata Hash`
: The metadata hash identifies the reference configuration.

`No patched CRs`
: The list of patched CRs.

:::note

Get the output in the `junit` format by adding `-o junit` to the command. For example:

```terminal
$ oc cluster-compare -r <path_to_reference_config>/metadata.yaml -o junit
```

The `junit` output includes the following result types:

- Passed results for each fully matched template.
- Failed results for differences found or missing required custom resources (CRs).
- Skipped results for differences patched using the user override mechanism.

:::

## Using the cluster-compare plugin with must-gather data {#using-cluster-compare-must-gather_using-cluster-compare-plugin}

You can use the `cluster-compare` plugin to compare a reference configuration with configuration custom resources (CRs) from `must-gather` data.

Validate cluster configurations by using `must-gather` data to troubleshoot configuration issues in production environments.

:::note

For production environments, use the `cluster-compare` plugin with `must-gather` data only.

:::

- You have access to `must-gather` data from a target cluster.
- You installed the OpenShift CLI (`oc`).
- You have downloaded the `cluster-compare` plugin and included it in your `PATH` environment variable.
- You have access to a reference configuration.

**Procedure**

- Compare the `must-gather` data to a reference configuration by running the following command:
  ```terminal
  $ oc cluster-compare -r <path_to_reference_config>/metadata.yaml -f "must-gather*/*/cluster-scoped-resources","must-gather*/*/namespaces" -R
  ```

  - `-r` specifies a path to the `metadata.yaml` file of the reference configuration. You can specify a local directory or a URI.
  - `-f` specifies the path to the `must-gather` data directory. You can specify a local directory or a URI. This example restricts the comparison to the relevant cluster configuration directories.
  - `-R` searches the target directories recursively.
    The following is example output:

    ```terminal
    ...

    **********************************

    Cluster CR: operator.openshift.io/v1_Console_cluster
    Reference File: optional/console-disable/ConsoleOperatorDisable.yaml
    Diff Output: diff -u -N /tmp/MERGED-622469311/operator-openshift-io-v1_console_cluster /tmp/LIVE-2358803347/operator-openshift-io-v1_console_cluster
    /tmp/MERGED-622469311/operator-openshift-io-v1_console_cluster	2024-11-20 15:43:42.888633602 +0000
    +++ /tmp/LIVE-2358803347/operator-openshift-io-v1_console_cluster	2024-11-20 15:43:42.888633602 +0000
    @@ -4,5 +4,5 @@
       name: cluster
     spec:
       logLevel: Normal
    -  managementState: Removed
    +  managementState: Managed
       operatorLogLevel: Normal

    **********************************

    …

    Summary
    CRs with diffs: 5/49
    CRs in reference missing from the cluster: 1
    required-cluster-tuning:
      cluster-tuning:
        Missing CRs:
        - required/cluster-tuning/disabling-network-diagnostics/DisableSnoNetworkDiag.yaml
    No CRs are unmatched to reference CRs
    Metadata Hash: 512a9bf2e57fd5a5c44bbdea7abb3ffd7739d4a1f14ef9021f6793d5cdf868f0
    No patched CRs
    ```

    where:

    <dl>
    <dt><code>Cluster CR</code></dt>
    <dd>The CR under comparison. The plugin displays each CR with a difference from the corresponding template.</dd>
    <dt><code>Reference File</code></dt>
    <dd>The template matching with the CR for comparison.</dd>
    <dt><code>Diff Output</code></dt>
    <dd>The output in Linux diff format shows the difference between the template and the cluster CR.</dd>
    <dt><code>Summary</code></dt>
    <dd>After the plugin reports the line diffs for each CR, the summary of differences are reported.</dd>
    <dt><code>CRs with diffs</code></dt>
    <dd>The number of CRs in the comparison with differences from the corresponding templates.</dd>
    <dt><code>CRs in reference missing from the cluster</code></dt>
    <dd>The number of CRs represented in the reference configuration, but missing from the live cluster.</dd>
    <dt><code>Missing CRs</code></dt>
    <dd>The list of CRs represented in the reference configuration, but missing from the live cluster.</dd>
    <dt><code>No CRs are unmatched to reference CRs</code></dt>
    <dd>The CRs that did not match to a corresponding template in the reference configuration.</dd>
    <dt><code>Metadata Hash</code></dt>
    <dd>The metadata hash identifies the reference configuration.</dd>
    <dt><code>No patched CRs</code></dt>
    <dd>The list of patched CRs.</dd>
    </dl>

:::note

Get the output in the `junit` format by adding `-o junit` to the command. For example:

```terminal
$ oc cluster-compare -r <path_to_reference_config>/metadata.yaml -f "must-gather*/*/cluster-scoped-resources","must-gather*/*/namespaces" -R -o junit
```

The `junit` output includes the following result types:

- Passed results for each fully matched template.
- Failed results for differences found or missing required custom resources (CRs).
- Skipped results for differences patched using the user override mechanism.

:::

**Additional resources**

- [Gathering data about your cluster](/docs/support/gathering-cluster-data#about-must-gather_gathering-cluster-data)

## Reference cluster-compare plugin options {#cluster-compare-reference-args_using-cluster-compare-plugin}

The following content describes the options for the `cluster-compare` plugin.

**Cluster-compare plugin options**

<table>
<tbody>
<tr>
  <td>Option</td>
  <td>Description</td>
</tr>
<tr>
  <td><code>-A</code>, <code>--all-resources</code></td>
  <td>When used with a live cluster, attempts to match all resources in the cluster that match a type in the reference configuration. When used with local files, attempts to match all resources in the local files that match a type in the reference configuration.</td>
</tr>
<tr>
  <td><code>--concurrency</code></td>
  <td>Specify an integer value for the number of templates to process in parallel when comparing with resources from the live version. A larger number increases speed but also memory, I/O, and CPU usage during that period. The default value is <code>4</code>.</td>
</tr>
<tr>
  <td><code>-c</code>, <code>--diff-config</code></td>
  <td>Specify the path to the user configuration file.</td>
</tr>
<tr>
  <td><code>-f</code>, <code>--filename</code></td>
  <td>Specify a filename, directory, or URL for the configuration custom resources that you want to use for a comparison with a reference configuration.</td>
</tr>
<tr>
  <td><code>--generate-override-for</code></td>
  <td>Specify the path for templates that requires a patch.</td>
</tr>
<tr>
  <td><code>--show-template-functions</code></td>
  <td>Displays the available template functions.<br><br><div class="alert alert--secondary"><strong>Note</strong><p>You must use a file path for the target template that is relative to the <code>metadata.yaml</code> file. For example, if the file path for the <code>metadata.yaml</code> file is <code>./compare/metadata.yaml</code>, a relative file path for the template might be <code>optional/my-template.yaml</code>.</p></div></td>
</tr>
<tr>
  <td><code>-h</code>, <code>--help</code></td>
  <td>Display help information.</td>
</tr>
<tr>
  <td><code>-k</code>, <code>--kustomize</code></td>
  <td>Specify a path to process the <code>kustomization</code> directory. This flag cannot be used together with <code>-f</code> or <code>-R</code>.</td>
</tr>
<tr>
  <td><code>-o</code>, <code>--output</code></td>
  <td>Specify the output format. Options include <code>json</code>, <code>yaml</code>, <code>junit</code>, or <code>generate-patches</code>.</td>
</tr>
<tr>
  <td><code>--override-reason</code></td>
  <td>Specify a reason for generating the override.</td>
</tr>
<tr>
  <td><code>-p</code>, <code>--overrides</code></td>
  <td>Specify a path to a patch override file for the reference configuration.</td>
</tr>
<tr>
  <td><code>-R</code>, <code>--recursive</code></td>
  <td>Processes the directory specified in <code>-f</code>, <code>--filename</code> recursively.</td>
</tr>
<tr>
  <td><code>-r</code>, <code>--reference</code></td>
  <td>Specify the path to the reference configuration <code>metadata.yaml</code> file.</td>
</tr>
<tr>
  <td><code>--show-managed-fields</code></td>
  <td>Specify <code>true</code> to include managed fields in the comparison.</td>
</tr>
<tr>
  <td><code>-v</code>, <code>--verbose</code></td>
  <td>Increases the verbosity of the plugin output.</td>
</tr>
</tbody>
</table>

## Example: Comparing a cluster with the telco core reference configuration {#using-cluster-compare-telco_ref_using-cluster-compare-plugin}

You can use the `cluster-compare` plugin to compare a reference configuration with a configuration from a live cluster or `must-gather` data.

This example compares a configuration from a live cluster with the telco core reference configuration. The telco core reference configuration is derived from the telco core reference design specifications (RDS). The telco core RDS is designed for clusters to support large scale telco applications including control plane and some centralized data plane functions.

The reference configuration is packaged in a container image with the telco core RDS.

For further examples of using the `cluster-compare` plugin with the telco core and telco RAN distributed unit (DU) profiles, see the "Additional resources" section.

**Prerequisites**

- You have access to the cluster as a user with the `cluster-admin` role.
- You have credentials to access the `registry.redhat.io` container image registry.
- You installed the `cluster-compare` plugin.

**Procedure**

1. Log on to the container image registry with your credentials by running the following command:
   ```terminal
   $ podman login registry.redhat.io
   ```
2. Extract the content from the `telco-core-rds-rhel9` container image by running the following commands:
   ```terminal
   $ mkdir -p ./out
   ```

   ```terminal
   $ podman run -it registry.redhat.io/openshift4/openshift-telco-core-rds-rhel9:v4.18 | base64 -d | tar xv -C out
   ```

   You can view the reference configuration in the `reference-crs-kube-compare/` directory.

   ```text
   out/telco-core-rds/configuration/reference-crs-kube-compare/
   ├── metadata.yaml
   ├── optional
   │   ├── logging
   │   ├── networking
   │   ├── other
   │   └── tuning
   └── required
       ├── networking
       ├── other
       ├── performance
       ├── scheduling
       └── storage
   ```

   where:

   <dl>
   <dt><code>metadata.yaml</code></dt>
   <dd>Configuration file for the reference configuration.</dd>
   <dt><code>optional</code></dt>
   <dd>Directory for optional templates.</dd>
   <dt><code>required</code></dt>
   <dd>Directory for required templates.</dd>
   </dl>
3. Compare the configuration for your cluster to the telco core reference configuration by running the following command:
   ```terminal
   $ oc cluster-compare -r out/telco-core-rds/configuration/reference-crs-kube-compare/metadata.yaml
   ```

   The following is example output:

   ```terminal
   W1212 14:13:06.281590   36629 compare.go:425] Reference Contains Templates With Types (kind) Not Supported By Cluster: BFDProfile, BGPAdvertisement, BGPPeer, ClusterLogForwarder, Community, IPAddressPool, MetalLB, MultiNetworkPolicy, NMState, NUMAResourcesOperator, NUMAResourcesScheduler, NodeNetworkConfigurationPolicy, SriovNetwork, SriovNetworkNodePolicy, SriovOperatorConfig, StorageCluster

   ...

   **********************************

   Cluster CR: config.openshift.io/v1_OperatorHub_cluster
   Reference File: required/other/operator-hub.yaml
   Diff Output: diff -u -N /tmp/MERGED-2801470219/config-openshift-io-v1_operatorhub_cluster /tmp/LIVE-2569768241/config-openshift-io-v1_operatorhub_cluster
   --- /tmp/MERGED-2801470219/config-openshift-io-v1_operatorhub_cluster	2024-12-12 14:13:22.898756462 +0000
   +++ /tmp/LIVE-2569768241/config-openshift-io-v1_operatorhub_cluster	2024-12-12 14:13:22.898756462 +0000
   @@ -1,6 +1,6 @@
    apiVersion: config.openshift.io/v1
    kind: OperatorHub
    metadata:
   +  annotations:
   +    include.release.openshift.io/hypershift: "true"
      name: cluster
   -spec:
   -  disableAllDefaultSources: true

   **********************************

   Summary
   CRs with diffs: 3/4
   CRs in reference missing from the cluster: 22
   other:
     other:
       Missing CRs:
       - optional/other/control-plane-load-kernel-modules.yaml
       - optional/other/worker-load-kernel-modules.yaml
   required-networking:
     networking-root:
       Missing CRs:
       - required/networking/nodeNetworkConfigurationPolicy.yaml
     networking-sriov:
       Missing CRs:
       - required/networking/sriov/sriovNetwork.yaml
       - required/networking/sriov/sriovNetworkNodePolicy.yaml
       - required/networking/sriov/SriovOperatorConfig.yaml
       - required/networking/sriov/SriovSubscription.yaml
       - required/networking/sriov/SriovSubscriptionNS.yaml
       - required/networking/sriov/SriovSubscriptionOperGroup.yaml
   required-other:
     scheduling:
       Missing CRs:
       - required/other/catalog-source.yaml
       - required/other/icsp.yaml
   required-performance:
     performance:
       Missing CRs:
       - required/performance/PerformanceProfile.yaml
   required-scheduling:
     scheduling:
       Missing CRs:
       - required/scheduling/nrop.yaml
       - required/scheduling/NROPSubscription.yaml
       - required/scheduling/NROPSubscriptionNS.yaml
       - required/scheduling/NROPSubscriptionOperGroup.yaml
       - required/scheduling/sched.yaml
   required-storage:
     storage-odf:
       Missing CRs:
       - required/storage/odf-external/01-rook-ceph-external-cluster-details.secret.yaml
       - required/storage/odf-external/02-ocs-external-storagecluster.yaml
       - required/storage/odf-external/odfNS.yaml
       - required/storage/odf-external/odfOperGroup.yaml
       - required/storage/odf-external/odfSubscription.yaml
   No CRs are unmatched to reference CRs
   Metadata Hash: fe41066bac56517be02053d436c815661c9fa35eec5922af25a1be359818f297
   No patched CRs
   ```

   where:

   <dl>
   <dt><code>Cluster CR</code></dt>
   <dd>The CR under comparison. The plugin displays each CR with a difference from the corresponding template.</dd>
   <dt><code>Reference File</code></dt>
   <dd>The template matching with the CR for comparison.</dd>
   <dt><code>Diff Output</code></dt>
   <dd>The output in Linux diff format shows the difference between the template and the cluster CR.</dd>
   <dt><code>Summary</code></dt>
   <dd>After the plugin reports the line diffs for each CR, the summary of differences are reported.</dd>
   <dt><code>CRs with diffs</code></dt>
   <dd>The number of CRs in the comparison with differences from the corresponding templates.</dd>
   <dt><code>CRs in reference missing from the cluster</code></dt>
   <dd>The number of CRs represented in the reference configuration, but missing from the live cluster.</dd>
   <dt><code>Missing CRs</code></dt>
   <dd>The list of CRs represented in the reference configuration, but missing from the live cluster.</dd>
   <dt><code>No CRs are unmatched to reference CRs</code></dt>
   <dd>The CRs that did not match to a corresponding template in the reference configuration.</dd>
   <dt><code>Metadata Hash</code></dt>
   <dd>The metadata hash identifies the reference configuration.</dd>
   <dt><code>No patched CRs</code></dt>
   <dd>The list of patched CRs.</dd>
   </dl>

:::note

Get the output in the `junit` format by adding `-o junit` to the command. For example:

```terminal
$ oc cluster-compare -r out/telco-core-rds/configuration/reference-crs-kube-compare/metadata.yaml -o junit
```

The `junit` output includes the following result types:

- Passed results for each fully matched template.
- Failed results for differences found or missing required custom resources (CRs).
- Skipped results for differences patched using the user override mechanism.

:::

**Additional resources**

- [Comparing a cluster with the telco RAN DU reference configuration](/docs/scalability_and_performance/telco-ran-du-rds#using-cluster-compare-telco-ran_ran-ref-design-crs)
- [Comparing a cluster with the telco core reference configuration](/docs/scalability_and_performance/telco-core-rds#using-cluster-compare-telco_core_telco-core)
