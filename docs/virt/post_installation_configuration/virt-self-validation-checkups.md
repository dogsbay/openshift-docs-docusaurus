---
title: Self validation checkup
sidebar_position: 5
---

# Self validation checkup {#virt-self-validation-checkups}

<a id="virt-self-validation-checkups"></a>

A self validation checkup runs conformance tests on critical subsystems to verify that the environment is fully functional and self-sustained before you deploy production workloads.

## Run a self validation checkup in the web console {#virt-run-self-validation-checkup-web-console_virt-self-validation-checkups}

Running a self validation checkup streamlines the process of running functional tests, which enables you to validate the stability, health, and compliance of an OpenShift Virtualization installation before deploying workloads.

**Prerequisites**

- You have cluster administrator permissions.
- You have access to an OpenShift Container Platform cluster where OpenShift Virtualization is installed.
- You are logged in to the OpenShift Container Platform web console.

**Procedure**

1. In the OpenShift Container Platform web console, go to **Virtualization** → **Checkups**.
2. Go to the **Self validation** tab.
3. Click **Run checkup**.
4. Configure settings for the test that you want to run.
5. Optional: You can enable a dry run test by clicking **Advanced settings** and then toggling the **Dry run** button.
6. Click **Run**. The **Self validation checkup details** page is displayed.
   You can observe the self validation checkup running in real time. The test can take several hours to complete.
7. After the test is complete, you can view high-level results in the **Self validation checkup details** page, including the names of any failing tests.
8. Optional: You can download detailed results as a ZIP file by clicking **Download results** in the **Self validation checkup details** page.
