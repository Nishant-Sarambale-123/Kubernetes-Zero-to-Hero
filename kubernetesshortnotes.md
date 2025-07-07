Here's a **detailed set of Kubernetes notes** specifically structured for **interview preparation**, covering core concepts, practical usage, and common interview areas with examples:

---

## 🔹 Kubernetes (K8s) Interview Notes

### ✅ 1. **What is Kubernetes?**

* **Definition**: Open-source container orchestration platform for automating deployment, scaling, and operations of application containers.
* Originally developed by Google, now maintained by CNCF (Cloud Native Computing Foundation).
* Works with Docker and other container runtimes (e.g., containerd).

---

### ✅ 2. **Core Concepts**

| Component       | Description                                                                     |
| --------------- | ------------------------------------------------------------------------------- |
| **Pod**         | Smallest deployable unit. Holds one or more containers.                         |
| **Node**        | Worker machine (VM or physical) in Kubernetes.                                  |
| **Cluster**     | Set of nodes managed by Kubernetes master components.                           |
| **Namespace**   | Virtual cluster inside a physical cluster. Used for multi-tenancy.              |
| **Service**     | Exposes a set of pods as a network service (ClusterIP, NodePort, LoadBalancer). |
| **Deployment**  | Provides declarative updates for Pods and ReplicaSets.                          |
| **ReplicaSet**  | Ensures a specified number of pod replicas are running.                         |
| **StatefulSet** | Manages stateful applications (e.g., databases).                                |
| **DaemonSet**   | Runs a pod on all (or selected) nodes.                                          |
| **Job/CronJob** | Creates pods to run batch or scheduled tasks.                                   |
| **Ingress**     | Manages external HTTP(S) access to services.                                    |

---

### ✅ 3. **Kubernetes Architecture**

#### Control Plane Components:

* `kube-apiserver`: Frontend for the Kubernetes control plane.
* `etcd`: Key-value store for all cluster data.
* `kube-scheduler`: Assigns pods to nodes.
* `kube-controller-manager`: Runs controllers (e.g., Node, Replication).
* `cloud-controller-manager`: Integrates with cloud providers.

#### Node Components:

* `kubelet`: Ensures containers are running in a pod.
* `kube-proxy`: Maintains network rules and load balances.
* `Container runtime`: e.g., Docker, containerd.

---

### ✅ 4. **Kubernetes Objects & YAML**

Example of a Deployment YAML:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

---

### ✅ 5. **Kubernetes Networking**

* **Each Pod** gets a unique IP.
* **Service** provides stable networking across dynamic pod IPs.
* **Cluster Networking Models**:

  * **ClusterIP**: Accessible only within the cluster.
  * **NodePort**: Exposes on each node’s IP.
  * **LoadBalancer**: Uses external load balancer (cloud platforms).
* **Network Policies**: Controls traffic flow between pods.

---

### ✅ 6. **Storage in Kubernetes**

* Volumes can be:

  * `emptyDir`: Temporary storage.
  * `hostPath`: Access host filesystem.
  * `persistentVolume` (PV) and `persistentVolumeClaim` (PVC): Decouples storage from pods.
* Supports dynamic provisioning (AWS EBS, GCP PD, etc.).

---

### ✅ 7. **Helm (Kubernetes Package Manager)**

* Used to define, install, and upgrade Kubernetes applications using **charts**.

```bash
helm install myapp ./mychart
helm upgrade myapp ./mychart
helm uninstall myapp
```

---

### ✅ 8. **ConfigMaps and Secrets**

* **ConfigMap**: Stores non-sensitive configuration data.
* **Secret**: Stores sensitive data like passwords, API keys.

---

### ✅ 9. **Probes (Health Checks)**

* `livenessProbe`: Checks if app is alive.
* `readinessProbe`: Checks if app is ready to serve traffic.
* `startupProbe`: For slow-starting containers.

---

### ✅ 10. **Labels and Selectors**

* Labels are key-value pairs attached to objects.
* Selectors are used to query objects based on labels.

---

### ✅ 11. **RBAC (Role-Based Access Control)**

* **Role**: Permissions within a namespace.
* **ClusterRole**: Permissions across the cluster.
* **RoleBinding** and **ClusterRoleBinding**: Assigns roles to users/groups.

---

### ✅ 12. **Common kubectl Commands**

```bash
kubectl get pods                          # List pods
kubectl describe pod <name>               # Detailed pod info
kubectl apply -f <file>.yaml              # Apply config
kubectl delete -f <file>.yaml             # Delete resource
kubectl logs <pod-name>                   # View logs
kubectl exec -it <pod> -- /bin/bash       # Access pod shell
kubectl get nodes                         # Show all nodes
kubectl top pod                           # Show resource usage
```

---

## 🔸 Kubernetes Interview Questions (with Answers)

### Q1. What is a pod in Kubernetes?

**A**: The smallest deployable unit that can contain one or more containers sharing the same network and storage.

---

### Q2. How does Kubernetes handle scaling?

**A**: Using the `replicas` field in Deployment, or using the **Horizontal Pod Autoscaler** based on CPU/memory metrics.

---

### Q3. What is the difference between Deployment, StatefulSet, and DaemonSet?

| Resource    | Purpose                                        |
| ----------- | ---------------------------------------------- |
| Deployment  | Stateless apps, easily scalable                |
| StatefulSet | Stateful apps, persistent identity/storage     |
| DaemonSet   | One pod per node, e.g., for logging/monitoring |

---

### Q4. What happens when a pod crashes?

**A**: The `kubelet` restarts it automatically. If part of a deployment, a new pod is scheduled.

---

### Q5. How does Kubernetes perform rolling updates?

**A**: Deployment strategy ensures pods are updated one by one to minimize downtime. Can rollback if issues occur.

---

### Q6. How are secrets secured in Kubernetes?

**A**: Stored in etcd (base64 encoded), access restricted via RBAC. Should be encrypted at rest.

---

### Q7. What is the use of Ingress?

**A**: Manages external HTTP/HTTPS traffic and routes to internal services via path or host rules.

---

### Q8. How do you perform blue-green or canary deployments?

* Use multiple deployments with different labels.
* Use a service to gradually shift traffic between versions.

---

### Q9. What is a ReplicaSet?

**A**: Ensures a specified number of pod replicas are running. Used automatically by Deployments.

---

### Q10. What is the difference between a ConfigMap and a Secret?

| Feature   | ConfigMap     | Secret             |
| --------- | ------------- | ------------------ |
| Data Type | Non-sensitive | Sensitive (base64) |
| Use Case  | App config    | Passwords, tokens  |

---

## 🔸 Scenarios to Prepare

1. **You need to deploy a web app with autoscaling and expose it externally.**
2. **How would you debug a CrashLoopBackOff pod?**
3. **A pod cannot access another pod in a different namespace. What could be wrong?**
4. **You want to upgrade your app with zero downtime. How will you achieve it in Kubernetes?**
5. **You want to restrict pod-to-pod communication. How?** → Use **NetworkPolicies**.

---

Would you like:

* A printable PDF version of these notes?
* A separate section for **real-life troubleshooting scenarios**?
* Notes with diagrams?

Let me know and I’ll generate accordingly.
