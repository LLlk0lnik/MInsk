# 05. Спецификация LangGraph workflow

## 5.1 Typed state

```text
AgentState:
  run_id: UUID
  thread_id: str
  source_message_ids: list[UUID]
  vacancy: Vacancy | None
  score: VacancyScore | None
  decision: Literal['skip','digest','apply'] | None
  generated_response: ApplicationDraft | None
  approval: Literal['pending','approved','rejected'] | None
  errors: list[WorkflowError]
  trace_context: TraceContext
```

State не должен содержать Telegram token, полный prompt или необработанный секрет; большие payloads и checkpoints хранятся redacted/в БД.

## 5.2 Nodes и вход/выход

| Node | Вход | Выход | Тип |
| --- | --- | --- | --- |
| `collect` | channel cursor | source messages | deterministic I/O |
| `extract` | raw text | `VacancyExtraction` | LLM structured output |
| `validate` | extraction | normalized vacancy или reject reason | Pydantic + Python |
| `deduplicate` | normalized vacancy | canonical vacancy/link | Python + optional embeddings |
| `score` | vacancy + profile | `VacancyScore` | LLM structured output |
| `route` | score + hard exclusions | decision | deterministic |
| `save_vacancy` | vacancy/score | persisted ids | DB transaction |
| `generate_digest` | scored vacancies | Telegram digest payload | Python template |
| `human_approval` | run id | interrupt/resume command | LangGraph interrupt |
| `generate_response` | vacancy + profile + approval | `ApplicationDraft` | LLM structured/text output |
| `send_response` | approved draft | send result | tool, dry-run aware |

## 5.3 Conditional edges

```text
validate -> [non_vacancy: END, python_backend: deduplicate, invalid: retry_or_review]
route -> [score < threshold: archive, score >= threshold: save_vacancy]
human_approval -> [reject: END, approve: generate_response, timeout: interrupted]
send_response -> [dry_run: completed, sent: completed, retryable_error: retry, terminal_error: failed]
```

## 5.4 Interrupt/resume контракт

1. Перед interrupt записать `workflow_runs.status=interrupted`, `approval=pending`, checkpoint и digest message id.
2. Callback содержит opaque `run_id`/nonce, а не доверенные поля вакансии.
3. Handler проверяет владельца, статус и срок действия callback.
4. `Command(resume={'approval': 'approved'})` продолжает тот же `thread_id`.
5. Повторный callback после завершения возвращает idempotent response и не вызывает send.

## 5.5 Retry policy

- LLM network/429/5xx: максимум 2 retries, backoff 1 s, 4 s, jitter.
- Pydantic validation: один repair pass через structured output; затем terminal review.
- Telegram FloodWait: ждать `retry_after` в worker, максимум из env `MAX_FLOOD_WAIT_SECONDS`.
- Неповторяемые ошибки авторизации/конфигурации: без retries, alert operator.

## 5.6 Checkpointing

Production: PostgreSQL checkpointer, `thread_id` стабилен для одного workflow run, checkpoint commit после каждой node. Local unit tests: in-memory saver. Никогда не полагаться на process memory для human-in-the-loop.
