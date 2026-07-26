# Declarative API Validation

**Declarative API Validation** is a mechanism that allows Kubernetes API validation rules to be defined directly in the Go type definitions using special comment tags (e.g., `+k8s:minimum=1`). This is the modern replacement for writing custom validation logic in separate Go files.

## How it Works

1.  **Tags in Code**: Developers add validation tags to the API type definitions (e.g., in `types.go`).
2.  **Code Generation**: A tool (`validation-gen`) uses these tags to generate optimized Go code for validation.
3.  **API Server Enforcement**: The `kube-apiserver` uses this generated code to validate incoming API requests.

## Rollout and Feature Gates

The rollout of declarative validation for existing APIs is managed carefully using a shadow mode and feature gates to ensure stability.

-   **`+k8s:alpha`**: Puts a new declarative rule into **shadow mode**. The old hand-written validation is still authoritative, but the new rule is also run. Any mismatches are logged in the `declarative_validation_mismatch_total` metric, allowing developers to find bugs without affecting users.

-   **`+k8s:beta`**: Promotes a rule to be enforced. This is controlled by the `DeclarativeValidationBeta` feature gate.

### Key Feature Gates

-   **`DeclarativeValidation` (GA, enabled by default)**: Enables the overall declarative validation framework.
-   **`DeclarativeValidationBeta` (Beta, enabled by default)**: Controls whether rules marked with `+k8s:beta` are actually enforced. If this gate is disabled, `+k8s:beta` rules revert to shadow mode.

## What This Means for Administrators

While primarily a developer feature, administrators should be aware of it for troubleshooting.

### When to Disable `DeclarativeValidationBeta`

You might consider disabling this feature gate (`--feature-gates=DeclarativeValidationBeta=false` on the API server) as a temporary safety measure if you observe:

-   **Unexpected Validation Errors**: New validation rules are incorrectly rejecting valid objects.
-   **Performance Regressions**: A noticeable increase in API server request latency after an upgrade.
-   **High Mismatch Rate**: The `declarative_validation_mismatch_total` metric is high, indicating potential bugs in the new validation logic.

Disabling the gate provides a quick rollback to the older, hand-written validation logic while the issue is investigated.

---
*This document is a summary of the detailed information found in the official Declarative API Validation documentation.*