# Descheduler in Kubernetes

This file explains the Kubernetes descheduler and when to use it.

## What is the descheduler?
- A component that evicts pods to improve cluster balance and node utilization.
- It is typically run as a Kubernetes job or cronjob.
- It is not part of the core scheduler.

## Use cases
- rebalance pods after node labels change
- evict pods from overloaded nodes
- improve topology spread after scaling events
- remove pods from nodes with taints or insufficient resources

## How it works
- The descheduler evaluates current pod placement.
- It identifies pods that should be evicted for better balance.
- Evicted pods are recreated by their owning controller.

## Important points
- Descheduler is useful in clusters with dynamic topology.
- It can help maintain even distribution of pods.
- It should be used carefully to avoid unnecessary evictions.
