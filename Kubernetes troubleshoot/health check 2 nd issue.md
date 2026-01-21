Here’s **another strong, real-world Kubernetes production issue**, different from memory problems and **very commonly asked in interviews**.

---

## Production Issue: Traffic Not Reaching Pods (Ingress / ALB Issue)

### **Issue Description**

In production, users suddenly started receiving **502 Bad Gateway errors**, even though pods were **running and healthy** in the Kubernetes cluster.

---

### **Symptoms Observed**

* Application pods were in **Running** state
* ALB showed targets as **Unhealthy**
* Ingress was deployed, but traffic was failing
* No application-level errors in pod logs

---

### **Troubleshooting Steps**

#### **1. Verified Pod & Service**

```bash
kubectl get pods -n prod
kubectl get svc -n prod
```

* Pods were healthy
* Service existed and endpoints were present

```bash
kubectl get endpoints <service-name> -n prod
```

---

#### **2. Checked Ingress Configuration**

```bash
kubectl describe ingress <ingress-name> -n prod
```

Found:

* ALB health check path was `/`
* Application health endpoint was `/health`

---

#### **3. Verified ALB Target Group**

* ALB was checking **wrong health path**
* Targets marked **unhealthy**, so traffic was dropped

---

#### **4. Checked Security Groups**

* ALB security group allowed traffic
* Node security group **did not allow ALB SG**
* Fixed inbound rules

---

### **Root Cause**

* **Incorrect health check path** in Ingress annotations
* Missing **security group rule** between ALB and worker nodes

---

### **Resolution Implemented**

#### **Ingress Fix**

```yaml
alb.ingress.kubernetes.io/healthcheck-path: /health
```

#### **Security Group Fix**

* Allowed inbound traffic from **ALB SG → Node SG** on application port

---

### **Validation**

```bash
kubectl get ingress -n prod
```

* ALB targets turned **healthy**
* Traffic started flowing normally

---

### **Outcome**

* 502 errors resolved
* Zero downtime after fix
* Improved Ingress validation checklist

---

### **Key Learnings**

* Ingress issues are often **infra-level**, not app-level
* Always align **health check paths**
* Security groups are a frequent hidden cause

---

### **One-Line Interview Summary**

> *The issue was caused by a misconfigured ALB health check and security group, which I resolved by correcting the health path and network rules.*

---

If you want:

* 🔹 **Node NotReady issue**
* 🔹 **HPA not scaling**
* 🔹 **ImagePullBackOff**
* 🔹 **etcd / control-plane issue**

Just say the name 👌
