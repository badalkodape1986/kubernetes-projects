# 📘 Project 1: Kubernetes Basics – Pods & Deployments

This project introduces the **core building blocks of Kubernetes**:

- **Pod** → Smallest deployable unit in Kubernetes.  
- **Deployment** → Manages replicas, scaling, and rolling updates.  
- **Service** → Exposes Pods inside the cluster.  

---

## 🔹 Real-World Use Case  

Deploy a simple **Nginx web server** inside Kubernetes.  

1. Create a **Pod** (manual).  
2. Create a **Deployment** to manage replicas.  
3. Expose it with a **Service (ClusterIP)** for access inside the cluster.  

---

## 🛠️ Part 1: Manual Steps (kubectl + YAML)  

### Step 1: Create a Pod  

**nginx-pod.yaml**  

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80


Apply:

kubectl apply -f nginx-pod.yaml
Verify:


kubectl get pods
kubectl describe pod nginx-pod
Step 2: Create a Deployment
A Deployment ensures multiple replicas and self-healing.

nginx-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
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
        image: nginx:latest
        ports:
        - containerPort: 80
Apply:

kubectl apply -f nginx-deployment.yaml
Verify:

kubectl get deployments
kubectl get pods -l app=nginx
Step 3: Expose Deployment with a Service
nginx-service.yaml

apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP


Apply:

kubectl apply -f nginx-service.yaml


Check:

kubectl get svc
kubectl describe svc nginx-service
Step 4: Access the Service Inside Cluster
Run a test pod:

kubectl run test-client --rm -it --image=busybox:1.28 sh
Inside the pod:


wget -qO- http://nginx-service
✅ You should see the Nginx welcome page HTML.

⚡ Part 2: Bash Script Automation
k8s_pod_deployment.sh


#!/bin/bash

echo "🚀 Deploying Nginx on Kubernetes..."

# Step 1: Pod
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
EOF

# Step 2: Deployment
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
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
        image: nginx:latest
        ports:
        - containerPort: 80
EOF

# Step 3: Service
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
EOF

echo "✅ Nginx Deployment + Service Created!"
kubectl get all -l app=nginx
Run:
bash k8s_pod_deployment.sh

✅ Key Takeaways
Pods = smallest deployable unit in Kubernetes.

Deployments = handle replicas, rollouts, and self-healing.

Services = provide stable networking inside the cluster.
