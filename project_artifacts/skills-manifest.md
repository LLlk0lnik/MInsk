# Skills manifest

Навыки скачаны в проектный `.codex/skills/` и дополнительно установлены в глобальный `/Users/admin/.codex/skills/`, чтобы быть доступны в следующих задачах Codex. Источник — публичный репозиторий `openai/skills`, ветка `main`; установка выполнена скриптом системного `skill-installer`.

| Навык | Путь | Зачем проекту |
| --- | --- | --- |
| `openai-docs` | `.codex/skills/openai-docs/` | Актуальные официальные справочные материалы по OpenAI API, моделям и structured/tool calling при реализации provider adapter. |
| `security-best-practices` | `.codex/skills/security-best-practices/` | Secure-by-default проверки Python backend/secret handling на этапе написания кода. |
| `security-threat-model` | `.codex/skills/security-threat-model/` | Основание для отдельного threat review после появления фактических runtime-компонентов и deployment topology. |

Отдельно установлен официальный OpenSpec CLI `@fission-ai/openspec@1.14.1` глобально через npm и инициализирован в проекте. Его project-local Codex workflows находятся в `.agents/skills/`, а рабочие артефакты — в `openspec/`.

## Проверка установки

На момент подготовки артефактов в обеих целевых директориях присутствуют `SKILL.md`, `LICENSE.txt`, `agents/openai.yaml` и reference-файлы каждого навыка. Системные `.system`-навыки не дублировались в проект, потому что они уже предустановлены в Codex.

## Граница применения

Скачивание навыка не означает автоматического запуска внешних действий. При реализации нужно читать соответствующий `SKILL.md` перед применением навыка; секреты и сетевые вызовы проекта по-прежнему требуют явной конфигурации.
