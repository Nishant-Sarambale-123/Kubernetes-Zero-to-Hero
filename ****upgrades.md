Excellent follow-up 👏 — you’re absolutely right!
In **EKS upgrades**, after updating the **control plane** and **node groups**, one critical step often missed in interviews is **updating the add-ons**.

Let’s go over the **complete and production-ready EKS upgrade process**, including **add-on updates** — this is exactly how a DevOps engineer should explain it in an interview.

---

## 🚀 **Full EKS Upgrade Process (Production Ready)**

### **🎯 Objective**

Safely upgrade your **Amazon EKS cluster** with **zero downtime**, including:

1. Control Plane
2. Worker Nodes / Node Groups
3. Add-ons (CNI, CoreDNS, kube-proxy, etc.)

---

## 🪜 Step-by-Step Process

---

### **Step 1️⃣ — Check Current Cluster Version**

```bash
aws eks describe-cluster --name <cluster-name> --query "cluster.version"
```

✅ Output example: `1.27`

---

### **Step 2️⃣ — Review Supported Versions**

EKS supports **3 minor versions** at a time (e.g., 1.26 → 1.27 → 1.28 → 1.29).

Check:
👉 [EKS Kubernetes versions](https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html)

---

### **Step 3️⃣ — Upgrade EKS Control Plane**

Control plane must be updated **first**.

```bash
eksctl upgrade cluster --name <cluster-name> --version 1.29
```

* AWS performs rolling upgrades internally.
* No downtime for workloads.
* Takes around 10–15 minutes.

---

### **Step 4️⃣ — Update Local Tools**

After control plane upgrade:

```bash
kubectl version --client
eksctl version
aws --version
```

✅ Ensure versions are compatible with the new Kubernetes release.

---

### **Step 5️⃣ — Upgrade Node Groups**

#### If you use **Managed Node Groups**:

```bash
eksctl upgrade nodegroup \
  --cluster <cluster-name> \
  --name <nodegroup-name>
```

* Replaces nodes one by one (draining old, adding new).
* Maintains workload availability.

#### If you use **Self-Managed Node Groups**:

1. Create a new **Launch Template** with the updated EKS-optimized AMI:

   ```bash
   aws ssm get-parameter \
     --name /aws/service/eks/optimized-ami/1.29/amazon-linux-2/recommended/image_id
   ```
2. Update your Auto Scaling Group (ASG) to use this new AMI.
3. Drain old nodes:

   ```bash
   kubectl drain <node-name> --ignore-daemonsets --delete-local-data
   ```
4. Terminate old instances gracefully.

---

### **Step 6️⃣ — Verify Nodes and Pods**

```bash
kubectl get nodes
kubectl get pods -A
```

✅ All nodes should show `Ready`.
✅ All pods should be running and healthy.

---

### **Step 7️⃣ — Update Add-ons (Very Important)**

After every control plane upgrade, **update the core EKS add-ons** to ensure compatibility.

---

#### **1️⃣ Update Amazon VPC CNI Plugin**

Responsible for networking (Pod IP assignment).

```bash
eksctl utils update-aws-node --cluster <cluster-name> --approve
```

---

#### **2️⃣ Update CoreDNS**

Responsible for internal DNS resolution inside the cluster.

```bash
eksctl utils update-coredns --cluster <cluster-name> --approve
```

---

#### **3️⃣ Update kube-proxy**

Handles communication between Kubernetes services and pods.

```bash
eksctl utils update-kube-proxy --cluster <cluster-name> --approve
```

---

#### **4️⃣ (Optional) Update Add-ons via Console**

AWS now supports **managed add-ons** that you can update from console or CLI:

```bash
aws eks list-addons --cluster-name <cluster-name>
aws eks update-addon \
  --cluster-name <cluster-name> \
  --addon-name vpc-cni \
  --addon-version <latest-version> \
  --resolve-conflicts OVERWRITE
```

---

### **Step 8️⃣ — Validate Cluster and Application Health**

* Check node and pod status:

  ```bash
  kubectl get nodes -o wide
  kubectl get pods -A
  ```
* Check DNS:

  ```bash
  kubectl exec -it <pod> -- nslookup kubernetes.default
  ```
* Check metrics and logs in **CloudWatch / Prometheus / Grafana**.
* Ensure no CrashLoopBackOff or Pending pods.

---

### **Step 9️⃣ — Cleanup Old Node Groups (If Applicable)**

After validation:

```bash
eksctl delete nodegroup --cluster <cluster-name> --name <old-nodegroup>
```

---

