---
title: Exporting applications
sidebar_position: 5
---

# Exporting applications {#odc-exporting-applications}

<a id="odc-exporting-applications"></a>

As a developer, you can export your application in the ZIP file format. Based on your needs, import the exported application to another project in the same cluster or a different cluster by using the **Import YAML** option in the **+Add** view. Exporting your application helps you to reuse your application resources and saves your time.

## Prerequisites {#prerequisites_odc-exporting-applications}

- You have installed the gitops-primer Operator from the software catalog.
  :::note

  The **Export application** option is disabled in the **Topology** view even after installing the gitops-primer Operator.

  :::
- You have created an application in the **Topology** view to enable **Export application**.

## Procedure {#odc-exporting-applications-procedure}

1. In the developer perspective, perform one of the following steps:
   1. Navigate to the **+Add** view and click **Export application** in the **Application portability** tile.
   2. Navigate to the **Topology** view and click **Export application**.
2. Click **OK** in the **Export Application** dialog box. A notification opens to confirm that the export of resources from your project has started.
3. Optional steps that you might need to perform in the following scenarios:
   - If you have started exporting an incorrect application, click  **Export application** → **Cancel Export**.
   - If your export is already in progress and you want to start a fresh export, click  **Export application** → **Restart Export**.
   - If you want to view logs associated with exporting an application, click  **Export application** and the **View Logs** link. ![export-application-dialog-box](/images/export-application-dialog-box.png)
4. After a successful export, click **Download** in the dialog box to download application resources in ZIP format onto your machine.
