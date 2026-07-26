# Deprecated API Migration Guide

As the Kubernetes API evolves, older API versions are deprecated and eventually removed. This guide summarizes key API removals and the required migration path.

## General Migration Process

When an API version is removed, you must:
1.  Update your YAML manifests to use the new, stable API version.
2.  Update any client tools, scripts, or controllers that interact with the deprecated API.
3.  Run `kubectl convert` or other tools to help migrate existing manifests before upgrading the cluster.

---

## Removals by Kubernetes Version

### v1.32
-   **Removed**: `flowcontrol.apiserver.k8s.io/v1beta3` (FlowSchema, PriorityLevelConfiguration)
-   **Migrate to**: `flowcontrol.apiserver.k8s.io/v1` (available since v1.29)

### v1.29
-   **Removed**: `flowcontrol.apiserver.k8s.io/v1beta2` (FlowSchema, PriorityLevelConfiguration)
-   **Migrate to**: `flowcontrol.apiserver.k8s.io/v1` (available since v1.29)

### v1.27
-   **Removed**: `storage.k8s.io/v1beta1` (CSIStorageCapacity)
-   **Migrate to**: `storage.k8s.io/v1` (available since v1.24)

### v1.26
-   **Removed**: `autoscaling/v2beta2` (HorizontalPodAutoscaler)
-   **Migrate to**: `autoscaling/v2` (available since v1.23)

### v1.25
-   **Removed**: `batch/v1beta1` (CronJob)
-   **Migrate to**: `batch/v1` (available since v1.21)

---
*This is a summary. Always check the official Kubernetes release notes for a complete list of API changes before upgrading a cluster.*