---
title: Aggregated API client certificates
sidebar_position: 8
---

# Aggregated API client certificates {#cert-types-aggregated-api-client-certificates}

<a id="cert-types-aggregated-api-client-certificates"></a>

Review aggregated API client certificate validity and automatic rotation in OpenShift Container Platform to plan maintenance for extension API server authentication.

## Purpose {#aggregated-api-client-certificates-purpose_cert-types-aggregated-api-client-certificates}

Aggregated API client certificates are used to authenticate the `KubeAPIServer` when connecting to the aggregated API servers.

## Management {#aggregated-api-client-certificates-management_cert-types-aggregated-api-client-certificates}

These certificates are managed by the system and not the user.

## Expiration {#aggregated-api-client-certificates-expiration_cert-types-aggregated-api-client-certificates}

This certificate authority (CA) is valid for 30 days.

The managed client certificates are valid for 30 days.

CA and client certificates are rotated automatically through the use of controllers.

## Customization {#aggregated-api-client-certificates-customization_cert-types-aggregated-api-client-certificates}

You cannot customize the aggregated API server certificates.
