---
title: Updating a cluster in a disconnected environment using the OpenShift Update Service
sidebar_position: 3
---

# Updating a cluster in a disconnected environment using the OpenShift Update Service {#updating-disconnected-cluster-osus}

<a id="updating-disconnected-cluster-osus"></a>

You can install and configure the OpenShift Update Service (OSUS) in a disconnected environment to get an update experience similar to the connected clusters.

The following steps outline the high-level workflow about updating a cluster in a disconnected environment by using OSUS:

1. Configure access to a secured registry.
2. Update the global cluster pull secret to access your mirror registry.
3. Install the OSUS Operator.
4. Create a graph data container image for the OpenShift Update Service.
5. Install the OSUS application and configure your clusters to use the OpenShift Update Service in your environment.
6. Perform a supported update procedure from the documentation as you would with a connected cluster.

## Using the OpenShift Update Service in a disconnected environment {#update-service-overview_updating-disconnected-cluster-osus}

The OpenShift Update Service (OSUS) provides update recommendations to OpenShift Container Platform clusters. Red Hat publicly hosts the OpenShift Update Service, and clusters in a connected environment can connect to the service through public APIs to retrieve update recommendations.

However, clusters in a disconnected environment cannot access these public APIs to retrieve update information. To have a similar update experience in a disconnected environment, you can install and configure the OpenShift Update Service so that it is available within the disconnected environment.

A single OSUS instance is capable of serving recommendations to thousands of clusters. OSUS can be scaled horizontally to cater to more clusters by changing the replica value. So for most disconnected use cases, one OSUS instance is enough. For example, Red Hat hosts just one OSUS instance for the entire fleet of connected clusters.

If you want to keep update recommendations separate in different environments, you can run one OSUS instance for each environment. For example, in a case where you have separate test and stage environments, you might not want a cluster in a stage environment to receive update recommendations to version A if that version has not been tested in the test environment yet.

The following sections describe how to install an OSUS instance and configure it to provide update recommendations to a cluster.

**Additional resources**

