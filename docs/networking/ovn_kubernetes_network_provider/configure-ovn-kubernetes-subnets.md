---
title: Configuring OVN-Kubernetes internal IP address subnets
sidebar_position: 6
---

# Configuring OVN-Kubernetes internal IP address subnets {#configure-ovn-kubernetes-subnets}

<a id="configure-ovn-kubernetes-subnets"></a>

As a cluster administrator, you can change the IP address ranges that the OVN-Kubernetes network plugin uses for the join and transit subnets.

## Configuring the OVN-Kubernetes join subnet {#nw-ovn-kubernetes-change-join-subnet_configure-ovn-kubernetes-subnets}

You can change the join subnet used by OVN-Kubernetes to avoid conflicting with any existing subnets already in use in your environment.

**Prerequisites**

- Install the OpenShift CLI (`oc`).
- Log in to the cluster with a user with `cluster-admin` privileges.
- Ensure that the cluster uses the OVN-Kubernetes network plugin.

**Procedure**

- To change the OVN-Kubernetes join subnet, enter the following command:
  ```terminal
  $ oc patch network.operator.openshift.io cluster --type='merge' \
    -p='{"spec":{"defaultNetwork":{"ovnKubernetesConfig":
      {"ipv4":{"internalJoinSubnet": "<join_subnet>"},
      "ipv6":{"internalJoinSubnet": "<join_subnet>"}}}}}'
  ```

  where:

  <dl>
  <dt><code>&lt;join_subnet&gt;</code></dt>
  <dd>Specifies an IP address subnet for internal use by OVN-Kubernetes. The subnet must be larger than the number of nodes in the cluster and it must be large enough to accommodate one IP address per node in the cluster. This subnet cannot overlap with any other subnets used by OpenShift Container Platform or on the host itself. The default value for IPv4 is <code>100.64.0.0/16</code> and the default value for IPv6 is <code>fd98::/64</code>.</dd>
  </dl>

  ```text title="Example output"
  network.operator.openshift.io/cluster patched
  ```

**Verification**

- To confirm that the configuration is active, enter the following command:
  ```terminal
  $ oc get network.operator.openshift.io \
    -o jsonpath="{.items[0].spec.defaultNetwork}"
  ```

  The command operation can take up to 30 minutes for this change to take effect.

  ```text title="Example output"
  {
    "ovnKubernetesConfig": {
      "ipv4": {
        "internalJoinSubnet": "100.64.1.0/16"
      },
    },
    "type": "OVNKubernetes"
  }
  ```

## Configuring the OVN-Kubernetes masquerade subnet as a post-installation operation {#nw-ovn-k-day-2-masq-subnet_configure-ovn-kubernetes-subnets}

You can change the masquerade subnet used by OVN-Kubernetes as a post-installation operation to avoid conflicts with any existing subnets that are already in use in your environment.

**Prerequisites**

- Install the OpenShift CLI (`oc`).
- Log in to the cluster as a user with `cluster-admin` privileges.

**Procedure**

- Change your cluster’s masquerade subnet:
  - For dualstack clusters using IPv6, run the following command:
    ```terminal
    $ oc patch networks.operator.openshift.io cluster --type=merge -p '{"spec":{"defaultNetwork":{"ovnKubernetesConfig":{"gatewayConfig":{"ipv4":{"internalMasqueradeSubnet": "<ipv4_masquerade_subnet>"},"ipv6":{"internalMasqueradeSubnet": "<ipv6_masquerade_subnet>"}}}}}}'
    ```

    where:

    <dl>
    <dt><code>ipv4_masquerade_subnet</code></dt>
    <dd>Specifies an IP address to be used as the IPv4 masquerade subnet. This range cannot overlap with any other subnets used by OpenShift Container Platform or on the host itself. In versions of OpenShift Container Platform earlier than 4.17, the default value for IPv4 was <code>169.254.169.0/29</code>, and clusters that were upgraded to version 4.17 maintain this value. For new clusters starting from version 4.17, the default value is <code>169.254.0.0/17</code>.</dd>
    <dt><code>ipv6_masquerade_subnet</code></dt>
    <dd>Specifies an IP address to be used as the IPv6 masquerade subnet. This range cannot overlap with any other subnets used by OpenShift Container Platform or on the host itself. The default value for IPv6 is <code>fd69::/125</code>.</dd>
    </dl>
  - For clusters using IPv4, run the following command:
    ```terminal
    $ oc patch networks.operator.openshift.io cluster --type=merge -p '{"spec":{"defaultNetwork":{"ovnKubernetesConfig":{"gatewayConfig":{"ipv4":{"internalMasqueradeSubnet": "<ipv4_masquerade_subnet>"}}}}}}'
    ```

    where:

    `ipv4_masquerade_subnet`:Specifies[] an IP address to be used as the IPv4 masquerade subnet. This range cannot overlap with any other subnets used by OpenShift Container Platform or on the host itself. In versions of OpenShift Container Platform earlier than 4.17, the default value for IPv4 was `169.254.169.0/29`, and clusters that were upgraded to version 4.17 maintain this value. For new clusters starting from version 4.17, the default value is `169.254.0.0/17`.

## Configuring the OVN-Kubernetes transit subnet {#nw-ovn-kubernetes-change-transit-subnet_configure-ovn-kubernetes-subnets}

You can change the transit subnet used by OVN-Kubernetes to avoid conflicting with any existing subnets already in use in your environment.

**Prerequisites**

- Install the OpenShift CLI (`oc`).
- Log in to the cluster with a user with `cluster-admin` privileges.
- Ensure that the cluster uses the OVN-Kubernetes network plugin.

**Procedure**

- To change the OVN-Kubernetes transit subnet, enter the following command:
  ```terminal
  $ oc patch network.operator.openshift.io cluster --type='merge' \
    -p='{"spec":{"defaultNetwork":{"ovnKubernetesConfig":
      {"ipv4":{"internalTransitSwitchSubnet": "<transit_subnet>"},
      "ipv6":{"internalTransitSwitchSubnet": "<transit_subnet>"}}}}}'
  ```

  where:

  <dl>
  <dt><code>&lt;transit_subnet&gt;</code></dt>
  <dd>Specifies an IP address subnet for the distributed transit switch that enables east-west traffic. This subnet cannot overlap with any other subnets used by OVN-Kubernetes or on the host itself. The default value for IPv4 is <code>100.88.0.0/16</code> and the default value for IPv6 is <code>fd97::/64</code>.</dd>
  </dl>

  ```text title="Example output"
  network.operator.openshift.io/cluster patched
  ```

**Verification**

- To confirm that the configuration is active, enter the following command:
  ```terminal
  $ oc get network.operator.openshift.io \
    -o jsonpath="{.items[0].spec.defaultNetwork}"
  ```

  It can take up to 30 minutes for this change to take effect.

  ```text title="Example output"
  {
    "ovnKubernetesConfig": {
      "ipv4": {
        "internalTransitSwitchSubnet": "100.88.1.0/16"
      },
    },
    "type": "OVNKubernetes"
  }
  ```