### ⚙️ **Step 🔟 — Update Add-on Helm Charts (if used)**

If you installed add-ons like:

* **Metrics Server**
* **Cluster Autoscaler**
* **Ingress Controller (NGINX / ALB)**

Then upgrade them:

```bash
helm repo update
helm upgrade <release-name> <chart-name> --namespace <ns>
```

---

## 🧠 **Interview-Ready 45-Second Summary**

> “When updating a production EKS cluster, I always follow a controlled process.
> First, I upgrade the **control plane** using `eksctl upgrade cluster` — AWS manages that part safely.
> Then I update the **managed node groups**, which replaces nodes one by one while draining pods to prevent downtime.
> After that, I update core **EKS add-ons** — CNI, CoreDNS, and kube-proxy — using `eksctl utils update-*` commands or `aws eks update-addon`.
> Finally, I verify that all nodes are `Ready`, workloads are healthy, and cluster networking works fine.
> I always test this upgrade flow in a **staging cluster first**, ensure **PodDisruptionBudgets** and **autoscaling** are configured, and monitor via **CloudWatch and Grafana** to ensure smooth rollout.”

---

Would you like me to give a **text-based EKS upgrade flow diagram (control plane → node group → add-ons)** that you can memorize or show in an interview? It makes it easy to visualize the full upgrade path.
Excellent and very realistic question 👏 — and yes, this is exactly what interviewers look for.

Let’s go through this clearly 👇

---

## ⚙️ **When You Upgrade EKS in Production — What Actually Happens & Is There Any Impact?**

### ✅ **1️⃣ Control Plane Upgrade (No Direct Downtime)**

* The **control plane** (API server, etcd, scheduler, controller manager) is **managed by AWS**.
* AWS performs a **rolling upgrade** of control plane components **behind the scenes**.
* During this time:

  * The **API server** might have a few **short interruptions** (a few seconds) — usually negligible.
  * **Running workloads (pods, services)** continue to run normally.
  * **No impact on the application layer** unless you’re continuously hitting the Kubernetes API.

**👉 So in production:** No downtime, but it’s good to avoid deployments or API-heavy operations during the control plane upgrade.

---

### ✅ **2️⃣ Node Group Upgrade (Can Have Temporary Impact if Not Planned Well)**

When you upgrade node groups:

* Old nodes are replaced with new nodes running the newer Kubernetes version.
* EKS (or `eksctl`) will **drain** nodes one by one:

  * Pods on that node are **evicted** and rescheduled to other nodes.
  * This causes a brief restart of those pods.

**👉 In production, if properly configured:**

* There’s **no user-visible downtime** if:

  * You have **replicas > 1** for all critical deployments.
  * **PodDisruptionBudgets (PDBs)** are set correctly.
  * **Auto Scaling** ensures enough spare capacity.

**❌ Downtime risk only happens if:**

* You have **single-pod deployments**.
* No **PDBs** configured.
* Insufficient capacity in the cluster.

---

### ✅ **3️⃣ Add-on Upgrade (Low Risk but Important)**

When updating:

* **CNI** – affects pod networking; upgrade carefully.
* **CoreDNS** – affects DNS resolution inside cluster.
* **kube-proxy** – affects service routing.

AWS updates them in a **rolling, safe manner**, but it’s good to:

* Do one add-on at a time.
* Validate networking and DNS after each update.

**👉 Impact:** Minimal if done sequentially and validated.

---

### ✅ **4️⃣ Overall Recommendation for Production**

| Phase         | Downtime Risk            | Recommended Practice                           |
| ------------- | ------------------------ | ---------------------------------------------- |
| Control Plane | 🚫 None (Minor API blip) | Avoid deployments during upgrade               |
| Node Group    | ⚠️ Low to Medium         | Use replicas, PDBs, and drain nodes gracefully |
| Add-ons       | ⚠️ Low                   | Update one by one, verify cluster health       |
| Validation    | ✅ Always                 | Test in staging first                          |

---

### 🧠 **In Interview — How to Say It:**

> “In production, EKS upgrades are mostly non-disruptive because AWS handles the control plane in a rolling manner. However, when upgrading node groups, pods on old nodes are drained and rescheduled, so if you don’t have enough replicas or proper PDBs, you can experience brief service interruptions.
> To avoid that, I ensure high availability, test in staging, and perform rolling node updates. Add-ons like CNI and CoreDNS are upgraded sequentially with verification to ensure zero downtime.”

---

Would you like me to give a **safe upgrade checklist for production EKS (pre-, during-, post-upgrade)** — a short version that you can quote confidently in interviews or even use practically at work?
