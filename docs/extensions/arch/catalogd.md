---
title: Catalogd
sidebar_position: 3
---

# Catalogd

<a id="catalogd"></a>

Operator Lifecycle Manager (OLM) v1 uses the catalogd component and its resources to manage Operator and extension catalogs.

## About catalogs in OLM v1 {#olmv1-about-catalogs_catalogd}

You can discover installable content by querying a catalog for Kubernetes extensions, such as Operators and controllers, by using the catalogd component.

Catalogd is a Kubernetes extension that unpacks catalog content for on-cluster clients and is part of the Operator Lifecycle Manager (OLM) v1 suite of microservices. Currently, catalogd unpacks catalog content that is packaged and distributed as container images.

**Additional resources**

- [File-based catalogs](/docs/extensions/catalogs/fbc#fbc)
- [Adding a catalog to a cluster](/docs/extensions/catalogs/managing-catalogs#olmv1-adding-a-catalog-to-a-cluster_managing-catalogs)
- [Red Hat-provided catalogs](/docs/extensions/catalogs/rh-catalogs#rh-catalogs)
