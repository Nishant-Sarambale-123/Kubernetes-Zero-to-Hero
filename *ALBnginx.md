Here’s a **complete and detailed guide (with notes)** for setting up **NGINX Ingress Controller** in a Kubernetes cluster — especially in **AWS** environments like **EKS** or EC2-based clusters.

---

## 🧭 Overview

The **NGINX Ingress Controller** is one of the most popular ways to manage **external access to services** in Kubernetes using HTTP/HTTPS routes.

It works as a **reverse proxy and load balancer** inside your cluster and routes traffic to backend services based on rules defined in an **Ingress resource**.

---

## ⚙️ Key Concepts

| Concept                               | Description                                                                          |
| ------------------------------------- | ------------------------------------------------------------------------------------ |
| **Ingress Resource**                  | Defines routing rules (hostnames, paths) for HTTP/HTTPS traffic.                     |
| **Ingress Controller**                | A pod running in the cluster that implements those rules.                            |
| **Service (type=NodePort/ClusterIP)** | Exposes pods internally; NGINX routes traffic to it.                                 |
| **External Load Balancer**            | Created by AWS (if `type=LoadBalancer` service is used) to forward traffic to NGINX. |

---

## 🏗️ Architecture Diagram (Conceptual)

```
Client → AWS ELB (LoadBalancer Service) → NGINX Ingress Controller → Kubernetes Services → Pods
```

---

## 🧩 Setup Methods

You can install NGINX Ingress Controller in multiple ways:

1. **Helm (Recommended for production)**
2. **kubectl (YAML manifests from GitHub)**
3. **EKS Add-on (for managed EKS environments)**

---

## 🪜 Step-by-Step Setup (Helm method – recommended)

### 1. **Pre-requisites**

* Working Kubernetes or EKS cluster
* `kubectl` and `helm` configured
* Internet access for pods to pull images

---

### 2. **Create a Namespace**

```bash
kubectl create namespace ingress-nginx
```

---

### 3. **Add the NGINX Helm Repository**

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

---

### 4. **Install NGINX Ingress Controller**

```bash
helm install nginx-ingress ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --set controller.publishService.enabled=true
```

**Explanation:**

* `controller.publishService.enabled=true` → allows AWS ELB’s external IP to be published to Ingress resources.

---

### 5. **Verify Installation**

```bash
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

You should see a **LoadBalancer** type service like:

```
nginx-ingress-controller   LoadBalancer   <EXTERNAL-IP>   80:32588/TCP,443:32623/TCP
```

This is your **AWS ELB** created automatically by Kubernetes.

---

### 6. **Deploy a Sample Application**

**app-deployment.yaml:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-demo
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

```bash
kubectl apply -f app-deployment.yaml
```

---

### 7. **Create a Service**

**app-service.yaml:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-demo-service
spec:
  selector:
    app: nginx-demo
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

```bash
kubectl apply -f app-service.yaml
```

---

### 8. **Create an Ingress Resource**

**ingress.yaml:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-demo-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-demo-service
            port:
              number: 80
```

Apply it:

```bash
kubectl apply -f ingress.yaml
```

---

### 9. **Check Ingress Status**

```bash
kubectl get ingress
```

Output:

```
NAME                  CLASS    HOSTS              ADDRESS         PORTS   AGE
nginx-demo-ingress    nginx    myapp.example.com  a1b2c3d4.elb.amazonaws.com   80      5m
```

Copy the **ELB DNS name** (ADDRESS) and map it to your domain in Route53 or `/etc/hosts`.

---

## 🔒 HTTPS (TLS) Setup

### Step 1: Create a TLS Secret

```bash
kubectl create secret tls my-tls-secret \
  --key tls.key \
  --cert tls.crt
```

### Step 2: Update Ingress Resource

```yaml
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: my-tls-secret
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-demo-service
            port:
              number: 80
```

---

## ⚙️ Common Annotations (NGINX Ingress)

| Annotation                                           | Description                                        |
| ---------------------------------------------------- | -------------------------------------------------- |
| `nginx.ingress.kubernetes.io/rewrite-target`         | Rewrite URL path (e.g., `/` or `/app`)             |
| `nginx.ingress.kubernetes.io/ssl-redirect`           | Force redirect HTTP → HTTPS                        |
| `nginx.ingress.kubernetes.io/backend-protocol`       | Specify backend protocol (HTTP/HTTPS)              |
| `nginx.ingress.kubernetes.io/whitelist-source-range` | Restrict client IP access                          |
| `nginx.ingress.kubernetes.io/proxy-body-size`        | Increase upload limit (e.g., `100m`)               |
| `nginx.ingress.kubernetes.io/proxy-read-timeout`     | Set backend response timeout                       |
| `nginx.ingress.kubernetes.io/load-balance`           | Load balancing algorithm (round_robin, least_conn) |

---

## 🩺 Troubleshooting

| Issue                   | Fix / Check                                                    |
| ----------------------- | -------------------------------------------------------------- |
| No external IP assigned | Wait a few mins, check service type is `LoadBalancer`          |
| 404 from Ingress        | Check Ingress `pathType`, service name/port, and ingress class |
| SSL not working         | Verify TLS secret and domain                                   |
| NGINX pod crashloop     | Check logs with `kubectl logs <pod> -n ingress-nginx`          |
| ELB not created         | Verify Cloud Provider integration for AWS                      |
| Wrong DNS               | Update Route53 A-record with ELB DNS name                      |

---

## 🔍 Comparison: NGINX vs AWS ALB Ingress

| Feature                  | NGINX Ingress         | AWS ALB Ingress Controller      |
| ------------------------ | --------------------- | ------------------------------- |
| Load Balancer Type       | Internal (in-cluster) | AWS Managed ALB                 |
| SSL Termination          | Done inside NGINX     | Done by ALB                     |
| Scalability              | Scales with replicas  | AWS auto-managed                |
| Cost                     | Single ELB for NGINX  | Each Ingress may create new ALB |
| Advanced AWS Integration | ❌                     | ✅ (IAM, SG, Target Groups)      |
| Flexibility              | ✅ High (custom rules) | Medium (AWS-specific)           |

👉 **Choose NGINX** for multi-cloud portability or advanced routing
👉 **Choose AWS ALB** for tight AWS integration and managed scaling

---

## 🧠 Interview Notes

| Question                                               | Answer                                                                                                     |
| ------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------- |
| **What is an Ingress Controller?**                     | It’s a Kubernetes component that routes external HTTP(S) traffic to internal services using Ingress rules. |
| **Why use NGINX Ingress?**                             | It’s flexible, supports rewrite rules, rate limiting, custom headers, and is cloud-agnostic.               |
| **What’s the difference between Service and Ingress?** | Service exposes pods internally; Ingress exposes services externally using HTTP routing.                   |
| **How does NGINX Ingress get an external IP?**         | Through the Service of type `LoadBalancer` which provisions an AWS ELB.                                    |
| **What is rewrite-target annotation used for?**        | To modify the URL path when routing to backend services.                                                   |

---

## ✅ Summary

| Step | Description                         |
| ---- | ----------------------------------- |
| 1    | Create namespace and add Helm repo  |
| 2    | Install NGINX Ingress Controller    |
| 3    | Deploy your application and service |
| 4    | Create Ingress resource with rules  |
| 5    | Get ELB external IP and test access |
| 6    | (Optional) Add HTTPS via TLS secret |

---

Would you like me to give you a **Terraform version** of the NGINX Ingress setup (with EKS + Helm install + sample app)? It’s very useful for automating the entire deployment.
