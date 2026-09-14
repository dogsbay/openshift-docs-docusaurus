---
title: Installing a cluster on Azure using ARM templates
sidebar_position: 3
---

# Installing a cluster on Azure using ARM templates {#installing-azure-user-infra}

<a id="installing-azure-user-infra"></a>

To install OpenShift Container Platform on Microsoft Azure with infrastructure that you provide, you can use Azure Resource Manager (ARM) templates to create required resources and complete the user-provisioned installation.

Several ARM templates are provided to assist in completing these steps or to help model your own. See "Azure Resource Manager templates overview".

:::warning

The steps for performing a user-provisioned infrastructure installation are provided as an example only. Installing a cluster with infrastructure you provide requires knowledge of the cloud provider and the installation process of OpenShift Container Platform. Several ARM templates are provided to assist in completing these steps or to help model your own. You are also free to create the required resources through other methods; the templates are just an example.

:::

## Prerequisites {#installation-azure-user-infra-prerequisites_installing-azure-user-infra}

Before you install a cluster on Azure by using Azure Resource Manager (ARM) templates, complete the following prerequisites.

- You reviewed details about the OpenShift Container Platform installation and update processes. See "OpenShift Container Platform installation and update".
- You read the documentation on selecting a cluster installation method and preparing it for users. See "Selecting a cluster installation method and preparing it for users".
- You configured an Azure account to host the cluster. See "Configuring an Azure account".
- You downloaded the Azure CLI and installed it on your computer. The following documentation was last tested using version `2.49.0` of the Azure CLI. Azure CLI commands might perform differently based on the version you use. See "Install the Azure CLI".
- If the cloud identity and access management (IAM) APIs are not accessible in your environment, or if you do not want to store an administrator-level credential secret in the `kube-system` namespace, you reviewed alternatives to storing administrator-level secrets in the kube-system project. See "Alternatives to storing administrator-level secrets in the kube-system project".
- If you use a firewall and plan to use the Telemetry service, you configured the firewall to allow the sites that your cluster requires access to. See "Configuring the firewall to allow required sites".
  :::note

  Be sure to also review this site list if you are configuring a proxy.

  :::

**Additional resources**