- [About the OpenShift Update Service](/docs/updating/understanding_updates/intro-to-updates#update-service-about_understanding-openshift-updates)
- [Understanding update channels and releases](/docs/updating/understanding_updates/understanding-update-channels-release#understanding-update-channels-releases)

## Configuring access to a secured registry for the OpenShift Update Service {#registry-configuration-for-update-service_updating-disconnected-cluster-osus}

If your release images are contained in a registry whose HTTPS X.509 certificate is signed by a custom certificate authority, you must configure additional trust stores for image registry access for the update service.

**Prerequisites**

- You must have the `oc` command-line interface (CLI) tool installed.
- You must provision a container image registry in your environment with the container images for your update.

**Procedure**

1. Complete the steps in "Configuring additional trust stores for image registry access" along with the following changes for the update service.
   ```yaml title="Image registry CA config map example for the update service"
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: my-registry-ca
   data:
     updateservice-registry: |
       -----BEGIN CERTIFICATE-----
       ...
       -----END CERTIFICATE-----
     registry-with-port.example.com..5000: |
       -----BEGIN CERTIFICATE-----
       ...
       -----END CERTIFICATE-----
   ```

   where:

   <dl>
   <dt><code>data.updateservice-registry</code></dt>
   <dd>Specifies the required config map key name containing the registry certificate authority (CA) certificate.</dd>
   </dl>

   :::note

   If a registry has the port, such as `registry-with-port.example.com:5000`, `:` should be replaced with `..`.

   :::

**Additional resources**

- [Configuring additional trust stores for image registry access](/docs/registry/configuring-registry-operator#images-configuration-cas_configuring-registry-operator)

## Update the global cluster pull secret {#images-update-global-pull-secret_updating-disconnected-cluster-osus}

To add new registries or update authentication for your OpenShift Container Platform cluster, you can update the global pull secret by appending new credentials to the *additional-pull-secret*. To do this, you can use the `oc set data secret/additional-pull-secret -n kube-system` command. Hypershift manages the new credential propagation among the HostedCluster nodes.

Use this procedure when you need a separate registry to store images than the registry used during installation.

**Prerequisites**

- You have access to the cluster as a user with the `cluster-admin` role.

**Procedure**

1. Optional: To append a new pull secret to the existing pull secret:
   1. Download the pull secret by entering the following command:
      ```terminal
      $ oc get secret/pull-secret -n openshift-config --template='{{index .data ".dockerconfigjson" | base64decode}}' > <pull_secret_location>
      ```

      where:

      <dl>
      <dt><code>&lt;pull_secret_location&gt;</code></dt>
      <dd>Specifies the path to the pull secret file.</dd>
      </dl>
   2. Add the new pull secret by entering the following command:
      ```terminal
      $ oc registry login --registry="<registry>" \
      --auth-basic="<username>:<password>" \
      --to=<pull_secret_location>
      ```

      where:

      <dl>
      <dt><code>&lt;registry&gt;</code></dt>
      <dd>Specifies the new registry. You can include many repositories within the same registry, for example: <code>--registry="&lt;registry/my-namespace/my-repository&gt;</code>.</dd>
      <dt><code>&lt;username&gt;:&lt;password&gt;</code></dt>
      <dd>Specifies the credentials of the new registry.</dd>
      <dt><code>&lt;pull_secret_location&gt;</code></dt>
      <dd>Specifies the path to the pull secret file.</dd>
      </dl>
2. Update the global pull secret for your cluster by entering the following command. Note that this update rolls out to all nodes, which can take some time depending on the size of your cluster.
   ```terminal
   $ oc set data secret/pull-secret -n openshift-config \
     --from-file=.dockerconfigjson=<pull_secret_location>
   ```

   where:

   <dl>
   <dt><code>&lt;pull_secret_location&gt;</code></dt>
   <dd>Specifies the path to the new pull secret file.</dd>
   </dl>

   This merges your additional pull secret with the original HostedCluster pull secret, making it available to all nodes in the cluster.
3. Optional: Modify the additional pull secret added by entering the following command:
   ```terminal
   $ oc edit secret additional-pull-secret -n kube-system
   ```

   The secret must contain a valid DockerConfigJSON format.

   ```yaml title="Example pull secret"
   apiVersion: v1
   kind: Secret
   metadata:
     name: additional-pull-secret
     namespace: kube-system
   type: kubernetes.io/dockerconfigjson
   data:
     .dockerconfigjson: <base64-encoded-docker-config-json>
   ```

   This results in the following states of the each pull secret:

   - **Original**: immutable
   - **Additional**: mutable
   - **Global**: final state of both the original and additional pull secrets
4. Optional: Delete the additional pull secret added by entering the following command:
   ```terminal
   $ oc delete secret additional-pull-secret -n kube-system
   ```

   This triggers the automatic cleanup process across your nodes.

**Additional resources**

- [Transferring cluster ownership](https://docs.redhat.com/en/documentation/openshift_cluster_manager/1-latest/html-single/managing_clusters/index#transferring-cluster-ownership_downloading-and-updating-pull-secrets)

## Installing the OpenShift Update Service Operator {#update-service-install_updating-disconnected-cluster-osus}

To install the OpenShift Update Service, you must first install the OpenShift Update Service Operator by using the OpenShift Container Platform web console or CLI.

:::note

For clusters that are installed in disconnected environments, also known as disconnected clusters, Operator Lifecycle Manager by default cannot access the Red Hat-provided software catalog sources hosted on remote registries because those remote sources require full internet connectivity. For more information, see "Using Operator Lifecycle Manager in disconnected environments".

:::

### Installing the OpenShift Update Service Operator by using the web console {#update-service-install-web-console_updating-disconnected-cluster-osus}

You can use the web console to install the OpenShift Update Service Operator.

**Procedure**

1. In the web console, click **Ecosystem** → **Software Catalog**.
   :::note

   Enter `Update Service` into the **Filter by keyword...** field to find the Operator faster.

   :::
2. Choose **OpenShift Update Service** from the list of available Operators, and click **Install**.
   1. Select an **Update channel**.
   2. Select a **Version**.
   3. Select **A specific namespace on the cluster** under **Installation Mode**.
   4. Select a namespace for **Installed Namespace** or accept the recommended namespace `openshift-update-service`.
   5. Select an **Update approval** strategy:
      - The **Automatic** strategy allows Operator Lifecycle Manager (OLM) to automatically update the Operator when a new version is available.
      - The **Manual** strategy requires a cluster administrator to approve the Operator update.
   6. Click **Install**.
3. Go to **Ecosystem** → **Installed Operators** and verify that the OpenShift Update Service Operator is installed.
4. Ensure that **OpenShift Update Service** is listed in the correct namespace with a **Status** of **Succeeded**.

### Installing the OpenShift Update Service Operator by using the CLI {#update-service-install-cli_updating-disconnected-cluster-osus}

You can use the OpenShift CLI (`oc`) to install the OpenShift Update Service Operator.

**Procedure**

1. Create a namespace for the OpenShift Update Service Operator:
   1. Create a `Namespace` object YAML file, for example, `update-service-namespace.yaml`, for the OpenShift Update Service Operator:
      ```yaml
      apiVersion: v1
      kind: Namespace
      metadata:
        name: openshift-update-service
        annotations:
          openshift.io/node-selector: ""
        labels:
          openshift.io/cluster-monitoring: "true"
      ```

      Set the `openshift.io/cluster-monitoring` label to `"true"` to enable Operator-recommended cluster monitoring on this namespace.
   2. Create the namespace:
      ```terminal
      $ oc create -f <filename>.yaml
      ```

      For example:

      ```terminal
      $ oc create -f update-service-namespace.yaml
      ```
2. Install the OpenShift Update Service Operator by creating the following objects:
   1. Create an `OperatorGroup` object YAML file, for example, `update-service-operator-group.yaml`:
      ```yaml
      apiVersion: operators.coreos.com/v1
      kind: OperatorGroup
      metadata:
        name: update-service-operator-group
        namespace: openshift-update-service
      spec:
        targetNamespaces:
        - openshift-update-service
      ```
   2. Create an `OperatorGroup` object:
      ```terminal
      $ oc -n openshift-update-service create -f <filename>.yaml
      ```

      For example:

      ```terminal
      $ oc -n openshift-update-service create -f update-service-operator-group.yaml
      ```
   3. Create a `Subscription` object YAML file, for example, `update-service-subscription.yaml`:
      ```yaml title="Example Subscription"
      apiVersion: operators.coreos.com/v1alpha1
      kind: Subscription
      metadata:
        name: update-service-subscription
        namespace: openshift-update-service
      spec:
        channel: v1
        installPlanApproval: "Automatic"
        source: "redhat-operators"
        sourceNamespace: "openshift-marketplace"
        name: "cincinnati-operator"
      ```

      where:

      <dl>
      <dt><code>spec.source</code></dt>
      <dd>Specifies the name of the catalog source that provides the Operator. For clusters that do not use a custom Operator Lifecycle Manager (OLM), specify <code>redhat-operators</code>. If your OpenShift Container Platform cluster is installed in a disconnected environment, specify the name of the <code>CatalogSource</code> object created when you configured Operator Lifecycle Manager (OLM).</dd>
      </dl>
   4. Create the `Subscription` object:
      ```terminal
      $ oc create -f <filename>.yaml
      ```

      For example:

      ```terminal
      $ oc -n openshift-update-service create -f update-service-subscription.yaml
      ```

      The OpenShift Update Service Operator is installed to the `openshift-update-service` namespace and targets the `openshift-update-service` namespace.
3. Verify the Operator installation:
   ```terminal
   $ oc -n openshift-update-service get clusterserviceversions
   ```

   ```terminal title="Example output"
   NAME                             DISPLAY                    VERSION   REPLACES   PHASE
   update-service-operator.v4.6.0   OpenShift Update Service   4.6.0                Succeeded
   ...
   ```

   If the OpenShift Update Service Operator is listed, the installation was successful. The version number might be different from shown.

**Additional resources**

- [Using Operator Lifecycle Manager in disconnected environments](/docs/disconnected/using-olm#olm-restricted-networks)
- [Installing Operators in your namespace](/docs/operators/user/olm-installing-operators-in-namespace#olm-installing-operators-in-namespace)

## Creating the OpenShift Update Service graph data container image {#update-service-graph-data_updating-disconnected-cluster-osus}

The OpenShift Update Service requires a graph data container image, from which the OpenShift Update Service retrieves information about channel membership and blocked update edges.

Graph data is typically fetched directly from the update graph data repository. In environments where an internet connection is unavailable, loading this information from an init container is another way to make the graph data available to the OpenShift Update Service.

The role of the init container is to provide a local copy of the graph data, and during pod initialization, the init container copies the data to a volume that is accessible by the service.

:::note

The oc-mirror OpenShift CLI (`oc`) plugin creates this graph data container image in addition to mirroring release images. If you used the oc-mirror plugin to mirror your release images, you can skip this procedure.

:::

**Procedure**

1. Create a Dockerfile, for example, `./Dockerfile`, containing the following information:
   ```terminal
   FROM registry.access.redhat.com/ubi9/ubi:latest

   RUN curl -L -o cincinnati-graph-data.tar.gz https://api.openshift.com/api/upgrades_info/graph-data

   RUN mkdir -p /var/lib/cincinnati-graph-data && tar xvzf cincinnati-graph-data.tar.gz -C /var/lib/cincinnati-graph-data/ --no-overwrite-dir --no-same-owner

   CMD ["/bin/bash", "-c" ,"exec cp -rp /var/lib/cincinnati-graph-data/* /var/lib/cincinnati/graph-data"]
   ```
2. Use the docker file created in the above step to build a graph data container image, for example, `registry.example.com/openshift/graph-data:latest`:
   ```terminal
   $ podman build -f ./Dockerfile -t registry.example.com/openshift/graph-data:latest
   ```
3. Push the graph data container image created in the previous step to a repository that is accessible to the OpenShift Update Service, for example, `registry.example.com/openshift/graph-data:latest`:
   ```terminal
   $ podman push registry.example.com/openshift/graph-data:latest
   ```

   :::note

   To push a graph data image to a registry in a disconnected environment, copy the graph data container image created in the previous step to a repository that is accessible to the OpenShift Update Service. Run `oc image mirror --help` for available options.

   :::

## Creating an OpenShift Update Service application {#update-service-create-service_updating-disconnected-cluster-osus}

You can create an OpenShift Update Service application by using the OpenShift Container Platform web console or CLI.

### Creating an OpenShift Update Service application by using the web console {#update-service-create-service-web-console_updating-disconnected-cluster-osus}

You can use the OpenShift Container Platform web console to create an OpenShift Update Service application by using the OpenShift Update Service Operator.

**Prerequisites**

- The OpenShift Update Service Operator has been installed.
- The OpenShift Update Service graph data container image has been created and pushed to a repository that is accessible to the OpenShift Update Service.
- The current release and update target releases have been mirrored to a registry in the disconnected environment.

**Procedure**

1. In the web console, click **Ecosystem** → **Installed Operators**.
2. Choose **OpenShift Update Service** from the list of installed Operators.
3. Click the **Update Service** tab.
4. Click **Create UpdateService**.
5. Enter a name in the **Name** field, for example, `service`.
6. Enter the local pullspec in the **Graph Data Image** field to the graph data container image created in "Creating the OpenShift Update Service graph data container image", for example, `registry.example.com/openshift/graph-data:latest`.
7. In the **Releases** field, enter the registry and repository created to contain the release images in "Mirroring the OpenShift Container Platform image repository", for example, `registry.example.com/ocp4/openshift4-release-images`.
8. Enter `2` in the **Replicas** field.
9. Click **Create** to create the OpenShift Update Service application.
10. Verify the OpenShift Update Service application:
    - From the **UpdateServices** list in the **Update Service** tab, click the Update Service application just created.
    - Click the **Resources** tab.
    - Verify each application resource has a status of **Created**.

### Creating an OpenShift Update Service application by using the CLI {#update-service-create-service-cli_updating-disconnected-cluster-osus}

You can use the OpenShift CLI (`oc`) to create an OpenShift Update Service application.

**Prerequisites**

- The OpenShift Update Service Operator has been installed.
- The OpenShift Update Service graph data container image has been created and pushed to a repository that is accessible to the OpenShift Update Service.
- The current release and update target releases have been mirrored to a registry in the disconnected environment.

**Procedure**

1. Configure the OpenShift Update Service target namespace, for example, `openshift-update-service`:
   ```terminal
   $ NAMESPACE=openshift-update-service
   ```

   The namespace must match the `targetNamespaces` value from the operator group.
2. Configure the name of the OpenShift Update Service application, for example, `service`:
   ```terminal
   $ NAME=service
   ```
3. Configure the registry and repository for the release images as configured in "Mirroring the OpenShift Container Platform image repository", for example, `registry.example.com/ocp4/openshift4-release-images`:
   ```terminal
   $ RELEASE_IMAGES=registry.example.com/ocp4/openshift4-release-images
   ```
4. Set the local pullspec for the graph data image to the graph data container image created in "Creating the OpenShift Update Service graph data container image", for example, `registry.example.com/openshift/graph-data:latest`:
   ```terminal
   $ GRAPH_DATA_IMAGE=registry.example.com/openshift/graph-data:latest
   ```
5. Create an OpenShift Update Service application object:
   ```terminal
   $ oc -n "${NAMESPACE}" create -f - <<EOF
   apiVersion: updateservice.operator.openshift.io/v1
   kind: UpdateService
   metadata:
     name: ${NAME}
   spec:
     replicas: 2
     releases: ${RELEASE_IMAGES}
     graphDataImage: ${GRAPH_DATA_IMAGE}
   EOF
   ```
6. Verify the OpenShift Update Service application:
   1. Use the following command to obtain a policy engine route:
      ```terminal
      $ while sleep 1; do POLICY_ENGINE_GRAPH_URI="$(oc -n "${NAMESPACE}" get -o jsonpath='{.status.policyEngineURI}/api/upgrades_info/v1/graph{"\n"}' updateservice "${NAME}")"; SCHEME="${POLICY_ENGINE_GRAPH_URI%%:*}"; if test "${SCHEME}" = http -o "${SCHEME}" = https; then break; fi; done
      ```

      You might need to poll until the command succeeds.
   2. Retrieve a graph from the policy engine. Be sure to specify a valid version for `channel`. For example, if running in OpenShift Container Platform 4.22, use `stable-4.22`:
      ```terminal
      $ while sleep 10; do HTTP_CODE="$(curl --header Accept:application/json --output /dev/stderr --write-out "%{http_code}" "${POLICY_ENGINE_GRAPH_URI}?channel=stable-4.6")"; if test "${HTTP_CODE}" -eq 200; then break; fi; echo "${HTTP_CODE}"; done
      ```

      This polls until the graph request succeeds; however, the resulting graph might be empty depending on which release images you have mirrored.

      :::note

      The policy engine route name must not be more than 63 characters based on RFC-1123. If you see `ReconcileCompleted` status as `false`  with the reason `CreateRouteFailed` caused by `host must conform to DNS 1123 naming convention and must be no more than 63 characters`, try creating the Update Service with a shorter name.

      :::

## Configuring the Cluster Version Operator (CVO) {#update-service-configure-cvo}

After the OpenShift Update Service Operator has been installed and the OpenShift Update Service application has been created, the Cluster Version Operator (CVO) can be updated to pull graph data from the OpenShift Update Service installed in your environment.

**Prerequisites**

- The OpenShift Update Service Operator has been installed.
- The OpenShift Update Service graph data container image has been created and pushed to a repository that is accessible to the OpenShift Update Service.
- The current release and update target releases have been mirrored to a registry in the disconnected environment.
- The OpenShift Update Service application has been created.

**Procedure**

1. Set the OpenShift Update Service target namespace, for example, `openshift-update-service`:
   ```terminal
   $ NAMESPACE=openshift-update-service
   ```
2. Set the name of the OpenShift Update Service application, for example, `service`:
   ```terminal
   $ NAME=service
   ```
3. Obtain the policy engine route:
   ```terminal
   $ POLICY_ENGINE_GRAPH_URI="$(oc -n "${NAMESPACE}" get -o jsonpath='{.status.policyEngineURI}/api/upgrades_info/v1/graph{"\n"}' updateservice "${NAME}")"
   ```
4. Set the patch for the pull graph data:
   ```terminal
   $ PATCH="{\"spec\":{\"upstream\":\"${POLICY_ENGINE_GRAPH_URI}\"}}"
   ```
5. Patch the CVO to use the OpenShift Update Service in your environment:
   ```terminal
   $ oc patch clusterversion version -p $PATCH --type merge
   ```

   :::note

   For more information about configuring the CA to trust the update server, see "Configuring the cluster-wide proxy".

   :::

**Additional resources**

- [Configuring the cluster-wide proxy](/docs/networking/configuring_network_settings/enable-cluster-wide-proxy#enable-cluster-wide-proxy)

## Verifying your local OSUS installation {#verifying-local-osus-installation_updating-disconnected-cluster-osus}

Before you proceed to update your cluster, verify that the local OSUS instance is correctly installed and configured in your environment.

**Procedure**

1. Confirm that the following conditions are met:
   - The Cluster Version Operator (CVO) is configured to use your installed OpenShift Update Service application.
   - The release image signature config map for the new release is applied to your cluster.
     :::note

     The Cluster Version Operator (CVO) uses release image signatures to ensure that release images have not been modified, by verifying that the release image signatures match the expected result.

     :::
   - The current release and update target release images are mirrored to a registry in the disconnected environment.
   - A recent graph data container image has been mirrored to your registry.
   - A recent version of the OpenShift Update Service Operator is installed.
     :::note

     If you have not recently installed or updated the OpenShift Update Service Operator, there might be a more recent version available. For more information about how to update your OLM catalog in a disconnected environment, see "Using Operator Lifecycle Manager in disconnected environments".

     :::

     After you configure your cluster to use the installed OpenShift Update Service and local mirror registry, you can proceed to update your cluster.

**Additional resources**

- [Using Operator Lifecycle Manager in disconnected environments](/docs/disconnected/using-olm#olm-restricted-networks)

**Additional resources**

- [Mirroring OpenShift Container Platform images](/docs/disconnected/updating/mirroring-image-repository#mirroring-ocp-image-repository)
- [Updating a cluster using the web console](/docs/updating/updating_a_cluster/updating-cluster-web-console#updating-cluster-web-console)
- [Updating a cluster using the CLI](/docs/updating/updating_a_cluster/updating-cluster-cli#updating-cluster-cli)
- [Performing a Control Plane Only update](/docs/updating/updating_a_cluster/control-plane-only-update#control-plane-only-update)
- [Performing a canary rollout update](/docs/updating/updating_a_cluster/update-using-custom-machine-config-pools#update-using-custom-machine-config-pools)
