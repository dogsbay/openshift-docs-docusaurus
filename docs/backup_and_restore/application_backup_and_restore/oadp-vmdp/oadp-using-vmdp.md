---
title: Using OADP virtual machine data protection
sidebar_position: 2
---

# Using OADP virtual machine data protection {#oadp-using-vmdp}

<a id="oadp-using-vmdp"></a>

Install the OpenShift API for Data Protection virtual machine data protection (VMDP) command-line interface (CLI), configure a backup storage location, and back up and restore data from within your VM. This helps you to manage your own VM backups independently.

## Installing the OADP virtual machine data protection CLI {#oadp-vmdp-installing-cli_oadp-using-vmdp}

Install the OpenShift API for Data Protection virtual machine data protection (VMDP) command-line interface (CLI) inside your VM to back up and restore data. This helps you to download the correct binary for your VM guest operating system.

The OADP Operator deploys a download server in the cluster as the `openshift-adp-vmdp-server` service in the `openshift-adp` namespace. A `ConsoleCLIDownload` resource links to the download server routes, and users can access the download links from the OpenShift Container Platform web console or by using HTTP directly.

**Prerequisites**

- You have installed the OADP Operator.
- You have a running VM on OpenShift Virtualization with a supported guest operating system.
- You have installed the `virtctl` CLI tool.

**Procedure**

1. Get the cluster IP of the VMDP download server by running the following command:
   ```terminal
   $ oc get svc -n openshift-adp openshift-adp-vmdp-server
   ```

   Make a note of the `CLUSTER-IP` value from the output.
2. Install the VMDP CLI:
   - For Linux VMs, download the VMDP binary and make it executable inside the VM by running the following command:
     ```terminal
     $ virtctl ssh <vm_user>@<vm_name> -n <vm_namespace> \
       --command "curl -kLf 'http://<cluster_ip>:80/download/oadp-vmdp_linux_amd64' \
       -o oadp-vmdp_linux_amd64 && chmod +x oadp-vmdp_linux_amd64"
     ```

     where:

     <dl>
     <dt><code>&lt;vm_user&gt;</code></dt>
     <dd>Specifies the username for the VM. For example, <code>fedora</code>.</dd>
     <dt><code>&lt;vm_name&gt;</code></dt>
     <dd>Specifies the name of the VM.</dd>
     <dt><code>&lt;vm_namespace&gt;</code></dt>
     <dd>Specifies the namespace of the VM.</dd>
     <dt><code>&lt;cluster_ip&gt;</code></dt>
     <dd>Specifies the <code>CLUSTER-IP</code> value of the <code>openshift-adp-vmdp-server</code> service.</dd>
     </dl>
   - For Microsoft Windows VMs, access the VM by using Remote Desktop Protocol (RDP) or the VNC console, open PowerShell, and download the VMDP binary by running the following command:
     ```powershell
     PS> curl.exe -L -o oadp-vmdp.exe "http://<cluster_ip>:80/download/oadp-vmdp_windows_amd64.exe"
     ```

     Replace `<cluster_ip>` with the `CLUSTER-IP` value of the `openshift-adp-vmdp-server` service.

**Verification**

- Depending on your operating system, use one of the following steps to verify the installation:
  - For Linux VMs, verify that the binary is installed and working by running the following command:
    ```terminal
    $ virtctl ssh <vm_user>@<vm_name> -n <vm_namespace> \
      --command "./oadp-vmdp_linux_amd64 --help"
    ```

    ```terminal
    usage: oadp-vmdp [<flags>] <command> [<args> ...]
    OADP VM Data Protection - Virtual Machine Data Protection for OpenShift
        Virtualization

    Flags:
      --[no-]help                Show context-sensitive help (also try --help-long
                                 and --help-man).
      --[no-]version             Show application version.
      --log-file=LOG-FILE        Override log file.
      --[no-]disable-file-logging
                                 Disable file-based logging.
      ....
    ```
  - For Microsoft Windows VMs, open PowerShell and verify the binary by running the following command:
    ```powershell
    PS> .\oadp-vmdp.exe --help
    ```

## Creating a backup storage location with S3-compatible storage by using the VMDP CLI {#oadp-vmdp-creating-bsl-s3_oadp-using-vmdp}

Create a backup storage location (BSL) with S3-compatible storage to set up an encrypted repository for storing your virtual machine (VM) backups. This helps you to configure remote S3-compatible storage for your data.

When you create a BSL, the virtual machine data protection (VMDP) command-line interface (CLI) creates a personal encrypted repository in the specified storage backend. You provide the encryption password, which is used to encrypt all backup data. Only users with this password can access the backup data.

