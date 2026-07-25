# How `kube-proxy` routes traffic to Pods

This file explains how `kube-proxy` implements Kubernetes Service routing on each node.

## Overview
- `kube-proxy` watches `Service` and `Endpoints` objects from the API server.
- It programs node-local network rules so traffic to a Service IP forwards to backend Pod IPs.
- The Service IP is virtual; the actual pods are real endpoints.

## Modes of operation

### `iptables` mode
- `kube-proxy` creates `iptables` rules in the kernel.
- Traffic to `ServiceIP:port` is DNATed to a backend pod IP:port.
- Load balancing is done by chaining rules across all endpoints.

### `ipvs` mode
- `kube-proxy` configures IPVS virtual servers.
- IPVS provides in-kernel load balancing.
- Supports algorithms such as round-robin, least connections, and source hashing.

## Traffic flow
1. A client sends traffic to the Service cluster IP and port.
2. The node kernel matches a rule or IPVS virtual server.
3. The destination address is rewritten to a selected Pod IP.
4. The packet is forwarded to the pod.

## What `kube-proxy` supports
- `ClusterIP`
- `NodePort`
- `LoadBalancer`
- `ExternalIPs`

## Important details
- `kube-proxy` is a node-level data plane component.
- It does not run inside the pod or inspect application payloads.
- It simply programs network forwarding so Services are reachable.
- For multi-node clusters, `kube-proxy` on each node maintains local service rules.

## Interview points
- `kube-proxy` is not a proxy process in the traditional sense when in `iptables` or `ipvs` mode.
- It is a network rule manager that builds forwarding paths.
- Service IPs are virtual, backends are Pod IPs.
- The actual routing decision happens inside the kernel.
