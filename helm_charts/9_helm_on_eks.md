📘 Project 9: Helm on AWS EKS

We’ll deploy a Node.js API + PostgreSQL database on AWS EKS using Helm.

EKS provides the managed Kubernetes control plane.

Helm packages our workloads into reusable charts.

AWS services (EBS for storage, ALB for ingress) handle infra.

🔹 Real-World Use Case

You’re migrating a microservice (Node.js + PostgreSQL) from Minikube to AWS EKS.

Helm makes the chart portable.

EKS ensures scalability & AWS-native networking.

AWS Load Balancer exposes your service to the internet.

🛠️ Part 1: Manual Steps
Step 1: Create EKS Cluster

Install eksctl:

curl -sSL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin


Create cluster:

eksctl create cluster \
  --name helm-eks-cluster \
  --version 1.28 \
  --region us-east-1 \
  --nodegroup-name workers \
  --node-type t3.medium \
  --nodes 2


Update kubeconfig:

[Oaws eks update-kubeconfig --region us-east-1 --name helm-eks-cluster
kubectl get nodes

Step 2: Create Helm Chart
helm create node-postgres-chart
cd node-postgres-chart

Step 3: Define Values (values.yaml)
nodeapp:
  replicaCount: 2
  image:
    repository: my-dockerhub-username/nodejs-api
    tag: latest
  service:
    type: LoadBalancer
    port: 80

postgres:
  auth:
    username: admin
    password: supersecret
    database: mydb
  primary:
    persistence:
      enabled: true
      size: 5Gi
      storageClass: gp2

Step 4: Deployment Template (templates/nodeapp-deployment.yaml)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-nodeapp
spec:
  replicas: {{ .Values.nodeapp.replicaCount }}
  selector:
    matchLabels:
      app: nodeapp
  template:
    metadata:
      labels:
        app: nodeapp
    spec:
      containers:
      - name: nodeapp
        image: "{{ .Values.nodeapp.image.repository }}:{{ .Values.nodeapp.image.tag }}"
        ports:
        - containerPort: 3000
        env:
        - name: DB_HOST
          value: {{ .Release.Name }}-postgresql
        - name: DB_USER
          value: {{ .Values.postgres.auth.username }}
        - name: DB_PASSWORD
          value: {{ .Values.postgres.auth.password }}
        - name: DB_NAME
          value: {{ .Values.postgres.auth.database }}
---
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-nodeapp-service
spec:
  type: {{ .Values.nodeapp.service.type }}
  selector:
    app: nodeapp
  ports:
  - port: {{ .Values.nodeapp.service.port }}
    targetPort: 3000

Step 5: Add PostgreSQL as Dependency

Chart.yaml

dependencies:
  - name: postgresql
    version: "12.5.0"
    repository: "https://charts.bitnami.com/bitnami"


Update:

helm dependency update

Step 6: Install Chart on EKS
helm install node-postgres ./node-postgres-chart
kubectl get pods
kubectl get svc node-postgres-nodeapp-service


AWS will provision an ELB → copy EXTERNAL-IP.

Access API:

http://<ELB-DNS>:80


✅ Node.js API connected to PostgreSQL inside EKS.

⚡ Part 2: Bash Script Automation

Create helm_eks.sh

#!/bin/bash

CLUSTER_NAME="helm-eks-cluster"
REGION="us-east-1"
CHART_NAME="node-postgres-chart"
RELEASE_NAME="node-postgres"

echo "🚀 Deploying Node.js + PostgreSQL on AWS EKS with Helm..."

# Step 1: Create EKS cluster
eksctl create cluster \
  --name $CLUSTER_NAME \
  --version 1.28 \
  --region $REGION \
  --nodegroup-name workers \
  --node-type t3.medium \
  --nodes 2

# Step 2: Update kubeconfig
aws eks update-kubeconfig --region $REGION --name $CLUSTER_NAME

# Step 3: Create Helm chart
helm create $CHART_NAME
cd $CHART_NAME

# Step 4: Add PostgreSQL dependency
cat >> Chart.yaml <<EOF
dependencies:
  - name: postgresql
    version: "12.5.0"
    repository: "https://charts.bitnami.com/bitnami"
EOF

helm dependency update

# Step 5: Replace values.yaml
cat > values.yaml <<EOF
nodeapp:
  replicaCount: 2
  image:
    repository: my-dockerhub-username/nodejs-api
    tag: latest
  service:
    type: LoadBalancer
    port: 80

postgres:
  auth:
    username: admin
    password: supersecret
    database: mydb
  primary:
    persistence:
      enabled: true
      size: 5Gi
      storageClass: gp2
EOF

cd ..

# Step 6: Install chart
helm install $RELEASE_NAME ./$CHART_NAME

echo "✅ Deployment complete. Run 'kubectl get svc $RELEASE_NAME-nodeapp-service'"


Run:

bash helm_eks.sh

✅ Key Takeaways

Helm + EKS = production-ready deployments.

LoadBalancer services in Helm charts create AWS ELBs automatically.

gp2 storage class provisions AWS EBS volumes for Postgres.

Dependency management simplifies multi-service apps (Node.js + PostgreSQL).
