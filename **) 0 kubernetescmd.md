Sure! Here’s a **complete, comprehensive Kubernetes (`kubectl`) command list**, grouped by category for **easy reference and interviews**. This covers almost everything you need for **daily operations, debugging, and exam/interview prep**.

---

# **🟢 Kubernetes Commands Cheat Sheet (Complete)**

---

## **1️⃣ Cluster & Context**

```bash
kubectl version                        # Show client & server version
kubectl cluster-info                   # Show cluster info
kubectl get nodes                      # List all nodes
kubectl describe node <node>           # Detailed node info
kubectl top nodes                       # Node CPU/memory usage
kubectl config get-contexts            # List contexts
kubectl config current-context         # Show current context
kubectl config use-context <name>      # Switch context
kubectl get componentstatuses          # Show cluster component health
```

---

## **2️⃣ Namespace**

```bash
kubectl get ns                         # List namespaces
kubectl create ns <name>               # Create namespace
kubectl delete ns <name>               # Delete namespace
kubectl config set-context --current --namespace=<name>  # Set default ns
```

---

## **3️⃣ Pods**

```bash
kubectl get pods                        # List pods
kubectl get pods -n <namespace>         # List pods in a namespace
kubectl describe pod <pod>              # Pod details including events
kubectl get pod <pod> -o yaml           # Full pod YAML
kubectl logs <pod>                      # Logs of container
kubectl logs -f <pod>                   # Follow logs in real-time
kubectl logs <pod> -c <container>       # Logs for specific container
kubectl logs --previous <pod>           # Logs of previous container
kubectl exec -it <pod> -- /bin/bash     # Exec into pod
kubectl delete pod <pod>                # Delete pod
kubectl get pods -o wide                 # Show pod IPs, nodes, status
kubectl get pods --all-namespaces       # Pods in all namespaces
kubectl attach -it <pod>                # Attach to running pod
```

---

## **4️⃣ Deployment**

```bash
kubectl get deploy                       # List deployments
kubectl describe deploy <name>           # Detailed info
kubectl apply -f deploy.yaml             # Create/update deployment
kubectl delete deploy <name>             # Delete deployment
kubectl scale deploy <name> --replicas=N # Scale deployment
kubectl rollout status deploy <name>     # Check rollout status
kubectl rollout history deploy <name>    # Deployment rollout history
kubectl rollout undo deploy <name>       # Rollback deployment
kubectl set image deploy/<name> <container>=<image>:<tag>  # Update image
```

---

## **5️⃣ Services**

```bash
kubectl get svc / services               # List services
kubectl describe svc <name>              # Service details
kubectl apply -f svc.yaml                # Create/update service
kubectl delete svc <name>                # Delete service
kubectl get svc -o wide                  # Show ClusterIP, NodePort
kubectl port-forward <pod> <local>:<pod-port>  # Forward port
```

---

## **6️⃣ ConfigMap & Secret**

```bash
kubectl get cm                            # List ConfigMaps
kubectl describe cm <name>                # ConfigMap details
kubectl create cm <name> --from-literal=key=value
kubectl get secret                         # List Secrets
kubectl describe secret <name>             # Secret details
kubectl create secret generic <name> --from-literal=password=123
kubectl create secret generic <name> --from-file=<file>
kubectl get cm -o yaml                     # Export ConfigMap
kubectl get secret -o yaml                 # Export Secret
```

---

## **7️⃣ Nodes & Scheduling**

```bash
kubectl cordon <node>                     # Mark node unschedulable
kubectl uncordon <node>                   # Make node schedulable
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data  # Evict pods
kubectl top nodes                          # Node metrics
kubectl describe node <node>               # Node details
kubectl get events --all-namespaces        # Node/pod events
```

---

## **8️⃣ Labels & Annotations**

```bash
kubectl get pods --show-labels             # Show labels
kubectl label pod <pod> key=value          # Add/update label
kubectl annotate pod <pod> key=value       # Add annotation
kubectl get pods -l key=value              # Filter by label
kubectl get all --show-labels              # All resources with labels
```

---

## **9️⃣ Jobs & CronJobs**

```bash
kubectl get jobs
kubectl describe job <job-name>
kubectl delete job <job-name>
kubectl get cronjobs
kubectl describe cronjob <cronjob-name>
kubectl delete cronjob <cronjob-name>
kubectl create -f job.yaml
kubectl create -f cronjob.yaml
```

---

## **🔟 Ingress**

```bash
kubectl get ingress
kubectl describe ingress <name>
kubectl apply -f ingress.yaml
kubectl delete ingress <name>
```

---

## **1️⃣1️⃣ Persistent Volumes & PVC**

```bash
kubectl get pv
kubectl describe pv <name>
kubectl get pvc
kubectl describe pvc <name>
kubectl delete pvc <name>
kubectl apply -f pvc.yaml
```

---

## **1️⃣2️⃣ Logs & Debugging**

```bash
kubectl logs <pod>                         # Container logs
kubectl logs -f <pod>                      # Follow logs
kubectl logs <pod> -c <container>          # Multi-container pod logs
kubectl logs --previous <pod>              # Previous container logs
kubectl exec -it <pod> -- /bin/bash        # Shell into pod
kubectl attach -it <pod>                   # Attach to pod
kubectl describe pod <pod>                 # Events & status
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

## **1️⃣3️⃣ Miscellaneous / Utility**

```bash
kubectl get all                              # All resources in namespace
kubectl get all --all-namespaces             # All resources cluster-wide
kubectl apply -f <file.yaml>                 # Create/update resources
kubectl delete -f <file.yaml>                # Delete resources
kubectl explain <resource>                   # Explain fields
kubectl edit <resource> <name>               # Edit live resource
kubectl rollout status <deployment>          # Deployment rollout status
kubectl rollout undo <deployment>            # Rollback deployment
kubectl diff -f <file.yaml>                  # Show changes before applying
kubectl wait --for=condition=ready pod/<pod> # Wait until ready
```

---

✅ This is **basically all commonly used Kubernetes commands**.

---

If you want, I can also make a **super-condensed “one-page hidden flashcard version”**, so you can **hide commands and revise quickly**, perfect for **interview prep**.

Do you want me to make that?
