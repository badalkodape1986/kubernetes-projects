📘 Project 11: Helm Troubleshooting

We’ll simulate and debug 5 common Helm problems:

Release fails because of invalid templates.

Release succeeds but pods crash (CrashLoopBackOff).

Service doesn’t route traffic due to wrong labels.

Upgrade fails due to immutable fields.

Secret/ConfigMap changes not applied.

🔹 Real-World Use Case

Imagine you run helm install my-app ./chart, and either:

Pods never start.

Service doesn’t expose app.

Upgrade fails midway.

👉 As a DevOps engineer, you must debug & fix Helm issues fast.

🛠️ Part 1: Common Scenarios
🛑 Scenario 1: Invalid Template

Faulty deployment.yaml:

replicas: {{ .Values.replicaCount  # Missing closing braces


Helm install fails:

helm install bad-chart ./bad-chart


Debug:

helm template ./bad-chart   # Shows rendered YAML
helm lint ./bad-chart       # Finds syntax issues


✅ Fix: Add missing }}.

🛑 Scenario 2: CrashLoopBackOff

values.yaml:

image:
  repository: nginx
  tag: doesnotexist   # Invalid tag


Pods crash:

kubectl get pods
kubectl describe pod <pod>


Fix:

helm upgrade my-app ./chart --set image.tag=latest

🛑 Scenario 3: Service Not Working

service.yaml:

selector:
  app: wrong-label   # Doesn’t match deployment labels


Debug:

kubectl get endpoints my-app-service


Fix: Update service selector to match Deployment labels.

🛑 Scenario 4: Upgrade Fails (Immutable Field)

If you change service.type: ClusterIP → NodePort in Helm:

helm upgrade my-app ./chart


Kubernetes throws:

spec.clusterIP: Invalid value: "": field is immutable


✅ Fix:

kubectl delete svc my-app-service
helm upgrade my-app ./chart

🛑 Scenario 5: ConfigMap/Secret Not Updating

Helm doesn’t automatically restart Pods when ConfigMap/Secret changes.

Fix: Add checksum annotations in Deployment:

template:
  metadata:
    annotations:
      checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}


Now Pods restart when Config changes.

⚡ Part 2: Bash Script to Simulate Issues

Create helm_troubleshooting.sh:

#!/bin/bash

echo "🚨 Simulating Helm Troubleshooting Scenarios..."

# 1. Invalid Template
helm create bad-chart
sed -i 's/replicas: {{ .Values.replicaCount }}/replicas: {{ .Values.replicaCount/' bad-chart/templates/deployment.yaml
echo "➡️ Try: helm template ./bad-chart"

# 2. CrashLoopBackOff
helm create crash-chart
sed -i 's/tag: latest/tag: doesnotexist/' crash-chart/values.yaml
helm install crash-app ./crash-chart
echo "➡️ Check pods: kubectl get pods"

# 3. Wrong Service Selector
helm create wrong-svc-chart
sed -i 's/app: {{ include .Chart.Name . }}/app: wrong-label/' wrong-svc-chart/templates/service.yaml
helm install wrong-svc ./wrong-svc-chart
echo "➡️ Check endpoints: kubectl get endpoints wrong-svc"

# 4. Immutable Field Error
helm create immut-chart
helm install immut-app ./immut-chart
sed -i 's/type: ClusterIP/type: NodePort/' immut-chart/values.yaml
helm upgrade immut-app ./immut-chart || echo "❌ Immutable field error expected"

# 5. ConfigMap Change Not Triggering Restart
helm create config-chart
cat > config-chart/templates/configmap.yaml <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-config
data:
  LOG_LEVEL: "info"
EOF
echo "➡️ Fix by adding checksum annotations in Deployment"


Run:

bash helm_troubleshooting.sh

✅ Key Takeaways

helm lint + helm template → detect template issues before install.

kubectl describe pod → debug image/tag/env errors.

kubectl get endpoints → verify service selectors.

Immutable fields (like clusterIP) require recreation.

Use checksum annotations to trigger pod restarts on config changes.
