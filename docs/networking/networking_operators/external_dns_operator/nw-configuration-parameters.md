---
title: External DNS Operator configuration parameters
sidebar_position: 4
---

# External DNS Operator configuration parameters

<a id="external-dns-operator-configuration-parameters"></a>

To customize the behavior of the External DNS Operator, configure the available parameters in the `ExternalDNS` custom resource (CR). By configuraing parameters, you can control how the Operator synchronizes services and routes with your external DNS provider.

## External DNS Operator configuration parameters {#nw-external-dns-operator-configuration-parameters_external-dns-operator-configuration-parameters}

To customize the behavior of the External DNS Operator, configure the available parameters in the `ExternalDNS` custom resource (CR).

<table>
<thead>
<tr>
  <th>Parameter</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>spec</code></td>
  <td>Enables the type of a cloud provider.<br><br><pre><code>spec:&#10;  provider:&#10;    type: AWS&#10;    aws:&#10;      credentials:&#10;        name: aws-access-key</code></pre><ul><li><code>provider.type</code>: Specifies available options such as AWS, Google Cloud, Azure, and Infoblox.</li><li><code>provider.aws.credentials.name</code>: Specifies a secret name for your cloud provider.</li></ul></td>
</tr>
<tr>
  <td><code>zones</code></td>
  <td>Enables you to specify DNS zones by their domains. If you do not specify zones, the <code>ExternalDNS</code> resource discovers all of the zones present in your cloud provider account.<br><br><pre><code>zones:&#10;- "&lt;zone_id&gt;"</code></pre><ul><li><code>&lt;zone_id&gt;</code>: Specifies the name of DNS zones.</li></ul></td>
</tr>
<tr>
  <td><code>domains</code></td>
  <td>Enables you to specify AWS zones by their domains. If you do not specify domains, the <code>ExternalDNS</code> resource discovers all of the zones present in your cloud provider account.<br><br><pre><code>domains:&#10;- filterType: Include&#10;  matchType: Exact&#10;  name: "myzonedomain1.com"&#10;- filterType: Include&#10;  matchType: Pattern&#10;  pattern: ".*\\.otherzonedomain\\.com"</code></pre><ul><li><code>domains.filterType</code>: Specifies that the <code>ExternalDNS</code> resource includes the domain name.</li><li><code>domains.matchType</code>: Specifies that the domain matching has to be exact as opposed to regular expression match.</li><li><code>domains.name</code>: Specifies the name of the domain.</li><li><code>filterType.matchType</code>: Specifies the <code>regex-domain-filter</code> flag in the <code>ExternalDNS</code> resource. You can limit possible domains by using a Regex filter.</li><li><code>filterType.pattern</code>: Specifies the regex pattern to be used by the <code>ExternalDNS</code> resource to filter the domains of the target zones.</li></ul></td>
</tr>
<tr>
  <td><code>source</code></td>
  <td>Enables you to specify the source for the DNS records, <code>Service</code> or <code>Route</code>.<br><br><pre><code>source:&#10;  type: Service&#10;  service:&#10;    serviceType:&#10;      - LoadBalancer&#10;      - ClusterIP&#10;  labelFilter:&#10;    matchLabels:&#10;      external-dns.mydomain.org/publish: "yes"&#10;  hostnameAnnotation: "Allow"&#10;  fqdnTemplate:&#10;  - "{.Name}.myzonedomain.com"</code></pre><ul><li><code>source</code>: Specifies the settings for the source of DNS records.</li><li><code>source.type</code>: Specifies that the <code>ExternalDNS</code> CR uses the <code>Service</code> type as the source for creating DNS records.</li><li><code>service.serviceType</code>: Specifies the <code>service-type-filter</code> flag in the <code>ExternalDNS</code> resource. The <code>serviceType</code> contains the following fields: <code>default</code>: <code>LoadBalancer</code>; <code>expected</code>: <code>ClusterIP</code>; <code>NodePort</code>; <code>LoadBalancer</code>; <code>ExternalName</code>.</li><li><code>service.labelFilter</code>: Specifies that the controller considers only those resources that match with label filter.</li><li><code>hostnameAnnotation</code>: Specifies that the default value for <code>hostnameAnnotation</code> is <code>Ignore</code> which instructs <code>ExternalDNS</code> to generate DNS records by using the templates specified in the field <code>fqdnTemplates</code>. When the value is <code>Allow</code> the DNS records get generated based on the value specified in the <code>external-dns.alpha.kubernetes.io/hostname</code> annotation.</li><li><code>fqdnTemplate</code>: Specifies that the External DNS Operator uses a string to generate DNS names from sources that do not define a hostname, or to add a hostname suffix when paired with the fake source.</li></ul><pre><code>source:&#10;  type: OpenShiftRoute&#10;  openshiftRouteOptions:&#10;    routerName: default&#10;    labelFilter:&#10;      matchLabels:&#10;        external-dns.mydomain.org/publish: "yes"</code></pre><ul><li><code>source.type</code>: Specifies the creation of DNS records.</li><li><code>openshiftRouteOptions.routerName</code>: Specifies if the source type is <code>OpenShiftRoute</code>. If so, you can pass the Ingress Controller name. The <code>ExternalDNS</code> resource uses the canonical name of the Ingress Controller as the target for CNAME records.</li></ul></td>
</tr>
</tbody>
</table>
