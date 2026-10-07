# Tasks

## 1. Bootstrap и конфигурация

- [ ] 1.1 Создать Python 3.12 package layout, `pyproject.toml`, lint/type/test tooling и проверить чистую установку зависимостей командой `pytest`.
- [ ] 1.2 Добавить Dockerfile, `docker-compose.yml`, `.env.example` и healthchecks для app/worker/PostgreSQL; проверить `docker compose config` без секретов.
- [ ] 1.3 Реализовать typed settings с `DRY_RUN=true` по умолчанию, конфигурационными порогами, allowlist и retry limits; проверить тестами default/invalid environment values.

## 2. Domain schemas и persistence

- [ ] 2.1 Реализовать Pydantic-контракты extraction, vacancy, score, profile и application с диапазонами и nullable правилами из `project_artifacts/04-data-model.md`; проверить valid/invalid fixtures.
- [ ] 2.2 Создать SQLAlchemy async models/repositories для channels, source messages, vacancies, scores, profiles, workflow runs, applications и audit events; проверить unique/idempotency constraints интеграционными тестами.
- [ ] 2.3 Добавить Alembic migrations, seed profile/channels и PostgreSQL checkpointer tables; проверить migrate-up/down и rollback на тестовой БД.

## 3. Telegram ingestion

- [ ] 3.1 Реализовать Telethon reader с channel cursor, source URL и транзакционным cursor advance; проверить повторный polling fixture не создаёт дубль.
- [ ] 3.2 Добавить FloodWait/timeout classification и bounded backoff; проверить mocked provider errors сохраняют retryable status и не продвигают cursor.

## 4. Extraction и ranking

- [ ] 4.1 Реализовать provider adapter и extraction node со structured output/Pydantic без ручного JSON-парсинга; проверить vacancy/non-vacancy и malformed-output fixtures.
- [ ] 4.2 Добавить normalization и hard exclusions профиля; проверить canonical skills, salary range, remote tri-state и исключения.
- [ ] 4.3 Реализовать score node с matching/missing/reasons, profile version и threshold route; проверить score range и deterministic `recommended`.
- [ ] 4.4 Реализовать exact/fuzzy/embedding dedup adapters; проверить duplicate precision fixtures и сохранение всех source links.

## 5. LangGraph workflow

- [ ] 5.1 Собрать typed graph nodes/conditional edges из spec и design; проверить unit-тестами ветки non-vacancy, below-threshold и recommended.
- [ ] 5.2 Подключить durable checkpointing, `thread_id`, retry policy и run status transitions; проверить restart fixture продолжает незавершённую node без повторного side effect.

## 6. Digest и human-in-the-loop

- [ ] 6.1 Реализовать aiogram digest с title/company/score/reasons и opaque open/apply/skip callbacks; проверить Telegram length и callback parsing.
- [ ] 6.2 Реализовать authorization, expiry и interrupt/resume для approval; проверить unauthorized/stale/repeated callbacks не меняют чужой run.

## 7. Application и safe send

- [ ] 7.1 Реализовать персонализированный response generator с vacancy/profile/HR context и preview validation; проверить fixture draft содержит personalization facts и проходит length limits.
- [ ] 7.2 Реализовать `send_telegram_message` tool с `DRY_RUN`, allowlist и idempotency key; проверить dry-run не вызывает provider и повтор approved callback не отправляет дубль.

## 8. Observability, security и evals

- [ ] 8.1 Добавить redacted trace/audit spans для nodes/tools с prompt hash, latency, tokens, cost и error class; проверить поиск полного run по `run_id` без секретов.
- [ ] 8.2 Добавить security tests для secret redaction, prompt injection, callback authorization и approval guard; проверить все send-without-approval cases дают ноль отправок.
- [ ] 8.3 Собрать обезличенный eval dataset минимум из 50 сообщений и offline runner для detection/classification/extraction/ranking metrics; проверить отчёт с порогами из `project_artifacts/08-testing-and-evals.md`.

## 9. Интеграционная проверка

- [ ] 9.1 Выполнить end-to-end fixture `message -> extraction -> score -> digest -> approval -> dry-run`; проверить все статусы и audit events.
- [ ] 9.2 Проверить one-command startup, readiness, restart/resume и документацию README; команда `docker compose up --build` должна соответствовать фактическим сервисам.

## Workflow follow-up

- После завершения и review задач выполнить `$openspec-archive-change` и проверить синхронизацию main specs.
- Для следующего самостоятельного изменения создать новый change, а не перегружать этот foundation change.
