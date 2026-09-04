---
title: Usage of oc and kubectl commands
sidebar_position: 3
---

# Usage of oc and kubectl commands {#usage-oc-kubectl}

<a id="usage-oc-kubectl"></a>

Because OpenShift Container Platform is a certified Kubernetes distribution, you can use the Kubernetes CLI (`kubectl`) that ships with OpenShift Container Platform to interact with your cluster. You can also gain extended functionality specific to OpenShift Container Platform by using the OpenShift CLI (`oc`) binary.

## The oc binary {#oc-usage-oc_usage-oc-kubectl}

The OpenShift CLI (`oc`) binary offers the same capabilities as the `kubectl` binary, but it extends to natively support additional OpenShift Container Platform features.

Full support for OpenShift Container Platform resources
: Resources such as `DeploymentConfig`, `BuildConfig`, `Route`, `ImageStream`, and `ImageStreamTag` objects are specific to OpenShift Container Platform distributions, and build upon standard Kubernetes primitives.

Authentication
: The `oc` binary offers a built-in `login` command for authentication and lets you work with projects, which map Kubernetes namespaces to authenticated users. Read "Understanding authentication" for more information.

Additional commands
: The additional command `oc new-app`, for example, makes it easier to get new applications started using existing source code or pre-built images. Similarly, the additional command `oc new-project` makes it easier to start a project that you can switch to as your default.

:::warning

If you installed an earlier version of the `oc` binary, you cannot use it to complete all of the commands in OpenShift Container Platform 4.22 . If you want the latest features, you must download and install the latest version of the `oc` binary corresponding to your OpenShift Container Platform server version.

:::

Non-security API changes will involve, at minimum, two minor releases (4.1 to 4.2 to 4.3, for example) to allow older `oc` binaries to update. Using new capabilities might require newer `oc` binaries. A 4.3 server might have additional capabilities that a 4.2 `oc` binary cannot use and a 4.3 `oc` binary might have additional capabilities that are unsupported by a 4.2 server.

**Compatibility matrix**

|  |  |  |
| --- | --- | --- |
|  | **X.Y** (`oc` Client) | **X.Y+N** [^versionpolicyn] (`oc` Client) |
| **X.Y** (Server) | ![Red circle 1](/images/redcircle-1.png) | ![Red circle 3](/images/redcircle-3.png) |
| **X.Y+N** [^versionpolicyn] (Server) | ![Red circle 2](/images/redcircle-2.png) | ![Red circle 1](/images/redcircle-1.png) |

![Red circle 1](/images/redcircle-1.png) Fully compatible.

![Red circle 2](/images/redcircle-2.png) `oc` client might not be able to access server features.

![Red circle 3](/images/redcircle-3.png) `oc` client might provide options and features that might not be compatible with the accessed server.

**Additional resources**

- [Understanding authentication](/docs/authentication/understanding-authentication#understanding-authentication)

## The kubectl binary {#oc-usage-kubectl_usage-oc-kubectl}

The Kubernetes CLI (`kubectl`) binary is provided as a means to support existing workflows and scripts for new OpenShift Container Platform users coming from a standard Kubernetes environment, or for those who prefer to use the `kubectl` CLI. Existing users of `kubectl` can continue to use the binary to interact with Kubernetes primitives, with no changes required to the OpenShift Container Platform cluster.

You can install the supported `kubectl` binary by following the steps to install the OpenShift CLI. The `kubectl` binary is included in the archive if you download the binary, or is installed when you install the CLI by using an RPM.

**Additional resources**

- [kubectl (Kubernetes documentation)](https://kubernetes.io/docs/reference/kubectl/overview/)
- [Getting started with the OpenShift CLI](/docs/cli_reference/openshift_cli/getting-started-cli#cli-getting-started)

[^versionpolicyn]: Where **N** is a number greater than or equal to 1.
