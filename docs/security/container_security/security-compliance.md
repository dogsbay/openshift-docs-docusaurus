---
title: Understanding compliance
sidebar_position: 5
---

# Understanding compliance {#security-compliance}

<a id="security-compliance"></a>

You should understand the regulatory readiness, or compliance, that is required before any systems can be put into production. That regulatory readiness can be imposed by national standards, industry standards or the organization’s corporate governance framework.

## Understand compliance and risk management {#security-compliance-nist_security-compliance}

FIPS compliance is one of the most critical components required in highly secure environments to ensure that only supported cryptographic technologies are allowed on nodes.

:::warning

To enable FIPS mode for your cluster, you must run the installation program from a Red Hat Enterprise Linux (RHEL) computer configured to operate in FIPS mode. For more information about configuring FIPS mode on RHEL, see [Switching RHEL to FIPS mode](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/security_hardening/switching-rhel-to-fips-mode_security-hardening).

When running Red Hat Enterprise Linux (RHEL) or Red Hat Enterprise Linux CoreOS (RHCOS) booted in FIPS mode, OpenShift Container Platform core components use the RHEL cryptographic libraries that have been submitted to NIST for FIPS 140-2/140-3 Validation on only the x86_64, ppc64le, and s390x architectures.

:::

To understand Red Hat’s view of OpenShift Container Platform compliance frameworks, refer to the Risk Management and Regulatory Readiness chapter of the OpenShift Security Guide Book.

**Additional resources**

- [Installing a cluster in FIPS mode](/docs/installing/overview/installing-fips#installing-fips-mode_installing-fips)
