# Развёртывание EndPoint

## Требования

Linux x86-64, Docker Engine и Compose. Node.js 22 включён в образ frontend. Для демо рекомендуется
4 vCPU / 8 ГБ RAM / SSD от 80 ГБ. 4 ГБ RAM — ограниченный стенд, требует измерения нагрузки.
LLM использует внешний API; GPU не требуется. Judge0 желательно вынести на отдельную ВМ.
Версия ядра/cgroups должна соответствовать требованиям выбранной версии Judge0.

## Установка

1. Клонировать main репозитория Artem-Drynov/EndPoint, содержащую backend и frontend.
2. Скопировать `.env.example` в `.env`, задать уникальные секреты всех сервисов.
3. Заполнить GigaChat credentials и проверить сертификаты. Не считать тариф автоматически бесплатным.
4. Подготовить внешний Judge0, заменить `JUDGE0_URL` в Compose.
5. Запустить `docker compose up -d --build`; проверить `/health` и `/api/docs`.
6. Сервис frontend собирает интерфейс и запускает Express в Docker. Отдельный npm start не нужен.
7. Установить reverse proxy на порт фронтенда, домен и HTTPS. API/DB/Redis не публиковать наружу.

## Обновление только фронтенда

Сохранить серверный `.env`, загрузить папку `frontend` и обновлённый `compose.yml`.
Остановить прежний npm/PM2 процесс на порту 3001, затем из корня проекта:

```bash
docker compose -f compose.yml up -d --build --no-deps frontend
docker compose -f compose.yml ps frontend
docker compose -f compose.yml logs --tail 50 frontend
```

Backend должен уже работать в том же Compose-проекте как сервис api.
Не менять имя Compose-проекта при обновлении существующего стенда, чтобы сохранить volumes.
Nginx продолжает использовать `http://127.0.0.1:3001`.
В контейнере фронта API_URL задаётся как `http://api:8000`, независимо от значения для локального npm start.

## Данные и резервирование

Переносить PostgreSQL через pg_dump/pg_restore, файлы — из Docker volume endpoint-uploads.
Отдельно резервировать БД Reqcore. Сохранять исходные секреты шифрования и JWT при миграции.
Копии хранить вне сервера, проверить восстановление. Дамп и приватные файлы не коммитить.
Архив вакансий хранится в основной БД; это не резервная копия.

## Демонстрационный и рабочий режимы

Seed создаёт candidate@demo.local, employer@demo.local, admin@demo.local с demo1234 только
для демонстрации. Дополнительные локальные аккаунты не переносятся через Git.
В production сменить доступы, отключить BILLING_DEMO_ENABLED, настроить email и FSP.
Ollama профиль local-llm выключен; ATS через GigaChat настроена, но успешный сквозной отчёт
на текущем стенде пока не подтверждён. Не подменять ошибки AI оценкой «не подходит».

## Диагностика

`docker compose ps`, `docker compose logs --tail 100 api`, /health. Не публиковать логи
с Authorization, ключами или текстом резюме. Для SSH нужны правильный PEM-ключ, работающий sshd
и открытый порт 22; ошибка до SSH banner не связана с проверкой ключа.
