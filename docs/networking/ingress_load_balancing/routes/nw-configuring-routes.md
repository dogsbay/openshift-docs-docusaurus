---
title: Configuring routes
sidebar_position: 3
---

# Configuring routes {#nw-configuring-routes}

<a id="nw-configuring-routes"></a>

To customise route configuration for specific traffic behaviors, apply annotations, headers, and cookies. By using these mechanisms, you can define granular routing rules, extending standard capabilities to meet complex application requirements.

## Configure route timeouts {#nw-configuring-route-timeouts_configuring-routes}

You can configure the default timeouts for an existing route when you have services in need of a low timeout, which is required for Service Level Availability (SLA) purposes, or a high timeout, for cases with a slow back end.

:::warning

If you configured a user-managed external load balancer in front of your OpenShift Container Platform cluster, ensure that the timeout value for the user-managed external load balancer is higher than the timeout value for the route. This configuration prevents network congestion issues over the network that your cluster uses.

:::

**Prerequisites**

- You deployed an Ingress Controller on a running cluster.

**Procedure**

- Using the `oc annotate` command, add the timeout to the route:
  ```terminal
  $ oc annotate route <route_name> \
      --overwrite haproxy.router.openshift.io/timeout=<timeout><time_unit>
  ```
- `<timeout>`: Supported time units are microseconds (us), milliseconds (ms), seconds (s), minutes (m), hours (h), or days (d).
  The following example sets a timeout of two seconds on a route named `myroute`:

  ```terminal
  $ oc annotate route myroute --overwrite haproxy.router.openshift.io/timeout=2s
  ```

## HTTP header configuration {#nw-http-header-configuration_configuring-routes}

To customize request and response headers for your applications, configure the Ingress Controller or apply specific route annotations. Understanding the interaction between these configuration methods ensures you effectively manage global and route-specific header policies.

You can also set certain headers by using route annotations. The various ways of configuring headers can present challenges when working together.

:::note

You can only set or delete headers within an `IngressController` or `Route` CR, you cannot append them. If an HTTP header is set with a value, that value must be complete and not require appending in the future. In situations where it makes sense to append a header, such as the X-Forwarded-For header, use the `spec.httpHeaders.forwardedHeaderPolicy` field, instead of `spec.httpHeaders.actions`.

:::

Order of precedence
: When the same HTTP header is modified both in the Ingress Controller and in a route, HAProxy prioritizes the actions in certain ways depending on whether it is a request or response header.

  - For HTTP response headers, actions specified in the Ingress Controller are executed after the actions specified in a route. This means that the actions specified in the Ingress Controller take precedence.
  - For HTTP request headers, actions specified in a route are executed after the actions specified in the Ingress Controller. This means that the actions specified in the route take precedence.

For example, a cluster administrator sets the X-Frame-Options response header with the value `DENY` in the Ingress Controller using the following configuration:

```yaml title="Example IngressController spec"
apiVersion: operator.openshift.io/v1
kind: IngressController
# ...
spec:
  httpHeaders:
    actions:
      response:
      - name: X-Frame-Options
        action:
          type: Set
          set:
            value: DENY
```

A route owner sets the same response header that the cluster administrator set in the Ingress Controller, but with the value `SAMEORIGIN` using the following configuration:

```yaml title="Example Route spec"
apiVersion: route.openshift.io/v1
kind: Route
# ...
spec:
  httpHeaders:
    actions:
      response:
      - name: X-Frame-Options
        action:
          type: Set
          set:
            value: SAMEORIGIN
```

When both the `IngressController` spec and `Route` spec are configuring the X-Frame-Options response header, then the value set for this header at the global level in the Ingress Controller takes precedence, even if a specific route allows frames. For a request header, the `Route` spec value overrides the `IngressController` spec value.

This prioritization occurs because the `haproxy.config` file uses the following logic, where the Ingress Controller is considered the front end and individual routes are considered the back end. The header value `DENY` applied to the front end configurations overrides the same header with the value `SAMEORIGIN` that is set in the back end:

```text
frontend public
  http-response set-header X-Frame-Options 'DENY'

frontend fe_sni
  http-response set-header X-Frame-Options 'DENY'

frontend fe_no_sni
  http-response set-header X-Frame-Options 'DENY'

backend be_secure:openshift-monitoring:alertmanager-main
  http-response set-header X-Frame-Options 'SAMEORIGIN'
```

