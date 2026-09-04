---
title: Mounting OCI images and artifacts into a pod
sidebar_position: 11
---

# Mounting OCI images and artifacts into a pod {#nodes-pods-image-volume}

<a id="nodes-pods-image-volume"></a>

You can mount an Open Container Initiative (OCI)-compliant container image or artifact directly into a pod, making the OCI object accessible to the containers without the need to include them in the base image, which allows you to host the data in OCI-compliant registries.

## Understanding image volumes {#nodes-pods-image-volume-about_nodes-pods-node-selectors}

You can you use an *image volume* to mount an Open Container Initiative (OCI)-compliant container image or artifact directly into a pod as a native volume source, making the OCI object accessible to the containers without the need to include them in the base image.

OCI objects enable users to store and distribute arbitrary files and metadata through OCI-compliant container registries.

By using an image volume in a pod, you can take advantage of the OCI image and distribution specification standards to accomplish several tasks including the following use cases:

- You can share configuration files among multiple containers in a pod without needing to include the file in the base image, which minimizes security risks and image size.
- In an artificial intelligence environment, you can use image volumes to mount large language model weights or machine learning model weights in a pod alongside a model-server. You can efficiently serve model weights this way without including them in the model-server container image. Therefore, you can separate the model specifications and content from the executables that process them.
- You can use a public image for a malware scanner and mount it in a volume of private malware signatures, so that you can load those signatures without incorporating the image into a base image, which might not be allowed by the copyright on the public image.
- You can package and distribute binary artifacts and mount them directly into your pods, allowing you to streamline your CI/CD pipeline. This allows you to maintain a small set of base images by attaching the CI/CD artifacts to the image volumes instead.

To mount an image volume, include a path to the image or artifact in your pod spec with an optional pull policy as described in *Adding an image volume to a pod*.

## Adding an image volume to a pod {#nodes-pods-image-volume-adding_nodes-pods-node-selectors}

To mount an Open Container Initiative (OCI)-compliant container image or artifact, use the `volume` parameter in your pod spec to include a path to the image or artifact, along with an optional pull policy.

You can create the pod directly or use a controlling object, such as a deployment or replica set.

**Procedure**

1. Create a YAML file similar to the following.
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: image-volume
   spec:
     containers:
     - name: image-volume-container
       command: ["sleep", "infinity"]
       image: debian
       volumeMounts:
       - name: image
         mountPath: /image
     - name: artifact-volume-container
       image: busybox:latest
       command: ["/bin/sh"]
       args:
       - -c
       - trap 'exit 0' TERM INT; sleep infinity & wait
       volumeMounts:
       - name: artifact
         mountPath: /artifact
     volumes:
     - name: image
       image:
         reference: quay.io/crio/busybox:1
         pullPolicy: Always
     - name: artifact
       image:
         reference: quay.io/crio/artifact:singlefile
         pullPolicy: IfNotPresent
   ```

   where:

   <dl>
   <dt><code>spec.containers</code></dt>
   <dd>Specifies the configuration for one or more containers.</dd>
   <dt><code>spec.containers.volumeMounts</code></dt>
   <dd>Specifies the name of the volume to mount and where to mount that volume. The first example indicates the container with the name <code>image-volume-container</code> should mount the <code>image</code> volume under the path <code>/image</code>.</dd>
   <dt><code>spec.volumes</code></dt>
   <dd>Specifies the storage volumes that are available for the containers to use.</dd>
   <dt><code>spec.volumes.name</code></dt>
   <dd>Specifies a name for the volume.</dd>
   <dt><code>spec.volumes.image</code></dt>
   <dd>Specifies an OCI container image or artifact that is available on the host machine.</dd>
   <dt><code>spec.volumes.image.reference</code></dt>
   <dd>Specifies the path to the OCI object.</dd>
   <dt><code>spec.volumes.image.pullPolicy</code></dt>
   <dd>Specifies a pull policy, one of the following options: *   If <code>Always</code>, the kubelet always attempts to pull the OCI object. If the pull fails, the kubelet sets the pod to <code>Failed</code>. *   If <code>Never</code>, the kubelet never pulls the image and only uses an OCI object. The pod becomes <code>Failed</code> if any layers of the image are not present locally, or if the manifest for that image is not already cached. *   If <code>IfNotPresent</code>, the kubelet pulls the OCI object if it is not present. The pod becomes <code>Failed</code> if the OCI object is not present and the pull fails. This is the default.</dd>
   </dl>
2. Create the pod by running the following command:
   ```terminal
   $ oc create -f <file_name>.yaml
   ```

**Verification**

- Examine the pod to view detailed information about the image pull and mount by using a command similar to the following:
  ```terminal
  $ oc describe pod <pod_name>
  ```

  ```yaml title="Example output"
  Name:             image-volume
  Namespace:        default
  # ...
  Containers:
    image-volume-container:
  # ...
      Mounts:
        /image from image (rw)
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-svtzn (ro)
    artifact-volume-container:
  # ...
      Mounts:
        /artifact from artifact (rw)
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-svtzn (ro)
  # ...
  Volumes:
    image:
      Type:        Image (a container image or OCI artifact)
      Reference:   quay.io/crio/busybox:1
      PullPolicy:  Always
    artifact:
      Type:        Image (a container image or OCI artifact)
      Reference:   quay.io/crio/artifact:singlefile
      PullPolicy:  IfNotPresent
  # ...
  Events:
    Type    Reason          Age                From               Message
    ----    ------          ----               ----               -------
  # ...
    Normal  Pulling         10s (x3 over 15s)  kubelet            Pulling image "quay.io/crio/busybox:1"
    Normal  Pulled          10s                kubelet            Successfully pulled image "quay.io/crio/busybox:1" in 555ms (555ms including waiting). Image size: 1468102 bytes.
  # ...
    Normal  Pulling         15s                kubelet            Pulling image "quay.io/crio/artifact:singlefile"
    Normal  Pulled          13s                kubelet            Successfully pulled image "quay.io/crio/artifact:singlefile" in 1.493s (1.493s including waiting). Image size: 14 bytes.
  # ...
  ```

  - The `Containers` stanza shows that the two containers were created with the configured volume mounts.
  - The `Volumes` stanza shows that the two volumes were created.
  - The `Events` stanza shows that the images in the volumes were pulled.
