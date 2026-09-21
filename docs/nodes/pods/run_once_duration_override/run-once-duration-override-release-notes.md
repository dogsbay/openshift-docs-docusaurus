---
title: Run Once Duration Override Operator release notes
sidebar_position: 2
---

# Run Once Duration Override Operator release notes {#run-once-duration-override-release-notes}

<a id="run-once-duration-override-release-notes"></a>

The Run Once Duration Override Operator sets a maximum active deadline on run-once pods, terminating pods that exceed the configured duration.

To apply the run-once duration override from the Run Once Duration Override Operator to run-once pods, you must enable it on each applicable namespace.

These release notes track the development of the Run Once Duration Override Operator for OpenShift Container Platform.

**Additional resources**

- [About the Run Once Duration Override Operator](/docs/nodes/pods/run_once_duration_override#rodoo-about_run-once-duration-override-about)

## Run Once Duration Override Operator 1.4.2 {#run-once-duration-override-operator-release-notes-1-4-2_run-once-duration-override-release-notes}

Review the features, enhancements, and advisory for the release of Run Once Duration Override Operator 1.4.2.

Issued: 17 September 2026

The following advisory is available for the Run Once Duration Override Operator 1.4.2:

- [RHBA-2026:68626](https://access.redhat.com/errata/RHBA-2026:68626)

### New features and enhancements {#run-once-duration-override-operator-1-4-2-new-features_run-once-duration-override-release-notes}

- This release rebuilds the base image for the Run Once Duration Override Operator to improve its image grade.

## Run Once Duration Override Operator 1.4.1 {#rodoo-rn-1-4-1_run-once-duration-override-release-notes}

Review the features, enhancements, and advisory for the release of Run Once Duration Override Operator 1.4.1.

Issued: 17 June 2026

The following advisory is available for the Run Once Duration Override Operator 1.4.1:

- [RHBA-2026:26526](https://access.redhat.com/errata/RHBA-2026:26526)

### New features and enhancements {#rodoo-1-4-1-new-features-and-enhancements_run-once-duration-override-release-notes}

- This release of the Run Once Duration Override Operator updates the Kubernetes version to 1.35.

### Bug fixes {#rodoo-rn-1-4-1-bug-fixes_run-once-duration-override-release-notes}

- This release of the Run Once Duration Override Operator addresses several Common Vulnerabilities and Exposures (CVEs).

## Run Once Duration Override Operator 1.4.0 {#rodoo-rn-1-4-0_run-once-duration-override-release-notes}

Review the features, enhancements, and advisory for the release of Run Once Duration Override Operator 1.4.0.

Issued: 12 February 2026

The following advisory is available for the Run Once Duration Override Operator 1.4.0:

- [RHBA-2026:2649](https://access.redhat.com/errata/RHBA-2026:2649)

### New features and enhancements {#rodoo-1-4-0-new-features-and-enhancements_run-once-duration-override-release-notes}

- This release of the Run Once Duration Override Operator updates the Kubernetes version to 1.34.
- Users should set `.spec.managementState: Managed` in Run Once Duration Override Operator 1.4.0 custom resources (CR). In a future release, the `spec.managementState` field in the Run Once Duration Override Operator CR will be required to be set to `Managed`.

### Bug fixes {#rodoo-rn-1-4-0-bug-fixes_run-once-duration-override-release-notes}

- This release of the Run Once Duration Override Operator addresses several Common Vulnerabilities and Exposures (CVEs).
