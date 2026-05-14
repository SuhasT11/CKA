# 🚀 Kubernetes Master Node Setup Guide

This guide walks you through setting up a Kubernetes **master node** using `kubeadm` and `containerd`.

***

## 🖥️ Host Configuration

```bash
sudo hostnamectl set-hostname master-node
```

Edit hosts file:

```bash
sudo nano /etc/hosts
```

Add:

    127.0.0.1 master-node

Apply changes:

```bash
exec bash
```

***

## ⚙️ System Preparation

### Step 1: Update Packages

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg lsb-release
```

***

## 📦 Install and Configure containerd

### Step 2: Install containerd

```bash
sudo apt-get install -y containerd
```

### Step 3: Generate Default Configuration

```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

### Step 4: Enable Systemd Cgroup Driver

```bash
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
```

### Step 5: Restart & Enable containerd

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
sudo systemctl status containerd
```

***

## 🧠 Kernel & Networking Setup

### Step 6: Load Required Kernel Modules

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

Persist modules:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```

### Step 7: Configure sysctl for Kubernetes Networking

```bash
cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```

Apply settings:

```bash
sudo sysctl --system
```

***

## ✅ Verify containerd

```bash
containerd --version
```

***

# ☸️ Kubernetes Setup

### Step 1: Install Required Packages

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

***

### Step 2: Add Kubernetes Signing Key

```bash
sudo mkdir -p -m 755 /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key \
| sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

***

### Step 3: Add Kubernetes Repository

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' \
| sudo tee /etc/apt/sources.list.d/kubernetes.list
```

***

### Step 4: Install Kubernetes Components

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

***

### Step 5: Enable kubelet

```bash
sudo systemctl enable --now kubelet
```

***

## 🧩 Initialize Kubernetes Cluster

### Step 6: Initialize Master Node

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16
```

***

## 👤 Configure kubectl for User

### Step 7: Set Up kubeconfig

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
