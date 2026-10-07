# AI Job Hunter Agent

Спецификационный каркас pet-проекта из задания `ai_job_hunter_assignment.md`. Цель проекта — автоматически собирать новые вакансии из Telegram, извлекать из них структурированные данные, оценивать соответствие профилю Python-разработчика, показывать дайджест и после явного подтверждения пользователя генерировать и отправлять персональный отклик HR.

Текущий статус: подготовлены декомпозиция требований, технические спецификации, архитектурные решения, план реализации и локальный набор Codex skills. Код приложения, `docker-compose.yml` и миграции пока не реализованы; это намеренно зафиксировано в артефактах, чтобы следующий этап разработки шёл по согласованному ТЗ.

Локальная карта контекста для новых чатов — `PROJECT_CONTEXT.md`. Она намеренно не коммитится и содержит машинный путь/локальные настройки. В Git-версии проекта постоянными источниками остаются `AGENTS.md`, `project_artifacts/` и `openspec/`.

Проект также инициализирован в [OpenSpec](https://github.com/Fission-AI/OpenSpec) `1.14.1`. OpenSpec хранит контекст работы в Git-артефактах и позволяет вести отдельные чаты для отдельных изменений, не смешивая планирование и реализацию.

## Что используется

- Python 3.12+, async/await.
- Telethon для чтения каналов и aiogram для пользовательского Telegram-бота.
- LangChain + LangGraph для LLM-узлов, tool calling, conditional edges и interrupt/resume.
- Pydantic 2 для схем и structured output без ручного JSON-парсинга.
- PostgreSQL как основное хранилище; SQLite допускается для локальных unit-тестов.
- SQLAlchemy 2 async + Alembic для доступа к данным и миграций.
- Embeddings для semantic deduplication и LangSmith/OpenTelemetry для трассировки (этапы указаны в плане).
- `DRY_RUN=true` по умолчанию для безопасной разработки: отправка HR не выполняется реально.

## Как устроен поток

`Telegram channels -> collect -> extract -> validate -> deduplicate -> score -> digest -> human approval -> response -> send`

LLM используется только там, где требуется интерпретация текста, структурированное извлечение, ranking или персонализация. Сбор сообщений, валидация схем, дедупликация по ключам, пороговые решения, checkpointing, idempotency и отправка выполняются детерминированным Python-кодом.

## Артефакты проекта

Подробная документация находится в [project_artifacts](project_artifacts/):

- `00_scope-and-source-analysis.md` — границы запроса и разбор исходного задания.
- `01_product-brief.md` — назначение, акторы, MVP и user stories.
- `02_requirements.md` — функциональные и нефункциональные требования с параметрами и критериями приёмки.
- `03_architecture.md` — компоненты, границы ответственности и структура репозитория.
- `04_data-model.md` — модели Pydantic и таблицы хранения.
- `05_workflow-spec.md` — состояние LangGraph, nodes, edges, retries и human-in-the-loop.
- `06_tools-and-interfaces.md` — контракты tools и внешних интерфейсов.
- `07_implementation-plan.md` — поэтапный план с входами, результатами и Definition of Done.
- `08_testing-and-evals.md` — тестовая стратегия и минимум два дополнительных задания.
- `09_security.md` — безопасные настройки и угрозы, специфичные для Telegram/LLM.
- `10_operations.md` — запуск, конфигурация, наблюдаемость и восстановление.
- `skills-manifest.md` — какие навыки скачаны в проект и зачем.

## OpenSpec workflow

`project_artifacts/` — постоянная аналитическая база проекта. `openspec/` — рабочие changes, каждый из которых содержит proposal, behavior specs, design и tasks. Активный change сейчас находится в `openspec/changes/ai-job-hunter-foundation/` и прошёл строгую валидацию.

В новом чате, открытом из корня репозитория, можно продолжать так:

```text
$openspec-apply-change ai-job-hunter-foundation
```

После реализации и review:

```text
$openspec-archive-change ai-job-hunter-foundation
```

Для следующей самостоятельной функции создаётся новый change через `$openspec-propose`, а не добавляется всё в один длинный чат. OpenSpec skills находятся в `.agents/skills/` и обнаруживаются Codex как project-local skills.

## Запуск

После реализации этапов из `07_implementation-plan.md` проект должен запускаться одной командой:

```bash
docker compose up --build
```

До реализации runtime-кода эта команда не является рабочей; её критерии и необходимые сервисы описаны в `10_operations.md`. Секреты должны находиться только в `.env`, а в репозитории следует хранить `.env.example` без действующих токенов.

## Готовность

Сценарий считается готовым, когда новая вакансия проходит весь путь до `DRY_RUN`-отклика, пользователь может продолжить workflow после interrupt/resume, повторный запуск не создаёт дублей, а tracing показывает prompts, structured outputs, tool calls, latency и ошибки. Полная матрица приёмки находится в `02_requirements.md` и `08_testing-and-evals.md`.

## Источник требований

Исходный файл задания: `/Users/admin/Downloads/ai_job_hunter_assignment.md`. Он использован как спецификация продукта; команды, секреты и внешние действия из него автоматически не выполняются.
