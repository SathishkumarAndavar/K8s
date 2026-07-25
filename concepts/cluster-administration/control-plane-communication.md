# Communication between Nodes and the Control Plane

Summary:
- Nodes (kubelet, kube-proxy) communicate with the control plane (kube-apiserver) over API server endpoints. Ensure secure, reliable connectivity.

Key channels:
- **kubelet ↔ kube-apiserver:** kubelet uses HTTPS to the API server; mutual TLS or client certs commonly used.
- **api-server ↔ controllers:** control plane components talk to etcd and each other; network policies typically not applied to control-plane internals.
- **kube-proxy & networking:** kube-proxy programs iptables/ipvs based on API server state.

Networking requirements:
- **Firewall rules:** Allow nodes to reach API server ports (default 6443) and etcd where applicable. Control plane should reach nodes for SSH/probes if needed.
- **DNS & Service CIDR:** Ensure DNS resolution for control-plane load-balancers and correct routing for pod/service ranges.
- **Certificates & auth:** Use client certs, tokens, or cloud IAM integration for node authentication.

Best practices:
- Place API servers behind an HA load balancer with health checks.
- Use dedicated control-plane subnets and restrict access via security groups/firewalls.
- Monitor API server latency and dropped connections; instrument kubelet connectivity.

Notes:
- Update architecture diagrams to show API server endpoints, control-plane LB, and secure paths (TLS/mTLS).
