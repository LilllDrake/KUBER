# Лабораторная 1
## Этап 1. Верификация окружения и архитектуры
### Задание 1.1. Проверка установки Docker

Для выполнения работы необходим установленный и запущенный Docker.

Проверить версии клиента и сервера:

```bash
docker version --format 'Client: {{.Client.Version}}, Server: {{.Server.Version}}'
```

Проверить тип операционной системы и архитектуру Docker Engine:

```bash
docker info --format '{{.OSType}}/{{.Architecture}}'
```
`linux/aarch64`

Версия клиента: 29.1.3 </br>
Версия сервера: 29.1.3
______
### Задание 1.2. Исследование состояния локального хранилища

отображение использования диска для четырех типов объектов:
```bash
docker system df --format "table {{.Type}}\t{{.TotalCount}}\t{{.Active}}\t{{.Size}}"
```
`результат`: локальных образов и запущенных контенйеров нет
______
### Задание 1.3. Артефакт окружения

артефакт окружения:
``` bash
echo "Student: $(whoami)@$(hostname)" && \
echo "Date: $(date '+%Y-%m-%d %H:%M:%S')" && \
docker version --format "Docker {{.Server.Version}} on {{.Server.Os}}/{{.Server.Architecture}}"
```

Student: lildrake@linux-lab </br>
Date: 2026-09-23 23:04:48 </br>
Docker 29.1.3 on linux/arm64 

1. Если ввести команду docker system prune без флагов, то она удалит остановленные контейнеры, сети, которые не используются, образы без тега и неиспользуемый кеш сборки
2. docker version может завершиться ошибкой, даже если Docker установлен, когда у пользователя нет прав на подключение к /var/run/docker.sock. Тогда Docker CLI не может отправить запрос демону dockerd и получить версию сервера.

______
## Этап 2. Первые шаги: «игрушечные» образы
### Задание 2.1. Запуск классического примера
```bash
docker run --rm hello-world
```
Образ загружается из `docker.io/library/hello-world:latest` </br>
Сам процесс называется `hello`
_____
### Задание 2.2. Исследование минимального образа
Скачивание образа и запуск контейнера:
```bash
docker run --rm alpine:3.18 cat /etc/os-release
```

Значение поля: `PRETTY_NAME="Alpine Linux v3.18"` </br>
Размер образа: `3352365`
______
### Здание 2.3. Сравнение поведения двух контейнеров
Создание контейнеров:
```bash
docker run -d --name ter-stepanyan-alpine-1 alpine:3.18 sleep 300
docker run -d --name ter-stepanyan-alpine-2 alpine:3.18 sh -c "echo 'Hello from $(hostname)' && sleep 300"
```
Проверка статуса:
```bash
docker ps --filter "ter-stepanyan-alpine"
```

образ `hello-world` завершается сразу после запуска, а `alpine:3.18 sleep 300` работает 5 минут, потому что при написании команды для запуска контейнера мы использовали флаг `–-rm`, который означает, что после того, как процесс завершится, контейнер остановится и удалится. Для второй команды мы сразу задаем ему находиться в состоянии Up 5 минут, так как главный процесс не завершается. 


Если убрать флаг `--rm`, контейнер все равно завершится, но останется в списке остановленных контейнеров. Для удаления контейнера можно использовать команду `docker rm c63ceda0ffd8` либо `docker rm ter-stepanyan-alpine-1`.
_______
## Этап 3. Жизненный цикл контейнера
### Задание 3.1. Отслеживание всех состояний через последовательные команды
```bash
docker create --name ter-stepanyan-nginx nginx:alpine
docker start --name ter-stepanyan-nginx nginx:alpine
docker stop --name ter-stepanyan-nginx nginx:alpine
docker restart --name ter-stepanyan-nginx nginx:alpine
docker kill --name ter-stepanyan-nginx nginx:alpine
docker rm --name ter-stepanyan-nginx nginx:alpine
```
_______
### Задание 3.2. Фиксация переходов состояний
`docker create` - `STATUS:Created` </br>
`docker start` - `STATUS:Up` </br>
`docker stop` - `STATUS:Exited` </br>
`docker restart` - `STATUS:Up` </br>
`docker kill` - `STATUS:Exited` </br>
`docker rm` - `STATUS:`

`docker stop` отправляет сигнал `SIGTERM`

docker kill принудительно завершает процесс БД сигналом SIGKILL. База не успевает завершить работу, незавершённые операции прерываются, а данные, которые ещё не были надёжно записаны на диск, могут потеряться. 

Если выполнить docker rm для запущенного контейнера, он не удалится и Docker выдаст ошибку. Чтобы выполнить принудительное удаление, нужно добавить флаг -f.
_____
## Этап 4. Диагностика реального сервиса
### Задание 4.1. Запуск с ошибкой

```bash
docker run -d \
  --name ter-stepanyan-pg-broken \
  -p 5527:5432 \
  postgres:15
```
Диагностика:
```bash
# Статус контейнера
docker ps -a --filter "name=ter-stepanyan-pg"

# Логи с временной меткой (последние 20 строк)
docker logs --timestamps ter-stepanyan-pg-broken 2>&1 | tail -20

# Код завершения
docker inspect ter-stepanyan-pg-broken --format='{{.State.ExitCode}}'
```
_______
### Задание 4.2. Исправление и проверка
```bash
docker run -d \
  --name ter-stepanyan-pg-fixed \
  -e POSTGRES_PASSWORD=Pass_5527 \
  -e POSTGRES_USER=user_467697 \
  -p 5527:5432 \
  postgres:15
```
Проверка работоспособности:
```bash
# Ждём инициализацию БД
sleep 15

# Выполняем запрос к БД
docker exec ter-stepanyan-pg-fixed psql -U user_467697 -c "SELECT version();"
```
________
### Задание 4.3. Финальный артефакт
```bash
echo "=== Практика №1: $(date '+%Y-%m-%d %H:%M:%S') ===" && \
echo "Студент: $(whoami)@$(hostname)" && \
docker ps --filter "name=ter-stepanyan-pg-fixed" --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```
_________
```bash
=== Практика №1: 2026-09-24 01:30:37 ===
Студент: lildrake@linux-lab
NAMES                    STATUS         PORTS
ter-stepanyan-pg-fixed   Up 5 minutes   0.0.0.0:5527->5432/tcp, [::]:5527->5432/tcp
```