Additionally, any actions defined in either the Ingress Controller or a route override values set using route annotations.

Special case headers
: The following headers are either prevented entirely from being set or deleted, or allowed under specific circumstances:

**Special case header configuration options**

<table>
<thead>
<tr>
  <th>Header name</th>
  <th>Configurable using <code>IngressController</code> spec</th>
  <th>Configurable using <code>Route</code> spec</th>
  <th>Reason for disallowment</th>
  <th>Configurable using another method</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>proxy</code></td>
  <td>No</td>
  <td>No</td>
  <td>The <code>proxy</code> HTTP request header can be used to exploit vulnerable CGI applications by injecting the header value into the <code>HTTP_PROXY</code> environment variable. The <code>proxy</code> HTTP request header is also non-standard and prone to error during configuration.</td>
  <td>No</td>
</tr>
<tr>
  <td><code>host</code></td>
  <td>No</td>
  <td>Yes</td>
  <td>When the <code>host</code> HTTP request header is set using the <code>IngressController</code> CR, HAProxy can fail when looking up the correct route.</td>
  <td>No</td>
</tr>
<tr>
  <td><code>strict-transport-security</code></td>
  <td>No</td>
  <td>No</td>
  <td>The <code>strict-transport-security</code> HTTP response header is already handled using route annotations and does not need a separate implementation.</td>
  <td>Yes: the <code>haproxy.router.openshift.io/hsts_header</code> route annotation</td>
</tr>
<tr>
  <td><code>cookie</code> and <code>set-cookie</code></td>
  <td>No</td>
  <td>No</td>
  <td>The cookies that HAProxy sets are used for session tracking to map client connections to particular back-end servers. Allowing these headers to be set could interfere with HAProxy's session affinity and restrict HAProxy's ownership of a cookie.</td>
  <td>Yes:<br><br><ul><li>the <code>haproxy.router.openshift.io/disable_cookie</code> route annotation</li><li>the <code>haproxy.router.openshift.io/cookie_name</code> route annotation</li></ul></td>
</tr>
</tbody>
</table>

## Set or delete HTTP request and response headers in a route {#nw-route-set-or-delete-http-headers_configuring-routes}

You can set or delete certain HTTP request and response headers for compliance purposes or other reasons. You can set or delete these headers either for all routes served by an Ingress Controller or for specific routes.

For example, you might want to enable a web application to serve content in alternate locations for specific routes if that content is written in multiple languages, even if there is a default global location specified by the Ingress Controller serving the routes.

The following procedure creates a route that sets the Content-Location HTTP request header so that the URL associated with the application, `https://app.example.com`, directs to the location `https://app.example.com/lang/en-us`. Directing application traffic to this location means that anyone using that specific route is accessing web content written in American English.

**Prerequisites**

- You have installed the OpenShift CLI (`oc`).
- You are logged into an OpenShift Container Platform cluster as a project administrator.
- You have a web application that exposes a port and an HTTP or TLS endpoint listening for traffic on the port.

**Procedure**

1. Create a route definition and save it in a file called `app-example-route.yaml`:
   ```yaml title="YAML definition of the created route with HTTP header directives"
   apiVersion: route.openshift.io/v1
   kind: Route
   # ...
   spec:
     host: app.example.com
     tls:
       termination: edge
     to:
       kind: Service
       name: app-example
     httpHeaders:
       actions:
         response:
         - name: Content-Location
           action:
             type: Set
             set:
               value: /lang/en-us
   # ...
   ```

   where:

   <dl>
   <dt><code>actions</code></dt>
   <dd>Specifies the list of actions you want to perform on the HTTP headers.</dd>
   <dt><code>response</code></dt>
   <dd>Specifies the type of header you want to change. In this case, a response header.</dd>
   <dt><code>response.name</code></dt>
   <dd>Specifies the name of the header you want to change. For a list of available headers you can set or delete, see <em>HTTP header configuration</em>.</dd>
   <dt><code>action.type</code></dt>
   <dd>Specifies the type of action being taken on the header. This field can have the value <code>Set</code> or <code>Delete</code>.</dd>
   <dt><code>set.value</code></dt>
   <dd>When setting HTTP headers, you must provide a <code>value</code>. The value can be a string from a list of available directives for that header, for example <code>DENY</code>, or it can be a dynamic value that will be interpreted using HAProxy’s dynamic value syntax. In this case, the value is set to the relative location of the content.</dd>
   </dl>
