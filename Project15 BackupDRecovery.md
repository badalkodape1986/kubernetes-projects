# 📘 Project 15: Backup & Disaster Recovery

Data protection is critical in Kubernetes. We use:

- **Velero** → Backup & restore cluster resources  
- **PV Snapshots** → Persistent Volume backups  
- **Disaster Recovery workflows**

---

## 🔹 Real-World Use Case

- Cluster upgrade goes wrong → restore from backup  
- Accidental `kubectl delete` → restore resources  
- Persistent Volumes snapshot before migration  

---

## 🛠️ Part 1: Manual Setup

### Step 1: Install Velero CLI
```bash
curl -L https://github.com/vmware-tanzu/velero/releases/latest/download/velero-linux-amd64.tar.gz | tar -xz
mv velero /usr/local/bin/

Step 2: Install Velero in Cluster (AWS S3 backend)
velero install \
    --provider aws \
    --plugins velero/velero-plugin-for-aws:v1.5.2 \
    --bucket my-velero-backups \
    --backup-location-config region=us-east-1 \
    --secret-file ./credentials-velero

Step 3: Take a Backup
velero backup create my-backup --include-namespaces dev-team
velero backup get

Step 4: Restore
velero restore create --from-backup my-backup
🎯 Final Outcome
✅ Cluster & PV backups with Velero
✅ Restore workloads after failure
✅ Disaster recovery readiness
.
