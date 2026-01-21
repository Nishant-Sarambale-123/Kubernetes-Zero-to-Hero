Here’s a **clear, real production Kubernetes probe issue** that interviewers like, especially for **senior / director rounds**.

---

## Production Issue: Application Failing Due to Misconfigured Probes

### **Issue Description**

After a new production deployment, the application started **restarting continuously**, even though the application itself was healthy when accessed manually.

---

### **Symptoms Observed**

* Pods in **CrashLoopBackOff**
* Restart count increasing
* Traffic intermittently failing
* Application logs showed **no real errors**

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

Events showed:

```
Liveness probe failed: HTTP probe failed with status code 404
```

---

#### **3. Checked Probe Configuration**

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
```

---

#### **4. Verified Application Startup Time**

* Application took **60–70 seconds** to fully start
* `/health` endpoint became available **after startup**
* Probe was starting too early

---

### **Root Cause**

* **Liveness probe started before the application was ready**
* Kubernetes kept killing the container even though it was still starting
* Misuse of **liveness probe instead of startup probe**

---

### **Resolution Implemented**

#### **Correct Fix**

* Added **startupProbe**
* Adjusted readiness & liveness probes

```yaml
startupProbe:
  httpGet:
    path: /health
    port: 8080
  failureThreshold: 30
  periodSeconds: 10

livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 90

readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 60
```

---

### **Outcome**

* Pods stopped restarting
* Zero downtime during deployment
* Traffic routed only to **ready pods**
* Improved deployment stability

---

### **Key Learnings**

* **Liveness ≠ Readiness**
* Use **startupProbe** for slow-starting apps
* Incorrect probes can cause self-inflicted outages

---

### **One-Line Interview Summary**

> *The issue was caused by a misconfigured liveness probe that killed pods during startup, which I fixed by introducing a startup probe and tuning probe timings.*

---

### **If Interviewer Asks Follow-up**

**Q: How do you decide probe timings?**
👉 Based on **application startup time, load testing, and production metrics**

---

If you want, I can also give:

* 🔹 **TCP probe issue**
* 🔹 **Readiness probe causing 503s**
* 🔹 **Probe impact on rolling updates**

Just say 👍