2. Create a route to your existing web application using the newly created route definition:
   ```terminal
   $ oc -n app-example create -f app-example-route.yaml
   ```

   For HTTP request headers, the actions specified in the route definitions are executed after any actions performed on HTTP request headers in the Ingress Controller. This means that any values set for those request headers in a route will take precedence over the ones set in the Ingress Controller. For more information on the processing order of HTTP headers, see *HTTP header configuration*.

## Use cookies to keep route statefulness {#nw-using-cookies-keep-route-statefulness_configuring-routes}

To maintain stateful application traffic during pod restarts or scaling events, configure sticky sessions by using cookies. By using this method, you ensure that all incoming traffic reaches the same endpoint, preventing state loss even if the specific endpoint pod changes.

OpenShift Container Platform can use cookies to configure session persistence. The Ingress Controller selects an endpoint to handle any user requests, and creates a cookie for the session. The cookie is passed back in the response to the request and the user sends the cookie back with the next request in the session. The cookie tells the Ingress Controller which endpoint is handling the session, ensuring that client requests use the cookie so that they are routed to the same pod.

:::note

Cookies cannot be set on passthrough routes, because the HTTP traffic cannot be seen. Instead, a number is calculated based on the source IP address, which determines the backend.

If backends change, the traffic can be directed to the wrong server, making it less sticky. If you are using a load balancer, which hides source IP, the same number is set for all connections and traffic is sent to the same pod.

:::

### Annotate a route with a cookie {#nw-annotating-a-route-with-a-cookie-name_configuring-routes}

To enable applications to manage session persistence and load distribution, annotate the route with a custom cookie name. Overwriting the default cookie allows the backend application to identify and delete the specific cookie, forcing endpoint re-selection when necessary.

When a server is overloaded, the server tries to remove the requests from the client and redistribute the requests to other endpoints.

**Procedure**

1. Annotate the route with the specified cookie name:
   ```terminal
   $ oc annotate route <route_name> router.openshift.io/cookie_name="<cookie_name>"
   ```

   where:

   <dl>
   <dt><code>&lt;route_name&gt;</code></dt>
   <dd>Specifies the name of the route.</dd>
   <dt><code>&lt;cookie_name&gt;</code></dt>
   <dd>Specifies the name for the cookie. For example, to annotate the route <code>my_route</code> with the cookie name <code>my_cookie</code>:</dd>
   </dl>

   ```terminal
   $ oc annotate route my_route router.openshift.io/cookie_name="my_cookie"
   ```
2. Capture the route hostname in a variable:
   ```terminal
   $ ROUTE_NAME=$(oc get route <route_name> -o jsonpath='{.spec.host}')
   ```

   where:

   <dl>
   <dt><code>&lt;route_name&gt;</code></dt>
   <dd>Specifies the name of the route.</dd>
   </dl>
3. Save the cookie, and then access the route:
   ```terminal
   $ curl $ROUTE_NAME -k -c /tmp/cookie_jar
   ```

   Use the cookie saved by the previous command when connecting to the route:

   ```terminal
   $ curl $ROUTE_NAME -k -b /tmp/cookie_jar
   ```

## Route-specific annotations {#nw-route-specific-annotations_configuring-routes}

The Ingress Controller can set the default options for all the routes it exposes. An individual route can override some of these defaults by providing specific configurations in its annotations. Red Hat does not support adding a route annotation to an operator-managed route.

:::warning

To create an allow list with multiple source IPs or subnets, use a space-delimited list. Any other delimiter type causes the list to be ignored without a warning or error message.

:::

**Route annotations**

