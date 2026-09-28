---
title: "MetalLB logging, troubleshooting, and support"
sidebar_position: 10
---

# MetalLB logging, troubleshooting, and support {#metallb-logging-troubleshooting-support}

<a id="metallb-logging-troubleshooting-support"></a>

To diagnose and resolve MetalLB configuration issues, refer to this list of commonly used commands. By using these commands, you can verify network connectivity and inspect service states to ensure efficient error recovery.

## Set the MetalLB logging levels {#nw-metallb-setting-metalb-logging-levels_metallb-troubleshoot-support}

To manage log verbosity for the `FRRouting` (FRR) container, configure the `logLevel` specification. By adjusting this setting, you can reduce log volume from the default info level or increase detail for troubleshooting MetalLB configuration issues.

Gain a deeper insight into MetalLB by setting the `logLevel` to `debug`.

**Prerequisites**

- You have access to the cluster as a user with the `cluster-admin` role.
- You have installed the OpenShift CLI (`oc`).

**Procedure**

1. Create a file, such as `setdebugloglevel.yaml`, with content such as the following example:
   ```yaml
   apiVersion: metallb.io/v1beta1
   kind: MetalLB
   metadata:
     name: metallb
     namespace: metallb-system
   spec:
     logLevel: debug
     nodeSelector:
       node-role.kubernetes.io/worker: ""
   ```

   :::note

   While other fields like `speakerConfig` can be configured here, the `bgpBackend` field must be omitted. It is reserved for internal use by MetalLB to manage the active BGP implementation.

   :::
2. Apply the configuration by entering the following command:
   ```terminal
   $ oc replace -f setdebugloglevel.yaml
   ```

   :::note

   Use the `oc replace` command because the `metallb` CR was already created and you need to change only the log level.

   :::
3. Display the names of the `speaker` pods:
   ```terminal
   $ oc get -n metallb-system pods -l component=speaker
   ```

   ```text title="Example output"
   NAME                    READY   STATUS    RESTARTS   AGE
   speaker-2m9pm           4/4     Running   0          9m19s
   speaker-7m4qw           3/4     Running   0          19s
   speaker-szlmx           4/4     Running   0          9m19s
   ```

   :::note

   Speaker and controller pods are recreated to ensure the updated logging level is applied. The logging level is modified for all the components of MetalLB.

   :::
