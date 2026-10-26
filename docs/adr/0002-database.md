# 0002. Store notes in PostgreSQL / Бележките се пазят в PostgreSQL

**Status / Статус:** Accepted

## Context / Контекст

Without `DATABASE_URL` the app keeps notes in memory, so every restart or redeploy loses them. We need durable storage that runs the same way locally, in Codespaces and in production. / Без `DATABASE_URL` бележките са в паметта и се губят при всеки рестарт. Нужно е трайно хранилище, което работи еднакво локално, в Codespace и в продукция.

## Options / Варианти

- **In memory / В паметта:** no setup and fast tests, but data is lost on restart and cannot be shared between instances. / Без настройка, но данните се губят и не се споделят между инстанции.
- **SQLite:** a single file with no server, but it needs a persistent volume per instance, writes are poorly concurrent, and it does not scale to several app replicas. / Един файл без сървър, но слаба конкурентност при запис и не става за няколко реплики.
- **PostgreSQL:** a real server with concurrent access and an official Docker image (`postgres:17-alpine`), and the app already supports it through `psycopg`. Cons: one more service to run and keep healthy. / Истински сървър, официален образ, приложението вече го поддържа. Минус: още една услуга за поддръжка.

## Decision / Решение

Use PostgreSQL 17. In development it runs as the `db` service in `compose.yaml`, with its data on the named volume `db-data`. The app connects through `DATABASE_URL`, and the credentials come from `.env`, which is not committed. The in-memory store stays for tests and quick local runs. / Използваме PostgreSQL 17 като услуга `db` в `compose.yaml` с volume `db-data`; връзка чрез `DATABASE_URL`, тайните са в `.env`. Хранилището в паметта остава за тестове.

## Consequences / Последствия

- Notes survive `docker compose down` and are lost only with `down -v`. / Бележките оцеляват при `down`, губят се само при `down -v`.
- `app` waits for the `db` healthcheck (`pg_isready`), and `/ready` returns 503 when the database cannot be reached. / `app` чака healthcheck-а на `db`; `/ready` връща 503 без база.
- Backups and schema migrations become our responsibility. / Архивите и миграциите стават наша отговорност.
