# Runbook

## Start locally / Локално стартиране

Requires Python 3.12. Run from the repository root. / Нужен е Python 3.12. Изпълнява се от корена на хранилището.

Linux / macOS:

```bash
python3.12 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
uvicorn app.main:app --reload
```

Windows (PowerShell):

```powershell
py -3.12 -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements-dev.txt
uvicorn app.main:app --reload
```

Without `DATABASE_URL` notes are kept in memory. To use PostgreSQL, set it before starting uvicorn. / Без `DATABASE_URL` бележките са в паметта. За PostgreSQL задайте променливата преди `uvicorn`:

```bash
export DATABASE_URL=postgresql://notes:secret@localhost:5432/notes   # PowerShell: $env:DATABASE_URL = "postgresql://notes:secret@localhost:5432/notes"
```

## Check that it works / Проверка, че работи

The app listens on http://127.0.0.1:8000. / Приложението е на http://127.0.0.1:8000.

```bash
curl http://127.0.0.1:8000/health
# {"status":"ok"}

curl http://127.0.0.1:8000/ready
# {"status":"ready"}

curl -X POST http://127.0.0.1:8000/notes -H "Content-Type: application/json" -d '{"title": "First note", "body": "Hello"}'
# {"id":1,"title":"First note","body":"Hello","category":null}

curl http://127.0.0.1:8000/notes
# [{"id":1,"title":"First note","body":"Hello","category":null}]
```

In a browser, open http://127.0.0.1:8000/docs: you should see the interactive Swagger UI listing all endpoints. / В браузър отворете http://127.0.0.1:8000/docs: трябва да видите Swagger UI с всички адреси.

> On Windows PowerShell use `curl.exe` instead of `curl`. / В PowerShell използвайте `curl.exe` вместо `curl`.

## Run the tests / Тестове

With the virtual environment active: / С активирана виртуална среда:

```bash
pytest
ruff check .
```

All tests should pass. The PostgreSQL tests are skipped unless `TEST_DATABASE_URL` is set; to run them: / Тестовете за PostgreSQL се пропускат, ако `TEST_DATABASE_URL` не е зададена:

```bash
TEST_DATABASE_URL=postgresql://notes:secret@localhost:5432/notes pytest tests/test_postgres.py
```

## When something breaks / Когато нещо се счупи

1. **Is the process alive?** / **Работи ли процесът?**
   ```bash
   curl http://127.0.0.1:8000/health
   ```
   No answer → the app is not running. Check the uvicorn terminal for a traceback and restart with `uvicorn app.main:app --reload`. / Няма отговор → приложението не работи; вижте грешката в терминала на uvicorn и го стартирайте отново.

2. **Can it reach the database?** / **Достига ли базата?**
   ```bash
   curl -i http://127.0.0.1:8000/ready
   ```
   `503 {"status":"storage unavailable"}` → PostgreSQL is down or `DATABASE_URL` is wrong. Check the value and connect directly: / PostgreSQL не работи или `DATABASE_URL` е грешен:
   ```bash
   echo $DATABASE_URL          # PowerShell: $env:DATABASE_URL
   psql "$DATABASE_URL" -c "SELECT 1"
   ```

3. **Are the configuration and dependencies right?** / **Правилни ли са настройките и зависимостите?**
   ```bash
   curl http://127.0.0.1:8000/version
   curl http://127.0.0.1:8000/model
   pip install -r requirements-dev.txt
   ```
   Confirm the expected version is deployed and the right model (or none) is loaded. If the app fails at startup with `ModuleNotFoundError: joblib`, `MODEL_PATH` is set but `scikit-learn`/`joblib` are not installed: install them or unset `MODEL_PATH`. / Проверете версията и модела. При `ModuleNotFoundError: joblib` е зададен `MODEL_PATH` без инсталирани `scikit-learn`/`joblib`.

## Run with Docker / Стартиране с Docker

