---
title: Preparing to install Service Mesh
sidebar_position: 7
---

# Preparing to install Service Mesh {#preparing-ossm-installation}

<a id="preparing-ossm-installation"></a>

Before you can install Red Hat OpenShift Service Mesh, you must subscribe to OpenShift Container Platform and install OpenShift Container Platform in a supported configuration.

## Prerequisites {#_prerequisites}

- Maintain an active OpenShift Container Platform subscription on your Red Hat account. If you do not have a subscription, contact your sales representative for more information.
- Review the [OpenShift Container Platform 4.22 overview](/docs/architecture/architecture-installation#installation-overview_architecture-installation).
- Install OpenShift Container Platform 4.22. If you are installing Red Hat OpenShift Service Mesh on a [restricted network](/docs/installing/overview/installing-preparing#installing-preparing-supported-installation-methods-reference_installing-preparing), follow the instructions for your chosen OpenShift Container Platform infrastructure.
  - [Install OpenShift Container Platform 4.22 on AWS](/docs/installing/installing_aws/ipi/installing-aws-default#installing-aws-default)
  - [Install OpenShift Container Platform 4.22 on AWS with user-provisioned infrastructure](/docs/installing/installing_aws/upi/installing-aws-user-infra#installing-aws-user-infra)
  - [Install OpenShift Container Platform 4.22 on bare metal](/docs/installing/installing_bare_metal/upi/installing-bare-metal#installing-bare-metal)
  - [Install OpenShift Container Platform 4.22 on vSphere](/docs/installing/installing_vsphere/upi/installing-vsphere#installing-vsphere)
  - [Install OpenShift Container Platform 4.22 on IBM Z(R) and IBM(R) LinuxONE](/docs/installing/installing_ibm_z/upi/installing-ibm-z#installing-ibm-z)
  - [Install OpenShift Container Platform 4.22 on IBM Power(R)](/docs/installing/installing_ibm_power/installing-ibm-power#installing-ibm-power)
- Install the version of the OpenShift Container Platform command-line utility (the `oc` client tool) that matches your OpenShift Container Platform version and add it to your path.
  - If you are using OpenShift Container Platform 4.22, see [About the OpenShift CLI](/docs/cli_reference/openshift_cli/getting-started-cli#cli-about-cli_cli-developer-commands).

For additional information about Red Hat OpenShift Service Mesh lifecycle and supported platforms, refer to the [Support Policy](https://access.redhat.com/support/policy/updates/openshift#ossm).

## Supported configurations {#ossm-supported-configurations_preparing-ossm-installation}

The following configurations are supported for the current release of Red Hat OpenShift Service Mesh.

### Supported platforms {#ossm-supported-platforms_preparing-ossm-installation}

The Red Hat OpenShift Service Mesh Operator supports multiple versions of the `ServiceMeshControlPlane` resource. Version 2.6 Service Mesh control planes are supported on the following platform versions:

- Red Hat OpenShift Container Platform version 4.10 or later
- Red Hat OpenShift Dedicated version 4
- Azure Red Hat OpenShift (ARO) version 4
- Red Hat OpenShift Service on AWS
- Red Hat OpenShift Service on AWS classic architecture

### Unsupported configurations {#ossm-unsupported-configurations_preparing-ossm-installation}

Explicitly unsupported cases include:

- OpenShift Online is not supported for Red Hat OpenShift Service Mesh.
- Red Hat OpenShift Service Mesh does not support the management of microservices outside the cluster where Service Mesh is running.

### Supported network configurations {#ossm-supported-configurations-networks_preparing-ossm-installation}

Red Hat OpenShift Service Mesh supports the following network configurations.

- OpenShift-SDN
- OVN-Kubernetes is available on all supported versions of OpenShift Container Platform.
- Third-Party Container Network Interface (CNI) plugins that have been certified on OpenShift Container Platform and passed Service Mesh conformance testing. See [Certified OpenShift CNI Plug-ins](https://access.redhat.com/articles/5436171) for more information.

### Supported configurations for Service Mesh {#ossm-supported-configurations-sm_preparing-ossm-installation}

- This release of Red Hat OpenShift Service Mesh is only available on OpenShift Container Platform x86_64, IBM Z(R), and IBM Power(R).
  - IBM Z(R) is only supported on OpenShift Container Platform 4.10 and later.
  - IBM Power(R) is only supported on OpenShift Container Platform 4.10 and later.
- Configurations where all Service Mesh components are contained within a single OpenShift Container Platform cluster.
- Configurations that do not integrate external services such as virtual machines.
- Red Hat OpenShift Service Mesh does not support `EnvoyFilter` configuration except where explicitly documented.

### Supported configurations for Kiali {#ossm-supported-configurations-kiali_preparing-ossm-installation}

- The Kiali console is only supported on the two most recent releases of the Google Chrome, Microsoft Edge, Mozilla Firefox, or Apple Safari browsers.
- The `openshift` authentication strategy is the only supported authentication configuration when Kiali is deployed with Red Hat OpenShift Service Mesh (OSSM). The `openshift` strategy controls access based on the individual’s role-based access control (RBAC) roles of the OpenShift Container Platform.

### Supported configurations for Distributed Tracing {#ossm-supported-configurations-jaeger_preparing-ossm-installation}

- Jaeger agent as a sidecar is the only supported configuration for Jaeger. Jaeger as a daemonset is not supported for multitenant installations or OpenShift Dedicated.

### Supported WebAssembly module {#ossm-supported-configurations-webassembly_preparing-ossm-installation}

- 3scale WebAssembly is the only provided WebAssembly module. You can create custom WebAssembly modules.

## Next steps {#_next_steps}

- [Install Red Hat OpenShift Service Mesh](/docs/service_mesh/v2x/installing-ossm#installing-ossm) in your OpenShift Container Platform environment.
