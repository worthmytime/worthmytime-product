# Onboarding: jak wdrożyć się do WorthMyTime

Ten przewodnik prowadzi od pierwszego dnia do pierwszego scalonego pull requesta. Zajmie około godziny (większość to instalacja narzędzi).

## 1. Czym jest projekt

WorthMyTime to aplikacja finansowa budowana z pomocą sztucznej inteligencji, a zarazem projekt do nauki technologii i pracy w zespole. Zaczęła się od przeliczania ceny zakupu na godziny pracy, a rozwija się w stronę pełnej aplikacji do budżetu, wydatków, zobowiązań i celów. Co działa i dokąd zmierzamy: [README](README.md). Szczegóły produktu: dokumenty w tym repozytorium (`specs/`, `docs/`, `diagrams/`, w miarę jak powstają).

| Repozytorium | Co zawiera | Technologie |
|---|---|---|
| [worthmytime-backend](https://github.com/worthmytime/worthmytime-backend) | REST API | Python 3.12, Sanic, SQLAlchemy (async), Alembic, pytest, ruff |
| [worthmytime-frontend](https://github.com/worthmytime/worthmytime-frontend) | aplikacja webowa | Next.js (App Router), TypeScript, Tailwind v4, shadcn/ui, Playwright |
| [worthmytime-product](https://github.com/worthmytime/worthmytime-product) | dokumentacja, diagramy, specyfikacje | Markdown, Mermaid |

Tablica z zadaniami: [WorthMyTime (projekt)](https://github.com/orgs/worthmytime/projects/1).

## 2. Dostęp (raz, na początku)

0. **Nie masz jeszcze dostępu?** Skontaktuj się z właścicielem projektu: e-mail **poewer1@gmail.com** albo wiadomość na [**LinkedIn**](https://www.linkedin.com/in/michal-bialek-a48891267/) (Michał Białek). Dostaniesz zaproszenie do organizacji.
1. Przyjmij zaproszenie do organizacji **worthmytime** (mail od GitHuba albo [github.com/worthmytime](https://github.com/worthmytime)). Wchodzisz do zespołu **Developers**, który ma uprawnienie zapisu do wszystkich repozytoriów.
2. Włącz uwierzytelnianie dwuskładnikowe na swoim koncie GitHub (Settings, Password and authentication).
3. Skonfiguruj Git tak, żeby commity łączyły się z Twoim kontem GitHub (patrz [zasady commitów](#6-zasady-commitów)):
   ```powershell
   git config --global user.name "Imię Nazwisko"
   git config --global user.email "twoj-adres@..."   # adres dodany do konta GitHub albo ID+login@users.noreply.github.com
   ```
4. Uwierzytelnianie do `git push`: Git Credential Manager (instalowany z Git for Windows) albo SSH. Nie wklejaj tokenów do repozytorium.

## 3. Narzędzia

| Narzędzie | Po co | Uwagi |
|---|---|---|
| Git | kontrola wersji | Git for Windows zawiera Credential Manager |
| [uv](https://docs.astral.sh/uv/) i Python 3.12 | backend | `uv` sam pobierze właściwego Pythona |
| Node.js (aktualny LTS) i npm | frontend | |
| Edytor (np. VS Code) | | opcjonalnie rozszerzenia: Python, ESLint, Tailwind CSS IntelliSense |
| Docker | Postgres lokalnie | opcjonalny: domyślnie backend używa SQLite |

## 4. Uruchomienie lokalnie

Najpierw sklonuj repozytoria obok siebie:

```powershell
mkdir worthmytime; cd worthmytime
git clone https://github.com/worthmytime/worthmytime-backend.git
git clone https://github.com/worthmytime/worthmytime-frontend.git
git clone https://github.com/worthmytime/worthmytime-product.git   # dokumentacja
```

Domyślną gałęzią jest `dev`, więc od razu masz aktualny kod integracyjny.

### Backend (API na porcie 8001)

```powershell
cd worthmytime-backend
copy .env.example .env             # ustaw CORS_ORIGINS=http://localhost:3000 (sesja w cookie nie działa z *)
uv sync --python 3.12
uv run python -m alembic upgrade head     # tworzy bazę SQLite (worthmytime.db) i tabele
uv run -m app.main                 # http://localhost:8001/api/v1/health
```

> Na Windowsie `uv run alembic`, `uv run pytest` i `uv run ruff` bywają blokowane ("Odmowa dostępu"). Używaj `uv run python -m alembic ...`, `python -m pytest`, `python -m ruff`.

Postgres zamiast SQLite (opcjonalnie, tak jak na produkcji):

```powershell
docker run --name wmt-db -e POSTGRES_USER=wmt -e POSTGRES_PASSWORD=wmt -e POSTGRES_DB=worthmytime -p 5432:5432 -d postgres:16
# w .env: DATABASE_URL=postgresql+asyncpg://wmt:wmt@localhost:5432/worthmytime
uv run python -m alembic upgrade head
```

### Frontend (aplikacja na porcie 3000)

```powershell
cd worthmytime-frontend
copy .env.example .env.local       # NEXT_PUBLIC_API_URL=http://localhost:8001/api/v1
npm install
npm run dev                        # http://localhost:3000
```

Uruchom backend i frontend jednocześnie w dwóch terminalach. Bez backendu frontend działa w trybie anonimowym (kalkulator, dane w przeglądarce), a konto, historia i alerty wymagają API.

### Jak sprawdzić, że działa

1. `http://localhost:8001/api/v1/health` zwraca `{"status": "ok", "database": "up"}`.
2. Na `http://localhost:3000` wejdź w Profil, uzupełnij dochód, wróć do Kalkulatora i policz zakup.
3. Załóż konto testowe (Profil, "Nie mam konta"). Dane lądują w lokalnej bazie `worthmytime.db`.

## 5. Testowanie własnych zmian lokalnie

Zanim otworzysz PR, uruchom to samo, co robi CI:

| Gdzie | Polecenie | Co sprawdza |
|---|---|---|
| backend | `uv run python -m ruff check .` | styl i błędy statyczne |
| backend | `uv run python -m pytest -q` | testy jednostkowe i API (SQLite) |
| backend | `$env:TEST_DATABASE_URL="postgresql+asyncpg://..."` i pytest | testy na Postgresie (jak w CI) |
| frontend | `npm run lint` i `npx tsc --noEmit` | ESLint i typy |
| frontend | `npm run test:e2e` | Playwright, desktop i telefon, API zamockowane |

Wskazówki:
- Pierwszy raz do testów e2e: `npx playwright install chromium`. Testy budują wersję produkcyjną (kilka minut) i nie potrzebują backendu.
- Zmieniasz model w `app/models.py`? Dodaj migrację: `uv run python -m alembic revision --autogenerate -m "opis"`, przejrzyj ją i sprawdź w górę i w dół (`upgrade head`, `downgrade -1`). Test `test_models_match_migrations` wykryje brak migracji.
- Zmieniasz zachowanie aplikacji? Dodaj test. PR bez testu zmiany logiki jest odsyłany.
- Czysta baza do eksperymentów: usuń `worthmytime.db` i zrób `alembic upgrade head` (to tylko Twoja lokalna baza).
- Nie commituj plików lokalnych: `.env`, `.env.local`, `worthmytime.db`, `.venv`, `node_modules`, `.next*` (są w `.gitignore`).

## 6. Zasady commitów

Format: **`typ: krótki opis`** (do 72 znaków, bez kropki na końcu, po polsku lub angielsku, w jednym stylu w PR).

| Typ | Kiedy |
|---|---|
| `feat` | nowa funkcja |
| `fix` | poprawka błędu |
| `docs` | dokumentacja |
| `refactor` | zmiana struktury kodu bez zmiany działania |
| `test` | same testy |
| `chore` | porządki, zależności, konfiguracja |
| `ci` | workflowy GitHub Actions |

Przykłady: `feat: dzienny limit i prognoza na stronie Wydatki`, `fix: zaokrąglanie kwot w kalkulatorze`, `docs: opis wzoru na realną stawkę`.

Zasady:
- **Mały commit = jedna logiczna zmiana.** Kod i jego test w jednym commicie albo w sąsiednich.
- W treści commita (pod pustą linią) napisz **dlaczego**, jeśli powód nie jest oczywisty. Co się zmieniło, widać w diffie.
- Nigdy nie commituj sekretów, haseł, tokenów ani danych osobowych. Pomyłka? Zgłoś od razu, bo samo usunięcie w kolejnym commicie nie wystarczy (sekret trzeba unieważnić).
- Używaj adresu e-mail powiązanego z Twoim kontem GitHub, żeby commity były przypisane do Ciebie.
- Własne gałęzie możesz przepisywać (`git rebase`, `--amend`), ale **nie przepisuj `dev`, `stage` ani `main`** i nie force-pushuj do cudzych gałęzi.
- Historia zadania i tak zostaje scalona do jednego commita (squash), więc **tytuł PR jest ważniejszy niż pojedyncze commity**. Nadaj mu ten sam format `typ: opis`.

## 7. Jak wybrać i zabrać zadanie

1. Otwórz [tablicę](https://github.com/orgs/worthmytime/projects/1) i widok **Backlog** albo kolumnę **Ready** (zadania gotowe do wzięcia).
2. Szukaj etykiet:
   - **`good first issue`**: małe, dobrze opisane zadania na start,
   - **`help wanted`**: zadania, do których szukamy rąk do pracy.
3. Przeczytaj opis i kryterium ukończenia. Niejasne? Zapytaj w komentarzu pod issue **zanim** zaczniesz.
4. **Przypisz zadanie sobie** (*Assignees*) i przenieś je do **In progress**. Jedno zadanie na osobę naraz.
5. Nowy pomysł albo błąd? Utwórz issue (szablon *Zadanie* lub *Błąd*), a potem pracuj według punktu 8. Każda zmiana ma issue.

### Jak zgłosić błąd

Utwórz issue z szablonem **Błąd**: co robiłeś, czego oczekiwałeś, co się stało, środowisko (przeglądarka, system) oraz, jeśli możesz, zrzut ekranu lub treść błędu z konsoli. Im łatwiej odtworzyć problem, tym szybciej zostanie naprawiony.

## 8. Od zadania do scalonego PR

Zasada projektu: **issue = branch = pull request = merge = zamknięcie zadania = usunięcie brancha**.

```powershell
git switch dev; git pull
git switch -c feature/24-savings-rate        # <typ>/<numer-issue>-<opis>
# ... praca, testy lokalnie, małe commity ...
git push -u origin feature/24-savings-rate
```

- **Nazwa brancha:** `<typ>/<numer-issue>-<opis>`, np. `feature/24-savings-rate`, `fix/31-rounding`. Typy: `feature`, `fix`, `chore`, `docs`, `refactor`, `test`. Opis małymi literami i myślnikami.
- **Pull request do `dev`** (może być *Draft*, gdy chcesz wcześniej pokazać postęp). W opisie musi być `Closes #<numer>` z numerem z nazwy brancha. Wypełnij szablon: co i dlaczego, jak sprawdzić.
- **Kontrole:** `test` i `pr-policy` muszą być zielone. Czerwone? Popraw w tym samym branchu.
- **Review:** właściciel dostaje prośbę o przegląd automatycznie. Odpowiadaj na uwagi kolejnymi commitami.
- **Merge do `dev`:** *Squash and merge*, gdy kontrole są zielone. GitHub zamknie issue, usunie branch, a zadanie przejdzie do **Done**.
- **Sprzątanie lokalne:** `git switch dev; git pull; git fetch --prune; git branch -d feature/24-savings-rate`.

### Wydania

Do produkcji zmiany trafiają w kolejnych krokach: `dev` (integracja) → `stage` (testy przed produkcją) → `main` (produkcja). PR-y `dev` → `stage` i `stage` → `main` otwiera i scala **tylko właściciel** (`poewer`). Ty pracujesz wyłącznie z `dev`.

## 9. Definicja ukończenia

Zadanie jest skończone, gdy:
- [ ] spełnia kryterium z opisu issue,
- [ ] ma testy (lub uzasadnienie w PR, dlaczego nie),
- [ ] `ruff` i `pytest` (backend) albo `lint`, `tsc` i `e2e` (frontend) przechodzą lokalnie,
- [ ] zmiany w modelach mają migrację,
- [ ] opis PR wyjaśnia co, dlaczego i jak to sprawdzić,
- [ ] nie ma sekretów ani przypadkowych plików w diffie.

## 10. Pierwszy dzień: lista kontrolna

- [ ] przyjęte zaproszenie do organizacji i włączone 2FA
- [ ] skonfigurowany Git (imię, e-mail powiązany z kontem)
- [ ] sklonowane repozytoria, backend i frontend działają lokalnie
- [ ] uruchomione testy w repozytorium, w którym będziesz pracować
- [ ] wybrane zadanie z etykietą `good first issue`, przypisane do Ciebie
- [ ] pierwszy PR do `dev` otwarty (nawet jako Draft)

## 11. Praca z asystentami AI

Projekt powstaje z użyciem asystentów kodowania i to jest część nauki: chodzi o to, żeby umieć z nimi pracować odpowiedzialnie.

- Możesz korzystać z asystentów AI przy pisaniu kodu, testów i dokumentacji.
- **Odpowiadasz za każdą linię w swoim PR**, także wygenerowaną. Przeczytaj diff, uruchom testy lokalnie i zrozum, co robi kod, zanim go wyślesz.
- Weryfikuj założenia i wzory finansowe (zaokrąglenia, procenty, daty), bo to miejsca, w których AI mylą się najłatwiej. Pokrywaj je testami z konkretnymi liczbami.
- Nie wklejaj do asystentów sekretów, tokenów ani prawdziwych danych użytkowników.
- Jeśli znaczna część zmiany powstała z pomocą AI, napisz o tym w opisie PR (jedno zdanie, np. jakie narzędzie i do czego). To pomaga w przeglądzie i jest dobrym nawykiem.
- Wnioski z pracy z AI (co zadziałało, co nie) możesz zapisywać w `decisions/` jako krótkie notatki, z których uczy się cały zespół.

## 12. Gdzie pytać

Pytania o zadanie: komentarz pod issue (zostaje w historii). Decyzje produktowe i architektura: komentarz pod issue lub nowy dokument w `decisions/` w tym repozytorium. Problem z dostępem lub środowiskiem: napisz do właściciela (`@poewer`): e-mail **poewer1@gmail.com** albo wiadomość na [**LinkedIn**](https://www.linkedin.com/in/michal-bialek-a48891267/) (Michał Białek).
