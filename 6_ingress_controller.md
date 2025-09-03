# 📘 Project 6: Ingress Controller  

By default, we used **NodePort** or **LoadBalancer Services** to expose apps.  
But when you have multiple apps, assigning different ports/IPs isn’t scalable.  

👉 **Ingress Controller** solves this by routing traffic to different apps based on hostnames or paths.  

---

## 🔹 Real-World Use Case  

Imagine hosting multiple apps:  

- `/app1` → App1 (Nginx)  
- `/app2` → App2 (httpd)  

Instead of separate NodePorts, you use a **single domain/IP** and let **Ingress route traffic**.  

---

## 🛠️ Part 1: Manual Steps (kubectl + YAML)  

### Step 1: Install NGINX Ingress Controller  

For **Minikube**:  

```sh
minikube addons enable ingress
For generic Kubernetes:


kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
Check pods:


kubectl get pods -n ingress-nginx
Step 2: Deploy Two Sample Apps


📄 app1-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - name: app1
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: app1-service
spec:
  selector:
    app: app1
  ports:
  - port: 80
    targetPort: 80
📄 app2-deployment.yaml


apiVersion: apps/v1
kind: Deployment
metadata:
  name: app2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app2
  template:
    metadata:
      labels:
        app: app2
    spec:
      containers:
      - name: app2
        image: httpd
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: app2-service
spec:
  selector:
    app: app2
  ports:
  - port: 80
    targetPort: 80


Apply:
kubectl apply -f app1-deployment.yaml
kubectl apply -f app2-deployment.yaml



Step 3: Create Ingress Resource


📄 apps-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apps-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80


Apply:
kubectl apply -f apps-ingress.yaml
Check ingress:


kubectl get ingress


Step 4: Access Apps
Get Minikube IP:


minikube ip
Open in browser:


http://<MINIKUBE_IP>/app1   → Nginx welcome page
http://<MINIKUBE_IP>/app2   → Apache httpd page
✅ Both apps are served via one IP using Ingress.

⚡ Part 2: Bash Script Automation
📄 k8s_ingress_setup.sh


#!/bin/bash

echo "🚀 Setting up Ingress Controller with 2 apps..."

# Step 1: Enable ingress (for Minikube)

if command -v minikube &> /dev/null; then
  minikube addons enable ingress
fi

# Step 2: App1 (Nginx)

kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - name: app1
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: app1-service
spec:
  selector:
    app: app1
  ports:
  - port: 80
    targetPort: 80
EOF

# Step 3: App2 (Apache httpd)

kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app2
  template:
    metadata:
      labels:
        app: app2
    spec:
      containers:
      - name: app2
        image: httpd
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: app2-service
spec:
  selector:
    app: app2
  ports:
  - port: 80
    targetPort: 80
EOF

# Step 4: Ingress

kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apps-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
EOF

echo "✅ Ingress setup complete!"


kubectl get ingress

Run:
bash k8s_ingress_setup.sh

✅ Key Takeaways
Ingress Controller = Entry point for external traffic

Ingress Resource = Defines routing rules (e.g., /app1, /app2)

Useful for multi-service apps (microservices)

Reduces need for multiple NodePorts/LoadBalancers
