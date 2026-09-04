---
title: Jenkins agent
sidebar_position: 2
---

# Jenkins agent {#images-other-jenkins-agent}

<a id="images-other-jenkins-agent"></a>

OpenShift Container Platform provides a base image for use as a Jenkins agent.

The Base image for Jenkins agents does the following:

- Pulls in both the required tools, headless Java, the Jenkins JNLP client, and the useful ones, including `git`, `tar`, `zip`, and `nss`, among others.
- Establishes the JNLP agent as the entry point.
- Includes the `oc` client tool for invoking command-line operations from within Jenkins jobs.
- Provides Dockerfiles for both Red Hat Enterprise Linux (RHEL) and `localdev` images.

:::warning

Use a version of the agent image that is appropriate for your OpenShift Container Platform release version. Embedding an `oc` client version that is not compatible with the OpenShift Container Platform version can cause unexpected behavior.

:::

The OpenShift Container Platform Jenkins image also defines the following sample `java-builder` pod template to illustrate how you can use the agent image with the Jenkins Kubernetes plugin.

The `java-builder` pod template employs two containers:

- A `jnlp` container that uses the OpenShift Container Platform Base agent image and handles the JNLP contract for starting and stopping Jenkins agents.
- A `java` container that uses the `java` OpenShift Container Platform Sample ImageStream, which contains the various Java binaries, including the Maven binary `mvn`, for building code.

## Jenkins agent images {#images-other-jenkins-agent-images_images-other-jenkins-agent}

