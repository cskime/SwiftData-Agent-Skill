---
name: swiftdata-expert-skill
description: Design, review, and improve SwiftData code for iOS 26+ using correct modeling, querying, context isolation, history tracking, schema migration, custom data stores, inheritance, performance, and testing practices. Use when implementing persistence features, debugging data behavior, planning migrations, or reviewing SwiftData architecture.
---

# SwiftData Expert Skill

## Overview
Use this skill to build, review, and improve SwiftData code for iOS 26+ applications. Prioritize data correctness, predictable save boundaries, query efficiency, migration safety, and testability. Use native SwiftData APIs first, then add custom data store behavior only when product constraints require it.

## Workflow Decision Tree

### 1) Review existing SwiftData code
- Validate container setup and environment wiring (see `references/fundamentals-and-container.md`)
- Check model definitions, attributes, and relationship rules (see `references/modeling-and-relationships.md`)
- Inspect query strategy and predicate placement (see `references/query-and-predicate-patterns.md`)
- Verify context isolation and background write patterns (see `references/concurrency-and-context-isolation.md`)
- Confirm history processing strategy for cross-process and sync scenarios (see `references/history-and-change-tracking.md`)
- Validate schema evolution and migration staging (see `references/migration-and-versioned-schema.md`)
- Inspect hot paths and memory behavior (see `references/performance-patterns.md`)
- Verify custom store usage is justified and bounded (see `references/custom-datastore.md`)
- Check inheritance usage for clarity and query behavior (see `references/inheritance-and-polymorphism.md`)
- Confirm preview/test container isolation (see `references/testing-and-previews.md`)

### 2) Improve existing SwiftData code
- Consolidate container ownership and remove duplicate container creation
- Add explicit uniqueness/delete rules where domain constraints demand them
- Move filtering/sorting from view code into `@Query` or `FetchDescriptor`
- Separate background imports from UI context using actor-isolated data boundaries
- Introduce history-token checkpointing for incremental processing
- Formalize `VersionedSchema` and migration plan for every schema change
- Batch large reads/writes, reduce over-fetching, and trim memory fan-out
- Replace premature custom store abstraction with default store when requirements are not explicit
- Simplify inheritance hierarchies that do not improve modeling or query clarity
- Add deterministic in-memory tests before changing production migrations

### 3) Implement new SwiftData feature
- Define model invariants first (identity, uniqueness, deletion semantics)
- Design container/configuration topology (single or multiple configurations)
- Choose query API by workload (`@Query` for UI projections, manual fetch for workflows)
- Assign write ownership to one actor boundary per workflow
- Define save cadence and failure policy (retry, partial rollback, telemetry)
- Add history tracking if other processes or sync engines observe changes
- Version schema from day one and prepare migration stages
- Add performance guardrails for large collections and import jobs
- Build preview and test fixtures with isolated in-memory containers

## Core Guidelines

### Modeling and Relationships
- Model domain invariants explicitly with `@Attribute` and `@Relationship`
- Use `@Attribute(.unique)` only for truly global identity constraints
- Choose delete rules intentionally (`.cascade`, `.nullify`, `.deny`) and test them
- Keep relationship graph understandable; avoid hidden cyclic ownership

### Query Strategy
- Use `@Query` for view-driven data that must stay live with UI updates
- Use `FetchDescriptor` for workflow logic (imports, exports, migrations, audits)
- Define sort and filter at fetch time, not in `ForEach` or view rendering closures
- Limit result sets on large datasets and fetch only fields you need

### Context Isolation and Concurrency
- Treat model contexts as actor-bound state, not globally shared mutable state
- Perform heavy writes/imports in dedicated actor workflows
- Pass identifiers or value snapshots across concurrency boundaries, not mutable model objects
- Keep UI updates on the main actor and background writes isolated from rendering

### Save and Transaction Boundaries
- Save intentionally at business milestones, not on every field mutation
- Surface save failures with clear recovery paths
- Group logically related mutations in one transaction boundary
- Avoid hidden implicit saves inside utility helpers

