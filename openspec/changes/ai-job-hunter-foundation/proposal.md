# Proposal

## Why

Нужен единый spec-driven baseline для AI Job Hunter Agent: сейчас требования собраны в аналитических документах, но не оформлены как change, который Codex сможет безопасно продолжить в отдельном чате. OpenSpec связывает бизнес-цель, проверяемые сценарии, архитектурные решения и задачи реализации в версионируемых артефактах.

## What Changes

- Добавляется capability `job-hunter-workflow` для полного цикла от новых Telegram-сообщений до подтверждённого dry-run/production отклика.
- Формализуются structured extraction, нормализация, дедупликация и scoring вакансий.
- Формализуются LangGraph nodes/state/conditional edges, durable checkpointing, retries и human-in-the-loop interrupt/resume.
- Добавляются Telegram digest, approval callbacks, персональный response generator и безопасный sender с `DRY_RUN=true` по умолчанию.
- Добавляются persistence, idempotency и observability requirements.
- Breaking changes отсутствуют: проект находится на стадии подготовки и runtime API ещё не существует.

## Capabilities

### New Capabilities

- `job-hunter-workflow`: сбор, анализ, ранжирование и подтверждённая отправка откликов по Telegram-вакансиям.

### Modified Capabilities

- Нет: существующих OpenSpec specs в проекте не было.

## Impact

- Будущие Python-модули в `src/ai_job_hunter/` для agents, workflows, tools, services, bot и db.
- PostgreSQL/SQLite persistence, LangGraph checkpointer, Telegram API (Telethon и aiogram), LLM provider adapter и embeddings.
- Docker Compose, `.env.example`, миграции, тесты, eval dataset и tracing.
- Существующие аналитические документы в `project_artifacts/` остаются источником проектных решений; OpenSpec change превращает их в исполнимый план для конкретного цикла реализации.