**Prerequisites**

- You have installed the VMDP CLI inside your VM.
- You have the storage credentials for your S3-compatible storage.

**Procedure**

1. To create a BSL with S3-compatible storage, run the following command:
   ```terminal
   $ sshpass -p <vm_password> virtctl ssh vm/<vm_name> -n <vm_namespace> \
     --username <vm_username> \
     --local-ssh-opts=-oUserKnownHostsFile=/dev/null \
     --local-ssh-opts=-oStrictHostKeyChecking=no \
     --local-ssh-opts=-oPreferredAuthentications=password \
     --local-ssh-opts=-oPubkeyAuthentication=no \
     --command "./oadp-vmdp_linux_amd64 bsl create s3 --bucket <bucket_name> --endpoint <s3_endpoint> --access-key <access_key> --secret-access-key <secret_access_key> --disable-tls"
   ```

   where:

   <dl>
   <dt><code>&lt;vm_password&gt;</code></dt>
   <dd>Specifies the password for the VM user.</dd>
   <dt><code>&lt;vm_name&gt;</code></dt>
   <dd>Specifies the name of the VM.</dd>
   <dt><code>&lt;vm_namespace&gt;</code></dt>
   <dd>Specifies the namespace of the VM.</dd>
   <dt><code>&lt;vm_username&gt;</code></dt>
   <dd>Specifies the username for the VM. For example, <code>fedora</code>.</dd>
   <dt><code>&lt;bucket_name&gt;</code></dt>
   <dd>Specifies the name of the S3 bucket.</dd>
   <dt><code>&lt;s3_endpoint&gt;</code></dt>
   <dd>Specifies the S3 endpoint URL.</dd>
   <dt><code>&lt;access_key&gt;</code></dt>
   <dd>Specifies the access key ID.</dd>
   <dt><code>&lt;secret_access_key&gt;</code></dt>
   <dd>Specifies the secret access key.</dd>
   </dl>

   You should see an output similar to the following example:

   ```terminal
   Initializing repository with:
     block hash:          BLAKE2B-256-128
     encryption:          AES256-GCM-HMAC-SHA256
     key derivation:      scrypt-65536-8-1
     splitter:            DYNAMIC-4M-BUZHASH
   ```
2. Create an alias for the VMDP binary and name it `oadp-vmdp`.

**Verification**

- To verify the BSL connection status, run the following command:
  ```terminal
  $ oadp-vmdp bsl status
  ```

## Creating a backup storage location with file system storage by using the VMDP CLI {#oadp-vmdp-creating-bsl-filesystem_oadp-using-vmdp}

Create a backup storage location (BSL) with file system storage to set up an encrypted repository for storing your virtual machine (VM) backups. This helps you to configure local file system storage for your data.

When you create a BSL, the virtual machine data protection (VMDP) command-line interface (CLI) creates a personal encrypted repository in the specified storage backend. You provide the encryption password, which is used to encrypt all backup data. Only users with this password can access the backup data.

**Prerequisites**

- You have installed the VMDP CLI inside your VM.
- You have created an alias called `oadp-vmdp` for the VMDP CLI binary.
- You have access to a local file system path for backup storage.

**Procedure**

- To create a BSL with file system storage, run the following command:
  ```terminal
  $ oadp-vmdp bsl create filesystem \
    --path <storage_directory_path>
  ```

  Replace `<storage_directory_path>` with the absolute path to the storage directory.

**Verification**

- To verify the BSL connection status, run the following command:
  ```terminal
  $ oadp-vmdp bsl status
  ```

## Backing up data from a virtual machine by using the VMDP CLI {#oadp-vmdp-backing-up-data_oadp-using-vmdp}

Back up files and directories from within your virtual machine (VM) by using the OpenShift API for Data Protection VM data protection (VMDP) command-line interface (CLI). This helps you to protect user data, including data accessible over network file systems such as CIFS and NFS shares.

VMDP uses data deduplication to store data efficiently. If the same data exists in multiple locations, subsequent backups complete faster because only unique data blocks are stored.

**Prerequisites**

- You are connected to your virtual machine (VM) by using SSH.
- You have installed the VMDP CLI inside your VM.
- You have created an alias called `oadp-vmdp` for the VMDP CLI binary.
- You have created and connected to a backup storage location.

**Procedure**

1. To create a backup of a directory, run the following command:
   ```terminal
   $ oadp-vmdp backup create <path_to_data>
   ```

   Replace `<path_to_data>` with the path to the directory or files to back up.
2. To list all available backups, run the following command:
   ```terminal
   $ oadp-vmdp backup list
   ```
