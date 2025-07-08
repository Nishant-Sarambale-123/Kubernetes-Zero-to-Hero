Here are the **answers to common Kubernetes Namespaces interview questions**, explained clearly and concisely:

---

### ✅ **1. What are Kubernetes namespaces and why are they used?**

**Answer:**
Kubernetes **namespaces** are a way to divide cluster resources between multiple users or teams. They act like **virtual clusters** within the same physical cluster. Namespaces help in:

* **Isolating resources**
* **Managing multi-tenancy**
* **Applying access controls (RBAC)**
* **Setting resource limits (quotas)**

---

### ✅ **2. How do you list all namespaces and their pods?**

**Answer:**
To list all namespaces:

```bash
kubectl get namespaces
```

To list pods across all namespaces:

```bash
kubectl get pods --all-namespaces
```

To list pods in a specific namespace:

```bash
kubectl get pods -n <namespace-name>
```

---

### ✅ **3. How do you apply resource quotas in namespaces?**

**Answer:**
You define a `ResourceQuota` object and apply it to the desired namespace.

**Example:**

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

Apply with:

```bash
kubectl apply -f quota.yaml
```

---

### ✅ **4. What is the difference between a namespace and a context?**

**Answer:**

| Concept       | Description                                                                        |
| ------------- | ---------------------------------------------------------------------------------- |
| **Namespace** | A Kubernetes object used to logically isolate resources in a cluster.              |
| **Context**   | A `kubectl` configuration that includes cluster, user, and **namespace** settings. |

You can switch contexts to work in different clusters or namespaces using:

```bash
kubectl config use-context <context-name>
```

---

### ✅ **5. Can two pods with the same name exist in different namespaces?**

**Answer:**
Yes. Pod names **must be unique within a namespace**, but **can be the same across different namespaces**.

Example:

```bash
kubectl run nginx --image=nginx --namespace=dev
kubectl run nginx --image=nginx --namespace=prod
```

Both pods named `nginx`, but in different namespaces.

---

### ✅ **6. How do you restrict a user to access only a specific namespace?**

**Answer:**
Use **RBAC** (Role-Based Access Control) with a `Role` and `RoleBinding`.

**Example:**

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: dev
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: dev
subjects:
- kind: User
  name: alice
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

This gives **user `alice`** read-only access to pods in **`dev` namespace only**.

---

Let me know if you want these in **PDF format**, **flashcards**, or **practice MCQs** for revision!
