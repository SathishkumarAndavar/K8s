# Swap Memory Management

Summary:
- Kubernetes expects swap to be disabled on nodes; kubelet refuses to start when swap is enabled unless explicitly allowed (not recommended).

When to use:
- Disable swap for predictable performance and accurate scheduler decisions. Exception: experiments or special workloads with careful validation.

How to disable swap:
```bash
sudo swapoff -a
# Persist by removing or commenting swap entries in /etc/fstab
```

If enabling swap (not recommended):
- Configure `--fail-swap-on=false` on kubelet (risky). Kubernetes scheduling and memory eviction behavior assumes no swap.

Memory management tips:
- Set resource `requests` and `limits` to let scheduler make correct placement decisions.
- Use `oomScoreAdj`, `oom_kill_disable` only with caution; prefer proper requests/limits.

Notes:
- On systems with zswap/zram enabled, test kubelet behavior thoroughly; prefer node tuning over enabling swap.
