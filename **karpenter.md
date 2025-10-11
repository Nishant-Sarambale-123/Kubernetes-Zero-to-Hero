Perfect, Nishant — now let’s dive into **Karpenter**, another important EKS interview topic, especially when discussing **dynamic node provisioning** and **autoscaling**. I’ll explain it in **simple, clear terms**.

---

# **Karpenter in EKS – Detailed Notes**

## **1️⃣ What is Karpenter?**

* Karpenter is a **modern, open-source Kubernetes node autoscaler** developed by AWS.
* Unlike **Cluster Autoscaler**, which scales existing node groups, Karpenter **dynamically provisions new EC2 instances** (nodes) based on pod requirements.
* Focuses on **fast, efficient scaling** for production workloads.

📘 **Simple words:**

> Karpenter automatically launches and terminates the **right type of EC2 instance** for your pods, instead of just scaling pre-defined node groups.

---

## **2️⃣ Why Use Karpenter?**

* Faster scaling than Cluster Autoscaler.
* Supports **custom instance types, spot instances, and multiple Availability Zones**.
* Optimizes **cost** by choosing the most efficient instance type for each workload.
* Automatically terminates unused nodes, like Cluster Autoscaler.

---

## **3️⃣ How Karpenter Works**

1. **Pod Scheduling:** Pod cannot be scheduled because no suitable node exists.
2. **Provisioning:** Karpenter identifies **optimal instance type** (CPU, memory) and launches it in the correct subnet/AZ.
3. **Scheduling Pods:** Pod is scheduled on the new node.
4. **Scale Down:** Unused nodes are terminated automatically after pods are gone.

---

## **4️⃣ Components**

* **Controller:** Runs inside the cluster, monitors pending pods.
* **Provisioners:** CRDs defining rules for node provisioning, e.g., instance types, AZs, capacity types (on-demand/spot).
* **IAM Role:** Permissions to create/terminate EC2 instances and modify networking.

---

## **5️⃣ Installing Karpenter on EKS**

### **Step 1: Create IAM Role**

* Karpenter needs a role with permissions to launch EC2 instances:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:RunInstances",
        "ec2:TerminateInstances",
        "ec2:DescribeInstances",
        "ec2:CreateTags",
        "ec2:DeleteTags"
      ],
      "Resource": "*"
    }
  ]
}
```

* Attach role using **IRSA (recommended)**.

---

### **Step 2: Install Karpenter CRDs**

```bash
kubectl apply -f https://github.com/aws/karpenter/releases/latest/download/karpenter-crds.yaml
```

---

### **Step 3: Install Karpenter Controller**

```bash
kubectl apply -f https://github.com/aws/karpenter/releases/latest/download/karpenter-controller.yaml
```

---

### **Step 4: Create a Provisioner**

Example **provisioner.yaml**:

```yaml
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: default
spec:
  cluster:
    name: prod-cluster
    endpoint: https://XXXX.gr7.us-east-1.eks.amazonaws.com
  ttlSecondsAfterEmpty: 30
  limits:
    resources:
      cpu: 1000
  provider:
    subnetSelector:
      karpenter.sh/discovery: prod-cluster
    securityGroupSelector:
      karpenter.sh/discovery: prod-cluster
  requirements:
    - key: "karpenter.k8s.aws/instance-type"
      operator: In
      values: ["m5.large", "m5.xlarge", "t3.large"]
    - key: "karpenter.k8s.aws/capacity-type"
      operator: In
      values: ["spot", "on-demand"]
```

* **ttlSecondsAfterEmpty:** Nodes are terminated 30 seconds after pods leave.
* **requirements:** Specifies instance types and capacity types.

---

### **Step 5: Deploy a Test Workload**

```bash
kubectl run stress --image=busybox --replicas=5 -- /bin/sh -c "while true; do echo hello; sleep 10; done"
```

* Karpenter will **launch new EC2 nodes** to schedule these pods automatically.

---

## **6️⃣ Differences: Karpenter vs Cluster Autoscaler**

| Feature           | Cluster Autoscaler                 | Karpenter                                            |
| ----------------- | ---------------------------------- | ---------------------------------------------------- |
| Node scaling      | Only scales existing node groups   | Dynamically provisions new nodes                     |
| Speed             | Slower, waits for ASG to add nodes | Faster, launches EC2 instances directly              |
| Cost optimization | Works within node group limits     | Chooses optimal instance type and spot/on-demand mix |
| Configuration     | Requires ASG/node group min/max    | Provisioner CRD defines rules                        |
| Flexibility       | Limited to node group sizes        | Highly flexible, multi-AZ, multi-instance types      |

---

## **7️⃣ Interview-Ready Explanation**

> “Karpenter is a modern Kubernetes node autoscaler for EKS.
> Unlike Cluster Autoscaler, which scales pre-defined node groups, Karpenter dynamically provisions the right EC2 instance type based on pod requirements.
> It optimizes cost, supports spot instances, and terminates unused nodes automatically.
> In production, it’s ideal for variable workloads that require fast scaling and flexibility across instance types and availability zones.”

---

If you want, I can make a **full EKS Add-ons + Autoscaling + Karpenter + Cluster Autoscaler + EBS CSI + Kustomize cheat sheet** in **one page** — super useful for interviews and quick review.

Do you want me to create that?
