# Design

## Context

Репозиторий пока содержит спецификации, но не runtime-код. Исходный анализ, модели данных, workflow и операционные ограничения находятся в `project_artifacts/00-10`. Этот change превращает их в последовательный контракт для первой реализации.

## Goals / Non-Goals

**Goals:**

- Разделить deterministic I/O, валидацию, LLM interpretation и внешние side effects.
- Сохранить durable state для LangGraph interrupt/resume и повторных запусков.
- Сделать отправку HR безопасной по умолчанию и полностью наблюдаемой.
- Оставить provider, пороги и источники конфигурационными.

**Non-Goals:**

- Полноценный web-интерфейс или мультитенантность.
- Автономная рассылка без human approval.
- RAG, memory, MCP и agentic company research в первой реализации.

## Decisions

1. **Оркестрация:** LangGraph graph с отдельными nodes `collect`, `extract`, `validate`, `deduplicate`, `score`, `route`, `digest`, `human_approval`, `generate_response`, `send`. Это позволяет checkpoint/resume и локальные retries; один большой prompt исключён.
2. **Контракты:** Pydantic 2 structured output используется для extraction, scoring и application draft. Ручное извлечение JSON из текста не допускается.
3. **Хранилище:** PostgreSQL хранит domain tables и durable checkpoints в production; SQLite разрешён только для быстрых тестов. SQLAlchemy async и Alembic отделяют persistence от graph logic.
4. **Telegram:** Telethon читает каналы, aiogram обслуживает digest/callback UI, sender является отдельным mutating tool. Cursor и idempotency коммитятся транзакционно.
5. **Дедупликация:** exact source key и normalized hash проверяются первыми; embeddings добавляются как semantic signal, но не заменяют сохранение всех источников.
6. **Safety:** `DRY_RUN=true`, allowlisted candidate/admin ids, opaque callbacks, explicit approval и уникальный idempotency key защищают от случайной отправки.
7. **Наблюдаемость:** каждый node/tool пишет trace id, node, prompt version/hash, latency, token/cost metadata и redacted outcome. Секреты и необрезанный PII в traces не сохраняются.
8. **DDD и abstract interfaces:** domain layer владеет сущностями и инвариантами и не зависит от внешних SDK; application layer использует abstract interfaces/ports; Telegram, LLM, SQLAlchemy и checkpoint providers подключаются infrastructure adapters.

## Risks / Trade-offs

- **[Риск] LLM ошибочно классифицирует вакансию** → Pydantic validation, hard exclusions, threshold и human approval.
- **[Риск] Повторный callback вызывает повторную отправку** → уникальный idempotency key, send state machine и transactional guard.
- **[Риск] Telegram FloodWait** → уважать `retry_after`, ограничить sender concurrency и не делать массовую рассылку.
- **[Риск] Embeddings дают ложный duplicate** → exact/fuzzy checks, similarity evidence и review path.
- **[Риск] PostgreSQL недоступен** → readiness check, retryable run status и backup/restore procedure из `project_artifacts/10_operations.md`.

## Migration Plan

1. Создать package layout, config и Docker Compose.
2. Добавить schemas и миграции без внешних LLM/Telegram вызовов.
3. Реализовать ingestion, затем extraction/scoring и dedup fixtures.
4. Подключить graph/checkpointer и Telegram digest в dry-run.
5. Добавить evals/tracing; production send разрешать отдельной конфигурацией после smoke test.

Rollback: остановить worker, проверить `workflow_runs` и `applications`, вернуть предыдущий image и не продвигать cursor до подтверждения целостности. Схема БД откатывается только через проверенную migration strategy.

## Open Questions

- Какой конкретный LLM provider/model будет выбран для production budget.
- Какая политика retention допустима для raw Telegram text и traces.
- Какие Telegram user ids и HR recipients войдут в production allowlist.
