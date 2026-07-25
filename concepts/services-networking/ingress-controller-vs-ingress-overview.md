# Ingress Controller vs Ingress

Summary:
- **Ingress (resource):** API object that defines routing rules (host/path → service).
- **Ingress Controller:** The implementation that watches Ingress resources and configures the underlying proxy (NGINX, Traefik, HAProxy, cloud LB).

When to use:
- Use an **Ingress resource** to declare HTTP(S) routing inside the cluster.
- Deploy an **Ingress Controller** when you need a cluster-wide HTTP(S) entrypoint. Choose controller based on features (TLS, WebSocket, rate-limiting, CRDs, annotations).

Examples:
- Basic Ingress resource (assumes controller is installed):
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

- Installing a controller (example: NGINX ingress via Helm):
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace
```

Notes:
- Cloud environments often provide managed controllers (GCP/ALB/Contour) with additional integration (LB, TLS automation).
- For advanced routing (canary, header-based), evaluate controllers that support custom CRDs (e.g., Traefik, Istio Gateway, Contour).
