Here's a comprehensive **Kubernetes Ingress Interview Q\&A Guide**, including **scenario-based questions**, **answers**, and **official documentation links**.

---

# 🌐 Kubernetes Ingress – Interview Q\&A (With Scenarios)

---

## ✅ Basic Questions and Answers

---

### ❓ Q1. What is Ingress in Kubernetes?

**Answer:**

Ingress is a **Kubernetes API object** that manages **external access to services in a cluster**, typically HTTP or HTTPS. It allows fine-grained control over routing traffic based on hostnames or paths.

📘 [Official Docs – Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)

---

### ❓ Q2. How does Ingress differ from a LoadBalancer or NodePort service?

**Answer:**

| Feature         | NodePort                  | LoadBalancer           | Ingress                          |
| --------------- | ------------------------- | ---------------------- | -------------------------------- |
| Access Type     | Opens a port on each Node | Provisions external LB | Routes via single IP using rules |
| L7 Routing      | ❌                         | ❌                      | ✅ (path/host-based)              |
| TLS Termination | ❌                         | ✅ (some support)       | ✅                                |
| Central Mgmt    | ❌                         | ❌                      | ✅ (centralized rules)            |

📘 [Services vs Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/#what-is-ingress)

---

### ❓ Q3. What is an Ingress Controller?

**Answer:**

An Ingress Controller is a **daemon that implements the Ingress API**, typically running as a pod within the cluster. It listens for Ingress resource changes and updates underlying proxy configs (e.g., NGINX, HAProxy).

Examples:

* **NGINX Ingress Controller**
* **Traefik**
* **HAProxy**
* **AWS ALB Ingress Controller**

📘 [Ingress Controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)

---

### ❓ Q4. Can Ingress handle SSL/TLS termination?

**Answer:**

Yes. Ingress supports **TLS termination** by referencing a Kubernetes **Secret** containing the TLS private key and certificate.

```yaml
tls:
- hosts:
  - example.com
  secretName: example-tls
```

📘 [TLS in Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/#tls)

---

### ❓ Q5. How does path-based routing work in Ingress?

**Answer:**

Path-based routing enables requests like `example.com/api` and `example.com/web` to be routed to different backends based on the path.

```yaml
rules:
- host: example.com
  http:
    paths:
    - path: /api
      pathType: Prefix
      backend:
        service:
          name: api-service
          port:
            number: 80
```

---

### ❓ Q6. What is `pathType` in Ingress?

**Answer:**

`pathType` defines how the path is matched:

* `Prefix`: Matches if the request path starts with the specified path.
* `Exact`: Matches only if the path is exactly the same.
* `ImplementationSpecific`: Behavior depends on the controller.

📘 [Ingress pathType](https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types)

---

### ❓ Q7. How do you secure your Ingress?

**Answer:**

1. Use **TLS termination**.
2. Apply **authentication** (OAuth, basic-auth via annotations).
3. Enable **WAF** (Web Application Firewall).
4. Rate-limiting and IP whitelisting via Ingress Controller config.

---

### ❓ Q8. Can you use Ingress with services on different namespaces?

**Answer:**

Ingress can route to services in **other namespaces**, but it requires explicitly defining **namespace in the backend** if the Ingress itself is in a different one.

---

## 🧠 Scenario-Based Interview Questions

---

### 🔄 **Scenario 1:**

**Q:** You have two services:

* `api-service` serving at `/api`
* `frontend-service` serving at `/`
  How would you route both through a single Ingress?

**Answer:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

📘 [Ingress Example](https://kubernetes.io/docs/concepts/services-networking/ingress/#examples)

---

### 🔐 **Scenario 2:**

**Q:** Your application needs HTTPS support with a Let's Encrypt certificate. How do you configure Ingress?

**Answer:**

Use **cert-manager** to automate Let's Encrypt certs.

Ingress annotation for cert-manager:

```yaml
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls
```

📘 [cert-manager Ingress TLS](https://cert-manager.io/docs/usage/ingress/)

---

### 🔄 **Scenario 3:**

**Q:** You deployed an Ingress, but traffic is not reaching your application. What could be wrong?

**Answer:**

Checklist:

1. **Ingress Controller** is not installed or running.
2. **Service backend** ports are incorrect.
3. **DNS** doesn't resolve to the correct external IP.
4. **Firewall/Security group** blocking the Ingress IP.
5. **PathType** mismatch (`Prefix` vs `Exact`).
6. **Missing or incorrect annotations** for controller.

---

### 🌐 **Scenario 4:**

**Q:** You want `/admin` to be accessible **only from specific IPs**, but the rest of the app should be public.

**Answer:**

Use NGINX Ingress annotations:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/configuration-snippet: |
      if ($request_uri ~* "/admin") {
        allow 192.168.1.0/24;
        deny all;
      }
```

📘 [NGINX Ingress Annotations](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/)

---

### 🧪 **Scenario 5:**

**Q:** Your `/api` routes work, but `/api/v1` is giving a 404. What might be the cause?

**Answer:**

* If `pathType` is set to `Exact`, `/api` will not match `/api/v1`.
* Set `pathType: Prefix` for broader matching.

Fix:

```yaml
path: /api
pathType: Prefix
```

---

## 🔁 Advanced Interview Questions

---

### ❓ Q9. Can you apply canary deployments using Ingress?

**Answer:**

Yes, using annotations and weighted backends with tools like **NGINX Ingress + flagger**, you can direct a percentage of traffic to a canary deployment.

📘 [NGINX Canary Deployments](https://kubernetes.github.io/ingress-nginx/examples/affinity/canary/)

---

### ❓ Q10. How do multiple Ingress resources work under the same host?

**Answer:**

* Multiple Ingress resources can define rules for the **same host**, but care must be taken to **avoid overlapping paths**.
* Conflicts are resolved by **Ingress Controller behavior**, often leading to unpredictable results unless managed properly.

---

### ❓ Q11. What is the difference between Internal and External Ingress?

**Answer:**

* **External Ingress**: Exposed to public internet (e.g., internet-facing ALB/ELB).
* **Internal Ingress**: Private access within the VPC or cluster (e.g., internal ALB, internal IPs only).

---

## 🎯 Final Notes

* Always check if an Ingress Controller is deployed – Ingress won't work without it.
* Ingress is mostly Layer 7 (HTTP/HTTPS) – for TCP/UDP services, consider a Service of type LoadBalancer or use annotations.
* For production: Use HTTPS, rate-limiting, authentication.

---

Let me know if you'd like a **YAML lab setup** for practice or visual diagrams for Ingress flow.
