---
title: Learn more about OpenShift Container Platform
sidebar_position: 3
---

# Learn more about OpenShift Container Platform {#learn_more_about_openshift}

<a id="learn_more_about_openshift"></a>

To better use OpenShift Container Platform, you should first learn about and better understand how OpenShift Container Platform functions.

You can use the following sections to find content to help you learn about OpenShift Container Platform.

Learning and support
: | Learn about OpenShift Container Platform | Optional additional resources |
  | --- | --- |
  | [What’s new in OpenShift Container Platform](https://www.openshift.com/learn/whats-new) | [OpenShift blog](https://www.openshift.com/blog?hsLang=en-us) |
  | [OpenShift Container Platform Life Cycle Policy](https://access.redhat.com/support/policy/updates/openshift) | [OpenShift Container Platform life cycle](https://access.redhat.com/support/policy/updates/openshift#ocp4_phases) |
  | [OpenShift Interactive Learning Portal](https://learn.openshift.com/?extIdCarryOver=true&sc_cid=701f2000001Css5AAC) | [OpenShift Knowledgebase articles](https://access.redhat.com/articles/4217411) |
  | [Getting Support](/docs/support/getting-support#getting-support) | [Gathering data about your cluster](/docs/support/gathering-cluster-data#gathering-data) |

Architecture
: | Learn about OpenShift Container Platform | Optional additional resources |
  | --- | --- |
  | [Enterprise Kubernetes with OpenShift](https://www.openshift.com/blog/enterprise-kubernetes-with-openshift-part-one?extIdCarryOver=true&sc_cid=701f2000001Css5AAC) | [Tested platforms](https://access.redhat.com/articles/4128421) |
  | [Architecture](/docs/architecture/#architecture) | [Security and compliance](/docs/security/container_security/security-understanding#understanding-security) |
  | [Networking](/docs/networking/networking_overview/understanding-networking#understanding-networking) | [OVN-Kubernetes architecture](/docs/networking/ovn_kubernetes_network_provider/ovn-kubernetes-architecture-assembly#ovn-kubernetes-architecture-con) |
  | [Backup and restore](/docs/backup_and_restore#backup-restore-overview) | [Restoring to an earlier cluster state](/docs/backup_and_restore/control_plane_backup_and_restore/disaster_recovery/scenario-2-restoring-cluster-state#scenario-2-restoring-cluster-state) |

Installation
: Explore the following OpenShift Container Platform installation tasks:

  | Learn about installation on OpenShift Container Platform | Optional additional resources |
  | --- | --- |
  | [OpenShift Container Platform installation overview](/docs/installing/overview#ocp-installation-overview) | [Selecting a cluster installation method and preparing it for users](/docs/installing/overview/installing-preparing#installing-preparing) |
  | [Installing a cluster in FIPS mode](/docs/installing/overview/installing-fips#installing-fips-mode_installing-fips) | [About FIPS compliance](/docs/installing/installing_with_agent_based_installer/preparing-to-install-with-agent-based-installer#agent-installer-fips-compliance_preparing-to-install-with-agent-based-installer) |

Other cluster installer tasks
: | Learn about other installer tasks on OpenShift Container Platform | Optional additional resources |
  | --- | --- |
  | [Troubleshooting installation issues](/docs/installing/validation_and_troubleshooting/installing-troubleshooting#installing-troubleshooting) | [Validating an installation](/docs/installing/validation_and_troubleshooting/validating-an-installation#validating-an-installation) |
  | [Install Red Hat OpenShift Data Foundation](/docs/storage/persistent_storage/persistent-storage-ocs#red-hat-openshift-data-foundation) | [image mode for OpenShift](/docs/machine_configuration/mco-coreos-layering#mco-coreos-layering) |

Install a cluster in a restricted network

<table>
<thead>
<tr>
  <th>Learn about installing in a restricted network</th>
  <th>Optional additional resources</th>
</tr>
</thead>
<tbody>
<tr>
  <td><a href="/docs/disconnected#index">About disconnected installation mirroring</a></td>
  <td>If your cluster uses user-provisioned infrastructure, and the cluster does not have full access to the internet, you must mirror the OpenShift Container Platform installation images.<br><br><ul><li><a href="/docs/installing/installing_aws/upi/installing-restricted-networks-aws#installing-restricted-networks-aws">Amazon Web Services (AWS)</a></li><li><a href="/docs/installing/installing_gcp/installing-restricted-networks-gcp#installing-restricted-networks-gcp">Google Cloud</a></li><li><a href="/docs/installing/installing_vsphere/upi/installing-restricted-networks-vsphere#installing-restricted-networks-vsphere">vSphere</a></li><li><a href="/docs/installing/installing_ibm_cloud/installing-ibm-cloud-restricted#installing-ibm-cloud-restricted">IBM Cloud(R)</a></li><li><a href="/docs/installing/installing_ibm_z/preparing-to-install-on-ibm-z#preparing-to-install-on-ibm-z">IBM Z(R) and IBM(R) LinuxONE</a></li><li><a href="/docs/installing/installing_ibm_power/installing-restricted-networks-ibm-power#installing-restricted-networks-ibm-power">IBM Power(R)</a></li><li><a href="/docs/installing/installing_bare_metal/upi/installing-restricted-networks-bare-metal#installing-restricted-networks-bare-metal">bare metal</a></li></ul></td>
</tr>
</tbody>
</table>

Install a cluster in an existing network
: | Learn about installing in a restricted network | Optional additional resources |
  | --- | --- |
  | If you use an existing Virtual Private Cloud (VPC) in [Amazon Web Services (AWS)](/docs/installing/installing_aws/ipi/installing-aws-vpc#installing-aws-vpc) or [Google Cloud](/docs/installing/installing_gcp/installing-gcp-vpc#installing-gcp-vpc) or an existing [VNet](/docs/installing/installing_azure/ipi/installing-azure-vnet#installing-azure-vnet) on Microsoft Azure, you can install a cluster | [Installing a cluster on Google Cloud into a shared VPC](/docs/installing/installing_gcp/installing-gcp-shared-vpc#installation-gcp-shared-vpc-prerequisites_installing-gcp-shared-vpc) |

Cluster administration

<table>
<thead>
<tr>
  <th>Learn about OpenShift Container Platform cluster activities</th>
  <th>Optional additional resources</th>
</tr>
</thead>
<tbody>
<tr>
  <td><a href="/docs/architecture/#architecture-overview-architecture">Understand OpenShift Container Platform management</a></td>
  <td><ul><li><a href="/docs/machine_management#machine-api-overview_overview-of-machine-management">Machine API</a></li><li><a href="/docs/architecture/control-plane#operators-overview_control-plane">Operators</a></li><li><a href="/docs/etcd/etcd-overview#etc-overview">etcd</a></li></ul></td>
</tr>
<tr>
  <td><a href="/docs/installing/overview/cluster-capabilities#enabling-cluster-capabilities_cluster-capabilities">Enable cluster capabilities</a></td>
  <td><a href="/docs/installing/overview/cluster-capabilities#explanation_of_capabilities_cluster-capabilities">Optional cluster capabilities in OpenShift Container Platform 4.22</a></td>
</tr>
</tbody>
</table>

Managing cluster components

<table>
<thead>
<tr>
  <th>Learn about managing cluster components</th>
  <th>Optional additional resources</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Manage <a href="/docs/machine_management#machine-mgmt-intro-managing-compute_overview-of-machine-management">compute</a> and <a href="/docs/machine_management#machine-mgmt-intro-managing-control-plane_overview-of-machine-management">control plane</a> machines with machine sets</td>
  <td><a href="/docs/machine_management/deploying-machine-health-checks#deploying-machine-health-checks">Deploy machine health checks</a></td>
</tr>
<tr>
  <td><a href="/docs/machine_management/applying-autoscaling#applying-autoscaling">Apply autoscaling to an OpenShift Container Platform cluster</a></td>
  <td><a href="/docs/nodes/pods/nodes-pods-priority#nodes-pods-priority">Including pod priority in pod scheduling decisions</a></td>
</tr>
<tr>
  <td><a href="/docs/registry#registry-overview">Manage container registries</a></td>
  <td><a href="https://access.redhat.com/documentation/en-us/red_hat_quay/">Red&#160;Hat Quay</a></td>
</tr>
<tr>
  <td><a href="/docs/authentication/understanding-authentication#understanding-authentication">Manage users and groups</a></td>
  <td><a href="/docs/authentication/impersonating-system-admin#impersonating-system-admin">Impersonating the system:admin user</a></td>
</tr>
<tr>
  <td><a href="/docs/authentication/understanding-authentication#understanding-authentication">Manage authentication</a></td>
  <td><a href="/docs/authentication/understanding-identity-provider#supported-identity-providers_understanding-identity-provider">Supported identity providers</a></td>
</tr>
<tr>
  <td>Manage <a href="/docs/security/certificates/replacing-default-ingress-certificate#replacing-default-ingress">Ingress</a>, <a href="/docs/security/certificates/api-server#api-server-certificates">API server</a>, and <a href="/docs/security/certificates/service-serving-certificate#add-service-serving">Service</a> certificates</td>
  <td><a href="/docs/networking/network_security/network-policy-apis#network-policy-apis">Network security</a></td>
</tr>
<tr>
  <td><a href="/docs/networking/networking_overview/understanding-networking#understanding-networking">Manage networking</a></td>
  <td><ul><li><a href="/docs/networking/networking_operators/cluster-network-operator#nw-cluster-network-operator_cluster-network-operator">Cluster Network Operator</a></li><li><a href="/docs/networking/multiple_networks/understanding-multiple-networks#understanding-multiple-networks">Multiple network interfaces</a></li><li><a href="/docs/networking/network_security/network_policy/about-network-policy#about-network-policy">Network policy</a></li></ul></td>
</tr>
<tr>
  <td><a href="/docs/operators/understanding/olm-understanding-software-catalog#olm-understanding-software-catalog">Manage Operators</a></td>
  <td><a href="/docs/operators/user/olm-creating-apps-from-installed-operators#olm-creating-apps-from-installed-operators">Creating applications from installed Operators</a></td>
</tr>
</tbody>
</table>

Changing cluster components

<table>
<thead>
<tr>
  <th>Learn more about changing cluster components</th>
  <th>Optional additional resources</th>
</tr>
</thead>
<tbody>
<tr>
  <td><a href="/docs/updating/understanding_updates/intro-to-updates#intro-to-updates">Introduction to OpenShift updates</a></td>
  <td><ul><li><a href="/docs/updating/updating_a_cluster/updating-cluster-web-console#updating-cluster-web-console">Updating a cluster using the web console</a></li><li><a href="/docs/updating/updating_a_cluster/updating-cluster-cli#updating-cluster-cli">Updating using the CLI</a></li><li><a href="/docs/disconnected/updating#about-disconnected-updates">Using the OpenShift Update Service in a disconnected environment</a></li></ul></td>
</tr>
<tr>
  <td><a href="/docs/operators/understanding/crds/crd-extending-api-with-crds#crd-extending-api-with-crds">Use custom resource definitions (CRDs) to modify the cluster</a></td>
  <td><ul><li><a href="/docs/operators/understanding/crds/crd-extending-api-with-crds#crd-creating-custom-resources-definition_crd-extending-api-with-crds">Create a CRD</a></li><li><a href="/docs/operators/understanding/crds/crd-managing-resources-from-crds#crd-managing-resources-from-crds">Manage resources from CRDs</a></li></ul></td>
</tr>
<tr>
  <td><a href="/docs/applications/quotas/quotas-setting-per-project#quotas-setting-per-project">Set resource quotas</a></td>
  <td><a href="/docs/applications/quotas/quotas-setting-across-multiple-projects#quotas-setting-across-multiple-projects">Resource quotas across multiple projects</a></td>
</tr>
<tr>
  <td><a href="/docs/applications/pruning-objects#pruning-objects">Prune and reclaim resources</a></td>
  <td><a href="/docs/cicd/builds/advanced-build-operations#builds-build-pruning-advanced-build-operations">Performing advanced builds</a></td>
</tr>
<tr>
  <td><a href="/docs/scalability_and_performance/recommended-performance-scale-practices/recommended-infrastructure-practices#scaling-cluster-monitoring-operator">Scale</a> and <a href="/docs/scalability_and_performance/using-node-tuning-operator#using-node-tuning-operator">tune</a> clusters</td>
  <td><a href="/docs/scalability_and_performance#scalability-and-performance-overview">OpenShift Container Platform scalability and performance</a></td>
</tr>
</tbody>
</table>

Observe a cluster

<table>
<thead>
<tr>
  <th>Learn about OpenShift Container Platform</th>
  <th>Optional additional resources</th>
</tr>
</thead>
<tbody>
<tr>
  <td><a href="https://docs.redhat.com/en/documentation/red_hat_openshift_distributed_tracing_platform/latest/html/release_notes_for_the_distributed_tracing_platform/distr-tracing-rn">Release notes for the Red&#160;Hat OpenShift Distributed Tracing Platform</a></td>
  <td><a href="https://docs.redhat.com/en/documentation/red_hat_openshift_distributed_tracing_platform/latest">Red&#160;Hat OpenShift Distributed Tracing Platform</a></td>
</tr>
<tr>
  <td><a href="https://docs.redhat.com/en/documentation/red_hat_build_of_opentelemetry/latest/html/installing_red_hat_build_of_opentelemetry/install-otel">Red Hat build of OpenTelemetry</a></td>
  <td><a href="https://docs.redhat.com/en/documentation/red_hat_build_of_opentelemetry/latest/html/receiving_telemetry/otel-receiving-telemetry#otel-receiving-telemetry-from-multiple-clusters_otel-receiving-telemetry">Receiving telemetry data from multiple clusters</a></td>
</tr>
<tr>
  <td><a href="/docs/observability/network_observability/network-observability-overview#network-observability-overview">About Network Observability</a></td>
  <td><ul><li><a href="/docs/observability/network_observability/metrics-alerts-dashboards#metrics-alerts-dashboards_metrics-alerts-dashboards">Using metrics with dashboards and alerts</a></li><li><a href="/docs/observability/network_observability/observing-network-traffic#network-observability-trafficflow_nw-observe-network-traffic">Observing the network traffic from the Traffic flows view</a></li></ul></td>
</tr>
<tr>
  <td><a href="https://docs.redhat.com/en/documentation/monitoring_stack_for_red_hat_openshift/latest/html/about_monitoring/about-ocp-monitoring">About OpenShift Container Platform monitoring</a></td>
  <td><ul><li><a href="/docs/support/remote_health_monitoring/about-remote-health-monitoring#about-remote-health-monitoring_about-remote-health-monitoring">Remote health monitoring</a></li><li><a href="https://docs.redhat.com/en/documentation/power_monitoring_for_red_hat_openshift/latest/html/about_power_monitoring/about-power-monitoring">Power monitoring for Red Hat OpenShift (Technology Preview)</a></li></ul></td>
</tr>
</tbody>
</table>

Storage activities

<table>
<thead>
<tr>
  <th>Learn about OpenShift Container Platform</th>
  <th>Optional additional resources</th>
</tr>
</thead>
<tbody>
<tr>
  <td><a href="/docs/storage#storage-types">Storage types</a></td>
  <td><ul><li><a href="/docs/storage/understanding-persistent-storage#understanding-persistent-storage">Persistent storage</a></li><li><a href="/docs/storage/understanding-ephemeral-storage#understanding-ephemeral-storage">Ephemeral storage</a></li></ul></td>
</tr>
</tbody>
</table>

Application Site Reliability Engineer (App SRE)
: | Learn about OpenShift Container Platform | Optional additional resources |
  | --- | --- |
  | [Building applications overview](/docs/applications#building-applications-overview) | [Projects](/docs/applications/projects/working-with-projects#working-with-projects) |
  | [Operators](/docs/operators/understanding/olm-what-operators-are#olm-what-operators-are) | [Cluster Operator reference](/docs/operators/operator-reference#cluster-operator-reference) |

Developing applications
: OpenShift Container Platform is a platform for developing and deploying containerized applications. Read the following OpenShift Container Platform documentation, so that you can better understand OpenShift Container Platform functions:

<table>
<thead>
<tr>
  <th>Learn about application development in OpenShift Container Platform</th>
  <th>Optional additional resources</th>
</tr>
</thead>
<tbody>
<tr>
  <td><a href="https://developers.redhat.com/products/openshift/getting-started#assembly-field-sections-13455">Getting started with OpenShift for developers (interactive tutorial)</a></td>
  <td><ul><li><a href="/docs/architecture/understanding-development#understanding-development">Understanding OpenShift Container Platform development</a></li><li><a href="/docs/applications/projects/working-with-projects#working-with-projects">Working with projects</a></li><li><a href="/docs/applications/deployments/what-deployments-are#what-deployments-are">Create deployments</a></li></ul></td>
</tr>
<tr>
  <td><a href="https://developers.redhat.com/">Red Hat Developers site</a></td>
  <td><a href="/docs/cicd/builds/understanding-image-builds#understanding-image-builds">Understanding image builds</a></td>
</tr>
<tr>
  <td><a href="https://developers.redhat.com/products/openshift-dev-spaces/overview">Red&#160;Hat OpenShift Dev Spaces (formerly Red Hat CodeReady Workspaces)</a></td>
  <td><a href="/docs/operators/understanding/olm-what-operators-are#olm-what-operators-are">Operators</a></td>
</tr>
<tr>
  <td><a href="/docs/openshift_images#overview-of-images">Create container images</a></td>
  <td><a href="/docs/openshift_images/managing_images/managing-images-overview#managing-images-overview">Managing images overview</a></td>
</tr>
<tr>
  <td><a href="https://odo.dev/docs/introduction/"><code>odo</code></a></td>
  <td><a href="/docs/cli_reference/odo-important-update#odo-important_update">Developer-focused CLI</a></td>
</tr>
<tr>
  <td><a href="/docs/applications/odc-viewing-application-composition-using-topology-view#odc-viewing-application-topology_viewing-application-composition-using-topology-view">Viewing application composition using the Topology view</a></td>
  <td><a href="/docs/applications/odc-exporting-applications#odc-exporting-applications">Exporting applications</a></td>
</tr>
<tr>
  <td><a href="https://docs.openshift.com/pipelines/1.15/about/understanding-openshift-pipelines.html">Understanding OpenShift Pipelines</a></td>
  <td><a href="https://docs.openshift.com/pipelines/latest/create/creating-applications-with-cicd-pipelines.html">Create CI/CD Pipelines</a></td>
</tr>
<tr>
  <td><a href="https://docs.openshift.com/gitops/latest/declarative_clusterconfig/configuring-an-openshift-cluster-by-deploying-an-application-with-cluster-configurations.html">Configuring an OpenShift cluster by deploying an application with cluster configurations</a></td>
  <td><ul><li><a href="/docs/nodes/scheduling/nodes-scheduler-taints-tolerations#nodes-scheduler-taints-tolerations">Controlling pod placement using node taints</a></li><li><a href="/docs/machine_management/creating-infrastructure-machinesets#creating-infrastructure-machinesets">Creating infrastructure machine sets</a></li></ul></td>
</tr>
</tbody>
</table>

Hosted control planes

<table>
<thead>
<tr>
  <th>Learn about hosted control planes</th>
  <th>Optional additional resources</th>
</tr>
</thead>
<tbody>
<tr>
  <td><a href="/docs/hosted_control_planes#hosted-control-planes-overview">Hosted control planes overview</a></td>
  <td><a href="/docs/hosted_control_planes#hosted-control-planes-version-support_hcp-overview">Versioning for hosted control planes</a></td>
</tr>
<tr>
  <td>Preparing to deploy</td>
  <td><ul><li><a href="/docs/hosted_control_planes/hcp-prepare/hcp-requirements#hcp-requirements">Requirements for hosted control planes</a></li><li><a href="/docs/hosted_control_planes/hcp-prepare/hcp-sizing-guidance#hcp-sizing-guidance">Sizing guidance for hosted control planes</a></li><li><a href="/docs/hosted_control_planes/hcp-prepare/hcp-override-resource-util#hcp-override-resource-util">Overriding resource utilization measurements</a></li><li><a href="/docs/hosted_control_planes/hcp-prepare/hcp-cli#hcp-cli">Installing the hosted control planes command-line interface</a></li><li><a href="/docs/hosted_control_planes/hcp-prepare/hcp-distribute-workloads#hcp-distribute-workloads">Distributing hosted cluster workloads</a></li><li><a href="/docs/hosted_control_planes/hcp-prepare/hcp-enable-disable#hcp-enable-disable">Enabling or disabling the hosted control planes feature</a></li></ul></td>
</tr>
<tr>
  <td>Deploying hosted control planes</td>
  <td><ul><li><a href="/docs/hosted_control_planes/hcp-deploy/hcp-deploy-virt#hcp-deploy-virt">Deploying hosted control planes on OpenShift Virtualization</a></li><li><a href="/docs/hosted_control_planes/hcp-deploy/hcp-deploy-aws#hcp-deploy-aws">Deploying hosted control planes on AWS</a></li><li><a href="/docs/hosted_control_planes/hcp-deploy/hcp-deploy-bm#hcp-deploy-bm">Deploying hosted control planes on bare metal</a></li><li><a href="/docs/hosted_control_planes/hcp-deploy/hcp-deploy-non-bm#hcp-deploy-non-bm">Deploying hosted control planes on non-bare-metal agent machines</a></li><li><a href="/docs/hosted_control_planes/hcp-deploy/hcp-deploy-ibmz#hcp-deploy-ibmz">Deploying hosted control planes on IBM Z</a></li><li><a href="/docs/hosted_control_planes/hcp-deploy/hcp-deploy-ibm-power#hcp-deploy-ibm-power">Deploying hosted control planes on IBM Power</a></li></ul></td>
</tr>
<tr>
  <td>Deploying hosted control planes in a disconnected environment</td>
  <td><ul><li><a href="/docs/hosted_control_planes/hcp-disconnected/hcp-deploy-dc-bm#hcp-deploy-dc-bm">Deploying hosted control planes on bare metal in a disconnected environment</a></li><li><a href="/docs/hosted_control_planes/hcp-disconnected/hcp-deploy-dc-virt#hcp-deploy-dc-virt">Deploying hosted control planes on OpenShift Virtualization in a disconnected environment</a></li></ul></td>
</tr>
<tr>
  <td><a href="/docs/hosted_control_planes/hcp-troubleshooting#hcp-troubleshooting">Troubleshooting hosted control planes</a></td>
  <td><a href="/docs/hosted_control_planes/hcp-troubleshooting#hosted-control-planes-troubleshooting_hcp-troubleshooting">Gathering information to troubleshoot hosted control planes</a></td>
</tr>
</tbody>
</table>
