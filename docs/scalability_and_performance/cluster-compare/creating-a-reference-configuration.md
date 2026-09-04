---
title: Creating a reference configuration
sidebar_position: 4
---

# Creating a reference configuration

<a id="creating-a-reference-configuration"></a>

Configure a reference configuration to validate configuration resources from a cluster.

## Structure of the metadata.yaml file {#cluster-compare-metadata-structure_creating-a-reference-configuration}

The `metadata.yaml` file provides a central configuration point to define and configure the templates in a reference configuration. The file features a hierarchy of `parts` and `components`. `parts` are groups of `components` and `components` are groups of templates. Under each component, you can configure template dependencies, validation rules, and add descriptive metadata.

```yaml title="Example metadata.yaml file"
apiVersion: v2
parts:
  - name: <part_name>
    components:
      - name: <component_name>
        <component_configuration>
  - name: <part_name>
      - name: <component_name>
        <component_configuration>
```

where:

`<part_name>`
: Specify a `part` name. Every `part` typically describes a workload or a set of workloads.

`<component_name>`
: Specify a `component` name.

`<component_configuration>`
: Specify the configuration for a template. For example, define template relationships or configure what fields to use in a comparison.

## Configuring template relationships {#cluster-compare-template-groupings_creating-a-reference-configuration}

By defining relationships between templates in your reference configuration, you can support use-cases with complex dependencies. For example, you can configure a component to require specific templates, require one template from a group, or allow any template from a group, and so on.

**Procedure**

- Create a `metadata.yaml` file to match your use case. Use the following structure as an example:
  ```yaml title="Example metadata.yaml file"
  apiVersion: v2
  parts:
    - name: Part1
      components:
        - name: Component1
          allOf:
            - path: RequiredTemplate1.yaml
            - path: RequiredTemplate2.yaml
        - name: Component2
          allOrNoneOf:
            - path: OptionalBlockTemplate1.yaml
            - path: OptionalBlockTemplate2.yaml
        - name: Component3
          anyOf:
            - path: OptionalTemplate1.yaml
            - path: OptionalTemplate2.yaml
        - name: Component4
          noneOf:
            - path: BannedTemplate1.yaml
            - path: BannedTemplate2.yaml
        - name: Component5
          oneOf:
            - path: RequiredExclusiveTemplate1.yaml
            - path: RequiredExclusiveTemplate2.yaml
        - name: Component6
          anyOneOf:
            - path: OptionalExclusiveTemplate1.yaml
            - path: OptionalExclusiveTemplate2.yaml
  #...
  ```

  where:

  <dl>
  <dt><code>allOf</code></dt>
  <dd>Specifies required templates.</dd>
  <dt><code>allOrNoneOf</code></dt>
  <dd>Specifies a group of templates that are either all required or all optional. If one corresponding custom resource (CR) is present in the cluster, then all corresponding CRs must be present in the cluster.</dd>
  <dt><code>anyOf</code></dt>
  <dd>Specifies optional templates.</dd>
  <dt><code>noneOf</code></dt>
  <dd>Specifies templates to exclude. If a corresponding CR is present in the cluster, the plugin returns a validation error.</dd>
  <dt><code>oneOf</code></dt>
  <dd>Specifies templates where only one can be present. If none, or more than one of the corresponding CRs are present in the cluster, the plugin returns a validation error.</dd>
  <dt><code>anyOneOf</code></dt>
  <dd>Specifies templates where only one can be present in the cluster. If more than one of the corresponding CRs are present in the cluster, the plugin returns a validation error.</dd>
  </dl>

## Configuring expected variation in a template {#cluster-compare-templating_creating-a-reference-configuration}

You can handle variable content within a template by using Golang templating syntax. Using this syntax, you can configure validation logic that handles optional, required, and conditional content within the template.

:::note

- The `cluster-compare` plugin requires all templates to render as valid YAML. To avoid parsing errors for missing fields, use conditional templating syntax such as `{{- if .spec.<optional_field> }}` when implementing templating syntax. This conditional logic ensures templates process missing fields gracefully and maintains valid YAML formatting.
- You can use the Golang templating syntax with custom and built-in functions for complex use cases. All Golang built-in functions are supported including the functions in the Sprig library.

