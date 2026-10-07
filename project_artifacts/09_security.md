# 09. Безопасность и безопасные настройки

## Активы

- Telegram API credentials, bot token и session string.
- Профиль кандидата, raw vacancy text и HR contacts.
- Checkpoints и approval state.
- LLM prompts, outputs, tracing data и database backups.

## Основные угрозы и controls

| Угроза | Последствие | Контроль |
| --- | --- | --- |
| Токен/Telethon session попал в git или log | Захват аккаунта | `.env`, secret redaction, pre-commit scan, rotation, никогда не логировать env |
| Prompt injection в вакансии | LLM игнорирует policy или раскрывает context | raw text считать недоверенным, delimiters, schema validation, не передавать секреты в prompt |
| Поддельный/старый callback | Несанкционированная отправка | opaque nonce, owner check, status check, expiry, approval guard |
| Повторная доставка/worker retry | Дубли HR-сообщений | unique idempotency key и transactional send state |
| Telegram rate limit | блокировка/потеря доступности | backoff, FloodWait handling, single sender, no bulk outreach |
| PII в tracing | утечка резюме/контактов | redaction, payload hashes, retention, access control |
| Модель ошибочно рекомендует vacancy | нежелательный отклик | hard exclusions + human approval + dry-run |
| LLM output не соответствует схеме | corrupt data | SDK structured output + Pydantic + bounded repair |

## Safety gates

1. `DRY_RUN=true` — default и обязательный local development mode.
2. Реальная отправка возможна только при `approved`, непустом allowlisted HR contact и явном `DRY_RUN=false`.
3. Кнопки Telegram не передают доверенные бизнес-данные; всё загружается по id из БД.
4. Логи редактируют usernames, email, phone, tokens и исходный текст по умолчанию.
5. Сессия Telethon хранится в secret volume, а не в рабочей директории.

## До production

- Ограничить Telegram admin/candidate user id allowlist.
- Добавить dependency/image scanning и backup encryption.
- Задать retention policy для raw messages, prompts и traces.
- Проверить права PostgreSQL роли: app не должна иметь DDL в runtime.
- Провести ручной threat review после появления фактического кода и deployment topology.
