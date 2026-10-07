# 10. Эксплуатация

## Планируемые сервисы Docker Compose

- `app` — bot handlers и health endpoint.
- `worker` — polling и LangGraph runs.
- `postgres` — бизнес-данные и checkpoints.
- `otel-collector`/LangSmith — optional observability profile.

## Команды

```bash
docker compose up --build
docker compose exec app alembic upgrade head
docker compose exec app pytest
docker compose logs -f worker
```

Фактические имена сервисов должны совпасть с compose-файлом, который появится на этапе 0.

## Health/readiness

- `/healthz` — process жив.
- `/readyz` — DB migration/checkpoint store доступен.
- Worker metric `last_successful_poll_at` не старше двух poll intervals.
- Alert при `failed_retryable`, росте FloodWait или `DRY_RUN=false` без allowlist.

## Восстановление

1. Остановить только worker при подозрении на повторные отправки; bot и DB не удалять.
2. Проверить `workflow_runs` и `applications.send_status` по `run_id`.
3. Возобновлять только `interrupted`/`failed_retryable` после проверки idempotency.
4. При повреждении DB восстановить backup и повторить миграции; cursor продвигать только после проверки источников.

## Поставка

- CI: lint, type check, unit/integration/evals smoke, dependency scan.
- `.env.example` документирует все параметры, но не содержит реальные значения.
- Deployment должен иметь pinned image/dependency versions и rollback migration plan.