:::

**Procedure**

- Create a `metadata.yaml` file to match your use case. Use the following structure as an example:
  ```yaml
  apiVersion: v2
  kind: Service
  metadata:
    name: frontend
    namespace: {{ .metadata.namespace }}
    labels:
      app: guestbook
      tier: frontend
  spec:
    {{- if and .spec.type (eq (.spec.type) "NodePort" "LoadBalancer") }}
    type: {{.spec.type }}
    {{- else }}
    type: should be NodePort or LoadBalancer
    {{- end }}
    ports:
    - port: 80
    selector:
      app: guestbook
      {{- if .spec.selector.tier }}
      tier: frontend
      {{- end }}
  ```

  where:

  <dl>
  <dt><code>name: frontend</code></dt>
  <dd>Configures a required field that must match the specified value.</dd>
  <dt><code>namespace: \{{ .metadata.namespace }}</code></dt>
  <dd>Configures a required field that can have any value.</dd>
  <dt><code>type: \{{.spec.type }}</code></dt>
  <dd>Configures validation for the <code>.spec.type</code> field.</dd>
  <dt><code>\{{- if .spec.selector.tier }}</code></dt>
  <dd>Configures an optional field.</dd>
  </dl>

### Reference template functions {#cluster-compare-templating-reference_creating-a-reference-configuration}

The `cluster-compare` plugin supports all `sprig` library functions, except for the `env` and `expandenv` functions. For the full list of `sprig` library functions, see "Sprig Function Documentation".

The following table describes the additional template functions for the `cluster-compare` plugin:

**Additional cluster-compare template functions**

<table>
<thead>
<tr>
  <th>Function</th>
  <th>Description</th>
  <th>Example</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>fromJson</code></td>
  <td>Parses the incoming string as a structured JSON object.</td>
  <td><code>value: {obj := spec.jsontext | fromJson}{obj.field}</code></td>
</tr>
<tr>
  <td><code>fromJsonArray</code></td>
  <td>Parses the incoming string as a structured JSON array.</td>
  <td><code>value: {obj := spec.jsontext | fromJson}{index $obj 0}</code></td>
</tr>
<tr>
  <td><code>fromYaml</code></td>
  <td>Parses the incoming string as a structured YAML object.</td>
  <td><code>value: {obj := spec.yamltext | fromYaml}{obj.field}</code></td>
</tr>
<tr>
  <td><code>fromYamlArray</code></td>
  <td>Parses the incoming string as a structured YAML array.</td>
  <td><code>value: {obj := spec.yamltext | fromYaml}{index $obj 0 }</code></td>
