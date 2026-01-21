Perfect 👍 here’s **another very real-world Kubernetes production issue**, something that actually happens a lot in live systems and sounds very genuine in interviews.

---

## Production Issue: `ImagePullBackOff` After Deployment (ECR / IAM Issue)

### **Issue Description**

After a new production deployment, the application did not come up and users started facing **service unavailability**. On checking the cluster, the pods were failing to start.

---

### **Symptoms Observed**

* Pods stuck in **ImagePullBackOff**
* No application logs
* Deployment rollout failed
* Service endpoints were empty

---

### **Troubleshooting Steps**

#### **1. Checked Pod Status**

```bash
kubectl get pods -n prod
```

---

#### **2. Described the Pod**

```bash
kubectl describe pod <pod-name> -n prod
```

Error message:

```
Failed to pull image: pull access denied
```

---

#### **3. Verified Image Repository**

* Image existed in **Amazon ECR**
* Image tag was correct

---

#### **4. Checked Node IAM Role**

* Worker node IAM role was **missing ECR permissions**
* Recently, a new node group was added without:

  * `AmazonEC2ContainerRegistryReadOnly` policy

---

### **Root Cause**

* **IAM policy missing on new node group**
* Nodes could not authenticate to ECR
* Kubernetes failed to pull container images

---

### **Resolution Implemented**

#### **Immediate Fix**

* Attached required IAM policy to node role:

  * `AmazonEC2ContainerRegistryReadOnly`

* Restarted pods:

```bash
kubectl rollout restart deployment <deployment-name> -n prod
```

---

### **Outcome**

* Images pulled successfully
* Pods moved to **Running**
* Service restored

---

### **Key Learnings**

* Node IAM roles are critical in EKS
* Always validate permissions when adding new node groups
* ImagePullBackOff is often **infra-related**, not app-related

---

### **One-Line Interview Summary**

> *The production issue was caused by missing ECR permissions on a new node group, which prevented image pulls; fixing the IAM role resolved the outage.*

---

### If Interviewer Asks Follow-up

**Q: How do you prevent this?**
👉 Use **infrastructure-as-code**, validate IAM policies, and add deployment pre-checks.

---

If you want **one last killer example**, I can give:

* 🔹 **Node NotReady due to DiskPressure**
* 🔹 **ConfigMap wrong value causing crash**
* 🔹 **HPA not scaling during traffic spike**
* 🔹 **Rolling update caused downtime**

Just say which one 👌