| Variable | Description |
| --- | --- |
| `haproxy.router.openshift.io/balance` | Sets the load-balancing algorithm. Available options are `random`, `source`, `roundrobin`[<sup>1</sup>], and `leastconn`.  The default value is `source` for TLS passthrough routes. For all other routes, the default is `random`. |
| `haproxy.router.openshift.io/disable_cookies` | Disables the use of cookies to track related connections. If set to `'true'` or `'TRUE'`, the balance algorithm is used to choose which back-end serves connections for each incoming HTTP request. |
| `router.openshift.io/cookie_name` | Specifies an optional cookie to use for this route. The name must consist of any combination of upper and lower case letters, digits, "_", and "-". The default is the hashed internal key name for the route. |
| `haproxy.router.openshift.io/pod-concurrent-connections` | Sets the maximum number of connections that are allowed to a backing pod from a router. Note: If there are multiple pods, each can have this many connections. If you have multiple routers, there is no coordination among them, each may connect this many times. If not set, or set to 0, there is no limit. |
| `haproxy.router.openshift.io/rate-limit-connections` | Setting `'true'` or `'TRUE'` enables rate limiting functionality which is implemented through stick-tables on the specific backend per route. Note: Using this annotation provides basic protection against denial-of-service attacks. |
| `haproxy.router.openshift.io/rate-limit-connections.concurrent-tcp` | Limits the number of concurrent TCP connections made through the same source IP address. It accepts a numeric value. Note: Using this annotation provides basic protection against denial-of-service attacks. |
| `haproxy.router.openshift.io/rate-limit-connections.rate-http` | Limits the rate at which a client with the same source IP address can make HTTP requests. It accepts a numeric value. Note: Using this annotation provides basic protection against denial-of-service attacks. |
| `haproxy.router.openshift.io/rate-limit-connections.rate-tcp` | Limits the rate at which a client with the same source IP address can make TCP connections. It accepts a numeric value. |
| `router.openshift.io/haproxy.health.check.interval` | Sets the interval for the back-end health checks. (TimeUnits) |
| `haproxy.router.openshift.io/ip_allowlist` | Sets an allowlist for the route. The allowlist is a space-separated list of IP addresses and CIDR ranges for the approved source addresses. Requests from IP addresses that are not in the allowlist are dropped.<br>The maximum number of IP addresses and CIDR ranges directly visible in the `haproxy.config` file is 61. [<sup>2</sup>] |
| `haproxy.router.openshift.io/hsts_header` | Sets a Strict-Transport-Security header for the edge terminated or re-encrypt route. |
| `haproxy.router.openshift.io/rewrite-target` | Sets the rewrite path of the request on the backend. |
| `router.openshift.io/cookie-same-site` | Sets a value to restrict cookies. The values are:<br>`Lax`: the browser does not send cookies on cross-site requests, but does send cookies when users navigate to the origin site from an external site. This is the default browser behavior when the `SameSite` value is not specified.<br>`Strict`: the browser sends cookies only for same-site requests.<br>`None`: the browser sends cookies for both cross-site and same-site requests.<br>This value is applicable to re-encrypt and edge routes only. |
| `haproxy.router.openshift.io/set-forwarded-headers` | Sets the policy for handling the `Forwarded` and `X-Forwarded-For` HTTP headers per route. The values are:<br>`append`: appends the header, preserving any existing header. This is the default value.<br>`replace`: sets the header, removing any existing header.<br>`never`: never sets the header, but preserves any existing header.<br>`if-none`: sets the header if it is not already set. |

1. By default, the router reloads every 5 s which resets the balancing connection across pods from the beginning. As a result, the `roundrobin` state is not preserved across reloads. This algorithm works best when pods have nearly identical computing capabilites and storage capacity. If your application or service has continuously changing endpoints, for example, due to the use of a CI/CD pipeline, uneven balancing can result. In this case, use a different algorithm.
2. If the number of IP addresses and CIDR ranges in an allowlist exceeds 61, they are written into a separate file that is then referenced from the `haproxy.config` file. This file is stored in the `/var/lib/haproxy/router/allowlists` folder.
   :::note

   To ensure that the addresses are written to the allowlist, check that the full list of CIDR ranges are listed in the Ingress Controller configuration file. The etcd object size limit restricts how large a route annotation can be. Because of this, it creates a threshold for the maximum number of IP addresses and CIDR ranges that you can include in an allowlist.

   :::

```yaml title="A route that allows only one specific IP address"
metadata:
  annotations:
    haproxy.router.openshift.io/ip_allowlist: 192.168.1.10
```

```yaml title="A route that sets the load-balancing algorithm to leastconn"
metadata:
  annotations:
    haproxy.router.openshift.io/balance: leastconn
```

- The default load-balancing algorithm is `random`.

```yaml title="A route that allows several IP addresses"
metadata:
  annotations:
    haproxy.router.openshift.io/ip_allowlist: 192.168.1.10 192.168.1.11 192.168.1.12
```

```yaml title="A route that allows an IP address CIDR network"
metadata:
  annotations:
    haproxy.router.openshift.io/ip_allowlist: 192.168.1.0/24
```

