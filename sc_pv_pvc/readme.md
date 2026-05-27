# Kubernetes Storage Quick Reference

This guide covers the essential commands for working with **StorageClass (SC)**, **PersistentVolume (PV)**, and **PersistentVolumeClaim (PVC)**.

---

## 📦 StorageClass

- **List all StorageClasses**
  ```bash
  kubectl get storageclass
  ```

- **Describe a StorageClass**
  ```bash
  kubectl describe storageclass fast
  ```

- **Set a StorageClass as default**
  ```bash
  kubectl patch storageclass fast \
    -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
  ```

---

## 💾 PersistentVolume (PV)

- **List all PVs**
  ```bash
  kubectl get pv
  ```

- **Describe a PV**
  ```bash
  kubectl describe pv my-pv
  ```

- **Delete a PV**
  ```bash
  kubectl delete pv my-pv
  ```

- **Patch a Released PV to clear old claimRef (reuse PV)**
  ```bash
  kubectl patch pv my-pv -p '{"spec":{"claimRef":null}}'
  ```

---

## 📑 PersistentVolumeClaim (PVC)

- **List all PVCs**
  ```bash
  kubectl get pvc
  ```

- **Describe a PVC**
  ```bash
  kubectl describe pvc my-pvc
  ```

- **Delete a PVC**
  ```bash
  kubectl delete pvc my-pvc
  ```

---

## 🛠️ Pod with PVC

- **Apply Pod manifest**
  ```bash
  kubectl apply -f pod.yaml
  ```

- **Check Pod status**
  ```bash
  kubectl get pods
  ```

- **Describe Pod (to see volume mounts)**
  ```bash
  kubectl describe pod volume-demo
  ```

---
