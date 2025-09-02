📘 Project 2: Parameterizing Deployments with values.yaml

This project focuses on Helm templating:

Define configurable parameters in values.yaml.

Use those parameters inside Deployment and Service templates.

Deploy a Node.js app with different environments (dev, staging, prod).

🔹 Real-World Use Case

Instead of hardcoding things like:

image name

replica count

environment variables (DB_URL, API_KEY, etc.)

👉 We use values.yaml so the same Helm chart can deploy:

Dev: 1 replica, debug mode.

Prod: 5 replicas, optimized configs.

🛠️ Part 1: Manual Steps
Step 1: Create a Helm Chart
helm create nodejs-chart
cd nodejs-chart

Step 2: Define Parameters in values.yaml

Update values.yaml:

replicaCount: 2

image:
  repository: my-dockerhub-username/node-app
  tag: "latest"
  pullPolicy: IfNotPresent

service:
  type: NodePort
  port: 80
  nodePort: 30081

env:
  - name: NODE_ENV
    value: "production"
  - name: DB_HOST
    value: "mysql-service"
  - name: API_KEY
    value: "default-api-key"

Step 3: Update templates/deployment.yaml

Add env vars and use .Values:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-node
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}-node
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}-node
    spec:
      containers:
      - name: node-app
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - containerPort: 3000
        env:
{{- range .Values.env }}
        - name: {{ .name }}
          value: "{{ .value }}"
{{- end }}

Step 4: Update templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-node-service
spec:
  type: {{ .Values.service.type }}
  selector:
    app: {{ .Release.Name }}-node
  ports:
  - port: {{ .Values.service.port }}
    targetPort: 3000
    nodePort: {{ .Values.service.nodePort }}

Step 5: Install Chart
helm install my-node ./nodejs-chart
kubectl get pods
kubectl get svc


Access:

http://<NodeIP>:30081

Step 6: Override Values for Different Environments

For dev:

values-dev.yaml

replicaCount: 1
env:
  - name: NODE_ENV
    value: "development"
  - name: DB_HOST
    value: "dev-db"
  - name: API_KEY
    value: "dev-api-key"


Install with dev values:

helm install my-node-dev ./nodejs-chart -f values-dev.yaml


For prod:

values-prod.yaml

replicaCount: 5
env:
  - name: NODE_ENV
    value: "production"
  - name: DB_HOST
    value: "prod-db"
  - name: API_KEY
    value: "prod-api-key"


Install with prod values:

helm install my-node-prod ./nodejs-chart -f values-prod.yaml


✅ Same chart → multiple environments.

⚡ Part 2: Bash Script Automation

Create helm_nodejs.sh

#!/bin/bash

CHART_NAME="nodejs-chart"
RELEASE_NAME="my-node"

echo "🚀 Creating Node.js Helm Chart with parameterized values..."

# Step 1: Create chart
helm create $CHART_NAME

# Step 2: Replace values.yaml
cat > $CHART_NAME/values.yaml <<EOF
replicaCount: 2

image:
  repository: my-dockerhub-username/node-app
  tag: "latest"
  pullPolicy: IfNotPresent

service:
  type: NodePort
  port: 80
  nodePort: 30081

env:
  - name: NODE_ENV
    value: "production"
  - name: DB_HOST
    value: "mysql-service"
  - name: API_KEY
    value: "default-api-key"
EOF

# Step 3: Deploy chart
helm install $RELEASE_NAME ./$CHART_NAME

echo "✅ Node.js app deployed! Access at http://<NodeIP>:30081"


Run:

bash helm_nodejs.sh
