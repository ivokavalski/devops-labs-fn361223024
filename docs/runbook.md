# Runbook## Стартиране с Docker

### Изграждане на образа
```bash
docker build -t notes-api:0.3.0 .
```

### Стартиране
```bash
docker run -d --name notes -p 8000:8000 notes-api:0.3.0
```
Проверка: отворете `http://localhost:8000/health`.

Състояние на healthcheck:
```bash
docker inspect --format '{{.State.Health.Status}}' notes
```

### Логове
```bash
docker logs notes        # всички досега
docker logs -f notes     # следене на живо (Ctrl+C за изход)
```

### Спиране
```bash
docker stop notes
docker rm notes
```
