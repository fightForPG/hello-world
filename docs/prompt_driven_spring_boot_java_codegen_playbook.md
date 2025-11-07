# Prompt-Driven Spring Boot & Java Codegen Playbook

## 1. Purpose
- Establish a repeatable workflow for driving Spring Boot and broader Java development with large language models (LLMs).
- Capture best practices that keep generated code maintainable, secure, and testable.
- Provide prompt recipes and checklists that work with tools such as Cursor, GitHub Copilot, GPT-4/5, Claude, etc.

## 2. Core Principles
- **Context-rich prompts**: Always describe the architecture slice, existing modules, and constraints before asking for code.
- **Small, iterative deltas**: Generate one component or change at a time, run tests/formatters, then iterate.
- **Guardrail first**: Define quality bars (style, null-safety, logging, tests) in the prompt so the model plans for them.
- **Review mindset**: Treat LLM output as a draft; perform manual review and request refinements explicitly.
- **Traceable history**: Keep prompts and responses in commits or PR descriptions for reproducibility.

## 3. Preparation & Environment
- **Target stack**: Spring Boot 3.x, Java 17+, Gradle or Maven (specify which), JUnit 5, Testcontainers (if applicable).
- **Local tooling**:
  - Formatter (`spotless`, `google-java-format`, or IDE format).
  - Static analysis (`spotbugs`, `checkstyle`, `errorprone` if available).
  - Test command (e.g., `./mvnw test` or `./gradlew test`).
- **Project context package**:
  - Project goals, domain glossary, data models.
  - Module layout (packages, key classes).
  - Coding standards (null handling, logging, exception strategy).
  - Configuration specifics (DB, messaging, security).
- **Session setup prompt** (share once per feature):
  1. High-level architecture.
  2. Current state of the module being changed.
  3. The task statement and acceptance criteria.
  4. Testing/validation expectations.

## 4. Prompt Templates

### 4.1 Session Kickoff
```
You are assisting on a Spring Boot 3 service named {service-name}. 
Stack: Java 21, Gradle, Postgres via Spring Data JPA, Kafka, Testcontainers.
Architecture context: {short description}.
Coding standards: 
- Prefer records for DTOs, Lombok is disallowed.
- Use constructor injection; no field injection.
- Log with SLF4J at info for lifecycle events, debug for detail.
- Validations use jakarta.validation.
Quality gates:
- All new code must have unit tests with Mockito.
- Provide integration test if repository/database logic is added.
```

### 4.2 Feature Implementation
```
Task: Implement {feature}.
Scope:
- Controller endpoint signature {signature or contract}.
- Service responsibilities {details}.
- Persistence layer {entity/repository expectations}.
Constraints:
- Preserve existing API behavior {link/reference}.
- Use {DTO/MapStruct/etc}.
Deliverables:
1. Updated/created classes with annotations.
2. Tests (unit + integration where data layer touched).
3. Brief verification steps (curl, test command).
Return response as: 
- Summary of changes.
- File-by-file diff with explanations.
- Test plan.
```

### 4.3 Refactoring / Bugfix
```
Context:
- Current bug or smell and reproduction steps.
- Existing class excerpts or design.
Goal:
- Describe the intended fix or refactor outcome.
Constraints:
- No breaking API changes.
- Keep transactional boundaries intact.
Checklist:
- Update tests or add regression coverage.
- Call out any risk or follow-up.
```

## 5. Workflow Blueprint
1. **Clarify Task**: Write acceptance criteria; confirm with product/spec if ambiguous.
2. **Prime the Model**: Paste module overview, class snippets, config, and tests relevant to the change.
3. **Request Plan First**: Ask for a step-by-step plan before code to catch misunderstandings.
4. **Generate Incrementally**:
   - Controller or API contract.
   - Service logic.
   - Repositories/entities.
   - Configuration/properties.
   - Tests.
   Use separate prompts per component if complexity is high.
5. **Review & Adapt**: Check for framework-specific nuances (transactional boundaries, lazy loading, bean scopes).
6. **Run Tooling**: Formatters, static analysis, and tests locally. Feed failures back to the LLM with logs.
7. **Document**: Request the LLM to draft changelog/PR description while context is fresh.

## 6. Common Spring Boot Scenarios & Prompt Snippets

- **Creating a new REST endpoint**
  - Describe request/response schema, validation rules, error handling strategy.
  - Provide sample JSON payloads and expected HTTP status codes.
  - Ask for controller, service, DTOs, and tests.

- **Adding persistence with Spring Data JPA**
  - Supply entity relationships, cascade rules, indexes.
  - Specify whether to use projections or DTO mapping.
  - Request repository interface, entity class, schema migration script, and testcontainers-based tests.

