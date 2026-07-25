# Container Runtime Interface (CRI)

Summary:
- CRI is the pluggable API between kubelet and container runtimes (containerd, CRI-O). It defines how the kubelet starts, stops, and inspects containers.

Key points:
- **Socket:** kubelet connects to a runtime socket (e.g., `/var/run/dockershim.sock` or `/run/containerd/containerd.sock`).
- **Runtime responsibilities:** image pulling, container lifecycle, snapshots, runtime classes (where supported).
- **Deprecation note:** `dockershim` removal means Docker Engine is no longer directly used; use containerd or CRI-O as CRI implementations.

Troubleshooting:
- Check kubelet config for `container-runtime` and `container-runtime-endpoint`.
- Inspect runtime logs (`journalctl -u containerd`) and use `crictl` for debugging (`crictl ps`, `crictl images`).

Example kubelet flag (static):
```
--container-runtime=remote --container-runtime-endpoint=unix:///run/containerd/containerd.sock
```
