# 📘 Project 7: Monitoring & Logging – Prometheus + Grafana + EFK  

Kubernetes provides **built-in metrics/logs**, but for real-world clusters we need centralized **monitoring & logging**.  

- **Prometheus** → Collects metrics (CPU, memory, network, app-level)  
- **Grafana** → Visualizes metrics in dashboards  
- **EFK (Elasticsearch, Fluentd, Kibana)** → Collects & visualizes logs  

---

## 🔹 Real-World Use Case  

Imagine running **50 microservices** in Kubernetes:  

- You need **alerts** if CPU > 80% or Pods keep restarting  
- You need **centralized logs** (instead of checking each pod)  

👉 **Prometheus + Grafana + EFK** provide a **single pane of glass** for monitoring & debugging.  

---

## 🛠️ Part 1: Manual Steps (kubectl + Helm)  

### Step 1: Install Helm (if not installed)  

```sh
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
Verify:


helm version
Step 2: Install Prometheus

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus


Check:
kubectl get pods | grep prometheus
Expose Prometheus (NodePort for demo):

kubectl expose deployment prometheus-server --type=NodePort --name=prometheus-nodeport --port=9090
kubectl get svc prometheus-nodeport


Access in browser:
http://<NodeIP>:<NodePort>

Step 3: Install Grafana
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana


Check:
kubectl get pods | grep grafana


Expose Grafana:
kubectl expose deployment grafana --type=NodePort --name=grafana-nodeport --port=3000
kubectl get svc grafana-nodeport


Default login:
User: admin
Password: admin (or check secret)


kubectl get secret grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
✅ Add Prometheus as a data source in Grafana → Create dashboards



Step 4: Deploy EFK Stack (Elasticsearch + Fluentd + Kibana)
Install Elasticsearch
kubectl apply -f https://raw.githubusercontent.com/elastic/cloud-on-k8s/2.9/config/samples/elasticsearch/elasticsearch.yaml


Install Kibana
kubectl apply -f https://raw.githubusercontent.com/elastic/cloud-on-k8s/2.9/config/samples/kibana/kibana.yaml


Expose Kibana:
kubectl port-forward service/kibana 5601:5601

Access Kibana:
http://localhost:5601


Deploy Fluentd DaemonSet (log collector)
kubectl apply -f https://raw.githubusercontent.com/fluent/fluentd-kubernetes-daemonset/master/fluentd-daemonset-elasticsearch-rbac.yaml

✅ Logs from all Pods are shipped to Elasticsearch → visible in Kibana



⚡ Part 2: Bash Script Automation
📄 k8s_monitoring_logging.sh


#!/bin/bash

echo "🚀 Setting up Monitoring (Prometheus + Grafana) and Logging (EFK)..."

# Add repos
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Install Prometheus
helm install prometheus prometheus-community/prometheus
kubectl expose deployment prometheus-server --type=NodePort --name=prometheus-nodeport --port=9090

# Install Grafana
helm install grafana grafana/grafana
kubectl expose deployment grafana --type=NodePort --name=grafana-nodeport --port=3000

# Deploy Elasticsearch & Kibana
kubectl apply -f https://raw.githubusercontent.com/elastic/cloud-on-k8s/2.9/config/samples/elasticsearch/elasticsearch.yaml
kubectl apply -f https://raw.githubusercontent.com/elastic/cloud-on-k8s/2.9/config/samples/kibana/kibana.yaml

# Deploy Fluentd
kubectl apply -f https://raw.githubusercontent.com/fluent/fluentd-kubernetes-daemonset/master/fluentd-daemonset-elasticsearch-rbac.yaml

echo "✅ Monitoring & Logging setup complete!"
kubectl get svc prometheus-nodeport grafana-nodeport
Run:


bash k8s_monitoring_logging.sh
🎯 Final Outcome
✅ Prometheus collects cluster & app metrics

✅ Grafana visualizes metrics in dashboards

✅ EFK stack centralizes logs for troubleshooting

✅ Provides a single monitoring & logging solution for Kubernetes clusters
