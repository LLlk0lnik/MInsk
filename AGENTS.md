# AI Job Hunter Agent — project guidance

## Persistent context

- If present, read local-only `PROJECT_CONTEXT.md` first: it is the repository navigation map and startup checklist for new chats.
- If it is absent (for example, in a fresh clone), use this file and `project_artifacts/README.md` as the durable baseline.
- Start with `project_artifacts/README.md` for the project baseline and `openspec/status` for active changes.
- Treat `project_artifacts/` as the durable analytical reference and `openspec/` as the source of truth for the current implementation change.
- Do not rely on chat history for requirements that can be recorded in a Markdown artifact.

## OpenSpec workflow

- For a new feature or behavior change, use the project-local OpenSpec workflow and create a separate change instead of expanding an unrelated active change.
- Before implementation, inspect `openspec status --change <name>` and read all planning artifacts reported by the CLI.
- Planning and implementation are separate: use `$openspec-apply-change <name>` only after the proposal/spec/design/tasks are ready.
- Mark task checkboxes only when the specified behavior and its verification are complete.
- Archive a change only after implementation, review and verification are complete.

## Repository conventions

- Keep deterministic I/O, validation, persistence and side effects separate from LLM interpretation.
- Preserve `DRY_RUN=true` as the default for Telegram sending.
- Never place Telegram, LLM or database secrets in source, specs, logs or artifacts.
- When behavior or architecture changes, update the relevant OpenSpec artifact and the durable project documentation.

## Current entry points

- Baseline: `project_artifacts/README.md`
- Active change: `openspec/changes/ai-job-hunter-foundation/`
- Local OpenSpec skills: `.agents/skills/`
