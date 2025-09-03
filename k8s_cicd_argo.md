# 📘 Project 8: CI/CD on Kubernetes – Jenkins + ArgoCD  

We’ll use:  

- **Jenkins** → Automates build/test/push steps  
- **ArgoCD (GitOps)** → Syncs Kubernetes manifests from GitHub repo to the cluster  

---

## 🔹 Real-World Use Case  

Imagine you have a **Node.js API running in Kubernetes**:  

- When developers **push code** → Jenkins builds a **Docker image** & pushes it to **DockerHub/ECR**  
- Updated Kubernetes manifests (Deployment) are committed to **Git**  
- ArgoCD continuously syncs **cluster state with Git** → ensuring a **GitOps workflow**  

---

## 🛠️ Part 1: Manual Setup  

### Step 1: Deploy Jenkins in Kubernetes  

📄 **jenkins-deployment.yaml**  

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
      - name: jenkins
        image: jenkins/jenkins:lts
        ports:
        - containerPort: 8080
        - containerPort: 50000
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins-service
spec:
  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30009
  selector:
    app: jenkins


Apply:
kubectl apply -f jenkins-deployment.yaml


Access Jenkins:
http://<NodeIP>:30009
Get admin password:


kubectl exec -it <jenkins-pod> -- cat /var/jenkins_home/secrets/initialAdminPassword


Step 2: Configure Jenkins Pipeline
Install plugins:

Docker

Kubernetes CLI

GitHub

Pipeline example (Jenkinsfile):

pipeline {
    agent any
    stages {
        stage('Clone Repo') {
            steps {
                git 'https://github.com/<your-repo>/node-app.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t <your-dockerhub-username>/node-app:latest .'
            }
        }
        stage('Push Image') {
            steps {
                sh 'docker push <your-dockerhub-username>/node-app:latest'
            }
        }
        stage('Update K8s Manifests') {
            steps {
                sh '''
                sed -i "s|image: .*|image: <your-dockerhub-username>/node-app:latest|" k8s/deployment.yaml
                git config --global user.email "ci@example.com"
                git config --global user.name "jenkins"
                git commit -am "Updated image"
                git push origin main
                '''
            }
        }
    }
}


Step 3: Install ArgoCD

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
Expose ArgoCD UI (NodePort for demo):


kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
kubectl get svc -n argocd


Login to ArgoCD:
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode ; echo


Access in browser:
http://<NodeIP>:<NodePort>

Step 4: Create ArgoCD App
Point ArgoCD to your GitHub repo with Kubernetes manifests.


📄 argocd-app.yaml

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: node-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/<your-repo>/k8s-manifests.git'
    targetRevision: main
    path: .
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true


Apply:
kubectl apply -f argocd-app.yaml
✅ ArgoCD will sync cluster state with Git automatically


⚡ Part 2: Bash Script Automation
📄 k8s_cicd_setup.sh

#!/bin/bash

echo "🚀 Setting up Jenkins + ArgoCD CI/CD Pipeline..."

# Deploy Jenkins
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
      - name: jenkins
        image: jenkins/jenkins:lts
        ports:
        - containerPort: 8080
        - containerPort: 50000
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins-service
spec:
  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30009
  selector:
    app: jenkins
EOF

# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'

echo "✅ Jenkins at http://<NodeIP>:30009"
echo "✅ ArgoCD at http://<NodeIP>:<NodePort>"
Run:


bash k8s_cicd_setup.sh


🎯 Final Outcome
✅ Jenkins builds & pushes Docker images automatically

✅ Kubernetes manifests are updated & pushed to Git

✅ ArgoCD ensures the cluster is always in sync with Git (GitOps)

✅ Fully automated CI/CD pipeline on Kubernetes