4. View the `speaker` logs:
   ```terminal
   $ oc logs -n metallb-system speaker-7m4qw -c speaker
   ```

   ```text title="Example output"
   {"branch":"main","caller":"main.go:92","commit":"3d052535","goversion":"gc / go1.17.1 / amd64","level":"info","msg":"MetalLB speaker starting (commit 3d052535, branch main)","ts":"2022-05-17T09:55:05Z","version":""}
   {"caller":"announcer.go:110","event":"createARPResponder","interface":"ens4","level":"info","msg":"created ARP responder for interface","ts":"2022-05-17T09:55:05Z"}
   {"caller":"announcer.go:119","event":"createNDPResponder","interface":"ens4","level":"info","msg":"created NDP responder for interface","ts":"2022-05-17T09:55:05Z"}
   {"caller":"announcer.go:110","event":"createARPResponder","interface":"tun0","level":"info","msg":"created ARP responder for interface","ts":"2022-05-17T09:55:05Z"}
   {"caller":"announcer.go:119","event":"createNDPResponder","interface":"tun0","level":"info","msg":"created NDP responder for interface","ts":"2022-05-17T09:55:05Z"}
   I0517 09:55:06.515686      95 request.go:665] Waited for 1.026500832s due to client-side throttling, not priority and fairness, request: GET:https://172.30.0.1:443/apis/operators.coreos.com/v1alpha1?timeout=32s
   {"Starting Manager":"(MISSING)","caller":"k8s.go:389","level":"info","ts":"2022-05-17T09:55:08Z"}
   {"caller":"speakerlist.go:310","level":"info","msg":"node event - forcing sync","node addr":"10.0.128.4","node event":"NodeJoin","node name":"ci-ln-qb8t3mb-72292-7s7rh-worker-a-vvznj","ts":"2022-05-17T09:55:08Z"}
   {"caller":"service_controller.go:113","controller":"ServiceReconciler","enqueueing":"openshift-kube-controller-manager-operator/metrics","epslice":"{\"metadata\":{\"name\":\"metrics-xtsxr\",\"generateName\":\"metrics-\",\"namespace\":\"openshift-kube-controller-manager-operator\",\"uid\":\"ac6766d7-8504-492c-9d1e-4ae8897990ad\",\"resourceVersion\":\"9041\",\"generation\":4,\"creationTimestamp\":\"2022-05-17T07:16:53Z\",\"labels\":{\"app\":\"kube-controller-manager-operator\",\"endpointslice.kubernetes.io/managed-by\":\"endpointslice-controller.k8s.io\",\"kubernetes.io/service-name\":\"metrics\"},\"annotations\":{\"endpoints.kubernetes.io/last-change-trigger-time\":\"2022-05-17T07:21:34Z\"},\"ownerReferences\":[{\"apiVersion\":\"v1\",\"kind\":\"Service\",\"name\":\"metrics\",\"uid\":\"0518eed3-6152-42be-b566-0bd00a60faf8\",\"controller\":true,\"blockOwnerDeletion\":true}],\"managedFields\":[{\"manager\":\"kube-controller-manager\",\"operation\":\"Update\",\"apiVersion\":\"discovery.k8s.io/v1\",\"time\":\"2022-05-17T07:20:02Z\",\"fieldsType\":\"FieldsV1\",\"fieldsV1\":{\"f:addressType\":{},\"f:endpoints\":{},\"f:metadata\":{\"f:annotations\":{\".\":{},\"f:endpoints.kubernetes.io/last-change-trigger-time\":{}},\"f:generateName\":{},\"f:labels\":{\".\":{},\"f:app\":{},\"f:endpointslice.kubernetes.io/managed-by\":{},\"f:kubernetes.io/service-name\":{}},\"f:ownerReferences\":{\".\":{},\"k:{\\\"uid\\\":\\\"0518eed3-6152-42be-b566-0bd00a60faf8\\\"}\":{}}},\"f:ports\":{}}}]},\"addressType\":\"IPv4\",\"endpoints\":[{\"addresses\":[\"10.129.0.7\"],\"conditions\":{\"ready\":true,\"serving\":true,\"terminating\":false},\"targetRef\":{\"kind\":\"Pod\",\"namespace\":\"openshift-kube-controller-manager-operator\",\"name\":\"kube-controller-manager-operator-6b98b89ddd-8d4nf\",\"uid\":\"dd5139b8-e41c-4946-a31b-1a629314e844\",\"resourceVersion\":\"9038\"},\"nodeName\":\"ci-ln-qb8t3mb-72292-7s7rh-master-0\",\"zone\":\"us-central1-a\"}],\"ports\":[{\"name\":\"https\",\"protocol\":\"TCP\",\"port\":8443}]}","level":"debug","ts":"2022-05-17T09:55:08Z"}
   ```
5. List the FRR-K8s pods:
   ```terminal
   $ oc get -n openshift-frr-k8s pods
   ```

   ```text title="Example output"
   NAME                                      READY   STATUS    RESTARTS   AGE
   frr-k8s-bz2dn                            7/7     Running   0          4h
   frr-k8s-statuscleaner-59cf6f5d44-9wkfr   1/1     Running   0          4h
   ```
