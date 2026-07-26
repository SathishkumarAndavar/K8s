# Audit Annotations

When auditing is enabled in a Kubernetes cluster, the API server generates audit events. These events can be enriched with annotations that provide additional context about the request and its processing. This page serves as a reference for these annotations.

**Note:** These annotations apply to `Event` objects from the `audit.k8s.io` API group, not the standard `events.k8s.io` events you see with `kubectl get events`.

## Deprecation Annotations

-   **`k8s.io/deprecated`**: Set to `"true"` if the request used a deprecated API version.
-   **`k8s.io/removed-release`**: Indicates the Kubernetes version in which a deprecated API is scheduled for removal (e.g., `"1.22"`).

## Pod Security Annotations

-   **`pod-security.kubernetes.io/exempt`**: Indicates why a pod was exempted from Pod Security Standard enforcement (e.g., `namespace`, `user`).
-   **`pod-security.kubernetes.io/enforce-policy`**: Shows the Pod Security Standard level and version that was enforced (e.g., `restricted:latest`).
-   **`pod-security.kubernetes.io/audit-violations`**: Details the specific policy violations that were audited.

## Authorization Annotations

-   **`authorization.k8s.io/decision`**: The result of the authorization check, either `"allow"` or `"forbid"`.
-   **`authorization.k8s.io/reason`**: A human-readable reason for the authorization decision.

## Performance and Latency Annotations

-   **`apiserver.latency.k8s.io/etcd`**: The time taken to send the request to `etcd` and receive a response.
-   **`apiserver.latency.k8s.io/decode-response-object`**: The time taken to decode the response from `etcd`.
-   **`apiserver.latency.k8s.io/apf-queue-wait`**: The time a request spent queued by API Priority and Fairness (APF).

## Certificate and Security Warning Annotations

-   **`missing-san.invalid-cert.kubernetes.io/$hostname`**: Indicates a webhook or aggregated API server is using a certificate that relies on the legacy Common Name (CN) field instead of a Subject Alternative Name (SAN).
-   **`insecure-sha1.invalid-cert.kubernetes.io/$hostname`**: Indicates a webhook or aggregated API server is using a certificate signed with the insecure SHA-1 algorithm.

## Other Annotations

-   **`audit.k8s.io/truncated`**: Set to `"true"` if the audit event was too large and had to be truncated.
-   **`validation.policy.admission.k8s.io/validation_failure`**: A JSON object detailing a failure from a `ValidatingAdmissionPolicy`.

---
*This document is a summary of the detailed information found in the official Audit Annotations documentation.*