---
title: Webhook management in Operator Lifecycle Manager
sidebar_position: 9
---

# Webhook management in Operator Lifecycle Manager {#olm-webhooks}

<a id="olm-webhooks"></a>

Webhooks allow Operator authors to intercept, modify, and accept or reject resources before they are saved to the object store and handled by the Operator controller. Operator Lifecycle Manager (OLM) can manage the lifecycle of these webhooks when they are shipped alongside your Operator.

**Additional resources**

- [Types of webhook admission plugins](/docs/architecture/admission-plug-ins#admission-webhook-types_admission-plug-ins)
- [Validating admission webhooks (Kubernetes documentation)](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#validatingadmissionwebhook)
- [Mutating admission webhooks  (Kubernetes documentation)](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#mutatingadmissionwebhook)
- [Conversion webhooks (Kubernetes documentation)](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definition-versioning/#webhook-conversion)
