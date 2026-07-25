# CKA Basics — Answers and Practice

## 1. View every Pod

```sh
kubectl get pods -A -o wide
kubectl get pods -A --sort-by=.metadata.creationTimestamp
```

`-A` selects all namespaces. The important columns are `READY` (ready containers/total), `STATUS`, `RESTARTS`, `AGE`, and—with `-o wide`—the Pod IP and assigned node. Use `kubectl describe pod <name> -n <namespace>` when the status is not self-explanatory.

## 2. Check API access and discover resources

```sh
kubectl cluster-info
kubectl get --raw='/readyz?verbose'
kubectl api-resources
kubectl api-versions
```

`cluster-info` verifies client connectivity. `/readyz` is a control-plane health endpoint when your credentials permit it. `api-resources` is the fastest way to discover the correct resource name, API group, and whether a resource is namespaced.

## 3. Create a namespace and run a Pod

```sh
kubectl create namespace interview
kubectl run web -n interview --image=nginx:stable --port=80
kubectl get pod -n interview -w
```

Use a manifest when you need declarative, repeatable configuration:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web
  namespace: interview
  labels:
    app: web
spec:
  containers:
  - name: nginx
    image: nginx:stable
    ports:
    - containerPort: 80
```

Apply it with `kubectl apply -f pod.yaml`, validate with `kubectl get pod web -n interview`, and clean up with `kubectl delete namespace interview`.

## CKA habit

Set the namespace in your context before working: `kubectl config set-context --current --namespace=interview`. Always verify the active context first with `kubectl config current-context`.