</tr>
<tr>
  <td><code>toJson</code></td>
  <td>Renders incoming data as JSON while preserving object types.</td>
  <td><code>jsonstring: {{ $variable | toJson}</code></td>
</tr>
<tr>
  <td><code>toToml</code></td>
  <td>Renders the incoming string as structured TOML data.</td>
  <td><code>tomlstring: {$variable | toToml}</code></td>
</tr>
<tr>
  <td><code>toYaml</code></td>
  <td>Renders incoming data as YAML while preserving object types.</td>
  <td>For simple scalar values: <code>value: {$data | toYaml}</code><br><br>For lists or dictionaries: <code>value: {$dict | toYaml | nindent 2}</code></td>
</tr>
<tr>
  <td><code>doNotMatch</code></td>
  <td>Prevents a template from matching a cluster resource, even if it would normally match. You can use this function inside a template to conditionally exclude certain resources from correlation. The specified reason is logged when running with the <code>--verbose</code> flag. Templates excluded due to <code>doNotMatch</code> are not considered comparison failures. This function is especially useful when your template does not specify a fixed name or namespace. In these cases, you can use the <code>doNotMatch</code> function to exclude specific resources based on other fields, such as <code>labels</code> or <code>annotations</code>.</td>
  <td><code>{if $condition}{doNotMatch $reason}{end}</code></td>
</tr>
<tr>
  <td><code>lookupCRs</code></td>
  <td>Returns an array of objects that match the specified parameters. For example: <code>lookupCRs $apiVersion $kind $namespace $name</code>. If the <code>$namespace</code> parameter is an empty string (<code>""</code>) or <code>\*</code>, the function matches all namespaces. For cluster-scoped objects, the function matches objects with no namespace. If the <code>$name</code> is an empty string or <code>*</code>, the function matches any named object.</td>
  <td>-</td>
</tr>
<tr>
  <td><code>lookupCR</code></td>
  <td>Returns a single object that matches the parameters. If multiple objects match, the function returns nothing. This function takes the same arguments as the <code>lookupCRs</code> function.</td>
  <td>-</td>
</tr>
</tbody>
</table>

The following example shows how to use the `lookupCRs` function to retrieve and render values from multiple matching resources:

```yaml title="Config map example using lookupCRs"
kind: ConfigMap
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-settings
  namespace: kubernetes-dashboard
data:
  dashboard: {{ index (lookupCR "apps/v1" "Deployment" "kubernetes-dashboard" "kubernetes-dashboard") "metadata" "name" \| toYaml }}
  metrics: {{ (lookupCR "apps/v1" "Deployment" "kubernetes-dashboard" "dashboard-metrics-scraper").metadata.name \| toYaml }}
```

The following example shows how to use the `lookupCR` function to retrieve and use specific values from a single matching resource:

```yaml title="Config map example using lookupCR"
kind: ConfigMap
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-settings
  namespace: kubernetes-dashboard
data:
  {{- $objlist := lookupCRs "apps/v1" "Deployment" "kubernetes-dashboard" "*" }}
  {{- $dashboardName := "unknown" }}
  {{- $metricsName := "unknown" }}
  {{- range $obj := $objlist }}
    {{- $appname := index $obj "metadata" "labels" "k8s-app" }}
    {{- if contains "metrics" $appname }}
      {{- $metricsName = $obj.metadata.name }}
    {{- end }}
    {{- if eq "kubernetes-dashboard" $appname }}
      {{- $dashboardName = $obj.metadata.name }}
    {{- end }}
  {{- end }}
  dashboard: {{ $dashboardName }}
  metrics: {{ $metricsName }}
```

**Additional resources**

- [Sprig Function Documentation](https://masterminds.github.io/sprig/)

## Configuring the metadata.yaml file to exclude template fields {#cluster-compare-exclude-metadata_creating-a-reference-configuration}

You can configure the `metadata.yaml` file to exclude fields from a comparison. Exclude fields that are irrelevant to a comparison, for example annotations or labels that are inconsequential to a cluster configuration.

You can configure exclusions in the `metadata.yaml` file in the following ways:

- Exclude all fields in a custom resource not specified in a template.
- Exclude specific fields that you define using the `pathToKey` field.
  :::note

  `pathToKey` is a dot separated path. Use quotes to escape key values featuring a period.

  :::

### Excluding all fields not specified in a template {#cluster-compare-ignore-all-fields_creating-a-reference-configuration}

During the comparison process, the `cluster-compare` plugin renders a template by merging fields from the corresponding custom resource (CR). If you configure the `ignore-unspecified-fields` to `true`, all fields that are present in the CR, but not in the template, are excluded from the merge. Use this approach when you want to focus the comparison on the fields specified in the template only.

**Procedure**

- Create a `metadata.yaml` file to match your use case. Use the following structure as an example:
  ```yaml
  apiVersion: v2
  parts:
    - name: Part1
      components:
        - name: Namespace
          allOf:
            - path: namespace.yaml
              config:
                ignore-unspecified-fields: true
  #...
  ```

  where:

  <dl>
  <dt><code>ignore-unspecified-fields: true</code></dt>
  <dd>Specify <code>true</code> to exclude from the comparison all fields in a CR that are not explicitly configured in the corresponding <code>namespace.yaml</code> template.</dd>
  </dl>

### Excluding specific fields by setting default exclusion fields {#cluster-compare-ignore-default-fields_creating-a-reference-configuration}

You can exclude fields by defining a default value for `fieldsToOmitRefs` in the `defaultOmitRef` field. This default exclusion applies to all templates, unless overridden by the `config.fieldsToOmitRefs` field for a specific template.

**Procedure**

- Create a `metadata.yaml` file to match your use case. Use the following structure as an example:
  ```yaml title="Example metadata.yaml file"
  apiVersion: v2
  parts:

  #...

  fieldsToOmit:
     defaultOmitRef: default
     items:
        default:
           - pathToKey: a.custom.default."k8s.io"
  ```

  where:

  <dl>
  <dt><code>defaultOmitRef: default</code></dt>
  <dd>Sets the default exclusion for all templates, unless overridden by the <code>config.fieldsToOmitRefs</code> field for a specific template.</dd>
  <dt><code>pathToKey: a.custom.default."k8s.io"</code></dt>
  <dd>The value is excluded for all templates.</dd>
  </dl>

### Excluding specific fields {#cluster-compare-ignore-specified-fields_creating-a-reference-configuration}

You can specify fields to exclude by defining the path to the field, and then referencing the definition in the `config` section for a template.

**Procedure**

- Create a `metadata.yaml` file to match your use case. Use the following structure as an example:
  ```yaml title="Example metadata.yaml file"
  apiVersion: v2
  parts:
    - name: Part1
      components:
        - name: Component1
          - path: deployment.yaml
            config:
              fieldsToOmitRefs:
                - deployments

  #...

  fieldsToOmit:
     items:
        deployments:
           - pathToKey: spec.selector.matchLabels.k8s-app
  ```

  where:

  <dl>
  <dt><code>deployments</code></dt>
  <dd>References the <code>fieldsToOmit.items.deployments</code> item for the <code>deployment.yaml</code> template.</dd>
  <dt><code>pathToKey: spec.selector.matchLabels.k8s-app</code></dt>
  <dd>Excludes the <code>spec.selector.matchLabels.k8s-app</code> field from the comparison.</dd>
  </dl>

  :::note

  Setting `fieldsToOmitRefs` replaces the default value.

  :::

### Excluding specific fields by setting default exclusion groups {#cluster-compare-ignore-default-groups_creating-a-reference-configuration}

You can create default groups of fields to exclude. A group of exclusions can reference another group to avoid duplication when defining exclusions.

**Procedure**

- Create a `metadata.yaml` file to match your use case. Use the following structure as an example:
  ```yaml title="Example metadata.yaml file"
  apiVersion: v2
  parts:

  #...

  fieldsToOmit:
     defaultOmitRef: default
     items:
      common:
        - pathToKey: metadata.annotations."kubernetes.io/metadata.name"
        - pathToKey: metadata.annotations."kubernetes.io/metadata.name"
        - pathToKey: metadata.annotations."kubectl.kubernetes.io/last-applied-configuration"
        - pathToKey: metadata.creationTimestamp
        - pathToKey: metadata.generation
        - pathToKey: spec.ownerReferences
        - pathToKey: metadata.ownerReferences
      default:
        - include: common
        - pathToKey: status
  ```

  where:

  <dl>
  <dt><code>include: common</code></dt>
  <dd>The <code>common</code> group is included in the default group.</dd>
  </dl>

## Configuring inline validation for template fields {#cluster-compare-configure-inline-diff_creating-a-reference-configuration}

You can enable inline regular expressions to validate template fields, especially in scenarios where Golang templating syntax is difficult to maintain or overly complex. Using inline regular expressions simplifies templates, improves readability, and allows for more advanced validation logic.

The `cluster-compare` plugin provides two functions for inline validation:

- `regex`: Validates content in a field using a regular expression.
- `capturegroups`: Enhances multi-line text comparisons by processing non-capture group text as exact matches, applying regular expression matching only within named capture groups, and ensuring consistency for repeated capture groups.

When you use either the `regex` or `capturegroups` function for inline validation, the `cluster-compare` plugin enforces that identically named capture groups have the same values across multiple fields within a template. This means that if a named capture group, such as `(?<username>[a-z0-9]+)`, appears in multiple fields, the values for that group must be consistent throughout the template.

### Configuring inline validation with the regex function {#cluster-compare-configure-regex_creating-a-reference-configuration}

Use the `regex` inline function to validate fields using regular expressions.

**Procedure**

1. Create a `metadata.yaml` file to match your use case. Use the following structure as an example:
   ```yaml
   apiVersion: v2
   parts:
   - name: Part1
     components:
     - name: Example
       allOf:
       - path: example.yaml
         config:
           perField:
           - pathToKey: spec.bigTextBlock
             inlineDiffFunc: regex
   ```

   where:

   <dl>
   <dt><code>pathToKey: spec.bigTextBlock</code></dt>
   <dd>Specifies the field for inline validation.</dd>
   <dt><code>inlineDiffFunc: regex</code></dt>
   <dd>Enables inline validation using regular expressions.</dd>
   </dl>
2. Use a regular expression to validate the field in the associated template:
   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     namespace: dashboard
   data:
     username: "(?<username>[a-z0-9]+)"
     bigTextBlock: |-
       This is a big text block with some static content, like this line.
       It also has a place where (?<username>[a-z0-9]+) would put in their own name. (?<username>[a-z0-9]+) would put in their own name.
   ```

### Configuring inline validation with the capturegroups function {#cluster-compare-configure-capturegroups_creating-a-reference-configuration}

Use the `capturegroups` inline function for more precise validation of fields featuring multi-line strings. This function also ensures that identically named capture groups have the same values across multiple fields.

**Procedure**

1. Create a `metadata.yaml` file to match your use case. Use the following structure as an example:
   ```yaml
   apiVersion: v2
   parts:
   - name: Part1
     components:
     - name: Example
       allOf:
       - path: example.yaml
         config:
           perField:
           - pathToKey: data.username
             inlineDiffFunc: regex
           - pathToKey: spec.bigTextBlock
             inlineDiffFunc: capturegroups
   ```

   where:

   <dl>
   <dt><code>pathToKey: data.username</code></dt>
   <dd>Specifies the field for inline validation.</dd>
   <dt><code>inlineDiffFunc: regex</code></dt>
   <dd>Enables inline validation using capture groups.</dd>
   <dt><code>pathToKey: spec.bigTextBlock</code></dt>
   <dd>Specifies the multi-line field for capture-group validation.</dd>
   <dt><code>inlineDiffFunc: capturegroups</code></dt>
   <dd>Enables inline validation using capture groups.</dd>
   </dl>
2. Use a regular expression to validate the field in the associated template:
   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     namespace: dashboard
   data:
     username: "(?<username>[a-z0-9]+)"
     bigTextBlock: |-
       This static content outside of a capture group should match exactly.
       Here is a username capture group: (?<username>[a-z0-9]+).
       It should match this capture group: (?<username>[a-z0-9]+).
   ```

   If the username value captured in the `data.username` field does not match the value captured in `bigTextBlock`, the `cluster-compare` plugin warns you about the inconsistent matching.

The following is example output with a warning about the inconsistent matching:

```terminal
WARNING: Capturegroup (?<username>…) matched multiple values: « mismatchuser | exampleuser »
```

## Configuring descriptions for the output {#cluster-compare-output-description_creating-a-reference-configuration}

Each part, component, or template can include descriptions to provide additional context, instructions, or documentation links. These descriptions are helpful to convey why a specific template or structure is required.

**Procedure**

- Create a `metadata.yaml` file to match your use case. Use the following structure as an example:
  ```yaml
  apiVersion: v2
  parts:
    - name: Part1
      description: |-
        General text for every template under this part, unless overridden.
      components:
        - name: Component1
          # With no description set, this inherits the description from the part above.
          OneOf:
            - path: Template1.yaml
              # This inherits the component description, if set.
            - path: Template2.yaml
            - path: Template3.yaml
              description: |-
                This template has special instructions that don't apply to the others.
        - name: Component2
          description: |-
            This overrides the part text with something more specific.
            Multi-line text is supported, at all levels.
          allOf:
            - path: RequiredTemplate1.yaml
            - path: RequiredTemplate2.yaml
              description: |-
                Required for important reasons.
            - path: RequiredTemplate3.yaml
  ```