6. View the FRR logs by specifying the `frr` container in one of the `frr-k8s` pods:
   ```terminal
   $ oc logs -n openshift-frr-k8s <frr_k8s_pod_name> -c frr
   ```

   ```text title="Example output"
   2026/03/02 09:53:09 WATCHFRR: [T83RR-8SM5G] watchfrr 8.5.3 starting: vty@0
   2026/03/02 09:53:09 WATCHFRR: [ZCJ3S-SPH5S] zebra state -> down : initial connection attempt failed
   2026/03/02 09:53:09 WATCHFRR: [ZCJ3S-SPH5S] bgpd state -> down : initial connection attempt failed
   2026/03/02 09:53:09 WATCHFRR: [ZCJ3S-SPH5S] staticd state -> down : initial connection attempt failed
   2026/03/02 09:53:09 WATCHFRR: [ZCJ3S-SPH5S] bfdd state -> down : initial connection attempt failed
   2026/03/02 09:53:09 ZEBRA: [NNACN-54BDA][EC 4043309110] Disabling MPLS support (no kernel support)
   2026/03/02 09:53:09 WATCHFRR: [VTVCM-Y2NW3] Configuration Read in Took: 00:00:00
   2026/03/02 09:53:09 WATCHFRR: [QDG3Y-BY5TN] zebra state -> up : connect succeeded
   2026/03/02 09:53:09 WATCHFRR: [QDG3Y-BY5TN] bgpd state -> up : connect succeeded
   2026/03/02 09:53:09 WATCHFRR: [QDG3Y-BY5TN] staticd state -> up : connect succeeded
   2026/03/02 09:53:09 WATCHFRR: [QDG3Y-BY5TN] bfdd state -> up : connect succeeded
   2026/03/02 09:53:09 WATCHFRR: [KWE5Q-QNGFC] all daemons up, doing startup-complete notify
   2026/03/02 09:53:09 ZEBRA: [VTVCM-Y2NW3] Configuration Read in Took: 00:00:00
   2026/03/02 09:53:09 BGP: [VTVCM-Y2NW3] Configuration Read in Took: 00:00:00
   ```

### FRRouting (FRR) log levels {#frr-log-levels_metallb-troubleshoot-support}

To control the verbosity of network logs for troubleshooting or monitoring, refer to the `FRRouting` (FRR) logging levels.

The following values define the severity of recorded events, so that you can use them to filter output based on operational requirements:

**Log levels**

<table>
<thead>
<tr>
  <th>Log level</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>all</code></td>
  <td>Supplies all logging information for all logging levels.</td>
</tr>
<tr>
  <td><code>debug</code></td>
  <td>Information that is diagnostically helpful to people. Set to <code>debug</code> to give detailed troubleshooting information.</td>
</tr>
<tr>
  <td><code>info</code></td>
  <td> Provides information that always should be logged but under normal circumstances does not require user intervention. This is the default logging level.</td>
</tr>
<tr>
  <td><code>warn</code></td>
  <td> Anything that can potentially cause inconsistent <code>MetalLB</code> behaviour. Usually <code>MetalLB</code> automatically recovers from this type of error.</td>
</tr>
<tr>
  <td><code>error</code></td>
  <td>Any unrecoverable error in <code>MetalLB</code>. These errors usually require administrator intervention to fix.</td>
</tr>
<tr>
  <td><code>none</code></td>
  <td>Turn off all logging.</td>
</tr>
</tbody>
</table>

## Troubleshoot BGP issues {#nw-metallb-troubleshoot-bgp_metallb-troubleshoot-support}

To diagnose and resolve BGP configuration issues, run commands directly within the FRR container. By accessing the container, you can verify routing states and identify connectivity errors.

**Prerequisites**

- You have access to the cluster as a user with the `cluster-admin` role.
- You have installed the OpenShift CLI (`oc`).

**Procedure**

1. In OpenShift Container Platform 4.17 and later, MetalLB uses the FRR-K8s daemon for BGP. The Cluster Network Operator (CNO) deploys the FRR-K8s daemon set in the `openshift-frr-k8s` namespace, and the speaker pod no longer contains an FRR container. If the FRR-K8s daemon set is unavailable or its pods are unhealthy, troubleshoot the CNO deployment rather than the MetalLB Operator. Display the names of the `frr-k8s` pods by running the following command:
   ```terminal
   $ oc get pods -n openshift-frr-k8s
   ```

   ```text title="Example output"
   NAME                                     READY   STATUS    RESTARTS   AGE
   frr-k8s-bz2dn                            7/7     Running   0          15m
   frr-k8s-statuscleaner-59cf6f5d44-9wkfr   1/1     Running   0          15m
   ```
