---
title: Creating applications by using the CLI
sidebar_position: 4
---

# Creating applications by using the CLI {#creating-applications-using-cli}

<a id="creating-applications-using-cli"></a>

You can create applications on your OpenShift Container Platform cluster from a Git repository, a container image, or a template using the `oc new-app` command. Customize names, labels, environment variables, target projects, and other deployment options with command flags.

## Creating an application from source code {#applications-create-using-cli-source-code_creating-applications-using-cli}

You can create an application on your OpenShift Container Platform cluster from a local or remote Git repository using the `oc new-app` command. Use command flags to target a specific branch or subdirectory, authenticate to a private repository, or control the build strategy and builder image.

**Prerequisites**

- You have installed the OpenShift CLI (`oc`) and logged in to your cluster.
- You have access to a Git repository containing your application source code.

**Procedure**

1. Create an application from a Git repository in a local directory by running the following command:
   ```terminal
   $ oc new-app /<path_to_source_code>
   ```

   :::note

   If you use a local Git repository, the repository must have a remote named `origin` that points to a URL that is accessible by the OpenShift Container Platform cluster. If there is no recognized remote, running the `new-app` command creates a binary build.

   :::
2. Create an application from a public remote Git repository by running the following command:
   ```terminal
   $ oc new-app https://github.com/sclorg/cakephp-ex
   ```
3. Create an application from a private remote Git repository by running the following command:
   ```terminal
   $ oc new-app https://github.com/<your_user>/<your_private_repo> --source-secret=yoursecret
   ```

   :::note

   If you use a private remote Git repository, use the `--source-secret` flag to specify a source clone secret for access to the repository.

   :::
4. Use a subdirectory of your source repository by running the following command:
   ```terminal
   $ oc new-app https://github.com/sclorg/s2i-ruby-container.git \
       --context-dir=2.0/test/puma-test-app
   ```
5. Specify a Git branch by running the following command:
   ```terminal
   $ oc new-app https://github.com/openshift/ruby-hello-world.git#beta4
   ```
6. Override the automatically detected build strategy by running the following command:
   ```terminal
   $ oc new-app /home/user/code/myapp --strategy=docker
   ```

   :::note

   The `oc` command requires that files containing build sources are available in a remote Git repository. For all source builds, you must use `git remote -v`.

   :::
7. Specify the builder image and source repository:
   1. Specify the builder image and source repository for a remote repository by running the following command:
      ```terminal
      $ oc new-app myproject/my-ruby~https://github.com/openshift/ruby-hello-world.git
      ```
   2. Specify the builder image and source repository for a local repository by running the following command:
      ```terminal
      $ oc new-app openshift/ruby-20-centos7:latest~/home/user/code/my-ruby-app
      ```

## Build strategy and language detection for source applications {#applications-create-using-cli-source-reference_creating-applications-using-cli}

You can determine which build strategy and language builder the `oc new-app` command selects by reviewing files in the root or context directory of your Git repository. Use these detection rules to override the build strategy or specify a builder image when automatic detection does not apply.

### Build strategy detection {#build-strategy-detection_creating-applications-using-cli}

OpenShift Container Platform automatically determines which build strategy to use by detecting certain files:

- If a `Jenkinsfile` exists in the root or specified context directory of the source repository when creating a new application, OpenShift Container Platform generates a pipeline build strategy.
  :::note

  The `pipeline` build strategy is deprecated; consider using Red Hat OpenShift Pipelines instead.

  :::
- If a `Dockerfile` exists in the root or specified context directory of the source repository when creating a new application, OpenShift Container Platform generates a docker build strategy.
- If neither a `Jenkinsfile` nor a `Dockerfile` is detected, OpenShift Container Platform generates a source build strategy.

### Language detection {#language-detection_creating-applications-using-cli}

If you use the source build strategy, `new-app` detects the language builder from certain files in the root or context directory of the repository.

**Languages detected by **`new-app`

<table>
<thead>
<tr>
  <th>Language</th>
  <th>Files</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>dotnet</code></td>
  <td><code>project.json</code>, <code>pass:[*.csproj]</code></td>
