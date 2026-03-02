# Архитектура системы

## 1) Компоненты

### Twenty CRM (self-hosted)
- Postgres
- GraphQL API
- Custom Objects
- UI для администраторов

### Integration Service (Node/Nest)
- Webhook endpoint (YCLIENTS)
- Sync worker
- Outbox publisher
- Rule engine
- Message dispatcher

### Redis + BullMQ
- Очереди на ingestion/sync/outbox/message dispatch

### Telegram Bot (MVP)
- Привязка клиента к `contactId` через link token
- Доставка сообщений в приоритетном канале

---

## 2) Интеграционный pipeline

```text
YCLIENTS webhook
  -> Webhook Controller
  -> WebhookInbox (идемпотентность)
  -> Queue (BullMQ)
  -> Sync Worker
  -> YCLIENTS API (детали)
  -> Twenty GraphQL (upsert Contact/Appointment/Staff/Service/Payment)
  -> Outbox event
  -> Rule Engine
  -> MessageLog (queued)
  -> Message Dispatcher
```

### Шаги обработки webhook
1. Принять webhook.
2. Проверить идемпотентность и записать в `WebhookInbox`.
3. Поставить задачу в очередь.
4. Воркер выполняет sync:
   - подгружает детали из YCLIENTS API;
   - делает upsert сущностей в Twenty;
   - генерирует доменное событие в Outbox.

---

## 3) Outbox pattern

Integration Service **не отправляет уведомления напрямую** после sync.

1. После успешной синхронизации в Twenty пишет событие в `Outbox`.
2. Отдельный publisher читает Outbox и передает событие в Rule Engine.
3. Rule Engine создает `MessageLog` со статусом `queued`.
4. Dispatcher обрабатывает queued-сообщения и отправляет в канал.

### События MVP
- `crm.appointment.created`
- `crm.appointment.rescheduled`
- `crm.appointment.cancelled`
- `crm.appointment.completed`

---

## 4) Правила Rule Engine (MVP)

### remind_24h
- Trigger: `appointment.created`, `appointment.rescheduled`
- Conditions:
  - `notificationConsent = true`
  - `appointment.status` не `cancelled/completed`
  - до визита не меньше 25 часов
- Action:
  - создать `MessageLog`
  - `scheduledAt = startAt - 24h`

### remind_2h
- Trigger: `appointment.created/rescheduled`
- Action: `scheduledAt = startAt - 2h`

### aftercare
- Trigger: `appointment.completed`
- Action: `scheduledAt = now + 30m`

### review_request
- Trigger: `appointment.completed`
- Action: `scheduledAt = now + 24h`

### Reschedule-логика
При `crm.appointment.rescheduled`:
1. Найти queued-сообщения по `appointmentId`.
2. Пометить старые `remind_24h/remind_2h` как `skipped`.
3. Создать новые сообщения с пересчитанным `scheduledAt`.

---

## 5) Нефункциональные требования

- Идемпотентность webhook ingestion и message enqueue.
- Гарантия at-least-once для очередей + дедупликация на уровне `dedupeKey`.
- Наблюдаемость: structured logs + correlation IDs + retry metrics.
- Безопасность: проверка подписи webhook (если доступна), хранение секретов в env/secret manager.

---

## 6) Границы ответственности

Система:
- не заменяет расписание YCLIENTS;
- не заменяет кассу и фискализацию;
- не управляет эквайрингом.