```yaml title="A route that allows both IP an address and IP address CIDR networks"
metadata:
  annotations:
    haproxy.router.openshift.io/ip_allowlist: 180.5.61.153 192.168.1.0/24 10.0.0.0/8
```

```yaml title="A route specifying a rewrite target"
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  annotations:
    haproxy.router.openshift.io/rewrite-target: /
...
```

In this example, `/` is set as the rewrite path of the request on the backend.

Setting the `haproxy.router.openshift.io/rewrite-target` annotation on a route specifies that the Ingress Controller should rewrite paths in HTTP requests using this route before forwarding the requests to the backend application. The part of the request path that matches the path specified in `spec.path` is replaced with the rewrite target specified in the annotation.

The following table provides examples of the path rewriting behavior for various combinations of `spec.path`, request path, and rewrite target.

**rewrite-target examples**

| Route.spec.path | Request path | Rewrite target | Forwarded request path |
| --- | --- | --- | --- |
| /foo | /foo | / | / |
| /foo | /foo/ | / | / |
| /foo | /foo/bar | / | /bar |
| /foo | /foo/bar/ | / | /bar/ |
| /foo | /foo | /bar | /bar |
| /foo | /foo/ | /bar | /bar/ |
| /foo | /foo/bar | /baz | /baz/bar |
| /foo | /foo/bar/ | /baz | /baz/bar/ |
| /foo/ | /foo | / | N/A (request path does not match route path) |
| /foo/ | /foo/ | / | / |
| /foo/ | /foo/bar | / | /bar |

Certain special characters in `haproxy.router.openshift.io/rewrite-target` require special handling because they must be escaped properly. Refer to the following table to understand how these characters are handled.

**Special character handling**

| For character | Use characters | Notes |
| --- | --- | --- |
| # | # | Avoid # because it terminates the rewrite expression |
| % | % or %% | Avoid odd sequences such as %%% |
| ‘ | \’ | Avoid ‘ because it is ignored |

All other valid URL characters can be used without escaping.

**Additional resources**

- [Attributes (mdn documentation)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite)

## Throughput issue troubleshooting methods {#nw-throughput-troubleshoot_configuring-routes}

To diagnose and resolve network throughput issues, such as unusually high latency between specific services, apply troubleshooting methods. Identifying connectivity bottlenecks helps ensure stable application performance within OpenShift Container Platform.

If pod logs do not reveal any cause of the problem, use the following methods to analyze performance issues:

- Use a packet analyzer, such as `ping` or `tcpdump` to analyze traffic between a pod and its node.
  For example, run the `tcpdump` tool on each pod while reproducing the behavior that led to the issue. Review the captures on both sides to compare send and receive timestamps to analyze the latency of traffic to and from a pod. Latency can occur in OpenShift Container Platform if a node interface is overloaded with traffic from other pods, storage devices, or the data plane.

  ```terminal
  $ tcpdump -s 0 -i any -w /tmp/dump.pcap host <podip 1> && host <podip 2> (1)
  ```

  where:

  <dl>
  <dt><code>podip</code></dt>
  <dd>Specifies the IP address for the pod. Run the <code>oc get pod &lt;pod_name&gt; -o wide</code> command to get the IP address of a pod. The <code>tcpdump</code> command generates a file at <code>/tmp/dump.pcap</code> containing all traffic between these two pods. You can run the analyzer shortly before the issue is reproduced and stop the analyzer shortly after the issue is finished reproducing to minimize the size of the file. You can also run a packet analyzer between the nodes with:</dd>
  </dl>

  ```terminal
  $ tcpdump -s 0 -i any -w /tmp/dump.pcap port 4789
  ```
- Use a bandwidth measuring tool, such as `iperf`, to measure streaming throughput and UDP throughput. Locate any bottlenecks by running the tool from the pods first, and then running it from the nodes.
  - For information on installing and using `iperf`, see the Red Hat Solution.
- In some cases, the cluster might mark the node with the router pod as unhealthy due to latency issues. Use worker latency profiles to adjust the frequency that the cluster waits for a status update from the node before taking action.
- If your cluster has designated lower-latency and higher-latency nodes, configure the `spec.nodePlacement` field in the Ingress Controller to control the placement of the router pod.

**Additional resources**

