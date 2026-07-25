# Essential kubectl & Cluster Commands

Quick reference of high-value commands to practice for CKA and day-to-day ops.

- Cluster & nodes:
  - `kubectl cluster-info`
  - `kubectl get nodes -o wide`
  - `kubectl describe node <node>`

- Pods & workloads:
  - `kubectl get pods --all-namespaces`
  - `kubectl describe pod <pod> -n <ns>`
  - `kubectl logs <pod> -c <container>`
  - `kubectl exec -it <pod> -- /bin/sh`
  - `kubectl port-forward svc/<svc> 8080:80`

- Deployments & rollouts:
  - `kubectl get deployments`
  - `kubectl rollout status deployment/<name>`
  - `kubectl rollout undo deployment/<name>`
  - `kubectl scale deployment/<name> --replicas=3`

- Services & networking:
  - `kubectl get svc`
  - `kubectl describe svc <name>`
  - `kubectl get endpointslices -n <ns>`

- Storage:
  - `kubectl get pv,pvc,sc`
  - `kubectl describe pvc <name>`

- Node maintenance:
  - `kubectl cordon <node>`
  - `kubectl drain <node> --ignore-daemonsets --delete-emptydir-data`
  - `kubectl uncordon <node>`

- Debug & troubleshooting:
  - `kubectl top node` / `kubectl top pod`
  - `kubectl api-resources`
  - `kubectl api-versions`
  - `kubectl auth can-i create pods --as system:serviceaccount:default:sa`

- Editing & patching:
  - `kubectl edit deployment/<name>`
  - `kubectl patch deployment/<name> -p '{"spec":{"replicas":2}}'`

- Config & secrets:
  - `kubectl get configmap,secret -n <ns>`
  - `kubectl create secret generic regcred --from-file=.dockerconfigjson=/path --type=kubernetes.io/dockerconfigjson`

- Helpful aliases (bash):
  - `alias k='kubectl'`
  - `alias kgp='kubectl get pods'`

Practice these until they feel muscle-memory for CKA tasks.
