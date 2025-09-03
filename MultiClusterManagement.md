# 📘 Project 19: Multi-Cluster Management

Managing multiple Kubernetes clusters with:

- **Rancher** → Centralized management  
- **Cross-cluster discovery**  

---

## 🔹 Real-World Use Case

- Multiple clusters across **AWS, GCP, On-Prem**  
- Unified management via **Rancher**  
- Deploy workloads across clusters  

---

## 🛠️ Setup Steps

### Step 1: Install Rancher (Helm)
```bash
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
kubectl create namespace cattle-system
helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=myrancher.local
Step 2: Add Clusters to Rancher UI
Access Rancher Dashboard

Import clusters

Manage workloads across clusters

🎯 Final Outcome
✅ Single pane of glass for multiple clusters
✅ Cross-cluster workload management
✅ Unified security & policies
