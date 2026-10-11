# Фронтенд EndPoint в Docker

Загрузите файлы архива поверх существующего проекта. Сохраните серверный .env.
Backend должен быть запущен как api в том же Compose-проекте.
Сохраните прежнее имя Compose-проекта при обновлении.
Остановите прежний процесс npm/PM2 на порту 3001.

## Готовый образ, без сборки

Загрузите endpoint-frontend.tar на сервер и выполните из корня проекта:

```bash
sudo docker load -i /путь/endpoint-frontend.tar
sudo docker compose -f compose.yml up -d --no-build --no-deps frontend
```

## Сборка на сервере

```bash
sudo docker compose -f compose.yml up -d --build --no-deps frontend
```

## Проверка

```bash
sudo docker compose -f compose.yml ps frontend
sudo docker compose -f compose.yml logs --tail 50 frontend
curl -I http://127.0.0.1:3001/
```

Nginx продолжает проксировать http://127.0.0.1:3001.
Compose устанавливает внутри контейнера API_URL=http://api:8000.
Единый .env используется Compose из корня. Секреты backend фронту не передаются.
Готовый образ рассчитан на Linux amd64. Из исходников можно собрать образ под другую архитектуру.
Не запускайте docker compose down -v при обновлении.
