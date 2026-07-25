# Validate Node Setup

Checklist to validate a node is correctly configured for Kubernetes:

- **Kubelet & kube-proxy:** Ensure services are running (`systemctl status kubelet`) and kube-proxy is healthy.
- **Container runtime & cgroup driver:** Verify Docker/containerd and cgroup driver match kubelet configuration.
- **Kernel & modules:** Confirm required modules (overlay, br_netfilter) are loaded and `sysctl` settings (`net.bridge.bridge-nf-call-iptables=1`).
- **Time sync:** NTP/chrony is running and clock skew is low.
- **Networking:** Node can reach API server; pod CIDR routes exist; no host firewall blocking kubelet or kube-proxy.
- **Storage:** Disk capacity and IOPS checks; mount options and permissions for PVs.
- **Security:** SELinux/AppArmor mode, necessary kernel params, and required capabilities.
- **Node labels & taints:** Labels for scheduling and taints for special nodes correctly applied.

Useful commands:
```bash
kubectl get nodes
kubectl describe node <node-name>
ssh node -- "systemctl status kubelet && docker info || containerd --version"
kubectl logs -n kube-system kubelet-<pod> # where applicable
```

Notes:
- Automate checks via a bootstrap script or a node-exporter health job.
- Validate new node types in a test pool before wide rollout.
