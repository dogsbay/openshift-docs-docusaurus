---
title: Configuring the AWS Load Balancer Operator
sidebar_position: 5
---

# Configuring the AWS Load Balancer Operator {#configuring-aws-load-balancer-operator}

<a id="configuring-aws-load-balancer-operator"></a>

To automate the provisioning of AWS Load Balancers for your applications, configure the AWS Load Balancer Operator. This setup ensures that the Operator correctly manages ingress resources and external access to your cluster.

## Trust the certificate authority of the cluster-wide proxy {#nw-configuring-cluster-wide-proxy_aws-load-balancer-operator}

You can configure the cluster-wide proxy in the AWS Load Balancer Operator. After configuring the cluster-wide proxy, Operator Lifecycle Manager (OLM) automatically updates all the deployments of the Operators with the environment variables.

Environment variables include `HTTP_PROXY`, `HTTPS_PROXY`, and `NO_PROXY`. These variables are populated to the managed controller by the AWS Load Balancer Operator.

**Procedure**

1. Create the config map to contain the certificate authority (CA) bundle in the `aws-load-balancer-operator` namespace by running the following command:
   ```terminal
   $ oc -n aws-load-balancer-operator create configmap trusted-ca
   ```
2. To inject the trusted CA bundle into the config map, add the `config.openshift.io/inject-trusted-cabundle=true` label to the config map by running the following command:
   ```terminal
   $ oc -n aws-load-balancer-operator label cm trusted-ca config.openshift.io/inject-trusted-cabundle=true
   ```
3. Update the AWS Load Balancer Operator subscription to access the config map in the AWS Load Balancer Operator deployment by running the following command:
   ```terminal
   $ oc -n aws-load-balancer-operator patch subscription aws-load-balancer-operator --type='merge' -p '{"spec":{"config":{"env":[{"name":"TRUSTED_CA_CONFIGMAP_NAME","value":"trusted-ca"}],"volumes":[{"name":"trusted-ca","configMap":{"name":"trusted-ca"}}],"volumeMounts":[{"name":"trusted-ca","mountPath":"/etc/pki/tls/certs/albo-tls-ca-bundle.crt","subPath":"ca-bundle.crt"}]}}}'
   ```
4. After the AWS Load Balancer Operator is deployed, verify that the CA bundle is added to the `aws-load-balancer-operator-controller-manager` deployment by running the following command:
   ```terminal
   $ oc -n aws-load-balancer-operator exec deploy/aws-load-balancer-operator-controller-manager -c manager -- bash -c "ls -l /etc/pki/tls/certs/albo-tls-ca-bundle.crt; printenv TRUSTED_CA_CONFIGMAP_NAME"
   ```

   ```terminal title="Example output"
   -rw-r--r--. 1 root 1000690000 5875 Jan 11 12:25 /etc/pki/tls/certs/albo-tls-ca-bundle.crt
   trusted-ca
   ```
5. Optional: Restart deployment of the AWS Load Balancer Operator every time the config map changes by running the following command:
   ```terminal
   $ oc -n aws-load-balancer-operator rollout restart deployment/aws-load-balancer-operator-controller-manager
   ```

**Additional resources**

