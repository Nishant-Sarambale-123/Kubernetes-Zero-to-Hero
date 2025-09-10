Great question 👍 — this is exactly what comes up in **Kubernetes production interviews**. Let me break it down clearly for you.

---

# 🚀 Blue-Green vs Canary Deployments in Kubernetes (Production)

## 1. **Blue-Green Deployment**

👉 You run **two identical environments** (Blue = current version, Green = new version).
👉 Once the new version is ready & tested, you **switch traffic** from Blue → Green with minimal downtime.

### How to do it in Kubernetes:

* **Step 1:** Deploy the current version (`blue`) Deployment + Service.
* **Step 2:** Deploy the new version (`green`) Deployment, but don’t route traffic yet.
* **Step 3:** Run tests on `green` pods (smoke tests, health checks).
* **Step 4:** Update the **Service selector** (or Ingress) to point to `green`.
* **Step 5:** If everything is good → delete `blue`. If not → rollback by pointing back to `blue`.

🔹 **Tools used in production:**

* Native **Kubernetes Services** (simple cases).
* **Ingress controllers** (NGINX, ALB, Traefik) for traffic switching.
* **Argo Rollouts** (GitOps + advanced traffic control).
* **Service Mesh** (Istio/Linkerd) for 100% traffic shifting.

📌 Example Service switch:

```yaml
selector:
  app: my-app
  version: green   # switch from blue → green
```

---

## 2. **Canary Deployment**

👉 You release the new version gradually:

* Start with 5–10% traffic → monitor → increase → finally 100%.
* Safer for **production** since you detect issues early without a big-bang release.

### How to do it in Kubernetes:

* **Step 1:** Have two Deployments: `stable` + `canary`.
* **Step 2:** Use Service Mesh (Istio, Linkerd) or Ingress controller for **traffic splitting**.
* **Step 3:** Route small % traffic to canary.
* **Step 4:** Gradually increase if metrics/logs look healthy.
* **Step 5:** Promote to stable or rollback if issues.

🔹 **Tools used in production:**

* **Argo Rollouts** → declarative progressive delivery.
* **Flagger (FluxCD)** → automatic canary analysis with Prometheus/Datadog.
* **Istio / NGINX Ingress** → weighted traffic routing.

📌 Example Canary with Istio VirtualService:

```yaml
http:
- route:
  - destination:
      host: my-app
      subset: stable
    weight: 90
  - destination:
      host: my-app
      subset: canary
    weight: 10
```

---

## 🔑 Which is Better for Production?

* **Blue-Green:**
  ✅ Faster rollback, good for critical apps.
  ❌ Needs double resources (running two full environments).

* **Canary:**
  ✅ Safer, gradual rollout with monitoring.
  ❌ More complex (needs service mesh/traffic management).

👉 In **real-world production**:

* **Business-critical apps** → Canary (safer, progressive).
* **Smaller apps / non-critical** → Blue-Green (simple, fast rollback).
* Many teams use **Argo Rollouts** to handle both strategies declaratively.

---

Do you want me to also give you a **ready-made production Argo Rollouts YAML** for both **Blue-Green** and **Canary**, so you can directly show it in your interview?
Perfect 👌 You want **Ingress Controller YAML examples** for **Blue-Green** and **Canary** in Kubernetes (on AWS, usually with **AWS ALB Ingress Controller** or **NGINX Ingress Controller**).

Let’s do both step by step.

---

# 1️⃣ Blue-Green Deployment with Ingress (AWS ALB or NGINX)

Here we run two versions (`blue` and `green`) and switch traffic in Ingress.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: blue
  template:
    metadata:
      labels:
        app: my-app
        version: blue
    spec:
      containers:
      - name: app
        image: myrepo/my-app:1.0
        ports:
        - containerPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: green
  template:
    metadata:
      labels:
        app: my-app
        version: green
    spec:
      containers:
      - name: app
        image: myrepo/my-app:2.0
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
spec:
  selector:
    app: my-app
    version: blue   # 👉 Start with BLUE, later switch to GREEN
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  annotations:
    kubernetes.io/ingress.class: alb   # or nginx
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-svc
            port:
              number: 80
```

👉 To switch from **blue → green**, just update the **Service selector**:

```yaml
spec:
  selector:
    app: my-app
    version: green
```

---

# 2️⃣ Canary Deployment with Ingress (Weighted Traffic)

With **AWS ALB Ingress Controller** or **NGINX Ingress**, you can define **canary weights**.

### Canary using NGINX Ingress annotations:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-stable
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      track: stable
  template:
    metadata:
      labels:
        app: my-app
        track: stable
    spec:
      containers:
      - name: app
        image: myrepo/my-app:1.0
        ports:
        - containerPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
      track: canary
  template:
    metadata:
      labels:
        app: my-app
        track: canary
    spec:
      containers:
      - name: app
        image: myrepo/my-app:2.0
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: my-app-stable-svc
spec:
  selector:
    app: my-app
    track: stable
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: my-app-canary-svc
spec:
  selector:
    app: my-app
    track: canary
  ports:
  - port: 80
    targetPort: 80
---
# Stable ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-stable-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-stable-svc
            port:
              number: 80
---
# Canary ingress (routes 20% traffic to new version)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-canary-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "20"   # 👉 20% traffic
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-canary-svc
            port:
              number: 80
```

👉 Increase `canary-weight` gradually (20 → 50 → 100) until stable.

---

# ✅ Summary

* **Blue-Green:** Switch traffic by updating Service → simple & fast rollback.
* **Canary:** Gradual rollout with `nginx.ingress.kubernetes.io/canary-weight`.
* Works with **AWS ALB** or **NGINX Ingress Controller** in production.

---

Do you want me to also show the **Argo Rollouts YAML** for Blue-Green & Canary (production-ready GitOps way), so you can directly demo both in interviews?