2. Display the running configuration for FRR by running the following command:
   ```terminal
   $ oc exec -n openshift-frr-k8s <frr-k8s-pod> -c frr -- vtysh -c "show running-config"
   ```

   ```text title="Example output"
   Building configuration...

   Current configuration:
   !
   frr version 8.5.3
   frr defaults traditional
   hostname mysno-sno.demo.lab
   log file /etc/frr/frr.log informational
   log timestamp precision 3
   no ip forwarding
   no ipv6 forwarding
   service integrated-vtysh-config
   !
   router bgp 64501
    no bgp ebgp-requires-policy
    no bgp default ipv4-unicast
    bgp graceful-restart preserve-fw-state
    no bgp network import-check
    neighbor 192.168.122.12 remote-as 64500
    !
    address-family ipv4 unicast
     network 192.168.122.210/32
     neighbor 192.168.122.12 activate
     neighbor 192.168.122.12 route-map 192.168.122.12-in in
     neighbor 192.168.122.12 route-map 192.168.122.12-out out
    exit-address-family
   exit
   !
   ip prefix-list 192.168.122.12-inpl-ipv4 seq 1 deny any
   ip prefix-list 192.168.122.12-allowed-ipv4 seq 1 permit 192.168.122.210/32
   !
   ipv6 prefix-list 192.168.122.12-allowed-ipv6 seq 1 deny any
   ipv6 prefix-list 192.168.122.12-inpl-ipv4 seq 2 deny any
   !
   route-map 192.168.122.12-out permit 1
    match ip address prefix-list 192.168.122.12-allowed-ipv4
   exit
   !
   route-map 192.168.122.12-out permit 2
    match ipv6 address prefix-list 192.168.122.12-allowed-ipv6
   exit
   !
   route-map 192.168.122.12-in permit 3
    match ip address prefix-list 192.168.122.12-inpl-ipv4
   exit
   !
   route-map 192.168.122.12-in permit 4
    match ipv6 address prefix-list 192.168.122.12-inpl-ipv4
   exit
   !
   ip nht resolve-via-default
   !
   ipv6 nht resolve-via-default
   !
   end
   ```

   where:

   <dl>
   <dt><code>router bgp 64501</code></dt>
   <dd>This is the local Autonomous System Number (ASN) for your MetalLB speakers.</dd>
   <dt><code>neighbor 192.168.122.12 remote-as 64500</code></dt>
   <dd>This identifies the external BGP Peer. Specifies that a <code>neighbor &lt;ip-address&gt; remote-as &lt;peer-ASN&gt;</code> line exists for each BGP peer custom resource that you added. The local ASN is 64501. The remote ASN is 64500.</dd>
   <dt><code>network 192.168.122.210/32</code></dt>
   <dd>This is a specific LoadBalancer IP from your IPAddressPool. It is being advertised as a <code>/32</code> (a single host route), which is standard for MetalLB.</dd>
   <dt><code>neighbor 192.168.122.12 activate</code></dt>
   <dd>Enables the exchange of IPv4 routing information with that specific neighbor.</dd>
   <dt><code>route-map ... in/out</code></dt>
   <dd>These are "filters" or "policies." They ensure that the speaker only sends the IP addresses you have authorized and does not accidentally learn and install internal routes from your physical router.</dd>
   <dt><code>ip prefix-list ... permit 192.168.122.210/32</code></dt>
   <dd>This creates an allowlist. Only this specific IP is permitted to be advertised.</dd>
   <dt><code>route-map 192.168.122.12-out permit 1</code></dt>
   <dd>This tells the router: "If the IP matches the prefix-list above, permit it to be sent out to the neighbor."</dd>
   <dt><code>ip nht resolve-via-default</code></dt>
   <dd>(Next hop tracking) This is a common setting in MetalLB/FRR to ensure that the BGP next-hop can be resolved using the default route if a more specific route isn’t available. If BFD is enabled for the BGP peer, the output includes additional <code>bfd</code> lines under the neighbor configuration and a <code>bfd</code> section at the end. The following example shows the relevant differences:</dd>
   </dl>

   ```text title="Example output with BFD enabled"
   router bgp 64501
    ...
    neighbor 192.168.122.12 remote-as 64500
    neighbor 192.168.122.12 bfd
    neighbor 192.168.122.12 bfd profile bfd-profile
    ...
   exit
   !
   ...
   !
   bfd
    profile bfd-profile
     minimum-ttl 1
    exit
    !
   exit
   !
   end
   ```