- [Certificate injection using Operators](/docs/networking/configuring_network_settings/configuring-a-custom-pki#certificate-injection-using-operators_configuring-a-custom-pki)

## Add TLS termination on the AWS Load Balancer {#nw-adding-tls-termination_aws-load-balancer-operator}

You can route the traffic for the domain to pods of a service and add TLS termination on the AWS Load Balancer.

**Prerequisites**

- You have access to the OpenShift CLI (`oc`).

**Procedure**

1. Create a YAML file that defines the `AWSLoadBalancerController` resource:
   ```yaml title="Example add-tls-termination-albc.yaml file"
   apiVersion: networking.olm.openshift.io/v1
   kind: AWSLoadBalancerController
   metadata:
     name: cluster
   spec:
     subnetTagging: Auto
     ingressClass: tls-termination
   # ...
   ```

   where:

   <dl>
   <dt><code>spec.ingressClass</code></dt>
   <dd>Specifies the ingress class name. If the ingress class is not present in your cluster the AWS Load Balancer Controller creates one. The AWS Load Balancer Controller reconciles the additional ingress class values if <code>spec.controller</code> is set to <code>ingress.k8s.aws/alb</code>.</dd>
   </dl>
2. Create a YAML file that defines the `Ingress` resource:
   ```yaml title="Example add-tls-termination-ingress.yaml file"
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: <example>
     annotations:
       alb.ingress.kubernetes.io/scheme: internet-facing
       alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-west-2:xxxxx
   spec:
     ingressClassName: tls-termination
     rules:
     - host: example.com
       http:
           paths:
             - path: /
               pathType: Exact
               backend:
                 service:
                   name: <example_service>
                   port:
                     number: 80
   # ...
   ```

   where:

   <dl>
   <dt><code>metadata.name</code></dt>
   <dd>Specifies the ingress name.</dd>
   <dt><code>annotations.alb.ingress.kubernetes.io/scheme</code></dt>
   <dd>Specifies the controller that provisions the load balancer for ingress. The provisioning happens in a public subnet to access the load balancer over the internet.</dd>
   <dt><code>annotations.alb.ingress.kubernetes.io/certificate-arn</code></dt>
   <dd>Specifies the Amazon Resource Name (ARN) of the certificate that you attach to the load balancer.</dd>
   <dt><code>spec.ingressClassName</code></dt>
   <dd>Specifies the ingress class name.</dd>
   <dt><code>rules.host</code></dt>
   <dd>Specifies the domain for traffic routing.</dd>
   <dt><code>backend.service</code></dt>
   <dd>Specifies the service for traffic routing.</dd>
   </dl>

## Create multiple ingress resources through a single AWS Load Balancer {#nw-creating-multiple-ingress-through-single-alb_aws-load-balancer-operator}

To route traffic to different services within a single domain, configure multiple ingress resources on a single AWS Load Balancer. This setup allows each resource to provide different endpoints while sharing the same load balancing infrastructure.

**Prerequisites**

- You have access to the OpenShift CLI (`oc`).

**Procedure**

1. Create an `IngressClassParams` resource YAML file, for example, `sample-single-lb-params.yaml`, as follows:
   ```yaml
   apiVersion: elbv2.k8s.aws/v1beta1
   kind: IngressClassParams
   metadata:
     name: single-lb-params
   spec:
     group:
       name: single-lb
   ```

   where:

   <dl>
   <dt><code>apiVersion</code></dt>
   <dd>Specifies the API group and version of the <code>IngressClassParams</code> resource.</dd>
   <dt><code>metadata.name</code></dt>
   <dd>Specifies the <code>IngressClassParams</code> resource name.</dd>
   <dt><code>spec.group.name</code></dt>
   <dd>Specifies the <code>IngressGroup</code> resource name. All of the <code>Ingress</code> resources of this class belong to this <code>IngressGroup</code>.</dd>
   </dl>
2. Create the `IngressClassParams` resource by running the following command:
   ```terminal
   $ oc create -f sample-single-lb-params.yaml
   ```
3. Create the `IngressClass` resource YAML file, for example, `sample-single-lb-class.yaml`, as follows:
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: IngressClass
   metadata:
     name: single-lb
   spec:
     controller: ingress.k8s.aws/alb
     parameters:
       apiGroup: elbv2.k8s.aws
       kind: IngressClassParams
       name: single-lb-params
   ```

   where:

   <dl>
   <dt><code>apiVersion</code></dt>
   <dd>Specifies the API group and version of the <code>IngressClass</code> resource.</dd>
   <dt><code>metadata.name</code></dt>
   <dd>Specifies the ingress class name.</dd>
   <dt><code>spec.controller</code></dt>
   <dd>Specifies the controller name. The <code>ingress.k8s.aws/alb</code> value denotes that all ingress resources of this class should be managed by the AWS Load Balancer Controller.</dd>
   <dt><code>parameters.apiGroup</code></dt>
   <dd>Specifies the API group of the <code>IngressClassParams</code> resource.</dd>
   <dt><code>parameters.kind</code></dt>
   <dd>Specifies the resource type of the <code>IngressClassParams</code> resource.</dd>
   <dt><code>parameters.name</code></dt>
   <dd>Specifies the <code>IngressClassParams</code> resource name.</dd>
   </dl>
4. Create the `IngressClass` resource by running the following command:
   ```terminal
   $ oc create -f sample-single-lb-class.yaml
   ```
5. Create the `AWSLoadBalancerController` resource YAML file, for example, `sample-single-lb.yaml`, as follows:
   ```yaml
   apiVersion: networking.olm.openshift.io/v1
   kind: AWSLoadBalancerController
   metadata:
     name: cluster
   spec:
     subnetTagging: Auto
     ingressClass: single-lb
   ```

   where:

   <dl>
   <dt><code>spec.ingressClass</code></dt>
   <dd>Specifies the name of the <code>IngressClass</code> resource.</dd>
   </dl>
6. Create the `AWSLoadBalancerController` resource by running the following command:
   ```terminal
   $ oc create -f sample-single-lb.yaml
   ```
7. Create the `Ingress` resource YAML file, for example, `sample-multiple-ingress.yaml`, as follows:
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: example-1
     annotations:
       alb.ingress.kubernetes.io/scheme: internet-facing
       alb.ingress.kubernetes.io/group.order: "1"
       alb.ingress.kubernetes.io/target-type: instance
   spec:
     ingressClassName: single-lb
     rules:
     - host: example.com
       http:
           paths:
           - path: /blog
             pathType: Prefix
             backend:
               service:
                 name: example-1
                 port:
                   number: 80
   ---
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: example-2
     annotations:
       alb.ingress.kubernetes.io/scheme: internet-facing
       alb.ingress.kubernetes.io/group.order: "2"
       alb.ingress.kubernetes.io/target-type: instance
   spec:
     ingressClassName: single-lb
     rules:
     - host: example.com
       http:
           paths:
           - path: /store
             pathType: Prefix
             backend:
               service:
                 name: example-2
                 port:
                   number: 80
   ---
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: example-3
     annotations:
       alb.ingress.kubernetes.io/scheme: internet-facing
       alb.ingress.kubernetes.io/group.order: "3"
       alb.ingress.kubernetes.io/target-type: instance
   spec:
     ingressClassName: single-lb
     rules:
     - host: example.com
       http:
           paths:
           - path: /
             pathType: Prefix
             backend:
               service:
                 name: example-3
                 port:
                   number: 80
   ```

   where:

   <dl>
   <dt><code>metadata.name</code></dt>
   <dd>Specifies the ingress name.</dd>
   <dt><code>alb.ingress.kubernetes.io/scheme</code></dt>
   <dd>Specifies the load balancer to provision in the public subnet to access the internet.</dd>
   <dt><code>alb.ingress.kubernetes.io/group.order</code></dt>
   <dd>Specifies the order in which the rules from the multiple ingress resources are matched when the request is received at the load balancer.</dd>
   <dt><code>alb.ingress.kubernetes.io/target-type</code></dt>
   <dd>Specifies that the load balancer will target OpenShift Container Platform nodes to reach the service.</dd>
   <dt><code>spec.ingressClassName</code></dt>
   <dd>Specifies the ingress class that belongs to this ingress.</dd>
   <dt><code>rules.host</code></dt>
   <dd>Specifies a domain name used for request routing.</dd>
   <dt><code>http.paths.path</code></dt>
   <dd>Specifies the path that must route to the service.</dd>
   <dt><code>backend.service.name</code></dt>
   <dd>Specifies the service name that serves the endpoint configured in the <code>Ingress</code> resource.</dd>
   <dt><code>port.number</code></dt>
   <dd>Specifies the port on the service that serves the endpoint.</dd>
   </dl>
8. Create the `Ingress` resource by running the following command:
   ```terminal
   $ oc create -f sample-multiple-ingress.yaml
   ```

## AWS Load Balancer Operator logs {#nw-aws-load-balancer-operator-logs_aws-load-balancer-operator}

To troubleshoot the AWS Load Balancer Operator, view the logs using the `oc logs` command. By viewing the logs, you can diagnose issues and monitor the activity of the Operator.

**Procedure**

- View the logs of the AWS Load Balancer Operator by running the following command:
  ```terminal
  $ oc logs -n aws-load-balancer-operator deployment/aws-load-balancer-operator-controller-manager -c manager
  ```
