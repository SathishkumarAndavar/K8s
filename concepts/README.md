# Kubernetes Concepts

Focused Kubernetes notes, grouped by subject.

## Workloads

- [Deployment YAML fields](workloads/deployment-yaml-fields.md)
- [Workload types comparison](workloads/workload-types-comparison.md)
- [Workload controllers](workloads/workload-controllers.md)
- [Workloads and controllers overview](workloads/workloads-and-controllers-overview.md)
- [Jobs vs Pods](workloads/jobs-vs-pods.md)
- [Init containers, sidecars, and probes](workloads/init-containers-sidecars-and-probes.md)

## Scheduling

- [Node selectors and affinity](scheduling/node-selectors-and-affinity.md)
- [Taints and tolerations](scheduling/taints-and-tolerations.md)
- [Topology](scheduling/topology.md)
- [Descheduler](scheduling/descheduler.md)

## Workload autoscaling

- [HPA vs VPA](workloads/autoscaling/hpa-vs-vpa.md)

## Services, load balancing, and networking

- [Ingress vs Ingress Controller](services-networking/ingress-vs-ingress-controller.md)
- [Ingress Controller vs Ingress overview](services-networking/ingress-controller-vs-ingress-overview.md)
- [Services and networking](services-networking/services-and-networking.md)
- [User-to-Pod traffic flow](services-networking/user-to-pod-traffic-flow.md)

## Storage

- [PersistentVolumes, StorageClasses, and claims](storage/persistent-volumes-storageclasses-and-claims.md)
- [PV, StorageClass, and PVC overview](storage/pv-storageclass-pvc-overview.md)

## Security

- [Pod Security Standards](security/pod-security-standards.md)
- [Pod Security Standards enforcement](security/pod-security-standards-enforcement.md)
- [PKI certificates](security/pki-certificates.md)

## Containers

- [Container environment](containers/container-environment.md)
- [Container Runtime Interface](containers/container-runtime-interface.md)
- [RuntimeClass](containers/runtime-class.md)

## Cluster administration

- [Multi-zone deployments](cluster-administration/multi-zone-deployments.md)
- [Multi-zone considerations](cluster-administration/multi-zone-considerations.md)
- [Large-cluster considerations](cluster-administration/large-cluster-considerations.md)
- [Node setup validation](cluster-administration/node-setup-validation.md)
- [Control-plane communication](cluster-administration/control-plane-communication.md)
- [API server HTTP proxy](cluster-administration/api-server-http-proxy.md)
- [Swap memory management](cluster-administration/swap-memory-management.md)
- [Cluster and node upgrades](cluster-administration/cluster-and-node-upgrades.md)
