---
title: Configuring ingress cluster traffic using a load balancer
sidebar_position: 5
---

# Configuring ingress cluster traffic using a load balancer {#configuring-ingress-cluster-traffic-load-balancer}

<a id="configuring-ingress-cluster-traffic-load-balancer"></a>

OpenShift Container Platform provides methods for communicating from outside the cluster with services running in the cluster. This method uses a load balancer.

Before starting the following procedures, the administrator must complete the following prerequisite tasks:

- Set up the external port to the cluster networking environment so that requests can reach the cluster.
- Have an OpenShift Container Platform cluster with at least one control plane node, at least one compute node, and a system outside the cluster that has network access to the cluster. This procedure assumes that the external system is on the same subnet as the cluster. The additional networking required for external systems on a different subnet is out-of-scope for this topic.
- Make sure there is at least one user with cluster admin role. To add this role to a user, run the following command:
  ```terminal
  $ oc adm policy add-cluster-role-to-user cluster-admin username
  ```

## Using a load balancer to get traffic into the cluster {#nw-using-load-balancer-getting-traffic_configuring-ingress-cluster-traffic-load-balancer}

If you do not need a specific external IP address, you can configure a load balancer service to allow external access to an OpenShift Container Platform cluster.

A load balancer service allocates a unique IP. The load balancer has a single edge router IP, which can be a virtual IP (VIP), but is still a single machine for initial load balancing.

:::note

A pool gets configured at the infrastructure level and not the cluster administrator level.

:::

## Creating a project and service {#nw-creating-project-and-service_configuring-ingress-cluster-traffic-load-balancer}

If the project and service that you want to expose does not exist, create the project and then create the service.

If the project and service already exists, skip to the procedure on exposing the service to create a route.

**Prerequisites**

- Install the OpenShift CLI (`oc`) and log in as a cluster administrator.

**Procedure**

1. Create a new project for your service by running the `oc new-project` command:
   ```terminal
   $ oc new-project <project_name>
   ```
2. Use the `oc new-app` command to create your service:
   ```terminal
   $ oc new-app nodejs:12~https://github.com/sclorg/nodejs-ex.git
   ```
3. To verify that the service was created, run the following command:
   ```terminal
   $ oc get svc -n <project_name>
   ```

   ```terminal title="Example output"
   NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
   nodejs-ex   ClusterIP   172.30.197.157   <none>        8080/TCP   70s
   ```

   :::note

   By default, the new service does not have an external IP address.

   :::

## Exposing the service by creating a route {#nw-exposing-service_configuring-ingress-cluster-traffic-load-balancer}

To enable external access to your application that runs on OpenShift Container Platform, you can expose the service as a route by using the `oc expose` command.

**Prerequisites**

- You logged into OpenShift Container Platform.

**Procedure**

1. Log in to the project where the service you want to expose is located:
   ```terminal
   $ oc project <project_name>
   ```
2. Run the `oc expose service` command to expose the route:
   ```terminal
   $ oc expose service nodejs-ex
   ```

   ```terminal title="Example output"
   route.route.openshift.io/nodejs-ex exposed
   ```
3. To verify that the service is exposed, you can use a tool, such as `curl` to check that the service is accessible from outside the cluster.
   1. To find the hostname of the route, enter the following command:
      ```terminal
      $ oc get route
      ```

      ```terminal title="Example output"
      NAME        HOST/PORT                        PATH   SERVICES    PORT       TERMINATION   WILDCARD
      nodejs-ex   nodejs-ex-myproject.example.com         nodejs-ex   8080-tcp                 None
      ```
   2. To check that the host responds to a GET request, enter the following command:
      ```terminal title="Example curl command"
      $ curl --head nodejs-ex-myproject.example.com
      ```

      ```terminal title="Example output"
      HTTP/1.1 200 OK
      ...
      ```

## Creating a load balancer service {#nw-create-load-balancer-service_configuring-ingress-cluster-traffic-load-balancer}

To distribute incoming traffic efficiently and ensure high availability for your applications in OpenShift Container Platform, create a load balancer service.

**Prerequisites**

- Make sure that the project and service you want to expose exist.
- Your cloud provider supports load balancers.

**Procedure**

1. Log in to OpenShift Container Platform.
2. Load the project where the service you want to expose is located.
   ```terminal title="Example command"
   $ oc project project1
   ```
3. Open a text file on the control plane node and paste the following text into the file. Edit the file as needed.
   ```yaml title="Sample load balancer configuration file"
   apiVersion: v1
   kind: Service
   metadata:
     name: egress-2
   spec:
     ports:
     - name: db
       port: 3306
     loadBalancerIP:
     loadBalancerSourceRanges:
     - 10.0.0.0/8
     - 192.168.0.0/16
     type: LoadBalancer
     selector:
       name: mysql
   ```

   where:

   <dl>
   <dt><code>metadata.name</code></dt>
   <dd>Specifies a descriptive name for the load balancer service.</dd>
   <dt><code>ports.port</code></dt>
   <dd>Specifies the same port that the service you want to expose is listening on.</dd>
   <dt><code>loadBalancerSourceRanges</code></dt>
   <dd>Specifies a list of specific IP addresses to restrict traffic through the load balancer. The parameter is ignored if the cloud provider does not support the feature.</dd>
   <dt><code>type</code></dt>
   <dd>Specifies <code>Loadbalancer</code> as the type.</dd>
   <dt><code>selector.name</code></dt>
   <dd>Specifies the name of the service.</dd>
   </dl>

   :::note

   To restrict the traffic through the load balancer to specific IP addresses, use the `spec.endpointPublishingStrategy.loadBalancer.allowedSourceRanges` Ingress Controller parameter. Do not set the `loadBalancerSourceRanges` parameter.

   :::
4. Save and exit the file.
5. Run the following command to create the service:
   ```terminal
   $ oc create -f <file_name>
   ```

   For example:

   ```terminal
   $ oc create -f mysql-lb.yaml
   ```
6. Execute the following command to view the new service:
   ```terminal
   $ oc get svc
   ```

   ```terminal title="Example output"
   NAME       TYPE           CLUSTER-IP      EXTERNAL-IP                             PORT(S)          AGE
   egress-2   LoadBalancer   172.30.22.226   ad42f5d8b303045-487804948.example.com   3306:30357/TCP   15m
   ```

   The service has an external IP address automatically assigned if there is a cloud provider enabled.
7. On the master, use a tool, such as `curl`, to make sure you can reach the service by using the public IP address:
   ```terminal
   $ curl <public_ip>:<port>
   ```

   For example:

   ```terminal
   $ curl 172.29.121.74:3306
   ```

   The examples in this section use a MySQL service, which requires a client application. If you get a string of characters with the `Got packets out of order` message, you are connecting with the service:

   If you have a MySQL client, log in with the standard CLI command:

   ```terminal
   $ mysql -h 172.30.131.89 -u admin -p
   ```

   ```terminal title="Example output"
   Enter password:
   Welcome to the MariaDB monitor.  Commands end with ; or \g.

   MySQL [(none)]>
   ```
