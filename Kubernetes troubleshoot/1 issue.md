Here’s a **clear, interview-ready production issue answer** you can say confidently for a **DevOps / Kubernetes role** (director round friendly).

---

## Production Issue: Pods Restarting & Application Downtime in Kubernetes

### **Issue Description**

In one of our **production EKS clusters**, a critical microservice started **frequently restarting**, which caused **intermittent API failures and increased latency** for users.

---

### **Symptoms Observed**

* Users reported **502/504 errors**
* Pod status showed **CrashLoopBackOff**
* Kubernetes dashboard showed **high restart count**
* ALB target health checks were **failing intermittently**

---

### **Troubleshooting Steps**

#### **1. Checked Pod Status**

```bash
kubectl get pods -n prod
```

Found pods restarting continuously.

---

#### **2. Analyzed Pod Logs**

```bash
kubectl logs <pod-name> -n prod --previous
```

Observed **OutOfMemory (OOMKilled)** errors.

---

#### **3. Described the Pod**

```bash
kubectl describe pod <pod-name> -n prod
```

Confirmed:

* Container memory limit was **512Mi**
* Application was consuming **700–800Mi** during peak traffic

---

#### **4. Checked Resource Metrics**

```bash
kubectl top pod -n prod
```

Memory usage exceeded defined limits.

---

#### **5. Verified HPA & Traffic**

* HPA was scaling pods based on **CPU only**
* Memory spikes were not considered
* Sudden traffic increase caused memory pressure

---

### **Root Cause**

* **Incorrect resource limits** for the application
* **HPA not configured for memory metrics**
* High traffic caused memory exhaustion → pod restarts

---

### **Resolution Implemented**

#### **Immediate Fix**

* Increased memory limits:

```yaml
resources:
  requests:
    memory: "512Mi"
  limits:
    memory: "1Gi"
```

* Restarted deployment:

```bash
kubectl rollout restart deployment <deployment-name> -n prod
```

---

#### **Long-Term Fix**

* Enabled **HPA with CPU + memory metrics**
* Implemented **proper load testing**
* Added **Prometheus & Grafana alerts** for:

  * Pod restarts
  * Memory usage > 80%
* Tuned JVM/application memory parameters

---

### **Outcome**

* Pod restarts stopped completely
* Application stabilized under peak traffic
* API error rate dropped to **zero**
* Improved overall production reliability

---

### **Key Learnings (What Interviewers Like to Hear)**

* Always set **accurate resource requests & limits**
* Monitor **memory, not just CPU**
* Proactive alerts prevent outages
* Production issues require both **quick mitigation and permanent fixes**

---

### **One-Line Summary (Very Important)**

> *The production issue was caused by memory limit misconfiguration, which I identified using logs, metrics, and pod descriptions, and resolved by tuning resources, HPA, and monitoring.*

---

If you want, I can also give you:

* 🔹 **Another real issue** (Ingress / ALB / DNS / etcd)
* 🔹 **Director-level concise answer (30 seconds)**
* 🔹 **STAR format answer**

Just tell me 👍
