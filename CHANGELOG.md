# Changelog

## Unreleased

## v0.4.0 - 2026-10-26

- `compose.yaml`: the app and PostgreSQL 17 start with one command (`docker compose up -d --build`).
- The `db` service has a healthcheck (`pg_isready`), and `app` waits for it with `depends_on: service_healthy`.
- Notes are kept on the named volume `db-data`, so they survive `docker compose down`.
- Secrets live in `.env` (not committed). `.env.example` is the template.
- Optional Adminer on port 8081 via `docker compose --profile tools up -d`.
- Docs: "Run with Compose" section in the runbook, and ADR 0002 (PostgreSQL, Accepted).

## v0.3.0

- `Dockerfile` and `.dockerignore`: the app runs as a non-root container with a healthcheck.

## v0.1.0

- Initial template: notes API with health and version endpoints.