3. Display the BGP summary by running the following command:
   ```terminal
   $ oc exec -n openshift-frr-k8s <frr-k8s-pod> -c frr -- vtysh -c "show bgp summary"
   ```

   ```text title="Example output"

   IPv4 Unicast Summary (VRF default):
   BGP router identifier 192.168.122.12, local AS number 64501 vrf-id 0
   BGP table version 1
   RIB entries 1, using 192 bytes of memory
   Peers 1, using 725 KiB of memory

   Neighbor        V         AS   MsgRcvd   MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd   PfxSnt Desc
   192.168.122.12  4      64500        37        38        0    0    0 00:32:12            0        1 N/A

   Total number of neighbors 1
   ```

   where:

   <dl>
   <dt><code>BGP router identifier 192.168.122.12</code></dt>
   <dd>The BGP router identifier for the node, which is typically the IP address of the primary network interface.</dd>
   <dt><code>local AS number 64501</code></dt>
   <dd>This is the ASN you assigned to MetalLB in your <code>BGPPeer</code> or <code>MetalLB</code> CR.</dd>
   <dt><code>Neighbor</code></dt>
   <dd>The IP <code>192.168.122.12</code> of your external router (the "Peer").</dd>
   <dt><code>AS</code></dt>
   <dd>The ASN of the external router (the "Peer"), which should match the <code>remote-as</code> value in your BGP configuration.</dd>
   <dt><code>Up/Down</code></dt>
   <dd>The shows the session has been stable for 32 minutes.</dd>
   <dt><code>State/PfxRcd</code></dt>
   <dd>This shows the session is established since it is a number, but you have received 0 routes from the peer. Seeing 0 prefixes received is perfectly normal for a standard MetalLB deployment.</dd>
   <dt><code>PfxSnt</code></dt>
   <dd>This shows that you have successfully advertised one route the LoadBalancer IP to the peer. This confirms MetalLB is doing its job. It has taken one LoadBalancer service IP and successfully told the external router: "If you want to reach this IP, send the traffic to me." In the output, the <code>State</code> is a number <code>0</code>, which means the connection is successful. If the connection were broken, you would see text such as <code>Active</code>, <code>Connect</code>, or <code>Idle</code> here.</dd>
   </dl>
