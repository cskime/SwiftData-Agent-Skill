# SwiftData Review Checklist

## Scope
Use this checklist for PR review of SwiftData code. Prioritize data integrity, migration safety, concurrency correctness, and predictable performance.

## Do
- Review changes by invariants first: identity, uniqueness, delete semantics
- Validate migration and schema impact before UI-level refinements
- Require deterministic tests for new persistence behavior
- Flag broad fetches and actor isolation leaks early

## Don't
- Don't approve schema changes without version/migration updates
- Don't accept implicit relationship cleanup assumptions
- Don't ignore history token durability in sync flows
- Don't approve performance-sensitive changes without bounded fetch behavior

## Wrong vs Correct

```swift
// WRONG REVIEW OUTCOME
// "Looks fine visually" without checking migration impact.

// CORRECT REVIEW OUTCOME
// 1) Confirm VersionedSchema updated.
// 2) Confirm SchemaMigrationPlan stage exists.
// 3) Confirm fixture-based migration test added.
// 4) Confirm no unbounded fetch introduced in startup path.
```

## PR Review Gates
- [ ] Container ownership and configuration topology are explicit
- [ ] Model changes preserve or intentionally modify data invariants
- [ ] Relationship delete behavior is explicit and tested
- [ ] Query paths are bounded and appropriately sorted
- [ ] Context mutation is actor-isolated
- [ ] History processing is tokenized and idempotent
- [ ] Schema version and migration stage updates are present
- [ ] Custom store changes include integration coverage
- [ ] Inheritance use is domain-driven and test-backed
- [ ] Preview/test fixtures are deterministic

## Pitfalls
- Schema edits that skip migration planning are the highest-risk review miss
- Background write helpers that hide actor crossing are hard to debug post-release
- Performance regressions usually begin with convenient but unbounded fetches
