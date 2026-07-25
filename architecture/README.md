# Kubernetes Architecture

## Overview
Kubernetes is an open-source container orchestration platform designed to automate deployment, scaling, and management of containerized applications. Its architecture is based on a cluster model with two main node types:

- **Control Plane Nodes**
- **Worker Nodes**

## Control Plane
The control plane manages the cluster and makes global decisions.

### Primary control plane components
- **API Server** (`kube-apiserver`): Serves the Kubernetes API and acts as the front-end for the control plane.
- **etcd**: A distributed key-value store used to persist the cluster state.
- **Controller Manager** (`kube-controller-manager`): Runs controller processes that manage cluster state, including node lifecycle, deployment rollout, replication, and endpoints.
- **Scheduler** (`kube-scheduler`): Assigns pods to worker nodes based on resource requirements and scheduling policies.

### Control plane responsibilities
- Cluster state management
- Scheduling workloads
- Maintaining desired state
- Authentication and authorization
- Exposing Kubernetes API to users and automation tools

## Worker Node
Worker nodes host and run application containers.

### Primary worker node components
- **kubelet**: An agent that runs on each worker node and communicates with the control plane to manage pod lifecycle.
- **Container runtime**: Executes containers, such as containerd or CRI-O.
- **kube-proxy**: Handles networking for Kubernetes services, implementing load balancing and network rules.

### Worker node responsibilities
- Running pods and containers
- Reporting node and pod status to the control plane
- Managing container networking and service proxying
- Mounting and managing storage volumes for workloads

## Architecture diagram

![Kubernetes architecture](architecture-diagram.svg)

## Architecture diagram references
For architecture diagrams and deep-dive visuals, refer to:

- Official Kubernetes architecture docs: https://kubernetes.io/docs/concepts/overview/components/
- Kubernetes control plane doc: https://kubernetes.io/docs/concepts/overview/components/#control-plane-components
- Kubernetes node doc: https://kubernetes.io/docs/concepts/architecture/nodes/

## How to use this folder
- Use this README as a single point of reference for Kubernetes architecture.
- Keep the diagram and component roles in mind when explaining control plane vs worker node responsibilities.
- Add notes for multi-zone control plane, HA, networking, and storage as interview preparation grows.
