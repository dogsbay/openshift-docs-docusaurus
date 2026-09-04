---
title: Using remote health reporting in a restricted network
sidebar_position: 6
---

# Using remote health reporting in a restricted network {#remote-health-reporting-from-restricted-network}

<a id="remote-health-reporting-from-restricted-network"></a>

You can manually gather and upload Insights Operator archives to diagnose issues from a restricted network.

To use the Insights Operator in a restricted network, you must complete the following tasks:

- Create a copy of your Insights Operator archive.
- Upload the Insights Operator archive to the Red Hat Hybrid Cloud Console.

Additionally, you can select to obfuscate the Insights Operator data before data upload.

**Additional resources**

- [Red Hat Hybrid Cloud Console](https://console.redhat.com)
- [Enabling Insights Operator data obfuscation](/docs/support/remote_health_monitoring/remote-health-reporting-from-restricted-network#insights-operator-enable-obfuscation_remote-health-reporting-from-restricted-network)

## Running an Insights Operator gather operation {#insights-operator-one-time-gather_remote-health-reporting-from-restricted-network}

You must run a gather operation to create an Insights Operator archive.

**Prerequisites**

- You are logged in to OpenShift Container Platform as `cluster-admin`.

**Procedure**

1. Create a file named `gather-job.yaml` using this template:
   ```yaml

   ```

apiVersion: batch/v1 kind: Job metadata: name: insights-operator-job annotations: config.openshift.io/inject-proxy: insights-operator spec: backoffLimit: 6 ttlSecondsAfterFinished: 600 template: spec: restartPolicy: OnFailure serviceAccountName: operator nodeSelector: beta.kubernetes.io/os: linux node-role.kubernetes.io/master: "" tolerations: - effect: NoSchedule key: node-role.kubernetes.io/master operator: Exists - effect: NoExecute key: node.kubernetes.io/unreachable operator: Exists tolerationSeconds: 900 - effect: NoExecute key: node.kubernetes.io/not-ready operator: Exists tolerationSeconds: 900 volumes: - name: snapshots emptyDir: {} - name: service-ca-bundle configMap: name: service-ca-bundle optional: true initContainers: - name: insights-operator image: quay.io/openshift/origin-insights-operator:latest terminationMessagePolicy: FallbackToLogsOnError volumeMounts: - name: snapshots mountPath: /var/lib/insights-operator - name: service-ca-bundle mountPath: /var/run/configmaps/service-ca-bundle readOnly: true ports: - containerPort: 8443 name: https resources: requests: cpu: 10m memory: 70Mi args: - gather - -v=4 - --config=/etc/insights-operator/server.yaml containers: - name: sleepy image: quay.io/openshift/origin-base:latest args: - /bin/sh - -c - sleep 10m volumeMounts: [{name: snapshots, mountPath: /var/lib/insights-operator}] ```

1. Copy your `insights-operator` image version:
   ```terminal
   $ oc get -n openshift-insights deployment insights-operator -o yaml
   ```

   ```yaml title="Example output"
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: insights-operator
     namespace: openshift-insights
   # ...
   spec:
     template:
   # ...
       spec:
         containers:
         - args:
   # ...
           image: registry.ci.openshift.org/ocp/4.15-2023-10-12-212500@sha256:a0aa581400805ad0...
   # ...
   ```

   The `spec.template.spec.containers.image` field specifies your `insights-operator` image version.
2. Paste your image version in `gather-job.yaml`:
   ```yaml
   apiVersion: batch/v1
   kind: Job
   metadata:
     name: insights-operator-job
   # ...
   spec:
   # ...
     template:
       spec:
       initContainers:
       - name: insights-operator
         image: image: registry.ci.openshift.org/ocp/4.15-2023-10-12-212500@sha256:a0aa581400805ad0...
         terminationMessagePolicy: FallbackToLogsOnError
         volumeMounts:
   ```

   Replace the value of `spec.template.initContainers.image` with your `insights-operator` image version.
3. Create the gather job:
   ```terminal
   $ oc apply -n openshift-insights -f gather-job.yaml
   ```
4. Find the name of the job pod:
   ```terminal
   $ oc describe -n openshift-insights job/insights-operator-job
   ```

   ```terminal title="Example output"
   Name:             insights-operator-job
   Namespace:        openshift-insights
   # ...
   Events:
     Type    Reason            Age    From            Message
     ----    ------            ----   ----            -------
     Normal  SuccessfulCreate  7m18s  job-controller  Created pod: insights-operator-job-<your_job>
   ```

   Replace `insights-operator-job-<your_job>` with the name of the pod.
5. Verify that the operation has finished:
   ```terminal
   $ oc logs -n openshift-insights insights-operator-job-<your_job> insights-operator
   ```

   ```terminal title="Example output"
   I0407 11:55:38.192084       1 diskrecorder.go:34] Wrote 108 records to disk in 33ms
   ```
6. Save the created archive:
   ```terminal
   $ oc cp openshift-insights/insights-operator-job-_<your_job>_:/var/lib/insights-operator ./insights-data
   ```
7. Clean up the job:
   ```terminal
   $ oc delete -n openshift-insights job insights-operator-job
   ```

## Uploading an Insights Operator archive {#insights-operator-manual-upload_remote-health-reporting-from-restricted-network}

You can manually upload an Insights Operator archive to [console.redhat.com](https://console.redhat.com) to diagnose potential issues.

**Prerequisites**

- You are logged in to OpenShift Container Platform as `cluster-admin`.
- You have a workstation with unrestricted internet access.
- You have created a copy of the Insights Operator archive.

**Procedure**

1. Download the `dockerconfig.json` file:
   ```terminal
   $ oc extract secret/pull-secret -n openshift-config --to=.
   ```
2. Copy your `"cloud.openshift.com"` `"auth"` token from the `dockerconfig.json` file:
   ```json
   {
     "auths": {
       "cloud.openshift.com": {
         "auth": "_<your_token>_",
         "email": "asd@redhat.com"
       }
   }
   ```
3. Upload the archive to [console.redhat.com](https://console.redhat.com):
   ```terminal
   $ curl -v -H "User-Agent: insights-operator/one10time200gather184a34f6a168926d93c330 cluster/_<cluster_id>_" -H "Authorization: Bearer _<your_token>_" -F "upload=@_<path_to_archive>_; type=application/vnd.redhat.openshift.periodic+tar" https://console.redhat.com/api/ingress/v1/upload
   ```

   where:

   <dl>
   <dt><code>&lt;cluster_id&gt;</code></dt>
   <dd>Specifies the cluster ID.</dd>
   <dt><code>&lt;your_token&gt;</code></dt>
   <dd>Specifies the token from your pull secret.</dd>
   <dt><code>&lt;path_to_archive&gt;</code></dt>
   <dd>Specifies the path to the Insights Operator archive. If the operation is successful, the command returns a <code>"request_id"</code> and <code>"account_number"</code>:</dd>
   </dl>

   ```terminal title="Example output"
   * Connection #0 to host console.redhat.com left intact
   {"request_id":"393a7cf1093e434ea8dd4ab3eb28884c","upload":{"account_number":"6274079"}}%
   ```

**Verification**

1. Log in to [https://console.redhat.com/openshift](https://console.redhat.com/openshift).
2. Click the **Cluster List** menu in the left pane.
3. To display the details of the cluster, click the cluster name.
4. Open the **Red Hat Lightspeed Advisor** tab of the cluster.
   If the upload was successful, the tab displays one of the following:

   - **Your cluster passed all recommendations**, if the Red Hat Lightspeed advisor service did not identify any issues.
   - A list of issues that the Red Hat Lightspeed advisor service has detected, prioritized by risk (low, moderate, important, and critical).

## Enabling Insights Operator data obfuscation {#insights-operator-enable-obfuscation_remote-health-reporting-from-restricted-network}

You can enable obfuscation to mask sensitive and identifiable IPv4 addresses and cluster base domains that the Insights Operator sends to [console.redhat.com](https://console.redhat.com).

:::warning

Although this feature is available, Red Hat recommends keeping obfuscation disabled for a more effective support experience.

:::

Obfuscation assigns non-identifying values to cluster IPv4 addresses, and uses a translation table that is retained in memory to change IP addresses to their obfuscated versions throughout the Insights Operator archive before uploading the data to [console.redhat.com](https://console.redhat.com).

For cluster base domains, obfuscation changes the base domain to a hardcoded substring. For example, `cluster-api.openshift.example.com` becomes `cluster-api.<CLUSTER_BASE_DOMAIN>`.

The following procedure enables obfuscation using the `support` secret in the `openshift-config` namespace.

**Prerequisites**

- You are logged in to the OpenShift Container Platform web console as `cluster-admin`.

**Procedure**

1. Navigate to **Workloads** → **Secrets**.
2. Select the **openshift-config** project.
3. Search for the **support** secret using the **Search by name** field. If it does not exist, click **Create** → **Key/value secret** to create it.
4. Click the Options menu ![](/images/kebab.png "Options menu"), and then click **Edit Secret**.
5. Click **Add Key/Value**.
6. Create a key named `enableGlobalObfuscation` with a value of `true`, and click **Save**.
7. Navigate to **Workloads** → **Pods**
8. Select the `openshift-insights` project.
9. Find the `insights-operator` pod.
10. To restart the `insights-operator` pod, click the Options menu ![](/images/kebab.png "Options menu"), and then click **Delete Pod**.

**Verification**

1. Navigate to **Workloads** → **Secrets**.
2. Select the **openshift-insights** project.
3. Search for the **obfuscation-translation-table** secret using the **Search by name** field.
   If the `obfuscation-translation-table` secret exists, then obfuscation is enabled and working.

   Alternatively, you can inspect `/insights-operator/gathers.json` in your Insights Operator archive for the value `"is_global_obfuscation_enabled": true`.

**Additional resources**

- [Showing data collected by the Insights Operator](/docs/support/remote_health_monitoring/showing-data-collected-by-remote-health-monitoring#insights-operator-showing-data-collected-from-the-cluster_showing-data-collected-by-remote-health-monitoring)
