# 📘 Project 11: Kubernetes Troubleshooting  

This project focuses on debugging **common Kubernetes issues**:  

- Pods not starting  
- Crashes/restarts (**CrashLoopBackOff**)  
- Services not reachable  
- Persistent Volumes not mounting  
- Misconfigured Ingress/Network  

We’ll **simulate failures** and learn how to fix them.  

---

## 🔹 Real-World Use Case  

Imagine you deployed an app on **EKS/Minikube**, but:  

- Pod is stuck in **Pending** → Wrong PVC or no nodes  
- Pod is in **CrashLoopBackOff** → App failing to start  
- Service exists but **can’t connect** → Wrong selector labels  
- Ingress not routing → **Misconfigured host/path**  

👉 As an **SRE/DevOps engineer**, you must **detect & resolve** such issues quickly.  

---

## 🛠️ Part 1: Common Troubleshooting Scenarios  

### 🔹 Scenario 1: Pod stuck in ImagePullBackOff  

📄 **bad-pod.yaml**  

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: bad-pod
spec:
  containers:
  - name: bad-container
    image: nginx:doesnotexist   # Wrong image tag
    ports:
    - containerPort: 80


Deploy:
kubectl apply -f bad-pod.yaml
kubectl get pods


Fix:
kubectl describe pod bad-pod   # See ImagePull error
kubectl delete pod bad-pod
kubectl run good-pod --image=nginx:latest --port=80
🔹 Scenario 2: Pod in CrashLoopBackOff


📄 crash-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: crash-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "exit 1"]   # Container exits immediately


Deploy:
kubectl apply -f crash-pod.yaml
kubectl get pods


Fix:
kubectl logs crash-pod
kubectl describe pod crash-pod
kubectl run fixed-pod --image=busybox --command -- sleep 3600
🔹 Scenario 3: Service not working


📄 wrong-service.yaml

apiVersion: v1
kind: Service
metadata:
  name: wrong-service
spec:
  selector:
    app: wronglabel   # Label mismatch
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80


Check:
kubectl describe svc wrong-service
kubectl get endpoints wrong-service


Fix:
kubectl edit svc wrong-service   # Fix selector → correct label
kubectl get endpoints wrong-service   # Should now show pod IPs
🔹 Scenario 4: PVC not binding



📄 bad-pvc.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: bad-pvc
spec:
  accessModes:
    - ReadWriteMany   # But PV only supports ReadWriteOnce
  resources:
    requests:
      storage: 1Gi


Check:
kubectl describe pvc bad-pvc
kubectl get pv


Fix:
Change ReadWriteMany → ReadWriteOnce

Or create an NFS PV that supports RWX

🔹 Scenario 5: Ingress not routing
📄 bad-ingress.yaml


apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: bad-ingress
spec:
  rules:
  - host: wrong.local    # Host mismatch
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80


Check:
kubectl describe ingress bad-ingress


Fix:

Ensure host matches your DNS/minikube IP

Or test with /etc/hosts entry:

<NodeIP> myapp.local



⚡ Part 2: Bash Script to Simulate Troubleshooting


📄 k8s_troubleshooting.sh

#!/bin/bash

echo "🚨 Simulating Kubernetes Troubleshooting Scenarios..."

# Scenario 1: ImagePullBackOff
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: bad-pod
spec:
  containers:
  - name: bad-container
    image: nginx:doesnotexist
    ports:
    - containerPort: 80
EOF

# Scenario 2: CrashLoopBackOff
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: crash-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "exit 1"]
EOF

# Scenario 3: Wrong Service
kubectl run app --image=nginx --labels="app=myapp" --port=80
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: wrong-service
spec:
  selector:
    app: wronglabel
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
EOF

# Scenario 4: Bad PVC
kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: bad-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
EOF

# Scenario 5: Bad Ingress
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: bad-ingress
spec:
  rules:
  - host: wrong.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: wrong-service
            port:
              number: 80
EOF

echo "✅ Troubleshooting scenarios created! Now run:"
echo "kubectl describe pod bad-pod"
echo "kubectl logs crash-pod"
echo "kubectl describe svc wrong-service"
echo "kubectl describe pvc bad-pvc"
echo "kubectl describe ingress bad-ingress"


Run:
bash k8s_troubleshooting.sh


🎯 Final Outcome
✅ Learn to debug Pods stuck in Pending, ImagePullBackOff, CrashLoopBackOff

✅ Fix Service selector mismatches

✅ Troubleshoot PVC binding issues

✅ Debug Ingress misconfiguration

✅ Hands-on practice as an SRE/DevOps engineer