The OpenShift Container Platform Jenkins agent images are available on [Quay.io](https://quay.io) or [registry.redhat.io](https://registry.redhat.io).

Jenkins images are available through the Red Hat Registry:

```terminal
$ docker pull registry.redhat.io/ocp-tools-4/jenkins-rhel8:<image_tag>
```

```terminal
$ docker pull registry.redhat.io/ocp-tools-4/jenkins-agent-base-rhel8:<image_tag>
```

To use these images, you can either access them directly from [Quay.io](https://quay.io) or [registry.redhat.io](https://registry.redhat.io) or push them into your OpenShift Container Platform container image registry.

## Jenkins agent environment variables {#images-other-jenkins-agent-env-var_images-other-jenkins-agent}

Each Jenkins agent container can be configured with the following environment variables.

| Variable | Definition | Example values and settings |
| --- | --- | --- |
| `JAVA_MAX_HEAP_PARAM`, `CONTAINER_HEAP_PERCENT`, `JENKINS_MAX_HEAP_UPPER_BOUND_MB` | These values control the maximum heap size of the Jenkins JVM. If `JAVA_MAX_HEAP_PARAM` is set, its value takes precedence. Otherwise, the maximum heap size is dynamically calculated as `CONTAINER_HEAP_PERCENT` of the container memory limit, optionally capped at `JENKINS_MAX_HEAP_UPPER_BOUND_MB` MiB.<br>By default, the maximum heap size of the Jenkins JVM is set to 50% of the container memory limit with no cap. | `JAVA_MAX_HEAP_PARAM` example setting: `-Xmx512m`<br>`CONTAINER_HEAP_PERCENT` default: `0.5`, or 50%<br>`JENKINS_MAX_HEAP_UPPER_BOUND_MB` example setting: `512 MiB` |
| `JAVA_INITIAL_HEAP_PARAM`, `CONTAINER_INITIAL_PERCENT` | These values control the initial heap size of the Jenkins JVM. If `JAVA_INITIAL_HEAP_PARAM` is set, its value takes precedence. Otherwise, the initial heap size is dynamically calculated as `CONTAINER_INITIAL_PERCENT` of the dynamically calculated maximum heap size.<br>By default, the JVM sets the initial heap size. | `JAVA_INITIAL_HEAP_PARAM` example setting: `-Xms32m`<br>`CONTAINER_INITIAL_PERCENT` example setting: `0.1`, or 10% |
| `CONTAINER_CORE_LIMIT` | If set, specifies an integer number of cores used for sizing numbers of internal JVM threads. | Example setting: `2` |
| `JAVA_TOOL_OPTIONS` | Specifies options to apply to all JVMs running in this container. It is not recommended to override this value. | Default: `-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -Dsun.zip.disableMemoryMapping=true` |
| `JAVA_GC_OPTS` | Specifies Jenkins JVM garbage collection parameters. It is not recommended to override this value. | Default: `-XX:+UseParallelGC -XX:MinHeapFreeRatio=5 -XX:MaxHeapFreeRatio=10 -XX:GCTimeRatio=4 -XX:AdaptiveSizePolicyWeight=90` |
| `JENKINS_JAVA_OVERRIDES` | Specifies additional options for the Jenkins JVM. These options are appended to all other options, including the Java options above, and can be used to override any of them, if necessary. Separate each additional option with a space and if any option contains space characters, escape them with a backslash. | Example settings: `-Dfoo -Dbar`; `-Dfoo=first\ value -Dbar=second\ value` |
| `USE_JAVA_VERSION` | Specifies the version of Java version to use to run the agent in its container. The container base image has two versions of java installed: `java-11` and `java-1.8.0`. If you extend the container base image, you can specify any alternative version of java using its associated suffix. | The default value is `java-11`.<br>Example setting: `java-1.8.0` |

## Jenkins agent memory requirements {#images-other-jenkins-agent-memory_images-other-jenkins-agent}

A JVM is used in all Jenkins agents to host the Jenkins JNLP agent as well as to run any Java applications such as `javac`, Maven, or Gradle.

By default, the Jenkins JNLP agent JVM uses 50% of the container memory limit for its heap. This value can be modified by the `CONTAINER_HEAP_PERCENT` environment variable. It can also be capped at an upper limit or overridden entirely.

By default, any other processes run in the Jenkins agent container, such as shell scripts or `oc` commands run from pipelines, cannot use more than the remaining 50% memory limit without provoking an OOM kill.

By default, each further JVM process that runs in a Jenkins agent container uses up to 25% of the container memory limit for its heap. It might be necessary to tune this limit for many build workloads.

## Jenkins agent Gradle builds {#images-other-jenkins-agent-gradle_images-other-jenkins-agent}

Hosting Gradle builds in the Jenkins agent on OpenShift Container Platform presents additional complications because in addition to the Jenkins JNLP agent and Gradle JVMs, Gradle spawns a third JVM to run tests if they are specified.

The following settings are suggested as a starting point for running Gradle builds in a memory constrained Jenkins agent on OpenShift Container Platform. You can modify these settings as required.

- Ensure the long-lived Gradle daemon is disabled by adding `org.gradle.daemon=false` to the `gradle.properties` file.
- Disable parallel build execution by ensuring `org.gradle.parallel=true` is not set in the `gradle.properties` file and that `--parallel` is not set as a command-line argument.
- To prevent Java compilations running out-of-process, set `java { options.fork = false }` in the `build.gradle` file.
- Disable multiple additional test processes by ensuring `test { maxParallelForks = 1 }` is set in the `build.gradle` file.
- Override the Gradle JVM memory parameters by the `GRADLE_OPTS`, `JAVA_OPTS` or `JAVA_TOOL_OPTIONS` environment variables.
- Set the maximum heap size and JVM arguments for any Gradle test JVM by defining the `maxHeapSize` and `jvmArgs` settings in `build.gradle`, or through the `-Dorg.gradle.jvmargs` command-line argument.

## Jenkins agent pod retention {#images-other-jenkins-agent-pod-retention_images-other-jenkins-agent}

Jenkins agent pods, are deleted by default after the build completes or is stopped. This behavior can be changed by the Kubernetes plugin pod retention setting. Pod retention can be set for all Jenkins builds, with overrides for each pod template. The following behaviors are supported:

- `Always` keeps the build pod regardless of build result.
- `Default` uses the plugin value, which is the pod template only.
- `Never` always deletes the pod.
- `On Failure` keeps the pod if it fails during the build.

You can override pod retention in the pipeline Jenkinsfile:

```groovy
podTemplate(label: "mypod",
  cloud: "openshift",
  inheritFrom: "maven",
  podRetention: onFailure(), (1)
  containers: [
    ...
  ]) {
  node("mypod") {
    ...
  }
}
```

1. Allowed values for `podRetention` are `never()`, `onFailure()`, `always()`, and `default()`.

:::warning

Pods that are kept might continue to run and count against resource quotas.

:::
