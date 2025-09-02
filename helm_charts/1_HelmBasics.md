📘 Project 1: Helm Basics – Nginx Helm Chart

Helm is the package manager for Kubernetes.
Instead of writing long YAMLs, Helm lets you package apps into charts with reusable templates.

🔹 Real-World Use Case

Instead of manually applying nginx-deployment.yaml and nginx-service.yaml,
we create a Helm chart → making it reusable, versioned, and parameterized.

🛠️ Part 1: Manual Steps
Step 1: Install Helm

Download Helm (Linux/macOS):

curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash


Verify:

helm version

Step 2: Create a Helm Chart
helm create nginx-chart


This creates a folder:

nginx-chart/
  Chart.yaml
  values.yaml
  templates/
    deployment.yaml
    service.yaml
    _helpers.tpl

Step 3: Edit values.yaml

Set defaults for Nginx:

replicaCount: 2

image:
  repository: nginx
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: NodePort
  port: 80
  nodePort: 30080

Step 4: Edit templates/deployment.yaml

Use Helm templating syntax:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-nginx
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}-nginx
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}-nginx
    spec:
      containers:
      - name: nginx
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: 80

Step 5: Edit templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-service
spec:
  type: {{ .Values.service.type }}
  selector:
    app: {{ .Release.Name }}-nginx
  ports:
  - port: {{ .Values.service.port }}
    targetPort: 80
    nodePort: {{ .Values.service.nodePort }}

Step 6: Install Chart
helm install my-nginx ./nginx-chart


Check:

kubectl get all


Access in browser:

http://<NodeIP>:30080


✅ You should see Nginx default welcome page.

Step 7: Upgrade Chart

Change replicas in values.yaml:

replicaCount: 4


Upgrade release:

helm upgrade my-nginx ./nginx-chart
kubectl get pods

Step 8: Uninstall Chart
helm uninstall my-nginx

⚡ Part 2: Bash Script Automation

Create helm_nginx.sh

#!/bin/bash

CHART_NAME="nginx-chart"
RELEASE_NAME="my-nginx"

echo "🚀 Creating Helm Chart for Nginx..."

# Step 1: Create chart
helm create $CHART_NAME

# Step 2: Update values.yaml
cat > $CHART_NAME/values.yaml <<EOF
replicaCount: 2

image:
  repository: nginx
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: NodePort
  port: 80
  nodePort: 30080
EOF

# Step 3: Deploy chart
helm install $RELEASE_NAME ./$CHART_NAME

echo "✅ Helm chart deployed! Access at http://<NodeIP>:30080"


Run:

bash helm_nginx.sh
