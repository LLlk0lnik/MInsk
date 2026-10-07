# Project artifacts

Это рабочий пакет спецификаций для `AI Job Hunter Agent`. Документы разделяют требования исходного задания, проектные решения и план реализации.

Это аналитическая база проекта. Для пошаговой работы отдельных изменений используется соседний каталог `/Users/admin/Documents/GitHub/MInsk/openspec/`: там OpenSpec хранит proposal, delta specs, design и tasks, которые можно продолжать в новых чатах.

## Рекомендуемый порядок чтения

1. `00_scope-and-source-analysis.md` — что действительно требуется и какие решения добавлены.
2. `01_product-brief.md` — пользовательский сценарий и границы MVP.
3. `02_requirements.md` — измеримые требования, параметры и acceptance checklist.
4. `03_architecture.md` — компоненты и зоны ответственности.
5. `04_data-model.md` — схемы и persistence.
6. `05_workflow-spec.md` — LangGraph state, nodes, edges, retries, checkpointing.
7. `06_tools-and-interfaces.md` — tool contracts и callback API.
8. `07_implementation-plan.md` — этапы разработки и Definition of Done.
9. `08_testing-and-evals.md` — тесты, метрики и выбранные дополнительные задания.
10. `09_security.md` — safety gates и угрозы.
11. `10_operations.md` — запуск и восстановление.
12. `skills-manifest.md` — установленные навыки и их назначение.

Документация не утверждает, что runtime-код уже написан: до начала этапа 0 это спецификационный baseline.
