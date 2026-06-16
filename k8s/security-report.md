# Sigurnosno izvješće skeniranja slika — Secure Event Ticketing Platform

## Alat: Trivy v0.71.1
## Datum skeniranja: 16.06.2026.

---

## Sažetak rezultata

| Slika | CRITICAL | HIGH | MEDIUM | LOW | Ukupno |
|-------|----------|------|--------|-----|--------|
| ticketing-api:1.0 | 0 | 2 | 8 | 20 | 30 |
| ticketing-frontend:1.0 | 0 | 2 | 8 | 20 | 30 |
| ticketing-worker:1.0 | 0 | 2 | 8 | 20 | 30 |

---

## Analiza ranjivosti

### Kritične ranjivosti (CRITICAL)
Nije pronađena nijedna kritična ranjivost.

### Visoke ranjivosti (HIGH)

| CVE | Paket | Opis | Status |
|-----|-------|------|--------|
| CVE-2026-45447 | libcrypto3, libssl3 | Heap Use-After-Free u OpenSSL PKCS7_verify() | Fix dostupan (3.5.7-r0) |
| CVE-2024-21538 | cross-spawn | ReDoS u cross-spawn | Fix dostupan (7.0.5) |
| CVE-2025-64756 | glob | Command Injection via malicious filenames | Fix dostupan (11.1.0) |

### Srednje ranjivosti (MEDIUM)
Sve ranjivosti u openssl paketu (alpine 3.23.4) — fix dostupan u verziji 3.5.7-r0.

---

## Uzrok ranjivosti

Većina ranjivosti dolazi iz:
1. **Alpine base image** — openssl paketi zastarjele verzije
2. **npm interni paketi** — tar, minimatch, cross-spawn u node_modules npm-a (nisu direktne ovisnosti aplikacije)

---

## Korektivne mjere

| Mjera | Prioritet | Opis |
|-------|-----------|------|
| Ažurirati base image | Visok | Koristiti `node:20-alpine` s najnovijim patchevima |
| Ažurirati npm | Visok | Nova verzija npm-a uklanja ranjivosti u tar, minimatch |
| Implementirati policy | Srednji | Blokirati deploy ako ima HIGH/CRITICAL ranjivosti |
| Redovito skeniranje | Stalni | Trivy skeniranje u CI/CD pipelineu pri svakom buildu |

---

## Zaključak

Nijedna **CRITICAL** ranjivost nije pronađena. Pronađene HIGH ranjivosti nalaze se
u sistemskim paketima (openssl) i npm internim alatima — nisu dio aplikacijskog koda.
Preporuča se ažuriranje base imagea pri sljedećem buildu.

## Naredbe korištene za skeniranje

```bash
trivy image ghcr.io/josipmaric557/ticketing-api:1.0 --format json --output trivy-api.json
trivy image ghcr.io/josipmaric557/ticketing-frontend:1.0 --format json --output trivy-frontend.json
trivy image ghcr.io/josipmaric557/ticketing-worker:1.0 --format json --output trivy-worker.json
```