---
title: Installing the opm CLI
sidebar_position: 1
---

# Installing the opm CLI {#cli-opm-install}

<a id="cli-opm-install"></a>

## About the opm CLI {#olm-about-opm_cli-opm-install}

The `opm` CLI is an Operator Framework tool for creating and maintaining Operator catalogs from bundle images in OpenShift Container Platform. You can use it to build catalog container images that Operator Lifecycle Manager (OLM) references through catalog sources.

A catalog contains a database of pointers to Operator manifest content that can be queried through an included API that is served when the container image is run. On OpenShift Container Platform, Operator Lifecycle Manager (OLM) can reference the image in a catalog source, defined by a `CatalogSource` object, which polls the image at regular intervals to enable frequent updates to installed Operators on the cluster.

**Additional resources**

- See [Operator Framework packaging format](/docs/operators/understanding/olm-packaging-format#olm-bundle-format_olm-packaging-format) for more information about the bundle format.

## Installing the opm CLI {#olm-installing-opm_cli-opm-install}

You can install the `opm` CLI tool on your Linux, macOS, or Windows workstation.

**Prerequisites**

- For Red Hat Enterprise Linux (RHEL) 9.0 and later, you must provide the following packages:
  - `podman` version 1.9.3+ (version 2.0+ recommended)
  - `glibc` version 2.28+

**Procedure**

1. Navigate to the [OpenShift mirror site](https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/latest-4.22/) and download the latest version of the tarball that matches your operating system.
2. Unpack the archive.
   - For Linux or macOS:
     ```terminal
     $ tar xvf <file>
     ```
   - For Windows, unzip the archive with a ZIP program.
3. Place the file anywhere in your `PATH`.
   - For Linux or macOS:
     1. Check your `PATH`:
        ```terminal
        $ echo $PATH
        ```
     2. Move the file. For example:
        ```terminal
        $ sudo mv ./opm /usr/local/bin/
        ```
   - For Windows:
     1. Check your `PATH`:
        ```terminal
        C:\> path
        ```
     2. Move the file:
        ```terminal
        C:\> move opm.exe <directory>
        ```

**Verification**

- After you install the `opm` CLI, verify that it is available:
  ```terminal
  $ opm version
  ```

**Additional resources**

- See [Managing custom catalogs](/docs/operators/admin/olm-managing-custom-catalogs#olm-managing-custom-catalogs) for `opm` procedures including creating, updating, and pruning catalogs.
