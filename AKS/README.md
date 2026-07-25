# Azure Kubernetes Service (AKS)

## How it works

AKS manages the Kubernetes control plane. You manage node pools, workloads, Kubernetes resources, and selected integrations. Node pools are the unit for VM size, OS, availability-zone, autoscaling, and upgrade policy; keep system and user workloads separated where practical.

## Important features

- **Identity:** use Microsoft Entra ID for Kubernetes API authentication and Azure RBAC/Kubernetes RBAC for authorization. Prefer managed identities and Microsoft Entra Workload ID for Azure access from Pods.
- **Networking:** choose and plan Azure CNI or overlay networking, network policy, private-cluster access, and outbound routing before production rollout.
- **Storage:** AKS integrates CSI drivers for Azure Disk, Azure Files, and Blob scenarios; match access mode and performance to the workload.
- **Compute:** system/user node pools, cluster autoscaler, availability zones, Spot capacity, Windows nodes, and confidential/GPU options depend on configuration and region.
- **Observability and policy:** Azure Monitor/Container Insights, Azure Policy, Defender, and managed Prometheus/Grafana can be integrated according to requirements.

## Essential commands

```sh
az aks create \
  --resource-group <rg> \
  --name <cluster> \
  --location <region> \
  --node-count 3 \
  --node-vm-size Standard_D4s_v5 \
  --enable-managed-identity \
  --enable-oidc-issuer \
  --enable-workload-identity \
  --enable-aad \
  --enable-azure-rbac \
  --network-plugin azure \
  --network-plugin-mode overlay \
  --pod-cidr 192.168.0.0/16 \
  --enable-cluster-autoscaler \
  --min-count 2 --max-count 6 \
  --generate-ssh-keys
az aks get-credentials --resource-group <rg> --name <cluster>
kubectl get nodes -o wide

az aks show --resource-group <rg> --name <cluster>
az aks nodepool list --resource-group <rg> --cluster-name <cluster>
az aks nodepool add --resource-group <rg> --cluster-name <cluster> --name apps --node-count 3
az aks get-upgrades --resource-group <rg> --name <cluster>
```

## Add-ons and extensions

Common AKS integrations include monitoring, Azure Policy, Key Vault Secrets Provider, application routing, and workload identity. Discover enabled add-ons before changing them:

```sh
az aks addon list --resource-group <rg> --name <cluster>
az aks addon show --resource-group <rg> --name <cluster> --addon <addon-name>
```

Treat add-ons as versioned production dependencies. Check prerequisites, identity permissions, network reachability, and compatibility before enabling or upgrading one.

## Security guidance

Use Entra ID authentication in production and restrict cluster-admin credential access. Local admin credentials can bypass centralized identity controls; disable local accounts when the operational break-glass design supports it.

## Interview distinction

AKS couples Kubernetes with Azure identities, node-pool lifecycle, and Azure networking/storage. The important question is not only “how do Pods run?” but also “which managed identity, subnet, and node pool owns this path?”

## Official references

- [AKS CLI reference](https://learn.microsoft.com/en-us/cli/azure/aks)
- [AKS cluster authentication](https://learn.microsoft.com/en-us/azure/aks/concepts-cluster-authentication)
