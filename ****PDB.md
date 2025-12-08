Here is a clean **interview-ready explanation of PodDisruptionBudget (PDB) in Kubernetes**, in your preferred format:

---

# **PodDisruptionBudget (PDB) in Kubernetes**

### **1. Question**

**What is a PodDisruptionBudget (PDB) in Kubernetes?**

---

### **2. Short explanation**

PDB ensures that a minimum number of Pods of your application always remain running during **voluntary disruptions** (like node drain, upgrades, etc.).

---

### **3. Answer**

A PodDisruptionBudget (PDB) is a policy that defines the **minimum number of Pods that must remain available** or the **maximum number of Pods that can be unavailable** during voluntary disruptions. It protects applications from going down during node maintenance, cluster upgrades, or draining actions.

---

### **4. Detailed explanation**

### ✅ **Why PDB is needed?**

When you do:

* `kubectl drain <node>`
* Upgrade a node group
* Upgrade cluster version
* Delete a node intentionally

Kubernetes will try to **evict** the Pods from that node.
Without PDB → All Pods might get evicted at the same time → **service outage**.

With PDB → Kubernetes checks PDB and allows pod eviction **only if minimum availability is preserved**.

---

### ✅ **What disruptions PDB protects from?**

**Voluntary disruptions:**

* Node drain
* Cluster upgrade
* Pod eviction API
* Scaling down of deployments
* Spot termination (technically voluntary)

**Not protected:**
⚠️ **Involuntary disruptions**, like:

* Node crash
* Hardware failure
* Kernel panic
* OOM kill

---

### ⚙️ **How PDB Works?**

You can define:

* **minAvailable** → minimum number of Pods that must stay running
* **maxUnavailable** → max Pods allowed to be down

Kubernetes compares this to current ready Pods and decides whether eviction is allowed or denied.

---

### 📄 **YAML Example (minAvailable)**

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: myapp
```

Meaning:

* Application must have **at least 2 Pods running** always.
* If replicas = 3 → Only 1 pod can be evicted at a time.

---

### 📄 **YAML Example (maxUnavailable)**

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: cart-pdb
spec:
  maxUnavailable: 1
  selector:
    matchLabels:
      app: cart
```

Meaning:

* At most **1 Pod can be unavailable** at any moment.

---

### ⭐ Real-World Example

If your Deployment has **5 Pods**, and PDB says:

```yaml
minAvailable: 4
```

Then during node drain:

* Only **1 Pod** can be evicted at a time.
* Kubernetes will recreate a new Pod on another node.
* After it becomes Ready → next Pod is allowed eviction.

This ensures **zero downtime** during planned maintenance.

---

### **5. Summary Table**

| Parameter                   | Meaning                             | Example              |
| --------------------------- | ----------------------------------- | -------------------- |
| **minAvailable**            | Minimum Pods that must stay running | `minAvailable: 2`    |
| **maxUnavailable**          | Maximum Pods allowed to be down     | `maxUnavailable: 1`  |
| **Voluntary disruptions**   | Protected                           | node drain, upgrades |
| **Involuntary disruptions** | Not protected                       | node crash           |
| **Selector**                | Label selector for Pods             | `app: nginx`         |

---

### **6. Key Takeaway**

**PDB prevents all Pods from going down at once during maintenance by controlling how many Pods can be disrupted at a time.**

---

If you want, I can also provide:
✔ Scenario-based PDB questions
✔ PDB best practices in production
✔ PDB + Node Drain workflow diagrams
