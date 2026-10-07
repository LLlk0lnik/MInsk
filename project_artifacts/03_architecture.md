# 03. Архитектура

## 3.1 Компоненты

```text
Telegram channels
      |
  Telethon reader ----> ingestion service ----> PostgreSQL
                                                  |
                             LangGraph worker ---+
                              | extract / score / response
                              | tools + checkpoints
                              v
                         digest bot (aiogram)
                              |
                   candidate approval callbacks
                              |
                     Telegram sender (dry-run)
```

## 3.2 Границы ответственности

- `services/telegram_reader.py` — только получение сообщений и cursor; не вызывает LLM.
- `agents/vacancy_agent.py` — prompt + structured output extractor.
- `agents/scoring_agent.py` — сравнение vacancy/profile, typed score.
- `workflows/job_hunter.py` — orchestration, branching, interrupt/resume.
- `tools/` — явные инструменты с валидацией входа и audit events.
- `db/` — модели, репозитории и транзакционные границы.
- `bot/` — UI Telegram, callback routing и подтверждения.
- `prompts/` — versioned templates; данные подставляются через безопасный контекст.

## 3.3 Детерминированность против AI

| Задача | Решение | Причина |
| --- | --- | --- |
| Cursor, idempotency, транзакции | Python/DB | Точная семантика и повторяемость |
| Определить смысл свободного текста | LLM structured output | Regex не покрывает варианты Telegram-постов |
| Проверить типы/диапазоны | Pydantic + Python | LLM не является валидатором |
| Объяснить соответствие профилю | LLM + deterministic hard exclusions | Нужны семантические причины, но safety-ограничения фиксированы |
| Порог `score >= threshold` | Python | Нельзя отдавать критическое ветвление модели |
| Отправить сообщение | Python tool + allowlist | Внешнее действие должно быть контролируемым |

## 3.4 DDD и abstract interfaces

- Доменный слой содержит сущности, value objects, инварианты и domain services; он не импортирует Telegram SDK, LLM SDK, SQLAlchemy или LangGraph.
- Application layer координирует use cases и зависит только от abstract interfaces/ports.
- Infrastructure layer реализует порты через PostgreSQL, Telegram, LLM и другие внешние системы.
- Concrete adapters подключаются на composition root; framework-specific типы не должны проникать в domain contracts.

## 3.5 Рекомендуемая структура репозитория

```text
src/ai_job_hunter/
  agents/       # extraction, scoring, response
  bot/          # aiogram handlers/keyboards
  db/           # SQLAlchemy models/repositories/migrations
  prompts/      # versioned markdown prompts
  schemas/      # Pydantic domain contracts
  services/     # Telegram reader/sender, embeddings, tracing
  tools/        # LangChain tool wrappers
  workflows/    # LangGraph state and graph construction
  config.py
  main.py
tests/
  unit/ integration/ evals/ fixtures/
infra/
  docker-compose.yml alembic/
project_artifacts/
```

## 3.6 Отказы и восстановление

- LLM timeout/5xx: retry 2 раза, затем `run_status=failed_retryable`.
- Schema validation: не повторять тот же output бесконечно; сохранить redacted raw response и отправить в review queue.
- Telegram flood wait: уважить `retry_after`, не обходить ограничение параллельными запросами.
- Database failure: транзакция откатывается, cursor не продвигается до commit.
- Restart: worker загружает checkpoint по `thread_id`, повторяет только незавершённую node.

## 3.7 Архитектурные решения

1. PostgreSQL используется и для бизнес-данных, и для durable checkpointing в production.
2. SQLite допускается для unit-тестов, но не считается production checkpoint store.
3. Сначала реализуется один process/worker; горизонтальное масштабирование возможно после введения queue/lease.
4. Embeddings являются оптимизацией deduplication, а не единственным источником истины: exact source key всегда проверяется первым.
