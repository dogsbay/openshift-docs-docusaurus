---
title: Creating a project as another user
sidebar_position: 2
---

# Creating a project as another user {#creating-project-other-user}

<a id="creating-project-other-user"></a>

You can use impersonation to create a project on behalf of a different user account.

## API impersonation {#authentication-api-impersonation_creating-project-other-user}

You can configure API requests in OpenShift Container Platform to act as another user. Impersonation allows you to perform actions on behalf of another account without switching credentials.

## Impersonating a user when you create a project {#impersonation-project-creation_creating-project-other-user}

You can impersonate a different user when you create a project request. Because `system:authenticated:oauth` is the only bootstrap group that can create project requests, you must impersonate that group.

**Procedure**

- To create a project request on behalf of a different user:
  ```terminal
  $ oc new-project <project> --as=<user> \
      --as-group=system:authenticated --as-group=system:authenticated:oauth
  ```

**Additional resources**

- [User impersonation (Kubernetes documentation)](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#user-impersonation)
