# Produkcijski deployment — Secure Event Ticketing Platform

## Preduvjeti

- Kubernetes cluster (minikube, kind, ili cloud)
- kubectl instaliran i konfiguriran
- Docker ili Podman instaliran
- Trivy instaliran (skeniranje slika)

---

## Struktura k8s foldera
k8s/

├── namespace.yaml

├── configmap.yaml

├── secret.yaml

├── rbac.yaml

├── network-policy.yaml

├── ingress.yaml

├── postgres/

│   ├── postgres-pvc.yaml

│   ├── postgres-deployment.yaml

│   └── postgres-service.yaml

├── redis/

│   ├── redis-deployment.yaml

│   └── redis-service.yaml

├── api/

│   ├── api-deployment.yaml

│   └── api-service.yaml

├── worker/

│   └── worker-deployment.yaml

├── frontend/

│   ├── frontend-deployment.yaml

│   └── frontend-service.yaml

├── security-report.md

├── runbook.md

└── README-production.md

---

## Korak po korak deployment

### 1. Kloniraj repozitorij
```bash
git clone https://github.com/josipmaric557/devops-project-app
cd devops-project-app
```

### 2. Skeniraj slike prije deploya (DevSecOps)
```bash
trivy image ghcr.io/josipmaric557/ticketing-api:1.0
trivy image ghcr.io/josipmaric557/ticketing-frontend:1.0
trivy image ghcr.io/josipmaric557/ticketing-worker:1.0
```

### 3. Pokreni Minikube
```bash
minikube start
minikube addons enable ingress
```

### 4. Primijeni manifeste redom
```bash
# Namespace
kubectl apply -f k8s/namespace.yaml

# Konfiguracija i tajne
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secret.yaml

# RBAC
kubectl apply -f k8s/rbac.yaml

# Baza i cache
kubectl apply -f k8s/postgres/postgres-pvc.yaml
kubectl apply -f k8s/postgres/postgres-deployment.yaml
kubectl apply -f k8s/postgres/postgres-service.yaml
kubectl apply -f k8s/redis/redis-deployment.yaml
kubectl apply -f k8s/redis/redis-service.yaml

# Aplikacijski servisi
kubectl apply -f k8s/api/api-deployment.yaml
kubectl apply -f k8s/api/api-service.yaml
kubectl apply -f k8s/worker/worker-deployment.yaml
kubectl apply -f k8s/frontend/frontend-deployment.yaml
kubectl apply -f k8s/frontend/frontend-service.yaml

# Networking
kubectl apply -f k8s/network-policy.yaml
kubectl apply -f k8s/ingress.yaml
```

### 5. Provjeri status
```bash
kubectl get pods -n ticketing
kubectl get services -n ticketing
kubectl get ingress -n ticketing
```

Svi podovi trebaju biti `1/1 Running`.

### 6. Postavi lokalni DNS
Dodaj u `C:\Windows\System32\drivers\etc\hosts` (Windows) ili `/etc/hosts` (Linux/Mac):
192.168.49.2 ticketing.local

### 7. Provjeri aplikaciju
```bash
# Health check
curl http://ticketing.local/api/healthz

# Readiness check  
curl http://ticketing.local/api/readyz
```

Otvori browser na `http://ticketing.local`.

---

## Rolling update

```bash
# Deploy nove verzije
kubectl set image deployment/api api=ghcr.io/josipmaric557/ticketing-api:2.0 -n ticketing

# Prati status
kubectl rollout status deployment/api -n ticketing
```

## Rollback

```bash
# Rollback na prethodnu verziju
kubectl rollout undo deployment/api -n ticketing

# Rollback na specifičnu reviziju
kubectl rollout undo deployment/api --to-revision=1 -n ticketing
```

---

## Shutdown

```bash
# Obriši sve resurse
kubectl delete namespace ticketing
```