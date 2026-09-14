# ⚙️ Ansible Roles Library

To repozytorium stanowi centralną bibliotekę reużywalnych ról Ansible, wykorzystywanych do konfiguracji maszyn wirtualnych oraz instalacji oprogramowania w środowisku Proxmox.

## 📂 Struktura Repozytorium

Zgodnie z drzewem plików widocznym na image_cd9570.png, repozytorium zawiera następujące role i konfiguracje:

- **`.github/workflows/`**: Definicje procesów CI/CD (`verification.yml` do lintowania oraz `release.yml` do publikacji).
- **Katalogi ról (np. `base-os`, `kubernetes`)**: Każdy z pozostałych głównych folderów reprezentuje niezależną rolę (moduł) Ansible. Zgodnie ze standardem, każda z ról może posiadać wewnętrzną strukturę katalogów określającą jej zachowanie:
  - `tasks/` – Główne kroki i zadania do wykonania na serwerze docelowym (zazwyczaj zaczynające się od pliku `main.yml`).
  - `handlers/` – Procedury reaktywne (tzw. handlery), które uruchamiają się tylko wtedy, gdy zostaną wywołane przez inne zadanie (np. restart usługi po udanej zmianie pliku konfiguracyjnego).
  - `meta/` – Metadane roli określające m.in. jej zależności (np. wymóg wykonania innej roli w pierwszej kolejności).
  - _(Opcjonalnie)_ Inne standardowe katalogi Ansible dodawane w miarę potrzeb, takie jak `defaults/`, `vars/` (dla zmiennych), `templates/` (dla szablonów Jinja2) czy `files/` (dla statycznych plików).

## 🚀 Użycie ról w projekcie docelowym

Role z tego repozytorium nie posiadają własnego inventory i nie są uruchamiane samodzielnie. Środowiska docelowe: `proxmox-infrastructure` importuje je dynamicznie za pomocą pliku `requirements.yml`.

Aby bezpiecznie skorzystać z ról, zawsze przypisuj je do konkretnego, zamrożonego tagu wersji np:

```yaml
roles:
  - name: base-os
    src: [https://github.com/skni-kod/ansible-roles.git](https://github.com/skni-kod/ansible-roles.git)
    version: v1.1.2

  - name: kubernetes
    src: [https://github.com/skni-kod/ansible-roles.git](https://github.com/skni-kod/ansible-roles.git)
    version: v1.1.2
```

## 🔄 Cykl Życia i Sposób Pracy (Workflow CI/CD)

Repozytorium jest w pełni zautomatyzowane za pomocą GitHub Actions. Proces tworzenia i wdrażania zmian składa się z dwóch głównych etapów: weryfikacji kodu przed scaleniem (CI) oraz automatycznej publikacji nowej wersji roli (CD).

### 1. Weryfikacja kodu (Continuous Integration)

Po otwarciu Pull Requesta do gałęzi `main`, automatycznie uruchamia się workflow **`verification.yml`**. Dba on o spójność i jakość konfiguracji Ansible:

- **Linting i walidacja:** Skrypt uruchamia `ansible-lint`, który rygorystycznie analizuje cały kod YAML. Sprawdza on m.in. poprawne wcięcia, długość linii, odpowiednie użycie modułów oraz zgodność z dobrymi praktykami Ansible.

Jeśli w kodzie wystąpią błędy, Pull Request zostanie zablokowany do czasu wprowadzenia poprawek.

### 2. Publikacja po merge'u (Continuous Deployment)

Kiedy kod przejdzie Code Review i zostanie zmergowany do `main`, uruchamia się workflow **`release.yml`**.

- **Automatyczne tagowanie:** Potok analizuje historię commitów i automatycznie podbija wersję (generuje nowy tag).
- **GitHub Release:** Na podstawie historii zmian generowany jest czytelny changelog (lista wprowadzonych zmian) i tworzone jest nowe wydanie (Release) w zakładce repozytorium.

---

## 📝 Konwencja Commitów (Conventional Commits)

Ponieważ potok CD automatycznie nadaje numery wersji (Semantic Versioning), **bardzo ważne jest odpowiednie nazywanie commitów**. Używamy standardu _Conventional Commits_, który informuje mechanizm o tym, jak bardzo podbić numer nowej wersji.

Każdy commit (lub tytuł Pull Requesta przy squashowaniu) powinien zaczynać się od jednego z poniższych przedrostków:

- **`feat!:`** lub **`fix!:`** (z wykrzyknikiem) – Wprowadza zmiany niekompatybilne wstecz (Breaking Change) w zachowaniu roli.
  - _Skutek:_ Podbicie wersji **MAJOR** (np. z `v1.2.3` na `v2.0.0`).
  - _Przykład:_ `feat!: zmiana nazwy wymaganej zmiennej konfiguracyjnej Kubeadm`
- **`feat:`** – Dodanie nowej funkcjonalności do istniejącej roli (lub dodanie zupełnie nowej roli).
  - _Skutek:_ Podbicie wersji **MINOR** (np. z `v1.2.3` na `v1.3.0`).
  - _Przykład:_ `feat: dodanie taska instalującego pakiety narzędziowe w base-os`
- **`fix:`** – Naprawa błędu w istniejącej roli.
  - _Skutek:_ Podbicie wersji **PATCH** (np. z `v1.2.3` na `v1.2.4`).
  - _Przykład:_ `fix: dodanie flagi --force-conflicts do instalacji Calico`

**Dodatkowe przedrostki:**

- **`docs:`** – Zmiany wyłącznie w dokumentacji (np. aktualizacja README).
- **`chore:`** – Zmiany w konfiguracji repozytorium, potokach CI/CD (pliki `.yml`) lub narzędziach.
- **`refactor:`** – Zmiany w kodzie poprawiające jego strukturę, ale nie zmieniające funkcjonalności z perspektywy środowiska docelowego.
