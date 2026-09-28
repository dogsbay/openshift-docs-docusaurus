---
title: Tailoring the Compliance Operator
sidebar_position: 4
---

# Tailoring the Compliance Operator {#compliance-operator-tailor}

<a id="compliance-operator-tailor"></a>

Although the Compliance Operator includes ready-to-use profiles, you must modify the profiles to fit your organization’s requirements. The process of modifying a profile is called *tailoring*.

The Compliance Operator provides the `TailoredProfile` object to help tailor profiles.

## Create a new tailored profile {#compliance-new-tailored-profiles_compliance-tailor}

You can write a tailored profile from scratch by using the `TailoredProfile` object. Set an appropriate `title` and `description` and leave the `extends` field empty.

Indicate to the Compliance Operator what type of scan this custom profile will generate:

- Node scan: Scans the operating system.
- Platform scan: Scans the OpenShift Container Platform configuration.

**Procedure**

- Set the following annotation on the `TailoredProfile` object:
  ```yaml title="Example new-profile.yaml"
  apiVersion: compliance.openshift.io/v1alpha1
  kind: TailoredProfile
  metadata:
    name: new-profile
    annotations:
      compliance.openshift.io/product-type: Node
  spec:
    extends: ocp4-cis-node
    description: My custom profile
    title: Custom profile
    enableRules:
      - name: ocp4-etcd-unique-ca
        rationale: We really need to enable this
    disableRules:
      - name: ocp4-file-groupowner-cni-conf
        rationale: This does not apply to the cluster
  ```

  where:

  <dl>
  <dt><code>metadata.annotations.compliance.openshift.io/product-type</code></dt>
  <dd>Sets <code>Node</code> or <code>Platform</code> accordingly.</dd>
  <dt><code>spec.extends</code></dt>
  <dd>Optional field to specify the base profile.</dd>
  <dt><code>spec.description</code></dt>
  <dd>Specifies the function of the new <code>TailoredProfile</code> object.</dd>
  <dt><code>spec.title</code></dt>
  <dd>Specifies a title for the <code>TailoredProfile</code> object.</dd>
  </dl>

:::note

Adding the `-node` suffix to the `name` field of the `TailoredProfile` object is similar to adding the `Node` product type annotation and generates an operating system scan.

:::

## Use tailored profiles to extend existing ProfileBundles {#compliance-tailored-profiles_compliance-tailor}

Although the `TailoredProfile` CR enables the most common tailoring operations, you can use the XCCDF (Extensible Configuration Checklist Description Format) standard for even more flexibility in tailoring OpenSCAP profiles.

In addition, if your organization has been using OpenScap previously, you might have an existing XCCDF tailoring file and can reuse it.

The `ComplianceSuite` object has an optional `TailoringConfigMap` attribute that you can point to a custom tailoring file. The value of the `TailoringConfigMap` attribute is a name of a config map, which must contain a key called `tailoring.xml` and the value of this key is the tailoring contents.

**Procedure**

1. Browse the available rules for the Red Hat Enterprise Linux CoreOS (RHCOS) `ProfileBundle`:
   ```terminal
   $ oc get rules.compliance -n openshift-compliance -l compliance.openshift.io/profile-bundle=rhcos4
   ```
2. Browse the available variables in the same `ProfileBundle`:
   ```terminal
   $ oc get variables.compliance -n openshift-compliance -l compliance.openshift.io/profile-bundle=rhcos4
   ```
