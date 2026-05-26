# Kubernetes ConfigMaps, Secrets, and Environment Variables

## 📘 Concepts
- **ConfigMap** → Non-sensitive configuration data.  
- **Secret** → Sensitive configuration data (passwords, tokens).  
- **Environment Variables** → Inject values into containers at runtime.  

---

## ⚙️ Common Commands

### 🔹 Create ConfigMap (Imperative)
```bash
kubectl create configmap app-config-imperative \
  --from-literal=DB_HOST=db.company.com \
  --from-literal=LOG_LEVEL=info
```

### 🔹 Verify ConfigMap Injection
```bash
kubectl exec -it <pod-name> -- env | grep DB_HOST
kubectl exec -it <pod-name> -- env | grep LOG_LEVEL
```

### 🔹 Create Secret (Imperative)
```bash
kubectl create secret generic db-secret-imperative \
  --from-literal=DB_PASSWORD=supersecret \
  --from-literal=DB_USER=admin
```

---

## ➕ Additional Useful Commands

### 🔹 View ConfigMap Details
```bash
kubectl get configmap app-config-imperative -o yaml
```

### 🔹 View Secret Details (encoded)
```bash
kubectl get secret db-secret-imperative -o yaml
```

### 🔹 Decode Secret Value
```bash
kubectl get secret db-secret-imperative \
  -o jsonpath="{.data.DB_PASSWORD}" | base64 --decode
```

### 🔹 List All Environment Variables in a Pod
```bash
kubectl exec -it <pod-name> -- env
```

### 🔹 Delete ConfigMap or Secret
```bash
kubectl delete configmap app-config-imperative
kubectl delete secret db-secret-imperative
```

### 🔹 Apply Declarative Configs
```bash
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml
```

---

## 🔑 Key Takeaways
- ConfigMaps = non-sensitive configs.  
- Secrets = sensitive configs (base64 encoded in YAML).  
- Inject values into pods via `env` (single key) or `envFrom` (all keys).  
- Verify values inside pods using `kubectl exec`.  
- Use `jsonpath` + `base64 --decode` to read Secret values safely.  

---