4. Display the BGP peers that received an address pool by running the following command:
   ```terminal
   $ oc exec -n openshift-frr-k8s <frr-k8s-pod> -c frr -- vtysh -c "show bgp ipv4 unicast"
   ```

   Replace `ipv4` with `ipv6` to display the BGP peers that received an IPv6 address pool.

   ```text title="Example output"
   BGP table version is 1, local router ID is 192.168.122.12, vrf id 0
   Default local pref 100, local AS 64501
   Status codes:  s suppressed, d damped, h history, * valid, > best, = multipath,
                  i internal, r RIB-failure, S Stale, R Removed
   Nexthop codes: @NNN nexthop's vrf id, < announce-nh-self
   Origin codes:  i - IGP, e - EGP, ? - incomplete
   RPKI validation codes: V valid, I invalid, N Not found

       Network          Next Hop            Metric LocPrf Weight Path
    *> 192.168.122.210/32
                       0.0.0.0                  0         32768 i

   Displayed  1 routes and 1 total paths
   ```

   where:

   <dl>
   <dt><code>local router ID 192.168.122.12</code></dt>
   <dd>The unique identifier for this specific OpenShift node in the BGP topology.</dd>
   <dt><code>local AS 64501</code></dt>
   <dd>The Private Autonomous System Number (ASN) assigned to your MetalLB deployment.</dd>
   <dt><code>*&gt;</code> (Status Codes)</dt>
   <dd><code>*</code> indicates the route is valid. <code>&gt;</code> indicates the route is selected as the best path for advertisement. MetalLB only sends best paths to peers. If <code>&gt;</code> is missing, the peer will not receive the route.</dd>
   <dt><code>192.168.122.210/32</code> (Network)</dt>
   <dd>The specific IP assigned to the LoadBalancer service. The <code>/32</code> mask indicates a host route, ensuring traffic for this specific IP is attracted to this node.</dd>
   <dt><code>0.0.0.0</code> (Next Hop)</dt>
   <dd>Indicates the route is local to the node. <code>0.0.0.0</code> means the current node is the egress point for this service.</dd>
   <dt><code>0</code> (Metric)</dt>
   <dd>The Multi-Exit Discriminator (MED). Used to suggest a preferred path to external neighbors. Default is <code>0</code>.</dd>
   <dt><code>100</code> (LocPrf)</dt>
   <dd>Local Preference. Used to prefer an exit point for the entire AS. Default is <code>100</code>.</dd>
   <dt><code>32768</code> (Weight)</dt>
   <dd>An internal FRR priority value. Locally injected routes default to <code>32768</code>, ensuring the node prefers its own local service path over learned routes.</dd>
   <dt><code>i</code> (Path)</dt>
   <dd>The Origin code. <code>i</code> (IGP) signifies the route was originated internally through the MetalLB speaker injecting the <code>IPAddressPool</code> into the FRR stack. If the table is empty, check if the Service has an IP assigned using the <code>oc get svc</code> command. Verify that a <code>BGPAdvertisement</code> exists and that its <code>nodeSelector</code> or <code>labelSelector</code> matches the service and the node you are running the command on. If the Next Hop is not <code>0.0.0.0</code>, the node might be trying to forward the traffic elsewhere before it even reaches the pods, which can indicate a complex BGP peering issue or an issue with the node underlying routing table.</dd>
   </dl>

**Verification**

To confirm that BGP is functioning correctly, verify that all of the following conditions are met:

- The `show running-config` output contains a `router bgp` section with the correct local ASN and at least one `neighbor` entry with the expected peer IP and remote ASN.
- The `show bgp summary` output shows the BGP session as `Established`. The `State/PfxRcd` column displays a number, such as `0`, rather than a state name such as `Active`, `Connect`, or `Idle`.
- The `PfxSnt` column in the BGP summary shows at least `1`, which confirms that MetalLB is advertising a LoadBalancer IP to the peer.
- The `show bgp ipv4 unicast` output contains at least one route with the `*>` status code, which indicates that the route is valid and selected as the best path for advertisement.
- If BFD is configured, the `show running-config` output includes `neighbor <ip_address> bfd` lines and a `bfd` profile section.

## Troubleshoot BFD issues {#nw-metallb-troubleshoot-bfd_metallb-troubleshoot-support}

To diagnose and resolve Bidirectional Forwarding Detection (BFD) issues, run commands directly within the `FRRouting` (FRR) container. By accessing the container, you can verify that BFD peers are correctly configured with established BGP sessions.

The BFD implementation that Red Hat supports uses `FRRouting` (FRR) in a container that exists in an `frr-k8s` pod in the `openshift-frr-k8s` namespace.

**Prerequisites**

- You have access to the cluster as a user with the `cluster-admin` role.
- You have installed the OpenShift CLI (`oc`).

**Procedure**

1. Display the names of the FRR-K8s pods by running the following command:
   ```terminal
   $ oc get pods -n openshift-frr-k8s -l app=frr-k8s
   ```

   ```text title="Example output"
   NAME            READY   STATUS    RESTARTS   AGE
   frr-k8s-bz2dn   7/7     Running   0          106m
   ```
