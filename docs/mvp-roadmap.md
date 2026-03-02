# MVP Roadmap

1. Развернуть Twenty CRM.
2. Создать custom objects: Appointment, Staff, Service, Payment.
3. Поднять Integration Service (Node/Nest) + Redis/BullMQ.
4. Реализовать webhook ingestion (`WebhookInbox` + queue).
5. Реализовать sync/upsert в Twenty GraphQL.
6. Реализовать Outbox + Rule Engine с правилом `remind_24h`.
7. Подключить Telegram bot и flow `/start <token>`.
8. Добавить правило `remind_2h`.
9. Добавить post-visit правила `aftercare` и `review_request`.
