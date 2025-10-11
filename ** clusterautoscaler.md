Absolutely, Nishant! Let’s create **detailed notes for Cluster Autoscaler (CA) on EKS**, including **step-by-step explanation, YAML, and best practices** — everything you can use for interviews or practical deployment. I’ll keep it structured so it’s easy to memorize and explain.

---

# **Cluster Autoscaler on Amazon EKS – Detailed Notes**

## **1️⃣ What is Cluster Autoscaler (CA)?**

* A **Kubernetes component** that **automatically adjusts the number of worker nodes** in an EKS cluster.
* Scales nodes **up** if pods cannot be scheduled due to resource constraints.
* Scales nodes **down** if nodes are underutilized (no pods running for a while).

### **Key Points:**

* Works with **managed node groups**, **unmanaged node groups**, or **Auto Scaling Groups (ASG)**.
* Runs as a **deployment in the cluster** (`kube-system` namespace).
* Requires **AWS IAM permissions** (or IRSA) to modify node groups.

---

## **2️⃣ Why Use Cluster Autoscaler?**

* Automatically scales your cluster to meet demand.
* Reduces **costs** by removing idle nodes.
* Ensures workloads are **always scheduled** without manual intervention.
* Works best with **replicated workloads**, PodDisruptionBudgets (PDBs), and multi-AZ node groups.

---

## **3️⃣ Pre-requisites**

1. **EKS Cluster** (with at least one managed or unmanaged node group).
2. **Auto Scaling enabled** for the node group:

   ```bash
   aws eks update-nodegroup-config \
     --cluster-name prod-cluster \
     --nodegroup-name managed-ng \
     --scaling-config minSize=2,maxSize=5,desiredSize=2
   ```
3. **IAM Policy / IRSA** for Cluster Autoscaler:

**Example IAM Policy:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "autoscaling:DescribeAutoScalingGroups",
        "autoscaling:DescribeAutoScalingInstances",
        "autoscaling:SetDesiredCapacity",
        "autoscaling:TerminateInstanceInAutoScalingGroup",
        "autoscaling:DescribeLaunchTemplateVersions",
        "ec2:DescribeLaunchTemplateVersions"
      ],
      "Resource": "*"
    }
  ]
}
```

* Attach this to **Cluster Autoscaler’s ServiceAccount** via IRSA (recommended).

---

## **4️⃣ Deployment YAML for Cluster Autoscaler**

> The following example is production-ready and works with **managed node groups** using IRSA.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<ACCOUNT_ID>:role/ClusterAutoscalerRole
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
      annotations:
        cluster-autoscaler.kubernetes.io/safe-to-evict: "false"
    spec:
      serviceAccountName: cluster-autoscaler
      containers:
        - name: cluster-autoscaler
          image: registry.k8s.io/autoscaler/cluster-autoscaler:v1.30.0
          command:
            - ./cluster-autoscaler
            - --v=4
            - --cluster-name=prod-cluster
            - --balance-similar-node-groups
            - --skip-nodes-with-local-storage=false
            - --skip-nodes-with-system-pods=false
          resources:
            limits:
              cpu: 100m
              memory: 300Mi
            requests:
              cpu: 100m
              memory: 300Mi
```

### **Important Annotations & Flags**

| Flag / Annotation                                         | Purpose                                                 |
| --------------------------------------------------------- | ------------------------------------------------------- |
| `cluster-autoscaler.kubernetes.io/safe-to-evict: "false"` | Prevents CA pod from being terminated during scale-down |
| `--balance-similar-node-groups`                           | Distributes pods evenly across similar node groups      |
| `--skip-nodes-with-local-storage=false`                   | Allows nodes with local storage to be scaled down       |
| `--skip-nodes-with-system-pods=false`                     | Allows scaling down nodes with system pods if safe      |

---

## **5️⃣ How CA Works (Step-by-Step)**

### **Scale Up**

1. Pod cannot be scheduled due to CPU/memory limits.
2. Cluster Autoscaler detects pending pods.
3. Sends request to node group / ASG to **add nodes**.
4. New nodes are provisioned and pods are scheduled.

### **Scale Down**

1. Nodes are underutilized (pods moved or finished).
2. Cluster Autoscaler checks node’s **utilization & PodDisruptionBudgets (PDBs)**.
3. Safely drains and deletes nodes.

---

## **6️⃣ Verification**

* Check Cluster Autoscaler logs:

```bash
kubectl -n kube-system logs -f deployment/cluster-autoscaler
```

* Look for messages:

```
I0925 12:00:00 Scale-up: adding 1 node(s)
I0925 12:10:00 Scale-down: removing node ip-192-168-1-123.ec2.internal
```

* Test scale-up by creating a resource-heavy deployment:

```bash
kubectl run stress --image=busybox --replicas=10 -- /bin/sh -c "while true; do echo hello; sleep 10; done"
```

---

## **7️⃣ Best Practices**

* Use **IRSA** for secure permissions.
* Always define **min/max nodes** for node groups.
* Ensure **PodDisruptionBudgets** are configured for production workloads.
* Monitor **logs** and **node utilization metrics**.
* Test in **staging** before production deployment.
* Update CA image when new EKS Kubernetes versions are released.

---

## **8️⃣ Interview Explanation (Simple Version)**

> “Cluster Autoscaler in EKS automatically scales worker nodes based on workload demand.
> It adds nodes when pods can’t be scheduled and removes idle nodes to save cost.
> We deploy it in the kube-system namespace using a dedicated ServiceAccount with IAM permissions (IRSA).
> Important flags like `--balance-similar-node-groups` and `safe-to-evict` ensure safe scaling.
> This setup is critical for production clusters to handle variable workloads without manual intervention.”

---

If you want, I can **combine this with EBS CSI, VPC CNI, and CoreDNS notes** into **one complete “EKS Production Add-ons Cheat Sheet”** with all YAML and commands in **one page** — very handy for interviews.

Do you want me to do that next?
