# WorthMyTime: produkt

Miejsce na dokumentację, diagramy i istotne pliki produktowe projektu WorthMyTime. Kod aplikacji jest w osobnych repozytoriach.

## Repozytoria

| Repozytorium | Zawartość |
|---|---|
| [worthmytime-backend](https://github.com/poewer/worthmytime-backend) | REST API (Python, Sanic, SQLAlchemy, Alembic) |
| [worthmytime-frontend](https://github.com/poewer/worthmytime-frontend) | Aplikacja webowa (Next.js, Tailwind, shadcn/ui) |
| worthmytime-product (to repo) | Dokumentacja, diagramy, specyfikacje, makiety |

Backlog i status zadań: [tablica projektu](https://github.com/users/poewer/projects/6).

## Co tu trafia

- dokumenty produktowe i specyfikacje (np. `WorthMyTime_MVP.md`, `WorthMyTime_Budget_Model.md`)
- diagramy (architektura, model danych, przepływy) najlepiej jako tekst (Mermaid) albo pliki źródłowe obok eksportu
- notatki o decyzjach (krótko: kontekst, decyzja, skutki)
- odnośniki i eksporty makiet

## Proponowana struktura

```
docs/        opisy funkcji, wzory, założenia
diagrams/    diagramy i ich źródła
specs/       specyfikacje i modele (MVP, model budżetu)
decisions/   notatki o decyzjach
```

Struktura jest luźna i można ją zmieniać, gdy będzie potrzeba.

## Zasady pracy

Zmiany idą tak jak w pozostałych repozytoriach: zadanie, osobna gałąź, pull request, scalenie. Szczegóły w `CONTRIBUTING.md` w repozytoriach kodu.