- **Securing endpoints**
  - Describe authentication provider (JWT, OAuth2, Basic).
  - Provide existing `SecurityFilterChain` or config.
  - Ask for updates to security config, method-level annotations, tests using MockMvc/WebTestClient.

- **Asynchronous processing**
  - Clarify whether to use `@Async`, Spring Integration, or messaging (Kafka/RabbitMQ).
  - Provide message schema and retry/backoff requirements.
  - Request listener, producer, config, and contract tests.

- **Configuration & profiles**
  - State environment-specific overrides.
  - Ask for updates to `application.yml` and configuration properties classes with validation.

## 7. Quality Checklist Prompts
- **Code Review Draft**
  ```
  Review the diff for {feature}. Identify missing null checks, transactional issues, exception handling gaps, logging concerns, and alignment with domain rules. Summarize high-risk findings first, then medium, then low.
  ```
- **Test Coverage Assessment**
  ```
  Confirm that unit tests cover positive, negative, and edge cases for {class}. Suggest missing scenarios or data permutations.
  ```
- **Security Scan**
  ```
  Inspect the new code for security pitfalls: input validation, authentication/authorization, secret management, SQL injection. Propose fixes if any risks are found.
  ```

## 8. Integration with Tooling
- **Cursor / ChatGPT**: Use `@codebase` or file uploads to share relevant snippets; pin instructions for style rules.
- **Git Hooks**: Automate `./gradlew spotlessApply test` (or Maven equivalents) before commit.
- **CI Feedback Loop**: Paste failing build logs into the prompt with “analyze this failure and patch the code/test”.
- **Knowledge Base**: Store canonical prompts and project context in a shared doc or `docs/prompt-cheatsheet.md`.

## 9. Handling Generated Code Safely
- Always verify dependency versions and starter imports; LLMs may hallucinate artifact coordinates.
- Watch for blocking calls inside reactive code (`Mono`/`Flux`).
- Ensure transactional boundaries wrap only necessary methods.
- Validate serialization/deserialization for records/DTOs (Jackson annotations).
- Confirm configuration property names match `application.yml`.
- Scrutinize date/time handling (time zones, `Instant` vs `LocalDateTime`).

## 10. Example End-to-End Session Outline
1. **Kickoff prompt** with architecture + conventions.
2. **Plan request**: “Draft the plan to implement feature X given the context.”
3. **Iterative code generation**: Prompt per layer, provide diffs/logs back to model for corrections.
4. **Testing phase**: After running tests, share failures stack traces for fixes.
5. **Documentation prompt**: Ask the LLM to create API docs snippets or ADR updates.
6. **PR preparation**: Generate final summary, risk assessment, and manual test checklist.

## 11. Troubleshooting Patterns
- If the model produces incompatible annotations/imports, remind it of the Spring Boot version and dependencies.
- When output drifts, restate the key constraints and ask for a focused rewrite.
- Use comparison prompts: “Compare your suggestion with the existing strategy in `FooService` and align with it.”
- For large changes, request code in logical chunks and paste into the IDE manually to control context windows.

## 12. Reference Prompt Library

### CRUD Feature Template
```
Context: We have an entity {EntityName} with fields {fields}. Provide controller, service, repository, mapper, DTOs, validation annotations, and unit + integration tests using Testcontainers. Persist via Spring Data JPA.
Constraints: Use constructor injection, prefer records for DTOs, handle optimistic locking with @Version.
Output: Patch summary, code blocks grouped per file, and a test execution plan.
```

### Kafka Consumer Template
```
Goal: Create a Kafka consumer for topic {topic-name}. Messages follow schema {schema}. 
Requirements:
- Use Spring Kafka with manual ack.
- Validate payload with jakarta.validation.
- Send metrics to Micrometer counter `message.consumed`.
Deliverables: Consumer config, listener, error handler, tests with EmbeddedKafka.
```

### Scheduled Job Template
```
Add a scheduled job that runs cron {cron}. 
Responsibility: {job description}.
Ensure idempotency and log execution duration.
Return: Job component, configuration, and unit test validating scheduling and service call.
```

## 13. Evolving the Playbook
- Review and adapt prompts after each project milestone; capture what worked vs. failed.
- Maintain a changelog with date, improvement, and rationale.
- Encourage team contributions via PRs to keep the playbook current.

---
**Usage tip**: Keep a `prompts/` directory in the repo with JSON or Markdown prompt snippets that engineers can copy into their LLM sessions. Tie them to tasks in the issue tracker for transparency.
