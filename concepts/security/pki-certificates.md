# PKI Certificates and Requirements

Summary:
- Kubernetes components use TLS for authentication and encryption. Proper PKI design and rotation are essential.

Key certificates:
- **kube-apiserver server certs:** Include SANs for API server IPs and DNS (kubernetes.default, load balancer).
- **etcd TLS:** Client and peer certs for etcd; ensure mutual TLS.
- **kubelet client certs & server certs:** Kubelet may present server certs for metrics; kube-apiserver authenticates kubelet clients.
- **front-proxy & aggregator:** For API aggregation.

Requirements & best practices:
- Use RSA 2048/3072 or ECDSA keys (P-256/P-384); prefer ECDSA for smaller size and performance when supported.
- Include correct SANs for all endpoints (VIPs, node hostnames, cluster DNS).
- Automate certificate issuance and rotation (cert-manager, small-scripts, or cloud CA integrations).
- Secure private keys and limit access; use hardware-backed KMS for critical keys where possible.

Quick example (openssl CSR + self-signed for testing only):
```bash
openssl req -new -newkey rsa:2048 -nodes -keyout apiserver.key -subj "/CN=kube-apiserver" -out apiserver.csr
openssl x509 -req -in apiserver.csr -signkey apiserver.key -days 365 -out apiserver.crt
```

Notes:
- For production, prefer an organizational CA or cloud-managed PKI and enable automatic rotation. Avoid long-lived static certs.
