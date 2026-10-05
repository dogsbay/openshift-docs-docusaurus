---
title: Configuring advanced direct authentication fields
sidebar_position: 9
---

# Configuring advanced direct authentication fields {#structured-auth-config-fields}

<a id="structured-auth-config-fields"></a>

You can configure advanced direct authentication fields in the `authentications.config.openshift.io` custom resource definition (CRD) to enable enhanced OIDC configurations, security enforcement, and flexible token validation for standalone and hosted control plane (HCP) clusters.

:::warning

Advanced direct authentication fields is a Technology Preview feature only. Technology Preview features are not supported with Red Hat production service level agreements (SLAs) and might not be functionally complete. Red Hat does not recommend using them in production. These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.

For more information about the support scope of Red Hat Technology Preview features, see [Technology Preview Features Support Scope](https://access.redhat.com/support/offerings/techpreview/).

:::

## About advanced direct authentication fields {#structured-auth-config-about_structured-auth-config-fields}

Advanced direct authentication fields provide flexibility and security for OIDC-based authentication in OpenShift Container Platform. You can configure advanced OIDC settings, implement custom token validation logic, and enforce security policies on usernames and groups.

The following capabilities are available:

Custom OIDC discovery URL
: Specify a custom OIDC discovery endpoint when your identity provider does not use the standard discovery URL format. Useful for complex networking setups or non-standard identity providers.

CEL-based claim mapping
: Use Common Expression Language (CEL) expressions to construct usernames and groups from JWT token claims with fallback logic. This addresses scenarios where different user types require varying claim mappings.

Claim validation rules
: Use CEL expressions to implement advanced token validation logic, such as enforcing maximum token lifetimes, validating multiple claims, or implementing custom security policies.

User validation rules
: Enforce security policies on usernames and groups extracted from tokens to prevent privilege escalation by blocking reserved system usernames and group prefixes.

These fields extend the base OIDC authentication configuration introduced in OpenShift Container Platform 4.14. You must first configure an external OIDC identity provider before using these advanced fields.

These fields require the `TechPreviewNoUpgrade` feature set to be enabled. They are available on standalone OpenShift Container Platform clusters and hosted control plane (HCP) environments.

:::warning

These advanced authentication fields are available as a Technology Preview feature. Ensure you have a backup authentication method, such as a certificate-based kubeconfig file, before configuring these fields.

:::

## Configure a custom OIDC discovery URL {#structured-auth-config-discovery-url_structured-auth-config-fields}

Configure a custom OIDC discovery URL when your identity provider does not follow the standard discovery endpoint format.

**Prerequisites**

- You have configured an external OIDC identity provider for direct authentication.
- You have access to the cluster as a user with the `cluster-admin` role.
- You have access to a long-lived authentication method, such as a certificate-based kubeconfig file.

**Procedure**

1. Create a YAML file named `authentication-discovery-url.yaml` with your custom discovery URL configuration:
   ```yaml
   apiVersion: config.openshift.io/v1
   kind: Authentication
   metadata:
     name: cluster
   spec:
     type: OIDC
     oidcProviders:
     - name: my-oidc-provider
       issuer:
         issuerURL: https://idp.example.com
         discoveryURL: https://custom-discovery.example.com/.well-known/openid-configuration
         audiences:
         - my-audience
       claimMappings:
         username:
           claim: email
   ```

   where:

   <dl>
   <dt><code>issuerURL</code></dt>
   <dd>Specifies the issuer URL displayed in JWT token <code>iss</code> claim.</dd>
   <dt><code>discoveryURL</code></dt>
   <dd>Specifies the custom OIDC discovery endpoint URL. Must differ from <code>issuerURL</code>, use HTTPS, and must not contain query parameters, fragments, or user info. Maximum length: 2048 characters.</dd>
   </dl>

   :::note

   Replace the placeholder values (`my-oidc-provider`, `https://idp.example.com`, `https://custom-discovery.example.com/.well-known/openid-configuration`, `my-audience`) with your actual OIDC provider configuration.

   :::
2. Apply the configuration:
   ```terminal
   $ oc apply -f authentication-discovery-url.yaml
   ```

**Verification**

- Monitor the cluster authentication Operator status to ensure the configuration is applied successfully:
  ```terminal
  $ oc get clusteroperator authentication
  ```

  The Operator should report `Available=True` and `Degraded=False`.
- Check the cluster authentication Operator logs for any errors:
  ```terminal
  $ oc logs -n openshift-authentication-operator deployments/authentication-operator
  ```
- Verify the kube-apiserver is using the custom discovery URL by checking the authentication configuration:
  ```terminal
  $ oc get authentication.config.openshift.io/cluster -o jsonpath='{.spec.oidcProviders[0].issuer.discoveryURL}'
  ```

  The output should display your custom discovery URL.

## Configure CEL expressions for username and groups claim mapping {#structured-auth-config-cel-claim-mapping_structured-auth-config-fields}

You can use Common Expression Language (CEL) expressions to construct usernames and groups from JWT token claims. This provides flexible claim mapping, including fallback logic when specific claims are not present.

**Prerequisites**

- You have configured an external OIDC identity provider for direct authentication.
- You have access to the cluster as a user with the `cluster-admin` role.
- You have access to a long-lived authentication method, such as a certificate-based kubeconfig file.
- You are familiar with CEL expression syntax.

**Procedure**

1. Create a YAML file named `authentication-cel-mapping.yaml` with your CEL expression configuration:
   :::warning

   When using `expression`, do not set the `claim` field. You must use either `claim` or `expression`, but not both. Setting both will result in a validation error. Additionally, when using `expression`, do not set `prefixPolicy` to `Prefix`. Prefix policies are only compatible with `claim`-based mappings.

   :::

   :::note

   When using the `email` claim in CEL expressions, you must also validate `email_verified` to ensure the email address has been verified by the identity provider.

   :::

   ```yaml
   apiVersion: config.openshift.io/v1
   kind: Authentication
   metadata:
     name: cluster
   spec:
     type: OIDC
     oidcProviders:
     - name: my-oidc-provider
       issuer:
         issuerURL: https://idp.example.com
         audiences:
         - my-audience
       claimMappings:
         username:
           expression: 'claims.?upn.orValue(claims.?oid.orValue(claims.sub))'
         groups:
           expression: 'claims.?groups.orValue([])'
   # ...
   ```

   where:

   <dl>
   <dt><code>username.expression</code></dt>
   <dd>Specifies the fallback logic for username: <code>upn</code> if present, else <code>oid</code>, else <code>sub</code>.</dd>
   <dt><code>groups.expression</code></dt>
   <dd>Specifies that the <code>groups</code> claim is used if present, else an empty array.</dd>
   </dl>

   :::note

   Replace the placeholder values (`my-oidc-provider`, `https://idp.example.com`, `my-audience`) with your actual OIDC provider configuration.

   :::
2. Apply the configuration:
   ```terminal
   $ oc apply -f authentication-cel-mapping.yaml
   ```

**Verification**

- Verify that the authentication configuration is applied successfully:
  ```terminal
  $ oc get authentication.config.openshift.io/cluster -o yaml
  ```
- Authenticate with a user account and verify the username is constructed correctly:
  ```terminal
  $ oc whoami
  ```
- Monitor the cluster authentication Operator status:
  ```terminal
  $ oc get clusteroperator authentication
  ```

  The Operator should report `Available=True` and `Degraded=False`.

  :::note

  CEL expressions have access to standard CEL string functions (`lowerAscii()`, `upperAscii()`, `contains()`, `startsWith()`, `endsWith()`, `matches()`, `split()`), operators (`+`, `?`, `has()`, ternary `? :`), and the `orValue()` method for optional chaining. See the CEL specification link in Additional resources for the complete function reference.

  :::

You can use the following CEL expression patterns for claim mapping:

Use the optional chaining Operator `?` to safely access claims that might not exist
: ```yaml
  username:
    expression: 'claims.email_verified ? claims.email : claims.sub'
  ```

  Uses `email` if verified, otherwise `sub`. When using the `email` claim, you must also check `email_verified`.

Concatenate multiple claims
: ```yaml
  username:
    expression: 'claims.givenname + "." + claims.surname'
  ```

  Combines given name and surname claims.

Transform claim values
: ```yaml
  username:
    expression: 'claims.email.lowerAscii()'
  ```

  Converts email to lowercase.

Conditional logic for different user types
: ```yaml
  username:
    expression: 'has(claims.upn) ? claims.upn : claims.oid'
  ```

  Uses `upn` for regular users, `oid` for service principals.

Extract domain from email
: ```yaml
  groups:
    expression: 'claims.?email.orValue("").split("@").size() > 1 ? [claims.email.split("@")[1]] : []'
  ```

  Safely extracts domain from email address for group assignment, returning an empty array if email is missing or malformed.

Combine group sources
: ```yaml
  groups:
    expression: 'claims.?groups.orValue([]) + claims.?roles.orValue([])'
  ```

  Combines `groups` and `roles` claims.

## Configure claim validation rules {#structured-auth-config-claim-validation_structured-auth-config-fields}

Use Common Expression Language (CEL) expressions to define custom validation rules for JWT token claims and enforce advanced security policies such as maximum token lifetimes.

:::warning

All claim validation rules must pass for authentication to succeed. Incorrectly configured validation rules can lock all users out of the cluster. Ensure you complete the following tasks:

- Have a backup authentication method, such as a certificate-based kubeconfig file, before applying these rules
- Test validation rules in a non-production environment first
- Verify your CEL expressions are correct to avoid blocking valid users from accessing the cluster

:::

**Prerequisites**

- You have configured an external OIDC identity provider for direct authentication.
- You have access to the cluster as a user with the `cluster-admin` role.
- You have access to a long-lived authentication method, such as a certificate-based kubeconfig file.
- You are familiar with CEL expression syntax.

**Procedure**

1. Create a YAML file named `authentication-claim-validation.yaml` with your claim validation rules:
   ```yaml
   apiVersion: config.openshift.io/v1
   kind: Authentication
   metadata:
     name: cluster
   spec:
     type: OIDC
     oidcProviders:
     - name: my-oidc-provider
       issuer:
         issuerURL: https://idp.example.com
         audiences:
         - my-audience
       claimMappings:
         username:
           claim: email
       claimValidationRules:
       - type: CEL
         cel:
           expression: 'claims.exp - claims.nbf <= 86400'
           message: 'Total token lifetime must not exceed 24 hours'
       - type: CEL
         cel:
           expression: 'has(claims.email) && claims.email_verified && claims.email.contains("@example.com")'
           message: 'Email claim must be verified and from example.com domain'
   ```

   where:

   <dl>
   <dt><code>claimValidationRules</code></dt>
   <dd>Specifies an array of validation rules. All must pass for authentication.</dd>
   <dt><code>type</code></dt>
   <dd>Specifies the validation type. Set to <code>CEL</code> for CEL-based validation.</dd>
   <dt><code>cel.expression</code></dt>
   <dd>Specifies the CEL expression that must evaluate to <code>true</code>.</dd>
   <dt><code>cel.message</code></dt>
   <dd>Specifies the error message displayed when validation fails.</dd>
   </dl>

   :::note

   Replace the placeholder values (`my-oidc-provider`, `https://idp.example.com`, `my-audience`, `@example.com`) with your actual OIDC provider configuration and validation requirements.

   :::
2. Apply the configuration:
   ```terminal
   $ oc apply -f authentication-claim-validation.yaml
   ```

**Verification**

- Verify that the authentication configuration is applied successfully:
  ```terminal
  $ oc get authentication.config.openshift.io/cluster -o yaml
  ```
- Authenticate with a token that matches your validation rules to confirm they are enforced correctly.
- Check the cluster authentication Operator logs for validation errors:
  ```terminal
  $ oc logs -n openshift-authentication-operator deployments/authentication-operator
  ```

When writing CEL expressions for claim validation:

- Access claims using `claims` variable (for example, `claims.sub` or `claims.foo.bar` for nested claims)
- Expressions must evaluate to boolean values
- Use `has()` to check claim existence
- Standard CEL operators and functions available: `&&`, `||`, `!`, `contains()`, `startsWith()`, `endsWith()`

Common use cases include:

Enforce maximum token lifetime
: ```yaml
  claimValidationRules:
  - type: CEL
    cel:
      expression: 'claims.exp - claims.nbf <= 86400'
      message: 'Token lifetime must not exceed 24 hours'
  ```

Require specific claim values
: ```yaml
  claimValidationRules:
  - type: CEL
    cel:
      expression: 'claims.tenant == "production"'
      message: 'Only production tenant tokens are allowed'
  ```

Validate email domain
: ```yaml
  claimValidationRules:
  - type: CEL
    cel:
      expression: 'has(claims.email) && claims.email_verified && claims.email.endsWith("@trusted-domain.com")'
      message: 'Email must be verified and from trusted-domain.com'
  ```

  When using the `email` claim, you must also check `email_verified` to ensure the email address has been verified by the identity provider.

Combine conditions
: ```yaml
  claimValidationRules:
  - type: CEL
    cel:
      expression: 'has(claims.role) && (claims.role == "admin" || claims.role == "developer")'
      message: 'User must have admin or developer role'
  ```

If authentication fails after the configuration is applied

**Troubleshooting**
: Even if your claim validation rules pass the configuration validation gates, runtime authentication errors can occur. Check the kube-apiserver logs for detailed error messages explaining why authentication failed:

  ```terminal
  $ oc logs -n openshift-kube-apiserver -l app=openshift-kube-apiserver | grep -i auth
  ```

  These log messages will indicate which validation rule failed and include the custom `message` you specified in the CEL expression.

If you incorrectly configure validation rules and lock users out of the cluster
: 1. Use your certificate-based kubeconfig to authenticate as `cluster-admin`.
  2. Edit the Authentication custom resource to remove or fix invalid rules:

  ```terminal
  $ oc edit authentication.config.openshift.io/cluster
  ```

  1. Monitor the cluster authentication Operator to confirm it returns to `Available` status:

  ```terminal
  $ oc get clusteroperator authentication
  ```

## Configure user validation rules {#structured-auth-config-user-validation_structured-auth-config-fields}

You can define validation rules to enforce security policies on the user object created from an authenticated token. This helps prevent privilege escalation by blocking reserved usernames and group prefixes.

:::note

User validation rules are evaluated after claim mapping is complete, including all prefix transformations. Your CEL expressions must validate the final username and group names as they will appear in RBAC policies, not the raw claim values from the JWT token.

:::

:::warning

All user validation rules must pass for authentication to succeed. Incorrectly configured validation rules can lock all users out of the cluster. Ensure you:

- Have a backup authentication method, such as a certificate-based kubeconfig file, before applying these rules
- Test validation rules in a non-production environment first
- Verify your CEL expressions correctly validate the final username and groups to avoid blocking valid users or allowing unauthorized access

:::

**Prerequisites**

- You have configured an external OIDC identity provider for direct authentication.
- You have access to the cluster as a user with the `cluster-admin` role.
- You have access to a long-lived authentication method, such as a certificate-based kubeconfig file.
- You are familiar with CEL expression syntax.

**Procedure**

1. Create a YAML file named `authentication-user-validation.yaml` with your user validation rules:
   ```yaml
   apiVersion: config.openshift.io/v1
   kind: Authentication
   metadata:
     name: cluster
   spec:
     type: OIDC
     oidcProviders:
     - name: my-oidc-provider
       issuer:
         issuerURL: https://idp.example.com
         audiences:
         - my-audience
       claimMappings:
         username:
           claim: email
         groups:
           claim: groups
       userValidationRules:
       - expression: "!user.username.startsWith('system:')"
         message: 'Username cannot use reserved system: prefix'
       - expression: "!user.groups.exists(g, g.startsWith('system:'))"
         message: 'Groups cannot use reserved system: prefix'
   ```

   where:

   <dl>
   <dt><code>userValidationRules</code></dt>
   <dd>Specifies an array of validation rules. All must pass for authentication.</dd>
   <dt><code>expression</code></dt>
   <dd>Specifies the CEL expression that must evaluate to <code>true</code>.</dd>
   <dt><code>message</code></dt>
   <dd>Specifies the error message displayed when validation fails.</dd>
   </dl>

   :::note

   Replace the placeholder values (`my-oidc-provider`, `https://idp.example.com`, `my-audience`) with your actual OIDC provider configuration.

   :::
2. Apply the configuration:
   ```terminal
   $ oc apply -f authentication-user-validation.yaml
   ```

**Verification**

- Verify that the authentication configuration is applied successfully:
  ```terminal
  $ oc get authentication.config.openshift.io/cluster -o yaml
  ```
- Authenticate with credentials that would create a user matching your validation rules to confirm they are enforced correctly.
- Check the cluster authentication Operator logs for validation errors:
  ```terminal
  $ oc logs -n openshift-authentication-operator deployments/authentication-operator
  ```

When writing CEL expressions for user validation:

- Access user fields: `user.username`, `user.groups` (array), `user.uid`, `user.extra` (map)
- Expressions must evaluate to boolean values
- Use `startsWith()`, `endsWith()`, `contains()` for string matching
- Use `exists()` for array checks (for example, `user.groups.exists(g, g == "admin")`)

Common use cases include:

Prevent reserved username prefixes
: ```yaml
  userValidationRules:
  - expression: "!user.username.startsWith('system:')"
    message: 'Username cannot use reserved system: prefix'
  ```

Prevent reserved group prefixes
: ```yaml
  userValidationRules:
  - expression: "!user.groups.exists(g, g.startsWith('system:'))"
    message: 'Groups cannot use reserved system: prefix'
  ```

Require username format
: ```yaml
  userValidationRules:
  - expression: "user.username.matches('^[a-z0-9]([-a-z0-9]*[a-z0-9])?$')"
    message: 'Username must be a valid DNS subdomain'
  ```

Validate group membership
: ```yaml
  userValidationRules:
  - expression: "user.groups.exists(g, g == 'verified-users')"
    message: 'User must be a member of verified-users group'
  ```

Combine conditions
: ```yaml
  userValidationRules:
  - expression: "!user.username.startsWith('system:') && !user.username.startsWith('kube:')"
    message: 'Username cannot use reserved system: or kube: prefixes'
  ```

**Troubleshooting**

If you incorrectly configure validation rules and lock users out of the cluster:

1. Use your certificate-based kubeconfig to authenticate as `cluster-admin`.
2. Edit the Authentication custom resource to remove or fix invalid rules:
   ```terminal
   $ oc edit authentication.config.openshift.io/cluster
   ```
3. Monitor the cluster authentication Operator to confirm it returns to `Available` status:
   ```terminal
   $ oc get clusteroperator authentication
   ```

## Advanced authentication field reference {#structured-auth-config-fields-reference_structured-auth-config-fields}

The following table describes the advanced authentication configuration fields available as Technology Preview in OpenShift Container Platform.

**Advanced **`oidcProviders`** configuration fields**

<table>
<thead>
<tr>
  <th>Parameter</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>issuer.discoveryURL</code></td>
  <td>Optional parameter. Custom OIDC discovery endpoint URL for retrieving identity provider metadata from a non-standard location.<br><br>Requirements:<br><br><ul><li>Must be a valid HTTPS URL</li><li>Must differ from <code>issuer.issuerURL</code></li></ul>When not specified, OpenShift Container Platform constructs the discovery URL by using the standard OIDC format: <code>{issuerURL}/.well-known/openid-configuration</code>.<br><br><strong>Example:</strong><br><br><pre><code>issuer:&#10;  issuerURL: https://idp.example.com&#10;  discoveryURL: https://custom-discovery.example.com/.well-known/openid-configuration</code></pre></td>
</tr>
<tr>
  <td><code>claimValidationRules</code></td>
  <td>Optional parameter. Array of validation rules for JWT token claims using Common Expression Language (CEL) expressions. All rules must evaluate to <code>true</code> for authentication to succeed (AND operation).<br><br>Each rule has:<br><br><ul><li><code>type</code>: Set to <code>CEL</code> for CEL-based validation</li><li><code>cel</code>: Object with <code>expression</code> (must evaluate to <code>true</code>) and <code>message</code> (error text)</li></ul>CEL expressions access claims by using <code>claims</code> variable (for example, <code>claims.sub</code>).<br><br><strong>Example:</strong><br><br><pre><code>claimValidationRules:&#10;- type: CEL&#10;  cel:&#10;    expression: 'claims.exp - claims.nbf &lt;= 86400'&#10;    message: 'Total token lifetime must not exceed 24 hours'&#10;- type: CEL&#10;  cel:&#10;    expression: 'has(claims.email) &amp;&amp; claims.email.contains("@example.com")'&#10;    message: 'Email claim must be present and from example.com domain'</code></pre></td>
</tr>
<tr>
  <td><code>claimValidationRules[].type</code></td>
  <td>Required. Validation rule type. Set to <code>CEL</code> for CEL-based validation. Requires <code>cel</code> field.</td>
</tr>
<tr>
  <td><code>claimValidationRules[].cel</code></td>
  <td>Required when <code>type</code> is <code>CEL</code>. Contains <code>expression</code> (CEL expression to evaluate) and <code>message</code> (error text).</td>
</tr>
<tr>
  <td><code>claimValidationRules[].cel.expression</code></td>
  <td>Required. CEL expression that validates token claims. Must evaluate to <code>true</code> for authentication to succeed.<br><br>Constraints: 1-1024 characters, must evaluate to boolean.<br><br>Access claims by using <code>claims</code> variable: <code>claims.sub</code>, <code>claims.foo.bar</code> (nested), <code>has(claims.email)</code> (existence check).<br><br><div class="alert alert--secondary"><strong>Note</strong><p>When using the <code>email</code> claim in CEL expressions, you must also validate the <code>email_verified</code> claim to ensure the email address has been verified by the identity provider. For example: <code>claims.email_verified &amp;&amp; claims.email.endsWith("@example.com")</code>.</p></div></td>
</tr>
<tr>
  <td><code>claimValidationRules[].cel.message</code></td>
  <td>Required. Error message displayed when validation fails. Constraints: 1-256 characters.</td>
</tr>
<tr>
  <td><code>userValidationRules</code></td>
  <td>Optional parameter. Array of validation rules for user objects by using CEL expressions. All rules must evaluate to <code>true</code> for authentication to succeed (AND operation).<br><br>Each rule has <code>expression</code> (must evaluate to <code>true</code>) and <code>message</code> (error text).<br><br>CEL expressions access user object by using <code>user</code> variable: <code>user.username</code> (string), <code>user.groups</code> (array), <code>user.uid</code> (string), <code>user.extra</code> (map).<br><br><strong>Example:</strong><br><br><pre><code>userValidationRules:&#10;- expression: "!user.username.startsWith('system:')"&#10;  message: 'Username cannot use reserved system: prefix'&#10;- expression: "!user.groups.exists(g, g.startsWith('system:'))"&#10;  message: 'Groups cannot use reserved system: prefix'</code></pre></td>
</tr>
<tr>
  <td><code>userValidationRules[].expression</code></td>
  <td>Required. CEL expression that validates the user object. Must evaluate to <code>true</code> for authentication to succeed.<br><br>Constraints: 1-1024 characters, must evaluate to boolean.<br><br>Access user fields: <code>user.username.startsWith('system:')</code>, <code>user.groups.exists(g, g == "admin")</code>, <code>user.extra["example.com/role"]</code>.</td>
</tr>
<tr>
  <td><code>userValidationRules[].message</code></td>
  <td>Required. Error message displayed when validation fails. Must not be empty.</td>
</tr>
</tbody>
</table>

**Additional resources**

- [Enabling direct authentication with an external OIDC identity provider](/docs/authentication/external-auth#external-auth)
- [Common Expression Language (CEL) specification](https://cel.dev/)
- [Common Expression Language in Kubernetes](https://kubernetes.io/docs/reference/using-api/cel/)
