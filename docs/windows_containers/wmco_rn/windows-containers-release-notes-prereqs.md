---
title: Windows Machine Config Operator prerequisites
sidebar_position: 3
---

# Windows Machine Config Operator prerequisites {#windows-containers-release-notes-prereqs}

<a id="windows-containers-release-notes-prereqs"></a>

You can review the following information for details on the supported platform versions, Windows Server versions, and networking configurations for the Windows Machine Config Operator (WMCO). See the vSphere documentation for any information that is relevant to only that platform.

## WMCO supported installation method {#wmco-prerequisites-supported-install_windows-containers-release-notes-prereqs}

The WMCO fully supports installing Windows nodes into installer-provisioned infrastructure (IPI) clusters. This is the preferred OpenShift Container Platform installation method.

For user-provisioned infrastructure (UPI) clusters, the WMCO supports installing Windows nodes only into a UPI cluster installed with the `platform: none` field set in the `install-config.yaml` file (bare-metal or provider-agnostic) and only for the [BYOH (Bring Your Own Host)](/docs/windows_containers/byoh-windows-instance#byoh-windows-instance) use case. UPI is not supported for any other platform.

## WMCO supported platforms and Windows Server versions {#wmco-prerequisites-supported_windows-containers-release-notes-prereqs}

The following table lists the [Windows Server versions](https://docs.microsoft.com/en-us/windows/release-health/windows-server-release-info) that are supported by WMCO 10.20.0, based on the applicable platform. Windows Server versions not listed are not supported and attempting to use them will cause errors. To prevent these errors, use only an appropriate version for your platform.

<table>
<thead>
<tr>
  <th>Platform</th>
  <th>Supported Windows Server version</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Amazon Web Services (AWS)</td>
  <td><ul><li>Windows Server 2025, OS Build <a href="https://support.microsoft.com/en-us/topic/may-12-2026-kb5087539-os-build-26100-32860-fe3fd635-23fc-41bd-b7a7-00e57c1c4f91">10.0.26100</a> or later</li><li>Windows Server 2022, OS Build <a href="https://support.microsoft.com/en-us/topic/april-25-2022-kb5012637-os-build-20348-681-preview-2233d69c-d4a5-4be9-8c24-04a450861a8d">20348.681</a> or later <sup>[1]</sup></li><li>Windows Server 2019, version 1809</li></ul></td>
</tr>
<tr>
  <td>Microsoft Azure</td>
  <td><ul><li>Windows Server 2025, OS Build <a href="https://support.microsoft.com/en-us/topic/may-12-2026-kb5087539-os-build-26100-32860-fe3fd635-23fc-41bd-b7a7-00e57c1c4f91">10.0.26100</a> or later</li><li>Windows Server 2022, OS Build <a href="https://support.microsoft.com/en-us/topic/april-25-2022-kb5012637-os-build-20348-681-preview-2233d69c-d4a5-4be9-8c24-04a450861a8d">20348.681</a> or later</li><li>Windows Server 2019, version 1809</li></ul></td>
</tr>
<tr>
  <td>VMware vSphere</td>
  <td><ul><li>Windows Server 2025, OS Build <a href="https://support.microsoft.com/en-us/topic/may-12-2026-kb5087539-os-build-26100-32860-fe3fd635-23fc-41bd-b7a7-00e57c1c4f91">10.0.26100</a> or later</li><li>Windows Server 2022, OS Build <a href="https://support.microsoft.com/en-us/topic/april-25-2022-kb5012637-os-build-20348-681-preview-2233d69c-d4a5-4be9-8c24-04a450861a8d">20348.681</a> or later</li></ul></td>
</tr>
<tr>
  <td>Google Cloud</td>
  <td><ul><li>Windows Server 2025, OS Build <a href="https://support.microsoft.com/en-us/topic/may-12-2026-kb5087539-os-build-26100-32860-fe3fd635-23fc-41bd-b7a7-00e57c1c4f91">10.0.26100</a> or later</li><li>Windows Server 2022, OS Build <a href="https://support.microsoft.com/en-us/topic/april-25-2022-kb5012637-os-build-20348-681-preview-2233d69c-d4a5-4be9-8c24-04a450861a8d">20348.681</a> or later</li></ul></td>
</tr>
<tr>
  <td>Nutanix</td>
  <td><ul><li>Windows Server 2025, OS Build <a href="https://support.microsoft.com/en-us/topic/may-12-2026-kb5087539-os-build-26100-32860-fe3fd635-23fc-41bd-b7a7-00e57c1c4f91">10.0.26100</a> or later</li><li>Windows Server 2022, OS Build <a href="https://support.microsoft.com/en-us/topic/april-25-2022-kb5012637-os-build-20348-681-preview-2233d69c-d4a5-4be9-8c24-04a450861a8d">20348.681</a> or later</li></ul></td>
</tr>
<tr>
  <td>Bare metal or provider agnostic</td>
  <td><ul><li>Windows Server 2025, OS Build <a href="https://support.microsoft.com/en-us/topic/may-12-2026-kb5087539-os-build-26100-32860-fe3fd635-23fc-41bd-b7a7-00e57c1c4f91">10.0.26100</a> or later</li><li>Windows Server 2022, OS Build <a href="https://support.microsoft.com/en-us/topic/april-25-2022-kb5012637-os-build-20348-681-preview-2233d69c-d4a5-4be9-8c24-04a450861a8d">20348.681</a> or later</li></ul></td>
</tr>
</tbody>
</table>

1. For disconnected clusters, the Windows AMI must have the EC2LaunchV2 agent version 2.0.2107 or later installed. For more information, see "Install the latest version of EC2Launch v2 (AWS documentation)".

## Supported networking {#_supported_networking}

Hybrid networking with OVN-Kubernetes is the only supported networking configuration. See the additional resources below for more information on this functionality. The following tables outline the type of networking configuration and Windows Server versions to use based on your platform. You must specify the network configuration when you install the cluster.

:::note

- The WMCO does not support OVN-Kubernetes without hybrid networking or OpenShift SDN.
- Dual NIC is not supported on WMCO-managed Windows instances.

:::

**Platform networking support**

| Platform | Supported networking |
| --- | --- |
| Amazon Web Services (AWS) | Hybrid networking with OVN-Kubernetes |
| Microsoft Azure | Hybrid networking with OVN-Kubernetes |
| VMware vSphere | Hybrid networking with OVN-Kubernetes with a custom VXLAN port |
| Google Cloud | Hybrid networking with OVN-Kubernetes |
| Nutanix | Hybrid networking with OVN-Kubernetes |
| Bare metal or provider agnostic | Hybrid networking with OVN-Kubernetes |

**Hybrid OVN-Kubernetes Windows Server support**

<table>
<thead>
<tr>
  <th>Hybrid networking with OVN-Kubernetes</th>
  <th>Supported Windows Server version</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Default VXLAN port</td>
  <td><ul><li>Windows Server 2025, OS Build <a href="https://support.microsoft.com/en-us/topic/may-12-2026-kb5087539-os-build-26100-32860-fe3fd635-23fc-41bd-b7a7-00e57c1c4f91">10.0.26100</a> or later</li><li>Windows Server 2022, OS Build <a href="https://support.microsoft.com/en-us/topic/april-25-2022-kb5012637-os-build-20348-681-preview-2233d69c-d4a5-4be9-8c24-04a450861a8d">20348.681</a> or later</li><li>Windows Server 2019, version 1809</li></ul></td>
</tr>
<tr>
  <td>Custom VXLAN port</td>
  <td><ul><li>Windows Server 2025, OS Build <a href="https://support.microsoft.com/en-us/topic/may-12-2026-kb5087539-os-build-26100-32860-fe3fd635-23fc-41bd-b7a7-00e57c1c4f91">10.0.26100</a> or later</li><li>Windows Server 2022, OS Build <a href="https://support.microsoft.com/en-us/topic/april-25-2022-kb5012637-os-build-20348-681-preview-2233d69c-d4a5-4be9-8c24-04a450861a8d">20348.681</a> or later</li></ul></td>
</tr>
</tbody>
</table>

**Additional resources**

- [Hybrid networking](/docs/networking/ovn_kubernetes_network_provider/configuring-hybrid-networking#configuring-hybrid-networking)
