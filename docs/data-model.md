# Модель данных

## 1) Twenty CRM

## Contact (стандартный объект)
Поля:
- `firstName`
- `lastName`
- `phone` (dedupe key, E.164)
- `email`
- `birthdate`
- `marketingConsent` (custom)
- `notificationConsent` (custom)
- `lastVisitAt` (custom)
- `totalSpent` (custom)

## Appointment (custom object)
Поля:
- `startAt` (DateTime)
- `endAt` (DateTime)
- `status` (enum)
- `totalAmount` (number)
- `source` (enum: `yclients` | `manual`)
- `yclientsId` (string)
- `rawPayload` (JSON)

Связи:
- many-to-one -> Contact
- many-to-one -> Staff
- many-to-many -> Service

## Staff (custom object)
Поля:
- `name`
- `role`
- `isActive`
- `yclientsId`

## Service (custom object)
Поля:
- `name`
- `durationMin`
- `basePrice`
- `isActive`
- `yclientsId`

## Payment (custom object)
Поля:
- `amount`
- `method`
- `status`
- `paidAt`
- `receiptUrl`
- `yclientsId`
- `rawPayload`

Связи:
- many-to-one -> Contact
- many-to-one -> Appointment

---

## 2) Integration Service DB

## WebhookInbox
Назначение: идемпотентный прием событий от YCLIENTS.

Рекомендуемые поля:
- `id`
- `tenantId`
- `source` (`yclients`)
- `externalEventId`
- `eventType`
- `payload` (JSONB)
- `receivedAt`
- `processedAt`
- `status` (`received`/`queued`/`processed`/`failed`)

Уникальность:
- (`tenantId`, `source`, `externalEventId`)

## Outbox
Назначение: надежная публикация доменных событий в Rule Engine.

Поля:
- `id`
- `tenantId`
- `eventType`
- `aggregateType`
- `aggregateId`
- `payload` (JSONB)
- `createdAt`
- `publishedAt`
- `status` (`pending`/`published`/`failed`)

## MessageLog
Назначение: контроль жизненного цикла уведомлений.

Поля:
- `id`
- `tenantId`
- `contactId` (Twenty ID)
- `appointmentId` (Twenty ID)
- `channel`
- `templateCode`
- `status` (`queued`/`sent`/`failed`/`skipped`)
- `scheduledAt`
- `sentAt`
- `providerMessageId`
- `dedupeKey`

Уникальность:
- (`tenantId`, `dedupeKey`)