2. Run the following command against the `frr` container in the FRR-K8s pod to display the BFD peers:
   ```terminal
   $ oc exec -n openshift-frr-k8s <frr_k8s_pod_name> -c frr -- vtysh -c "show bfd peers brief"
   ```

   ```text title="Example output"
   Session count: 1
   SessionId  LocalAddress              PeerAddress              Status
   =========  ============              ===========              ======
   3909139637 10.0.1.2                  10.0.2.3                 up
   ```

   where:

   <dl>
   <dt><code>up</code></dt>
   <dd>Specifies that the <code>PeerAddress</code> column includes each BFD peer. If the output does not list a BFD peer IP address that you expected the output to include, troubleshoot BGP connectivity with the peer. If the status field indicates <code>down</code>, check for connectivity on the links and equipment between the node and the peer. You can determine the node name for the FRR-K8s pod with a command such as <code>oc get pods -n openshift-frr-k8s &lt;frr_k8s_pod_name&gt; -o jsonpath='{.spec.nodeName}'</code>.</dd>
   </dl>
3. Optional: To display detailed BFD peer information, run the following command:
   ```terminal
   $ oc exec -n openshift-frr-k8s <frr_k8s_pod_name> -c frr -- vtysh -c "show bfd peers"
   ```

   ```text title="Example output"
   BFD Peers:
   	peer 10.0.2.3 local-address 10.0.1.2 vrf default interface br-ex
   		ID: 3909139637
   		Remote ID: 2819913327
   		Active mode
   		Status: up
   		Uptime: 1 hour(s), 12 minute(s), 30 second(s)
   		Diagnostics: ok
   		Remote diagnostics: ok
   		Peer Type: dynamic
   		RTT min/avg/max: 301/512/4191 usec
   		Local timers:
   			Detect-multiplier: 3
   			Receive interval: 300ms
   			Transmission interval: 300ms
   			Echo receive interval: 50ms
   			Echo transmission interval: disabled
   		Remote timers:
   			Detect-multiplier: 3
   			Receive interval: 300ms
   			Transmission interval: 300ms
   			Echo receive interval: disabled
   ```

   where:

   <dl>
   <dt><code>Status</code></dt>
   <dd>The current state of the BFD session. A value of <code>up</code> indicates that the session is established and the link is healthy. A value of <code>down</code> indicates that the session has failed.</dd>
   <dt><code>Uptime</code> or <code>Downtime</code></dt>
   <dd>The duration that the session has been in its current state.</dd>
   <dt><code>Remote ID</code></dt>
   <dd>The session ID assigned by the remote peer. A value of <code>0</code> indicates that the remote peer has not responded.</dd>
   <dt><code>Diagnostics</code> and <code>Remote diagnostics</code></dt>
   <dd>The diagnostic codes for the local and remote peers. A value of <code>ok</code> indicates no errors.</dd>
   <dt><code>Detect-multiplier</code></dt>
   <dd>The number of missed packets before the session is declared down. With a receive interval of <code>300ms</code> and a detect-multiplier of <code>3</code>, the session is declared down after <code>900ms</code> of missed packets.</dd>
   </dl>

**Verification**

To confirm that BFD is functioning correctly, verify that all of the following conditions are met:

- The `show bfd peers brief` output lists each expected BFD peer with a `Status` of `up`.
- The `show bfd peers` detailed output shows a nonzero `Remote ID`, which confirms that the remote peer has responded.
- The `Diagnostics` and `Remote diagnostics` fields both display `ok`.
- If a session shows `down`, check the `Downtime` duration and `Remote diagnostics` field for error codes, and verify network connectivity between the node and the peer.

## MetalLB metrics for BGP and BFD {#nw-metallb-metrics_metallb-troubleshoot-support}

To monitor network connectivity and diagnose routing states, refer to the Prometheus metrics for MetalLB. These metrics provide visibility into the status of BGP peers and BFD profiles so that you can ensure stable external communication.

**MetalLB BFD metrics**

| Name | Description |
| --- | --- |
| `frrk8s_bfd_control_packet_input` | Counts the number of BFD control packets received from each BFD peer. |
| `frrk8s_bfd_control_packet_output` | Counts the number of BFD control packets sent to each BFD peer. |
| `frrk8s_bfd_echo_packet_input` | Counts the number of BFD echo packets received from each BFD peer. |
| `frrk8s_bfd_echo_packet_output` | Counts the number of BFD echo packets sent to each BFD. |
| `frrk8s_bfd_session_down_events` | Counts the number of times the BFD session with a peer entered the `down` state. |
| `frrk8s_bfd_session_up` | Indicates the connection state with a BFD peer. `1` indicates the session is `up` and `0` indicates the session is `down`. |
| `frrk8s_bfd_session_up_events` | Counts the number of times the BFD session with a peer entered the `up` state. |
| `frrk8s_bfd_zebra_notifications` | Counts the number of BFD Zebra notifications for each BFD peer. |

