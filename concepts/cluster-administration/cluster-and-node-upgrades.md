# Kubernetes Cluster Upgrade Process

## Interview answer

Upgrade one supported **minor version at a time**, moving from the control plane to worker nodes. First validate compatibility and backups; then upgrade one primary control-plane node, the remaining control-plane nodes sequentially, and workers in batches that preserve application capacity. Drain each node before a minor-version kubelet upgrade, validate health after every step, and keep a tested recovery plan.

```text
Plan + backup
     ↓
Primary control plane
     ↓
Other control-plane nodes (one at a time)
     ↓
CNI / compatible add-ons
     ↓
Workers (drain → upgrade → validate → uncordon)
     ↓
Cluster and workload validation
```

## 1. Plan the change

- Confirm the cluster type: **managed** (EKS, GKE, AKS, and so on), **kubeadm**, or custom deployment. Use the provider's documented workflow for a managed control plane.
- Read release notes, removed/deprecated API notices, and the Kubernetes version-skew policy.
- Upgrade to the latest patch of the current minor version first, then the latest patch of the target minor version.
- Do not skip Kubernetes minor versions. In an HA cluster, the oldest and newest API servers must be no more than one minor version apart.
- Check compatibility for CNI, CSI, CoreDNS, kube-proxy, admission webhooks, device plugins, ingress controllers, monitoring, and critical operators.
- Test the exact plan in staging, including application behavior, PDBs, backups, and recovery.
- Choose a maintenance window and batch size that retain enough Ready replicas and node capacity.

## 2. Establish a recovery point

- Take and verify an etcd snapshot for self-managed etcd. Store it outside the cluster and protect it as sensitive data because it contains cluster state and potentially Secrets.
- Back up application data separately; etcd does **not** back up database data stored in PVCs or external services.
- Export manifests, GitOps state, package versions, and node configuration. Ensure someone has console access if the API becomes unavailable.
- Define rollback boundaries: package rollback can help a failed node upgrade, but control-plane rollback and etcd restore are separate, higher-risk recovery operations.

## 3. Pre-flight checks

```sh
kubectl version
kubectl get nodes -o wide
kubectl get pods -A
kubectl get --raw='/readyz?verbose'
kubectl get events -A --sort-by=.lastTimestamp
```

Resolve existing control-plane errors, NotReady nodes, Pending workloads, and failing critical Pods before beginning. An upgrade should not be the first time you investigate existing instability.

## 4. kubeadm: upgrade the first control-plane node

Run this on the control-plane node that has `/etc/kubernetes/admin.conf`. Replace `<target>` with the chosen supported release, such as `v1.36.x`; use your operating system's package instructions and the matching `pkgs.k8s.io` repository for the target minor release.

```sh
# Install the target kubeadm package version, then verify it.
kubeadm version
sudo kubeadm upgrade plan

# Apply the selected target version on the first control-plane node.
sudo kubeadm upgrade apply <target>
```

`kubeadm upgrade plan` checks the upgrade path and shows component configuration. `kubeadm upgrade apply` upgrades the control-plane static Pods and kubeadm-managed add-ons at the appropriate point in the HA workflow; it does not upgrade kubelet packages for you.

Then drain, update the node binaries, restart kubelet, validate, and return the node to scheduling:

```sh
kubectl drain <control-plane-node> --ignore-daemonsets

# Upgrade kubelet and kubectl to the chosen supported package version.
sudo systemctl daemon-reload
sudo systemctl restart kubelet

kubectl get node <control-plane-node>
kubectl uncordon <control-plane-node>
```

Only add `--delete-emptydir-data` if the impact is understood; it deletes Pod-local `emptyDir` data. Respect PodDisruptionBudgets rather than bypassing them by default.

## 5. kubeadm: remaining control-plane nodes

Upgrade each remaining control-plane node **one at a time**:

1. Install the target `kubeadm` package.
2. Run `sudo kubeadm upgrade node`.
3. Drain the node, upgrade kubelet and kubectl, restart kubelet.
4. Confirm it is `Ready`, check critical control-plane Pods, then uncordon it.

Sequential control-plane upgrades preserve API availability and keep the API-server version skew within policy. In HA deployments, wait for each node to recover before starting the next one.

## 6. Upgrade CNI and add-ons

Follow the CNI vendor's instructions; network plugins commonly run as DaemonSets and may require a compatible version before or during the node rollout. Also verify CSI drivers, CoreDNS, kube-proxy, ingress controllers, device plugins, and admission webhooks. A control plane can be healthy while networking or storage is not.

## 7. kubeadm: upgrade workers

Process workers one at a time or in a carefully sized batch:

```sh
# From an administrative host
kubectl drain <worker> --ignore-daemonsets

# On the worker: install target kubeadm, then refresh kubelet configuration
sudo kubeadm upgrade node
# Upgrade kubelet and kubectl packages, then:
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# From an administrative host
kubectl get node <worker>
kubectl uncordon <worker>
```

Before a **minor-version** kubelet upgrade, drain the node. Do not proceed to the next batch until nodes are Ready and workloads have recovered.

## 8. Validate after each stage and at completion

```sh
kubectl get nodes
kubectl get pods -A
kubectl get deployment,statefulset,daemonset -A
kubectl get events -A --sort-by=.lastTimestamp
kubectl rollout status deployment/<critical-workload> -n <namespace>
```

Validate more than version numbers: DNS, Service-to-Pod traffic, storage mount/attach behavior, ingress, autoscaling, scheduled jobs, and application SLOs. Monitor API error rate, node conditions, and workload availability during the rollout.

## Failure handling and rollback

- `kubeadm upgrade` is designed to be repeatable; fix the underlying issue and rerun the same command before attempting broad rollback actions.
- Kubeadm keeps temporary manifest and local-etcd backup directories under `/etc/kubernetes/tmp`; use them only with the documented recovery procedure.
- For a failed worker, keep it cordoned, restore a known-good image/package set or replace the node, then validate it before uncordoning.
- For control-plane or etcd recovery, stop and follow the tested restore runbook. Do not overwrite a live etcd data directory impulsively.

## Managed Kubernetes clusters

Managed control planes remove some kubeadm steps but not the responsibility to plan. Typical order is: upgrade control plane through the provider, update compatible add-ons, roll managed node groups or replacement nodes, then validate workloads. Always follow the provider's supported version path and rollback model.

## Key version-skew facts

- `kubelet` must never be newer than the API server and can normally be up to three minor versions older.
- `kube-proxy` must not be newer than the API server and can normally be up to three minor versions older.
- `kubectl` is supported within one minor version of the API server.
- Deployment tooling can impose stricter requirements than upstream policy.

## Official references

- [Upgrade a cluster](https://kubernetes.io/docs/tasks/administer-cluster/cluster-upgrade/)
- [Upgrading kubeadm clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
- [Version skew policy](https://kubernetes.io/releases/version-skew-policy/)
