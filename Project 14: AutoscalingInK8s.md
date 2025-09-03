# 📘 Project 14: Autoscaling in Kubernetes

Kubernetes provides multiple scaling options to ensure applications handle varying workloads efficiently:

- **Horizontal Pod Autoscaler (HPA)** → Scales pods based on CPU/Memory usage  
- **Vertical Pod Autoscaler (VPA)** → Adjusts resource requests/limits for pods  
- **Cluster Autoscaler** → Scales worker nodes in cloud providers like AWS/GCP  

---

## 🔹 Real-World Use Case

Imagine running an **e-commerce site**:  
- During sales, traffic spikes → HPA automatically adds pods  
- Under-utilized workloads → VPA reduces requests  
- Cluster out of capacity → Cluster Autoscaler provisions new nodes  

---

## 🛠️ Part 1: Manual Setup

### Step 1: Deploy Metrics Server (for HPA)

```sh
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl get pods -n kube-system | grep metrics-server


Step 2: Deploy Sample App
📄 nginx-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
Apply:
kubectl apply -f nginx-deployment.yaml


Step 3: Enable HPA
kubectl autoscale deployment nginx-deployment --cpu-percent=50 --min=1 --max=5
kubectl get hpa

Step 4: Install VPA (optional)
kubectl apply -f https://github.com/kubernetes/autoscaler/releases/latest/download/vertical-pod-autoscaler.yaml

Step 5: Enable Cluster Autoscaler (on AWS EKS / GCP GKE)
For AWS EKS:
kubectl apply -f https://github.com/kubernetes/autoscaler/releases/latest/download/cluster-autoscaler-autodiscover.yaml

🎯 Final Outcome
✅ HPA auto-scales pods

✅ VPA tunes pod resources

✅ Cluster Autoscaler scales nodes on demand
