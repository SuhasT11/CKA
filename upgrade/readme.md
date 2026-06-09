# 📘 Etcd Backup, Restore, and Kubernetes Upgrade Guide

## 🔧 Prerequisites
- Ubuntu/Debian system with Kubernetes cluster installed.
- Access to `/etc/kubernetes/pki/etcd` certificates.
- Sudo privileges.

---

## 📦 Install Etcd Client
```bash
sudo apt install etcd-client
```

---

## 🩺 Check Etcd Health
```bash
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt endpoint health
```

---

## 💾 Backup Etcd Data
```bash
ETCDCTL_API=3 etcdctl snapshot save /var/lib/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt
```

### Verify Snapshot
```bash
ETCDCTL_API=3 etcdctl snapshot status /var/lib/etcd-backup.db
```

---

## 🔄 Restore Etcd from Snapshot

### Step 1: Stop kube-apiserver
```bash
sudo mv /etc/kubernetes/manifests/* /tmp/
```

### Step 2: Move Existing Etcd Data
```bash
sudo mv /var/lib/etcd /var/lib/etcd-old
```

### Step 3: Restore Snapshot
```bash
ETCDCTL_API=3 etcdctl snapshot restore /var/lib/etcd-backup.db \
  --data-dir=/var/lib/etcd
```

### Step 4: Verify Data
```bash
ls -l /var/lib/etcd
```

### Step 5: Update Etcd Config (if needed)
Edit `/etc/kubernetes/manifests/etcd.yaml`:
```yaml
command:
  - etcd
  - --data-dir=/var/lib/etcd
```

### Step 6: Restart Components
```bash
sudo mv /tmp/*.yaml /etc/kubernetes/manifests/
```

### Step 7: Confirm Health
```bash
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt endpoint health
```

### Step 8: Check Cluster
```bash
kubectl get nodes
kubectl get pods -A
```

---

## 🚀 Kubernetes Upgrade

### Master Node

1. Update Kubernetes repo:
   ```bash
   nano /etc/apt/sources.list.d/kubernetes.list
   deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /
   sudo apt update
   ```

2. Check available kubeadm versions:
   ```bash
   sudo apt-cache madison kubeadm
   ```


3. Upgrade `kubeadm`:
   ```bash
   sudo apt-mark unhold kubeadm && \
   sudo apt-get install -y kubeadm='1.36.0-1.1' && \
   sudo apt-mark hold kubeadm
   ```

4. Plan & apply upgrade:
   ```bash
   sudo kubeadm upgrade plan
   sudo kubeadm upgrade apply v1.36.0
   ```

5. Drain node:
   ```bash
   kubectl drain master-node --ignore-daemonsets
   ```

6. Upgrade `kubelet` & `kubectl`:
   ```bash
   sudo apt-mark unhold kubelet kubectl && \
   sudo apt-get install -y kubelet='1.36.0-1.1' kubectl='1.36.0-1.1' && \
   sudo apt-mark hold kubelet kubectl
   ```

7. Restart services:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart kubelet
   ```

8. Uncordon node:
   ```bash
   kubectl uncordon master-node
   ```

---

### Worker Node

1. Update repo:
   ```bash
   nano /etc/apt/sources.list.d/kubernetes.list
   deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /
   sudo apt update
   ```

2. Check available kubeadm versions:
   ```bash
   sudo apt-cache madison kubeadm
   ```

3. Upgrade `kubeadm`:
   ```bash
   sudo apt-mark unhold kubeadm && \
   sudo apt-get install -y kubeadm='1.36.0-1.1' && \
   sudo apt-mark hold kubeadm
   ```

4. Upgrade node:
   ```bash
   sudo kubeadm upgrade node
   ```

5. Drain node:
   ```bash
   kubectl drain worker-node-1 --ignore-daemonsets
   ```

6. Upgrade `kubelet` & `kubectl`:
   ```bash
   sudo apt-mark unhold kubelet kubectl && \
   sudo apt-get install -y kubelet='1.36.0-1.1' kubectl='1.36.0-1.1' && \
   sudo apt-mark hold kubelet kubectl
   ```

7. Restart services:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart kubelet
   ```

8. Uncordon node:
   ```bash
   kubectl uncordon worker-node-1
   ```

---

## ✅ Verification
- Run `kubectl get nodes` to confirm all nodes are Ready.
- Run `kubectl get pods -A` to ensure workloads are healthy.
