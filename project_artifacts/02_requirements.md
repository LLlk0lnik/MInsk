# 02. Подробные требования

## 2.1 Functional requirements

| ID | Требование | Параметры и ограничения | Критерий приёмки |
| --- | --- | --- | --- |
| FR-01 | Сбор новых сообщений | `channels: list[str]`, `poll_interval_seconds >= 30`, cursor per channel | После двух polling одного сообщения есть ровно одна запись `source_message`. |
| FR-02 | Сохранение источника | `channel_id`, `message_id`, `published_at`, `source_url`, `raw_text` | По записи можно открыть исходный Telegram message. |
| FR-03 | Извлечение вакансии | LLM structured output, `is_vacancy`, `is_python_backend`, nullable fields | Невакансионное сообщение не проходит в scoring; невалидный output не сохраняется как vacancy. |
| FR-04 | Нормализация | canonical title/company/skills, salary range, remote/location | Одинаковые значения в разных форматах сравниваются предсказуемо. |
| FR-05 | Профиль кандидата | versioned profile: experience, stack, interested, excluded | Scoring использует опубликованную версию профиля и сохраняет её id. |
| FR-06 | Score | integer `0..100`, skills, reasons, recommended | `recommended = score >= threshold` плюс deterministic hard exclusions. |
| FR-07 | Дедупликация | exact source key, normalized hash, optional embedding cosine threshold | Повтор вакансии связывается с `canonical_vacancy_id`, но источники не теряются. |
| FR-08 | Workflow | nodes, typed state, conditional edges, tools, retries, checkpoint | Каждая node имеет отдельный trace span и тест на happy/error path. |
| FR-09 | Digest | batch size `1..20`, callback payload signed/opaque | Пользователь видит score, reasons и кнопки без переполнения Telegram message limit. |
| FR-10 | HITL | interrupt before application generation/send; explicit approve/reject | После interrupt состояние хранится; resume не повторяет завершённые nodes. |
| FR-11 | Response generation | vacancy + candidate profile + company + HR name | Черновик объяснимо привязан к данным вакансии и проходит moderation/length check. |
| FR-12 | Sending | `DRY_RUN`, allowlist, idempotency key, rate limit | В dry-run нет Telegram API send; production send повторно не выполняется. |
| FR-13 | Tools | profile, messages, search, vacancy, save, generate, send | Tool schemas валидируются до вызова, ошибки являются typed results. |
| FR-14 | Persistence | channels, messages, vacancies, scores, applications, runs, checkpoints | Рестарт контейнера не теряет cursor/run state. |
| FR-15 | Observability | trace id, node, prompt hash, latency, tokens, cost, error class | По run id восстанавливается полный путь без секретов. |

## 2.2 Non-functional requirements

- **Reliability:** повторяемые операции idempotent; LLM timeout 30 s, до 2 retries с exponential backoff; Telegram API ошибки классифицируются.
- **Safety:** default `DRY_RUN=true`; отправка требует source HR contact и двух user actions; секреты только через env/secret store.
- **Performance:** один канал polling p95 < 5 s без LLM; digest формируется не более чем за 60 s для batch 20 при доступном провайдере.
- **Data integrity:** все внешние идентификаторы и callback tokens opaque; записи имеют `created_at`, `updated_at` и audit status.
- **Maintainability:** node не смешивает I/O, prompt и бизнес-решения; prompts versioned и тестируются отдельно.
- **Privacy:** retention raw Telegram text и prompt payload конфигурируется; логи по умолчанию редактируют username, токены и контакты.

## 2.3 Конфигурация

| Переменная | Тип | Default | Назначение |
| --- | --- | --- | --- |
| `APP_ENV` | enum | `development` | Окружение |
| `DATABASE_URL` | URL | required | PostgreSQL/SQLite DSN |
| `TELEGRAM_API_ID` | int | required | Telethon |
| `TELEGRAM_API_HASH` | secret | required | Telethon |
| `TELEGRAM_BOT_TOKEN` | secret | required | aiogram |
| `TELEGRAM_SESSION_STRING` | secret | required | Session Telethon |
| `LLM_PROVIDER` | enum | `openai` | Provider adapter |
| `LLM_MODEL` | string | configured | Модель extraction/score/response |
| `LLM_API_KEY` | secret | required | LLM provider |
| `SCORE_THRESHOLD` | int | `70` | Порог recommendation |
| `POLL_INTERVAL_SECONDS` | int | `300` | Интервал polling |
| `DRY_RUN` | bool | `true` | Блокирует реальную отправку |
| `LANGSMITH_TRACING` | bool | `false` | Optional tracing |

## 2.4 Acceptance checklist

- [ ] Все FR-01..FR-15 покрыты тестами или ручным demo script.
- [ ] `docker compose up --build` поднимает app, worker и database.
- [ ] Повторный polling и повторный callback безопасны.
- [ ] Невакансионные и hard-excluded сообщения не доходят до HR.
- [ ] При падении LLM/Telegram run получает понятный статус и может быть retry/resume.
