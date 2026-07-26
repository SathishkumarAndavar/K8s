# Services, Load Balancing, and Networking

Kubernetes networking addresses four main concerns:
1.  Containers within a Pod communicating with each other.
2.  Pod-to-Pod communication across the cluster.
3.  Exposing a set of Pods as a stable network Service.
4.  Allowing external traffic into the cluster.

## Core Networking Resources

-   **Service**: An abstract way to expose an application running on a set of Pods as a network service. Provides a stable IP address and DNS name.
-   **Ingress**: An API object that manages external access to the services in a cluster, typically HTTP. It can provide load balancing, SSL termination, and name-based virtual hosting.
-   **NetworkPolicy**: Specifies how groups of pods are allowed to communicate with each other and other network endpoints.
-   **Pod-to-Pod Communication**: The underlying models for how pods communicate within a node and across nodes.
-   **EndpointSlice**: A scalable and extensible way to track the IP addresses, ports, and readiness of Pods backing a Service.

*See also: Networking Concepts*