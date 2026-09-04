---
title: Introduction to OpenShift API for Data Protection
sidebar_position: 1
---

# Introduction to OpenShift API for Data Protection {#oadp-introduction}

<a id="oadp-introduction"></a>

Use OpenShift API for Data Protection (OADP) to safeguard applications, application-related cluster resources, persistent volumes, and internal images on OpenShift Container Platform. OADP backs up containerized applications and virtual machines (VMs). This helps you ensure disaster recovery.

However, OADP does not serve as a disaster recovery solution for `etcd` or OpenShift Operators.

:::warning

OADP support is applicable to customer workload namespaces and cluster scope resources.

Full cluster `backup` and `restore` are not supported.

:::

## OpenShift API for Data Protection APIs {#oadp-apis_oadp-api}

OADP provides APIs that enable multiple approaches to customizing backups and preventing the inclusion of unnecessary or inappropriate resources.

OADP provides the following APIs. See the *Additional resources* section for more details.

- `Backup`
- `Restore`
- `Schedule`
- `BackupStorageLocation`
- `VolumeSnapshotLocation`

### Support for OpenShift API for Data Protection {#oadp-operator-supported_oadp-api}

Review the OADP support matrix for version compatibility with OpenShift Container Platform releases and lifecycle policy information, including Extended Update Support (EUS) options.

**Supported versions of OADP**

<table>
<thead>
<tr>
  <th>Version</th>
  <th>OpenShift Container Platform version</th>
  <th>General availability</th>
  <th>Full support ends</th>
  <th>Maintenance ends</th>
  <th>Extended Update Support (EUS)</th>
  <th>Extended Update Support Term 2 (EUS Term 2)</th>
</tr>
</thead>
<tbody>
<tr>
  <td>1.6</td>
  <td><ul><li>4.22</li></ul></td>
  <td>09 Jun 2026</td>
  <td>Release of 1.7</td>
  <td>Release of 1.8</td>
  <td>30 Jun 2028</td>
  <td>30 Jun 2029</td>
</tr>
<tr>
  <td>1.5</td>
  <td><ul><li>4.19</li><li>4.20</li><li>4.21</li></ul></td>
  <td>17 June 2025</td>
  <td>Release of 1.6</td>
  <td>Release of 1.7</td>
  <td>30 Jun 2028</td>
  <td>30 Jun 2029<br><br><br><br>EUS must be on OpenShift Container Platform 4.21</td>
</tr>
<tr>
  <td>EUS Term 2 must be on OpenShift Container Platform 4.21</td>
  <td>1.4</td>
  <td><ul><li>4.14</li><li>4.15</li><li>4.16</li><li>4.17</li><li>4.18</li></ul></td>
  <td>10 Jul 2024</td>
  <td>Release of 1.5</td>
  <td>Release of 1.6</td>
  <td>27 Jun 2026<br><br>EUS must be on OpenShift Container Platform 4.16</td>
</tr>
<tr>
  <td>27 Jun 2027<br><br>EUS Term 2 must be on OpenShift Container Platform 4.16</td>
  <td>1.3</td>
  <td><ul><li>4.12</li><li>4.13</li><li>4.14</li><li>4.15</li></ul></td>
  <td>29 Nov 2023</td>
  <td>10 Jul 2024</td>
  <td>Release of 1.5</td>
  <td>31 Oct 2025<br><br>EUS must be on OpenShift Container Platform 4.14</td>
</tr>
<tr>
  <td>31 Oct 2026<br><br>EUS Term 2 must be on OpenShift Container Platform 4.14</td>
</tr>
</tbody>
</table>

#### Unsupported versions of the OADP Operator {#oadp-operator-unsupported_oadp-api}

**Previous versions of the OADP Operator which are no longer supported**

|  |  |  |  |
| --- | --- | --- | --- |
| Version | General availability | Full support ended | Maintenance ended |
| 1.2 | 14 Jun 2023 | 29 Nov 2023 | 10 Jul 2024 |
| 1.1 | 01 Sep 2022 | 14 Jun 2023 | 29 Nov 2023 |
| 1.0 | 09 Feb 2022 | 01 Sep 2022 | 14 Jun 2023 |

For more details about EUS, see [Extended Update Support](https://access.redhat.com/support/policy/updates/openshift#eus).

For more details about EUS Term 2, see [Extended Update Support Term 2](https://access.redhat.com/support/policy/updates/openshift#eust2).

**Additional resources**

- [Backup](/docs/backup_and_restore/application_backup_and_restore/backing_up_and_restoring/backing-up-applications#backing-up-applications)
- [Restore](/docs/backup_and_restore/application_backup_and_restore/backing_up_and_restoring/restoring-applications#restoring-applications)
- [Schedule](/docs/backup_and_restore/application_backup_and_restore/backing_up_and_restoring/oadp-scheduling-backups-doc#oadp-scheduling-backups-doc)
- [BackupStorageLocation](/docs/backup_and_restore/application_backup_and_restore/installing/installing-oadp-aws#oadp-about-backup-snapshot-locations_installing-oadp-aws)
- [VolumeSnapshotLocation](/docs/backup_and_restore/application_backup_and_restore/backing_up_and_restoring/oadp-backing-up-pvs-csi-doc#oadp-backing-up-pvs-csi-doc)
- [Backing up etcd](/docs/backup_and_restore/control_plane_backup_and_restore/backing-up-etcd#backup-etcd)
