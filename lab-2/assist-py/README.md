## Хранение данных

### Что создаётся

| Ресурс | Имя | Назначение |
| --- | --- | --- |
| Volume | `ter-stepanyan-pgdata` | файлы базы PostgreSQL, сохраняются при пересоздании контейнеров |
| Сеть | `ter-stepanyan-backend` | пользовательская сеть: приложение находит БД по имени контейнера |
| Bind mount | `./config` → `/app/config` (read-only) | файл конфигурации `app.conf` с хоста |

### Создание volume и сети

```bash
docker volume create ter-stepanyan-pgdata
docker network create ter-stepanyan-backend
```

### Запуск PostgreSQL

```bash
docker run -d \
--name ter-stepanyan-postgres \
--network ter-stepanyan-backend \
-e POSTGRES_USER=user_97 \
-e POSTGRES_PASSWORD=pgpass_97 \
-e POSTGRES_DB=assistant_97 \
-v ter-stepanyan-pgdata:/var/lib/postgresql/data \
postgres:15
```

### Сборка и запуск приложения

```bash
docker build -f Dockerfile.optimized -t ter-stepanyan-assistant:v3 .

docker run -d \
--name ter-stepanyan-assistant \
--network ter-stepanyan-backend \
-p 8097:8097 \
-e APP_OWNER=ter-stepanyan \
-e APP_SECRET=secret97ter-stepanyan \
-e DATABASE_URL="postgresql://user_97:pgpass_97@ter-stepanyan-postgres:5432/assistant_97" \
-v $(pwd)/config:/app/config:ro \
ter-stepanyan-assistant:v3
```

### Как проверить

```bash
curl -X POST http://localhost:8097/notes \
-H "Content-Type: application/json" \
-d '{"text": "test"}'
curl http://localhost:8097/notes
```

Заметки сохраняются после удаления и пересоздания контейнеров, так как данные лежат в volume `ter-stepanyan-pgdata`.
