# NBP USD/PLN — wdrożenie

Repozytorium odpowiada za wdrażanie aplikacji na Google Kubernetes Engine za pomocą **Helm i Cloud Build**.

**Aplikacja:** https://136-69-119-54.sslip.io/

## Technologie

Helm, Kubernetes, GKE Autopilot, Google Cloud Build, cert-manager, Let's Encrypt.

## Konfiguracja Kubernetes

- Deployment — minimum 2 repliki aplikacji.
- Service — komunikacja wewnątrz klastra.
- Ingress — publiczny dostęp HTTPS.
- HPA — automatyczne skalowanie od 2 do 5 replik.
- Liveness i readiness probes.
- Cert-manager — automatyczne zarządzanie certyfikatami TLS.
- FrontendConfig — przekierowanie HTTP → HTTPS (301).

Obraz aplikacji przypięty jest przez **digest SHA256**, co zapewnia wdrożenie dokładnie wskazanej wersji.

## Helm

Chart aplikacji znajduje się w `charts/nbp-app/`.

Konfiguracja produkcyjna: `charts/nbp-app/values-prod.yaml`.

Walidacja lokalna:

```bash
helm lint charts/nbp-app -f charts/nbp-app/values-prod.yaml
```

## CI/CD

1. Zmiana konfiguracji Helm lub digesta obrazu trafia do Pull Request.
2. Cloud Build sprawdza poprawność chartu.
3. Merge do `main` uruchamia wdrożenie.
4. Helm aktualizuje aplikację w namespace `nbp-prod`.

## Docelowy model środowisk

Na potrzeby zadania wdrażamy wyłącznie do **PROD**.

W większym projekcie zastosowalibyśmy następujący proces promocji zmian:

```text
main     → kod bazowy, bez wdrożenia
  ↓
dev      → projekt GCP DEV
  ↓
staging  → projekt GCP STAGING
  ↓
prod     → projekt GCP PROD
```

Każde środowisko miałoby osobną konfigurację Helm:

- `values-dev.yaml`
- `values-staging.yaml`
- `values-prod.yaml`

Merge do odpowiedniego brancha uruchamiałby wdrożenie wyłącznie do przypisanego projektu GCP.

Wdrożenie produkcyjne wymagałoby dodatkowego zatwierdzenia. Pozwoliłoby to przetestować zmiany na DEV i STAGING przed ich promocją na PROD.

## Weryfikacja wdrożenia

```bash
kubectl get pods -n nbp-prod
kubectl get ingress -n nbp-prod
kubectl get hpa -n nbp-prod
```