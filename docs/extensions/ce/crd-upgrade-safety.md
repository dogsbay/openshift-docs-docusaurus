---
title: Custom resource definition (CRD) upgrade safety
sidebar_position: 6
---

# Custom resource definition (CRD) upgrade safety {#crd-upgrade-safety}

<a id="crd-upgrade-safety"></a>

When you update a custom resource definition (CRD) provided by a cluster extension, Operator Lifecycle Manager (OLM) v1 runs a CRD upgrade safety preflight check to ensure compatibility with earlier versions.

The CRD update must pass the validation checks before the change is allowed to progress on a cluster.

**Additional resources**

- [Updating a cluster extension](/docs/extensions/ce/managing-ce#olmv1-updating-an-operator_managing-ce)

## Prohibited CRD upgrade changes {#prohibited-crd-upgrades_crd-upgrade-safety}

To avoid making a modification that does not validate, review the custom resource definiton (CRD) changes that are blocked by the upgrade safety preflight check.

The CRD upgrade safety preflight check blocks an upgrade if it detects any of the following changes to an existing CRD:

- Adding a new required field to an existing version
- Removing an existing field from an existing version
- Changing an existing field type in an existing version
- Adding a default value to a field that did not previously have one
- Changing the default value of an existing field
- Removing the default value of an existing field
- Adding enum restrictions to a field that did not previously have them
- Removing existing enum values from an existing field
- Increasing the minimum value of an existing field in an existing version
- Decreasing the maximum value of an existing field in an existing version
- Adding minimum or maximum constraints to a field that did not previously have them

:::note

Rules for minimum and maximum values apply to the `minimum`, `minLength`, `minProperties`, `minItems`, `maximum`, `maxLength`, `maxProperties`, and `maxItems` constraints.

:::

The preflight check also blocks an upgrade for the following structural changes, which are handled by the Kubernetes API server:

- Changing the CRD scope between `Cluster` and `Namespace`
- Removing an existing stored version of the CRD

If the CRD upgrade safety preflight check detects any prohibited change, it logs an error for each violation.

:::tip

If a CRD change is neither explicitly allowed nor categorized as a known prohibited change, the preflight check blocks the upgrade and logs an "unknown change" error.

:::

## Allowed CRD upgrade changes {#allowed-crd-changes_crd-upgrade-safety}

Reference which custom resource definition (CRD) changes are compatible with earlier versions to avoid unexpected halts during the upgrade safety preflight check.

The following CRD changes are compatible with earlier versions and pass the upgrade safety preflight check:

- Adding new values to an existing enum field
- Changing an existing required field to optional in an existing version
- Decreasing the minimum value of an existing field in an existing version
- Increasing the maximum value of an existing field in an existing version
- Adding a new version of the CRD without modifying existing versions

## Disabling the CRD upgrade safety preflight check {#disabling-crd-preflight_crd-upgrade-safety}

You can disable the custom resource definition (CRD) upgrade safety preflight check. In the `ClusterExtension` object that provides the CRD, set the `install.preflight.crdUpgradeSafety.enforcement` field with the value of `None`.

:::warning

Disabling the CRD upgrade safety preflight check could break backwards compatibility with stored versions of the CRD and cause other unintended consequences on the cluster.

:::

You cannot disable individual field validators. If you disable the CRD upgrade safety preflight check, you disable all field validators.

:::note

If you disable the CRD upgrade safety preflight check in Operator Lifecycle Manager (OLM) v1, the Kubernetes API server still prevents the following operations:

- Changing scope from `Cluster` to `Namespace` or from `Namespace` to `Cluster`
- Removing an existing stored version of the CRD

:::

**Prerequisites**

- You have a cluster extension installed.

**Procedure**

1. Edit the `ClusterExtension` object of the CRD:
   ```terminal
   $ oc edit clusterextension <clusterextension_name>
   ```
2. Set the `install.preflight.crdUpgradeSafety.enforcement` field to `None`:
   ```yaml title="Example ClusterExtension object"
   apiVersion: olm.operatorframework.io/v1
   kind: ClusterExtension
   metadata:
     name: clusterextension-sample
   spec:
     namespace: default
     serviceAccount:
       name: sa-example
     source:
       sourceType: "Catalog"
       catalog:
         packageName: argocd-operator
         version: 0.6.0
     install:
       preflight:
         crdUpgradeSafety:
           enforcement: None
   ```

## Examples of unsafe CRD changes {#examples-unsafe_crd-upgrade-safety}

Review the example unsafe custom resource definition (CRD) changes to recognize modifications that trigger the CRD upgrade safety preflight check.

The following examples use this baseline `CustomResourceDefinition` object:

```yaml title="Example CRD object"
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.13.0
  name: example.test.example.com
spec:
  group: test.example.com
  names:
    kind: Sample
    listKind: SampleList
    plural: samples
    singular: sample
  scope: Namespaced
  versions:
  - name: v1alpha1
    schema:
      openAPIV3Schema:
        properties:
          apiVersion:
            type: string
          kind:
            type: string
          metadata:
            type: object
          spec:
            type: object
          status:
            type: object
          pollInterval:
            type: string
        type: object
    served: true
    storage: true
    subresources:
      status: {}
```

### Scope change {#scope-change_crd-upgrade-safety}

The following example changes the `spec.scope` field from `Namespaced` to `Cluster`:

```yaml title="Example scope change in a CRD"
spec:
  group: test.example.com
  names:
    kind: Sample
    listKind: SampleList
    plural: samples
    singular: sample
  scope: Cluster
  versions:
  - name: v1alpha1
```

```text title="Example error output"
validating upgrade for CRD "test.example.com" failed: CustomResourceDefinition test.example.com failed upgrade safety validation. "NoScopeChange" validation failed: scope changed from "Namespaced" to "Cluster"
```

### Removal of a stored version {#stored-version-removal_crd-upgrade-safety}

The following example removes the existing stored version, `v1alpha1`:

```yaml title="Example removal of a stored version in a CRD"
versions:
- name: v1alpha2
  schema:
    openAPIV3Schema:
      properties:
        apiVersion:
          type: string
        kind:
          type: string
        metadata:
          type: object
        spec:
          type: object
        status:
          type: object
        pollInterval:
          type: string
      type: object
```

```text title="Example error output"
validating upgrade for CRD "test.example.com" failed: CustomResourceDefinition test.example.com failed upgrade safety validation. "NoStoredVersionRemoved" validation failed: stored version "v1alpha1" removed
```

### Removal of an existing field {#removal-existing-field_crd-upgrade-safety}

The following example removes the `pollInterval` property field from the `v1alpha1` schema:

```yaml title="Example removal of an existing field in a CRD"
versions:
- name: v1alpha1
  schema:
    openAPIV3Schema:
      properties:
        apiVersion:
          type: string
        kind:
          type: string
        metadata:
          type: object
        spec:
          type: object
        status:
          type: object
      type: object
```

```text title="Example error output"
validating upgrade for CRD "test.example.com" failed: CustomResourceDefinition test.example.com failed upgrade safety validation. "NoExistingFieldRemoved" validation failed: crd/test.example.com version/v1alpha1 field/^.spec.pollInterval may not be removed
```

### Addition of a required field {#addition-required-field_crd-upgrade-safety}

The following example changes the `pollInterval` property to a required field:

```yaml title="Example addition of a required field in a CRD"
versions:
- name: v1alpha2
  schema:
    openAPIV3Schema:
      properties:
        apiVersion:
          type: string
        kind:
          type: string
        metadata:
          type: object
        spec:
          type: object
        status:
          type: object
        pollInterval:
          type: string
      type: object
      required:
      - pollInterval
```

```text title="Example error output"
validating upgrade for CRD "test.example.com" failed: CustomResourceDefinition test.example.com failed upgrade safety validation. "ChangeValidator" validation failed: version "v1alpha1", field "^": new required fields added: [pollInterval]
```
