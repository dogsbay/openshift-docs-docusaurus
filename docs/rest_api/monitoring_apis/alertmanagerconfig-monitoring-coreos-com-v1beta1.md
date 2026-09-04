---
title: "AlertmanagerConfig [monitoring.coreos.com/v1beta1]"
sidebar_position: 3
---

# AlertmanagerConfig [monitoring.coreos.com/v1beta1] {#alertmanagerconfig-monitoring-coreos-com-v1beta1}

<a id="alertmanagerconfig-monitoring-coreos-com-v1beta1"></a>

Description
: The `AlertmanagerConfig` custom resource definition (CRD) defines how `Alertmanager` objects process Prometheus alerts. It allows to specify alert grouping and routing, notification receivers and inhibition rules.

  `Alertmanager` objects select `AlertmanagerConfig` objects using label and namespace selectors.

Type
: ```
  `object`
  ```

Required
: - `spec`

## Specification {#_specification}

| Property | Type | Description |
| --- | --- | --- |
| `apiVersion` | `string` | APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources |
| `kind` | `string` | Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds |
| `metadata` | [`ObjectMeta`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-ObjectMeta) | Standard object’s metadata. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata |
| `spec` | `object` | spec defines the specification of AlertmanagerConfigSpec |
| `status` | `object` | status defines the status subresource. It is under active development and is updated only when the "StatusForConfigurationResources" feature gate is enabled.<br>Most recent observed status of the ServiceMonitor. Read-only. More info: https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#spec-and-status |

### .spec {#_spec}

Description
: spec defines the specification of AlertmanagerConfigSpec

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `inhibitRules` | `array` | inhibitRules defines the list of inhibition rules. The rules will only apply to alerts matching the resource’s namespace. |
| `inhibitRules[]` | `object` | InhibitRule defines an inhibition rule that allows to mute alerts when other alerts are already firing. See https://prometheus.io/docs/alerting/latest/configuration/#inhibit_rule |
| `receivers` | `array` | receivers defines the list of receivers. |
| `receivers[]` | `object` | Receiver defines one or more notification integrations. |
| `route` | `object` | route defines the Alertmanager route definition for alerts matching the resource’s namespace. If present, it will be added to the generated Alertmanager configuration as a first-level route. |
| `timeIntervals` | `array` | timeIntervals defines the list of timeIntervals specifying when the routes should be muted. |
| `timeIntervals[]` | `object` | TimeInterval specifies the periods in time when notifications will be muted or active. |

### .spec.inhibitRules {#_specinhibitrules}

Description
: inhibitRules defines the list of inhibition rules. The rules will only apply to alerts matching the resource’s namespace.

Type
: ```
  `array`
  ```

### .spec.inhibitRules[] {#_specinhibitrules}

Description
: InhibitRule defines an inhibition rule that allows to mute alerts when other alerts are already firing. See https://prometheus.io/docs/alerting/latest/configuration/#inhibit_rule

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `equal` | `array (string)` | equal defines labels that must have an equal value in the source and target alert for the inhibition to take effect. This ensures related alerts are properly grouped. |
| `sourceMatch` | `array` | sourceMatch defines matchers for which one or more alerts have to exist for the inhibition to take effect. The operator enforces that the alert matches the resource’s namespace. These are the "trigger" alerts that cause other alerts to be inhibited. |
| `sourceMatch[]` | `object` | Matcher defines how to match on alert’s labels. |
| `targetMatch` | `array` | targetMatch defines matchers that have to be fulfilled in the alerts to be muted. The operator enforces that the alert matches the resource’s namespace. When these conditions are met, matching alerts will be inhibited (silenced). |
| `targetMatch[]` | `object` | Matcher defines how to match on alert’s labels. |

### .spec.inhibitRules[].sourceMatch {#_specinhibitrulessourcematch}

Description
: sourceMatch defines matchers for which one or more alerts have to exist for the inhibition to take effect. The operator enforces that the alert matches the resource’s namespace. These are the "trigger" alerts that cause other alerts to be inhibited.

Type
: ```
  `array`
  ```

### .spec.inhibitRules[].sourceMatch[] {#_specinhibitrulessourcematch}

Description
: Matcher defines how to match on alert’s labels.

Type
: ```
  `object`
  ```

Required
: - `name`

| Property | Type | Description |
| --- | --- | --- |
| `matchType` | `string` | matchType defines the match operation available with AlertManager >= v0.22.0. Takes precedence over Regex (deprecated) if non-empty. Valid values: "=" (equality), "!=" (inequality), "=~" (regex match), "!~" (regex non-match). |
| `name` | `string` | name defines the label to match. This specifies which alert label should be evaluated. |
| `value` | `string` | value defines the label value to match. This is the expected value for the specified label. |

### .spec.inhibitRules[].targetMatch {#_specinhibitrulestargetmatch}

Description
: targetMatch defines matchers that have to be fulfilled in the alerts to be muted. The operator enforces that the alert matches the resource’s namespace. When these conditions are met, matching alerts will be inhibited (silenced).

Type
: ```
  `array`
  ```

### .spec.inhibitRules[].targetMatch[] {#_specinhibitrulestargetmatch}

Description
: Matcher defines how to match on alert’s labels.

Type
: ```
  `object`
  ```

Required
: - `name`

| Property | Type | Description |
| --- | --- | --- |
| `matchType` | `string` | matchType defines the match operation available with AlertManager >= v0.22.0. Takes precedence over Regex (deprecated) if non-empty. Valid values: "=" (equality), "!=" (inequality), "=~" (regex match), "!~" (regex non-match). |
| `name` | `string` | name defines the label to match. This specifies which alert label should be evaluated. |
| `value` | `string` | value defines the label value to match. This is the expected value for the specified label. |

### .spec.receivers {#_specreceivers}

Description
: receivers defines the list of receivers.

Type
: ```
  `array`
  ```

### .spec.receivers[] {#_specreceivers}

Description
: Receiver defines one or more notification integrations.

Type
: ```
  `object`
  ```

Required
: - `name`

| Property | Type | Description |
| --- | --- | --- |
| `discordConfigs` | `array` | discordConfigs defines the list of Discord configurations. |
| `discordConfigs[]` | `object` | DiscordConfig configures notifications via Discord. See https://prometheus.io/docs/alerting/latest/configuration/#discord_config |
| `emailConfigs` | `array` | emailConfigs defines the list of Email configurations. |
| `emailConfigs[]` | `object` | EmailConfig configures notifications via Email. |
| `msteamsConfigs` | `array` | msteamsConfigs defines the list of MSTeams configurations. It requires Alertmanager >= 0.26.0. |
| `msteamsConfigs[]` | `object` | MSTeamsConfig configures notifications via Microsoft Teams. It requires Alertmanager >= 0.26.0. |
| `msteamsv2Configs` | `array` | msteamsv2Configs defines the list of MSTeamsV2 configurations. It requires Alertmanager >= 0.28.0. |
| `msteamsv2Configs[]` | `object` | MSTeamsV2Config configures notifications via Microsoft Teams using the new message format with adaptive cards as required by flows. See https://prometheus.io/docs/alerting/latest/configuration/#msteamsv2_config It requires Alertmanager >= 0.28.0. |
| `name` | `string` | name defines the name of the receiver. Must be unique across all items from the list. |
| `opsgenieConfigs` | `array` | opsgenieConfigs defines the list of OpsGenie configurations. |
| `opsgenieConfigs[]` | `object` | OpsGenieConfig configures notifications via OpsGenie. See https://prometheus.io/docs/alerting/latest/configuration/#opsgenie_config |
| `pagerdutyConfigs` | `array` | pagerdutyConfigs defines the List of PagerDuty configurations. |
| `pagerdutyConfigs[]` | `object` | PagerDutyConfig configures notifications via PagerDuty. See https://prometheus.io/docs/alerting/latest/configuration/#pagerduty_config |
| `pushoverConfigs` | `array` | pushoverConfigs defines the list of Pushover configurations. |
| `pushoverConfigs[]` | `object` | PushoverConfig configures notifications via Pushover. See https://prometheus.io/docs/alerting/latest/configuration/#pushover_config |
| `rocketchatConfigs` | `array` | rocketchatConfigs defines the list of RocketChat configurations. It requires Alertmanager >= 0.28.0. |
| `rocketchatConfigs[]` | `object` | RocketChatConfig configures notifications via RocketChat. It requires Alertmanager >= 0.28.0. |
| `slackConfigs` | `array` | slackConfigs defines the list of Slack configurations. |
| `slackConfigs[]` | `object` | SlackConfig configures notifications via Slack. See https://prometheus.io/docs/alerting/latest/configuration/#slack_config |
| `snsConfigs` | `array` | snsConfigs defines the list of SNS configurations |
| `snsConfigs[]` | `object` | SNSConfig configures notifications via AWS SNS. See https://prometheus.io/docs/alerting/latest/configuration/#sns_configs |
| `telegramConfigs` | `array` | telegramConfigs defines the list of Telegram configurations. |
| `telegramConfigs[]` | `object` | TelegramConfig configures notifications via Telegram. See https://prometheus.io/docs/alerting/latest/configuration/#telegram_config |
| `victoropsConfigs` | `array` | victoropsConfigs defines the list of VictorOps configurations. |
| `victoropsConfigs[]` | `object` | VictorOpsConfig configures notifications via VictorOps. See https://prometheus.io/docs/alerting/latest/configuration/#victorops_config |
| `webexConfigs` | `array` | webexConfigs defines the list of Webex configurations. |
| `webexConfigs[]` | `object` | WebexConfig configures notification via Cisco Webex See https://prometheus.io/docs/alerting/latest/configuration/#webex_config |
| `webhookConfigs` | `array` | webhookConfigs defines the List of webhook configurations. |
| `webhookConfigs[]` | `object` | WebhookConfig configures notifications via a generic receiver supporting the webhook payload. See https://prometheus.io/docs/alerting/latest/configuration/#webhook_config |
| `wechatConfigs` | `array` | wechatConfigs defines the list of WeChat configurations. |
| `wechatConfigs[]` | `object` | WeChatConfig configures notifications via WeChat. See https://prometheus.io/docs/alerting/latest/configuration/#wechat_config |

### .spec.receivers[].discordConfigs {#_specreceiversdiscordconfigs}

Description
: discordConfigs defines the list of Discord configurations.

Type
: ```
  `array`
  ```

### .spec.receivers[].discordConfigs[] {#_specreceiversdiscordconfigs}

Description
: DiscordConfig configures notifications via Discord. See https://prometheus.io/docs/alerting/latest/configuration/#discord_config

Type
: ```
  `object`
  ```

Required
: - `apiURL`

| Property | Type | Description |
| --- | --- | --- |
| `apiURL` | `object` | apiURL defines the secret’s key that contains the Discord webhook URL. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `avatarURL` | `string` | avatarURL defines the avatar url of the message sender. |
| `content` | `string` | content defines the template of the content’s body. |
| `httpConfig` | `object` | httpConfig defines HTTP client configuration. |
| `message` | `string` | message defines the template of the message’s body. |
| `sendResolved` | `boolean` | sendResolved defines whether or not to notify about resolved alerts. |
| `title` | `string` | title defines the template of the message’s title. |
| `username` | `string` | username defines the username of the message sender. |

### .spec.receivers[].discordConfigs[].apiURL {#_specreceiversdiscordconfigsapiurl}