- [Azure Resource Manager templates overview (Azure documentation)](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/overview)
- [OpenShift Container Platform installation and update](/docs/architecture/architecture-installation#architecture-installation)
- [Selecting a cluster installation method and preparing it for users](/docs/installing/overview/installing-preparing#installing-preparing)
- [Configuring an Azure account](/docs/installing/installing_azure/installing-azure-account#installing-azure-account)
- [Install the Azure CLI (Azure documentation)](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
- [Alternatives to storing administrator-level secrets in the kube-system project](/docs/installing/installing_azure/ipi/installing-azure-customizations#installing-azure-manual-modes_installing-azure-customizations)
- [Configuring the firewall to allow required sites](/docs/installing/install_config/configuring-firewall#configuring-firewall-module_configuring-firewall)

## Internet access for OpenShift Container Platform {#cluster-entitlements_installing-azure-user-infra}

In OpenShift Container Platform 4.22, you require access to the internet to install your cluster.

You must have internet access to perform the following actions:

- Access Red Hat Hybrid Cloud Console to download the installation program and perform subscription management. If the cluster has internet access and you do not disable Telemetry, that service automatically entitles your cluster.
- Access Quay.io to obtain the packages that are required to install your cluster.
- Obtain the packages that are required to perform cluster updates.

:::warning

If your cluster cannot have direct internet access, you can perform a restricted network installation on some types of infrastructure that you provision. During that process, you download the required content and use it to populate a mirror registry with the installation packages. With some installation types, the environment that you install your cluster in will not require internet access. Before you update the cluster, you update the content of the mirror registry.

:::

## Configuring your Azure project {#installation-azure-config-project-overview_installing-azure-user-infra}

Before you install OpenShift Container Platform on Microsoft Azure, you must configure an Azure project that can host the cluster and its resources.

:::warning

All Azure resources that are available through public endpoints are subject to resource name restrictions, and you cannot create resources that use certain terms. For a list of terms that Azure restricts, see "Resolve reserved resource name errors".

:::

**Additional resources**

- [Resolve reserved resource name errors (Azure documentation)](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-reserved-resource-name)

### Azure account limits {#installation-azure-limits_installing-azure-user-infra}

The OpenShift Container Platform cluster uses a number of Microsoft Azure components. Default subscription and service limits, quotas, and constraints can affect your ability to install OpenShift Container Platform clusters.

For more information on Azure subscription and service limits, see "Azure subscription and service limits, quotas, and constraints".

:::warning

Default limits vary by offer category types, such as Free Trial and Pay-As-You-Go, and by series, such as Dv2, F, and G. For example, the default for Enterprise Agreement subscriptions is 350 cores.

Check the limits for your subscription type and if necessary, increase quota limits for your account before you install a default cluster on Azure.

:::

The following table summarizes the Azure components whose limits can impact your ability to install and run OpenShift Container Platform clusters.

<table>
<thead>
<tr>
  <th>Component</th>
  <th>Number of components required by default</th>
  <th>Default Azure limit</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>vCPU</td>
  <td>40</td>
  <td>20 per region</td>
  <td>A default cluster requires 40 vCPUs, so you must increase the account limit.<br><br>By default, each cluster creates the following instances:<br><br><ul><li>One bootstrap machine, which is removed after installation</li><li>Three control plane machines</li><li>Three compute machines</li></ul>   Because the bootstrap machine uses <code>Standard_D4s_v3</code> machines, which use 4 vCPUs, the control plane machines use <code>Standard_D8s_v3</code> virtual machines, which use 8 vCPUs, and the worker machines use <code>Standard_D4s_v3</code> virtual machines, which use 4 vCPUs, a default cluster requires 40 vCPUs. The bootstrap node VM, which uses 4 vCPUs, is used only during installation.  <br><br>To deploy more worker nodes, enable autoscaling, deploy large workloads, or use a different instance type, you must further increase the vCPU limit for your account to ensure that your cluster can deploy the machines that you require.</td>
</tr>
<tr>
  <td>OS Disk</td>
  <td>7</td>
  <td></td>
  <td>Each cluster machine must have a minimum of 100 GB of storage and 300 IOPS.<div class="alert alert--secondary"><strong>Note</strong><p>Faster storage is recommended for production clusters and clusters with intensive workloads. For more information about optimizing storage for performance, see the page titled "Optimizing storage" in the "Scalability and performance" section.</p></div></td>
</tr>
<tr>
  <td>VNet</td>
  <td>1</td>
  <td>1000 per region</td>
  <td>Each default cluster requires one Virtual Network (VNet), which contains two subnets.</td>
</tr>
<tr>
  <td>Network interfaces</td>
  <td>7</td>
  <td>65,536 per region</td>
  <td>Each default cluster requires seven network interfaces. If you create more machines or your deployed workloads create load balancers, your cluster uses more network interfaces.</td>
</tr>
<tr>
  <td>Network security groups</td>
  <td>2</td>
  <td>5000</td>
  <td>Each cluster creates network security groups for each subnet in the VNet. The default cluster creates network security groups for the control plane and for the compute node subnets:<br><br><dl><dt><code>controlplane</code></dt><dd>Allows the control plane machines to be reached on port 6443 from anywhere</dd><dt><code>node</code></dt><dd>Allows worker nodes to be reached from the internet on ports 80 and 443</dd></dl></td>
</tr>
<tr>
  <td>Network load balancers</td>
  <td>3</td>
  <td>1000 per region</td>
  <td>Each cluster creates the following <a href="https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-overview">load balancers</a>:<br><br><dl><dt><code>default</code></dt><dd>Public IP address that load balances requests to ports 80 and 443 across worker machines</dd><dt><code>internal</code></dt><dd>Private IP address that load balances requests to ports 6443 and 22623 across control plane machines</dd><dt><code>external</code></dt><dd>Public IP address that load balances requests to port 6443 across control plane machines</dd></dl>If your applications create more Kubernetes <code>LoadBalancer</code> service objects, your cluster uses more load balancers.</td>
</tr>
<tr>
  <td>Public IP addresses</td>
  <td>3</td>
  <td></td>
  <td>Each of the two public load balancers uses a public IP address. The bootstrap machine also uses a public IP address so that you can SSH into the machine to troubleshoot issues during installation. The IP address for the bootstrap node is used only during installation.</td>
</tr>
<tr>
  <td>Private IP addresses</td>
  <td>7</td>
  <td></td>
  <td>The internal load balancer, each of the three control plane machines, and each of the three worker machines each use a private IP address.</td>
</tr>
<tr>
  <td>Spot VM vCPUs (optional)</td>
  <td>0<br><br>If you configure spot VMs, your cluster must have two spot VM vCPUs for every compute node.</td>
  <td>20 per region</td>
  <td>This is an optional component. To use spot VMs, you must increase the Azure default limit to at least twice the number of compute nodes in your cluster.<div class="alert alert--secondary"><strong>Note</strong><p>Using spot VMs for control plane nodes is not recommended.</p></div></td>
</tr>
</tbody>
</table>

To increase an account limit, file a support request on the Azure portal. For more information, see [Request a quota limit increase for Azure Deployment Environments resources](https://learn.microsoft.com/en-us/azure/deployment-environments/how-to-request-quota-increase).

**Additional resources**

- [Optimizing storage](/docs/scalability_and_performance/optimization/optimizing-storage#optimizing-storage)
- [Azure subscription and service limits, quotas, and constraints (Azure documentation)](https://docs.microsoft.com/en-us/azure/azure-subscription-service-limits)

### Configuring a public DNS zone in Azure {#installation-azure-network-config_installing-azure-user-infra}

To install OpenShift Container Platform, the Microsoft Azure account you use must have a dedicated public hosted DNS zone in your account that is authoritative for the domain. This zone provides cluster DNS resolution and name lookup for external connections to the cluster.

**Procedure**

1. Identify your domain, or subdomain, and registrar. You can transfer an existing domain and registrar or obtain a new one through Azure or another source.
   - To purchase a new domain through Azure, see [Buy a custom domain name for Azure App Service](https://docs.microsoft.com/en-us/azure/app-service/manage-custom-dns-buy-domain).
   - If you are using an existing domain and registrar, migrate its DNS to Azure. For more information, see [Migrate an active DNS name to Azure App Service](https://docs.microsoft.com/en-us/azure/app-service/manage-custom-dns-migrate-domain) in the Azure documentation.
2. Configure DNS for your domain, which includes creating a public hosted zone for your domain or subdomain, extracting the new authoritative name servers, and updating the registrar records for the name servers that your domain uses. For more information, see [Tutorial: Host your domain in Azure DNS](https://docs.microsoft.com/en-us/azure/dns/dns-delegate-domain-azure-dns).
   Use an appropriate root domain, such as `openshiftcorp.com`, or subdomain, such as `clusters.openshiftcorp.com`.
3. If you use a subdomain, follow your organization’s procedures to add its delegation records to the parent domain.

**Additional resources**

- [Example for creating DNS zones](/docs/installing/installing_azure/upi/installing-azure-user-infra#installation-azure-create-dns-zones_installing-azure-user-infra)

### Certificate signing requests management {#csr-management_installing-azure-user-infra}

On user-provisioned infrastructure, you must implement a mechanism for approving cluster certificate signing requests (CSRs) after installation when your cluster has limited access to automatic machine management.

The `kube-controller-manager` only approves the kubelet client CSRs. The `machine-approver` cannot guarantee the validity of a serving certificate that kubelet credentials request because it cannot confirm that the correct machine issued the request. You must find and implement a method of verifying the validity of the kubelet serving certificate requests and approving them.

### Recording the subscription and tenant IDs {#installation-azure-subscription-tenant-id_installing-azure-user-infra}

To record the subscription and tenant IDs that the installation program requires for your Azure account, you can use the Azure CLI.

**Prerequisites**

- You have installed or updated the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-yum?view=azure-cli-latest).

**Procedure**

1. Log in to the Azure CLI by running the following command:
   ```terminal
   $ az login
   ```
2. Ensure that you are using the right subscription:
   1. View a list of available subscriptions by running the following command:
      ```terminal
      $ az account list --refresh
      ```

      ```terminal title="Example output"
      [
        {
          "cloudName": "AzureCloud",
          "id": "8xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
          "isDefault": true,
          "name": "Subscription Name 1",
          "state": "Enabled",
          "tenantId": "6xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
          "user": {
            "name": "you@example.com",
            "type": "user"
          }
        },
        {
          "cloudName": "AzureCloud",
          "id": "9xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
          "isDefault": false,
          "name": "Subscription Name 2",
          "state": "Enabled",
          "tenantId": "7xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
          "user": {
            "name": "you2@example.com",
            "type": "user"
          }
        }
      ]
      ```
   2. View the details of the active account, and confirm that this is the subscription you want to use, by running the following command:
      ```terminal
      $ az account show
      ```

      ```terminal title="Example output"
      {
        "environmentName": "AzureCloud",
        "id": "8xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "isDefault": true,
        "name": "Subscription Name 1",
        "state": "Enabled",
        "tenantId": "6xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "user": {
          "name": "you@example.com",
          "type": "user"
        }
      }
      ```
3. If you are not using the right subscription:
   1. Change the active subscription by running the following command:
      ```terminal
      $ az account set -s <subscription_id>
      ```
   2. Verify that you are using the subscription you need by running the following command:
      ```terminal
      $ az account show
      ```

      ```terminal title="Example output"
      {
        "environmentName": "AzureCloud",
        "id": "9xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "isDefault": true,
        "name": "Subscription Name 2",
        "state": "Enabled",
        "tenantId": "7xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "user": {
          "name": "you2@example.com",
          "type": "user"
        }
      }
      ```
4. Record the `id` and `tenantId` parameter values from the output. You require these values to install an OpenShift Container Platform cluster.

### Supported identities to access Azure resources {#installation-azure-identities_installing-azure-user-infra}

An OpenShift Container Platform cluster requires an Azure identity to create and manage Azure resources. You need a service principal, a system-assigned managed identity, or a user-assigned managed identity to complete the installation.

- A service principal
- A system-assigned managed identity
- A user-assigned managed identity

For more information on Azure identities, see "Managed identity types".

### Required Azure permissions for user-provisioned infrastructure {#minimum-required-permissions-upi-azure_installing-azure-user-infra}

The installation program requires access to an Azure service principal or managed identity with the necessary permissions to deploy the cluster and to maintain its daily operation. These permissions must be granted to the Azure subscription that is associated with the identity.

The following options are available to you:

- You can assign the identity the `Contributor` and `User Access Administrator` roles. Assigning these roles is the quickest way to grant all of the required permissions.
  For more information about assigning roles, see the Azure documentation for [managing access to Azure resources using the Azure portal](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-portal).
- If your organization’s security policies require a more restrictive set of permissions, you can create a [custom role](https://learn.microsoft.com/en-us/azure/role-based-access-control/custom-roles) with the necessary permissions.

The following permissions are required for creating an OpenShift Container Platform cluster on Microsoft Azure.

<details>
<summary>Required permissions for creating authorization resources</summary>

- `Microsoft.Authorization/policies/audit/action`
- `Microsoft.Authorization/policies/auditIfNotExists/action`
- `Microsoft.Authorization/roleAssignments/read`
- `Microsoft.Authorization/roleAssignments/write`

</details>

<details>
<summary>Required permissions for creating compute resources</summary>

- `Microsoft.Compute/images/read`
- `Microsoft.Compute/images/write`
- `Microsoft.Compute/images/delete`
- `Microsoft.Compute/availabilitySets/read`
- `Microsoft.Compute/disks/beginGetAccess/action`
- `Microsoft.Compute/disks/delete`
- `Microsoft.Compute/disks/read`
- `Microsoft.Compute/disks/write`
- `Microsoft.Compute/galleries/images/read`
- `Microsoft.Compute/galleries/images/versions/read`
- `Microsoft.Compute/galleries/images/versions/write`
- `Microsoft.Compute/galleries/images/write`
- `Microsoft.Compute/galleries/read`
- `Microsoft.Compute/galleries/write`
- `Microsoft.Compute/snapshots/read`
- `Microsoft.Compute/snapshots/write`
- `Microsoft.Compute/snapshots/delete`
- `Microsoft.Compute/virtualMachines/delete`
- `Microsoft.Compute/virtualMachines/powerOff/action`
- `Microsoft.Compute/virtualMachines/read`
- `Microsoft.Compute/virtualMachines/write`
- `Microsoft.Compute/virtualMachines/deallocate/action`

</details>

<details>
<summary>Required permissions for creating identity management resources</summary>

- `Microsoft.ManagedIdentity/userAssignedIdentities/assign/action`
- `Microsoft.ManagedIdentity/userAssignedIdentities/read`
- `Microsoft.ManagedIdentity/userAssignedIdentities/write`

</details>

<details>
<summary>Required permissions for creating network resources</summary>

- `Microsoft.Network/dnsZones/A/write`
- `Microsoft.Network/dnsZones/CNAME/write`
- `Microsoft.Network/dnszones/CNAME/read`
- `Microsoft.Network/dnszones/read`
- `Microsoft.Network/loadBalancers/backendAddressPools/join/action`
- `Microsoft.Network/loadBalancers/backendAddressPools/read`
- `Microsoft.Network/loadBalancers/backendAddressPools/write`
- `Microsoft.Network/loadBalancers/read`
- `Microsoft.Network/loadBalancers/write`
- `Microsoft.Network/networkInterfaces/delete`
- `Microsoft.Network/networkInterfaces/join/action`
- `Microsoft.Network/networkInterfaces/read`
- `Microsoft.Network/networkInterfaces/write`
- `Microsoft.Network/networkSecurityGroups/join/action`
- `Microsoft.Network/networkSecurityGroups/read`
- `Microsoft.Network/networkSecurityGroups/securityRules/delete`
- `Microsoft.Network/networkSecurityGroups/securityRules/read`
- `Microsoft.Network/networkSecurityGroups/securityRules/write`
- `Microsoft.Network/networkSecurityGroups/write`
- `Microsoft.Network/privateDnsZones/A/read`
- `Microsoft.Network/privateDnsZones/A/write`
- `Microsoft.Network/privateDnsZones/A/delete`
- `Microsoft.Network/privateDnsZones/SOA/read`
- `Microsoft.Network/privateDnsZones/read`
- `Microsoft.Network/privateDnsZones/virtualNetworkLinks/read`
- `Microsoft.Network/privateDnsZones/virtualNetworkLinks/write`
- `Microsoft.Network/privateDnsZones/write`
- `Microsoft.Network/publicIPAddresses/delete`
- `Microsoft.Network/publicIPAddresses/join/action`
- `Microsoft.Network/publicIPAddresses/read`
- `Microsoft.Network/publicIPAddresses/write`
- `Microsoft.Network/virtualNetworks/join/action`
- `Microsoft.Network/virtualNetworks/read`
- `Microsoft.Network/virtualNetworks/subnets/join/action`
- `Microsoft.Network/virtualNetworks/subnets/read`
- `Microsoft.Network/virtualNetworks/subnets/write`
- `Microsoft.Network/virtualNetworks/write`

</details>

<details>
<summary>Required permissions for checking the health of resources</summary>

- `Microsoft.Resourcehealth/healthevent/Activated/action`
- `Microsoft.Resourcehealth/healthevent/InProgress/action`
- `Microsoft.Resourcehealth/healthevent/Pending/action`
- `Microsoft.Resourcehealth/healthevent/Resolved/action`
- `Microsoft.Resourcehealth/healthevent/Updated/action`

</details>

<details>
<summary>Required permissions for creating a resource group</summary>

- `Microsoft.Resources/subscriptions/resourceGroups/read`
- `Microsoft.Resources/subscriptions/resourcegroups/write`

</details>

<details>
<summary>Required permissions for creating resource tags</summary>

- `Microsoft.Resources/tags/write`

</details>

<details>
<summary>Required permissions for creating storage resources</summary>

- `Microsoft.Storage/storageAccounts/blobServices/read`
- `Microsoft.Storage/storageAccounts/blobServices/containers/write`
- `Microsoft.Storage/storageAccounts/fileServices/read`
- `Microsoft.Storage/storageAccounts/fileServices/shares/read`
- `Microsoft.Storage/storageAccounts/fileServices/shares/write`
- `Microsoft.Storage/storageAccounts/fileServices/shares/delete`
- `Microsoft.Storage/storageAccounts/listKeys/action`
- `Microsoft.Storage/storageAccounts/read`
- `Microsoft.Storage/storageAccounts/write`

</details>

<details>
<summary>Required permissions for creating deployments</summary>

- `Microsoft.Resources/deployments/read`
- `Microsoft.Resources/deployments/write`
- `Microsoft.Resources/deployments/validate/action`
- `Microsoft.Resources/deployments/operationstatuses/read`

</details>

<details>
<summary>Optional permissions for creating compute resources</summary>

- `Microsoft.Compute/availabilitySets/delete`
- `Microsoft.Compute/availabilitySets/write`

</details>

<details>
<summary>Optional permissions for creating marketplace virtual machine resources</summary>

- `Microsoft.MarketplaceOrdering/offertypes/publishers/offers/plans/agreements/read`
- `Microsoft.MarketplaceOrdering/offertypes/publishers/offers/plans/agreements/write`

</details>

<details>
<summary>Optional permissions for enabling user-managed encryption</summary>

- `Microsoft.Compute/diskEncryptionSets/read`
- `Microsoft.Compute/diskEncryptionSets/write`
- `Microsoft.Compute/diskEncryptionSets/delete`
- `Microsoft.KeyVault/vaults/read`
- `Microsoft.KeyVault/vaults/write`
- `Microsoft.KeyVault/vaults/delete`
- `Microsoft.KeyVault/vaults/deploy/action`
- `Microsoft.KeyVault/vaults/keys/read`
- `Microsoft.KeyVault/vaults/keys/write`
- `Microsoft.Features/providers/features/register/action`

</details>

The following permissions are required for deleting an OpenShift Container Platform cluster on Microsoft Azure.

<details>
<summary>Required permissions for deleting authorization resources</summary>

- `Microsoft.Authorization/roleAssignments/delete`

</details>

<details>
<summary>Required permissions for deleting compute resources</summary>

- `Microsoft.Compute/disks/delete`
- `Microsoft.Compute/galleries/delete`
- `Microsoft.Compute/galleries/images/delete`
- `Microsoft.Compute/galleries/images/versions/delete`
- `Microsoft.Compute/virtualMachines/delete`
- `Microsoft.Compute/images/delete`

</details>

<details>
<summary>Required permissions for deleting identity management resources</summary>

- `Microsoft.ManagedIdentity/userAssignedIdentities/delete`

</details>

<details>
<summary>Required permissions for deleting network resources</summary>

- `Microsoft.Network/dnszones/read`
- `Microsoft.Network/dnsZones/A/read`
- `Microsoft.Network/dnsZones/A/delete`
- `Microsoft.Network/dnsZones/CNAME/read`
- `Microsoft.Network/dnsZones/CNAME/delete`
- `Microsoft.Network/loadBalancers/delete`
- `Microsoft.Network/networkInterfaces/delete`
- `Microsoft.Network/networkSecurityGroups/delete`
- `Microsoft.Network/privateDnsZones/read`
- `Microsoft.Network/privateDnsZones/A/read`
- `Microsoft.Network/privateDnsZones/delete`
- `Microsoft.Network/privateDnsZones/virtualNetworkLinks/delete`
- `Microsoft.Network/publicIPAddresses/delete`
- `Microsoft.Network/virtualNetworks/delete`

</details>

<details>
<summary>Required permissions for checking the health of resources</summary>

- `Microsoft.Resourcehealth/healthevent/Activated/action`
- `Microsoft.Resourcehealth/healthevent/Resolved/action`
- `Microsoft.Resourcehealth/healthevent/Updated/action`

</details>

<details>
<summary>Required permissions for deleting a resource group</summary>

- `Microsoft.Resources/subscriptions/resourcegroups/delete`

</details>

<details>
<summary>Required permissions for deleting storage resources</summary>

- `Microsoft.Storage/storageAccounts/delete`
- `Microsoft.Storage/storageAccounts/listKeys/action`

</details>

:::note

To install OpenShift Container Platform on Azure, you must scope the permissions related to resource group creation to your subscription. After the resource group is created, you can scope the rest of the permissions to the created resource group. If the public DNS zone is present in a different resource group, then the network DNS zone related permissions must always be applied to your subscription.

You can scope all the permissions to your subscription when deleting an OpenShift Container Platform cluster.

:::

### Using Azure managed identities {#installation-using-azure-managed-identities_installing-azure-user-infra}

To provide the identity that the installation program requires on Azure, you can use a system-assigned or user-assigned managed identity.

If you are unable to use a managed identity, you can use a service principal.

**Procedure**

1. If you are using a system-assigned managed identity, enable it on the virtual machine that you will run the installation program from.
2. If you are using a user-assigned managed identity:
   1. Assign it to the virtual machine that you will run the installation program from.
   2. Record its client ID. You require this value when installing the cluster.
3. Verify that the required permissions are assigned to the managed identity.

### Creating a service principal {#installation-creating-azure-service-principal_installing-azure-user-infra}

To provide the identity that the installation program requires on Azure, you can create a service principal.

If you are unable to use a service principal, you can use a managed identity.

**Prerequisites**

- You have installed or updated the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-yum?view=azure-cli-latest).
- You have an Azure subscription ID.
- If you are not assigning the `Contributor` and `User Administrator Access` roles to the service principal, you have created a custom role with the required Azure permissions.

**Procedure**

1. Create the service principal for your account by running the following command:
   ```terminal
   $ az ad sp create-for-rbac --role <role_name> \
        --name <service_principal> \
        --scopes /subscriptions/<subscription_id>
   ```

   where:

   <dl>
   <dt><code>&lt;role_name&gt;</code></dt>
   <dd>Specifies the role name. You can use the <code>Contributor</code> role, or you can specify a custom role which contains the necessary permissions.</dd>
   <dt><code>&lt;service_principal&gt;</code></dt>
   <dd>Specifies the service principal name.</dd>
   <dt><code>&lt;subscription_id&gt;</code></dt>
   <dd>Specifies the subscription ID.</dd>
   </dl>

   ```terminal title="Example output"
   Creating 'Contributor' role assignment under scope '/subscriptions/<subscription_id>'
   The output includes credentials that you must protect. Be sure that you do not
   include these credentials in your code or check the credentials into your source
   control. For more information, see https://aka.ms/azadsp-cli
   {
     "appId": "axxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
     "displayName": <service_principal>",
     "password": "00000000-0000-0000-0000-000000000000",
     "tenantId": "8xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
   }
   ```

Record the values of the `appId` and `password` parameters from the output. You require these values when installing the cluster.

1. If you assigned the `Contributor` role to your service principal, assign the `User Administrator Access` role by running the following command:
   ```terminal
   $ az role assignment create --role "User Access Administrator" \
     --assignee-object-id $(az ad sp show --id <appId> --query id -o tsv) \
     --scope /subscriptions/<subscription_id>
   ```

   where:

   <dl>
   <dt><code>&lt;appId&gt;</code></dt>
   <dd>Specifies the <code>appId</code> parameter value for your service principal.</dd>
   <dt><code>&lt;subscription_id&gt;</code></dt>
   <dd>Specifies the subscription ID.</dd>
   </dl>

**Additional resources**

- [About the Cloud Credential Operator](/docs/authentication/managing_cloud_provider_credentials/about-cloud-credential-operator#about-cloud-credential-operator-modes)

### Supported Azure regions {#installation-azure-regions_installing-azure-user-infra}

Based on your subscription, the installation program dynamically generates the list of available Microsoft Azure public regions.

#### Supported Azure public regions {#_supported_azure_short_public_regions}

- `australiacentral` (Australia Central)
- `australiaeast` (Australia East)
- `australiasoutheast` (Australia South East)
- `austriaeast` (Austria East)
- `belgiumcentral` (Belgium Central)
- `brazilsouth` (Brazil South)
- `canadacentral` (Canada Central)
- `canadaeast` (Canada East)
- `centralindia` (Central India)
- `centralus` (Central US)
- `chilecentral` (Chile Central)
- `eastasia` (East Asia)
- `eastus` (East US)
- `eastus2` (East US 2)
- `francecentral` (France Central)
- `germanywestcentral` (Germany West Central)
- `indonesiacentral` (Indonesia Central)
- `israelcentral` (Israel Central)
- `italynorth` (Italy North)
- `japaneast` (Japan East)
- `japanwest` (Japan West)
- `koreacentral` (Korea Central)
- `koreasouth` (Korea South)
- `malaysiawest` (Malaysia West)
- `mexicocentral` (Mexico Central)
- `newzealandnorth` (New Zealand North)
- `northcentralus` (North Central US)
- `northeurope` (North Europe)
- `norwayeast` (Norway East)
- `polandcentral` (Poland Central)
- `qatarcentral` (Qatar Central)
- `southafricanorth` (South Africa North)
- `southcentralus` (South Central US)
- `southeastasia` (Southeast Asia)
- `southindia` (South India)
- `spaincentral` (Spain Central)
- `swedencentral` (Sweden Central)
- `switzerlandnorth` (Switzerland North)
- `uaenorth` (UAE North)
- `uksouth` (UK South)
- `ukwest` (UK West)
- `westcentralus` (West Central US)
- `westeurope` (West Europe)
- `westindia` (West India)
- `westus` (West US)
- `westus2` (West US 2)
- `westus3` (West US 3)

#### Supported Azure Government regions {#_supported_azure_short_government_regions}

Support for the following Microsoft Azure Government (MAG) regions was added in OpenShift Container Platform version 4.6:

- `usgovtexas` (US Gov Texas)
- `usgovvirginia` (US Gov Virginia)

You can reference all available MAG regions in the [Azure documentation](https://azure.microsoft.com/en-us/global-infrastructure/geographies/#geographies). Other provided MAG regions are expected to work with OpenShift Container Platform, but have not been tested.

## Requirements for a cluster with user-provisioned infrastructure {#installation-requirements-user-infra_installing-azure-user-infra}

For a cluster that uses user-provisioned infrastructure, you must deploy all required machines.

### Required machines for cluster installation {#installation-machine-requirements_installing-azure-user-infra}

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

### Minimum resource requirements for cluster installation {#installation-minimum-resource-requirements_installing-azure-user-infra}

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

:::warning

You must use Azure virtual machines that have the `premiumIO` parameter set to `true`.

:::

If an instance type for your platform meets the minimum requirements for cluster machines, it is supported to use in OpenShift Container Platform.

**Additional resources**

- [Optimizing storage](/docs/scalability_and_performance/optimization/optimizing-storage#optimizing-storage)

### Tested instance types for Azure {#installation-azure-tested-machine-types_installing-azure-user-infra}

There are several Microsoft Azure instance types tested with OpenShift Container Platform. Choose a listed instance type when you install a cluster on 64-bit x86 infrastructure.

See the following machine types based on 64-bit x86 architecture:

##### General Purpose

| Azure VM Series | Family Name |
| --- | --- |
| Basv2-series | `standardBasv2Family` |
| BS-series | `standardBSFamily` |
| Bsv2-series | `standardBsv2Family` |
| Dadsv5-series | `standardDADSv5Family` |
| Dadsv6-series | `standardDadv6Family` |
| Daldsv6-series | `standardDaldv6Family` |
| Dalsv6-series | `standardDalv6Family` |
| Dasv4-series | `standardDASv4Family` |
| Dasv5-series | `standardDASv5Family` |
| Dasv6-series | `standardDav6Family` |
| DCas_cc_v5-series | `standardDCACCV5Family` |
| DCads_cc_v5-series | `standardDCADCCV5Family` |
| DCadsv5-series | `standardDCADSv5Family` |
| DCasv5-series | `standardDCASv5Family` |
| DCsv2-series | `standardDCSv2Family` |
| DCsv3-series | `standardDCSv3Family` |
| DCdsv3-series | `standardDDCSv3Family` |
| DCedsv5-series | `standardDCEDV5Family` |
| DCesv5-series | `standardDCEV5Family` |
| Ddsv4-series | `standardDDSv4Family` |
| Ddsv5-series | `standardDDSv5Family` |
| Ddsv6-series | `StandardDdsv6Family` |
| Dldsv5-series | `standardDLDSv5Family` |
| Dldsv6-series | `StandardDldsv6Family` |
| Dlsv5-series | `standardDLSv5Family` |
| Dlsv6-series | `StandardDlsv6Family` |
| DS-series | `standardDSFamily` |
| Dsv2-series | `standardDSv2Family` |
| Dsv3-series | `standardDSv3Family` |
| Dsv4-series | `standardDSv4Family` |
| Dsv5-series | `standardDSv5Family` |
| Dsv6-series | `StandardDsv6Family` |

##### Memory Optimized

| Azure VM Series | Family Name |
| --- | --- |
| Eadsv5-series | `standardEADSv5Family`, `standardEIADSv5Family` |
| Eadsv6-series | `standardEadv6Family` |
| Easv4-series | `standardEASv4Family`, `standardEIASv4Family` |
| Easv5-series | `standardEASv5Family`, `standardEIASv5Family` |
| Easv6-series | `standardEav6Family` |
| Ebdsv5-series | `standardEBDSv5Family`, `standardEIBDSv5Family` |
| Ebsv5-series | `standardEBSv5Family`, `standardEIBSv5Family` |
| ECas_cc_v5-series | `standardECACCV5Family` |
| ECads_cc_v5-series | `standardECADCCV5Family` |
| ECadsv5-series | `standardECADSv5Family` |
| ECasv5-series | `standardECASv5Family` |
| ECedsv5-series | `standardECEDV5Family` |
| ECesv5-series | `standardECEV5Family` |
| Edsv4-series | `standardEDSv4Family` |
| Edsv5-series | `standardEDSv5Family`, `standardEIDSv5Family` |
| Edsv6-series | `StandardEdsv6Family` |
| Esv3-series | `standardESv3Family`, `standardEISv3Family` |
| Esv4-series | `standardESv4Family`, `standardXEISv4Family` |
| Esv5-series | `standardESv5Family`, `standardEISv5Family` |
| Esv6-series | `StandardEsv6Family` |
| M-series | `standardMSFamily` |
| Mbdsv3-series | `StandardMBDSMediumMemoryv3Family` |
| Mbsv3-series | `StandardMBSMediumMemoryv3Family` |
| Mdsv3 High Memory-series | `standardMDSHighMemoryv3Family`, `standardMIDSHighMemoryv3Family` |
| Mdsv2 Medium Memory-series | `standardMDSMediumMemoryv2Family`, `standardMIDSMediumMemoryv2Family` |
| Mdsv3 Medium Memory-series | `standardMDSMediumMemoryv3Family` |
| Msv3 High Memory-series | `standardMISHighMemoryv3Family`, `standardMSHighMemoryv3Family` |
| Msv2 Medium Memory-series | `standardMISMediumMemoryv2Family`, `standardMSMediumMemoryv2Family` |
| Msv3 Medium Memory-series | `standardMSMediumMemoryv3Family` |

##### Compute Optimized

| Azure VM Series | Family Name |
| --- | --- |
| Falsv6-series | `StandardFalsv6Family` |
| Famsv6-series | `StandardFamsv6Family` |
| Fasv6-series | `StandardFasv6Family` |
| FS-series | `standardFSFamily` |
| Fsv2-series | `standardFSv2Family` |
| FXmdsv2-series | `StandardFXmdsv2Family` |
| FX-series | `standardFXMDVSFamily` |
| FXmsv2-series | `StandardFXmsv2Family` |

##### Storage Optimized

| Azure VM Series | Family Name |
| --- | --- |
| GS-series | `standardGSFamily` |
| Laosv4-series | `standardLaosv4Family` |
| Lasv3-series | `standardLASv3Family` |
| Lasv4-series | `standardLasv4Family` |
| Ls-series | `standardLSFamily` |
| Lsv2-series | `standardLSv2Family` |
| Lsv3-series | `standardLSv3Family` |
| Lsv4-series | `standardLsv4Family` |

##### GPU Accelerated

| Azure VM Series | Family Name |
| --- | --- |
| NC_A100_v4-series | `StandardNCADSA100v4Family` |
| NCads_H100_v5-series | `StandardNCadsH100v5Family` |
| NCCads_H100_v5-series | `StandardNCCads2023Family` |
| NCasT4_v3-series | `Standard NCASv3_T4 Family` |
| NCv3-series | `standardNCSv3Family` |
| NDasrA100_v4-series | `Standard NDASv4_A100 Family` |
| ND-H200-v5-series | `standardNDISRH200V5Family` |
| ND-H100-v5-series | `standardNDSH100v5Family` |
| NDv2-series | `standardNDSv2Family` |
| NGads_V620-series | `StandardNGADSV620v1Family` |
| NVadsA10_v5-series | `StandardNVADSA10v5Family` |
| NVads V710 v5-series | `StandardNVadsV710v5Family` |
| NVv3-series | `standardNVSv3Family` |

##### FPGA Accelerated

| Azure VM Series | Family Name |
| --- | --- |
| NPS-series | `standardNPSFamily` |

##### High Performance Compute

| Azure VM Series | Family Name |
| --- | --- |
| HBv2-series | `standardHBrsv2Family` |
| HBv4-series | `standardHBv4Family` |
| HBv5-series | `standardHBv5Family` |
| HC-series | `standardHCSFamily` |
| HX-series | `standardHXFamily` |

### Tested instance types for Azure on 64-bit ARM infrastructures {#installation-azure-arm-tested-machine-types_installing-azure-user-infra}

There are several Microsoft Azure ARM64 instance types tested with OpenShift Container Platform. Choose a listed instance type when you install a cluster on 64-bit ARM infrastructure.

See the following machine types based on 64-bit ARM architecture:

##### General Purpose (ARM64)

| Azure VM Series | Family Name |
| --- | --- |
| Bpsv2-series | `standardBpsv2Family` |
| Dpdsv5-series | `standardDPDSv5Family` |
| Dpldsv5-series | `standardDPLDSv5Family` |
| Dplsv5-series | `standardDPLSv5Family` |
| Dpsv5-series | `standardDPSv5Family` |
| Dpdsv6-series | `StandardDpdsv6Family` |
| Dpldsv6-series | `StandardDpldsv6Family` |
| Dplsv6-series | `StandardDplsv6Family` |
| Dpsv6-series | `StandardDpsv6Family` |

##### Memory Optimized (ARM64)

| Azure VM Series | Family Name |
| --- | --- |
| Epdsv5-series | `standardEPDSv5Family` |
| Epsv5-series | `standardEPSv5Family` |
| Epdsv6-series | `StandardEpdsv6Family` |
| Epsv6-series | `StandardEpsv6Family` |

## Using the Azure Marketplace offering {#installation-azure-marketplace-subscribe_installing-azure-user-infra}

You can use the Azure Marketplace offering to deploy an OpenShift Container Platform cluster, which is billed on pay-per-use basis (hourly, per core) through Azure, while still being supported directly by Red Hat.

To deploy an OpenShift Container Platform cluster using the Azure Marketplace offering, you must first obtain the Azure Marketplace image. The installation program uses this image to deploy worker or control plane nodes. When obtaining your image, consider the following:

- While the images are the same, the Azure Marketplace publisher is different depending on your region. If you are located in North America, specify `redhat` as the publisher. If you are located in EMEA, specify `redhat-limited` as the publisher.
- The offer includes a `rh-ocp-worker` SKU and a `rh-ocp-worker-gen1` SKU. The `rh-ocp-worker` SKU represents a Hyper-V generation version 2 VM image. The default instance types used in OpenShift Container Platform are version 2 compatible. If you plan to use an instance type that is only version 1 compatible, use the image associated with the `rh-ocp-worker-gen1` SKU. The `rh-ocp-worker-gen1` SKU represents a Hyper-V version 1 VM image.

:::warning

Installing images with the Azure marketplace is not supported on clusters with 64-bit ARM instances.

You should only modify the RHCOS image for compute machines to use an Azure Marketplace image. Control plane machines and infrastructure nodes do not require an OpenShift Container Platform subscription and use the public RHCOS default image by default, which does not incur subscription costs on your Azure bill. Therefore, you should not modify the cluster default boot image or the control plane boot images. Applying the Azure Marketplace image to them will incur additional licensing costs that cannot be recovered.

:::

**Prerequisites**

- You have installed the Azure CLI client `(az)`.
- Your Azure account is entitled for the offer and you have logged into this account with the Azure CLI client.

**Procedure**

1. Display all of the available OpenShift Container Platform images by running one of the following commands:
   - North America:
     ```terminal
     $  az vm image list --all --offer rh-ocp-worker --publisher redhat -o table
     ```

     ```terminal title="Example output"
     Offer          Publisher       Sku                 Urn                                                             Version
     -------------  --------------  ------------------  --------------------------------------------------------------  -----------------
     rh-ocp-worker  RedHat          rh-ocp-worker       RedHat:rh-ocp-worker:rh-ocp-worker:4.17.2024100419              4.17.2024100419
     rh-ocp-worker  RedHat          rh-ocp-worker-gen1  RedHat:rh-ocp-worker:rh-ocp-worker-gen1:4.17.2024100419         4.17.2024100419
     ```
   - EMEA:
     ```terminal
     $  az vm image list --all --offer rh-ocp-worker --publisher redhat-limited -o table
     ```

     ```terminal title="Example output"
     Offer          Publisher       Sku                 Urn                                                                     Version
     -------------  --------------  ------------------  --------------------------------------------------------------          -----------------
     rh-ocp-worker  redhat-limited  rh-ocp-worker       redhat-limited:rh-ocp-worker:rh-ocp-worker:4.17.2024100419              4.17.2024100419
     rh-ocp-worker  redhat-limited  rh-ocp-worker-gen1  redhat-limited:rh-ocp-worker:rh-ocp-worker-gen1:4.17.2024100419         4.17.2024100419
     ```

   :::note

   Use the latest image that is available for compute and control plane nodes. If required, your VMs are automatically upgraded as part of the installation process.

   :::
2. Inspect the image for your offer by running one of the following commands:
   - North America:
     ```terminal
     $ az vm image show --urn redhat:rh-ocp-worker:rh-ocp-worker:<version>
     ```
   - EMEA:
     ```terminal
     $ az vm image show --urn redhat-limited:rh-ocp-worker:rh-ocp-worker:<version>
     ```
3. Review the terms of the offer by running one of the following commands:
   - North America:
     ```terminal
     $ az vm image terms show --urn redhat:rh-ocp-worker:rh-ocp-worker:<version>
     ```
   - EMEA:
     ```terminal
     $ az vm image terms show --urn redhat-limited:rh-ocp-worker:rh-ocp-worker:<version>
     ```
4. Accept the terms of the offering by running one of the following commands:
   - North America:
     ```terminal
     $ az vm image terms accept --urn redhat:rh-ocp-worker:rh-ocp-worker:<version>
     ```
   - EMEA:
     ```terminal
     $ az vm image terms accept --urn redhat-limited:rh-ocp-worker:rh-ocp-worker:<version>
     ```
5. Record the image details of your offer. If you use the Azure Resource Manager (ARM) template to deploy your compute nodes:
   1. Update `storageProfile.imageReference` by deleting the `id` parameter and adding the `offer`, `publisher`, `sku`, and `version` parameters by using the values from your offer.
   2. Specify a `plan` for the virtual machines (VMs).
      ```json title="Example 06_workers.json ARM template with an updated storageProfile.imageReference object and a specified plan"
      ...
        "plan" : {
          "name": "rh-ocp-worker",
          "product": "rh-ocp-worker",
          "publisher": "redhat"
        },
        "dependsOn" : [
          "[concat('Microsoft.Network/networkInterfaces/', concat(variables('vmNames')[copyIndex()], '-nic'))]"
        ],
        "properties" : {
      ...
        "storageProfile": {
          "imageReference": {
          "offer": "rh-ocp-worker",
          "publisher": "redhat",
          "sku": "rh-ocp-worker",
          "version": "413.92.2023101700"
          }
          ...
         }
      ...
        }
      ```

## Obtaining the installation program {#installation-obtaining-installer_installing-azure-user-infra}

Before you install OpenShift Container Platform, download the installation file on the host you are using for installation, so that installation assets exist for deployment in your environment.

**Prerequisites**

- You have a computer that runs Linux or macOS, with 500 MB of local disk space.

**Procedure**

1. Go to the [Cluster Type](https://console.redhat.com/openshift/install) page on the Red Hat Hybrid Cloud Console. If you have a Red Hat account, log in with your credentials. If you do not, create an account.
   :::tip

   You can also [download the binaries for a specific OpenShift Container Platform release](https://mirror.openshift.com/pub/openshift-v4/clients/ocp/).

   :::
2. Select your infrastructure provider from the **Run it yourself** section of the page.
3. Select your host operating system and architecture from the dropdown menus under **OpenShift Installer** and click **Download Installer**.
4. Place the downloaded file in the directory where you want to store the installation configuration files.
   :::warning

   - The installation program creates several files on the computer that you use to install your cluster. You must keep the installation program and the files that the installation program creates after you finish installing the cluster. Both of the files are required to delete the cluster.
   - Deleting the files created by the installation program does not remove your cluster, even if the cluster failed during installation. To remove your cluster, complete the OpenShift Container Platform uninstallation procedures for your specific cloud provider.

   :::
5. Extract the installation program. For example, on a computer that uses a Linux operating system, run the following command:
   ```terminal
   $ tar -xvf openshift-install-linux.tar.gz
   ```
6. Download your installation [pull secret from Red Hat OpenShift Cluster Manager](https://console.redhat.com/openshift/install/pull-secret). This pull secret allows you to authenticate with the services that are provided by the included authorities, including Quay.io, which serves the container images for OpenShift Container Platform components.
   :::tip

   Alternatively, you can retrieve the installation program from the [Red Hat Customer Portal](https://access.redhat.com/downloads/content/290/), where you can specify a version of the installation program to download. However, you must have an active subscription to access this page.

   :::

## Generating a key pair for cluster node SSH access {#ssh-agent-using_installing-azure-user-infra}

During an OpenShift Container Platform installation, you can provide an SSH public key to the installation program. The key is passed to the Red Hat Enterprise Linux CoreOS (RHCOS) nodes through their Ignition config files and is used to authenticate SSH access to the nodes. The key is added to the `~/.ssh/authorized_keys` list for the `core` user on each node, which enables password-less authentication.

The key is added to the `~/.ssh/authorized_keys` list for the `core` user on each node, which enables password-less authentication. After the key is passed to the nodes, you can use the key pair to SSH in to the RHCOS nodes as the user `core`. To access the nodes through SSH, the private key identity must be managed by SSH for your local user.

If you want to SSH in to your cluster nodes to perform installation debugging or disaster recovery, you must provide the SSH public key during the installation process. The `./openshift-install gather` command also requires the SSH public key to be in place on the cluster nodes.

:::warning

Do not skip this procedure in production environments, where disaster recovery and debugging is required.

:::

:::note

You must use a local key, not one that you configured with platform-specific approaches.

:::

**Procedure**

1. If you do not have an existing SSH key pair on your local machine to use for authentication onto your cluster nodes, create one. For example, on a computer that uses a Linux operating system, run the following command:
   ```terminal
   $ ssh-keygen -t ed25519 -N '' -f <path>/<file_name>
   ```

   Specifies the path and file name, such as `~/.ssh/id_ed25519`, of the new SSH key. If you have an existing key pair, ensure your public key is in the your `~/.ssh` directory.

   :::note

   If you plan to install an OpenShift Container Platform cluster that uses the RHEL cryptographic libraries that have been submitted to NIST for FIPS 140-2/140-3 Validation on only the `x86_64`, `ppc64le`, and `s390x` architectures, do not create a key that uses the `ed25519` algorithm. Instead, create a key that uses the `rsa` or `ecdsa` algorithm.

   :::
2. View the public SSH key:
   ```terminal
   $ cat <path>/<file_name>.pub
   ```

   For example, run the following to view the `~/.ssh/id_ed25519.pub` public key:

   ```terminal
   $ cat ~/.ssh/id_ed25519.pub
   ```
3. Add the SSH private key identity to the SSH agent for your local user, if it has not already been added. SSH agent management of the key is required for password-less SSH authentication onto your cluster nodes, or if you want to use the `./openshift-install gather` command.
   :::note

   On some distributions, default SSH private key identities such as `~/.ssh/id_rsa` and `~/.ssh/id_dsa` are managed automatically.

   :::

   1. If the `ssh-agent` process is not already running for your local user, start it as a background task:
      ```terminal
      $ eval "$(ssh-agent -s)"
      ```

      ```terminal title="Example output"
      Agent pid 31874
      ```

      :::note

      If your cluster is in FIPS mode, only use FIPS-compliant algorithms to generate the SSH key. The key must be either RSA or ECDSA.

      :::
4. Add your SSH private key to the `ssh-agent`:
   ```terminal
   $ ssh-add <path>/<file_name>
   ```

   Specify the path and file name for your SSH private key, such as `~/.ssh/id_ed25519`.

   ```terminal title="Example output"
   Identity added: /home/<you>/<path>/<file_name> (<computer_name>)
   ```

**Next steps**

- When you install OpenShift Container Platform, provide the SSH public key to the installation program. If you install a cluster on infrastructure that you provision, you must provide the key to the installation program.

## Creating the installation files for Azure {#installation-user-infra-generate_installing-azure-user-infra}

To install OpenShift Container Platform on Microsoft Azure by using user-provisioned infrastructure, you must generate the files that the installation program needs to deploy your cluster and modify them so that the cluster creates only the machines that it will use.

You generate and customize the `install-config.yaml` file, Kubernetes manifests, and Ignition config files. You also have the option to first set up a separate `var` partition during the preparation phases of installation.

### Creating a separate `/var` partition {#installation-disk-partitioning-upi-templates_installing-azure-user-infra}

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

### Creating the installation configuration file {#installation-initializing_installing-azure-user-infra}

You can customize the OpenShift Container Platform cluster you install on Microsoft Azure.

:::warning

Do not specify `windows`, `microsoft`, or other variants of these words in the `metadata.name` parameter of the `install-config.yaml` file. Specifying one of these words for the cluster name causes the installation program to generate an error message like the following example message:

```terminal
The resource name 'windows-xxxx-identity' or a part of the name is a trademarked or reserved word.
```

Additionally, specifying `login` at the beginning of the name in the `metadata.name` parameter of the `install-config.yaml` file results in the generation of an error message. You can specify `login` in the middle or end of the name.

:::

**Prerequisites**

- You have the OpenShift Container Platform installation program and the pull secret for your cluster.
- You have an Azure subscription ID and tenant ID.
- If you are installing the cluster using a service principal, you have its application ID and password.
- If you are installing the cluster using a system-assigned managed identity, you have enabled it on the virtual machine that you will run the installation program from.
- If you are installing the cluster using a user-assigned managed identity, you have met these prerequisites:
  - You have its client ID.
  - You have assigned it to the virtual machine that you will run the installation program from.

**Procedure**

1. Optional: If you have run the installation program on this computer before, and want to use an alternative service principal or managed identity, go to the `~/.azure/` directory and delete the `osServicePrincipal.json` configuration file.
   Deleting this file prevents the installation program from automatically reusing subscription and authentication values from a previous installation.
2. Create the `install-config.yaml` file.
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
      2. Select **azure** as the platform to target.
         If the installation program cannot locate the `osServicePrincipal.json` configuration file from a previous installation, you are prompted for Azure subscription and authentication values.
      3. Enter the following Azure parameter values for your subscription:
         - **azure subscription id**: Enter the subscription ID to use for the cluster.
         - **azure tenant id**: Enter the tenant ID.
      4. Depending on the Azure identity you are using to deploy the cluster, do one of the following when prompted for the **azure service principal client id**:
         - If you are using a service principal, enter its application ID.
         - If you are using a system-assigned managed identity, leave this value blank.
         - If you are using a user-assigned managed identity, specify its client ID.
      5. Depending on the Azure identity you are using to deploy the cluster, do one of the following when prompted for the **azure service principal client secret**:
         - If you are using a service principal, enter its password.
         - If you are using a system-assigned managed identity, leave this value blank.
         - If you are using a user-assigned managed identity, leave this value blank.
      6. Select the region to deploy the cluster to.
      7. Select the base domain to deploy the cluster to. The base domain corresponds to the Azure DNS Zone that you created for your cluster.
      8. Enter a descriptive name for your cluster.
         :::warning

         All Azure resources that are available through public endpoints are subject to resource name restrictions, and you cannot create resources that use certain terms. For a list of terms that Azure restricts, see [Resolve reserved resource name errors](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-reserved-resource-name) in the Azure documentation.

         :::
3. Modify the `install-config.yaml` file. You can find more information about the available parameters in the "Installation configuration parameters" section.
   :::note

   If you are installing a three-node cluster, be sure to set the `compute.replicas` parameter to `0`. This ensures that the cluster’s control planes are schedulable. For more information, see "Installing a three-node cluster on Azure".

   :::
4. Back up the `install-config.yaml` file so that you can use it to install multiple clusters.
   :::warning

   The `install-config.yaml` file is consumed during the installation process. If you want to reuse the file, you must back it up now.

   :::

   If previously not detected, the installation program creates an `osServicePrincipal.json` configuration file and stores this file in the `~/.azure/` directory on your computer. This ensures that the installation program can load the profile when it is creating an OpenShift Container Platform cluster on the target platform.

### Configuring the cluster-wide proxy during installation {#installation-configure-proxy_installing-azure-user-infra}

Production environments can deny direct access to the internet and instead have an HTTP or HTTPS proxy available. You can configure a new OpenShift Container Platform cluster to use a proxy by configuring the proxy settings in the `install-config.yaml` file.

**Prerequisites**

- You have an existing `install-config.yaml` file.
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

### Exporting common variables for ARM templates {#installation-user-infra-exporting-common-variables-arm-templates_installing-azure-user-infra}

To deploy Azure infrastructure with the provided ARM templates, you must export a common set of variables that are used with the provided Azure Resource Manager (ARM) templates used to assist in completing a user-provided infrastructure install on Microsoft Azure.

:::note

Specific ARM templates can also require additional exported variables, which are detailed in their related procedures.

:::

**Prerequisites**

- Obtain the OpenShift Container Platform installation program and the pull secret for your cluster.

**Procedure**

1. Export common variables found in the `install-config.yaml` to be used by the provided ARM templates:
   ```terminal
   $ export CLUSTER_NAME=<cluster_name>
   ```

   where:

   <dl>
   <dt><code>&lt;cluster_name&gt;</code></dt>
   <dd>The value of the <code>.metadata.name</code> attribute from the <code>install-config.yaml</code> file.</dd>
   </dl>

   ```terminal
   $ export AZURE_REGION=<azure_region>
   ```

   where:

   <dl>
   <dt><code>&lt;azure_region&gt;</code></dt>
   <dd>The region to deploy the cluster into, for example <code>centralus</code>. This is the value of the <code>.platform.azure.region</code> attribute from the <code>install-config.yaml</code> file.</dd>
   </dl>

   ```terminal
   $ export SSH_KEY=<ssh_key>
   ```

   where: `<ssh_key>`:: The SSH RSA public key file as a string. You must enclose the SSH key in quotes since it contains spaces. This is the value of the `.sshKey` attribute from the `install-config.yaml` file.

   ```terminal
   $ export BASE_DOMAIN=<base_domain>
   ```

   where:

   <dl>
   <dt><code>&lt;base_domain&gt;</code></dt>
   <dd>The base domain to deploy the cluster to. The base domain corresponds to the public DNS zone that you created for your cluster. This is the value of the <code>.baseDomain</code> attribute from the <code>install-config.yaml</code> file.</dd>
   </dl>

   ```terminal
   $ export BASE_DOMAIN_RESOURCE_GROUP=<base_domain_resource_group>
   ```

   where:

   <dl>
   <dt><code>&lt;base_domain_resource_group&gt;</code></dt>
   <dd>The resource group where the public DNS zone exists. This is the value of the <code>.platform.azure.baseDomainResourceGroupName</code> attribute from the <code>install-config.yaml</code> file.</dd>
   </dl>

   For example:

   ```terminal
   $ export CLUSTER_NAME=test-cluster
   ```

   ```terminal
   $ export AZURE_REGION=centralus
   ```

   ```terminal
   $ export SSH_KEY="ssh-rsa xxx/xxx/xxx= user@email.com"
   ```

   ```terminal
   $ export BASE_DOMAIN=example.com
   ```

   ```terminal
   $ export BASE_DOMAIN_RESOURCE_GROUP=ocp-cluster
   ```
2. Export the kubeadmin credentials:
   ```terminal
   $ export KUBECONFIG=<installation_directory>/auth/kubeconfig
   ```

   where:

   <dl>
   <dt><code>&lt;installation_directory&gt;</code></dt>
   <dd>Specify the path to the directory that you stored the installation files in.</dd>
   </dl>

### Creating the Kubernetes manifest and Ignition config files {#installation-user-infra-generate-k8s-manifest-ignition_installing-azure-user-infra}

Because you manually provision infrastructure, you must generate the Kubernetes manifest and Ignition config files that the cluster requires.

The installation program converts the installation configuration into Kubernetes manifests and then wraps them into Ignition configuration files. You use these Ignition files to configure the cluster machines.

:::warning

- The Ignition config files that the OpenShift Container Platform installation program generates contain certificates that expire after 24 hours, which the system then renews. If you shut down the cluster before the system renews the certificates and you later restart the cluster after the 24 hours have elapsed, the cluster automatically recovers the expired certificates. The exception is that you must manually approve the pending `node-bootstrapper` certificate signing requests (CSRs) to recover kubelet certificates. See the documentation for *Recovering from expired control plane certificates* for more information.
- Use Ignition config files within 12 hours after you generate them, because the 24-hour certificate rotates from 16 to 22 hours after you install the cluster. By using the Ignition config files within 12 hours, you can avoid installation failure if the certificate update runs during installation.

:::

**Prerequisites**

- You obtained the OpenShift Container Platform installation program.
- You created the `install-config.yaml` installation configuration file.

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
4. Remove the Kubernetes manifest files that define the worker machines:
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
7. When you configure Azure on user-provisioned infrastructure, you must export some common variables defined in the manifest files to use later in the Azure Resource Manager (ARM) templates:
   1. Export the infrastructure ID by using the following command:
      ```terminal
      $ export INFRA_ID=<infra_id>
      ```

      where:

      <dl>
      <dt><code>&lt;infra_id&gt;</code></dt>
      <dd>Specifies the OpenShift Container Platform cluster identifier (<code>INFRA_ID</code>) in the form of <code>&lt;cluster_name&gt;-&lt;random_string&gt;</code>. Most resources that the provided ARM templates create use this identifier as the base name. This is the value of the <code>.status.infrastructureName</code> attribute from the <code>manifests/cluster-infrastructure-02-config.yml</code> file.</dd>
      </dl>
   2. Export the resource group by using the following command:
      ```terminal
      $ export RESOURCE_GROUP=<resource_group>
      ```

      where:

      <dl>
      <dt><code>&lt;resource_group&gt;</code></dt>
      <dd>Specifies the <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/overview#resource-groups">resource group</a> that contains all resources in this Azure deployment. The resource group name is also based on the <code>INFRA_ID</code>, in the form of <code>&lt;cluster_name&gt;-&lt;random_string&gt;-rg</code>. This is the value of the <code>.status.platformStatus.azure.resourceGroupName</code> attribute from the <code>manifests/cluster-infrastructure-02-config.yml</code> file.</dd>
      </dl>
8. To create the Ignition configuration files, run the following command from the directory that contains the installation program:
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

## Creating the Azure resource group {#installation-azure-create-resource-group-and-identity_installing-azure-user-infra}

You must create a Microsoft Azure resource group and an identity for that resource group. Both are used when you install your OpenShift Container Platform cluster on Azure.

For more information, see "Azure resource groups".

**Procedure**

1. Create the resource group in a supported Azure region:

```terminal
$ az group create --name ${RESOURCE_GROUP} --location ${AZURE_REGION}
```

1. Create an Azure identity for the resource group:
   ```terminal
   $ az identity create -g ${RESOURCE_GROUP} -n ${INFRA_ID}-identity
   ```

   This is used to grant the required access to Operators in your cluster. For example, this allows the Ingress Operator to create a public IP and its load balancer. You must assign the Azure identity to a role.
2. Grant the Contributor role to the Azure identity:
   1. Export the following variables required by the Azure role assignment:
      ```terminal
      $ export PRINCIPAL_ID=`az identity show -g ${RESOURCE_GROUP} -n ${INFRA_ID}-identity --query principalId --out tsv`
      ```

      ```terminal
      $ export RESOURCE_GROUP_ID=`az group show -g ${RESOURCE_GROUP} --query id --out tsv`
      ```
   2. Assign the Contributor role to the identity:
      ```terminal
      $ az role assignment create --assignee "${PRINCIPAL_ID}" --role 'Contributor' --scope "${RESOURCE_GROUP_ID}"
      ```

      :::note

      If you want to assign a custom role with all the required permissions to the identity, run the following command:

      ```terminal
      $ az role assignment create --assignee "${PRINCIPAL_ID}" --role <custom_role> \
      --scope "${RESOURCE_GROUP_ID}"
      ```

      Replace `<custom_role>` with the custom role name.

      :::

**Additional resources**

- [Azure resource groups (Azure documentation)](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/overview#resource-groups)

## Uploading the RHCOS cluster image and bootstrap Ignition config file {#installation-azure-user-infra-uploading-rhcos_installing-azure-user-infra}

To make the RHCOS cluster image and bootstrap Ignition config accessible during deployment, you can upload them to an Azure storage container.

The Azure client does not support deployments based on files existing locally. You must copy and store the RHCOS virtual hard disk (VHD) cluster image and bootstrap Ignition config file in a storage container so they are accessible during deployment.

**Prerequisites**

- Generate the Ignition config files for your cluster.

**Procedure**

1. Create an Azure storage account to store the VHD cluster image:
   ```terminal
   $ az storage account create -g ${RESOURCE_GROUP} --location ${AZURE_REGION} --name ${CLUSTER_NAME}sa --kind Storage --sku Standard_LRS
   ```

   :::warning

   The Azure storage account name must be between 3 and 24 characters in length and use numbers and lower-case letters only. If your `CLUSTER_NAME` variable does not follow these restrictions, you must manually define the Azure storage account name. For more information on Azure storage account name restrictions, see [Resolve errors for storage account names](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/error-storage-account-name) in the Azure documentation.

   :::
2. Export the storage account key as an environment variable:
   ```terminal
   $ export ACCOUNT_KEY=`az storage account keys list -g ${RESOURCE_GROUP} --account-name ${CLUSTER_NAME}sa --query "[0].value" -o tsv`
   ```
3. Export the URL of the RHCOS VHD to an environment variable:
   ```terminal
   $ export VHD_URL=`openshift-install coreos print-stream-json | jq -r '.architectures.<architecture>."rhel-coreos-extensions"."azure-disk".url'`
   ```

   where:

   <dl>
   <dt><code>&lt;architecture&gt;</code></dt>
   <dd>Specifies the architecture, valid values include <code>x86_64</code> or <code>aarch64</code>.</dd>
   </dl>

   :::warning

   The RHCOS images might not change with every release of OpenShift Container Platform. You must specify an image with the highest version that is less than or equal to the OpenShift Container Platform version that you install. Use the image version that matches your OpenShift Container Platform version if it is available.

   :::
4. Create the storage container for the VHD:
   ```terminal
   $ az storage container create --name vhd --account-name ${CLUSTER_NAME}sa --account-key ${ACCOUNT_KEY}
   ```
5. Copy the local VHD to a blob:
   ```terminal
   $ az storage blob copy start --account-name ${CLUSTER_NAME}sa --account-key ${ACCOUNT_KEY} --destination-blob "rhcos.vhd" --destination-container vhd --source-uri "${VHD_URL}"
   ```
6. Create a blob storage container and upload the generated `bootstrap.ign` file:
   ```terminal
   $ az storage container create --name files --account-name ${CLUSTER_NAME}sa --account-key ${ACCOUNT_KEY}
   ```

   ```terminal
   $ az storage blob upload --account-name ${CLUSTER_NAME}sa --account-key ${ACCOUNT_KEY} -c "files" -f "<installation_directory>/bootstrap.ign" -n "bootstrap.ign"
   ```

## Example for creating DNS zones {#installation-azure-create-dns-zones_installing-azure-user-infra}

To create the required DNS zones for a user-provisioned cluster, you can add public and private DNS zones that resolve your cluster domain. You should choose the DNS strategy that fits your scenario.

For this example, [Azure’s DNS solution](https://docs.microsoft.com/en-us/azure/dns/dns-overview) is used, so you will create a new public DNS zone for external (internet) visibility and a private DNS zone for internal cluster resolution.

:::note

The public DNS zone is not required to exist in the same resource group as the cluster deployment and might already exist in your organization for the desired base domain. If that is the case, you can skip creating the public DNS zone; be sure the installation config you generated earlier reflects that scenario.

:::

**Procedure**

1. Create the new public DNS zone in the resource group exported in the `BASE_DOMAIN_RESOURCE_GROUP` environment variable:

```terminal
$ az network dns zone create -g ${BASE_DOMAIN_RESOURCE_GROUP} -n ${CLUSTER_NAME}.${BASE_DOMAIN}
```

You can skip this step if you are using a public DNS zone that already exists.

1. Create the private DNS zone in the same resource group as the rest of this deployment:
   ```terminal
   $ az network private-dns zone create -g ${RESOURCE_GROUP} -n ${CLUSTER_NAME}.${BASE_DOMAIN}
   ```

**Additional resources**

- [Configuring a public DNS zone in Azure](#installation-azure-network-config_installing-azure-user-infra)

## Creating a VNet in Azure {#installation-creating-azure-vnet_installing-azure-user-infra}

To provide network connectivity for your cluster on Microsoft Azure, you can create a virtual network (VNet) by using the Azure Resource Manager (ARM) template.

:::note

If you do not use the provided ARM template to create your Azure infrastructure, you must review the provided information and manually create the infrastructure. If your cluster does not initialize correctly, you might have to contact Red Hat support with your installation logs.

:::

**Procedure**

1. Copy the template from the **ARM template for the VNet** section of this topic and save it as `01_vnet.json` in your cluster’s installation directory. This template describes the VNet that your cluster requires.
2. Create the deployment by using the `az` CLI:
   ```terminal
   $ az deployment group create -g ${RESOURCE_GROUP} \
     --template-file "<installation_directory>/01_vnet.json" \
     --parameters baseName="${INFRA_ID}"
   ```

   `baseName` specifies the base name to be used in resource names; this is usually the cluster’s infrastructure ID.
3. Link the VNet template to the private DNS zone:
   ```terminal
   $ az network private-dns link vnet create -g ${RESOURCE_GROUP} -z ${CLUSTER_NAME}.${BASE_DOMAIN} -n ${INFRA_ID}-network-link -v "${INFRA_ID}-vnet" -e false
   ```

### ARM template for the VNet {#installation-arm-vnet_installing-azure-user-infra}

Use the `01_vnet.json` Azure Resource Manager (ARM) template to deploy the virtual network (VNet) for your OpenShift Container Platform cluster.

<details>
<summary>`01_vnet.json` ARM template</summary>

```json

{
  "$schema" : "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion" : "1.0.0.0",
  "parameters" : {
    "baseName" : {
      "type" : "string",
      "minLength" : 1,
      "metadata" : {
        "description" : "Base name to be used in resource names (usually the cluster's Infra ID)"
      }
    }
  },
  "variables" : {
    "location" : "[resourceGroup().location]",
    "virtualNetworkName" : "[concat(parameters('baseName'), '-vnet')]",
    "addressPrefix" : "10.0.0.0/16",
    "masterSubnetName" : "[concat(parameters('baseName'), '-master-subnet')]",
    "masterSubnetPrefix" : "10.0.0.0/24",
    "nodeSubnetName" : "[concat(parameters('baseName'), '-worker-subnet')]",
    "nodeSubnetPrefix" : "10.0.1.0/24",
    "clusterNsgName" : "[concat(parameters('baseName'), '-nsg')]"
  },
  "resources" : [
    {
      "apiVersion" : "2018-12-01",
      "type" : "Microsoft.Network/virtualNetworks",
      "name" : "[variables('virtualNetworkName')]",
      "location" : "[variables('location')]",
      "dependsOn" : [
        "[concat('Microsoft.Network/networkSecurityGroups/', variables('clusterNsgName'))]"
      ],
      "properties" : {
        "addressSpace" : {
          "addressPrefixes" : [
            "[variables('addressPrefix')]"
          ]
        },
        "subnets" : [
          {
            "name" : "[variables('masterSubnetName')]",
            "properties" : {
              "addressPrefix" : "[variables('masterSubnetPrefix')]",
              "serviceEndpoints": [],
              "networkSecurityGroup" : {
                "id" : "[resourceId('Microsoft.Network/networkSecurityGroups', variables('clusterNsgName'))]"
              }
            }
          },
          {
            "name" : "[variables('nodeSubnetName')]",
            "properties" : {
              "addressPrefix" : "[variables('nodeSubnetPrefix')]",
              "serviceEndpoints": [],
              "networkSecurityGroup" : {
                "id" : "[resourceId('Microsoft.Network/networkSecurityGroups', variables('clusterNsgName'))]"
              }
            }
          }
        ]
      }
    },
    {
      "type" : "Microsoft.Network/networkSecurityGroups",
      "name" : "[variables('clusterNsgName')]",
      "apiVersion" : "2018-10-01",
      "location" : "[variables('location')]",
      "properties" : {
        "securityRules" : [
          {
            "name" : "apiserver_in",
            "properties" : {
              "protocol" : "Tcp",
              "sourcePortRange" : "*",
              "destinationPortRange" : "6443",
              "sourceAddressPrefix" : "*",
              "destinationAddressPrefix" : "*",
              "access" : "Allow",
              "priority" : 101,
              "direction" : "Inbound"
            }
          }
        ]
      }
    }
  ]
}

```

</details>

## Deploying the RHCOS cluster image for the Azure infrastructure {#installation-azure-user-infra-deploying-rhcos_installing-azure-user-infra}

To provision cluster nodes on Microsoft Azure, you must use a valid Red Hat Enterprise Linux CoreOS (RHCOS) image for Microsoft Azure for your OpenShift Container Platform nodes.

**Prerequisites**

- Store the RHCOS virtual hard disk (VHD) cluster image in an Azure storage container.
- Store the bootstrap Ignition config file in an Azure storage container.

**Procedure**

1. Copy the template from the **ARM template for image storage** section of this topic and save it as `02_storage.json` in your cluster’s installation directory. This template describes the image storage that your cluster requires.
2. Export the RHCOS VHD blob URL as a variable:
   ```terminal
   $ export VHD_BLOB_URL=`az storage blob url --account-name ${CLUSTER_NAME}sa --account-key ${ACCOUNT_KEY} -c vhd -n "rhcos.vhd" -o tsv`
   ```
3. Deploy the cluster image:
   ```terminal
   $ az deployment group create -g ${RESOURCE_GROUP} \
     --template-file "<installation_directory>/02_storage.json" \
     --parameters vhdBlobURL="${VHD_BLOB_URL}" \
     --parameters baseName="${INFRA_ID}" \
     --parameters storageAccount="${CLUSTER_NAME}sa" \
     --parameters architecture="<architecture>"
   ```

   where:

   <dl>
   <dt><code>vhdBlobURL</code></dt>
   <dd>Specifies the blob URL of the RHCOS VHD to be used to create master and worker machines.</dd>
   <dt><code>baseName</code></dt>
   <dd>Specifies the base name to be used in resource names; this is usually the cluster’s infrastructure ID.</dd>
   <dt><code>storageAccount</code></dt>
   <dd>Specifies the name of your Azure storage account.</dd>
   <dt><code>architecture</code></dt>
   <dd>Specifies the system architecture. Valid values are <code>x64</code> (default) or <code>Arm64</code>.</dd>
   </dl>

### ARM template for image storage {#installation-arm-image-storage_installing-azure-user-infra}

Use the `02_storage.json` Azure Resource Manager (ARM) template to deploy stored Red Hat Enterprise Linux CoreOS (RHCOS) image resources for your OpenShift Container Platform cluster.

<details>
<summary>`02_storage.json` ARM template</summary>

```json

{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "architecture": {
      "type": "string",
      "metadata": {
        "description": "The architecture of the Virtual Machines"
      },
      "defaultValue": "x64",
      "allowedValues": [
        "Arm64",
        "x64"
      ]
    },
    "baseName": {
      "type": "string",
      "minLength": 1,
      "metadata": {
        "description": "Base name to be used in resource names (usually the cluster's Infra ID)"
      }
    },
    "storageAccount": {
      "type": "string",
      "metadata": {
        "description": "The Storage Account name"
      }
    },
    "vhdBlobURL": {
      "type": "string",
      "metadata": {
        "description": "URL pointing to the blob where the VHD to be used to create master and worker machines is located"
      }
    }
  },
  "variables": {
    "location": "[resourceGroup().location]",
    "galleryName": "[concat('gallery_', replace(parameters('baseName'), '-', '_'))]",
    "imageName": "[parameters('baseName')]",
    "imageNameGen2": "[concat(parameters('baseName'), '-gen2')]",
    "imageRelease": "1.0.0"
  },
  "resources": [
    {
      "apiVersion": "2021-10-01",
      "type": "Microsoft.Compute/galleries",
      "name": "[variables('galleryName')]",
      "location": "[variables('location')]",
      "resources": [
        {
          "apiVersion": "2021-10-01",
          "type": "images",
          "name": "[variables('imageName')]",
          "location": "[variables('location')]",
          "dependsOn": [
            "[variables('galleryName')]"
          ],
          "properties": {
            "architecture": "[parameters('architecture')]",
            "hyperVGeneration": "V1",
            "identifier": {
              "offer": "rhcos",
              "publisher": "RedHat",
              "sku": "basic"
            },
            "osState": "Generalized",
            "osType": "Linux"
          },
          "resources": [
            {
              "apiVersion": "2022-03-03",
              "type": "versions",
              "name": "[variables('imageRelease')]",
              "location": "[variables('location')]",
              "dependsOn": [
                "[variables('imageName')]"
              ],
              "properties": {
                "publishingProfile": {
                  "storageAccountType": "Standard_LRS",
                  "targetRegions": [
                    {
                      "name": "[variables('location')]",
                      "regionalReplicaCount": "1"
                    }
                  ]
                },
                "storageProfile": {
                  "osDiskImage": {
                    "source": {
                      "storageAccountId": "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccount'))]",
                      "uri": "[parameters('vhdBlobURL')]"
                    }
                  }
                }
              }
            }
          ]
        },
        {
          "apiVersion": "2021-10-01",
          "type": "images",
          "name": "[variables('imageNameGen2')]",
          "location": "[variables('location')]",
          "dependsOn": [
            "[variables('galleryName')]"
          ],
          "properties": {
            "architecture": "[parameters('architecture')]",
            "hyperVGeneration": "V2",
            "identifier": {
              "offer": "rhcos-gen2",
              "publisher": "RedHat-gen2",
              "sku": "gen2"
            },
            "osState": "Generalized",
            "osType": "Linux"
          },
          "resources": [
            {
              "apiVersion": "2022-03-03",
              "type": "versions",
              "name": "[variables('imageRelease')]",
              "location": "[variables('location')]",
              "dependsOn": [
                "[variables('imageNameGen2')]"
              ],
              "properties": {
                "publishingProfile": {
                  "storageAccountType": "Standard_LRS",
                  "targetRegions": [
                    {
                      "name": "[variables('location')]",
                      "regionalReplicaCount": "1"
                    }
                  ]
                },
                "storageProfile": {
                  "osDiskImage": {
                    "source": {
                      "storageAccountId": "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccount'))]",
                      "uri": "[parameters('vhdBlobURL')]"
                    }
                  }
                }
              }
            }
          ]
        }
      ]
    }
  ]
}

```

</details>

## Networking requirements for user-provisioned infrastructure {#installation-network-user-infra_installing-azure-user-infra}

You must configure networking for all the Red Hat Enterprise Linux CoreOS (RHCOS) machines in `initramfs` during boot, so that they can fetch their Ignition config files.

### Network connectivity requirements {#installation-network-connectivity-user-infra_installing-azure-user-infra}

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

## Creating networking and load balancing components in Azure {#installation-creating-azure-dns_installing-azure-user-infra}

To enable cluster communication on Microsoft Azure, you must deploy networking and load balancing components by using the Azure Resource Manager (ARM) template.

:::note

If you do not use the provided ARM template to create your Azure infrastructure, you must review the provided information and manually create the infrastructure. If your cluster does not initialize correctly, you might have to contact Red Hat support with your installation logs.

:::

**Prerequisites**

- Create and configure a VNet and associated subnets in Azure.

**Procedure**

1. Copy the template from the **ARM template for the network and load balancers** section of this topic and save it as `03_infra.json` in your cluster’s installation directory. This template describes the networking and load balancing objects that your cluster requires.
2. Create the deployment by using the `az` CLI:
   ```terminal
   $ az deployment group create -g ${RESOURCE_GROUP} \
     --template-file "<installation_directory>/03_infra.json" \
     --parameters privateDNSZoneName="${CLUSTER_NAME}.${BASE_DOMAIN}" \
     --parameters baseName="${INFRA_ID}"
   ```

   where:

   <dl>
   <dt><code>privateDNSZoneName</code></dt>
   <dd>Specifies the name of the private DNS zone.</dd>
   <dt><code>baseName</code></dt>
   <dd>Specifies the base name to be used in resource names; this is usually the cluster’s infrastructure ID.</dd>
   </dl>
3. Create an `api` DNS record in the public zone for the API public load balancer. The `${BASE_DOMAIN_RESOURCE_GROUP}` variable must point to the resource group where the public DNS zone exists.
   1. Export the following variable:
      ```terminal
      $ export PUBLIC_IP=`az network public-ip list -g ${RESOURCE_GROUP} --query "[?name=='${INFRA_ID}-master-pip'] | [0].ipAddress" -o tsv`
      ```
   2. Create the `api` DNS record in a new public zone:
      ```terminal
      $ az network dns record-set a add-record -g ${BASE_DOMAIN_RESOURCE_GROUP} -z ${CLUSTER_NAME}.${BASE_DOMAIN} -n api -a ${PUBLIC_IP} --ttl 60
      ```

      If you are adding the cluster to an existing public zone, you can create the `api` DNS record in it instead:

      ```terminal
      $ az network dns record-set a add-record -g ${BASE_DOMAIN_RESOURCE_GROUP} -z ${BASE_DOMAIN} -n api.${CLUSTER_NAME} -a ${PUBLIC_IP} --ttl 60
      ```

### ARM template for the network and load balancers {#installation-arm-dns_installing-azure-user-infra}

Use the `03_infra.json` Azure Resource Manager (ARM) template to deploy networking objects and load balancers for your OpenShift Container Platform cluster.

<details>
<summary>`03_infra.json` ARM template</summary>

```json

{
  "$schema" : "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion" : "1.0.0.0",
  "parameters" : {
    "baseName" : {
      "type" : "string",
      "minLength" : 1,
      "metadata" : {
        "description" : "Base name to be used in resource names (usually the cluster's Infra ID)"
      }
    },
    "vnetBaseName": {
      "type": "string",
      "defaultValue": "",
      "metadata" : {
        "description" : "The specific customer vnet's base name (optional)"
      }
    },
    "privateDNSZoneName" : {
      "type" : "string",
      "metadata" : {
        "description" : "Name of the private DNS zone"
      }
    }
  },
  "variables" : {
    "location" : "[resourceGroup().location]",
    "virtualNetworkName" : "[concat(if(not(empty(parameters('vnetBaseName'))), parameters('vnetBaseName'), parameters('baseName')), '-vnet')]",
    "virtualNetworkID" : "[resourceId('Microsoft.Network/virtualNetworks', variables('virtualNetworkName'))]",
    "masterSubnetName" : "[concat(if(not(empty(parameters('vnetBaseName'))), parameters('vnetBaseName'), parameters('baseName')), '-master-subnet')]",
    "masterSubnetRef" : "[concat(variables('virtualNetworkID'), '/subnets/', variables('masterSubnetName'))]",
    "masterPublicIpAddressName" : "[concat(parameters('baseName'), '-master-pip')]",
    "masterPublicIpAddressID" : "[resourceId('Microsoft.Network/publicIPAddresses', variables('masterPublicIpAddressName'))]",
    "masterLoadBalancerName" : "[parameters('baseName')]",
    "masterLoadBalancerID" : "[resourceId('Microsoft.Network/loadBalancers', variables('masterLoadBalancerName'))]",
    "internalLoadBalancerName" : "[concat(parameters('baseName'), '-internal-lb')]",
    "internalLoadBalancerID" : "[resourceId('Microsoft.Network/loadBalancers', variables('internalLoadBalancerName'))]",
    "skuName": "Standard"
  },
  "resources" : [
    {
      "apiVersion" : "2018-12-01",
      "type" : "Microsoft.Network/publicIPAddresses",
      "name" : "[variables('masterPublicIpAddressName')]",
      "location" : "[variables('location')]",
      "sku": {
        "name": "[variables('skuName')]"
      },
      "properties" : {
        "publicIPAllocationMethod" : "Static",
        "dnsSettings" : {
          "domainNameLabel" : "[variables('masterPublicIpAddressName')]"
        }
      }
    },
    {
      "apiVersion" : "2018-12-01",
      "type" : "Microsoft.Network/loadBalancers",
      "name" : "[variables('masterLoadBalancerName')]",
      "location" : "[variables('location')]",
      "sku": {
        "name": "[variables('skuName')]"
      },
      "dependsOn" : [
        "[concat('Microsoft.Network/publicIPAddresses/', variables('masterPublicIpAddressName'))]"
      ],
      "properties" : {
        "frontendIPConfigurations" : [
          {
            "name" : "public-lb-ip-v4",
            "properties" : {
              "publicIPAddress" : {
                "id" : "[variables('masterPublicIpAddressID')]"
              }
            }
          }
        ],
        "backendAddressPools" : [
          {
            "name" : "[variables('masterLoadBalancerName')]"
          }
        ],
        "loadBalancingRules" : [
          {
            "name" : "api-internal",
            "properties" : {
              "frontendIPConfiguration" : {
                "id" :"[concat(variables('masterLoadBalancerID'), '/frontendIPConfigurations/public-lb-ip-v4')]"
              },
              "backendAddressPool" : {
                "id" : "[concat(variables('masterLoadBalancerID'), '/backendAddressPools/', variables('masterLoadBalancerName'))]"
              },
              "protocol" : "Tcp",
              "loadDistribution" : "Default",
              "idleTimeoutInMinutes" : 30,
              "frontendPort" : 6443,
              "backendPort" : 6443,
              "probe" : {
                "id" : "[concat(variables('masterLoadBalancerID'), '/probes/api-internal-probe')]"
              }
            }
          }
        ],
        "probes" : [
          {
            "name" : "api-internal-probe",
            "properties" : {
              "protocol" : "Https",
              "port" : 6443,
              "requestPath": "/readyz",
              "intervalInSeconds" : 10,
              "numberOfProbes" : 3
            }
          }
        ]
      }
    },
    {
      "apiVersion" : "2018-12-01",
      "type" : "Microsoft.Network/loadBalancers",
      "name" : "[variables('internalLoadBalancerName')]",
      "location" : "[variables('location')]",
      "sku": {
        "name": "[variables('skuName')]"
      },
      "properties" : {
        "frontendIPConfigurations" : [
          {
            "name" : "internal-lb-ip",
            "properties" : {
              "privateIPAllocationMethod" : "Dynamic",
              "subnet" : {
                "id" : "[variables('masterSubnetRef')]"
              },
              "privateIPAddressVersion" : "IPv4"
            }
          }
        ],
        "backendAddressPools" : [
          {
            "name" : "internal-lb-backend"
          }
        ],
        "loadBalancingRules" : [
          {
            "name" : "api-internal",
            "properties" : {
              "frontendIPConfiguration" : {
                "id" : "[concat(variables('internalLoadBalancerID'), '/frontendIPConfigurations/internal-lb-ip')]"
              },
              "frontendPort" : 6443,
              "backendPort" : 6443,
              "enableFloatingIP" : false,
              "idleTimeoutInMinutes" : 30,
              "protocol" : "Tcp",
              "enableTcpReset" : false,
              "loadDistribution" : "Default",
              "backendAddressPool" : {
                "id" : "[concat(variables('internalLoadBalancerID'), '/backendAddressPools/internal-lb-backend')]"
              },
              "probe" : {
                "id" : "[concat(variables('internalLoadBalancerID'), '/probes/api-internal-probe')]"
              }
            }
          },
          {
            "name" : "sint",
            "properties" : {
              "frontendIPConfiguration" : {
                "id" : "[concat(variables('internalLoadBalancerID'), '/frontendIPConfigurations/internal-lb-ip')]"
              },
              "frontendPort" : 22623,
              "backendPort" : 22623,
              "enableFloatingIP" : false,
              "idleTimeoutInMinutes" : 30,
              "protocol" : "Tcp",
              "enableTcpReset" : false,
              "loadDistribution" : "Default",
              "backendAddressPool" : {
                "id" : "[concat(variables('internalLoadBalancerID'), '/backendAddressPools/internal-lb-backend')]"
              },
              "probe" : {
                "id" : "[concat(variables('internalLoadBalancerID'), '/probes/sint-probe')]"
              }
            }
          }
        ],
        "probes" : [
          {
            "name" : "api-internal-probe",
            "properties" : {
              "protocol" : "Https",
              "port" : 6443,
              "requestPath": "/readyz",
              "intervalInSeconds" : 10,
              "numberOfProbes" : 3
            }
          },
          {
            "name" : "sint-probe",
            "properties" : {
              "protocol" : "Https",
              "port" : 22623,
              "requestPath": "/healthz",
              "intervalInSeconds" : 10,
              "numberOfProbes" : 3
            }
          }
        ]
      }
    },
    {
      "apiVersion": "2018-09-01",
      "type": "Microsoft.Network/privateDnsZones/A",
      "name": "[concat(parameters('privateDNSZoneName'), '/api')]",
      "location" : "[variables('location')]",
      "dependsOn" : [
        "[concat('Microsoft.Network/loadBalancers/', variables('internalLoadBalancerName'))]"
      ],
      "properties": {
        "ttl": 60,
        "aRecords": [
          {
            "ipv4Address": "[reference(variables('internalLoadBalancerName')).frontendIPConfigurations[0].properties.privateIPAddress]"
          }
        ]
      }
    },
    {
      "apiVersion": "2018-09-01",
      "type": "Microsoft.Network/privateDnsZones/A",
      "name": "[concat(parameters('privateDNSZoneName'), '/api-int')]",
      "location" : "[variables('location')]",
      "dependsOn" : [
        "[concat('Microsoft.Network/loadBalancers/', variables('internalLoadBalancerName'))]"
      ],
      "properties": {
        "ttl": 60,
        "aRecords": [
          {
            "ipv4Address": "[reference(variables('internalLoadBalancerName')).frontendIPConfigurations[0].properties.privateIPAddress]"
          }
        ]
      }
    }
  ]
}

```

</details>

## Creating the bootstrap machine in Azure {#installation-creating-azure-bootstrap_installing-azure-user-infra}

To initialize your OpenShift Container Platform cluster on Microsoft Azure, you must deploy the bootstrap machine by using the `04_bootstrap.json` ARM template.

:::note

If you do not use the provided ARM template to create your bootstrap machine, you must review the provided information and manually create the infrastructure. If your cluster does not initialize correctly, you might have to contact Red Hat support with your installation logs.

:::

**Prerequisites**

- Create and configure networking and load balancers in Azure.
- Create the Azure identity and grant the appropriate roles.

**Procedure**

1. Copy the template from the **ARM template for the bootstrap machine** section of this topic and save it as `04_bootstrap.json` in your cluster’s installation directory. This template describes the bootstrap machine that your cluster requires.
2. Export the bootstrap URL variable:
   ```terminal
   $ bootstrap_url_expiry=`date -u -d "10 hours" '+%Y-%m-%dT%H:%MZ'`
   ```

   ```terminal
   $ export BOOTSTRAP_URL=`az storage blob generate-sas -c 'files' -n 'bootstrap.ign' --https-only --full-uri --permissions r --expiry $bootstrap_url_expiry --account-name ${CLUSTER_NAME}sa --account-key ${ACCOUNT_KEY} -o tsv`
   ```
3. Export the bootstrap ignition variable:
   ```terminal
   $ export BOOTSTRAP_IGNITION=`jq -rcnM --arg v "3.2.0" --arg url ${BOOTSTRAP_URL} '{ignition:{version:$v,config:{replace:{source:$url}}}}' | base64 | tr -d '\n'`
   ```
4. Create the deployment by using the `az` CLI:
   ```terminal
   $ az deployment group create -g ${RESOURCE_GROUP} \
     --template-file "<installation_directory>/04_bootstrap.json" \
     --parameters bootstrapIgnition="${BOOTSTRAP_IGNITION}" \
     --parameters baseName="${INFRA_ID}" \
     --parameter bootstrapVMSize="Standard_D4s_v3"
   ```

   where:

   <dl>
   <dt><code>bootstrapIgnition</code></dt>
   <dd>Specifies the bootstrap Ignition content for the bootstrap cluster.</dd>
   <dt><code>baseName</code></dt>
   <dd>Specifies the base name to be used in resource names; this is usually the cluster’s infrastructure ID.</dd>
   <dt><code>bootstrapVMSize</code></dt>
   <dd>Specifies the size of the bootstrap VM. Use a VM size compatible with your specified architecture. If this value is not defined, the default value from the template is set. This parameter is optional.</dd>
   </dl>

### ARM template for the bootstrap machine {#installation-arm-bootstrap_installing-azure-user-infra}

Use the `04_bootstrap.json` Azure Resource Manager (ARM) template to deploy the bootstrap machine for your OpenShift Container Platform cluster.

<details>
<summary>`04_bootstrap.json` ARM template</summary>

```json

{
  "$schema" : "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion" : "1.0.0.0",
  "parameters" : {
    "baseName" : {
      "type" : "string",
      "minLength" : 1,
      "metadata" : {
        "description" : "Base name to be used in resource names (usually the cluster's Infra ID)"
      }
    },
    "vnetBaseName": {
      "type": "string",
      "defaultValue": "",
      "metadata" : {
        "description" : "The specific customer vnet's base name (optional)"
      }
    },
    "bootstrapIgnition" : {
      "type" : "string",
      "minLength" : 1,
      "metadata" : {
        "description" : "Bootstrap ignition content for the bootstrap cluster"
      }
    },
    "sshKeyData" : {
      "type" : "securestring",
      "defaultValue" : "Unused",
      "metadata" : {
        "description" : "Unused"
      }
    },
    "bootstrapVMSize" : {
      "type" : "string",
      "defaultValue" : "Standard_D4s_v3",
      "metadata" : {
        "description" : "The size of the Bootstrap Virtual Machine"
      }
    },
    "hyperVGen": {
      "type": "string",
      "metadata": {
        "description": "VM generation image to use"
      },
      "defaultValue": "V2",
      "allowedValues": [
        "V1",
        "V2"
      ]
    }
  },
  "variables" : {
    "location" : "[resourceGroup().location]",
    "virtualNetworkName" : "[concat(if(not(empty(parameters('vnetBaseName'))), parameters('vnetBaseName'), parameters('baseName')), '-vnet')]",
    "virtualNetworkID" : "[resourceId('Microsoft.Network/virtualNetworks', variables('virtualNetworkName'))]",
    "masterSubnetName" : "[concat(if(not(empty(parameters('vnetBaseName'))), parameters('vnetBaseName'), parameters('baseName')), '-master-subnet')]",
    "masterSubnetRef" : "[concat(variables('virtualNetworkID'), '/subnets/', variables('masterSubnetName'))]",
    "masterLoadBalancerName" : "[parameters('baseName')]",
    "internalLoadBalancerName" : "[concat(parameters('baseName'), '-internal-lb')]",
    "sshKeyPath" : "/home/core/.ssh/authorized_keys",
    "identityName" : "[concat(parameters('baseName'), '-identity')]",
    "vmName" : "[concat(parameters('baseName'), '-bootstrap')]",
    "nicName" : "[concat(variables('vmName'), '-nic')]",
    "galleryName": "[concat('gallery_', replace(parameters('baseName'), '-', '_'))]",
    "imageName" : "[concat(parameters('baseName'), if(equals(parameters('hyperVGen'), 'V2'), '-gen2', ''))]",
    "clusterNsgName" : "[concat(if(not(empty(parameters('vnetBaseName'))), parameters('vnetBaseName'), parameters('baseName')), '-nsg')]",
    "sshPublicIpAddressName" : "[concat(variables('vmName'), '-ssh-pip')]"
  },
  "resources" : [
    {
      "apiVersion" : "2018-12-01",
      "type" : "Microsoft.Network/publicIPAddresses",
      "name" : "[variables('sshPublicIpAddressName')]",
      "location" : "[variables('location')]",
      "sku": {
        "name": "Standard"
      },
      "properties" : {
        "publicIPAllocationMethod" : "Static",
        "dnsSettings" : {
          "domainNameLabel" : "[variables('sshPublicIpAddressName')]"
        }
      }
    },
    {
      "apiVersion" : "2018-06-01",
      "type" : "Microsoft.Network/networkInterfaces",
      "name" : "[variables('nicName')]",
      "location" : "[variables('location')]",
      "dependsOn" : [
        "[resourceId('Microsoft.Network/publicIPAddresses', variables('sshPublicIpAddressName'))]"
      ],
      "properties" : {
        "ipConfigurations" : [
          {
            "name" : "pipConfig",
            "properties" : {
              "privateIPAllocationMethod" : "Dynamic",
              "publicIPAddress": {
                "id": "[resourceId('Microsoft.Network/publicIPAddresses', variables('sshPublicIpAddressName'))]"
              },
              "subnet" : {
                "id" : "[variables('masterSubnetRef')]"
              },
              "loadBalancerBackendAddressPools" : [
                {
                  "id" : "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/loadBalancers/', variables('masterLoadBalancerName'), '/backendAddressPools/', variables('masterLoadBalancerName'))]"
                },
                {
                  "id" : "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/loadBalancers/', variables('internalLoadBalancerName'), '/backendAddressPools/internal-lb-backend')]"
                }
              ]
            }
          }
        ]
      }
    },
    {
      "apiVersion" : "2018-06-01",
      "type" : "Microsoft.Compute/virtualMachines",
      "name" : "[variables('vmName')]",
      "location" : "[variables('location')]",
      "identity" : {
        "type" : "userAssigned",
        "userAssignedIdentities" : {
          "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/', variables('identityName'))]" : {}
        }
      },
      "dependsOn" : [
        "[concat('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
      ],
      "properties" : {
        "hardwareProfile" : {
          "vmSize" : "[parameters('bootstrapVMSize')]"
        },
        "osProfile" : {
          "computerName" : "[variables('vmName')]",
          "adminUsername" : "core",
          "adminPassword" : "NotActuallyApplied!",
          "customData" : "[parameters('bootstrapIgnition')]",
          "linuxConfiguration" : {
            "disablePasswordAuthentication" : false
          }
        },
        "storageProfile" : {
          "imageReference": {
            "id": "[resourceId('Microsoft.Compute/galleries/images', variables('galleryName'), variables('imageName'))]"
          },
          "osDisk" : {
            "name": "[concat(variables('vmName'),'_OSDisk')]",
            "osType" : "Linux",
            "createOption" : "FromImage",
            "managedDisk": {
              "storageAccountType": "Premium_LRS"
            },
            "diskSizeGB" : 100
          }
        },
        "networkProfile" : {
          "networkInterfaces" : [
            {
              "id" : "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
            }
          ]
        }
      }
    },
    {
      "apiVersion" : "2018-06-01",
      "type": "Microsoft.Network/networkSecurityGroups/securityRules",
      "name" : "[concat(variables('clusterNsgName'), '/bootstrap_ssh_in')]",
      "location" : "[variables('location')]",
      "dependsOn" : [
        "[resourceId('Microsoft.Compute/virtualMachines', variables('vmName'))]"
      ],
      "properties": {
        "protocol" : "Tcp",
        "sourcePortRange" : "*",
        "destinationPortRange" : "22",
        "sourceAddressPrefix" : "*",
        "destinationAddressPrefix" : "*",
        "access" : "Allow",
        "priority" : 100,
        "direction" : "Inbound"
      }
    }
  ]
}

```

</details>

## Creating the control plane machines in Azure {#installation-creating-azure-control-plane_installing-azure-user-infra}

To form the control plane for your cluster on Microsoft Azure, you must deploy control plane machines by using the Azure Resource Manager (ARM) template.

:::note

By default, Microsoft Azure places control plane machines and compute machines in a pre-set availability zone. You can manually set an availability zone for a compute node or control plane node. To do this, modify a vendor’s Azure Resource Manager (ARM) template by specifying each of your availability zones in the `zones` parameter of the virtual machine resource.

:::

If you do not use the provided ARM template to create your control plane machines, you must review the provided information and manually create the infrastructure. If your cluster does not initialize correctly, consider contacting Red Hat support with your installation logs.

**Prerequisites**

- Create the bootstrap machine.

**Procedure**

1. Copy the template from the **ARM template for control plane machines** section of this topic and save it as `05_masters.json` in your cluster’s installation directory. This template describes the control plane machines that your cluster requires.
2. Export the following variable needed by the control plane machine deployment:
   ```terminal
   $ export MASTER_IGNITION=`cat <installation_directory>/master.ign | base64 | tr -d '\n'`
   ```
3. Create the deployment by using the `az` CLI:
   ```terminal
   $ az deployment group create -g ${RESOURCE_GROUP} \
     --template-file "<installation_directory>/05_masters.json" \
     --parameters masterIgnition="${MASTER_IGNITION}" \
     --parameters baseName="${INFRA_ID}" \
     --parameters masterVMSize="Standard_D8s_v3"
   ```

   where:

   <dl>
   <dt><code>masterIgnition</code></dt>
   <dd>Specifies the Ignition content for the control plane nodes.</dd>
   <dt><code>baseName</code></dt>
   <dd>Specifies the base name to be used in resource names; this is usually the cluster’s infrastructure ID.</dd>
   <dt><code>masterVMSize</code></dt>
   <dd>Specifies the size of the Control Plane VM. Use a VM size compatible with your specified architecture. If this value is not defined, the default value from the template is set. This parameter is optional.</dd>
   </dl>

### ARM template for control plane machines {#installation-arm-control-plane_installing-azure-user-infra}

Use the `05_masters.json` Azure Resource Manager (ARM) template to deploy the control plane machines for your OpenShift Container Platform cluster.

<details>
<summary>`05_masters.json` ARM template</summary>

```json

{
  "$schema" : "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion" : "1.0.0.0",
  "parameters" : {
    "baseName" : {
      "type" : "string",
      "minLength" : 1,
      "metadata" : {
        "description" : "Base name to be used in resource names (usually the cluster's Infra ID)"
      }
    },
    "vnetBaseName": {
      "type": "string",
      "defaultValue": "",
      "metadata" : {
        "description" : "The specific customer vnet's base name (optional)"
      }
    },
    "masterIgnition" : {
      "type" : "string",
      "metadata" : {
        "description" : "Ignition content for the master nodes"
      }
    },
    "numberOfMasters" : {
      "type" : "int",
      "defaultValue" : 3,
      "minValue" : 2,
      "maxValue" : 30,
      "metadata" : {
        "description" : "Number of OpenShift masters to deploy"
      }
    },
    "sshKeyData" : {
      "type" : "securestring",
      "defaultValue" : "Unused",
      "metadata" : {
        "description" : "Unused"
      }
    },
    "privateDNSZoneName" : {
      "type" : "string",
      "defaultValue" : "",
      "metadata" : {
        "description" : "unused"
      }
    },
    "masterVMSize" : {
      "type" : "string",
      "defaultValue" : "Standard_D8s_v3",
      "metadata" : {
        "description" : "The size of the Master Virtual Machines"
      }
    },
    "diskSizeGB" : {
      "type" : "int",
      "defaultValue" : 1024,
      "metadata" : {
        "description" : "Size of the Master VM OS disk, in GB"
      }
    },
    "hyperVGen": {
      "type": "string",
      "metadata": {
        "description": "VM generation image to use"
      },
      "defaultValue": "V2",
      "allowedValues": [
        "V1",
        "V2"
      ]
    }
  },
  "variables" : {
    "location" : "[resourceGroup().location]",
    "virtualNetworkName" : "[concat(if(not(empty(parameters('vnetBaseName'))), parameters('vnetBaseName'), parameters('baseName')), '-vnet')]",
    "virtualNetworkID" : "[resourceId('Microsoft.Network/virtualNetworks', variables('virtualNetworkName'))]",
    "masterSubnetName" : "[concat(if(not(empty(parameters('vnetBaseName'))), parameters('vnetBaseName'), parameters('baseName')), '-master-subnet')]",
    "masterSubnetRef" : "[concat(variables('virtualNetworkID'), '/subnets/', variables('masterSubnetName'))]",
    "masterLoadBalancerName" : "[parameters('baseName')]",
    "internalLoadBalancerName" : "[concat(parameters('baseName'), '-internal-lb')]",
    "sshKeyPath" : "/home/core/.ssh/authorized_keys",
    "identityName" : "[concat(parameters('baseName'), '-identity')]",
    "galleryName": "[concat('gallery_', replace(parameters('baseName'), '-', '_'))]",
    "imageName" : "[concat(parameters('baseName'), if(equals(parameters('hyperVGen'), 'V2'), '-gen2', ''))]",
    "copy" : [
      {
        "name" : "vmNames",
        "count" :  "[parameters('numberOfMasters')]",
        "input" : "[concat(parameters('baseName'), '-master-', copyIndex('vmNames'))]"
      }
    ]
  },
  "resources" : [
    {
      "apiVersion" : "2018-06-01",
      "type" : "Microsoft.Network/networkInterfaces",
      "copy" : {
        "name" : "nicCopy",
        "count" : "[length(variables('vmNames'))]"
      },
      "name" : "[concat(variables('vmNames')[copyIndex()], '-nic')]",
      "location" : "[variables('location')]",
      "properties" : {
        "ipConfigurations" : [
          {
            "name" : "pipConfig",
            "properties" : {
              "privateIPAllocationMethod" : "Dynamic",
              "subnet" : {
                "id" : "[variables('masterSubnetRef')]"
              },
              "loadBalancerBackendAddressPools" : [
                {
                  "id" : "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/loadBalancers/', variables('masterLoadBalancerName'), '/backendAddressPools/', variables('masterLoadBalancerName'))]"
                },
                {
                  "id" : "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/loadBalancers/', variables('internalLoadBalancerName'), '/backendAddressPools/internal-lb-backend')]"
                }
              ]
            }
          }
        ]
      }
    },
    {
      "apiVersion" : "2018-06-01",
      "type" : "Microsoft.Compute/virtualMachines",
      "copy" : {
        "name" : "vmCopy",
        "count" : "[length(variables('vmNames'))]"
      },
      "name" : "[variables('vmNames')[copyIndex()]]",
      "location" : "[variables('location')]",
      "identity" : {
        "type" : "userAssigned",
        "userAssignedIdentities" : {
          "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/', variables('identityName'))]" : {}
        }
      },
      "dependsOn" : [
        "[concat('Microsoft.Network/networkInterfaces/', concat(variables('vmNames')[copyIndex()], '-nic'))]"
      ],
      "properties" : {
        "hardwareProfile" : {
          "vmSize" : "[parameters('masterVMSize')]"
        },
        "osProfile" : {
          "computerName" : "[variables('vmNames')[copyIndex()]]",
          "adminUsername" : "core",
          "adminPassword" : "NotActuallyApplied!",
          "customData" : "[parameters('masterIgnition')]",
          "linuxConfiguration" : {
            "disablePasswordAuthentication" : false
          }
        },
        "storageProfile" : {
          "imageReference": {
            "id": "[resourceId('Microsoft.Compute/galleries/images', variables('galleryName'), variables('imageName'))]"
          },
          "osDisk" : {
            "name": "[concat(variables('vmNames')[copyIndex()], '_OSDisk')]",
            "osType" : "Linux",
            "createOption" : "FromImage",
            "caching": "ReadOnly",
            "writeAcceleratorEnabled": false,
            "managedDisk": {
              "storageAccountType": "Premium_LRS"
            },
            "diskSizeGB" : "[parameters('diskSizeGB')]"
          }
        },
        "networkProfile" : {
          "networkInterfaces" : [
            {
              "id" : "[resourceId('Microsoft.Network/networkInterfaces', concat(variables('vmNames')[copyIndex()], '-nic'))]",
              "properties": {
                "primary": false
              }
            }
          ]
        }
      }
    }
  ]
}

```

</details>

## Wait for bootstrap completion and remove bootstrap resources in Azure {#installation-azure-user-infra-wait-for-bootstrap_installing-azure-user-infra}

To complete cluster initialization on Microsoft Azure, you can wait for the bootstrap process to finish and then delete bootstrap resources.

**Prerequisites**

- Create the control plane machines.

**Procedure**

1. Change to the directory that contains the installation program and run the following command:
   ```terminal
   $ ./openshift-install wait-for bootstrap-complete --dir <installation_directory> \
       --log-level info
   ```

   where:

   <dl>
   <dt><code>&lt;installation_directory&gt;</code></dt>
   <dd>Specifies the path to the directory that you stored the installation files in.</dd>
   <dt><code>--log-level info</code></dt>
   <dd>Specifies the installation details. Specify <code>warn</code>, <code>debug</code>, or <code>error</code> instead of <code>info</code> to view different installation details. If the command exits without a <code>FATAL</code> warning, your production control plane has initialized.</dd>
   </dl>
2. Delete the bootstrap resources:
   ```terminal
   $ az network nsg rule delete -g ${RESOURCE_GROUP} --nsg-name ${INFRA_ID}-nsg --name bootstrap_ssh_in
   ```

   ```terminal
   $ az vm stop -g ${RESOURCE_GROUP} --name ${INFRA_ID}-bootstrap
   ```

   ```terminal
   $ az vm deallocate -g ${RESOURCE_GROUP} --name ${INFRA_ID}-bootstrap
   ```

   ```terminal
   $ az vm delete -g ${RESOURCE_GROUP} --name ${INFRA_ID}-bootstrap --yes
   ```

   ```terminal
   $ az disk delete -g ${RESOURCE_GROUP} --name ${INFRA_ID}-bootstrap_OSDisk --no-wait --yes
   ```

   ```terminal
   $ az network nic delete -g ${RESOURCE_GROUP} --name ${INFRA_ID}-bootstrap-nic --no-wait
   ```

   ```terminal
   $ az storage blob delete --account-key ${ACCOUNT_KEY} --account-name ${CLUSTER_NAME}sa --container-name files --name bootstrap.ign
   ```

   ```terminal
   $ az network public-ip delete -g ${RESOURCE_GROUP} --name ${INFRA_ID}-bootstrap-ssh-pip
   ```

   :::note

   If you do not delete the bootstrap server, installation may not succeed due to API traffic being routed to the bootstrap server.

   :::

## Creating additional worker machines in Azure {#installation-creating-azure-worker_installing-azure-user-infra}

To add compute capacity on Microsoft Azure, you can create worker machines in Microsoft Azure for your cluster to use by launching individual instances discretely or by automated processes outside the cluster, such as auto scaling groups.

You can also take advantage of the built-in cluster scaling mechanisms and the machine API in OpenShift Container Platform.

:::note

If you are installing a three-node cluster, skip this step. A three-node cluster consists of three control plane machines, which also act as compute machines.

:::

In this example, you manually launch one instance by using the Azure Resource Manager (ARM) template. Additional instances can be launched by including additional resources of type `06_workers.json` in the file.

:::note

By default, Microsoft Azure places control plane machines and compute machines in a pre-set availability zone. You can manually set an availability zone for a compute node or control plane node. To do this, modify a vendor’s ARM template by specifying each of your availability zones in the `zones` parameter of the virtual machine resource.

:::

If you do not use the provided ARM template to create your control plane machines, you must review the provided information and manually create the infrastructure. If your cluster does not initialize correctly, consider contacting Red Hat support with your installation logs.

**Procedure**

1. Copy the template from the **ARM template for worker machines** section of this topic and save it as `06_workers.json` in your cluster’s installation directory. This template describes the worker machines that your cluster requires.
2. Export the following variable needed by the worker machine deployment:
   ```terminal
   $ export WORKER_IGNITION=`cat <installation_directory>/worker.ign | base64 | tr -d '\n'`
   ```
3. Create the deployment by using the `az` CLI:
   ```terminal
   $ az deployment group create -g ${RESOURCE_GROUP} \
     --template-file "<installation_directory>/06_workers.json" \
     --parameters workerIgnition="${WORKER_IGNITION}" \
     --parameters baseName="${INFRA_ID}" \
     --parameters nodeVMSize="Standard_D4s_v3"
   ```

   where:

   <dl>
   <dt><code>workerIgnition</code></dt>
   <dd>Specifies the Ignition content for the worker nodes.</dd>
   <dt><code>baseName</code></dt>
   <dd>Specifies the base name to be used in resource names; this is usually the cluster’s infrastructure ID.</dd>
   <dt><code>nodeVMSize</code></dt>
   <dd>Specifies the size of the compute node VM. Use a VM size compatible with your specified architecture. If this value is not defined, the default value from the template is set. This parameter is optional.</dd>
   </dl>

### ARM template for worker machines {#installation-arm-worker_installing-azure-user-infra}

Use the `06_workers.json` Azure Resource Manager (ARM) template to deploy worker machines for your OpenShift Container Platform cluster.

<details>
<summary>`06_workers.json` ARM template</summary>

```json

{
  "$schema" : "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion" : "1.0.0.0",
  "parameters" : {
    "baseName" : {
      "type" : "string",
      "minLength" : 1,
      "metadata" : {
        "description" : "Base name to be used in resource names (usually the cluster's Infra ID)"
      }
    },
    "vnetBaseName": {
      "type": "string",
      "defaultValue": "",
      "metadata" : {
        "description" : "The specific customer vnet's base name (optional)"
      }
    },
    "workerIgnition" : {
      "type" : "string",
      "metadata" : {
        "description" : "Ignition content for the worker nodes"
      }
    },
    "numberOfNodes" : {
      "type" : "int",
      "defaultValue" : 3,
      "minValue" : 2,
      "maxValue" : 30,
      "metadata" : {
        "description" : "Number of OpenShift compute nodes to deploy"
      }
    },
    "sshKeyData" : {
      "type" : "securestring",
      "defaultValue" : "Unused",
      "metadata" : {
        "description" : "Unused"
      }
    },
    "nodeVMSize" : {
      "type" : "string",
      "defaultValue" : "Standard_D4s_v3",
      "metadata" : {
        "description" : "The size of the each Node Virtual Machine"
      }
    },
    "hyperVGen": {
      "type": "string",
      "metadata": {
        "description": "VM generation image to use"
      },
      "defaultValue": "V2",
      "allowedValues": [
        "V1",
        "V2"
      ]
    }
  },
  "variables" : {
    "location" : "[resourceGroup().location]",
    "virtualNetworkName" : "[concat(if(not(empty(parameters('vnetBaseName'))), parameters('vnetBaseName'), parameters('baseName')), '-vnet')]",
    "virtualNetworkID" : "[resourceId('Microsoft.Network/virtualNetworks', variables('virtualNetworkName'))]",
    "nodeSubnetName" : "[concat(if(not(empty(parameters('vnetBaseName'))), parameters('vnetBaseName'), parameters('baseName')), '-worker-subnet')]",
    "nodeSubnetRef" : "[concat(variables('virtualNetworkID'), '/subnets/', variables('nodeSubnetName'))]",
    "infraLoadBalancerName" : "[parameters('baseName')]",
    "sshKeyPath" : "/home/capi/.ssh/authorized_keys",
    "identityName" : "[concat(parameters('baseName'), '-identity')]",
    "galleryName": "[concat('gallery_', replace(parameters('baseName'), '-', '_'))]",
    "imageName" : "[concat(parameters('baseName'), if(equals(parameters('hyperVGen'), 'V2'), '-gen2', ''))]",
    "copy" : [
      {
        "name" : "vmNames",
        "count" :  "[parameters('numberOfNodes')]",
        "input" : "[concat(parameters('baseName'), '-worker-', variables('location'), '-', copyIndex('vmNames', 1))]"
      }
    ]
  },
  "resources" : [
    {
      "apiVersion" : "2019-05-01",
      "name" : "[concat('node', copyIndex())]",
      "type" : "Microsoft.Resources/deployments",
      "copy" : {
        "name" : "nodeCopy",
        "count" : "[length(variables('vmNames'))]"
      },
      "properties" : {
        "mode" : "Incremental",
        "template" : {
          "$schema" : "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion" : "1.0.0.0",
          "resources" : [
            {
              "apiVersion" : "2018-06-01",
              "type" : "Microsoft.Network/networkInterfaces",
              "name" : "[concat(variables('vmNames')[copyIndex()], '-nic')]",
              "location" : "[variables('location')]",
              "properties" : {
                "ipConfigurations" : [
                  {
                    "name" : "pipConfig",
                    "properties" : {
                      "privateIPAllocationMethod" : "Dynamic",
                      "subnet" : {
                        "id" : "[variables('nodeSubnetRef')]"
                      }
                    }
                  }
                ]
              }
            },
            {
              "apiVersion" : "2018-06-01",
              "type" : "Microsoft.Compute/virtualMachines",
              "name" : "[variables('vmNames')[copyIndex()]]",
              "location" : "[variables('location')]",
              "tags" : {
                "kubernetes.io-cluster-ffranzupi": "owned"
              },
              "identity" : {
                "type" : "userAssigned",
                "userAssignedIdentities" : {
                  "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/', variables('identityName'))]" : {}
                }
              },
              "dependsOn" : [
                "[concat('Microsoft.Network/networkInterfaces/', concat(variables('vmNames')[copyIndex()], '-nic'))]"
              ],
              "properties" : {
                "hardwareProfile" : {
                  "vmSize" : "[parameters('nodeVMSize')]"
                },
                "osProfile" : {
                  "computerName" : "[variables('vmNames')[copyIndex()]]",
                  "adminUsername" : "capi",
                  "adminPassword" : "NotActuallyApplied!",
                  "customData" : "[parameters('workerIgnition')]",
                  "linuxConfiguration" : {
                    "disablePasswordAuthentication" : false
                  }
                },
                "storageProfile" : {
                  "imageReference": {
                    "id": "[resourceId('Microsoft.Compute/galleries/images', variables('galleryName'), variables('imageName'))]"
                  },
                  "osDisk" : {
                    "name": "[concat(variables('vmNames')[copyIndex()],'_OSDisk')]",
                    "osType" : "Linux",
                    "createOption" : "FromImage",
                    "managedDisk": {
                      "storageAccountType": "Premium_LRS"
                    },
                    "diskSizeGB": 128
                  }
                },
                "networkProfile" : {
                  "networkInterfaces" : [
                    {
                      "id" : "[resourceId('Microsoft.Network/networkInterfaces', concat(variables('vmNames')[copyIndex()], '-nic'))]",
                      "properties": {
                        "primary": true
                      }
                    }
                  ]
                }
              }
            }
          ]
        }
      }
    }
  ]
}

```

</details>

## Installing the OpenShift CLI on Linux {#cli-installing-cli-linux_installing-azure-user-infra}

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

## Installing the OpenShift CLI on Windows {#cli-installing-cli-windows_installing-azure-user-infra}

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

## Installing the OpenShift CLI on macOS {#cli-installing-cli-macos_installing-azure-user-infra}

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

## Logging in to the cluster by using the CLI {#cli-logging-in-kubeadmin_installing-azure-user-infra}

To log in to your cluster as the default system user, export the `kubeconfig` file. This configuration enables the CLI to authenticate and connect to the specific API server created during OpenShift Container Platform installation.

The `kubeconfig` file is specific to a cluster and OpenShift Container Platform generates it during installation.

**Prerequisites**

- You deployed an OpenShift Container Platform cluster.
- You installed the OpenShift CLI (`oc`).

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

## Approving the certificate signing requests for your machines {#installation-approve-csrs_installing-azure-user-infra}

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

## Adding the Ingress DNS records {#installation-azure-create-ingress-dns-records_installing-azure-user-infra}

If you removed the DNS Zone configuration when creating Kubernetes manifests and generating Ignition configs, you must manually create DNS records that point at the Ingress load balancer. You can create either a wildcard `*.apps.{baseDomain}.` or specific records.

You can use A, CNAME, and other records per your requirements.

**Prerequisites**

- You deployed an OpenShift Container Platform cluster on Microsoft Azure by using infrastructure that you provisioned.
- Install the OpenShift CLI (`oc`).
- Install or update the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-yum?view=azure-cli-latest).

**Procedure**

1. Confirm the Ingress router has created a load balancer and populated the `EXTERNAL-IP` field:
   ```terminal
   $ oc -n openshift-ingress get service router-default
   ```

   ```terminal title="Example output"
   NAME             TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                      AGE
   router-default   LoadBalancer   172.30.20.10   35.130.120.110   80:32288/TCP,443:31215/TCP   20
   ```
2. Export the Ingress router IP as a variable:
   ```terminal
   $ export PUBLIC_IP_ROUTER=`oc -n openshift-ingress get service router-default --no-headers | awk '{print $4}'`
   ```
3. Add a `*.apps` record to the public DNS zone.
   1. If you are adding this cluster to a new public zone, run:
      ```terminal
      $ az network dns record-set a add-record -g ${BASE_DOMAIN_RESOURCE_GROUP} -z ${CLUSTER_NAME}.${BASE_DOMAIN} -n *.apps -a ${PUBLIC_IP_ROUTER} --ttl 300
      ```
   2. If you are adding this cluster to an already existing public zone, run:
      ```terminal
      $ az network dns record-set a add-record -g ${BASE_DOMAIN_RESOURCE_GROUP} -z ${BASE_DOMAIN} -n *.apps.${CLUSTER_NAME} -a ${PUBLIC_IP_ROUTER} --ttl 300
      ```
4. Add a `*.apps` record to the private DNS zone:
   1. Create a `*.apps` record by using the following command:
      ```terminal
      $ az network private-dns record-set a create -g ${RESOURCE_GROUP} -z ${CLUSTER_NAME}.${BASE_DOMAIN} -n *.apps --ttl 300
      ```
   2. Add the `*.apps` record to the private DNS zone by using the following command:
      ```terminal
      $ az network private-dns record-set a add-record -g ${RESOURCE_GROUP} -z ${CLUSTER_NAME}.${BASE_DOMAIN} -n *.apps -a ${PUBLIC_IP_ROUTER}
      ```

If you prefer to add explicit domains instead of using a wildcard, you can create entries for each of the cluster’s current routes:

```terminal
$ oc get --all-namespaces -o jsonpath='{range .items[*]}{range .status.ingress[*]}{.host}{"\n"}{end}{end}' routes
```

.Example output

```terminal
oauth-openshift.apps.cluster.basedomain.com
console-openshift-console.apps.cluster.basedomain.com
downloads-openshift-console.apps.cluster.basedomain.com
alertmanager-main-openshift-monitoring.apps.cluster.basedomain.com
prometheus-k8s-openshift-monitoring.apps.cluster.basedomain.com
```

## Completing an Azure installation on user-provisioned infrastructure {#installation-azure-user-infra-completing_installing-azure-user-infra}

After you start the OpenShift Container Platform installation on Microsoft Azure user-provisioned infrastructure, you can monitor cluster events with the installation program until the cluster is ready.

**Prerequisites**

- Deploy the bootstrap machine for an OpenShift Container Platform cluster on user-provisioned Azure infrastructure.
- Install the `oc` CLI and log in.

**Procedure**

- Complete the cluster installation:
  ```terminal
  $ ./openshift-install --dir <installation_directory> wait-for install-complete
  ```

  For `<installation_directory>`, specify the path to the directory that you stored the installation files in.

  ```terminal title="Example output"
  INFO Waiting up to 30m0s for the cluster to initialize...
  ```

  :::warning

  - The Ignition config files that the installation program generates contain certificates that expire after 24 hours, which are then renewed at that time. If the cluster is shut down before renewing the certificates and the cluster is later restarted after the 24 hours have elapsed, the cluster automatically recovers the expired certificates. The exception is that you must manually approve the pending `node-bootstrapper` certificate signing requests (CSRs) to recover kubelet certificates. See the documentation for *Recovering from expired control plane certificates* for more information.
  - It is recommended that you use Ignition config files within 12 hours after they are generated because the 24-hour certificate rotates from 16 to 22 hours after the cluster is installed. By using the Ignition config files within 12 hours, you can avoid installation failure if the certificate update runs during installation.

  :::

## Telemetry access for OpenShift Container Platform {#cluster-telemetry_installing-azure-user-infra}

To provide metrics about cluster health and the success of updates, the Telemetry service requires internet access. When connected, this service runs automatically by default and registers your cluster to [OpenShift Cluster Manager](https://console.redhat.com/openshift).

After you confirm that your [OpenShift Cluster Manager](https://console.redhat.com/openshift) inventory is correct, either maintained automatically by Telemetry or manually by using OpenShift Cluster Manager,use subscription watch to track your OpenShift Container Platform subscriptions at the account or multi-cluster level. For more information about subscription watch, see "Data Gathered and Used by Red Hat’s subscription services" in the *Additional resources* section.

**Additional resources**

- [About remote health monitoring](/docs/support/remote_health_monitoring/about-remote-health-monitoring#about-remote-health-monitoring)

**Additional resources**

- [Azure Resource Manager templates overview (Azure documentation)](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/overview)
