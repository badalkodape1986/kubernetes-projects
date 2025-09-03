# 📘 Project 5: Storage & Persistence – WordPress + MySQL on Kubernetes  

Kubernetes Pods are **ephemeral** → if a Pod dies, its data is lost.  

To persist data (like MySQL DB or WordPress files), we use:  

- **PersistentVolume (PV):** Actual storage (local disk, NFS, EBS, etc.)  
- **PersistentVolumeClaim (PVC):** Request for storage by a Pod  

---

## 🔹 Real-World Use Case  

Deploy a **WordPress blog with MySQL**:  

- MySQL stores data in a PVC  
- WordPress stores uploads/themes in a PVC  
- Even if Pods restart, the data persists  

---

## 🛠️ Part 1: Manual Deployment (kubectl + YAML)  

### Step 1: Create Persistent Volumes  

For demo, we use **hostPath** (works in Minikube).  

📄 **pv.yaml**  

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data/mysql"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: wordpress-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data/wordpress"


Apply:
kubectl apply -f pv.yaml


Step 2: Create PVCs
📄 pvc.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wordpress-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi


Apply:
kubectl apply -f pvc.yaml

Check:
kubectl get pv,pvc


Step 3: Deploy MySQL
📄 mysql-deployment.yaml

apiVersion: v1
kind: Secret
metadata:
  name: mysql-pass
type: Opaque
data:
  password: cGFzc3dvcmQ=   # "password" base64
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql:5.7
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-storage
        persistentVolumeClaim:
          claimName: mysql-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  ports:
  - port: 3306
  selector:
    app: mysql


Apply:
kubectl apply -f mysql-deployment.yaml


Step 4: Deploy WordPress
📄 wordpress-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - image: wordpress:latest
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: mysql-service
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 80
        volumeMounts:
        - name: wordpress-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-storage
        persistentVolumeClaim:
          claimName: wordpress-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: wordpress-service
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30008
  selector:
    app: wordpress


Apply:
kubectl apply -f wordpress-deployment.yaml


Step 5: Access WordPress
Get Node IP:
kubectl get nodes -o wide
Open in browser:


http://<NodeIP>:30008
✅ You should see the WordPress installation page

⚡ Part 2: Bash Script Automation
📄 k8s_wordpress_mysql.sh


#!/bin/bash

echo "🚀 Deploying WordPress + MySQL with Persistent Storage..."



# Step 1: PVs

kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data/mysql"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: wordpress-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data/wordpress"
EOF


# Step 2: PVCs

kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wordpress-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
EOF

# Step 3: Secret for MySQL

kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: mysql-pass
type: Opaque
data:
  password: cGFzc3dvcmQ=   # "password"
EOF


# Step 4: MySQL Deployment + Service

kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql:5.7
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-storage
        persistentVolumeClaim:
          claimName: mysql-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  ports:
  - port: 3306
  selector:
    app: mysql
EOF

# Step 5: WordPress Deployment + Service

kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - image: wordpress:latest
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: mysql-service
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 80
        volumeMounts:
        - name: wordpress-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-storage
        persistentVolumeClaim:
          claimName: wordpress-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: wordpress-service
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30008
  selector:
    app: wordpress
EOF

echo "✅ WordPress is available at http://<NodeIP>:30008"
kubectl get svc wordpress-service

Run:
bash k8s_wordpress_mysql.sh
🎯 Final Outcome
✅ Persistent Storage ensures WordPress uploads and MySQL data survive Pod restarts

✅ Manual + Automated Deployments for real-world reproducibility

✅ NodePort Service exposes WordPress at:


http://<NodeIP>:30008
