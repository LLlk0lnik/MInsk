# 07. Поэтапный план реализации

Каждый этап должен завершаться тестами и коротким demo; следующий этап не должен скрывать незакрытые acceptance criteria предыдущего.

## Этап 0 — Bootstrap и окружение

**Вход:** этот пакет спецификаций. **Сделать:** `pyproject.toml`, src layout, ruff/mypy/pytest, Dockerfile, compose с app/worker/postgres, `.env.example`, базовый logging. **Выход:** контейнеры поднимаются, healthcheck отвечает. **DoD:** clean install и `pytest` запускаются одной командой.

## Этап 1 — Domain schemas и profile

**Сделать:** `VacancyExtraction`, `Vacancy`, `VacancyScore`, `CandidateProfile`, `ApplicationDraft`, validators и profile loader. **Параметры:** score 0..100, строки с лимитами из 04, schema version. **DoD:** unit tests на valid/invalid/nullable cases.

## Этап 2 — Persistence и миграции

**Сделать:** SQLAlchemy models/repositories для таблиц из 04, Alembic, transaction boundaries, seed profile/channels. **DoD:** migrate-up/down, unique constraints и rollback tests.

## Этап 3 — Telegram ingestion

**Сделать:** Telethon reader, channel config, cursor, source URL builder, flood-wait handling. **DoD:** fixture/patched client обрабатывает только новые messages, повтор не дублирует запись.

## Этап 4 — Extraction и normalization

**Сделать:** LangChain model adapter, structured output, extraction prompt, validation/repair path, normalization. **DoD:** dataset messages classified, manual JSON parsing отсутствует, provider errors typed.

## Этап 5 — Deduplication и scoring

**Сделать:** exact hash, fuzzy baseline, embeddings adapter, scorer prompt, hard exclusions and threshold route. **DoD:** duplicate fixture links to canonical vacancy; score explains match/missing skills.

## Этап 6 — LangGraph и durable checkpointing

**Сделать:** graph nodes/edges from 05, PostgreSQL checkpointer, retries, run/audit records. **DoD:** killed worker resumes at last checkpoint without repeating side effects.

## Этап 7 — Digest и human approval

**Сделать:** aiogram handlers, inline keyboard, opaque callbacks, digest batching, authorization checks. **DoD:** candidate can open/skip/apply; interrupt survives restart and rejects unauthorized callback.

## Этап 8 — Response и safe send

**Сделать:** personalized response prompt, preview screen, second confirmation, dry-run sender, idempotency. **DoD:** no send without approval; dry-run produces auditable preview.

## Этап 9 — Observability

**Сделать:** trace spans per node/tool, prompt hash, latency/tokens/cost, redaction, error dashboard/log queries. **DoD:** one run is inspectable end-to-end by `run_id`.

## Этап 10 — Evals, hardening и demo

**Сделать:** 50-message eval set, CI, security checks, backup/restore notes, demo script and README examples. **DoD:** metrics meet 08, compose one-command start, full acceptance walkthrough recorded.

## Обязательные контрольные точки

- После этапа 2: данные и миграции не зависят от LLM.
- После этапа 6: workflow восстанавливается после рестарта.
- После этапа 8: реальная отправка всё ещё выключена по умолчанию.
- Перед production: отдельный smoke test с allowlisted HR и explicit `DRY_RUN=false`.
