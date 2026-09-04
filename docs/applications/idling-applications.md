---
title: Reducing resource consumption with application idling
sidebar_position: 14
---

# Reducing resource consumption with application idling {#idling-applications}

<a id="idling-applications"></a>

As an administrator, you can reduce cluster resource consumption and lower public cloud costs by temporarily scaling inactive application resources to zero replicas.

If any scalable resources are not in use, OpenShift Container Platform discovers and idles them by scaling their replicas to `0`. The next time network traffic is directed to the resources, the resources are unidled by scaling up the replicas, and normal operation continues.

Applications are made of services, as well as other scalable resources, such as deployment configs. The action of idling an application involves idling all associated resources.

## Application idling {#idle-idling-applications_idling-applications}

Identify the scalable resources for one or more services, such as deployment configurations and replication controllers, and scale them down to zero replicas to optimize cluster capacity.

You can use the `oc idle` command to idle a single service, or use the `--resource-names-file` option to idle multiple services.

### Idling a single service {#idle-idling-applications-single_idling-applications}

Scale down the scalable resources of a specific service to zero replicas to reduce cluster consumption.

**Procedure**

1. To idle a single service, run:
   ```terminal
   $ oc idle <service>
   ```

### Idling multiple services {#idle-idling-applications-multiple_idling-applications}

Scale multiple inactive services down to zero replicas to optimize cluster capacity.

Idling multiple services is helpful if an application spans across a set of services within a project, or when idling multiple services in conjunction with a script to idle multiple applications in bulk within the same project.

**Procedure**

1. Create a file containing a list of the services, each on their own line.
2. Idle the services using the `--resource-names-file` option:
   ```terminal
   $ oc idle --resource-names-file <filename>
   ```

   :::note

   The `idle` command is limited to a single project. For idling applications across a cluster, run the `idle` command for each project individually.

   :::

## Unidling applications {#idle-unidling-applications_idling-applications}

Restore normal application operations by scaling up the replicas when network traffic is directed back to the idled resources.

Application services become active again when they receive network traffic and are scaled back up to their previous state. This includes both traffic to the services and traffic passing through routes. Applications can also be manually unidled by scaling up the resources.

**Procedure**

- To scale up a DeploymentConfig, run:
  ```terminal
  $ oc scale --replicas=1 dc <dc_name>
  ```

  :::note

  Automatic unidling by a router is currently only supported by the default HAProxy router.

  :::
