📘 Project 10: Helm + Service Mesh (Istio)

We’ll use Helm to deploy:

Two versions of a backend service (v1 & v2).

Istio Gateway + VirtualService to control routing.

DestinationRule to define subsets for traffic splitting.

🔹 Real-World Use Case

Your team runs a backend API in Kubernetes:

v1 = stable version

v2 = new candidate release

Instead of editing Istio YAML by hand → Helm lets you define traffic weights in values.yaml.
👉 This makes canary deployments reproducible and easy to tune.

🛠️ Part 1: Manual Steps
Step 1: Install Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-*
export PATH=$PWD/bin:$PATH
istioctl install --set profile=demo -y


Enable auto-injection:

kubectl label namespace default istio-injection=enabled --overwrite

Step 2: Create Helm Chart
helm create istio-canary
cd istio-canary

Step 3: Define Values (values.yaml)
backend:
  replicas: 1
  image:
    repository: hashicorp/http-echo
    tag: latest
  versions:
    - name: v1
      text: "Hello from Backend v1"
      weight: 90
    - name: v2
      text: "Hello from Backend v2"
      weight: 10

ingress:
  host: mybackend.local

Step 4: Deployment Template (templates/deployment.yaml)
{{- range .Values.backend.versions }}
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-{{ .name }}
spec:
  replicas: {{ $.Values.backend.replicas }}
  selector:
    matchLabels:
      app: backend
      version: {{ .name }}
  template:
    metadata:
      labels:
        app: backend
        version: {{ .name }}
    spec:
      containers:
      - name: backend
        image: "{{ $.Values.backend.image.repository }}:{{ $.Values.backend.image.tag }}"
        args:
        - "-text={{ .text }}"
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 5678
{{- end }}

Step 5: Istio Templates

DestinationRule (templates/destinationrule.yaml)

apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: backend
spec:
  host: backend
  subsets:
{{- range .Values.backend.versions }}
  - name: {{ .name }}
    labels:
      version: {{ .name }}
{{- end }}


VirtualService + Gateway (templates/virtualservice-gateway.yaml)

apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: backend-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - {{ .Values.ingress.host }}
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: backend
spec:
  hosts:
  - {{ .Values.ingress.host }}
  gateways:
  - backend-gateway
  http:
  - route:
{{- range .Values.backend.versions }}
    - destination:
        host: backend
        subset: {{ .name }}
      weight: {{ .weight }}
{{- end }}

Step 6: Install Chart
helm install canary ./istio-canary
kubectl get pods
kubectl get svc backend
kubectl get virtualservice


Test:

Add entry in /etc/hosts:

<NodeIP> mybackend.local


Hit:

curl http://mybackend.local/backend


✅ ~90% responses → Hello from Backend v1, ~10% → Hello from Backend v2.

⚡ Part 2: Bash Script Automation

Create helm_istio_canary.sh

[O#!/bin/bash

CHART_NAME="istio-canary"
RELEASE_NAME="canary"

echo "🚀 Deploying Istio Canary Helm Chart..."

# Step 1: Create chart
helm create $CHART_NAME
cd $CHART_NAME

# Step 2: Replace values.yaml
cat > values.yaml <<EOF
backend:
  replicas: 1
  image:
    repository: hashicorp/http-echo
    tag: latest
  versions:
    - name: v1
      text: "Hello from Backend v1"
      weight: 80
    - name: v2
      text: "Hello from Backend v2"
      weight: 20

ingress:
  host: mybackend.local
EOF

cd ..

# Step 3: Install chart
helm install $RELEASE_NAME ./$CHART_NAME

echo "✅ Istio canary deployed! Add '<NodeIP> mybackend.local' to /etc/hosts and curl it."


Run:

bash helm_istio_canary.sh

✅ Key Takeaways

Helm + Istio = automated traffic shifting between versions.

Canary deployment weights are configurable in values.yaml.

Great for blue/green & gradual rollouts.

Combine with monitoring (Prometheus/Grafana) for safe progressive delivery.
