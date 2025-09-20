Got it 👍 Let’s go step by step and build **detailed notes on IRSA (IAM Roles for Service Accounts in EKS)** so you’ll have both conceptual clarity and interview-ready points.

---

# **IAM Roles for Service Accounts (IRSA) – Detailed Notes**

## **1. What is IRSA?**

* **IRSA = IAM Roles for Service Accounts**
* Feature in **Amazon EKS (Elastic Kubernetes Service)** that allows Kubernetes **pods** to securely access AWS services **without needing node-level IAM roles or static credentials**.
* Instead of giving **all pods on a node the same permissions** (via EC2 instance profile), IRSA lets you assign **fine-grained IAM roles** to individual Kubernetes **service accounts**.
* Uses **OIDC (OpenID Connect)** to establish trust between Kubernetes and IAM.

---

## **2. Why IRSA is Needed?**

Traditionally, there were two ways to give AWS permissions to workloads inside EKS:

1. **Node IAM Role (Instance Profile)**

   * Every pod running on the node inherited the same permissions.
   * **Problem**: Over-permissive access (least-privilege violation).
2. **Static AWS Credentials (Access Key & Secret Key inside pod)**

   * Insecure and hard to rotate.

👉 **IRSA solves both problems**:

* Provides **fine-grained, pod-level IAM permissions**.
* Uses short-lived, automatically rotated credentials.
* Strongly aligned with **least privilege** security principles.

---

## **3. How IRSA Works**

1. **Enable OIDC Provider for EKS**

   * AWS EKS cluster is associated with an **OIDC identity provider**.
   * OIDC issuer URL is unique per cluster (e.g., `https://oidc.eks.ap-south-1.amazonaws.com/id/XXXXXX`).

2. **Create IAM Role with OIDC Trust Policy**

   * Role is trusted by the OIDC provider.
   * Trust policy specifies **which service account in which namespace** can assume the role.

3. **Create a Service Account in Kubernetes**

   * Annotate it with the IAM Role ARN.

4. **Pod uses the Service Account**

   * When a pod runs with that service account:

     * kubelet automatically injects a **web identity token (JWT)** into the pod.
     * AWS SDK retrieves temporary credentials from **STS (Security Token Service)** using that token.
     * Pod gets **IAM role permissions** securely.

---

## **4. Key Components**

* **OIDC Provider**: Enables federated identity between Kubernetes and AWS IAM.
* **IAM Role with Trust Policy**: Grants access to service account identities.
* **Service Account Annotation**: Binds service account to IAM role.
* **Web Identity Token File**: Mounted inside the pod (`/var/run/secrets/eks.amazonaws.com/serviceaccount/token`).
* **STS AssumeRoleWithWebIdentity**: Exchanges token for temporary AWS credentials.

---

## **5. Benefits of IRSA**

* **Least Privilege Access**: Different pods get different roles.
* **No Static Keys**: Eliminates hardcoded credentials.
* **Granular Security**: Role binding is namespace + service account specific.
* **Auditing**: CloudTrail records `sts:AssumeRoleWithWebIdentity` calls → better traceability.
* **Scalable**: Works across thousands of pods, multi-tenant EKS clusters.
* **Compatible**: Works with AWS SDKs & CLI out of the box.

---

## **6. IRSA Setup Steps**

### Step 1: Associate OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider \
  --region ap-south-1 \
  --cluster my-eks-cluster \
  --approve
```

### Step 2: Create IAM Policy

```bash
aws iam create-policy \
  --policy-name MyAppS3Policy \
  --policy-document file://s3-policy.json
```

### Step 3: Create IAM Role for Service Account

```bash
eksctl create iamserviceaccount \
  --name my-app-sa \
  --namespace default \
  --cluster my-eks-cluster \
  --attach-policy-arn arn:aws:iam::<account-id>:policy/MyAppS3Policy \
  --approve
```

### Step 4: Deploy Pod with Service Account

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app-sa
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<account-id>:role/MyAppRole
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      serviceAccountName: my-app-sa
      containers:
      - name: my-app
        image: amazonlinux
        command: ["sleep","3600"]
```

---

## **7. Real-World Use Cases**

* Pod needs to read/write from **S3**.
* Application writing logs to **CloudWatch Logs**.
* Service needing access to **DynamoDB / RDS**.
* Controllers (like **Cluster Autoscaler, ALB Ingress Controller, ExternalDNS**) that need AWS API permissions.

---

## **8. Best Practices**

* Use **least privilege policies** (restrict to specific resources).
* Rotate IAM role permissions regularly.
* Use **separate roles per app/service**.
* Monitor `sts:AssumeRoleWithWebIdentity` activity in **CloudTrail**.
* Use `eksctl` for simpler setup, but Terraform/CloudFormation for production IaC.
* Avoid using **default service account** for workloads with AWS access.

---

## **9. Interview Questions (with Answers)**

### Q1: What problem does IRSA solve?

**A:** It removes the need to share node IAM roles or hardcode AWS credentials, allowing pod-level IAM access with least privilege.

---

### Q2: How does IRSA technically work?

**A:**

* EKS uses OIDC as an identity provider.
* A pod’s service account token (JWT) is exchanged with AWS STS.
* STS returns temporary credentials bound to the IAM role.
* Pod uses those credentials to call AWS APIs securely.

---

### Q3: Can multiple service accounts share the same IAM role?

**A:** Yes, multiple service accounts can map to the same IAM role, but best practice is **one role per app/service**.

---

### Q4: What is the difference between node IAM role and IRSA?

**A:**

* **Node IAM Role** → Permissions shared by all pods on that node.
* **IRSA** → Fine-grained, per-pod permissions via service accounts.

---

### Q5: Where are the temporary credentials stored inside the pod?

**A:** In a **web identity token file** (`/var/run/secrets/eks.amazonaws.com/serviceaccount/token`). AWS SDK automatically retrieves & exchanges them for STS credentials.

---

### Q6: What happens if OIDC provider is not enabled?

**A:** IRSA will not work because the trust relationship between IAM and Kubernetes cannot be established.

---

### Q7: How do you monitor IRSA usage?

**A:** By checking `sts:AssumeRoleWithWebIdentity` events in **AWS CloudTrail**.

---

## **10. Limitations**

* Only works with **EKS (not self-managed Kubernetes on EC2)**.
* Requires enabling OIDC provider.
* Doesn’t support **AWS services not integrated with STS**.
* Misconfigured trust policies can cause pod startup failures.

---

✅ That’s the **complete detailed IRSA notes + setup + interview Q\&A**.

Do you want me to also create a **diagram-style flow (pod → service account → OIDC → IAM → STS → AWS Service)** so it’s easier to visualize in interviews?
