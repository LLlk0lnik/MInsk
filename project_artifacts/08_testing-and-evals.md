# 08. Тестирование и evals

## 8.1 Пирамида тестов

- **Unit:** Pydantic validators, normalization, hash/dedup, hard exclusions, callback parsing, idempotency.
- **Integration:** PostgreSQL repositories, Alembic, LangGraph checkpoint/resume, mocked Telethon/aiogram/LLM.
- **Contract:** structured output schema, tool input/output, provider adapter, Telegram message limits.
- **End-to-end:** fixture message -> vacancy -> score -> digest -> approval -> dry-run application.
- **Security:** secret redaction, unauthorized callback, prompt injection fixture, send guard, flood-wait behavior.

## 8.2 Eval dataset

Собрать минимум 50 обезличенных Telegram messages: Python vacancies, другие backend, frontend/1C/PHP, шум, дубли, неполные salary/contact/location. Для каждого хранить gold labels и schema fields, без токенов и личных данных.

## 8.3 Метрики и стартовые пороги

| Метрика | Формула | Целевой порог MVP |
| --- | --- | --- |
| Vacancy detection F1 | binary F1 | >= 0.90 |
| Python/backend classification F1 | binary F1 | >= 0.85 |
| Required-field validity | valid structured records / records | >= 0.95 |
| Salary extraction exact/range accuracy | normalized match | >= 0.80 |
| Score rank agreement | Spearman vs reviewer | >= 0.70 |
| Duplicate precision | correct duplicate links / links | >= 0.90 |
| Approval safety | sends without approval | 0 |
| Resume correctness | runs resumed without duplicate side effect | 100% fixtures |

Пороги являются стартовыми и должны быть пересмотрены после размеченной выборки; качество LLM нельзя оценивать только на одном demo message.

## 8.4 Выбранные дополнительные задания

1. **Semantic Search:** embeddings для похожих вакансий/дубликатов после exact hash; порог cosine хранится в конфигурации и тестируется на eval set.
2. **Evals:** dataset, gold labels, offline runner и отчёт с указанными метриками.

RAG, memory, MCP и agentic search оставлены как post-MVP backlog, чтобы не размывать обязательный workflow.

## 8.5 Demo checklist

- Новое сообщение обнаружено.
- Структурированный объект виден в trace.
- Score и причины отображаются в digest.
- Interrupt записан в БД.
- После restart callback resume работает.
- Draft персонализирован профилем.
- `DRY_RUN` показывает payload, но не отправляет его.
- Повтор любого шага не создаёт дубль.
