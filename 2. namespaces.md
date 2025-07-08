Here are detailed notes on **Namespaces in Kubernetes** – suitable for interviews, certification prep, and practical usage:

---

## 📘 **Kubernetes Namespaces - Overview**

### 🔹 What is a Namespace?

A **namespace** in Kubernetes is a logical partition or virtual cluster within a physical Kubernetes cluster. It allows you to divide cluster resources among multiple users (or teams) and isolate workloads.

---

## 🎯 **Why Use Namespaces?**

| Use Case                | Description                                                                               |
| ----------------------- | ----------------------------------------------------------------------------------------- |
| **Multi-tenancy**       | Isolate environments for different teams, projects, or applications.                      |
| **Resource Management** | Apply resource quotas, limits, and policies at a namespace level.                         |
| **Organization**        | Group related resources (Pods, Services, etc.) together logically.                        |
| **Access Control**      | Use **RBAC** (Role-Based Access Control) to define who can access what in each namespace. |

---

## 📂 **Default Namespaces in Kubernetes**

| Namespace         | Purpose                                                             |
| ----------------- | ------------------------------------------------------------------- |
| `default`         | Used when no other namespace is specified.                          |
| `kube-system`     | Contains system-level pods like DNS, controller manager, scheduler. |
| `kube-public`     | Publicly readable data; mostly used for bootstrapping.              |
| `kube-node-lease` | Tracks heartbeats from nodes for faster node failure detection.     |

---

## 🛠️ **Creating and Using Namespaces**

### ✅ Create a Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
```

```bash
kubectl apply -f namespace.yaml
```

### ✅ Create Resource in a Namespace

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: my-namespace
spec:
  containers:
    - name: my-container
      image: nginx
```

Or use CLI:

```bash
kubectl run nginx --image=nginx --namespace=my-namespace
```

---

## 🔍 **Working with Namespaces – kubectl Commands**

| Task                                | Command                                                |
| ----------------------------------- | ------------------------------------------------------ |
| List all namespaces                 | `kubectl get namespaces`                               |
| Create a namespace                  | `kubectl create namespace dev`                         |
| View resources in a namespace       | `kubectl get pods -n dev`                              |
| Set default namespace (per context) | `kubectl config set-context --current --namespace=dev` |
| Delete a namespace                  | `kubectl delete namespace dev`                         |

---

## ⚙️ **Namespace in Context Switching**

```bash
kubectl config set-context my-context --namespace=my-namespace
kubectl config use-context my-context
```

---

## 📏 **Resource Quotas and Limits in Namespaces**

You can limit the resource usage in a namespace using:

### ResourceQuota Example

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
  namespace: dev
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 1Gi
    limits.cpu: "4"
    limits.memory: 2Gi
```

### LimitRange Example

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
  namespace: dev
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```

---

## 🛡️ **RBAC in Namespaces**

RBAC can restrict access to specific namespaces.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: dev
subjects:
- kind: User
  name: alice
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## 🧪 **Best Practices**

* Use namespaces for environment separation (dev, staging, prod).
* Set **quotas** to avoid resource starvation.
* Avoid using `default` namespace for production workloads.
* Implement **RBAC** per namespace for security.
* Use **network policies** to control traffic between namespaces.

---

## ❓ Common Interview Questions

1. **What are Kubernetes namespaces and why are they used?**
2. **How do you list all namespaces and their pods?**
3. **How do you apply resource quotas in namespaces?**
4. **What is the difference between a namespace and a context?**
5. **Can two pods with the same name exist in different namespaces?**
6. **How do you restrict a user to access only a specific namespace?**

---

Let me know if you want a diagram or mind map of namespace architecture, or if you'd like flashcards or MCQs for practice.