**MetalLB BGP metrics**

| Name | Description |
| --- | --- |
| `frrk8s_bgp_announced_prefixes_total` | Counts the number of load balancer IP address prefixes that are advertised to BGP peers. The terms *prefix* and *aggregated route* have the same meaning. |
| `frrk8s_bgp_session_up` | Indicates the connection state with a BGP peer. `1` indicates the session is `up` and `0` indicates the session is `down`. |
| `frrk8s_bgp_updates_total` | Counts the number of BGP update messages sent to each BGP peer. |
| `frrk8s_bgp_opens_sent` | Counts the number of BGP open messages sent to each BGP peer. |
| `frrk8s_bgp_opens_received` | Counts the number of BGP open messages received from each BGP peer. |
| `frrk8s_bgp_notifications_sent` | Counts the number of BGP notification messages sent to each BGP peer. |
| `frrk8s_bgp_updates_total_received` | Counts the number of BGP update messages received from each BGP peer. |
| `frrk8s_bgp_keepalives_sent` | Counts the number of BGP `keepalive` messages sent to each BGP peer. |
| `frrk8s_bgp_keepalives_received` | Counts the number of BGP `keepalive` messages received from each BGP peer. |
| `frrk8s_bgp_route_refresh_sent` | Counts the number of BGP route refresh messages sent to each BGP peer. |
| `frrk8s_bgp_total_sent` | Counts the number of total BGP messages sent to each BGP peer. |
| `frrk8s_bgp_total_received` | Counts the number of total BGP messages received from each BGP peer. |
| `frrk8s_bgp_received_prefixes_total` | Counts the number of load balancer IP address prefixes received from each BGP peer. |

**Additional resources**

- [Querying metrics for all projects with the monitoring dashboard](https://docs.redhat.com/en/documentation/monitoring_stack_for_red_hat_openshift/latest/html/accessing_metrics/accessing-metrics-as-an-administrator#querying-metrics-for-all-projects-with-mon-dashboard_accessing-metrics-as-an-administrator)

## About collecting MetalLB data {#nw-metallb-collecting-data_metallb-troubleshoot-support}

To collect diagnostic data for debugging or support analysis, run the `oc adm must-gather` CLI command. This utility captures essential information regarding the cluster, the MetalLB configuration, and the MetalLB Operator state.

The following list details features and objects related to MetalLB and the MetalLB Operator:

- The namespace and child objects where you deploy the MetalLB Operator
- All MetalLB Operator custom resource definitions (CRDs)

The command collects the following information from `FRRouting` (FRR), which Red Hat uses to implement BGP and BFD:

- `/etc/frr/frr.conf`
- `/etc/frr/frr.log`
- `/etc/frr/daemons` configuration file
- `/etc/frr/vtysh.conf`

The command collects log and configuration files from the `frr` container that exists in each `frr-k8s` pod in the `openshift-frr-k8s` namespace. Additionally, the command collects the output from the following `vtysh` commands:

- `show running-config`
- `show bgp ipv4`
- `show bgp ipv6`
- `show bgp neighbor`
- `show bfd peer`

No additional configuration is required when you run the command.

**Additional resources**

- [Managing symmetric routing with MetalLB](/docs/networking/ingress_load_balancing/metallb/metallb-configure-return-traffic#metallb-configure-return-traffic)
- [Querying metrics for all projects with the monitoring dashboard](https://docs.redhat.com/en/documentation/monitoring_stack_for_red_hat_openshift/4.21/html/accessing_metrics/accessing-metrics-as-an-administrator#querying-metrics-for-all-projects-with-mon-dashboard_accessing-metrics-as-an-administrator)
- [Gathering data about your cluster](/docs/support/gathering-cluster-data#gathering-cluster-data)
