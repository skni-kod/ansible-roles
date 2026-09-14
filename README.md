# ⚙️ Ansible Roles Library

Repozytorium stanowi bibliotekę reużywalnych ról Ansible, wykorzystywanych do konfiguracji maszyn wirtualnych w środowisku Proxmox.

## 📂 Struktura Repozytorium

```text
- **`.github/workflows/`**: Definicje procesów CI/CD (`verification.yml` do lintowania oraz `release.yml` do publikacji).
- **Katalogi ról (np. `base-os`, `kubernetes`)**: Każdy z pozostałych głównych folderów rolę (moduł) Ansible. Każda z ról może posiadać wewnętrzną strukturę katalogów określającą jej właściwości:
  - `tasks/` – Główne kroki i zadania do wykonania na serwerze docelowym (zazwyczaj zaczynające się od pliku `main.yml`).
  - `handlers/` – Procedury (tzw. handlery), które uruchamiają się tylko wtedy, gdy zostaną wywołane przez inne zadanie (np. restart usługi po udanej zmianie pliku konfiguracyjnego).
  - `meta/` – Metadane roli określające m.in. jej zależności (np. wymóg wykonania innej roli w pierwszej kolejności).
  - _(Opcjonalnie)_ Inne standardowe katalogi Ansible dodawane w miarę potrzeb, takie jak `defaults/`, `vars/` (dla zmiennych), `templates/` (dla szablonów Jinja2) czy `files/` (dla statycznych plików).
```

## 🚀 Użycie ról w projekcie docelowym

Role z tego repozytorium nie posiadają własnego inventory i nie są uruchamiane samodzielnie. Środowiska docelowe: `proxmox-infrastructure` importuje je za pomocą pliku `requirements.yml`.

Aby bezpiecznie skorzystać z ról, zawsze przypisuj je do konkretnego tagu wersji np:

```yaml
roles:
  - name: base-os
    src: [https://github.com/skni-kod/ansible-roles.git](https://github.com/skni-kod/ansible-roles.git)
    version: v1.1.2

  - name: kubernetes
    src: [https://github.com/skni-kod/ansible-roles.git](https://github.com/skni-kod/ansible-roles.git)
    version: v1.1.2
```

## 🔄 Sposób Pracy (Workflow CI/CD)

Repozytorium jest w pełni zautomatyzowane za pomocą GitHub Actions. Proces tworzenia i wdrażania zmian składa się z dwóch głównych etapów: weryfikacji kodu przed scaleniem (CI) oraz automatycznego releasu nowej wersji roli (CD).

### 1. Weryfikacja kodu (Continuous Integration)

Po otwarciu Pull Requesta do gałęzi `main`, automatycznie uruchamia się workflow **`verification.yml`**. Dba on o spójność i jakość konfiguracji Ansible:

- **Linting i walidacja:** Skrypt uruchamia `ansible-lint`, który analizuje cały kod YAML. Sprawdza on m.in. poprawne wcięcia, długość linii, odpowiednie użycie modułów oraz zgodność z dobrymi praktykami Ansible.

Jeśli w kodzie wystąpią błędy, Pull Request zostanie zablokowany.

### 2. Publikacja po merge'u (Continuous Deployment)

Kiedy kod zostanie zmergowany do `main`, uruchamia się workflow **`release.yml`**.

- **Automatyczne tagowanie:** Workflow analizuje historię commitów i automatycznie podbija wersję (generuje nowy tag).
- **GitHub Release:** Na podstawie historii zmian generowany jest czytelny changelog (lista wprowadzonych zmian) i tworzone jest nowe wydanie (Release) w zakładce repozytorium.

---

## 📝 Konwencja Commitów (Conventional Commits)

Ponieważ workflow CD automatycznie nadaje numery wersji (Semantic Versioning), **ważne jest odpowiednie nazywanie commitów**. Używamy standardu _Conventional Commits_, który informuje mechanizm o tym, jak bardzo podbić numer nowej wersji.

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
