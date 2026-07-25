# Amazon EKS

## How it works

Amazon EKS manages the Kubernetes control plane. You choose and operate the data plane: EKS managed node groups (EC2), self-managed nodes, Fargate for compatible Pods, or an EKS-managed compute option where applicable. Worker nodes join the cluster over the VPC and run the normal Kubernetes kubelet, runtime, CNI, and kube-proxy components.

## Important features

- **Managed control plane:** AWS operates API servers and etcd; you still own workload availability, Kubernetes objects, node capacity, and add-on compatibility.
- **Compute:** managed node groups provide an AWS-managed node lifecycle; use separate groups for system, application, GPU, Spot, or zone-specific workloads.
- **Networking:** the Amazon VPC CNI gives Pods VPC-routable addresses. Plan VPC/subnet IP capacity before high Pod density.
- **Identity:** prefer EKS Pod Identity or IRSA for Pod-to-AWS permissions; do not distribute node IAM credentials to applications.
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
  configurationValues: |-
    replicaCount: 2
- name: kube-proxy
- name: aws-ebs-csi-driver
```

```sh
eksctl create cluster -f cluster.yaml
eksctl get addon --cluster <cluster-name> --region <aws-region>
```

## Upgrade order

1. Review the EKS target-version compatibility and add-on versions.
2. Upgrade the EKS control plane through AWS.
3. Update compatible managed add-ons and controllers.
4. Roll managed node groups or replacement nodes; drain safely and respect PDBs.
5. Validate DNS, storage, Ingress/load balancers, autoscaling, and applications.

## Interview distinction

EKS manages the control plane, but it does not remove responsibility for VPC design, IAM boundaries, node/workload lifecycle, Kubernetes add-ons, or application reliability.

## Official references

- [Amazon EKS add-ons](https://docs.aws.amazon.com/eks/latest/userguide/eks-add-ons.html)
- [EKS managed node groups](https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html)
- [eksctl add-on management](https://docs.aws.amazon.com/eks/latest/eksctl/addons.html)
