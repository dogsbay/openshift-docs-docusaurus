---
title: Source-to-image
sidebar_position: 2
---

# Source-to-image {#using-s21-images}

<a id="using-s21-images"></a>

To create containerized applications in OpenShift Container Platform without manually configuring runtime environments, you can use Source-to-Image (S2I) images. S2I images are runtime base images for languages such as Node.js, Python, and Java that you can insert your code into. You can use Red Hat Software Collections images as a foundation for applications that rely on specific runtime environments, and access S2I images through the Cluster Samples Operator.

## Accessing S2I builder images in the OpenShift Container Platform Developer Console {#accessing-s2i-builder-images-in-developer-console_using-s21-images}

You can access S2I builder images through the Developer Console in the web console. You need these images to build containerized applications from your source code.

**Procedure**

1. Log in to the OpenShift Container Platform web console using your login credentials. The default view for the OpenShift Container Platform web console is the **Administrator** perspective.
2. Use the perspective switcher to switch to the **Developer** perspective.
3. In the **+Add** view, use the **Project** drop-down list to select an existing project or create a new project.
4. Click **All services** in the **Developer Catalog** tile.
5. Click **Builder Images** under **Type** to see the available S2I images.

## Source-to-image build process overview {#images-s2i-build-process-overview_using-s21-images}

Source-to-image (S2I) is a build process in OpenShift Container Platform that injects your source code into a container image. S2I automates the creation of ready-to-run container images from your application source code without manual configuration.

S2I performs the following steps:

1. Runs the `FROM <builder image>` command
2. Copies the source code to a defined location in the builder image
3. Runs the assemble script in the builder image
4. Sets the run script in the builder image as the default command

Buildah then creates the container image.

**Additional resources**

- [Red Hat Software Collections container images](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html-single/using_red_hat_software_collections_container_images/index)
- [Introduction to source-to-image for OpenShift with Red Hat build of OpenJDK](https://docs.redhat.com/en/documentation/red_hat_build_of_openjdk/11/html/using_source-to-image_for_openshift_with_red_hat_build_of_openjdk_11/openjdk-overview-s2i-openshift)
- [Configuring the Cluster Samples Operator](/docs/openshift_images/configuring-samples-operator#configuring-samples-operator)
- [Using build strategies](/docs/cicd/builds/build-strategies#builds-strategy-s2i-build_build-strategies)
- [Troubleshooting the Source-to-Image process](/docs/support/troubleshooting/troubleshooting-s2i#troubleshooting-s2i)
- [Creating images from source code with source-to-image](/docs/openshift_images/create-images#images-create-s2i_create-images)
- [About testing source-to-image images](/docs/openshift_images/create-images#images-test-s2i_create-images)