### History and Change Processing
- Persist history tokens after successful processing
- Process changes incrementally in bounded batches
- Separate history ingestion from UI rendering concerns
- Build idempotent consumers so replaying tokens is safe

### Migration and Schema Evolution
- Record every schema revision in `VersionedSchema`
- Prefer lightweight migration when it preserves meaning and constraints
- Use custom migration stages for data transformations and integrity fixes
- Verify migration behavior with deterministic fixtures before release

### Performance
- Avoid unbounded fetches in startup paths
- Batch import and maintenance workloads
- Reduce query fan-out from broad relationships
- Profile memory and lock contention in realistic data volumes

### Testing and Previews
- Use in-memory containers for previews and unit tests
- Seed deterministic fixtures; avoid tests that depend on insertion order side effects
- Add migration tests for each schema step
- Verify relationship cleanup behavior after deletes

## Quick Reference

| API or Type | Use For |
|-------------|---------|
| `@Model` | Declare persistent model types |
| `@Attribute` | Add uniqueness, rename metadata, and field semantics |
| `@Relationship` | Declare relationship, delete rules, and inverse behavior |
| `ModelContainer` | Own schema + storage configuration |
| `ModelConfiguration` | Configure persistence topology and store behavior |
| `ModelContext` | Perform inserts, deletes, and saves within one actor boundary |
| `@Query` | Live, view-driven query results |
| `FetchDescriptor` | Programmatic fetches for workflow logic |
| `SortDescriptor` | Stable query ordering |
| `#Predicate` | Type-safe filter definitions |
| `VersionedSchema` | Version each schema snapshot |
| `SchemaMigrationPlan` | Define migration stages between schema versions |
| `MigrationStage` | Lightweight or custom migration execution |
| `HistoryDescriptor` | Fetch bounded history windows |
| `ModelTransaction` | Inspect transaction-level history changes |
| `DataStore` | Plug in custom persistence backend when required |

## Review Checklist

### Fundamentals and Container
- [ ] Container is created once at app composition boundary
- [ ] Configuration topology (single vs multi) is intentional and documented
- [ ] Environment uses one clear container source of truth

### Modeling and Relationships
- [ ] Identity and uniqueness rules reflect business invariants
- [ ] Delete rules are explicit and tested
- [ ] Inverse relationships are defined where bidirectional navigation is required

### Query and Predicates
- [ ] No inline filtering/sorting in view rendering for large datasets
- [ ] Query API choice (`@Query` vs `FetchDescriptor`) matches workload
- [ ] Fetches are bounded for high-cardinality domains

### Concurrency and Context Isolation
- [ ] Context mutation does not cross actor boundaries unsafely
- [ ] Background workflows do not directly mutate UI-owned models
- [ ] Save ownership is explicit per workflow

### History and Sync
- [ ] History token checkpointing is durable
- [ ] Processing is incremental and idempotent
- [ ] Change handling is separated from presentation logic

### Migration
- [ ] Every schema change has a version entry
- [ ] Lightweight vs custom stage choice is explicit
- [ ] Migration behavior is covered by tests/fixtures

### Performance
- [ ] Unbounded fetch paths are avoided
- [ ] Batch import/write patterns are used for bulk operations
- [ ] Memory pressure and relationship fan-out are reviewed

### Custom Data Store
- [ ] Default store is used unless clear custom backend requirements exist
- [ ] Custom store boundary is isolated and tested
- [ ] Snapshot mapping maintains identity and ordering guarantees

### Inheritance
- [ ] Inheritance is used for natural polymorphism, not convenience
- [ ] Query behavior across base/subclass types is verified
- [ ] Hierarchy depth remains maintainable

### Testing and Previews
- [ ] Previews use in-memory containers
- [ ] Unit tests use deterministic fixtures
- [ ] Migration tests cover real upgrade sequences

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
