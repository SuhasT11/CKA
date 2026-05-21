# Kubernetes Quick Commands: DaemonSet, Job, and CronJob

This README provides the most commonly used commands for managing DaemonSets, Jobs, and CronJobs in Kubernetes.

---

## 🌀 DaemonSet Commands

- **Get DaemonSets**  
  ```bash
  kubectl get daemonset
  ```

- **Describe DaemonSet**  
  ```bash
  kubectl describe daemonset node-logger
  ```

---

## ⚙️ Job Commands

- **Get Jobs**  
  ```bash
  kubectl get job
  ```

- **Describe Job**  
  ```bash
  kubectl describe job demo-job
  ```

---

## ⏰ CronJob Commands

- **Get CronJobs**  
  ```bash
  kubectl get cronjob
  ```

- **Describe CronJob**  
  ```bash
  kubectl describe cronjob demo-job
  ```

---

## Notes
- Use `kubectl logs <pod-name>` to view logs from pods created by Jobs or CronJobs.
- Use `kubectl delete <resource> <name>` to remove resources (e.g., `kubectl delete job demo-job`).
- Always check the namespace with `-n <namespace>` if your resources are not in `default`.
