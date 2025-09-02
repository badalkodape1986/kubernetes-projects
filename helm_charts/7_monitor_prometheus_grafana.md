📘 Project 7: Helm Chart for Monitoring Stack (Prometheus + Grafana)

Instead of deploying Prometheus and Grafana separately with YAML, we’ll use Helm to:

Deploy Prometheus for metrics collection.

Deploy Grafana for dashboards.

Connect Grafana to Prometheus as a data source.

🔹 Real-World Use Case

In production, you need to monitor cluster health (CPU, memory, network) and visualize metrics.
Instead of manual setup:
👉 Use Helm charts to install Prometheus + Grafana quickly and configure with values.yaml.

🛠️ Part 1: Manual Steps
Step 1: Add Helm Repos
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

Step 2: Install Prometheus with Helm
helm install prometheus prometheus-community/prometheus


Check:

kubectl get pods | grep prometheus
kubectl get svc prometheus-server


Expose Prometheus via NodePort (for demo):

kubectl expose deployment prometheus-server --type=NodePort --name=prometheus-nodeport --port=9090
kubectl get svc prometheus-nodeport


Access:

http://<NodeIP>:<NodePort>

Step 3: Install Grafana with Helm
helm install grafana grafana/grafana


Expose Grafana:

kubectl expose deployment grafana --type=NodePort --name=grafana-nodeport --port=3000
kubectl get svc grafana-nodeport


Get Grafana password:
[O
kubectl get secret grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo


Login:

User: admin

Password: (above)

Step 4: Connect Grafana to Prometheus

In Grafana UI → Add Data Source → Prometheus → set URL:

http://prometheus-server:80


Now you can create dashboards.

🛠️ Part 2: Custom Helm Umbrella Chart

Instead of installing separately, create an umbrella chart that manages both.

helm create monitoring-stack
cd monitoring-stack

Step 5: Add Dependencies in Chart.yaml
apiVersion: v2
name: monitoring-stack
description: A Helm chart for Prometheus + Grafana
type: application
version: 0.1.0
appVersion: "1.0"

dependencies:
  - name: prometheus
    version: "25.0.0"
    repository: "https://prometheus-community.github.io/helm-charts"
  - name: grafana
    version: "8.3.0"
    repository: "https://grafana.github.io/helm-charts"


Update dependencies:

helm dependency update

Step 6: Configure in values.yaml
prometheus:
  alertmanager:
    enabled: false
  pushgateway:
    enabled: false
  server:
    service:
      type: NodePort
      nodePort: 30090

grafana:
  adminUser: admin
  adminPassword: grafana123
  service:
    type: NodePort
    nodePort: 30091

Step 7: Install Umbrella Chart
helm install monitoring ./monitoring-stack
kubectl get pods
kubectl get svc


Access in browser:

Prometheus → http://<NodeIP>:30090

Grafana → http://<NodeIP>:30091

⚡ Part 3: Bash Script Automation

Create helm_monitoring.sh

#!/bin/bash

CHART_NAME="monitoring-stack"
RELEASE_NAME="monitoring"

echo "🚀 Deploying Prometheus + Grafana with Helm..."

# Step 1: Create chart
helm create $CHART_NAME
cd $CHART_NAME

# Step 2: Add dependencies
cat > Chart.yaml <<EOF
apiVersion: v2
name: monitoring-stack
description: A Helm chart for Prometheus + Grafana
type: application
version: 0.1.0
appVersion: "1.0"

dependencies:
  - name: prometheus
    version: "25.0.0"
    repository: "https://prometheus-community.github.io/helm-charts"
  - name: grafana
    version: "8.3.0"
    repository: "https://grafana.github.io/helm-charts"
EOF

# Step 3: Update dependencies
helm dependency update

# Step 4: Configure values.yaml
cat > values.yaml <<EOF
prometheus:
  server:
    service:
      type: NodePort
      nodePort: 30090

grafana:
  adminUser: admin
  adminPassword: grafana123
  service:
    type: NodePort
    nodePort: 30091
EOF

cd ..

# Step 5: Install chart
helm install $RELEASE_NAME ./$CHART_NAME

echo "✅ Prometheus: http://<NodeIP>:30090"
echo "✅ Grafana: http://<NodeIP>:30091"


Run:

bash helm_monitoring.sh

✅ Key Takeaways

Helm makes it easy to install complex apps like Prometheus/Grafana.

Umbrella charts manage multiple dependencies.

values.yaml customizes service type, ports, admin credentials.

Perfect for production monitoring setup.
