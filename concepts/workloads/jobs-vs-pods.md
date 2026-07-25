# Jobs vs Pods

This file explains the difference between Kubernetes Jobs and bare Pods.

## Pod
- The basic execution unit in Kubernetes.
- Represents one or more containers running together.
- Pods are ephemeral.
- If a pod is deleted, it is gone unless recreated by a controller.

## Job
- A Job creates one or more pods to complete a task.
- It ensures that a specified number of pods successfully terminate.
- Useful for batch processing or one-time tasks.

## Job types
- **Classic Job**: runs pods to completion and stops.
- **Parallel Job**: can run multiple pods in parallel.
- **Indexed Job**: pods get stable indices.

## Example
- Use a bare Pod for a long-running service container.
- Use a Job for a database migration script.

## Important points
- Pods are the runtime object.
- Jobs are controllers that manage pod completion.
- A Job can restart pods until the task succeeds.
- A Pod alone has no completion guarantee beyond its own lifecycle.
