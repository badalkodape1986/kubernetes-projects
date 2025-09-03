📘 Project 16: GitOps with FluxCD

FluxCD is a GitOps tool that automates Kubernetes deployments.

- Alternative to **ArgoCD**  
- Continuous delivery via **GitOps**  
- **Helm + Flux** integration  

---

## 🔹 Real-World Use Case

- Developers commit manifests to Git  
- FluxCD syncs **Git → Kubernetes cluster**  
- Ensures **Git = source of truth**  

---

## 🛠️ Setup Steps

### Step 1: Install Flux CLI
```bash
curl -s https://fluxcd.io/install.sh | sudo bash

Step 2: Bootstrap Flux
flux bootstrap github \
  --owner=<your-github-user> \
  --repository=k8s-flux-repo \
  --branch=main \
  --path=./clusters/my-cluster


Step 3: Deploy an App with Flux
Create and commit a deployment manifest:

📄 nginx-deployment.yaml (committed to Git repo)

Flux will automatically sync this to the cluster.

🎯 Final Outcome
✅ GitOps pipeline with FluxCD
✅ Helm + Git integration
✅ Continuous Delivery to Kubernetes
