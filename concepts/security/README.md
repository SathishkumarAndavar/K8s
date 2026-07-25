# Extending Kubernetes

This section covers how to extend the Kubernetes API and add custom functionality to your cluster.

## Core Extension Concepts

-   **Custom Resources (CRDs)**: A powerful feature that allows you to define your own API objects in Kubernetes. This is the foundation for building custom controllers and operators.
-   **Operator Pattern**: A method of packaging, deploying, and managing a Kubernetes application. An Operator is a custom controller that uses CRDs to manage an application and its components.
-   **API Aggregation Layer**: Allows you to extend Kubernetes with more APIs by running an "extension API server" alongside the main API server.

*See also: Extending Kubernetes Concepts*