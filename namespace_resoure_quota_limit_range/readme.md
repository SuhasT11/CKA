# Kubernetes: Namespaces, ResourceQuota, and LimitRange

This guide provides a hands-on walkthrough of Kubernetes resource isolation and management using **Namespaces**, **ResourceQuota**, and **LimitRange**.

---

## 🔹 Namespaces
Namespaces logically isolate resources for teams or projects.

### Create a namespace
```bash
kubectl create namespace dev-team
```

### List all namespaces
```bash
kubectl get namespaces
```

### Switch context to a namespace
```bash
kubectl config set-context --current --namespace=dev-team
```

---

## 🔹 API Resources
### Namespace-scoped resources
```bash
kubectl api-resources --namespaced=true
```

### Cluster-scoped resources
```bash
kubectl api-resources --namespaced=false
```

---

## 🔹 ResourceQuota
ResourceQuota ensures fair usage by capping resource consumption.

### Apply a ResourceQuota
```bash
kubectl apply -f resourcequota.yaml
```

### List ResourceQuotas in a namespace
```bash
kubectl get resourcequota -n dev-team
```

### Describe a ResourceQuota
```bash
kubectl describe resourcequota dev-quota -n dev-team
```

### Or
```bash
kubectl describe quota dev-quota -n dev-team
```

---

## 🔹 LimitRange
LimitRange enforces default, minimum, and maximum resource requests/limits per container.

### Apply a LimitRange
```bash
kubectl apply -f limitrange.yaml
```

### Describe a LimitRange
```bash
kubectl describe limitrange dev-limits -n dev-team
```

---

## 🔹 Pods with Requests & Limits
Deploy a Pod that respects ResourceQuota and LimitRange:
```bash
kubectl apply -f nginx-pod.yaml
```

Check Pod resource allocation:
```bash
kubectl describe pod nginx-pod -n dev-team
```

---

## 🔹 Useful Commands Recap
- `kubectl get cm -n kube-public` → Cluster info ConfigMap  
- `kubectl api-resources --namespaced=true` → Namespace-scoped resources  
- `kubectl api-resources --namespaced=false` → Cluster-scoped resources  
- `kubectl get resourcequota -n dev-team` → List quotas  
- `kubectl describe resourcequota dev-quota -n dev-team` → Quota details  
- `kubectl describe limitrange dev-limits -n dev-team` → LimitRange details  
- `kubectl get pods -n dev-team` → List Pods in namespace  
- `kubectl describe pod <pod-name> -n dev-team` → Pod resource usage  

---
