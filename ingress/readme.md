
# 🌐 Kubernetes Ingress Basic Commands

```bash
# Step 1: Install NGINX Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.15.1/deploy/static/provider/aws/deploy.yaml

Need to add this in service annotation

service.beta.kubernetes.io/aws-load-balancer-scheme: internet-facing

# Verify controller pods
kubectl get pods -n ingress-nginx
```

```bash
# Step 2: Create a sample deployment
kubectl create deployment web-deploy --image=nginx -n demo

# Expose deployment as a service
kubectl expose deployment web-deploy --port=80 --target-port=80 -n demo
```

```bash
# Step 3: Create ingress resource
kubectl create ingress web-ingress \
  --rule="myapp.local/*=web-deploy:80" -n demo
```

```bash
# Step 4: Verify ingress
kubectl get ingress -n demo
kubectl get ingressclass​
kubectl describe ingress web-ingress -n demo
```

```bash
# Step 5: Test ingress
# Add entry in /etc/hosts: <INGRESS_CONTROLLER_IP> myapp.local
curl http://myapp.local
```

```bash
# Step 6: Useful checks
kubectl get ingress -A
kubectl describe ingress -n demo
kubectl logs -n ingress-nginx deploy/ingress-nginx-controller
```
## Connect to an EKS Cluster

### 1. Configure AWS credentials

Configure your AWS credentials:

```bash
aws configure
```

Verify the configured credentials:

```bash
aws sts get-caller-identity
```

### 2. Update kubeconfig for the EKS cluster

Update your local kubeconfig with the EKS cluster details:

```bash
aws eks update-kubeconfig --region <region> --name <cluster-name>
```

Example:

```bash
aws eks update-kubeconfig --region ap-south-1 --name my-eks-cluster
```

### 3. Verify the connection

Verify that you can connect to the EKS cluster:

```bash
kubectl get nodes
```

You can also verify the current Kubernetes context:

```bash
kubectl config current-context
```
