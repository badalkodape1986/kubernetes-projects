📘 Project 12: Upgrading Kubernetes Master & Worker Nodes

Kubernetes versions are released frequently, and clusters must be upgraded carefully:

Control Plane (Master nodes) → upgraded first.

Worker nodes → upgraded next, one at a time.

Ensure zero downtime by draining nodes and using Deployments with replicas.

🔹 Real-World Use Case

Your cluster is running Kubernetes v1.25, but a new feature requires v1.28.
You need to upgrade both master and worker nodes without downtime.

🛠️ Part 1: Upgrade Steps (Manual – kubeadm clusters)
Step 1: Check Current Version

On master node:

kubectl version --short
kubectl get nodes

Step 2: Drain the Node (before upgrade)
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data


This safely evicts workloads to other nodes.

Step 3: Upgrade Master Node

Upgrade kubeadm tool:

apt-get update && apt-get install -y --allow-change-held-packages kubeadm=1.28.0-00


Plan the upgrade:

kubeadm upgrade plan


Apply the upgrade:

kubeadm upgrade apply v1.28.0


Upgrade kubelet & kubectl:

apt-get install -y --allow-change-held-packages kubelet=1.28.0-00 kubectl=1.28.0-00
systemctl daemon-reexec
systemctl restart kubelet


Uncordon master node:

kubectl uncordon <master-node-name>

Step 4: Upgrade Worker Nodes

Repeat on each worker node:

Drain node:

kubectl drain <worker-node-name> --ignore-daemonsets --delete-emptydir-data


Upgrade kubeadm:

apt-get update && apt-get install -y --allow-change-held-packages kubeadm=1.28.0-00


Upgrade node:

kubeadm upgrade node


Upgrade kubelet & kubectl:

apt-get install -y --allow-change-held-packages kubelet=1.28.0-00 kubectl=1.28.0-00
systemctl daemon-reexec
systemctl restart kubelet


Uncordon node:

kubectl uncordon <worker-node-name>

Step 5: Verify Cluster
kubectl get nodes
kubectl version --short


✅ All nodes should now be running the new Kubernetes version.

⚡ Part 2: Bash Script Automation (Debian/Ubuntu kubeadm setup)

Create k8s_upgrade.sh

#!/bin/bash

TARGET_VERSION="1.28.0-00"

echo "🚀 Upgrading Kubernetes Node to version $TARGET_VERSION"

# Step 1: Drain node (only if not master)
NODE_NAME=$(hostname)
if kubectl get node $NODE_NAME >/dev/null 2>&1; then
  echo "Draining node $NODE_NAME..."
  kubectl drain $NODE_NAME --ignore-daemonsets --delete-emptydir-data
fi

# Step 2: Upgrade kubeadm
echo "Upgrading kubeadm..."
apt-get update && apt-get install -y --allow-change-held-packages kubeadm=$TARGET_VERSION

# Step 3: If master node, upgrade control plane
if [[ "$NODE_NAME" == *"master"* ]] || [[ "$NODE_NAME" == *"control"* ]]; then
  echo "Upgrading control plane..."
  kubeadm upgrade apply v${TARGET_VERSION%-00} -y
else
  echo "Upgrading worker node..."
  kubeadm upgrade node
fi

# Step 4: Upgrade kubelet and kubectl
echo "Upgrading kubelet and kubectl..."
apt-get install -y --allow-change-held-packages kubelet=$TARGET_VERSION kubectl=$TARGET_VERSION
systemctl daemon-reexec
systemctl restart kubelet

# Step 5: Uncordon node
echo "Uncordoning node $NODE_NAME..."
kubectl uncordon $NODE_NAME

echo "✅ Upgrade completed on $NODE_NAME"


Run on each master and worker node:

bash k8s_upgrade.sh
