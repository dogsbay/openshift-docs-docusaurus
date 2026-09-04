---
title: Uninstalling the Compliance Operator
sidebar_position: 4
---

# Uninstalling the Compliance Operator {#compliance-operator-uninstallation}

<a id="compliance-operator-uninstallation"></a>

You can remove the OpenShift Compliance Operator from your cluster by using the OpenShift Container Platform web console or the OpenShift CLI (`oc`).

## Uninstalling the OpenShift Compliance Operator from OpenShift Container Platform using the web console {#compliance-operator-uninstall_compliance-operator-uninstallation}

To remove the Compliance Operator, you must first delete the objects in the namespace. After the objects are removed, you can remove the Operator and its namespace by deleting the **openshift-compliance** project.

**Prerequisites**

- Access to an OpenShift Container Platform cluster by using an account with `cluster-admin` permissions.
- The OpenShift Compliance Operator is installed.

**Procedure**

1. Go to the **Ecosystem** → **Installed Operators** → **Compliance Operator** page.
   1. Click **All instances**.
   2. In **All namespaces**, click the Options menu ![](/images/kebab.png "Options menu") and delete all ScanSettingBinding, ComplainceSuite, ComplianceScan, and ProfileBundle objects.
2. Switch to the **Administration** → **Ecosystem** → **Installed Operators** page.
3. Click the Options menu ![](/images/kebab.png "Options menu") on the **Compliance Operator** entry and select **Uninstall Operator**.
4. Switch to the **Home** → **Projects** page.
5. Search for 'compliance'.
6. Click the Options menu ![](/images/kebab.png "Options menu") next to the **openshift-compliance** project, and select **Delete Project**.
   1. Confirm the deletion by typing `openshift-compliance` in the dialog box, and click **Delete**.

## Uninstalling the OpenShift Compliance Operator from OpenShift Container Platform using the CLI {#compliance-operator-uninstall-cli_compliance-operator-uninstallation}

To remove the Compliance Operator, you must first delete the objects in the namespace. After the objects are removed, you can remove the Operator and its namespace by deleting the **openshift-compliance** project.

**Prerequisites**

- Access to an OpenShift Container Platform cluster by using an account with `cluster-admin` permissions.
- The OpenShift Compliance Operator is installed.

**Procedure**

1. Delete all objects in the namespace.
   1. Delete the `ScanSettingBinding` objects:
      ```terminal
      $ oc delete ssb --all -n openshift-compliance
      ```
   2. Delete the `ScanSetting` objects:
      ```terminal
      $ oc delete ss --all -n openshift-compliance
      ```
   3. Delete the `ComplianceSuite` objects:
      ```terminal
      $ oc delete suite --all -n openshift-compliance
      ```
   4. Delete the `ComplianceScan` objects:
      ```terminal
      $ oc delete scan --all -n openshift-compliance
      ```
   5. Delete the `ProfileBundle` objects:
      ```terminal
      $ oc delete profilebundle.compliance --all -n openshift-compliance
      ```
2. Delete the Subscription object:
   ```terminal
   $ oc delete sub --all -n openshift-compliance
   ```
3. Delete the CSV object:
   ```terminal
   $ oc delete csv --all -n openshift-compliance
   ```
4. Delete the project:
   ```terminal
   $ oc delete project openshift-compliance
   ```

   ```terminal title="Example output"
   project.project.openshift.io "openshift-compliance" deleted
   ```

**Verification**

1. Confirm the namespace is deleted:
   ```terminal
   $ oc get project/openshift-compliance
   ```

   ```terminal title="Example output"
   Error from server (NotFound): namespaces "openshift-compliance" not found
   ```

**Additional resources**

- [Installing the Compliance Operator](/security/compliance_operator/compliance-operator-installation#compliance-operator-installation)
- [Managing the Compliance Operator](/security/compliance_operator/compliance-operator-manage#compliance-operator-understanding)