### Build the image / Изграждане на образа
```bash
docker build -t notes-api:0.3.0 .
```

### Run / Стартиране
```bash
docker run -d --name notes -p 8000:8000 notes-api:0.3.0
```
Проверка: отворете `http://localhost:8000/health`.

Състояние на healthcheck:
```bash
docker inspect --format '{{.State.Health.Status}}' notes
```

### Logs / Логове
```bash
docker logs notes        # всички досега
docker logs -f notes     # следене на живо (Ctrl+C за изход)
```

### Stop / Спиране
```bash
docker stop notes
docker rm notes
```

## Run with Compose / Стартиране с Compose

The app and PostgreSQL start together from `compose.yaml`. / Приложението и PostgreSQL тръгват заедно от `compose.yaml`.

First time only: copy the example and set a real password. / Само първия път: копирайте примера и задайте истинска парола.

```bash
cp .env.example .env        # PowerShell: Copy-Item .env.example .env
```

### Start / Стартиране
```bash
docker compose up -d --build     # build the app image and start app + db
```
`app` starts only after `db` is healthy (`depends_on: condition: service_healthy`). / `app` тръгва чак когато `db` е healthy.

### Status / Състояние
```bash
docker compose ps                # db and app should be (healthy)
curl localhost:8000/ready        # {"status":"ready"}
```

### Logs / Логове
```bash
docker compose logs app          # one service / една услуга
docker compose logs -f           # all services, live (Ctrl+C to exit)
```

### Stop / Спиране
```bash
docker compose down              # removes containers and network, KEEPS the db-data volume
docker compose down -v           # also removes volumes: ALL NOTES ARE DELETED
```
`down` is safe: after `up -d` the notes are still there. `down -v` deletes the `db-data` volume, so the database starts empty. / `down` е безопасно: бележките остават. `down -v` трие volume-а `db-data` и базата започва празна.

### Inspect / Поглед отвътре
```bash
docker compose config                                         # the file after variable substitution (shows the password!)
docker compose exec db psql -U notes -d notes -c "SELECT * FROM notes"
docker compose --profile tools up -d                          # Adminer on http://localhost:8081 (server: db)
```

## Deploy and roll back / Пускане и връщане назад

CI pushes every image to GHCR (`ghcr.io/ivokavalski/devops-labs-fn361223024`). The server builds nothing: `compose.prod.yaml` pulls a ready image. / CI качва всеки образ в GHCR. Сървърът не строи нищо: `compose.prod.yaml` тегли готов образ.

| Tag / Таг | When it appears / Кога се появява |
|-----------|-----------------------------------|
| `:<sha>`  | every push to `main` or a `v*` tag; never changes / при всеки push; никога не се сменя |
| `:latest` | every merge to `main`; moves / при всяко сливане в `main`; мести се |
| `:v0.6.0` | push of git tag `v0.6.0` / при push на git таг |

### Deploy the latest / Пускане на последното
```bash
docker compose -f compose.prod.yaml pull
docker compose -f compose.prod.yaml up -d
curl localhost:8000/ready
```

### Release a version / Издание с версия
```bash
git tag v0.6.0 && git push origin v0.6.0          # CI builds :v0.6.0
APP_TAG=v0.6.0 docker compose -f compose.prod.yaml up -d
```

### Roll back / Връщане назад
Use the full commit SHA of the last working version (or a `v*` tag), never `latest`. / Използвайте пълния SHA на последната работеща версия (или `v*` таг), никога `latest`.
```bash
git log --format=%H -5                            # pick a previous commit / изберете предишен комит
APP_TAG=<sha> docker compose -f compose.prod.yaml up -d
docker compose -f compose.prod.yaml ps            # IMAGE column shows the tag / колоната IMAGE показва тага
```
Notes survive: they live in the `db-data` volume, only the `app` container is replaced. / Бележките остават: те са във volume-а `db-data`, сменя се само контейнерът `app`.