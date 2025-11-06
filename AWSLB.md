Here’s a **complete guide (with detailed notes)** on setting up **AWS Application Load Balancer (ALB)** for **Kubernetes Ingress**, including concepts, setup, commands, YAML examples, and troubleshooting.

---

## 🧭 Overview

When running Kubernetes on **AWS (EKS or self-managed cluster)**, you can expose services to the internet using a **Load Balancer**.
There are mainly two approaches:

| Type                                                                         | Controller  | Description                                                              |
| ---------------------------------------------------------------------------- | ----------- | ------------------------------------------------------------------------ |
| **Nginx Ingress Controller**                                                 | Nginx-based | Runs inside the cluster, uses a Service of type `LoadBalancer`           |
| **AWS ALB Ingress Controller** (now called **AWS Load Balancer Controller**) | AWS-native  | Integrates with AWS Application/Network Load Balancer (ALB/NLB) directly |

👉 For **EKS**, the **AWS Load Balancer Controller** is the **recommended** and most efficient way to use AWS native load balancers with Kubernetes Ingress.

---

## ⚙️ Key Concepts

| Component                        | Description                                                                                                              |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **AWS Load Balancer Controller** | A Kubernetes controller that manages AWS Elastic Load Balancers (ALB/NLB) for Kubernetes services and ingress resources. |
| **Ingress Resource**             | Kubernetes object that defines routing rules (HTTP/HTTPS) for external access.                                           |
| **Target Groups**                | Created automatically in AWS for pods behind your Ingress.                                                               |
| **IAM Role**                     | Grants permissions to the controller to create/manage ALBs, target groups, listeners, etc.                               |
| **Annotations**                  | Key-value pairs in the Ingress YAML that control ALB behavior (scheme, ports, SSL, health checks, etc.).                 |

---

## 🏗️ Step-by-Step Setup (AWS ALB Ingress Controller on EKS)

### 1. **Pre-requisites**

* AWS CLI configured
* `kubectl` configured for your cluster
* `eksctl` (optional, for easy IAM role setup)
* IAM permissions to create roles, policies, ALBs
* Cluster must have the **OIDC provider** enabled

---

### 2. **Enable OIDC for EKS Cluster**

```bash
eksctl utils associate-iam-oidc-provider \
  --region <region> \
  --cluster <cluster-name> \
  --approve
```

This allows EKS service accounts to assume IAM roles.

---

### 3. **Create IAM Policy for Controller**

Download AWS’s recommended IAM policy:

```bash
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
```

Create the policy:

```bash
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam-policy.json
```

---

### 4. **Create IAM Role and Service Account**

Using `eksctl`:

```bash
eksctl create iamserviceaccount \
  --cluster <cluster-name> \
  --namespace kube-system \
  --name aws-load-balancer-controller \
  --attach-policy-arn arn:aws:iam::<account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

This binds the IAM policy to the Kubernetes service account used by the controller.

---

### 5. **Install AWS Load Balancer Controller (Helm)**

Add the Helm repo:

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```

Install the controller:

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=<cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<aws-region> \
  --set vpcId=<vpc-id>
```

Verify:

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```

---

### 6. **Deploy Sample Application**

**app-deployment.yaml:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
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

### 7. **Create Service**

**app-service.yaml:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx
  type: NodePort
```

```bash
kubectl apply -f app-service.yaml
```

---

### 8. **Create Ingress Resource**

**alb-ingress.yaml:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: default
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}]'
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

Apply:

```bash
kubectl apply -f alb-ingress.yaml
```

---

### 9. **Check ALB in AWS Console**

Go to:

> EC2 → Load Balancers → You’ll see a new ALB created by the controller.

You can also describe via CLI:

```bash
kubectl get ingress
```

This will show the **ALB DNS name**.

---

## 🧩 Important Annotations

| Annotation                                           | Description                           |
| ---------------------------------------------------- | ------------------------------------- |
| `alb.ingress.kubernetes.io/scheme`                   | `internet-facing` or `internal`       |
| `alb.ingress.kubernetes.io/target-type`              | `ip` (pods) or `instance` (nodes)     |
| `alb.ingress.kubernetes.io/listen-ports`             | Configure listener ports (HTTP/HTTPS) |
| `alb.ingress.kubernetes.io/certificate-arn`          | Attach ACM SSL certificate for HTTPS  |
| `alb.ingress.kubernetes.io/healthcheck-path`         | Custom health check path              |
| `alb.ingress.kubernetes.io/load-balancer-attributes` | Set ALB attributes like idle timeout  |
| `alb.ingress.kubernetes.io/security-groups`          | Attach custom SGs                     |
| `alb.ingress.kubernetes.io/subnets`                  | Specify subnet IDs manually           |

---

## 🔒 HTTPS Setup (Optional)

If you have an ACM certificate:

```yaml
alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:region:account-id:certificate/uuid
```

Make sure to update DNS (Route53) to point your domain → ALB DNS name.

---

## 🩺 Troubleshooting Tips

| Issue                       | Check                                                                                  |
| --------------------------- | -------------------------------------------------------------------------------------- |
| ALB not created             | Controller logs: `kubectl logs -n kube-system deployment/aws-load-balancer-controller` |
| Target group unhealthy      | Check target group health check path & security group rules                            |
| 403 Forbidden               | Check Ingress rules and backend service ports                                          |
| DNS not resolving           | Check Route53 and domain mapping                                                       |
| Pod IPs not in target group | Ensure target type is `ip`, not `instance`                                             |

---

## 🧠 Interview & Concept Notes

| Question                                                 | Answer                                                                                       |
| -------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| **What is AWS Load Balancer Controller?**                | It manages AWS ALBs and NLBs for Kubernetes Ingress and Service objects.                     |
| **Difference between Nginx Ingress and ALB Controller?** | Nginx is internal; ALB integrates natively with AWS for managed scaling and SSL termination. |
| **Why target-type = ip?**                                | So pods can be directly registered as targets instead of node-level IPs.                     |
| **Can you have multiple Ingresses?**                     | Yes, each Ingress can create its own ALB or share one using annotations.                     |
| **How to restrict ALB to internal use?**                 | Set `alb.ingress.kubernetes.io/scheme: internal`.                                            |

---

## ✅ Summary

| Step | Description                     |
| ---- | ------------------------------- |
| 1    | Enable OIDC                     |
| 2    | Create IAM policy               |
| 3    | Create service account          |
| 4    | Install Helm chart              |
| 5    | Deploy app + service            |
| 6    | Create Ingress with annotations |
| 7    | Verify ALB in AWS console       |

---

Would you like me to include a **Terraform version** of this ALB Ingress setup (for automated EKS + ALB deployment)? It’s often useful for DevOps interview or project setups.
