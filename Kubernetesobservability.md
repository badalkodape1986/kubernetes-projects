# 📘 Project 17: Advanced Monitoring

Kubernetes observability stack with:

- **Prometheus** → Metrics  
- **Grafana Loki** → Logging (alternative to EFK)  
- **Alertmanager** → Alerts to Slack/Email  

---

## 🔹 Real-World Use Case

- Alert when **CPU > 80%**  
- Centralized logs with **Loki**  
- Slack notifications via **Alertmanager**  

---

## 🛠️ Setup Steps

### Step 1: Install Prometheus & Grafana via Helm
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

helm install prometheus prometheus-community/kube-prometheus-stack
helm install grafana grafana/grafana


Step 2: Install Loki (Logging)
helm repo add grafana https://grafana.github.io/helm-charts
helm install loki grafana/loki-stack


Step 3: Setup Alerts
📄 alertmanager.yaml → configure routes to Slack/Email.

🎯 Final Outcome
✅ Metrics in Grafana
✅ Logs in Loki
✅ Alerts to Slack/Email
