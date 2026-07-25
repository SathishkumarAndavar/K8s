# CKA Practice Questions — Answers

This file provides detailed answers and command sequences for the scenarios in `200-practice-questions.md`.

---

## Cluster architecture, installation, and administration (1–40)

**1. Identify every control-plane component on a kubeadm control-plane node and state its responsibility.**

*   **Answer:** On a standard `kubeadm` cluster, the control plane components run as static Pods in the `kube-system` namespace.
    *   **`kube-apiserver`**: The front-end for the control plane. It exposes the Kubernetes API, processes requests, and validates them before persisting state in etcd.
    *   **`etcd`**: A consistent and highly-available key-value store used as Kubernetes' backing store for all cluster data.
    *   **`kube-scheduler`**: Watches for newly created Pods with no assigned node and selects a node for them to run on based on resource requirements, policies, and affinity rules.
    *   **`kube-controller-manager`**: Runs all the built-in Kubernetes controllers (e.g., Deployment controller, Node controller). These controllers watch the cluster state and work to move the current state towards the desired state.
*   **Verification:**
    ```sh
    # List the static pods running on the control plane node
    kubectl get pods -n kube-system

    # Example Output (names will include the node name):
    # NAME                                   READY   STATUS    RESTARTS   AGE
    # coredns-787d4945-r22v8                 1/1     Running   0          10m
    # etcd-controlplane                      1/1     Running   0          10m
    # kube-apiserver-controlplane            1/1     Running   0          10m
    # kube-controller-manager-controlplane   1/1     Running   0          10m
    # kube-proxy-abcde                       1/1     Running   0          10m
    # kube-scheduler-controlplane            1/1     Running   0          10m
    ```

---

**2. Show how the API server discovers and persists a newly created Deployment.**

*   **Answer:** This is a multi-step process involving client interaction, authentication, authorization, admission control, and finally persistence.
    1.  **Client Request**: `kubectl` sends the `Deployment` manifest as an HTTP POST request to the `kube-apiserver`.
    2.  **Authentication**: The API server authenticates the user (e.g., via client certificate or token).
    3.  **Authorization**: It checks if the authenticated user is authorized to create Deployments in the specified namespace (e.g., via RBAC).
    4.  **Admission Control**: A series of admission controllers (mutating and validating) inspect the request. They can modify the object (e.g., inject a sidecar) or reject it if it violates policy (e.g., exceeds a `ResourceQuota`).
    5.  **Persistence**: If the request passes all checks, the API server writes the `Deployment` object into its backing data store, `etcd`.
    6.  **Response**: The API server sends a success response back to the client.
*   **Verification:** You can trace this by increasing the API server's log level.
    ```sh
    # Use -v=9 on kubectl to see the HTTP request being made
    kubectl apply -f deployment.yaml -v=9

    # On the control plane node, you can inspect the API server logs
    # (path may vary depending on the container runtime)
    kubectl logs kube-apiserver-controlplane -n kube-system
    ```

---

**3. Find the etcd data directory and client certificate paths from a static Pod manifest.**

*   **Answer:** The `etcd` static Pod manifest is located at `/etc/kubernetes/manifests/etcd.yaml` on a `kubeadm` control-plane node. You can inspect this file to find the volume mounts and command arguments that specify these paths.
*   **Verification:**
    ```sh
    # SSH into the control-plane node and inspect the manifest
    cat /etc/kubernetes/manifests/etcd.yaml

    # Look for the '--data-dir' flag in the 'command' section.
    # Example:
    # - --data-dir=/var/lib/etcd

    # Look for volumeMounts and volumes to find the certificate paths.
    # Example:
    # volumeMounts:
    # - mountPath: /etc/kubernetes/pki/etcd
    #   name: etcd-certs
    # volumes:
    # - hostPath:
    #     path: /etc/kubernetes/pki/etcd
    #     type: DirectoryOrCreate
    #   name: etcd-certs
    ```
    The client certificates used by components like the API server to talk to etcd are typically found in `/etc/kubernetes/pki/`.

---

**4. Take an etcd snapshot, verify its status, and explain where it must be stored securely.**

*   **Answer:** Use the `etcdctl` utility with the correct API version and credentials to create a snapshot. The snapshot is a single file containing a full backup of the etcd database. This file must be stored securely **off-cluster** (e.g., in a secure object storage bucket or an encrypted backup server), as it contains all cluster state, including Secrets.
*   **Verification:**
    ```sh
    # Define credentials for etcdctl (paths from the etcd.yaml manifest)
    export ETCDCTL_API=3
    export ETCDCTL_ENDPOINTS=https://127.0.0.1:2379
    export ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
    export ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt
    export ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key

    # Take the snapshot
    sudo etcdctl snapshot save /tmp/etcd-snapshot.db

    # Verify the snapshot's integrity
    sudo etcdctl snapshot status /tmp/etcd-snapshot.db -w table

    # Example Output:
    # +----------+----------+------------+------------+
    # |   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
    # +----------+----------+------------+------------+
    # | a1b2c3d4 |   12345  |       1500 |     5.2 MB |
    # +----------+----------+------------+------------+
    ```

---

**5. Restore an etcd snapshot to a new data directory in a single-control-plane lab.**

*   **Answer:** Restoring a snapshot is a destructive operation that replaces the cluster state. The general process is to stop the control plane, restore the data to a *new* directory, and reconfigure the `etcd` static Pod to use that new directory.
*   **Verification:**
    1.  Stop the `kube-apiserver` and `etcd` static Pods by moving their manifests out of `/etc/kubernetes/manifests/`.
    2.  Run the restore command, pointing to a new data directory.
    3.  Update `/etc/kubernetes/manifests/etcd.yaml` to point its `hostPath` volume to the new restored directory.
    4.  Move the `etcd.yaml` and `kube-apiserver.yaml` manifests back into `/etc/kubernetes/manifests/`.
    5.  The kubelet will restart the control plane components. Verify cluster health with `kubectl get nodes` and `kubectl get pods -A`.
    ```sh
    # (On the control plane node)
    # Step 1: Stop the control plane
    sudo mv /etc/kubernetes/manifests/etcd.yaml /tmp/
    sudo mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/
    # Wait for pods to stop

    # Step 2: Restore to a new directory
    sudo etcdctl snapshot restore /tmp/etcd-snapshot.db --data-dir /var/lib/etcd-new

    # Step 3: Update etcd.yaml to use /var/lib/etcd-new
    sudo sed -i 's|/var/lib/etcd|/var/lib/etcd-new|g' /tmp/etcd.yaml

    # Step 4: Restart the control plane
    sudo mv /tmp/etcd.yaml /etc/kubernetes/manifests/
    sudo mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/

    # Step 5: Verify from a machine with kubectl
    kubectl get nodes
    ```

---

*Answers for questions 6-10 would follow a similar pattern.*