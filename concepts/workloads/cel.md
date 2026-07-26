# Common Expression Language (CEL) in Kubernetes

The **Common Expression Language (CEL)** is an open-source, non-Turing complete language used within the Kubernetes API to declare validation rules, policy rules, and other constraints.

## Why it is Used

CEL provides a convenient and efficient alternative to out-of-process webhooks for many extensibility use cases. Because CEL expressions are evaluated directly in the `kube-apiserver`, they are fast and don't introduce external network dependencies for validation.

## Language Overview

-   **Syntax**: CEL has a C-like syntax, designed for short, single-expression programs.
-   **Inputs**: CEL programs operate on variables provided by the context where the expression is used. For example, in CustomResourceDefinition validation, the `self` and `oldSelf` variables refer to the current and previous state of the object.

## Example CEL Expressions

| Rule                                                                 | Purpose                                                              |
| :------------------------------------------------------------------- | :------------------------------------------------------------------- |
| `self.minReplicas <= self.replicas && self.replicas <= self.maxReplicas` | Validate that three replica fields are ordered correctly.            |
| `self.list1.size() > 0`                                              | Validate that a list is not empty.                                   |
| `!has(self.expired) || self.created + self.ttl < self.expired`       | Validate that an expiration date is after a creation date plus a TTL. |
| `self.health.startsWith('ok')`                                       | Validate a string field has a specific prefix.                       |
| `self.metadata.name == 'singleton'`                                  | Validate that an object's name is a specific value.                  |

## Libraries and Features

Kubernetes extends CEL with a rich set of standard and custom libraries to make writing rules easier. These include:

-   **Standard Macros**: `has()`, `all()`, `exists()`, `map()`, `filter()`.
-   **String Functions**: `startsWith()`, `matches()`, `contains()`.
-   **Kubernetes-specific Libraries**:
    -   **List Library**: Functions for working with lists (e.g., set operations).
    -   **Regex Library**: For pattern matching.
    -   **URL Library**: For parsing and validating URLs.
    -   **Quantity Library**: For working with Kubernetes resource quantities (e.g., `1Gi`, `500m`).

---
*This document is a summary of the detailed information found in the official Common Expression Language documentation.*