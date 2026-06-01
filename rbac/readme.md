
# 🛡️ Kubernetes RBAC Demo Guide

## 1. Create a User with Certificates
```bash
# Generate private key
openssl genrsa -out alice.key 2048

# Create CSR with CN=username and O=group
openssl req -new -key alice.key -out alice.csr -subj "/CN=alice/O=dev"

# Sign CSR with cluster CA
sudo openssl x509 -req -in alice.csr \
  -CA /etc/kubernetes/pki/ca.crt \
  -CAkey /etc/kubernetes/pki/ca.key \
  -CAcreateserial \
  -out alice.crt -days 365
```

---

## 2. Encode Certificates for Kubeconfig
```bash
# Encode CA cert
cat /etc/kubernetes/pki/ca.crt | base64 | tr -d '\n'

# Encode alice cert
cat alice.crt | base64 | tr -d '\n'

# Encode alice key
cat alice.key | base64 | tr -d '\n'
```

---

## 3. Create Kubeconfig for Alice
```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: <BASE64_CA_CERT>
    server: https://10.0.1.235:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: alice
  name: alice@kubernetes
current-context: alice@kubernetes
kind: Config
users:
- name: alice
  user:
    client-certificate-data: <BASE64_ALICE_CERT>
    client-key-data: <BASE64_ALICE_KEY>
```

Test:
```bash
kubectl get no --kubeconfig=/root/rbac/user/alice_config
```

---

## 4. RBAC Verbs
- **get** → read a single resource (`kubectl get pod mypod`)  
- **list** → read all resources of a type (`kubectl get pods`)  
- **watch** → stream changes (`kubectl get pods -w`)

---

## 5. Verify Role Bindings
```bash
kubectl get role -n dev
kubectl get rolebinding -n dev
kubectl get roles,rolebindings -n dev
kubectl get clusterrole node-reader -o yaml
kubectl get clusterrolebinding bind-node-reader
```

Check permissions:
```bash
kubectl auth can-i list pods --as alice -n dev
kubectl auth can-i delete pods --as alice -n dev
kubectl auth can-i list pods --as alice -n default
```

---

## 6. ServiceAccount Demo
```bash
# Create ServiceAccount
kubectl create serviceaccount my-app-sa -n prod
kubectl get sa -n prod

# Create Role
kubectl create role app-role \
  --verb=get --verb=list --verb=watch \
  --resource=pods -n prod

# Bind Role to ServiceAccount
kubectl create rolebinding app-role-binding \
  --role=app-role \
  --serviceaccount=prod:my-app-sa \
  -n prod
```

---

## 7. Pod Using ServiceAccount
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  namespace: prod
spec:
  serviceAccountName: my-app-sa
  automountServiceAccountToken: true
  containers:
  - name: app
    image: curlimages/curl
    command: ["sleep", "3600"]
```

---

## 8. Inspect Token Inside Pod
```bash
kubectl exec -it my-app -n prod -- sh

cat /var/run/secrets/kubernetes.io/serviceaccount/token
```

Decode JWT payload:
```bash
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)

curl -s --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
  -H "Authorization: Bearer $TOKEN" \
  https://kubernetes.default.svc/api/v1/namespaces/prod/pods
```
