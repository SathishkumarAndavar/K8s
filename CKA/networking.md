# CKA Networking — Answers and Practice

## 1. Service types

- `ClusterIP` is reachable only inside the cluster and is the normal default.
- `NodePort` exposes a port on each node; it is mainly a building block or simple lab exposure method.
- `LoadBalancer` asks a supported implementation to provision an external load balancer; it also includes Service routing inside the cluster.

```sh
kubectl expose deployment web --name=web --port=80 --target-port=8080
kubectl get svc web
kubectl get endpointslice -l kubernetes.io/service-name=web
```

A Service selects Pods by labels and sends traffic to ready endpoints. If there are no EndpointSlices, first check the Service selector and Pod labels.

## 2. Debug DNS from the Pod outward

```sh
kubectl exec -it <pod> -- cat /etc/resolv.conf
kubectl exec -it <pod> -- nslookup kubernetes.default.svc.cluster.local
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl get svc -n kube-system kube-dns
kubectl get endpointslice -n kube-system -l kubernetes.io/service-name=kube-dns
```

Expected result: the fully-qualified service name resolves to the cluster IP. If it fails, distinguish Pod DNS configuration, CoreDNS availability/logs, the kube-dns Service/endpoints, and the CNI/network path. Test a direct Service IP separately from DNS.

## 3. Allow ingress only from one namespace

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-monitoring
  namespace: app
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes: [Ingress]
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: monitoring
    ports:
    - protocol: TCP
      port: 8080
```

NetworkPolicy works only if the installed CNI enforces it. Once a Pod is selected by an ingress policy, only explicitly allowed ingress reaches it; return traffic is normally allowed because policies are stateful.
