📘 Project 10: Service Mesh with Istio

Istio is a popular service mesh that provides:

Traffic management → Blue/Green, Canary releases, routing.

Security → mTLS between services.

Observability → Tracing, logging, and monitoring with tools like Kiali, Jaeger, Prometheus.

🔹 Real-World Use Case

Imagine you’re running microservices in Kubernetes:

frontend → user interface

backend → API

payments → payment gateway

You want to:

Route only 10% of traffic to a new backend version (canary).

Secure all traffic with mTLS.

Monitor requests with tracing & dashboards.

👉 Solution → Deploy Istio as service mesh.

🛠️ Part 1: Manual Steps (kubectl + Istioctl)
Step 1: Install Istio

Download and install istioctl:

curl -L https://istio.io/downloadIstio | sh -
cd istio-*
export PATH=$PWD/bin:$PATH


Install Istio (default profile):

istioctl install --set profile=demo -y


Label default namespace for Istio auto-injection:

kubectl label namespace default istio-injection=enabled

Step 2: Deploy Sample Apps

frontend.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  selector:
    app: frontend
  ports:
  - port: 80


backend.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
      version: v1
  template:
    metadata:
      labels:
        app: backend
        version: v1
    spec:
      containers:
      - name: backend
        image: hashicorp/http-echo
        args:
        - "-text=Hello from Backend v1"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
      version: v2
  template:
    metadata:
      labels:
        app: backend
        version: v2
    spec:
      containers:
      - name: backend
        image: hashicorp/http-echo
        args:
        - "-text=Hello from Backend v2"
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


Apply apps:

kubectl apply -f frontend.yaml
kubectl apply -f backend.yaml

Step 3: Configure Istio Gateway & VirtualService

istio-gateway.yaml

apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: app-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: backend-routing
spec:
  hosts:
  - "*"
  gateways:
  - app-gateway
  http:
  - match:
    - uri:
        prefix: /backend
    route:
    - destination:
        host: backend
        subset: v1
      weight: 90
    - destination:
        host: backend
        subset: v2
      weight: 10

Step 4: Define DestinationRules

destination-rules.yaml

apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: backend
spec:
  host: backend
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2


Apply:

kubectl apply -f istio-gateway.yaml
kubectl apply -f destination-rules.yaml


✅ Now Istio routes 90% traffic → backend-v1, 10% traffic → backend-v2.

Step 5: Observability

Deploy Istio addons:

kubectl apply -f samples/addons


This installs:

Prometheus → Metrics

Grafana → Dashboards

Jaeger → Tracing

Kiali → Service mesh visualization

Access addons (using port-forward):

kubectl port-forward svc/kiali -n istio-system 20001:20001
kubectl port-forward svc/jaeger-query -n istio-system 16686:16686

⚡ Part 2: Bash Script Automation

Create k8s_istio_setup.sh

#!/bin/bash

echo "🚀 Installing Istio Service Mesh..."

# Install Istio (demo profile)
istioctl install --set profile=demo -y

# Enable auto-injection
kubectl label namespace default istio-injection=enabled --overwrite

# Deploy Frontend
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  selector:
    app: frontend
  ports:
  - port: 80
EOF

# Deploy Backend v1 & v2
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
      version: v1
  template:
    metadata:
      labels:
        app: backend
        version: v1
    spec:
      containers:
      - name: backend
        image: hashicorp/http-echo
        args:
        - "-text=Hello from Backend v1"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
      version: v2
  template:
    metadata:
      labels:
        app: backend
        version: v2
    spec:
      containers:
      - name: backend
        image: hashicorp/http-echo
        args:
        - "-text=Hello from Backend v2"
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
EOF

# Create Gateway + VirtualService + DestinationRule
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: app-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: backend-routing
spec:
  hosts:
  - "*"
  gateways:
  - app-gateway
  http:
  - match:
    - uri:
        prefix: /backend
    route:
    - destination:
        host: backend
        subset: v1
      weight: 90
    - destination:
        host: backend
        subset: v2
      weight: 10
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: backend
spec:
  host: backend
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
EOF

echo "✅ Istio setup complete! Access Grafana/Jaeger/Kiali for observability."


Run:

bash k8s_istio_setup.sh
