# CKA Troubleshooting — Runbooks

## 1. `CrashLoopBackOff`

```sh
kubectl get pod <pod> -n <ns> -o wide
kubectl describe pod <pod> -n <ns>
kubectl logs <pod> -n <ns> -c <container> --previous
kubectl get events -n <ns> --sort-by=.lastTimestamp
```

Read the previous container log because the current container may have exited. Then check the command/args, image tag, environment and mounted configuration, probes, resource limits (`OOMKilled`), ServiceAccount permissions, and dependencies. Fix the Deployment template—not a one-off Pod—and watch the rollout with `kubectl rollout status deployment/<name>`.

## 2. `NotReady` node

```sh
kubectl describe node <node>
kubectl get pods -A --field-selector spec.nodeName=<node>
kubectl get --raw='/readyz?verbose'
```

On the node, inspect `systemctl status kubelet`, `journalctl -u kubelet`, container-runtime status, disk/inode and memory pressure, CNI configuration, certificates, and network reachability to the API server. Cordon first if the node is unsafe; drain only after understanding workloads and PDB impact. Uncordon after recovery.

## 3. etcd snapshot restore

This is control-plane-specific and must follow the distribution's recovery procedure. The safe sequence is:

1. Identify the correct snapshot and verify it before touching the control plane.
2. Record the current static-Pod manifest and etcd data directory; stop or isolate the affected etcd member(s) as required by your topology.
3. Restore to a **new, empty** data directory using the matching `etcdctl` version and correct certificates/endpoints.
4. Update the etcd static-Pod manifest or service to use the restored directory; in HA, use the documented member-recovery procedure rather than treating it like a single-node restore.
5. Verify etcd and API server health, then verify nodes, namespaces, and critical workloads.

Never overwrite a live etcd data directory casually. Treat snapshots as sensitive: they contain cluster Secrets in encrypted or plaintext form depending on cluster configuration.
