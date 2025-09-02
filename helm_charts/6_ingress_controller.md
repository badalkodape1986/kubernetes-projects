📘 Project 6: Helm + Ingress Controller

Helm can template Ingress objects, letting you parameterize:

Domain names / hosts

Paths (/app1, /app2)

TLS configuration

🔹 Real-World Use Case

Imagine you’re hosting:

app1 → Nginx frontend

app2 → Node.js backend

Instead of exposing them with NodePorts, you want:

http://example.com/app1 → routes to app1-service

http://example.com/app2 → routes to app2-service

👉 Helm chart + values make this reusable for multiple apps.

🛠️ Part 1: Manual Steps
Step 1: Install NGINX Ingress Controller

On Minikube:

minikube addons enable ingress


On other clusters:

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml


Check:

kubectl get pods -n ingress-nginx

Step 2: Create Helm Chart
helm create ingress-apps
cd ingress-apps

Step 3: Update values.yaml
apps:
  - name: app1
    image: nginx
    servicePort: 80
    path: /app1
  - name: app2
    image: httpd
    servicePort: 80
    path: /app2

ingress:
  enabled: true
  className: nginx
  host: myapps.local

Step 4: Add Deployment + Service Template

templates/deployment-service.yaml

{{- range .Values.apps }}
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .name }}-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: {{ .name }}
  template:
    metadata:
      labels:
        app: {{ .name }}
    spec:
      containers:
      - name: {{ .name }}
        image: {{ .image }}
        ports:
        - containerPort: {{ .servicePort }}
---
apiVersion: v1
kind: Service
metadata:
  name: {{ .name }}-service
spec:
  selector:
    app: {{ .name }}
  ports:
  - port: {{ .servicePort }}
    targetPort: {{ .servicePort }}
{{- end }}

Step 5: Add Ingress Template

templates/ingress.yaml

{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apps-ingress
  annotations:
    kubernetes.io/ingress.class: {{ .Values.ingress.className }}
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: {{ .Values.ingress.host }}
    http:
      paths:
      {{- range .Values.apps }}
      - path: {{ .path }}
        pathType: Prefix
        backend:
          service:
            name: {{ .name }}-service
            port:
              number: {{ .servicePort }}
      {{- end }}
{{- end }}

Step 6: Install Chart
helm install my-ingress ./ingress-apps


Verify:

kubectl get ingress

Step 7: Test Access

Add to /etc/hosts:

<MinikubeIP> myapps.local


Access in browser:

http://myapps.local/app1 → Nginx page  
http://myapps.local/app2 → Apache httpd page  


✅ Multiple apps accessible via Ingress + Helm chart.

⚡ Part 2: Bash Script Automation

Create helm_ingress.sh

#!/bin/bash

CHART_NAME="ingress-apps"
RELEASE_NAME="my-ingress"

echo "🚀 Creating Helm Chart with Ingress for multiple apps..."

# Step 1: Create chart
helm create $CHART_NAME
cd $CHART_NAME

# Step 2: Replace values.yaml
cat > values.yaml <<EOF
apps:
  - name: app1
    image: nginx
    servicePort: 80
    path: /app1
  - name: app2
    image: httpd
    servicePort: 80
    path: /app2

ingress:
  enabled: true
  className: nginx
  host: myapps.local
EOF

cd ..

# Step 3: Install chart
helm install $RELEASE_NAME ./$CHART_NAME

echo "✅ Ingress chart deployed! Add '<NodeIP> myapps.local' to /etc/hosts"


Run:

bash helm_ingress.sh

✅ Key Takeaways

Helm charts can manage Ingress rules dynamically.

values.yaml lets you add/remove apps without editing templates.

Makes multi-service apps easy to deploy under a single domain.
