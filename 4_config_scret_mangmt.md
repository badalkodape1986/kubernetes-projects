# 📘 Project 4: Config & Secrets Management

Kubernetes provides two key resources for managing app data:

- **ConfigMap** → Stores non-sensitive configuration (e.g., DB host, app settings).  
- **Secret** → Stores sensitive information (e.g., DB passwords, API keys).  

---

## 🔹 Real-World Use Case  

Imagine deploying a **MySQL database and a backend application**:  

- Instead of hardcoding database details in YAML,  
- Use **ConfigMaps** for DB host/username and **Secrets** for DB password.  

👉 This makes your app **portable, secure, and manageable**.  

---

## 🛠️ Part 1: Manual Steps (kubectl + YAML)  

### Step 1: Create a ConfigMap  

**db-config.yaml**  

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-config
data:
  DB_HOST: mysql-service
  DB_USER: admin


Apply:
kubectl apply -f db-config.yaml


Check:
kubectl get configmap db-config -o yaml



Step 2: Create a Secret

db-secret.yaml

apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQxMjM=   # base64 of "password123"
⚡ Encode secret with:


echo -n "password123" | base64


Apply:
kubectl apply -f db-secret.yaml


Check:
kubectl get secret db-secret -o yaml



Step 3: Use ConfigMap & Secret in a Deployment

app-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: nginx:latest
        env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: db-config
              key: DB_HOST
        - name: DB_USER
          valueFrom:
            configMapKeyRef:
              name: db-config
              key: DB_USER
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: DB_PASSWORD


Apply:
kubectl apply -f app-deployment.yaml


Verify environment variables:
kubectl exec -it <backend-pod-name> -- env | grep DB_
✅ You should see DB_HOST, DB_USER, and DB_PASSWORD.

Step 4: (Optional) Mount Config & Secret as Files
You can also mount them inside containers:

        volumeMounts:
        - name: config-volume
          mountPath: /etc/config
        - name: secret-volume
          mountPath: /etc/secret
      volumes:
      - name: config-volume
        configMap:
          name: db-config
      - name: secret-volume
        secret:
          secretName: db-secret



⚡ Part 2: Bash Script Automation


k8s_config_secrets.sh

#!/bin/bash

echo "🚀 Creating ConfigMap and Secret for Backend App..."

# Step 1: ConfigMap
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-config
data:
  DB_HOST: mysql-service
  DB_USER: admin
EOF

# Step 2: Secret
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQxMjM=   # "password123"
EOF

# Step 3: Deployment using Config & Secret
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: nginx:latest
        env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: db-config
              key: DB_HOST
        - name: DB_USER
          valueFrom:
            configMapKeyRef:
              name: db-config
              key: DB_USER
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: DB_PASSWORD
EOF

echo "✅ ConfigMap + Secret + Deployment Created!"
kubectl get configmap,secret,deployment


Run:
bash k8s_config_secrets.sh


✅ Key Takeaways
ConfigMaps → non-sensitive app configs.

Secrets → sensitive data (must be base64 encoded).

Avoid hardcoding credentials → use Kubernetes-native resources.

Configs and Secrets can be mounted as env vars or files.
