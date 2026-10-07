# 04. Модели данных и контракты

## 4.1 Pydantic-контракты

### `VacancyExtraction`

| Поле | Тип | Ограничение |
| --- | --- | --- |
| `is_vacancy` | `bool` | required |
| `is_python_backend` | `bool` | required |
| `title` | `str | None` | 1..200, required when vacancy |
| `company` | `str | None` | max 200 |
| `salary_min`, `salary_max` | `int | None` | >= 0, min <= max |
| `currency` | `str | None` | ISO-like short code |
| `technologies` | `list[str]` | normalized, unique, max 50 |
| `experience` | `str | None` | free normalized text |
| `location` | `str | None` | max 200 |
| `remote` | `bool | None` | tri-state: unknown is valid |
| `description` | `str` | 1..10000 |
| `hr_contact` | `str | None` | username/link/email redacted in logs |
| `source_url` | `AnyUrl` | required when vacancy |

`is_vacancy=false` допускает пустые vacancy fields, но source message всё равно сохраняется.

### `VacancyScore`

`score: int (0..100)`, `matching_skills: list[str]`, `missing_skills: list[str]`, `reasons: list[str] (1..8)`, `recommended: bool`, `profile_version_id: UUID`, `model_metadata`.

`recommended` пересчитывается кодом как `score >= SCORE_THRESHOLD` после hard exclusions; значение модели не является авторитетным.

### `ApplicationDraft`

`vacancy_id: UUID`, `candidate_profile_version_id: UUID`, `hr_contact: str`, `subject: str | None`, `body: str (20..4000)`, `personalization_facts: list[str]`, `model_metadata`, `requires_approval=true`.

## 4.2 Таблицы

| Таблица | Ключевые поля и индексы |
| --- | --- |
| `channels` | `id UUID`, `telegram_ref UNIQUE`, `enabled`, `last_message_id`, `last_polled_at` |
| `source_messages` | `id UUID`, `channel_id FK`, `telegram_message_id`, `published_at`, `source_url`, `raw_text`, `raw_text_hash`; `UNIQUE(channel_id, telegram_message_id)` |
| `vacancies` | `id UUID`, `canonical_hash UNIQUE`, normalized fields, `status`, `first_seen_at`, `last_seen_at` |
| `vacancy_sources` | `vacancy_id FK`, `source_message_id FK`, `UNIQUE(vacancy_id, source_message_id)` |
| `vacancy_scores` | `id UUID`, `vacancy_id`, `profile_version_id`, `score`, JSON arrays, `recommended`, model metadata |
| `candidate_profiles` | `id UUID`, `version int UNIQUE`, `content`, `stack JSONB`, `excluded JSONB`, `active` |
| `workflow_runs` | `id UUID`, `thread_id UNIQUE`, `vacancy_id`, `status`, `current_node`, `interrupt_reason`, timestamps |
| `applications` | `id UUID`, `vacancy_id`, `run_id`, `draft`, `approval_status`, `send_status`, `idempotency_key UNIQUE`, timestamps |
| `audit_events` | `id UUID`, `run_id`, `node`, `event_type`, redacted payload, latency/tokens/cost |

## 4.3 Статусы

- Vacancy: `new -> extracted -> scored -> digest_sent -> selected|skipped -> archived`.
- Workflow: `running -> interrupted -> resumed -> completed|failed_retryable|failed_terminal`.
- Application: `drafted -> awaiting_approval -> approved|rejected -> dry_run_sent|sent|send_failed`.

## 4.4 Инварианты

- Нет `sent` без `approved`.
- Нет cursor advance без committed source message.
- Один `idempotency_key` даёт максимум одну попытку внешней отправки.
- У score всегда сохраняется версия профиля и model metadata.
- Raw text не перезаписывается при появлении дубля; источники связываются отдельно.