Description
: apiURL defines the secret’s key that contains the Discord webhook URL. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].discordConfigs[].httpConfig {#_specreceiversdiscordconfigshttpconfig}

Description
: httpConfig defines HTTP client configuration.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `authorization` | `object` | authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+. |
| `basicAuth` | `object` | basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence. |
| `bearerTokenSecret` | `object` | bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `enableHttp2` | `boolean` | enableHttp2 can be used to disable HTTP2. |
| `followRedirects` | `boolean` | followRedirects defines whether HTTP requests follow HTTP 3xx redirects. When true, the client will automatically follow redirect responses. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `oauth2` | `object` | oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyURL` | `string` | proxyURL defines an optional proxy URL for HTTP requests. If defined, this field takes precedence over `proxyUrl`. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options. |

### .spec.receivers[].discordConfigs[].httpConfig.authorization {#_specreceiversdiscordconfigshttpconfigauthorization}

Description
: authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `credentials` | `object` | credentials defines a key of a Secret in the namespace that contains the credentials for authentication. |
| `type` | `string` | type defines the authentication type. The value is case-insensitive.<br>"Basic" is not a supported value.<br>Default: "Bearer" |

### .spec.receivers[].discordConfigs[].httpConfig.authorization.credentials {#_specreceiversdiscordconfigshttpconfigauthorizationcredentials}

Description
: credentials defines a key of a Secret in the namespace that contains the credentials for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].discordConfigs[].httpConfig.basicAuth {#_specreceiversdiscordconfigshttpconfigbasicauth}

Description
: basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `password` | `object` | password defines a key of a Secret containing the password for authentication. |
| `username` | `object` | username defines a key of a Secret containing the username for authentication. |

### .spec.receivers[].discordConfigs[].httpConfig.basicAuth.password {#_specreceiversdiscordconfigshttpconfigbasicauthpassword}

Description
: password defines a key of a Secret containing the password for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].discordConfigs[].httpConfig.basicAuth.username {#_specreceiversdiscordconfigshttpconfigbasicauthusername}

Description
: username defines a key of a Secret containing the username for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].discordConfigs[].httpConfig.bearerTokenSecret {#_specreceiversdiscordconfigshttpconfigbearertokensecret}

Description
: bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].discordConfigs[].httpConfig.oauth2 {#_specreceiversdiscordconfigshttpconfigoauth2}

Description
: oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests.

Type
: ```
  `object`
  ```

Required
: - `clientId`
  - `clientSecret`
  - `tokenUrl`

| Property | Type | Description |
| --- | --- | --- |
| `clientId` | `object` | clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID. |
| `clientSecret` | `object` | clientSecret defines a key of a Secret containing the OAuth2 client’s secret. |
| `endpointParams` | `object (string)` | endpointParams configures the HTTP parameters to append to the token URL. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `scopes` | `array (string)` | scopes defines the OAuth2 scopes used for the token request. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0. |
| `tokenUrl` | `string` | tokenUrl defines the URL to fetch the token from. |

### .spec.receivers[].discordConfigs[].httpConfig.oauth2.clientId {#_specreceiversdiscordconfigshttpconfigoauth2clientid}

Description
: clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].discordConfigs[].httpConfig.oauth2.clientId.configMap {#_specreceiversdiscordconfigshttpconfigoauth2clientidconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].discordConfigs[].httpConfig.oauth2.clientId.secret {#_specreceiversdiscordconfigshttpconfigoauth2clientidsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].discordConfigs[].httpConfig.oauth2.clientSecret {#_specreceiversdiscordconfigshttpconfigoauth2clientsecret}

Description
: clientSecret defines a key of a Secret containing the OAuth2 client’s secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].discordConfigs[].httpConfig.oauth2.proxyConnectHeader {#_specreceiversdiscordconfigshttpconfigoauth2proxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].discordConfigs[].httpConfig.oauth2.proxyConnectHeader{} {#_specreceiversdiscordconfigshttpconfigoauth2proxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].discordConfigs[].httpConfig.oauth2.proxyConnectHeader{}[] {#_specreceiversdiscordconfigshttpconfigoauth2proxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].discordConfigs[].httpConfig.oauth2.tlsConfig {#_specreceiversdiscordconfigshttpconfigoauth2tlsconfig}

Description
: tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].discordConfigs[].httpConfig.oauth2.tlsConfig.ca {#_specreceiversdiscordconfigshttpconfigoauth2tlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].discordConfigs[].httpConfig.oauth2.tlsConfig.ca.configMap {#_specreceiversdiscordconfigshttpconfigoauth2tlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].discordConfigs[].httpConfig.oauth2.tlsConfig.ca.secret {#_specreceiversdiscordconfigshttpconfigoauth2tlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].discordConfigs[].httpConfig.oauth2.tlsConfig.cert {#_specreceiversdiscordconfigshttpconfigoauth2tlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].discordConfigs[].httpConfig.oauth2.tlsConfig.cert.configMap {#_specreceiversdiscordconfigshttpconfigoauth2tlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].discordConfigs[].httpConfig.oauth2.tlsConfig.cert.secret {#_specreceiversdiscordconfigshttpconfigoauth2tlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].discordConfigs[].httpConfig.oauth2.tlsConfig.keySecret {#_specreceiversdiscordconfigshttpconfigoauth2tlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].discordConfigs[].httpConfig.proxyConnectHeader {#_specreceiversdiscordconfigshttpconfigproxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].discordConfigs[].httpConfig.proxyConnectHeader{} {#_specreceiversdiscordconfigshttpconfigproxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].discordConfigs[].httpConfig.proxyConnectHeader{}[] {#_specreceiversdiscordconfigshttpconfigproxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].discordConfigs[].httpConfig.tlsConfig {#_specreceiversdiscordconfigshttpconfigtlsconfig}

Description
: tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].discordConfigs[].httpConfig.tlsConfig.ca {#_specreceiversdiscordconfigshttpconfigtlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].discordConfigs[].httpConfig.tlsConfig.ca.configMap {#_specreceiversdiscordconfigshttpconfigtlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].discordConfigs[].httpConfig.tlsConfig.ca.secret {#_specreceiversdiscordconfigshttpconfigtlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].discordConfigs[].httpConfig.tlsConfig.cert {#_specreceiversdiscordconfigshttpconfigtlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].discordConfigs[].httpConfig.tlsConfig.cert.configMap {#_specreceiversdiscordconfigshttpconfigtlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].discordConfigs[].httpConfig.tlsConfig.cert.secret {#_specreceiversdiscordconfigshttpconfigtlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].discordConfigs[].httpConfig.tlsConfig.keySecret {#_specreceiversdiscordconfigshttpconfigtlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].emailConfigs {#_specreceiversemailconfigs}

Description
: emailConfigs defines the list of Email configurations.

Type
: ```
  `array`
  ```

### .spec.receivers[].emailConfigs[] {#_specreceiversemailconfigs}

Description
: EmailConfig configures notifications via Email.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `authIdentity` | `string` | authIdentity defines the identity to use for SMTP authentication. This is typically used with PLAIN authentication mechanism. |
| `authPassword` | `object` | authPassword defines the secret’s key that contains the password to use for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `authSecret` | `object` | authSecret defines the secret’s key that contains the CRAM-MD5 secret. This is used for CRAM-MD5 authentication mechanism. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `authUsername` | `string` | authUsername defines the username to use for SMTP authentication. This is used for SMTP AUTH when the server requires authentication. |
| `forceImplicitTLS` | `boolean` | forceImplicitTLS defines whether to force use of implicit TLS (direct TLS connection) for better security. true: force use of implicit TLS (direct TLS connection on any port) false: force disable implicit TLS (use explicit TLS/STARTTLS if required) nil (default): auto-detect based on port (465=implicit, other=explicit) for backward compatibility It requires Alertmanager >= v0.31.0. |
| `from` | `string` | from defines the sender address for email notifications. This appears as the "From" field in the email header. |
| `headers` | `array` | headers defines additional email header key/value pairs. These override any headers previously set by the notification implementation. |
| `headers[]` | `object` | KeyValue defines a (key, value) tuple. |
| `hello` | `string` | hello defines the hostname to identify to the SMTP server. This is used in the SMTP HELO/EHLO command during the connection handshake. |
| `html` | `string` | html defines the HTML body of the email notification. This allows for rich formatting in the email content. |
| `requireTLS` | `boolean` | requireTLS defines the SMTP TLS requirement. Note that Go does not support unencrypted connections to remote SMTP endpoints. |
| `sendResolved` | `boolean` | sendResolved defines whether or not to notify about resolved alerts. |
| `smarthost` | `string` | smarthost defines the SMTP host and port through which emails are sent. Format should be "hostname:port", e.g. "smtp.example.com:587". |
| `text` | `string` | text defines the plain text body of the email notification. This provides a fallback for email clients that don’t support HTML. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration for SMTP connections. This includes settings for certificates, CA validation, and TLS protocol options. |
| `to` | `string` | to defines the email address to send notifications to. This is the recipient address for alert notifications. |

### .spec.receivers[].emailConfigs[].authPassword {#_specreceiversemailconfigsauthpassword}

Description
: authPassword defines the secret’s key that contains the password to use for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].emailConfigs[].authSecret {#_specreceiversemailconfigsauthsecret}

Description
: authSecret defines the secret’s key that contains the CRAM-MD5 secret. This is used for CRAM-MD5 authentication mechanism. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].emailConfigs[].headers {#_specreceiversemailconfigsheaders}

Description
: headers defines additional email header key/value pairs. These override any headers previously set by the notification implementation.

Type
: ```
  `array`
  ```

### .spec.receivers[].emailConfigs[].headers[] {#_specreceiversemailconfigsheaders}

Description
: KeyValue defines a (key, value) tuple.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `value`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the tuple. This is the identifier or name part of the key-value pair. |
| `value` | `string` | value defines the value of the tuple. This is the data or content associated with the key. |

### .spec.receivers[].emailConfigs[].tlsConfig {#_specreceiversemailconfigstlsconfig}

Description
: tlsConfig defines the TLS configuration for SMTP connections. This includes settings for certificates, CA validation, and TLS protocol options.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].emailConfigs[].tlsConfig.ca {#_specreceiversemailconfigstlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].emailConfigs[].tlsConfig.ca.configMap {#_specreceiversemailconfigstlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].emailConfigs[].tlsConfig.ca.secret {#_specreceiversemailconfigstlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].emailConfigs[].tlsConfig.cert {#_specreceiversemailconfigstlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].emailConfigs[].tlsConfig.cert.configMap {#_specreceiversemailconfigstlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].emailConfigs[].tlsConfig.cert.secret {#_specreceiversemailconfigstlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].emailConfigs[].tlsConfig.keySecret {#_specreceiversemailconfigstlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsConfigs {#_specreceiversmsteamsconfigs}

Description
: msteamsConfigs defines the list of MSTeams configurations. It requires Alertmanager >= 0.26.0.

Type
: ```
  `array`
  ```

### .spec.receivers[].msteamsConfigs[] {#_specreceiversmsteamsconfigs}

Description
: MSTeamsConfig configures notifications via Microsoft Teams. It requires Alertmanager >= 0.26.0.

Type
: ```
  `object`
  ```

Required
: - `webhookUrl`

| Property | Type | Description |
| --- | --- | --- |
| `httpConfig` | `object` | httpConfig defines the HTTP client configuration for Teams webhook requests. |
| `sendResolved` | `boolean` | sendResolved defines whether or not to notify about resolved alerts. |
| `summary` | `string` | summary defines the message summary template for Teams notifications. This provides a brief overview that appears in Teams notification previews. It requires Alertmanager >= 0.27.0. |
| `text` | `string` | text defines the message body template for Teams notifications. This contains the detailed content of the Teams message. |
| `title` | `string` | title defines the message title template for Teams notifications. This appears as the main heading of the Teams message card. |
| `webhookUrl` | `object` | webhookUrl defines the MSTeams webhook URL for sending notifications. This is the incoming webhook URL configured in your Teams channel. |

### .spec.receivers[].msteamsConfigs[].httpConfig {#_specreceiversmsteamsconfigshttpconfig}

Description
: httpConfig defines the HTTP client configuration for Teams webhook requests.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `authorization` | `object` | authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+. |
| `basicAuth` | `object` | basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence. |
| `bearerTokenSecret` | `object` | bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `enableHttp2` | `boolean` | enableHttp2 can be used to disable HTTP2. |
| `followRedirects` | `boolean` | followRedirects defines whether HTTP requests follow HTTP 3xx redirects. When true, the client will automatically follow redirect responses. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `oauth2` | `object` | oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyURL` | `string` | proxyURL defines an optional proxy URL for HTTP requests. If defined, this field takes precedence over `proxyUrl`. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options. |

### .spec.receivers[].msteamsConfigs[].httpConfig.authorization {#_specreceiversmsteamsconfigshttpconfigauthorization}

Description
: authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `credentials` | `object` | credentials defines a key of a Secret in the namespace that contains the credentials for authentication. |
| `type` | `string` | type defines the authentication type. The value is case-insensitive.<br>"Basic" is not a supported value.<br>Default: "Bearer" |

### .spec.receivers[].msteamsConfigs[].httpConfig.authorization.credentials {#_specreceiversmsteamsconfigshttpconfigauthorizationcredentials}

Description
: credentials defines a key of a Secret in the namespace that contains the credentials for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsConfigs[].httpConfig.basicAuth {#_specreceiversmsteamsconfigshttpconfigbasicauth}

Description
: basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `password` | `object` | password defines a key of a Secret containing the password for authentication. |
| `username` | `object` | username defines a key of a Secret containing the username for authentication. |

### .spec.receivers[].msteamsConfigs[].httpConfig.basicAuth.password {#_specreceiversmsteamsconfigshttpconfigbasicauthpassword}

Description
: password defines a key of a Secret containing the password for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsConfigs[].httpConfig.basicAuth.username {#_specreceiversmsteamsconfigshttpconfigbasicauthusername}

Description
: username defines a key of a Secret containing the username for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsConfigs[].httpConfig.bearerTokenSecret {#_specreceiversmsteamsconfigshttpconfigbearertokensecret}

Description
: bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].msteamsConfigs[].httpConfig.oauth2 {#_specreceiversmsteamsconfigshttpconfigoauth2}

Description
: oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests.

Type
: ```
  `object`
  ```

Required
: - `clientId`
  - `clientSecret`
  - `tokenUrl`

| Property | Type | Description |
| --- | --- | --- |
| `clientId` | `object` | clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID. |
| `clientSecret` | `object` | clientSecret defines a key of a Secret containing the OAuth2 client’s secret. |
| `endpointParams` | `object (string)` | endpointParams configures the HTTP parameters to append to the token URL. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `scopes` | `array (string)` | scopes defines the OAuth2 scopes used for the token request. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0. |
| `tokenUrl` | `string` | tokenUrl defines the URL to fetch the token from. |

### .spec.receivers[].msteamsConfigs[].httpConfig.oauth2.clientId {#_specreceiversmsteamsconfigshttpconfigoauth2clientid}

Description
: clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].msteamsConfigs[].httpConfig.oauth2.clientId.configMap {#_specreceiversmsteamsconfigshttpconfigoauth2clientidconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].msteamsConfigs[].httpConfig.oauth2.clientId.secret {#_specreceiversmsteamsconfigshttpconfigoauth2clientidsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsConfigs[].httpConfig.oauth2.clientSecret {#_specreceiversmsteamsconfigshttpconfigoauth2clientsecret}

Description
: clientSecret defines a key of a Secret containing the OAuth2 client’s secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsConfigs[].httpConfig.oauth2.proxyConnectHeader {#_specreceiversmsteamsconfigshttpconfigoauth2proxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].msteamsConfigs[].httpConfig.oauth2.proxyConnectHeader{} {#_specreceiversmsteamsconfigshttpconfigoauth2proxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].msteamsConfigs[].httpConfig.oauth2.proxyConnectHeader{}[] {#_specreceiversmsteamsconfigshttpconfigoauth2proxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsConfigs[].httpConfig.oauth2.tlsConfig {#_specreceiversmsteamsconfigshttpconfigoauth2tlsconfig}

Description
: tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].msteamsConfigs[].httpConfig.oauth2.tlsConfig.ca {#_specreceiversmsteamsconfigshttpconfigoauth2tlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].msteamsConfigs[].httpConfig.oauth2.tlsConfig.ca.configMap {#_specreceiversmsteamsconfigshttpconfigoauth2tlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].msteamsConfigs[].httpConfig.oauth2.tlsConfig.ca.secret {#_specreceiversmsteamsconfigshttpconfigoauth2tlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsConfigs[].httpConfig.oauth2.tlsConfig.cert {#_specreceiversmsteamsconfigshttpconfigoauth2tlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].msteamsConfigs[].httpConfig.oauth2.tlsConfig.cert.configMap {#_specreceiversmsteamsconfigshttpconfigoauth2tlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].msteamsConfigs[].httpConfig.oauth2.tlsConfig.cert.secret {#_specreceiversmsteamsconfigshttpconfigoauth2tlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsConfigs[].httpConfig.oauth2.tlsConfig.keySecret {#_specreceiversmsteamsconfigshttpconfigoauth2tlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsConfigs[].httpConfig.proxyConnectHeader {#_specreceiversmsteamsconfigshttpconfigproxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].msteamsConfigs[].httpConfig.proxyConnectHeader{} {#_specreceiversmsteamsconfigshttpconfigproxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].msteamsConfigs[].httpConfig.proxyConnectHeader{}[] {#_specreceiversmsteamsconfigshttpconfigproxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsConfigs[].httpConfig.tlsConfig {#_specreceiversmsteamsconfigshttpconfigtlsconfig}

Description
: tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].msteamsConfigs[].httpConfig.tlsConfig.ca {#_specreceiversmsteamsconfigshttpconfigtlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].msteamsConfigs[].httpConfig.tlsConfig.ca.configMap {#_specreceiversmsteamsconfigshttpconfigtlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].msteamsConfigs[].httpConfig.tlsConfig.ca.secret {#_specreceiversmsteamsconfigshttpconfigtlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsConfigs[].httpConfig.tlsConfig.cert {#_specreceiversmsteamsconfigshttpconfigtlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].msteamsConfigs[].httpConfig.tlsConfig.cert.configMap {#_specreceiversmsteamsconfigshttpconfigtlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].msteamsConfigs[].httpConfig.tlsConfig.cert.secret {#_specreceiversmsteamsconfigshttpconfigtlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsConfigs[].httpConfig.tlsConfig.keySecret {#_specreceiversmsteamsconfigshttpconfigtlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsConfigs[].webhookUrl {#_specreceiversmsteamsconfigswebhookurl}

Description
: webhookUrl defines the MSTeams webhook URL for sending notifications. This is the incoming webhook URL configured in your Teams channel.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsv2Configs {#_specreceiversmsteamsv2configs}

Description
: msteamsv2Configs defines the list of MSTeamsV2 configurations. It requires Alertmanager >= 0.28.0.

Type
: ```
  `array`
  ```

### .spec.receivers[].msteamsv2Configs[] {#_specreceiversmsteamsv2configs}

Description
: MSTeamsV2Config configures notifications via Microsoft Teams using the new message format with adaptive cards as required by flows. See https://prometheus.io/docs/alerting/latest/configuration/#msteamsv2_config It requires Alertmanager >= 0.28.0.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `httpConfig` | `object` | httpConfig defines the HTTP client configuration for Teams webhook requests. |
| `sendResolved` | `boolean` | sendResolved defines whether or not to notify about resolved alerts. |
| `text` | `string` | text defines the message body template for adaptive card notifications. This contains the detailed content displayed in the Teams adaptive card format. |
| `title` | `string` | title defines the message title template for adaptive card notifications. This appears as the main heading in the Teams adaptive card. |
| `webhookURL` | `object` | webhookURL defines the MSTeams incoming webhook URL for adaptive card notifications. This webhook must support the newer adaptive cards format required by Teams flows. |

### .spec.receivers[].msteamsv2Configs[].httpConfig {#_specreceiversmsteamsv2configshttpconfig}

Description
: httpConfig defines the HTTP client configuration for Teams webhook requests.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `authorization` | `object` | authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+. |
| `basicAuth` | `object` | basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence. |
| `bearerTokenSecret` | `object` | bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `enableHttp2` | `boolean` | enableHttp2 can be used to disable HTTP2. |
| `followRedirects` | `boolean` | followRedirects defines whether HTTP requests follow HTTP 3xx redirects. When true, the client will automatically follow redirect responses. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `oauth2` | `object` | oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyURL` | `string` | proxyURL defines an optional proxy URL for HTTP requests. If defined, this field takes precedence over `proxyUrl`. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options. |

### .spec.receivers[].msteamsv2Configs[].httpConfig.authorization {#_specreceiversmsteamsv2configshttpconfigauthorization}

Description
: authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `credentials` | `object` | credentials defines a key of a Secret in the namespace that contains the credentials for authentication. |
| `type` | `string` | type defines the authentication type. The value is case-insensitive.<br>"Basic" is not a supported value.<br>Default: "Bearer" |

### .spec.receivers[].msteamsv2Configs[].httpConfig.authorization.credentials {#_specreceiversmsteamsv2configshttpconfigauthorizationcredentials}

Description
: credentials defines a key of a Secret in the namespace that contains the credentials for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsv2Configs[].httpConfig.basicAuth {#_specreceiversmsteamsv2configshttpconfigbasicauth}

Description
: basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `password` | `object` | password defines a key of a Secret containing the password for authentication. |
| `username` | `object` | username defines a key of a Secret containing the username for authentication. |

### .spec.receivers[].msteamsv2Configs[].httpConfig.basicAuth.password {#_specreceiversmsteamsv2configshttpconfigbasicauthpassword}

Description
: password defines a key of a Secret containing the password for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsv2Configs[].httpConfig.basicAuth.username {#_specreceiversmsteamsv2configshttpconfigbasicauthusername}

Description
: username defines a key of a Secret containing the username for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsv2Configs[].httpConfig.bearerTokenSecret {#_specreceiversmsteamsv2configshttpconfigbearertokensecret}

Description
: bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].msteamsv2Configs[].httpConfig.oauth2 {#_specreceiversmsteamsv2configshttpconfigoauth2}

Description
: oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests.

Type
: ```
  `object`
  ```

Required
: - `clientId`
  - `clientSecret`
  - `tokenUrl`

| Property | Type | Description |
| --- | --- | --- |
| `clientId` | `object` | clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID. |
| `clientSecret` | `object` | clientSecret defines a key of a Secret containing the OAuth2 client’s secret. |
| `endpointParams` | `object (string)` | endpointParams configures the HTTP parameters to append to the token URL. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `scopes` | `array (string)` | scopes defines the OAuth2 scopes used for the token request. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0. |
| `tokenUrl` | `string` | tokenUrl defines the URL to fetch the token from. |

### .spec.receivers[].msteamsv2Configs[].httpConfig.oauth2.clientId {#_specreceiversmsteamsv2configshttpconfigoauth2clientid}

Description
: clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].msteamsv2Configs[].httpConfig.oauth2.clientId.configMap {#_specreceiversmsteamsv2configshttpconfigoauth2clientidconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].msteamsv2Configs[].httpConfig.oauth2.clientId.secret {#_specreceiversmsteamsv2configshttpconfigoauth2clientidsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsv2Configs[].httpConfig.oauth2.clientSecret {#_specreceiversmsteamsv2configshttpconfigoauth2clientsecret}

Description
: clientSecret defines a key of a Secret containing the OAuth2 client’s secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsv2Configs[].httpConfig.oauth2.proxyConnectHeader {#_specreceiversmsteamsv2configshttpconfigoauth2proxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].msteamsv2Configs[].httpConfig.oauth2.proxyConnectHeader{} {#_specreceiversmsteamsv2configshttpconfigoauth2proxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].msteamsv2Configs[].httpConfig.oauth2.proxyConnectHeader{}[] {#_specreceiversmsteamsv2configshttpconfigoauth2proxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsv2Configs[].httpConfig.oauth2.tlsConfig {#_specreceiversmsteamsv2configshttpconfigoauth2tlsconfig}

Description
: tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].msteamsv2Configs[].httpConfig.oauth2.tlsConfig.ca {#_specreceiversmsteamsv2configshttpconfigoauth2tlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].msteamsv2Configs[].httpConfig.oauth2.tlsConfig.ca.configMap {#_specreceiversmsteamsv2configshttpconfigoauth2tlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].msteamsv2Configs[].httpConfig.oauth2.tlsConfig.ca.secret {#_specreceiversmsteamsv2configshttpconfigoauth2tlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsv2Configs[].httpConfig.oauth2.tlsConfig.cert {#_specreceiversmsteamsv2configshttpconfigoauth2tlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].msteamsv2Configs[].httpConfig.oauth2.tlsConfig.cert.configMap {#_specreceiversmsteamsv2configshttpconfigoauth2tlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].msteamsv2Configs[].httpConfig.oauth2.tlsConfig.cert.secret {#_specreceiversmsteamsv2configshttpconfigoauth2tlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsv2Configs[].httpConfig.oauth2.tlsConfig.keySecret {#_specreceiversmsteamsv2configshttpconfigoauth2tlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsv2Configs[].httpConfig.proxyConnectHeader {#_specreceiversmsteamsv2configshttpconfigproxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].msteamsv2Configs[].httpConfig.proxyConnectHeader{} {#_specreceiversmsteamsv2configshttpconfigproxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].msteamsv2Configs[].httpConfig.proxyConnectHeader{}[] {#_specreceiversmsteamsv2configshttpconfigproxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsv2Configs[].httpConfig.tlsConfig {#_specreceiversmsteamsv2configshttpconfigtlsconfig}

Description
: tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].msteamsv2Configs[].httpConfig.tlsConfig.ca {#_specreceiversmsteamsv2configshttpconfigtlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].msteamsv2Configs[].httpConfig.tlsConfig.ca.configMap {#_specreceiversmsteamsv2configshttpconfigtlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].msteamsv2Configs[].httpConfig.tlsConfig.ca.secret {#_specreceiversmsteamsv2configshttpconfigtlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsv2Configs[].httpConfig.tlsConfig.cert {#_specreceiversmsteamsv2configshttpconfigtlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].msteamsv2Configs[].httpConfig.tlsConfig.cert.configMap {#_specreceiversmsteamsv2configshttpconfigtlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].msteamsv2Configs[].httpConfig.tlsConfig.cert.secret {#_specreceiversmsteamsv2configshttpconfigtlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsv2Configs[].httpConfig.tlsConfig.keySecret {#_specreceiversmsteamsv2configshttpconfigtlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].msteamsv2Configs[].webhookURL {#_specreceiversmsteamsv2configswebhookurl}

Description
: webhookURL defines the MSTeams incoming webhook URL for adaptive card notifications. This webhook must support the newer adaptive cards format required by Teams flows.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].opsgenieConfigs {#_specreceiversopsgenieconfigs}

Description
: opsgenieConfigs defines the list of OpsGenie configurations.

Type
: ```
  `array`
  ```

### .spec.receivers[].opsgenieConfigs[] {#_specreceiversopsgenieconfigs}

Description
: OpsGenieConfig configures notifications via OpsGenie. See https://prometheus.io/docs/alerting/latest/configuration/#opsgenie_config

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `actions` | `string` | actions defines a comma separated list of actions that will be available for the alert. These appear as action buttons in the OpsGenie interface. |
| `apiKey` | `object` | apiKey defines the secret’s key that contains the OpsGenie API key. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `apiURL` | `string` | apiURL defines the URL to send OpsGenie API requests to. When not specified, defaults to the standard OpsGenie API endpoint. |
| `description` | `string` | description defines the detailed description of the incident. This provides additional context beyond the message field. |
| `details` | `array` | details defines a set of arbitrary key/value pairs that provide further detail about the incident. These appear as additional fields in the OpsGenie alert. |
| `details[]` | `object` | KeyValue defines a (key, value) tuple. |
| `entity` | `string` | entity defines an optional field that can be used to specify which domain alert is related to. This helps group related alerts together in OpsGenie. |
| `httpConfig` | `object` | httpConfig defines the HTTP client configuration for OpsGenie API requests. |
| `message` | `string` | message defines the alert text limited to 130 characters. This appears as the main alert title in OpsGenie. |
| `note` | `string` | note defines an additional alert note. This provides supplementary information about the alert. |
| `priority` | `string` | priority defines the priority level of alert. Possible values are P1, P2, P3, P4, and P5, where P1 is highest priority. |
| `responders` | `array` | responders defines the list of responders responsible for notifications. These determine who gets notified when the alert is created. |
| `responders[]` | `object` | OpsGenieConfigResponder defines a responder to an incident. One of `id`, `name` or `username` has to be defined. |
| `sendResolved` | `boolean` | sendResolved defines whether or not to notify about resolved alerts. |
| `source` | `string` | source defines the backlink to the sender of the notification. This helps identify where the alert originated from. |
| `tags` | `string` | tags defines a comma separated list of tags attached to the notifications. These help categorize and filter alerts within OpsGenie. |

### .spec.receivers[].opsgenieConfigs[].apiKey {#_specreceiversopsgenieconfigsapikey}

Description
: apiKey defines the secret’s key that contains the OpsGenie API key. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].opsgenieConfigs[].details {#_specreceiversopsgenieconfigsdetails}

Description
: details defines a set of arbitrary key/value pairs that provide further detail about the incident. These appear as additional fields in the OpsGenie alert.

Type
: ```
  `array`
  ```

### .spec.receivers[].opsgenieConfigs[].details[] {#_specreceiversopsgenieconfigsdetails}

Description
: KeyValue defines a (key, value) tuple.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `value`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the tuple. This is the identifier or name part of the key-value pair. |
| `value` | `string` | value defines the value of the tuple. This is the data or content associated with the key. |

### .spec.receivers[].opsgenieConfigs[].httpConfig {#_specreceiversopsgenieconfigshttpconfig}

Description
: httpConfig defines the HTTP client configuration for OpsGenie API requests.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `authorization` | `object` | authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+. |
| `basicAuth` | `object` | basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence. |
| `bearerTokenSecret` | `object` | bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `enableHttp2` | `boolean` | enableHttp2 can be used to disable HTTP2. |
| `followRedirects` | `boolean` | followRedirects defines whether HTTP requests follow HTTP 3xx redirects. When true, the client will automatically follow redirect responses. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `oauth2` | `object` | oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyURL` | `string` | proxyURL defines an optional proxy URL for HTTP requests. If defined, this field takes precedence over `proxyUrl`. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options. |

### .spec.receivers[].opsgenieConfigs[].httpConfig.authorization {#_specreceiversopsgenieconfigshttpconfigauthorization}

Description
: authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `credentials` | `object` | credentials defines a key of a Secret in the namespace that contains the credentials for authentication. |
| `type` | `string` | type defines the authentication type. The value is case-insensitive.<br>"Basic" is not a supported value.<br>Default: "Bearer" |

### .spec.receivers[].opsgenieConfigs[].httpConfig.authorization.credentials {#_specreceiversopsgenieconfigshttpconfigauthorizationcredentials}

Description
: credentials defines a key of a Secret in the namespace that contains the credentials for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].opsgenieConfigs[].httpConfig.basicAuth {#_specreceiversopsgenieconfigshttpconfigbasicauth}

Description
: basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `password` | `object` | password defines a key of a Secret containing the password for authentication. |
| `username` | `object` | username defines a key of a Secret containing the username for authentication. |

### .spec.receivers[].opsgenieConfigs[].httpConfig.basicAuth.password {#_specreceiversopsgenieconfigshttpconfigbasicauthpassword}

Description
: password defines a key of a Secret containing the password for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].opsgenieConfigs[].httpConfig.basicAuth.username {#_specreceiversopsgenieconfigshttpconfigbasicauthusername}

Description
: username defines a key of a Secret containing the username for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].opsgenieConfigs[].httpConfig.bearerTokenSecret {#_specreceiversopsgenieconfigshttpconfigbearertokensecret}

Description
: bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].opsgenieConfigs[].httpConfig.oauth2 {#_specreceiversopsgenieconfigshttpconfigoauth2}

Description
: oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests.

Type
: ```
  `object`
  ```

Required
: - `clientId`
  - `clientSecret`
  - `tokenUrl`

| Property | Type | Description |
| --- | --- | --- |
| `clientId` | `object` | clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID. |
| `clientSecret` | `object` | clientSecret defines a key of a Secret containing the OAuth2 client’s secret. |
| `endpointParams` | `object (string)` | endpointParams configures the HTTP parameters to append to the token URL. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `scopes` | `array (string)` | scopes defines the OAuth2 scopes used for the token request. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0. |
| `tokenUrl` | `string` | tokenUrl defines the URL to fetch the token from. |

### .spec.receivers[].opsgenieConfigs[].httpConfig.oauth2.clientId {#_specreceiversopsgenieconfigshttpconfigoauth2clientid}

Description
: clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].opsgenieConfigs[].httpConfig.oauth2.clientId.configMap {#_specreceiversopsgenieconfigshttpconfigoauth2clientidconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].opsgenieConfigs[].httpConfig.oauth2.clientId.secret {#_specreceiversopsgenieconfigshttpconfigoauth2clientidsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].opsgenieConfigs[].httpConfig.oauth2.clientSecret {#_specreceiversopsgenieconfigshttpconfigoauth2clientsecret}

Description
: clientSecret defines a key of a Secret containing the OAuth2 client’s secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].opsgenieConfigs[].httpConfig.oauth2.proxyConnectHeader {#_specreceiversopsgenieconfigshttpconfigoauth2proxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].opsgenieConfigs[].httpConfig.oauth2.proxyConnectHeader{} {#_specreceiversopsgenieconfigshttpconfigoauth2proxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].opsgenieConfigs[].httpConfig.oauth2.proxyConnectHeader{}[] {#_specreceiversopsgenieconfigshttpconfigoauth2proxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].opsgenieConfigs[].httpConfig.oauth2.tlsConfig {#_specreceiversopsgenieconfigshttpconfigoauth2tlsconfig}

Description
: tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].opsgenieConfigs[].httpConfig.oauth2.tlsConfig.ca {#_specreceiversopsgenieconfigshttpconfigoauth2tlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].opsgenieConfigs[].httpConfig.oauth2.tlsConfig.ca.configMap {#_specreceiversopsgenieconfigshttpconfigoauth2tlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].opsgenieConfigs[].httpConfig.oauth2.tlsConfig.ca.secret {#_specreceiversopsgenieconfigshttpconfigoauth2tlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].opsgenieConfigs[].httpConfig.oauth2.tlsConfig.cert {#_specreceiversopsgenieconfigshttpconfigoauth2tlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].opsgenieConfigs[].httpConfig.oauth2.tlsConfig.cert.configMap {#_specreceiversopsgenieconfigshttpconfigoauth2tlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].opsgenieConfigs[].httpConfig.oauth2.tlsConfig.cert.secret {#_specreceiversopsgenieconfigshttpconfigoauth2tlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].opsgenieConfigs[].httpConfig.oauth2.tlsConfig.keySecret {#_specreceiversopsgenieconfigshttpconfigoauth2tlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].opsgenieConfigs[].httpConfig.proxyConnectHeader {#_specreceiversopsgenieconfigshttpconfigproxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].opsgenieConfigs[].httpConfig.proxyConnectHeader{} {#_specreceiversopsgenieconfigshttpconfigproxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].opsgenieConfigs[].httpConfig.proxyConnectHeader{}[] {#_specreceiversopsgenieconfigshttpconfigproxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].opsgenieConfigs[].httpConfig.tlsConfig {#_specreceiversopsgenieconfigshttpconfigtlsconfig}

Description
: tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].opsgenieConfigs[].httpConfig.tlsConfig.ca {#_specreceiversopsgenieconfigshttpconfigtlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].opsgenieConfigs[].httpConfig.tlsConfig.ca.configMap {#_specreceiversopsgenieconfigshttpconfigtlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].opsgenieConfigs[].httpConfig.tlsConfig.ca.secret {#_specreceiversopsgenieconfigshttpconfigtlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].opsgenieConfigs[].httpConfig.tlsConfig.cert {#_specreceiversopsgenieconfigshttpconfigtlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].opsgenieConfigs[].httpConfig.tlsConfig.cert.configMap {#_specreceiversopsgenieconfigshttpconfigtlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].opsgenieConfigs[].httpConfig.tlsConfig.cert.secret {#_specreceiversopsgenieconfigshttpconfigtlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].opsgenieConfigs[].httpConfig.tlsConfig.keySecret {#_specreceiversopsgenieconfigshttpconfigtlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].opsgenieConfigs[].responders {#_specreceiversopsgenieconfigsresponders}

Description
: responders defines the list of responders responsible for notifications. These determine who gets notified when the alert is created.

Type
: ```
  `array`
  ```

### .spec.receivers[].opsgenieConfigs[].responders[] {#_specreceiversopsgenieconfigsresponders}

Description
: OpsGenieConfigResponder defines a responder to an incident. One of `id`, `name` or `username` has to be defined.

Type
: ```
  `object`
  ```

Required
: - `type`

| Property | Type | Description |
| --- | --- | --- |
| `id` | `string` | id defines the unique identifier of the responder. This corresponds to the responder’s ID within OpsGenie. |
| `name` | `string` | name defines the display name of the responder. This is used when the responder is identified by name rather than ID. |
| `type` | `string` | type defines the type of responder. Valid values include "user", "team", "schedule", and "escalation". This determines how OpsGenie interprets the other identifier fields. |
| `username` | `string` | username defines the username of the responder. This is typically used for user-type responders when identifying by username. |

### .spec.receivers[].pagerdutyConfigs {#_specreceiverspagerdutyconfigs}

Description
: pagerdutyConfigs defines the List of PagerDuty configurations.

Type
: ```
  `array`
  ```

### .spec.receivers[].pagerdutyConfigs[] {#_specreceiverspagerdutyconfigs}

Description
: PagerDutyConfig configures notifications via PagerDuty. See https://prometheus.io/docs/alerting/latest/configuration/#pagerduty_config

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `class` | `string` | class defines the class/type of the event. |
| `client` | `string` | client defines the client identification. |
| `clientURL` | `string` | clientURL defines the backlink to the sender of notification. |
| `component` | `string` | component defines the part or component of the affected system that is broken. |
| `description` | `string` | description of the incident. |
| `details` | `array` | details defines the arbitrary key/value pairs that provide further detail about the incident. |
| `details[]` | `object` | KeyValue defines a (key, value) tuple. |
| `group` | `string` | group defines a cluster or grouping of sources. |
| `httpConfig` | `object` | httpConfig defines the HTTP client configuration. |
| `pagerDutyImageConfigs` | `array` | pagerDutyImageConfigs defines a list of image details to attach that provide further detail about an incident. |
| `pagerDutyImageConfigs[]` | `object` | PagerDutyImageConfig attaches images to an incident |
| `pagerDutyLinkConfigs` | `array` | pagerDutyLinkConfigs defines a list of link details to attach that provide further detail about an incident. |
| `pagerDutyLinkConfigs[]` | `object` | PagerDutyLinkConfig attaches text links to an incident |
| `routingKey` | `object` | routingKey defines the secret’s key that contains the PagerDuty integration key (when using Events API v2). Either this field or `serviceKey` needs to be defined. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `sendResolved` | `boolean` | sendResolved defines whether or not to notify about resolved alerts. |
| `serviceKey` | `object` | serviceKey defines the secret’s key that contains the PagerDuty service key (when using integration type "Prometheus"). Either this field or `routingKey` needs to be defined. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `severity` | `string` | severity of the incident. |
| `source` | `string` | source defines the unique location of the affected system. |
| `timeout` | `string` | timeout is the maximum time allowed to invoke the pagerduty It requires Alertmanager >= v0.30.0. |
| `url` | `string` | url defines the URL to send requests to. |

### .spec.receivers[].pagerdutyConfigs[].details {#_specreceiverspagerdutyconfigsdetails}

Description
: details defines the arbitrary key/value pairs that provide further detail about the incident.

Type
: ```
  `array`
  ```

### .spec.receivers[].pagerdutyConfigs[].details[] {#_specreceiverspagerdutyconfigsdetails}

Description
: KeyValue defines a (key, value) tuple.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `value`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the tuple. This is the identifier or name part of the key-value pair. |
| `value` | `string` | value defines the value of the tuple. This is the data or content associated with the key. |

### .spec.receivers[].pagerdutyConfigs[].httpConfig {#_specreceiverspagerdutyconfigshttpconfig}

Description
: httpConfig defines the HTTP client configuration.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `authorization` | `object` | authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+. |
| `basicAuth` | `object` | basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence. |
| `bearerTokenSecret` | `object` | bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `enableHttp2` | `boolean` | enableHttp2 can be used to disable HTTP2. |
| `followRedirects` | `boolean` | followRedirects defines whether HTTP requests follow HTTP 3xx redirects. When true, the client will automatically follow redirect responses. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `oauth2` | `object` | oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyURL` | `string` | proxyURL defines an optional proxy URL for HTTP requests. If defined, this field takes precedence over `proxyUrl`. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options. |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.authorization {#_specreceiverspagerdutyconfigshttpconfigauthorization}

Description
: authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `credentials` | `object` | credentials defines a key of a Secret in the namespace that contains the credentials for authentication. |
| `type` | `string` | type defines the authentication type. The value is case-insensitive.<br>"Basic" is not a supported value.<br>Default: "Bearer" |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.authorization.credentials {#_specreceiverspagerdutyconfigshttpconfigauthorizationcredentials}

Description
: credentials defines a key of a Secret in the namespace that contains the credentials for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.basicAuth {#_specreceiverspagerdutyconfigshttpconfigbasicauth}

Description
: basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `password` | `object` | password defines a key of a Secret containing the password for authentication. |
| `username` | `object` | username defines a key of a Secret containing the username for authentication. |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.basicAuth.password {#_specreceiverspagerdutyconfigshttpconfigbasicauthpassword}

Description
: password defines a key of a Secret containing the password for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.basicAuth.username {#_specreceiverspagerdutyconfigshttpconfigbasicauthusername}

Description
: username defines a key of a Secret containing the username for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.bearerTokenSecret {#_specreceiverspagerdutyconfigshttpconfigbearertokensecret}

Description
: bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.oauth2 {#_specreceiverspagerdutyconfigshttpconfigoauth2}

Description
: oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests.

Type
: ```
  `object`
  ```

Required
: - `clientId`
  - `clientSecret`
  - `tokenUrl`

| Property | Type | Description |
| --- | --- | --- |
| `clientId` | `object` | clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID. |
| `clientSecret` | `object` | clientSecret defines a key of a Secret containing the OAuth2 client’s secret. |
| `endpointParams` | `object (string)` | endpointParams configures the HTTP parameters to append to the token URL. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `scopes` | `array (string)` | scopes defines the OAuth2 scopes used for the token request. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0. |
| `tokenUrl` | `string` | tokenUrl defines the URL to fetch the token from. |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.oauth2.clientId {#_specreceiverspagerdutyconfigshttpconfigoauth2clientid}

Description
: clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.oauth2.clientId.configMap {#_specreceiverspagerdutyconfigshttpconfigoauth2clientidconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.oauth2.clientId.secret {#_specreceiverspagerdutyconfigshttpconfigoauth2clientidsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.oauth2.clientSecret {#_specreceiverspagerdutyconfigshttpconfigoauth2clientsecret}

Description
: clientSecret defines a key of a Secret containing the OAuth2 client’s secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.oauth2.proxyConnectHeader {#_specreceiverspagerdutyconfigshttpconfigoauth2proxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].pagerdutyConfigs[].httpConfig.oauth2.proxyConnectHeader{} {#_specreceiverspagerdutyconfigshttpconfigoauth2proxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].pagerdutyConfigs[].httpConfig.oauth2.proxyConnectHeader{}[] {#_specreceiverspagerdutyconfigshttpconfigoauth2proxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.oauth2.tlsConfig {#_specreceiverspagerdutyconfigshttpconfigoauth2tlsconfig}

Description
: tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.oauth2.tlsConfig.ca {#_specreceiverspagerdutyconfigshttpconfigoauth2tlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.oauth2.tlsConfig.ca.configMap {#_specreceiverspagerdutyconfigshttpconfigoauth2tlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.oauth2.tlsConfig.ca.secret {#_specreceiverspagerdutyconfigshttpconfigoauth2tlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.oauth2.tlsConfig.cert {#_specreceiverspagerdutyconfigshttpconfigoauth2tlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.oauth2.tlsConfig.cert.configMap {#_specreceiverspagerdutyconfigshttpconfigoauth2tlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.oauth2.tlsConfig.cert.secret {#_specreceiverspagerdutyconfigshttpconfigoauth2tlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.oauth2.tlsConfig.keySecret {#_specreceiverspagerdutyconfigshttpconfigoauth2tlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.proxyConnectHeader {#_specreceiverspagerdutyconfigshttpconfigproxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].pagerdutyConfigs[].httpConfig.proxyConnectHeader{} {#_specreceiverspagerdutyconfigshttpconfigproxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].pagerdutyConfigs[].httpConfig.proxyConnectHeader{}[] {#_specreceiverspagerdutyconfigshttpconfigproxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.tlsConfig {#_specreceiverspagerdutyconfigshttpconfigtlsconfig}

Description
: tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.tlsConfig.ca {#_specreceiverspagerdutyconfigshttpconfigtlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.tlsConfig.ca.configMap {#_specreceiverspagerdutyconfigshttpconfigtlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.tlsConfig.ca.secret {#_specreceiverspagerdutyconfigshttpconfigtlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.tlsConfig.cert {#_specreceiverspagerdutyconfigshttpconfigtlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.tlsConfig.cert.configMap {#_specreceiverspagerdutyconfigshttpconfigtlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.tlsConfig.cert.secret {#_specreceiverspagerdutyconfigshttpconfigtlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pagerdutyConfigs[].httpConfig.tlsConfig.keySecret {#_specreceiverspagerdutyconfigshttpconfigtlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pagerdutyConfigs[].pagerDutyImageConfigs {#_specreceiverspagerdutyconfigspagerdutyimageconfigs}

Description
: pagerDutyImageConfigs defines a list of image details to attach that provide further detail about an incident.

Type
: ```
  `array`
  ```

### .spec.receivers[].pagerdutyConfigs[].pagerDutyImageConfigs[] {#_specreceiverspagerdutyconfigspagerdutyimageconfigs}

Description
: PagerDutyImageConfig attaches images to an incident

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `alt` | `string` | alt is the optional alternative text for the image. |
| `href` | `string` | href defines the optional URL; makes the image a clickable link. |
| `src` | `string` | src of the image being attached to the incident |

### .spec.receivers[].pagerdutyConfigs[].pagerDutyLinkConfigs {#_specreceiverspagerdutyconfigspagerdutylinkconfigs}

Description
: pagerDutyLinkConfigs defines a list of link details to attach that provide further detail about an incident.

Type
: ```
  `array`
  ```

### .spec.receivers[].pagerdutyConfigs[].pagerDutyLinkConfigs[] {#_specreceiverspagerdutyconfigspagerdutylinkconfigs}

Description
: PagerDutyLinkConfig attaches text links to an incident

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `alt` | `string` | alt defines the text that describes the purpose of the link, and can be used as the link’s text. |
| `href` | `string` | href defines the URL of the link to be attached |

### .spec.receivers[].pagerdutyConfigs[].routingKey {#_specreceiverspagerdutyconfigsroutingkey}

Description
: routingKey defines the secret’s key that contains the PagerDuty integration key (when using Events API v2). Either this field or `serviceKey` needs to be defined. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].pagerdutyConfigs[].serviceKey {#_specreceiverspagerdutyconfigsservicekey}

Description
: serviceKey defines the secret’s key that contains the PagerDuty service key (when using integration type "Prometheus"). Either this field or `routingKey` needs to be defined. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].pushoverConfigs {#_specreceiverspushoverconfigs}

Description
: pushoverConfigs defines the list of Pushover configurations.

Type
: ```
  `array`
  ```

### .spec.receivers[].pushoverConfigs[] {#_specreceiverspushoverconfigs}

Description
: PushoverConfig configures notifications via Pushover. See https://prometheus.io/docs/alerting/latest/configuration/#pushover_config

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `device` | `string` | device defines the name of a specific device to send the notification to. If not specified, the notification is sent to all user’s devices. |
| `expire` | `string` | expire defines how long your notification will continue to be retried for, unless the user acknowledges the notification. Only applies to priority 2 notifications. |
| `html` | `boolean` | html defines whether notification message is HTML or plain text. When true, the message can include HTML formatting tags. html and monospace formatting are mutually exclusive. |
| `httpConfig` | `object` | httpConfig defines the HTTP client configuration for Pushover API requests. |
| `message` | `string` | message defines the notification message content. This is the main body text of the Pushover notification. |
| `monospace` | `boolean` | monospace optional HTML/monospace formatting for the message, see https://pushover.net/api#html html and monospace formatting are mutually exclusive. |
| `priority` | `string` | priority defines the notification priority level. See https://pushover.net/api#priority for valid values and behavior. |
| `retry` | `string` | retry defines how often the Pushover servers will send the same notification to the user. Must be at least 30 seconds. Only applies to priority 2 notifications. |
| `sendResolved` | `boolean` | sendResolved defines whether or not to notify about resolved alerts. |
| `sound` | `string` | sound defines the name of one of the sounds supported by device clients. This overrides the user’s default sound choice for this notification. |
| `title` | `string` | title defines the notification title displayed in the Pushover message. This appears as the bold header text in the notification. |
| `token` | `object` | token defines the secret’s key that contains the registered application’s API token. See https://pushover.net/apps for application registration. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. Either `token` or `tokenFile` is required. |
| `tokenFile` | `string` | tokenFile defines the token file that contains the registered application’s API token. See https://pushover.net/apps for application registration. Either `token` or `tokenFile` is required. It requires Alertmanager >= v0.26.0. |
| `ttl` | `string` | ttl defines the time to live for the alert notification. This determines how long the notification remains active before expiring. |
| `url` | `string` | url defines a supplementary URL shown alongside the message. This creates a clickable link within the Pushover notification. |
| `urlTitle` | `string` | urlTitle defines a title for the supplementary URL. If not specified, the raw URL is shown instead. |
| `userKey` | `object` | userKey defines the secret’s key that contains the recipient user’s user key. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. Either `userKey` or `userKeyFile` is required. |
| `userKeyFile` | `string` | userKeyFile defines the user key file that contains the recipient user’s user key. Either `userKey` or `userKeyFile` is required. It requires Alertmanager >= v0.26.0. |

### .spec.receivers[].pushoverConfigs[].httpConfig {#_specreceiverspushoverconfigshttpconfig}

Description
: httpConfig defines the HTTP client configuration for Pushover API requests.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `authorization` | `object` | authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+. |
| `basicAuth` | `object` | basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence. |
| `bearerTokenSecret` | `object` | bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `enableHttp2` | `boolean` | enableHttp2 can be used to disable HTTP2. |
| `followRedirects` | `boolean` | followRedirects defines whether HTTP requests follow HTTP 3xx redirects. When true, the client will automatically follow redirect responses. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `oauth2` | `object` | oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyURL` | `string` | proxyURL defines an optional proxy URL for HTTP requests. If defined, this field takes precedence over `proxyUrl`. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options. |

### .spec.receivers[].pushoverConfigs[].httpConfig.authorization {#_specreceiverspushoverconfigshttpconfigauthorization}

Description
: authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `credentials` | `object` | credentials defines a key of a Secret in the namespace that contains the credentials for authentication. |
| `type` | `string` | type defines the authentication type. The value is case-insensitive.<br>"Basic" is not a supported value.<br>Default: "Bearer" |

### .spec.receivers[].pushoverConfigs[].httpConfig.authorization.credentials {#_specreceiverspushoverconfigshttpconfigauthorizationcredentials}

Description
: credentials defines a key of a Secret in the namespace that contains the credentials for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pushoverConfigs[].httpConfig.basicAuth {#_specreceiverspushoverconfigshttpconfigbasicauth}

Description
: basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `password` | `object` | password defines a key of a Secret containing the password for authentication. |
| `username` | `object` | username defines a key of a Secret containing the username for authentication. |

### .spec.receivers[].pushoverConfigs[].httpConfig.basicAuth.password {#_specreceiverspushoverconfigshttpconfigbasicauthpassword}

Description
: password defines a key of a Secret containing the password for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pushoverConfigs[].httpConfig.basicAuth.username {#_specreceiverspushoverconfigshttpconfigbasicauthusername}

Description
: username defines a key of a Secret containing the username for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pushoverConfigs[].httpConfig.bearerTokenSecret {#_specreceiverspushoverconfigshttpconfigbearertokensecret}

Description
: bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].pushoverConfigs[].httpConfig.oauth2 {#_specreceiverspushoverconfigshttpconfigoauth2}

Description
: oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests.

Type
: ```
  `object`
  ```

Required
: - `clientId`
  - `clientSecret`
  - `tokenUrl`

| Property | Type | Description |
| --- | --- | --- |
| `clientId` | `object` | clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID. |
| `clientSecret` | `object` | clientSecret defines a key of a Secret containing the OAuth2 client’s secret. |
| `endpointParams` | `object (string)` | endpointParams configures the HTTP parameters to append to the token URL. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `scopes` | `array (string)` | scopes defines the OAuth2 scopes used for the token request. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0. |
| `tokenUrl` | `string` | tokenUrl defines the URL to fetch the token from. |

### .spec.receivers[].pushoverConfigs[].httpConfig.oauth2.clientId {#_specreceiverspushoverconfigshttpconfigoauth2clientid}

Description
: clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].pushoverConfigs[].httpConfig.oauth2.clientId.configMap {#_specreceiverspushoverconfigshttpconfigoauth2clientidconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].pushoverConfigs[].httpConfig.oauth2.clientId.secret {#_specreceiverspushoverconfigshttpconfigoauth2clientidsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pushoverConfigs[].httpConfig.oauth2.clientSecret {#_specreceiverspushoverconfigshttpconfigoauth2clientsecret}

Description
: clientSecret defines a key of a Secret containing the OAuth2 client’s secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pushoverConfigs[].httpConfig.oauth2.proxyConnectHeader {#_specreceiverspushoverconfigshttpconfigoauth2proxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].pushoverConfigs[].httpConfig.oauth2.proxyConnectHeader{} {#_specreceiverspushoverconfigshttpconfigoauth2proxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].pushoverConfigs[].httpConfig.oauth2.proxyConnectHeader{}[] {#_specreceiverspushoverconfigshttpconfigoauth2proxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pushoverConfigs[].httpConfig.oauth2.tlsConfig {#_specreceiverspushoverconfigshttpconfigoauth2tlsconfig}

Description
: tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].pushoverConfigs[].httpConfig.oauth2.tlsConfig.ca {#_specreceiverspushoverconfigshttpconfigoauth2tlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].pushoverConfigs[].httpConfig.oauth2.tlsConfig.ca.configMap {#_specreceiverspushoverconfigshttpconfigoauth2tlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].pushoverConfigs[].httpConfig.oauth2.tlsConfig.ca.secret {#_specreceiverspushoverconfigshttpconfigoauth2tlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pushoverConfigs[].httpConfig.oauth2.tlsConfig.cert {#_specreceiverspushoverconfigshttpconfigoauth2tlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].pushoverConfigs[].httpConfig.oauth2.tlsConfig.cert.configMap {#_specreceiverspushoverconfigshttpconfigoauth2tlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].pushoverConfigs[].httpConfig.oauth2.tlsConfig.cert.secret {#_specreceiverspushoverconfigshttpconfigoauth2tlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pushoverConfigs[].httpConfig.oauth2.tlsConfig.keySecret {#_specreceiverspushoverconfigshttpconfigoauth2tlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pushoverConfigs[].httpConfig.proxyConnectHeader {#_specreceiverspushoverconfigshttpconfigproxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].pushoverConfigs[].httpConfig.proxyConnectHeader{} {#_specreceiverspushoverconfigshttpconfigproxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].pushoverConfigs[].httpConfig.proxyConnectHeader{}[] {#_specreceiverspushoverconfigshttpconfigproxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pushoverConfigs[].httpConfig.tlsConfig {#_specreceiverspushoverconfigshttpconfigtlsconfig}

Description
: tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].pushoverConfigs[].httpConfig.tlsConfig.ca {#_specreceiverspushoverconfigshttpconfigtlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].pushoverConfigs[].httpConfig.tlsConfig.ca.configMap {#_specreceiverspushoverconfigshttpconfigtlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].pushoverConfigs[].httpConfig.tlsConfig.ca.secret {#_specreceiverspushoverconfigshttpconfigtlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pushoverConfigs[].httpConfig.tlsConfig.cert {#_specreceiverspushoverconfigshttpconfigtlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].pushoverConfigs[].httpConfig.tlsConfig.cert.configMap {#_specreceiverspushoverconfigshttpconfigtlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].pushoverConfigs[].httpConfig.tlsConfig.cert.secret {#_specreceiverspushoverconfigshttpconfigtlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pushoverConfigs[].httpConfig.tlsConfig.keySecret {#_specreceiverspushoverconfigshttpconfigtlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].pushoverConfigs[].token {#_specreceiverspushoverconfigstoken}

Description
: token defines the secret’s key that contains the registered application’s API token. See https://pushover.net/apps for application registration. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. Either `token` or `tokenFile` is required.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].pushoverConfigs[].userKey {#_specreceiverspushoverconfigsuserkey}

Description
: userKey defines the secret’s key that contains the recipient user’s user key. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. Either `userKey` or `userKeyFile` is required.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].rocketchatConfigs {#_specreceiversrocketchatconfigs}

Description
: rocketchatConfigs defines the list of RocketChat configurations. It requires Alertmanager >= 0.28.0.

Type
: ```
  `array`
  ```

### .spec.receivers[].rocketchatConfigs[] {#_specreceiversrocketchatconfigs}

Description
: RocketChatConfig configures notifications via RocketChat. It requires Alertmanager >= 0.28.0.

Type
: ```
  `object`
  ```

Required
: - `token`
  - `tokenID`

| Property | Type | Description |
| --- | --- | --- |
| `actions` | `array` | actions defines interactive actions to include in the message. These appear as buttons that users can click to trigger responses. |
| `actions[]` | `object` | RocketChatActionConfig defines actions for RocketChat messages. |
| `apiURL` | `string` | apiURL defines the API URL for RocketChat. Defaults to https://open.rocket.chat/ if not specified. |
| `channel` | `string` | channel defines the channel to send alerts to. This can be a channel name (e.g., "#alerts") or a direct message recipient. |
| `color` | `string` | color defines the message color displayed in RocketChat. This appears as a colored bar alongside the message. |
| `emoji` | `string` | emoji defines the emoji to be displayed as an avatar. If provided, this emoji will be used instead of the default avatar or iconURL. |
| `fields` | `array` | fields defines additional fields for the message attachment. These appear as structured key-value pairs within the message. |
| `fields[]` | `object` | RocketChatFieldConfig defines a field for RocketChat messages. |
| `httpConfig` | `object` | httpConfig defines the HTTP client configuration for RocketChat API requests. |
| `iconURL` | `string` | iconURL defines the icon URL for the message avatar. This displays a custom image as the message sender’s avatar. |
| `imageURL` | `string` | imageURL defines the image URL to display within the message. This embeds an image directly in the message attachment. |
| `linkNames` | `boolean` | linkNames defines whether to enable automatic linking of usernames and channels. When true, @username and #channel references become clickable links. |
| `sendResolved` | `boolean` | sendResolved defines whether or not to notify about resolved alerts. |
| `shortFields` | `boolean` | shortFields defines whether to use short fields in the message layout. When true, fields may be displayed side by side to save space. |
| `text` | `string` | text defines the message text to send. This is optional because attachments can be used instead of or alongside text. |
| `thumbURL` | `string` | thumbURL defines the thumbnail URL for the message. This displays a small thumbnail image alongside the message content. |
| `title` | `string` | title defines the message title displayed prominently in the message. This appears as bold text at the top of the message attachment. |
| `titleLink` | `string` | titleLink defines the URL that the title will link to when clicked. This makes the message title clickable in the RocketChat interface. |
| `token` | `object` | token defines the sender token for RocketChat authentication. This is the personal access token or bot token used to authenticate API requests. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `tokenID` | `object` | tokenID defines the sender token ID for RocketChat authentication. This is the user ID associated with the token used for API requests. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |

### .spec.receivers[].rocketchatConfigs[].actions {#_specreceiversrocketchatconfigsactions}

Description
: actions defines interactive actions to include in the message. These appear as buttons that users can click to trigger responses.

Type
: ```
  `array`
  ```

### .spec.receivers[].rocketchatConfigs[].actions[] {#_specreceiversrocketchatconfigsactions}

Description
: RocketChatActionConfig defines actions for RocketChat messages.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `msg` | `string` | msg defines the message to send when the button is clicked. This allows the button to post a predefined message to the channel. |
| `text` | `string` | text defines the button text displayed to users. This is the label that appears on the interactive button. |
| `url` | `string` | url defines the URL the button links to when clicked. This creates a clickable button that opens the specified URL. |

### .spec.receivers[].rocketchatConfigs[].fields {#_specreceiversrocketchatconfigsfields}

Description
: fields defines additional fields for the message attachment. These appear as structured key-value pairs within the message.

Type
: ```
  `array`
  ```

### .spec.receivers[].rocketchatConfigs[].fields[] {#_specreceiversrocketchatconfigsfields}

Description
: RocketChatFieldConfig defines a field for RocketChat messages.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `short` | `boolean` | short defines whether this field should be a short field. When true, the field may be displayed inline with other short fields to save space. |
| `title` | `string` | title defines the title of this field. This appears as bold text labeling the field content. |
| `value` | `string` | value defines the value of this field, displayed underneath the title. This contains the actual data or content for the field. |

### .spec.receivers[].rocketchatConfigs[].httpConfig {#_specreceiversrocketchatconfigshttpconfig}

Description
: httpConfig defines the HTTP client configuration for RocketChat API requests.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `authorization` | `object` | authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+. |
| `basicAuth` | `object` | basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence. |
| `bearerTokenSecret` | `object` | bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `enableHttp2` | `boolean` | enableHttp2 can be used to disable HTTP2. |
| `followRedirects` | `boolean` | followRedirects defines whether HTTP requests follow HTTP 3xx redirects. When true, the client will automatically follow redirect responses. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `oauth2` | `object` | oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyURL` | `string` | proxyURL defines an optional proxy URL for HTTP requests. If defined, this field takes precedence over `proxyUrl`. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options. |

### .spec.receivers[].rocketchatConfigs[].httpConfig.authorization {#_specreceiversrocketchatconfigshttpconfigauthorization}

Description
: authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `credentials` | `object` | credentials defines a key of a Secret in the namespace that contains the credentials for authentication. |
| `type` | `string` | type defines the authentication type. The value is case-insensitive.<br>"Basic" is not a supported value.<br>Default: "Bearer" |

### .spec.receivers[].rocketchatConfigs[].httpConfig.authorization.credentials {#_specreceiversrocketchatconfigshttpconfigauthorizationcredentials}

Description
: credentials defines a key of a Secret in the namespace that contains the credentials for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].rocketchatConfigs[].httpConfig.basicAuth {#_specreceiversrocketchatconfigshttpconfigbasicauth}

Description
: basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `password` | `object` | password defines a key of a Secret containing the password for authentication. |
| `username` | `object` | username defines a key of a Secret containing the username for authentication. |

### .spec.receivers[].rocketchatConfigs[].httpConfig.basicAuth.password {#_specreceiversrocketchatconfigshttpconfigbasicauthpassword}

Description
: password defines a key of a Secret containing the password for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].rocketchatConfigs[].httpConfig.basicAuth.username {#_specreceiversrocketchatconfigshttpconfigbasicauthusername}

Description
: username defines a key of a Secret containing the username for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].rocketchatConfigs[].httpConfig.bearerTokenSecret {#_specreceiversrocketchatconfigshttpconfigbearertokensecret}

Description
: bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].rocketchatConfigs[].httpConfig.oauth2 {#_specreceiversrocketchatconfigshttpconfigoauth2}

Description
: oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests.

Type
: ```
  `object`
  ```

Required
: - `clientId`
  - `clientSecret`
  - `tokenUrl`

| Property | Type | Description |
| --- | --- | --- |
| `clientId` | `object` | clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID. |
| `clientSecret` | `object` | clientSecret defines a key of a Secret containing the OAuth2 client’s secret. |
| `endpointParams` | `object (string)` | endpointParams configures the HTTP parameters to append to the token URL. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `scopes` | `array (string)` | scopes defines the OAuth2 scopes used for the token request. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0. |
| `tokenUrl` | `string` | tokenUrl defines the URL to fetch the token from. |

### .spec.receivers[].rocketchatConfigs[].httpConfig.oauth2.clientId {#_specreceiversrocketchatconfigshttpconfigoauth2clientid}

Description
: clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].rocketchatConfigs[].httpConfig.oauth2.clientId.configMap {#_specreceiversrocketchatconfigshttpconfigoauth2clientidconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].rocketchatConfigs[].httpConfig.oauth2.clientId.secret {#_specreceiversrocketchatconfigshttpconfigoauth2clientidsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].rocketchatConfigs[].httpConfig.oauth2.clientSecret {#_specreceiversrocketchatconfigshttpconfigoauth2clientsecret}

Description
: clientSecret defines a key of a Secret containing the OAuth2 client’s secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].rocketchatConfigs[].httpConfig.oauth2.proxyConnectHeader {#_specreceiversrocketchatconfigshttpconfigoauth2proxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].rocketchatConfigs[].httpConfig.oauth2.proxyConnectHeader{} {#_specreceiversrocketchatconfigshttpconfigoauth2proxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].rocketchatConfigs[].httpConfig.oauth2.proxyConnectHeader{}[] {#_specreceiversrocketchatconfigshttpconfigoauth2proxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].rocketchatConfigs[].httpConfig.oauth2.tlsConfig {#_specreceiversrocketchatconfigshttpconfigoauth2tlsconfig}

Description
: tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].rocketchatConfigs[].httpConfig.oauth2.tlsConfig.ca {#_specreceiversrocketchatconfigshttpconfigoauth2tlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].rocketchatConfigs[].httpConfig.oauth2.tlsConfig.ca.configMap {#_specreceiversrocketchatconfigshttpconfigoauth2tlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].rocketchatConfigs[].httpConfig.oauth2.tlsConfig.ca.secret {#_specreceiversrocketchatconfigshttpconfigoauth2tlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].rocketchatConfigs[].httpConfig.oauth2.tlsConfig.cert {#_specreceiversrocketchatconfigshttpconfigoauth2tlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].rocketchatConfigs[].httpConfig.oauth2.tlsConfig.cert.configMap {#_specreceiversrocketchatconfigshttpconfigoauth2tlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].rocketchatConfigs[].httpConfig.oauth2.tlsConfig.cert.secret {#_specreceiversrocketchatconfigshttpconfigoauth2tlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].rocketchatConfigs[].httpConfig.oauth2.tlsConfig.keySecret {#_specreceiversrocketchatconfigshttpconfigoauth2tlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].rocketchatConfigs[].httpConfig.proxyConnectHeader {#_specreceiversrocketchatconfigshttpconfigproxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].rocketchatConfigs[].httpConfig.proxyConnectHeader{} {#_specreceiversrocketchatconfigshttpconfigproxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].rocketchatConfigs[].httpConfig.proxyConnectHeader{}[] {#_specreceiversrocketchatconfigshttpconfigproxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].rocketchatConfigs[].httpConfig.tlsConfig {#_specreceiversrocketchatconfigshttpconfigtlsconfig}

Description
: tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].rocketchatConfigs[].httpConfig.tlsConfig.ca {#_specreceiversrocketchatconfigshttpconfigtlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].rocketchatConfigs[].httpConfig.tlsConfig.ca.configMap {#_specreceiversrocketchatconfigshttpconfigtlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].rocketchatConfigs[].httpConfig.tlsConfig.ca.secret {#_specreceiversrocketchatconfigshttpconfigtlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].rocketchatConfigs[].httpConfig.tlsConfig.cert {#_specreceiversrocketchatconfigshttpconfigtlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].rocketchatConfigs[].httpConfig.tlsConfig.cert.configMap {#_specreceiversrocketchatconfigshttpconfigtlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].rocketchatConfigs[].httpConfig.tlsConfig.cert.secret {#_specreceiversrocketchatconfigshttpconfigtlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].rocketchatConfigs[].httpConfig.tlsConfig.keySecret {#_specreceiversrocketchatconfigshttpconfigtlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].rocketchatConfigs[].token {#_specreceiversrocketchatconfigstoken}

Description
: token defines the sender token for RocketChat authentication. This is the personal access token or bot token used to authenticate API requests. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].rocketchatConfigs[].tokenID {#_specreceiversrocketchatconfigstokenid}

Description
: tokenID defines the sender token ID for RocketChat authentication. This is the user ID associated with the token used for API requests. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].slackConfigs {#_specreceiversslackconfigs}

Description
: slackConfigs defines the list of Slack configurations.

Type
: ```
  `array`
  ```

### .spec.receivers[].slackConfigs[] {#_specreceiversslackconfigs}

Description
: SlackConfig configures notifications via Slack. See https://prometheus.io/docs/alerting/latest/configuration/#slack_config

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `actions` | `array` | actions defines a list of Slack actions that are sent with each notification. |
| `actions[]` | `object` | SlackAction configures a single Slack action that is sent with each notification. See https://api.slack.com/docs/message-attachments#action_fields and https://api.slack.com/docs/message-buttons for more information. |
| `apiURL` | `object` | apiURL defines the secret’s key that contains the Slack webhook URL. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `callbackId` | `string` | callbackId defines an identifier for the message used in interactive components. |
| `channel` | `string` | channel defines the channel or user to send notifications to. |
| `color` | `string` | color defines the color of the left border of the Slack message attachment. Can be a hex color code (e.g., "#ff0000") or a predefined color name. |
| `fallback` | `string` | fallback defines a plain-text summary of the attachment for clients that don’t support attachments. |
| `fields` | `array` | fields defines a list of Slack fields that are sent with each notification. |
| `fields[]` | `object` | SlackField configures a single Slack field that is sent with each notification. Each field must contain a title, value, and optionally, a boolean value to indicate if the field is short enough to be displayed next to other fields designated as short. See https://api.slack.com/docs/message-attachments#fields for more information. |
| `footer` | `string` | footer defines small text displayed at the bottom of the message attachment. |
| `httpConfig` | `object` | httpConfig defines the HTTP client configuration. |
| `iconEmoji` | `string` | iconEmoji defines the emoji to use as the bot’s avatar (e.g., ":ghost:"). |
| `iconURL` | `string` | iconURL defines the URL to an image to use as the bot’s avatar. |
| `imageURL` | `string` | imageURL defines the URL to an image file that will be displayed inside the message attachment. |
| `linkNames` | `boolean` | linkNames enables automatic linking of channel names and usernames in the message. When true, @channel and @username will be converted to clickable links. |
| `messageText` | `string` | messageText defines text content of the Slack message. If set, this is sent as the top-level 'text' field in the Slack payload. It requires Alertmanager >= v0.31.0. |
| `mrkdwnIn` | `array (string)` | mrkdwnIn defines which fields should be parsed as Slack markdown. Valid values include "pretext", "text", and "fields". |
| `pretext` | `string` | pretext defines optional text that appears above the message attachment block. |
| `sendResolved` | `boolean` | sendResolved defines whether or not to notify about resolved alerts. |
| `shortFields` | `boolean` | shortFields determines whether fields are displayed in a compact format. When true, fields are shown side by side when possible. |
| `text` | `string` | text defines the main text content of the Slack message attachment. |
| `thumbURL` | `string` | thumbURL defines the URL to an image file that will be displayed as a thumbnail on the right side of the message attachment. |
| `timeout` | `string` | timeout defines the maximum time to wait for a webhook request to complete, before failing the request and allowing it to be retried. It requires Alertmanager >= v0.30.0. |
| `title` | `string` | title defines the title text displayed in the Slack message attachment. |
| `titleLink` | `string` | titleLink defines the URL that the title will link to when clicked. |
| `username` | `string` | username defines the slack bot user name. |

### .spec.receivers[].slackConfigs[].actions {#_specreceiversslackconfigsactions}

Description
: actions defines a list of Slack actions that are sent with each notification.

Type
: ```
  `array`
  ```

### .spec.receivers[].slackConfigs[].actions[] {#_specreceiversslackconfigsactions}

Description
: SlackAction configures a single Slack action that is sent with each notification. See https://api.slack.com/docs/message-attachments#action_fields and https://api.slack.com/docs/message-buttons for more information.

Type
: ```
  `object`
  ```

Required
: - `text`
  - `type`

| Property | Type | Description |
| --- | --- | --- |
| `confirm` | `object` | confirm defines an optional confirmation dialog that appears before the action is executed. When set, users must confirm their intent before the action proceeds. |
| `name` | `string` | name defines a unique identifier for the action within the message. This value is sent back to your application when the action is triggered. |
| `style` | `string` | style defines the visual appearance of the action element. Valid values include "default", "primary" (green), and "danger" (red). |
| `text` | `string` | text defines the user-visible label displayed on the action element. For buttons, this is the button text. For select menus, this is the placeholder text. |
| `type` | `string` | type defines the type of interactive component. Common values include "button" for clickable buttons and "select" for dropdown menus. |
| `url` | `string` | url defines the URL to open when the action is triggered. Only applicable for button-type actions. When set, clicking the button opens this URL. |
| `value` | `string` | value defines the payload sent when the action is triggered. This data is included in the callback sent to your application. |

### .spec.receivers[].slackConfigs[].actions[].confirm {#_specreceiversslackconfigsactionsconfirm}

Description
: confirm defines an optional confirmation dialog that appears before the action is executed. When set, users must confirm their intent before the action proceeds.

Type
: ```
  `object`
  ```

Required
: - `text`

| Property | Type | Description |
| --- | --- | --- |
| `dismissText` | `string` | dismissText defines the label for the cancel button in the dialog. When not specified, defaults to "Cancel". This button cancels the action. |
| `okText` | `string` | okText defines the label for the confirmation button in the dialog. When not specified, defaults to "Okay". This button proceeds with the action. |
| `text` | `string` | text defines the main message displayed in the confirmation dialog. This should be a clear question or statement asking the user to confirm their action. |
| `title` | `string` | title defines the title text displayed at the top of the confirmation dialog. When not specified, a default title will be used. |

### .spec.receivers[].slackConfigs[].apiURL {#_specreceiversslackconfigsapiurl}

Description
: apiURL defines the secret’s key that contains the Slack webhook URL. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].slackConfigs[].fields {#_specreceiversslackconfigsfields}

Description
: fields defines a list of Slack fields that are sent with each notification.

Type
: ```
  `array`
  ```

### .spec.receivers[].slackConfigs[].fields[] {#_specreceiversslackconfigsfields}

Description
: SlackField configures a single Slack field that is sent with each notification. Each field must contain a title, value, and optionally, a boolean value to indicate if the field is short enough to be displayed next to other fields designated as short. See https://api.slack.com/docs/message-attachments#fields for more information.

Type
: ```
  `object`
  ```

Required
: - `title`
  - `value`

| Property | Type | Description |
| --- | --- | --- |
| `short` | `boolean` | short determines whether this field can be displayed alongside other short fields. When true, Slack may display this field side by side with other short fields. When false or not specified, the field takes the full width of the message. |
| `title` | `string` | title defines the label or header text displayed for this field. This appears as bold text above the field value in the Slack message. |
| `value` | `string` | value defines the content or data displayed for this field. This appears below the title and can contain plain text or Slack markdown. |

### .spec.receivers[].slackConfigs[].httpConfig {#_specreceiversslackconfigshttpconfig}

Description
: httpConfig defines the HTTP client configuration.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `authorization` | `object` | authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+. |
| `basicAuth` | `object` | basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence. |
| `bearerTokenSecret` | `object` | bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `enableHttp2` | `boolean` | enableHttp2 can be used to disable HTTP2. |
| `followRedirects` | `boolean` | followRedirects defines whether HTTP requests follow HTTP 3xx redirects. When true, the client will automatically follow redirect responses. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `oauth2` | `object` | oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyURL` | `string` | proxyURL defines an optional proxy URL for HTTP requests. If defined, this field takes precedence over `proxyUrl`. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options. |

### .spec.receivers[].slackConfigs[].httpConfig.authorization {#_specreceiversslackconfigshttpconfigauthorization}

Description
: authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `credentials` | `object` | credentials defines a key of a Secret in the namespace that contains the credentials for authentication. |
| `type` | `string` | type defines the authentication type. The value is case-insensitive.<br>"Basic" is not a supported value.<br>Default: "Bearer" |

### .spec.receivers[].slackConfigs[].httpConfig.authorization.credentials {#_specreceiversslackconfigshttpconfigauthorizationcredentials}

Description
: credentials defines a key of a Secret in the namespace that contains the credentials for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].slackConfigs[].httpConfig.basicAuth {#_specreceiversslackconfigshttpconfigbasicauth}

Description
: basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `password` | `object` | password defines a key of a Secret containing the password for authentication. |
| `username` | `object` | username defines a key of a Secret containing the username for authentication. |

### .spec.receivers[].slackConfigs[].httpConfig.basicAuth.password {#_specreceiversslackconfigshttpconfigbasicauthpassword}

Description
: password defines a key of a Secret containing the password for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].slackConfigs[].httpConfig.basicAuth.username {#_specreceiversslackconfigshttpconfigbasicauthusername}

Description
: username defines a key of a Secret containing the username for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].slackConfigs[].httpConfig.bearerTokenSecret {#_specreceiversslackconfigshttpconfigbearertokensecret}

Description
: bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].slackConfigs[].httpConfig.oauth2 {#_specreceiversslackconfigshttpconfigoauth2}

Description
: oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests.

Type
: ```
  `object`
  ```

Required
: - `clientId`
  - `clientSecret`
  - `tokenUrl`

| Property | Type | Description |
| --- | --- | --- |
| `clientId` | `object` | clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID. |
| `clientSecret` | `object` | clientSecret defines a key of a Secret containing the OAuth2 client’s secret. |
| `endpointParams` | `object (string)` | endpointParams configures the HTTP parameters to append to the token URL. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `scopes` | `array (string)` | scopes defines the OAuth2 scopes used for the token request. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0. |
| `tokenUrl` | `string` | tokenUrl defines the URL to fetch the token from. |

### .spec.receivers[].slackConfigs[].httpConfig.oauth2.clientId {#_specreceiversslackconfigshttpconfigoauth2clientid}

Description
: clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].slackConfigs[].httpConfig.oauth2.clientId.configMap {#_specreceiversslackconfigshttpconfigoauth2clientidconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].slackConfigs[].httpConfig.oauth2.clientId.secret {#_specreceiversslackconfigshttpconfigoauth2clientidsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].slackConfigs[].httpConfig.oauth2.clientSecret {#_specreceiversslackconfigshttpconfigoauth2clientsecret}

Description
: clientSecret defines a key of a Secret containing the OAuth2 client’s secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].slackConfigs[].httpConfig.oauth2.proxyConnectHeader {#_specreceiversslackconfigshttpconfigoauth2proxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].slackConfigs[].httpConfig.oauth2.proxyConnectHeader{} {#_specreceiversslackconfigshttpconfigoauth2proxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].slackConfigs[].httpConfig.oauth2.proxyConnectHeader{}[] {#_specreceiversslackconfigshttpconfigoauth2proxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].slackConfigs[].httpConfig.oauth2.tlsConfig {#_specreceiversslackconfigshttpconfigoauth2tlsconfig}

Description
: tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].slackConfigs[].httpConfig.oauth2.tlsConfig.ca {#_specreceiversslackconfigshttpconfigoauth2tlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].slackConfigs[].httpConfig.oauth2.tlsConfig.ca.configMap {#_specreceiversslackconfigshttpconfigoauth2tlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].slackConfigs[].httpConfig.oauth2.tlsConfig.ca.secret {#_specreceiversslackconfigshttpconfigoauth2tlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].slackConfigs[].httpConfig.oauth2.tlsConfig.cert {#_specreceiversslackconfigshttpconfigoauth2tlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].slackConfigs[].httpConfig.oauth2.tlsConfig.cert.configMap {#_specreceiversslackconfigshttpconfigoauth2tlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].slackConfigs[].httpConfig.oauth2.tlsConfig.cert.secret {#_specreceiversslackconfigshttpconfigoauth2tlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].slackConfigs[].httpConfig.oauth2.tlsConfig.keySecret {#_specreceiversslackconfigshttpconfigoauth2tlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].slackConfigs[].httpConfig.proxyConnectHeader {#_specreceiversslackconfigshttpconfigproxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].slackConfigs[].httpConfig.proxyConnectHeader{} {#_specreceiversslackconfigshttpconfigproxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].slackConfigs[].httpConfig.proxyConnectHeader{}[] {#_specreceiversslackconfigshttpconfigproxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].slackConfigs[].httpConfig.tlsConfig {#_specreceiversslackconfigshttpconfigtlsconfig}

Description
: tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].slackConfigs[].httpConfig.tlsConfig.ca {#_specreceiversslackconfigshttpconfigtlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].slackConfigs[].httpConfig.tlsConfig.ca.configMap {#_specreceiversslackconfigshttpconfigtlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].slackConfigs[].httpConfig.tlsConfig.ca.secret {#_specreceiversslackconfigshttpconfigtlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].slackConfigs[].httpConfig.tlsConfig.cert {#_specreceiversslackconfigshttpconfigtlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].slackConfigs[].httpConfig.tlsConfig.cert.configMap {#_specreceiversslackconfigshttpconfigtlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].slackConfigs[].httpConfig.tlsConfig.cert.secret {#_specreceiversslackconfigshttpconfigtlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].slackConfigs[].httpConfig.tlsConfig.keySecret {#_specreceiversslackconfigshttpconfigtlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].snsConfigs {#_specreceiverssnsconfigs}

Description
: snsConfigs defines the list of SNS configurations

Type
: ```
  `array`
  ```

### .spec.receivers[].snsConfigs[] {#_specreceiverssnsconfigs}

Description
: SNSConfig configures notifications via AWS SNS. See https://prometheus.io/docs/alerting/latest/configuration/#sns_configs

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `apiURL` | `string` | apiURL defines the SNS API URL, e.g. https://sns.us-east-2.amazonaws.com. If not specified, the SNS API URL from the SNS SDK will be used. |
| `attributes` | `object (string)` | attributes defines SNS message attributes as key-value pairs. These provide additional metadata that can be used for message filtering and routing. |
| `httpConfig` | `object` | httpConfig defines the HTTP client configuration for SNS API requests. |
| `message` | `string` | message defines the message content of the SNS notification. This is the actual notification text that will be sent to subscribers. |
| `phoneNumber` | `string` | phoneNumber defines the phone number if message is delivered via SMS in E.164 format. If you don’t specify this value, you must specify a value for the TopicARN or TargetARN. |
| `sendResolved` | `boolean` | sendResolved defines whether or not to notify about resolved alerts. |
| `sigv4` | `object` | sigv4 configures AWS’s Signature Verification 4 signing process to sign requests. This includes AWS credentials and region configuration for authentication. |
| `subject` | `string` | subject defines the subject line when the message is delivered to email endpoints. This field is only used when sending to email subscribers of an SNS topic. |
| `targetARN` | `string` | targetARN defines the mobile platform endpoint ARN if message is delivered via mobile notifications. If you don’t specify this value, you must specify a value for the TopicARN or PhoneNumber. |
| `topicARN` | `string` | topicARN defines the SNS topic ARN, e.g. arn:aws:sns:us-east-2:698519295917:My-Topic. If you don’t specify this value, you must specify a value for the PhoneNumber or TargetARN. |

### .spec.receivers[].snsConfigs[].httpConfig {#_specreceiverssnsconfigshttpconfig}

Description
: httpConfig defines the HTTP client configuration for SNS API requests.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `authorization` | `object` | authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+. |
| `basicAuth` | `object` | basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence. |
| `bearerTokenSecret` | `object` | bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `enableHttp2` | `boolean` | enableHttp2 can be used to disable HTTP2. |
| `followRedirects` | `boolean` | followRedirects defines whether HTTP requests follow HTTP 3xx redirects. When true, the client will automatically follow redirect responses. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `oauth2` | `object` | oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyURL` | `string` | proxyURL defines an optional proxy URL for HTTP requests. If defined, this field takes precedence over `proxyUrl`. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options. |

### .spec.receivers[].snsConfigs[].httpConfig.authorization {#_specreceiverssnsconfigshttpconfigauthorization}

Description
: authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `credentials` | `object` | credentials defines a key of a Secret in the namespace that contains the credentials for authentication. |
| `type` | `string` | type defines the authentication type. The value is case-insensitive.<br>"Basic" is not a supported value.<br>Default: "Bearer" |

### .spec.receivers[].snsConfigs[].httpConfig.authorization.credentials {#_specreceiverssnsconfigshttpconfigauthorizationcredentials}

Description
: credentials defines a key of a Secret in the namespace that contains the credentials for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].snsConfigs[].httpConfig.basicAuth {#_specreceiverssnsconfigshttpconfigbasicauth}

Description
: basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `password` | `object` | password defines a key of a Secret containing the password for authentication. |
| `username` | `object` | username defines a key of a Secret containing the username for authentication. |

### .spec.receivers[].snsConfigs[].httpConfig.basicAuth.password {#_specreceiverssnsconfigshttpconfigbasicauthpassword}

Description
: password defines a key of a Secret containing the password for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].snsConfigs[].httpConfig.basicAuth.username {#_specreceiverssnsconfigshttpconfigbasicauthusername}

Description
: username defines a key of a Secret containing the username for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].snsConfigs[].httpConfig.bearerTokenSecret {#_specreceiverssnsconfigshttpconfigbearertokensecret}

Description
: bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].snsConfigs[].httpConfig.oauth2 {#_specreceiverssnsconfigshttpconfigoauth2}

Description
: oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests.

Type
: ```
  `object`
  ```

Required
: - `clientId`
  - `clientSecret`
  - `tokenUrl`

| Property | Type | Description |
| --- | --- | --- |
| `clientId` | `object` | clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID. |
| `clientSecret` | `object` | clientSecret defines a key of a Secret containing the OAuth2 client’s secret. |
| `endpointParams` | `object (string)` | endpointParams configures the HTTP parameters to append to the token URL. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `scopes` | `array (string)` | scopes defines the OAuth2 scopes used for the token request. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0. |
| `tokenUrl` | `string` | tokenUrl defines the URL to fetch the token from. |

### .spec.receivers[].snsConfigs[].httpConfig.oauth2.clientId {#_specreceiverssnsconfigshttpconfigoauth2clientid}

Description
: clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].snsConfigs[].httpConfig.oauth2.clientId.configMap {#_specreceiverssnsconfigshttpconfigoauth2clientidconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].snsConfigs[].httpConfig.oauth2.clientId.secret {#_specreceiverssnsconfigshttpconfigoauth2clientidsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].snsConfigs[].httpConfig.oauth2.clientSecret {#_specreceiverssnsconfigshttpconfigoauth2clientsecret}

Description
: clientSecret defines a key of a Secret containing the OAuth2 client’s secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].snsConfigs[].httpConfig.oauth2.proxyConnectHeader {#_specreceiverssnsconfigshttpconfigoauth2proxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].snsConfigs[].httpConfig.oauth2.proxyConnectHeader{} {#_specreceiverssnsconfigshttpconfigoauth2proxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].snsConfigs[].httpConfig.oauth2.proxyConnectHeader{}[] {#_specreceiverssnsconfigshttpconfigoauth2proxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].snsConfigs[].httpConfig.oauth2.tlsConfig {#_specreceiverssnsconfigshttpconfigoauth2tlsconfig}

Description
: tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].snsConfigs[].httpConfig.oauth2.tlsConfig.ca {#_specreceiverssnsconfigshttpconfigoauth2tlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].snsConfigs[].httpConfig.oauth2.tlsConfig.ca.configMap {#_specreceiverssnsconfigshttpconfigoauth2tlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].snsConfigs[].httpConfig.oauth2.tlsConfig.ca.secret {#_specreceiverssnsconfigshttpconfigoauth2tlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].snsConfigs[].httpConfig.oauth2.tlsConfig.cert {#_specreceiverssnsconfigshttpconfigoauth2tlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].snsConfigs[].httpConfig.oauth2.tlsConfig.cert.configMap {#_specreceiverssnsconfigshttpconfigoauth2tlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].snsConfigs[].httpConfig.oauth2.tlsConfig.cert.secret {#_specreceiverssnsconfigshttpconfigoauth2tlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].snsConfigs[].httpConfig.oauth2.tlsConfig.keySecret {#_specreceiverssnsconfigshttpconfigoauth2tlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].snsConfigs[].httpConfig.proxyConnectHeader {#_specreceiverssnsconfigshttpconfigproxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].snsConfigs[].httpConfig.proxyConnectHeader{} {#_specreceiverssnsconfigshttpconfigproxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].snsConfigs[].httpConfig.proxyConnectHeader{}[] {#_specreceiverssnsconfigshttpconfigproxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].snsConfigs[].httpConfig.tlsConfig {#_specreceiverssnsconfigshttpconfigtlsconfig}

Description
: tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].snsConfigs[].httpConfig.tlsConfig.ca {#_specreceiverssnsconfigshttpconfigtlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].snsConfigs[].httpConfig.tlsConfig.ca.configMap {#_specreceiverssnsconfigshttpconfigtlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].snsConfigs[].httpConfig.tlsConfig.ca.secret {#_specreceiverssnsconfigshttpconfigtlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].snsConfigs[].httpConfig.tlsConfig.cert {#_specreceiverssnsconfigshttpconfigtlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].snsConfigs[].httpConfig.tlsConfig.cert.configMap {#_specreceiverssnsconfigshttpconfigtlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].snsConfigs[].httpConfig.tlsConfig.cert.secret {#_specreceiverssnsconfigshttpconfigtlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].snsConfigs[].httpConfig.tlsConfig.keySecret {#_specreceiverssnsconfigshttpconfigtlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].snsConfigs[].sigv4 {#_specreceiverssnsconfigssigv4}

Description
: sigv4 configures AWS’s Signature Verification 4 signing process to sign requests. This includes AWS credentials and region configuration for authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `accessKey` | `object` | accessKey defines the AWS API key. If not specified, the environment variable `AWS_ACCESS_KEY_ID` is used. |
| `profile` | `string` | profile defines the named AWS profile used to authenticate. |
| `region` | `string` | region defines the AWS region. If blank, the region from the default credentials chain used. |
| `roleArn` | `string` | roleArn defines the named AWS profile used to authenticate. |
| `secretKey` | `object` | secretKey defines the AWS API secret. If not specified, the environment variable `AWS_SECRET_ACCESS_KEY` is used. |
| `useFIPSSTSEndpoint` | `boolean` | useFIPSSTSEndpoint defines the FIPS mode for the AWS STS endpoint. It requires Prometheus >= v2.54.0. |

### .spec.receivers[].snsConfigs[].sigv4.accessKey {#_specreceiverssnsconfigssigv4accesskey}

Description
: accessKey defines the AWS API key. If not specified, the environment variable `AWS_ACCESS_KEY_ID` is used.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].snsConfigs[].sigv4.secretKey {#_specreceiverssnsconfigssigv4secretkey}

Description
: secretKey defines the AWS API secret. If not specified, the environment variable `AWS_SECRET_ACCESS_KEY` is used.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].telegramConfigs {#_specreceiverstelegramconfigs}

Description
: telegramConfigs defines the list of Telegram configurations.

Type
: ```
  `array`
  ```

### .spec.receivers[].telegramConfigs[] {#_specreceiverstelegramconfigs}

Description
: TelegramConfig configures notifications via Telegram. See https://prometheus.io/docs/alerting/latest/configuration/#telegram_config

Type
: ```
  `object`
  ```

Required
: - `chatID`

| Property | Type | Description |
| --- | --- | --- |
| `apiURL` | `string` | apiURL defines the Telegram API URL, e.g. https://api.telegram.org. If not specified, the default Telegram API URL will be used. |
| `botToken` | `object` | botToken defines the Telegram bot token. It is mutually exclusive with `botTokenFile`. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. Either `botToken` or `botTokenFile` is required. |
| `botTokenFile` | `string` | botTokenFile defines the file to read the Telegram bot token from. It is mutually exclusive with `botToken`. Either `botToken` or `botTokenFile` is required. It requires Alertmanager >= v0.26.0. |
| `chatID` | `integer` | chatID defines the Telegram chat ID where messages will be sent. This can be a user ID, group ID, or channel ID (with @ prefix for public channels). |
| `disableNotifications` | `boolean` | disableNotifications controls whether Telegram notifications are sent silently. When true, users will receive the message without notification sounds. |
| `httpConfig` | `object` | httpConfig defines the HTTP client configuration for Telegram API requests. |
| `message` | `string` | message defines the message template for the Telegram notification. This is the content that will be sent to the specified chat. |
| `messageThreadID` | `integer` | messageThreadID defines the Telegram Group Topic ID for threaded messages. This allows sending messages to specific topics within Telegram groups. It requires Alertmanager >= 0.26.0. |
| `parseMode` | `string` | parseMode defines the parse mode for telegram message formatting. Valid values are "MarkdownV2", "Markdown", and "HTML". This determines how text formatting is interpreted in the message. |
| `sendResolved` | `boolean` | sendResolved defines whether or not to notify about resolved alerts. |

### .spec.receivers[].telegramConfigs[].botToken {#_specreceiverstelegramconfigsbottoken}

Description
: botToken defines the Telegram bot token. It is mutually exclusive with `botTokenFile`. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. Either `botToken` or `botTokenFile` is required.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].telegramConfigs[].httpConfig {#_specreceiverstelegramconfigshttpconfig}

Description
: httpConfig defines the HTTP client configuration for Telegram API requests.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `authorization` | `object` | authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+. |
| `basicAuth` | `object` | basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence. |
| `bearerTokenSecret` | `object` | bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `enableHttp2` | `boolean` | enableHttp2 can be used to disable HTTP2. |
| `followRedirects` | `boolean` | followRedirects defines whether HTTP requests follow HTTP 3xx redirects. When true, the client will automatically follow redirect responses. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `oauth2` | `object` | oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyURL` | `string` | proxyURL defines an optional proxy URL for HTTP requests. If defined, this field takes precedence over `proxyUrl`. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options. |

### .spec.receivers[].telegramConfigs[].httpConfig.authorization {#_specreceiverstelegramconfigshttpconfigauthorization}

Description
: authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `credentials` | `object` | credentials defines a key of a Secret in the namespace that contains the credentials for authentication. |
| `type` | `string` | type defines the authentication type. The value is case-insensitive.<br>"Basic" is not a supported value.<br>Default: "Bearer" |

### .spec.receivers[].telegramConfigs[].httpConfig.authorization.credentials {#_specreceiverstelegramconfigshttpconfigauthorizationcredentials}

Description
: credentials defines a key of a Secret in the namespace that contains the credentials for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].telegramConfigs[].httpConfig.basicAuth {#_specreceiverstelegramconfigshttpconfigbasicauth}

Description
: basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `password` | `object` | password defines a key of a Secret containing the password for authentication. |
| `username` | `object` | username defines a key of a Secret containing the username for authentication. |

### .spec.receivers[].telegramConfigs[].httpConfig.basicAuth.password {#_specreceiverstelegramconfigshttpconfigbasicauthpassword}

Description
: password defines a key of a Secret containing the password for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].telegramConfigs[].httpConfig.basicAuth.username {#_specreceiverstelegramconfigshttpconfigbasicauthusername}

Description
: username defines a key of a Secret containing the username for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].telegramConfigs[].httpConfig.bearerTokenSecret {#_specreceiverstelegramconfigshttpconfigbearertokensecret}

Description
: bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].telegramConfigs[].httpConfig.oauth2 {#_specreceiverstelegramconfigshttpconfigoauth2}

Description
: oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests.

Type
: ```
  `object`
  ```

Required
: - `clientId`
  - `clientSecret`
  - `tokenUrl`

| Property | Type | Description |
| --- | --- | --- |
| `clientId` | `object` | clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID. |
| `clientSecret` | `object` | clientSecret defines a key of a Secret containing the OAuth2 client’s secret. |
| `endpointParams` | `object (string)` | endpointParams configures the HTTP parameters to append to the token URL. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `scopes` | `array (string)` | scopes defines the OAuth2 scopes used for the token request. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0. |
| `tokenUrl` | `string` | tokenUrl defines the URL to fetch the token from. |

### .spec.receivers[].telegramConfigs[].httpConfig.oauth2.clientId {#_specreceiverstelegramconfigshttpconfigoauth2clientid}

Description
: clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].telegramConfigs[].httpConfig.oauth2.clientId.configMap {#_specreceiverstelegramconfigshttpconfigoauth2clientidconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].telegramConfigs[].httpConfig.oauth2.clientId.secret {#_specreceiverstelegramconfigshttpconfigoauth2clientidsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].telegramConfigs[].httpConfig.oauth2.clientSecret {#_specreceiverstelegramconfigshttpconfigoauth2clientsecret}

Description
: clientSecret defines a key of a Secret containing the OAuth2 client’s secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].telegramConfigs[].httpConfig.oauth2.proxyConnectHeader {#_specreceiverstelegramconfigshttpconfigoauth2proxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].telegramConfigs[].httpConfig.oauth2.proxyConnectHeader{} {#_specreceiverstelegramconfigshttpconfigoauth2proxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].telegramConfigs[].httpConfig.oauth2.proxyConnectHeader{}[] {#_specreceiverstelegramconfigshttpconfigoauth2proxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].telegramConfigs[].httpConfig.oauth2.tlsConfig {#_specreceiverstelegramconfigshttpconfigoauth2tlsconfig}

Description
: tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].telegramConfigs[].httpConfig.oauth2.tlsConfig.ca {#_specreceiverstelegramconfigshttpconfigoauth2tlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].telegramConfigs[].httpConfig.oauth2.tlsConfig.ca.configMap {#_specreceiverstelegramconfigshttpconfigoauth2tlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].telegramConfigs[].httpConfig.oauth2.tlsConfig.ca.secret {#_specreceiverstelegramconfigshttpconfigoauth2tlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].telegramConfigs[].httpConfig.oauth2.tlsConfig.cert {#_specreceiverstelegramconfigshttpconfigoauth2tlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].telegramConfigs[].httpConfig.oauth2.tlsConfig.cert.configMap {#_specreceiverstelegramconfigshttpconfigoauth2tlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].telegramConfigs[].httpConfig.oauth2.tlsConfig.cert.secret {#_specreceiverstelegramconfigshttpconfigoauth2tlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].telegramConfigs[].httpConfig.oauth2.tlsConfig.keySecret {#_specreceiverstelegramconfigshttpconfigoauth2tlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].telegramConfigs[].httpConfig.proxyConnectHeader {#_specreceiverstelegramconfigshttpconfigproxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].telegramConfigs[].httpConfig.proxyConnectHeader{} {#_specreceiverstelegramconfigshttpconfigproxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].telegramConfigs[].httpConfig.proxyConnectHeader{}[] {#_specreceiverstelegramconfigshttpconfigproxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].telegramConfigs[].httpConfig.tlsConfig {#_specreceiverstelegramconfigshttpconfigtlsconfig}

Description
: tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].telegramConfigs[].httpConfig.tlsConfig.ca {#_specreceiverstelegramconfigshttpconfigtlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].telegramConfigs[].httpConfig.tlsConfig.ca.configMap {#_specreceiverstelegramconfigshttpconfigtlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].telegramConfigs[].httpConfig.tlsConfig.ca.secret {#_specreceiverstelegramconfigshttpconfigtlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].telegramConfigs[].httpConfig.tlsConfig.cert {#_specreceiverstelegramconfigshttpconfigtlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].telegramConfigs[].httpConfig.tlsConfig.cert.configMap {#_specreceiverstelegramconfigshttpconfigtlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].telegramConfigs[].httpConfig.tlsConfig.cert.secret {#_specreceiverstelegramconfigshttpconfigtlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].telegramConfigs[].httpConfig.tlsConfig.keySecret {#_specreceiverstelegramconfigshttpconfigtlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].victoropsConfigs {#_specreceiversvictoropsconfigs}

Description
: victoropsConfigs defines the list of VictorOps configurations.

Type
: ```
  `array`
  ```

### .spec.receivers[].victoropsConfigs[] {#_specreceiversvictoropsconfigs}

Description
: VictorOpsConfig configures notifications via VictorOps. See https://prometheus.io/docs/alerting/latest/configuration/#victorops_config

Type
: ```
  `object`
  ```

Required
: - `routingKey`

| Property | Type | Description |
| --- | --- | --- |
| `apiKey` | `object` | apiKey defines the secret’s key that contains the API key to use when talking to the VictorOps API. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `apiUrl` | `string` | apiUrl defines the VictorOps API URL. When not specified, defaults to the standard VictorOps API endpoint. |
| `customFields` | `array` | customFields defines additional custom fields for notification. These provide extra metadata that will be included with the VictorOps incident. |
| `customFields[]` | `object` | KeyValue defines a (key, value) tuple. |
| `entityDisplayName` | `string` | entityDisplayName contains a summary of the alerted problem. This appears as the main title or identifier for the incident. |
| `httpConfig` | `object` | httpConfig defines the HTTP client’s configuration for VictorOps API requests. |
| `messageType` | `string` | messageType describes the behavior of the alert. Valid values are "CRITICAL", "WARNING", and "INFO". |
| `monitoringTool` | `string` | monitoringTool defines the monitoring tool the state message is from. This helps identify the source system that generated the alert. |
| `routingKey` | `string` | routingKey defines a key used to map the alert to a team. This determines which VictorOps team will receive the alert notification. |
| `sendResolved` | `boolean` | sendResolved defines whether or not to notify about resolved alerts. |
| `stateMessage` | `string` | stateMessage contains a long explanation of the alerted problem. This provides detailed context about the incident. |

### .spec.receivers[].victoropsConfigs[].apiKey {#_specreceiversvictoropsconfigsapikey}

Description
: apiKey defines the secret’s key that contains the API key to use when talking to the VictorOps API. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].victoropsConfigs[].customFields {#_specreceiversvictoropsconfigscustomfields}

Description
: customFields defines additional custom fields for notification. These provide extra metadata that will be included with the VictorOps incident.

Type
: ```
  `array`
  ```

### .spec.receivers[].victoropsConfigs[].customFields[] {#_specreceiversvictoropsconfigscustomfields}

Description
: KeyValue defines a (key, value) tuple.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `value`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the tuple. This is the identifier or name part of the key-value pair. |
| `value` | `string` | value defines the value of the tuple. This is the data or content associated with the key. |

### .spec.receivers[].victoropsConfigs[].httpConfig {#_specreceiversvictoropsconfigshttpconfig}

Description
: httpConfig defines the HTTP client’s configuration for VictorOps API requests.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `authorization` | `object` | authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+. |
| `basicAuth` | `object` | basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence. |
| `bearerTokenSecret` | `object` | bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `enableHttp2` | `boolean` | enableHttp2 can be used to disable HTTP2. |
| `followRedirects` | `boolean` | followRedirects defines whether HTTP requests follow HTTP 3xx redirects. When true, the client will automatically follow redirect responses. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `oauth2` | `object` | oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyURL` | `string` | proxyURL defines an optional proxy URL for HTTP requests. If defined, this field takes precedence over `proxyUrl`. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options. |

### .spec.receivers[].victoropsConfigs[].httpConfig.authorization {#_specreceiversvictoropsconfigshttpconfigauthorization}

Description
: authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `credentials` | `object` | credentials defines a key of a Secret in the namespace that contains the credentials for authentication. |
| `type` | `string` | type defines the authentication type. The value is case-insensitive.<br>"Basic" is not a supported value.<br>Default: "Bearer" |

### .spec.receivers[].victoropsConfigs[].httpConfig.authorization.credentials {#_specreceiversvictoropsconfigshttpconfigauthorizationcredentials}

Description
: credentials defines a key of a Secret in the namespace that contains the credentials for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].victoropsConfigs[].httpConfig.basicAuth {#_specreceiversvictoropsconfigshttpconfigbasicauth}

Description
: basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `password` | `object` | password defines a key of a Secret containing the password for authentication. |
| `username` | `object` | username defines a key of a Secret containing the username for authentication. |

### .spec.receivers[].victoropsConfigs[].httpConfig.basicAuth.password {#_specreceiversvictoropsconfigshttpconfigbasicauthpassword}

Description
: password defines a key of a Secret containing the password for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].victoropsConfigs[].httpConfig.basicAuth.username {#_specreceiversvictoropsconfigshttpconfigbasicauthusername}

Description
: username defines a key of a Secret containing the username for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].victoropsConfigs[].httpConfig.bearerTokenSecret {#_specreceiversvictoropsconfigshttpconfigbearertokensecret}

Description
: bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].victoropsConfigs[].httpConfig.oauth2 {#_specreceiversvictoropsconfigshttpconfigoauth2}

Description
: oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests.

Type
: ```
  `object`
  ```

Required
: - `clientId`
  - `clientSecret`
  - `tokenUrl`

| Property | Type | Description |
| --- | --- | --- |
| `clientId` | `object` | clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID. |
| `clientSecret` | `object` | clientSecret defines a key of a Secret containing the OAuth2 client’s secret. |
| `endpointParams` | `object (string)` | endpointParams configures the HTTP parameters to append to the token URL. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `scopes` | `array (string)` | scopes defines the OAuth2 scopes used for the token request. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0. |
| `tokenUrl` | `string` | tokenUrl defines the URL to fetch the token from. |

### .spec.receivers[].victoropsConfigs[].httpConfig.oauth2.clientId {#_specreceiversvictoropsconfigshttpconfigoauth2clientid}

Description
: clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].victoropsConfigs[].httpConfig.oauth2.clientId.configMap {#_specreceiversvictoropsconfigshttpconfigoauth2clientidconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].victoropsConfigs[].httpConfig.oauth2.clientId.secret {#_specreceiversvictoropsconfigshttpconfigoauth2clientidsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].victoropsConfigs[].httpConfig.oauth2.clientSecret {#_specreceiversvictoropsconfigshttpconfigoauth2clientsecret}

Description
: clientSecret defines a key of a Secret containing the OAuth2 client’s secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].victoropsConfigs[].httpConfig.oauth2.proxyConnectHeader {#_specreceiversvictoropsconfigshttpconfigoauth2proxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].victoropsConfigs[].httpConfig.oauth2.proxyConnectHeader{} {#_specreceiversvictoropsconfigshttpconfigoauth2proxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].victoropsConfigs[].httpConfig.oauth2.proxyConnectHeader{}[] {#_specreceiversvictoropsconfigshttpconfigoauth2proxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].victoropsConfigs[].httpConfig.oauth2.tlsConfig {#_specreceiversvictoropsconfigshttpconfigoauth2tlsconfig}

Description
: tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].victoropsConfigs[].httpConfig.oauth2.tlsConfig.ca {#_specreceiversvictoropsconfigshttpconfigoauth2tlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].victoropsConfigs[].httpConfig.oauth2.tlsConfig.ca.configMap {#_specreceiversvictoropsconfigshttpconfigoauth2tlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].victoropsConfigs[].httpConfig.oauth2.tlsConfig.ca.secret {#_specreceiversvictoropsconfigshttpconfigoauth2tlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].victoropsConfigs[].httpConfig.oauth2.tlsConfig.cert {#_specreceiversvictoropsconfigshttpconfigoauth2tlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].victoropsConfigs[].httpConfig.oauth2.tlsConfig.cert.configMap {#_specreceiversvictoropsconfigshttpconfigoauth2tlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].victoropsConfigs[].httpConfig.oauth2.tlsConfig.cert.secret {#_specreceiversvictoropsconfigshttpconfigoauth2tlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].victoropsConfigs[].httpConfig.oauth2.tlsConfig.keySecret {#_specreceiversvictoropsconfigshttpconfigoauth2tlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].victoropsConfigs[].httpConfig.proxyConnectHeader {#_specreceiversvictoropsconfigshttpconfigproxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].victoropsConfigs[].httpConfig.proxyConnectHeader{} {#_specreceiversvictoropsconfigshttpconfigproxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].victoropsConfigs[].httpConfig.proxyConnectHeader{}[] {#_specreceiversvictoropsconfigshttpconfigproxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].victoropsConfigs[].httpConfig.tlsConfig {#_specreceiversvictoropsconfigshttpconfigtlsconfig}

Description
: tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].victoropsConfigs[].httpConfig.tlsConfig.ca {#_specreceiversvictoropsconfigshttpconfigtlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].victoropsConfigs[].httpConfig.tlsConfig.ca.configMap {#_specreceiversvictoropsconfigshttpconfigtlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].victoropsConfigs[].httpConfig.tlsConfig.ca.secret {#_specreceiversvictoropsconfigshttpconfigtlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].victoropsConfigs[].httpConfig.tlsConfig.cert {#_specreceiversvictoropsconfigshttpconfigtlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].victoropsConfigs[].httpConfig.tlsConfig.cert.configMap {#_specreceiversvictoropsconfigshttpconfigtlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].victoropsConfigs[].httpConfig.tlsConfig.cert.secret {#_specreceiversvictoropsconfigshttpconfigtlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].victoropsConfigs[].httpConfig.tlsConfig.keySecret {#_specreceiversvictoropsconfigshttpconfigtlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webexConfigs {#_specreceiverswebexconfigs}

Description
: webexConfigs defines the list of Webex configurations.

Type
: ```
  `array`
  ```

### .spec.receivers[].webexConfigs[] {#_specreceiverswebexconfigs}

Description
: WebexConfig configures notification via Cisco Webex See https://prometheus.io/docs/alerting/latest/configuration/#webex_config

Type
: ```
  `object`
  ```

Required
: - `roomID`

| Property | Type | Description |
| --- | --- | --- |
| `apiURL` | `string` | apiURL defines the Webex Teams API URL i.e. https://webexapis.com/v1/messages |
| `httpConfig` | `object` | httpConfig defines the HTTP client’s configuration. You must use this configuration to supply the bot token as part of the HTTP `Authorization` header. |
| `message` | `string` | message defines the message template |
| `roomID` | `string` | roomID defines the ID of the Webex Teams room where to send the messages. |
| `sendResolved` | `boolean` | sendResolved defines whether or not to notify about resolved alerts. |

### .spec.receivers[].webexConfigs[].httpConfig {#_specreceiverswebexconfigshttpconfig}

Description
: httpConfig defines the HTTP client’s configuration. You must use this configuration to supply the bot token as part of the HTTP `Authorization` header.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `authorization` | `object` | authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+. |
| `basicAuth` | `object` | basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence. |
| `bearerTokenSecret` | `object` | bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `enableHttp2` | `boolean` | enableHttp2 can be used to disable HTTP2. |
| `followRedirects` | `boolean` | followRedirects defines whether HTTP requests follow HTTP 3xx redirects. When true, the client will automatically follow redirect responses. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `oauth2` | `object` | oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyURL` | `string` | proxyURL defines an optional proxy URL for HTTP requests. If defined, this field takes precedence over `proxyUrl`. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options. |

### .spec.receivers[].webexConfigs[].httpConfig.authorization {#_specreceiverswebexconfigshttpconfigauthorization}

Description
: authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `credentials` | `object` | credentials defines a key of a Secret in the namespace that contains the credentials for authentication. |
| `type` | `string` | type defines the authentication type. The value is case-insensitive.<br>"Basic" is not a supported value.<br>Default: "Bearer" |

### .spec.receivers[].webexConfigs[].httpConfig.authorization.credentials {#_specreceiverswebexconfigshttpconfigauthorizationcredentials}

Description
: credentials defines a key of a Secret in the namespace that contains the credentials for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webexConfigs[].httpConfig.basicAuth {#_specreceiverswebexconfigshttpconfigbasicauth}

Description
: basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `password` | `object` | password defines a key of a Secret containing the password for authentication. |
| `username` | `object` | username defines a key of a Secret containing the username for authentication. |

### .spec.receivers[].webexConfigs[].httpConfig.basicAuth.password {#_specreceiverswebexconfigshttpconfigbasicauthpassword}

Description
: password defines a key of a Secret containing the password for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webexConfigs[].httpConfig.basicAuth.username {#_specreceiverswebexconfigshttpconfigbasicauthusername}

Description
: username defines a key of a Secret containing the username for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webexConfigs[].httpConfig.bearerTokenSecret {#_specreceiverswebexconfigshttpconfigbearertokensecret}

Description
: bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].webexConfigs[].httpConfig.oauth2 {#_specreceiverswebexconfigshttpconfigoauth2}

Description
: oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests.

Type
: ```
  `object`
  ```

Required
: - `clientId`
  - `clientSecret`
  - `tokenUrl`

| Property | Type | Description |
| --- | --- | --- |
| `clientId` | `object` | clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID. |
| `clientSecret` | `object` | clientSecret defines a key of a Secret containing the OAuth2 client’s secret. |
| `endpointParams` | `object (string)` | endpointParams configures the HTTP parameters to append to the token URL. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `scopes` | `array (string)` | scopes defines the OAuth2 scopes used for the token request. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0. |
| `tokenUrl` | `string` | tokenUrl defines the URL to fetch the token from. |

### .spec.receivers[].webexConfigs[].httpConfig.oauth2.clientId {#_specreceiverswebexconfigshttpconfigoauth2clientid}

Description
: clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].webexConfigs[].httpConfig.oauth2.clientId.configMap {#_specreceiverswebexconfigshttpconfigoauth2clientidconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].webexConfigs[].httpConfig.oauth2.clientId.secret {#_specreceiverswebexconfigshttpconfigoauth2clientidsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webexConfigs[].httpConfig.oauth2.clientSecret {#_specreceiverswebexconfigshttpconfigoauth2clientsecret}

Description
: clientSecret defines a key of a Secret containing the OAuth2 client’s secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webexConfigs[].httpConfig.oauth2.proxyConnectHeader {#_specreceiverswebexconfigshttpconfigoauth2proxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].webexConfigs[].httpConfig.oauth2.proxyConnectHeader{} {#_specreceiverswebexconfigshttpconfigoauth2proxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].webexConfigs[].httpConfig.oauth2.proxyConnectHeader{}[] {#_specreceiverswebexconfigshttpconfigoauth2proxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webexConfigs[].httpConfig.oauth2.tlsConfig {#_specreceiverswebexconfigshttpconfigoauth2tlsconfig}

Description
: tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].webexConfigs[].httpConfig.oauth2.tlsConfig.ca {#_specreceiverswebexconfigshttpconfigoauth2tlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].webexConfigs[].httpConfig.oauth2.tlsConfig.ca.configMap {#_specreceiverswebexconfigshttpconfigoauth2tlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].webexConfigs[].httpConfig.oauth2.tlsConfig.ca.secret {#_specreceiverswebexconfigshttpconfigoauth2tlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webexConfigs[].httpConfig.oauth2.tlsConfig.cert {#_specreceiverswebexconfigshttpconfigoauth2tlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].webexConfigs[].httpConfig.oauth2.tlsConfig.cert.configMap {#_specreceiverswebexconfigshttpconfigoauth2tlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].webexConfigs[].httpConfig.oauth2.tlsConfig.cert.secret {#_specreceiverswebexconfigshttpconfigoauth2tlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webexConfigs[].httpConfig.oauth2.tlsConfig.keySecret {#_specreceiverswebexconfigshttpconfigoauth2tlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webexConfigs[].httpConfig.proxyConnectHeader {#_specreceiverswebexconfigshttpconfigproxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].webexConfigs[].httpConfig.proxyConnectHeader{} {#_specreceiverswebexconfigshttpconfigproxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].webexConfigs[].httpConfig.proxyConnectHeader{}[] {#_specreceiverswebexconfigshttpconfigproxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webexConfigs[].httpConfig.tlsConfig {#_specreceiverswebexconfigshttpconfigtlsconfig}

Description
: tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].webexConfigs[].httpConfig.tlsConfig.ca {#_specreceiverswebexconfigshttpconfigtlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].webexConfigs[].httpConfig.tlsConfig.ca.configMap {#_specreceiverswebexconfigshttpconfigtlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].webexConfigs[].httpConfig.tlsConfig.ca.secret {#_specreceiverswebexconfigshttpconfigtlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webexConfigs[].httpConfig.tlsConfig.cert {#_specreceiverswebexconfigshttpconfigtlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].webexConfigs[].httpConfig.tlsConfig.cert.configMap {#_specreceiverswebexconfigshttpconfigtlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].webexConfigs[].httpConfig.tlsConfig.cert.secret {#_specreceiverswebexconfigshttpconfigtlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webexConfigs[].httpConfig.tlsConfig.keySecret {#_specreceiverswebexconfigshttpconfigtlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webhookConfigs {#_specreceiverswebhookconfigs}

Description
: webhookConfigs defines the List of webhook configurations.

Type
: ```
  `array`
  ```

### .spec.receivers[].webhookConfigs[] {#_specreceiverswebhookconfigs}

Description
: WebhookConfig configures notifications via a generic receiver supporting the webhook payload. See https://prometheus.io/docs/alerting/latest/configuration/#webhook_config

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `httpConfig` | `object` | httpConfig defines the HTTP client configuration for webhook requests. |
| `maxAlerts` | `integer` | maxAlerts defines the maximum number of alerts to be sent per webhook message. When 0, all alerts are included in the webhook payload. |
| `sendResolved` | `boolean` | sendResolved defines whether or not to notify about resolved alerts. |
| `timeout` | `string` | timeout defines the maximum time to wait for a webhook request to complete, before failing the request and allowing it to be retried. It requires Alertmanager >= v0.28.0. |
| `url` | `string` | url defines the URL to send HTTP POST requests to. urlSecret takes precedence over url. One of urlSecret and url should be defined. |
| `urlSecret` | `object` | urlSecret defines the secret’s key that contains the webhook URL to send HTTP requests to. urlSecret takes precedence over url. One of urlSecret and url should be defined. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |

### .spec.receivers[].webhookConfigs[].httpConfig {#_specreceiverswebhookconfigshttpconfig}

Description
: httpConfig defines the HTTP client configuration for webhook requests.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `authorization` | `object` | authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+. |
| `basicAuth` | `object` | basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence. |
| `bearerTokenSecret` | `object` | bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `enableHttp2` | `boolean` | enableHttp2 can be used to disable HTTP2. |
| `followRedirects` | `boolean` | followRedirects defines whether HTTP requests follow HTTP 3xx redirects. When true, the client will automatically follow redirect responses. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `oauth2` | `object` | oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyURL` | `string` | proxyURL defines an optional proxy URL for HTTP requests. If defined, this field takes precedence over `proxyUrl`. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options. |

### .spec.receivers[].webhookConfigs[].httpConfig.authorization {#_specreceiverswebhookconfigshttpconfigauthorization}

Description
: authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `credentials` | `object` | credentials defines a key of a Secret in the namespace that contains the credentials for authentication. |
| `type` | `string` | type defines the authentication type. The value is case-insensitive.<br>"Basic" is not a supported value.<br>Default: "Bearer" |

### .spec.receivers[].webhookConfigs[].httpConfig.authorization.credentials {#_specreceiverswebhookconfigshttpconfigauthorizationcredentials}

Description
: credentials defines a key of a Secret in the namespace that contains the credentials for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webhookConfigs[].httpConfig.basicAuth {#_specreceiverswebhookconfigshttpconfigbasicauth}

Description
: basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `password` | `object` | password defines a key of a Secret containing the password for authentication. |
| `username` | `object` | username defines a key of a Secret containing the username for authentication. |

### .spec.receivers[].webhookConfigs[].httpConfig.basicAuth.password {#_specreceiverswebhookconfigshttpconfigbasicauthpassword}

Description
: password defines a key of a Secret containing the password for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webhookConfigs[].httpConfig.basicAuth.username {#_specreceiverswebhookconfigshttpconfigbasicauthusername}

Description
: username defines a key of a Secret containing the username for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webhookConfigs[].httpConfig.bearerTokenSecret {#_specreceiverswebhookconfigshttpconfigbearertokensecret}

Description
: bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].webhookConfigs[].httpConfig.oauth2 {#_specreceiverswebhookconfigshttpconfigoauth2}

Description
: oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests.

Type
: ```
  `object`
  ```

Required
: - `clientId`
  - `clientSecret`
  - `tokenUrl`

| Property | Type | Description |
| --- | --- | --- |
| `clientId` | `object` | clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID. |
| `clientSecret` | `object` | clientSecret defines a key of a Secret containing the OAuth2 client’s secret. |
| `endpointParams` | `object (string)` | endpointParams configures the HTTP parameters to append to the token URL. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `scopes` | `array (string)` | scopes defines the OAuth2 scopes used for the token request. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0. |
| `tokenUrl` | `string` | tokenUrl defines the URL to fetch the token from. |

### .spec.receivers[].webhookConfigs[].httpConfig.oauth2.clientId {#_specreceiverswebhookconfigshttpconfigoauth2clientid}

Description
: clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].webhookConfigs[].httpConfig.oauth2.clientId.configMap {#_specreceiverswebhookconfigshttpconfigoauth2clientidconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].webhookConfigs[].httpConfig.oauth2.clientId.secret {#_specreceiverswebhookconfigshttpconfigoauth2clientidsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webhookConfigs[].httpConfig.oauth2.clientSecret {#_specreceiverswebhookconfigshttpconfigoauth2clientsecret}

Description
: clientSecret defines a key of a Secret containing the OAuth2 client’s secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webhookConfigs[].httpConfig.oauth2.proxyConnectHeader {#_specreceiverswebhookconfigshttpconfigoauth2proxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].webhookConfigs[].httpConfig.oauth2.proxyConnectHeader{} {#_specreceiverswebhookconfigshttpconfigoauth2proxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].webhookConfigs[].httpConfig.oauth2.proxyConnectHeader{}[] {#_specreceiverswebhookconfigshttpconfigoauth2proxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webhookConfigs[].httpConfig.oauth2.tlsConfig {#_specreceiverswebhookconfigshttpconfigoauth2tlsconfig}

Description
: tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].webhookConfigs[].httpConfig.oauth2.tlsConfig.ca {#_specreceiverswebhookconfigshttpconfigoauth2tlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].webhookConfigs[].httpConfig.oauth2.tlsConfig.ca.configMap {#_specreceiverswebhookconfigshttpconfigoauth2tlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].webhookConfigs[].httpConfig.oauth2.tlsConfig.ca.secret {#_specreceiverswebhookconfigshttpconfigoauth2tlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webhookConfigs[].httpConfig.oauth2.tlsConfig.cert {#_specreceiverswebhookconfigshttpconfigoauth2tlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].webhookConfigs[].httpConfig.oauth2.tlsConfig.cert.configMap {#_specreceiverswebhookconfigshttpconfigoauth2tlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].webhookConfigs[].httpConfig.oauth2.tlsConfig.cert.secret {#_specreceiverswebhookconfigshttpconfigoauth2tlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webhookConfigs[].httpConfig.oauth2.tlsConfig.keySecret {#_specreceiverswebhookconfigshttpconfigoauth2tlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webhookConfigs[].httpConfig.proxyConnectHeader {#_specreceiverswebhookconfigshttpconfigproxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].webhookConfigs[].httpConfig.proxyConnectHeader{} {#_specreceiverswebhookconfigshttpconfigproxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].webhookConfigs[].httpConfig.proxyConnectHeader{}[] {#_specreceiverswebhookconfigshttpconfigproxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webhookConfigs[].httpConfig.tlsConfig {#_specreceiverswebhookconfigshttpconfigtlsconfig}

Description
: tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].webhookConfigs[].httpConfig.tlsConfig.ca {#_specreceiverswebhookconfigshttpconfigtlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].webhookConfigs[].httpConfig.tlsConfig.ca.configMap {#_specreceiverswebhookconfigshttpconfigtlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].webhookConfigs[].httpConfig.tlsConfig.ca.secret {#_specreceiverswebhookconfigshttpconfigtlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webhookConfigs[].httpConfig.tlsConfig.cert {#_specreceiverswebhookconfigshttpconfigtlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].webhookConfigs[].httpConfig.tlsConfig.cert.configMap {#_specreceiverswebhookconfigshttpconfigtlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].webhookConfigs[].httpConfig.tlsConfig.cert.secret {#_specreceiverswebhookconfigshttpconfigtlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webhookConfigs[].httpConfig.tlsConfig.keySecret {#_specreceiverswebhookconfigshttpconfigtlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].webhookConfigs[].urlSecret {#_specreceiverswebhookconfigsurlsecret}

Description
: urlSecret defines the secret’s key that contains the webhook URL to send HTTP requests to. urlSecret takes precedence over url. One of urlSecret and url should be defined. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].wechatConfigs {#_specreceiverswechatconfigs}

Description
: wechatConfigs defines the list of WeChat configurations.

Type
: ```
  `array`
  ```

### .spec.receivers[].wechatConfigs[] {#_specreceiverswechatconfigs}

Description
: WeChatConfig configures notifications via WeChat. See https://prometheus.io/docs/alerting/latest/configuration/#wechat_config

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `agentID` | `string` | agentID defines the application agent ID within WeChat Work. This identifies which WeChat Work application will send the notifications. |
| `apiSecret` | `object` | apiSecret defines the secret’s key that contains the WeChat API key. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `apiURL` | `string` | apiURL defines the WeChat API URL. When not specified, defaults to the standard WeChat Work API endpoint. |
| `corpID` | `string` | corpID defines the corp id for authentication. This is the unique identifier for your WeChat Work organization. |
| `httpConfig` | `object` | httpConfig defines the HTTP client configuration for WeChat API requests. |
| `message` | `string` | message defines the API request data as defined by the WeChat API. This contains the actual notification content to be sent. |
| `messageType` | `string` | messageType defines the type of message to send. Valid values include "text", "markdown", and other WeChat Work supported message types. |
| `sendResolved` | `boolean` | sendResolved defines whether or not to notify about resolved alerts. |
| `toParty` | `string` | toParty defines the target department(s) to receive the notification. Can be a single department ID or multiple department IDs separated by '\|'. |
| `toTag` | `string` | toTag defines the target tag(s) to receive the notification. Can be a single tag ID or multiple tag IDs separated by '\|'. |
| `toUser` | `string` | toUser defines the target user(s) to receive the notification. Can be a single user ID or multiple user IDs separated by '\|'. |

### .spec.receivers[].wechatConfigs[].apiSecret {#_specreceiverswechatconfigsapisecret}

Description
: apiSecret defines the secret’s key that contains the WeChat API key. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].wechatConfigs[].httpConfig {#_specreceiverswechatconfigshttpconfig}

Description
: httpConfig defines the HTTP client configuration for WeChat API requests.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `authorization` | `object` | authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+. |
| `basicAuth` | `object` | basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence. |
| `bearerTokenSecret` | `object` | bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator. |
| `enableHttp2` | `boolean` | enableHttp2 can be used to disable HTTP2. |
| `followRedirects` | `boolean` | followRedirects defines whether HTTP requests follow HTTP 3xx redirects. When true, the client will automatically follow redirect responses. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `oauth2` | `object` | oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyURL` | `string` | proxyURL defines an optional proxy URL for HTTP requests. If defined, this field takes precedence over `proxyUrl`. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options. |

### .spec.receivers[].wechatConfigs[].httpConfig.authorization {#_specreceiverswechatconfigshttpconfigauthorization}

Description
: authorization defines the authorization header configuration for the client. This is mutually exclusive with BasicAuth and is only available starting from Alertmanager v0.22+.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `credentials` | `object` | credentials defines a key of a Secret in the namespace that contains the credentials for authentication. |
| `type` | `string` | type defines the authentication type. The value is case-insensitive.<br>"Basic" is not a supported value.<br>Default: "Bearer" |

### .spec.receivers[].wechatConfigs[].httpConfig.authorization.credentials {#_specreceiverswechatconfigshttpconfigauthorizationcredentials}

Description
: credentials defines a key of a Secret in the namespace that contains the credentials for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].wechatConfigs[].httpConfig.basicAuth {#_specreceiverswechatconfigshttpconfigbasicauth}

Description
: basicAuth defines the basic authentication credentials for the client. This is mutually exclusive with Authorization. If both are defined, BasicAuth takes precedence.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `password` | `object` | password defines a key of a Secret containing the password for authentication. |
| `username` | `object` | username defines a key of a Secret containing the username for authentication. |

### .spec.receivers[].wechatConfigs[].httpConfig.basicAuth.password {#_specreceiverswechatconfigshttpconfigbasicauthpassword}

Description
: password defines a key of a Secret containing the password for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].wechatConfigs[].httpConfig.basicAuth.username {#_specreceiverswechatconfigshttpconfigbasicauthusername}

Description
: username defines a key of a Secret containing the username for authentication.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].wechatConfigs[].httpConfig.bearerTokenSecret {#_specreceiverswechatconfigshttpconfigbearertokensecret}

Description
: bearerTokenSecret defines the secret’s key that contains the bearer token to be used by the client for authentication. The secret needs to be in the same namespace as the AlertmanagerConfig object and accessible by the Prometheus Operator.

Type
: ```
  `object`
  ```

Required
: - `key`
  - `name`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | key defines the key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | name defines the name of the secret in the object’s namespace to select from. |

### .spec.receivers[].wechatConfigs[].httpConfig.oauth2 {#_specreceiverswechatconfigshttpconfigoauth2}

Description
: oauth2 defines the OAuth2 client credentials used to fetch a token for the targets. This enables OAuth2 authentication flow for HTTP requests.

Type
: ```
  `object`
  ```

Required
: - `clientId`
  - `clientSecret`
  - `tokenUrl`

| Property | Type | Description |
| --- | --- | --- |
| `clientId` | `object` | clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID. |
| `clientSecret` | `object` | clientSecret defines a key of a Secret containing the OAuth2 client’s secret. |
| `endpointParams` | `object (string)` | endpointParams configures the HTTP parameters to append to the token URL. |
| `noProxy` | `string` | noProxy defines a comma-separated string that can contain IPs, CIDR notation, domain names that should be excluded from proxying. IP and domain names can contain port numbers.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader` | `object` | proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyConnectHeader{}` | `array` |  |
| `proxyConnectHeader{}[]` | `object` | SecretKeySelector selects a key of a Secret. |
| `proxyFromEnvironment` | `boolean` | proxyFromEnvironment defines whether to use the proxy configuration defined by environment variables (HTTP_PROXY, HTTPS_PROXY, and NO_PROXY).<br>It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0. |
| `proxyUrl` | `string` | proxyUrl defines the HTTP proxy server to use. |
| `scopes` | `array (string)` | scopes defines the OAuth2 scopes used for the token request. |
| `tlsConfig` | `object` | tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0. |
| `tokenUrl` | `string` | tokenUrl defines the URL to fetch the token from. |

### .spec.receivers[].wechatConfigs[].httpConfig.oauth2.clientId {#_specreceiverswechatconfigshttpconfigoauth2clientid}

Description
: clientId defines a key of a Secret or ConfigMap containing the OAuth2 client’s ID.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].wechatConfigs[].httpConfig.oauth2.clientId.configMap {#_specreceiverswechatconfigshttpconfigoauth2clientidconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].wechatConfigs[].httpConfig.oauth2.clientId.secret {#_specreceiverswechatconfigshttpconfigoauth2clientidsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].wechatConfigs[].httpConfig.oauth2.clientSecret {#_specreceiverswechatconfigshttpconfigoauth2clientsecret}

Description
: clientSecret defines a key of a Secret containing the OAuth2 client’s secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].wechatConfigs[].httpConfig.oauth2.proxyConnectHeader {#_specreceiverswechatconfigshttpconfigoauth2proxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].wechatConfigs[].httpConfig.oauth2.proxyConnectHeader{} {#_specreceiverswechatconfigshttpconfigoauth2proxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].wechatConfigs[].httpConfig.oauth2.proxyConnectHeader{}[] {#_specreceiverswechatconfigshttpconfigoauth2proxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].wechatConfigs[].httpConfig.oauth2.tlsConfig {#_specreceiverswechatconfigshttpconfigoauth2tlsconfig}

Description
: tlsConfig defines the TLS configuration to use when connecting to the OAuth2 server. It requires Prometheus >= v2.43.0.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].wechatConfigs[].httpConfig.oauth2.tlsConfig.ca {#_specreceiverswechatconfigshttpconfigoauth2tlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].wechatConfigs[].httpConfig.oauth2.tlsConfig.ca.configMap {#_specreceiverswechatconfigshttpconfigoauth2tlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].wechatConfigs[].httpConfig.oauth2.tlsConfig.ca.secret {#_specreceiverswechatconfigshttpconfigoauth2tlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].wechatConfigs[].httpConfig.oauth2.tlsConfig.cert {#_specreceiverswechatconfigshttpconfigoauth2tlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].wechatConfigs[].httpConfig.oauth2.tlsConfig.cert.configMap {#_specreceiverswechatconfigshttpconfigoauth2tlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].wechatConfigs[].httpConfig.oauth2.tlsConfig.cert.secret {#_specreceiverswechatconfigshttpconfigoauth2tlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].wechatConfigs[].httpConfig.oauth2.tlsConfig.keySecret {#_specreceiverswechatconfigshttpconfigoauth2tlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].wechatConfigs[].httpConfig.proxyConnectHeader {#_specreceiverswechatconfigshttpconfigproxyconnectheader}

Description
: proxyConnectHeader optionally specifies headers to send to proxies during CONNECT requests.

  It requires Prometheus >= v2.43.0, Alertmanager >= v0.25.0 or Thanos >= v0.32.0.

Type
: ```
  `object`
  ```

### .spec.receivers[].wechatConfigs[].httpConfig.proxyConnectHeader{} {#_specreceiverswechatconfigshttpconfigproxyconnectheader}

Description

Type
: ```
  `array`
  ```

### .spec.receivers[].wechatConfigs[].httpConfig.proxyConnectHeader{}[] {#_specreceiverswechatconfigshttpconfigproxyconnectheader}

Description
: SecretKeySelector selects a key of a Secret.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].wechatConfigs[].httpConfig.tlsConfig {#_specreceiverswechatconfigshttpconfigtlsconfig}

Description
: tlsConfig defines the TLS configuration for the client. This includes settings for certificates, CA validation, and TLS protocol options.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `ca` | `object` | ca defines the Certificate authority used when verifying server certificates. |
| `cert` | `object` | cert defines the Client certificate to present when doing client-authentication. |
| `insecureSkipVerify` | `boolean` | insecureSkipVerify defines how to disable target certificate validation. |
| `keySecret` | `object` | keySecret defines the Secret containing the client key file for the targets. |
| `maxVersion` | `string` | maxVersion defines the maximum acceptable TLS version.<br>It requires Prometheus >= v2.41.0 or Thanos >= v0.31.0. |
| `minVersion` | `string` | minVersion defines the minimum acceptable TLS version.<br>It requires Prometheus >= v2.35.0 or Thanos >= v0.28.0. |
| `serverName` | `string` | serverName is used to verify the hostname for the targets. |

### .spec.receivers[].wechatConfigs[].httpConfig.tlsConfig.ca {#_specreceiverswechatconfigshttpconfigtlsconfigca}

Description
: ca defines the Certificate authority used when verifying server certificates.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].wechatConfigs[].httpConfig.tlsConfig.ca.configMap {#_specreceiverswechatconfigshttpconfigtlsconfigcaconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].wechatConfigs[].httpConfig.tlsConfig.ca.secret {#_specreceiverswechatconfigshttpconfigtlsconfigcasecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].wechatConfigs[].httpConfig.tlsConfig.cert {#_specreceiverswechatconfigshttpconfigtlsconfigcert}

Description
: cert defines the Client certificate to present when doing client-authentication.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `configMap` | `object` | configMap defines the ConfigMap containing data to use for the targets. |
| `secret` | `object` | secret defines the Secret containing data to use for the targets. |

### .spec.receivers[].wechatConfigs[].httpConfig.tlsConfig.cert.configMap {#_specreceiverswechatconfigshttpconfigtlsconfigcertconfigmap}

Description
: configMap defines the ConfigMap containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to select. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the ConfigMap or its key must be defined |

### .spec.receivers[].wechatConfigs[].httpConfig.tlsConfig.cert.secret {#_specreceiverswechatconfigshttpconfigtlsconfigcertsecret}

Description
: secret defines the Secret containing data to use for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.receivers[].wechatConfigs[].httpConfig.tlsConfig.keySecret {#_specreceiverswechatconfigshttpconfigtlsconfigkeysecret}

Description
: keySecret defines the Secret containing the client key file for the targets.

Type
: ```
  `object`
  ```

Required
: - `key`

| Property | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key of the secret to select from.  Must be a valid secret key. |
| `name` | `string` | Name of the referent. This field is effectively required, but due to backwards compatibility is allowed to be empty. Instances of this type with an empty value here are almost certainly wrong. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names |
| `optional` | `boolean` | Specify whether the Secret or its key must be defined |

### .spec.route {#_specroute}

Description
: route defines the Alertmanager route definition for alerts matching the resource’s namespace. If present, it will be added to the generated Alertmanager configuration as a first-level route.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `activeTimeIntervals` | `array (string)` | activeTimeIntervals is a list of TimeInterval names when this route should be active. |
| `continue` | `boolean` | continue defines the boolean indicating whether an alert should continue matching subsequent sibling nodes. It will always be overridden to true for the first-level route by the Prometheus operator. |
| `groupBy` | `array (string)` | groupBy defines the list of labels to group by. Labels must not be repeated (unique list). Special label "..." (aggregate by all possible labels), if provided, must be the only element in the list. |
| `groupInterval` | `string` | groupInterval defines how long to wait before sending an updated notification. Must be greater than 0. Example: "5m" |
| `groupWait` | `string` | groupWait defines how long to wait before sending the initial notification. Example: "30s" |
| `matchers` | `array` | matchers defines the list of matchers that the alert’s labels should match. For the first level route, the operator removes any existing equality and regexp matcher on the `namespace` label and adds a `namespace: &lt;object namespace>` matcher. |
| `matchers[]` | `object` | Matcher defines how to match on alert’s labels. |
| `muteTimeIntervals` | `array (string)` | muteTimeIntervals is a list of MuteTimeInterval names that will mute this route when matched, |
| `receiver` | `string` | receiver defines the name of the receiver for this route. If not empty, it should be listed in the `receivers` field. |
| `repeatInterval` | `string` | repeatInterval defines how long to wait before repeating the last notification. Must be greater than 0. Example: "4h" |
| `routes` | `array (undefined)` | routes defines the child routes. |

### .spec.route.matchers {#_specroutematchers}

Description
: matchers defines the list of matchers that the alert’s labels should match. For the first level route, the operator removes any existing equality and regexp matcher on the `namespace` label and adds a `namespace: &lt;object   namespace>` matcher.

Type
: ```
  `array`
  ```

### .spec.route.matchers[] {#_specroutematchers}

Description
: Matcher defines how to match on alert’s labels.

Type
: ```
  `object`
  ```

Required
: - `name`

| Property | Type | Description |
| --- | --- | --- |
| `matchType` | `string` | matchType defines the match operation available with AlertManager >= v0.22.0. Takes precedence over Regex (deprecated) if non-empty. Valid values: "=" (equality), "!=" (inequality), "=~" (regex match), "!~" (regex non-match). |
| `name` | `string` | name defines the label to match. This specifies which alert label should be evaluated. |
| `value` | `string` | value defines the label value to match. This is the expected value for the specified label. |

### .spec.timeIntervals {#_spectimeintervals}

Description
: timeIntervals defines the list of timeIntervals specifying when the routes should be muted.

Type
: ```
  `array`
  ```

### .spec.timeIntervals[] {#_spectimeintervals}

Description
: TimeInterval specifies the periods in time when notifications will be muted or active.

Type
: ```
  `object`
  ```

Required
: - `name`

| Property | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the time interval. |
| `timeIntervals` | `array` | timeIntervals defines a list of TimePeriod. |
| `timeIntervals[]` | `object` | TimePeriod describes periods of time. |

### .spec.timeIntervals[].timeIntervals {#_spectimeintervalstimeintervals}

Description
: timeIntervals defines a list of TimePeriod.

Type
: ```
  `array`
  ```

### .spec.timeIntervals[].timeIntervals[] {#_spectimeintervalstimeintervals}

Description
: TimePeriod describes periods of time.

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `daysOfMonth` | `array` | daysOfMonth defines a list of DayOfMonthRange |
| `daysOfMonth[]` | `object` | DayOfMonthRange is an inclusive range of days of the month beginning at 1 |
| `months` | `array (string)` | months defines a list of MonthRange |
| `times` | `array` | times defines a list of TimeRange |
| `times[]` | `object` | TimeRange defines a start and end time in 24hr format |
| `weekdays` | `array (string)` | weekdays defines a list of WeekdayRange |
| `years` | `array (string)` | years defines a list of YearRange |

### .spec.timeIntervals[].timeIntervals[].daysOfMonth {#_spectimeintervalstimeintervalsdaysofmonth}

Description
: daysOfMonth defines a list of DayOfMonthRange

Type
: ```
  `array`
  ```

### .spec.timeIntervals[].timeIntervals[].daysOfMonth[] {#_spectimeintervalstimeintervalsdaysofmonth}

Description
: DayOfMonthRange is an inclusive range of days of the month beginning at 1

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `end` | `integer` | end of the inclusive range |
| `start` | `integer` | start of the inclusive range |

### .spec.timeIntervals[].timeIntervals[].times {#_spectimeintervalstimeintervalstimes}

Description
: times defines a list of TimeRange

Type
: ```
  `array`
  ```

### .spec.timeIntervals[].timeIntervals[].times[] {#_spectimeintervalstimeintervalstimes}

Description
: TimeRange defines a start and end time in 24hr format

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `endTime` | `string` | endTime defines the end time in 24hr format. |
| `startTime` | `string` | startTime defines the start time in 24hr format. |

### .status {#_status}

Description
: status defines the status subresource. It is under active development and is updated only when the "StatusForConfigurationResources" feature gate is enabled.

  Most recent observed status of the ServiceMonitor. Read-only. More info: https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

Type
: ```
  `object`
  ```

| Property | Type | Description |
| --- | --- | --- |
| `bindings` | `array` | bindings defines the list of workload resources (Prometheus, PrometheusAgent, ThanosRuler or Alertmanager) which select the configuration resource. |
| `bindings[]` | `object` | WorkloadBinding is a link between a configuration resource and a workload resource. |

### .status.bindings {#_statusbindings}

Description
: bindings defines the list of workload resources (Prometheus, PrometheusAgent, ThanosRuler or Alertmanager) which select the configuration resource.

Type
: ```
  `array`
  ```

### .status.bindings[] {#_statusbindings}

Description
: WorkloadBinding is a link between a configuration resource and a workload resource.

Type
: ```
  `object`
  ```

Required
: - `group`
  - `name`
  - `namespace`
  - `resource`

| Property | Type | Description |
| --- | --- | --- |
| `conditions` | `array` | conditions defines the current state of the configuration resource when bound to the referenced Workload object. |
| `conditions[]` | `object` | ConfigResourceCondition describes the status of configuration resources linked to Prometheus, PrometheusAgent, Alertmanager or ThanosRuler. |
| `group` | `string` | group defines the group of the referenced resource. |
| `name` | `string` | name defines the name of the referenced object. |
| `namespace` | `string` | namespace defines the namespace of the referenced object. |
| `resource` | `string` | resource defines the type of resource being referenced (e.g. Prometheus, PrometheusAgent, ThanosRuler or Alertmanager). |

### .status.bindings[].conditions {#_statusbindingsconditions}

Description
: conditions defines the current state of the configuration resource when bound to the referenced Workload object.

Type
: ```
  `array`
  ```

### .status.bindings[].conditions[] {#_statusbindingsconditions}

Description
: ConfigResourceCondition describes the status of configuration resources linked to Prometheus, PrometheusAgent, Alertmanager or ThanosRuler.

Type
: ```
  `object`
  ```

Required
: - `lastTransitionTime`
  - `status`
  - `type`

| Property | Type | Description |
| --- | --- | --- |
| `lastTransitionTime` | `string` | lastTransitionTime defines the time of the last update to the current status property. |
| `message` | `string` | message defines the human-readable message indicating details for the condition’s last transition. |
| `observedGeneration` | `integer` | observedGeneration defines the .metadata.generation that the condition was set based upon. For instance, if `.metadata.generation` is currently 12, but the `.status.conditions[].observedGeneration` is 9, the condition is out of date with respect to the current state of the object. |
| `reason` | `string` | reason for the condition’s last transition. |
| `status` | `string` | status of the condition. |
| `type` | `string` | type of the condition being reported. Currently, only "Accepted" is supported. |

## API endpoints {#_api_endpoints}

The following API endpoints are available:

- `/apis/monitoring.coreos.com/v1beta1/alertmanagerconfigs`
  - `GET`: list objects of kind AlertmanagerConfig
- `/apis/monitoring.coreos.com/v1beta1/namespaces/{namespace}/alertmanagerconfigs`
  - `DELETE`: delete collection of AlertmanagerConfig
  - `GET`: list objects of kind AlertmanagerConfig
  - `POST`: create an AlertmanagerConfig
- `/apis/monitoring.coreos.com/v1beta1/namespaces/{namespace}/alertmanagerconfigs/{name}`
  - `DELETE`: delete an AlertmanagerConfig
  - `GET`: read the specified AlertmanagerConfig
  - `PATCH`: partially update the specified AlertmanagerConfig
  - `PUT`: replace the specified AlertmanagerConfig
- `/apis/monitoring.coreos.com/v1beta1/namespaces/{namespace}/alertmanagerconfigs/{name}/status`
  - `GET`: read status of the specified AlertmanagerConfig
  - `PATCH`: partially update status of the specified AlertmanagerConfig
  - `PUT`: replace status of the specified AlertmanagerConfig

### /apis/monitoring.coreos.com/v1beta1/alertmanagerconfigs {#_apismonitoringcoreoscomv1beta1alertmanagerconfigs}

HTTP method
: ```
  `GET`
  ```

Description
: ```
  list objects of kind AlertmanagerConfig
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`AlertmanagerConfigList`](/docs/rest_api/objects#com-coreos-monitoring-v1beta1-AlertmanagerConfigList) schema |
| 401 - Unauthorized | Empty |

### /apis/monitoring.coreos.com/v1beta1/namespaces/{namespace}/alertmanagerconfigs {#_apismonitoringcoreoscomv1beta1namespaces_namespace_alertmanagerconfigs}

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete collection of AlertmanagerConfig
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Status`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-Status) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  list objects of kind AlertmanagerConfig
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`AlertmanagerConfigList`](/docs/rest_api/objects#com-coreos-monitoring-v1beta1-AlertmanagerConfigList) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `POST`
  ```

Description
: ```
  create an AlertmanagerConfig
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`AlertmanagerConfig`](/docs/rest_api/monitoring_apis/alertmanagerconfig-monitoring-coreos-com-v1beta1#alertmanagerconfig-monitoring-coreos-com-v1beta1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`AlertmanagerConfig`](/docs/rest_api/monitoring_apis/alertmanagerconfig-monitoring-coreos-com-v1beta1#alertmanagerconfig-monitoring-coreos-com-v1beta1) schema |
| 201 - Created | [`AlertmanagerConfig`](/docs/rest_api/monitoring_apis/alertmanagerconfig-monitoring-coreos-com-v1beta1#alertmanagerconfig-monitoring-coreos-com-v1beta1) schema |
| 202 - Accepted | [`AlertmanagerConfig`](/docs/rest_api/monitoring_apis/alertmanagerconfig-monitoring-coreos-com-v1beta1#alertmanagerconfig-monitoring-coreos-com-v1beta1) schema |
| 401 - Unauthorized | Empty |

### /apis/monitoring.coreos.com/v1beta1/namespaces/{namespace}/alertmanagerconfigs/{name} {#_apismonitoringcoreoscomv1beta1namespaces_namespace_alertmanagerconfigs_name}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the AlertmanagerConfig |

HTTP method
: ```
  `DELETE`
  ```

Description
: ```
  delete an AlertmanagerConfig
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`Status`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-Status) schema |
| 202 - Accepted | [`Status`](/docs/rest_api/objects#io-k8s-apimachinery-pkg-apis-meta-v1-Status) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read the specified AlertmanagerConfig
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`AlertmanagerConfig`](/docs/rest_api/monitoring_apis/alertmanagerconfig-monitoring-coreos-com-v1beta1#alertmanagerconfig-monitoring-coreos-com-v1beta1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update the specified AlertmanagerConfig
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`AlertmanagerConfig`](/docs/rest_api/monitoring_apis/alertmanagerconfig-monitoring-coreos-com-v1beta1#alertmanagerconfig-monitoring-coreos-com-v1beta1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace the specified AlertmanagerConfig
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`AlertmanagerConfig`](/docs/rest_api/monitoring_apis/alertmanagerconfig-monitoring-coreos-com-v1beta1#alertmanagerconfig-monitoring-coreos-com-v1beta1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`AlertmanagerConfig`](/docs/rest_api/monitoring_apis/alertmanagerconfig-monitoring-coreos-com-v1beta1#alertmanagerconfig-monitoring-coreos-com-v1beta1) schema |
| 201 - Created | [`AlertmanagerConfig`](/docs/rest_api/monitoring_apis/alertmanagerconfig-monitoring-coreos-com-v1beta1#alertmanagerconfig-monitoring-coreos-com-v1beta1) schema |
| 401 - Unauthorized | Empty |

### /apis/monitoring.coreos.com/v1beta1/namespaces/{namespace}/alertmanagerconfigs/{name}/status {#_apismonitoringcoreoscomv1beta1namespaces_namespace_alertmanagerconfigs_name_status}

**Global path parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | name of the AlertmanagerConfig |

HTTP method
: ```
  `GET`
  ```

Description
: ```
  read status of the specified AlertmanagerConfig
  ```

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`AlertmanagerConfig`](/docs/rest_api/monitoring_apis/alertmanagerconfig-monitoring-coreos-com-v1beta1#alertmanagerconfig-monitoring-coreos-com-v1beta1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PATCH`
  ```

Description
: ```
  partially update status of the specified AlertmanagerConfig
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`AlertmanagerConfig`](/docs/rest_api/monitoring_apis/alertmanagerconfig-monitoring-coreos-com-v1beta1#alertmanagerconfig-monitoring-coreos-com-v1beta1) schema |
| 401 - Unauthorized | Empty |

HTTP method
: ```
  `PUT`
  ```

Description
: ```
  replace status of the specified AlertmanagerConfig
  ```

**Query parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `dryRun` | `string` | When present, indicates that modifications should not be persisted. An invalid or unrecognized dryRun directive will result in an error response and no further processing of the request. Valid values are: - All: all dry run stages will be processed |
| `fieldValidation` | `string` | fieldValidation instructs the server on how to handle objects in the request (POST/PUT/PATCH) containing unknown or duplicate fields. Valid values are: - Ignore: This will ignore any unknown fields that are silently dropped from the object, and will ignore all but the last duplicate field that the decoder encounters. This is the default behavior prior to v1.23. - Warn: This will send a warning via the standard warning response header for each unknown field that is dropped from the object, and for each duplicate field that is encountered. The request will still succeed if there are no other errors, and will only persist the last of any duplicate fields. This is the default in v1.23+ - Strict: This will fail the request with a BadRequest error if any unknown fields would be dropped from the object, or if any duplicate fields are present. The error returned from the server will contain all unknown and duplicate fields encountered. |

**Body parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `body` | [`AlertmanagerConfig`](/docs/rest_api/monitoring_apis/alertmanagerconfig-monitoring-coreos-com-v1beta1#alertmanagerconfig-monitoring-coreos-com-v1beta1) schema |  |

**HTTP responses**

| HTTP code | Reponse body |
| --- | --- |
| 200 - OK | [`AlertmanagerConfig`](/docs/rest_api/monitoring_apis/alertmanagerconfig-monitoring-coreos-com-v1beta1#alertmanagerconfig-monitoring-coreos-com-v1beta1) schema |
| 201 - Created | [`AlertmanagerConfig`](/docs/rest_api/monitoring_apis/alertmanagerconfig-monitoring-coreos-com-v1beta1#alertmanagerconfig-monitoring-coreos-com-v1beta1) schema |
| 401 - Unauthorized | Empty |
