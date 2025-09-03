# 📘 Project 2: Exposing Applications – Services & NodePort

By default, a **ClusterIP Service** is only accessible **inside the cluster**.  
To make it available to **external users**, we use a **NodePort Service**.

---

## 🔹 Real-World Use Case  

Imagine your **Nginx app** running in Kubernetes:  

- Other Pods can access it via **ClusterIP** (from **Project 1**).  
- But users outside the cluster (like your browser) **cannot**.  

👉 **Solution**: Expose the Deployment with a **NodePort** → accessible at: `http://<NodeIP>:<NodePort>`



---

## 🛠️ Part 1: Manual Steps (kubectl + YAML)  

### Step 1: Deployment (reuse from Project 1)  

**nginx-deployment.yaml**  

```yaml
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
Step 2: Create a NodePort Service
nginx-nodeport.yaml


apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080   # custom port (range: 30000–32767)
Apply:


kubectl apply -f nginx-nodeport.yaml
Verify:


kubectl get svc nginx-nodeport
Output example:


NAME             TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx-nodeport   NodePort   10.96.25.140   <none>        80:30080/TCP   5s
Step 3: Access Application from Browser
Get the Node IP:


kubectl get nodes -o wide
Open in browser:


http://<NodeIP>:30080
✅ You should see the Nginx welcome page.

⚡ Part 2: Bash Script Automation
k8s_nodeport_service.sh


#!/bin/bash

echo "🚀 Deploying Nginx with NodePort Service..."

# Step 1: Deployment
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

# Step 2: NodePort Service
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
EOF

echo "✅ Nginx exposed at NodePort 30080"
kubectl get svc nginx-nodeport

Run:

bash k8s_nodeport_service.sh
✅ Key Takeaways
ClusterIP → internal-only access (default).

NodePort → exposes app on <NodeIP>:<NodePort>.

NodePort range is 30000–32767.

Great for testing, but in production we prefer LoadBalancer or Ingress.
