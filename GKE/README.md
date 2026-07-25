# Google Kubernetes Engine (GKE)

## How it works

GKE manages the Kubernetes control plane. Choose **Autopilot** when Google should manage node configuration, scaling, and many security defaults, or **Standard** when you need more direct control of node pools and infrastructure settings. Regional clusters spread control-plane replicas across a region; zonal clusters have one zonal control plane.

## Important features

- **Autopilot:** workload-driven infrastructure management with enforced operational and security defaults; suitable for most production workloads.
- **Standard:** configurable node pools, machine types, node images, networking, and operational controls.
- **Networking:** VPC-native networking, optional Dataplane V2, private clusters, and policy enforcement choices.
- **Identity:** Workload Identity Federation for GKE lets Pods access Google Cloud APIs without node-wide credentials.
- **Storage:** CSI-backed Persistent Disks and Filestore are common choices; select the storage class based on latency, topology, and access-mode needs.
- **Lifecycle:** release channels and maintenance windows control how GKE receives upgrades.

## Essential commands

```sh
# Create examples: choose one operating mode deliberately
gcloud container clusters create-auto <cluster> --location <region>
gcloud container clusters create <cluster> --location <region>

# Configure kubectl and inspect the cluster
gcloud container clusters get-credentials <cluster> --location <region>
kubectl get nodes -o wide
gcloud container clusters describe <cluster> --location <region>

# Standard-only node-pool operations
gcloud container node-pools list --cluster <cluster> --location <region>
gcloud container node-pools create <pool> --cluster <cluster> --location <region>
```

## Features and add-ons

GKE offers cluster capabilities such as Cloud DNS, network policy/Dataplane V2, GCS FUSE CSI, backup, managed metrics, logging, and policy/security integrations. Availability and configuration vary by GKE mode, version, region, and edition. Use `gcloud container clusters describe` to inspect enabled features, and configure features through the supported GKE workflow instead of treating every capability as a generic Kubernetes add-on.

## Operational guidance

- Choose Autopilot for reduced node operations; choose Standard only when the additional control is necessary.
- Plan private/public API access, VPC/subnet ranges, release channel, and maintenance window before creation; several choices are difficult or disruptive to change later.
- Validate workload resource requests: they influence scheduling, autoscaling, and cost, especially in Autopilot.

## Interview distinction

The central GKE design choice is **Autopilot versus Standard**: Autopilot shifts more infrastructure operation to Google; Standard leaves more node and configuration responsibility with the platform team.

## Official references

- [GKE configuration choices](https://cloud.google.com/kubernetes-engine/docs/concepts/configuration-overview)
- [GKE Autopilot overview](https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-overview)
- [Get GKE credentials](https://cloud.google.com/sdk/gcloud/reference/container/clusters/get-credentials)
