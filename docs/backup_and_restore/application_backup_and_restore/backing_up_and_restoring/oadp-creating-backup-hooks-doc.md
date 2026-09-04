---
title: Creating backup hooks
sidebar_position: 7
---

# Creating backup hooks {#oadp-creating-backup-hooks-doc}

<a id="oadp-creating-backup-hooks-doc"></a>

Create backup hooks to run commands in a container in a pod by editing the `Backup` custom resource (CR). This helps you to run pre-backup and post-backup actions such as quiescing a database or flushing data to disk.

The commands can be configured to performed before any custom action processing (*Pre* hooks), or after all custom actions have been completed and any additional items specified by the custom action have been backed up (*Post* hooks).

**Procedure**

- Add a hook to the `spec.hooks` block of the `Backup` CR, as in the following example:
  ```yaml
  apiVersion: velero.io/v1
  kind: Backup
  metadata:
    name: <backup>
    namespace: openshift-adp
  spec:
    hooks:
      resources:
        - name: <hook_name>
          includedNamespaces:
          - <namespace>
          excludedNamespaces:
          - <namespace>
          includedResources: []
          - pods
          excludedResources: []
          labelSelector:
            matchLabels:
              app: velero
              component: server
          pre:
            - exec:
                container: <container>
                command:
                - /bin/uname
                - -a
                onError: Fail
                timeout: 30s
          post:
  ...
  ```

  where:

  <dl>
  <dt><code>&lt;namespace&gt;</code></dt>
  <dd>Optional: Specifies the namespaces to which the hook applies. If this value is not specified, the hook applies to all namespaces.</dd>
  <dt><code>excludedNamespaces</code></dt>
  <dd>Optional: Specifies the namespaces to which the hook does not apply.</dd>
  <dt><code>pods</code></dt>
  <dd>Currently, pods are the only supported resource that hooks can apply to.</dd>
  <dt><code>excludedResources</code></dt>
  <dd>Optional: Specifies the resources to which the hook does not apply.</dd>
  <dt><code>labelSelector</code></dt>
  <dd>Optional: This hook only applies to objects matching the label. If this value is not specified, the hook applies to all objects.</dd>
  <dt><code>pre</code></dt>
  <dd>Specifies an array of hooks to run before the backup.</dd>
  <dt><code>&lt;container&gt;</code></dt>
  <dd>Optional: Specifies the container in which the command runs. If the container is not specified, the command runs in the first container in the pod.</dd>
  <dt><code>/bin/uname</code></dt>
  <dd>Specifies the entry point for the <code>init</code> container being added.</dd>
  <dt><code>onError: Fail</code></dt>
  <dd>Specifies the error handling behavior. Allowed values are <code>Fail</code> and <code>Continue</code>. The default is <code>Fail</code>.</dd>
  <dt><code>timeout: 30s</code></dt>
  <dd>Optional: Specifies how long to wait for the commands to run. The default is <code>30s</code>.</dd>
  <dt><code>post</code></dt>
  <dd>Specifies an array of hooks to run after the backup, with the same parameters as the pre-backup hooks.</dd>
  </dl>
