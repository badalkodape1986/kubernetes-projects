📘 Project 5: WordPress + MySQL Helm Chart (Dependencies)

Helm supports umbrella charts → one parent chart that pulls in other charts as dependencies.
We’ll deploy WordPress + MySQL as a single Helm chart, with MySQL password injected via secrets and persistent storage for both.

🔹 Real-World Use Case

You want to run WordPress in Kubernetes, but it needs MySQL as a backend DB.
Instead of deploying them separately:
👉 Create one Helm chart that manages both using dependencies.

🛠️ Part 1: Manual Steps
Step 1: Create Parent Chart
helm create wordpress-chart
cd wordpress-chart

Step 2: Define Dependencies in Chart.yaml
apiVersion: v2
name: wordpress-chart
description: A Helm chart for WordPress + MySQL
type: application
version: 0.1.0
appVersion: "1.0"

dependencies:
  - name: mysql
    version: "9.10.0"
    repository: "https://charts.bitnami.com/bitnami"


Update dependencies:

helm dependency update


This pulls MySQL chart into charts/.

Step 3: Define Values in values.yaml
wordpress:
  replicaCount: 1
  image:
    repository: wordpress
    tag: latest
  service:
    type: NodePort
    port: 80
    nodePort: 30084
  persistence:
    enabled: true
    size: 1Gi

mysql:
  auth:
    rootPassword: supersecret
    database: wordpress
    username: wp_user
    password: wp_pass
  primary:
    persistence:
      enabled: true
      size: 1Gi

Step 4: Add WordPress Deployment Template

templates/wordpress-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-wordpress
spec:
  replicas: {{ .Values.wordpress.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}-wordpress
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}-wordpress
    spec:
      containers:
      - name: wordpress
        image: "{{ .Values.wordpress.image.repository }}:{{ .Values.wordpress.image.tag }}"
        ports:
        - containerPort: 80
        env:
        - name: WORDPRESS_DB_HOST
          value: {{ .Release.Name }}-mysql
        - name: WORDPRESS_DB_USER
          value: {{ .Values.mysql.auth.username }}
        - name: WORDPRESS_DB_PASSWORD
          value: {{ .Values.mysql.auth.password }}
        - name: WORDPRESS_DB_NAME
          value: {{ .Values.mysql.auth.database }}
        volumeMounts:
        - name: wordpress-data
          mountPath: /var/www/html
      volumes:
      - name: wordpress-data
        persistentVolumeClaim:
          claimName: {{ .Release.Name }}-wordpress-pvc
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: {{ .Release.Name }}-wordpress-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: {{ .Values.wordpress.persistence.size }}
---
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-wordpress-service
spec:
  type: {{ .Values.wordpress.service.type }}
  selector:
    app: {{ .Release.Name }}-wordpress
  ports:
  - port: {{ .Values.wordpress.service.port }}
    targetPort: 80
    nodePort: {{ .Values.wordpress.service.nodePort }}

Step 5: Install Chart
helm install my-blog ./wordpress-chart


Check resources:

kubectl get pods
kubectl get svc


Access in browser:

http://<NodeIP>:30084


✅ WordPress setup page should appear.

⚡ Part 2: Bash Script Automation

Create helm_wordpress_mysql.sh

#!/bin/bash

CHART_NAME="wordpress-chart"
RELEASE_NAME="my-blog"

echo "🚀 Deploying WordPress + MySQL Helm Chart..."

# Step 1: Create parent chart
helm create $CHART_NAME

cd $CHART_NAME

# Step 2: Add MySQL dependency
cat > Chart.yaml <<EOF
apiVersion: v2
name: wordpress-chart
description: A Helm chart for WordPress + MySQL
type: application
version: 0.1.0
appVersion: "1.0"

dependencies:
  - name: mysql
    version: "9.10.0"
    repository: "https://charts.bitnami.com/bitnami"
EOF

# Step 3: Update dependencies
helm dependency update

# Step 4: Add values.yaml
cat > values.yaml <<EOF
wordpress:
  replicaCount: 1
  image:
    repository: wordpress
    tag: latest
  service:
    type: NodePort
    port: 80
    nodePort: 30084
  persistence:
    enabled: true
    size: 1Gi

mysql:
  auth:
    rootPassword: supersecret
    database: wordpress
    username: wp_user
    password: wp_pass
  primary:
    persistence:
      enabled: true
      size: 1Gi
EOF

cd ..

# Step 5: Install chart
helm install $RELEASE_NAME ./$CHART_NAME

echo "✅ WordPress + MySQL deployed! Access at http://<NodeIP>:30084"


Run:

bash helm_wordpress_mysql.sh

✅ Key Takeaways

Helm dependencies let you combine multiple charts.

WordPress + MySQL can be managed as one chart.

Values are used to configure MySQL credentials + storage.

Perfect for multi-service apps (LAMP stack, microservices).
