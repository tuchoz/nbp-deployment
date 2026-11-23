# NBP USD/PLN — wdrożenie

Repozytorium odpowiada za wdrażanie aplikacji Java 21 + Spring Boot 4 na Google Kubernetes Engine za pomocą Helm i Cloud Build.

Aplikacja: https://136-69-119-54.sslip.io/

## Technologie

Helm, Kubernetes, GKE Autopilot, Google Cloud Build, cert-manager, Let's Encrypt.

## Konfiguracja Kubernetes

- Deployment — minimum 2 repliki.
- Service i Ingress — dostęp HTTPS.
- HPA — autoskalowanie od 2 do 5 replik.
- Liveness, readiness i startup probes — Spring Actuator na porcie 9090.
- Cert-manager — automatyczne certyfikaty TLS.
- RollingUpdate i automatyczny rollback Helm.

## Helm

Chart: `charts/nbp-app/`

Konfiguracja produkcyjna: `charts/nbp-app/values-prod.yaml`.

```bash
helm lint charts/nbp-app -f charts/nbp-app/values-prod.yaml
```

## CI/CD

1. Operator wybiera digest SHA256 obrazu w `values-prod.yaml`.
2. Pull Request uruchamia walidację Helm.
3. Merge do `main` uruchamia Cloud Build CD.
4. Helm wdraża zatwierdzony obraz do `nbp-prod`.

## Weryfikacja wdrożenia

```bash
kubectl rollout status deployment/nbp-app -n nbp-prod
kubectl get pods,hpa,ingress -n nbp-prod
```

W projekcie wykorzystano jedno środowisko produkcyjne GKE.