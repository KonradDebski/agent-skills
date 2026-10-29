# Agent skills

Osobista kolekcja skilli dla agentów AI, współdzielona między komputerami.

## Skille

- [create-pr](create-pr/SKILL.md): tworzenie i scalanie pull requestów przez GitHub CLI.
- [famicord-sync](famicord-sync/SKILL.md): synchronizacja motywów WordPress między projektami Famicord.
- [figma](figma/SKILL.md): brief implementacji sekcji WordPress/ACF na podstawie Figmy.
- [unslop](unslop/SKILL.md): usuwanie charakterystycznych zwrotów AI z tekstu.
- [wordpress-audit](wordpress-audit/SKILL.md): sprawdzanie plików i bazy WordPress pod kątem infekcji.
- [wp-update](wp-update/SKILL.md): aktualizacja WordPressa, wtyczek i ACF Pro.
- [wp-update-list](wp-update-list/SKILL.md): porządkowanie list aktualizacji WordPressa.

## Submoduły

`.upstream/cursor-plugins` zawiera repozytorium [cursor/plugins](https://github.com/cursor/plugins), śledzące gałąź `main`. Katalog `unslop` jest symlinkiem do skilla w tym submodule.

## Klonowanie

Jeśli katalog docelowy już istnieje, przenieś go przed klonowaniem.

```bash
git clone --recurse-submodules git@github.com:KonradDebski/agent-skills.git ~/.agents/skills
```

## Pobieranie zmian

Pobranie zmian repozytorium i zapisanej w nim wersji submodułu:

```bash
cd ~/.agents/skills
git pull --ff-only
git submodule update --init --recursive
```

Pobranie najnowszej wersji `unslop` ze źródłowej gałęzi `main`:

```bash
git submodule update --init --remote --checkout -- .upstream/cursor-plugins
```

Polecenie wykonaj w katalogu repozytorium. Aby udostępnić tę wersję innym komputerom, zapisz zmianę wskazania submodułu w commicie i wykonaj `git push`.
