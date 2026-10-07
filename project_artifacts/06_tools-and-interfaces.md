# 06. Tools и внешние интерфейсы

## 6.1 Правила tools

- Каждый tool имеет typed input/output и явный `ToolError`.
- LLM может выбирать read/search tools, но не получает право обходить approval.
- Mutating tools (`save_vacancy`, `send_telegram_message`) проверяют authorization, idempotency и audit context.
- В tool result не возвращаются токены, session strings и необрезанный секрет.

## 6.2 Контракты

### `get_candidate_profile`

**Input:** `{profile_version: int | null}`. **Output:** активный `CandidateProfile` с `experience_years`, `stack`, `interested_in`, `not_interested_in`, `language_preferences`.

### `get_new_telegram_messages`

**Input:** `{channel_ref: str, after_message_id: int, limit: int 1..100}`. **Output:** `list[SourceMessage]`. Cursor обновляется только репозиторием после commit.

### `search_vacancies`

**Input:** `{query: str, min_score: int | null, limit: int 1..50}`. **Output:** redacted summaries with ids and source URLs.

### `get_vacancy`

**Input:** `{vacancy_id: UUID}`. **Output:** canonical vacancy, score and source links.

### `save_vacancy`

**Input:** canonical vacancy + source links + score. **Output:** `{vacancy_id, created: bool}`. Idempotent by `canonical_hash`.

### `generate_application`

**Input:** `{vacancy_id, profile_version_id, run_id}`. **Output:** `ApplicationDraft`. Tool не меняет approval status.

### `send_telegram_message`

**Input:** `{recipient: str, text: str, idempotency_key: str, dry_run: bool}`. **Output:** `{status: dry_run|sent|failed, provider_message_id: str | null}`. При dry-run сохраняет preview и гарантированно не вызывает Telegram send endpoint.

## 6.3 Telegram callback API

- `vacancy:open:<opaque_id>` — показать детали.
- `vacancy:apply:<opaque_id>` — создать/возобновить application approval.
- `vacancy:skip:<opaque_id>` — пометить skip.
- `application:approve:<opaque_id>` — разрешить генерацию/отправку.
- `application:reject:<opaque_id>` — завершить без отправки.

Старые/чужие callbacks отклоняются, payload не доверяется без lookup в БД.

## 6.4 Prompt contracts

Каждый prompt принимает именованные параметры: `raw_message`, `normalized_vacancy`, `candidate_profile`, `score_threshold`, `output_schema_version`. Prompt version и hash пишутся в audit event. JSON schema предоставляется SDK structured output, ручное извлечение JSON из строки запрещено.
