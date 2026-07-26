# CNI vs. kube-proxy

CNI (Container Network Interface) and `kube-proxy` are two fundamental components of Kubernetes networking, but they have completely different responsibilities. Understanding their distinct roles is key to troubleshooting network issues.

---

## CNI (Container Network Interface)

-   **Purpose**: **Pod-to-Pod Networking**. The CNI plugin is responsible for the "east-west" traffic within the cluster. Its job is to ensure that every pod gets a unique IP address and can communicate with every other pod.
-   **How it Works**: When a pod is scheduled to a node, the `kubelet` calls the CNI plugin installed on that node. The plugin then:
    1.  Creates a network namespace for the new pod.
    2.  Assigns an IP address to the pod (IP Address Management - IPAM).
    3.  Connects the pod's network namespace to the node's network, often by creating a virtual Ethernet (`veth`) pair and connecting it to a Linux bridge.
    4.  Sets up the necessary routes or overlay network configuration to enable communication with pods on other nodes.
-   **Examples**: Calico, Flannel, Cilium, Weave Net, AWS VPC CNI.

```mermaid
graph TD
    subgraph "Node 1"
        Kubelet1[Kubelet] --> CNI1[CNI Plugin]
        CNI1 -- "Assigns IP 10.1.1.2" --> PodA[Pod A]
    end
    subgraph "Node 2"
        Kubelet2[Kubelet] --> CNI2[CNI Plugin]
        CNI2 -- "Assigns IP 10.1.2.2" --> PodB[Pod B]
    end
    PodA -- "Packet to 10.1.2.2" --> Overlay[Overlay Network (managed by CNI)]
    Overlay --> PodB
```

---

## kube-proxy

-   **Purpose**: **Implementing Services**. `kube-proxy` is responsible for making the `Service` abstraction work. It translates a request to a virtual `Service` IP into a request to one of the healthy backend pods.
-   **How it Works**: `kube-proxy` runs as a DaemonSet on every node. It watches the API server for changes to `Service` and `EndpointSlice` objects. When a `Service` is created or its backend pods change, `kube-proxy` updates the networking rules on the node (e.g., using `iptables` or `IPVS`) to:
    1.  Capture traffic destined for a `Service`'s `ClusterIP` and port.
    2.  Load balance that traffic by rewriting the destination to the IP and port of one of the backend pods listed in the `EndpointSlice`.
-   **Key Point**: `kube-proxy` is not a proxy in the traditional sense (it doesn't sit in the data path for every packet). It is a **network rule manager**.

```mermaid
graph TD
    ClientPod[Client Pod] -- "Request to Service IP <br> 10.96.10.20" --> NodeKernel[Node Kernel]
    subgraph "Node Kernel Rules"
        IPTables[iptables/IPVS rules <br> (programmed by kube-proxy)]
    end
    NodeKernel --> IPTables
    IPTables -- "DNAT to Pod IP <br> 10.1.1.5" --> BackendPod[Backend Pod]

    KubeProxy[kube-proxy] -- "Watches API Server" --> APIServer(API Server)
    APIServer -- "Service & EndpointSlice Info" --> KubeProxy
    KubeProxy -- "Programs rules" --> IPTables
```

---

## Summary

| Feature | CNI Plugin | kube-proxy |
| :--- | :--- | :--- |
| **Primary Job** | Pod IP addressing and connectivity | Service IP routing and load balancing |
| **Traffic Type** | Pod-to-Pod (East-West) | Pod-to-Service (Internal North-South) |
| **How it's invoked** | By `kubelet` during pod creation/deletion | Runs continuously, watches API server |
| **Analogy** | The road network and postal addresses | The business phone directory and receptionist |