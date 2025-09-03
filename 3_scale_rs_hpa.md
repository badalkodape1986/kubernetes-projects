# 📘 Project 3: Scaling & Self-Healing – ReplicaSets & HPA

This project focuses on **scalability and fault-tolerance in Kubernetes**.

- **ReplicaSet** → Ensures a specified number of Pods are always running.  
- **Horizontal Pod Autoscaler (HPA)** → Scales Pods up/down automatically based on CPU/Memory usage.  

---

## 🔹 Real-World Use Case  

Imagine your **Nginx app** from the last project:  

- During **low traffic**, 2 Pods are enough.  
- During **peak hours**, you need 10 Pods.  

Instead of scaling manually → **HPA auto-scales Pods** based on CPU load.  

---

## 🛠️ Part 1: Manual Steps (kubectl + YAML)  

### Step 1: Create a ReplicaSet  

**nginx-replicaset.yaml**  

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
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
kubectl apply -f nginx-replicaset.yaml



Verify:
kubectl get rs
kubectl get pods -l app=nginx
✅ If you delete a pod, ReplicaSet will create a new one automatically.



Step 2: Deploy Nginx with a Deployment (better than ReplicaSet)
ReplicaSets are usually managed by Deployments (which also support rolling updates).

nginx-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
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
        resources:
          limits:
            cpu: "200m"
            memory: "256Mi"
          requests:
            cpu: "100m"
            memory: "128Mi"



Apply:
kubectl apply -f nginx-deployment.yaml




Step 3: Enable Metrics Server
HPA needs metrics-server to monitor CPU usage.

Install (on Minikube or any cluster):

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml


Verify:
kubectl top nodes
kubectl top pods




Step 4: Create Horizontal Pod Autoscaler (HPA)
nginx-hpa.yaml

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50



Apply:
kubectl apply -f nginx-hpa.yaml



Check status:
kubectl get hpa


Step 5: Simulate Load
Run a busybox container to generate traffic:
kubectl run -i --tty load-generator --image=busybox /bin/sh

Inside container:
while true; do wget -q -O- http://nginx-deployment; done
Check scaling:


kubectl get hpa
kubectl get pods -l app=nginx
✅ You’ll see Pods increasing when CPU usage rises.


⚡ Part 2: Bash Script Automation


k8s_hpa_setup.sh

#!/bin/bash

echo "🚀 Deploying Nginx with HPA..."

# Step 1: Deployment
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
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
        resources:
          limits:
            cpu: "200m"
            memory: "256Mi"
          requests:
            cpu: "100m"
            memory: "128Mi"
EOF

# Step 2: HPA
kubectl apply -f - <<EOF
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
EOF


echo "✅ Nginx HPA Created!"
kubectl get hpa


Run:
bash k8s_hpa_setup.sh


✅ Key Takeaways
ReplicaSets maintain pod count, but Deployments manage them better (rolling updates).

Metrics Server is required for HPA to work.

HPA scales Pods dynamically based on resource utilization.

Autoscaling ensures high availability during traffic spikes and cost efficiency during low load.
