📘 Project 9: Kubernetes on AWS (EKS)

Amazon Elastic Kubernetes Service (EKS) is a managed Kubernetes service.
Instead of running your own control plane, AWS manages it for you.
You just manage worker nodes + deployments.

🔹 Real-World Use Case

Imagine your startup runs Kubernetes workloads in Minikube.
When moving to production, you need:

High availability (multi-AZ cluster)

AWS networking (VPC, Subnets, LoadBalancers)

Integration with AWS services (RDS, S3, CloudWatch)

👉 Solution: EKS – fully managed K8s on AWS.

🛠️ Part 1: Manual Steps (AWS Console + CLI)
Step 1: Create an EKS Cluster (Console)

Open AWS Console → EKS → Create Cluster.

Name: my-eks-cluster.

Kubernetes version: latest stable.

Networking: Choose VPC/Subnets/Security Groups.

Create IAM role for EKS (with AmazonEKSClusterPolicy).

Click Create.

Step 2: Create Worker Node Group

Go to Compute → Add Node Group.

Name: worker-nodes.

Attach IAM role (with AmazonEKSWorkerNodePolicy).

Choose EC2 type (t3.medium).

Desired nodes: 2.

Click Create.

Step 3: Configure kubectl for EKS

Install eksctl (recommended tool):

curl -sSL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin


Create EKS cluster with eksctl:

eksctl create cluster \
  --name my-eks-cluster \
  --version 1.28 \
  --region us-east-1 \
  --nodegroup-name workers \
  --node-type t3.medium \
  --nodes 2


Update kubeconfig:

aws eks update-kubeconfig --region us-east-1 --name my-eks-cluster


Verify:

kubectl get nodes


✅ You should see worker nodes registered.

Step 4: Deploy an Application

Use the Nginx Deployment from earlier projects:

nginx-deployment.yaml

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
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80


Apply:

kubectl apply -f nginx-deployment.yaml
kubectl get svc nginx-service


✅ AWS will provision an Elastic Load Balancer (ELB) → copy the EXTERNAL-IP and access in browser.

⚡ Part 2: Bash Script Automation

Create eks_deploy_nginx.sh

#!/bin/bash

CLUSTER_NAME="my-eks-cluster"
REGION="us-east-1"

echo "🚀 Deploying EKS cluster + Nginx app..."

# Create EKS cluster (2 worker nodes)
eksctl create cluster \
  --name $CLUSTER_NAME \
  --version 1.28 \
  --region $REGION \
  --nodegroup-name workers \
  --node-type t3.medium \
  --nodes 2

# Update kubeconfig
aws eks update-kubeconfig --region $REGION --name $CLUSTER_NAME

# Deploy Nginx app with LoadBalancer
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
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
EOF

echo "✅ Nginx deployed on EKS. Run: kubectl get svc nginx-service"


Run:

bash eks_deploy_nginx.sh