- [Running tcpdump inside an OpenShift pod to capture network traffic (Red Hat Knowledgebase)](https://access.redhat.com/solutions/4569211)
- [Running tcpdump from a RHEL CoreOS OpenShift node (Red Hat Knowledgebase)](https://access.redhat.com/solutions/5074041)
- [How to run iPerf network performance tests in OpenShift Container Platform 4 (Red Hat Knowledgebase)](https://access.redhat.com/solutions/6129701)
- [How to test network bandwidth? (Red Hat Knowledgebase)](https://access.redhat.com/solutions/33103)

## Configure the route admission policy {#nw-route-admission-policy_configuring-routes}

Administrators and application developers can run applications in multiple namespaces with the same domain name. This is for organizations where multiple teams develop microservices that are exposed on the same hostname.

:::warning

Allowing claims across namespaces should only be enabled for clusters with trust between namespaces, otherwise a malicious user could take over a hostname. For this reason, the default admission policy disallows hostname claims across namespaces.

:::

**Prerequisites**

- Cluster administrator privileges.

**Procedure**

- Edit the `.spec.routeAdmission` field of the `ingresscontroller` resource variable using the following command:
  ```terminal
  $ oc -n openshift-ingress-operator patch ingresscontroller/default --patch '{"spec":{"routeAdmission":{"namespaceOwnership":"InterNamespaceAllowed"}}}' --type=merge
  ```

  ```yaml title="Sample Ingress Controller configuration"
  spec:
    routeAdmission:
      namespaceOwnership: InterNamespaceAllowed
  ...
  ```

  :::tip

  You can alternatively apply the following YAML to configure the route admission policy:

  ```yaml
  apiVersion: operator.openshift.io/v1
  kind: IngressController
  metadata:
    name: default
    namespace: openshift-ingress-operator
  spec:
    routeAdmission:
      namespaceOwnership: InterNamespaceAllowed
  ```

  :::

## Configure the OpenShift Container Platform Ingress Controller for dual-stack networking {#nw-router-configuring-dual-stack_configuring-routes}

If your OpenShift Container Platform cluster is configured for IPv4 and IPv6 dual-stack networking, your cluster is externally reachable by OpenShift Container Platform routes.

The Ingress Controller automatically serves services that have both IPv4 and IPv6 endpoints, but you can configure the Ingress Controller for single-stack or dual-stack services.

**Prerequisites**

- You deployed an OpenShift Container Platform cluster on bare metal.
- You installed the OpenShift CLI (`oc`).

**Procedure**

1. To have the Ingress Controller serve traffic over IPv4/IPv6 to a workload, you can create a service YAML file or modify an existing service YAML file by setting the `ipFamilies` and `ipFamilyPolicy` fields. For example:
   ```yaml title="Sample service YAML file"
   apiVersion: v1
   kind: Service
   metadata:
     creationTimestamp: yyyy-mm-ddT00:00:00Z
     labels:
       name: <service_name>
       manager: kubectl-create
       operation: Update
       time: yyyy-mm-ddT00:00:00Z
     name: <service_name>
     namespace: <namespace_name>
     resourceVersion: "<resource_version_number>"
     selfLink: "/api/v1/namespaces/<namespace_name>/services/<service_name>"
     uid: <uid_number>
   spec:
     clusterIP: 172.30.0.0/16
     clusterIPs:
     - 172.30.0.0/16
     - <second_IP_address>
     ipFamilies:
     - IPv4
     - IPv6
     ipFamilyPolicy: RequireDualStack
     ports:
     - port: 8080
       protocol: TCP
       targetport: 8080
     selector:
       name: <namespace_name>
     sessionAffinity: None
     type: ClusterIP
   status:
     loadbalancer: {}
   ```

   where:

   <dl>
   <dt><code>spec.clusterIPs</code></dt>
   <dd>Specifies the cluster IP addresses assigned to the service. In a dual-stack instance, there are two different <code>clusterIPs</code> provided.</dd>
   </dl>

`spec.ipFamilies`
: Specifies the IP family. For a single-stack instance, enter `IPv4` or `IPv6`. For a dual-stack instance, enter both `IPv4` and `IPv6`.

`spec.ipFamilyPolicy`
: Specifies the IP family policy. For a single-stack instance, enter `SingleStack`. For a dual-stack instance, enter `RequireDualStack`. These resources generate corresponding `endpoints`. The Ingress Controller now watches `endpointslices`.

1. To view `endpoints`, enter the following command:
   ```terminal
   $ oc get endpoints
   ```
2. To view `endpointslices`, enter the following command:
   ```terminal
   $ oc get endpointslices
   ```
