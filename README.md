# PAWCHO_Zadanie2 – CI/CD z Docker i GitHub Actions

## 📌 Opis zadania

Celem projektu było przygotowanie pełnego łańcucha CI/CD w GitHub Actions, który:

- buduje obraz kontenera na podstawie aplikacji z Zadania 1 (Flask – prognoza pogody),
- wspiera wieloarchitekturne budowanie (dla `linux/amd64` i `linux/arm64`),
- wykorzystuje mechanizm cache przy użyciu `DockerHub`,
- wykonuje test bezpieczeństwa obrazu z użyciem skanera Trivy (CVE),
- przesyła gotowy obraz do publicznego repozytorium autora w GitHub Container Registry (GHCR).

---

## 🔄 Pipeline (GitHub Actions)

Workflow zdefiniowany został w pliku `.github/workflows/docker-ci.yml`.

Główne etapy:

1. **Checkout kodu**
2. **Konfiguracja Docker Buildx**
3. **Logowanie do GHCR oraz DockerHub**
4. **Budowanie obrazu z wykorzystaniem cache**
5. **Skanowanie obrazu za pomocą Trivy (CVE: HIGH, CRITICAL)**
6. **Wysyłka do GHCR tylko jeśli obraz jest bezpieczny**

---

## 🐳 Budowanie i tagowanie obrazu

Obrazy tagowane są według schematu:

- `ghcr.io/<nazwa_użytkownika>/<nazwa_repo>:latest`
- `ghcr.io/<nazwa_użytkownika>/<nazwa_repo>:sha-<skrót_commita>`

Do tagowania wykorzystano `docker/metadata-action`.

---

## 🛡️ Skanowanie CVE (Trivy)

Wykorzystano narzędzie [Trivy](https://github.com/aquasecurity/trivy) do automatycznego skanowania obrazu pod kątem luk bezpieczeństwa.

- Skan zatrzymuje pipeline jeśli wykryje luki **CRITICAL** lub **HIGH** (`exit-code: 1`)
- Dla potrzeb testów CI/CD, dopuszczono czasowo `exit-code: 0`, aby zobaczyć pełne działanie łańcucha (jeśli obraz bazowy zawiera znane luki, np. w `debian`)

---

## 🧠 Uzasadnienie doboru technologii

- **Trivy** – prosta integracja z GitHub Actions, szerokie wsparcie (debian, python, binaria), szybka konfiguracja
- **DockerHub jako cache registry** – dostępność publiczna + kompatybilność z `cache-from` i `cache-to` w Buildx
- **Tagowanie typu `sha` i `latest`** – umożliwia odtwarzalność oraz łatwe odwołania do najnowszego obrazu