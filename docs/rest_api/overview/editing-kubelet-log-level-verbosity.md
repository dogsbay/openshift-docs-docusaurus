---
title: Editing kubelet log level verbosity and gathering logs
sidebar_position: 3
---

# Editing kubelet log level verbosity and gathering logs {#editing-kubelet-log-level-verbosity}

<a id="editing-kubelet-log-level-verbosity"></a>

To troubleshoot some issues with nodes, establish the kubelet’s log level verbosity depending on the issue to be tracked.

## Modifying the kubelet as a one-time scenario {#modifying-kubelet-one-time_editing-kubelet-log-level-verbosity}

To modify the kubelet in a one-time scenario without rebooting the node due to the change of `machine-config(spec":{"paused":false}})`, allowing you to modify the kubelet without affecting the service, follow this procedure.

**Procedure**

1. Connect to the node in debug mode:
   ```terminal
   $ oc debug node/<node>
   ```

   ```terminal
   $ chroot /host
   ```

   Alternatively, it is possible to SSH to the node and become root.
2. After access is established, check the default log level:
   ```terminal
   $ systemctl cat kubelet
   ```

   ```terminal title="Example output"
   # /etc/systemd/system/kubelet.service.d/20-logging.conf
   [Service]
   Environment="KUBELET_LOG_LEVEL=2"
   ```
3. Define the new verbosity required in a new `/etc/systemd/system/kubelet.service.d/30-logging.conf` file, which overrides `/etc/systemd/system/kubelet.service.d/20-logging.conf`. In this example, the verbosity is changed from `2` to `8`:
   ```terminal
   $ echo -e "[Service]\nEnvironment=\"KUBELET_LOG_LEVEL=8\"" > /etc/systemd/system/kubelet.service.d/30-logging.conf
   ```
4. Reload systemd and restart the service:
   ```terminal
   $ systemctl daemon-reload
   ```

   ```terminal
   $ systemctl restart kubelet
   ```
5. Gather the logs, and then revert the log level increase:
   ```terminal
   $ rm -f /etc/systemd/system/kubelet.service.d/30-logging.conf
   ```

   ```terminal
   $ systemctl daemon-reload
   ```

   ```terminal
   $ systemctl restart kubelet
   ```

## Persistent kubelet log level configuration {#persistent-kubelet-log-level-configuration_editing-kubelet-log-level-verbosity}

**Procedure**

- Use the following `MachineConfig` object for persistent kubelet log level configuration:
  ```yaml
   apiVersion: machineconfiguration.openshift.io/v1
   kind: MachineConfig
   metadata:
     labels:
       machineconfiguration.openshift.io/role: master
     name: 99-master-kubelet-loglevel
   spec:
     config:
       ignition:
         version: 3.2.0
       systemd:
         units:
           - name: kubelet.service
             enabled: true
             dropins:
               - name: 30-logging.conf
                 contents: |
                   [Service]
                   Environment="KUBELET_LOG_LEVEL=2"
  ```

  Generally, it is recommended to apply `0-4` as debug-level logs and `5-8` as trace-level logs.

## Log verbosity descriptions {#log-verbosity-descriptions_editing-kubelet-log-level-verbosity}

<table>
<thead>
<tr>
  <th>Log verbosity</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>--v=0</code></td>
  <td>Always visible to an Operator.</td>
</tr>
<tr>
  <td><code>--v=1</code></td>
  <td>A reasonable default log level if you do not want verbosity.</td>
</tr>
<tr>
  <td><code>--v=2</code></td>
  <td>Useful steady state information about the service and important log messages that might correlate to significant changes in the system. This is the recommended default log level.</td>
</tr>
<tr>
  <td><code>--v=3</code></td>
  <td>Extended information about changes.</td>
</tr>
<tr>
  <td><code>--v=4</code></td>
  <td>Debug level verbosity.</td>
</tr>
<tr>
  <td><code>--v=6</code></td>
  <td>Display requested resources.</td>
</tr>
<tr>
  <td><code>--v=7</code></td>
  <td>Display HTTP request headers.</td>
</tr>
<tr>
  <td><code>--v=8</code></td>
  <td>Display HTTP request contents.</td>
</tr>
</tbody>
</table>

## Gathering kubelet logs {#gathering-kubelet-logs_editing-kubelet-log-level-verbosity}

**Procedure**

- After the kubelet’s log level verbosity is configured properly, you can gather logs by running the following commands:
  ```terminal
  $ oc adm node-logs --role master -u kubelet
  ```

  ```terminal
  $ oc adm node-logs --role worker -u kubelet
  ```

  Alternatively, inside the node, run the following command:

  ```terminal
  $ journalctl -b -f -u kubelet.service
  ```
- To collect master container logs, run the following command:
  ```terminal
  $ sudo tail -f /var/log/containers/*
  ```
- To directly gather the logs of all nodes, run the following command:
  ```terminal
  - for n in $(oc get node --no-headers | awk '{print $1}'); do oc adm node-logs $n | gzip > $n.log.gz; done
  ```
