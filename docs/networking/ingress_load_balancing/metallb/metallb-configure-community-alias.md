---
title: Configuring community alias
sidebar_position: 4
---

# Configuring community alias {#metallb-configure-community-alias}

<a id="metallb-configure-community-alias"></a>

As a cluster administrator, you can configure a community alias and use it across different advertisements.

## About the community custom resource {#nw-metallb-community-cr_configure-community-alias}

To simplify BGP configuration, define named aliases for community values by using the community custom resource. You can reference these aliases when advertising `ipAddressPools` with the `BGPAdvertisement` resource.

The fields for the `community` custom resource are described in the following table.

:::note

The `community` CRD applies only to BGPAdvertisement.

:::

**MetalLB community custom resource**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>metadata.name</code></td>
  <td><code>string</code></td>
  <td>Specifies the name for the <code>community</code>.</td>
</tr>
<tr>
  <td><code>metadata.namespace</code></td>
  <td><code>string</code></td>
  <td>Specifies the namespace for the <code>community</code>. Specify the same namespace that the MetalLB Operator uses.</td>
</tr>
<tr>
  <td><code>spec.communities</code></td>
  <td><code>string</code></td>
  <td>Specifies a list of BGP community aliases that can be used in BGPAdvertisements. A community alias consists of a pair of name (alias) and value (number:number). Link the BGPAdvertisement to a community alias by referring to the alias name in its <code>spec.communities</code> field.</td>
</tr>
</tbody>
</table>

**CommunityAlias**

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>name</code></td>
  <td><code>string</code></td>
  <td>The name of the alias for the <code>community</code>.</td>
</tr>
<tr>
  <td><code>value</code></td>
  <td><code>string</code></td>
  <td>The BGP <code>community</code> value corresponding to the given name.</td>
</tr>
</tbody>
</table>

## Configure MetalLB with a BGP advertisement and community alias {#nw-metallb-configure-BGP-advertisement-community-alias_configure-community-alias}

To advertise an `IPAddressPool` by using the BGP protocol, configure MetalLB with a community alias. This configuration sets the alias to the numeric value of the `NO_ADVERTISE` community.

In the following example, the peer BGP router `doc-example-peer-community` receives one `203.0.113.200/32` route and one `fc00:f853:ccd:e799::1/128` route for each load-balancer IP address that MetalLB assigns to a service. A community alias is configured with the `NO_ADVERTISE` community.

**Prerequisites**

- Install the OpenShift CLI (`oc`)
- Log in as a user with `cluster-admin` privileges.

**Procedure**

1. Create an IP address pool.
   1. Create a file, such as `ipaddresspool.yaml`, with content like the following example:
      ```yaml
      apiVersion: metallb.io/v1beta1
      kind: IPAddressPool
      metadata:
        namespace: metallb-system
        name: doc-example-bgp-community
      spec:
        addresses:
          - 203.0.113.200/30
          - fc00:f853:ccd:e799::/124
      ```
   2. Apply the configuration for the IP address pool:
      ```terminal
      $ oc apply -f ipaddresspool.yaml
      ```
2. Create a community alias named `community1`.
   ```yaml
   apiVersion: metallb.io/v1beta1
   kind: Community
   metadata:
     name: community1
     namespace: metallb-system
   spec:
     communities:
       - name: NO_ADVERTISE
         value: '65535:65282'
   ```
3. Create a BGP peer named `doc-example-bgp-peer`.
   1. Create a file, such as `bgppeer.yaml`, with content like the following example:
      ```yaml
      apiVersion: metallb.io/v1beta2
      kind: BGPPeer
      metadata:
        namespace: metallb-system
        name: doc-example-bgp-peer
      spec:
        peerAddress: 10.0.0.1
        peerASN: 64501
        myASN: 64500
        routerID: 10.10.10.10
      ```
   2. Apply the configuration for the BGP peer:
      ```terminal
      $ oc apply -f bgppeer.yaml
      ```
4. Create a BGP advertisement with the community alias.
   1. Create a file, such as `bgpadvertisement.yaml`, with content like the following example:
      ```yaml
      apiVersion: metallb.io/v1beta1
      kind: BGPAdvertisement
      metadata:
        name: bgp-community-sample
        namespace: metallb-system
      spec:
        aggregationLength: 32
        aggregationLengthV6: 128
        communities:
          - NO_ADVERTISE
        ipAddressPools:
          - doc-example-bgp-community
        peers:
          - doc-example-peer
      ```

      where:

      `NO_ADVERTISE`: Specifies the `CommunityAlias.name` here and not the community custom resource (CR) name.
   2. Apply the configuration:
      ```terminal
      $ oc apply -f bgpadvertisement.yaml
      ```
