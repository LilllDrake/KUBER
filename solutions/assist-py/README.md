## Docker Compose

Весь стек поднимается через Docker Compose.

### Из чего состоит стек

| Сервис | Контейнер | Что делает |
|---|---|---|
| `app` | `ter-stepanyan-assistant` | FastAPI-приложение, собирается из `Dockerfile.optimized` |
| `db` | `ter-stepanyan-postgres` | PostgreSQL 15, хранит заметки, данные в томе `ter-stepanyan-pgdata` |
| `redis` | `ter-stepanyan-redis` | Redis 7, кэш погоды (`/weather/{city}`) на `CACHE_TTL` секунд |

Все сервисы в сети `ter-stepanyan-backend` и обращаются друг к другу по имени сервиса (`db`, `redis`).
`app` стартует только после того, как `db` и `redis` пройдут healthcheck.

### Запуск

```bash
cp .env.example .env    # заполнить пароль и секрет
docker compose up -d --build
docker compose ps
```

### Порты

| Порт на хосте | Куда ведёт |
|---|---|
| `8097` | приложение (`APP_PORT`) |
| `6467` | Redis (`REDIS_PORT`), нужен только для доступа с хоста |

PostgreSQL слушает `5527` (`PG_PORT`) внутри сети, наружу не опубликован.

Проверка:

```bash
curl http://localhost:8097/health
curl http://localhost:8097/weather/Moscow     # первый раз source: api, второй source: cache
curl http://localhost:8097/cache/stats
```

### Масштабирование

Для масштабирования есть `docker-compose.scale.yml`: у `app` нет `container_name`, а порт на хосте Docker выбирает сам.

```bash
docker compose down
docker compose -f docker-compose.scale.yml up -d --build --scale app=3
docker compose -f docker-compose.scale.yml ps app    # какие порты достались копиям
```

Все копии используют общие PostgreSQL и Redis, поэтому кэш у них общий.

Вернуться к основной конфигурации:

```bash
docker compose -f docker-compose.scale.yml down
docker compose up -d
```
