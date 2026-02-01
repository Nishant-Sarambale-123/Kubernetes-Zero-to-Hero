Got it — interview-style answer. Here’s a **clean, real-world troubleshooting explanation** you can say confidently 👇

---

### **How I troubleshooted a pod memory leak issue in Kubernetes**

**Situation:**
One of the application pods was getting **OOMKilled** and restarting frequently. Memory usage kept increasing over time and never came down, indicating a **possible memory leak**.

**Step 1: Identify the problem pod**
I first checked pod restarts and memory usage:

* `kubectl get pods`
* `kubectl describe pod <pod-name>`
  This confirmed the pod was getting **OOMKilled**.

**Step 2: Monitor memory usage**
I used:

* `kubectl top pod <pod-name>`
  Memory usage was continuously increasing, even with steady traffic.

**Step 3: Check resource limits**
I verified the pod spec:

* Requests and limits for memory were defined
  The pod was hitting the **memory limit**, causing Kubernetes to kill it.

**Step 4: Analyze application behavior**
I checked application logs and metrics (via **Prometheus/Grafana**):

* Memory graph showed a steady upward trend
* No garbage collection drop
  This confirmed it was not a traffic spike but a **memory leak in the application**

**Step 5: Validate by restarting pod**
After restarting the pod:

* Memory dropped to normal
* Slowly started increasing again
  This reinforced the memory leak conclusion.

**Step 6: Take corrective actions**

* Informed the development team with metrics and logs
* Temporarily increased memory limits to stabilize production
* Enabled heap dump / profiling (JVM or language-specific tools)
* Suggested fixing unclosed connections, caches, or object retention

**Result:**
After the code fix was deployed, memory usage became stable and pod restarts stopped.

---

### **One-line summary for interview**

> “I identified a memory leak by observing continuously increasing pod memory usage and OOMKills, validated it using metrics and logs, and worked with the dev team to fix the application while stabilizing production with proper memory limits.”

---

If you want, I can also give:

* 🔹 **Very short 5-line answer**
* 🔹 **JVM-specific / Node.js / Python version**
* 🔹 **STAR format for managerial interview**

Just say the word 👍