3. Create a tailored profile named `nist-moderate-modified`:
   1. Choose which rules you want to add to the `nist-moderate-modified` tailored profile. This example extends the `rhcos4-moderate` profile by disabling two rules and changing one value. Use the `rationale` value to describe why these changes were made:
      ```yaml title="Example new-profile-node.yaml"
      apiVersion: compliance.openshift.io/v1alpha1
      kind: TailoredProfile
      metadata:
        name: nist-moderate-modified
      spec:
        extends: rhcos4-moderate
        description: NIST moderate profile
        title: My modified NIST moderate profile
        disableRules:
        - name: rhcos4-file-permissions-var-log-messages
          rationale: The file contains logs of error messages in the system
        - name: rhcos4-account-disable-post-pw-expiration
          rationale: No need to check this as it comes from the IdP
        setValues:
        - name: rhcos4-var-selinux-state
          rationale: Organizational requirements
          value: permissive
      ```

      **Attributes for spec variables**

<table>
<thead>
<tr>
  <th>Attribute</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>extends</code></td>
  <td>Name of the <code>Profile</code> object upon which this <code>TailoredProfile</code> is built.</td>
</tr>
<tr>
  <td><code>title</code></td>
  <td>Human-readable title of the <code>TailoredProfile</code>.</td>
</tr>
<tr>
  <td><code>disableRules</code></td>
  <td>A list of name and rationale pairs. Each name refers to a name of a rule object that is to be disabled. The rationale value is human-readable text describing why the rule is disabled.</td>
</tr>
<tr>
  <td><code>manualRules</code></td>
  <td>A list of name and rationale pairs. When a manual rule is added, the check result status will always be <code>manual</code> and remediation will not be generated. This attribute is automatic and by default has no values when set as a manual rule.</td>
</tr>
<tr>
  <td><code>enableRules</code></td>
  <td>A list of name and rationale pairs. Each name refers to a name of a rule object that is to be enabled. The rationale value is human-readable text describing why the rule is enabled.</td>
</tr>
<tr>
  <td><code>description</code></td>
  <td>Human-readable text describing the <code>TailoredProfile</code>.</td>
</tr>
<tr>
  <td><code>setValues</code></td>
  <td>A list of name, rationale, and value groupings. Each name refers to a name of the value set. The rationale is human-readable text describing the set. The value is the actual setting.</td>
</tr>
</tbody>
</table>

````
1.  Add the `tailoredProfile.spec.manualRules` attribute:
    ```yaml title="Example tailoredProfile.spec.manualRules.yaml"
    apiVersion: compliance.openshift.io/v1alpha1
    kind: TailoredProfile
    metadata:
      name: ocp4-manual-scc-check
    spec:
      extends: ocp4-cis
      description: This profile extends ocp4-cis by forcing the SCC check to always return MANUAL
      title: OCP4 CIS profile with manual SCC check
      manualRules:
        - name: ocp4-scc-limit-container-allowed-capabilities
          rationale: We use third party software that installs its own SCC with extra privileges
    ```
1.  Create the `TailoredProfile` object:
    ```terminal
    $ oc create -n openshift-compliance -f new-profile-node.yaml
    ```
    *   The `TailoredProfile` object is created in the default `openshift-compliance` namespace.
        ```terminal title="Example output"
        tailoredprofile.compliance.openshift.io/nist-moderate-modified created
        ```
````

1. Define the `ScanSettingBinding` object to bind the new `nist-moderate-modified` tailored profile to the default `ScanSetting` object.
   ```yaml title="Example new-scansettingbinding.yaml"
   apiVersion: compliance.openshift.io/v1alpha1
   kind: ScanSettingBinding
   metadata:
     name: nist-moderate-modified
   profiles:
     - apiGroup: compliance.openshift.io/v1alpha1
       kind: Profile
       name: ocp4-moderate
     - apiGroup: compliance.openshift.io/v1alpha1
       kind: TailoredProfile
       name: nist-moderate-modified
   settingsRef:
     apiGroup: compliance.openshift.io/v1alpha1
     kind: ScanSetting
     name: default
   ```
2. Create the `ScanSettingBinding` object:
   ```terminal
   $ oc create -n openshift-compliance -f new-scansettingbinding.yaml
   ```

   ```terminal title="Example output"
   scansettingbinding.compliance.openshift.io/nist-moderate-modified created
   ```
