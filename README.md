# Salon CRM надстройка (Twenty CRM + YCLIENTS)

Этот репозиторий содержит технический blueprint MVP для CRM-надстройки салона красоты:

- **Twenty CRM** — ядро данных и UI для администраторов
- **YCLIENTS** — source of truth по расписанию и онлайн-записи
- **Integration Service (Node/Nest)** — интеграция, rule engine, outbox, отправка уведомлений

## Цели MVP

1. Синхронизировать клиентов и записи из YCLIENTS в Twenty.
2. Запускать автоматические уведомления по событиям записи.
3. Использовать Telegram как первый канал доставки.

## Состав документации

- `docs/architecture.md` — архитектура, потоки, компоненты.
- `docs/data-model.md` — модель данных Twenty и Integration Service.
- `docs/mvp-roadmap.md` — этапы внедрения MVP.

## Высокоуровневая схема

```text
YCLIENTS
    ↓ webhooks
Integration Service
    ↓ GraphQL
Twenty CRM (ядро данных + UI)
    ↓ события
Rule Engine
    ↓
Message Dispatcher
    ↓
Telegram / WA / SMS
```

## Быстрый старт: Twenty CRM в Docker

1. Скопируйте env-файл:

   ```bash
   cp .env.example .env
   ```

2. (Важно) Замените секреты в `.env` на безопасные значения.

3. Запустите сервисы:

   ```bash
   docker compose up -d
   ```

4. Откройте UI Twenty CRM:

   - `http://localhost:3000`

5. Для остановки:

   ```bash
   docker compose down
   ```

Файлы для старта:
- `docker-compose.yml`
- `.env.example`