</tr>
<tr>
  <td><code>jee</code></td>
  <td><code>pom.xml</code></td>
</tr>
<tr>
  <td><code>nodejs</code></td>
  <td><code>app.json</code>, <code>package.json</code></td>
</tr>
<tr>
  <td><code>perl</code></td>
  <td><code>cpanfile</code>, <code>index.pl</code></td>
</tr>
<tr>
  <td><code>php</code></td>
  <td><code>composer.json</code>, <code>index.php</code></td>
</tr>
<tr>
  <td><code>python</code></td>
  <td><code>requirements.txt</code>, <code>setup.py</code></td>
</tr>
<tr>
  <td><code>ruby</code></td>
  <td><code>Gemfile</code>, <code>Rakefile</code>, <code>config.ru</code></td>
</tr>
<tr>
  <td><code>scala</code></td>
  <td><code>build.sbt</code></td>
</tr>
<tr>
  <td><code>golang</code></td>
  <td><code>Godeps</code>, <code>main.go</code></td>
</tr>
</tbody>
</table>

After a language is detected, the `new-app` command searches the OpenShift Container Platform server for image stream tags with a matching `supports` annotation or image streams that match the language name. If a match is not found, the `new-app` command searches the Docker Hub registry for an image that matches the detected language based on name.

When you specify an image and repository with the `~` separator, build strategy detection and language detection are not carried out.

:::note

Language detection requires the Git client to be locally installed so that your repository can be cloned and inspected. If Git is not available, you can avoid the language detection step by specifying the builder image to use with your repository with the `<image>~<repository>` syntax.

The `-i <image> <repository>` invocation requires that `new-app` attempt to clone `repository` to determine what type of artifact it is, so the command fails if Git is not available.

The `-i <image> --code <repository>` invocation requires that `new-app` clone `repository` to learn whether `image` is a builder for the source or a separate deployment, such as a database image.

:::

## Creating an application from an image {#applications-create-using-cli-image_creating-applications-using-cli}

You can use the `oc new-app` command to create an application from a container image in Docker Hub, a private registry, or an image stream on your cluster. Use this procedure when you know the container image name or image stream you want to deploy.

Use the command that matches where your container image is stored.

:::note

If you specify an image from your local Docker repository, you must ensure that the same image is available to the OpenShift Container Platform cluster nodes.

:::

**Prerequisites**

- You have installed the OpenShift CLI (`oc`) and logged in to your cluster.
- You know the container image name or image stream you want to deploy.

**Procedure**

- Create an application from the Docker Hub MySQL image by running the following command:
  ```terminal
  $ oc new-app mysql
  ```
- Create an application from an image in a private registry by specifying the full image path in the following command:
  ```terminal
  $ oc new-app myregistry:5000/example/myimage
  ```
- Create an application from an existing image stream and optional image stream tag by running the following command:
  ```terminal
  $ oc new-app my-stream:v1
  ```

**Additional resources**

