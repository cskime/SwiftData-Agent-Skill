# SwiftData Testing and Previews

## Scope
Use isolated, deterministic containers in previews and tests. Validate behavior with repeatable fixtures, including migration and relationship cleanup scenarios.

## Do
- Use in-memory `ModelConfiguration` for previews and unit tests
- Seed fixtures with deterministic values (IDs/timestamps)
- Add upgrade-path tests for migration plans
- Keep test setup centralized to avoid drift

## Don't
- Don't point tests/previews at production store paths
- Don't rely on insertion order side effects for assertions
- Don't hide fixture randomness in helper defaults
- Don't skip delete-rule assertions

## Wrong vs Correct

```swift
import SwiftData
import SwiftUI

@Model
final class Project {
    @Attribute(.unique) var id: UUID
    var name: String

    init(id: UUID = UUID(), name: String) {
        self.id = id
        self.name = name
    }
}

// WRONG: Preview accidentally reuses disk-backed default configuration.
#Preview("Project List Wrong") {
    ProjectListView()
        .modelContainer(for: [Project.self])
}

// CORRECT: Preview uses explicit in-memory container and deterministic fixtures.
#Preview("Project List Correct") {
    guard
        let alphaID = UUID(uuidString: "00000000-0000-0000-0000-000000000001"),
        let betaID = UUID(uuidString: "00000000-0000-0000-0000-000000000002")
    else {
        return Text("Invalid preview fixture IDs")
    }

    do {
        let container = try ModelContainer(
            for: Project.self,
            configurations: ModelConfiguration(isStoredInMemoryOnly: true)
        )

        let context = container.mainContext
        context.insert(Project(id: alphaID, name: "Alpha"))
        context.insert(Project(id: betaID, name: "Beta"))
        try context.save()

        return ProjectListView()
            .modelContainer(container)
    } catch {
        return Text("Preview setup failed: \(error.localizedDescription)")
    }
}
```

## Test Matrix
- CRUD behavior with deterministic fixtures
- Relationship delete rule behavior
- Query ordering stability
- Migration from previous schema versions
- History checkpoint persistence and replay idempotence

## Pitfalls
- Shared mutable fixture helpers introduce cross-test coupling
- Non-deterministic timestamps/UUIDs lead to flaky assertions
- Missing migration tests cause release-time upgrade surprises
