---
title: Using build strategies
sidebar_position: 5
---

# Using build strategies {#build-strategies}

<a id="build-strategies"></a>

The following sections define the primary supported build strategies, and how to use them.

## Docker build {#builds-strategy-docker-build_build-strategies}

OpenShift Container Platform uses Buildah to build a container image from a Dockerfile. For more information on building container images with Dockerfiles, see [the Dockerfile reference documentation](https://docs.docker.com/engine/reference/builder/).

:::tip

If you set Docker build arguments by using the `buildArgs` array, see [Understand how ARG and FROM interact](https://docs.docker.com/engine/reference/builder/#understand-how-arg-and-from-interact) in the Dockerfile reference documentation.

:::

### Replacing the Dockerfile FROM image {#builds-strategy-docker-from-image_build-strategies}

You can replace the `FROM` instruction of the Dockerfile with the `from` parameters of the `BuildConfig` object. If the Dockerfile uses multi-stage builds, the image in the last `FROM` instruction will be replaced.

**Procedure**

- To replace the `FROM` instruction of the Dockerfile with the `from` parameters of the `BuildConfig` object, add the following settings to the `BuildConfig` object:
  ```yaml
  strategy:
    dockerStrategy:
      from:
        kind: "ImageStreamTag"
        name: "debian:latest"
  ```

### Using Dockerfile path {#builds-strategy-dockerfile-path_build-strategies}

By default, docker builds use a Dockerfile located at the root of the context specified in the `BuildConfig.spec.source.contextDir` field.

The `dockerfilePath` field allows the build to use a different path to locate your Dockerfile, relative to the `BuildConfig.spec.source.contextDir` field. It can be a different file name than the default Dockerfile, such as `MyDockerfile`, or a path to a Dockerfile in a subdirectory, such as `dockerfiles/app1/Dockerfile`.

**Procedure**

- Set the `dockerfilePath` field for the build to use a different path to locate your Dockerfile:
  ```yaml
  strategy:
    dockerStrategy:
      dockerfilePath: dockerfiles/app1/Dockerfile
  ```

### Using docker environment variables {#builds-strategy-docker-environment-variables_build-strategies}

To make environment variables available to the docker build process and resulting image, you can add environment variables to the `dockerStrategy` definition of the build configuration.

The environment variables defined there are inserted as a single `ENV` Dockerfile instruction right after the `FROM` instruction, so that it can be referenced later on within the Dockerfile.

The variables are defined during build and stay in the output image, therefore they will be present in any container that runs that image as well.

For example, defining a custom HTTP proxy to be used during build and runtime:

```yaml
dockerStrategy:
...
  env:
    - name: "HTTP_PROXY"
      value: "http://myproxy.net:5187/"
```

You can also manage environment variables defined in the build configuration with the `oc set env` command.

### Adding Docker build arguments {#builds-strategy-docker-build-arguments_build-strategies}

You can set [Docker build arguments](https://docs.docker.com/engine/reference/builder/#arg) using the `buildArgs` array. The build arguments are passed to Docker when a build is started.

:::tip

See [Understand how ARG and FROM interact](https://docs.docker.com/engine/reference/builder/#understand-how-arg-and-from-interact) in the Dockerfile reference documentation.

:::

**Procedure**

- To set Docker build arguments, add entries to the `buildArgs` array, which is located in the `dockerStrategy` definition of the `BuildConfig` object. For example:
  ```yaml
  dockerStrategy:
  ...
    buildArgs:
      - name: "version"
        value: "latest"
  ```

  :::note

  Only the `name` and `value` fields are supported. Any settings on the `valueFrom` field are ignored.

  :::

### Squashing layers with docker builds {#builds-strategy-docker-squash-layers_build-strategies}

Docker builds normally create a layer representing each instruction in a Dockerfile. Setting the `imageOptimizationPolicy` to `SkipLayers` merges all instructions into a single layer on top of the base image.

**Procedure**

- Set the `imageOptimizationPolicy` to `SkipLayers`:
  ```yaml
  strategy:
    dockerStrategy:
      imageOptimizationPolicy: SkipLayers
  ```

### Using build volumes {#builds-using-build-volumes_build-strategies-docker}

You can mount build volumes to give running builds access to information that you do not want to persist in the output container image.

Build volumes provide sensitive information, such as repository credentials, that the build environment or configuration only needs at build time. Build volumes are different from build inputs, whose data can persist in the output container image.

The mount points of build volumes, from which the running build reads data, are functionally similar to [pod volume mounts](https://kubernetes.io/docs/concepts/storage/volumes/).

**Prerequisites**

- You have added an input secret, config map, or both to a BuildConfig object.

**Procedure**

- In the `dockerStrategy` definition of the `BuildConfig` object, add any build volumes to the `volumes` array. For example:
  ```yaml
  spec:
    dockerStrategy:
      volumes:
        - name: secret-mvn
          mounts:
          - destinationPath: /opt/app-root/src/.ssh
          source:
            type: Secret
            secret:
              secretName: my-secret
        - name: settings-mvn
          mounts:
          - destinationPath: /opt/app-root/src/.m2
          source:
            type: ConfigMap
            configMap:
              name: my-config
        - name: my-csi-volume
          mounts:
          - destinationPath: /opt/app-root/src/some_path
          source:
            type: CSI
            csi:
              driver: csi.sharedresource.openshift.io
              readOnly: true
              volumeAttributes:
                attribute: value
  ```

  where:

  <dl>
  <dt><code>name</code></dt>
  <dd>Specifies a unique name.</dd>
  <dt><code>destinationPath</code></dt>
  <dd>Specifies the absolute path of the mount point. It must not contain <code>..</code> or <code>:</code> and does not collide with the destination path generated by the builder. The <code>/opt/app-root/src</code> is the default home directory for many Red Hat S2I-enabled images.</dd>
  <dt><code>type</code></dt>
  <dd>Specifies the type of source, <code>ConfigMap</code>, <code>Secret</code>, or <code>CSI</code>.</dd>
  <dt><code>secretName</code></dt>
  <dd>Specifies the name of the source.</dd>
  <dt><code>driver</code></dt>
  <dd>Specifies the driver that provides the ephemeral CSI volume.</dd>
  <dt><code>readOnly</code></dt>
  <dd>Specifies that a read-only volume is provided. This value must be set to <code>true</code>.</dd>
  <dt><code>volumeAttributes</code></dt>
  <dd>(Optional) Specifies the volume attributes of the ephemeral CSI volume. Consult the CSI driver’s documentation for supported attribute keys and values.</dd>
  </dl>

**Additional resources**

- [Build inputs](/docs/cicd/builds/creating-build-inputs#builds-define-build-inputs_creating-build-inputs)
- [Input secrets and config maps](/docs/cicd/builds/creating-build-inputs#builds-input-secrets-configmaps_creating-build-inputs)

## Source-to-image build {#builds-strategy-s2i-build_build-strategies-docker}

Source-to-image (S2I) is a tool for building reproducible container images. It produces ready-to-run images by injecting application source into a container image and assembling a new image. The new image incorporates the base image, the builder, and built source and is ready to use with the `buildah run` command. S2I supports incremental builds, which re-use previously downloaded dependencies, previously built artifacts, and so on.

### Perform source-to-image incremental builds {#builds-strategy-s2i-incremental-builds_build-strategies-docker}

Source-to-image (S2I) can perform incremental builds, which means it reuses artifacts from previously-built images.

See S2I Requirements for information on how to create a builder image supporting incremental builds.

**Procedure**

- To create an incremental build, create a  with the following modification to the strategy definition:
  ```yaml
  strategy:
    sourceStrategy:
      from:
        kind: "ImageStreamTag"
        name: "incremental-image:latest"
      incremental: true
  ```

  where:

  <dl>
  <dt><code>strategy.sourceStrategy.from.name</code></dt>
  <dd>Specifies an image that supports incremental builds. Consult the documentation of the builder image to determine if it supports this behavior.</dd>
  <dt><code>strategy.sourceStrategy.incremental</code></dt>
  <dd>Specifies whether an incremental build is attempted. If the builder image does not support incremental builds, the build will still succeed, but you will get a log message stating the incremental build was not successful because of a missing <code>save-artifacts</code> script.</dd>
  </dl>

### Override source-to-image builder image scripts {#builds-strategy-s2i-override-builder-image-scripts_build-strategies-docker}

You can override the `assemble`, `run`, and `save-artifacts` source-to-image (S2I) scripts provided by the builder image.

**Procedure**

- To override the `assemble`, `run`, and `save-artifacts` S2I scripts provided by the builder image, complete one of the following actions:
  - Provide an `assemble`, `run`, or `save-artifacts` script in the `.s2i/bin` directory of your application source repository.
  - Provide a URL of a directory containing the scripts as part of the strategy definition in the `BuildConfig` object. For example:
    ```yaml
    strategy:
      sourceStrategy:
        from:
          kind: "ImageStreamTag"
          name: "builder-image:latest"
        scripts: "http://somehost.com/scripts_directory" (1)
    ```

    1. The build process appends `run`, `assemble`, and `save-artifacts` to the path. If any or all scripts with these names exist, the build process uses these scripts in place of scripts with the same name that are provided in the image.

    :::note

    Files located at the `scripts` URL take precedence over files located in `.s2i/bin` of the source repository.

    :::

### Source-to-image environment variables {#builds-strategy-s2i-environment-variables_build-strategies-docker}

There are two ways to make environment variables available to the source build process and resulting image: environment files and `BuildConfig` environment values. The variables that you provide using either method will be present during the build process and in the output image.

#### Use source-to-image environment files {#builds-strategy-s2i-environment-files_build-strategies-docker}

Source build enables you to set environment values, one per line, inside your application, by specifying them in a `.s2i/environment` file in the source repository. The environment variables specified in this file are present during the build process and in the output image.

If you provide a `.s2i/environment` file in your source repository, source-to-image (S2I) reads this file during the build. This allows customization of the build behavior as the `assemble` script may use these variables. The complete list of supported environment variables is available in the using images section for each image.

**Procedure**

1. To disable assets compilation for your Rails application during the build, add the following line to the `.s2i/environment` file:
   ```text
   DISABLE_ASSET_COMPILATION=true
   ```
2. In addition to builds, the specified environment variables are also available in the running application itself. To start the Rails application in `development` mode instead of `production` mode, add the following line to the `.s2i/environment` file:
   ```text
   RAILS_ENV=development
   ```

#### Using source-to-image build configuration environment {#builds-strategy-s2i-buildconfig-environment_build-strategies-docker}

You can add environment variables to the `sourceStrategy` definition of the build configuration. The environment variables defined there are visible during the `assemble` script execution and will be defined in the output image, making them also available to the `run` script and application code.

**Procedure**

- For example, to disable assets compilation for your Rails application:
  ```yaml
  sourceStrategy:
  ...
    env:
      - name: "DISABLE_ASSET_COMPILATION"
        value: "true"
  ```

**Additional resources**

- The build environment section provides more advanced instructions.
- You can also manage environment variables defined in the build configuration with the `oc set env` command.

### Ignore source-to-image source files {#builds-strategy-s2i-ignore-source-files_build-strategies-docker}

Source-to-image (S2I) supports a `.s2iignore` file, which contains a list of file patterns that should be ignored. Files in the build working directory, as provided by the various input sources, that match a pattern found in the `.s2iignore` file will not be made available to the `assemble` script.

### Create images from source code with source-to-image {#images-create-s2i_build-strategies-docker}

Source-to-image (S2I) is a framework that makes it easy to write images that take application source code as an input and produce a new image that runs the assembled application as output.

The main advantage of using S2I for building reproducible container images is the ease of use for developers. As a builder image author, you must understand two basic concepts in order for your images to provide the best S2I performance, the build process and S2I scripts.

#### Understand the source-to-image build process {#images-create-s2i-build_build-strategies-docker}

Leverage the Source-to-image (S2I) process in OpenShift Container Platform to seamlessly transform application source code into ready-to-run, reproducible container images.

The build process consists of the following three fundamental elements, which are combined into a final container image:

- Sources
- S2I scripts
- Builder image

S2I generates a Dockerfile with the builder image as the first `FROM` instruction. The Dockerfile generated by S2I is then passed to Buildah.

#### How to write source-to-image scripts {#images-create-s2i-scripts_build-strategies-docker}

Define mandatory Source-to-image (S2I) scripts, such as `assemble` and `run` to enable OpenShift Container Platform to build, customize, and execute highly reproducible container images.

You can write S2I scripts in any programming language, as long as the scripts are executable inside the builder image. S2I supports multiple options providing `assemble`/`run`/`save-artifacts` scripts. All of these locations are checked on each build in the following order:

1. A script specified in the build configuration.
2. A script found in the application source `.s2i/bin` directory.
3. A script found at the default image URL with the `io.openshift.s2i.scripts-url` label.

Both the `io.openshift.s2i.scripts-url` label specified in the image and the script specified in a build configuration can take one of the following forms:

- `image:///path_to_scripts_dir`: absolute path inside the image to a directory where the S2I scripts are located.
- `$$file:///path_to_scripts_dir$$`: relative or absolute path to a directory on the host where the S2I scripts are located.
- `http(s)://path_to_scripts_dir`: URL to a directory where the S2I scripts are located.

**S2I scripts**

<table>
<thead>
<tr>
  <th>Script</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>assemble</code></td>
  <td>The <code>assemble</code> script builds the application artifacts from a source and places them into appropriate directories inside the image. This script is required. The workflow for this script is:<br><br><ol><li>Optional: Restore build artifacts. If you want to support incremental builds, make sure to define <code>save-artifacts</code> as well.</li><li>Place the application source in the desired location.</li><li>Build the application artifacts.</li><li>Install the artifacts into locations appropriate for them to run.</li></ol></td>
</tr>
<tr>
  <td><code>run</code></td>
  <td>The <code>run</code> script executes your application. This script is required.</td>
</tr>
<tr>
  <td><code>save-artifacts</code></td>
  <td>The <code>save-artifacts</code> script gathers all dependencies that can speed up the build processes that follow. This script is optional. For example:<br><br><ul><li>For Ruby, <code>gems</code> installed by Bundler.</li><li>For Java, <code>.m2</code> contents.</li></ul>These dependencies are gathered into a <code>tar</code> file and streamed to the standard output.</td>
</tr>
<tr>
  <td><code>usage</code></td>
  <td>The <code>usage</code> script allows you to inform the user how to properly use your image. This script is optional.</td>
</tr>
<tr>
  <td><code>test/run</code></td>
  <td>The <code>test/run</code> script allows you to create a process to check if the image is working correctly. This script is optional. The proposed flow of that process is:<br><br><ol><li>Build the image.</li><li>Run the image to verify the <code>usage</code> script.</li><li>Run <code>s2i build</code> to verify the <code>assemble</code> script.</li><li>Optional: Run <code>s2i build</code> again to verify the <code>save-artifacts</code> and <code>assemble</code> scripts save and restore artifacts functionality.</li><li>Run the image to verify the test application is working.</li></ol><div class="alert alert--secondary"><strong>Note</strong><p>The suggested location to put the test application built by your <code>test/run</code> script is the <code>test/test-app</code> directory in your image repository.</p></div></td>
</tr>
</tbody>
</table>

**Example S2I scripts**

The following example S2I scripts are written in Bash. Each example assumes its `tar` contents are unpacked into the `/tmp/s2i` directory.

```bash title="assemble script:"
#!/bin/bash

# restore build artifacts
if [ "$(ls /tmp/s2i/artifacts/ 2>/dev/null)" ]; then
    mv /tmp/s2i/artifacts/* $HOME/.
fi

# move the application source
mv /tmp/s2i/src $HOME/src

# build application artifacts
pushd ${HOME}
make all

# install the artifacts
make install
popd
```

```bash title="run script:"
#!/bin/bash

# run the application
/opt/application/run.sh
```

```bash title="save-artifacts script:"
#!/bin/bash

pushd ${HOME}
if [ -d deps ]; then
    # all deps contents to tar stream
    tar cf - deps
fi
popd
```

```bash title="usage script:"
#!/bin/bash

# inform the user how to use the image
cat <<EOF
This is a S2I sample builder image, to use it, install
https://github.com/openshift/source-to-image
EOF
```

**Additional resources**

- [S2I Image Creation Tutorial](https://blog.openshift.com/create-s2i-builder-image/)

### Using build volumes {#builds-using-build-volumes_build-strategies-s2i}

You can mount build volumes to give running builds access to information that you do not want to persist in the output container image.

Build volumes provide sensitive information, such as repository credentials, that the build environment or configuration only needs at build time. Build volumes are different from build inputs, whose data can persist in the output container image.

The mount points of build volumes, from which the running build reads data, are functionally similar to [pod volume mounts](https://kubernetes.io/docs/concepts/storage/volumes/).

**Prerequisites**

- You have added an input secret, config map, or both to a BuildConfig object.

**Procedure**

- In the `sourceStrategy` definition of the `BuildConfig` object, add any build volumes to the `volumes` array. For example:
  ```yaml
  spec:
    sourceStrategy:
      volumes:
        - name: secret-mvn
          mounts:
          - destinationPath: /opt/app-root/src/.ssh
          source:
            type: Secret
            secret:
              secretName: my-secret
        - name: settings-mvn
          mounts:
          - destinationPath: /opt/app-root/src/.m2
          source:
            type: ConfigMap
            configMap:
              name: my-config
        - name: my-csi-volume
          mounts:
          - destinationPath: /opt/app-root/src/some_path
          source:
            type: CSI
            csi:
              driver: csi.sharedresource.openshift.io
              readOnly: true
              volumeAttributes:
                attribute: value
  ```

  where:

  <dl>
  <dt><code>name</code></dt>
  <dd>Specifies a unique name.</dd>
  <dt><code>destinationPath</code></dt>
  <dd>Specifies the absolute path of the mount point. It must not contain <code>..</code> or <code>:</code> and does not collide with the destination path generated by the builder. The <code>/opt/app-root/src</code> is the default home directory for many Red Hat S2I-enabled images.</dd>
  <dt><code>type</code></dt>
  <dd>Specifies the type of source: <code>ConfigMap</code>, <code>Secret</code>, or <code>CSI</code>.</dd>
  <dt><code>secretName</code></dt>
  <dd>Specifies the name of the source.</dd>
  <dt><code>driver</code></dt>
  <dd>Specifies the driver that provides the ephemeral CSI volume.</dd>
  <dt><code>readOnly</code></dt>
  <dd>Specifies that a read-only volume is provided. This value must be set to <code>true</code>.</dd>
  <dt><code>volumeAttributes</code></dt>
  <dd>(Optional) Specifies the volume attributes of the ephemeral CSI volume. Consult the CSI driver’s documentation for supported attribute keys and values.</dd>
  </dl>

**Additional resources**

- [Build inputs](/docs/cicd/builds/creating-build-inputs#builds-define-build-inputs_creating-build-inputs)
- [Input secrets and config maps](/docs/cicd/builds/creating-build-inputs#builds-input-secrets-configmaps_creating-build-inputs)

## Custom build {#builds-strategy-custom-build_build-strategies-s2i}

The custom build strategy allows developers to define a specific builder image responsible for the entire build process. Using your own builder image allows you to customize your build process.

A custom builder image is a plain container image embedded with build process logic, for example for building RPMs or base images.

Custom builds run with a high level of privilege and are not available to users by default. Only users who can be trusted with cluster administration permissions should be granted access to run custom builds.

### Using FROM image for custom builds {#builds-strategy-custom-from-image_build-strategies-s2i}

You can use the `customStrategy.from` section to indicate the image to use for the custom build.

**Procedure**

- Set the `customStrategy.from` section:
  ```yaml
  strategy:
    customStrategy:
      from:
        kind: "DockerImage"
        name: "openshift/sti-image-builder"
  ```

### Using secrets in custom builds {#builds-strategy-custom-secrets_build-strategies-s2i}

In addition to secrets for source and images that can be added to all build types, custom strategies allow adding an arbitrary list of secrets to the builder pod.

**Procedure**

- To mount each secret at a specific location, edit the `secretSource` and `mountPath` fields of the `strategy` YAML file:
  ```yaml
  strategy:
    customStrategy:
      secrets:
        - secretSource: (1)
            name: "secret1"
          mountPath: "/tmp/secret1" (2)
        - secretSource:
            name: "secret2"
          mountPath: "/tmp/secret2"
  ```

  1. `secretSource` is a reference to a secret in the same namespace as the build.
  2. `mountPath` is the path inside the custom builder where the secret should be mounted.

### Using environment variables for custom builds {#builds-strategy-custom-environment-variables_build-strategies-s2i}

To make environment variables available to the custom build process, you can add environment variables to the `customStrategy` definition of the build configuration.

The environment variables defined there are passed to the pod that runs the custom build.

**Procedure**

1. Define a custom HTTP proxy to be used during build:
   ```yaml
   customStrategy:
   ...
     env:
       - name: "HTTP_PROXY"
         value: "http://myproxy.net:5187/"
   ```
2. To manage environment variables defined in the build configuration, enter the following command:
   ```terminal
   $ oc set env <enter_variables>
   ```

### Using custom builder images {#images-custom_build-strategies-s2i}

OpenShift Container Platform's custom build strategy enables you to define a specific builder image responsible for the entire build process. When you need a build to produce individual artifacts such as packages, JARs, WARs, installable ZIPs, or base images, use a custom builder image using the custom build strategy.

A custom builder image is a plain container image embedded with build process logic, which is used for building artifacts such as RPMs or base container images.

Additionally, the custom builder allows implementing any extended build process, such as a CI/CD flow that runs unit or integration tests.

#### Custom builder image {#images-custom-builder-image-ref_build-strategies-s2i}

Upon invocation, a custom builder image receives the following environment variables with the information needed to proceed with the build:

**Custom Builder Environment Variables**

| Variable Name | Description |
| --- | --- |
| `BUILD` | The entire serialized JSON of the `Build` object definition. If you must use a specific API version for serialization, you can set the `buildAPIVersion` parameter in the custom strategy specification of the build configuration. |
| `SOURCE_REPOSITORY` | The URL of a Git repository with source to be built. |
| `SOURCE_URI` | Uses the same value as `SOURCE_REPOSITORY`. Either can be used. |
| `SOURCE_CONTEXT_DIR` | Specifies the subdirectory of the Git repository to be used when building. Only present if defined. |
| `SOURCE_REF` | The Git reference to be built. |
| `ORIGIN_VERSION` | The version of the OpenShift Container Platform master that created this build object. |
| `OUTPUT_REGISTRY` | The container image registry to push the image to. |
| `OUTPUT_IMAGE` | The container image tag name for the image being built. |
| `PUSH_DOCKERCFG_PATH` | The path to the container registry credentials for running a `podman push`  operation. |

#### Custom builder workflow {#images-custom-builder-flow_build-strategies-s2i}

Although custom builder image authors have flexibility in defining the build process, your builder image must adhere to the following required steps necessary for running a build inside of OpenShift Container Platform:

1. The `Build` object definition contains all the necessary information about input parameters for the build.
2. Run the build process.
3. If your build produces an image, push it to the output location of the build if it is defined. Other output locations can be passed with environment variables.

## Pipeline build {#builds-strategy-pipeline-build_build-strategies-s2i}

:::warning

The Pipeline build strategy is deprecated in OpenShift Container Platform 4. Equivalent and improved functionality is present in the OpenShift Container Platform Pipelines based on Tekton.

Jenkins images on OpenShift Container Platform are fully supported and users should follow Jenkins user documentation for defining their `jenkinsfile` in a job or store it in a Source Control Management system.

:::

The Pipeline build strategy allows developers to define a Jenkins pipeline for use by the Jenkins pipeline plugin. The build can be started, monitored, and managed by OpenShift Container Platform in the same way as any other build type.

Pipeline workflows are defined in a `jenkinsfile`, either embedded directly in the build configuration, or supplied in a Git repository and referenced by the build configuration.

### Understanding OpenShift Container Platform pipelines {#builds-understanding-openshift-pipeline_build-strategies-s2i}

:::warning

The Pipeline build strategy is deprecated in OpenShift Container Platform 4. Equivalent and improved functionality is present in the OpenShift Container Platform Pipelines based on Tekton.

Jenkins images on OpenShift Container Platform are fully supported and users should follow Jenkins user documentation for defining their `jenkinsfile` in a job or store it in a Source Control Management system.

:::

Pipelines give you control over building, deploying, and promoting your applications on OpenShift Container Platform. Using a combination of the Jenkins Pipeline build strategy, `jenkinsfiles`, and the OpenShift Container Platform Domain Specific Language (DSL) provided by the Jenkins Client Plugin, you can create advanced build, test, deploy, and promote pipelines for any scenario.

**OpenShift Container Platform Jenkins Sync Plugin**

The OpenShift Container Platform Jenkins Sync Plugin keeps the build configuration and build objects in sync with Jenkins jobs and builds, and provides the following:

- Dynamic job and run creation in Jenkins.
- Dynamic creation of agent pod templates from image streams, image stream tags, or config maps.
- Injection of environment variables.
- Pipeline visualization in the OpenShift Container Platform web console.
- Integration with the Jenkins Git plugin, which passes commit information from OpenShift Container Platform builds to the Jenkins Git plugin.
- Synchronization of secrets into Jenkins credential entries.

**OpenShift Container Platform Jenkins Client Plugin**

The OpenShift Container Platform Jenkins Client Plugin is a Jenkins plugin which aims to provide a readable, concise, comprehensive, and fluent Jenkins Pipeline syntax for rich interactions with the OpenShift Container Platform API Server. The plugin uses the OpenShift Container Platform command-line tool, `oc`, which must be available on the nodes executing the script.

The Jenkins Client Plugin must be installed on your Jenkins master so the OpenShift Container Platform DSL will be available to use within the `jenkinsfile` for your application. This plugin is installed and enabled by default when using the OpenShift Container Platform Jenkins image.

For OpenShift Container Platform Pipelines within your project, you will must use the Jenkins Pipeline Build Strategy. This strategy defaults to using a `jenkinsfile` at the root of your source repository, but also provides the following configuration options:

- An inline `jenkinsfile` field within your build configuration.
- A `jenkinsfilePath` field within your build configuration that references the location of the `jenkinsfile` to use relative to the source `contextDir`.

:::note

The optional `jenkinsfilePath` field specifies the name of the file to use, relative to the source `contextDir`. If `contextDir` is omitted, it defaults to the root of the repository. If `jenkinsfilePath` is omitted, it defaults to `jenkinsfile`.

:::

### Providing the Jenkins file for pipeline builds {#builds-strategy-pipeline-providing-jenkinsfile_build-strategies-s2i}

:::warning

The Pipeline build strategy is deprecated in OpenShift Container Platform 4. Equivalent and improved functionality is present in the OpenShift Container Platform Pipelines based on Tekton.

Jenkins images on OpenShift Container Platform are fully supported and users should follow Jenkins user documentation for defining their `jenkinsfile` in a job or store it in a Source Control Management system.

:::

The `jenkinsfile` uses the standard groovy language syntax to allow fine grained control over the configuration, build, and deployment of your application.

You can supply the `jenkinsfile` in one of the following ways:

- A file located within your source code repository.
- Embedded as part of your build configuration using the `jenkinsfile` field.

When using the first option, the `jenkinsfile` must be included in your applications source code repository at one of the following locations:

- A file named `jenkinsfile` at the root of your repository.
- A file named `jenkinsfile` at the root of the source `contextDir` of your repository.
- A file name specified via the `jenkinsfilePath` field of the `JenkinsPipelineStrategy` section of your BuildConfig, which is relative to the source `contextDir` if supplied, otherwise it defaults to the root of the repository.

The `jenkinsfile` is run on the Jenkins agent pod, which must have the OpenShift Container Platform client binaries available if you intend to use the OpenShift Container Platform DSL.

**Procedure**

To provide the Jenkins file, you can either:

- Embed the Jenkins file in the build configuration.
- Include in the build configuration a reference to the Git repository that contains the Jenkins file.

```yaml title="Embedded Definition"
kind: "BuildConfig"
apiVersion: "v1"
metadata:
  name: "sample-pipeline"
spec:
  strategy:
    jenkinsPipelineStrategy:
      jenkinsfile: |-
        node('agent') {
          stage 'build'
          openshiftBuild(buildConfig: 'ruby-sample-build', showBuildLogs: 'true')
          stage 'deploy'
          openshiftDeploy(deploymentConfig: 'frontend')
        }
```

```yaml title="Reference to Git Repository"
kind: "BuildConfig"
apiVersion: "v1"
metadata:
  name: "sample-pipeline"
spec:
  source:
    git:
      uri: "https://github.com/openshift/ruby-hello-world"
  strategy:
    jenkinsPipelineStrategy:
      jenkinsfilePath: some/repo/dir/filename (1)
```

```
1.  The optional `jenkinsfilePath` field specifies the name of the file to use, relative to the source `contextDir`. If `contextDir` is omitted, it defaults to the root of the repository. If `jenkinsfilePath` is omitted, it defaults to `jenkinsfile`.
```

### Using environment variables for pipeline builds {#builds-strategy-pipeline-environment-variables_build-strategies-s2i}

:::warning

The Pipeline build strategy is deprecated in OpenShift Container Platform 4. Equivalent and improved functionality is present in the OpenShift Container Platform Pipelines based on Tekton.

Jenkins images on OpenShift Container Platform are fully supported and users should follow Jenkins user documentation for defining their `jenkinsfile` in a job or store it in a Source Control Management system.

:::

To make environment variables available to the Pipeline build process, you can add environment variables to the `jenkinsPipelineStrategy` definition of the build configuration.

Once defined, the environment variables will be set as parameters for any Jenkins job associated with the build configuration.

**Procedure**

- To define environment variables to be used during build, edit the YAML file:
  ```yaml
  jenkinsPipelineStrategy:
  ...
    env:
      - name: "FOO"
        value: "BAR"
  ```

You can also manage environment variables defined in the build configuration with the `oc set env` command.

#### Mapping between BuildConfig environment variables and Jenkins job parameters {#builds-strategy-pipeline-mapping-buildconfig-jenkins_build-strategies-s2i}

When a Jenkins job is created or updated based on changes to a Pipeline strategy build configuration, any environment variables in the build configuration are mapped to Jenkins job parameters definitions, where the default values for the Jenkins job parameters definitions are the current values of the associated environment variables.

After the Jenkins job’s initial creation, you can still add additional parameters to the job from the Jenkins console. The parameter names differ from the names of the environment variables in the build configuration. The parameters are honored when builds are started for those Jenkins jobs.

How you start builds for the Jenkins job dictates how the parameters are set.

- If you start with `oc start-build`, the values of the environment variables in the build configuration are the parameters set for the corresponding job instance. Any changes you make to the parameters' default values from the Jenkins console are ignored. The build configuration values take precedence.
- If you start with `oc start-build -e`, the values for the environment variables specified in the `-e` option take precedence.
  - If you specify an environment variable not listed in the build configuration, they will be added as a Jenkins job parameter definitions.
  - Any changes you make from the Jenkins console to the parameters corresponding to the environment variables are ignored. The build configuration and what you specify with `oc start-build -e` takes precedence.
- If you start the Jenkins job with the Jenkins console, then you can control the setting of the parameters with the Jenkins console as part of starting a build for the job.

:::note

It is recommended that you specify in the build configuration all possible environment variables to be associated with job parameters. Doing so reduces disk I/O and improves performance during Jenkins processing.

:::

### Pipeline build tutorial {#builds-tutorial-pipeline_build-strategies-s2i}

:::warning

The Pipeline build strategy is deprecated in OpenShift Container Platform 4. Equivalent and improved functionality is present in the OpenShift Container Platform Pipelines based on Tekton.

Jenkins images on OpenShift Container Platform are fully supported and users should follow Jenkins user documentation for defining their `jenkinsfile` in a job or store it in a Source Control Management system.

:::

This example demonstrates how to create an OpenShift Container Platform Pipeline that will build, deploy, and verify a `Node.js/MongoDB` application using the `nodejs-mongodb.json` template.

**Procedure**

1. Create the Jenkins master:
   ```terminal
     $ oc project <project_name>
   ```

   Select the project that you want to use or create a new project with `oc new-project <project_name>`.

   ```terminal
     $ oc new-app jenkins-ephemeral (2)
   ```

   If you want to use persistent storage, use `jenkins-persistent` instead.
2. Create a file named `nodejs-sample-pipeline.yaml` with the following content:
   :::note

   This creates a `BuildConfig` object that employs the Jenkins pipeline strategy to build, deploy, and scale the `Node.js/MongoDB` example application.

   :::

   ```yaml
   kind: "BuildConfig"
   apiVersion: "v1"
   metadata:
     name: "nodejs-sample-pipeline"
   spec:
     strategy:
       jenkinsPipelineStrategy:
         jenkinsfile: <pipeline content from below>
       type: JenkinsPipeline
   ```
3. After you create a `BuildConfig` object with a `jenkinsPipelineStrategy`, tell the pipeline what to do by using an inline `jenkinsfile`:
   :::note

   This example does not set up a Git repository for the application.

   The following `jenkinsfile` content is written in Groovy using the OpenShift Container Platform DSL. For this example, include inline content in the `BuildConfig` object using the YAML Literal Style, though including a `jenkinsfile` in your source repository is the preferred method.

   :::

   ```groovy
   def templatePath = 'https://raw.githubusercontent.com/openshift/nodejs-ex/master/openshift/templates/nodejs-mongodb.json' (1)
   def templateName = 'nodejs-mongodb-example' (2)
   pipeline {
     agent {
       node {
         label 'nodejs' (3)
       }
     }
     options {
       timeout(time: 20, unit: 'MINUTES') (4)
     }
     stages {
       stage('preamble') {
           steps {
               script {
                   openshift.withCluster() {
                       openshift.withProject() {
                           echo "Using project: ${openshift.project()}"
                       }
                   }
               }
           }
       }
       stage('cleanup') {
         steps {
           script {
               openshift.withCluster() {
                   openshift.withProject() {
                     openshift.selector("all", [ template : templateName ]).delete() (5)
                     if (openshift.selector("secrets", templateName).exists()) { (6)
                       openshift.selector("secrets", templateName).delete()
                     }
                   }
               }
           }
         }
       }
       stage('create') {
         steps {
           script {
               openshift.withCluster() {
                   openshift.withProject() {
                     openshift.newApp(templatePath) (7)
                   }
               }
           }
         }
       }
       stage('build') {
         steps {
           script {
               openshift.withCluster() {
                   openshift.withProject() {
                     def builds = openshift.selector("bc", templateName).related('builds')
                     timeout(5) { (8)
                       builds.untilEach(1) {
                         return (it.object().status.phase == "Complete")
                       }
                     }
                   }
               }
           }
         }
       }
       stage('deploy') {
         steps {
           script {
               openshift.withCluster() {
                   openshift.withProject() {
                     def rm = openshift.selector("dc", templateName).rollout()
                     timeout(5) { (9)
                       openshift.selector("dc", templateName).related('pods').untilEach(1) {
                         return (it.object().status.phase == "Running")
                       }
                     }
                   }
               }
           }
         }
       }
       stage('tag') {
         steps {
           script {
               openshift.withCluster() {
                   openshift.withProject() {
                     openshift.tag("${templateName}:latest", "${templateName}-staging:latest") (10)
                   }
               }
           }
         }
       }
     }
   }
   ```

   1. Path of the template to use.
   2. Name of the template that will be created.
   3. Spin up a `node.js` agent pod on which to run this build.
   4. Set a timeout of 20 minutes for this pipeline.
   5. Delete everything with this template label.
   6. Delete any secrets with this template label.
   7. Create a new application from the `templatePath`.
   8. Wait up to five minutes for the build to complete.
   9. Wait up to five minutes for the deployment to complete.
   10. If everything else succeeded, tag the `$ {templateName}:latest` image as `$ {templateName}-staging:latest`. A pipeline build configuration for the staging environment can watch for the `$ {templateName}-staging:latest` image to change and then deploy it to the staging environment.

   :::note

   The previous example was written using the declarative pipeline style, but the older scripted pipeline style is also supported.

   :::
4. Create the Pipeline `BuildConfig` in your OpenShift Container Platform cluster:
   ```terminal
   $ oc create -f nodejs-sample-pipeline.yaml
   ```

   1. If you do not want to create your own file, you can use the sample from the Origin repository by running:
      ```terminal
      $ oc create -f https://raw.githubusercontent.com/openshift/origin/master/examples/jenkins/pipeline/nodejs-sample-pipeline.yaml
      ```
5. Start the Pipeline:
   ```terminal
   $ oc start-build nodejs-sample-pipeline
   ```

   :::note

   Alternatively, you can start your pipeline with the OpenShift Container Platform web console by navigating to the Builds → Pipeline section and clicking **Start Pipeline**, or by visiting the Jenkins Console, navigating to the Pipeline that you created, and clicking **Build Now**.

   :::

   Once the pipeline is started, you should see the following actions performed within your project:

   - A job instance is created on the Jenkins server.
   - An agent pod is launched, if your pipeline requires one.
   - The pipeline runs on the agent pod, or the master if no agent is required.
     - Any previously created resources with the `template=nodejs-mongodb-example` label will be deleted.
     - A new application, and all of its associated resources, will be created from the `nodejs-mongodb-example` template.
     - A build will be started using the `nodejs-mongodb-example` `BuildConfig`.
       - The pipeline will wait until the build has completed to trigger the next stage.
     - A deployment will be started using the `nodejs-mongodb-example` deployment configuration.
       - The pipeline will wait until the deployment has completed to trigger the next stage.
     - If the build and deploy are successful, the `nodejs-mongodb-example:latest` image will be tagged as `nodejs-mongodb-example:stage`.
   - The agent pod is deleted, if one was required for the pipeline.
     :::note

     The best way to visualize the pipeline execution is by viewing it in the OpenShift Container Platform web console. You can view your pipelines by logging in to the web console and navigating to Builds → Pipelines.

     :::

## Adding secrets with web console {#builds-strategy-secrets-web-console_build-strategies-s2i}

You can add a secret to your build configuration so that it can access a private repository.

**Procedure**

To add a secret to your build configuration so that it can access a private repository from the OpenShift Container Platform web console:

1. Create a new OpenShift Container Platform project.
2. Create a secret that contains credentials for accessing a private source code repository.
3. Create a build configuration.
4. On the build configuration editor page or in the `create app from builder image` page of the web console, set the **Source Secret**.
5. Click **Save**.

## Enabling pulling and pushing {#builds-strategy-enable-pulling-pushing_build-strategies-s2i}

You can enable pulling to a private registry by setting the pull secret and pushing by setting the push secret in the build configuration.

**Procedure**

To enable pulling to a private registry:

- Set the pull secret in the build configuration.

To enable pushing:

- Set the push secret in the build configuration.
