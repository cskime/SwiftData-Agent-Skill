# SwiftData Migration and Versioned Schema

## Scope
Version schema evolution explicitly with `VersionedSchema` and `SchemaMigrationPlan`. Choose lightweight stages for structural compatibility and custom stages for semantic transformation.

## Do
- Add a new schema version for every model-breaking change
- Keep migration stages explicit and ordered
- Choose lightweight migration only when semantics remain equivalent
- Add custom migration stages for data transformations and cleanup

## Don't
- Don't edit old schema versions retroactively
- Don't collapse multiple release migrations into one ambiguous stage
- Don't ship schema changes without fixture-based upgrade tests
- Don't rely on manual post-migration cleanup in app UI code

## Wrong vs Correct

```swift
import SwiftData

@Model
final class TripV1 {
    var name: String

    init(name: String) {
        self.name = name
    }
}

@Model
final class TripV2 {
    var name: String
    var startDate: Date

    init(name: String, startDate: Date) {
        self.name = name
        self.startDate = startDate
    }
}

// WRONG: No versioning contract; migration intent is implicit.
// Model edits are made in place with no migration plan.

// CORRECT: Versioned schemas with explicit migration stage.
enum TravelSchemaV1: VersionedSchema {
    static var versionIdentifier: Schema.Version = .init(1, 0, 0)
    static var models: [any PersistentModel.Type] { [TripV1.self] }
}

enum TravelSchemaV2: VersionedSchema {
    static var versionIdentifier: Schema.Version = .init(2, 0, 0)
    static var models: [any PersistentModel.Type] { [TripV2.self] }
}

struct TravelMigrationPlan: SchemaMigrationPlan {
    static var schemas: [any VersionedSchema.Type] {
        [TravelSchemaV1.self, TravelSchemaV2.self]
    }

    static let migrateV1ToV2 = MigrationStage.custom(
        fromVersion: TravelSchemaV1.self,
        toVersion: TravelSchemaV2.self,
        willMigrate: { _ in },
        didMigrate: { context in
            let trips = try context.fetch(FetchDescriptor<TripV2>())
            for trip in trips where trip.startDate == .distantPast {
                trip.startDate = .now
            }
            try context.save()
        }
    )

    static var stages: [MigrationStage] {
        [migrateV1ToV2]
    }
}
```

## Lightweight vs Custom Rule
- Use **lightweight** when renames and additive fields keep meaning intact
- Use **custom** when you must remap values, split/merge fields, or enforce new invariants

## Pitfalls
- Editing old versions breaks reproducibility of historical migrations
- Missing tests for multi-hop upgrades causes release-only failures
- Data transformations without idempotence can corrupt retry scenarios
