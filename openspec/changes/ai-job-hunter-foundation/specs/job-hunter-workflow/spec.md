# Spec Delta

## Purpose

Capability for reliably turning new Telegram vacancy messages into ranked, human-approved application drafts while preserving state, auditability and protection against duplicate or accidental sends.

## ADDED Requirements

### Requirement: New source messages are collected idempotently
Система SHALL получать только сообщения после сохранённого cursor каждого канала и сохранять исходный текст, дату, канал, message id и source URL.

#### Scenario: Polling a channel twice
- **WHEN** worker polls the same channel twice without a new message
- **THEN** the second poll creates no duplicate source message and does not advance the cursor incorrectly

#### Scenario: Persisting a new message
- **WHEN** a new Telegram message is received
- **THEN** the system stores it atomically with its source metadata before advancing the channel cursor

### Requirement: Vacancy extraction returns validated structured data
Система SHALL классифицировать сообщение как вакансию/Python-backend vacancy и возвращать поля вакансии в валидируемой structured schema.

#### Scenario: Non-vacancy message
- **WHEN** a message is not a vacancy or is outside Python/backend scope
- **THEN** the system stores the source message but does not create a ranked vacancy

#### Scenario: Incomplete vacancy
- **WHEN** a vacancy omits salary, company or HR contact
- **THEN** the system keeps those fields unknown/null and still preserves the vacancy description and source URL

### Requirement: Vacancies are ranked against a versioned candidate profile
Система SHALL сравнивать нормализованную вакансию с конкретной версией профиля и сохранять score 0..100, matching skills, missing skills, reasons и recommendation.

#### Scenario: Matching vacancy
- **WHEN** vacancy skills and interests match the active profile
- **THEN** the score contains explainable matching reasons and recommendation is true only when the configured threshold is met

#### Scenario: Hard exclusion
- **WHEN** vacancy matches a profile exclusion such as PHP, 1C or manual QA
- **THEN** the system marks it not recommended regardless of the model's raw recommendation

### Requirement: Duplicate vacancies resolve to one canonical vacancy
Система SHALL detect repeated vacancies across source messages and link every source to one canonical vacancy without deleting source evidence.

#### Scenario: Exact duplicate
- **WHEN** normalized source text and identifying fields match an existing vacancy
- **THEN** the system reuses the canonical vacancy and adds the new source link idempotently

#### Scenario: Semantic near duplicate
- **WHEN** embeddings indicate that two differently worded vacancies exceed the configured similarity threshold
- **THEN** the system flags them for canonical linking or review and records the similarity evidence

### Requirement: Digest requires explicit human approval
Система SHALL show qualifying vacancies in a Telegram digest with open/apply/skip actions and SHALL pause application workflow until the candidate chooses an action.

#### Scenario: Digest delivery
- **WHEN** scored vacancies meet the digest threshold
- **THEN** the candidate receives title, company, score, reasons and opaque callback actions

#### Scenario: Resume after approval
- **WHEN** the candidate approves an application callback
- **THEN** the same workflow run resumes from its checkpoint and does not repeat completed side effects

### Requirement: Application drafts are personalized and guarded
Система SHALL generate a draft using vacancy data and the selected candidate profile, and SHALL block sending until approval, valid HR contact and idempotency checks pass.

#### Scenario: Personalized draft
- **WHEN** an application is approved for generation
- **THEN** the draft references relevant vacancy requirements and candidate experience instead of using one universal template

#### Scenario: Dry run
- **WHEN** `DRY_RUN=true`
- **THEN** the system stores and displays the exact send preview but does not call the Telegram send operation

### Requirement: Workflow state survives restart and failures are observable
Система SHALL persist workflow state, checkpoints, statuses and redacted audit events so a restart or bounded retry cannot lose approval state or create duplicate sends.

#### Scenario: Worker restart during approval
- **WHEN** the worker restarts while a run is interrupted
- **THEN** the candidate can resume the same run using its thread id and the previous state remains available

#### Scenario: Retryable provider error
- **WHEN** an LLM or Telegram provider returns a bounded retryable error
- **THEN** the system records the error class, retries according to policy and exposes the run id and current node for diagnosis
