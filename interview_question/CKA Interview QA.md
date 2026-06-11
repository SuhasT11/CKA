# CKA Interview Questions & Answers

-----

## Table of Contents

1. [Kubernetes Deep Architecture & Components](#1-kubernetes-deep-architecture--components)
1. [Kubernetes Installation using kubeadm](#2-kubernetes-installation-using-kubeadm)
1. [Static Pods & Component Configuration](#3-static-pods--component-configuration)
1. [Namespaces & Resource Management](#4-namespaces--resource-management)
1. [Labels, Selectors & Annotations](#5-labels-selectors--annotations)
1. [Pod Design](#6-pod-design)
1. [Deployments & ReplicaSets](#7-deployments--replicasets)
1. [DaemonSets, Jobs & CronJobs](#8-daemonsets-jobs--cronjobs)
1. [Services & DNS](#9-services--dns)
1. [Ingress Controllers & Networking](#10-ingress-controllers--networking)
1. [ConfigMaps & Secrets](#11-configmaps--secrets)
1. [Resource Management & Probes](#12-resource-management--probes)
1. [Volumes & Persistent Storage](#13-volumes--persistent-storage)
1. [Storage Classes & StatefulSets](#14-storage-classes--statefulsets)
1. [Kubernetes Security (TLS, RBAC)](#15-kubernetes-security-tls-rbac)
1. [Authentication & Authorization](#16-authentication--authorization)
1. [Logging & Monitoring](#17-logging--monitoring)
1. [Troubleshooting Cluster Issues](#18-troubleshooting-cluster-issues)
1. [Upgrade Cluster using kubeadm](#19-upgrade-cluster-using-kubeadm)
1. [JSONPath & kubectl Mastery](#20-jsonpath--kubectl-mastery)

-----

## 1. Kubernetes Deep Architecture & Components

**Q1. What are the main components of the Kubernetes control plane?**

> The control plane consists of the API Server (central gateway for all REST operations), Scheduler (assigns pods to nodes), Controller Manager (runs reconciliation loops), and etcd (distributed key-value store for all cluster state).

**Q2. What is the role of the kube-apiserver?**

> It is the front-end of the control plane. All internal and external communication goes through it. It validates and processes REST requests, authenticates/authorizes them, and persists state to etcd.

**Q3. What does the kube-scheduler do?**

> It watches for newly created pods with no assigned node and selects the best node based on resource requirements, affinity/anti-affinity rules, taints/tolerations, and other constraints.

**Q4. What is the Controller Manager and what controllers does it run?**

> The kube-controller-manager runs multiple controllers in a single process: Node Controller, Replication Controller, Endpoints Controller, Service Account & Token Controllers, and more. Each controller watches cluster state and reconciles actual vs desired state.

**Q5. What is etcd and why is it critical?**

> etcd is a consistent, distributed key-value store used as Kubernetes’ backing store for all cluster data. If etcd is lost without a backup, the entire cluster state is lost.

**Q6. What is the role of kubelet?**

> The kubelet runs on every worker node. It registers the node with the API server, watches for PodSpecs, ensures containers described in those specs are running and healthy, and reports node/pod status back.

**Q7. What does kube-proxy do?**

> kube-proxy maintains network rules on each node to allow network communication to pods. It implements the Kubernetes Service concept using iptables, IPVS, or userspace mode.

**Q8. What is the difference between the control plane and worker node?**

> The control plane manages the cluster (scheduling, API, state storage). Worker nodes run the actual application workloads via kubelet and container runtime.

**Q9. What container runtimes does Kubernetes support?**

> Kubernetes supports any CRI-compliant runtime: containerd, CRI-O, and Docker (via cri-dockerd shim). containerd is the most widely used today.

**Q10. What is the CRI (Container Runtime Interface)?**

> CRI is a plugin interface that allows kubelet to use different container runtimes without recompilation. It defines gRPC APIs (ImageService and RuntimeService).

**Q11. How does the Kubernetes API server handle requests?**

> Requests go through: Authentication → Authorization (RBAC/ABAC) → Admission Controllers → Validation → Persistence to etcd. The response is returned after the object is stored.

**Q12. What happens when you run `kubectl apply -f pod.yaml`?**

> kubectl serializes the manifest and sends a REST request to the API server. The API server authenticates, authorizes, runs admission controllers, validates the object, persists it to etcd, and the scheduler then picks a node for the pod.

**Q13. What is the purpose of the cloud-controller-manager?**

> It integrates with cloud provider APIs to manage cloud-specific resources like load balancers, storage volumes, and node lifecycle. It separates cloud-specific logic from the core kube-controller-manager.

**Q14. How does etcd achieve high availability?**

> etcd uses the Raft consensus algorithm. A quorum of `(n/2)+1` members must agree before a write is committed. For HA, etcd is deployed as an odd-numbered cluster (3 or 5 nodes).

**Q15. What is the difference between imperative and declarative Kubernetes management?**

> Imperative: you tell Kubernetes *what to do* (`kubectl run`, `kubectl delete`). Declarative: you describe *desired state* via YAML manifests and use `kubectl apply`. Declarative is preferred for GitOps and repeatability.

-----

## 2. Kubernetes Installation using kubeadm

**Q1. What is kubeadm and what problem does it solve?**

> kubeadm is an official tool that automates cluster bootstrapping — generating certificates, configuring the control plane, and joining worker nodes — following Kubernetes best practices.

**Q2. What are the prerequisites before running `kubeadm init`?**

> Disable swap, ensure unique hostname/MAC/product_uuid per node, open required ports (6443, 2379-2380, 10250, 10251, 10252), install a CRI (e.g. containerd), and install kubeadm/kubelet/kubectl.

**Q3. What does `kubeadm init` do step by step?**

> It performs preflight checks, generates a CA and certificates, writes kubeconfig files, generates static pod manifests for control plane components, waits for the control plane to be ready, installs the CoreDNS and kube-proxy addons, and prints the join command.

**Q4. What is the kubeconfig file and where is it located by default?**

> It stores cluster connection info (API server URL, certificates, credentials). Default location: `~/.kube/config`. You can override with `KUBECONFIG` env variable or `--kubeconfig` flag.

**Q5. What is a CNI plugin and why must one be installed after `kubeadm init`?**

> CNI (Container Network Interface) plugins provide pod networking. Without a CNI, pods cannot communicate across nodes and the CoreDNS pods will stay Pending. Examples: Calico, Flannel, Cilium, Weave.

**Q6. How do you join a worker node to a cluster?**

> Use the `kubeadm join` command printed after `kubeadm init`, providing the API server address, token, and CA cert hash. Example:
> 
> ```
> kubeadm join <control-plane-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
> ```

**Q7. How do you regenerate the join token if it expires?**

> ```
> kubeadm token create --print-join-command
> ```
> 
> Tokens expire after 24 hours by default.

**Q8. What is the difference between `kubeadm init` and `kubeadm reset`?**

> `kubeadm init` bootstraps a cluster. `kubeadm reset` tears down the cluster setup on a node (removes config, certs, and etcd data) — it’s used to clean up before re-initializing.

**Q9. How do you deploy Calico as the CNI after cluster initialization?**

> ```
> kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
> ```
> 
> Calico supports NetworkPolicy enforcement in addition to basic pod networking.

**Q10. How do you verify that the cluster is healthy after installation?**

> ```
> kubectl get nodes          # All nodes should be Ready
> kubectl get pods -n kube-system  # All system pods should be Running
> kubectl cluster-info
> ```

**Q11. What are the required ports for a Kubernetes control plane node?**

> 6443 (API server), 2379-2380 (etcd), 10250 (kubelet), 10251 (scheduler), 10252 (controller-manager). Worker nodes need 10250 (kubelet) and 30000-32767 (NodePort services).

**Q12. What is `--pod-network-cidr` in `kubeadm init`?**

> It sets the IP range for pod networking. Required by some CNI plugins (e.g. Calico needs `192.168.0.0/16`, Flannel needs `10.244.0.0/16`). Must not overlap with node or service CIDRs.

**Q13. How can you perform a dry-run of `kubeadm init`?**

> ```
> kubeadm init --dry-run
> ```
> 
> This validates configuration and shows what would happen without making any changes.

**Q14. Where does kubeadm store cluster certificates?**

> `/etc/kubernetes/pki/` — contains the CA cert/key, API server cert/key, etcd certs, service account keys, and front-proxy certs.

**Q15. What is the `kubeadm config` command used for?**

> It lets you view, print, and migrate kubeadm configuration. Useful for customizing cluster init with a config file instead of CLI flags:
> 
> ```
> kubeadm config print init-defaults > kubeadm-config.yaml
> kubeadm init --config kubeadm-config.yaml
> ```

-----

## 3. Static Pods & Component Configuration

**Q1. What is a static pod?**

> A static pod is managed directly by kubelet on a specific node, not by the API server. The kubelet watches a directory for pod manifests and creates/restarts them automatically. Control plane components (API server, scheduler, etc.) run as static pods.

**Q2. Where are static pod manifests stored by default?**

> `/etc/kubernetes/manifests/` — kubelet watches this directory. Any YAML placed here is automatically created as a static pod.

**Q3. How do static pods appear in `kubectl get pods`?**

> They appear with the node name as a suffix, e.g. `kube-apiserver-controlplane`. They are mirror pods — you can view them but cannot delete them via kubectl (deleting the manifest file is the only way to remove them).

**Q4. How do you create a static pod manually?**

> Create a pod manifest YAML and place it in `/etc/kubernetes/manifests/`. kubelet picks it up automatically within seconds.

**Q5. How do you configure kubelet?**

> kubelet is configured via:
> 
> - Flags in the systemd unit file (`/etc/systemd/system/kubelet.service.d/`)
> - A KubeletConfiguration file (e.g. `/var/lib/kubelet/config.yaml`)
> - `--config` flag pointing to the config file

**Q6. How do you change the staticPodPath for kubelet?**

> Edit the kubelet config file (`/var/lib/kubelet/config.yaml`) and set `staticPodPath: /path/to/dir`, then restart kubelet with `systemctl restart kubelet`.

**Q7. What happens if you delete a static pod’s manifest file?**

> kubelet detects the deletion and stops/removes the container immediately. The mirror pod also disappears from the API server.

**Q8. How do you view control plane component manifests?**

> ```
> ls /etc/kubernetes/manifests/
> cat /etc/kubernetes/manifests/kube-apiserver.yaml
> ```

**Q9. How do you restart kubelet?**

> ```
> systemctl daemon-reload
> systemctl restart kubelet
> systemctl status kubelet
> ```

**Q10. What is the difference between a static pod and a regular pod?**

> Static pods are managed by kubelet directly from local manifests. Regular pods are scheduled by the API server and scheduler. Static pods cannot be managed via `kubectl` (except for viewing mirror pods).

**Q11. How do you validate that a static pod is running?**

> ```
> kubectl get pods -n kube-system   # Check mirror pod
> crictl ps                          # Check on the node directly
> systemctl status kubelet
> ```

**Q12. What kubelet configuration option sets the cgroup driver?**

> The `cgroupDriver` field in KubeletConfiguration. It must match the container runtime’s cgroup driver (both should be `systemd` on modern systems).

**Q13. How do you troubleshoot a static pod that isn’t starting?**

> Check kubelet logs: `journalctl -u kubelet -f`, inspect the manifest YAML for errors, verify the file is in the correct `staticPodPath`, and check container runtime logs with `crictl logs <container-id>`.

**Q14. Can static pods be created in any namespace?**

> Static pods are always created in the namespace defined in the manifest. If not specified, they go to the `default` namespace. Control plane static pods use `kube-system`.

**Q15. What is the `--manifest-url` flag in kubelet?**

> It allows kubelet to fetch pod manifests from a URL (in addition to the local staticPodPath). Less commonly used but useful for bootstrapping from remote configs.

-----

## 4. Namespaces & Resource Management

**Q1. What is a Kubernetes namespace?**

> A namespace is a logical partition within a cluster that isolates groups of resources. Names within a namespace must be unique, but the same name can exist in different namespaces.

**Q2. What namespaces exist by default in Kubernetes?**

> `default` (for user workloads when no namespace is specified), `kube-system` (system components), `kube-public` (publicly readable data, e.g. cluster-info ConfigMap), and `kube-node-lease` (node heartbeat Lease objects).

**Q3. How do you create a namespace?**

> ```
> kubectl create namespace dev
> # or declaratively:
> kubectl apply -f - <<EOF
> apiVersion: v1
> kind: Namespace
> metadata:
>   name: dev
> EOF
> ```

**Q4. What is a ResourceQuota?**

> A ResourceQuota sets aggregate limits on resource consumption within a namespace — e.g. total CPU, memory, number of pods, services, PVCs, etc.

**Q5. How do you apply a ResourceQuota to a namespace?**

> ```yaml
> apiVersion: v1
> kind: ResourceQuota
> metadata:
>   name: dev-quota
>   namespace: dev
> spec:
>   hard:
>     requests.cpu: "4"
>     requests.memory: 8Gi
>     limits.cpu: "8"
>     limits.memory: 16Gi
>     pods: "20"
> ```

**Q6. What is a LimitRange?**

> A LimitRange sets default and max/min resource constraints for individual pods/containers within a namespace. Unlike ResourceQuota (which limits totals), LimitRange limits per-object resource usage.

**Q7. What happens if a pod is created in a namespace with a ResourceQuota but the pod has no resource requests/limits?**

> If the namespace has a LimitRange with defaults, those are applied. Without defaults, the pod creation is rejected because ResourceQuota requires all pods to declare resource requests/limits.

**Q8. How do you view current quota usage in a namespace?**

> ```
> kubectl describe resourcequota -n dev
> kubectl get resourcequota -n dev -o yaml
> ```

**Q9. How do you set a default namespace for kubectl commands?**

> ```
> kubectl config set-context --current --namespace=dev
> ```
> 
> After this, all commands run in the `dev` namespace unless overridden with `-n`.

**Q10. Can cluster-scoped resources belong to a namespace?**

> No. Resources like Nodes, PersistentVolumes, ClusterRoles, and Namespaces themselves are cluster-scoped and not namespaced.

**Q11. How do you list all resources across all namespaces?**

> ```
> kubectl get pods --all-namespaces
> # or shorthand:
> kubectl get pods -A
> ```

**Q12. What is the difference between `requests` and `limits` in a LimitRange?**

> `requests` is the minimum guaranteed resource. `limits` is the maximum the container can use. If `limits` are exceeded, the container is throttled (CPU) or OOMKilled (memory).

**Q13. How do you delete a namespace and all its resources?**

> ```
> kubectl delete namespace dev
> ```
> 
> This cascades and deletes all resources within the namespace.

**Q14. How do you deploy a pod to a specific namespace imperatively?**

> ```
> kubectl run nginx --image=nginx -n dev
> ```

**Q15. What happens to resources when a namespace is stuck in `Terminating` state?**

> It usually means a finalizer is preventing deletion. Check with `kubectl get namespace dev -o yaml`, identify the finalizers, and patch them out:
> 
> ```
> kubectl patch namespace dev -p '{"metadata":{"finalizers":[]}}' --type=merge
> ```

-----

## 5. Labels, Selectors & Annotations

**Q1. What are labels in Kubernetes?**

> Labels are key-value pairs attached to Kubernetes objects. They are used to identify and select subsets of objects. Example: `app: nginx`, `env: production`.

**Q2. What is the difference between labels and annotations?**

> Labels are for identification and selection (used by selectors, Services, ReplicaSets). Annotations store arbitrary non-identifying metadata (build info, contact, tool configs) and cannot be used in selectors.

**Q3. How do you add a label to a running pod?**

> ```
> kubectl label pod nginx-pod app=web
> ```

**Q4. How do you remove a label from a resource?**

> ```
> kubectl label pod nginx-pod app-
> ```
> 
> The trailing `-` removes the label.

**Q5. What is a label selector?**

> A label selector filters Kubernetes objects based on their labels. There are two types: equality-based (`app=nginx`, `env!=prod`) and set-based (`app in (nginx, apache)`, `env notin (dev)`).

**Q6. How do you query pods with a specific label using kubectl?**

> ```
> kubectl get pods -l app=nginx
> kubectl get pods -l 'env in (prod,staging)'
> kubectl get pods --selector="app=nginx,tier=frontend"
> ```

**Q7. What is `matchLabels` vs `matchExpressions` in a selector?**

> `matchLabels` is equality-based (AND logic). `matchExpressions` supports set-based operators (`In`, `NotIn`, `Exists`, `DoesNotExist`) for more complex selection.

**Q8. How do you update an annotation on a resource?**

> ```
> kubectl annotate pod nginx-pod description="frontend web server"
> # Overwrite existing:
> kubectl annotate pod nginx-pod description="updated" --overwrite
> ```

**Q9. What is the recommended label schema for Kubernetes resources?**

> The `app.kubernetes.io/` prefix defines standard labels:
> 
> - `app.kubernetes.io/name` — app name
> - `app.kubernetes.io/version` — version
> - `app.kubernetes.io/component` — component role
> - `app.kubernetes.io/part-of` — higher-level app
> - `app.kubernetes.io/managed-by` — tool managing the resource

**Q10. How does a Service use label selectors to route traffic?**

> A Service’s `spec.selector` matches pods by their labels. Any pod with matching labels becomes an endpoint of the Service, and traffic is load-balanced across them.

**Q11. Can two pods have the same label?**

> Yes. Labels are not unique identifiers. Multiple pods can share the same labels, and selectors rely on this to group related resources.

**Q12. How do you filter resources by multiple labels?**

> ```
> kubectl get pods -l app=nginx,env=prod
> ```
> 
> Multiple selectors are ANDed together.

**Q13. What happens if you change the labels on a pod that’s selected by a ReplicaSet?**

> If you remove the label that the ReplicaSet selects on, the pod is no longer controlled by the ReplicaSet. The ReplicaSet then creates a new pod to maintain the desired replica count.

**Q14. Can annotations store large amounts of data?**

> Technically yes — annotation values can hold up to 256KB. They’re commonly used for storing `kubectl.kubernetes.io/last-applied-configuration` or CI/CD metadata.

**Q15. How do you view all labels on a pod?**

> ```
> kubectl get pod nginx-pod --show-labels
> kubectl describe pod nginx-pod   # Shows labels in the metadata section
> ```

-----

## 6. Pod Design

**Q1. What is a Pod in Kubernetes?**

> A Pod is the smallest deployable unit in Kubernetes. It encapsulates one or more containers that share the same network namespace, IP address, and storage volumes.

**Q2. What are multi-container pod patterns?**

> Three common patterns:
> 
> - **Sidecar**: helper container augments the main container (e.g. log shipper)
> - **Ambassador**: proxy for external communication (e.g. service mesh proxy)
> - **Adapter**: transforms output of the main container (e.g. format conversion)

**Q3. What is an init container?**

> An init container runs to completion before any app containers start. Used for setup tasks like waiting for a dependency, populating a shared volume, or running database migrations.

**Q4. How do init containers differ from regular containers?**

> Init containers run sequentially and must complete successfully before app containers start. They don’t support liveness/readiness probes and always restart on failure (based on Pod restartPolicy).

**Q5. How do you create a pod with an init container?**

> ```yaml
> spec:
>   initContainers:
>   - name: init-wait
>     image: busybox
>     command: ['sh', '-c', 'until nslookup myservice; do sleep 2; done']
>   containers:
>   - name: app
>     image: nginx
> ```

**Q6. What restart policies are available for pods?**

> - `Always` (default): always restart on exit
> - `OnFailure`: restart only on non-zero exit
> - `Never`: never restart

**Q7. How do containers in the same pod communicate?**

> Via `localhost` — they share the same network namespace. They must use different ports to avoid conflicts.

**Q8. How do you share data between containers in a pod?**

> Via a shared `emptyDir` volume mounted in both containers. Data persists for the life of the pod.

**Q9. How do you debug a failing pod?**

> ```
> kubectl describe pod <name>     # Events section shows scheduling/image issues
> kubectl logs <name> -c <container>  # Container logs
> kubectl exec -it <name> -- sh   # Interactive shell
> kubectl get pod <name> -o yaml  # Full spec and status
> ```

**Q10. What is the difference between `kubectl logs` and `kubectl exec`?**

> `kubectl logs` retrieves stdout/stderr from a container. `kubectl exec` runs a command inside a running container (interactive shell or one-off command).

**Q11. What does the `command` field override in a pod spec?**

> It overrides the Docker `ENTRYPOINT`. The `args` field overrides the Docker `CMD`. Both can be set independently.

**Q12. What is a Pod’s `restartPolicy` when used in a Job?**

> Jobs should use `OnFailure` or `Never`. Using `Always` with a Job is not recommended as it creates an infinite restart loop.

**Q13. What is an ephemeral container and when is it used?**

> Ephemeral containers are temporary containers added to a running pod for debugging purposes when the main container doesn’t have a shell. Added with:
> 
> ```
> kubectl debug -it <pod> --image=busybox --target=<container>
> ```

**Q14. What is the significance of the pod’s `phase`?**

> Pod phase summarizes its lifecycle: `Pending` (not yet scheduled or images pulling), `Running` (at least one container running), `Succeeded` (all containers exited 0), `Failed` (containers exited non-zero), `Unknown` (state cannot be determined).

**Q15. How can you force delete a stuck pod?**

> ```
> kubectl delete pod <name> --grace-period=0 --force
> ```
> 
> This bypasses the graceful termination period. Use only when necessary.

-----

## 7. Deployments & ReplicaSets

**Q1. What is a Deployment in Kubernetes?**

> A Deployment is a higher-level abstraction that manages ReplicaSets and provides declarative updates, rolling updates, and rollback capabilities for stateless applications.

**Q2. What is the relationship between a Deployment and a ReplicaSet?**

> A Deployment creates and manages ReplicaSets. Each update to the Deployment creates a new ReplicaSet. The Deployment controller scales up the new RS and scales down the old one during rolling updates.

**Q3. How do you create a Deployment imperatively?**

> ```
> kubectl create deployment nginx --image=nginx --replicas=3
> ```

**Q4. How do you scale a Deployment?**

> ```
> kubectl scale deployment nginx --replicas=5
> # or update the manifest and apply
> ```

**Q5. What is a rolling update strategy in Deployments?**

> Rolling update gradually replaces old pods with new ones. Controlled by:
> 
> - `maxUnavailable`: max pods unavailable during update (default 25%)
> - `maxSurge`: max pods created above desired count (default 25%)

**Q6. How do you update the image of a running Deployment?**

> ```
> kubectl set image deployment/nginx nginx=nginx:1.21
> # or edit the manifest:
> kubectl edit deployment nginx
> ```

**Q7. How do you check rollout status?**

> ```
> kubectl rollout status deployment/nginx
> kubectl rollout history deployment/nginx
> ```

**Q8. How do you rollback a Deployment to a previous version?**

> ```
> kubectl rollout undo deployment/nginx
> # Rollback to specific revision:
> kubectl rollout undo deployment/nginx --to-revision=2
> ```

**Q9. What is `minReadySeconds` in a Deployment?**

> It specifies the minimum number of seconds a newly created pod should be ready (no container crashes) before it’s considered available. Helps prevent premature traffic routing.

**Q10. What does `revisionHistoryLimit` control?**

> The number of old ReplicaSets retained for rollback. Default is 10. Set to 0 to disable rollback history.

**Q11. What happens to old ReplicaSets after a rolling update?**

> They are kept with 0 replicas (for rollback purposes) up to the `revisionHistoryLimit`. They are not deleted immediately.

**Q12. What is the difference between `Recreate` and `RollingUpdate` strategies?**

> `Recreate` terminates all old pods first, then creates new ones (causes downtime). `RollingUpdate` incrementally replaces pods with zero or minimal downtime.

**Q13. How do you pause and resume a Deployment rollout?**

> ```
> kubectl rollout pause deployment/nginx
> # Make multiple changes...
> kubectl rollout resume deployment/nginx
> ```
> 
> Pausing prevents the rollout from progressing while you batch changes.

**Q14. How can you check which ReplicaSet a Deployment is using?**

> ```
> kubectl get replicaset -l app=nginx
> kubectl describe deployment nginx  # Shows NewReplicaSet field
> ```

**Q15. What is `progressDeadlineSeconds` in a Deployment?**

> The duration in seconds for a Deployment to make progress before it’s considered failed. Default is 600 seconds. The Deployment controller marks the Deployment as failed with a `ProgressDeadlineExceeded` condition.

-----

## 8. DaemonSets, Jobs & CronJobs

**Q1. What is a DaemonSet?**

> A DaemonSet ensures that a copy of a pod runs on all (or selected) nodes. Used for cluster-wide services like log collectors (Fluentd), monitoring agents (Prometheus Node Exporter), or network plugins.

**Q2. How does a DaemonSet handle new nodes being added to the cluster?**

> The DaemonSet controller automatically schedules the pod on any newly added node that matches the DaemonSet’s node selector or affinity rules.

**Q3. How do you restrict a DaemonSet to specific nodes?**

> Use `nodeSelector`, `nodeAffinity`, or tolerations in the pod template spec:
> 
> ```yaml
> spec:
>   template:
>     spec:
>       nodeSelector:
>         disktype: ssd
> ```

**Q4. What is a Job in Kubernetes?**

> A Job creates one or more pods to run a task to completion. It ensures a specified number of completions succeed before it’s considered done. Useful for batch processing, one-time migrations, etc.

**Q5. What happens to pods created by a Job after it completes?**

> Pods are retained (not deleted) so you can inspect logs. The Job itself tracks completion. You can configure `ttlSecondsAfterFinished` to auto-delete Jobs and their pods.

**Q6. What is `completions` and `parallelism` in a Job spec?**

> `completions`: total number of successful pod runs required. `parallelism`: number of pods running concurrently. E.g., `completions: 10, parallelism: 3` runs 3 pods at a time until 10 succeed.

**Q7. What are the three completion modes for Jobs?**

> - **NonIndexed** (default): any pod completion counts
> - **Indexed**: each pod gets a unique index (0 to completions-1) via `JOB_COMPLETION_INDEX` env var
> - **Fixed**: similar to indexed but for ordered processing

**Q8. What is `backoffLimit` in a Job?**

> The number of retries before the Job is marked as failed. Default is 6. Each failed pod counts as one retry.

**Q9. What is a CronJob?**

> A CronJob creates Jobs on a time-based schedule using standard cron syntax. Example: `"0 2 * * *"` runs at 2 AM daily.

**Q10. What is `successfulJobsHistoryLimit` and `failedJobsHistoryLimit` in a CronJob?**

> They control how many completed/failed Jobs are retained for history. Defaults are 3 and 1 respectively.

**Q11. What happens if a CronJob misses its scheduled time?**

> Controlled by `startingDeadlineSeconds`. If the Job is not started within this window after the scheduled time, it’s counted as missed. After 100 missed schedules, the CronJob is suspended.

**Q12. What is `concurrencyPolicy` in a CronJob?**

> Controls what happens when a new Job is triggered while a previous one is still running:
> 
> - `Allow` (default): run concurrently
> - `Forbid`: skip new run
> - `Replace`: cancel current, start new

**Q13. How do you manually trigger a CronJob?**

> ```
> kubectl create job manual-run --from=cronjob/my-cronjob
> ```

**Q14. How is a DaemonSet update performed?**

> Two strategies: `RollingUpdate` (default — replaces pods one at a time) and `OnDelete` (new pods only created when old ones are manually deleted).

**Q15. How do you suspend a CronJob temporarily?**

> ```
> kubectl patch cronjob my-cronjob -p '{"spec":{"suspend":true}}'
> ```
> 
> Set to `false` to resume.

-----

## 9. Services & DNS

**Q1. What is a Kubernetes Service?**

> A Service provides a stable network endpoint to access a group of pods. It load-balances traffic across matching pods using label selectors and maintains a consistent ClusterIP and DNS name.

**Q2. What are the four types of Kubernetes Services?**

> - `ClusterIP` (default): internal cluster access only
> - `NodePort`: exposes on a static port on every node
> - `LoadBalancer`: provisions a cloud load balancer
> - `ExternalName`: maps to an external DNS name

**Q3. What is a ClusterIP Service?**

> It assigns a virtual IP accessible only within the cluster. Ideal for internal service-to-service communication.

**Q4. What is a NodePort Service and what port range does it use?**

> NodePort exposes the service on a port on every cluster node (range: 30000-32767). External traffic can reach pods via `<NodeIP>:<NodePort>`.

**Q5. How does Kubernetes DNS work for Services?**

> CoreDNS provides DNS for services. A Service `my-svc` in namespace `my-ns` is accessible as:
> 
> - `my-svc` (within same namespace)
> - `my-svc.my-ns`
> - `my-svc.my-ns.svc.cluster.local` (FQDN)

**Q6. What is a Headless Service?**

> A Service with `clusterIP: None`. It doesn’t load-balance — instead, DNS returns all pod IPs directly. Used with StatefulSets for direct pod addressing.

**Q7. What is an Endpoints object?**

> An Endpoints object lists the IP:port pairs of pods backing a Service. Automatically managed by the Endpoints controller based on label selectors.

**Q8. How do you expose a Deployment as a Service?**

> ```
> kubectl expose deployment nginx --port=80 --target-port=80 --type=ClusterIP
> ```

**Q9. What is the difference between `port`, `targetPort`, and `nodePort` in a Service?**

> - `port`: port the Service listens on (ClusterIP)
> - `targetPort`: port on the pod containers
> - `nodePort`: port exposed on each node (NodePort/LoadBalancer only)

**Q10. How does kube-proxy implement Service routing?**

> By default, kube-proxy uses iptables rules to NAT traffic from the Service ClusterIP to one of the backing pod IPs. In IPVS mode, it uses kernel-level load balancing for better performance.

**Q11. What is `sessionAffinity` in a Service?**

> When set to `ClientIP`, all requests from the same client IP are routed to the same pod. Default is `None` (random load balancing).

**Q12. How do you test DNS resolution inside the cluster?**

> ```
> kubectl run dnstest --image=busybox --rm -it -- nslookup my-svc.my-ns.svc.cluster.local
> ```

**Q13. What is an ExternalName Service?**

> It maps a Kubernetes Service name to an external DNS name without proxying. Example:
> 
> ```yaml
> spec:
>   type: ExternalName
>   externalName: my.database.example.com
> ```

**Q14. What are `endpointSlices`?**

> EndpointSlices are scalable replacements for Endpoints objects. They shard endpoint data across multiple objects (max 100 endpoints per slice) to reduce API server load in large clusters.

**Q15. What is the `publishNotReadyAddresses` field in a Service?**

> When set to `true`, the Service includes pod IPs even if pods are not ready. Useful for StatefulSets where pods need to discover each other during initialization.

-----

## 10. Ingress Controllers & Networking

**Q1. What is an Ingress resource?**

> An Ingress exposes HTTP/HTTPS routes from outside the cluster to Services inside. It supports host/path-based routing, TLS termination, and virtual hosting.

**Q2. What is an Ingress Controller?**

> The Ingress Controller is the implementation that reads Ingress resources and configures a load balancer/reverse proxy (e.g. NGINX, Traefik, HAProxy). The Ingress resource is just a config — it needs a controller to act on it.

**Q3. How do you install the NGINX Ingress Controller?**

> ```
> kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.4/deploy/static/provider/cloud/deploy.yaml
> ```

**Q4. How do you create a path-based Ingress rule?**

> ```yaml
> spec:
>   rules:
>   - host: myapp.example.com
>     http:
>       paths:
>       - path: /api
>         pathType: Prefix
>         backend:
>           service:
>             name: api-svc
>             port:
>               number: 80
>       - path: /web
>         pathType: Prefix
>         backend:
>           service:
>             name: web-svc
>             port:
>               number: 80
> ```

**Q5. What are `pathType` options in Ingress?**

> - `Exact`: exact path match
> - `Prefix`: prefix match (most common)
> - `ImplementationSpecific`: controller decides

**Q6. How do you configure TLS in an Ingress?**

> ```yaml
> spec:
>   tls:
>   - hosts:
>     - myapp.example.com
>     secretName: my-tls-secret
> ```
> 
> The Secret must contain `tls.crt` and `tls.key`.

**Q7. What is a NetworkPolicy?**

> A NetworkPolicy controls ingress and egress traffic at the pod level. Without any NetworkPolicy, all pods can communicate with all other pods. Once applied, only explicitly allowed traffic is permitted.

**Q8. How do you deny all ingress traffic to pods in a namespace?**

> ```yaml
> apiVersion: networking.k8s.io/v1
> kind: NetworkPolicy
> metadata:
>   name: deny-all
> spec:
>   podSelector: {}
>   policyTypes:
>   - Ingress
> ```

**Q9. What CNI plugins support NetworkPolicy enforcement?**

> Calico, Cilium, Weave Net, and Antrea. Flannel does NOT enforce NetworkPolicy by default.

**Q10. What is the difference between Ingress and a LoadBalancer Service?**

> A LoadBalancer Service creates one cloud LB per Service (expensive). Ingress consolidates routing for many Services behind a single load balancer using host/path rules.

**Q11. What is an `IngressClass`?**

> IngressClass specifies which Ingress controller should handle an Ingress resource. Multiple controllers can coexist, each handling different IngressClasses.

**Q12. How do you allow only specific pods to talk to a backend pod using NetworkPolicy?**

> ```yaml
> spec:
>   podSelector:
>     matchLabels:
>       app: backend
>   ingress:
>   - from:
>     - podSelector:
>         matchLabels:
>           app: frontend
> ```

**Q13. How do you test that an Ingress is working?**

> ```
> curl -H "Host: myapp.example.com" http://<ingress-controller-ip>/api
> kubectl describe ingress my-ingress   # Check backend and rules
> kubectl get ingress                   # Check ADDRESS field
> ```

**Q14. What is a default backend in Ingress?**

> The default backend handles requests that don’t match any rule. Configured as:
> 
> ```yaml
> spec:
>   defaultBackend:
>     service:
>       name: default-svc
>       port:
>         number: 80
> ```

**Q15. How does Cilium enhance NetworkPolicy over standard Kubernetes?**

> Cilium supports L7 (application layer) policies based on HTTP methods/paths, DNS names, and Kafka topics — far beyond the L3/L4 (IP/port) level of standard NetworkPolicy.

-----

## 11. ConfigMaps & Secrets

**Q1. What is a ConfigMap?**

> A ConfigMap stores non-sensitive configuration data as key-value pairs. Pods can consume it as environment variables, command-line arguments, or mounted files.

**Q2. How do you create a ConfigMap from a file?**

> ```
> kubectl create configmap app-config --from-file=config.properties
> kubectl create configmap app-config --from-env-file=.env
> ```

**Q3. How do you inject a ConfigMap as environment variables?**

> ```yaml
> envFrom:
> - configMapRef:
>     name: app-config
> ```
> 
> Or for specific keys:
> 
> ```yaml
> env:
> - name: LOG_LEVEL
>   valueFrom:
>     configMapKeyRef:
>       name: app-config
>       key: log.level
> ```

**Q4. How do you mount a ConfigMap as a volume?**

> ```yaml
> volumes:
> - name: config-vol
>   configMap:
>     name: app-config
> containers:
> - volumeMounts:
>   - name: config-vol
>     mountPath: /etc/config
> ```
> 
> Each key becomes a file at the mount path.

**Q5. What is a Kubernetes Secret?**

> A Secret stores sensitive data (passwords, tokens, TLS certs). Values are base64-encoded by default. For encryption at rest, etcd encryption must be enabled separately.

**Q6. What are the built-in Secret types?**

> - `Opaque` (default): arbitrary key-value data
> - `kubernetes.io/dockerconfigjson`: Docker registry credentials
> - `kubernetes.io/tls`: TLS cert and key
> - `kubernetes.io/service-account-token`: auto-mounted SA token

**Q7. How do you create a Secret imperatively?**

> ```
> kubectl create secret generic db-secret \
>   --from-literal=username=admin \
>   --from-literal=password=s3cr3t
> ```

**Q8. How do you base64-encode a value for a Secret manifest?**

> ```
> echo -n 'mypassword' | base64
> ```
> 
> Paste the output as the value in the Secret YAML.

**Q9. How is a Secret different from a ConfigMap in terms of security?**

> Secrets are stored encoded (not encrypted by default) in etcd. They’re only sent to nodes that need them and stored in `tmpfs` in pods. Enable etcd encryption and restrict RBAC access for true security.

**Q10. How do you mount a specific Secret key as a file?**

> ```yaml
> volumes:
> - name: secret-vol
>   secret:
>     secretName: db-secret
>     items:
>     - key: password
>       path: db-password.txt
> ```

**Q11. What is an `imagePullSecret`?**

> A Secret of type `kubernetes.io/dockerconfigjson` referenced in a pod spec to authenticate to a private container registry:
> 
> ```yaml
> spec:
>   imagePullSecrets:
>   - name: regcred
> ```

**Q12. Can ConfigMaps be updated without restarting pods?**

> Yes — when mounted as volumes, ConfigMap updates propagate to the mounted files automatically (with some delay). When injected as environment variables, the pod must be restarted to pick up changes.

**Q13. What is the size limit for a ConfigMap or Secret?**

> 1 MiB per object. For larger configs, consider mounting from external sources or using a configuration management tool.

**Q14. How do you view the decoded value of a Secret?**

> ```
> kubectl get secret db-secret -o jsonpath='{.data.password}' | base64 --decode
> ```

**Q15. What is the risk of using Secrets as environment variables vs volume mounts?**

> Environment variables are visible in process listings, logs, and child processes. Volume-mounted secrets are only accessible as files and are safer. Always prefer volume mounts for sensitive data.

-----

## 12. Resource Management & Probes

**Q1. What is the difference between resource `requests` and `limits`?**

> `requests` is what the scheduler uses to place the pod — the guaranteed minimum. `limits` is the maximum a container can use. If a container exceeds its memory limit, it’s OOMKilled.

**Q2. What happens when a container exceeds its CPU limit?**

> CPU is throttled — the container is not killed, but it won’t get more CPU than the limit. This can cause latency issues.

**Q3. What is a liveness probe?**

> A liveness probe checks if a container is still running correctly. If it fails, kubelet restarts the container. Used to recover from deadlocks or application crashes.

**Q4. What is a readiness probe?**

> A readiness probe checks if a container is ready to receive traffic. If it fails, the pod is removed from Service endpoints. The container is NOT restarted.

**Q5. What is a startup probe?**

> A startup probe checks if the application has started. While it’s running, liveness and readiness probes are disabled. Useful for slow-starting applications to avoid premature restarts.

**Q6. What are the three types of probe handlers?**

> - `exec`: runs a command; success = exit 0
> - `httpGet`: HTTP GET request; success = 200-399
> - `tcpSocket`: TCP connection; success = port accepts connection

**Q7. What probe configuration fields are important?**

> - `initialDelaySeconds`: wait before first probe
> - `periodSeconds`: how often to probe (default 10)
> - `failureThreshold`: consecutive failures before action (default 3)
> - `successThreshold`: consecutive successes to consider healthy (default 1)
> - `timeoutSeconds`: probe timeout (default 1)

**Q8. What is a Horizontal Pod Autoscaler (HPA)?**

> HPA automatically scales the number of pod replicas based on observed metrics (CPU, memory, custom metrics). It queries the metrics-server and adjusts the Deployment/ReplicaSet replicas.

**Q9. How do you create an HPA?**

> ```
> kubectl autoscale deployment nginx --cpu-percent=50 --min=2 --max=10
> # or declaratively with autoscaling/v2 for multiple metrics
> ```

**Q10. What is the metrics-server and why is it needed for HPA?**

> metrics-server is a lightweight aggregator of resource usage data (CPU/memory) from kubelets. HPA and `kubectl top` depend on it. Without it, HPA cannot function.

**Q11. What is Vertical Pod Autoscaler (VPA)?**

> VPA automatically adjusts CPU/memory requests and limits for pods based on usage history. Unlike HPA, it changes resource specs rather than replica counts.

**Q12. What is Quality of Service (QoS) class in Kubernetes?**

> QoS class determines pod eviction priority under resource pressure:
> 
> - `Guaranteed`: requests == limits for all containers
> - `Burstable`: requests < limits (or only some containers set)
> - `BestEffort`: no requests or limits set (evicted first)

**Q13. What is `LimitRange` used for in resource management?**

> LimitRange enforces default and min/max resource constraints per container/pod in a namespace. It prevents pods with no resource specs from being scheduled in namespaces with ResourceQuota.

**Q14. How do you check current CPU and memory usage of pods?**

> ```
> kubectl top pods
> kubectl top pods -n kube-system
> kubectl top nodes
> ```
> 
> Requires metrics-server to be installed.

**Q15. What is the `resources.requests.ephemeral-storage` field?**

> It reserves local ephemeral storage (logs, emptyDir volumes) for a container. If exceeded, the pod is evicted. Useful to prevent one pod from starving others of node disk space.

-----

## 13. Volumes & Persistent Storage

**Q1. What is a Volume in Kubernetes?**

> A Volume is a directory accessible to containers in a pod. Unlike container storage, volumes survive container restarts (within the same pod lifecycle). Many types exist: `emptyDir`, `hostPath`, `configMap`, `secret`, `nfs`, `persistentVolumeClaim`, etc.

**Q2. What is an `emptyDir` volume?**

> A temporary directory created when a pod is assigned to a node. It exists as long as the pod runs and is deleted when the pod is removed. Useful for sharing data between containers in the same pod.

**Q3. What is a `hostPath` volume and when should you use it?**

> `hostPath` mounts a file or directory from the node’s filesystem. Used for accessing node-level resources (e.g. Docker socket, node logs). Avoid in production multi-node setups due to data location dependency.

**Q4. What is a PersistentVolume (PV)?**

> A PV is a cluster-level storage resource provisioned by an admin (or dynamically by a StorageClass). It has a lifecycle independent of any pod.

**Q5. What is a PersistentVolumeClaim (PVC)?**

> A PVC is a user’s request for storage. It specifies size, access mode, and optionally a StorageClass. Kubernetes binds the PVC to a matching PV.

**Q6. What are the access modes for PVs?**

> - `ReadWriteOnce (RWO)`: one node, read-write
> - `ReadOnlyMany (ROX)`: many nodes, read-only
> - `ReadWriteMany (RWX)`: many nodes, read-write
> - `ReadWriteOncePod (RWOP)`: single pod only (Kubernetes 1.22+)

**Q7. What is the PV reclaim policy?**

> - `Retain`: PV is kept after PVC deletion (manual cleanup required)
> - `Delete`: PV and underlying storage are deleted with the PVC
> - `Recycle` (deprecated): basic scrub (`rm -rf`) before reuse

**Q8. How do you create a PV and PVC?**

> ```yaml
> # PV
> apiVersion: v1
> kind: PersistentVolume
> metadata:
>   name: my-pv
> spec:
>   capacity:
>     storage: 5Gi
>   accessModes:
>   - ReadWriteOnce
>   hostPath:
>     path: /data/my-pv
> ---
> # PVC
> apiVersion: v1
> kind: PersistentVolumeClaim
> metadata:
>   name: my-pvc
> spec:
>   accessModes:
>   - ReadWriteOnce
>   resources:
>     requests:
>       storage: 5Gi
> ```

**Q9. How do you mount a PVC in a pod?**

> ```yaml
> volumes:
> - name: data
>   persistentVolumeClaim:
>     claimName: my-pvc
> containers:
> - volumeMounts:
>   - name: data
>     mountPath: /data
> ```

**Q10. What is the binding process between PVC and PV?**

> The PVC controller finds a PV that satisfies the PVC’s requirements (size ≥ requested, matching access modes, matching StorageClass). Once bound, the PV is exclusively reserved for that PVC.

**Q11. What PVC states exist?**

> - `Pending`: no matching PV found yet
> - `Bound`: successfully bound to a PV
> - `Lost`: bound PV is no longer available

**Q12. What happens to a PVC-bound PV when the PVC is deleted (Retain policy)?**

> The PV moves to `Released` state. It cannot be automatically bound to another PVC. An admin must manually clean and re-create the PV to make it available.

**Q13. How do you expand a PVC?**

> Edit the PVC and increase `spec.resources.requests.storage`. The StorageClass must have `allowVolumeExpansion: true`. The pod may need to be restarted for the resize to take effect at the filesystem level.

**Q14. What is the difference between static and dynamic provisioning?**

> Static: admin manually creates PVs. Dynamic: a StorageClass automatically provisions a PV when a PVC is created (no pre-created PVs needed).

**Q15. What is a `projected` volume?**

> A projected volume maps multiple volume sources (secrets, configmaps, serviceAccountTokens, downwardAPI) into a single directory in the pod.

-----

## 14. Storage Classes & StatefulSets

**Q1. What is a StorageClass?**

> A StorageClass defines the type of storage to provision dynamically. It specifies the provisioner (e.g. `kubernetes.io/aws-ebs`, `driver.longhorn.io`), parameters, and reclaim policy.

**Q2. What is dynamic provisioning?**

> When a PVC references a StorageClass, Kubernetes automatically creates a matching PV using the StorageClass provisioner — no admin intervention needed.

**Q3. How do you set a default StorageClass?**

> Annotate it with:
> 
> ```
> kubectl patch storageclass standard -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
> ```

**Q4. What is the `volumeBindingMode` in a StorageClass?**

> - `Immediate`: PV is provisioned as soon as PVC is created
> - `WaitForFirstConsumer`: PV is provisioned only when a pod using the PVC is scheduled (respects pod topology constraints)

**Q5. What is a StatefulSet?**

> A StatefulSet manages stateful applications. Unlike Deployments, it provides stable pod names (pod-0, pod-1…), stable network identities (via Headless Service), and ordered, graceful deployment/scaling.

**Q6. How do pods in a StatefulSet get their names?**

> They are named `<statefulset-name>-<ordinal>`, e.g. `mysql-0`, `mysql-1`, `mysql-2`. These names are stable and predictable.

**Q7. What is the role of a Headless Service with a StatefulSet?**

> The Headless Service (`clusterIP: None`) gives each pod its own DNS entry:
> `<pod-name>.<service-name>.<namespace>.svc.cluster.local`
> This allows pods to address each other directly.

**Q8. How does StatefulSet handle pod rescheduling?**

> If a pod dies, StatefulSet recreates it with the same name and reattaches the same PVC — maintaining its identity and storage.

**Q9. What is `volumeClaimTemplates` in a StatefulSet?**

> It automatically creates a PVC for each pod using the specified template:
> 
> ```yaml
> volumeClaimTemplates:
> - metadata:
>     name: data
>   spec:
>     accessModes: ["ReadWriteOnce"]
>     resources:
>       requests:
>         storage: 10Gi
> ```
> 
> Pod `mysql-0` gets PVC `data-mysql-0`, pod `mysql-1` gets `data-mysql-1`, etc.

**Q10. How are StatefulSet pods scaled down?**

> In reverse ordinal order (highest first): `mysql-2` is deleted before `mysql-1`, then `mysql-0`. This ensures proper shutdown order for stateful applications.

**Q11. What update strategies does StatefulSet support?**

> - `RollingUpdate` (default): updates pods from highest ordinal to lowest
> - `OnDelete`: pods only updated when manually deleted

**Q12. What is the `partition` field in StatefulSet RollingUpdate?**

> Only pods with ordinal >= partition value are updated. Useful for canary releases: set `partition: 2` to only update pod-2, keeping pod-0 and pod-1 on the old version.

**Q13. What is the difference between StatefulSet and Deployment?**

> Deployment: stateless, pods are interchangeable, random names, shared or no persistent storage. StatefulSet: stateful, pods have unique identities, stable storage, ordered operations.

**Q14. When should you NOT use a StatefulSet?**

> For stateless apps (web servers, API services) — use Deployment. StatefulSets add ordering overhead and complexity that isn’t needed when pods are interchangeable.

**Q15. How do you delete a StatefulSet without deleting its pods?**

> ```
> kubectl delete statefulset mysql --cascade=orphan
> ```
> 
> Orphaned pods continue running but are no longer managed.

-----

## 15. Kubernetes Security (TLS, RBAC)

**Q1. What is RBAC in Kubernetes?**

> Role-Based Access Control (RBAC) regulates access to Kubernetes API resources based on the roles assigned to users, groups, or ServiceAccounts.

**Q2. What are the four key RBAC objects?**

> - `Role`: namespaced permissions
> - `ClusterRole`: cluster-wide permissions
> - `RoleBinding`: binds Role to subject in a namespace
> - `ClusterRoleBinding`: binds ClusterRole to subject cluster-wide

**Q3. How do you create a Role that allows reading pods?**

> ```yaml
> apiVersion: rbac.authorization.k8s.io/v1
> kind: Role
> metadata:
>   namespace: dev
>   name: pod-reader
> rules:
> - apiGroups: [""]
>   resources: ["pods"]
>   verbs: ["get", "watch", "list"]
> ```

**Q4. How do you bind a Role to a user?**

> ```yaml
> apiVersion: rbac.authorization.k8s.io/v1
> kind: RoleBinding
> metadata:
>   name: read-pods
>   namespace: dev
> subjects:
> - kind: User
>   name: jane
>   apiGroup: rbac.authorization.k8s.io
> roleRef:
>   kind: Role
>   name: pod-reader
>   apiGroup: rbac.authorization.k8s.io
> ```

**Q5. What is a ServiceAccount?**

> A ServiceAccount provides an identity for processes running in pods. It allows pods to authenticate to the Kubernetes API and is granted permissions via RBAC.

**Q6. How do you create and use a ServiceAccount in a pod?**

> ```
> kubectl create serviceaccount my-sa -n dev
> ```
> 
> Reference in pod spec:
> 
> ```yaml
> spec:
>   serviceAccountName: my-sa
> ```

**Q7. What TLS certificates does Kubernetes use?**

> - CA certificate (root of trust)
> - API server certificate
> - etcd certificates
> - kubelet certificates (for node auth)
> - Front-proxy certificate (for aggregation layer)
> - Service account signing key pair

**Q8. Where are Kubernetes TLS certificates stored?**

> `/etc/kubernetes/pki/` on the control plane node. Key files: `ca.crt`, `ca.key`, `apiserver.crt`, `apiserver.key`, `etcd/ca.crt`, etc.

**Q9. What does `kubectl auth can-i` do?**

> Checks if the current user has permission to perform an action:
> 
> ```
> kubectl auth can-i create pods
> kubectl auth can-i delete deployments --namespace=prod
> kubectl auth can-i list secrets --as=jane
> ```

**Q10. What is the difference between ClusterRole and Role?**

> Role is namespaced (applies only within one namespace). ClusterRole is cluster-wide and can also be used in RoleBindings to grant namespace-scoped access from a cluster-level definition.

**Q11. How do you list all ClusterRoleBindings for a specific user?**

> ```
> kubectl get clusterrolebindings -o json | jq '.items[] | select(.subjects[]?.name=="jane")'
> ```

**Q12. What is the `system:masters` group in Kubernetes?**

> Members of `system:masters` get full cluster-admin access, bypassing RBAC. The default cluster-admin user (kubeconfig) belongs to this group.

**Q13. What is the principle of least privilege in Kubernetes RBAC?**

> Grant only the minimum permissions required. Use namespaced Roles over ClusterRoles, avoid wildcard `*` verbs/resources, and audit ServiceAccount permissions regularly.

**Q14. How do you check what a ServiceAccount is allowed to do?**

> ```
> kubectl auth can-i list pods --as=system:serviceaccount:dev:my-sa -n dev
> ```

**Q15. What is `automountServiceAccountToken` and when should you disable it?**

> By default, Kubernetes mounts a ServiceAccount token into every pod. For pods that don’t need API access, disable it to reduce attack surface:
> 
> ```yaml
> spec:
>   automountServiceAccountToken: false
> ```

-----

## 16. Authentication & Authorization

**Q1. What authentication methods does Kubernetes support?**

> - X.509 client certificates
> - Static token files
> - Bootstrap tokens
> - Service Account tokens (JWT)
> - OpenID Connect (OIDC) tokens
> - Webhook token authentication
> - Authenticating proxy

**Q2. What is the difference between authentication and authorization in Kubernetes?**

> Authentication verifies *who you are* (identity). Authorization determines *what you can do* (permissions). Kubernetes runs both as separate phases for each API request.

**Q3. What authorization modes does Kubernetes support?**

> - `RBAC` (most common)
> - `ABAC` (attribute-based, legacy)
> - `Node` (restricts kubelet access)
> - `Webhook` (external authorization service)
> - `AlwaysAllow` / `AlwaysDeny` (for testing)

**Q4. What are Admission Controllers?**

> Admission controllers are plugins that intercept API requests after authentication/authorization but before persistence. They can mutate (change) or validate (accept/reject) requests.

**Q5. Name five important Admission Controllers.**

> - `NamespaceLifecycle`: prevents resources in terminating/non-existent namespaces
> - `LimitRanger`: enforces LimitRange constraints
> - `ResourceQuota`: enforces ResourceQuota
> - `PodSecurity`: enforces Pod Security Standards
> - `MutatingAdmissionWebhook` / `ValidatingAdmissionWebhook`: custom logic via webhooks

**Q6. What is the difference between mutating and validating admission webhooks?**

> Mutating webhooks modify the incoming request (e.g. inject sidecars, set defaults). Validating webhooks only accept or reject — no modifications. Mutating runs first.

**Q7. What is OIDC and how is it used with Kubernetes?**

> OIDC (OpenID Connect) allows Kubernetes to delegate authentication to an external identity provider (Okta, Keycloak, Google). The API server validates the JWT token using the provider’s public keys.

**Q8. How does Node authorization work?**

> The Node authorizer allows kubelets to only read/write resources related to their own node (pods scheduled on it, secrets/configmaps needed by those pods). Prevents a compromised node from accessing all cluster data.

**Q9. What is a kubeconfig context?**

> A context groups: a cluster (API server endpoint), a user (credentials), and a namespace. Multiple contexts let you switch between clusters/environments:
> 
> ```
> kubectl config use-context production
> kubectl config get-contexts
> ```

**Q10. How do you create a kubeconfig for a new user with a client certificate?**

> 1. Generate private key: `openssl genrsa -out user.key 2048`
> 1. Create CSR: `openssl req -new -key user.key -out user.csr -subj "/CN=jane/O=dev"`
> 1. Sign with cluster CA: `openssl x509 -req -in user.csr -CA ca.crt -CAkey ca.key -out user.crt`
> 1. Add to kubeconfig: `kubectl config set-credentials jane --client-certificate=user.crt --client-key=user.key`

**Q11. What is the `CertificateSigningRequest` (CSR) API?**

> A Kubernetes-native way to request certificate signing. Submit a CSR object, approve it via `kubectl certificate approve`, and retrieve the signed cert.

**Q12. How do you enable an admission controller on the API server?**

> Add to the `--enable-admission-plugins` flag in the API server manifest:
> 
> ```
> --enable-admission-plugins=NodeRestriction,ResourceQuota,LimitRanger
> ```

**Q13. What is the `PodSecurity` admission controller?**

> It enforces Pod Security Standards (`privileged`, `baseline`, `restricted`) at the namespace level via namespace labels:
> 
> ```
> kubectl label namespace dev pod-security.kubernetes.io/enforce=restricted
> ```

**Q14. What happens if an authorization check fails?**

> The API server returns HTTP 403 Forbidden. The request is not passed to admission controllers or persisted to etcd.

**Q15. What are bootstrap tokens used for?**

> Bootstrap tokens authenticate nodes joining the cluster via `kubeadm join`. They are short-lived (24h by default) and stored as Secrets in the `kube-system` namespace.

-----

## 17. Logging & Monitoring

**Q1. How do you view logs for a pod?**

> ```
> kubectl logs <pod-name>
> kubectl logs <pod-name> -c <container-name>   # multi-container pod
> kubectl logs <pod-name> --previous             # crashed container logs
> kubectl logs -f <pod-name>                     # follow/tail
> ```

**Q2. What is `kubectl logs --previous` used for?**

> It shows logs from the previous (terminated) instance of a container. Crucial for debugging crash loops where the container restarts before you can view logs.

**Q3. How does Kubernetes handle container log rotation?**

> kubelet rotates container logs automatically. Default: 10MB per file, 5 files per container. Configurable via `containerLogMaxSize` and `containerLogMaxFiles` in kubelet config.

**Q4. What is the metrics-server and how do you install it?**

> metrics-server collects CPU/memory metrics from kubelets and exposes them via the Metrics API. Install:
> 
> ```
> kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
> ```

**Q5. How do you monitor resource usage with kubectl?**

> ```
> kubectl top nodes
> kubectl top pods
> kubectl top pods -n kube-system --sort-by=cpu
> ```

**Q6. What is the difference between `kubectl logs` and accessing logs via the node?**

> `kubectl logs` retrieves logs via the API server from the kubelet. Direct node access (`/var/log/containers/`) gives raw log files, which is useful when the API server is down.

**Q7. What logging architectures are common in Kubernetes?**

> - **Node-level**: log agent as DaemonSet (Fluentd, Fluent Bit) ships logs to a centralized store (Elasticsearch, Loki, CloudWatch)
> - **Sidecar**: log agent container in the same pod
> - **Direct**: application writes to external logging service

**Q8. How do you stream logs from all pods in a Deployment?**

> ```
> kubectl logs -l app=nginx -f --max-log-requests=10
> ```
> 
> Or use tools like `stern`:
> 
> ```
> stern nginx
> ```

**Q9. What is Prometheus and how does it relate to Kubernetes monitoring?**

> Prometheus is the de-facto monitoring system for Kubernetes. It scrapes metrics from pods (via `/metrics` endpoint), stores them as time-series, and supports alerting via Alertmanager.

**Q10. What are Kubernetes events and how do you view them?**

> Events record state changes in the cluster (pod scheduling, image pulls, probe failures). View with:
> 
> ```
> kubectl get events -n dev
> kubectl get events --sort-by='.lastTimestamp'
> kubectl describe pod <name>   # Events shown at the bottom
> ```

**Q11. What is the kube-state-metrics project?**

> kube-state-metrics listens to the Kubernetes API and generates metrics about the state of Kubernetes objects (deployment replicas, pod status, node conditions). Complementary to metrics-server.

**Q12. How do you check the kubelet’s log on a node?**

> ```
> journalctl -u kubelet -f
> journalctl -u kubelet --since "1 hour ago"
> ```

**Q13. What is the difference between metrics-server and Prometheus?**

> metrics-server provides short-term, lightweight CPU/memory data for HPA and `kubectl top`. Prometheus stores historical time-series data, supports custom metrics, alerting, and complex queries via PromQL.

**Q14. How do you view logs for a node-level system component?**

> ```
> journalctl -u kube-apiserver    # If running as systemd service
> kubectl logs kube-apiserver-controlplane -n kube-system  # If static pod
> ```

**Q15. What is `kubectl describe` useful for in monitoring/debugging?**

> `kubectl describe` shows full resource spec, status, and recent Events. It’s the first command to run when diagnosing why a pod is Pending, a node is NotReady, or a deployment is stalled.

-----

## 18. Troubleshooting Cluster Issues

**Q1. A pod is stuck in `Pending` state. How do you troubleshoot?**

> ```
> kubectl describe pod <name>   # Look at Events section
> ```
> 
> Common causes: insufficient resources, no matching node (taint/affinity/nodeSelector), PVC not bound, image pull backoff.

**Q2. A pod is in `CrashLoopBackOff`. What are the steps to debug?**

> ```
> kubectl logs <pod> --previous   # Logs from crashed container
> kubectl describe pod <pod>      # Check exit code and reason
> kubectl get pod <pod> -o yaml   # Full spec review
> ```
> 
> Common causes: wrong command/args, missing env vars/secrets, app errors, liveness probe misconfiguration.

**Q3. How do you troubleshoot a node in `NotReady` state?**

> ```
> kubectl describe node <node>    # Check Conditions section
> ssh <node>
> systemctl status kubelet
> journalctl -u kubelet -f
> ```
> 
> Common causes: kubelet stopped, disk pressure, memory pressure, network issues, container runtime failure.

**Q4. How do you fix a scheduling issue caused by a taint?**

> If a node has a taint that pods don’t tolerate:
> 
> ```
> kubectl taint node <node> key=value:NoSchedule-   # Remove taint
> # Or add toleration to the pod spec
> ```

**Q5. How do you troubleshoot DNS resolution failures in pods?**

> ```
> kubectl run dnstest --image=busybox --rm -it -- nslookup kubernetes
> kubectl get pods -n kube-system | grep coredns
> kubectl logs -n kube-system <coredns-pod>
> kubectl describe configmap coredns -n kube-system
> ```

**Q6. How do you troubleshoot a Service that is not routing traffic?**

> ```
> kubectl describe service <svc>       # Check selector and endpoints
> kubectl get endpoints <svc>          # Are there any IPs?
> kubectl get pods -l <selector>       # Do pods match the selector?
> ```
> 
> Common causes: label mismatch between Service selector and pod labels, pods not ready, wrong targetPort.

**Q7. How do you debug a node with disk pressure?**

> ```
> kubectl describe node <node>   # Check DiskPressure condition
> df -h                           # Check disk usage on the node
> du -sh /var/lib/docker/*        # Identify large consumers
> docker system prune             # Clean up unused images/containers
> ```

**Q8. How do you identify which node a pod is running on?**

> ```
> kubectl get pod <name> -o wide
> kubectl describe pod <name> | grep Node:
> ```

**Q9. How do you fix a pod stuck in `Terminating` state?**

> ```
> kubectl delete pod <name> --grace-period=0 --force
> ```
> 
> If still stuck, check for finalizers in the pod spec and patch them out.

**Q10. What does `kubectl get componentstatuses` show?**

> It shows the health of control plane components: etcd, controller-manager, and scheduler. Deprecated in newer Kubernetes versions but still useful for older clusters.

**Q11. How do you troubleshoot an etcd failure?**

> ```
> # Check etcd pod logs
> kubectl logs -n kube-system etcd-controlplane
> # Check etcd member list
> etcdctl --endpoints=https://127.0.0.1:2379 \
>   --cacert=/etc/kubernetes/pki/etcd/ca.crt \
>   --cert=/etc/kubernetes/pki/etcd/peer.crt \
>   --key=/etc/kubernetes/pki/etcd/peer.key \
>   member list
> ```

**Q12. How do you check why a container image is not pulling?**

> ```
> kubectl describe pod <name>    # ImagePullBackOff events
> ```
> 
> Common causes: wrong image name/tag, private registry with no imagePullSecret, registry unreachable from nodes.

**Q13. How do you resolve a `failed to create pod sandbox` error?**

> This is typically a container runtime or CNI issue. Check:
> 
> ```
> journalctl -u kubelet | grep sandbox
> crictl ps                     # Container runtime state
> systemctl status containerd   # Runtime health
> ```

**Q14. How do you debug network connectivity between two pods?**

> ```
> kubectl exec -it pod-a -- curl http://<pod-b-ip>:8080
> kubectl exec -it pod-a -- nc -zv <pod-b-ip> 8080
> kubectl exec -it pod-a -- ping <pod-b-ip>
> ```
> 
> Also check NetworkPolicy rules if connectivity is unexpectedly blocked.

**Q15. A Deployment is not updating pods during a rolling update. How do you investigate?**

> ```
> kubectl rollout status deployment/<name>   # See if it's stalled
> kubectl describe deployment <name>         # Check progressDeadlineSeconds, events
> kubectl get replicasets                    # Check new RS pod count
> kubectl get pods                           # Any pods in error state?
> ```
> 
> Common causes: readiness probe failing, image pull error, insufficient resources.

-----

## 19. Upgrade Cluster using kubeadm

**Q1. What is the recommended upgrade order for a Kubernetes cluster?**

> Always upgrade the control plane first, then worker nodes. Within the control plane, upgrade one node at a time. Never skip minor versions (e.g. 1.27 → 1.29 is not supported; go 1.27 → 1.28 → 1.29).

**Q2. How do you check available upgrade versions?**

> ```
> kubeadm upgrade plan
> ```
> 
> This shows current versions, available upgrades, and the kubeadm configuration.

**Q3. What is the first step to upgrade a control plane node?**

> Upgrade kubeadm itself first:
> 
> ```
> apt-get update
> apt-get install -y kubeadm=1.28.0-00
> kubeadm version
> ```

**Q4. How do you apply the control plane upgrade?**

> ```
> kubeadm upgrade apply v1.28.0
> ```
> 
> This upgrades: kube-apiserver, kube-controller-manager, kube-scheduler, kube-proxy, and CoreDNS.

**Q5. How do you drain a node before upgrading it?**

> ```
> kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
> ```
> 
> This cordons the node (no new pods scheduled) and evicts all pods.

**Q6. After draining, how do you upgrade kubelet and kubectl on a node?**

> ```
> apt-get install -y kubelet=1.28.0-00 kubectl=1.28.0-00
> systemctl daemon-reload
> systemctl restart kubelet
> ```

**Q7. How do you uncordon a node after upgrade?**

> ```
> kubectl uncordon <node>
> ```
> 
> This marks the node as schedulable again.

**Q8. How do you upgrade a worker node?**

> 1. Drain the worker: `kubectl drain <worker> --ignore-daemonsets`
> 1. SSH to worker, upgrade kubeadm: `apt-get install -y kubeadm=1.28.0-00`
> 1. Apply node upgrade: `kubeadm upgrade node`
> 1. Upgrade kubelet/kubectl and restart kubelet
> 1. Uncordon: `kubectl uncordon <worker>`

**Q9. What does `kubeadm upgrade node` do on worker nodes?**

> It updates the local kubelet configuration (from the configmap stored by the control plane upgrade) and renews kubelet certificates if needed.

**Q10. How do you verify a successful cluster upgrade?**

> ```
> kubectl get nodes     # All nodes should show new version
> kubectl version       # Check client and server versions
> kubectl get pods -n kube-system   # All system pods running
> ```

**Q11. What is the version skew policy in Kubernetes?**

> - kubelet can be at most 2 minor versions behind the API server
> - kubectl can be ±1 minor version of the API server
> - kube-proxy must match the API server minor version

**Q12. How do you back up etcd before upgrading?**

> ```
> ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
>   --endpoints=https://127.0.0.1:2379 \
>   --cacert=/etc/kubernetes/pki/etcd/ca.crt \
>   --cert=/etc/kubernetes/pki/etcd/server.crt \
>   --key=/etc/kubernetes/pki/etcd/server.key
> ```

**Q13. How do you restore etcd from a snapshot?**

> ```
> ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db \
>   --data-dir=/var/lib/etcd-restore
> ```
> 
> Then update the etcd static pod manifest to point to the new data directory.

**Q14. What happens to running workloads during a control plane upgrade?**

> Running pods on worker nodes continue operating. The control plane being upgraded causes a brief API server unavailability (seconds). New pod scheduling and API calls may fail temporarily.

**Q15. How do you handle a failed upgrade midway?**

> - Check `kubeadm upgrade plan` for state
> - Review kubelet and API server logs
> - If a node is stuck, manually fix kubeadm config: `kubeadm upgrade node --dry-run` to diagnose
> - Restore etcd from backup if cluster state is corrupted

-----

## 20. JSONPath & kubectl Mastery

**Q1. What is JSONPath in the context of kubectl?**

> JSONPath is a query language for extracting specific fields from JSON output. Used with `-o jsonpath='{...}'` to parse kubectl output programmatically.

**Q2. How do you get just the IP address of a pod?**

> ```
> kubectl get pod nginx -o jsonpath='{.status.podIP}'
> ```

**Q3. How do you get all pod names in a namespace?**

> ```
> kubectl get pods -o jsonpath='{.items[*].metadata.name}'
> # Newline-separated:
> kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}'
> ```

**Q4. How do you extract the image name of all containers across all pods?**

> ```
> kubectl get pods -A -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[*].image}{"\n"}{end}'
> ```

**Q5. How do you sort pods by restart count?**

> ```
> kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'
> ```

**Q6. How do you get node names along with their status?**

> ```
> kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.conditions[-1].type}{"\n"}{end}'
> ```

**Q7. How do you filter resources using `--field-selector`?**

> ```
> kubectl get pods --field-selector=status.phase=Running
> kubectl get pods --field-selector=spec.nodeName=worker-1
> kubectl get events --field-selector=type=Warning
> ```

**Q8. What is `kubectl explain` and when do you use it?**

> `kubectl explain` shows the schema and documentation for any Kubernetes resource field:
> 
> ```
> kubectl explain pod.spec.containers.resources
> kubectl explain deployment.spec.strategy --recursive
> ```
> 
> Essential for writing YAML without memorizing every field.

**Q9. How do you quickly generate a YAML manifest without creating the resource?**

> ```
> kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
> kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deploy.yaml
> kubectl create configmap my-config --from-literal=key=val --dry-run=client -o yaml
> ```

**Q10. How do you run a one-time command in the cluster without saving the pod?**

> ```
> kubectl run tmp --image=busybox --rm -it --restart=Never -- wget -qO- http://my-svc
> ```

**Q11. How do you use `kubectl get` with custom columns?**

> ```
> kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName
> ```

**Q12. How do you patch a resource quickly without editing YAML?**

> ```
> # Strategic merge patch
> kubectl patch deployment nginx -p '{"spec":{"replicas":5}}'
> # JSON patch
> kubectl patch pod nginx --type='json' -p='[{"op":"replace","path":"/spec/containers/0/image","value":"nginx:1.21"}]'
> ```

**Q13. What is `kubectl diff` used for?**

> Shows what changes would be made if `kubectl apply` were run — without actually applying them:
> 
> ```
> kubectl diff -f deployment.yaml
> ```

**Q14. How do you extract a specific context’s cluster server from kubeconfig using JSONPath?**

> ```
> kubectl config view -o jsonpath='{.clusters[?(@.name=="production")].cluster.server}'
> ```

**Q15. What are essential kubectl aliases and shortcuts for the CKA exam?**

> ```bash
> alias k=kubectl
> alias kgp='kubectl get pods'
> alias kgs='kubectl get svc'
> alias kgn='kubectl get nodes'
> export do='--dry-run=client -o yaml'
> export now='--grace-period=0 --force'
> 
> # Usage:
> k run nginx --image=nginx $do > pod.yaml
> k delete pod nginx $now
> ```

-----
