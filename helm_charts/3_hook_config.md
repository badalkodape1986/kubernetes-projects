📘 Project 3: Helm Hooks & ConfigMaps

Helm hooks are lifecycle events that let you run Kubernetes objects (like Jobs, Pods) before or after certain actions (install, upgrade, delete).

ConfigMaps let you inject environment-specific configs into Pods.

🔹 Real-World Use Case

Imagine a backend app that needs:

Config values like APP_MODE=production, LOG_LEVEL=info.

A pre-install init Job that runs database migrations before deploying the app.

👉 Helm hooks + ConfigMaps solve this.

🛠️ Part 1: Manual Steps
Step 1: Create Helm Chart
helm create backend-chart
cd backend-chart

Step 2: Define Config in values.yaml

Add custom configs:

replicaCount: 1

image:
  repository: my-dockerhub-username/backend-app
  tag: "latest"
  pullPolicy: IfNotPresent

service:
  type: NodePort
  port: 80
  nodePort: 30082

config:
  APP_MODE: "production"
  LOG_LEVEL: "info"

Step 3: Add ConfigMap Template

templates/configmap.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-config
data:
{{- range $key, $value := .Values.config }}
  {{ $key }}: "{{ $value }}"
{{- end }}

Step 4: Inject ConfigMap into Deployment

Edit templates/deployment.yaml:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-backend
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}-backend
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}-backend
    spec:
      containers:
      - name: backend-app
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - containerPort: 3000
        envFrom:
        - configMapRef:
            name: {{ .Release.Name }}-config

Step 5: Add Pre-Install Hook (DB Migration Job)

templates/db-migrate-job.yaml

apiVersion: batch/v1
kind: Job
metadata:
  name: {{ .Release.Name }}-db-migrate
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded
spec:
  template:
    spec:
      containers:
      - name: migrate
        image: alpine
        command: ["sh", "-c", "echo Running DB migration... && sleep 5"]
      restartPolicy: Never


pre-install → Runs before chart install.

hook-delete-policy → Removes old jobs before creating new ones.

Step 6: Install Chart
helm install my-backend ./backend-chart
kubectl get pods


Check logs of Job:

kubectl logs job/my-backend-db-migrate


Check if app got configs:

kubectl exec -it <backend-pod> -- env | grep APP_MODE


✅ Config injected, and pre-install Job executed.

⚡ Part 2: Bash Script Automation

Create helm_backend_hooks.sh

#!/bin/bash

CHART_NAME="backend-chart"
RELEASE_NAME="my-backend"

echo "🚀 Creating Backend Helm Chart with ConfigMaps + Hooks..."

# Step 1: Create chart
helm create $CHART_NAME

# Step 2: Replace values.yaml
cat > $CHART_NAME/values.yaml <<EOF
replicaCount: 1

image:
  repository: my-dockerhub-username/backend-app
  tag: "latest"
  pullPolicy: IfNotPresent

service:
  type: NodePort
  port: 80
  nodePort: 30082

config:
  APP_MODE: "production"
  LOG_LEVEL: "info"
EOF

# Step 3: Add ConfigMap template
cat > $CHART_NAME/templates/configmap.yaml <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-config
data:
{{- range \$key, \$value := .Values.config }}
  {{ \$key }}: "{{ \$value }}"
{{- end }}
EOF

# Step 4: Add Hook Job
cat > $CHART_NAME/templates/db-migrate-job.yaml <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ .Release.Name }}-db-migrate
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded
spec:
  template:
    spec:
      containers:
      - name: migrate
        image: alpine
        command: ["sh", "-c", "echo Running DB migration... && sleep 5"]
      restartPolicy: Never
EOF

# Step 5: Install chart
helm install $RELEASE_NAME ./$CHART_NAME

echo "✅ Backend app deployed with ConfigMap + pre-install hook"


Run:

bash helm_backend_hooks.sh

✅ Key Takeaways

ConfigMaps → Store environment-specific configs.

Helm Hooks → Run jobs at lifecycle events (pre-install, post-install, pre-upgrade).

Useful for DB migrations, init scripts, or cleanup tasks.
