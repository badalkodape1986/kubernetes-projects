📘 Project 8: Helm CI/CD with ArgoCD & Jenkins

We’ll combine:

Helm → Package apps into charts.

Jenkins → Automate build/test/push Helm charts to a Git repo or Helm repo.

ArgoCD → Continuously deploy Helm charts into Kubernetes from Git.

🔹 Real-World Use Case

Imagine a Node.js API managed with Helm:

Developers push code → Jenkins builds Docker image + updates Helm chart values (image tag).

Jenkins commits changes to a Git repo containing Helm charts.

ArgoCD watches the Git repo and automatically syncs Helm charts into Kubernetes.

👉 End result = Fully automated CI/CD GitOps pipeline with Helm.

🛠️ Part 1: Manual Steps
Step 1: Setup Jenkins in Kubernetes

Deploy Jenkins:

helm repo add jenkins https://charts.jenkins.io
helm repo update
helm install jenkins jenkins/jenkins --set controller.serviceType=NodePort --set controller.nodePort=30092


Check:

kubectl get pods
kubectl get svc jenkins


Access Jenkins:

http://<NodeIP>:30092


Get admin password:

kubectl exec -it <jenkins-pod> -- cat /var/jenkins_home/secrets/initialAdminPassword

Step 2: Setup ArgoCD

Install ArgoCD:

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


Expose ArgoCD UI:

kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort", "ports":[{"port":80,"targetPort":8080,"nodePort":30093}]}' 


Get ArgoCD admin password:

kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode ; echo


Access ArgoCD:

http://<NodeIP>:30093

Step 3: Jenkins Pipeline (CI)

Example Jenkinsfile:

pipeline {
    agent any
    stages {
        stage('Clone Repo') {
            steps {
                git 'https://github.com/<your-repo>/nodejs-app.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t my-dockerhub-username/nodejs-app:${BUILD_NUMBER} .'
            }
        }
        stage('Push Image') {
            steps {
                sh 'docker push my-dockerhub-username/nodejs-app:${BUILD_NUMBER}'
            }
        }
        stage('Update Helm Values') {
            steps {
                sh '''
                sed -i "s/tag:.*/tag: \\"${BUILD_NUMBER}\\"/" helm/nodejs/values.yaml
                git config --global user.email "ci@company.com"
                git config --global user.name "jenkins"
                git add helm/nodejs/values.yaml
                git commit -m "Update image tag to ${BUILD_NUMBER}"
                git push origin main
                '''
            }
        }
    }
}

Step 4: ArgoCD App (CD)

ArgoCD watches Helm chart repo.

argocd-app.yaml:

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nodejs-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/<your-repo>/helm-charts.git'
    targetRevision: main
    path: nodejs
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true


Apply:

kubectl apply -f argocd-app.yaml


✅ Now, Jenkins updates Helm values → pushes to Git → ArgoCD syncs → app auto-deploys.

⚡ Part 2: Bash Script Automation

Create helm_cicd.sh

#!/bin/bash

echo "🚀 Setting up Jenkins + ArgoCD CI/CD for Helm..."

# Step 1: Deploy Jenkins
helm repo add jenkins https://charts.jenkins.io
helm repo update
helm install jenkins jenkins/jenkins --set controller.serviceType=NodePort --set controller.nodePort=30092

# Step 2: Deploy ArgoCD
kubectl create namespace argocd || true
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort", "ports":[{"port":80,"targetPort":8080,"nodePort":30093}]}'

echo "✅ Jenkins: http://<NodeIP>:30092"
echo "✅ ArgoCD: http://<NodeIP>:30093"


Run:

bash helm_cicd.sh

✅ Key Takeaways

Jenkins handles CI → builds images, updates Helm values, pushes to Git.

ArgoCD handles CD → syncs Helm charts from Git into Kubernetes.

Together, they enable GitOps-driven CI/CD for Helm.
