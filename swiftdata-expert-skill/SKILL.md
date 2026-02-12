---
name: swiftdata-expert-skill
description: SwiftData guidance for iOS 26+ focused on modeling, query design, context isolation, history processing, migration, custom data store decisions, performance, and testability.
---

# SwiftData Expert Skill

## When To Use
Use this skill when the request includes one or more of the following:
- SwiftData model design, relationship rules, or schema invariants
- `@Query`, `FetchDescriptor`, `#Predicate`, or fetch performance issues
- actor isolation, background writes, or context ownership bugs
- history token processing, cross-process sync, or replay safety
- `VersionedSchema` and `SchemaMigrationPlan` planning/review
- custom `DataStore` adoption decisions
- preview/test container setup and deterministic fixtures

## Default Operating Mode
- Load the minimum reference set needed for the request (start with one primary file).
- Keep answers anchored to data correctness, migration safety, and bounded workload behavior.
- Use default SwiftData store unless the user provides explicit non-default requirements.

## Situation Routing Matrix
| Situation | Primary Reference | Load Next If Needed |
|---|---|---|
| Container wiring, topology, app boundary | `references/fundamentals-and-container.md` | `references/testing-and-previews.md` |
| Model invariants, uniqueness, delete rules | `references/modeling-and-relationships.md` | `references/review-checklist.md` |
| UI query design, predicate/sort issues | `references/query-and-predicate-patterns.md` | `references/performance-patterns.md` |
| Actor isolation, context ownership, background writes | `references/concurrency-and-context-isolation.md` | `references/review-checklist.md` |
| Incremental history processing, token durability | `references/history-and-change-tracking.md` | `references/concurrency-and-context-isolation.md` |
| Schema evolution, upgrade planning | `references/migration-and-versioned-schema.md` | `references/testing-and-previews.md` |
| Large dataset performance and write amplification | `references/performance-patterns.md` | `references/query-and-predicate-patterns.md` |
| Non-default persistence backend discussion | `references/custom-datastore.md` | `references/review-checklist.md` |
| Inheritance and polymorphic query design | `references/inheritance-and-polymorphism.md` | `references/modeling-and-relationships.md` |
| Test/previews fixture determinism, in-memory setup | `references/testing-and-previews.md` | `references/migration-and-versioned-schema.md` |
| PR review of SwiftData changes | `references/review-checklist.md` | Load files for touched concern areas |

## Response Contract
Choose one template per request type.

### Design or Implementation
1. Constraints and invariants
2. Proposed model/query/concurrency design
3. Save/migration/history implications
4. Test plan (deterministic fixtures)

### Debug or Incident Triage
1. Likely failure modes (ordered)
2. Minimal repro and instrumentation points
3. Fix strategy and rollback safety
4. Verification steps

### Code Review
1. Findings first (severity-ordered)
2. Data integrity and migration risks
3. Concurrency/performance risks
4. Missing test coverage

### Migration Planning
1. Version map (`from -> to`)
2. Stage type (lightweight vs custom) with rationale
3. Data transformation and idempotency plan
4. Multi-hop upgrade test matrix

## Guardrails
- Do not move filters/sorts into view render paths when dataset size can grow.
- Do not pass mutable model instances across actor boundaries.
- Do not persist history checkpoint tokens before downstream effects are durable.
- Do not ship schema edits without explicit version and migration path.
- Treat `try!` and force unwrap as preview/test-only shortcuts; use explicit error handling in app/runtime code.

## Reference Index
- `references/fundamentals-and-container.md`
- `references/modeling-and-relationships.md`
- `references/query-and-predicate-patterns.md`
- `references/concurrency-and-context-isolation.md`
- `references/history-and-change-tracking.md`
- `references/migration-and-versioned-schema.md`
- `references/performance-patterns.md`
- `references/custom-datastore.md`
- `references/inheritance-and-polymorphism.md`
- `references/testing-and-previews.md`
- `references/review-checklist.md`
