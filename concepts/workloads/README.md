# Kubernetes API Reference

This section provides a reference for the Kubernetes API itself.

*   **[API Concepts](./api-concepts.md)**: A detailed guide to API terminology, verbs, resource versions, and interaction patterns.
## Key API Concepts

-   **API Groups**: The Kubernetes API is grouped for extensibility. Core resources are in the `core` group (often seen as `/api/v1`), while others are in named groups like `apps/v1`, `batch/v1`, and `networking.k8s.io/v1`.
-   **Server-Side Apply**: A mechanism for controllers and users to manage resource ownership declaratively.
-   **Deprecation Policy & Migration**: How and when API versions are removed, and how to migrate to newer versions. See the Deprecated API Migration Guide.
-   **API Health Endpoints**: The API server exposes health endpoints like `/livez` and `/readyz` to monitor its status.
-   **API Priority and Fairness (Flow Control)**: How the API server manages high request volumes to maintain stability.

## API Access and Control

-   **API Access Control**: A detailed look at how Kubernetes secures access to the API through authentication, authorization, and admission control.

*See also: Kubernetes API Concepts*