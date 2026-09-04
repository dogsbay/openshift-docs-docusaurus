---
title: Feature gates in a hosted cluster
sidebar_position: 12
---

# Feature gates in a hosted cluster {#hcp-using-feature-gates}

<a id="hcp-using-feature-gates"></a>

You can use feature gates in a hosted cluster to enable features that are not part of the default set of features. You can enable the `TechPreviewNoUpgrade` feature set by using feature gates in your hosted cluster.

## Enabling feature sets by using feature gates {#hcp-enable-feature-sets_hcp-using-feature-gates}

You can enable the `TechPreviewNoUpgrade` feature set in a hosted cluster by editing the `HostedCluster` custom resource (CR) with the OpenShift CLI.

**Prerequisites**

- You installed the OpenShift CLI (`oc`).

**Procedure**

1. Open the `HostedCluster` CR for editing on the hosting cluster by running the following command:
   ```terminal
   $ oc edit hostedcluster <hosted_cluster_name> -n <hosted_cluster_namespace>
   ```
2. Define the feature set by entering a value in the `featureSet` field. For example:
   ```yaml
   apiVersion: hypershift.openshift.io/v1beta1
   kind: HostedCluster
   metadata:
     name: <hosted_cluster_name>
     namespace: <hosted_cluster_namespace>
   spec:
     configuration:
       featureGate:
         featureSet: TechPreviewNoUpgrade
   ```

   - `metadata.name` specifies your hosted cluster name.
   - `metadata.namespace` specifies your hosted cluster namespace.
   - `spec.configuration.featureGate.featureSet` is a subset of the current Technology Preview features.
     :::warning

     Enabling the `TechPreviewNoUpgrade` feature set on your cluster cannot be undone and prevents minor version updates. With this feature set, you can enable these Technology Preview features on test clusters, where you can fully test them. Do not enable this feature set on production clusters.

     :::
3. Save the file to apply the changes.

**Verification**

- Verify that the `TechPreviewNoUpgrade` feature gate is enabled in your hosted cluster by running the following command:
  ```terminal
  $ oc get featuregate cluster -o yaml
  ```

**Additional resources**

- [FeatureGate [config.openshift.io/v1](/docs/rest_api/config_apis/featuregate-config-openshift-io-v1#featuregate-config-openshift-io-v1)]
