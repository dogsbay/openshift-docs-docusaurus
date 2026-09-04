---
title: Configuring an IBM Cloud account
sidebar_position: 2
---

# Configuring an IBM Cloud account {#installing-ibm-cloud-account-power-vs}

<a id="installing-ibm-cloud-account-power-vs"></a>

To install OpenShift Container Platform on IBM Power(R) Virtual Server, you must configure an IBM Cloud(R) account with the correct quotas, DNS resolution, and IAM policies.

## Prerequisites {#_prerequisites}

- You have an IBM Cloud(R) account with a subscription. You cannot install OpenShift Container Platform on a free or on a trial IBM Cloud(R) account.

## Quotas and limits on IBM Power Virtual Server {#quotas-and-limits-ibm-power-vs_installing-ibm-cloud-account-power-vs}

The OpenShift Container Platform cluster uses several IBM Cloud(R) and IBM Power(R) Virtual Server components. Default quotas and limits affect your ability to install clusters, so you might need to request additional resources for your IBM Cloud(R) account depending on your configuration, region, or number of clusters.

### Virtual private cloud {#_virtual_private_cloud}

Each OpenShift Container Platform cluster creates its own Virtual Private Cloud (VPC). The default quota of VPC instances per region is 10. If you have 10 VPC instances created, you must increase your quota before attempting an installation.

### Application load balancer {#_application_load_balancer}

By default, each cluster creates two application load balancers (ALBs):

- Internal load balancer for the control plane API server
- External load balancer for the control plane API server

You can create additional `LoadBalancer` service objects to create additional ALBs. The default quota of VPC ALBs are 50 per region. To have more than 50 ALBs, you must increase this quota.

VPC ALBs are supported. Classic ALBs are not supported for IBM Power(R) Virtual Server.

### Transit gateways {#_transit_gateways}

Each OpenShift Container Platform cluster creates its own transit gateway to enable communication with a VPC. The default quota of transit gateways per IBM Cloud(R) account is 10. If you have 10 transit gateways created, you must increase your quota before attempting an installation.

### Dynamic host configuration protocol (DHCP) service {#_dynamic_host_configuration_protocol_dhcp_service}

There is a limit of one Dynamic Host Configuration Protocol (DHCP) service per IBM Power(R) Virtual Server instance.

### Virtual server instances {#_virtual_server_instances}

By default, a cluster creates server instances with the following resources:

- 0.5 CPUs
- 32 GB RAM
- System Type: `s922`
- Processor Type: `uncapped`, `shared`
- Storage Tier: `Tier-3`

The installation program creates the following nodes:

- One bootstrap machine, which the installation process removes after the installation is complete
- Three control plane nodes
- Three compute nodes

**Additional resources**

