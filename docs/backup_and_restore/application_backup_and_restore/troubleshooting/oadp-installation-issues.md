---
title: OADP installation issues
sidebar_position: 5
---

# OADP installation issues

<a id="oadp-installation-issues"></a>

Resolve common installation issues with the Data Protection Application (DPA), such as invalid backup storage directories and incorrect cloud provider credentials. This helps you successfully install and configure OADP in your environment.

## Resolving invalid directories in backup storage {#resolving-backup-storage-contains-invalid-directories-issue_installation-issues}

Resolve the `Backup storage contains invalid top-level directories` error that occurs when object storage contains non-Velero directories. This helps you configure the correct bucket prefix for shared object storage.

**Procedure**

- If the object storage is not dedicated to Velero, you must specify a prefix for the bucket by setting the `spec.backupLocations.velero.objectStorage.prefix` parameter in the `DataProtectionApplication` manifest.

## Resolving incorrect AWS credentials {#resolving-incorrect-aws-credentials-issue_installation-issues}

Resolve credential errors such as `InvalidAccessKeyId` or `NoCredentialProviders` that occur when the `credentials-velero` file is incorrectly formatted. This helps you configure valid AWS credentials for OADP backup operations.

If you incorrectly format the `credentials-velero` file used for creating the `Secret` object, multiple errors might occur, including the following examples:

- The `oadp-aws-registry` pod log displays the following error message:
  ```text
  `InvalidAccessKeyId: The AWS Access Key Id you provided does not exist in our records.`
  ```
- The `Velero` pod log displays the following error message:
  ```text
  NoCredentialProviders: no valid providers in chain.
  ```

**Procedure**

- Ensure that the `credentials-velero` file is correctly formatted, as shown in the following example:
  ```
  [default]
  aws_access_key_id=AKIAIOSFODNN7EXAMPLE
  aws_secret_access_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
  ```

  where:

  <dl>
  <dt><code>[default]</code></dt>
  <dd>Specifies the AWS default profile.</dd>
  <dt><code>aws_access_key_id</code></dt>
  <dd>Do not enclose the values with quotation marks (<code>"</code>, <code>'</code>).</dd>
  </dl>
