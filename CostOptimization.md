# 📘 Project 20: Cost Optimization in Kubernetes

Reduce cloud spend using:

- **Resource requests/limits best practices**  
- **Spot instances (AWS EKS)**  
- **KubeCost** → Cost visibility  

---

## 🔹 Real-World Use Case

- Developers request too much CPU/Memory → wasted costs  
- Non-critical workloads → run on **spot instances**  
- **KubeCost dashboard** → monitor cluster spend  

---

## 🛠️ Setup Steps

### Step 1: Enforce Resource Requests & Limits

📄 `deployment.yaml`

```yaml
resources:
  requests:
    cpu: "200m"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"


Step 2: Run Non-Critical Pods on Spot Instances (EKS)
Use nodeSelector / taints to schedule spot workloads.

Step 3: Install KubeCost

helm repo add kubecost https://kubecost.github.io/cost-analyzer/
helm install kubecost kubecost/cost-analyzer \
  --namespace kubecost \
  --create-namespace


Access the dashboard:
kubectl port-forward --namespace kubecost deployment/kubecost-cost-analyzer 9090:9090

🎯 Final Outcome
✅ Enforced resource requests & limits
✅ Spot instances reduce cloud bills
✅ KubeCost dashboard for cost monitoring
