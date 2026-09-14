---
title: Installing a cluster on user-provisioned infrastructure in Google Cloud by using Infrastructure Manager templates
sidebar_position: 9
---

# Installing a cluster on user-provisioned infrastructure in Google Cloud by using Infrastructure Manager templates {#installing-gcp-user-infra}

<a id="installing-gcp-user-infra"></a>

In OpenShift Container Platform version 4.22, you can install a cluster on Google Cloud that uses infrastructure that you provide.

The steps for performing a user-provided infrastructure install are outlined here. Several Infrastructure Manager templates are provided to assist in completing these steps or to help model your own. You are also free to create the required resources through other methods.

:::warning

The steps for performing a user-provisioned infrastructure installation are provided as an example only. Installing a cluster with infrastructure you provide requires knowledge of the cloud provider and the installation process of OpenShift Container Platform. Several Infrastructure Manager templates are provided to assist in completing these steps or to help model your own. You are also free to create the required resources through other methods; the templates are just an example.

:::

## Prerequisites {#_prerequisites}

- You reviewed details about the OpenShift Container Platform installation and update processes. For more information, see "Installation and update".
- You read the documentation on selecting a cluster installation method and preparing it for users. For more information, see "Selecting a cluster installation method and preparing it for users".
- If you use a firewall and plan to use the Telemetry service, you configured the firewall to allow the sites that your cluster requires access to. For more information, see "Configuring your firewall for OpenShift Container Platform".
- If the cloud identity and access management (IAM) APIs are not accessible in your environment, or if you do not want to store an administrator-level credential secret in the `kube-system` namespace, you can manually create and maintain long-term credentials. For more information, see "Manually creating long-term credentials".
  :::note

  Be sure to also review this site list if you are configuring a proxy.

  :::

**Additional resources**