- [Quotas and service limits](https://cloud.ibm.com/docs/vpc?topic=vpc-quotas)
- [Creating an IBM Power Virtual Server](https://cloud.ibm.com/docs/power-iaas?topic=power-iaas-creating-power-virtual-server)

## Configuring DNS resolution {#configuring-dns-resolution-powervs_installing-ibm-cloud-account-power-vs}

DNS resolution configuration for OpenShift Container Platform on IBM Power(R) Virtual Server depends on whether you are installing a public or private cluster. Public clusters use IBM Cloud(R) Internet Services (CIS) and private clusters use IBM Cloud(R) DNS Services.

- If you are installing a public cluster, you use IBM Cloud(R) Internet Services (CIS).
- If you are installing a private cluster, you use IBM Cloud(R) DNS Services (DNS Services).

## Using IBM Cloud Internet Services for DNS resolution {#installation-cis-ibm-cloud_installing-ibm-cloud-account-power-vs}

The installation program uses IBM Cloud(R) Internet Services (CIS) to configure cluster DNS resolution and provide name lookup for a public cluster.

:::note

This offering does not support IPv6, so dual stack or IPv6 environments are not possible.

:::

You must create a domain zone in CIS in the same account as your cluster. You must also ensure the zone is authoritative for the domain. You can do this using a root domain or subdomain.

**Prerequisites**

- You have installed the IBM Cloud(R) CLI. For more information, see "IBM Cloud(R) CLI".
- You have an existing domain and registrar. For more information, see the "IBM(R) DNS documentation".

**Procedure**

1. Create a CIS instance to use with your cluster:
   1. Install the CIS plugin:
      ```terminal
      $ ibmcloud plugin install cis
      ```
   2. Log in to IBM Cloud(R) by using the CLI:
      ```terminal
      $ ibmcloud login
      ```
   3. Create the CIS instance:
      ```terminal
      $ ibmcloud cis instance-create <instance_name> standard-next
      ```

      At a minimum, you require a `Standard Next` plan for CIS to manage the cluster subdomain and its DNS records.

      :::note

      After you have configured your registrar or DNS provider, it can take up to 24 hours for the changes to take effect.

      :::
2. Connect an existing domain to your CIS instance:
   1. Set the context instance for CIS:
      ```terminal
      $ ibmcloud cis instance-set <instance_CRN>
      ```

      Replace `<instance_CRN>` with the instance CRN (Cloud Resource Name). For example: `ibmcloud cis instance-set crn:v1:bluemix:public:power-iaas:osa21:a/65b64c1f1c29460d8c2e4bbfbd893c2c:c09233ac-48a5-4ccb-a051-d1cfb3fc7eb5::`
   2. Add the domain for CIS:
      ```terminal
      $ ibmcloud cis domain-add <domain_name>
      ```

      Replace `<domain_name>` with the fully qualified domain name. You can use either the root domain or subdomain value as the domain name, depending on which you plan to configure.

      :::note

      A root domain uses the form `openshiftcorp.com`. A subdomain uses the form `clusters.openshiftcorp.com`.

      :::
3. Open the CIS web console, navigate to the **Overview** page, and note your CIS name servers. These name servers are used in the next step. For more information, see "CIS web console".
4. Configure the name servers for your domains or subdomains at the domain’s registrar or DNS provider. For more information, see the IBM Cloud(R) documentation for "Configuring name servers".

## IBM Cloud IAM policies and API key {#installation-ibm-cloud-iam-policies-api-key_installing-ibm-cloud-account-power-vs}

To install OpenShift Container Platform into your IBM Cloud(R) account, the installation program requires an IAM API key, which provides authentication and authorization to access IBM Cloud(R) service APIs. You can use an existing IAM API key that contains the required policies or create a new one.

For an IBM Cloud(R) IAM overview, see the "IBM Cloud(R) IAM overview" documentation.

### Prerequisite permissions {#pre-requisite-permissions-ibm-cloud_installing-ibm-cloud-account-power-vs}

**Prerequisite permissions**

| Role | Access |
| --- | --- |
| Viewer, Operator, Editor, Administrator, Reader, Writer, Manager | Internet Services service in <resource_group> resource group |
| Viewer, Operator, Editor, Administrator, User API key creator, Service ID creator | IAM Identity Service service |
| Viewer, Operator, Administrator, Editor, Reader, Writer, Manager, Console Administrator | VPC Infrastructure Services service in <resource_group> resource group |
| Viewer | Resource Group: Access to view the resource group itself. The resource type should equal `Resource group`, with a value of <your_resource_group_name>. |

### Cluster-creation permissions {#cluster-creation-permissions-ibm-cloud_installing-ibm-cloud-account-power-vs}

**Cluster-creation permissions**

| Role | Access |
| --- | --- |
| Viewer | <resource_group> (Resource Group Created for Your Team) |
| Viewer, Operator, Editor, Reader, Writer, Manager | All Identity and IAM enabled services in Default resource group |
| Viewer, Reader | Internet Services service |
| Viewer, Operator, Reader, Writer, Manager, Content Reader, Object Reader, Object Writer, Editor | Cloud Object Storage service |
| Viewer | Default resource group: The resource type should equal `Resource group`, with a value of `Default`. If your account administrator changed your account’s default resource group to something other than Default, use that value instead. |
| Viewer, Operator, Editor, Reader, Manager | Workspace for IBM Power(R) Virtual Server service in <resource_group> resource group |
| Viewer, Operator, Editor, Reader, Writer, Manager, Administrator | Internet Services service in <resource_group> resource group: CIS functional scope string equals reliability |
| Viewer, Operator, Editor | Transit Gateway service |
| Viewer, Operator, Editor, Administrator, Reader, Writer, Manager, Console Administrator | VPC Infrastructure Services service <resource_group> resource group |

### Access policy assignment {#access-policy-assignment-ibm-cloud_installing-ibm-cloud-account-power-vs}

In IBM Cloud(R) IAM, access policies can be attached to different subjects:

- Access group (Recommended)
- Service ID
- User

:::note

The recommended method is to define IAM access policies in an access group. This helps organize all the access required for OpenShift Container Platform and enables you to onboard users and service IDs to this group. You can also assign access to users and service IDs directly, if desired.

For more information, see "Access groups" and "Users and service IDs".

:::

**Additional resources**

- [IBM Cloud(R) IAM overview](https://cloud.ibm.com/docs/account?topic=account-iamoverview)
- [Access groups (IBM Cloud(R) documentation)](https://cloud.ibm.com/docs/account?topic=account-groups)
- [Users and service IDs (IBM Cloud(R) documentation)](https://cloud.ibm.com/docs/account?topic=account-assign-access-resources)

### Creating an API key {#installation-ibm-cloud-creating-api-key_installing-ibm-cloud-account-power-vs}

You must create a user API key or a service ID API key for your IBM Cloud(R) account.

**Prerequisites**

- You have assigned the required access policies to your IBM Cloud(R) account.
- You have attached your IAM access policies to an access group, or other appropriate resource.

**Procedure**

- Create an API key, depending on how you defined your IAM access policies.
  For example, if you assigned your access policies to a user, you must create a user API key. If you assigned your access policies to a service ID, you must create a service ID API key. If your access policies are assigned to an access group, you can use either API key type. For more information on IBM Cloud(R) API keys, see "User API key", "Service ID API key", and "Understanding API keys".

**Additional resources**

- [User API key (IBM Cloud(R) documentation)](https://cloud.ibm.com/docs/account?topic=account-userapikey)
- [Service ID API key (IBM Cloud(R) documentation)](https://cloud.ibm.com/docs/account?topic=account-serviceidapikeys)
- [Understanding API keys (IBM Cloud(R) documentation)](https://cloud.ibm.com/docs/account?topic=account-manapikey&interface=ui)

## Supported IBM Power Virtual Server regions and zones {#installation-ibm-power-vs-regions_installing-ibm-cloud-account-power-vs}

When installing OpenShift Container Platform, you must choose a supported region or zone for your cloud provider deployment.

You can deploy an OpenShift Container Platform cluster to the following regions:

- `tor` (Toronto, Canada)
  - `tor01`
- `dal` (Dallas, USA)
  - `dal10`
  - `dal12`
- `eu-de` (Frankfurt, Germany)
  - `eu-de-1`
  - `eu-de-2`
- `lon` (London, UK)
  - `lon04`
  - `lon06`
- `mad` (Madrid, Spain)
  - `mad02`
  - `mad04`
- `osa` (Osaka, Japan)
  - `osa21`
- `sao` (Sao Paulo, Brazil)
  - `sao01`
  - `sao04`
- `syd` (Sydney, Australia)
  - `syd04`
  - `syd05`
- `wdc` (Washington DC, USA)
  - `wdc06`
  - `wdc07`
- `us-east` (Washington DC, United States)
  - `us-east`
- `us-south` (Dallas, United States)
  - `us-south`

You might optionally specify the IBM Cloud(R) region in which the installation program creates any VPC components.

:::note

If you do not specify the region, the installation program selects the region closest to IBM Power Virtual Server zone you are deploying to.

:::

IBM Cloud(R) supports the following regions:

- `us-east`
- `us-south`
- `eu-de`
- `eu-es`
- `eu-gb`
- `jp-osa`
- `au-syd`
- `br-sao`
- `ca-tor`
- `jp-tok`

**Additional resources**

- [Creating an IBM Power(R) Virtual Server workspace](/docs/installing/installing_ibm_powervs/creating-ibm-power-vs-workspace#creating-ibm-power-vs-workspace)
