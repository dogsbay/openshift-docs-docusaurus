---
title: Uninstalling the File Integrity Operator
sidebar_position: 10
---

# Uninstalling the File Integrity Operator  {#fio-uninstalling}

<a id="fio-uninstalling"></a>

You can remove the File Integrity Operator from your cluster by using the OpenShift Container Platform web console.

## Uninstall the File Integrity Operator using the web console {#fio-uninstall-console_fio-uninstalling}

To remove the File Integrity Operator, you must first delete the `FileIntegrity` objects in all namespaces. After the objects are removed, you can then remove the Operator and its namespace.

**Prerequisites**

- You have access to an OpenShift Container Platform cluster that uses an account with `cluster-admin` permissions.
- The File Integrity Operator is installed.

**Procedure**

1. Navigate to the **Ecosystem** → **Installed Operators** → **File Integrity Operator** page.
2. From the **File Integrity** tab, ensure the **Show operands in: All namespaces** default option is selected to list all `FileIntegrity` objects in all namespaces.
3. Click the Options menu ![](/images/kebab.png "Options menu") for a `FileIntegrity` object.
4. Select **Delete FileIntegrity**.
5. Repeat the previous two steps until no `FileIntegrity` objects remain.
6. Go to the **Administration** → **Ecosystem** → **Installed Operators** page.
7. Click the Options menu ![](/images/kebab.png "Options menu") on the **File Integrity Operator** entry.
8. Select **Uninstall Operator**.
9. Go to the **Home** → **Projects** page.
10. Search for `openshift-file-integrity`.
11. Click the Options menu ![](/images/kebab.png "Options menu") for the **openshift-file-integrity** project entry.
12. Select **Delete Project**. A **Delete Project** dialog box opens on the web console.

**Verification**

- Type `openshift-file-integrity` in the **Delete Project** dialog box.
- Click the **Delete** button.
