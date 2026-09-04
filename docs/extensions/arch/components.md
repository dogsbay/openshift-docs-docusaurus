---
title: OLM v1 components overview
sidebar_position: 1
---

# OLM v1 components overview {#olm-components}

<a id="olm-components"></a>

Operator Lifecycle Manager (OLM) v1 uses two key microservice components, Operator Controller and Catalogd, to unpack content and manage extensions on your cluster.

Operator Controller
: Extends Kubernetes with an API to install and manage Operators and extensions using metadata from Catalogd.

Catalogd
: Unpacks file-based catalog (FBC) content and hosts metadata so users can discover installable extensions.

**Additional resources**

- [Operator Controller](/docs/extensions/arch/operator-controller#operator-controller)
- [Catalogd](/docs/extensions/arch/catalogd#catalogd)
