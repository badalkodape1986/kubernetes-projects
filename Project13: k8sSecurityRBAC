# 📘 Project 13: Kubernetes Security & RBAC

Security in Kubernetes is critical for protecting clusters, workloads, and sensitive data.  
This project focuses on **Role-Based Access Control (RBAC), Service Accounts, and Network Policies**.

- **RBAC** → Controls who can access what  
- **Service Accounts** → Identity for pods to access the API  
- **Network Policies** → Control pod-to-pod and pod-to-external communication  

---

## 🔹 Real-World Use Case

Imagine you’re running a **multi-team Kubernetes cluster**:

- Developers should **only view** logs & pods in their namespace  
- CI/CD pipelines should use **Service Accounts** for deployments  
- Sensitive services (e.g., `db`) should **not be reachable** from all pods  

👉 Solution: Use **RBAC + Service Accounts + Network Policies**

---

## 🛠️ Part 1: Manual Steps (RBAC + Service Accounts + Network Policies)

### Step 1: Create a Namespace for a Team

```sh
kubectl create namespace dev-team


Step 2: Create a Role (view-only access)
📄 role-viewer.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev-team
  name: pod-viewer
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]


Apply:
kubectl apply -f role-viewer.yaml

Step 3: Bind Role to a User
📄 rolebinding.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-viewer-binding
  namespace: dev-team
subjects:
- kind: User
  name: dev-user   # Replace with actual username
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-viewer
  apiGroup: rbac.authorization.k8s.io


Apply:
kubectl apply -f rolebinding.yaml
✅ Now dev-user can only view pods/logs in dev-team namespace

Step 4: Create a Service Account for CI/CD
📄 serviceaccount.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: cicd-bot
  namespace: dev-team


📄 rolebinding-cicd.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cicd-deployer-binding
  namespace: dev-team
subjects:
- kind: ServiceAccount
  name: cicd-bot
  namespace: dev-team
roleRef:
  kind: ClusterRole
  name: edit   # "edit" allows create/update/delete in namespace
  apiGroup: rbac.authorization.k8s.io


Apply:
kubectl apply -f serviceaccount.yaml
kubectl apply -f rolebinding-cicd.yaml
✅ Now cicd-bot ServiceAccount can be used in Jenkins/ArgoCD for deployments

Step 5: Apply Network Policies
📄 networkpolicy.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: dev-team
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
This denies all traffic by default in dev-team namespace.

📄 allow-frontend-to-backend.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: dev-team
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80


Apply:
kubectl apply -f networkpolicy.yaml
kubectl apply -f allow-frontend-to-backend.yaml
✅ Only pods with label app=frontend can talk to app=backend

⚡ Part 2: Bash Script Automation

📄 k8s_security_rbac.sh

#!/bin/bash

echo "🚀 Setting up Kubernetes Security (RBAC + Service Accounts + Network Policies)..."

# Step 1: Create Namespace
kubectl create namespace dev-team

# Step 2: Role (view-only access)
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev-team
  name: pod-viewer
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
EOF

# Step 3: RoleBinding for a user
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-viewer-binding
  namespace: dev-team
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-viewer
  apiGroup: rbac.authorization.k8s.io
EOF

# Step 4: ServiceAccount for CI/CD
kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cicd-bot
  namespace: dev-team
EOF

kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cicd-deployer-binding
  namespace: dev-team
subjects:
- kind: ServiceAccount
  name: cicd-bot
  namespace: dev-team
roleRef:
  kind: ClusterRole
  name: edit
  apiGroup: rbac.authorization.k8s.io
EOF

# Step 5: Network Policies
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: dev-team
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
EOF

kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: dev-team
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
EOF

echo "✅ RBAC, ServiceAccount, and Network Policies configured!"


Run:
bash k8s_security_rbac.sh
🎯 Final Outcome
✅ RBAC enforces least-privilege access for users

✅ Service Accounts for CI/CD pipelines

✅ Network Policies enforce zero-trust networking

✅ Secure multi-team Kubernetes cluster
