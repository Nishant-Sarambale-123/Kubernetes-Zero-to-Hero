Sure, here’s a **quick command reference** for upgrading **EKS cluster, nodes, and add-ons** using AWS CLI and `eksctl`.

---

## **1️⃣ Upgrade EKS Cluster Kubernetes Version**

```bash
# Using AWS CLI
aws eks update-cluster-version --name <cluster-name> --kubernetes-version <new-version>

# Using eksctl
eksctl upgrade cluster --name <cluster-name> --approve
```

---

## **2️⃣ Upgrade Managed Node Group**

```bash
# Using AWS CLI
aws eks update-nodegroup-version --cluster-name <cluster-name> --nodegroup-name <nodegroup-name> --kubernetes-version <new-version>

# Using eksctl
eksctl upgrade nodegroup --cluster <cluster-name> --name <nodegroup-name> --approve
```

---

## **3️⃣ Upgrade Add-ons (Default + Extra)**

### **a) CoreDNS**

```bash
aws eks update-addon --cluster-name <cluster-name> --addon-name coredns --resolve-conflicts OVERWRITE
```

### **b) kube-proxy**

```bash
aws eks update-addon --cluster-name <cluster-name> --addon-name kube-proxy --resolve-conflicts OVERWRITE
```

### **c) VPC CNI**

```bash
aws eks update-addon --cluster-name <cluster-name> --addon-name aws-vpc-cni --resolve-conflicts OVERWRITE
```

### **d) EBS CSI Driver (Extra Add-on)**

```bash
aws eks update-addon --cluster-name <cluster-name> --addon-name aws-ebs-csi-driver --resolve-conflicts OVERWRITE
```

---

✅ **Notes:**

* Always check **current versions** before upgrading:

```bash
aws eks describe-cluster --name <cluster-name>
aws eks describe-nodegroup --cluster-name <cluster-name> --nodegroup-name <nodegroup-name>
aws eks describe-addon-versions --addon-name coredns
```

* Use `--approve` with `eksctl` to skip confirmation prompts.

---

I can also make a **one-line cheat sheet of all upgrade commands for EKS cluster, nodes, and add-ons**, ready for interview explanation.

Do you want me to make that?
