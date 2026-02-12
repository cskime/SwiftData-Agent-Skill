# SwiftData Agent Skill

An expert SwiftData skill for iOS 26+ focused on modeling, querying, context isolation, history tracking, migration, custom data stores, inheritance, performance, and testing.

## Repository Structure

```text
.
├── AGENTS.md
├── CONTRIBUTING.md
├── LICENSE
├── README.md
├── .claude-plugin/
├── .github/
│   └── workflows/
│       └── validate-skill.yml
└── swiftdata-expert-skill/
    ├── SKILL.md
    └── references/
        ├── concurrency-and-context-isolation.md
        ├── custom-datastore.md
        ├── fundamentals-and-container.md
        ├── history-and-change-tracking.md
        ├── inheritance-and-polymorphism.md
        ├── migration-and-versioned-schema.md
        ├── modeling-and-relationships.md
        ├── performance-patterns.md
        ├── query-and-predicate-patterns.md
        ├── review-checklist.md
        └── testing-and-previews.md
```

## Install (Codex)

Copy the skill folder into your local Codex skills directory:

```bash
cp -R swiftdata-expert-skill ~/.codex/skills/swiftdata-expert-skill
```

Then restart Codex.

## Usage

Mention `swiftdata-expert-skill` in your prompt, or ask SwiftData-specific tasks such as:

- Design a migration plan with `VersionedSchema` and `SchemaMigrationPlan`
- Review relationship delete rules and model invariants
- Optimize SwiftData query and context performance
- Add history-token processing for cross-process sync

