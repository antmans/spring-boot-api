# spring-boot-api

# Spring Boot API – Wdrożenie z Argo CD, Helm i Kubernetes

## Opis

To repozytorium zawiera konfigurację do wdrożenia aplikacji **Spring Boot API** z użyciem **Argo CD**, **Helm** i **Kubernetes**.  
Wdrożenie obsługuje środowiska **deweloperskie** i **produkcyjne**, zapewniając wysoką dostępność, bezpieczeństwo oraz automatyzację.

---

## Wymagania

### Argo CD
- Wykorzystuje **Application Set** z **List Generator**.
- Tworzy **Namespace**, jeśli nie istnieje.
- Wdraża aplikację za pomocą **Helm Chart**.
- Pliki konfiguracyjne dla środowisk są przechowywane w osobnym repozytorium.

### Helm
- Prosty i czytelny **Helm Chart**, zawierający pliki YAML dla wymaganych zasobów.
- **Sekrety** generowane dynamicznie z losowymi wartościami.
- **Deployment** zawiera sumę kontrolną w **adnotacjach**, aby wykrywać zmiany.

### Kubernetes
- **ConfigMap**:
  - Zawiera przykładowy plik JSON.
  - Montowany do kontenera jako `/app/config.json`.

- **Deployment**:
  - **Liczba replik**:
    - `3` dla **środowiska deweloperskiego**.
    - `5` dla **środowiska produkcyjnego**.
  - **RollingUpdate** bez przestojów.
  - **Argumenty startowe w zależności od środowiska**:
    - `--spring.profiles.active=dev` dla **deweloperskiego**.
    - `--spring.profiles.active=prd` dla **produkcyjnego**.
  - **Zdarzenie przy zakończeniu pracy**:
    - `wget http://localhost:8080/service/shutdown` przed zamknięciem.

- **ReplicaSet**:
  - **Liveness i Readiness probes** na porcie `8080`.
  - **Zmienna środowiskowa** `POD_IP` dynamicznie pobierana z IP podu.
  - **Limity i żądania zasobów** dla podów.

- **Ekspozycja portów**:
  - Pod udostępnia porty **8080, 8081, 8082**.

- **Serwis (Service)**:
  - Typ: **ClusterIP**.
  - Udostępnia trzy porty: **8080, 8081, 8082**.

- **Ingress**:
  - Osobne **hosty** dla środowisk deweloperskiego i produkcyjnego.
  - **Ścieżki zależne od portu**:
    - `/api` → `8080`
    - `/logs` → `8081`
    - `/soap` → `8082`

- **Sekrety**:
  - Losowo generowane **klucze-wartości**.
  - Wstrzykiwane do **Deployment**.
