# Лабораторная 1
## Задание 1.1. Проверка установки Docker

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
## Задание 1.2. Исследование состояния локального хранилища

отображение использования диска для четырех типов объектов:
```bash
docker system df --format "table {{.Type}}\t{{.TotalCount}}\t{{.Active}}\t{{.Size}}"
```
`результат`: локальных образов и запущенных контенйеров нет
______
## Задание 1.3. Артефакт окружения

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




