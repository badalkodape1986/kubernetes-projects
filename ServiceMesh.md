# 📘 Project 18: Service Mesh Advanced

Advanced Istio Service Mesh features:

- **mTLS** → Secure pod-to-pod communication  
- **Circuit breaking**  
- **Retries & failover**  

---

## 🔹 Real-World Use Case

- Ensure zero-trust networking with **mTLS**  
- Retry failed requests automatically  
- Circuit breaking for unreliable services  

---

## 🛠️ Setup Steps

### Step 1: Install Istio (demo profile)
```bash
istioctl install --set profile=demo -y
kubectl label namespace default istio-injection=enabled
Step 2: Configure DestinationRule for Circuit Breaking


📄 destinationrule.yaml

apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: backend
spec:
  host: backend
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 1
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
    outlierDetection:
      consecutive5xxErrors: 1
      interval: 1s
      baseEjectionTime: 30s
      maxEjectionPercent: 100

🎯 Final Outcome
✅ Secure pod-to-pod with mTLS
✅ Circuit breaking & retries
✅ Resilient microservices with Istio
