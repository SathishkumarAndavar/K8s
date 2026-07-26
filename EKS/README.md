# Amazon EKS

## How it works

Amazon EKS provides a managed Kubernetes control plane. You are responsible for the data plane, which can be EKS Managed Node Groups (EC2), self-managed EC2 nodes, or serverless with AWS Fargate. Worker nodes run in your VPC and are your responsibility to provision and scale.

## Important features

- **Managed control plane:** AWS operates API servers and etcd; you still own workload availability, Kubernetes objects, node capacity, and add-on compatibility.
- **Compute:** managed node groups provide an AWS-managed node lifecycle; use separate groups for system, application, GPU, Spot, or zone-specific workloads.
-   **EKS Managed Node Groups vs. Self-Managed Nodes**:
    -   **Managed Node Groups**: AWS handles node updates, patching, and graceful termination via an Auto Scaling Group. This is the recommended approach for most workloads.
    -   **Self-Managed Nodes**: You are fully responsible for the EC2 instances, including AMI updates, instance health, and cluster joining logic. This offers more control but requires more operational overhead.
- **Networking:** the Amazon VPC CNI gives Pods VPC-routable addresses. Plan VPC/subnet IP capacity before high Pod density.
- **Identity:** use **EKS Pod Identity** (recommended) or the older IAM Roles for Service Accounts (IRSA) to provide AWS permissions to Pods. Avoid using node IAM profiles for applications.
- **Storage:** use the EBS CSI driver for block volumes and EFS CSI driver for shared file workloads.
- **Load balancing:** use the AWS Load Balancer Controller when you need ALB/NLB resources from Kubernetes Services or Ingress/Gateway resources.

## Essential commands

```sh
# Authenticate kubectl to the cluster
aws eks update-kubeconfig --region <region> --name <cluster>
kubectl get nodes -o wide

# Inspect control plane and managed node groups
aws eks describe-cluster --name <cluster> --region <region>
aws eks list-nodegroups --cluster-name <cluster> --region <region>
aws eks describe-nodegroup --cluster-name <cluster> --nodegroup-name <group> --region <region>

# eksctl alternatives
eksctl get cluster --region <region>
eksctl get nodegroup --cluster <cluster> --region <region>
eksctl scale nodegroup --cluster <cluster> --name <group> --nodes 3 --nodes-min 2 --nodes-max 5 --wait
```

## Add-ons

Core add-ons are commonly VPC CNI, CoreDNS, and kube-proxy. Common operational add-ons include EBS CSI, EFS CSI, Metrics Server, AWS Load Balancer Controller, Pod Identity Agent, observability, and snapshot-controller components. Manage versions deliberately and check compatibility with the target EKS/Kubernetes version.

```sh
aws eks list-addons --cluster-name <cluster> --region <region>
aws eks describe-addon --cluster-name <cluster> --addon-name vpc-cni --region <region>
eksctl get addon --cluster <cluster> --region <region>
```

For recent eksctl-created clusters, use `eksctl update addon` to update EKS-managed add-ons rather than older `eksctl utils update-*` commands.

## Complete eksctl example

Replace every placeholder before applying. This is a template, not a universal production configuration: subnet IDs, IAM roles, Kubernetes version, instance types, and add-on versions must be selected for the account and region.

```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: <cluster-name>
  region: <aws-region>
  version: "<kubernetes-version>"
iam:
  withOIDC: true
vpc:
  subnets:
    private:
      <aws-region>a: { id: subnet-aaaaaaaa }
      <aws-region>b: { id: subnet-bbbbbbbb }
managedNodeGroups:
- name: system
  instanceType: m7i.large # Choose instance types appropriate for your workload and region availability.
  privateNetworking: true
  minSize: 2
  maxSize: 5
  desiredCapacity: 3
  volumeSize: 80
  volumeType: gp3
  volumeEncrypted: true
  labels: { workload: system }
  taints:
  - key: CriticalAddonsOnly
    effect: NoSchedule
  updateConfig:
    maxUnavailable: 1
addons:
- name: vpc-cni
- name: coredns
  # You can override default addon configurations
  configurationValues: |-
    replicaCount: 2
- name: kube-proxy
- name: aws-ebs-csi-driver
# Add another node group for memory-intensive applications
- name: memory-apps
  instanceType: r7i.large # 'r' series for memory-optimized
  privateNetworking: true
  minSize: 1
  maxSize: 10
  desiredCapacity: 1
  labels: { workload: memory-apps, "node.kubernetes.io/instance-type": r7i.large }
  taints:
    - key: memory-apps
      value: "true"
      effect: NoSchedule
# Add a node group for compute-intensive applications
- name: compute-apps
  instanceType: c7i.large # 'c' series for compute-optimized
  privateNetworking: true
  minSize: 1
  maxSize: 10
  desiredCapacity: 1
  labels: { workload: compute-apps, "node.kubernetes.io/instance-type": c7i.large }
  taints:
    - key: compute-apps
      value: "true"
      effect: NoSchedule
```

```sh
eksctl create cluster -f cluster.yaml
eksctl get addon --cluster <cluster-name> --region <aws-region>
```

## Upgrade order

1.  **Plan**: Review the EKS target-version compatibility guide, checking for required add-on version updates (VPC CNI, CoreDNS, kube-proxy, EBS CSI driver) and any deprecated APIs your workloads use.
2.  **Backup**: Take an `etcd` snapshot if you have a process for it (though AWS manages this) and back up critical application state (databases, PVCs).
3.  **Upgrade Control Plane**: Initiate the control plane upgrade via the AWS Console, CLI, or IaC tool. This is a non-disruptive process managed by AWS. Wait for it to complete.
4.  **Upgrade Add-ons**: After the control plane is upgraded, update the core EKS add-ons (`vpc-cni`, `coredns`, `kube-proxy`, `aws-ebs-csi-driver`) to versions compatible with the new control plane.
5.  **Upgrade Data Plane (Nodes)**:
    -   **For EKS Managed Node Groups**: Initiate a version update for the node group. EKS will perform a rolling update, creating new nodes with the updated AMI and draining old nodes, while respecting PDBs.
    -   **For Karpenter or Self-Managed Nodes**: You must define a new `EC2NodeClass` with an updated AMI or launch new instances with the new EKS-optimized AMI, then drain and terminate the old nodes.
6.  **Validate**: After the upgrade, thoroughly test the cluster: check node status, verify core services (DNS), test application connectivity, storage, and ingress.

## Interview distinction

EKS manages the control plane, but it does not remove responsibility for VPC design, IAM boundaries, node/workload lifecycle, Kubernetes add-ons, or application reliability.

## Templating and IaC

-   **eksctl**: Excellent for demos, labs, and simple cluster setups.
-   **AWS CDK / Terraform**: The standard for production environments. Use the official EKS Blueprints for Terraform or the AWS EKS CDK constructs to define the entire cluster—VPC, control plane, node groups, and add-ons—as code for repeatable, version-controlled infrastructure.

## Official references

- [Amazon EKS add-ons](https://docs.aws.amazon.com/eks/latest/userguide/eks-add-ons.html)
- [EKS managed node groups](https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html)
- [eksctl add-on management](https://docs.aws.amazon.com/eks/latest/eksctl/addons.html)
