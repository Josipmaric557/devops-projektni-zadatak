# Runbook za incidente — Secure Event Ticketing Platform

## Incident 1: Pad baze (PostgreSQL)

### Simptomi
- API vraća 503 na `/readyz`
- Worker ne procesira narudžbe
- Greška: `connection refused` ili `ECONNREFUSED`

### Dijagnostika
```bash
kubectl get pods -n ticketing
kubectl logs deployment/postgres -n ticketing
kubectl describe pod -l app=postgres -n ticketing
```

### Korektivne mjere
```bash
# Restart PostgreSQL poda
kubectl rollout restart deployment/postgres -n ticketing

# Provjeri PVC
kubectl get pvc -n ticketing

# Provjeri je li baza dostupna
kubectl exec -it deployment/postgres -n ticketing -- pg_isready -U ticketing_user
```

### Validacija
```bash
kubectl get pods -n ticketing
curl http://ticketing.local/api/readyz
```

---

## Incident 2: Loš image tag (ErrImagePull)

### Simptomi
- Pod u statusu `ErrImagePull` ili `ImagePullBackOff`
- Novi deployment ne može startati

### Dijagnostika
```bash
kubectl get pods -n ticketing
kubectl describe pod <pod-name> -n ticketing
# Gledaj Events sekciju na dnu
```

### Korektivne mjere
```bash
# Rollback na prethodnu verziju
kubectl rollout undo deployment/api -n ticketing

# Ili postavi ispravan tag
kubectl set image deployment/api api=ghcr.io/josipmaric557/ticketing-api:1.0 -n ticketing
```

### Validacija
```bash
kubectl rollout status deployment/api -n ticketing
kubectl get pods -n ticketing
```

---

## Incident 3: Neispravan Secret

### Simptomi
- API ili Worker se restartaju
- Greška u logovima: `password authentication failed`

### Dijagnostika
```bash
kubectl logs deployment/api -n ticketing
kubectl get secret ticketing-secret -n ticketing -o yaml
```

### Korektivne mjere
```bash
# Ažuriraj secret
kubectl delete secret ticketing-secret -n ticketing
kubectl apply -f secret.yaml

# Restart servisa koji koriste secret
kubectl rollout restart deployment/api -n ticketing
kubectl rollout restart deployment/worker -n ticketing
```

### Validacija
```bash
kubectl get pods -n ticketing
kubectl logs deployment/api -n ticketing
```

---

## Opći troubleshooting postupak

1. `kubectl get pods -n ticketing` — provjeri status svih podova
2. `kubectl describe pod <ime> -n ticketing` — detalji i Events
3. `kubectl logs <ime> -n ticketing` — logovi aplikacije
4. `kubectl logs <ime> -n ticketing --previous` — logovi crashanog poda
5. `kubectl get events -n ticketing --sort-by='.lastTimestamp'` — kronološki eventi