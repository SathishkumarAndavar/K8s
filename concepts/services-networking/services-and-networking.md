# Services, Load Balancing, and Networking

This note collects core concepts and when to use them. Refer to the linked Kubernetes docs for deep dives.

- **Service (ClusterIP / NodePort / LoadBalancer):** Logical abstraction that groups pods and provides stable network identity. Use `ClusterIP` for internal communication, `LoadBalancer` for cloud LBs, and `NodePort` for simple external access.
- **Ingress / Ingress Controllers:** Ingress is an API resource describing HTTP(S) routing; Ingress Controller implements it and configures an edge proxy. Use for host/path routing and TLS termination.
- **Gateway API:** Successor to Ingress for richer, API-first routing with CRDs (GatewayClass, Gateway, HTTPRoute) — use when you need multi-tenant, extensible gateway features.
- **EndpointSlices:** Scalable representation of network endpoints for Services; controllers automatically create EndpointSlices.
- **Network Policies:** Declarative firewall rules for pod-to-pod traffic; use to restrict lateral movement and enforce zero-trust segmentation.
- **DNS for Services & Pods:** CoreDNS exposes service and pod DNS names; rely on CoreDNS for service discovery.
- **IPv4/IPv6 dual-stack:** Configure cluster-level dual-stack support for mixed networking requirements.
- **Topology Aware Routing:** Use topology-aware service routing and `topology.kubernetes.io/zone` to prefer local endpoints and reduce cross-zone traffic.
- **Networking on Windows:** Platform-specific constraints; ensure CNI and service routing support Windows nodes.
- **Service ClusterIP allocation & Internal Traffic Policy:** Control ClusterIP allocation ranges and `internalTrafficPolicy` to prefer node-local endpoints.

See the official docs for each topic:

- https://kubernetes.io/docs/concepts/services-networking/
- https://kubernetes.io/docs/concepts/services-networking/service/
- https://kubernetes.io/docs/concepts/services-networking/ingress/
- https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/
- https://kubernetes.io/docs/concepts/services-networking/gateway/
- https://kubernetes.io/docs/concepts/services-networking/endpoint-slices/
- https://kubernetes.io/docs/concepts/services-networking/network-policies/
- https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/
- https://kubernetes.io/docs/concepts/services-networking/dual-stack/
- https://kubernetes.io/docs/concepts/services-networking/topology-aware-routing/
- https://kubernetes.io/docs/concepts/services-networking/windows-networking/
- https://kubernetes.io/docs/concepts/services-networking/cluster-ip-allocation/
- https://kubernetes.io/docs/concepts/services-networking/service-traffic-policy/
