# 200 CKA Practice Questions

These are original hands-on practice scenarios based on common CKA and Kubernetes administrator interview patterns. They are **not** leaked, recalled, or official past-exam questions. Attempt each in a disposable cluster, record the commands you used, and validate the requested outcome before viewing any notes.

## Cluster architecture, installation, and administration (1–40)

1. Identify every control-plane component on a kubeadm control-plane node and state its responsibility.

    *   **Answer:** On a standard `kubeadm` cluster, the control plane components run as static Pods in the `kube-system` namespace.
        *   **`kube-apiserver`**: The front-end for the control plane. It exposes the Kubernetes API, processes requests, and validates them before persisting state in etcd.
        *   **`etcd`**: A consistent and highly-available key-value store used as Kubernetes' backing store for all cluster data.
        *   **`kube-scheduler`**: Watches for newly created Pods with no assigned node and selects a node for them to run on based on resource requirements, policies, and affinity rules.
        *   **`kube-controller-manager`**: Runs all the built-in Kubernetes controllers (e.g., Deployment controller, Node controller). These controllers watch the cluster state and work to move the current state towards the desired state.
    *   **Verification:**
        ```sh
        # List the static pods running on the control plane node
        kubectl get pods -n kube-system
        ```

2. Show how the API server discovers and persists a newly created Deployment.

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
        kubectl logs kube-apiserver-controlplane -n kube-system
        ```

3. Find the etcd data directory and client certificate paths from a static Pod manifest.

    *   **Answer:** The `etcd` static Pod manifest is located at `/etc/kubernetes/manifests/etcd.yaml` on a `kubeadm` control-plane node. You can inspect this file to find the volume mounts and command arguments that specify these paths.
    *   **Verification:**
        ```sh
        # SSH into the control-plane node and inspect the manifest
        cat /etc/kubernetes/manifests/etcd.yaml
        ```

4. Take an etcd snapshot, verify its status, and explain where it must be stored securely.

    *   **Answer:** Use the `etcdctl` utility with the correct API version and credentials to create a snapshot. The snapshot is a single file containing a full backup of the etcd database. This file must be stored securely **off-cluster** (e.g., in a secure object storage bucket or an encrypted backup server), as it contains all cluster state, including Secrets.
    *   **Verification:**
        ```sh
        # Define credentials for etcdctl (paths from the etcd.yaml manifest)
        export ETCDCTL_API=3
        export ETCDCTL_ENDPOINTS=https://127.0.0.1:2379
        export ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
        export ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt
        export ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key

        # Take the snapshot and verify its integrity
        sudo etcdctl snapshot save /tmp/etcd-snapshot.db
        sudo etcdctl snapshot status /tmp/etcd-snapshot.db -w table
        ```

5. Restore an etcd snapshot to a new data directory in a single-control-plane lab.

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

6. Diagnose an API server that is not becoming Ready after an etcd restore.
7. Explain why only the API server should directly access etcd.
8. Compare stacked and external-etcd high-availability topologies.
9. Add a worker node using the supplied kubeadm join command.
10. Generate a new worker join command when the old token has expired.
11. Upgrade a kubeadm control-plane node one supported minor version.
12. Upgrade a worker node after the control plane.
13. Safely drain, upgrade, and uncordon a worker node.
14. Explain when `--ignore-daemonsets` and `--delete-emptydir-data` are appropriate during drain.
15. Cordon a node, prove new regular Pods do not schedule there, then uncordon it.
16. Create a static Pod and locate its mirror Pod in the API.
17. Diagnose why a static Pod manifest is not producing a running Pod.
18. Find the kubelet configuration and inspect its active flags.
19. Check whether a node is using containerd and list its running containers with `crictl`.
20. Explain the relationship between kubelet, CRI, and the container runtime.
21. Verify a node can reach the API server and identify the control-plane endpoint it uses.
22. Rotate an expired kubelet client certificate in a kubeadm lab.
23. Identify which certificates are nearing expiry and plan a safe renewal.
24. Explain the role of the cloud-controller-manager in a cloud cluster.
25. Identify the CNI configuration directory and explain its effect on Pod networking.
26. Create a local kind or kubeadm lab with one control plane and two workers.
27. Use `kubectl api-resources` to find the correct API group for a requested resource.
28. Show the Kubernetes server and client versions and explain version-skew concerns.
29. Identify the current context and switch to a named context safely.
30. Set the default namespace for the current context and undo the change.
31. Explain how a Lease is used for node heartbeats and leader election.
32. Find the current leader-election Lease for a control-plane component.
33. Compare cgroup v1 and cgroup v2 implications for a Kubernetes node.
34. Inspect node capacity, allocatable resources, and system-reserved pressure.
35. Create a taint that reserves a node for a platform workload.
36. Configure a node label and schedule a matching Pod.
37. Explain the control-plane failure domains that matter for multi-zone clusters.
38. Plan an etcd backup and restore drill with a stated RPO and RTO.
39. List the cluster add-ons required for DNS, metrics, and network policy in a minimal cluster.
40. Explain how to verify the health of the control plane without relying only on `kubectl get nodes`.

## Workloads, Pods, and lifecycle (41–80)

41. Create a Pod from a manifest and inspect all of its status conditions.
42. Create a Deployment with three replicas and verify its ReplicaSet ownership.
43. Perform a rolling update and watch the old and new ReplicaSets.
44. Pause a Deployment rollout, make a second change, then resume it.
45. Roll back a failed Deployment to the previous revision.
46. Configure `maxSurge` and `maxUnavailable` for a zero-downtime rollout.
47. Explain why a Deployment is preferred over individually managed Pods for a stateless API.
48. Create a StatefulSet with a headless Service and one PVC per replica.
49. Scale a StatefulSet down and explain what happens to its PVCs.
50. Create a DaemonSet that runs a log agent on all Linux worker nodes.
51. Restrict a DaemonSet to a selected node pool.
52. Create a Job that runs to completion and inspect its completion status.
53. Configure a Job with retries, backoff, and a deadline.
54. Create a CronJob that runs every five minutes without overlapping executions.
55. Suspend a CronJob, inspect its history, and resume it.
56. Configure automatic cleanup for finished Jobs using TTL.
57. Explain the difference between a Pod restart and controller-created replacement Pod.
58. Create an init container that blocks startup until a dependency is reachable.
59. Add a sidecar container and explain shared network and volume behavior inside a Pod.
60. Add liveness, readiness, and startup probes to a slow-starting service.
61. Demonstrate why a failing readiness probe removes an endpoint without restarting the container.
62. Diagnose a liveness probe that causes `CrashLoopBackOff`.
63. Use an ephemeral container to debug a distroless application container.
64. Explain the limitations of ephemeral containers for resource and port changes.
65. Configure graceful termination with `terminationGracePeriodSeconds` and a `preStop` hook.
66. Explain Pod phases versus container states and readiness conditions.
67. Create a Pod with a custom hostname and subdomain and inspect DNS results.
68. Identify the QoS class of Pods with Guaranteed, Burstable, and BestEffort resources.
69. Use the Downward API to inject a Pod name and label into a container.
70. Use a ConfigMap as an environment source and as a mounted volume.
71. Explain when a ConfigMap volume update is visible and when an environment variable update is not.
72. Create a Secret and mount it as a read-only file.
73. Configure a Pod with resource requests, limits, and an `emptyDir` size limit.
74. Diagnose a Pod evicted for ephemeral-storage pressure.
75. Create a PodDisruptionBudget and explain voluntary versus involuntary disruption.
76. Drain a node containing a PDB-protected Deployment and interpret the result.
77. Use `kubectl debug node/<node>` to investigate a node, if the cluster permits it.
78. Explain why a bare Pod is unsuitable for long-running production work.
79. Configure a multi-container Pod that shares an `emptyDir` volume.
80. Given a Pod stuck in `ContainerCreating`, isolate whether the cause is image, volume, CNI, or runtime related.

## Scheduling, resources, and autoscaling (81–115)

81. Explain the scheduler filter and score phases with one example constraint for each.
82. Diagnose a Pending Pod with `Insufficient cpu` events.
83. Schedule a Pod only on nodes labeled `disk=ssd`.
84. Use required node affinity to choose a zone and preferred affinity to favor a node type.
85. Create a Pod anti-affinity rule that spreads replicas by hostname.
86. Taint a node with `NoSchedule` and add the minimum needed toleration.
87. Compare `NoSchedule`, `PreferNoSchedule`, and `NoExecute` taint effects.
88. Evict existing Pods with a `NoExecute` taint and a bounded `tolerationSeconds`.
89. Use `topologySpreadConstraints` to spread an application across zones.
90. Explain `DoNotSchedule` versus `ScheduleAnyway` for topology spread.
91. Define a PriorityClass for a platform-critical workload.
92. Explain why preemption may still fail to schedule a high-priority Pod.
93. Configure a Pod with an explicit `preemptionPolicy: Never` and explain the trade-off.
94. Use a ResourceQuota to cap namespace CPU, memory, and object count.
95. Use a LimitRange to set default container requests and limits.
96. Diagnose a rejected Pod caused by ResourceQuota or LimitRange.
97. Explain how resource requests influence scheduling and HPA CPU utilization.
98. Create an HPA targeting a Deployment using `autoscaling/v2`.
99. Diagnose an HPA with unknown CPU metrics.
100. Configure HPA scale-up and scale-down behavior to reduce flapping.
101. Explain what happens when several HPA metrics recommend different replica counts.
102. Explain when VPA recommendation-only mode is safer than automatic updates.
103. Combine HPA and node autoscaling and describe the event sequence under load.
104. Diagnose a Pod that remains Pending even though node CPU usage looks low.
105. Explain Pod overhead and when it affects scheduling.
106. Reserve CPU and memory for kubelet and system daemons on a node.
107. Identify node memory, disk, PID, and network pressure conditions.
108. Tune a workload request after observing OOMKills and throttling.
109. Explain the difference between descheduling and scheduling.
110. Use the descheduler safely to rebalance an underutilized cluster.
111. Explain gang scheduling and why the default scheduler does not provide it as a core primitive.
112. Diagnose volume topology preventing a Pod from scheduling.
113. Explain how `WaitForFirstConsumer` participates in scheduling decisions.
114. Use `kubectl top` and explain what it does not tell you about requests.
115. Build a checklist for a Pod that is unschedulable due to multiple constraints.
115a. Configure a Pod with `podSchedulingGate` and explain how an external controller would use it to manage scheduling readiness.
115b. Explain the concept of "gang scheduling" and how it can be implemented using a custom scheduler or framework plugins.
115c. Describe a scenario where resource bin packing is desirable and how to encourage it via scheduler configuration.
115d. Diagnose a node under `MemoryPressure` and explain how node-pressure eviction works.

## Services, networking, DNS, and ingress (116–145)

116. Create a ClusterIP Service for a Deployment and verify EndpointSlices.
117. Explain the difference between a Service selector and manually managed EndpointSlices.
118. Expose a Service with NodePort and test it from a node.
119. Explain the responsibilities of a cloud LoadBalancer Service versus an Ingress controller.
120. Diagnose a Service with no endpoints.
121. Diagnose a Service with endpoints but no reachable traffic.
122. Verify DNS resolution for `kubernetes.default.svc.cluster.local` from a Pod.
123. Debug a CoreDNS outage using Pods, Service, EndpointSlices, and logs.
124. Explain how `ndots` and search domains affect Pod DNS lookups.
125. Create a headless Service and explain its DNS behavior for a StatefulSet.
126. Explain how kube-proxy routes Service traffic in iptables or IPVS mode.
127. Explain the role of a CNI plugin versus kube-proxy.
128. Create a NetworkPolicy that allows ingress only from one namespace.
129. Add an egress rule that permits DNS and one external CIDR.
130. Explain why a NetworkPolicy may have no effect in a particular cluster.
131. Create an Ingress with host, path, Service backend, and TLS secret.
132. Diagnose an Ingress returning 404, 502, or 503 in a controller-backed cluster.
133. Explain the difference between the Ingress object and Ingress controller.
134. Compare Ingress and Gateway API at a high level.
135. Inspect an `IngressClass` and select it explicitly in an Ingress manifest.
136. Explain internal versus external traffic policy for a Service.
137. Explain topology-aware routing and when EndpointSlice hints are useful.
138. Describe dual-stack Service and Pod addressing requirements.
139. Diagnose a Pod-to-Pod connectivity failure across nodes.
140. Trace traffic from an external client through load balancer, Ingress, Service, EndpointSlice, and Pod.
141. Explain why readiness affects Service traffic while liveness does not directly.
142. Create a ServiceAccount-based debug Pod and test in-cluster API DNS.
143. Inspect `resolv.conf` in a Pod and identify a wrong DNS configuration.
144. Explain how a `hostNetwork` Pod changes DNS and network isolation considerations.
145. Compare `externalTrafficPolicy: Cluster` and `Local` for source IP preservation.

## Storage, configuration, and security (146–170)

146. Create a PVC using an existing StorageClass and mount it in a Pod.
147. Diagnose a PVC stuck in Pending.
148. Explain PV, PVC, StorageClass, CSI driver, and volume provisioning roles.
149. Compare `ReadWriteOnce`, `ReadOnlyMany`, and `ReadWriteMany` with provider limitations.
150. Configure a StorageClass reclaim policy and explain `Delete` versus `Retain`.
151. Restore a PVC from a VolumeSnapshot in a CSI-capable cluster.
152. Explain application-consistent versus crash-consistent snapshots.
153. Use an `emptyDir`, `hostPath`, and PVC; state the risk and suitable use case for each.
154. Diagnose a Pod that cannot mount a bound PVC.
155. Explain local ephemeral storage accounting and eviction risk.
156. Create a ConfigMap from literals and a file, then consume it in a Pod.
157. Create a generic Secret and explain why base64 encoding is not encryption.
158. Enable or verify encryption at rest for Secrets conceptually.
159. Create a ServiceAccount and inspect the projected token volume behavior.
160. Create a Role and RoleBinding that allow read-only Pod access in one namespace.
161. Explain the difference between RoleBinding and ClusterRoleBinding.
162. Use `kubectl auth can-i` to debug an authorization failure.
163. Explain authentication, authorization, admission, and validation in API request order.
164. Create a Namespace with Pod Security Admission labels for the baseline profile.
165. Identify a Pod manifest that violates restricted Pod Security Standards.
166. Explain why PodSecurityPolicy is not the current mechanism for admission control.
167. Configure a `securityContext` with non-root execution, dropped capabilities, and read-only root filesystem.
168. Explain how NetworkPolicy, RBAC, and Pod Security Standards address different security layers.
169. Diagnose an image pull failure caused by a private registry credential.
170. Build a namespace onboarding checklist covering quota, limits, RBAC, PSA, and network policy.
170a. Explain the risks of allowing Pods to bypass the API server to access etcd and how to prevent it.
170b. Create a `ValidatingAdmissionPolicy` that rejects any Pod that uses a `hostPath` volume.
170c. Describe the purpose of `CertificateSigningRequest` and demonstrate how to manually approve one.
170d. Explain the difference between `ProcessID` limits at the node level and Pod level.

## Troubleshooting and production scenarios (171–200)

171. Triage a `CrashLoopBackOff` using describe, previous logs, events, and resource status.
172. Triage an `ImagePullBackOff` caused by a bad tag, registry access, or imagePullSecret.
173. Triage a `CreateContainerConfigError` caused by a missing ConfigMap or Secret.
174. Triage a `CreateContainerError` caused by an invalid command or volume mount.
175. Triage a Pod stuck in Pending because of an unbound immediate PVC.
176. Triage a Pod stuck in Terminating because of finalizers or an unreachable node.
177. Recover a node that is NotReady because kubelet is stopped.
178. Recover a node reporting `DiskPressure` without deleting unknown data.
179. Diagnose a node whose container runtime socket is unavailable.
180. Diagnose a CNI failure that prevents sandbox creation.
181. Diagnose a CoreDNS failure that affects only some namespaces.
182. Diagnose a Service that selects the wrong Pods after a label change.
183. Diagnose an Ingress that reaches the controller but not the backend Service.
184. Diagnose a NetworkPolicy that blocks DNS unexpectedly.
185. Diagnose a rollout that stalls because new Pods never become Ready.
186. Diagnose an HPA that refuses to scale because the target lacks resource requests.
187. Diagnose a Deployment unable to progress because of quota exhaustion.
188. Diagnose a StatefulSet blocked by ordered startup and a broken first replica.
189. Diagnose a Job that keeps retrying and decide whether to change code or backoff settings.
190. Diagnose a CronJob that misses schedules or creates too many concurrent Jobs.
191. Diagnose a DaemonSet with fewer Pods than eligible nodes.
192. Diagnose RBAC denial for a ServiceAccount used by a controller.
193. Diagnose a failing admission webhook and explain its cluster-wide risk.
194. Diagnose certificate expiry causing kubelet or API-server communication failures.
195. Explain the minimum evidence you would collect before restoring etcd.
196. Build a command sequence for a suspected control-plane outage.
197. Build a command sequence for an application outage limited to one namespace.
198. Build a command sequence for a cross-node network outage.
199. Explain how you would communicate risk before draining a node hosting stateful workloads.
200. Given an unfamiliar failure, use Kubernetes events, object status, logs, metrics, and component ownership to produce a root-cause hypothesis and a safe rollback plan.

## How to score yourself

For each scenario, award one point only if you can complete the task and explain why the result is correct. Flag any answer that relies on memorized commands without verification; the CKA and real operations reward observable evidence.
