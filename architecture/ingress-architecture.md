# Ingress Architecture in Kubernetes

This file explains Kubernetes Ingress architecture and how it routes external traffic.

## What is Ingress?
- `Ingress` is an API object that defines HTTP/S routing rules for external traffic.
- It provides a single entry point to services in a cluster.

## Components involved
- **Ingress resource**: defines host/path rules.
- **Ingress controller**: implements the rules and configures a load balancer or proxy.
- **Service**: backends referenced by the Ingress.
- **LoadBalancer / NodePort**: exposes the Ingress controller externally.

## Example flow
1. User creates an `Ingress` object with rules like `/api` and `/web`.
2. The Ingress controller watches the object via the API server.
3. The controller updates its proxy configuration.
4. External requests arrive at the Ingress controller endpoint.
5. The controller routes traffic to the appropriate Service.

## Important points
- Ingress is only the rule object; the controller does the actual traffic handling.
- Different controllers exist: NGINX, Traefik, HAProxy, Istio, etc.
- TLS can be terminated at the Ingress controller.
- Ingress can also support rewrites, redirects, and virtual hosts.

## Interview-ready summary
- Ingress provides HTTP/S routing rules.
- The Ingress controller implements those rules.
- Services remain the cluster-internal backends.
- Ingress is the bridge between external traffic and internal services.

## Diagram

![Ingress architecture](ingress-architecture-diagram.svg)
