📘 Helm Projects Roadmap (Step-by-Step)

We’ll cover Helm from basics → real-world use cases → advanced charts.

Each project will follow the same structure:

Manual Steps (Helm CLI + YAML templates)

Helm Chart Creation

Bash Automation

(Optional) Push to Helm Repo

🔹 Project 1: Helm Basics – Install Nginx Using a Chart

Use helm create to scaffold a chart.

Deploy an Nginx app using values (replicaCount, image, service).

Override default values via --set and values.yaml.

🔹 Project 2: Parameterizing Deployments with values.yaml

Deploy a Node.js app.

Store image name, replica count, and environment variables in values.yaml.

Demonstrate how to use templating ({{ .Values }}) in Deployment.

🔹 Project 3: Helm Hooks & ConfigMaps

Deploy an app that loads config from a ConfigMap.

Use Helm templating to generate ConfigMap from values.yaml.

Introduce Helm hooks for init jobs.

🔹 Project 4: Helm Secrets (Sealed Secrets or External Secrets)

Store DB credentials securely in Helm chart.

Use values.yaml + Kubernetes Secret templates.

(Optional) integrate with Sealed Secrets.

🔹 Project 5: WordPress + MySQL Helm Chart (Dependencies)

Use Helm dependencies (requirements.yaml / Chart.yaml).

Deploy WordPress and MySQL as a single chart.

Customize values (DB password, PV size).

🔹 Project 6: Helm + Ingress Controller

Deploy two apps under one Helm chart.

Configure Ingress rules via values.yaml.

Show path-based routing (/app1, /app2).

🔹 Project 7: Helm Chart for Monitoring Stack (Prometheus + Grafana)

Deploy monitoring stack using a Helm umbrella chart.

Customize dashboards & retention via values.

🔹 Project 8: Helm CI/CD with ArgoCD/Jenkins

Store Helm chart in GitHub.

ArgoCD syncs Helm chart automatically.

Jenkins pipeline builds chart + pushes to Helm repo.

🔹 Project 9: Helm on EKS (Production-Ready Chart)

Deploy a real app (Node.js API + Postgres) on AWS EKS using Helm.

Use custom values for staging vs production.

Integrate with AWS services (RDS, ALB Ingress).
