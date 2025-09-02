📘 Project 4: Helm Secrets

Helm supports Secrets out of the box, but storing them in plain values.yaml is insecure.
We’ll see:

How to template Kubernetes Secrets with Helm.

How to inject them into Deployments.

(Optional) How to use Sealed Secrets for GitOps-safe secret storage.

🔹 Real-World Use Case

Imagine a MySQL + Backend App:

MySQL needs a MYSQL_ROOT_PASSWORD.

Backend needs a DB_PASSWORD secret.

We don’t want passwords hardcoded → Helm chart should handle them securely.

🛠️ Part 1: Manual Steps
Step 1: Create Helm Chart
helm create secure-app
cd secure-app

Step 2: Add Secret Values in values.yaml
replicaCount: 1

image:
  repository: my-dockerhub-username/backend-app
  tag: "latest"
  pullPolicy: IfNotPresent

service:
  type: NodePort
  port: 80
  nodePort: 30083

secrets:
  DB_PASSWORD: "SuperSecret123"

Step 3: Add Secret Template

templates/secret.yaml

apiVersion: v1
kind: Secret
metadata:
  name: {{ .Release.Name }}-secret
type: Opaque
data:
{{- range $key, $value := .Values.secrets }}
  {{ $key }}: {{ $value | b64enc | quote }}
{{- end }}


This encodes secrets into base64.

Step 4: Update Deployment to Use Secret

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
      - name: backend
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - containerPort: 3000
        env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: {{ .Release.Name }}-secret
              key: DB_PASSWORD

Step 5: Install Chart
helm install my-secure-app ./secure-app
kubectl get secret my-secure-app-secret -o yaml


Check pod env:

kubectl exec -it <backend-pod> -- env | grep DB_PASSWORD


✅ Secret injected without exposing in plain YAML.

🛠️ Part 2: Optional – Sealed Secrets (For GitOps)

If you want to store secrets in Git repos securely:

Install Sealed Secrets controller:

kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.25.0/controller.yaml


Encrypt secret using kubeseal:

kubectl create secret generic db-secret --from-literal=DB_PASSWORD=SuperSecret123 --dry-run=client -o yaml | kubeseal --format yaml > sealedsecret.yaml


Commit sealedsecret.yaml into Git.

Controller decrypts it at runtime.

⚡ Part 3: Bash Script Automation

Create helm_secrets.sh

#!/bin/bash

CHART_NAME="secure-app"
RELEASE_NAME="my-secure-app"

echo "🚀 Creating Helm Chart with Secrets..."

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
  nodePort: 30083

secrets:
  DB_PASSWORD: "SuperSecret123"
EOF

# Step 3: Add Secret template
cat > $CHART_NAME/templates/secret.yaml <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: {{ .Release.Name }}-secret
type: Opaque
data:
{{- range \$key, \$value := .Values.secrets }}
  {{ \$key }}: {{ \$value | b64enc | quote }}
{{- end }}
EOF

# Step 4: Install chart
helm install $RELEASE_NAME ./$CHART_NAME

echo "✅ Secret deployed. Run 'kubectl get secret ${RELEASE_NAME}-secret -o yaml'"


Run:

bash helm_secrets.sh

✅ Key Takeaways

Helm can template Secrets just like ConfigMaps.

Always use b64enc to encode sensitive values.

For GitOps workflows, use Sealed Secrets to commit safely.

Avoid putting raw secrets in values.yaml (use external secret managers like AWS Secrets Manager, Vault).