- [Docker Hub registry (Docker)](https://registry.hub.docker.com)

## Creating an application from a template {#applications-create-using-cli-template_creating-applications-using-cli}

You can use the `oc new-app` command to create an application from a template stored in your project or from a template file on your local system. Use this procedure when you have a template JSON or YAML file, or a template in the template library of your current project.

**Prerequisites**

- You have installed the OpenShift CLI (`oc`) and logged in to your cluster.
- You have a template JSON or YAML file, or a template stored in the template library of your current project.

**Procedure**

1. Upload an application template to the template library of your current project by running the following command:
   ```terminal
   $ oc create -f examples/sample-app/application-template-stibuild.json
   ```
2. Create a new application from a stored template by running the following command:
   ```terminal
   $ oc new-app ruby-helloworld-sample
   ```
3. Create a new application from a template file on your local file system without storing it in OpenShift Container Platform by running the following command:
   ```terminal
   $ oc new-app -f examples/sample-app/application-template-stibuild.json
   ```
4. Set template parameter values when creating an application by running the following command:
   ```terminal
   $ oc new-app ruby-helloworld-sample \
       -p ADMIN_USERNAME=admin -p ADMIN_PASSWORD=mypassword
   ```
5. Store template parameters in a file by creating a file such as `helloworld.params` with the following content:
   ```terminal
   ADMIN_USERNAME=admin
   ADMIN_PASSWORD=mypassword
   ```

   You can store your parameters in a file, then use that file with `--param-file` when instantiating a template. If you want to read the parameters from standard input, use `--param-file=-`.
6. Create a new application from a template by using a parameter file by running the following command:
   ```terminal
   $ oc new-app ruby-helloworld-sample --param-file=helloworld.params
   ```

   :::note

   To read parameters from standard input, use `--param-file=-`.

   :::

## Customization options for application creation {#applications-create-using-cli-modify_creating-applications-using-cli}

You can customize how the `oc new-app` command creates applications by setting names, labels, environment variables, target projects, and other options. Use these flags to control the objects the command generates before you deploy.

`new-app`** output objects**

<table>
<thead>
<tr>
  <th>Object</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>BuildConfig</code></td>
  <td>A <code>BuildConfig</code> object is created for each source repository that is specified in the command line. The <code>BuildConfig</code> object specifies the strategy to use, the source location, and the build output location.</td>
</tr>
<tr>
  <td><code>ImageStreams</code></td>
  <td>For the <code>BuildConfig</code> object, two image streams are usually created. One represents the input image. With source builds, this is the builder image. With <code>Docker</code> builds, this is the <strong>FROM</strong> image. The second one represents the output image. If a container image was specified as input to <code>new-app</code>, then an image stream is created for that image as well.</td>
</tr>
<tr>
  <td><code>DeploymentConfig</code></td>
  <td>A <code>DeploymentConfig</code> object is created either to deploy the output of a build, or a specified image. The <code>new-app</code> command creates <code>emptyDir</code> volumes for all Docker volumes that are specified in containers included in the resulting <code>DeploymentConfig</code> object.</td>
</tr>
<tr>
  <td><code>Service</code></td>
  <td>The <code>new-app</code> command attempts to detect exposed ports in input images. It uses the lowest numeric exposed port to generate a service that exposes that port. To expose a different port, after <code>new-app</code> has completed, use the <code>oc expose</code> command to generate additional services.</td>
</tr>
<tr>
  <td>Other</td>
  <td>Other objects can be generated when creating applications from templates, according to the template.</td>
</tr>
</tbody>
</table>

### Specifying environment variables {#specifying-environment-variables_creating-applications-using-cli}

When generating applications from a template, source, or an image, you can use the `-e|--env` argument to pass environment variables to the application container at run time.

```terminal
$ oc new-app openshift/postgresql-92-centos7 \
    -e POSTGRESQL_USER=user \
    -e POSTGRESQL_DATABASE=db \
    -e POSTGRESQL_PASSWORD=password
```

The variables can also be read from file using the `--env-file` argument. The following is an example file called `postgresql.env`:

```terminal
POSTGRESQL_USER=user
POSTGRESQL_DATABASE=db
POSTGRESQL_PASSWORD=password
```

Read the variables from the file:

```terminal
$ oc new-app openshift/postgresql-92-centos7 --env-file=postgresql.env
```

Additionally, environment variables can be given on standard input by using the `--env-file=-` argument:

```terminal
$ cat postgresql.env | oc new-app openshift/postgresql-92-centos7 --env-file=-
```

:::note

Any `BuildConfig` objects created as part of `new-app` processing are not updated with environment variables passed with the `-e|--env` or `--env-file` argument.

:::

### Specifying build environment variables {#specifying-build-environment-variables_creating-applications-using-cli}

When generating applications from a template, source, or an image, you can use the `--build-env` argument to pass environment variables to the build container at run time:

```terminal
$ oc new-app openshift/ruby-23-centos7 \
    --build-env HTTP_PROXY=http://myproxy.net:1337/ \
    --build-env GEM_HOME=~/.gem
```

The variables can also be read from a file using the `--build-env-file` argument. The following is an example file called `ruby.env`:

```terminal
HTTP_PROXY=http://myproxy.net:1337/
GEM_HOME=~/.gem
```

Read the variables from the file:

```terminal
$ oc new-app openshift/ruby-23-centos7 --build-env-file=ruby.env
```

Additionally, environment variables can be given on standard input by using `--build-env-file=-`:

```terminal
$ cat ruby.env | oc new-app openshift/ruby-23-centos7 --build-env-file=-
```

### Specifying labels {#specifying-labels_creating-applications-using-cli}

When generating applications from source, images, or templates, you can use the `-l|--label` argument to add labels to the created objects. Labels make it easy to collectively select, configure, and delete objects associated with the application.

```terminal
$ oc new-app https://github.com/openshift/ruby-hello-world -l name=hello-world
```

### Viewing the output without creation {#viewing-output-without-creation_creating-applications-using-cli}

You can preview objects without creating them by using `-o` or `--output` with a `yaml` or `json` value. Redirect the output to a file, edit the file, then create the objects with `oc create`.

```terminal title="Writing new-app output to a file"
$ oc new-app https://github.com/openshift/ruby-hello-world \
    -o yaml > myapp.yaml
```

```terminal title="Creating objects from an edited file"
$ oc create -f myapp.yaml
```

### Creating objects with different names {#creating-objects-different-names_creating-applications-using-cli}

Objects created by `new-app` are normally named after the source repository, or the image used to generate them. You can set the name of the objects produced by adding a `--name` flag to the command:

```terminal
$ oc new-app https://github.com/openshift/ruby-hello-world --name=myapp
```

### Creating objects in a different project {#creating-objects-different-project_creating-applications-using-cli}

Normally, `new-app` creates objects in the current project. However, you can create objects in a different project by using the `-n|--namespace` argument:

```terminal
$ oc new-app https://github.com/openshift/ruby-hello-world -n myproject
```

### Creating multiple objects {#creating-multiple-objects_creating-applications-using-cli}

You can create multiple applications by specifying multiple parameters to `new-app`. Labels specified in the command line apply to all objects created by the single command. Environment variables apply to all components created from source or images.

To create an application from a source repository and a Docker Hub image:

```terminal
$ oc new-app https://github.com/openshift/ruby-hello-world mysql
```

:::note

If a source code repository and a builder image are specified as separate arguments, `new-app` uses the builder image as the builder for the source code repository. If this is not the intent, specify the required builder image for the source using the `~` separator.

:::

### Grouping images and source in a single pod {#grouping-images-source-single-pod_creating-applications-using-cli}

You can deploy multiple images together in a single pod. To specify which images to group together, use the `+` separator. The `--group` command-line argument can also be used to specify the images that should be grouped together. To group the image built from a source repository with other images, specify the builder image for the source in the group:

```terminal
$ oc new-app ruby+mysql
```

To deploy an image built from source and an external image together:

```terminal
$ oc new-app \
    ruby~https://github.com/openshift/ruby-hello-world \
    mysql \
    --group=ruby+mysql
```

### Searching for images, templates, and other inputs {#searching-for-images-templates-other-inputs_creating-applications-using-cli}

To search for images, templates, and other inputs for the `oc new-app` command, add the `--search` and `--list` flags. For example, to find all of the images or templates that include PHP:

```terminal
$ oc new-app --search php
```

### Setting the import mode {#setting-the-import-mode_creating-applications-using-cli}

To set the import mode when using `oc new-app`, add the `--import-mode` flag. This flag can be appended with `Legacy` or `PreserveOriginal`, which provides users the option to create image streams using a single sub-manifest, or all manifests, respectively.

```terminal
$ oc new-app --image=registry.redhat.io/ubi8/httpd-24:latest --import-mode=Legacy --name=test
```

```terminal
$ oc new-app --image=registry.redhat.io/ubi8/httpd-24:latest --import-mode=PreserveOriginal --name=test
```
