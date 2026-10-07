# Współpraca w repozytorium product

To repozytorium zawiera dokumentację, diagramy i specyfikacje WorthMyTime. Zasady pracy (issue, branch, PR, commity) są takie same jak w repozytoriach z kodem i opisane w **[ONBOARDING.md](ONBOARDING.md)**. Najważniejsze:

> **issue = branch = pull request = merge = zamknięcie zadania = usunięcie brancha**

1. Weź zadanie z [tablicy](https://github.com/orgs/worthmytime/projects/1) (etykiety `good first issue`, `help wanted`) i przypisz je sobie.
2. Branch z `dev`: `<typ>/<numer-issue>-<opis>`, np. `docs/3-model-danych`.
3. Commity: `typ: krótki opis` (patrz [zasady commitów](ONBOARDING.md#6-zasady-commitów)).
4. Pull request do `dev` z `Closes #<numer>`. Do `stage` i `main` scala tylko właściciel.

## Dokumentacja i diagramy

- **Diagramy jako tekst** (Mermaid w blokach ` ```mermaid `), żeby dało się je porównywać w PR. Eksport PNG/SVG trzymaj obok źródła, nie zamiast niego.
- **Struktura:** `docs/` (opisy funkcji i wzorów), `diagrams/` (diagramy i źródła), `specs/` (specyfikacje i modele), `decisions/` (notatki o decyzjach: kontekst, decyzja, skutki).
- **Spójność z kodem:** zmieniasz zachowanie w backendzie lub frontendzie? Zaktualizuj odpowiedni dokument w tym samym tygodniu albo załóż issue.
- Pisz po polsku, z polskimi znakami. Pokazuj założenia i przykłady z liczbami (np. wzór, a pod nim wynik dla konkretnych danych).
- Linki względne między dokumentami, żeby działały w całym repozytorium.

## Przed otwarciem PR

- [ ] Mermaid renderuje się w podglądzie GitHuba
- [ ] linki działają
- [ ] brak danych osobowych i sekretów
- [ ] opis PR mówi, co się zmienia i dlaczego
