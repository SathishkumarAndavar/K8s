# Using an HTTP proxy to access the Kubernetes API

This note covers common ways to proxy HTTP(S) traffic to the Kubernetes API server for local development, debugging, and when operating behind corporate proxies.

1) `kubectl proxy` (recommended for local use)

- Starts a local proxy that authenticates using your kubeconfig and forwards requests to the API server.
- Default binds to `127.0.0.1:8001` and restricts hosts for safety.

Example:
```bash
kubectl proxy --port=8001
# then from the same machine:
curl -sS http://127.0.0.1:8001/api/v1/namespaces/default/pods | jq .
```

Options and security:
- `--address=0.0.0.0` exposes the proxy to all interfaces — only use behind a firewall and with additional access controls.
- `--accept-hosts` / `--accept-paths` restrict incoming requests (defaults are conservative).

2) Using corporate HTTP/HTTPS proxy to reach external API servers

- If your environment requires an HTTP(S) proxy to reach external endpoints, set the `HTTPS_PROXY` / `HTTP_PROXY` environment variables for `kubectl` and client tools.
- Use `NO_PROXY` (or `no_proxy`) to bypass the proxy for cluster-internal addresses (API server IP, service CIDR, internal DNS names).

Example:
```bash
export HTTPS_PROXY="http://proxy.company.local:8080"
export NO_PROXY="127.0.0.1,localhost,10.0.0.0/8,api.cluster.example.com"
kubectl get nodes
```

Notes:
- Some proxy appliances rewrite TLS or require explicit CONNECT tunneling — ensure the proxy allows `CONNECT` to the API server's host:port.
- If using a proxy that performs TLS interception, you may need to configure certificate trust for the proxy CA.

3) Direct API calls through proxy using credentials

- You can call the API directly (without `kubectl proxy`) by providing credentials and skipping the proxy for the API server (or routing through it):

Example using a bearer token:
```bash
APISERVER=https://api.cluster.example.com:6443
TOKEN=$(cat /path/to/token)
curl -k -H "Authorization: Bearer $TOKEN" $APISERVER/api/v1/namespaces/default/pods
```

4) Port-forwarding and SSH tunnels

- For single-node access or debugging, `kubectl port-forward` or an SSH tunnel to a bastion host can be safer than exposing `kubectl proxy` on 0.0.0.0.

Example (SSH tunnel):
```bash
# on local
ssh -L 6443:api.cluster.internal:6443 jump-host
# then point curl/kubectl at https://localhost:6443 with proper credentials
```

5) Security considerations

- Avoid binding `kubectl proxy` to public interfaces without authentication and network controls.
- Rotate tokens/certs and use short-lived credentials where possible.
- Ensure `NO_PROXY` includes internal cluster endpoints to prevent unintentional routing through corporate proxies.

6) Troubleshooting tips

- If `kubectl` hangs in a proxied environment, verify `HTTPS_PROXY`/`NO_PROXY` and try `curl` directly to the API server.
- Use `kubectl proxy --v=9` and capture logs for debug information.
- For CRI/runtimes that contact remote registries, ensure `imagePull` traffic has proper proxy/no-proxy configuration.
