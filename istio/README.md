# Istio Service Mesh

## What it does

Istio adds a traffic-management, security, and observability layer to Kubernetes workloads. The control plane (`istiod`) distributes configuration and identity information; Envoy data-plane proxies enforce the traffic behavior. Istio supports sidecar mode and ambient mode, which has a different data-plane architecture.

## Core concepts

- **Gateway / Gateway API:** receives or controls edge traffic.
- **VirtualService or HTTPRoute:** describes host, path, header, and weighted traffic routing.
- **DestinationRule:** configures traffic policy such as subsets, load balancing, TLS, and outlier detection.
- **PeerAuthentication / AuthorizationPolicy:** configure mTLS and workload-level access control.
- **ServiceEntry:** registers external services in the mesh.

## How traffic works

In sidecar mode, a workload Pod's inbound and outbound traffic is intercepted by an Envoy sidecar. Envoy applies routing, retries/timeouts, telemetry, and mTLS according to configuration received from `istiod`. In ambient mode, traffic handling is split between node-level and waypoint components instead of one proxy per workload Pod.

## Essential commands

```sh
# Install or verify a default profile in a test cluster
istioctl install --set profile=default -y
istioctl verify-install

# Enable sidecar injection for a namespace, then restart workload Pods
kubectl label namespace <namespace> istio-injection=enabled
kubectl rollout restart deployment -n <namespace>

# Diagnose configuration and data plane
istioctl analyze -n <namespace>
istioctl proxy-status
istioctl proxy-config routes <pod> -n <namespace>
kubectl get gateway,virtualservice,destinationrule -A
```

## Add-ons and observability

Kiali, Prometheus, Grafana, and Jaeger/Zipkin are commonly used for evaluation and observability. Install only the components you operate and secure in production; sample add-ons are not a production observability design.

## Complete traffic-splitting example

The Service must select Pods labeled with `app: reviews` and versions `v1`/`v2`. Use fully qualified service names so the rule remains unambiguous across namespaces.

```yaml
apiVersion: networking.istio.io/v1
kind: DestinationRule
metadata:
  name: reviews
  namespace: production
spec:
  host: reviews.production.svc.cluster.local
  trafficPolicy:
    loadBalancer: { simple: LEAST_REQUEST }
  subsets:
  - name: v1
    labels: { version: v1 }
  - name: v2
    labels: { version: v2 }
---
apiVersion: networking.istio.io/v1
kind: VirtualService
metadata:
  name: reviews
  namespace: production
spec:
  hosts: [reviews.production.svc.cluster.local]
  http:
  - match:
    - uri: { prefix: /api/ }
    route:
    - destination: { host: reviews.production.svc.cluster.local, subset: v1, port: { number: 8080 } }
      weight: 90
    - destination: { host: reviews.production.svc.cluster.local, subset: v2, port: { number: 8080 } }
      weight: 10
    retries: { attempts: 2, perTryTimeout: 2s }
    timeout: 5s
```

## Important operational guidance

- Begin with explicit timeouts, retries, and mTLS policy; unbounded retries can amplify outages.
- Test sidecar/ambient compatibility with your CNI, network policies, probes, and application protocols.
- Use revision-based upgrades or a controlled canary approach for production mesh upgrades.
- Do not use `EnvoyFilter` as the first customization tool; it has a high compatibility and maintenance cost.

## Interview distinction

Kubernetes Services provide basic discovery and load balancing. Istio adds policy-driven Layer 7 routing, mTLS identity, retries, telemetry, and traffic shifting without requiring every application to implement those concerns itself.

## Official references

- [Istio installation guides](https://istio.io/latest/docs/setup/install/)
- [Istio traffic-management API](https://istio.io/latest/docs/reference/config/networking/)
