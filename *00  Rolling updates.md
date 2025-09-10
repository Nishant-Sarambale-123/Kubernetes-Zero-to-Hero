Good one 👍 Rolling updates are the **default update strategy** in Kubernetes — they let you update Deployments without downtime.

Here’s a breakdown you can use in an interview or in production:

---

## 🔹 1. **How Rolling Update Works**

* When you update a **Deployment** (e.g., new image, env var, replica count), Kubernetes gradually replaces old Pods with new ones.
* It respects two parameters:

  * **maxUnavailable** → how many Pods can be down during the update.
  * **maxSurge** → how many extra Pods can be created above the desired count during the update.

---

## 🔹 2. **Deployment YAML Example**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1    # At most 1 pod unavailable
      maxSurge: 1          # At most 1 extra pod above desired count
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:v1   # later update to v2
        ports:
        - containerPort: 8080
```

---

## 🔹 3. **Triggering a Rolling Update**

You can trigger by:

1. **Updating the Deployment YAML** (`kubectl apply -f deployment.yaml`) with a new image tag.
2. Or directly with `kubectl`:

   ```bash
   kubectl set image deployment/myapp myapp=myrepo/myapp:v2
   ```

---

## 🔹 4. **Monitor the Rolling Update**

* Check rollout status:

  ```bash
  kubectl rollout status deployment/myapp
  ```
* Watch Pods being replaced:

  ```bash
  kubectl get pods -l app=myapp -w
  ```

---

## 🔹 5. **Rollback if Something Breaks**

* Rollback to the previous version:

  ```bash
  kubectl rollout undo deployment/myapp
  ```
* Check history:

  ```bash
  kubectl rollout history deployment/myapp
  ```

---

## 🔹 6. **Best Practices for Production**

* Use **readinessProbes** so traffic only routes to healthy new Pods.
* Keep `maxUnavailable=0` for critical apps (no downtime).
* Keep `maxSurge=1` or more for faster rollouts.
* Use **progressDeadlineSeconds** to detect stuck updates.
* Combine with **HPA** (Horizontal Pod Autoscaler) to handle load during rollout.

---

✅ So in short:
**Rolling updates = Kubernetes Deployment default strategy**. You configure `maxUnavailable` and `maxSurge`, update the Deployment, Kubernetes gradually replaces old Pods, and you can rollback if needed.

---

👉 Do you also want me to show you the **difference between RollingUpdate, Recreate, Blue-Green, and Canary** in Kubernetes (with YAML examples)? That usually comes as a follow-up in interviews.
