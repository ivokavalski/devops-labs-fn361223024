# Service / Услугата

## Purpose / Предназначение

Notes API is a small REST service for creating and reading short text notes, written in Python with FastAPI. Each note has a `title` (required, 1 to 100 characters) and an optional `body`. When a trained ML model is configured, each new note also gets a `category` automatically. The app is deliberately simple: it is the starting point for the DevOps course labs.

Notes API е малка REST услуга за създаване и четене на кратки текстови бележки, написана на Python с FastAPI. Всяка бележка има `title` (задължително, от 1 до 100 знака) и незадължително `body`. Когато е зададен обучен ML модел, всяка нова бележка автоматично получава и `category`. Приложението нарочно е просто: то е отправната точка за упражненията в курса по DevOps.

## Endpoints / Адреси

| Method | Path | What it does / Какво прави |
|---|---|---|
| GET | /health | Liveness check: always returns `{"status": "ok"}` while the process is running. / Проверка дали приложението работи: връща `{"status": "ok"}`. |
| GET | /ready | Readiness check: tests whether the storage can be reached. Returns `{"status": "ready"}`, or 503 with `{"status": "storage unavailable"}` if the database is down. / Готово ли е да обслужва: проверява базата; връща 503, ако не я достига. |
| GET | /version | Returns the app version from `APP_VERSION` (default `0.1.0`). / Връща версията от `APP_VERSION` (по подразбиране `0.1.0`). |
| GET | /model | Returns the path of the loaded ML model, or `null` if none is loaded. / Връща пътя до заредения ML модел или `null`, ако няма такъв. |
| GET | /notes | Lists all notes, ordered by id. / Списък с всички бележки, подредени по id. |
| POST | /notes | Creates a note from JSON `{"title": "...", "body": "..."}` and returns it with status 201; 422 if `title` is missing or too long. / Създава бележка и я връща със статус 201; 422 при невалидно `title`. |
| GET | /notes/{id} | Returns one note by its id; 404 if it does not exist. / Връща една бележка по id; 404, ако не съществува. |

## Dependencies / Зависимости

- **Python 3.12** with `fastapi`, `uvicorn` and `psycopg[binary]` (see `requirements.txt`). / **Python 3.12** с пакетите от `requirements.txt`.
- **PostgreSQL** (optional): used when `DATABASE_URL` is set, e.g. `postgresql://notes:secret@db:5432/notes`. The `notes` table is created automatically. Without `DATABASE_URL` notes are kept in memory and lost on restart. / **PostgreSQL** (по желание): използва се при зададен `DATABASE_URL`; без него бележките са в паметта и се губят при рестарт.
- **ML model** (optional): a joblib file pointed to by `MODEL_PATH`; requires `scikit-learn` and `joblib`. Without it notes have no category. / **ML модел** (по желание): файл, посочен от `MODEL_PATH`; изисква `scikit-learn` и `joblib`.

| Variable / Променлива | Default / По подразбиране | Purpose / Предназначение |
|---|---|---|
| `DATABASE_URL` | not set (in-memory) | PostgreSQL connection string |
| `APP_VERSION` | `0.1.0` | Value returned by `/version` |
| `MODEL_PATH` | not set (no model) | Path to the category model |