3. To delete a specific backup, run the following command:
   ```terminal
   $ oadp-vmdp backup delete <backup_id>
   ```

   Replace `<backup_id>` with the ID of the backup to delete.

   :::warning

   After you delete backups, orphaned data objects such as pack blobs are not automatically cleaned up from the underlying storage backend. You must manually remove these orphaned objects directly from your storage backend, for example, by deleting them from your S3 bucket.

   :::

## Restoring data to a virtual machine by using the VMDP CLI {#oadp-vmdp-restoring-data_oadp-using-vmdp}

Restore data from a backup to your virtual machine (VM) by using the OpenShift API for Data Protection VM data protection (VMDP) command-line interface (CLI). This helps you to recover files to the original location or to a different directory.

You can restore from the most recent backup or from a specific backup by specifying its ID. You can also restore to a different VM by connecting to the BSL of the new VM.

**Prerequisites**

- You are connected to your VM by using SSH.
- You have installed the VMDP CLI inside your VM.
- You have created an alias called `oadp-vmdp` for the VMDP CLI binary.
- You have connected to the backup storage location that contains the backup.

**Procedure**

- Depending on your specific use case, use one of the following steps to restore data:
  - To restore data from the most recent backup, run the following command:
    ```terminal
    $ oadp-vmdp restore <path_to_restore>
    ```

    Replace `<path_to_restore>` with the path to restore.
  - To restore a specific backup to a custom location, run the following commands:
    1. Retrieve the source backup ID:
       ```terminal
       $ oadp-vmdp backup list
       ```
    2. Restore the backup by running the following command:
       ```terminal
       $ oadp-vmdp restore <source_id> <restore_path>
       ```

       where:

       <dl>
       <dt><code>&lt;source_id&gt;</code></dt>
       <dd>Specifies the ID of the backup to restore.</dd>
       <dt><code>&lt;restore_path&gt;</code></dt>
       <dd>Specifies the directory path where the data is restored.</dd>
       </dl>
  - To restore data from a different VM, run the following commands:
    1. To connect to the existing BSL from the new VM, run the following command:
       ```terminal
       $ oadp-vmdp bsl connect s3 \
         --bucket <bucket_name> \
         --endpoint <s3_endpoint> \
         --access-key <access_key> \
         --secret-access-key <secret_access_key>
       ```
    2. Restore from the BSL by running the following command:
       ```terminal
       $ oadp-vmdp restore <path_to_data>
       ```

## OADP VMDP BSL management commands {#oadp-vmdp-managing-bsl_oadp-using-vmdp}

Review the available commands for managing backup storage locations (BSL) with the virtual machine data protection (VMDP) command-line interface (CLI). This helps you to connect, disconnect, and manage your BSL connections.

**BSL management commands**

| Command | Description |
| --- | --- |
| `oadp-vmdp bsl create` | Create and connect to a new BSL. |
| `oadp-vmdp bsl connect` | Connect to an existing BSL. |
| `oadp-vmdp bsl disconnect` | Disconnect from the current BSL. |
| `oadp-vmdp bsl status` | Show the current BSL connection status. |
| `oadp-vmdp bsl change-password` | Change the BSL encryption password. |

## Troubleshooting OADP virtual machine data protection {#oadp-vmdp-troubleshooting_oadp-using-vmdp}

Troubleshoot common issues and solutions for the OpenShift API for Data Protection virtual machine data protection (VMDP) command-line interface (CLI). This helps you to resolve connection and configuration problems.

### Not connected to a backup storage location {#_not_connected_to_a_backup_storage_location}

If you receive a `Not connected to a Backup Storage Location` error message, check the connection status and reconnect:

```terminal
$ oadp-vmdp bsl status
```

```terminal
$ oadp-vmdp bsl connect s3 \
  --bucket <bucket_name> \
  --endpoint <s3_endpoint> \
  --access-key <access_key> \
  --secret-access-key <secret_access_key>
```

### Prefix must not contain oadp-vmdp {#_prefix_must_not_contain_oadp-vmdp}

The `oadp-vmdp/` prefix is added automatically. Do not include `oadp-vmdp` as a path segment in the `--prefix` option. Ensure that the `--prefix` value does not start or end with whitespace.

### S3 connection issues {#_s3_connection_issues}

For S3-compatible services, you might need to use the following options:

- `--disable-tls` for non-HTTPS endpoints.
- `--disable-tls-verification` for self-signed certificates.
- `--root-ca-pem-path` to specify a custom CA certificate.

### Getting help {#_getting_help}

- To view available commands and options, run the following command:

```terminal
$ oadp-vmdp --help
```

- To view help for a specific command, run the following command:

```terminal
$ oadp-vmdp <command> --help
```