- [Installation and update](/docs/architecture/architecture-installation#architecture-installation)
- [Selecting a cluster installation method and preparing it for users](/docs/installing/overview/installing-preparing#installing-preparing)
- [Configuring your firewall for OpenShift Container Platform](/docs/installing/install_config/configuring-firewall#configuring-firewall-module_configuring-firewall)
- [Manually creating long-term credentials](/docs/installing/installing_gcp/installing-gcp-customizations#manually-create-iam_installing-gcp-customizations)

## Certificate signing requests management {#csr-management_installing-gcp-user-infra}

On user-provisioned infrastructure, you must implement a mechanism for approving cluster certificate signing requests (CSRs) after installation when your cluster has limited access to automatic machine management.

The `kube-controller-manager` only approves the kubelet client CSRs. The `machine-approver` cannot guarantee the validity of a serving certificate that kubelet credentials request because it cannot confirm that the correct machine issued the request. You must find and implement a method of verifying the validity of the kubelet serving certificate requests and approving them.

## Internet access for OpenShift Container Platform {#cluster-entitlements_installing-gcp-user-infra}

In OpenShift Container Platform 4.22, you require access to the internet to install your cluster.

You must have internet access to perform the following actions:

- Access Red Hat Hybrid Cloud Console to download the installation program and perform subscription management. If the cluster has internet access and you do not disable Telemetry, that service automatically entitles your cluster.
- Access Quay.io to obtain the packages that are required to install your cluster.
- Obtain the packages that are required to perform cluster updates.

:::warning

If your cluster cannot have direct internet access, you can perform a restricted network installation on some types of infrastructure that you provision. During that process, you download the required content and use it to populate a mirror registry with the installation packages. With some installation types, the environment that you install your cluster in will not require internet access. Before you update the cluster, you update the content of the mirror registry.

:::

## Configuring your Google Cloud project {#installation-gcp-user-infra-config-project_installing-gcp-user-infra}

Before you can install OpenShift Container Platform, you must configure a Google Cloud project to host it. Proper project configuration provides the API services, service account, and permissions that the installation requires.

### Creating a Google Cloud project {#installation-gcp-project_installing-gcp-user-infra}

To install OpenShift Container Platform, you must create a project in your Google Cloud account to host the cluster.

**Procedure**

- Create a project to host your OpenShift Container Platform cluster. See [Creating and Managing Projects](https://cloud.google.com/resource-manager/docs/creating-managing-projects) in the Google Cloud documentation.
  :::warning

  Your Google Cloud project must use the Premium Network Service Tier if you are using installer-provisioned infrastructure. The Standard Network Service Tier is not supported for clusters installed using the installation program. The installation program configures internal load balancing for the `api-int.<cluster_name>.<base_domain>` URL; the Premium Tier is required for internal load balancing.

  :::

### Enabling API services in Google Cloud {#installation-gcp-enabling-api-services_installing-gcp-user-infra}

You must enable several API services in your Google Cloud project to complete OpenShift Container Platform installation.

**Prerequisites**

- You created a project to host your cluster.

**Procedure**

- Enable the following required API services in the project that hosts your cluster. You can also enable optional API services which are not required for installation. See [Enabling services](https://cloud.google.com/service-usage/docs/enable-disable#enabling) in the Google Cloud documentation.
  **Required API services**

<table>
<thead>
<tr>
  <th>API service</th>
  <th>Console service name</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Compute Engine API</td>
  <td><code>compute.googleapis.com</code></td>
</tr>
<tr>
  <td>Cloud Resource Manager API</td>
  <td><code>cloudresourcemanager.googleapis.com</code></td>
</tr>
<tr>
  <td>Cloud DNS API</td>
  <td><code>dns.googleapis.com</code></td>
</tr>
<tr>
  <td>IAM Service Account Credentials API</td>
  <td><code>iamcredentials.googleapis.com</code></td>
</tr>
<tr>
  <td>Identity and Access Management (IAM) API</td>
  <td><code>iam.googleapis.com</code></td>
</tr>
<tr>
  <td>Service Usage API</td>
  <td><code>serviceusage.googleapis.com</code></td>
</tr>
</tbody>
</table>

```
**Optional API services**
```

<table>
<thead>
<tr>
  <th>API service</th>
  <th>Console service name</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Cloud Deployment Manager V2 API</td>
  <td><code>deploymentmanager.googleapis.com</code></td>
</tr>
<tr>
  <td>Google Cloud APIs</td>
  <td><code>cloudapis.googleapis.com</code></td>
</tr>
<tr>
  <td>Service Management API</td>
  <td><code>servicemanagement.googleapis.com</code></td>
</tr>
<tr>
  <td>Google Cloud Storage JSON API</td>
  <td><code>storage-api.googleapis.com</code></td>
</tr>
<tr>
  <td>Cloud Storage</td>
  <td><code>storage-component.googleapis.com</code></td>
</tr>
</tbody>
</table>

### Configuring DNS for Google Cloud {#installation-gcp-dns_installing-gcp-user-infra}

Configure a public hosted zone in your Google Cloud account to provide DNS resolution and name lookup for your OpenShift Container Platform cluster.

To install OpenShift Container Platform, the Google Cloud account you use must have a dedicated public hosted zone in the same project that you host the OpenShift Container Platform cluster. This zone must be authoritative for the domain. The DNS service provides cluster DNS resolution and name lookup for external connections to the cluster.

**Procedure**

1. Identify your domain, or subdomain, and registrar. You can transfer an existing domain and registrar or obtain a new one through Google Cloud or another source.
   :::note

   If you purchase a new domain, it can take time for the relevant DNS changes to propagate. For more information about purchasing domains through Google, see [Google Domains](https://domains.google/).

   :::
2. Create a public hosted zone for your domain or subdomain in your Google Cloud project. See [Creating public zones](https://cloud.google.com/dns/zones/#creating_public_zones) in the Google Cloud documentation.
   Use an appropriate root domain, such as `openshiftcorp.com`, or subdomain, such as `clusters.openshiftcorp.com`.
3. Extract the new authoritative name servers from the hosted zone records. See [Look up your Cloud DNS name servers](https://cloud.google.com/dns/docs/update-name-servers#look_up_your_name_servers) in the Google Cloud documentation.
   You typically have four name servers.
4. Update the registrar records for the name servers that your domain uses. For example, if you registered your domain to Google Domains, see the following topic in the Google Domains Help: [How to switch to custom name servers](https://support.google.com/domains/answer/3290309?hl=en).
5. If you migrated your root domain to Google Cloud DNS, migrate your DNS records. See [Migrating to Cloud DNS](https://cloud.google.com/dns/docs/migrating) in the Google Cloud documentation.
6. If you use a subdomain, follow your company’s procedures to add its delegation records to the parent domain. This process might include a request to your company’s IT department or the division that controls the root domain and DNS services for your company.

### Google Cloud account limits {#installation-gcp-limits_installing-gcp-user-infra}

A default OpenShift Container Platform cluster consumes specific Google Cloud resource quotas that you might need to increase before installation, depending on your region and cluster size.

A default cluster, which contains three compute and three control plane machines, uses the following resources. Note that some resources are required only during the bootstrap process and are removed after the cluster deploys.

**Google Cloud resources used in a default cluster**

<table>
<thead>
<tr>
  <th>Service</th>
  <th>Component</th>
  <th>Location</th>
  <th>Total resources required</th>
  <th>Resources removed after bootstrap</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Service account</td>
  <td>IAM</td>
  <td>Global</td>
  <td>6</td>
  <td>1</td>
</tr>
<tr>
  <td>Firewall rules</td>
  <td>Networking</td>
  <td>Global</td>
  <td>11</td>
  <td>1</td>
</tr>
<tr>
  <td>Forwarding rules</td>
  <td>Compute</td>
  <td>Global</td>
  <td>2</td>
  <td>0</td>
</tr>
<tr>
  <td>Health checks</td>
  <td>Compute</td>
  <td>Global</td>
  <td>2</td>
  <td>0</td>
</tr>
<tr>
  <td>Images</td>
  <td>Compute</td>
  <td>Global</td>
  <td>1</td>
  <td>0</td>
</tr>
<tr>
  <td>Networks</td>
  <td>Networking</td>
  <td>Global</td>
  <td>1</td>
  <td>0</td>
</tr>
<tr>
  <td>Routers</td>
  <td>Networking</td>
  <td>Global</td>
  <td>1</td>
  <td>0</td>
</tr>
<tr>
  <td>Routes</td>
  <td>Networking</td>
  <td>Global</td>
  <td>2</td>
  <td>0</td>
</tr>
<tr>
  <td>Subnetworks</td>
  <td>Compute</td>
  <td>Global</td>
  <td>2</td>
  <td>0</td>
</tr>
<tr>
  <td>Target pools</td>
  <td>Networking</td>
  <td>Global</td>
  <td>2</td>
  <td>0</td>
</tr>
</tbody>
</table>

:::note

If any of the quotas are insufficient during installation, the installation program displays an error that states both which quota was exceeded and the region.

:::

Be sure to consider your actual cluster size, planned cluster growth, and any usage from other clusters that are associated with your account. The CPU, static IP addresses, and persistent disk SSD (storage) quotas are the ones that are most likely to be insufficient.

If you plan to deploy your cluster in one of the following regions, you will exceed the maximum storage quota and are likely to exceed the CPU quota limit:

- `asia-east2`
- `asia-northeast2`
- `asia-south1`
- `australia-southeast1`
- `europe-north1`
- `europe-west2`
- `europe-west3`
- `europe-west6`
- `northamerica-northeast1`
- `southamerica-east1`
- `us-west2`

You can increase resource quotas from the Google Cloud console, but you might need to file a support ticket. Be sure to plan your cluster size early so that you can allow time to resolve the support ticket before you install your OpenShift Container Platform cluster.

**Additional resources**

- [Manage your quotas using the console (Google Cloud documentation)](https://cloud.google.com/docs/quota)
- [Google Cloud console](https://console.cloud.google.com/iam-admin/quotas)

### Creating a service account in Google Cloud {#installation-gcp-service-account_installing-gcp-user-infra}

OpenShift Container Platform requires a Google Cloud service account that provides authentication and authorization to access data in the Google APIs. If you do not have an existing IAM service account that contains the required roles in your project, you must create one.

:::note

To reduce the scope of permissions granted to the main service account in your Google Cloud project while still being able to use the Google Cloud Container Storage Interface (CSI) Driver Operator, you can transfer the control of permissions from the project-wide service account to the control plane and compute node service accounts instead, thus reducing the scope of the permission. For more information, see Section *Reducing permissions while using the Google Cloud CSI Driver Operator*.

:::

**Prerequisites**

- You created a project to host your cluster.

**Procedure**

1. Create a service account in the project that you use to host your OpenShift Container Platform cluster. See [Creating a service account](https://cloud.google.com/iam/docs/creating-managing-service-accounts#creating_a_service_account) in the Google Cloud documentation.
2. Grant the service account the appropriate permissions. You can either grant the individual permissions that follow or assign the `Owner` role to it. See [Granting roles to a service account for specific resources](https://cloud.google.com/iam/docs/granting-roles-to-service-accounts#granting_access_to_a_service_account_for_a_resource).
   :::note

   While making the service account an owner of the project is the easiest way to gain the required permissions, it means that service account has complete control over the project. You must determine if the risk that comes from offering that power is acceptable.

   :::
3. You can create the service account key in JSON format, or attach the service account to a Google Cloud virtual machine. See [Creating service account keys](https://cloud.google.com/iam/docs/creating-managing-service-account-keys#creating_service_account_keys) and [Creating and enabling service accounts for instances](https://cloud.google.com/compute/docs/access/create-enable-service-accounts-for-instances) in the Google Cloud documentation.
   :::note

   If you use a virtual machine with an attached service account to create your cluster, you must set `credentialsMode: Manual` in the `install-config.yaml` file before installation.

   :::

### Required Google Cloud roles {#installation-gcp-permissions_installing-gcp-user-infra}

Your Google Cloud service account requires specific roles to install and manage an OpenShift Container Platform cluster, which you can scope based on your organization’s security requirements.

When you attach the `Owner` role to the service account that you create, you grant that service account all permissions, including those that are required to install OpenShift Container Platform. If your organization’s security policies require a more restrictive set of permissions, you can create a service account with the following permissions. If you deploy your cluster into an existing virtual private cloud (VPC), the service account does not require certain networking permissions, which are noted in the following lists:

The installation program requires the following roles:

- Compute Admin
- Role Administrator
- Security Admin
- Service Account Admin
- Service Account Key Admin
- Service Account User
- Storage Admin

Creating network resources during installation requires the following role:

- DNS Administrator

Using the Cloud Credential Operator in passthrough mode requires the following roles:

- Compute Load Balancer Admin
- Tag User

User-provisioned Google Cloud infrastructure requires the following role:

- Cloud Infrastructure Manager Admin

The following roles are applied to the service accounts that the control plane and compute machines use:

**Google Cloud service account roles**

<table>
<thead>
<tr>
  <th>Account</th>
  <th>Roles</th>
</tr>
</thead>
<tbody>
<tr>
  <td rowspan="5">Control Plane</td>
  <td><code>roles/compute.instanceAdmin</code></td>
</tr>
<tr>
  <td><code>roles/compute.networkAdmin</code></td>
</tr>
<tr>
  <td><code>roles/compute.securityAdmin</code></td>
</tr>
<tr>
  <td><code>roles/storage.admin</code></td>
</tr>
<tr>
  <td><code>roles/iam.serviceAccountUser</code></td>
</tr>
<tr>
  <td rowspan="3">Compute</td>
  <td><code>roles/compute.viewer</code></td>
</tr>
<tr>
  <td><code>roles/storage.admin</code></td>
</tr>
<tr>
  <td><code>roles/artifactregistry.reader</code></td>
</tr>
</tbody>
</table>

### Required Google Cloud permissions for user-provisioned infrastructure {#minimum-required-permissions-upi-gcp_installing-gcp-user-infra}

Your Google Cloud service account requires permissions to create and manage user-provisioned OpenShift Container Platform infrastructure. You can attach the `Owner` role to grant all permissions, or create a custom role with only the minimum permissions your organization’s security policies require.

If your organization’s security policies require a more restrictive set of permissions, you can create [custom roles](https://cloud.google.com/iam/docs/creating-custom-roles) with the necessary permissions. The following permissions are required for the user-provisioned infrastructure for creating and deleting the OpenShift Container Platform cluster.

The following permissions are required for creating network resources:

- `compute.addresses.create`
- `compute.addresses.createInternal`
- `compute.addresses.delete`
- `compute.addresses.get`
- `compute.addresses.list`
- `compute.addresses.use`
- `compute.addresses.useInternal`
- `compute.firewalls.create`
- `compute.firewalls.delete`
- `compute.firewalls.get`
- `compute.firewalls.list`
- `compute.forwardingRules.create`
- `compute.forwardingRules.get`
- `compute.forwardingRules.list`
- `compute.forwardingRules.setLabels`
- `compute.globalAddresses.create`
- `compute.globalAddresses.get`
- `compute.globalAddresses.use`
- `compute.globalForwardingRules.create`
- `compute.globalForwardingRules.get`
- `compute.globalForwardingRules.setLabels`
- `compute.networks.create`
- `compute.networks.get`
- `compute.networks.list`
- `compute.networks.updatePolicy`
- `compute.networks.use`
- `compute.routers.create`
- `compute.routers.get`
- `compute.routers.list`
- `compute.routers.update`
- `compute.routes.list`
- `compute.subnetworks.create`
- `compute.subnetworks.get`
- `compute.subnetworks.list`
- `compute.subnetworks.use`
- `compute.subnetworks.useExternalIp`

The following permissions are required for creating load balancer resources:

- `compute.backendServices.create`
- `compute.backendServices.get`
- `compute.backendServices.list`
- `compute.backendServices.update`
- `compute.backendServices.use`
- `compute.regionBackendServices.create`
- `compute.regionBackendServices.get`
- `compute.regionBackendServices.list`
- `compute.regionBackendServices.update`
- `compute.regionBackendServices.use`
- `compute.targetPools.addInstance`
- `compute.targetPools.create`
- `compute.targetPools.get`
- `compute.targetPools.list`
- `compute.targetPools.removeInstance`
- `compute.targetPools.use`
- `compute.targetTcpProxies.create`
- `compute.targetTcpProxies.get`
- `compute.targetTcpProxies.use`

The following permissions are required for creating DNS resources:

- `dns.changes.create`
- `dns.changes.get`
- `dns.managedZones.create`
- `dns.managedZones.get`
- `dns.managedZones.list`
- `dns.networks.bindPrivateDNSZone`
- `dns.resourceRecordSets.create`
- `dns.resourceRecordSets.list`
- `dns.resourceRecordSets.update`

The following permissions are required for creating Service Account resources:

- `iam.serviceAccountKeys.create`
- `iam.serviceAccountKeys.delete`
- `iam.serviceAccountKeys.get`
- `iam.serviceAccountKeys.list`
- `iam.serviceAccounts.actAs`
- `iam.serviceAccounts.create`
- `iam.serviceAccounts.delete`
- `iam.serviceAccounts.get`
- `iam.serviceAccounts.list`
- `resourcemanager.projects.get`
- `resourcemanager.projects.getIamPolicy`
- `resourcemanager.projects.setIamPolicy`

The following permissions are required for creating compute resources:

- `compute.disks.create`
- `compute.disks.get`
- `compute.disks.list`
- `compute.instanceGroups.create`
- `compute.instanceGroups.delete`
- `compute.instanceGroups.get`
- `compute.instanceGroups.list`
- `compute.instanceGroups.update`
- `compute.instanceGroups.use`
- `compute.instances.create`
- `compute.instances.delete`
- `compute.instances.get`
- `compute.instances.list`
- `compute.instances.setLabels`
- `compute.instances.setMetadata`
- `compute.instances.setServiceAccount`
- `compute.instances.setTags`
- `compute.instances.use`
- `compute.machineTypes.get`
- `compute.machineTypes.list`

The following permissions are required for creating storage resources:

- `storage.buckets.create`
- `storage.buckets.delete`
- `storage.buckets.get`
- `storage.buckets.list`
- `storage.objects.create`
- `storage.objects.delete`
- `storage.objects.get`
- `storage.objects.list`

The following permissions are required for creating health check resources:

- `compute.healthChecks.create`
- `compute.healthChecks.get`
- `compute.healthChecks.list`
- `compute.healthChecks.useReadOnly`
- `compute.httpHealthChecks.create`
- `compute.httpHealthChecks.get`
- `compute.httpHealthChecks.list`
- `compute.httpHealthChecks.useReadOnly`
- `compute.regionHealthChecks.create`
- `compute.regionHealthChecks.get`
- `compute.regionHealthChecks.useReadOnly`

The following permissions are required to get Google Cloud zone and region related information:

- `compute.globalOperations.get`
- `compute.regionOperations.get`
- `compute.regions.get`
- `compute.regions.list`
- `compute.zoneOperations.get`
- `compute.zones.get`
- `compute.zones.list`

The following permissions are required for checking services and quotas:

- `monitoring.timeSeries.list`
- `serviceusage.quotas.get`
- `serviceusage.services.list`

The following IAM permission is required for installation:

- `iam.roles.get`

The following permission is required when authenticating without a service account key:

- `iam.serviceAccounts.signBlob`

The following permission is required when providing Key Management Service (KMS) key rings:

- `cloudkms.keyRings.list`

The following Images permissions are required for installation:

- `compute.images.create`
- `compute.images.delete`
- `compute.images.get`
- `compute.images.list`

The following permission is optional for running gather bootstrap:

- `compute.instances.getSerialPortOutput`

The following permissions are required for deleting network resources:

- `compute.addresses.delete`
- `compute.addresses.deleteInternal`
- `compute.addresses.list`
- `compute.addresses.setLabels`
- `compute.firewalls.delete`
- `compute.firewalls.list`
- `compute.forwardingRules.delete`
- `compute.forwardingRules.list`
- `compute.globalAddresses.delete`
- `compute.globalAddresses.list`
- `compute.globalForwardingRules.delete`
- `compute.globalForwardingRules.list`
- `compute.networks.delete`
- `compute.networks.list`
- `compute.networks.updatePolicy`
- `compute.routers.delete`
- `compute.routers.list`
- `compute.routes.list`
- `compute.subnetworks.delete`
- `compute.subnetworks.list`

The following permissions are required for deleting load balancer resources:

- `compute.backendServices.delete`
- `compute.backendServices.list`
- `compute.regionBackendServices.delete`
- `compute.regionBackendServices.list`
- `compute.targetPools.delete`
- `compute.targetPools.list`
- `compute.targetTcpProxies.delete`
- `compute.targetTcpProxies.list`

The following permissions are required for deleting DNS resources:

- `dns.changes.create`
- `dns.managedZones.delete`
- `dns.managedZones.get`
- `dns.managedZones.list`
- `dns.resourceRecordSets.delete`
- `dns.resourceRecordSets.list`

The following permissions are required for deleting Service Account resources:

- `iam.serviceAccounts.delete`
- `iam.serviceAccounts.get`
- `iam.serviceAccounts.list`
- `resourcemanager.projects.getIamPolicy`
- `resourcemanager.projects.setIamPolicy`

The following permissions are required for deleting compute resources:

- `compute.disks.delete`
- `compute.disks.list`
- `compute.instanceGroups.delete`
- `compute.instanceGroups.list`
- `compute.instances.delete`
- `compute.instances.list`
- `compute.instances.stop`
- `compute.machineTypes.list`

The following permissions are required for deleting storage resources:

- `storage.buckets.delete`
- `storage.buckets.getIamPolicy`
- `storage.buckets.list`
- `storage.objects.delete`
- `storage.objects.list`

The following permissions are required for deleting health check resources:

- `compute.healthChecks.delete`
- `compute.healthChecks.list`
- `compute.httpHealthChecks.delete`
- `compute.httpHealthChecks.list`
- `compute.regionHealthChecks.delete`
- `compute.regionHealthChecks.list`

The following Images permissions are required for deletion:

- `compute.images.delete`
- `compute.images.list`

The following permission is required to get Region related information:

- `compute.regions.get`

The following Deployment Manager permissions are required:

- config.deployments.create
- config.deployments.delete
- config.deployments.get
- config.deployments.list
- config.operations.get
- config.resources.list
- cloudbuild.builds.create
- cloudbuild.builds.get

**Additional resources**

- [Optimizing storage](/docs/scalability_and_performance/optimization/optimizing-storage#optimizing-storage)

### Supported Google Cloud regions {#installation-gcp-regions_installing-gcp-user-infra}

You can deploy an OpenShift Container Platform cluster to specific Google Cloud regions, which determine the physical location and available machine types for your cluster infrastructure.

You can deploy to the following Google Cloud regions:

- `africa-south1` (Johannesburg, South Africa)
- `asia-east1` (Changhua County, Taiwan)
- `asia-east2` (Hong Kong)
- `asia-northeast1` (Tokyo, Japan)
- `asia-northeast2` (Osaka, Japan)
- `asia-northeast3` (Seoul, South Korea)
- `asia-south1` (Mumbai, India)
- `asia-south2` (Delhi, India)
- `asia-southeast1` (Jurong West, Singapore)
- `asia-southeast2` (Jakarta, Indonesia)
- `australia-southeast1` (Sydney, Australia)
- `australia-southeast2` (Melbourne, Australia)
- `europe-central2` (Warsaw, Poland)
- `europe-north1` (Hamina, Finland)
- `europe-southwest1` (Madrid, Spain)
- `europe-west1` (St. Ghislain, Belgium)
- `europe-west2` (London, England, UK)
- `europe-west3` (Frankfurt, Germany)
- `europe-west4` (Eemshaven, Netherlands)
- `europe-west6` (Zürich, Switzerland)
- `europe-west8` (Milan, Italy)
- `europe-west9` (Paris, France)
- `europe-west12` (Turin, Italy)
- `me-central1` (Doha, Qatar, Middle East)
- `me-central2` (Dammam, Saudi Arabia, Middle East)
- `me-west1` (Tel Aviv, Israel)
- `northamerica-northeast1` (Montréal, Québec, Canada)
- `northamerica-northeast2` (Toronto, Ontario, Canada)
- `southamerica-east1` (São Paulo, Brazil)
- `southamerica-west1` (Santiago, Chile)
- `us-central1` (Council Bluffs, Iowa, USA)
- `us-east1` (Moncks Corner, South Carolina, USA)
- `us-east4` (Ashburn, Northern Virginia, USA)
- `us-east5` (Columbus, Ohio)
- `us-south1` (Dallas, Texas)
- `us-west1` (The Dalles, Oregon, USA)
- `us-west2` (Los Angeles, California, USA)
- `us-west3` (Salt Lake City, Utah, USA)
- `us-west4` (Las Vegas, Nevada, USA)

:::note

To determine which machine type instances are available by region and zone, see the Google [documentation](https://cloud.google.com/compute/docs/regions-zones#available).

:::

### Installing and configuring CLI tools for Google Cloud {#installation-gcp-install-cli_installing-gcp-user-infra}

Before you deploy OpenShift Container Platform on Google Cloud with user-provisioned infrastructure, you must set up the required CLI tools to create and manage your cloud resources.

**Prerequisites**

- You created a project to host your cluster.
- You created a service account and granted it the required permissions.

**Procedure**

1. Install the following binaries in `$PATH`:
   - `gcloud`
   - `gsutil`

   See [Install the latest Cloud SDK version](https://cloud.google.com/sdk/docs/#install_the_latest_cloud_tools_version_cloudsdk_current_version) in the Google Cloud documentation.
2. Authenticate using the `gcloud` tool with your configured service account.
   See [Authorizing with a service account](https://cloud.google.com/sdk/docs/authorizing#authorizing_with_a_service_account) in the Google Cloud documentation.

## Requirements for a cluster with user-provisioned infrastructure {#installation-requirements-user-infra_installing-gcp-user-infra}

For a cluster that contains user-provisioned infrastructure, you must deploy all of the required machines. Reviewing these requirements before deployment helps you provision machines that meet the minimum resource needs of the cluster.

### Required machines for cluster installation {#installation-machine-requirements_installing-gcp-user-infra}

You must specify the minimum required machines or hosts for your cluster so that your cluster remains stable if a node fails.

The smallest OpenShift Container Platform clusters require the following hosts:

:::warning

For a cluster that has user-provisioned infrastructure, you must deploy all of the required machines.

:::

**Minimum required hosts**

| Hosts | Description |
| --- | --- |
| One temporary bootstrap machine | The cluster requires the bootstrap machine to deploy the OpenShift Container Platform cluster on the three control plane machines. You can remove the bootstrap machine after you install the cluster. |
| Three control plane machines | The control plane machines run the Kubernetes and OpenShift Container Platform services that form the control plane. |
| At least two compute machines, which are also known as worker machines. | The workloads requested by OpenShift Container Platform users run on the compute machines. |

:::warning

To maintain high availability of your cluster, use separate physical hosts for these cluster machines.

:::

The bootstrap and control plane machines must use Red Hat Enterprise Linux CoreOS (RHCOS) as the operating system. However, the compute machines can use Red Hat Enterprise Linux CoreOS (RHCOS), Red Hat Enterprise Linux (RHEL) 8.6 and later.

RHCOS is based on Red Hat Enterprise Linux (RHEL) 9.8 and inherits all of its hardware certifications and requirements. See [Red Hat Enterprise Linux technology capabilities and limits](https://access.redhat.com/articles/rhel-limits).

### Minimum resource requirements for cluster installation {#installation-minimum-resource-requirements_installing-gcp-user-infra}

To ensure that your OpenShift Container Platform cluster runs as expected, each cluster machine must meet minimum CPU, memory, and storage requirements.

**Minimum resource requirements**

<table>
<thead>
<tr>
  <th>Machine</th>
  <th>Operating system</th>
  <th>vCPU</th>
  <th>Virtual RAM</th>
  <th>Storage</th>
  <th>Input/Output Per Second (IOPS)</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Bootstrap</td>
  <td>RHCOS</td>
  <td>4</td>
  <td>16 GB</td>
  <td>100 GB</td>
  <td>300</td>
</tr>
<tr>
  <td>Control plane</td>
  <td>RHCOS</td>
  <td>4</td>
  <td>16 GB</td>
  <td>100 GB</td>
  <td>300</td>
</tr>
<tr>
  <td>Compute</td>
  <td>RHCOS</td>
  <td>2</td>
  <td>8 GB</td>
  <td>100 GB</td>
  <td>300</td>
</tr>
</tbody>
</table>

- One vCPU is equal to one physical core when simultaneous multithreading (SMT), or Hyper-Threading, is not enabled. When enabled, use the following formula to calculate the corresponding ratio: (threads per core × cores) × sockets = vCPUs.
- OpenShift Container Platform and Kubernetes are sensitive to disk performance, and Red Hat recommends faster storage, particularly for etcd on the control plane nodes which require a 10 ms p99 fsync duration. On many cloud platforms, storage size and IOPS scale together, so you might need to provision more storage to get enough performance.
- As with all user-provisioned installations, if you choose to use RHEL compute machines in your cluster, you take responsibility for all operating system life cycle management and maintenance, including performing system updates, applying patches, and completing all other required tasks. OpenShift Container Platform 4.10 and later do not support RHEL 7 compute machines.

:::note

In OpenShift Container Platform version 4.22, RHCOS uses RHEL version 9.8, which updates the micro-architecture requirements. Each architecture requires the following minimum instruction set architectures (ISA):

- x86-64 architecture requires x86-64-v2 ISA
- ARM64 architecture requires ARMv8.0-A ISA
- ppc64le architecture requires IBM(R) Power9 ISA
- s390x architecture requires IBM(R) z14 ISA

For more information, see [Architectures](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html-single/9.8_release_notes/index#architectures) in the RHEL documentation.

:::

If an instance type for your platform meets the minimum requirements for cluster machines, it is supported to use in OpenShift Container Platform.

**Additional resources**

- [Optimizing storage](/docs/scalability_and_performance/optimization/optimizing-storage#optimizing-storage)

### Tested instance types for Google Cloud {#installation-gcp-tested-machine-types_installing-gcp-user-infra}

OpenShift Container Platform supports specific Google Cloud instance types that have been validated for cluster deployment.

:::note

Not all instance types are available in all regions and zones. For a detailed breakdown of which instance types are available in which zones, see [regions and zones](https://cloud.google.com/compute/docs/regions-zones#available) (Google documentation).

Some instance types require the use of Hyperdisk storage. If you use an instance type that requires Hyperdisk storage, all of the nodes in your cluster must support Hyperdisk storage, and you must change the default storage class to use Hyperdisk storage. For more information, see [machine series support for Hyperdisk](https://cloud.google.com/compute/docs/disks/hyperdisks#machine-type-support) (Google documentation). For instructions on modifying storage classes, see the "GCE PersistentDisk (gcePD) object definition" section in the Dynamic Provisioning page in *Storage*.

:::

See the following machine series:

- `A2`
- `A3`
- `C2`
- `C2D`
- `C3`
- `C3D`
- `C4`
- `E2`
- `M1`
- `N1`
- `N2`
- `N2D`
- `N4`
- `Tau T2D`

### Tested instance types for Google Cloud on 64-bit ARM infrastructures {#installation-gcp-tested-machine-types-arm_installing-gcp-user-infra}

OpenShift Container Platform supports specific Google Cloud 64-bit ARM instance types that have been validated for cluster deployment.

See the following machine series for 64-bit ARM machines:

- `C4A`
- `N4A`
- `Tau T2A`

### Using custom machine types {#installation-custom-machine-types_installing-gcp-user-infra}

If the predefined Google Cloud machine types do not meet your workload requirements, you can configure a custom machine type in the `install-config.yaml` file during OpenShift Container Platform installation.

Consider the following when using a custom machine type:

- Similar to predefined instance types, custom machine types must meet the minimum resource requirements for control plane and compute machines. For more information, see "Minimum resource requirements for cluster installation".
- The name of the custom machine type must adhere to the following syntax: `custom-<number_of_cpus>-<amount_of_memory_in_mb>`
  For example, `custom-6-20480`.

## Creating the installation files for Google Cloud {#installation-user-infra-generate_installing-gcp-user-infra}

To install OpenShift Container Platform on Google Cloud by using user-provisioned infrastructure, you must generate the files that the installation program needs to deploy your cluster and modify them so that the cluster creates only the machines that it will use.

You generate and customize the `install-config.yaml` file, Kubernetes manifests, and Ignition config files. You also have the option to first set up a separate `var` partition during the preparation phases of installation.

### Creating a separate `/var` partition {#installation-disk-partitioning-upi-templates_installing-gcp-user-infra}

To isolate growing storage for containers, etcd, or logs, you can optionally create a separate `/var` partition on worker nodes before you generate Ignition configs.

It is recommended that disk partitioning for OpenShift Container Platform be left to the installation program. However, there are cases where you might want to create separate partitions in a part of the filesystem that you expect to grow.

OpenShift Container Platform supports the addition of a single partition to attach storage to either the `/var` partition or a subdirectory of `/var`. For example:

- `/var/lib/containers`: Holds container-related content that can grow as more images and containers are added to a system.
- `/var/lib/etcd`: Holds data that you might want to keep separate for purposes such as performance optimization of etcd storage.
- `/var`: Holds data that you might want to keep separate for purposes such as auditing.

Storing the contents of a `/var` directory separately makes it easier to grow storage for those areas as needed and reinstall OpenShift Container Platform at a later date and keep that data intact. With this method, you will not have to pull all your containers again, nor will you have to copy massive log files when you update systems.

Because `/var` must be in place before a fresh installation of Red Hat Enterprise Linux CoreOS (RHCOS), the following procedure sets up the separate `/var` partition by creating a machine config manifest that is inserted during the `openshift-install` preparation phases of an OpenShift Container Platform installation.

:::warning

If you follow the steps to create a separate `/var` partition in this procedure, it is not necessary to create the Kubernetes manifest and Ignition config files again as described later in this section.

:::

**Procedure**

1. Create a directory to hold the OpenShift Container Platform installation files:
   ```terminal
   $ mkdir $HOME/clusterconfig
   ```
2. Run `openshift-install` to create a set of files in the `manifest` and `openshift` subdirectories. Answer the system questions as you are prompted:
   ```terminal
   $ openshift-install create manifests --dir $HOME/clusterconfig
   ```

   ```terminal title="Example output"
   ? SSH Public Key ...
   INFO Credentials loaded from the "myprofile" profile in file "/home/myuser/.aws/credentials"
   INFO Consuming Install Config from target directory
   INFO Manifests created in: $HOME/clusterconfig/manifests and $HOME/clusterconfig/openshift
   ```
3. Optional: Confirm that the installation program created manifests in the `clusterconfig/openshift` directory:
   ```terminal
   $ ls $HOME/clusterconfig/openshift/
   ```

   ```terminal title="Example output"
   99_kubeadmin-password-secret.yaml
   99_openshift-cluster-api_master-machines-0.yaml
   99_openshift-cluster-api_master-machines-1.yaml
   99_openshift-cluster-api_master-machines-2.yaml
   ...
   ```
4. Create a Butane config that configures the additional partition. For example, name the file `$HOME/clusterconfig/98-var-partition.bu`, change the disk device name to the name of the storage device on the `worker` systems, and set the storage size as appropriate. This example places the `/var` directory on a separate partition:
   ```yaml
   variant: openshift
   version: 4.22.0
   metadata:
     labels:
       machineconfiguration.openshift.io/role: worker
     name: 98-var-partition
   storage:
     disks:
     - device: /dev/disk/by-id/<device_name>
       partitions:
       - label: var
         start_mib: <partition_start_offset>
         size_mib: <partition_size>
         number: 5
     filesystems:
       - device: /dev/disk/by-partlabel/var
         path: /var
         format: xfs
         mount_options: [defaults, prjquota]
         with_mount_unit: true
   ```

   where:

   <dl>
   <dt><code>&lt;device_name&gt;</code></dt>
   <dd>Specifies the storage device name of the disk that you want to partition.</dd>
   <dt><code>&lt;partition_start_offset&gt;</code></dt>
   <dd>Specifies the <code>start_mib</code> parameter. When adding a data partition to the boot disk, a minimum value of 25000 MiB (Mebibytes) is recommended. The root file system is automatically resized to fill all available space up to the specified offset. If no value is specified, or if the specified value is smaller than the recommended minimum, the resulting root file system will be too small, and future reinstalls of RHCOS might overwrite the beginning of the data partition.</dd>
   <dt><code>&lt;partition_size&gt;</code></dt>
   <dd>Specifies the size of the data partition in mebibytes.</dd>
   <dt><code>storage.filesystems.mount_options</code></dt>
   <dd>The <code>prjquota</code> mount option must be enabled for filesystems used for container storage.</dd>
   </dl>

   :::note

   When creating a separate `/var` partition, you cannot use different instance types for worker nodes, if the different instance types do not have the same device name.

   :::
5. Create a manifest from the Butane config and save it to the `clusterconfig/openshift` directory. For example, run the following command:
   ```terminal
   $ butane $HOME/clusterconfig/98-var-partition.bu -o $HOME/clusterconfig/openshift/98-var-partition.yaml
   ```
6. Run `openshift-install` again to create Ignition configs from a set of files in the `manifest` and `openshift` subdirectories:
   ```terminal
   $ openshift-install create ignition-configs --dir $HOME/clusterconfig
   ```

   ```terminal
   $ ls $HOME/clusterconfig/
   auth  bootstrap.ign  master.ign  metadata.json  worker.ign
   ```

   You can now use the Ignition config files as input to the installation procedures to install Red Hat Enterprise Linux CoreOS (RHCOS) systems.

### Creating the installation configuration file {#installation-initializing_installing-gcp-user-infra}

You can customize the OpenShift Container Platform cluster you install on Google Cloud.

**Prerequisites**

- You have the OpenShift Container Platform installation program and the pull secret for your cluster.
- Configure a Google Cloud account.

**Procedure**

1. Create the `install-config.yaml` file.
   1. Change to the directory that contains the installation program and run the following command:
      ```terminal
      $ ./openshift-install create install-config --dir <installation_directory>
      ```

      - `<installation_directory>`: For `<installation_directory>`, specify the directory name to store the files that the installation program creates.
        When specifying the directory:
      - Verify that the directory has the `execute` permission. This permission is required to run Terraform binaries under the installation directory.
      - Use an empty directory. Some installation assets, such as bootstrap X.509 certificates, have short expiration intervals, therefore you must not reuse an installation directory. If you want to reuse individual files from another cluster installation, you can copy them into your directory. However, the file names for the installation assets might change between releases. Use caution when copying installation files from an earlier OpenShift Container Platform version.
   2. At the prompts, provide the configuration details for your cloud:
      1. Optional: Select an SSH key to use to access your cluster machines.
         :::note

         For production OpenShift Container Platform clusters on which you want to perform installation debugging or disaster recovery, specify an SSH key that your `ssh-agent` process uses.

         :::
      2. Select **gcp** as the platform to target.
      3. If you have not configured the service account key for your Google Cloud account on your computer, you must obtain it from Google Cloud and paste the contents of the file or enter the absolute path to the file.
      4. Select the project ID to provision the cluster in. The default value is specified by the service account that you configured.
      5. Select the region to deploy the cluster to.
      6. Select the base domain to deploy the cluster to. The base domain corresponds to the public DNS zone that you created for your cluster.
      7. Enter a descriptive name for your cluster.
2. Modify the `install-config.yaml` file. You can find more information about the available parameters in the "Installation configuration parameters" section.
   :::note

   If you are installing a three-node cluster, be sure to set the `compute.replicas` parameter to `0`. This ensures that the cluster’s control planes are schedulable. For more information, see "Installing a three-node cluster on Google Cloud".

   :::
3. Back up the `install-config.yaml` file so that you can use it to install multiple clusters.
   :::warning

   The `install-config.yaml` file is consumed during the installation process. If you want to reuse the file, you must back it up now.

   :::

**Additional resources**

- [Installation configuration parameters for Google Cloud](/docs/installing/installing_gcp/installation-config-parameters-gcp#installation-config-parameters-gcp)

### Enabling Shielded VMs {#installation-gcp-enabling-shielded-vms_installing-gcp-user-infra}

You can use Shielded VMs when installing your OpenShift Container Platform cluster. Shielded VMs have extra security features including secure boot, firmware and integrity monitoring, and rootkit detection.

For more information, see Google’s documentation on [Shielded VMs](https://cloud.google.com/shielded-vm).

:::note

Shielded VMs are currently not supported on clusters with 64-bit ARM infrastructures.

:::

**Procedure**

- Use a text editor to edit the `install-config.yaml` file before deploying your cluster and add one of the following stanzas:
  1. To use shielded VMs for only control plane machines:
     ```yaml
     controlPlane:
       platform:
         gcp:
            secureBoot: Enabled
     ```
  2. To use shielded VMs for only compute machines:
     ```yaml
     compute:
     - platform:
         gcp:
            secureBoot: Enabled
     ```
  3. To use shielded VMs for all machines:
     ```yaml
     platform:
       gcp:
         defaultMachinePlatform:
            secureBoot: Enabled
     ```

### Enabling Confidential VMs {#installation-gcp-enabling-confidential-vms_installing-gcp-user-infra}

You can use Confidential VMs when installing your OpenShift Container Platform cluster. Confidential VMs encrypt data during processing.

For more information, see Google’s documentation on [Confidential Computing](https://cloud.google.com/confidential-computing). You can enable Confidential VMs and Shielded VMs at the same time, although they are not dependent on each other.

:::note

Confidential VMs are currently not supported on 64-bit ARM architectures.

:::

**Procedure**

- Use a text editor to edit the `install-config.yaml` file before deploying your cluster and add one of the following stanzas:
  1. To use confidential VMs for only control plane machines:
     ```yaml
     controlPlane:
       platform:
         gcp:
            confidentialCompute: AMDEncryptedVirtualizationNestedPaging
            type: n2d-standard-8
            onHostMaintenance: Terminate
     ```

     where:

     <dl>
     <dt><code>confidentialCompute</code></dt>
     <dd>Enables confidential VMs with AMD Secure Encrypted Virtualization Secure Nested Paging (AMD SEV-SNP). For more information about available options, see "Additional Google Cloud configuration parameters".</dd>
     <dt><code>type</code></dt>
     <dd>Specifies a machine type that supports Confidential VMs. Confidential VMs require the N2D, C2D, C3D, or C3 series of machine types. For more information on supported machine types, see <a href="https://cloud.google.com/compute/confidential-vm/docs/os-and-machine-type#machine-type">Supported operating systems and machine types</a>.</dd>
     <dt><code>onHostMaintenance</code></dt>
     <dd>Specifies the behavior of the VM during a host maintenance event, such as a hardware or software update. For a machine that uses Confidential VM, this value must be set to <code>Terminate</code>, which stops the VM. Confidential VMs do not support live VM migration.</dd>
     </dl>
  2. To use confidential VMs for only compute machines:
     ```yaml
     compute:
     - platform:
         gcp:
            confidentialCompute: AMDEncryptedVirtualizationNestedPaging
            type: n2d-standard-8
            onHostMaintenance: Terminate
     ```
  3. To use confidential VMs for all machines:
     ```yaml
     platform:
       gcp:
         defaultMachinePlatform:
            confidentialCompute: AMDEncryptedVirtualizationNestedPaging
            type: n2d-standard-8
            onHostMaintenance: Terminate
     ```

**Additional resources**

- [Additional Google Cloud configuration parameters](/docs/installing/installing_gcp/installation-config-parameters-gcp#installation-configuration-parameters-additional-gcp_installation-config-parameters-gcp)

### Configuring the cluster-wide proxy during installation {#installation-configure-proxy_installing-gcp-user-infra}

Production environments can deny direct access to the internet and instead have an HTTP or HTTPS proxy available. You can configure a new OpenShift Container Platform cluster to use a proxy by configuring the proxy settings in the `install-config.yaml` file.

**Prerequisites**

- You have reviewed the sites that your cluster requires access to and determined whether any of them need to bypass the proxy. By default, the proxy handles all cluster egress traffic, including calls to hosting cloud provider APIs. You added sites to the `Proxy` object’s `spec.noProxy` field to bypass the proxy if necessary.
  :::note

  The `Proxy` object `status.noProxy` field includes the values of the `networking.machineNetwork[].cidr`, `networking.clusterNetwork[].cidr`, and `networking.serviceNetwork[]` fields from your installation configuration.

  For installations on Amazon Web Services (AWS), Google Cloud, Microsoft Azure, and Red Hat OpenStack Platform (RHOSP), the `Proxy` object `status.noProxy` field also includes the instance metadata endpoint (`169.254.169.254`).

  :::

**Procedure**

1. Edit your `install-config.yaml` file and add the proxy settings. For example:
   ```yaml
   apiVersion: v1
   baseDomain: my.domain.com
   proxy:
     httpProxy: http://<username>:<pswd>@<ip>:<port>
     httpsProxy: https://<username>:<pswd>@<ip>:<port>
     noProxy: example.com
   additionalTrustBundle: |
       -----BEGIN CERTIFICATE-----
       <MY_TRUSTED_CA_CERT>
       -----END CERTIFICATE-----
   additionalTrustBundlePolicy: <policy_to_add_additionalTrustBundle>
   # ...
   ```

   where:

   <dl>
   <dt><code>proxy.httpProxy</code></dt>
   <dd>Specifies a proxy URL to use for creating HTTP connections outside the cluster. The URL scheme must be <code>http</code>.</dd>
   <dt><code>proxy.httpsProxy</code></dt>
   <dd>Specifies a proxy URL to use for creating HTTPS connections outside the cluster.</dd>
   <dt><code>proxy.noProxy</code></dt>
   <dd>Specifies a comma-separated list of destination domain names, IP addresses, or other network CIDRs to exclude from proxying. Preface a domain with <code>.</code> to match subdomains only. For example, <code>.y.com</code> matches <code>x.y.com</code>, but not <code>y.com</code>. Use <code>*</code> to bypass the proxy for all destinations.</dd>
   <dt><code>additionalTrustBundle</code></dt>
   <dd>If you specify this value, the installation program generates a config map named <code>user-ca-bundle</code> in the <code>openshift-config</code> namespace to hold the additional CA certificates. If you specify <code>additionalTrustBundle</code> and at least one proxy setting, the <code>Proxy</code> object references the <code>user-ca-bundle</code> config map in the <code>trustedCA</code> field. The Cluster Network Operator then creates a <code>trusted-ca-bundle</code> config map that merges the contents specified for the <code>trustedCA</code> parameter with the RHCOS trust bundle. You must set the <code>additionalTrustBundle</code> field unless an authority from the RHCOS trust bundle signs the proxy’s identity certificate.</dd>
   <dt><code>additionalTrustBundlePolicy</code></dt>
   <dd>Specifies the policy that determines the configuration of the <code>Proxy</code> object to reference the <code>user-ca-bundle</code> config map in the <code>trustedCA</code> field. The allowed values are <code>Proxyonly</code> and <code>Always</code>. Use <code>Proxyonly</code> to reference the <code>user-ca-bundle</code> config map only when you configure an <code>http/https</code> proxy. Use <code>Always</code> to always reference the <code>user-ca-bundle</code> config map. The default value is <code>Proxyonly</code>. Optional parameter.</dd>
   </dl>

   :::note

   The installation program does not support the proxy `readinessEndpoints` field.

   :::

   :::note

   If the installation program times out, restart and then complete the deployment by using the `wait-for` command of the installation program. For example:

   ```terminal
   $ ./openshift-install wait-for install-complete --log-level debug
   ```

   :::
2. Save the file and reference it when installing OpenShift Container Platform.
   The installation program creates a cluster-wide proxy named `cluster` that uses the proxy settings in the `install-config.yaml` file. If you do not give proxy settings, the installation program still creates a `cluster` `Proxy` object, but it has a nil `spec`.

   :::note

   Only the `Proxy` object named `cluster` is supported, and you cannot create additional proxies.

   :::

### Creating the Kubernetes manifest and Ignition config files {#installation-user-infra-generate-k8s-manifest-ignition_installing-gcp-user-infra}

Because you manually provision infrastructure, you must generate the Kubernetes manifest and Ignition config files that the cluster requires.

The installation program converts the installation configuration into Kubernetes manifests and then wraps them into Ignition configuration files. You use these Ignition files to configure the cluster machines.

:::warning

- The Ignition config files that the OpenShift Container Platform installation program generates contain certificates that expire after 24 hours, which the system then renews. If you shut down the cluster before the system renews the certificates and you later restart the cluster after the 24 hours have elapsed, the cluster automatically recovers the expired certificates. The exception is that you must manually approve the pending `node-bootstrapper` certificate signing requests (CSRs) to recover kubelet certificates. See the documentation for *Recovering from expired control plane certificates* for more information.
- Use Ignition config files within 12 hours after you generate them, because the 24-hour certificate rotates from 16 to 22 hours after you install the cluster. By using the Ignition config files within 12 hours, you can avoid installation failure if the certificate update runs during installation.

:::

**Procedure**

1. Change to the directory that contains the OpenShift Container Platform installation program and generate the Kubernetes manifests for the cluster:
   ```terminal
   $ ./openshift-install create manifests --dir <installation_directory>
   ```

   where:

   <dl>
   <dt><code>&lt;installation_directory&gt;</code></dt>
   <dd>Specifies the installation directory that contains the <code>install-config.yaml</code> file you created.</dd>
   </dl>
2. Remove the Kubernetes manifest files that define the control plane machines:
   ```terminal
   $ rm -f <installation_directory>/openshift/99_openshift-cluster-api_master-machines-*.yaml
   ```

   By removing these files, you prevent the cluster from automatically generating control plane machines.
3. Remove the Kubernetes manifest files that define the control plane machine set:
   ```terminal
   $ rm -f <installation_directory>/openshift/99_openshift-machine-api_master-control-plane-machine-set.yaml
   ```
4. Optional: If you do not want the cluster to provision compute machines, remove the Kubernetes manifest files that define the worker machines:
   ```terminal
   $ rm -f <installation_directory>/openshift/99_openshift-cluster-api_worker-machineset-*.yaml
   ```

   :::warning

   If you disabled the `MachineAPI` capability when installing a cluster on user-provisioned infrastructure, you must remove the Kubernetes manifest files that define the worker machines. Otherwise, your cluster fails to install.

   :::

   Because you create and manage the worker machines yourself, you do not need to initialize these machines.

   :::warning

   If you are installing a three-node cluster, skip the following step to allow the control plane nodes to be schedulable.

   :::

   :::warning

   When you configure control plane nodes from the default unschedulable to schedulable, you require additional subscriptions because control plane nodes then become compute nodes.

   :::
5. Verify that the `mastersSchedulable` parameter in the `<installation_directory>/manifests/cluster-scheduler-02-config.yml` Kubernetes manifest file is set to `false`. This setting prevents pods from being scheduled on the control plane machines:
   1. Open the `<installation_directory>/manifests/cluster-scheduler-02-config.yml` file.
   2. Locate the `mastersSchedulable` parameter and verify that it is set to `false`.
   3. Save and exit the file.
6. Optional: If you do not want [the Ingress Operator](https://github.com/openshift/cluster-ingress-operator) to create DNS records on your behalf, remove the `privateZone` and `publicZone` sections from the `<installation_directory>/manifests/cluster-dns-02-config.yml` DNS configuration file:
   ```yaml
   apiVersion: config.openshift.io/v1
   kind: DNS
   metadata:
     creationTimestamp: null
     name: cluster
   spec:
     baseDomain: example.openshift.com
     privateZone:
       id: mycluster-100419-private-zone
     publicZone:
       id: example.openshift.com
   status: {}
   ```

   `spec.privateZone`: Remove this section completely.

   If you do so, you must add ingress DNS records manually in a later step.
7. To create the Ignition configuration files, run the following command from the directory that contains the installation program:
   ```terminal
   $ ./openshift-install create ignition-configs --dir <installation_directory>
   ```

   where:

   <dl>
   <dt><code>&lt;installation_directory&gt;</code></dt>
   <dd>Specifies the same installation directory. The installation program creates Ignition config files for the bootstrap, control plane, and compute nodes in the installation directory. The program also creates the <code>kubeadmin-password</code> and <code>kubeconfig</code> files in the <code>./&lt;installation_directory&gt;/auth</code> directory:</dd>
   </dl>

   ```
   .
   ├── auth
   │   ├── kubeadmin-password
   │   └── kubeconfig
   ├── bootstrap.ign
   ├── master.ign
   ├── metadata.json
   └── worker.ign
   ```

**Additional resources**

- [Adding the ingress DNS records](/docs/installing/installing_gcp/installing-gcp-user-infra#installation-gcp-user-infra-adding-ingress_installing-gcp-user-infra)

## Extracting the infrastructure name {#installation-extracting-infraid_installing-gcp-user-infra}

To identify your cluster resources in Google Cloud, extract the unique infrastructure name from the Ignition config files.

The Ignition config files contain a unique cluster identifier that you can use to uniquely identify your cluster in Google Cloud. The infrastructure name is also used to locate the appropriate Google Cloud resources during an OpenShift Container Platform installation. The provided Infrastructure Manager templates contain references to this infrastructure name, so you must extract it.

:::warning

Do not run the `openshift-install create manifests` command again after creating any Google Cloud resources. Running the command again generates a new cluster identifier, which will cause errors in existing resources. If you need to regenerate the manifests because you modified the `install-config.yaml` file, delete any Google Cloud resources you created and re-create them with the new cluster identifier.

:::

**Prerequisites**

- You installed the `jq` package.

**Procedure**

- To extract and view the infrastructure name from the Ignition config file metadata, run the following command:
  ```terminal
  $ jq -r .infraID <installation_directory>/metadata.json
  ```

  where `<installation_directory>` is the path to the directory that you stored the installation files in.

  ```terminal title="Example output"
  openshift-vw9j6
  ```

  The output of this command is your cluster name and a random string.

## Exporting common variables for Infrastructure Manager templates {#installation-user-infra-exporting-common-variables_installing-gcp-user-infra}

You must export a common set of variables that the Infrastructure Manager templates reference to provision the resources for a cluster that uses user-provisioned infrastructure on Google Cloud.

:::note

Specific Infrastructure Manager templates can also require additional exported variables, which are detailed in their related procedures.

:::

**Procedure**

- Export the following common variables to be used by the provided Infrastructure Manager templates. For any command with `<installation_directory>`, specify the path to the directory that you stored the installation files in.
  - Export the `BASE_DOMAIN` variable by running the following command:
    ```terminal
    $ export BASE_DOMAIN='<base_domain>'
    ```

    <dl>
    <dt><code>&lt;base_domain&gt;</code></dt>
    <dd>If you are installing a cluster into a shared VPC, specify the value for the host project.</dd>
    </dl>
  - Export the `BASE_DOMAIN_ZONE_NAME` variable by running the following command:
    ```terminal
    $ export BASE_DOMAIN_ZONE_NAME='<base_domain_zone_name>'
    ```

    <dl>
    <dt><code>&lt;base_domain_zone_name&gt;</code></dt>
    <dd>Specifies the base domain zone name.</dd>
    </dl>
  - Export the `NETWORK_CIDR` variable by running the following command:
    ```terminal
    $ export NETWORK_CIDR='<network_cidr>'
    ```

    <dl>
    <dt><code>&lt;network_cidr&gt;</code></dt>
    <dd>Specifies the network CIDR your cluster uses. For example, <code>10.0.0.0/16</code>.</dd>
    </dl>
  - Export the `MASTER_SUBNET_CIDR` variable by running the following command:
    ```terminal
    $ export MASTER_SUBNET_CIDR='<master_subnet_cidr>'
    ```

    <dl>
    <dt><code>&lt;master_subnet_cidr&gt;</code></dt>
    <dd>Specifies the network CIDR that your cluster’s control plane uses. For example, <code>10.0.0.0/17</code>.</dd>
    </dl>
  - Export the `WORKER_SUBNET_CIDR` variable by running the following command:
    ```terminal
    $ export WORKER_SUBNET_CIDR='<worker_subnet_cidr>'
    ```

    <dl>
    <dt><code>&lt;worker_subnet_cidr&gt;</code></dt>
    <dd>Specifies the network CIDR that your cluster’s compute machines use. For example, <code>10.0.128.0/17</code>.</dd>
    </dl>
  - Export the `KUBECONFIG` variable by running the following command:
    ```terminal
    $ export KUBECONFIG=<installation_directory>/auth/kubeconfig
    ```
  - Export the `CLUSTER_NAME` variable by running the following command:
    ```terminal
    $ export CLUSTER_NAME=`jq -r .clusterName <installation_directory>/metadata.json`
    ```
  - Export the `INFRA_ID` variable by running the following command:
    ```terminal
    $ export INFRA_ID=`jq -r .infraID <installation_directory>/metadata.json`
    ```
  - Export the `PROJECT_NAME` variable by running the following command:
    ```terminal
    $ export PROJECT_NAME=`jq -r .gcp.projectID <installation_directory>/metadata.json`
    ```
  - If you are installing a cluster into a shared VPC, export the `HOST_PROJECT` variable by running the following command:
    ```terminal
    $ export HOST_PROJECT=<host_project_name>
    ```

    `<host_project_name>` specifies the name of the host project that contains the shared VPC.
  - If you are installing a cluster into a shared VPC, export the `HOST_PROJECT_ACCOUNT` variable by running the following command:
    ```terminal
    $ export HOST_PROJECT_ACCOUNT=<host_project_account>
    ```

    `<host_project_account>` specifies the name of an account that can access the host project that contains the shared VPC.
  - Export the `REGION` variable by running the following command:
    ```terminal
    $ export REGION=`jq -r .gcp.region <installation_directory>/metadata.json`
    ```
  - Export the `ZONE_0` variable by running the following command:
    ```terminal
    $ export ZONE_0=$(gcloud compute regions describe ${REGION} --format=json | jq -r '.zones[0]' | cut -d "/" -f9)
    ```
  - Export the `ZONE_1` variable by running the following command:
    ```terminal
    $ export ZONE_1=$(gcloud compute regions describe ${REGION} --format=json | jq -r '.zones[1]' | cut -d "/" -f9)
    ```
  - Export the `ZONE_2` variable by running the following command:
    ```terminal
    $ export ZONE_2=$(gcloud compute regions describe ${REGION} --format=json | jq -r '.zones[2]' | cut -d "/" -f9)
    ```
  - Export the `SERVICE_ACCOUNT_EMAIL` variable by running the following command:
    ```terminal
    $ export SERVICE_ACCOUNT_EMAIL="<service_account_email>"
    ```

    <dl>
    <dt><code>&lt;service_account_email&gt;</code></dt>
    <dd>Specifies the email address of the service account you used for the installation.</dd>
    </dl>
  - Export the `INSTALL_SERVICE_ACCOUNT` variable by running the following command:
    ```terminal
    $ export INSTALL_SERVICE_ACCOUNT="projects/${PROJECT_NAME}/serviceAccounts/${SERVICE_ACCOUNT_EMAIL}"
    ```
  - Export the `CLUSTER_DOMAIN` variable by running the following command:
    ```terminal
    $ export CLUSTER_DOMAIN="${CLUSTER_NAME}.${BASE_DOMAIN}"
    ```

## Creating a VPC in Google Cloud {#installation-creating-gcp-vpc_installing-gcp-user-infra}

You must create a VPC in Google Cloud for your OpenShift Container Platform cluster to use. You can customize the VPC to meet your requirements. One way to create the VPC is to modify the provided Infrastructure Manager template.

:::note

If you do not use the provided Infrastructure Manager template to create your Google Cloud infrastructure, you must review the provided information and manually create the infrastructure. If your cluster does not initialize correctly, you might have to contact Red Hat support with your installation logs.

:::

**Prerequisites**

- You have defined the variables in the *Exporting common variables* section.

**Procedure**

1. Copy the template from the **Infrastructure Manager template for the VPC** section of this topic and save it as `01_vpc.tf` in a directory called `01_vpc` on your computer. This template describes the VPC that your cluster requires.
2. Create a VPC by running the following command:
   ```terminal
   $ gcloud infra-manager deployments apply <vpc_deployment_name> \
     --location=${REGION} \
     --input-values=infra_id=${INFRA_ID},project=${PROJECT_NAME},region=${REGION},master_subnet_cidr=${MASTER_SUBNET_CIDR},worker_subnet_cidr=${WORKER_SUBNET_CIDR} \
     --project=${PROJECT_NAME} \
     --local-source=./01_vpc \
     --service-account=${INSTALL_SERVICE_ACCOUNT}
   ```

   `<vpc_deployment_name>` specifies the name of the VPC deployment you create.
3. Configure environment variables that will be used to create other cluster infrastructure.
   1. Configure the `CLUSTER_NETWORK` environment variable by running the following command:
      ```terminal
      $ export CLUSTER_NETWORK=$(gcloud compute networks describe ${INFRA_ID}-network --format json | jq -r .selfLink)
      ```
   2. Configure the `CONTROL_SUBNET` environment variable by running the following command:
      ```terminal
      $ export CONTROL_SUBNET=$(gcloud compute networks subnets describe ${INFRA_ID}-master-subnet --region=${REGION} --format json | jq -r .selfLink)
      ```
   3. Configure the `COMPUTE_SUBNET` environment variable by running the following command:
      ```terminal
      $ export COMPUTE_SUBNET=$(gcloud compute networks subnets describe ${INFRA_ID}-worker-subnet --region=${REGION} --format json | jq -r .selfLink)
      ```

**Verification**

1. Verify the deployment is active by running the following command:
   ```terminal
   $ gcloud infra-manager deployments describe <deployment_name> --format='value(state)'
   ```

   Replace `<deployment_name>` with the name of the deployment you created.

   ```terminal title="Example output"
   ACTIVE
   ```

### Infrastructure Manager template for the VPC {#installation-infrastructure-manager-vpc_installing-gcp-user-infra}

You can use the following `01_vpc.tf` Infrastructure Manager template to deploy the VPC that you need for your OpenShift Container Platform cluster:

```terraform
terraform {
  # Infra manager supports specific Terraform versions; ensure compatibility
  required_version = ">=1.2.3"
  required_providers {
    google = {
      source = "hashicorp/google"
      version = ">= 4.0.0"
    }
    google-beta = {
        source = "hashicorp/google-beta",
        version = ">= 4.0.0"
    }
  }
}

provider "google-beta" {
  project = "${var.project}"
  region = "${var.region}"
}

variable "infra_id" {
  type        = string
  description = "OpenShift Installer Infrastructure ID"
}

variable "project" {
  type        = string
  description = "Project ID"
}

variable "region" {
  type        = string
  description = "GCP Region where the resources will be created."
  default     = "us-central1"
}

variable "master_subnet_cidr" {
  type        = string
  description = "CIDR for the control plane subnet."
}

variable "worker_subnet_cidr" {
  type        = string
  description = "CIDR for the compute subnet."
}

resource "google_compute_network" "cluster_network" {
  provider = google-beta

  name = "${var.infra_id}-network"
  auto_create_subnetworks = false
}

resource "google_compute_subnetwork" "master_subnet" {
  provider = google-beta

  name = "${var.infra_id}-master-subnet"
  ip_cidr_range = "${var.master_subnet_cidr}"
  region = "${var.region}"
  network = google_compute_network.cluster_network.self_link
}

resource "google_compute_subnetwork" "worker_subnet" {
  provider = google-beta

  name = "${var.infra_id}-worker-subnet"
  ip_cidr_range = "${var.worker_subnet_cidr}"
  region = "${var.region}"
  network = google_compute_network.cluster_network.self_link
}

#tfimport-terraform import google_compute_router._router  __project__//-router
resource "google_compute_router" "router" {
  provider = google-beta

  name = "${var.infra_id}-router"
  network = google_compute_network.cluster_network.self_link
  region = "${var.region}"
}
resource "google_compute_router_nat" "master_nat" {
  provider = google-beta

  name = "${var.infra_id}-nat-master"
  source_subnetwork_ip_ranges_to_nat = "LIST_OF_SUBNETWORKS"
  nat_ip_allocate_option = "AUTO_ONLY"
  min_ports_per_vm = 7168
  subnetwork {
    name = google_compute_subnetwork.master_subnet.self_link
    source_ip_ranges_to_nat = ["ALL_IP_RANGES"]
  }

  router = google_compute_router.router.name
  region = "${var.region}"

  depends_on = [
    google_compute_router.router
  ]
}
resource "google_compute_router_nat" "worker_nat" {
  provider = google-beta

  name = "${var.infra_id}-nat-worker"
  source_subnetwork_ip_ranges_to_nat = "LIST_OF_SUBNETWORKS"
  nat_ip_allocate_option = "AUTO_ONLY"
  min_ports_per_vm = 512
  subnetwork {
    name = google_compute_subnetwork.worker_subnet.self_link
    source_ip_ranges_to_nat = ["ALL_IP_RANGES"]
  }

  router = google_compute_router.router.name
  region = "${var.region}"

  depends_on = [
    google_compute_router.router
  ]
}
```

## Networking requirements for user-provisioned infrastructure {#installation-network-user-infra_installing-gcp-user-infra}

You must configure networking for all the Red Hat Enterprise Linux CoreOS (RHCOS) machines in `initramfs` during boot, so that they can fetch their Ignition config files.

### Setting the cluster node hostnames through DHCP {#installation-host-names-dhcp-user-infra_installing-gcp-user-infra}

On Red Hat Enterprise Linux CoreOS (RHCOS) machines, the hostname is set through NetworkManager. By default, the machines obtain their hostname through DHCP. If the hostname is not provided by DHCP, set statically through kernel arguments, or another method, it is obtained through a reverse DNS lookup. Reverse DNS lookup occurs after the network has been initialized on a node and can take time to resolve. Other system services can start before this and detect the hostname as `localhost` or similar. You can avoid this by using DHCP to provide the hostname for each cluster node.

Additionally, setting the hostnames through DHCP can bypass any manual DNS record name configuration errors in environments that have a DNS split-horizon implementation.

### Network connectivity requirements {#installation-network-connectivity-user-infra_installing-gcp-user-infra}

You must configure the network connectivity between machines to allow OpenShift Container Platform cluster components to communicate. Each machine must be able to resolve the hostnames of all other machines in the cluster.

This section provides details about the ports that are required.

:::warning

In connected OpenShift Container Platform environments, all nodes are required to have internet access to pull images for platform containers and provide telemetry data to Red Hat.

:::

**Ports used for all-machine to all-machine communications**

<table>
<thead>
<tr>
  <th>Protocol</th>
  <th>Port</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>ICMP</td>
  <td>N/A</td>
  <td>Network reachability tests</td>
</tr>
<tr>
  <td rowspan="4">TCP</td>
  <td><code>1936</code></td>
  <td>Metrics</td>
</tr>
<tr>
  <td><code>9000</code>-<code>9999</code></td>
  <td>Host level services, including the node exporter on ports <code>9100</code>-<code>9101</code> and the Cluster Version Operator on port <code>9099</code>.</td>
</tr>
<tr>
  <td><code>10250</code>-<code>10259</code></td>
  <td>The default ports that Kubernetes reserves</td>
</tr>
<tr>
  <td><code>22623</code></td>
  <td>The port handles traffic from the Machine Config Server and directs the traffic to the control plane machines.</td>
</tr>
<tr>
  <td rowspan="5">UDP</td>
  <td><code>6081</code></td>
  <td>Geneve</td>
</tr>
<tr>
  <td><code>9000</code>-<code>9999</code></td>
  <td>Host level services, including the node exporter on ports <code>9100</code>-<code>9101</code>.</td>
</tr>
<tr>
  <td><code>500</code></td>
  <td>IPsec IKE packets</td>
</tr>
<tr>
  <td><code>4500</code></td>
  <td>IPsec NAT-T packets</td>
</tr>
<tr>
  <td><code>123</code></td>
  <td>Network Time Protocol (NTP) on UDP port <code>123</code>. If an external NTP time server is configured, you must open UDP port <code>123</code>.</td>
</tr>
<tr>
  <td>TCP/UDP</td>
  <td><code>30000</code>-<code>32767</code></td>
  <td>Kubernetes node port</td>
</tr>
<tr>
  <td>ESP</td>
  <td>N/A</td>
  <td>IPsec Encapsulating Security Payload (ESP)</td>
</tr>
</tbody>
</table>

**Ports used for all-machine to control plane communications**

<table>
<thead>
<tr>
  <th>Protocol</th>
  <th>Port</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>TCP</td>
  <td><code>6443</code></td>
  <td>Kubernetes API</td>
</tr>
</tbody>
</table>

**Ports used for control plane machine to control plane machine communications**

<table>
<thead>
<tr>
  <th>Protocol</th>
  <th>Port</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>TCP</td>
  <td><code>2379</code>-<code>2380</code></td>
  <td>etcd server and peer ports</td>
</tr>
</tbody>
</table>

## Creating load balancers in Google Cloud {#installation-creating-gcp-lb_installing-gcp-user-infra}

You must configure load balancers in Google Cloud for your OpenShift Container Platform cluster to use. One way to create these components is to modify the provided Infrastructure Manager template.

:::note

If you do not use the provided template to create your Google Cloud infrastructure, you must review the provided information and manually create the infrastructure. If your cluster does not initialize correctly, you might have to contact Red Hat support with your installation logs.

:::

**Prerequisites**

- You have defined the variables in the *Exporting common variables* section.
- If you are not installing a cluster into a shared VPC, you have defined the variables in the *Creating a VPC in Google Cloud* section.

**Procedure**

1. If you are installing a cluster into a shared VPC, set environment variables for the cluster network and control plane subnet.
   1. Determine the shared VPC network name by running the following command:
      ```terminal
      $ gcloud compute networks list
      ```
   2. Set the `CLUSTER_NETWORK` variable by running the following command:
      ```terminal
      $ export CLUSTER_NETWORK=$(gcloud compute networks describe <network_name> --format json | jq -r .selfLink)
      ```

      `<network_name>` specifies the name of the network you determined.
   3. List the available network subnets by running the following command:
      ```terminal
      $ gcloud compute networks subnets list --network=<network_name>
      ```

      `<network_name>` specifies the name of the network you determined.
   4. Select a subnet from the list, and set the `CONTROL_SUBNET` variable by running the following command:
      ```terminal
      $ export CONTROL_SUBNET=<control_subnet>
      ```

      `<control_subnet>` specifies the name of the subnet you selected from the list of subnets.
2. Copy the template from the **Infrastructure Manager template for the internal load balancer** section of this topic and save it as `02_lb_int.tf` in a directory called `02_lb_int` on your computer. This template describes the internal load balancing objects that your cluster requires.
   1. Create an internal load balancer by running the following command:
      ```terminal
      $ gcloud infra-manager deployments apply <internal_lb_deployment_name> \
        --location=${REGION} \
        --input-values=infra_id=${INFRA_ID},project=${PROJECT_NAME},region=${REGION},cluster_network=${CLUSTER_NETWORK},control_subnet=${CONTROL_SUBNET},zone_0=${ZONE_0},zone_1=${ZONE_1},zone_2=${ZONE_2} \
        --project=${PROJECT_NAME} \
        --local-source=./02_lb_int \
        --service-account=${INSTALL_SERVICE_ACCOUNT}
      ```

      `<internal_lb_deployment_name>` specifies the name of the internal load balancer deployment you create.
   2. Export the `CLUSTER_IP` variable by running the following command:
      ```terminal
      $ export CLUSTER_IP=$(gcloud compute addresses describe ${INFRA_ID}-cluster-ip --region=${REGION} --format json | jq -r .address)
      ```
3. Optional: For a public or externally available cluster, copy the template from the **Infrastructure Manager template for the external load balancer** section of this topic and save it as `02_lb_ext.tf` in a directory called `02_lb_ext` on your computer. This template describes the external load balancing objects that your cluster requires.
   1. Create an external load balancer by running the following command:
      ```terminal
      $ gcloud infra-manager deployments apply <external_lb_deployment_name> \
        --location=${REGION} \
        --input-values=infra_id=${INFRA_ID},project=${PROJECT_NAME},region=${REGION} \
        --project=${PROJECT_NAME} \
        --local-source=./02_lb_ext \
        --service-account=${INSTALL_SERVICE_ACCOUNT}
      ```

      `<external_lb_deployment_name>` specifies the name of the external load balancer deployment you create.
   2. Export the `CLUSTER_PUBLIC_IP` variable by running the following command:
      ```terminal
      $ export CLUSTER_PUBLIC_IP=$(gcloud compute addresses describe ${INFRA_ID}-cluster-public-ip --region=${REGION} --format json | jq -r .address)
      ```

**Verification**

1. Verify the deployment is active by running the following command:
   ```terminal
   $ gcloud infra-manager deployments describe <deployment_name> --format='value(state)'
   ```

   Replace `<deployment_name>` with the name of the deployment you created.

   ```terminal title="Example output"
   ACTIVE
   ```

### Infrastructure Manager template for the external load balancer {#installation-infrastructure-manager-ext-lb_installing-gcp-user-infra}

You can use the following `02_lb_ext.tf` Infrastructure Manager template to deploy the external load balancer that you need for your OpenShift Container Platform cluster:

```terraform
terraform {
  # Infra manager supports specific Terraform versions; ensure compatibility
  required_version = ">=1.2.3"
  required_providers {
    google = {
      source = "hashicorp/google"
      version = ">= 4.0.0"
    }
    google-beta = {
        source = "hashicorp/google-beta",
        version = ">= 4.0.0"
    }
  }
}

provider "google-beta" {
  project = "${var.project}"
  region = "${var.region}"
}
variable "infra_id" {
  type        = string
  description = "OpenShift Installer Infrastructure ID"
}
variable "project" {
  type        = string
  description = "Project ID"
}
variable "region" {
  type        = string
  description = "GCP Region where the resources will be created."
  default     = "us-central1"
}

resource "google_compute_address" "cluster_public_ip" {
  provider = google-beta

  name = "${var.infra_id}-cluster-public-ip"
  region = "${var.region}"
}

resource "google_compute_http_health_check" "api_http_health_check" {
  provider = google-beta

  name = "${var.infra_id}-api-http-health-check"
  port = 6080
  request_path = "/readyz"
}

resource "google_compute_target_pool" "api_target_pool" {
  provider = google-beta

  name = "${var.infra_id}-api-target-pool"
  region = "${var.region}"
  health_checks = [
    google_compute_http_health_check.api_http_health_check.id
  ]
}

resource "google_compute_forwarding_rule" "api_forwarding_rule" {
  provider = google-beta

  name = "${var.infra_id}-api-forwarding-rule"
  ip_address = google_compute_address.cluster_public_ip.address
  port_range = "6443"
  region = "${var.region}"
  target = google_compute_target_pool.api_target_pool.id
}
```

### Infrastructure Manager template for the internal load balancer {#installation-infrastructure-manager-int-lb_installing-gcp-user-infra}

You can use the following `02_lb_int.tf` Infrastructure Manager template to deploy the internal load balancer that you need for your OpenShift Container Platform cluster:

```terraform
terraform {
  # Infra manager supports specific Terraform versions; ensure compatibility
  required_version = ">=1.2.3"
  required_providers {
    google = {
      source = "hashicorp/google"
      version = ">= 4.0.0"
    }
    google-beta = {
        source = "hashicorp/google-beta",
        version = ">= 4.0.0"
    }
  }
}

provider "google-beta" {
  project = "${var.project}"
  region = "${var.region}"
}
variable "infra_id" {
  type        = string
  description = "OpenShift Installer Infrastructure ID"
}
variable "project" {
  type        = string
  description = "Project ID"
}
variable "region" {
  type        = string
  description = "GCP Region where the resources will be created."
  default     = "us-central1"
}
variable "control_subnet" {
  type        = string
  description = "Subnet for the control plane instances."
}
variable "cluster_network" {
  type        = string
  description = "Full link to the cluster network."
}

# Terraform handles lists but the infra-manager --input-values only
# supports scalar types.
# If you require more or less zones, you must manually add them below
# as a single variable for each. You must add the zones to the
# locals `zones` list below.
variable "zone_0" {
  type        = string
  description = "Zone 1 for the instance types."
}

variable "zone_1" {
  type        = string
  description = "Zone 2 for the instance types."
}

variable "zone_2" {
  type        = string
  description = "Zone 3 for the instance types."
}

locals {
  zones = ["${var.zone_0}", "${var.zone_1}", "${var.zone_2}"]
}

resource "google_compute_address" "cluster_ip" {
  provider = google-beta

  name = "${var.infra_id}-cluster-ip"
  address_type = "INTERNAL"
  region = "${var.region}"
  subnetwork = "${var.control_subnet}"
}

resource "google_compute_health_check" "api_internal_health_check" {
  provider = google-beta

  name = "${var.infra_id}-api-internal-health-check"
  https_health_check {
    port = 6443
  }
}

resource "google_compute_region_backend_service" "api_internal" {
  provider = google-beta

  name = "${var.infra_id}-api-internal"
  timeout_sec = 120
  protocol = "TCP"
  region = "${var.region}"
  load_balancing_scheme = "INTERNAL"
  health_checks = [
    google_compute_health_check.api_internal_health_check.id
  ]

  dynamic "backend" {
    for_each = google_compute_instance_group.master_ig

    content {
      balancing_mode = "CONNECTION"
      group = backend.value.self_link
    }
  }
}

resource "google_compute_forwarding_rule" "api_internal_forwarding_rule" {
  provider = google-beta

  name = "${var.infra_id}-api-internal-forwarding-rule"
  ip_address = google_compute_address.cluster_ip.address
  backend_service = google_compute_region_backend_service.api_internal.id
  load_balancing_scheme = "INTERNAL"
  ports = [
    "6443",
    "22623"
  ]
  region = "${var.region}"
  subnetwork = "${var.control_subnet}"
}

resource "google_compute_instance_group" "master_ig" {
  provider = google-beta

  for_each = toset(local.zones)

  name = "${var.infra_id}-master-${each.key}-ig"
  network = "${var.cluster_network}"
  zone = "${each.key}"
  named_port {
    name = "ignition"
    port = 22623
  }
  named_port {
    name = "https"
    port = 6443
  }
}
```

## Creating a private DNS zone in Google Cloud {#installation-creating-gcp-private-dns_installing-gcp-user-infra}

You must configure a private DNS zone in Google Cloud for your OpenShift Container Platform cluster to use. One way to create this component is to modify the provided Infrastructure Manager template.

:::note

If you do not use the provided template to create your Google Cloud infrastructure, you must review the provided information and manually create the infrastructure. If your cluster does not initialize correctly, you might have to contact Red Hat support with your installation logs.

:::

**Prerequisites**

- Ensure you defined the variables in the *Exporting common variables* and *Creating load balancers in Google Cloud* sections.

**Procedure**

1. Copy the template from the **Infrastructure Manager template for the private DNS** section of this topic and save it as `02_dns.tf` in a folder called `02_dns` on your computer. This template describes the private DNS objects that your cluster requires.
2. If you are installing a cluster into a shared VPC, and the host project already has a private DNS zone, skip this step. Create the DNS zone by running the following command:
   ```terminal
   $ gcloud infra-manager deployments apply <dns_zone_deployment_name> \
     --location=${REGION} \
     --input-values=infra_id=${INFRA_ID},project=${PROJECT_NAME},region=${REGION},cluster_domain=${CLUSTER_DOMAIN},cluster_network=${CLUSTER_NETWORK} \
     --project=${PROJECT_NAME} \
     --local-source=./02_dns \
     --service-account=${INSTALL_SERVICE_ACCOUNT}
   ```

   `<dns_zone_deployment_name>` specifies the name of the DNS zone deployment you create.
3. The templates do not create DNS entries due to limitations of Infrastructure Manager, so you must create them manually:
   1. Add the internal DNS entries by running the following commands:
      ```terminal
      $ if [ -f transaction.yaml ]; then rm transaction.yaml; fi
      ```

      ```terminal
      $ gcloud dns record-sets transaction start --zone ${INFRA_ID}-private-zone
      ```

      ```terminal
      $ gcloud dns record-sets transaction add ${CLUSTER_IP} --name api.${CLUSTER_NAME}.${BASE_DOMAIN}. --ttl 60 --type A --zone ${INFRA_ID}-private-zone
      ```

      ```terminal
      $ gcloud dns record-sets transaction add ${CLUSTER_IP} --name api-int.${CLUSTER_NAME}.${BASE_DOMAIN}. --ttl 60 --type A --zone ${INFRA_ID}-private-zone
      ```

      ```terminal
      $ gcloud dns record-sets transaction execute --zone ${INFRA_ID}-private-zone
      ```
   2. For an external cluster, also add the external DNS entries by running the following commands:
      ```terminal
      $ if [ -f transaction.yaml ]; then rm transaction.yaml; fi
      ```

      ```terminal
      $ gcloud dns record-sets transaction start --zone ${BASE_DOMAIN_ZONE_NAME}
      ```

      ```terminal
      $ gcloud dns record-sets transaction add ${CLUSTER_PUBLIC_IP} --name api.${CLUSTER_NAME}.${BASE_DOMAIN}. --ttl 60 --type A --zone ${BASE_DOMAIN_ZONE_NAME}
      ```

      ```terminal
      $ gcloud dns record-sets transaction execute --zone ${BASE_DOMAIN_ZONE_NAME}
      ```

**Verification**

1. Verify the deployment is active by running the following command:
   ```terminal
   $ gcloud infra-manager deployments describe <deployment_name> --format='value(state)'
   ```

   Replace `<deployment_name>` with the name of the deployment you created.

   ```terminal title="Example output"
   ACTIVE
   ```

### Infrastructure Manager template for the private DNS {#installation-infrastructure-manager-private-dns_installing-gcp-user-infra}

You can use the following `02_dns.tf` Infrastructure Manager template to deploy the private DNS that you need for your OpenShift Container Platform cluster:

```terraform
terraform {
  # Infra manager supports specific Terraform versions; ensure compatibility
  required_version = ">=1.2.3"
  required_providers {
    google = {
      source = "hashicorp/google"
      version = ">= 4.0.0"
    }
    google-beta = {
        source = "hashicorp/google-beta",
        version = ">= 4.0.0"
    }
  }
}

provider "google-beta" {
  project = "${var.project}"
  region = "${var.region}"
}

variable "infra_id" {
  type        = string
  description = "OpenShift Installer Infrastructure ID"
}

variable "project" {
  type        = string
  description = "Project ID"
}

variable "region" {
  type        = string
  description = "GCP Region where the resources will be created."
  default     = "us-central1"
}

variable "cluster_domain" {
  type        = string
  description = "ClusterName.BaseDomain"
}

variable "cluster_network" {
  type        = string
  description = "Full link to the cluster network."
}

resource "google_dns_managed_zone" "private_zone" {
  provider = google-beta

  name = "${var.infra_id}-private-zone"
  dns_name = "${var.cluster_domain}."
  description = "OpenShift Installer UPI create private DNS zone."
  visibility = "private"
  private_visibility_config {
    networks {
      network_url = "${var.cluster_network}"
    }
  }

  force_destroy = false
}
```

## Creating firewall rules and IAM roles in Google Cloud {#installation-creating-gcp-firewall-rules-vpc_installing-gcp-user-infra}

You must create firewall rules and IAM roles in Google Cloud for your OpenShift Container Platform cluster to use. One way to create these components is to modify the provided Infrastructure Manager template. If you are installing a cluster into a shared VPC and the host project already has the necessary firewall rules and IAM roles, you can skip creating these resources.

:::note

If you do not use the provided Infrastructure Manager template to create your Google Cloud infrastructure, you must review the provided information and manually create the infrastructure. If your cluster does not initialize correctly, you might have to contact Red Hat support with your installation logs.

:::

**Prerequisites**

- Ensure you defined the variables in the *Exporting common variables* and *Creating load balancers in Google Cloud* sections.

**Procedure**

1. Copy the template from the **Infrastructure Manager template for firewall rules and IAM roles** section of this topic and save it as `03_security.tf` in a folder called `03_security` on your computer. This template describes the security groups that your cluster requires.
2. Create the firewall rules and IAM roles by running the following command:
   ```terminal
   $ gcloud infra-manager deployments apply <security_deployment_name> \
     --location=${REGION} \
     --project=${PROJECT_NAME} \
     --local-source=./03_security \
     --input-values=infra_id=${INFRA_ID},project=${PROJECT_NAME},region=${REGION},cluster_network=${CLUSTER_NETWORK},network_cidr=${NETWORK_CIDR} \
     --service-account=${INSTALL_SERVICE_ACCOUNT}
   ```

   `<security_deployment_name>` specifies the name of the deployment of firewall rules and IAM roles.
3. Configure service account variables based on the roles you created by running the following commands:
   ```terminal
   $ export MASTER_SERVICE_ACCOUNT=$(gcloud iam service-accounts list --filter "email~^${INFRA_ID}-m@${PROJECT_NAME}." --format json | jq -r '.[0].email')
   ```

   ```terminal
   $ export WORKER_SERVICE_ACCOUNT=$(gcloud iam service-accounts list --filter "email~^${INFRA_ID}-w@${PROJECT_NAME}." --format json | jq -r '.[0].email')
   ```

**Verification**

1. Verify the deployment is active by running the following command:
   ```terminal
   $ gcloud infra-manager deployments describe <deployment_name> --format='value(state)'
   ```

   Replace `<deployment_name>` with the name of the deployment you created.

   ```terminal title="Example output"
   ACTIVE
   ```

### Infrastructure Manager template for firewall rules and IAM roles {#installation-infrastructure-manager-firewall-rules_installing-gcp-user-infra}

You can use the following `03_security.tf` Infrastructure Manager template to deploy the firewall rules and IAM roles that you need for your OpenShift Container Platform cluster:

```terraform
terraform {
  # Infra manager supports specific Terraform versions; ensure compatibility
  required_version = ">=1.2.3"
  required_providers {
    google = {
      source = "hashicorp/google"
      version = ">= 4.0.0"
    }
    google-beta = {
        source = "hashicorp/google-beta",
        version = ">= 4.0.0"
    }
  }
}

provider "google-beta" {
  project = "${var.project}"
  region = "${var.region}"
}

variable "infra_id" {
  type        = string
  description = "OpenShift Installer Infrastructure ID"
}

variable "project" {
  type        = string
  description = "Project ID"
}

variable "region" {
  type        = string
  description = "GCP Region where the resources will be created."
  default     = "us-central1"
}

variable "cluster_network" {
  type        = string
  description = "Full link to the cluster network."
}

variable "network_cidr" {
  type        = string
  description = "CIDR for network of the cluster."
}

variable "allowed_external_cidr" {
  type        = string
  description = "Allowed external CIDR for firewall rule."
  default     = "0.0.0.0/0"
}

resource "google_compute_firewall" "bootstrap_in_ssh" {
  provider = google-beta

  name = "${var.infra_id}-bootstrap-in-ssh"
  source_ranges = [
    "${var.allowed_external_cidr}"
  ]
  target_tags = [
    "${var.infra_id}-bootstrap"
  ]
  network = "${var.cluster_network}"
  allow {
    protocol = "tcp"
    ports = ["22"]
  }
}

resource "google_compute_firewall" "api" {
  provider = google-beta

  name = "${var.infra_id}-api"
  source_ranges = [
   "${var.allowed_external_cidr}"
  ]
  target_tags = [
    "${var.infra_id}-master"
  ]
  network = "${var.cluster_network}"
  allow {
    protocol = "tcp"
    ports = ["6443"]
  }
}

resource "google_compute_firewall" "health_checks" {
  provider = google-beta

  name = "${var.infra_id}-health-checks"
  source_ranges = [
    "35.191.0.0/16",
    "130.211.0.0/22",
    "209.85.152.0/22",
    "209.85.204.0/22"
  ]
  target_tags = [
    "${var.infra_id}-master"
  ]
  network = "${var.cluster_network}"
  allow {
    protocol = "tcp"
    ports = ["6080", "6443", "22624"]
  }
}

resource "google_compute_firewall" "etcd" {
  provider = google-beta

  name = "${var.infra_id}-etcd"
  source_tags = [
    "${var.infra_id}-master"
  ]
  target_tags = [
    "${var.infra_id}-master"
  ]
  network = "${var.cluster_network}"
  allow {
    protocol = "tcp"
    ports = ["2379-2380"]
  }
}

resource "google_compute_firewall" "control_plane" {
  provider = google-beta

  name = "${var.infra_id}-control-plane"
  source_tags = [
    "${var.infra_id}-master",
    "${var.infra_id}-worker"
  ]
  target_tags = [
    "${var.infra_id}-master"
  ]
  network = "${var.cluster_network}"
  allow {
    protocol = "tcp"
    ports = ["10257"]
  }
  allow {
    protocol = "tcp"
    ports = ["10259"]
  }
  allow {
    protocol = "tcp"
    ports = ["22623"]
  }
}

resource "google_compute_firewall" "internal_network" {
  provider = google-beta

  name = "${var.infra_id}-internal-network"
  source_ranges = [
    "${var.network_cidr}"
  ]
  target_tags = [
    "${var.infra_id}-master",
    "${var.infra_id}-worker"
  ]
  network = "${var.cluster_network}"
  allow {
    protocol = "icmp"
  }
  allow {
    protocol = "tcp"
    ports = ["22"]
  }
}

resource "google_compute_firewall" "internal_cluster" {
  provider = google-beta

  name = "${var.infra_id}-internal-cluster"
  source_tags = [
    "${var.infra_id}-master",
    "${var.infra_id}-worker"
  ]
  target_tags = [
    "${var.infra_id}-master",
    "${var.infra_id}-worker"
  ]
  network = "${var.cluster_network}"
  allow {
    protocol = "udp"
    ports = ["4789", "6081"]
  }
  allow {
    protocol = "udp"
    ports = ["500", "4500"]
  }
  allow {
    protocol = "esp"
  }
  allow {
    protocol = "tcp"
    ports = ["9000-9999"]
  }
  allow {
    protocol = "udp"
    ports = ["9000-9999"]
  }
  allow {
    protocol = "tcp"
    ports = ["10250"]
  }
  allow {
    protocol = "tcp"
    ports = ["30000-32767"]
  }
  allow {
    protocol = "udp"
    ports = ["30000-32767"]
  }
}

resource "google_service_account" "master_node_sa" {
  provider = google-beta

  account_id = "${var.infra_id}-m"
  display_name = "${var.infra_id}-master-node"
}

resource "google_service_account" "worker_node_sa" {
  provider = google-beta

  account_id = "${var.infra_id}-w"
  display_name = "${var.infra_id}-worker-node"
}
```

## Creating IAM policy bindings in Google Cloud {#installation-creating-gcp-iam-shared-vpc_installing-gcp-user-infra}

You must create IAM policy bindings in Google Cloud for your OpenShift Container Platform cluster to use.

**Prerequisites**

- You have defined the variables in the *Exporting common variables* section.

**Procedure**

1. Export the variable for the subnet that hosts the compute machines by running the following command:
   ```terminal
   $ export COMPUTE_SUBNET=(`gcloud compute networks subnets describe ${INFRA_ID}-worker-subnet --region=${REGION} --format json | jq -r .selfLink`)
   ```
2. The templates do not create the policy bindings due to limitations of Infrastructure Manager, so you must create them manually by running the following commands:
   ```terminal
   $ gcloud projects add-iam-policy-binding ${PROJECT_NAME} --member "serviceAccount:${MASTER_SERVICE_ACCOUNT}" --role "roles/compute.instanceAdmin"
   ```

   ```terminal
   $ gcloud projects add-iam-policy-binding ${PROJECT_NAME} --member "serviceAccount:${MASTER_SERVICE_ACCOUNT}" --role "roles/compute.networkAdmin"
   ```

   ```terminal
   $ gcloud projects add-iam-policy-binding ${PROJECT_NAME} --member "serviceAccount:${MASTER_SERVICE_ACCOUNT}" --role "roles/compute.securityAdmin"
   ```

   ```terminal
   $ gcloud projects add-iam-policy-binding ${PROJECT_NAME} --member "serviceAccount:${MASTER_SERVICE_ACCOUNT}" --role "roles/iam.serviceAccountUser"
   ```

   ```terminal
   $ gcloud projects add-iam-policy-binding ${PROJECT_NAME} --member "serviceAccount:${MASTER_SERVICE_ACCOUNT}" --role "roles/storage.admin"
   ```

   ```terminal
   $ gcloud projects add-iam-policy-binding ${PROJECT_NAME} --member "serviceAccount:${WORKER_SERVICE_ACCOUNT}" --role "roles/compute.viewer"
   ```

   ```terminal
   $ gcloud projects add-iam-policy-binding ${PROJECT_NAME} --member "serviceAccount:${WORKER_SERVICE_ACCOUNT}" --role "roles/storage.admin"
   ```
3. Create a service account key and store it locally for later use by running the following command:
   ```terminal
   $ gcloud iam service-accounts keys create service-account-key.json --iam-account=${MASTER_SERVICE_ACCOUNT}
   ```

## Creating the RHCOS cluster image for the Google Cloud infrastructure {#installation-gcp-user-infra-rhcos_installing-gcp-user-infra}

To deploy OpenShift Container Platform nodes on Google Cloud, you must create a valid Red Hat Enterprise Linux CoreOS (RHCOS) image in your Google Cloud project because RHCOS images are not pre-published on Google Cloud.

**Prerequisites**

- You have downloaded the `openshift-install` binary.

**Procedure**

1. Obtain the image name by running the following command:
   ```terminal
   $ source_image=$(openshift-install coreos print-stream-json | jq -r '.architectures.x86_64.images.gcp.name')
   ```
2. Obtain the project name by running the following command:
   ```terminal
   $ source_project=$(openshift-install coreos print-stream-json | jq -r '.architectures.x86_64.images.gcp.project')
   ```
3. Create the image by running the following command:
   ```terminal
   $ gcloud compute images create "${INFRA_ID}-rhcos-image" \
       --source-image="${source_image}" --source-image-project="${source_project}"
   ```

## Creating the bootstrap machine in Google Cloud {#installation-creating-gcp-bootstrap_installing-gcp-user-infra}

You must create the bootstrap machine in Google Cloud to use during OpenShift Container Platform cluster initialization. One way to create this machine is to modify the provided Infrastructure Manager template.

:::note

If you do not use the provided Infrastructure Manager template to create your bootstrap machine, you must review the provided information and manually create the infrastructure. If your cluster does not initialize correctly, you might have to contact Red Hat support with your installation logs.

If you need to redeploy the bootstrap machine for any reason, delete the existing bootstrap VM first. If you redeploy the bootstrap machine without deleting the existing VM, Infrastructure Manager will update the metadata and appear to succeed, but the Ignition file will not be executed again. This will result in the VM still being based on the old Ignition data.

:::

**Prerequisites**

- Ensure you defined the variables in the *Exporting common variables* and *Creating load balancers in Google Cloud* sections.

**Procedure**

1. Copy the template from the **Infrastructure Manager template for the bootstrap machine** section of this topic and save it as `04_bootstrap.tf` in a folder called `04_bootstrap` on your computer. This template describes the bootstrap machine that your cluster requires.
   - You can edit the `04_bootstrap.tf` file to add additional tags to the bootstrap machine, by modifying the existing `tags` stanza as follows:
     ```bash
     resource "google_compute_instance" "bootstrap" {
     # ...
       tags = [
         "${var.infra_id}-master",
         "${var.infra_id}-bootstrap",
         "custom-tag-example"
       ]
     # ...
     }
     ```
2. Export the location of the Red Hat Enterprise Linux CoreOS (RHCOS) image that the installation program requires by running the following command:
   ```terminal
   $ export CLUSTER_IMAGE=(`gcloud compute images describe ${INFRA_ID}-rhcos-image --format json | jq -r .selfLink`)
   ```
3. Create a bucket by running the following command:
   ```terminal
   $ gcloud storage buckets create "gs://${INFRA_ID}-bootstrap-ignition"
   ```
4. Upload the `bootstrap.ign` file by running the following command:
   ```terminal
   $ gcloud storage cp bootstrap.ign "gs://${INFRA_ID}-bootstrap-ignition/"
   ```
5. Create a signed URL for the bootstrap instance and export the URL from the output as a variable by running the following command:
   ```terminal
   $ export BOOTSTRAP_IGN="$(gcloud storage sign-url --duration=2h --private-key-file=service-account-key.json "gs://${INFRA_ID}-bootstrap-ignition/bootstrap.ign" | grep "^signed_url:" | awk '{print $2}')"
   ```
6. Create the bootstrap deployment by running the following command:
   ```terminal
   $ gcloud infra-manager deployments apply <bootstrap_deployment_name> \
     --location=${REGION} \
     --project=${PROJECT_NAME} \
     --local-source=./04_bootstrap \
     --input-values=infra_id=${INFRA_ID},project=${PROJECT_NAME},region=${REGION},zone=${ZONE_0},cluster_network=${CLUSTER_NETWORK},subnet=${CONTROL_SUBNET},image=${CLUSTER_IMAGE},bootstrap_ign="${BOOTSTRAP_IGN}",is_public_cluster=<public_cluster_status> \
     --service-account=${INSTALL_SERVICE_ACCOUNT}
   ```

   where:

   <dl>
   <dt><code>&lt;bootstrap_deployment_name&gt;</code></dt>
   <dd>Specifies the name of the bootstrap deployment.</dd>
   <dt><code>&lt;public_cluster_status&gt;</code></dt>
   <dd>Specifies whether the cluster is public or private. If it is a public cluster, specify <code>true</code>. If it is a private cluster, specify <code>false</code>.</dd>
   </dl>
7. The templates do not manage load balancer membership due to limitations of Infrastructure Manager, so you must add the bootstrap machine manually.
   1. Add the bootstrap instance to the internal load balancer instance group by running the following command:
      ```terminal
      $ gcloud compute instance-groups unmanaged add-instances \
          ${INFRA_ID}-bootstrap-ig --zone=${ZONE_0} --instances=${INFRA_ID}-bootstrap
      ```
   2. Add the bootstrap instance group to the internal load balancer backend service by running the following command:
      ```terminal
      $ gcloud compute backend-services add-backend \
          ${INFRA_ID}-api-internal --region=${REGION} --instance-group=${INFRA_ID}-bootstrap-ig --instance-group-zone=${ZONE_0}
      ```

**Verification**

1. Verify the deployment is active by running the following command:
   ```terminal
   $ gcloud infra-manager deployments describe <deployment_name> --format='value(state)'
   ```

   Replace `<deployment_name>` with the name of the deployment you created.

   ```terminal title="Example output"
   ACTIVE
   ```

### Infrastructure Manager template for the bootstrap machine {#installation-infrastructure-manager-bootstrap_installing-gcp-user-infra}

You can use the following `04_bootstrap.tf` Infrastructure Manager template to deploy the bootstrap machine that you need for your OpenShift Container Platform cluster:

```terraform
terraform {
  # Infra manager supports specific Terraform versions; ensure compatibility
  required_version = ">=1.2.3"
  required_providers {
    google = {
      source = "hashicorp/google"
      version = ">= 4.0.0"
    }
    google-beta = {
        source = "hashicorp/google-beta",
        version = ">= 4.0.0"
    }
  }
}

provider "google-beta" {
  project = "${var.project}"
  region = "${var.region}"
}

variable "infra_id" {
  type        = string
  description = "OpenShift Installer Infrastructure ID"
}

variable "project" {
  type        = string
  description = "Project ID"
}

variable "region" {
  type        = string
  description = "GCP Region where the resources will be created."
  default     = "us-central1"
}

variable "zone" {
  type        = string
  description = "Zone inside of the region where the bootstrap node is created."
}

variable "cluster_network" {
  type        = string
  description = "Full link to the cluster network."
}

variable "subnet" {
  type        = string
  description = "Control plane subnet."
}

variable "image" {
  type        = string
  description = "Cluster Image."
}

variable "machine_type" {
  type        = string
  description = "Machine type for the bootstrap machine."
  default     = "n1-standard-4"
}

variable "root_volume_size" {
  type        = string
  description = "Size in GB for the root volume."
  default     = "128"
}

variable "bootstrap_ign" {
  type        = string
  description = "Bootstrap ignition data."
}

variable "is_public_cluster" {
  type        = bool
  default     = true
  description = "Whether the publish policy is the default External"
}

resource "google_compute_address" "bootstrap_public_ip" {
  provider = google-beta
  count = var.is_public_cluster ? 1 : 0

  name = "${var.infra_id}-bootstrap-public-ip"
  region = "${var.region}"
}

resource "google_compute_instance" "bootstrap" {
  provider = google-beta

  name = "${var.infra_id}-bootstrap"
  zone = "${var.zone}"
  machine_type = "${var.machine_type}"
  tags = [
    "${var.infra_id}-master",
    "${var.infra_id}-bootstrap"
  ]
  boot_disk {
    auto_delete = true
    initialize_params {
      size = "${var.root_volume_size}"
      image = "${var.image}"
    }
  }
  network_interface {
    subnetwork = "${var.subnet}"

    # Dynamic block to conditionally create access_config
    dynamic "access_config" {
      for_each = var.is_public_cluster ? [1] : []
      content {
        nat_ip = google_compute_address.bootstrap_public_ip[0].address
      }
    }
  }
  metadata = {
    user-data = "{\"ignition\":{\"config\":{\"replace\":{\"source\":\"${var.bootstrap_ign}\"}},\"version\":\"3.2.0\"}}"
  }
}

resource "google_compute_instance_group" "bootstrap_ig" {
  provider = google-beta

  name = "${var.infra_id}-bootstrap-ig"
  network = "${var.cluster_network}"
  zone = "${var.zone}"
  named_port {
    name = "ignition"
    port = 22623
  }
  named_port {
    name = "https"
    port = 6443
  }
}
```

## Creating the control plane machines in Google Cloud {#installation-creating-gcp-control-plane_installing-gcp-user-infra}

You must create the control plane machines in Google Cloud for your cluster to use. One way to create these machines is to modify the provided Infrastructure Manager template.

:::note

If you do not use the provided template to create your control plane machines, you must review the provided information and manually create the infrastructure. If your cluster does not initialize correctly, you might have to contact Red Hat support with your installation logs.

:::

**Prerequisites**

- You defined the variables in the *Exporting common variables*, *Creating load balancers in Google Cloud*, *Creating IAM roles in Google Cloud*, and *Creating the bootstrap machine in Google Cloud* sections.
- You created the bootstrap machine.
- You created the Ignition configuration files.

**Procedure**

1. Copy the template from the **Infrastructure Manager template for control plane machines** section of this topic and save it as `05_control_plane.tf` in a folder called `05_control_plane` on your computer. This template describes the control plane machines that your cluster requires.
   - You can edit the `05_control_plane.tf` file to add additional tags to the control plane machines, by modifying the existing `tags` stanza. The following example adds a custom tag to the first control plane machine, which is named `master_0`:
     ```bash
     resource "google_compute_instance" "master_0" {
     # ...
       tags = [
         "${var.infra_id}-master",
         "custom_tag_example"
       ]
     # ...
     }
     ```
2. Copy the `master.ign` file from your installation directory into the `05_control_plane` folder by running the following command:
   ```terminal
   $ cp <installation_directory>/master.ign 05_control_plane/master.ign
   ```

   `<installation_directory>` specifies the directory where you created the Ignition configuration files.
3. Create the control plane deployment by running the following command:
   ```terminal
   $ gcloud infra-manager deployments apply <control_plane_deployment> \
     --location=${REGION} \
     --project=${PROJECT_NAME} \
     --local-source=./05_control_plane \
     --input-values=infra_id=${INFRA_ID},project=${PROJECT_NAME},region=${REGION},zone_0=${ZONE_0},zone_1=${ZONE_1},zone_2=${ZONE_2},subnet=${CONTROL_SUBNET},image=${CLUSTER_IMAGE},service_account_email=${MASTER_SERVICE_ACCOUNT} \
     --service-account=${INSTALL_SERVICE_ACCOUNT}
   ```

   `<control_plane_deployment>` specifies the name of the control plane deployment.
4. Delete the temporary ignition file from the `05_control_plane` folder by running the following command:
   ```terminal
   $ rm 05_control_plane/master.ign
   ```
5. The templates do not manage load balancer membership due to limitations of Infrastructure Manager, so you must add the control plane machines manually.
   1. Add the first control plane machine to an internal load balancer instance group by running the following command:
      ```terminal
      $ gcloud compute instance-groups unmanaged add-instances ${INFRA_ID}-master-${ZONE_0}-ig --zone=${ZONE_0} --instances=${INFRA_ID}-master-0
      ```
   2. Add the second control plane machine to an internal load balancer instance group by running the following command:
      ```terminal
      $ gcloud compute instance-groups unmanaged add-instances ${INFRA_ID}-master-${ZONE_1}-ig --zone=${ZONE_1} --instances=${INFRA_ID}-master-1
      ```
   3. Add the third control plane machine to an internal load balancer instance group by running the following command:
      ```terminal
      $ gcloud compute instance-groups unmanaged add-instances ${INFRA_ID}-master-${ZONE_2}-ig --zone=${ZONE_2} --instances=${INFRA_ID}-master-2
      ```
6. For an external cluster, you must also add the control plane machines to external load balancer target pools.
   1. Add the first control plane machine to an external load balancer pool by running the following command:
      ```terminal
      $ gcloud compute target-pools add-instances ${INFRA_ID}-api-target-pool --instances-zone="${ZONE_0}" --instances=${INFRA_ID}-master-0
      ```
   2. Add the second control plane machine to an external load balancer pool by running the following command:
      ```terminal
      $ gcloud compute target-pools add-instances ${INFRA_ID}-api-target-pool --instances-zone="${ZONE_1}" --instances=${INFRA_ID}-master-1
      ```
   3. Add the third control plane machine to an external load balancer pool by running the following command:
      ```terminal
      $ gcloud compute target-pools add-instances ${INFRA_ID}-api-target-pool --instances-zone="${ZONE_2}" --instances=${INFRA_ID}-master-2
      ```

**Verification**

1. Verify the deployment is active by running the following command:
   ```terminal
   $ gcloud infra-manager deployments describe <deployment_name> --format='value(state)'
   ```

   Replace `<deployment_name>` with the name of the deployment you created.

   ```terminal title="Example output"
   ACTIVE
   ```

### Infrastructure Manager template for control plane machines {#installation-infrastructure-manager-control-plane_installing-gcp-user-infra}

You can use the following `05_control_plane.tf` Infrastructure Manager template to deploy the control plane machines that you need for your OpenShift Container Platform cluster:

```terraform
terraform {
  # Infra manager supports specific Terraform versions; ensure compatibility
  required_version = ">=1.2.3"
  required_providers {
    google = {
      source = "hashicorp/google"
      version = ">= 4.0.0"
    }
    google-beta = {
        source = "hashicorp/google-beta",
        version = ">= 4.0.0"
    }
    local = {
        source = "hashicorp/local",
        version = ">= 2.0.0"
    }
  }
}

provider "google-beta" {
  project = "${var.project}"
  region = "${var.region}"
}

variable "infra_id" {
  type        = string
  description = "OpenShift Installer Infrastructure ID"
}

variable "project" {
  type        = string
  description = "Project ID"
}

variable "region" {
  type        = string
  description = "GCP Region where the resources will be created."
  default     = "us-central1"
}

# Terraform handles lists but the infra-manager --input-values only
# supports scalar types.
# If you require more or less zones, you must manually add them below
# as a single variable for each. You must add the zones to the
# locals `zones` list below.
variable "zone_0" {
  type        = string
  description = "Zone 1 for the instance types."
}

variable "zone_1" {
  type        = string
  description = "Zone 2 for the instance types."
}

variable "zone_2" {
  type        = string
  description = "Zone 3 for the instance types."
}

variable "subnet" {
  type        = string
  description = "Control plane subnet."
}

variable "image" {
  type        = string
  description = "Cluster Image."
}

variable "machine_type" {
  type        = string
  description = "Machine type for the control plane machine."
  default     = "n1-standard-4"
}

variable "disk_size" {
  type        = string
  description = "Size in GB for the root volume."
  default     = "128"
}

variable "disk_type" {
  type        = string
  description = "Type of storage disk for the vm."
  default     = "pd-ssd"
}

variable "service_account_email" {
  type        = string
  description = "Email for the service account attached to the control planes."
}

data "local_file" "ignition_file" {
  filename = "${path.module}/master.ign"
}

resource "google_compute_instance" "master_0" {
  provider = google-beta

  name = "${var.infra_id}-master-0"
  zone = "${var.zone_0}"
  machine_type = "${var.machine_type}"
  tags = [
    "${var.infra_id}-master"
  ]
  boot_disk {
    auto_delete = true
    initialize_params {
      size = "${var.disk_size}"
      image = "${var.image}"
      type = "${var.disk_type}"
    }
  }
  network_interface {
    subnetwork = "${var.subnet}"
  }
  metadata = {
    user-data = data.local_file.ignition_file.content
  }
  service_account {
    email = "${var.service_account_email}"
    scopes = ["https://www.googleapis.com/auth/cloud-platform"]
  }
}

resource "google_compute_instance" "master_1" {
  provider = google-beta

  name = "${var.infra_id}-master-1"
  zone = "${var.zone_1}"
  machine_type = "${var.machine_type}"
  tags = [
    "${var.infra_id}-master"
  ]
  boot_disk {
    auto_delete = true
    initialize_params {
      size = "${var.disk_size}"
      image = "${var.image}"
      type = "${var.disk_type}"
    }
  }
  network_interface {
    subnetwork = "${var.subnet}"
  }
  metadata = {
    user-data = data.local_file.ignition_file.content
  }
  service_account {
    email = "${var.service_account_email}"
    scopes = ["https://www.googleapis.com/auth/cloud-platform"]
  }
}

resource "google_compute_instance" "master_2" {
  provider = google-beta

  name = "${var.infra_id}-master-2"
  zone = "${var.zone_2}"
  machine_type = "${var.machine_type}"
  tags = [
    "${var.infra_id}-master"
  ]
  boot_disk {
    auto_delete = true
    initialize_params {
      size = "${var.disk_size}"
      image = "${var.image}"
      type = "${var.disk_type}"
    }
  }
  network_interface {
    subnetwork = "${var.subnet}"
  }
  metadata = {
    user-data = data.local_file.ignition_file.content
  }
  service_account {
    email = "${var.service_account_email}"
    scopes = ["https://www.googleapis.com/auth/cloud-platform"]
  }
}
```

## Creating additional worker machines in Google Cloud {#installation-creating-gcp-worker_installing-gcp-user-infra}

You can create worker machines in Google Cloud for your cluster by using the Infrastructure Manager template. You can adjust the number of machines by modifying the number of `google_compute_instance` resources in the provided template.

:::note

If you do not use the provided Infrastructure Manager template to create your compute machines, you must review the provided information and manually create the infrastructure. If your cluster does not initialize correctly, you might have to contact Red Hat support with your installation logs.

If you are installing a three-node cluster, skip this step. A three-node cluster consists of three control plane machines, which also act as compute machines.

:::

**Prerequisites**

- Ensure you defined the variables in the *Exporting common variables*, *Creating load balancers in Google Cloud*, and *Creating the bootstrap machine in Google Cloud* sections.
- Create the bootstrap machine.
- Create the control plane machines.

**Procedure**

1. Copy the template from the **Infrastructure Manager template for worker machines** section of this topic and save it as `06_worker.tf` in a folder called `06_worker` on your computer. This template describes the worker machines that your cluster requires.
   - You can edit the `06_worker.tf` file to add additional tags to the compute machines, by modifying the existing `tags` stanza as follows:
     ```bash
     resource "google_compute_instance" "worker_0" {
     # ...
       tags = [
         "${var.infra_id}-worker-0",
         "custom-tag-example"
       ]
     # ...
     }
     ```
2. Copy the `worker.ign` file from your installation directory into the `06_worker` folder by running the following command:
   ```terminal
   $ cp <installation_directory>/worker.ign 06_worker/worker.ign
   ```

   `<installation_directory>` specifies the directory where you created the Ignition configuration files.
3. Create the deployment by running the following command:
   ```terminal
   $ gcloud infra-manager deployments apply <worker_deployment_name> \
     --location=${REGION} \
     --project=${PROJECT_NAME} \
     --local-source=./06_worker \
     --input-values=infra_id=${INFRA_ID},project=${PROJECT_NAME},region=${REGION},zone_0=${ZONE_0},zone_1=${ZONE_1},subnet=${COMPUTE_SUBNET},image=${CLUSTER_IMAGE},service_account_email=${WORKER_SERVICE_ACCOUNT} \
     --service-account=${INSTALL_SERVICE_ACCOUNT}
   ```

   `<worker_deployment_name>` specifies the name of the deployment.
4. Remove the `worker.ign` file by running the following command:
   ```terminal
   $ rm 06_worker/worker.ign
   ```

**Verification**

1. Verify the deployment is active by running the following command:
   ```terminal
   $ gcloud infra-manager deployments describe <deployment_name> --format='value(state)'
   ```

   Replace `<deployment_name>` with the name of the deployment you created.

   ```terminal title="Example output"
   ACTIVE
   ```

### Infrastructure Manager template for worker machines {#installation-infrastructure-manager-worker_installing-gcp-user-infra}

You can use the following `06_worker.tf` Infrastructure Manager template to deploy the worker machines that you need for your OpenShift Container Platform cluster:

```terraform
terraform {
  # Infra manager supports specific Terraform versions; ensure compatibility
  required_version = ">=1.2.3"
  required_providers {
    google = {
      source = "hashicorp/google"
      version = ">= 4.0.0"
    }
  }
}

provider "google-beta" {
  project = "${var.project}"
  region = "${var.region}"
}

variable "infra_id" {
  type        = string
  description = "OpenShift Installer Infrastructure ID"
}

variable "project" {
  type        = string
  description = "Project ID"
}

variable "region" {
  type        = string
  description = "GCP Region where the resources will be created."
  default     = "us-central1"
}

# Terraform handles lists but the infra-manager --input-values only
# supports scalar types.
# If you require more or less zones, you must manually add them below
# as a single variable for each. You must add the zones to the
# locals `zones` list below.
variable "zone_0" {
  type        = string
  description = "Zone 1 for the instance types."
}

variable "zone_1" {
  type        = string
  description = "Zone 2 for the instance types."
}

variable "subnet" {
  type        = string
  description = "Compute subnet."
}

variable "image" {
  type        = string
  description = "Cluster Image."
}

variable "machine_type" {
  type        = string
  description = "Machine type for the compute machine."
  default     = "n1-standard-4"
}

variable "disk_size" {
  type        = string
  description = "Size in GB for the root volume."
  default     = "128"
}

variable "disk_type" {
  type        = string
  description = "Type of storage disk for the vm."
  default     = "pd-ssd"
}

variable "service_account_email" {
  type        = string
  description = "Email for the service account attached to the compute nodes."
}

data "local_file" "ignition_file" {
  filename = "${path.module}/worker.ign"
}

resource "google_compute_instance" "worker_0" {
  provider = google-beta

  name = "${var.infra_id}-worker-0"
  zone = "${var.zone_0}"
  machine_type = "${var.machine_type}"
  tags = [
    "${var.infra_id}-worker"
  ]
  boot_disk {
    auto_delete = true
    initialize_params {
      size = "${var.disk_size}"
      image = "${var.image}"
      type = "${var.disk_type}"
    }
  }
  network_interface {
    subnetwork = "${var.subnet}"
  }
  metadata = {
    user-data = data.local_file.ignition_file.content
  }
  service_account {
    email = "${var.service_account_email}"
    scopes = ["https://www.googleapis.com/auth/cloud-platform"]
  }
}

resource "google_compute_instance" "worker_1" {
  provider = google-beta

  name = "${var.infra_id}-worker-1"
  zone = "${var.zone_1}"
  machine_type = "${var.machine_type}"
  tags = [
    "${var.infra_id}-worker"
  ]
  boot_disk {
    auto_delete = true
    initialize_params {
      size = "${var.disk_size}"
      image = "${var.image}"
      type = "${var.disk_type}"
    }
  }
  network_interface {
    subnetwork = "${var.subnet}"
  }
  metadata = {
    user-data = data.local_file.ignition_file.content
  }
  service_account {
    email = "${var.service_account_email}"
    scopes = ["https://www.googleapis.com/auth/cloud-platform"]
  }
}
```

## Removing bootstrap resources in Google Cloud {#installation-gcp-user-infra-wait-for-bootstrap_installing-gcp-user-infra}

After the bootstrap process completes on your Google Cloud infrastructure, you can remove the bootstrap resources to reclaim the capacity that they consume, because the cluster no longer requires them.

**Prerequisites**

- Ensure you defined the variables in the *Exporting common variables* and *Creating load balancers in Google Cloud* sections.
- Create the bootstrap machine.
- Create the control plane machines.

**Procedure**

1. Change to the directory that includes the installation program and run the following command:
   ```terminal
   $ ./openshift-install wait-for bootstrap-complete --dir <installation_directory> \
       --log-level info
   ```

   where:

   <dl>
   <dt><code>&lt;installation_directory&gt;</code></dt>
   <dd>Specifies the path to the directory where you stored the installation files.</dd>
   <dt><code>--log-level</code></dt>
   <dd>Specifies the log level. To view different installation details, specify <code>warn</code>, <code>debug</code>, or <code>error</code> instead of <code>info</code>.</dd>
   </dl>

   If the command exits without a `FATAL` warning, your production control plane has initialized.
2. To remove the bootstrap instance group from the backend services' backends, run the following commands:
   ```terminal
   $ gcloud compute backend-services remove-backend ${INFRA_ID}-api-internal --region=${REGION} --instance-group=${INFRA_ID}-bootstrap-ig --instance-group-zone=${ZONE_0}
   ```

   ```terminal
   $ ingress_backendservice=$(gcloud compute backend-services list --filter="backends.group~${INFRA_ID}" --format='value(name)' | grep -v "${INFRA_ID}")
   ```

   1. If `ingress_backendservice` is not empty, run the following `describe` command for the bootstrap group:
      ```terminal
      $ gcloud compute backend-services describe ${ingress_backendservice} --region=${REGION}
      ```
   2. If the `describe` command displays that the bootstrap group is one of its backends, run the following `remove-backend` command to remove the bootstrap group from the backends:
      ```terminal
      $ gcloud compute backend-services remove-backend ${ingress_backendservice} --region=${REGION} --instance-group=${INFRA_ID}-bootstrap-ig --instance-group-zone=${ZONE_0}
      ```
   3. To remove the bucket and the deployment, run the following commands:
      ```terminal
      $ gcloud storage rm "gs://${INFRA_ID}-bootstrap-ignition/bootstrap.ign"
      ```

      ```terminal
      $ gcloud storage rm --recursive "gs://${INFRA_ID}-bootstrap-ignition/"
      ```

      ```terminal
      $ gcloud infra-manager deployments delete <bootstrap_deployment_name> \
          --project=${PROJECT_NAME} --location=${REGION} --quiet
      ```

      Specify the name of the bootstrap deployment you created for `<bootstrap_deployment_name>`.

## Installing the OpenShift CLI on Linux {#cli-installing-cli-linux_installing-gcp-user-infra}

To manage your cluster and deploy applications from the command line on Linux, install the OpenShift CLI (`oc`) binary. You can download the OpenShift CLI (`oc`) from the Red  Customer Portal.

:::warning

If you installed an earlier version of `oc`, you cannot use it to complete all of the commands in OpenShift Container Platform.

Download and install the new version of `oc`.

:::

**Procedure**

1. Navigate to the [Download OpenShift Container Platform](https://access.redhat.com/downloads/content/290) page on the Red Hat Customer Portal.
2. Select the architecture from the **Product Variant** list.
3. Select the appropriate version from the **Version** list.
4. Click **Download Now** next to the **OpenShift v4.22 Linux Clients** entry and save the file.
5. Unpack the archive:
   ```terminal
   $ tar xvf <file>
   ```
6. Place the `oc` binary in a directory that is on your `PATH`.
   To check your `PATH`, run the following command:

   ```terminal
   $ echo $PATH
   ```

**Verification**

- After you install the OpenShift CLI, it is available using the `oc` command:
  ```terminal
  $ oc <command>
  ```

## Installing the OpenShift CLI on Windows {#cli-installing-cli-windows_installing-gcp-user-infra}

To manage your cluster and deploy applications from the command line on Windows, install the OpenShift CLI (`oc`) binary. You can download the OpenShift CLI (`oc`) from the Red  Customer Portal.

:::warning

If you installed an earlier version of `oc`, you cannot use it to complete all of the commands in OpenShift Container Platform.

Download and install the new version of `oc`.

:::

**Procedure**

1. Navigate to the [Download OpenShift Container Platform](https://access.redhat.com/downloads/content/290) page on the Red Hat Customer Portal.
2. Select the appropriate version from the **Version** list.
3. Click **Download Now** next to the **OpenShift v4.22 Windows Client** entry and save the file.
4. Extract the archive with a ZIP program.
5. Move the `oc` binary to a directory that is on your `PATH` variable.
   To check your `PATH` variable, open the Command Prompt and run the following command:

   ```terminal
   C:\> path
   ```

**Verification**

- After you install the OpenShift CLI, it is available using the `oc` command:
  ```terminal
  C:\> oc <command>
  ```

## Installing the OpenShift CLI on macOS {#cli-installing-cli-macos_installing-gcp-user-infra}

To manage your cluster and deploy applications from the command line on macOS, install the OpenShift CLI (`oc`) binary. You can download the OpenShift CLI (`oc`) from the Red  Customer Portal.

:::warning

If you installed an earlier version of `oc`, you cannot use it to complete all of the commands in OpenShift Container Platform.

Download and install the new version of `oc`.

:::

**Procedure**

1. Navigate to the [Download OpenShift Container Platform](https://access.redhat.com/downloads/content/290) page on the Red Hat Customer Portal.
2. Select the architecture from the **Product Variant** list.
3. Select the appropriate version from the **Version** list.
4. Click **Download Now** next to the **OpenShift v4.22 macOS Clients** entry and save the file.
   :::note

   For macOS arm64, choose the **OpenShift v4.22 macOS arm64 Client** entry.

   :::
5. Extract the archive.
6. Move the `oc` binary to a directory on your `PATH` variable.
   To check your `PATH` variable, open a terminal and run the following command:

   ```terminal
   $ echo $PATH
   ```

**Verification**

- Verify your installation by using an `oc` command:
  ```terminal
  $ oc <command>
  ```

## Logging in to the cluster by using the CLI {#cli-logging-in-kubeadmin_installing-gcp-user-infra}

To log in to your cluster as the default system user, export the `kubeconfig` file. This configuration enables the CLI to authenticate and connect to the specific API server created during OpenShift Container Platform installation.

The `kubeconfig` file is specific to a cluster and OpenShift Container Platform generates it during installation.

**Prerequisites**

- You installed the OpenShift CLI (`oc`).
- Ensure the bootstrap process completed successfully.

**Procedure**

1. Export the `kubeadmin` credentials by running the following command:
   ```terminal
   $ export KUBECONFIG=<installation_directory>/auth/kubeconfig
   ```

   where:

   <dl>
   <dt><code>&lt;installation_directory&gt;</code></dt>
   <dd>Specifies the path to the directory that stores the installation files.</dd>
   </dl>
2. Verify you can run `oc` commands successfully using the exported configuration by running the following command:
   ```terminal
   $ oc whoami
   ```

   ```terminal title="Example output"
   system:admin
   ```

**Next steps**

- "Customize your cluster"
- "Remote health reporting"

## Approving the certificate signing requests for your machines {#installation-approve-csrs_installing-gcp-user-infra}

To allow newly added machines to join your OpenShift Container Platform cluster, confirm that the cluster approves pending certificate signing requests (CSRs), or approve them yourself. Approve client requests first, then server requests.

**Prerequisites**

- You added machines to your cluster.

**Procedure**

1. Confirm that the cluster recognizes the machines:
   ```terminal
   $ oc get nodes
   ```

   ```terminal title="Example output"
   NAME      STATUS    ROLES   AGE  VERSION
   master-0  Ready     master  63m  v1.35.4
   master-1  Ready     master  63m  v1.35.4
   master-2  Ready     master  64m  v1.35.4
   ```

   The output lists all of the machines that you created.

   :::note

   The preceding output might not include the compute nodes until you approve some CSRs.

   :::
2. Review the pending CSRs and ensure that you see the client requests with the `Pending` or `Approved` status for each machine that you added to the cluster:
   ```terminal
   $ oc get csr
   ```

   ```terminal title="Example output"
   NAME        AGE     REQUESTOR                                                                   CONDITION
   csr-8b2br   15m     system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Pending
   csr-8vnps   15m     system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Pending
   ...
   ```

   In this example, two machines are joining the cluster. You might see more approved CSRs in the list.
3. If the CSRs were not approved, after all of the pending CSRs for the machines you added are in `Pending` status, approve the CSRs for your cluster machines:
   :::note

   You must approve your CSRs within an hour of adding the machines to the cluster. If you do not approve them within an hour, the certificates rotate, and more than two certificates are present for each node. You must approve all of these certificates. After you approve the client CSR, the kubelet creates a secondary CSR for the serving certificate, which requires manual approval. The `machine-approver` then automatically approves later serving certificate renewal requests if the kubelet requests a new certificate with the same parameters.

   :::

   :::note

   For clusters running on platforms that are not machine API enabled, such as bare metal and other user-provisioned infrastructure, you must implement a method of automatically approving the kubelet serving certificate requests (CSRs). If you do not approve a request, the `oc exec`, `oc rsh`, and `oc logs` commands cannot succeed, because the API server requires a serving certificate when it connects to the kubelet. Any operation that contacts the kubelet endpoint requires this certificate approval to be in place. The method must watch for new CSRs, confirm that the `node-bootstrapper` service account in the `system:node` or `system:admin` groups submitted the CSR, and confirm the identity of the node.

   :::

   - To approve them individually, run the following command for each valid CSR:
     ```terminal
     $ oc adm certificate approve <csr_name>
     ```

     where:

     <dl>
     <dt><code>&lt;csr_name&gt;</code></dt>
     <dd>Specifies the name of a CSR from the list of current CSRs.</dd>
     </dl>
   - To approve all pending CSRs, run the following command:
     ```terminal
     $ oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs --no-run-if-empty oc adm certificate approve
     ```

     :::note

     Some Operators might not become available until you approve some CSRs. Each node submits two CSRs, so you might need to run the command to approve CSRs many times.

     :::
4. After you approve your client requests, review the server requests for each machine that you added to the cluster:
   ```terminal
   $ oc get csr
   ```

   ```terminal title="Example output"
   NAME        AGE     REQUESTOR                                                                   CONDITION
   csr-bfd72   5m26s   system:node:ip-10-0-50-126.us-east-2.compute.internal                       Pending
   csr-c57lv   5m26s   system:node:ip-10-0-95-157.us-east-2.compute.internal                       Pending
   ...
   ```
5. If the remaining CSRs are not approved, and are in the `Pending` status, approve the CSRs for your cluster machines:
   - To approve them individually, run the following command for each valid CSR:
     ```terminal
     $ oc adm certificate approve <csr_name>
     ```

     where:

     <dl>
     <dt><code>&lt;csr_name&gt;</code></dt>
     <dd>Specifies the name of a CSR from the list of current CSRs.</dd>
     </dl>
   - To approve all pending CSRs, run the following command:
     ```terminal
     $ oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc adm certificate approve
     ```
6. After you approve all client and server CSRs, the machines have the `Ready` status. Verify this by running the following command:
   ```terminal
   $ oc get nodes
   ```

   ```terminal title="Example output"
   NAME      STATUS    ROLES   AGE  VERSION
   master-0  Ready     master  73m  v1.35.4
   master-1  Ready     master  73m  v1.35.4
   master-2  Ready     master  74m  v1.35.4
   worker-0  Ready     worker  11m  v1.35.4
   worker-1  Ready     worker  11m  v1.35.4
   ```

   :::note

   You might need to wait a few minutes after approval of the server CSRs for the machines to reach the `Ready` status.

   :::

## Adding the ingress DNS records {#installation-gcp-user-infra-adding-ingress_installing-gcp-user-infra}

If you removed the DNS zone configuration when creating Kubernetes manifests and generating Ignition configs, you must manually create DNS records that point at the ingress load balancer so that external clients can reach the applications that run on your cluster.

You can create either a wildcard `*.apps.{baseDomain}.` or specific records. You can use A, CNAME, and other records per your requirements.

**Prerequisites**

- Ensure you defined the variables in the *Exporting common variables* section.
- Remove the DNS Zone configuration when creating Kubernetes manifests and generating Ignition configs.
- Ensure the bootstrap process completed successfully.

**Procedure**

1. Wait for the Ingress router to create a load balancer and populate the `EXTERNAL-IP` field:
   ```terminal
   $ oc -n openshift-ingress get service router-default
   ```

   ```terminal title="Example output"
   NAME             TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                      AGE
   router-default   LoadBalancer   172.30.18.154   35.233.157.184   80:32288/TCP,443:31215/TCP   98
   ```
2. Add the A record to your zones:
   - To use A records:
     1. Export the variable for the router IP address:
        ```terminal
        $ export ROUTER_IP=`oc -n openshift-ingress get service router-default --no-headers | awk '{print $4}'`
        ```
     2. Add the A record to the private zones:
        ```terminal
        $ if [ -f transaction.yaml ]; then rm transaction.yaml; fi
        ```

        ```terminal
        $ gcloud dns record-sets transaction start --zone ${INFRA_ID}-private-zone
        ```

        ```terminal
        $ gcloud dns record-sets transaction add ${ROUTER_IP} --name \*.apps.${CLUSTER_NAME}.${BASE_DOMAIN}. --ttl 300 --type A --zone ${INFRA_ID}-private-zone
        ```

        ```terminal
        $ gcloud dns record-sets transaction execute --zone ${INFRA_ID}-private-zone
        ```
     3. For an external cluster, also add the A record to the public zones:
        ```terminal
        $ if [ -f transaction.yaml ]; then rm transaction.yaml; fi
        ```

        ```terminal
        $ gcloud dns record-sets transaction start --zone ${BASE_DOMAIN_ZONE_NAME}
        ```

        ```terminal
        $ gcloud dns record-sets transaction add ${ROUTER_IP} --name \*.apps.${CLUSTER_NAME}.${BASE_DOMAIN}. --ttl 300 --type A --zone ${BASE_DOMAIN_ZONE_NAME}
        ```

        ```terminal
        $ gcloud dns record-sets transaction execute --zone ${BASE_DOMAIN_ZONE_NAME}
        ```
   - To add explicit domains instead of using a wildcard, create entries for each of the cluster’s current routes:
     ```terminal
     $ oc get --all-namespaces -o jsonpath='{range .items[*]}{range .status.ingress[*]}{.host}{"\n"}{end}{end}' routes
     ```

     ```terminal title="Example output"
     oauth-openshift.apps.your.cluster.domain.example.com
     console-openshift-console.apps.your.cluster.domain.example.com
     downloads-openshift-console.apps.your.cluster.domain.example.com
     alertmanager-main-openshift-monitoring.apps.your.cluster.domain.example.com
     prometheus-k8s-openshift-monitoring.apps.your.cluster.domain.example.com
     ```

## Completing a Google Cloud installation on user-provisioned infrastructure {#installation-gcp-user-infra-installation_installing-gcp-user-infra}

After you start the OpenShift Container Platform installation on Google Cloud user-provisioned infrastructure, you can monitor the cluster events to confirm that the installation completes successfully and the cluster is ready for use.

**Prerequisites**

- Ensure the bootstrap process completed successfully.

**Procedure**

1. Complete the cluster installation:
   ```terminal
   $ ./openshift-install --dir <installation_directory> wait-for install-complete
   ```

   where `<installation_directory>` specifies the path to the directory that you stored the installation files in.

   ```terminal title="Example output"
   INFO Waiting up to 30m0s for the cluster to initialize...
   ```

   :::warning

   - The Ignition config files that the installation program generates contain certificates that expire after 24 hours, which are then renewed at that time. If the cluster is shut down before renewing the certificates and the cluster is later restarted after the 24 hours have elapsed, the cluster automatically recovers the expired certificates. The exception is that you must manually approve the pending `node-bootstrapper` certificate signing requests (CSRs) to recover kubelet certificates. See the documentation for *Recovering from expired control plane certificates* for more information.
   - It is recommended that you use Ignition config files within 12 hours after they are generated because the 24-hour certificate rotates from 16 to 22 hours after the cluster is installed. By using the Ignition config files within 12 hours, you can avoid installation failure if the certificate update runs during installation.

   :::
2. Observe the running state of your cluster.
   1. Run the following command to view the current cluster version and status:
      ```terminal
      $ oc get clusterversion
      ```

      ```terminal title="Example output"
      NAME      VERSION   AVAILABLE   PROGRESSING   SINCE   STATUS
      version             False       True          24m     Working towards 4.5.4: 99% complete
      ```
   2. Run the following command to view the Operators managed on the control plane by the Cluster Version Operator (CVO):
      ```terminal
      $ oc get clusteroperators
      ```

      ```terminal title="Example output"
      NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE
      authentication                             4.5.4     True        False         False      7m56s
      cloud-credential                           4.5.4     True        False         False      31m
      cluster-autoscaler                         4.5.4     True        False         False      16m
      console                                    4.5.4     True        False         False      10m
      csi-snapshot-controller                    4.5.4     True        False         False      16m
      dns                                        4.5.4     True        False         False      22m
      etcd                                       4.5.4     False       False         False      25s
      image-registry                             4.5.4     True        False         False      16m
      ingress                                    4.5.4     True        False         False      16m
      insights                                   4.5.4     True        False         False      17m
      kube-apiserver                             4.5.4     True        False         False      19m
      kube-controller-manager                    4.5.4     True        False         False      20m
      kube-scheduler                             4.5.4     True        False         False      20m
      kube-storage-version-migrator              4.5.4     True        False         False      16m
      machine-api                                4.5.4     True        False         False      22m
      machine-config                             4.5.4     True        False         False      22m
      marketplace                                4.5.4     True        False         False      16m
      monitoring                                 4.5.4     True        False         False      10m
      network                                    4.5.4     True        False         False      23m
      node-tuning                                4.5.4     True        False         False      23m
      openshift-apiserver                        4.5.4     True        False         False      17m
      openshift-controller-manager               4.5.4     True        False         False      15m
      openshift-samples                          4.5.4     True        False         False      16m
      operator-lifecycle-manager                 4.5.4     True        False         False      22m
      operator-lifecycle-manager-catalog         4.5.4     True        False         False      22m
      operator-lifecycle-manager-packageserver   4.5.4     True        False         False      18m
      service-ca                                 4.5.4     True        False         False      23m
      service-catalog-apiserver                  4.5.4     True        False         False      23m
      service-catalog-controller-manager         4.5.4     True        False         False      23m
      storage                                    4.5.4     True        False         False      17m
      ```
   3. Run the following command to view your cluster pods:
      ```terminal
      $ oc get pods --all-namespaces
      ```

      ```terminal title="Example output"
      NAMESPACE                                               NAME                                                                READY     STATUS      RESTARTS   AGE
      kube-system                                             etcd-member-ip-10-0-3-111.us-east-2.compute.internal                1/1       Running     0          35m
      kube-system                                             etcd-member-ip-10-0-3-239.us-east-2.compute.internal                1/1       Running     0          37m
      kube-system                                             etcd-member-ip-10-0-3-24.us-east-2.compute.internal                 1/1       Running     0          35m
      openshift-apiserver-operator                            openshift-apiserver-operator-6d6674f4f4-h7t2t                       1/1       Running     1          37m
      openshift-apiserver                                     apiserver-fm48r                                                     1/1       Running     0          30m
      openshift-apiserver                                     apiserver-fxkvv                                                     1/1       Running     0          29m
      openshift-apiserver                                     apiserver-q85nm                                                     1/1       Running     0          29m
      ...
      openshift-service-ca-operator                           openshift-service-ca-operator-66ff6dc6cd-9r257                      1/1       Running     0          37m
      openshift-service-ca                                    apiservice-cabundle-injector-695b6bcbc-cl5hm                        1/1       Running     0          35m
      openshift-service-ca                                    configmap-cabundle-injector-8498544d7-25qn6                         1/1       Running     0          35m
      openshift-service-ca                                    service-serving-cert-signer-6445fc9c6-wqdqn                         1/1       Running     0          35m
      openshift-service-catalog-apiserver-operator            openshift-service-catalog-apiserver-operator-549f44668b-b5q2w       1/1       Running     0          32m
      openshift-service-catalog-controller-manager-operator   openshift-service-catalog-controller-manager-operator-b78cr2lnm     1/1       Running     0          31m
      ```

      When the current cluster version is `AVAILABLE`, the installation is complete.

## Telemetry access for OpenShift Container Platform {#cluster-telemetry_installing-gcp-user-infra}

To provide metrics about cluster health and the success of updates, the Telemetry service requires internet access. When connected, this service runs automatically by default and registers your cluster to [OpenShift Cluster Manager](https://console.redhat.com/openshift).

After you confirm that your [OpenShift Cluster Manager](https://console.redhat.com/openshift) inventory is correct, either maintained automatically by Telemetry or manually by using OpenShift Cluster Manager,use subscription watch to track your OpenShift Container Platform subscriptions at the account or multi-cluster level. For more information about subscription watch, see "Data Gathered and Used by Red Hat’s subscription services" in the *Additional resources* section.

**Additional resources**

- [About remote health monitoring](/docs/support/remote_health_monitoring/about-remote-health-monitoring#about-remote-health-monitoring)
- [Customizing your cluster](/docs/post_installation_configuration/cluster-tasks#available_cluster_customizations)
- [Remote health reporting](/docs/support/remote_health_monitoring/remote-health-reporting#remote-health-reporting)
- [Configuring Global Access for an Ingress Controller on Google Cloud](/docs/networking/networking_operators/ingress-operator#nw-ingress-controller-configuration-gcp-global-access_configuring-ingress)
